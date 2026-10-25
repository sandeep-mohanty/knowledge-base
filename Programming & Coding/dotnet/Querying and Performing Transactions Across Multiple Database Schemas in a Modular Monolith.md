# Querying and Performing Transactions Across Multiple Database Schemas in a Modular Monolith

Building a [Modular Monolith](https://antondevtips.com/blog/building-a-modular-monolith-with-vertical-slice-architecture-in-dotnet) gives you clear boundaries between modules, but it also introduces a challenge: how do you query data that lives in different schemas? And how do you maintain data consistency when a business operation spans multiple modules?

In a traditional monolith, you could join tables across the database.

But in a Modular Monolith, each module owns its schema. Direct database access between modules breaks the boundaries you worked hard to establish.

I have been working with Modular Monoliths for years, and I have seen teams struggle with this exact problem. They start with good boundaries, create separate schemas for each module, but then they need to show a report that combines data from three different modules. What do they do?

Some teams give up and start querying across schemas directly. Others try to solve it with complex event chains that are hard to debug. And some just avoid the problem altogether by keeping everything in one module.

But there is a better way.

Today, I want to show you proven approaches for querying data across schemas and strategies for managing transactions in a Modular Monolith.

In this post, we will explore:

-   Why You Can't Join Tables Across Multiple Schemas
-   Recommended Approaches for Cross-Schema Queries
    -   Inter-module API Calls
    -   Domain Events with Eventual Consistency
    -   Database Views
    -   Composite View Pattern (BFF with YARP)
    -   Reporting/Analysis Module
-   Performing Transactions Across Multiple Schemas
    -   Domain Events for Eventual Consistency
    -   Transaction across EF Core DbContexts for Strong Consistency
-   Choosing Between Eventual and Strong Consistency

Let's dive in!

---

## Why You Can't Join Tables Across Multiple Schemas

In a [Modular Monolith](https://antondevtips.com/blog/building-a-modular-monolith-with-vertical-slice-architecture-in-dotnet), each module has its own database schema and DbContext in EF Core. This separation is intentional. It enforces boundaries and makes it possible to extract a module into a microservice later.

But this separation creates two main challenges:

First, you cannot simply join tables across schemas in your queries. If you want to show shipment details along with carrier information and stock levels, you cannot write a single SQL or EF Core query that joins all three schemas.

Second, you cannot use a single database transaction that spans multiple modules. If creating a shipment requires updating stock levels and registering with a carrier, you need a strategy to keep all three operations consistent.

Let me show you a concrete example.

Imagine you need to build a dashboard that shows:

-   All shipments created today
-   The carrier assigned to each shipment
-   Current stock levels for products in those shipments

In a traditional Monolith, you might write something like this:

```csharp
var dashboard = await context.Shipments
    .Include(s => s.Stocks)
    .Include(s => s.Carriers)
    .Where(s => s.CreatedAt.Date == DateTime.UtcNow.Date)
    .Select(x => new
    {
        ShipmentNumber = x.Number,
        CarrierName = x.Carrier.Name,
        StockLevel = x.Stocks.First().Quantity
    })
    .ToListAsync();
```

But in a Modular Monolith, this code will not work. The `ShipmentsDbContext` does not know about `Carriers` or `Stocks` entities. They live in different schemas with different DbContexts.

So how do you solve this?

---

## Recommended Approaches for Cross-Schema Queries

[Copied](#intermodule-api-calls)

### Inter-module API Calls

The most straightforward approach is to call other modules through their public APIs. Each module exposes an interface that other modules can use to query data.

This is the approach is the simplest and the cheapest. It respects module boundaries and makes dependencies explicit.

Let's say you need to display shipment details with carrier information. Here is how you would implement it:

First, the `Carriers` module exposes a public API:

```csharp
public interface ICarrierModuleApi 
{
    Task<CarrierDetailsResponse?> GetCarrierByNameAsync(
        string carrierName, 
        CancellationToken cancellationToken = default);
}
```

Now in the `Shipments` module, you can create a [handler](https://antondevtips.com/blog/refactoring-a-modular-monolith-without-mediatr-in-dotnet) that combines data from multiple modules:

```csharp
internal sealed class GetShipmentDetailsHandler(
    ShipmentsDbContext context,
    ICarrierModuleApi carrierApi,
    IStockModuleApi stockApi) 
{
    public async Task<ErrorOr<ShipmentDetailsResponse>> HandleAsync(
        string shipmentNumber,
        CancellationToken cancellationToken)
    {
        // 1. Get shipment from the local database
        var shipment = await context.Shipments
            .Include(s => s.Items)
            .FirstOrDefaultAsync(s => s.Number == shipmentNumber, cancellationToken);
            
        if (shipment is null)
        {
            return Error.NotFound("ShipmentNotFound", $"Shipment {shipmentNumber} not found");
        }

        // 2. Get carrier details from the Carriers module
        var carrierDetails = await carrierApi.GetCarrierByNameAsync(
            shipment.Carrier, 
            cancellationToken);

        // 3. Get stock levels for each product from the Stocks module
        var stockLevels = new Dictionary<string, int>();
        foreach (var item in shipment.Items)
        {
            var stockResponse = await stockApi.GetStockLevelAsync(item.Product, cancellationToken);
            if (stockResponse.IsSuccess)
            {
                stockLevels[item.Product] = stockResponse.Quantity;
            }
        }

        // 4. Combine all data into the response
        return new ShipmentDetailsResponse
        {
            ShipmentNumber = shipment.Number,
            OrderId = shipment.OrderId,
            Status = shipment.Status.ToString(),
            CreatedAt = shipment.CreatedAt,
            Carrier = carrierDetails,
            Items = shipment.Items.Select(i => new ShipmentItemDetails
            {
                Product = i.Product,
                Quantity = i.Quantity,
                CurrentStockLevel = stockLevels.GetValueOrDefault(i.Product, 0)
            }).ToList()
        };
    }
}
```

**This approach has several advantages:**

-   **Clear boundaries:** each module controls what data it exposes
-   **Type safety:** you work with strongly typed interfaces
-   **Easy to test:** you can mock the module APIs in unit tests
-   **Flexible:** each module can change its internal implementation without affecting others

**But it also has some drawbacks:**

-   **Multiple database queries:** you make separate calls to each module
-   **N+1 query problem:** if you need to enrich a list of shipments with carrier details, you will make one query per shipment

For most scenarios, this approach works well. If you have a few items, the performance overhead is usually acceptable, especially when you add caching or support for bulk operations.

### Domain Events with Eventual Consistency

Sometimes you need to query data that does not change frequently. In these cases, you can duplicate the data across modules using integration events.

This approach works well when you need to denormalize data for read performance.

Let's say you want to display the carrier name on the shipment list without having to call the Carriers module each time. You can store the carrier name directly in the Shipments schema.

When a carrier is updated in the Carriers module, it publishes an event:

```csharp
public sealed record CarrierUpdatedEvent(
    Guid CarrierId,
    string CarrierName,
    string ContactEmail,
    bool IsActive) : IEvent;

internal sealed class UpdateCarrierHandler(
    CarriersDbContext context,
    IEventPublisher eventPublisher)
    : IUpdateCarrierHandler 
{
    public async Task<ErrorOr<CarrierResponse>> HandleAsync(
        UpdateCarrierRequest request,
        CancellationToken cancellationToken)
    {
        var carrier = await context.Carriers
            .FirstOrDefaultAsync(c => c.Id == request.CarrierId, cancellationToken);
            
        if (carrier is null)
        {
            return Error.NotFound("CarrierNotFound", "Carrier not found");
        }

        carrier.ContactEmail = request.ContactEmail;
        carrier.PhoneNumber = request.PhoneNumber;
        carrier.IsActive = request.IsActive;

        // Publish event for other modules
        var carrierUpdatedEvent = new CarrierUpdatedEvent(
            carrier.Id,
            carrier.Name,
            carrier.ContactEmail,
            carrier.IsActive);
            
        await eventPublisher.PublishAsync(carrierUpdatedEvent, cancellationToken);
        await context.SaveChangesAsync(cancellationToken);

        return carrier.MapToResponse();
    }
}
```

The Shipments module subscribes to this event and updates its local copy:

```csharp
internal sealed class UpdateShipmentCarrierDetailsEventHandler(
    ShipmentsDbContext context)
    : IEventHandler<CarrierUpdatedEvent> 
{
    public async Task HandleAsync(
        CarrierUpdatedEvent @event,
        CancellationToken cancellationToken)
    {
        var shipments = await context.Shipments
            .Where(s => s.CarrierName == @event.CarrierName)
            .ToListAsync(cancellationToken);

        foreach (var shipment in shipments)
        {
            shipment.CarrierContactEmail = @event.ContactEmail;
            shipment.UpdatedAt = DateTime.UtcNow;
        }

        await context.SaveChangesAsync(cancellationToken);
    }
}
```

Now you can query shipments with carrier details in the Shipments module without having to call the Carriers module.

**This approach has the following advantages:**

-   **Fast queries:** all data is in one schema, no joins needed
-   **No runtime dependencies:** modules do not need to call each other during queries
-   **Better performance:** single database query instead of multiple calls

**But it comes with trade-offs:**

-   **Eventual consistency:** data might be temporarily out of sync
-   **Data duplication:** you store the same data in multiple places
-   **More complex:** you need to handle events and keep data synchronized

Use this approach when read performance is critical and you can accept eventual consistency. This approach also allows you to change and scale each module independently.

When using events - I highly recommend using [Open Telemetry](https://antondevtips.com/blog/getting-started-with-open-telemetry-in-dotnet-with-jaeger-and-seq) to monitor your application.

### Database Views

Database views provide a way to query data across multiple schemas at the database level. You create a view that joins tables from different schemas, and then map it to a read-only entity in EF Core.

This approach works well for reporting and analytics scenarios where you need to combine data from multiple modules.

Let's create a view that combines shipments with carrier information:

```sql
CREATE VIEW shipments_report_view AS 
SELECT 
    s.Id,
    s.Number,
    s.OrderId,
    s.Status,
    s.CreatedAt,
    c.Name AS CarrierName,
    c.ContactEmail AS CarrierContactEmail,
    c.PhoneNumber AS CarrierPhoneNumber,
    c.IsActive AS CarrierIsActive
FROM Shipments.Shipments s
LEFT JOIN Carriers.Carriers c ON s.Carrier = c.Name
WHERE c.IsActive = 1;
```

You can map this view to a read-only entity in EF Core. Create a separate DbContext for read models:

```csharp
public class ShipmentWithCarrier 
{
    public Guid Id { get; set; }
    public string Number { get; set; }
    public string OrderId { get; set; }
    public string Status { get; set; }
    public DateTime CreatedAt { get; set; }
    public string CarrierName { get; set; }
    public string CarrierContactEmail { get; set; }
    public string CarrierPhoneNumber { get; set; }
    public bool CarrierIsActive { get; set; }
}

public class ReadModelsDbContext(DbContextOptions<ReadModelsDbContext> options)
    : DbContext(options) 
{
    public DbSet<ShipmentWithCarrier> ShipmentsWithCarriers { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        modelBuilder.Entity<ShipmentWithCarrier>()
            .HasNoKey()
            .ToView("shipments_report_view");
    }
}
```

Now you can query the view from your handler:

```csharp
var shipments = await context.ShipmentsWithCarriers
    .OrderByDescending(s => s.CreatedAt)
    .Take(request.PageSize)
    .ToListAsync(cancellationToken);

return shipments.Select(s => new ShipmentWithCarrierResponse 
{
    ShipmentNumber = s.Number,
    OrderId = s.OrderId,
    Status = s.Status,
    CreatedAt = s.CreatedAt,
    CarrierName = s.CarrierName,
    CarrierContactEmail = s.CarrierContactEmail,
    CarrierPhoneNumber = s.CarrierPhoneNumber
}).ToList();
```

**This approach has the following advantages:**

-   **Single query:** the database handles the join, so you get all data in one query
-   **Good performance:** database views are optimized by the query planner
-   **Simple code:** you query the view like a regular table
-   **No application-level joins:** the database does the heavy lifting

**But it has some limitations:**

-   **Breaks module boundaries:** the view directly accesses multiple schemas
-   **Database coupling:** modules are coupled at the database level
-   **Harder to extract to microservices:** you need to remove the view first
-   **Schema changes require view updates:** if you change a table structure, you need to update the view

Use database views for reporting and analytics when performance is critical, and you do not plan to extract modules into microservices soon.

I recommend using a separate database user for the database views with restricted permissions.

### Composite View Pattern (BFF with YARP)

The Composite View Pattern, also known as Backend for Frontend (BFF), involves creating a separate service that aggregates data from multiple modules. This service sits between your frontend and your Modular Monolith.

This approach is particularly useful when you have complex UI requirements that need data from many modules.

In our case, we can create a BFF service that queries multiple modules and combines the results. To access this BFF service, we use YARP as an API Gateway.

YARP (Yet Another Reverse Proxy) is a reverse proxy toolkit from Microsoft that you can use to route requests to different services. I have written a detailed guide on how to set up YARP as an API Gateway. You can read it here: [YARP as API Gateway in .NET](https://antondevtips.com/blog/yarp-as-api-gateway-in-dotnet).

Here is how the architecture looks:

-   Frontend calls YARP Gateway
-   YARP routes requests to either the main Modular Monolith or the BFF service
-   BFF service queries the Modular Monolith modules and aggregates the data
-   BFF returns the combined response to the frontend

Let's create a BFF service that provides a dashboard view:

```csharp
public class ShipmentDashboardService(
    IHttpClientFactory httpClientFactory) 
{
    public async Task<ShipmentDashboardResponse> GetDashboardAsync(
        CancellationToken cancellationToken)
    {
        var client = httpClientFactory.CreateClient("ModularMonolith");

        // Query shipments from the Shipments module
        var shipmentsResponse = await client.GetAsync(
            "/api/shipments?pageSize=10", 
            cancellationToken);
        var shipments = await shipmentsResponse.Content
            .ReadFromJsonAsync<List<ShipmentResponse>>(cancellationToken);

        // Query carriers from the Carriers module
        var carriersResponse = await client.GetAsync(
            "/api/carriers", 
            cancellationToken);
        var carriers = await carriersResponse.Content
            .ReadFromJsonAsync<List<CarrierResponse>>(cancellationToken);

        // Query stock levels from the Stocks module
        var stocksResponse = await client.GetAsync(
            "/api/stocks/summary", 
            cancellationToken);
        var stockSummary = await stocksResponse.Content
            .ReadFromJsonAsync<StockSummaryResponse>(cancellationToken);

        // Combine all data into a dashboard view
        return new ShipmentDashboardResponse
        {
            // ... logic to map fields
        };
    }
}
```

The BFF service exposes its own API endpoint:

```csharp
public class DashboardEndpoint : ICarterModule 
{
    public void AddRoutes(IEndpointRouteBuilder app)
    {
        app.MapGet("/api/bff/dashboard", Handle);
    }

    private static async Task<IResult> Handle(
        ShipmentDashboardService dashboardService,
        CancellationToken cancellationToken)
    {
        var dashboard = await dashboardService.GetDashboardAsync(cancellationToken);
        return Results.Ok(dashboard);
    }
}
```

**This approach has the following advantages:**

-   **Separation of concerns:** the BFF handles UI-specific data aggregation
-   **Reduced frontend complexity:** the frontend makes one call instead of many
-   **Optimized for UI:** you can shape the response exactly as the UI needs it
-   **Independent scaling:** you can scale the BFF separately from the main application
-   **Better performance for frontend:** fewer HTTP calls from the browser

**But it has some drawbacks:**

-   **Additional service:** you need to deploy and maintain another application
-   **Network overhead:** the BFF makes HTTP calls to the main application
-   **Duplication:** you might duplicate some logic between the BFF and the main application

Use the BFF pattern when you have complex UI requirements that need data from many modules, or when you want to optimize the API for specific frontend needs.

**P.S.:** For BFF, I can highly recommend using GraphQL. It simplifies a lot of things. If you are interested in GraphQL, you can read my article on [HotChocolate GraphQL in .NET](https://antondevtips.com/blog/getting-started-with-hot-chocolate-graphql).

### Reporting/Analysis Module

The final approach is to create a dedicated Reporting or Analysis module that has permission to query multiple schemas. This module is the only one allowed to break the boundary rules.

This approach works well when you need complex reporting or analytics that require data from multiple modules.

The key is to enforce this rule with architecture tests. You can use tools like NetArchTest to ensure that only the Reporting module can access multiple schemas.

I have written a detailed guide on architecture tests. You can read it here: [Why Do You Need to Write Architecture Tests in .NET](https://antondevtips.com/blog/why-do-you-need-to-write-architecture-tests-in-dotnet).

With this approach you can create complex reports that join data from multiple modules.

To ensure that only the Reporting module can access multiple schemas, use separate database users for each module. Configure connection strings with specific permissions:

```json
{
  "ConnectionStrings": {
    "Shipments": "Server=localhost;Database=ModularMonolith;User Id=shipments_user;Password=***;",
    "Carriers": "Server=localhost;Database=ModularMonolith;User Id=carriers_user;Password=***;",
    "Stocks": "Server=localhost;Database=ModularMonolith;User Id=stocks_user;Password=***;",
    "Reporting": "Server=localhost;Database=ModularMonolith;User Id=reporting_user;Password=***;"
  }
}
```

Grant the reporting user read access to all schemas, while other users only have access to their own schemas.

**This approach has the following advantages:**

-   **Powerful queries:** you can write complex SQL joins across all schemas
-   **Good performance:** single database query with proper indexes
-   **Centralized reporting:** all reporting logic is in one place

**But it has some limitations:**

-   **Breaks module boundaries:** the Reporting module knows about other modules' schemas
-   **Database coupling:** modules are coupled at the database level
-   **Requires discipline:** without architecture tests you can break the module boundaries
-   **Schema changes impact reporting:** if you change a table structure, you need to update reports

Use this approach when you need complex reporting and analytics, and you can accept that one module has special privileges.

---

## Performing Transactions Across Multiple Schemas

Querying data across schemas is one challenge, but maintaining data consistency when you modify data in multiple modules is another.

When you create a shipment, you need to update stock levels and register with a carrier. All three operations should succeed or fail together. But each module has its own DbContext and schema.

How do you ensure consistency?

You have two main strategies: eventual consistency with domain events, or strong consistency with a shared transaction.

### Domain Events for Eventual Consistency

The first strategy is to use domain events. One module completes its local transaction and publishes an event. Other modules subscribe to this event and perform their own local transactions in response.

This results in eventual consistency. All parts of the system will eventually be consistent, but they might be temporarily out of sync.

Let's see how this works with the CreateShipment use case:

```csharp
internal sealed class CreateShipmentHandler(
    ShipmentsDbContext context,
    IStockModuleApi stockApi,
    IEventPublisher eventPublisher)
    : ICreateShipmentHandler 
{
    public async Task<ErrorOr<ShipmentResponse>> HandleAsync(
        CreateShipmentRequest request,
        CancellationToken cancellationToken)
    {
        // 1. Check if the shipment already exists
        // 2. Check stock levels (read-only operation)
        // 3. Save the shipment in the local database
        
        var shipment = new Shipment { /* ... initialization ... */ };
        await context.Shipments.AddAsync(shipment, cancellationToken);
        
        // 4. Publish an event for other modules to react
        var shipmentCreatedEvent = new ShipmentCreatedEvent(/* ... parameters ... */);
        await eventPublisher.PublishAsync(shipmentCreatedEvent, cancellationToken);
        
        await context.SaveChangesAsync(cancellationToken);
        return shipment.MapToResponse();
    }
}
```

Now the Carriers and Stocks module subscribes to the `ShipmentCreatedEvent`.

**This approach has the following advantages:**

-   **Loose coupling:** modules do not depend on each other directly
-   **Resilience:** if one module fails, others can continue
-   **Scalability:** you can process events asynchronously
-   **Easy to add new handlers:** you can add new modules that react to the same event

**But it has some challenges:**

-   **Eventual consistency:** data might be temporarily out of sync
-   **Error handling:** if an event handler fails, you need a retry mechanism
-   **Debugging:** it is harder to trace the flow of execution
-   **Complexity:** you need to handle partial failures and compensating transactions

For better reliability, you should implement the [Outbox pattern](https://antondevtips.com/blog/use-masstransit-to-implement-outbox-pattern-with-ef-core-and-mongodb). This pattern stores events in the same database transaction as your business data, and then publishes them in a separate process. This ensures that events are never lost.

In a production system, you might use a robust event messaging bus such as [RabbitMQ](https://antondevtips.com/blog/masstransit-rabbitmq-and-azure-service-bus-is-it-worth-a-commercial-license).

### TransactionManager for Strong Consistency

The second strategy is to use a shared transaction across multiple DbContexts. This gives you strong consistency: either all operations succeed, or all fail together.

To implement this, you need a TransactionManager that coordinates the transaction across multiple modules.

Here is the TransactionManager implementation:

```csharp
public interface ITransactionManager 
{
    IDbContextTransaction? CurrentTransaction { get; }
    void SetTransaction(IDbContextTransaction transaction);
    Task CommitTransactionAsync(CancellationToken cancellationToken = default);
    Task RollbackTransactionAsync(CancellationToken cancellationToken = default);
}

public class TransactionManager : ITransactionManager 
{
    private IDbContextTransaction? _currentTransaction;
    public IDbContextTransaction? CurrentTransaction => _currentTransaction;

    public void SetTransaction(IDbContextTransaction transaction)
    {
        _currentTransaction = transaction;
    }

    public async Task CommitTransactionAsync(CancellationToken cancellationToken = default)
    {
        if (_currentTransaction == null)
        {
            throw new InvalidOperationException("No transaction in progress");
        }

        try
        {
            await _currentTransaction.CommitAsync(cancellationToken);
        }
        finally
        {
            _currentTransaction.Dispose();
            _currentTransaction = null;
        }
    }

    public async Task RollbackTransactionAsync(CancellationToken cancellationToken = default)
    {
        if (_currentTransaction == null)
        {
            return;
        }

        try
        {
            await _currentTransaction.RollbackAsync(cancellationToken);
        }
        finally
        {
            _currentTransaction.Dispose();
            _currentTransaction = null;
        }
    }
}
```

Register the TransactionManager as a scoped service:

```csharp
builder.Services.AddScoped<ITransactionManager, TransactionManager>();
```

Now update your DbContexts to use the shared transaction:

```csharp
public class ShipmentsDbContext : DbContext 
{
    private readonly ITransactionManager _transactionManager;
    
    public ShipmentsDbContext(
        DbContextOptions<ShipmentsDbContext> options,
        ITransactionManager transactionManager) : base(options)
    {
        _transactionManager = transactionManager;
    }

    public DbSet<Shipment> Shipments { get; set; }
    public DbSet<ShipmentItem> ShipmentItems { get; set; }

    public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // If there is a shared transaction, use it
        if (_transactionManager.CurrentTransaction != null && Database.CurrentTransaction == null)
        {
            await Database.UseTransactionAsync(
                _transactionManager.CurrentTransaction.GetDbTransaction(), cancellationToken);
        }
        return await base.SaveChangesAsync(cancellationToken);
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.HasDefaultSchema(DbConsts.ShipmentsSchemaName);
    }
}
```

Do the same for `CarriersDbContext` and `StocksDbContext`.

Now you can use the TransactionManager in your handler:

```csharp
public async Task<ErrorOr<ShipmentResponse>> HandleAsync(
    CreateShipmentRequest request,
    CancellationToken cancellationToken) 
{
    // Start a shared transaction
    var transaction = await shipmentsContext.Database.BeginTransactionAsync(cancellationToken);
    transactionManager.SetTransaction(transaction);

    // ... perform logic
             
    await stocksContext.SaveChangesAsync(cancellationToken);
    await shipmentsContext.SaveChangesAsync(cancellationToken);
    await carriersContext.SaveChangesAsync(cancellationToken);

    await transactionManager.CommitTransactionAsync(cancellationToken);

    return shipment.MapToResponse();
}
```

**This approach has the following advantages:**

-   **Strong consistency:** all operations succeed or fail together
-   **ACID guarantees:** you get the full benefits of database transactions
-   **Simpler error handling:** if anything fails, everything rolls back
-   **Easier to reason about:** the flow is linear and predictable

**But it has some drawbacks:**

-   **Tight coupling:** the handler needs to know about all DbContexts
-   **Breaks module boundaries:** you are directly accessing other modules' DbContexts
-   **Harder to extract to microservices:** you need to remove the coupled transaction

Use this approach when you absolutely need strong consistency and cannot accept any temporary inconsistency.

I recommend using a separate database user for this kind of TransactionManager that is not available to other modules.

---

## Choosing Between Eventual and Strong Consistency

Now that you have seen both approaches, how do you decide which one to use?

The answer depends on your business requirements. Let me walk you through some practical scenarios.

### When to Use Eventual Consistency

Eventual consistency with domain events is the better choice in most scenarios. Here is when you should use it:

**Scenario 1: Order Processing**
When a customer places an order, you create a shipment, update stock levels, and notify the carrier. If the carrier notification fails, it does not affect the customer's order. You can retry the notification later. In this case, eventual consistency is acceptable.

**Scenario 2: Analytics and Reporting**
When you update a shipment status, you might want to update analytics data in a separate module. The analytics data does not need to be updated immediately.

**Scenario 3: Sending Notifications**
When a shipment is created, you want to send an email to the customer. If the email service is temporarily down, you do not want to fail the entire shipment creation.

**Scenario 4: Cross-Module Data Synchronization**
When carrier information changes, you want to update the denormalized carrier data in the Shipments module. This update can happen a few seconds later.

The key insight is this: if a failure in one module should not prevent the main operation from succeeding, use eventual consistency.

### When to Use Strong Consistency

Strong consistency with a shared transaction is necessary when you absolutely cannot accept any temporary inconsistency.

**Scenario 1: Financial Transactions**
When you process a payment, you need to deduct money from one account and add it to another. Both operations must succeed or fail together.

**Scenario 2: Inventory Reservation**
When a customer adds a product to their cart and proceeds to checkout, you need to reserve the inventory. If the reservation fails, the checkout should fail as well.

**Scenario 3: Booking Systems**
When a customer books a hotel room or a flight seat, you need to mark it as unavailable immediately. You cannot have two customers booking the same room or seat.

---

## Summary

In practice, you will often use a hybrid approach. Use strong consistency for critical operations like inventory updates and financial transactions. Use eventual consistency for non-critical operations, such as notifications and analytics.

The key is to understand your business requirements and choose the right approach for each scenario. Do not default to strong consistency everywhere.

Eventual consistency is often good enough and gives you better resilience and scalability.

Remember that a Modular Monolith is about maintaining clear boundaries while keeping everything in a single deployable unit. These patterns help you maintain those boundaries while still allowing modules to work together effectively.

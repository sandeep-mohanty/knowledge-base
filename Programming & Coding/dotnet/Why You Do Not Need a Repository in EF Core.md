# Why You Don't Need a Repository in EF Core

Most senior developers will tell you to wrap EF Core inside your own repository interfaces.

But have you ever wondered: why do we need a repository on top of a repository?

In real-world projects, this advice often results in writing a lot of redundant boilerplate code and leads to over-engineered solutions.

Each feature now takes more time to implement and maintain than it should.

There is a better way.

In this post, we will explore:

-   Why You Don't Need a Repository in EF Core
-   How to Use EF Core Without a Repository
-   Best Practices for EF Core in:
    -   N-Layered Applications
    -   Clean Architecture (pragmatic version)
    -   Vertical Slice Architecture
-   Using Specification Pattern for Query Reuse
-   How This Approach Affects Testability
-   When You Might Still Need a Repository

Let's dive in!

[](#why-you-dont-need-a-repository-in-ef-core)

## Why You Don't Need a Repository in EF Core

One of the most common problems is that Repositories tend to grow rapidly as business requirements evolve.

When your application is small, using the Repository Pattern seems easy.

What starts as a simple CRUD operation with 4 methods quickly becomes a large class with database read and write queries for all possible cases.

As your domain grows, you face a critical decision: **should you create a Repository for each entity?**

Every new business requirement means adding another method to the Repository. Over time, you end up with classes full of similar methods:

```csharp
public class ShipmentRepository 
{     
    public Task<List<Shipment>> GetShipmentsByOrder(int userId) { ... }     
    public Task<List<Shipment>> GetCancelledPosts() { ... }     
    public Task<List<Shipment>> GetDeliveredShipmentsByCategory(string category) { ... }     
    public Task<List<Shipment>> GetRecentShipments(int daysBack) { ... }     
    // ...and many more! 
}
```

It gets harder to find the correct method or even remember what's already in every repository.

What if you have multiple entities?

When dealing with `Shipments`, `ShipmentItems`, and `Orders`, following the traditional approach leads to:

```csharp
public interface IShipmentRepository 
{     
    Task<ShipmentDto> GetByIdAsync(int id);     
    Task<IEnumerable<ShipmentDto>> GetAllAsync();     
    // ... 
} 

public interface IShipmentItemRepository 
{     
    Task<ShipmentItemDto> GetByIdAsync(int id);     
    Task<IEnumerable<ShipmentItemDto>> GetByShipmentIdAsync(int shipmentId);     
    // ... 
} 

public interface IOrderRepository 
{     
    Task<OrderDto> GetByIdAsync(int id);     
    Task<IEnumerable<OrderDto>> GetByUserIdAsync(int userId);     
    // ... 
}
```

But what happens when you need to fetch related entities together? For example:

-   Get a shipment with all its items
-   Get an order with its associated shipments
-   Get shipment historical data that loads multiple entities

**Where do these cross-entity methods belong?**

Many developers end up with a lot of repositories that don't do enough. And when you implement a new feature, you start thinking about where to add a new method in one of N repositories.

[...]

## How to Use EF Core Without a Repository

EF Core's DbContext already implements the Repository and Unit of Work patterns, as stated in the official DbContext's code summary.

When we create a repository over EF Core, we create an abstraction over an abstraction, leading to over-engineered solutions.

Each `DbSet<TEntity>` in your DbContext represents a collection of entities, just like a typical repository.

It allows you to:

-   Query data using LINQ
-   Add, update, and remove entities
-   Project data to other types

When you need to find all shipments for an order, you write code like this:

```csharp
var shipments = await dbContext.Shipments
    .Include(s => s.Items)
    .Where(s => s.OrderId == orderId)
    .ToListAsync();
```

What if you have a more complex query? You can always extract it as an extension method to `DbSet<Shipment>` and reuse it more conveniently:

```csharp
var shipments = await dbContext.Shipments
    .GetActiveShipmentsForOrder(orderId)
    .ToListAsync();
```

### Inject DbContext in Your Services or Use Cases

Instead of injecting `IShipmentRepository`, `IShipmentItemRepository`, and `IOrderRepository`, just inject DbContext.

```csharp
internal sealed class CreateShipmentCommandHandler(
    ShipmentsDbContext context,
    ILogger<CreateShipmentCommandHandler> logger)
    : IRequestHandler<CreateShipmentCommand, ErrorOr<ShipmentResponse>>
{
    public async Task<ErrorOr<ShipmentResponse>> Handle(
        CreateShipmentCommand request,
        CancellationToken cancellationToken)
    {
        var shipmentExists = await context.Shipments
            .AnyAsync(x => x.OrderId == request.OrderId, cancellationToken);
        if (shipmentExists)
        {
            return Error.Conflict("Shipment already exists");
        }

        var shipment = request.MapToShipment();
        await context.Shipments.AddAsync(shipment, cancellationToken);
        await context.SaveChangesAsync(cancellationToken);
        return shipment.MapToResponse();
    }
}
```

## Using EF Core Directly in N-Layered Applications

Here is a classic implementation example of `ShipmentService`:

```csharp
public sealed class ShipmentService(
    IShipmentRepository repository,
    ILogger<ShipmentServiceWithRepo> logger)
{
    public async Task<ErrorOr<ShipmentResponse>> CreateAsync(
        CreateShipmentCommand request,
        CancellationToken token = default)
    {
        var alreadyExists = await repository.ExistsForOrderAsync(request.OrderId, token);
        if (alreadyExists)
        {
            logger.LogInformation("Shipment for order '{OrderId}' already exists", request.OrderId);
            return Error.Conflict($"Shipment for order '{request.OrderId}' is already created");
        }

        var shipmentNumber = new Faker().Commerce.Ean8();
        var shipment = request.MapToShipment(shipmentNumber);
        await repository.AddAsync(shipment, token);
        await repository.SaveChangesAsync(token);
        logger.LogInformation("Created shipment: {@Shipment}", shipment);
        return shipment.MapToResponse();
    }

    public async Task<ErrorOr<ShipmentResponse>> GetByIdAsync(
        Guid shipmentId,
        CancellationToken token = default)
    {
        var shipment = await repository.GetByIdAsync(shipmentId, token);
        if (shipment is null)
        {
            return Error.NotFound($"Shipment '{shipmentId}' not found");
        }
        return shipment.MapToResponse();
    }
}
```

Calling infrastructure services doesn't mean you need to hide EF Core behind a repository. Instead, you can inject your DbContext directly into application services or handlers:

```csharp
public sealed class ShipmentService(
    ShipmentsDbContext context,
    ILogger<ShipmentService> logger)
{
    public async Task<ErrorOr<ShipmentResponse>> CreateAsync(
        CreateShipmentCommand request,
        CancellationToken token = default)
    {
        var shipmentAlreadyExists = await context.Shipments
            .AnyAsync(x => x.OrderId == request.OrderId, token);
        if (shipmentAlreadyExists)
        {
            logger.LogInformation("Shipment for order '{OrderId}' is already created", request.OrderId);
            return Error.Conflict($"Shipment for order '{request.OrderId}' is already created");
        }

        var shipmentNumber = new Faker().Commerce.Ean8();
        var shipment = request.MapToShipment(shipmentNumber);
        await context.Shipments.AddAsync(shipment, token);
        await context.SaveChangesAsync(token);
        logger.LogInformation("Created shipment: {@Shipment}", shipment);
        return shipment.MapToResponse();
    }

    public async Task<ErrorOr<ShipmentResponse>> GetByIdAsync(
        Guid shipmentId,
        CancellationToken token = default)
    {
        var shipment = await context.Shipments
            .Include(s => s.Items)
            .FirstOrDefaultAsync(s => s.Id == shipmentId, token);
        if (shipment is null)
        {
            return Error.NotFound($"Shipment '{shipmentId}' not found");
        }
        return shipment.MapToResponse();
    }
}
```

## Using EF Core Directly in Clean Architecture

Here is how the `CreateShipmentCommandHandler` changes when we get rid of repositories:

```csharp
internal sealed class CreateShipmentCommandHandler(
    ShipmentsDbContext context,
    ILogger<CreateShipmentCommandHandler> logger)
    : IRequestHandler<CreateShipmentCommand, ErrorOr<ShipmentResponse>>
{
    public async Task<ErrorOr<ShipmentResponse>> Handle(
        CreateShipmentCommand request,
        CancellationToken cancellationToken)
    {
        var shipmentAlreadyExists = await context.Shipments
            .AnyAsync(x => x.OrderId == request.OrderId, cancellationToken);
        if (shipmentAlreadyExists)
        {
            logger.LogInformation("Shipment for order '{OrderId}' is already created", request.OrderId);
            return Error.Conflict($"Shipment for order '{OrderId}' is already created");
        }

        var shipmentNumber = new Faker().Commerce.Ean8();
        var shipment = request.MapToShipment(shipmentNumber);
        await context.Shipments.AddAsync(shipment, cancellationToken);
        await context.SaveChangesAsync(cancellationToken);
        logger.LogInformation("Created shipment: {@Shipment}", shipment);
        var response = shipment.MapToResponse();
        return response;
    }
}
```

## Using the Specification Pattern with EF Core

Here is an example of a Specification that returns viral posts in a social media application with at least 150 likes:

```csharp
public class ViralPostSpecification : Specification<Post> 
{     
    public ViralPostSpecification(int minLikesCount = 150)     
    {         
        AddFilteringQuery(post => post.Likes.Count >= minLikesCount);         
        AddOrderByDescendingQuery(post => post.Likes.Count);     
    } 
}
```

## Testability with EF Core

### Option 1: Use EF Core InMemory Provider

```csharp
var options = new DbContextOptionsBuilder<ShipmentsDbContext>()
    .UseInMemoryDatabase("ShipmentsTestDb")
    .Options; 

using var context = new ShipmentsDbContext(options);

// Arrange
context.Shipments.Add(new Shipment { OrderId = Guid.NewGuid(), Address = "Berlin" });
await context.SaveChangesAsync();

// Act
var shipment = await context.Shipments.FirstOrDefaultAsync();

// Assert
Assert.NotNull(shipment);
```

---

In modern .NET applications, using EF Core directly in your application layer or vertical slices is often the cleanest, simplest, and most pragmatic choice.

Repositories are not dead — but they're no longer the default. Use them only when they truly add value.

There is no single correct way to write the software; you need to pick whatever works best in each particular project and case.

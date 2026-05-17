# .NET Architecture Without the Cargo Cult: Hard-Won Lessons From 18 Years of Shipping Software

> *"Best practices" applied without judgment aren't best practices — they're rituals. Here's how to think about .NET architecture decisions instead of just following them.*

---

## The Problem With Dogma

Eighteen years of writing .NET code teaches you one thing above all else: the most dangerous phrase in software engineering is "we do it this way because that's the best practice."

Best practices are context-dependent. A pattern that rescued a 200-person team building an enterprise platform will strangle a five-person startup building an MVP. A pattern that makes a financial ledger rock-solid will make a simple CRUD app an unmaintainable maze of indirection.

This isn't an argument against architecture. It's an argument against architecture on autopilot — applying patterns as rituals rather than as deliberate responses to real problems.

What follows are the lessons that took years of production incidents, failed refactorings, and salvage operations on "clean" codebases to learn.

---

## Name Your Conditions — Always

The smallest pattern with the largest daily impact.

```csharp
// What does this actually mean?
if ((user.IsEnabled && !user.IsLockedOut) &&
   (doc.IsPublic || user.IsAdmin || (user.DepartmentId == doc.DepartmentId && !doc.IsArchived)))
{
   // ...
}
```

At 2 AM during an incident, this condition requires you to mentally re-execute the business rules from scratch. Every time. What's "locked out"? What's the relationship between department access and archive status?

```csharp
// This reads like requirements, not code
if (IsUserAuthenticated(user) && CanUserAccessDocument(user, doc))
{
   // ...
}

private static bool IsUserAuthenticated(User user) =>
   user.IsEnabled && !user.IsLockedOut;

private static bool CanUserAccessDocument(User user, Document doc) =>
   doc.IsPublic ||
   user.IsAdmin ||
   (user.DepartmentId == doc.DepartmentId && !doc.IsArchived);
```

Three benefits stack here simultaneously. The if statement reads like a sentence. Each condition can be unit tested in complete isolation — no need to construct elaborate object graphs to hit a specific branch. And when the business rule for document access changes, you change it in exactly one place, with a method name that makes the intent unmistakable.

This costs nothing. Do it everywhere.

---

## Introduce Interfaces When the Pain Is Real, Not Hypothetical

The `IWhateverService` reflex — wrapping every concrete class in an interface before writing the first implementation — is one of the most pervasive patterns in .NET codebases, and one of the least justified.

The argument is always some variation of: *"What if we need to swap it out later?"*

In practice: you almost never do. And when you do, extracting an interface takes 30 seconds in any modern IDE.

**Start with a concrete, sealed class:**

```csharp
public sealed class SmsSender
{
   private readonly TwilioClient _client;

   public SmsSender(TwilioClient client) => _client = client;

   public async Task SendAsync(string to, string message)
   {
       await _client.Messages.CreateAsync(
           body: message,
           from: new PhoneNumber("+15551234567"),
           to: new PhoneNumber(to)
       );
   }
}
```

Your services depend on `SmsSender`. This is fine. Now you write unit tests for a service that sends SMS. You don't want to hit Twilio and pay per message. The pain is now real and specific. Extract the interface:

```csharp
public interface ISmsSender
{
   Task SendAsync(string to, string message);
}

public sealed class SmsSender : ISmsSender { /* ... */ }
public sealed class FakeSmsSender : ISmsSender
{
   public List<(string To, string Message)> SentMessages { get; } = new();
   public Task SendAsync(string to, string message)
   {
       SentMessages.Add((to, message));
       return Task.CompletedTask;
   }
}
```

You now have an interface with a real purpose: it exists to allow a test double. Not as a ritual. Not as insurance against a hypothetical future swap. As a response to an actual need.

**The rule:** add an interface when you have, or are about to have, a second real implementation — whether that's a test double, a stub, or an alternative production implementation. Not before.

---

## The 2 AM Principle: Readability on Your Critical Path

Every codebase has paths that matter more than others. The endpoint that processes payments. The one that handles authentication. The one that, if it fails at 3 AM, wakes someone up.

For those paths, there is one overriding architectural goal: **a tired, stressed on-call developer should be able to read the code top to bottom and understand the complete flow of a request without jumping between files.**

The classic layered architecture fails this test. Consider a controller that calls a service that calls a repository — three files just to follow one request. Finding where `SaveChangesAsync` is called, what gets committed when, and in what order side effects happen requires mental mapping across layers that wasn't designed for operational clarity.

**The before: logic scattered by architecture**

```csharp
// File 1: Controller delegates immediately
[HttpPost("/orders")]
public async Task<IActionResult> Create(CreateOrderRequest request)
{
   var result = await _orderService.CreateAsync(request);
   return result.Success ? Ok(result.OrderId) : BadRequest(result.Errors);
}

// File 2: Service delegates further
public async Task<OrderResult> CreateAsync(CreateOrderRequest request)
{
   if (!await _inventoryService.IsInStockAsync(request.ProductId, request.Quantity))
       return OrderResult.Fail("Item not in stock.");

   var order = new Order(request.ProductId, request.Quantity, request.Total);

   var paymentSuccess = await _paymentGateway.ChargeAsync(request.CreditCard, order.Total);
   if (!paymentSuccess)
       return OrderResult.Fail("Payment failed.");

   await _orderRepository.AddAsync(order);
   // Where is SaveChangesAsync? Is it in the repository? An interceptor?
   // You have to open a third file to find out.

   return OrderResult.Succeed(order.Id);
}
```

**The after: the whole story in one place**

```csharp
app.MapPost("/orders", async (
   CreateOrderRequest req,
   AppDbContext db,
   IInventoryService inventory,
   IPaymentGateway payments,
   ILogger<Program> logger) =>
{
   var validation = new CreateOrderRequestValidator().Validate(req);
   if (!validation.IsValid)
       return Results.ValidationProblem(validation.ToDictionary());

   logger.LogInformation("Starting order creation for product {ProductId}", req.ProductId);

   if (!await inventory.IsInStockAsync(req.ProductId, req.Quantity))
   {
       logger.LogWarning("Product {ProductId} out of stock", req.ProductId);
       return Results.BadRequest(new { Error = "Item not in stock." });
   }

   var order = new Order(req.ProductId, req.Quantity, req.Total);
   db.Orders.Add(order);

   var paymentSuccess = await payments.ChargeAsync(req.CreditCard, order.Total);
   if (!paymentSuccess)
   {
       logger.LogError("Payment failed for order {OrderId}", order.Id);
       return Results.BadRequest(new { Error = "Payment failed." });
   }

   // The commit. Right here. Unmissable.
   await db.SaveChangesAsync();

   logger.LogInformation("Order {OrderId} created successfully", order.Id);
   return Results.Ok(new { OrderId = order.Id });
});
```

Every step is visible. The inventory check, the payment charge, the database commit, the logging — a developer reading this at 2 AM has the complete picture in one scroll.

**Caveat:** this is appropriate for the critical, high-traffic paths where operational clarity outweighs decoupling. It is not a license to put business logic in HTTP handlers generally. The moment you're encoding domain rules — complex invariants, multi-step state transitions, calculations — inside a `MapPost` lambda, extract that logic into a testable class. HTTP handlers handle HTTP. Business logic belongs in a dedicated layer.

---

## Stop Fighting EF Core With Generic Repositories

The `IRepository<T>` pattern made sense before modern ORMs existed. Against EF Core, it's an active anti-pattern that discards the ORM's most powerful feature.

EF Core's `DbSet<T>` *is* a repository. `DbContext` *is* a unit of work. Wrapping them in `IRepository<T>` forces a choice between two bad options:

**Option A — Return `IQueryable<T>`:** You've now leaked your data access technology through your abstraction. The service layer builds LINQ expressions. You haven't hidden the ORM — you've just renamed it.

**Option B — Return `IEnumerable<T>` or `List<T>`:** Every query materializes the full dataset into application memory. For a product search across a million-row table, this loads all million products, then filters in C#.

```csharp
// What this looks like in practice — a performance disaster
public async Task<List<ProductSearchResultDto>> SearchProductsAsync(ProductSearchQuery query)
{
   var allProducts = await _productRepository.ListAllAsync(); // Loads EVERYTHING

   return allProducts  // Filtering happens in memory, in C#
       .Where(p => query.CategoryId == null || p.CategoryId == query.CategoryId)
       .Where(p => query.MaxPrice == null || p.Price <= query.MaxPrice)
       .Where(p => string.IsNullOrEmpty(query.SearchTerm) || p.Name.Contains(query.SearchTerm))
       .Select(p => new ProductSearchResultDto { /* ... */ })
       .ToList();
}
```

**The EF Core approach — let the database do the work:**

```csharp
public async Task<List<ProductSearchResultDto>> SearchProductsAsync(
   ProductSearchQuery query,
   CancellationToken ct = default)
{
   var q = _context.Products.AsQueryable();

   // Each condition adds to the SQL WHERE clause — nothing runs yet
   if (query.CategoryId.HasValue)
       q = q.Where(p => p.CategoryId == query.CategoryId.Value);

   if (query.MaxPrice.HasValue)
       q = q.Where(p => p.Price <= query.MaxPrice.Value);

   if (!string.IsNullOrWhiteSpace(query.SearchTerm))
       q = q.Where(p => p.Name.Contains(query.SearchTerm));

   // Project to DTO in the query — EF Core translates to SQL SELECT
   return await q
       .Select(p => new ProductSearchResultDto
       {
           Id = p.Id,
           Name = p.Name,
           Price = p.Price,
           CategoryName = p.Category.Name
       })
       .ToListAsync(ct);
}
```

EF Core translates this entire chain — every `.Where()`, every `.Select()`, the navigation property join — into a single optimized SQL query. The database filters, projects, and joins. Only the matching rows, with only the needed columns, travel over the network to your application.

```sql
SELECT p.Id, p.Name, p.Price, c.Name AS CategoryName
FROM Products p
INNER JOIN Categories c ON p.CategoryId = c.Id
WHERE p.CategoryId = @categoryId
 AND CHARINDEX(@searchTerm, p.Name) > 0
```

The `IQueryable` expression tree translator is EF Core's defining feature. The generic repository pattern throws it away. Use `DbContext` directly in your application services or MediatR handlers, and let the ORM do what it was designed to do.

---

## Base Class Hell: Composition Is the Way Out

It starts innocently:

```csharp
public abstract class BaseController : ControllerBase
{
   protected Guid CurrentUserId =>
       Guid.Parse(User.Claims.First(c => c.Type == "sub").Value);
}
```

Then a logger gets added. Then feature flags. Then subscription checks. Then audit logging. Six months later:

```csharp
// A lie masquerading as a base class
public abstract class BaseController : ControllerBase
{
   private ILogger _logger;
   private IFeatureFlagService _featureFlags;
   private ISubscriptionService _subscriptions;
   private IAuditService _audit;

   protected ILogger Logger =>
       _logger ??= HttpContext.RequestServices.GetRequiredService<ILogger>();
   protected IFeatureFlagService FeatureFlags =>
       _featureFlags ??= HttpContext.RequestServices.GetRequiredService<IFeatureFlagService>();
   // ...

   protected async Task<bool> CheckSubscriptionLevel(string level) { /* ... */ }
   protected void LogAuditEvent(string action) { /* ... */ }
}
```

This is broken in three distinct ways:

**It lies about dependencies.** A `ProductsController` that inherits this base class implicitly depends on subscription services it never uses. Looking at the constructor tells you nothing about what the class actually needs.

**It uses the Service Locator anti-pattern.** `HttpContext.RequestServices.GetRequiredService<T>()` is dependency injection's nemesis — it hides dependencies at runtime rather than declaring them at construction time. You can't know what a class needs by reading it.

**It's untestable.** Unit testing any controller requires mocking `HttpContext`, `RequestServices`, the claims principal, and half a dozen services — just to instantiate the thing.

**The fix: inject exactly what you need**

```csharp
[ApiController]
public class DocumentsController : ControllerBase
{
   private readonly ICurrentUserProvider _currentUser;
   private readonly IAuditLogger _audit;
   private readonly AppDbContext _context;

   // The constructor is a complete, honest manifest of dependencies
   public DocumentsController(
       ICurrentUserProvider currentUser,
       IAuditLogger audit,
       AppDbContext context)
   {
       _currentUser = currentUser;
       _audit = audit;
       _context = context;
   }

   [HttpPost]
   public async Task<IActionResult> Create(CreateDocumentRequest request)
   {
       var userId = _currentUser.GetUserId();
       _audit.Log(userId, "CreateDocument", request);

       var doc = new Document(request.Title, request.Content, userId);
       _context.Documents.Add(doc);
       await _context.SaveChangesAsync();

       return Ok(new { doc.Id });
   }
}

// Small, focused, independently testable
public class CurrentUserProvider : ICurrentUserProvider
{
   private readonly IHttpContextAccessor _accessor;

   public CurrentUserProvider(IHttpContextAccessor accessor) => _accessor = accessor;

   public Guid GetUserId() =>
       Guid.Parse(_accessor.HttpContext!.User.Claims
           .First(c => c.Type == "sub").Value);

   public bool IsAdmin() =>
       _accessor.HttpContext!.User.IsInRole("Admin");
}
```

A `BlogController` that doesn't need audit logging doesn't inject `IAuditLogger`. No inherited baggage. No hidden dependencies. Unit tests pass mock implementations directly into the constructor. Each service does one thing.

**The rule:** if you find yourself adding something to a base class, stop. Ask whether the consuming class actually needs it. If only some subclasses need it, it doesn't belong in the base class — it belongs in a dedicated service injected where needed.

---

## Isolate Side Effects From Core Transactions

A failing email service should never roll back a successful payment. This sounds obvious. It happens constantly in production.

The failure mode is subtle: you process an order, send a confirmation email, and the email service throws an exception inside the same database transaction. The entire transaction rolls back. The customer sees an error. They retry. They get charged twice.

**The Outbox Pattern eliminates this class of bug entirely.**

The principle: within a single atomic transaction, save your business entity *and* a record of the side effect you intend to produce. Then commit. A separate background process handles the actual side effect — asynchronously, with retry logic, completely decoupled from the transaction.

```csharp
// The outbox message entity
public class OutboxMessage
{
   public Guid Id { get; init; } = Guid.NewGuid();
   public string EventType { get; init; } = string.Empty;
   public string Payload { get; init; } = string.Empty;
   public DateTime OccurredAt { get; init; } = DateTime.UtcNow;
   public DateTime? ProcessedAt { get; set; }
   public int RetryCount { get; set; }
   public string? LastError { get; set; }
}
```

```csharp
public async Task<Guid> CreateOrderAsync(CreateOrderRequest request, CancellationToken ct)
{
   var order = new Order(request.ProductId, request.Quantity, request.CustomerId);

   var outboxMessage = new OutboxMessage
   {
       EventType = nameof(OrderCreatedEvent),
       Payload = JsonSerializer.Serialize(new OrderCreatedEvent(order.Id, order.CustomerId))
   };

   // Both writes in the same transaction — atomic by default
   _context.Orders.Add(order);
   _context.OutboxMessages.Add(outboxMessage);
   await _context.SaveChangesAsync(ct);

   // At this point: order is saved, event is queued. Guaranteed consistent.
   return order.Id;
}
```

```csharp
public class OutboxProcessor : BackgroundService
{
   protected override async Task ExecuteAsync(CancellationToken ct)
   {
       while (!ct.IsCancellationRequested)
       {
           using var scope = _serviceProvider.CreateScope();
           var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
           var bus = scope.ServiceProvider.GetRequiredService<IEventBus>();

           var messages = await db.OutboxMessages
               .Where(m => m.ProcessedAt == null && m.RetryCount < 5)
               .OrderBy(m => m.OccurredAt)
               .Take(20)
               .ToListAsync(ct);

           foreach (var message in messages)
           {
               try
               {
                   await bus.PublishAsync(message.EventType, message.Payload, ct);
                   message.ProcessedAt = DateTime.UtcNow;
               }
               catch (Exception ex)
               {
                   message.RetryCount++;
                   message.LastError = ex.Message;
                   // ProcessedAt stays null — will retry on next pass
               }
           }

           await db.SaveChangesAsync(ct);
           await Task.Delay(TimeSpan.FromSeconds(10), ct);
       }
   }
}
```

If the email service is down when the processor runs, the message stays in the outbox with `ProcessedAt = null`. It retries. The order is never affected. Your system is now resilient to downstream failures by design, not by luck.

For high-throughput systems, add `FOR UPDATE SKIP LOCKED` to the outbox query to prevent multiple instances from processing the same message simultaneously.

---

## Never Expose EF Core Entities From Your API

Returning a raw EF Core entity from an API endpoint is one of those decisions that feels like a shortcut and reveals itself as a trap within three months.

```csharp
public class User
{
   public Guid Id { get; set; }
   public string Email { get; set; } = string.Empty;
   public string PasswordHash { get; set; } = string.Empty; // Should NEVER leave the server
   public virtual ICollection<Order> Orders { get; set; } = new List<Order>();
}

// THE WRONG WAY
[HttpGet("api/users/{id}")]
public async Task<ActionResult<User>> GetUser(Guid id)
{
   return await _context.Users.FindAsync(id); // Three simultaneous bugs
}
```

Three production incidents hidden in one endpoint:

**Security breach:** `PasswordHash` is serialized into the HTTP response. It's visible in browser dev tools, in log aggregators that capture response bodies, and in any proxy sitting between client and server. The fact that your frontend doesn't display it is irrelevant.

**N+1 performance disaster:** If lazy loading is enabled, the JSON serializer touching `Orders` triggers a database query per user. Return a list of 50 users and you've issued 51 database queries.

**Brittle API contract:** Your database schema is now your public API contract. Rename `PasswordHash` to `PasswordSaltedHash` during a security upgrade and you've silently broken every client consuming your API.

**The fix: project to a DTO at the database level**

```csharp
public record UserProfileDto(Guid Id, string Email, int OrderCount, DateTime MemberSince);

[HttpGet("api/users/{id}")]
public async Task<ActionResult<UserProfileDto>> GetUser(Guid id)
{
   var profile = await _context.Users
       .Where(u => u.Id == id)
       .Select(u => new UserProfileDto(
           u.Id,
           u.Email,
           u.Orders.Count(),        // COUNT in SQL — not loaded into memory
           u.CreatedAt
       ))
       .FirstOrDefaultAsync();

   return profile is null ? NotFound() : Ok(profile);
}
```

EF Core translates the projection into efficient SQL:

```sql
SELECT TOP(1) u.Id, u.Email,
   (SELECT COUNT(*) FROM Orders o WHERE o.UserId = u.Id) AS OrderCount,
   u.CreatedAt
FROM Users u
WHERE u.Id = @id
```

`PasswordHash` never touches application memory. `Orders` is a COUNT, not a collection load. And you can refactor your `User` entity freely without breaking the API contract — as long as you can still populate the DTO, clients are unaffected.

---

## Favor Vertical Slices Over Horizontal Layers

The classic `Controllers/Services/Repositories/Models` folder structure organizes code by technical role. When you need to understand, modify, or debug a single feature, you traverse the entire folder tree.

Vertical Slice Architecture organizes code by feature. Everything that implements one use case lives together.

**Classic layered structure for "Create Product":**
```
Controllers/
 ProductsController.cs       (handles HTTP)
Services/
 IProductService.cs          (interface)
 ProductService.cs           (business logic)
Repositories/
 IProductRepository.cs       (interface)
 ProductRepository.cs        (data access)
DTOs/
 CreateProductRequest.cs
 ProductDto.cs
```

Six files across six folders to understand one feature.

**Vertical slice structure:**
```
Features/
 Products/
   CreateProduct/
     CreateProduct.cs        (command, validator, handler — all here)
   GetProduct/
     GetProduct.cs
   SearchProducts/
     SearchProducts.cs
```

```csharp
// Features/Products/CreateProduct/CreateProduct.cs
// Everything for this use case in one file

public static class CreateProduct
{
   public record Command(string Name, decimal Price, Guid CategoryId) : IRequest<Guid>;

   public class Validator : AbstractValidator<Command>
   {
       public Validator()
       {
           RuleFor(x => x.Name).NotEmpty().MaximumLength(200);
           RuleFor(x => x.Price).GreaterThan(0);
           RuleFor(x => x.CategoryId).NotEmpty();
       }
   }

   internal sealed class Handler : IRequestHandler<Command, Guid>
   {
       private readonly AppDbContext _context;

       public Handler(AppDbContext context) => _context = context;

       public async Task<Guid> Handle(Command request, CancellationToken ct)
       {
           var product = new Product
           {
               Id = Guid.NewGuid(),
               Name = request.Name,
               Price = request.Price,
               CategoryId = request.CategoryId
           };

           _context.Products.Add(product);
           await _context.SaveChangesAsync(ct);

           return product.Id;
       }
   }
}

// Registered separately, pointing at the handler
app.MapPost("/api/products", async (CreateProduct.Command cmd, ISender sender) =>
{
   var id = await sender.Send(cmd);
   return Results.Created($"/api/products/{id}", new { Id = id });
});
```

A developer new to the codebase can open `Features/Products/CreateProduct/` and read the complete feature from input validation through to database persistence. No file-hopping required.

**The acceptable trade-off:** some code duplication between slices. If two features have similar validation logic, they may repeat some rules. This is usually fine. The cost of duplication is measured in lines of code. The cost of high coupling and low cohesion in a growing codebase is measured in developer hours.

---

## Protect Domain Invariants With IReadOnlyList\<T\>

A subtle but powerful practice: never expose mutable collections as public properties on domain entities.

```csharp
// This looks harmless
public class Order
{
   public List<LineItem> LineItems { get; set; } = new();
}

// But this is now valid code anywhere in the application
order.LineItems.Add(new LineItem(...));    // Bypasses all business rules
order.LineItems.Clear();                  // Wipes the order without any checks
order.LineItems[0].Quantity = -5;         // Invalid state, no validation
```

When any code anywhere can modify `LineItems` directly, you have no way to enforce domain invariants. The `TotalAmount` computed property can become stale. The "maximum 20 items per order" rule can be silently violated.

```csharp
public class Order
{
   private readonly List<LineItem> _lineItems = new();

   public Guid Id { get; private set; } = Guid.NewGuid();
   public decimal TotalAmount => _lineItems.Sum(li => li.Price * li.Quantity);

   // Read-only view — consumers can query but not mutate
   public IReadOnlyList<LineItem> LineItems => _lineItems.AsReadOnly();

   public void AddLineItem(Product product, int quantity)
   {
       ArgumentNullException.ThrowIfNull(product);

       if (quantity <= 0)
           throw new ArgumentException("Quantity must be positive.", nameof(quantity));

       if (_lineItems.Count >= 20)
           throw new InvalidOperationException("Order cannot exceed 20 line items.");

       if (_lineItems.Any(li => li.ProductId == product.Id))
           throw new InvalidOperationException("Product already in order. Use UpdateQuantity instead.");

       _lineItems.Add(new LineItem(product.Id, quantity, product.Price));
   }

   public void RemoveLineItem(Guid productId)
   {
       var item = _lineItems.FirstOrDefault(li => li.ProductId == productId)
           ?? throw new InvalidOperationException("Line item not found.");

       _lineItems.Remove(item);
   }
}
```

`IReadOnlyList<T>` over `IEnumerable<T>` specifically because it preserves the performance characteristics of a list — indexed access, `.Count` without re-iteration — while signaling immutability to consumers.

Every modification goes through a method. Every method can enforce rules, update derived state, and raise domain events. The entity becomes impossible to put into an invalid state from outside code.

---

## A Three-Question Test Before Every Abstraction

Before adding an interface, a service layer, a MediatR pipeline, or any other layer of indirection, answer three questions:

**1. What concrete, upcoming change does this make cheaper?**
If you can't name a specific change, you're insuring against a risk you've imagined, not one you've identified. Speculative abstractions create complexity without delivering value.

**2. Does this make production failures easier to diagnose?**
Some abstractions genuinely help — structured logging interfaces, observable pipeline steps. Others obscure failures by adding indirection that makes stack traces harder to follow. Which is this?

**3. Can a new engineer understand this in one sitting?**
Complexity compounds. An abstraction that makes sense to the person who wrote it, but requires 30 minutes of explanation to every new team member, has a recurring cost that never goes away.

If you can't answer yes to at least one of these, keep it simple. The abstraction can always be added later when the pain of not having it is real.

---

## The Honest Case For CQRS and Layered Architecture

After everything above, the summary is not "throw out your architecture." It's "apply it deliberately."

**CQRS earns its complexity when:**
- Your read workload is fundamentally different from your write workload and needs independent optimization
- Your read models are projections or denormalized views that would be awkward to derive from write models
- Teams are large enough that read and write paths being separate genuinely reduces coordination overhead

**CQRS is overkill when:**
- Your application is fundamentally CRUD — the same shape flows from form to database and back
- You're in early-stage development where iteration speed matters more than architectural purity
- You'd be maintaining two models that are nearly identical

**Layered architecture earns its complexity when:**
- You need enforced boundaries — the compiler preventing domain code from importing HTTP types
- The system will live for years and pass through multiple teams who benefit from a shared structure
- Testability at multiple levels (unit, integration, end-to-end independently) is a genuine requirement

**Layered architecture becomes a liability when:**
- Layers exist only to satisfy the pattern — a service that does nothing but delegate to a repository
- The architecture becomes a place to avoid making design decisions rather than a map through them
- Every new feature requires touching five files before writing the first line of business logic

The real skill is calibrating how much architecture your system *actually needs today*, with the discipline to add more only when the current complexity genuinely demands it.

Good architecture isn't what you add at the beginning. It's what you add when the pain of not having it is specific, measurable, and real.

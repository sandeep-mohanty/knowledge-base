# The Clean Architecture Spectrum: CRUD to Hexagonal

Clean Architecture isn’t binary. Here’s how to choose the right level of structure for your team’s real complexity.

---

## Why This Spectrum Exists
Every .NET team eventually asks: **“Do we really need Clean Architecture?”**

The honest answer: **it depends on where you are on the spectrum.** After seven years of building everything from 3 K-line internal tools to 150 K-line enterprise platforms, I stopped treating architecture like a yes/no switch.

Architecture grows with you. Your system will outgrow its structure long before you notice. Clean Architecture debates usually split into two camps: **“It’s essential for maintainability”** vs. **“It’s over-engineering.”** Both are right — and both are wrong.

What matters is finding the level that fits **today**.

---

## The Five-Level Spectrum
Clean Architecture is a continuum of separation and complexity, not a checklist. Each level trades speed for structure in a way that makes sense for its stage.



**Summary of the five levels — Clean Architecture evolves as systems grow in complexity.**

| Level | Name | Typical LOC | Team Size | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Transaction Script | < 5K | 1-2 | Prototypes, Internal Tools |
| **2** | Service Layer | 5K - 50K | 3 - 10 | CRUD-heavy apps |
| **3** | Basic Clean | 30K - 200K | 8 - 25 | Scaling SaaS, Business Logic |
| **4** | CQRS + Clean | 150K - 500K | 15 - 40 | Read-heavy systems |
| **5** | Hexagonal + DDD | 300K+ | 30+ | Complex Enterprise Domains |

These ranges overlap intentionally. Team size, code volume, and architecture level are **signals**, not rules. Domain complexity and change frequency often matter more.

---

### Level 1: When Simplicity Beats Structure (Transaction Script)
**When it works best:** Internal tools, admin dashboards, or quick prototypes — typically under 5K LOC.

```csharp
// .NET 8 Minimal API approach
app.MapGet("/reports/daily-sales", async (AppDbContext db) =>
{
    var sales = await db.Orders
        .Where(o => o.CreatedAt.Date == DateTime.UtcNow.Date)
        .GroupBy(o => o.ProductId)
        .Select(g => new
        {
            ProductId = g.Key,
            TotalSales = g.Sum(o => o.Amount),
            OrderCount = g.Count()
        })
        .ToListAsync();

    return Results.Ok(sales);
});
```

**In production:**
* ✅ Fastest path from idea to production
* ✅ Zero ceremony
* ❌ Tightly coupled to ORM
* ❌ Hard to test without a live DB

🧩 **If the tool starts living longer than you planned, it’s a sign to move up.**

---

### Level 2: When Logic Starts Leaking (Service Layer Only)
**When it fits:** CRUD-heavy apps with moderate business logic (5 K–50 K LOC, 3–10 devs).

```csharp
public class OrderService
{
    private readonly AppDbContext _context;
    private readonly ILogger<OrderService> _logger;

    public OrderService(AppDbContext context, ILogger<OrderService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<OrderResult> CreateOrderAsync(CreateOrderRequest request)
    {
        if (request.Items.Count == 0)
            return OrderResult.Failure("Order must contain at least one item");

        var customer = await _context.Customers
            .FirstOrDefaultAsync(c => c.Id == request.CustomerId);
        
        if (customer == null)
            return OrderResult.Failure("Customer not found");

        var order = new Order
        {
            CustomerId = request.CustomerId,
            CreatedAt = DateTime.UtcNow,
            Status = OrderStatus.Pending
        };

        foreach (var item in request.Items)
        {
            var product = await _context.Products.FindAsync(item.ProductId);
            if (product.Stock < item.Quantity)
                return OrderResult.Failure($"Insufficient stock for {product.Name}");

            order.Items.Add(new OrderItem
            {
                ProductId = item.ProductId,
                Quantity = item.Quantity,
                UnitPrice = product.Price
            });
            product.Stock -= item.Quantity;
        }

        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Order {OrderId} created for {CustomerId}", order.Id, customer.Id);
        return OrderResult.Success(order.Id);
    }
}
```

**In production:**
* ✅ Centralized logic that reviewers could reason about
* ✅ Moderately testable
* ❌ Coupled to EF Core
* ❌ Becomes painful when business logic outgrows CRUD boundaries

💡 **When to evolve:** When your service layer starts coordinating multiple repositories or cross-domain logic — time to introduce boundaries.

---

### Level 3: When Testing Finally Matters (Basic Clean Architecture)
**Where it shines:** SaaS systems (30K–200K LOC, 8–25 devs) with growing rules, multi-env deployments, and frequent releases.



```csharp
// Domain entity (Core layer - no infrastructure deps)
// Using FluentResults NuGet package for railway-oriented error handling
public class Subscription
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public Plan Plan { get; private set; }
    public SubscriptionStatus Status { get; private set; }
    public DateTime NextBillingDate { get; private set; }

    private Subscription() { }

    public static Result<Subscription> Create(Guid customerId, Plan plan)
    {
        if (plan.Price < 0)
            return Result<Subscription>.Failure("Plan price cannot be negative");

        return Result<Subscription>.Success(new Subscription
        {
            Id = Guid.NewGuid(),
            CustomerId = customerId,
            Plan = plan,
            Status = SubscriptionStatus.Active,
            NextBillingDate = DateTime.UtcNow.AddMonths(1)
        });
    }

    public Result Renew()
    {
        if (Status != SubscriptionStatus.Active)
            return Result.Failure("Cannot renew inactive subscription");

        NextBillingDate = NextBillingDate.AddMonths(1);
        return Result.Success();
    }
}

// Use case (Application layer)
public class RenewSubscriptionUseCase
{
    private readonly ISubscriptionRepository _subscriptions;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IUnitOfWork _unitOfWork;

    public RenewSubscriptionUseCase(
        ISubscriptionRepository subscriptions,
        IPaymentGateway paymentGateway,
        IUnitOfWork unitOfWork)
    {
        _subscriptions = subscriptions;
        _paymentGateway = paymentGateway;
        _unitOfWork = unitOfWork;
    }

    public async Task<Result<RenewalReceipt>> ExecuteAsync(Guid subscriptionId)
    {
        var subscription = await _subscriptions.GetByIdAsync(subscriptionId);
        if (subscription == null) return Result<RenewalReceipt>.Failure("Subscription not found");

        var renewalResult = subscription.Renew();
        if (!renewalResult.IsSuccess) return Result<RenewalReceipt>.Failure(renewalResult.Error);

        var paymentResult = await _paymentGateway.ChargeAsync(subscription.CustomerId, subscription.Plan.Price);
        if (!paymentResult.IsSuccess) return Result<RenewalReceipt>.Failure("Payment failed");

        await _unitOfWork.CommitAsync();
        
        return Result<RenewalReceipt>.Success(new RenewalReceipt {
            SubscriptionId = subscription.Id,
            Amount = subscription.Plan.Price,
            NextBillingDate = subscription.NextBillingDate
        });
    }
}
```

**In production:**
* ✅ Business rules testable in isolation
* ✅ Infrastructure swappable without rewriting logic
* ✅ Teams can work in parallel safely
* ❌ 25–35% more code overhead
* ❌ Mapping friction between layers

⚖️ **Aim for pragmatic decoupling, not purity.** Clean Architecture should make iteration easier, not slower.

---

### Level 4: When Reads Start to Dominate (CQRS + Clean)
**When it fits:** Read-dominant systems (5–10× reads per write) or teams needing independent scaling for reads/writes.



```csharp
// Query (Read side - optimized for reads)
public record GetShipmentDetailsQuery(string TrackingNumber);
public class GetShipmentDetailsQueryHandler
{
    private readonly IReadDbContext _readDb; // Separate read database

    public async Task<ShipmentDetailsDto?> HandleAsync(GetShipmentDetailsQuery query)
    {
        // Direct query against denormalized read model
        return await _readDb.ShipmentDetails
            .Where(s => s.TrackingNumber == query.TrackingNumber)
            .Select(s => new ShipmentDetailsDto
            {
                TrackingNumber = s.TrackingNumber,
                CurrentStatus = s.CurrentStatus,
                EstimatedDelivery = s.EstimatedDelivery,
                StatusHistory = s.StatusHistory // Pre-joined, no N+1 queries
            })
            .FirstOrDefaultAsync();
    }
}
```

**In production:**
* ✅ Read performance skyrocketed (10× faster queries)
* ✅ Write logic clarity improved
* ✅ Independent scaling of read/writes
* ❌ 100–500 ms eventual consistency lag
* ❌ More infra complexity and debugging overhead

🧠 **CQRS isn’t a badge of maturity; it’s a response to read load.**

---

### Level 5: Full Hexagonal + DDD
**When it’s worth it:** Complex domains (finance, healthcare, logistics) with 10+ integrations and long-lived logic.



```csharp
// Port (interface in domain)
public interface IInsuranceProviderGateway
{
    Task<ProviderResponse> SubmitClaimAsync(Claim claim);
    Task<PolicyDetails> GetPolicyAsync(PolicyNumber policyNumber);
}

// Adapter (implementation in infrastructure)
public class AetnaApiAdapter : IInsuranceProviderGateway
{
    private readonly HttpClient _httpClient;
    public async Task<ProviderResponse> SubmitClaimAsync(Claim claim)
    {
        // External API integration isolated from domain
        var request = new AetnaClaimRequest { /* Map domain to provider specific */ };
        var response = await _httpClient.PostAsJsonAsync("/claims/submit", request);
        // ... handling result
    }
}
```

**In production:**
* ✅ External dependencies fully isolated
* ✅ Core domain testable without mocks
* ✅ Parallel development across domains
* ❌ Steep learning curve (months, not weeks)
* ❌ Easy to over-abstract if started too early

---

## From “Do We Need Clean Architecture?” to “Which Level Fits Right Now?”
Architecture should evolve with domain complexity and team maturity — not ambition.



**Heuristics to help match architecture level with system scale and team maturity.**

| Factor | Evolve to Next Level If... |
| :--- | :--- |
| **Logic** | Business rules are duplicated across endpoints or UI. |
| **Testing** | It takes a live database to test a simple calculation. |
| **Integrations** | External API changes break unrelated business logic. |
| **Team** | Developers are constantly stepping on each other's toes in the same files. |

🪜 **Evolve one level at a time. Architecture maturity compounds — rewrites rarely pay off.**

---

## Red Flags You’re at the Wrong Level
**Under-architected:**
* Logic duplicated across endpoints
* Integration changes break unrelated features
* Business rules untestable without a DB

**Over-architected:**
* Interfaces with single implementations
* DTOs mirror entities 1:1
* Onboarding takes longer than the first release

## The Bottom Line
Every system grows faster than its structure. The goal isn’t to start “clean” — it’s to know **when to level up**. The right architecture isn’t the most sophisticated one — it’s the one your team can ship, maintain, and evolve sustainably.
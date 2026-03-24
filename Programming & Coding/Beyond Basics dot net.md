# Beyond Basics: Top 10 .NET Questions for Mid to Senior Developers
**Architecture, Performance, and Pragmatism**

Navigating a **.NET interview** can feel like walking a tightrope between **theory** and **real-world** experience. **Mid-level** developers are expected to demonstrate solid understanding of **C#**, **design principles**, and **frameworks**, while **senior developers** must show **judgment** in **architecture**, **patterns**, and **performance**.

This guide covers questions that go beyond the **basics** — spanning **SOLID principles**, **dependency injection**, common **design patterns**, **Blazor**, **Entity Framework**, **caching**, and **performance** management. You’ll get both **conceptual questions** and **practical scenarios**, helping you prepare for **interviews** that test not just **knowledge**, but how you **think** as a **.NET developer**.

Whether you’re brushing up for a **technical screen** or **preparing** for a **senior-level discussion**, these questions are designed to **challenge** your **understanding** and showcase your **expertise**.

---

## 1️⃣ SOLID Principles: The Backbone of Clean .NET Code



**SOLID** isn’t just a buzzword; it’s a **roadmap** for **maintainable**, **testable**, and **scalable** code. Understanding the **principles** is one thing; knowing when and how to apply them is another.

### Mid-Level Questions
These focus on **comprehension** and **practical application**:

#### Single Responsibility Principle (SRP)
* **What is SRP, and why is it important?** SRP states that a **class** should have only **one reason to change**, meaning it should have a **single responsibility**. This makes code **easier** to **maintain**, **test**, and **understand**.
* **Can you give an example of a class that violates SRP and how you would refactor it?** To refactor, split into separate classes: `InvoiceCalculator`, `InvoiceRepository`, and `InvoiceNotifier`.

```csharp
class InvoiceProcessor
{
    public void CalculateTotal() { /*...*/ }
    public void SaveToDatabase() { /*...*/ } // Violates SRP
    public void SendEmailConfirmation() { /*...*/ } // Violates SRP
}
```

**Senior-Level Considerations**
* ✅ **Practical judgment:** Splitting classes for SRP is good, but don’t over-split. Too many tiny classes can make code harder to navigate.
* ✅ **Legacy code:** Sometimes a single class may have multiple responsibilities; refactor incrementally rather than rewriting everything at once.
* ✅ **Guiding question:** “Does this class have more than one reason to change?”

#### Open/Closed Principle (OCP)
* **How does OCP encourage extensibility without modifying existing code?** OCP states that software entities should be **open for extension, closed for modification**. You can add **new behavior** by **extending** classes or implementing **interfaces**, without changing **existing tested code**.
* **Provide an example of using interfaces or inheritance to follow OCP.** Open for extension: You can introduce new discount strategies by implementing `IDiscountStrategy`.

```csharp
interface IDiscountStrategy { decimal Apply(decimal total); }

class SeasonalDiscount : IDiscountStrategy { /*...*/ }
class ClearanceDiscount : IDiscountStrategy { /*...*/ }

class Order
{
    public decimal CalculateTotal(IDiscountStrategy discount) => discount.Apply(100);
}
```

**Senior-Level Considerations**
* ✅ **When to apply:** Great for code that frequently changes, like plugin systems, payment processors, or discount strategies.
* ✅ **Overuse risk:** Creating interfaces for every tiny feature can lead to excessive abstraction.
* ✅ **Guiding question:** “Will this module need future extensions?”

#### Liskov Substitution Principle (LSP)
* **What does LSP mean in practice?** Subtypes must be **substitutable** for their **base types** without altering **expected behavior**.
* **How can violating LSP lead to runtime bugs?** Violating LSP can cause **runtime errors** because a subclass doesn’t fully support the behavior expected from its base class, breaking code that relies on that contract.

```csharp
class Bird { public virtual void Fly() { /*...*/ } }
class Ostrich : Bird { public override void Fly() { throw new NotSupportedException(); } } // Violates LSP
```

**Senior-Level Considerations**
* ✅ **Practical judgment:** Ensure subclasses honor expected behavior. If a subclass requires changing client code to use it, LSP is violated.
* ✅ **Guiding question:** “Can I replace the base class with this subclass everywhere without breaking anything?”

#### Interface Segregation Principle (ISP)
* **Why avoid “fat” interfaces?** Large interfaces force classes to implement methods they don’t need, creating unnecessary dependencies.
* **How would you refactor a large interface into smaller ones?**

```csharp
interface IWorker { void Work(); void Eat(); } // Fat interface

interface IWorkable { void Work(); }
interface IFeedable { void Eat(); } // Segregated interfaces
```

#### Dependency Inversion Principle (DIP)
* **How does DIP relate to dependency injection in .NET?** DIP states that **high-level modules** should not depend on **low-level modules**, rather they should depend on **abstractions** (interfaces or abstract classes). **Dependency Injection** provides these abstractions at runtime.

```csharp
public interface ILogger { void Log(string message); }
public class ConsoleLogger : ILogger { public void Log(string msg) => Console.WriteLine(msg); }

public class OrderService
{
    private readonly ILogger _logger;
    public OrderService(ILogger logger) { _logger = logger; }
    public void Process() { _logger.Log("Order processed."); }
}
```

---

## 2️⃣ Dependency Injection: Decoupling Made Practical



### Mid-Level Questions
* **What is Dependency Injection?** DI is a design pattern where a class receives its dependencies from the outside rather than creating them internally.
* **What are the common types of DI in .NET?** Constructor injection (most common), Property injection, and Method injection.
* **How do you register services in .NET?** Use the built-in `IServiceCollection` in `Program.cs`:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<ILogger, ConsoleLogger>();
builder.Services.AddTransient<EmailNotifier>();
```

### Senior-Level Considerations
* ✅ **Lifetime management:**
    * **Transient:** New instance every time.
    * **Scoped:** One instance per request (ideal for web requests).
    * **Singleton:** One instance for the app lifetime (watch for thread safety).
* ✅ **Avoid over-abstraction:** Not every class needs an interface. Ask: “Will this class ever have multiple implementations?”
* ✅ **DI in Blazor context:** Be mindful that in WebAssembly, Singleton and Transient lifetimes behave differently due to the single-user session nature.

---

## 3️⃣ MediatR, CQRS & Vertical Slice Architecture



### Mid-Level Questions
* **What is CQRS and why use it?** **CQRS** (Command Query Responsibility Segregation) separates **read operations** (queries) from **write operations** (commands). It optimizes for scalability and simplifies complex business logic.
* **What is MediatR in .NET?** A library that implements the mediator pattern, decoupling senders from handlers.

```csharp
public class GetOrdersQuery : IRequest<List<Order>> { }
public class GetOrdersQueryHandler : IRequestHandler<GetOrdersQuery, List<Order>>
{
    private readonly DbContext _context;
    public GetOrdersQueryHandler(DbContext context) => _context = context;
    public Task<List<Order>> Handle(GetOrdersQuery request, CancellationToken ct)
        => _context.Orders.ToListAsync(ct);
}
```

* **What is Vertical Slice Architecture?** Instead of **layering** by technical concerns, **Vertical Slice** organizes code by **feature/endpoint**. Each slice contains everything needed for that feature (command, handler, validation).

### Senior-Level Considerations
* ✅ **Trade-offs of MediatR:**
    * **Pros:** Decouples components, enforces SRP per handler.
    * **Cons:** Can lead to too many small classes; adds slight latency.
* ✅ **Design judgment:** Should a query return DTOs or domain entities? Generally, returning DTOs keeps your API stable and prevents exposing internal models.

---

## 4️⃣ Design Patterns That Actually Matter

### Mid-Level Questions
* **Factory Pattern:** Centralizes object creation, useful when the type isn't known until runtime.
* **Strategy Pattern:** Encapsulates interchangeable behaviors.
* **Observer Pattern:** Allows objects to subscribe and react to events in other objects.
* **BFF (Backend For Frontend):** Tailors backend endpoints for specific frontends (mobile vs. web) to reduce over-fetching.

### Senior-Level Considerations
* ✅ **When to apply:** Patterns are **tools**, not rules. Ask: “Does this simplify the design or add unnecessary complexity?”
* ✅ **Pattern selection:** Favor built-in abstractions (e.g., `IServiceProvider`) before building custom factories.

---

## 5️⃣ Entity Framework Core — Performance & Pitfalls



### Mid-Level Questions
* **Lazy Loading vs Eager Loading:** Lazy loading loads related entities when accessed (risks N+1). Eager loading uses `.Include()` to load data upfront.
* **What is the N+1 problem?** Occurs when a query triggers additional queries per entity.

```csharp
// N+1 Problem
var blogs = context.Blogs.ToList(); // 1 Query
foreach (var blog in blogs)
{
    Console.WriteLine(blog.Posts.Count); // N Queries
}
```

* **Tracking vs No-Tracking:** Use `AsNoTracking()` for read-only queries to improve performance.

### Senior-Level Considerations
* ✅ **Monitoring:** Use SQL Profiler or Application Insights to detect slow queries.
* ✅ **Database design:** Indexes and constraints impact EF query efficiency.
* ✅ **Raw SQL:** Know when EF-generated SQL is inefficient and switch to raw SQL or Dapper.

---

## 6️⃣ Caching Strategies & Invalidation

### Mid-Level Questions
* **MemoryCache:** For single-server apps.
* **Distributed Cache:** Shared cache (Redis) for multi-server apps.
* **Cache Invalidation:** TTL (Time-based), Event-based, or Manual removal.

### Senior-Level Considerations
* ✅ **Race conditions:** Use **locks or double-checked patterns** to prevent multiple requests from refreshing the same cache entry simultaneously.
* ✅ **Cache-aside pattern:** Check cache first, load from DB if missing, then update cache.

---

## 7️⃣ State Management (Blazor-Focused)

### Mid-Level Questions
* **UI State:** Component-specific (form inputs).
* **Application State:** Shared (logged-in user info).
* **Persistence:** Use `LocalStorage` or `ProtectedBrowserStorage` to keep state across refreshes.

### Senior-Level Considerations
* ✅ **Blazor Server:** Scoped services live per user connection; manage memory carefully for concurrent users.
* ✅ **State Patterns:** Consider Flux/Redux-like approaches for complex interactions.

---

## 8️⃣ Performance & Scalability Under Load



### Mid-Level Questions
* **API Performance:** Use `async/await`, pagination, and efficient indexing.
* **Race Conditions:** Occur when shared mutable state in a Singleton is accessed concurrently.

### Senior-Level Considerations
* ✅ **Patterns:** Use `ValueTask` for high-frequency methods to reduce allocations.
* ✅ **Concurrency:** Use `ConcurrentDictionary` or immutable objects for shared state.

---

## 9️⃣ Logging, Observability & Diagnostics



### Mid-Level Questions
* **ILogger:** Built-in lightweight logging.
* **Serilog:** Powerful structured logging with multiple sinks.
* **Application Insights:** Cloud-based telemetry.

### Senior-Level Considerations
* ✅ **Distributed Tracing:** Use **correlation IDs** to track requests across microservices.
* ✅ **Log Level Discipline:** Avoid logging everything at `Information`; use `Debug` or `Warning` appropriately.

---

## 10. Security & Cross-Cutting Concerns

### Mid-Level Questions
* **Authentication vs Authorization:** Identity vs. Permissions.
* **Vulnerabilities:** SQL Injection (use EF), XSS (Razor encodes by default), CSRF (use tokens).
* **Validation:** Use `FluentValidation` for complex, reusable rules.

### Senior-Level Considerations
* ✅ **Centralization:** Use middleware or filters for cross-cutting concerns (Security, Logging) rather than sprinkling them in business logic.
* ✅ **Resilience:** Combine exception handling with retry policies (e.g., **Polly**).

---

Mastering .NET isn’t just about knowing syntax or patterns — it’s about **applying principles with judgment**. Understanding the “why” behind the “how” separates competent developers from exceptional ones.

Would you like me to find a **specific code sample for a global exception handling middleware with integrated structured logging** to help you implement Section 9 and 10 in your current project?
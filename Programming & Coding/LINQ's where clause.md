# LINQ’s .Where() Has Been Executing Differently Than You Think. Here’s What Actually Happens.
**The silent switch from IQueryable to IEnumerable that’s pulling entire tables into your API’s memory**

Last week I spent three hours tracking down a performance issue that turned out to be one line of code. An endpoint that returned 50 orders was making the database server work harder than it should. SQL Profiler showed the query: `SELECT * FROM Orders`. No WHERE clause. No filtering at all.

The C# code looked fine:
```csharp
public IEnumerable<Order> GetRecentOrders(int customerId)
{
    return _context.Orders
        .Where(o => o.CustomerId == customerId)
        .Where(o => o.CreatedAt > DateTime.UtcNow.AddDays(-30));
}
```
Two `.Where()` calls. Both with clear filter conditions. So why was EF Core 10 pulling the entire Orders table?

The answer is something I should have known years ago but never fully understood until I looked at what `.Where()` actually does under the hood.

---

### The same method. Two completely different behaviors.

Here’s the thing most .NET developers don’t realize: there are two versions of `.Where()`. One lives in `System.Linq.Enumerable`…



1.  When you call `.Where()` on an **IQueryable<T>** (like a DbSet in EF Core), you're calling the **Queryable** version. It doesn't run your filter. It builds an **expression tree**, a data structure that represents your filter logic as code. EF Core later reads that expression tree and translates it to SQL.
2.  When you call `.Where()` on an **IEnumerable<T>**, you're calling the **Enumerable** version. It runs your filter **in memory**, on your application server, using a compiled delegate.



The problem is that C# will silently switch between these two depending on how your code is structured. And that switch can turn a 2ms database query into a 2-second memory dump.

---

### The moment everything breaks

Go back to my original code:
```csharp
public IEnumerable<Order> GetRecentOrders(int customerId)
{
    return _context.Orders
        .Where(o => o.CustomerId == customerId)
        .Where(o => o.CreatedAt > DateTime.UtcNow.AddDays(-30));
}
```
This works correctly. Both `.Where()` calls execute on `IQueryable<Order>`, the expression tree gets built, EF Core generates proper SQL with a WHERE clause, and only matching rows come back.

Now look at this version:
```csharp
public IEnumerable<Order> GetRecentOrders(int customerId)
{
    IEnumerable<Order> orders = _context.Orders;
    
    return orders
        .Where(o => o.CustomerId == customerId)
        .Where(o => o.CreatedAt > DateTime.UtcNow.AddDays(-30));
}
```
Same logic. Different result. That first line casts `DbSet<Order>` (which is `IQueryable<Order>`) to `IEnumerable<Order>`. Now when `.Where()` is called, C# resolves it to `Enumerable.Where()` instead of `Queryable.Where()`.

The moment you enumerate this, EF Core executes `SELECT * FROM Orders`, loads every row into memory, and then your application filters them using the lambda. If you have 500,000 orders, you just loaded 500,000 orders into RAM to return 50.

---

### Where this actually happens in production

Nobody writes that explicit cast on purpose. But equivalent patterns show up everywhere:

#### Pattern 1: Method signatures that return IEnumerable
```csharp
public interface IOrderRepository
{
    IEnumerable<Order> GetAll();
}

// In the calling code
var recent = _orderRepository.GetAll()
    .Where(o => o.CreatedAt > DateTime.UtcNow.AddDays(-30));
```
The repository returns `IEnumerable<Order>`. Even if the underlying implementation returns an `IQueryable`, the interface signature forces the cast. Every `.Where()` after `GetAll()` runs in memory.

#### Pattern 2: ToList() in the middle of a chain
```csharp
var orders = _context.Orders
    .Include(o => o.Items)
    .ToList()  // This is where it breaks
    .Where(o => o.CustomerId == customerId);
```
Maybe someone added `.ToList()` to fix a lazy loading issue. Maybe they wanted to materialize the query early for debugging. Either way, everything after `.ToList()` runs in memory.

#### Pattern 3: Passing data between methods
```csharp
public IEnumerable<Order> ApplyFilters(IEnumerable<Order> orders, FilterCriteria criteria)
{
    if (criteria.CustomerId.HasValue)
        orders = orders.Where(o => o.CustomerId == criteria.CustomerId.Value);
    
    if (criteria.MinAmount.HasValue)
        orders = orders.Where(o => o.Total >= criteria.MinAmount.Value);
    
    return orders;
}

// Called like this
var filtered = ApplyFilters(_context.Orders, criteria);
```
The method accepts `IEnumerable<Order>`. When you pass in a DbSet, it gets cast at the method boundary. All filtering happens client-side.

---

### How to see what’s actually happening

EF Core 10 makes this visible if you configure logging:
```csharp
optionsBuilder.LogTo(Console.WriteLine, LogLevel.Information);
```
Or enable the **CA1851** analyzer that Microsoft added specifically for this issue. Add this to your `.editorconfig`:
`dotnet_diagnostic.CA1851.severity = warning`

It flags code that enumerates the same `IEnumerable` multiple times, which often correlates with this problem.

But the most direct way to catch this is to check your SQL. If you write `.Where(o => o.CustomerId == 5)` and your SQL shows `SELECT * FROM Orders` without a WHERE clause, you've hit the IQueryable-to-IEnumerable switch.



---

### The fix is about types, not logic

Once you understand what’s happening, the fix is straightforward: keep things as `IQueryable<T>` for as long as possible.

**Change repository interfaces:**
```csharp
public interface IOrderRepository
{
    IQueryable<Order> GetAll();
}
```

**Change method signatures:**
```csharp
public IQueryable<Order> ApplyFilters(IQueryable<Order> orders, FilterCriteria criteria)
```

Move materialization (`.ToList()`, `.ToArray()`) to the last possible moment, after all filtering is complete. And when you do need to materialize early, be explicit about it. Don’t hide `.ToList()` in a method that returns `IEnumerable<T>`. Make the caller see that they're working with in-memory data.

---

### The 25% performance difference nobody talks about

There’s another layer to this. Even when both IQueryable and IEnumerable versions hit the database correctly, they don’t perform identically.

.NET 10’s JIT improvements (better inlining, faster expression tree walking, improved row materialization) benefit `IQueryable` paths more than `IEnumerable` paths. Benchmarks from the EF Core team show 25-50% faster query execution in .NET 10 compared to .NET 8 on identical queries. But those gains assume you're actually using `IQueryable` throughout your chain.

If you’re accidentally switching to `IEnumerable` mid-chain, you lose most of those optimizations. The .NET 10 improvements to expression tree caching don't help code that never builds an expression tree in the first place.

---

### What I changed after finding this

Three things:
1.  **First**, I audited every repository interface in our codebase. Anywhere we returned `IEnumerable<T>` when we could have returned `IQueryable<T>`, I changed it. This was about 40 methods.
2.  **Second**, I added the CA1851 analyzer at warning level. It’s noisy at first but catches real problems.
3.  **Third**, I stopped treating `IEnumerable<T>` as "the safe abstraction." It's not. It hides critical information about how your data is being fetched. `IQueryable<T>` is the right abstraction for database-backed collections. `IReadOnlyCollection<T>` or `IReadOnlyList<T>` are better than `IEnumerable<T>` for in-memory collections because they don't defer execution.

The endpoint that started this investigation now runs in 12ms instead of 1,400ms. Same logic. Different types. Different execution path.
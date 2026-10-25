# Using EF Core Like a Pro: Performance, Patterns, and Modern Features in .NET 8–10
**Master modern EF Core features — compiled queries, split queries, JSON columns, EF.Functions, and global filters — to write clean, efficient, and scalable data access code.**

Entity Framework Core has evolved into a **powerful, full-featured ORM** that lets .NET developers work efficiently with relational databases while keeping code **clean** and **maintainable**.

With EF Core 8+ and beyond, new features like **compiled queries, advanced aggregations, split queries, JSON column support, EF.Functions, and global query filters** make it easier than ever to write **high-performance, expressive, and maintainable data access code**.

We’ll explore practical, real-world examples that demonstrate how to leverage these tools to optimize queries, simplify complex data operations, and avoid common pitfalls — all without sacrificing readability or domain modeling.

---

## 1. Compiled Queries: Hot Path Optimization

In EF Core, every LINQ query is translated into SQL each time it runs. For most applications, this is fine. However, in **high-traffic APIs or performance-critical paths**, this translation can become a **bottleneck**. That’s where **compiled queries** shine.

A **compiled query** is essentially a **preprocessed version of a LINQ query** that EF Core stores in **memory** so it doesn’t have to translate it into SQL every time it executes. By **caching** the **query plan**, EF Core can skip the **parsing** and **translation** steps, which reduces CPU overhead and speeds up **repeated executions**, especially for queries that run hundreds or thousands of times per second.



✨ **Example:** fetching a user by ID frequently in a **Blazor app** or **API endpoint**.

✅ Reduces the overhead of repeated LINQ-to-SQL translation.  
✅ Especially beneficial for **hot paths** like dashboards, lookups, or frequently accessed entities.  
✅ Can be combined with **AsNoTracking** for **even better read performance**.

```csharp
// Standard LINQ query
var user = await db.Users
    .AsNoTracking()
    .FirstOrDefaultAsync(u => u.Id == userId);

// Compiled query version
private static readonly Func<AppDbContext, Guid, User?> _getUserById =
    EF.CompileQuery(
        (AppDbContext db, Guid id) =>
            db.Users
              .AsNoTracking()
              .FirstOrDefault(u => u.Id == id));

// Usage
var userCompiled = _getUserById(db, userId);
```

💡 **Don’t use compiled queries everywhere.** Reserve them for queries executed frequently.

---

## 2. Tracking vs No-Tracking and Identity Resolution

EF Core allows entities to be **tracked** or **untracked**. Tracking is convenient for updates, but comes with a cost in **memory and change tracking**.

* **Tracking:** EF Core remembers every entity you load. If two queries return the same entity (same primary key), EF Core gives you the same object instance. This is called **identity resolution**, and it happens automatically for tracked queries.
* **No-Tracking (AsNoTracking()):** EF Core doesn’t track entities. Every time you query the same entity, EF Core creates a new object instance, even if it represents the same database row. This saves **memory and CPU**, but sometimes leads to **duplicate objects**.



### Why identity resolution matters
Imagine you load orders with their products:

```csharp
var orders = db.Orders
    .AsNoTracking()
    .Include(o => o.Items)
    .ThenInclude(i => i.Product)
    .ToList();
```

Without identity resolution (`AsNoTracking()`), if multiple orders include the same product, EF Core will create **separate** `Product` objects for each occurrence. This is inefficient and can cause problems if you:
* Compare object references (`product1 == product2` is false even if it’s the same product).
* Serialize objects to JSON and end up with duplicates.
* Rely on reference equality in your code.

### Why combine no-tracking with identity resolution
* **Memory-efficient:** You skip full change tracking for all entities.
* **Safe object graph:** Navigation properties referencing the same entity will now **share the same object**, which avoids duplication.
* **Perfect for read-heavy queries:** Dashboards, reporting, or API endpoints where you load entities for read-only purposes but still want a consistent object graph.

✨ **Example: no-tracking with identity resolution**
Multiple orders containing the same product will all point to the same `Product` instance.

```csharp
var orders = await db.Orders
    .AsNoTrackingWithIdentityResolution()
    .Include(o => o.Items)
    .ThenInclude(i => i.Product)
    .Where(o => o.OrderDate >= DateTime.UtcNow.AddDays(-30))
    .ToListAsync();
```

💡 **No-tracking with Identity Resolution** is **the best of both worlds** for read-heavy queries: **low memory + consistent object references**.

---

## 3. Bulk Operations: ExecuteUpdate & ExecuteDelete

Older EF Core versions required you to load entities into memory, modify them, and then call `SaveChanges()`. For **bulk updates or deletes**, this is **inefficient and slow**.

EF Core 8+ introduces **ExecuteUpdate** and **ExecuteDelete**, allowing changes **directly in the database**.

✨ **Example: delete orders older than a year**
Deletes all orders older than a year directly in the database without loading them into memory.

```csharp
await db.Orders
    .Where(o => o.OrderDate < DateTime.UtcNow.AddYears(-1))
    .ExecuteDeleteAsync();
```

✨ **Example: bulk update order statuses**
Updates all pending orders older than a week directly in the database to mark them as “Expired”.

```csharp
await db.Orders
    .Where(o => o.Status == "Pending" && o.OrderDate < DateTime.UtcNow.AddDays(-7))
    .ExecuteUpdateAsync(o => o.SetProperty(order => order.Status, "Expired"));
```

✅ Eliminates the overhead of fetching entities into memory.  
✅ Reduces database round-trips and memory usage.  
✅ Perfect for **maintenance tasks, scheduled jobs, or background services**.

💡 Use these methods for **hot paths and bulk operations**, but remember that they bypass EF Core change tracking.

---

## 4. Modern Aggregates & GroupBy

EF Core 8+ now translates **Count, Sum, Min/Max, Average, and GroupBy** directly into SQL.

✨ **Example: Group By**
Calculates the number of orders and total sales per customer for the last month directly in the database.

```csharp
var orderStats = await db.Orders
    .Where(o => o.OrderDate >= DateTime.UtcNow.AddMonths(-1))
    .GroupBy(o => o.CustomerId)
    .Select(g => new 
    {
        CustomerId = g.Key,
        OrdersCount = g.Count(),
        TotalAmount = g.Sum(o => o.Total)
    })
    .ToListAsync();
```

✨ **Example: Multiple Aggregates Together**
One query gives you a **full snapshot of customer order activity**, all in a single server-side operation.

```csharp
// Combine Count, Sum, Min, Max, Average
var customerStats = await db.Orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        OrdersCount = g.Count(),
        TotalAmount = g.Sum(o => o.Total),
        MinOrder = g.Min(o => o.Total),
        MaxOrder = g.Max(o => o.Total),
        AverageOrder = g.Average(o => o.Total)
    })
    .ToListAsync();
```

---

## 5. Split Queries & Include Optimization

When querying entities with multiple related tables, EF Core can generate **huge SQL joins** that create **duplicate rows** and slow queries, a problem known as a **cartesian explosion**.

EF Core 8+ introduces **split queries** (`AsSplitQuery()`) that execute multiple smaller queries instead of one giant join.



✨ **Example: Load orders with items and products safely**

```csharp
var customers = await db.Customers
    .AsNoTracking()
    .AsSplitQuery()
    .Include(c => c.Orders)
        .ThenInclude(o => o.Items)
            .ThenInclude(i => i.Product)
    .Include(c => c.Addresses)
    .Where(c => c.IsActive)
    .ToListAsync();
```

✅ Reduces SQL query size and complexity.  
✅ Prevents loading duplicate entities in memory.  
✅ Keeps queries fast and memory-efficient.

💡 Combine split queries with **logging or interceptors** to see if they actually improve performance in your specific scenario.

---

## 6. Interceptors & Logging

EF Core lets you **intercept queries, commands, and save operations**, giving you **deep insight** into your database activity.

✨ **Example: Logging long queries**

```csharp
public class QueryInterceptor : DbCommandInterceptor
{
    public override async Task<InterceptionResult<DbDataReader>> ReaderExecutingAsync(
        DbCommand command,
        CommandEventData eventData,
        InterceptionResult<DbDataReader> result,
        CancellationToken cancellationToken = default)
    {
        if (command.CommandText.Length > 1000)
        {
            Console.WriteLine($"Long query detected: {command.CommandText}");
        }

        return await base.ReaderExecutingAsync(command, eventData, result, cancellationToken);
    }
}
```

✨ **Example: Query tagging for easier profiling**

```csharp
var activeCustomers = await db.Customers
    .TagWith("Fetching active customers for dashboard")
    .Where(c => c.IsActive)
    .ToListAsync();
```

---

## 7. JSON Columns & Flexible Data

EF Core 8+ lets you map **JSON columns** to properties, allowing you to store **complex, semi-structured data** while still being able to query it efficiently.

✨ **Example: Filter products by JSON metadata**

```csharp
var redProducts = await db.Products
    .Where(p => p.Metadata["Color"].ToString() == "Red")
    .ToListAsync();
```

---

## 8. Raw SQL Queries & Mapping to DTOs

Sometimes LINQ isn’t enough. EF Core allows **raw SQL queries** that map directly to **DTOs**, bypassing unnecessary tracking.

✨ **Example: Top customers by total spending**

```csharp
var topCustomers = await db.TopCustomers
    .FromSqlInterpolated($@"
        SELECT c.Id, c.Name, SUM(o.Total) as TotalSpent
        FROM Customers c
        JOIN Orders o ON c.Id = o.CustomerId
        GROUP BY c.Id, c.Name
        HAVING SUM(o.Total) > {minSpend}
    ")
    .ToListAsync();
```

---

## 9. EF.Functions & SQL Helpers

EF Core exposes **database functions directly in LINQ**, allowing you to perform SQL-level operations efficiently.

✨ **Example: Date Difference Calculations**
```csharp
// Orders in the last 30 days
var recentOrders = await db.Orders
    .Where(o => EF.Functions.DateDiffDay(o.OrderDate, DateTime.UtcNow) < 30)
    .ToListAsync();
```

✨ **Example: Case-Insensitive Pattern Matching (PostgreSQL)**
```csharp
var redProductsIgnoreCase = await db.Products
    .Where(p => EF.Functions.ILike(p.Name, "%red%"))
    .ToListAsync();
```

---

## 10. Global Query Filters

Global query filters allow you to **automatically filter entities at the DbContext level**.



✨ **Example: Soft delete filter**
```csharp
modelBuilder.Entity<User>()
    .HasQueryFilter(u => !u.IsDeleted);
```

---

## 11. Enum & Value Conversions

EF Core allows mapping enums or custom value objects to database columns with conversions.

✨ **Example: Custom value object**
```csharp
modelBuilder.Entity<User>()
    .Property(u => u.Email)
    .HasConversion(
        v => v.Value,   // store as string
        v => new Email(v) // retrieve as Email object
    );
```

---

## 12. Common Pitfalls & Gotchas

* **N+1 Queries:** Forgetting `Include()` leading to parent-row-looping queries.
* **Client-Side Evaluation:** LINQ that can't translate to SQL gets executed in memory, causing bloat.
* **Tracking Overhead:** Default tracking adds memory/CPU load for read-only data.
* **Raw SQL Risks:** Always parameterize to prevent SQL injection.

💡 Use **logging, interceptors, and query tagging** to catch these pitfalls early. Seeing the exact SQL your LINQ generates makes debugging much easier.

Modern EF Core gives developers the tools to handle everything from **complex aggregates and nested relationships** to **performance-critical queries and semi-structured JSON data** — all while keeping code clean, expressive, and maintainable.
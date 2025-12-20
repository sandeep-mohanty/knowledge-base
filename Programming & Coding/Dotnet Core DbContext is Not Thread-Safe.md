# DbContext is Not Thread-Safe: Parallelizing EF Core Queries the Right Way

**Published:** 2025  
**Read time:** ~10 min  

---

## Table of Contents
- [Introduction](#introduction)
- [The False Promise of Task.WhenAll](#the-false-promise-of-taskwhenall)
- [The Solution: Using IDbContextFactory](#the-solution)
- [Key Concepts](#key-concepts)
- [The Benchmark](#the-benchmark)
- [Trade-offs & Conclusion](#trade-offs--conclusion)

---

## Introduction
We have all built *that* endpoint.  

You know the one: the **Executive Dashboard** or the **User Summary** screen. It needs to fetch three or four completely unrelated sets of data to paint a complete picture for the user:

- The last 50 orders  
- Current system health logs  
- User’s profile settings  
- Notification count  

### Standard Approach
```csharp
var orders = await GetRecentOrdersAsync(userId);
var logs = await GetSystemLogsAsync();
var stats = await GetUserStatsAsync(userId);

return new DashboardDto(orders, logs, stats);
```

This works, but it’s slow.  
If each call takes ~300–400ms, users wait **1 full second**. Running them in parallel should cut this to ~400ms.  

But with Entity Framework Core, the naive parallel approach **crashes**.

---

## The False Promise of Task.WhenAll
A common mistake is wrapping repository calls in tasks and awaiting them all at once:

```csharp
// ❌ DO NOT DO THIS
public async Task<DashboardData> GetDashboardData(int userId)
{
    // These methods all use the same injected _dbContext
    var ordersTask = _repository.GetOrdersAsync(userId);
    var logsTask = _repository.GetLogsAsync();
    var statsTask = _repository.GetStatsAsync(userId);

    await Task.WhenAll(ordersTask, logsTask, statsTask); // BOOM 💥

    return new DashboardData(ordersTask.Result, logsTask.Result, statsTask.Result);
}
```

This triggers the dreaded EF Core exception:

> A second operation started on this context before a previous operation completed.  
> This is usually caused by different threads using the same instance of DbContext, however instance members are not guaranteed to be thread safe.

### Why?
- `DbContext` is **not thread-safe**.  
- It maintains a **Change Tracker** and wraps a **single database connection**.  
- Multiple threads cannot share that connection simultaneously.  

---

## The Solution
Since **.NET 5**, EF Core introduced `IDbContextFactory<T>`.

Instead of injecting a scoped `DbContext`, inject a factory that creates **independent contexts** on demand.

### Registering the Factory
```csharp
// Program.cs
builder.Services.AddDbContextFactory<AppDbContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("db"));
});
```

### Refactored Service
```csharp
using Microsoft.EntityFrameworkCore;

public class DashboardService(IDbContextFactory<AppDbContext> contextFactory)
{
    public async Task<DashboardDto> GetDashboardAsync(int userId)
    {
        // Start tasks immediately
        var ordersTask = GetOrdersAsync(userId);
        var logsTask = GetSystemLogsAsync();
        var statsTask = GetUserStatsAsync(userId);

        // Wait for all
        await Task.WhenAll(ordersTask, logsTask, statsTask);

        // Return results
        return new DashboardDto(
            await ordersTask,
            await logsTask,
            await statsTask
        );
    }

    private async Task<List<Order>> GetOrdersAsync(int userId)
    {
        await using var context = await contextFactory.CreateDbContextAsync();
        return await context.Orders
            .AsNoTracking()
            .Where(o => o.UserId == userId)
            .OrderByDescending(o => o.CreatedAt)
            .ThenByDescending(o => o.Amount)
            .Take(50)
            .ToListAsync();
    }

    private async Task<List<SystemLog>> GetSystemLogsAsync()
    {
        await using var context = await contextFactory.CreateDbContextAsync();
        return await context.SystemLogs
            .AsNoTracking()
            .OrderByDescending(l => l.Timestamp)
            .Take(50)
            .ToListAsync();
    }

    private async Task<UserStats?> GetUserStatsAsync(int userId)
    {
        await using var context = await contextFactory.CreateDbContextAsync();
        return await context.Users
            .Where(u => u.Id == userId)
            .Select(u => new UserStats { OrderCount = u.Orders.Count })
            .FirstOrDefaultAsync();
    }
}
```

---

## Key Concepts
1. **Isolation**  
   Each task gets its own `DbContext` and database connection. No contention.  

2. **Disposal**  
   Using `await using` ensures the context is disposed immediately after use, returning the connection to the pool.  

---

## The Benchmark
Built with **.NET 10 + PostgreSQL**:

- **Sequential Execution:** ~36ms  
  Each query waits for the previous one.  

- **Parallel Execution:** ~13ms  
  All queries start together and complete together.  

---

## Trade-offs & Conclusion
`IDbContextFactory` bridges EF Core’s unit-of-work design with modern parallel requirements.  

⚠️ Use sparingly:
- **Connection pool starvation:** One request may consume multiple connections.  
- **Context overhead:** For very fast queries, creating multiple contexts may be slower.  

✅ Next time you see a slow dashboard, check your awaits. If they’re lined up sequentially, parallelization with `IDbContextFactory` might be the fix.


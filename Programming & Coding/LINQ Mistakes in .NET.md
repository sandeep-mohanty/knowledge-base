# 5 LINQ Mistakes in .NET Even Experienced Developers Make
**Because expressiveness does not mean efficiency**

### Introduction
LINQ is one of the best things to ever happen to C#. It made collection-heavy code readable, composable, and expressive. It also made it incredibly easy to hide expensive work behind clean-looking method chains.

That tradeoff is usually worth it. Until it isn’t.

Most performance issues caused by LINQ are not obvious mistakes. They are not “don’t use LINQ in a loop” level advice. They are subtle algorithmic choices that look reasonable in isolation and even pass code review because the code reads well. The problem is that LINQ abstracts **what** you want to do, not **how much work** it actually takes to do it.

This post is about five LINQ patterns that quietly turn cheap operations into expensive ones. Not because LINQ is bad, but because abstraction always leaks eventually. If you understand where it leaks, you can keep using LINQ confidently instead of swinging between overuse and paranoia.

---

### Sorting Everything Just to Read One…
This is one of the most expensive LINQ mistakes that still shows up in production code.

```csharp
var youngest = people
    .OrderBy(p => p.Age)
    .First();
```

At a glance, this looks harmless. You want the smallest value, so you sort and take the first element. The issue is that sorting is an **O(n log n)** operation, and you only needed an **O(n)** scan.



What makes this mistake dangerous is that the cost is invisible. LINQ does not warn you. The code does not look suspicious. But under load, especially on large collections, this becomes a real problem.

The correct approach is to use the APIs that express intent without unnecessary work:
```csharp
var youngest = people.MinBy(p => p.Age);
```
`MinBy` walks the sequence once and keeps track of the current best candidate. No sorting, no extra allocations, no wasted comparisons.

The same applies to `MaxBy`, and more generally to any scenario where you only care about a single extremum. If you ever see `OrderBy().First()` or `OrderByDescending().First()`, that is a red flag. You are paying for an entire ordering when you only need a comparison.

---

### Using GroupBy as a Hammer for Every Aggregation Problem
`GroupBy` is powerful, expressive, and frequently misused.

Consider this pattern:
```csharp
var totals = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        Total = g.Sum(o => o.Amount)
    })
    .ToList();
```

This works. It is readable. It is also doing more work than many people realize.

`GroupBy` builds groupings internally. That means it buffers elements, allocates grouping objects, and often materializes collections under the hood. If all you need is accumulation, a dictionary-based approach is usually cheaper and more explicit:

```csharp
var totals = new Dictionary<int, decimal>();
foreach (var order in orders)
{
    if (totals.TryGetValue(order.CustomerId, out var current))
    {
        totals[order.CustomerId] = current + order.Amount;
    }
    else
    {
        totals[order.CustomerId] = order.Amount;
    }
}
```

This version is boring, but it makes the algorithm explicit. One pass, no grouping allocations, no deferred execution surprises.

The mistake is not using `GroupBy`. The mistake is using it automatically, without thinking about whether you actually need grouping semantics or just aggregation. LINQ makes grouping easy, which encourages people to reach for it even when the problem does not require it.

---

### Accidentally Downgrading O(1) Operations to O(n)
This is one of the most subtle LINQ-related performance issues, and it has nothing to do with the operators themselves.

The problem starts when everything becomes `IEnumerable<T>`.

```csharp
IEnumerable<Order> orders = GetOrders();
if (orders.Count() > 0)
{
    // do something
}
```

If `orders` is backed by a `List<T>`, calling `Count` on the list is an **O(1)** operation. But once it is typed as `IEnumerable<T>`, LINQ has no guarantee that count is cheap. So it enumerates.

You just turned a constant-time check into a full traversal.

The same issue shows up with `Contains`, `Any`, and other operations that collections can do efficiently, but LINQ cannot assume they can.

The fix is not “never use LINQ”. The fix is to preserve collection types when it matters, or to check for collection interfaces explicitly:

```csharp
if (orders is ICollection<Order> collection)
{
    if (collection.Count > 0)
    {
        // fast path
    }
}
else if (orders.Any())
{
    // fallback
}
```

This is the kind of thing that only shows up after profiling, and once you see it, you start noticing it everywhere. LINQ is not slow by default, but abstraction boundaries can erase valuable information if you are not careful.

---

### Putting LINQ in Hot Paths Where Allocations Actually Matter
LINQ allocates. Not always a lot, but not zero either.

Every iterator, every captured lambda, every deferred execution chain introduces small allocations and state machines. In most application code, this is fine. In hot paths, it is not.

Consider something like this:
```csharp
foreach (var batch in batches)
{
    var validItems = batch
        .Where(IsValid)
        .Select(Transform)
        .ToArray();

    Process(validItems);
}
```

This code is clean and expressive. It also allocates a new iterator chain and array on every iteration. Under load, this shows up as **GC pressure**, not CPU usage, which makes it harder to diagnose.



A manual loop version may look less elegant, but it gives you control:

```csharp
var buffer = new List<Result>(batch.Count);

foreach (var item in batch)
{
    if (!IsValid(item))
        continue;
    buffer.Add(Transform(item));
}

Process(CollectionsMarshal.AsSpan(buffer));
buffer.Clear();
```

This is not about banning LINQ. It is about recognizing when LINQ’s elegance costs you more than it gives you. Hot paths are where abstractions go to die, and LINQ is no exception.

---

### Flattening with SelectMany When a Loop Would Be Better
`SelectMany` is one of the most misunderstood LINQ operators.

```csharp
var allItems = orders
    .SelectMany(o => o.Items)
    .Where(i => i.IsActive)
    .ToList();
```

This reads nicely, but it hides a nested iteration. In many cases, that is fine. In others, especially when additional conditions or early exits are needed, it becomes harder to reason about performance and control flow.

A loop makes the cost explicit:

```csharp
var allItems = new List<Item>();

foreach (var order in orders)
{
    foreach (var item in order.Items)
    {
        if (!item.IsActive)
            continue;
        allItems.Add(item);
    }
}
```

The advantage here is not raw speed, but clarity. You can add short-circuiting, batching, or limits without fighting the abstraction. `SelectMany` is excellent for pure projections. Once logic creeps in, the abstraction starts to work against you.

---

### Why This Matters
None of these mistakes are about LINQ being slow. They are about LINQ hiding algorithmic cost behind clean syntax.

The danger is not that LINQ makes code unreadable. The danger is that it makes expensive work look cheap. Once that happens, performance issues slip through reviews, testing, and even production for a long time before anyone notices.

Understanding these patterns does not mean abandoning LINQ. It means using it with intent, knowing when it is the right tool and when it is just a convenient one.

### Conclusion
LINQ is a power tool. Used well, it makes code clearer and more maintainable. Used blindly, it hides work you would never accept if it were written out explicitly.

The five patterns in this post are not exotic edge cases. They are the kinds of things that accumulate quietly in mature codebases and only surface when scale, load, or latency starts to matter.

If you can recognize these patterns early, you can keep writing expressive LINQ without paying accidental performance taxes later. That balance is what separates comfortable LINQ usage from confident LINQ usage.
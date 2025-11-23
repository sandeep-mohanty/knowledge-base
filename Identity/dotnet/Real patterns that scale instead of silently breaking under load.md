# Real patterns that scale instead of silently breaking under load

## Introduction
There is a moment in every .NET developer’s journey where async and concurrency stop feeling like a fancy language feature and start looking like a trap you can accidentally fall into. Writing `await` everywhere does not automatically make code scalable and throwing `Task` around does not mean it is asynchronous in any real sense.

The difference between “works on my laptop” and “handles real production load without turning the thread pool into smoke” often comes down to a handful of decisions that are easy to ignore, especially when the codebase is still small and nobody is benchmarking anything yet.

Let’s get right into it.

---

## 1. Use `ChannelWriter.TryWrite` for Fast, Non-blocking Message Publishing
Channels are one of the easiest ways to build producer and consumer pipelines. Many developers immediately reach for `await writer.WriteAsync(item)` because it feels natural in an async codebase. The problem is that waiting to publish is not always what you want. If the pipeline is temporarily slow, you might want to drop, buffer elsewhere, batch, backpressure, or log. Blocking the producer is often the worst possible choice.

**The wrong approach**
```csharp
await channel.Writer.WriteAsync(message);
```
If the consumer cannot keep up, this line suspends your producer indefinitely. In high-load systems, this can cause cascading stalls.

**The better approach**
```csharp
if (!channel.Writer.TryWrite(message))
{
    // handle overflow policy
    Log.Warning("Channel full, message dropped");
}
```
`TryWrite` gives you immediate feedback. You decide what backpressure strategy makes sense. This turns a fragile producer into a resilient one.

**Why it matters**  
Every scalable system requires clear backpressure rules. Unbounded async queues look pretty until they take down your server. `TryWrite` helps you reveal these policies instead of hiding them under scheduler behavior.

## 2. Prefer `WaitAsync` Instead of `Task.Wait` for Cancellation Aware Synchronization
A very common performance and stability mistake is mixing non-cancellable waiting with async tasks. When developers use `.Wait()` or `.Result`, they block a thread pool worker that could be doing useful work. Worse, they completely ignore cancellation tokens, which modern systems rely on for graceful shutdowns and timeouts.

**The wrong approach**
```csharp
task.Wait(); // blocks, ignores cancellation
```
This can freeze UI threads, generate thread starvation, and cause deadlocks depending on the synchronization context.

**The better approach**
```csharp
await task.WaitAsync(cancellationToken);
```
This respects cancellation, and because it is awaited, it does not block thread pool threads.

**Why it matters**  
Cancellation is not an optional feature. It is a correctness guarantee, a resource management tool, and a production safety net. You do not want threads stuck doing nothing when load spikes.

## 3. Use `TaskCreationOptions.AttachedToParent` Only When You Fully Understand the Hierarchy
Attached tasks are powerful, but misusing them can delay completion, compound exceptions, and produce debugging nightmares. Most developers have seen this flag but avoid it because it feels mysterious. The truth is that it is extremely useful in parallel workflows that must not complete until every child completes, but extremely dangerous when thrown in randomly.

**The wrong approach**
```csharp
var child = Task.Factory.StartNew(
    () => ProcessChunk(chunk),
    TaskCreationOptions.AttachedToParent
);
```
Many developers copy-paste this from old TPL examples without understanding the parent lifetime contract. If you attach a child task that never finishes or throws unexpectedly, the parent will stay incomplete longer than you think.

**The better mental model**  
Use it only when:
- The parent task represents a logical container workflow.  
- Every child must complete before the parent is considered finished.  
- There is a known bounded execution time.  
- Failure handling and rethrow policy is well defined.  

If you do not meet those rules, use a simple awaited task instead or a channel-based worker pool.

**Why it matters**  
`AttachedToParent` is not bad. It is precise tooling. When used correctly, it creates clean hierarchical joins. When misused, it becomes a hidden latch blocking entire flows without obvious logs.

## 4. Prefer `ValueTask` Only When Pooling or Reusing Work for Hot Async Paths
Some developers replace `Task` with `ValueTask` everywhere after hearing that it reduces heap allocations. That is not how `ValueTask` should be used. You should only use `ValueTask` in hot paths where returning synchronously is common or where pooling can be applied. If you wrap `Task` into a `ValueTask` or do not maintain usage rules, you will create more complexity than savings.

**The wrong approach**
```csharp
public ValueTask<string> GetNameAsync() =>
    new ValueTask<string>(Task.FromResult("Test"));
```
This gives you no benefit and adds unnecessary wrapping.

**The better approach**
```csharp
private static readonly ValueTask<string> Cached = new ValueTask<string>("Test");

public ValueTask<string> GetNameAsync(bool cached)
{
    if (cached)
        return Cached;

    return new ValueTask<string>(FetchFromDbAsync());
}
```

**Why it matters**  
`ValueTask` is about eliminating allocations when synchronous completion is likely. It is not a drop-in `Task` replacement. Used correctly, it keeps memory pressure low in very hot paths.


## 5. Never Mix `lock` With Asynchronous Code. Use `SemaphoreSlim` Instead
This is one of the most common async mistakes in code reviews. Developers try to protect shared mutable state using `lock` but the awaited code inside the block breaks the entire synchronization contract. The code compiles, but the lock is released early or used incorrectly because `lock` is not async-aware.

**The wrong approach**
```csharp
lock (_syncObj)
{
    await DoWorkAsync(); // dangerous
}
```
This is unsafe and misleading. Even worse, it can deadlock when contention occurs under async conditions.

**The better approach**
```csharp
await _semaphore.WaitAsync();
try
{
    await DoWorkAsync();
}
finally
{
    _semaphore.Release();
}
```

**Why it matters**  
Locks were built for synchronous sections. Async code changes the execution flow. Once you mix both, you no longer have predictable mutual exclusion.


# Conclusion
High-performance async code is less about sprinkling `await` everywhere and more about respecting the concurrency model. Real-world systems require responsible backpressure, cancellation, error policy, well-defined scheduling, and safe mutual exclusion. Each of these five techniques pushes your code away from fragile assumptions and closer to predictable, scalable behavior.

***
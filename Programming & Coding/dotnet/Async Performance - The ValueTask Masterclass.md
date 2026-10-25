# Async Performance: The ValueTask Masterclass

In high-throughput .NET applications, "allocations are the enemy." While `async/await` is essential for scalability, returning a standard `Task<T>` carries a hidden cost: **Heap Allocation.** This article explores how `ValueTask<T>` eliminates this overhead and how to implement advanced pooling for zero-allocation async code.

---

## 1. The "Task" Tax
Every time an `async Task<T>` method is called, a `Task` object is allocated on the heap. In high-frequency paths—like middleware, socket handling, or caching—this creates massive Garbage Collector (GC) pressure. 

Even if the result is already available (synchronous completion), a `Task<T>` must still be instantiated unless it’s a specifically cached task.



## 2. The Solution: `ValueTask<T>`
`ValueTask<T>` is a `readonly struct`. 
* **If the result is ready:** It wraps the value directly on the stack. **Zero allocations.**
* **If the result is pending:** It wraps a `Task<T>` (or an `IValueTaskSource`) and behaves normally.

### Comparison Table
| Feature | `Task<T>` | `ValueTask<T>` |
| :--- | :--- | :--- |
| **Type** | Class (Reference) | Struct (Value) |
| **Allocation (Sync)** | ~72 bytes | **0 bytes** |
| **Allocation (Async)** | ~72 bytes | ~80 bytes (slightly higher) |

---

## 3. Implementation Patterns

### Pattern A: The Fast-Path Cache
This is the most common use case. If the data is in memory, we avoid the heap entirely.

```csharp
public ValueTask<string> GetConfigValueAsync(string key)
{
    // Synchronous path: No allocation
    if (_inMemoryCache.TryGetValue(key, out var value))
    {
        return new ValueTask<string>(value);
    }

    // Asynchronous path: Allocates Task
    return new ValueTask<string>(FetchFromDbAsync(key));
}
```

### Pattern B: The Advanced "Pooled" Provider
To achieve zero allocations even on the **asynchronous path**, we use `IValueTaskSource` and a simple object pool. This pattern is used internally by .NET in `Socket` and `Stream` operations.



```csharp
using System.Threading.Tasks.Sources;
using System.Collections.Concurrent;

public class PooledDataProvider
{
    // A simple pool to reuse our state objects
    private readonly ConcurrentQueue<PooledSource> _pool = new();

    public ValueTask<int> GetDataAsync()
    {
        // 1. Try to complete synchronously
        if (TryGetSyncResult(out int fastResult))
        {
            return new ValueTask<int>(fastResult);
        }

        // 2. Rent a source from the pool for the async path
        if (!_pool.TryDequeue(out var source))
        {
            source = new PooledSource(this);
        }

        // 3. Start the async work
        source.StartWork();
        
        // Return the ValueTask backed by our pooled source
        return new ValueTask<int>(source, source.Version);
    }

    private void ReturnToPool(PooledSource source) => _pool.Enqueue(source);

    // Mock logic
    private bool TryGetSyncResult(out int res) { res = 0; return false; }

    private class PooledSource : IValueTaskSource<int>
    {
        private readonly PooledDataProvider _parent;
        private ManualResetValueTaskSourceCore<int> _core;

        public PooledSource(PooledDataProvider parent) => _parent = parent;

        public short Version => _core.Version;

        public void StartWork()
        {
            // Simulate async work (e.g., a network callback)
            Task.Run(async () => {
                await Task.Delay(100); 
                _core.SetResult(42); 
            });
        }

        // IValueTaskSource Implementation
        public int GetResult(short token)
        {
            try { return _core.GetResult(token); }
            finally 
            {
                _core.Reset();
                _parent.ReturnToPool(this); // Recycle the object
            }
        }

        public ValueTaskSourceStatus GetStatus(short token) => _core.GetStatus(token);
        public void OnCompleted(Action<object?> continuation, object? state, short token, ValueTaskSourceOnCompletedFlags flags) 
            => _core.OnCompleted(continuation, state, token, flags);
    }
}
```

---

## 4. The Rules of `ValueTask`
Because it is a struct, `ValueTask` has strict usage rules to avoid memory corruption:

1.  **Never await twice:** The underlying object may have been returned to a pool.
2.  **Never call `.Result` before completion:** It does not have the same blocking safety as `Task`.
3.  **Use `.AsTask()` for combinators:** If using `Task.WhenAll`, convert it first.

---

## 5. Prove It: The Benchmark
Using `BenchmarkDotNet`, we can see the impact of these optimizations.

```csharp
[MemoryDiagnoser]
public class ValueTaskBenchmark
{
    private readonly string _cachedData = "Hello World";

    [Benchmark]
    public async Task<string> UsingTask() => await Task.FromResult(_cachedData);

    [Benchmark]
    public async ValueTask<string> UsingValueTask() => await new ValueTask<string>(_cachedData);
}
```

### Typical Results
| Method | Mean | Allocated |
| :--- | :--- | :--- |
| **UsingTask** | 12.5 ns | **72 B** |
| **UsingValueTask** | 4.2 ns | **0 B** |

---

## Summary
* **Use `Task<T>`** for general-purpose APIs and methods that are always asynchronous.
* **Use `ValueTask<T>`** for high-performance "hot paths" where the method often completes synchronously.
* **Use `IValueTaskSource`** for the ultimate performance tier: zero-allocation async via pooling.

**Next Step:** Identify your app's most frequent async call. If it's called millions of times per day, converting it to a `ValueTask` could significantly reduce your cloud compute costs by lowering CPU usage associated with GC.
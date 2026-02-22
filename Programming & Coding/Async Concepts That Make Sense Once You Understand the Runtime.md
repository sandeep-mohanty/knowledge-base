# Async Concepts That Make Sense Once You Understand the Runtime
**Why TaskCompletionSource, custom awaiters, and continuations behave the way they do**

### Introduction
Once you move past everyday `async` and `await`, you eventually run into APIs that feel… uncomfortable. `TaskCompletionSource<T>`, `ValueTask`, custom awaiters, and flags like `RunContinuationsAsynchronously`.

They look sharp, easy to misuse, and poorly documented outside of reference source and blog posts written by runtime engineers. That discomfort usually comes from missing context. If you write libraries, infrastructure, or high-throughput systems, understanding these is not optional. It is how you stop fighting the runtime and start working with it.

---

### 1. TaskCompletionSource<T> Failure Modes and Race Conditions
`TaskCompletionSource<T>` exists to bridge callback-based or event-driven code into the `Task` world. Conceptually, it is simple: you hold a `TaskCompletionSource<T>`, expose the `Task`, and complete it later.

```csharp
var tcs = new TaskCompletionSource<int>();

DoWork(
    onSuccess: value => tcs.SetResult(value),
    onFailure: ex => tcs.SetException(ex)
);

return tcs.Task;
```

The important detail is this: **TaskCompletionSource<T> is a state machine**, and that state machine can only transition once. Calling `SetResult`, `SetException`, or `SetCanceled` more than once is a logic error. The runtime enforces this by throwing an `InvalidOperationException`.

The common failure mode is not manual repetition, but **races**. Multiple callbacks, timeouts racing with completions, or cancellation paths racing with success paths can all trigger this.

```csharp
var tcs = new TaskCompletionSource<int>();
RegisterCallback(result => tcs.TrySetResult(result));
RegisterTimeout(() => tcs.TrySetCanceled());
```
Notice the use of `TrySetXxx`. This acknowledges that the runtime cannot serialize these paths for you. Additionally, by default, continuations may run **inline** when you call `SetResult`, meaning user code might execute synchronously on your completion thread.

---

### 2. ManualResetValueTaskSourceCore<T> and Reusable Awaitables
`ValueTask` exists to avoid allocations, but it must point to an `IValueTaskSource<T>` when the operation isn't complete. `ManualResetValueTaskSourceCore<T>` is the low-level helper making reusable awaitables possible for high-scale types like `Socket` or `PipeReader`.



At its core, it is a manually managed async state machine:
```csharp
private ManualResetValueTaskSourceCore<int> _core;

public ValueTask<int> ReadAsync()
{
    _core.Reset();
    StartOperation();
    return new ValueTask<int>(this, _core.Version);
}
```
The runtime tracks correctness using a **version token**. Every reset increments the version. If you misuse the awaitable (await twice or after reuse), the runtime throws. Reusable awaitables trade safety for performance; you become responsible for the lifecycle rather than the runtime.

---

### 3. Custom Awaiters and GetAwaiter()
`await` is pattern-based, not special syntax. When the compiler sees `await something;`, it looks for a specific pattern:
1.  A `GetAwaiter()` method.
2.  An awaiter with an `IsCompleted` property.
3.  A `GetResult()` method.
4.  An `OnCompleted` or `UnsafeOnCompleted` hook.



This is how `Task`, `ValueTask`, and framework primitives integrate with the language. Awaiters decide whether a continuation runs synchronously and how it is scheduled.

---

### 4. Building Your Own Awaitable Types in C#
Once you understand the pattern, building awaitables is mechanical. A minimal awaiter looks like this:

```csharp
public readonly struct SimpleAwaiter : INotifyCompletion
{
    public bool IsCompleted => false;

    public void OnCompleted(Action continuation)
    {
        ThreadPool.QueueUserWorkItem(_ => continuation());
    }
   
    public void GetResult() { }
}
```

The runtime only cares about the protocol. However, mistakes here are dangerous: if your awaiter resumes synchronously when it shouldn't, you can cause **stack overflows or deadlocks**. At this level, you are participating in the scheduler.

---

### 5. Synchronous vs Asynchronous Continuations
By default, completing a task may execute continuations inline on your current thread. This is fast but risky:
* Arbitrary code runs on your thread.
* Deep call stacks are possible.
* Reentrancy bugs become likely.

This is why `TaskCompletionSource<T>` provides `TaskCreationOptions.RunContinuationsAsynchronously`.

```csharp
new TaskCompletionSource<int>(
    TaskCreationOptions.RunContinuationsAsynchronously
);
```
With this flag, continuations are queued rather than executed inline. You pay a small scheduling cost but gain predictability. High-throughput systems often prefer synchronous continuations for speed, while library code should almost never allow them.

---

### Conclusion
At this level, async stops being a convenience and becomes a runtime model. Tasks are not just promises; they are instructions for a complex state machine. Understanding how continuations are scheduled is the difference between a system that works and one that is performant.

Would you like me to find a **comparison table between Task and ValueTask performance metrics** to help you identify exactly when the allocation savings justify the added complexity?
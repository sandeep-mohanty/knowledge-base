# ExecutionContext vs SynchronizationContext in C#: The Async Internals Most Developers Never Learn
**How continuations are scheduled, how ambient state flows, and why await behaves the way it does.**

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*fp3DOaF2MxUSMcZDRrIF4g.png)
---

### Introduction
Most developers think `await` is about threads. **It is not.**

If you’re not a member, **I’ve got you covered!** ❤  
If you enjoy it, consider **clapping, subscribing, or buying me a coffee** to show your support! ❤

`await` is primarily about **continuations**: what happens after an asynchronous operation completes. Once you understand that, the next question becomes far more important:
*   Where does the continuation run?
*   What state follows the continuation?
*   Why do some apps deadlock while others do not?
*   Why does `AsyncLocal<T>` survive thread switches?
*   Why did `ConfigureAwait(false)` matter so much in older codebases?

The answer lives in two types many developers mention, but few truly understand: **SynchronizationContext** and **ExecutionContext**. They sound related, but they are not. They solve different problems and interact with `await` in different ways.

---

### Two Contexts, Two Entirely Different Jobs
If you remember only one thing, remember this:

| Context | Responsibility |
| :--- | :--- |
| **SynchronizationContext** | Controls **where** a continuation runs (Dispatch). |
| **ExecutionContext** | Controls **what logical ambient state** flows with execution (Data). |



---

### What Is SynchronizationContext Really?
`SynchronizationContext` is an abstraction for scheduling work onto some environment. It exposes methods like:
*   `Post(SendOrPostCallback callback, object? state)`
*   `Send(SendOrPostCallback callback, object? state)`

Think of it as: **“If code needs to resume, where should I dispatch it?”**

#### Implementations by Application Model:
*   **WinForms/WPF:** Continuation resumes on the UI thread/Dispatcher.
*   **Legacy ASP.NET:** Continuation resumes inside the request context.
*   **Console App / ASP.NET Core:** Usually **no custom synchronization context** exists; continuations run on the ThreadPool.



---

### What Is ExecutionContext Really?
`ExecutionContext` carries **logical call state** across asynchronous boundaries. It is not about *where* code runs, but *what* state accompanies it.

#### What it carries:
*   `AsyncLocal<T>` values
*   Current culture
*   Security principal
*   Tracing/Activity context
*   Impersonation data



Before async code became common, thread-local storage worked. But once continuations started hopping threads, thread-local assumptions broke. `ExecutionContext` ensures that when execution continues elsewhere, your ambient data remains visible.

---

### await and the Two Contexts
When you write `await SomeOperationAsync();`, the runtime performs two distinct actions:

1.  **Continuation Placement:** The runtime may capture the current `SynchronizationContext` so it knows where to resume. (**Where**)
2.  **Ambient State Flow:** The runtime flows the `ExecutionContext` so logical state remains visible. (**What**)

#### The AsyncLocal Proof
```csharp
private static readonly AsyncLocal<string> _user = new();
_user.Value = "Lorem Ipsum";
await Task.Delay(100);
Console.WriteLine(_user.Value); // Prints "Lorem Ipsum"
```
Even if the continuation resumes on a different thread, the value survives because `ExecutionContext` flowed the logical state.

---

### ConfigureAwait(false) and the Most Common Misunderstanding
Many developers assume `await Task.Delay(100).ConfigureAwait(false);` disables all context capture. **It does not.**

*   It primarily affects **SynchronizationContext**. It tells the runtime not to marshal back to the captured sync context.
*   It **does not** stop `ExecutionContext` flow.

This is why `AsyncLocal<T>` still works even after a `ConfigureAwait(false)` call.

---

### Why Deadlocks Happened
Classic UI/Legacy ASP.NET deadlock:
```csharp
public string GetData() => GetDataAsync().Result; // Blocks UI thread

public async Task<string> GetDataAsync() {
    await Task.Delay(1000); // Tries to resume on UI thread
    return "done";
}
```
1.  Caller blocks the UI thread.
2.  `await` completes and wants to resume on the captured UI `SynchronizationContext`.
3.  The UI thread is already blocked waiting for the `.Result`.
4.  **Deadlock.**



---

### Why ASP.NET Core Changed the Story
ASP.NET Core **does not install a SynchronizationContext**. Continuations generally do not need to marshal back to a specific request thread. This eliminates a whole class of deadlocks and is why `ConfigureAwait(false)` is no longer mandatory for every line of server-side code in modern .NET.

---

### Expert-Level: Suppressing ExecutionContext Flow
Sometimes flowing ambient state has a performance cost or is undesirable. You can use:
*   `ExecutionContext.SuppressFlow()`
*   `ThreadPool.UnsafeQueueUserWorkItem(...)`

**Why "unsafe"?** Because you lose logical state like tracing, correlation IDs, and `AsyncLocal` values. These are performance tools for experts, not defaults.

---

### Practical Rules for Modern C#
1.  **Stop thinking await means "new thread":** It means continuation.
2.  **Use ConfigureAwait(false) intentionally:** Know that it targets the sync context, not the execution context.
3.  **Use AsyncLocal<T> sparingly:** It's powerful, but easy to overuse.
4.  **Avoid blocking async code:** Especially in environments with thread affinity.
5.  **Monitor Context Flow:** If correlation IDs disappear in your logs, you likely have a context flow suppression issue.

### Conclusion
`SynchronizationContext` decides **where** continuations resume. `ExecutionContext` decides **what** logical state flows with them. One is dispatch; one is ambient state. Understanding this machinery transforms "async magic" into predictable engineering.
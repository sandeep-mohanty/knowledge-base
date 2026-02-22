# Java Virtual Threads vs CompletableFuture

### Custom Benchmark (Project Github)

**======== WARM UP RESULTS (IGNORE) ========**
* Starting: CompletableFuture IO -> completed in 4614 ms
* Starting: VirtualThread IO -> completed in 129 ms
* Starting: CompletableFuture CPU -> completed in 74056 ms
* Starting: VirtualThread CPU -> completed in 73110 ms

**======== REAL RUN (CONSIDER) ========**
* Starting: CompletableFuture IO -> **completed in 4581 ms**
* Starting: VirtualThread IO -> **completed in 112 ms**
* Starting: CompletableFuture CPU -> completed in 73937 ms
* Starting: VirtualThread CPU -> completed in 75526 ms

---

## From First Principles to Architecture Decisions
Java concurrency has always been powerful, but rarely simple. For years, Java developers were forced to choose between:
1. Writing simple synchronous code that does not scale.
2. Writing scalable asynchronous code that is difficult to read, debug, and maintain.

With the introduction of **Virtual Threads**, Java fundamentally changed this tradeoff. At the same time, `CompletableFuture` remains one of the most important concurrency abstractions in modern Java, especially for parallel and non-blocking workloads.

---

## The Original Java Thread Model and Its Limits
A traditional Java thread maps directly to an operating system (OS) thread. This means:
* Each thread allocates a fixed-size stack (often several megabytes).
* Creating threads is expensive.
* Context switching is expensive.
* **Blocking I/O wastes threads.**



To work around this, Java applications evolved patterns such as fixed-size thread pools, asynchronous callbacks, and reactive frameworks. All of these were workarounds for one problem: **Threads are expensive, but concurrency demands many of them.**

---

## CompletableFuture Was the First Major Escape Hatch
`CompletableFuture` was introduced to help Java developers avoid blocking threads. Instead of waiting for a result, you describe what should happen when the result becomes available.

### A Simple CompletableFuture Example
```java
CompletableFuture<User> userFuture =
    CompletableFuture.supplyAsync(() -> userService.fetchUser(id));

CompletableFuture<Order> orderFuture =
    CompletableFuture.supplyAsync(() -> orderService.fetchOrders(id));

CompletableFuture<AccountSummary> summaryFuture =
    userFuture.thenCombine(orderFuture,
        (user, orders) -> new AccountSummary(user, orders));

summaryFuture.thenAccept(summary -> {
    response.send(summary);
});
```

**The Cognitive Cost:**
`CompletableFuture` introduces **inverted control flow**. Business logic is fragmented across lambdas, and stack traces no longer reflect logical execution order. It optimizes for **non-blocking execution, not for human understanding.**

---

## Virtual Threads Change the Model Entirely
Virtual Threads do not try to make blocking faster; **they make blocking cheap.** A virtual thread is a lightweight thread managed by the JVM, not permanently bound to an OS thread.

### Creating a Virtual Thread
```java
Thread.startVirtualThread(() -> {
    User user = userService.fetchUser(id);
    Order orders = orderService.fetchOrders(id);
    AccountSummary summary = new AccountSummary(user, orders);
    response.send(summary);
});
```
This code scales to millions of concurrent executions because the JVM handles the suspension and resumption.

---

## How Virtual Threads Work Internally: Carrier Threads
Virtual threads run on a small pool of OS threads called **carrier threads**.



**Execution flow:**
1. A virtual thread is scheduled on a carrier thread.
2. When it reaches a blocking I/O operation, the JVM **parks** the virtual thread.
3. Its stack is saved to the heap, and the carrier thread is freed to do other work.
4. When I/O completes, the virtual thread is scheduled again.

**Important Limitation:** Virtual threads can be "pinned" to the carrier thread if they enter long `synchronized` sections or perform native blocking calls, preventing efficient parking.

---

## Performance Characteristics Compared

| Feature | Virtual Threads | CompletableFuture |
| :--- | :--- | :--- |
| **Primary Goal** | High Concurrency (I/O) | Parallelism (CPU/Async) |
| **Programming Model** | Sequential / Imperative | Functional / Declarative |
| **Blocking** | Encouraged (it's cheap) | Forbidden (blocks threads) |
| **Stack Traces** | Readable and complete | Fragmented / Hard to trace |
| **Resource Usage** | Very low per thread | Efficient via callbacks |



---

## When to Choose What?

### Choose Virtual Threads if:
* Your bottleneck is **I/O** (JDBC, REST, File I/O).
* You want simple, maintainable code.
* You need massive concurrency for request-response systems (web servers, microservices).

### Choose CompletableFuture if:
* Work is **CPU-intensive**.
* You need fine-grained async control or parallel data processing.
* You are building event-driven systems or real-time analytics.

---

## The Hybrid Model
Modern Java architectures often combine both. Use **Virtual Threads** for request-level concurrency and **CompletableFuture** inside for parallel CPU work.

```java
Thread.startVirtualThread(() -> {
    CompletableFuture<ResultA> a = CompletableFuture.supplyAsync(this::computeA);
    CompletableFuture<ResultB> b = CompletableFuture.supplyAsync(this::computeB);

    Result result = combine(a.join(), b.join()); // Blocks V-Thread safely
    send(result);
});
```

### Final Thought
Virtual Threads do not make `CompletableFuture` obsolete; they make **unnecessary complexity** obsolete. 

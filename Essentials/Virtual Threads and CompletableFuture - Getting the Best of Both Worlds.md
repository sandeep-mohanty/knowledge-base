# Virtual Threads & CompletableFuture: Getting the Best of Both Worlds

> **Target JDK:** Java 21+ (JEP 444) · Concepts apply through Java 25+  
> **Prerequisites:** Familiarity with basic Java threading, lambdas, and `ExecutorService`

---

## Table of Contents

1. [The Big Picture: Why Combine Both?](#1-the-big-picture-why-combine-both)
2. [Quick Refresher: What Each Tool Brings](#2-quick-refresher-what-each-tool-brings)
3. [The Virtual Thread Executor: The Bridge](#3-the-virtual-thread-executor-the-bridge)
4. [Step 1 — Replacing the Default ForkJoinPool](#4-step-1--replacing-the-default-forkjoinpool)
5. [Step 2 — Chaining Stages on Virtual Threads](#5-step-2--chaining-stages-on-virtual-threads)
6. [Step 3 — Fan-Out with `allOf` and `anyOf`](#6-step-3--fan-out-with-allof-and-anyof)
7. [Step 4 — Writing Blocking Code Freely](#7-step-4--writing-blocking-code-freely)
8. [Step 5 — Error Handling Patterns](#8-step-5--error-handling-patterns)
9. [Step 6 — Timeouts Done Right](#9-step-6--timeouts-done-right)
10. [Step 7 — Converting Legacy `Future` to `CompletableFuture`](#10-step-7--converting-legacy-future-to-completablefuture)
11. [Step 8 — Bounding Concurrency with Semaphores](#11-step-8--bounding-concurrency-with-semaphores)
12. [Pitfalls to Avoid](#12-pitfalls-to-avoid)
13. [Thread Pinning and How to Prevent It](#13-thread-pinning-and-how-to-prevent-it)
14. [Spring Boot Integration](#14-spring-boot-integration)
15. [Decision Guide: When to Use What](#15-decision-guide-when-to-use-what)
16. [Full Real-World Example](#16-full-real-world-example)
17. [Summary Cheat Sheet](#17-summary-cheat-sheet)

---

## 1. The Big Picture: Why Combine Both?

Virtual threads and `CompletableFuture` are often positioned as competing tools, but they are genuinely complementary when used thoughtfully.

**Virtual threads** (JEP 444, Java 21) give you:
- Millions of cheap, JVM-managed threads.
- The ability to block freely without consuming OS threads.
- Simple, imperative, readable code for async logic.

**`CompletableFuture`** (since Java 8) gives you:
- A fluent, declarative API for composing async pipelines.
- First-class support for fan-out (`allOf`, `anyOf`), transformation (`thenApply`), and sequencing (`thenCompose`).
- Built-in error-handling stages (`exceptionally`, `handle`).
- Timeout support (`orTimeout`, `completeOnTimeout`).

The problem with using `CompletableFuture` alone is that its default executor — the common `ForkJoinPool` — is sized for CPU-bound work and is a shared global resource. Blocking inside it (for I/O, JDBC, HTTP, etc.) starves it. The problem with virtual threads alone is that composing dependent async steps requires hand-rolled code.

The solution: **use `CompletableFuture`'s composability, but back it entirely with a virtual-thread executor**. You get declarative pipelines that can block freely, at massive scale, without callback hell.

```
                ┌─────────────────────────────────────────────┐
                │         Your Application Code               │
                │                                             │
                │  CompletableFuture API (composition,        │
                │  chaining, error handling, timeouts)        │
                │                                             │
                └──────────────────┬──────────────────────────┘
                                   │ backed by
                ┌──────────────────▼──────────────────────────┐
                │   Executors.newVirtualThreadPerTaskExecutor()│
                │   (one cheap virtual thread per task,        │
                │    blocks without pinning OS threads)        │
                └─────────────────────────────────────────────┘
```

---

## 2. Quick Refresher: What Each Tool Brings

### `CompletableFuture` API at a Glance

| Method | Purpose |
|---|---|
| `supplyAsync(supplier, executor)` | Start an async computation returning a value |
| `runAsync(runnable, executor)` | Start an async computation with no return value |
| `thenApply(fn)` | Transform result (same thread) |
| `thenApplyAsync(fn, executor)` | Transform result (on given executor) |
| `thenCompose(fn)` | Flat-map: chain a dependent `CompletableFuture` |
| `thenCombine(other, fn)` | Combine results of two independent futures |
| `allOf(futures...)` | Wait for ALL futures to complete |
| `anyOf(futures...)` | Complete as soon as ANY one completes |
| `exceptionally(fn)` | Recover from an exception |
| `handle(fn)` | Handle both result and exception in one stage |
| `orTimeout(duration, unit)` | Complete exceptionally with `TimeoutException` |
| `completeOnTimeout(value, duration, unit)` | Complete with a fallback value on timeout |

### Virtual Thread Creation

```java
// Option 1: One new virtual thread per submitted task (most common)
ExecutorService vte = Executors.newVirtualThreadPerTaskExecutor();

// Option 2: Start a single named virtual thread manually
Thread vt = Thread.ofVirtual().name("my-vt").start(() -> doWork());

// Option 3: Build a ThreadFactory for custom wiring
ThreadFactory vtf = Thread.ofVirtual().name("worker-", 0).factory();
```

---

## 3. The Virtual Thread Executor: The Bridge

The single most important line in this tutorial:

```java
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

This executor spawns a **brand new virtual thread** for every `submit()` or task dispatch. Because virtual threads are cheap (microseconds to create, ~few KB of memory), this is fine even for hundreds of thousands of tasks. Crucially, **there is no pool to size** — you never need to tune thread counts again for I/O-bound work.

Pass this executor as the second argument to any `CompletableFuture` factory method, and every stage runs on its own virtual thread, free to block without harming the rest of the JVM.

```java
// ✅ Every stage gets a fresh virtual thread; blocking is free
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {

    CompletableFuture<String> cf = CompletableFuture
        .supplyAsync(() -> callSlowApi(), executor)
        .thenApplyAsync(result -> transform(result), executor)
        .thenApplyAsync(result -> format(result), executor);

    System.out.println(cf.join());
}
```

> **Tip:** The executor implements `AutoCloseable` in Java 21+. Always use it in a `try-with-resources` block to ensure all virtual threads are cleaned up when you're done.

---

## 4. Step 1 — Replacing the Default ForkJoinPool

### The Problem

By default, `CompletableFuture.supplyAsync(supplier)` runs on `ForkJoinPool.commonPool()`. This pool:
- Is shared across the entire JVM (including parallel streams, etc.).
- Is sized to `Runtime.getRuntime().availableProcessors() - 1` threads.
- Should only run **short, non-blocking, CPU-bound** tasks.

Calling a database, HTTP endpoint, or file system inside it is a bug.

```java
// ❌ BAD: Blocking I/O inside the common ForkJoinPool
CompletableFuture<String> bad = CompletableFuture.supplyAsync(() -> {
    return httpClient.get("https://api.example.com/data"); // BLOCKS a shared thread!
});
```

### The Fix

```java
// ✅ GOOD: Blocking I/O on a virtual-thread executor
ExecutorService vte = Executors.newVirtualThreadPerTaskExecutor();

CompletableFuture<String> good = CompletableFuture.supplyAsync(() -> {
    return httpClient.get("https://api.example.com/data"); // fine — virtual thread
}, vte);
```

### Creating a Shared Application-Wide Executor

For most applications, create a single virtual-thread executor and share it:

```java
public class AppExecutors {

    // Shared executor for all I/O-bound async work
    public static final ExecutorService VIRTUAL = 
        Executors.newVirtualThreadPerTaskExecutor();

    // Separate bounded pool for CPU-intensive tasks (e.g., image processing)
    public static final ExecutorService CPU_BOUND =
        Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
}
```

Use them consistently:

```java
CompletableFuture<String>  ioBound  = CompletableFuture.supplyAsync(() -> fetchFromDb(), AppExecutors.VIRTUAL);
CompletableFuture<Integer> cpuBound = CompletableFuture.supplyAsync(() -> crunchNumbers(), AppExecutors.CPU_BOUND);
```

---

## 5. Step 2 — Chaining Stages on Virtual Threads

### `thenApply` vs `thenApplyAsync`

There are three variants of every chaining method:

```java
// thenApply(fn)             — runs on whichever thread completed the prior stage
// thenApplyAsync(fn)        — runs on the default ForkJoinPool (avoid for I/O!)
// thenApplyAsync(fn, exec)  — runs on your given executor ✅
```

When combining with virtual threads, **always prefer the three-argument `Async` variant** for stages that do any I/O or blocking:

```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

    CompletableFuture<Report> report = CompletableFuture
        // Stage 1: fetch raw data from DB (I/O — use virtual thread)
        .supplyAsync(() -> db.query("SELECT ..."), exec)

        // Stage 2: enrich with an HTTP call (I/O — use virtual thread)
        .thenApplyAsync(rows -> enrichWithExternalData(rows), exec)

        // Stage 3: pure transformation — CPU-only, can use thenApply
        .thenApply(enriched -> Report.from(enriched))

        // Stage 4: save result (I/O — use virtual thread)
        .thenApplyAsync(r -> { repository.save(r); return r; }, exec);

    System.out.println(report.join());
}
```

### `thenCompose` for Dependent Async Calls

Use `thenCompose` (not `thenApply`) when each stage itself returns a `CompletableFuture`. This avoids the dreaded `CompletableFuture<CompletableFuture<T>>` nesting:

```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

    CompletableFuture<String> result = CompletableFuture
        .supplyAsync(() -> getUserId(username), exec)
        // Returns CF<Profile> not CF<CF<Profile>> — flat and clean
        .thenComposeAsync(userId ->
            CompletableFuture.supplyAsync(() -> fetchProfile(userId), exec), exec)
        .thenComposeAsync(profile ->
            CompletableFuture.supplyAsync(() -> renderPage(profile), exec), exec);

    System.out.println(result.join());
}
```

---

## 6. Step 3 — Fan-Out with `allOf` and `anyOf`

Fan-out is where this combination really shines. Launch many independent I/O calls in parallel — each in its own virtual thread — then collect all results.

### `allOf` — All Tasks Must Complete

```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

    // Fan out: three independent service calls
    CompletableFuture<User>     userFuture     = CompletableFuture.supplyAsync(() -> userService.get(id), exec);
    CompletableFuture<Orders>   ordersFuture   = CompletableFuture.supplyAsync(() -> orderService.get(id), exec);
    CompletableFuture<Wishlist> wishlistFuture = CompletableFuture.supplyAsync(() -> wishService.get(id), exec);

    // Wait for all to complete
    CompletableFuture.allOf(userFuture, ordersFuture, wishlistFuture).join();

    // Collect results — all futures are already completed, .join() returns instantly
    Dashboard dashboard = new Dashboard(
        userFuture.join(),
        ordersFuture.join(),
        wishlistFuture.join()
    );
}
```

### Fan-Out Over a Dynamic List

```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

    List<String> productIds = List.of("p1", "p2", "p3", "p4", "p5");

    List<CompletableFuture<ProductDetails>> futures = productIds.stream()
        .map(pid -> CompletableFuture.supplyAsync(() -> catalogService.fetch(pid), exec))
        .toList();

    // Wait for all, then collect
    List<ProductDetails> allDetails = CompletableFuture
        .allOf(futures.toArray(CompletableFuture[]::new))
        .thenApply(v -> futures.stream().map(CompletableFuture::join).toList())
        .join();

    allDetails.forEach(System.out::println);
}
```

### `anyOf` — Race Pattern (First to Succeed Wins)

```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

    // Try three regional replicas — use whichever responds first
    CompletableFuture<String> eu   = CompletableFuture.supplyAsync(() -> callEuReplica(), exec);
    CompletableFuture<String> us   = CompletableFuture.supplyAsync(() -> callUsReplica(), exec);
    CompletableFuture<String> asia = CompletableFuture.supplyAsync(() -> callAsiaReplica(), exec);

    // anyOf returns Object — cast needed
    String fastest = (String) CompletableFuture.anyOf(eu, us, asia).join();
    System.out.println("Fastest replica responded: " + fastest);
}
```

> **Note:** With `anyOf`, the other futures continue running in their virtual threads until completion. Since virtual threads are cheap and auto-managed, this is acceptable for most cases. For strict cancellation semantics, prefer `StructuredTaskScope` with `Joiner.anySuccessfulOrThrow()`.

---

## 7. Step 4 — Writing Blocking Code Freely

One of the biggest benefits is that you can write **simple, imperative, blocking code** inside your `CompletableFuture` stages. No callbacks, no reactive operators, no `.flatMap()` gymnastics.

### Before (reactive-style, with platform threads)

```java
// ❌ Complex reactive pipeline — hard to read and debug
CompletableFuture
    .supplyAsync(info::getUrl, platformPool)
    .thenCompose(url  -> getBodyAsync(url, ofString()))
    .thenApply(info::findImage)
    .thenCompose(url  -> getBodyAsync(url, ofByteArray()))
    .thenApply(info::setImageData)
    .thenAccept(this::process)
    .exceptionally(t -> { t.printStackTrace(); return null; });
```

### After (imperative-style, with virtual threads)

```java
// ✅ Simple, readable blocking code — runs on a virtual thread
CompletableFuture.runAsync(() -> {
    try {
        String page      = getBody(info.getUrl(), ofString());     // blocks freely
        String imageUrl  = info.findImage(page);
        byte[] imageData = getBody(imageUrl, ofByteArray());       // blocks freely
        info.setImageData(imageData);
        process(info);
    } catch (Exception e) {
        e.printStackTrace();
    }
}, Executors.newVirtualThreadPerTaskExecutor());
```

The second version reads like sequential code, is trivially debuggable with a stack trace, and scales just as well — because the virtual thread yields the carrier thread whenever it blocks.

---

## 8. Step 5 — Error Handling Patterns

`CompletableFuture` provides several error-handling stages. Here's how to use them effectively with virtual threads.

### `exceptionally` — Recover from a Specific Failure

```java
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> callExternalService(), exec)
    .exceptionally(ex -> {
        log.warn("Service call failed: {}", ex.getMessage());
        return "fallback-value"; // substitutes the failed value
    });
```

### `handle` — Inspect Both Success and Failure

```java
CompletableFuture<Response> result = CompletableFuture
    .supplyAsync(() -> userService.findById(id), exec)
    .handle((user, ex) -> {
        if (ex != null) {
            // ex is the exception (wrapped in CompletionException)
            return Response.error("User not found: " + ex.getCause().getMessage());
        }
        return Response.ok(user);
    });
```

### `whenComplete` — Side-Effects on Completion (logging, metrics)

```java
CompletableFuture<Order> order = CompletableFuture
    .supplyAsync(() -> orderService.fetch(orderId), exec)
    .whenComplete((result, ex) -> {
        if (ex != null) {
            metrics.increment("order.fetch.error");
        } else {
            metrics.increment("order.fetch.success");
        }
        // Note: whenComplete does NOT change the result — it passes through
    });
```

### Chaining Multiple Error Handlers

```java
CompletableFuture<String> pipeline = CompletableFuture
    .supplyAsync(() -> primarySource.fetch(), exec)
    .exceptionallyComposeAsync(
        ex -> CompletableFuture.supplyAsync(() -> fallbackSource.fetch(), exec), exec)
    .exceptionally(ex -> "hardcoded-last-resort"); // last line of defense
```

> **Rule:** Always attach an `exceptionally` or `handle` stage at the **end** of every pipeline. Unhandled async exceptions are silently swallowed by default.

---

## 9. Step 6 — Timeouts Done Right

`CompletableFuture` has built-in timeout support since Java 9 — no need to block on `get(timeout, unit)`.

### `orTimeout` — Complete Exceptionally on Timeout

```java
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> slowService.getData(), exec)
    .orTimeout(3, TimeUnit.SECONDS)  // throws TimeoutException after 3s
    .exceptionally(ex -> {
        if (ex.getCause() instanceof TimeoutException) {
            return "timed-out-fallback";
        }
        return "error-fallback";
    });
```

### `completeOnTimeout` — Supply a Default Value on Timeout

```java
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> recommendationEngine.getSuggestions(userId), exec)
    .completeOnTimeout("default-recommendations", 2, TimeUnit.SECONDS);
    // If it takes more than 2 seconds, quietly returns the default
```

### Setting a Timeout on Fan-Out

```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

    List<CompletableFuture<String>> futures = services.stream()
        .map(svc -> CompletableFuture
            .supplyAsync(svc::call, exec)
            .completeOnTimeout("N/A", 1, TimeUnit.SECONDS))
        .toList();

    CompletableFuture.allOf(futures.toArray(CompletableFuture[]::new)).join();

    List<String> results = futures.stream().map(CompletableFuture::join).toList();
    // Any service that timed out contributes "N/A" to the list
}
```

---

## 10. Step 7 — Converting Legacy `Future` to `CompletableFuture`

Many older libraries (including the JDK's own `ExecutorService`) return `Future<T>` instead of `CompletableFuture<T>`. Bridging them used to require polling or blocking a pool thread. Virtual threads make this clean:

### Pattern: Bridge a `Future` using a Virtual Thread

```java
public static <T> CompletableFuture<T> toCompletableFuture(Future<T> future) {
    CompletableFuture<T> cf = new CompletableFuture<>();

    // Spin up a virtual thread whose ONLY job is to block on future.get()
    // This is free — it's a virtual thread, not a platform thread.
    Thread.ofVirtual().start(() -> {
        try {
            cf.complete(future.get());           // blocks the virtual thread, not an OS thread
        } catch (ExecutionException e) {
            cf.completeExceptionally(e.getCause());
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            cf.completeExceptionally(e);
        }
    });

    return cf;
}
```

Usage:

```java
Future<String> legacyFuture = legacyClient.fetchAsync(query);

// Now it's composable
CompletableFuture<String> cf = toCompletableFuture(legacyFuture)
    .thenApplyAsync(result -> processResult(result), exec)
    .exceptionally(ex -> "fallback");
```

> **Why this works now:** Before virtual threads, blocking a thread inside `future.get()` was expensive. With a virtual thread, blocking is essentially free — the carrier thread is released immediately, so creating one virtual thread per legacy `Future` scales to millions.

---

## 11. Step 8 — Bounding Concurrency with Semaphores

`Executors.newVirtualThreadPerTaskExecutor()` spawns an **unlimited** number of virtual threads. While they are cheap, unbounded concurrency can still exhaust downstream resources such as database connection pools, rate-limited APIs, or file descriptors.

**Do not** use a `FixedThreadPool` to limit virtual thread concurrency — that defeats the purpose. Use a `Semaphore` instead:

```java
// Allow at most 20 concurrent calls to the downstream database
private static final Semaphore DB_SEMAPHORE = new Semaphore(20);

CompletableFuture<List<Order>> fetchOrders(List<Long> ids) {
    try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {

        List<CompletableFuture<Order>> futures = ids.stream()
            .map(id -> CompletableFuture.supplyAsync(() -> {
                DB_SEMAPHORE.acquireUninterruptibly(); // blocks virtual thread (not OS thread)
                try {
                    return orderRepository.findById(id); // DB call
                } finally {
                    DB_SEMAPHORE.release();
                }
            }, exec))
            .toList();

        return CompletableFuture
            .allOf(futures.toArray(CompletableFuture[]::new))
            .thenApply(v -> futures.stream().map(CompletableFuture::join).toList());
    }
}
```

The semaphore blocks virtual threads that exceed the limit, but since a blocked virtual thread releases its carrier, the OS thread is free to run other virtual threads. **You get back-pressure without wasting OS threads.**

---

## 12. Pitfalls to Avoid

### ❌ Pitfall 1: Using `thenApplyAsync` without specifying an executor

```java
// ❌ Stages after the first run on ForkJoinPool.commonPool()
CompletableFuture
    .supplyAsync(() -> fetchData(), virtualExec)     // virtual thread ✅
    .thenApplyAsync(data -> parseData(data))          // ForkJoinPool! ❌
    .thenApplyAsync(parsed -> saveData(parsed));      // ForkJoinPool! ❌
```

```java
// ✅ All stages use the virtual-thread executor
CompletableFuture
    .supplyAsync(() -> fetchData(),   exec)
    .thenApplyAsync(data   -> parseData(data), exec)
    .thenApplyAsync(parsed -> saveData(parsed), exec);
```

### ❌ Pitfall 2: Pooling virtual threads

```java
// ❌ Never do this — it negates every benefit of virtual threads
ExecutorService wrongPool = Executors.newFixedThreadPool(100,
    Thread.ofVirtual().factory()); // virtual threads in a fixed pool
```

Virtual threads are designed to be created per task and discarded. They are not meant to be pooled. Use `newVirtualThreadPerTaskExecutor()`.

### ❌ Pitfall 3: Storing large objects in `ThreadLocal`

```java
// ❌ With millions of virtual threads, this can exhaust heap space
ThreadLocal<LargeDataStructure> cache = new ThreadLocal<>();
```

Each virtual thread gets its own `ThreadLocal` copy. With millions of threads, this adds up fast. Use `ScopedValue` (finalized in Java 25) for thread-local data that needs to flow through virtual threads.

### ❌ Pitfall 4: Calling `.get()` without a timeout on the main thread

```java
// ❌ Can block forever if the virtual thread hangs
String result = future.get(); // no timeout!
```

```java
// ✅ Always use orTimeout or a timed get()
String result = future.orTimeout(5, TimeUnit.SECONDS).join();
// or
String result = future.get(5, TimeUnit.SECONDS);
```

### ❌ Pitfall 5: Forgetting to handle exceptions at the end of a chain

```java
// ❌ If processData() throws, the exception is silently swallowed
CompletableFuture.supplyAsync(() -> fetchData(), exec)
    .thenApplyAsync(data -> processData(data), exec);
    // no exceptionally() — exception disappears into the void
```

```java
// ✅ Always terminate chains with error handling
CompletableFuture.supplyAsync(() -> fetchData(), exec)
    .thenApplyAsync(data -> processData(data), exec)
    .exceptionally(ex -> { log.error("Pipeline failed", ex); return null; });
```

---

## 13. Thread Pinning and How to Prevent It

Thread pinning is the most important virtual thread caveat to understand. A virtual thread becomes **pinned** to its carrier OS thread (preventing other virtual threads from using it) in two cases:

1. The virtual thread runs code inside a `synchronized` block **and** calls a blocking operation within it.
2. The virtual thread calls a native method or foreign function (JNI/FFM).

### JDK Version Context

| JDK | `synchronized` pinning behaviour |
|---|---|
| JDK 21–23 | Pinning occurs inside `synchronized` blocks with blocking I/O |
| JDK 24+ | **JEP 491 fixes this** — `synchronized` no longer pins virtual threads |

On JDK 21–23, if you must do blocking I/O inside a critical section, replace `synchronized` with `ReentrantLock`:

```java
// ❌ JDK 21-23: synchronized + blocking I/O = thread pinning
private synchronized String fetchAndCache(String key) {
    return cache.computeIfAbsent(key, k -> httpClient.get(k)); // blocks → pins!
}
```

```java
// ✅ JDK 21-23: ReentrantLock does NOT pin virtual threads
private final ReentrantLock lock = new ReentrantLock();

private String fetchAndCache(String key) {
    lock.lock();
    try {
        return cache.computeIfAbsent(key, k -> httpClient.get(k)); // blocks → safe
    } finally {
        lock.unlock();
    }
}
```

> **On JDK 24+** with JEP 491, synchronized is safe again and this workaround is no longer required.

### Detecting Pinning with JFR

```bash
# Add this JVM flag to print stack traces when pinning occurs (> 20ms)
-Djdk.tracePinnedThreads=full

# Or use Java Flight Recorder and watch for:
# jdk.VirtualThreadPinned events
```

---

## 14. Spring Boot Integration

If you use Spring Boot 3.2+, enabling virtual threads for the entire web server is a two-line configuration:

```properties
# application.properties
spring.threads.virtual.enabled=true
```

This configures Tomcat, Jetty, and Spring's `@Async` task executor to use virtual threads automatically.

For manual `CompletableFuture` integration:

```java
@Configuration
public class AsyncConfig {

    @Bean(TaskExecutionAutoConfiguration.APPLICATION_TASK_EXECUTOR_BEAN_NAME)
    public AsyncTaskExecutor asyncTaskExecutor() {
        return new TaskExecutorAdapter(Executors.newVirtualThreadPerTaskExecutor());
    }
}
```

Now every `@Async` method and `CompletableFuture.supplyAsync()` call using the default executor runs on virtual threads:

```java
@Service
public class ReportService {

    @Async // runs on virtual thread when asyncTaskExecutor is configured
    public CompletableFuture<Report> generateAsync(String type) {
        Report report = reportEngine.generate(type); // blocking call, fine
        return CompletableFuture.completedFuture(report);
    }
}
```

---

## 15. Decision Guide: When to Use What

```
Is your task I/O-bound (HTTP, DB, file, messaging)?
│
├─ YES ──► Use virtual threads (Executors.newVirtualThreadPerTaskExecutor())
│
└─ NO (CPU-bound calculation, in-memory processing)
   └──► Use a fixed platform thread pool sized to available processors


Do you need to COMPOSE or CHAIN dependent async steps?
│
├─ YES ──► Use CompletableFuture's fluent API (thenApply, thenCompose, etc.)
│
└─ NO (single standalone task, imperative style is fine)
   └──► Plain virtual thread with Thread.ofVirtual().start(...)


Do you need strict PARENT-CHILD lifecycle (no thread leaks, structured cleanup)?
│
├─ YES ──► StructuredTaskScope (see companion tutorial)
│
└─ NO (fire-and-join, or you manage lifecycle yourself)
   └──► CompletableFuture + virtual executor is sufficient


Do you need to limit access to a downstream resource?
│
├─ YES ──► Semaphore (not a fixed thread pool)
│
└─ NO
   └──► Unbounded newVirtualThreadPerTaskExecutor() is fine
```

---

## 16. Full Real-World Example

Below is a complete example: building an e-commerce **Order Summary Page** by concurrently fetching order details, customer info, product catalog data, and a personalized recommendation — with proper timeout handling, error recovery, and resource bounding.

```java
import java.util.concurrent.*;
import java.util.List;

public class OrderSummaryService {

    // Shared virtual-thread executor for all I/O
    private static final ExecutorService EXEC =
        Executors.newVirtualThreadPerTaskExecutor();

    // Limit concurrent DB connections to 30
    private static final Semaphore DB_GATE = new Semaphore(30);

    record OrderSummary(Order order, Customer customer,
                        List<ProductDetails> products, String recommendation) {}

    public OrderSummary buildSummary(long orderId) throws Exception {

        // ── Stage 1: Fetch order and customer concurrently ──────────────────
        CompletableFuture<Order> orderFuture = CompletableFuture
            .supplyAsync(() -> {
                DB_GATE.acquireUninterruptibly();
                try   { return orderRepo.findById(orderId); }
                finally { DB_GATE.release(); }
            }, EXEC)
            .orTimeout(3, TimeUnit.SECONDS);

        CompletableFuture<Customer> customerFuture = orderFuture
            .thenComposeAsync(order ->
                CompletableFuture.supplyAsync(() -> {
                    DB_GATE.acquireUninterruptibly();
                    try   { return customerRepo.findById(order.customerId()); }
                    finally { DB_GATE.release(); }
                }, EXEC), EXEC)
            .orTimeout(3, TimeUnit.SECONDS);

        // ── Stage 2: Fan out — fetch product details for each line item ──────
        CompletableFuture<List<ProductDetails>> productsFuture = orderFuture
            .thenComposeAsync(order -> {
                List<CompletableFuture<ProductDetails>> pFutures = order.lineItems()
                    .stream()
                    .map(item -> CompletableFuture
                        .supplyAsync(() -> catalogService.getDetails(item.productId()), EXEC)
                        .completeOnTimeout(ProductDetails.UNKNOWN, 2, TimeUnit.SECONDS))
                    .toList();

                return CompletableFuture
                    .allOf(pFutures.toArray(CompletableFuture[]::new))
                    .thenApply(v -> pFutures.stream().map(CompletableFuture::join).toList());
            }, EXEC);

        // ── Stage 3: Recommendation — optional, non-critical ─────────────────
        CompletableFuture<String> recFuture = orderFuture
            .thenComposeAsync(order ->
                CompletableFuture.supplyAsync(
                    () -> mlService.recommend(order.customerId()), EXEC), EXEC)
            .completeOnTimeout("No recommendations available", 1, TimeUnit.SECONDS)
            .exceptionally(ex -> "Recommendations unavailable"); // never fail the page

        // ── Stage 4: Combine all results ──────────────────────────────────────
        return CompletableFuture
            .allOf(orderFuture, customerFuture, productsFuture, recFuture)
            .thenApply(v -> new OrderSummary(
                orderFuture.join(),
                customerFuture.join(),
                productsFuture.join(),
                recFuture.join()
            ))
            .exceptionally(ex -> {
                log.error("Failed to build order summary for {}", orderId, ex);
                throw new RuntimeException("Order summary unavailable", ex);
            })
            .join();
    }
}
```

### What this demonstrates

| Feature | Where used |
|---|---|
| Virtual thread executor | `EXEC` shared across all stages |
| Concurrent fan-out | Order + Customer fetched in parallel; products fanned out per line item |
| Resource bounding | `DB_GATE` semaphore limits DB connections to 30 |
| Timeout per stage | `orTimeout(3s)` on critical paths; `completeOnTimeout` on optional paths |
| Graceful degradation | Recommendations never fail the page (`exceptionally`) |
| Composable chaining | `thenComposeAsync` for dependent stages; `allOf` for final assembly |
| Blocking freely | All repository calls are plain blocking calls — no reactive wrappers |

---

## 17. Summary Cheat Sheet

```java
// ── Setup ─────────────────────────────────────────────────────────────────────
ExecutorService exec = Executors.newVirtualThreadPerTaskExecutor(); // one per task

// ── Basic async task ──────────────────────────────────────────────────────────
CompletableFuture<T> cf = CompletableFuture.supplyAsync(() -> blockingOp(), exec);

// ── Chaining (always pass exec to Async variants for I/O stages) ─────────────
cf.thenApplyAsync(result -> transform(result), exec)   // map result
  .thenComposeAsync(r -> asyncOp(r), exec)              // flat-map (avoid CF<CF<T>>)
  .thenAcceptAsync(r -> save(r), exec);                 // consume result, no return

// ── Fan-out ───────────────────────────────────────────────────────────────────
CompletableFuture.allOf(cf1, cf2, cf3).join();          // wait for all
CompletableFuture.anyOf(cf1, cf2, cf3).join();          // first to complete wins

// ── Error handling ────────────────────────────────────────────────────────────
cf.exceptionally(ex -> fallback)                         // recover with value
  .handle((result, ex) -> ex != null ? err : result)    // handle both cases
  .whenComplete((r, ex) -> log(r, ex));                  // side-effect only

// ── Timeouts ─────────────────────────────────────────────────────────────────
cf.orTimeout(3, TimeUnit.SECONDS)                       // throw TimeoutException
  .completeOnTimeout(defaultVal, 3, TimeUnit.SECONDS);  // silent fallback

// ── Resource bounding ─────────────────────────────────────────────────────────
Semaphore gate = new Semaphore(20);
CompletableFuture.supplyAsync(() -> {
    gate.acquireUninterruptibly();
    try   { return dbCall(); }
    finally { gate.release(); }
}, exec);

// ── Legacy Future bridge ──────────────────────────────────────────────────────
Thread.ofVirtual().start(() -> {
    try   { cf.complete(legacyFuture.get()); }           // cheap blocking on VT
    catch (Exception e) { cf.completeExceptionally(e); }
});

// ── Cleanup ───────────────────────────────────────────────────────────────────
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) { ... } // AutoCloseable
```

### The Golden Rules

1. **Always pass your virtual-thread executor** to every `*Async` stage that does I/O.
2. **Never pool virtual threads** — use `newVirtualThreadPerTaskExecutor()`.
3. **Use Semaphore, not thread pool**, to bound access to downstream resources.
4. **Always terminate pipelines** with `exceptionally` or `handle`.
5. **Always set timeouts** with `orTimeout` or `completeOnTimeout`.
6. **Prefer `thenCompose` over `thenApply`** when stages return a `CompletableFuture`.
7. On **JDK 21–23**, replace `synchronized` + blocking I/O with `ReentrantLock` to avoid pinning. On **JDK 24+** (JEP 491), this is no longer necessary.

---

*References: JEP 444 (Virtual Threads) · JEP 491 (Synchronized Pinning Fix) · Oracle Virtual Threads Docs · Spring Blog: Embracing Virtual Threads*
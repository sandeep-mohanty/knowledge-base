# Virtual Threads + Project Reactor: Getting the Best of Both Worlds

> **Target stack:** Java 21+ · reactor-core 3.6+ · Spring Boot 3.2+ (optional)  
> **Prerequisites:** Familiarity with `Mono`, `Flux`, basic Reactor operators, and Java virtual threads

---

## Table of Contents

1. [The Big Picture: Why Combine Both?](#1-the-big-picture-why-combine-both)
2. [Quick Refresher: What Each Tool Brings](#2-quick-refresher-what-each-tool-brings)
3. [The Reactor Threading Model and Why It Matters](#3-the-reactor-threading-model-and-why-it-matters)
4. [The Bridge: Creating a Virtual-Thread Scheduler](#4-the-bridge-creating-a-virtual-thread-scheduler)
5. [Step 1 — Replacing boundedElastic with Virtual Threads](#5-step-1--replacing-boundedelastic-with-virtual-threads)
6. [Step 2 — Wrapping Blocking Calls with `Mono.fromCallable`](#6-step-2--wrapping-blocking-calls-with-monofromcallable)
7. [Step 3 — `subscribeOn` vs `publishOn` with Virtual Threads](#7-step-3--subscribeon-vs-publishon-with-virtual-threads)
8. [Step 4 — Fan-Out with `Flux.flatMap` and Virtual Threads](#8-step-4--fan-out-with-fluxflatmap-and-virtual-threads)
9. [Step 5 — Bridging Virtual Threads into the Reactive Pipeline](#9-step-5--bridging-virtual-threads-into-the-reactive-pipeline)
10. [Step 6 — Backpressure and Streaming: Where Reactor Stays King](#10-step-6--backpressure-and-streaming-where-reactor-stays-king)
11. [Step 7 — Error Handling Across the Boundary](#11-step-7--error-handling-across-the-boundary)
12. [Step 8 — Context Propagation and Tracing](#12-step-8--context-propagation-and-tracing)
13. [Step 9 — Spring WebFlux + Virtual Threads Together](#13-step-9--spring-webflux--virtual-threads-together)
14. [Pitfalls to Avoid](#14-pitfalls-to-avoid)
15. [Decision Guide: Reactor vs Virtual Threads vs Both](#15-decision-guide-reactor-vs-virtual-threads-vs-both)
16. [Full Real-World Example](#16-full-real-world-example)
17. [Summary Cheat Sheet](#17-summary-cheat-sheet)

---

## 1. The Big Picture: Why Combine Both?

Project Reactor and virtual threads are often framed as competing solutions, but they solve subtly different aspects of the scalability and concurrency problem. Understanding this distinction is the key to using them together effectively.

**Project Reactor** gives you:
- A declarative, composable pipeline API (`Mono`, `Flux`, operators).
- Built-in backpressure — consumers can signal how fast they want data.
- Rich streaming operators (`flatMap`, `zip`, `groupBy`, `window`, `buffer`, etc.).
- Event-driven, non-blocking I/O when paired with reactive drivers (R2DBC, WebClient, etc.).

**Virtual threads** give you:
- The ability to call **blocking APIs freely** without consuming OS threads.
- Simple, imperative, sequential code that reads and debugs like normal Java.
- Massive scalability (millions of threads) at the cost of almost zero extra complexity.
- Full compatibility with the existing Java ecosystem (JDBC, blocking HTTP clients, legacy libraries).

The problems arise when you force one tool to do the other's job:
- Using Reactor with blocking JDBC/legacy code stalls the event-loop and kills scalability.
- Writing a pure virtual-thread solution loses Reactor's streaming operators, backpressure, and non-blocking I/O compositions.

The sweet spot: **keep Reactor as the pipeline orchestrator and use virtual threads as the execution context for any blocking work that enters or exits the reactive world**.

```
  ┌────────────────────────────────────────────────────────────────┐
  │              Reactive Pipeline (Project Reactor)               │
  │                                                                │
  │  Flux / Mono operators — map, flatMap, zip, retry, buffer...  │
  │  Backpressure, streaming, error handling, context propagation  │
  │                                                                │
  │            ┌─────────────────────────────────┐               │
  │            │   Blocking Work Boundary         │               │
  │            │   (JDBC, legacy HTTP, files...)  │               │
  │            │   Runs on Virtual Thread         │               │
  │            │   Scheduler — free to block,     │               │
  │            │   no OS thread wasted            │               │
  │            └─────────────────────────────────┘               │
  └────────────────────────────────────────────────────────────────┘
```

---

## 2. Quick Refresher: What Each Tool Brings

### Key Reactor Types and Operators

| Type / Operator | Purpose |
|---|---|
| `Mono<T>` | 0 or 1 async result |
| `Flux<T>` | 0 to N async elements (stream) |
| `map(fn)` | Synchronous 1:1 transformation |
| `flatMap(fn)` | Async 1:N transformation (concurrent, unordered) |
| `concatMap(fn)` | Async 1:N transformation (sequential, ordered) |
| `flatMapSequential(fn)` | Async 1:N (parallel execution, ordered output) |
| `zip(m1, m2)` | Combine two independent Monos |
| `subscribeOn(scheduler)` | Set the scheduler for the **source** subscription |
| `publishOn(scheduler)` | Switch the scheduler for all **downstream** operators |
| `Mono.fromCallable(fn)` | Wrap a blocking/checked call lazily as a `Mono` |
| `Mono.fromFuture(fn)` | Wrap a `CompletableFuture` as a `Mono` |

### Reactor's Built-in Schedulers (before Virtual Threads)

| Scheduler | Backed by | Use for |
|---|---|---|
| `Schedulers.parallel()` | Fixed CPU-count thread pool | CPU-bound, non-blocking ops |
| `Schedulers.single()` | Single reusable thread | Serialized, low-latency ops |
| `Schedulers.boundedElastic()` | Capped platform-thread pool (10x CPUs) | Blocking I/O offload |
| `Schedulers.immediate()` | Caller thread (no switch) | Testing or simple pipelines |
| `Schedulers.fromExecutorService(exec)` | Any `ExecutorService` | **← This is your virtual thread entry point** |

---

## 3. The Reactor Threading Model and Why It Matters

Reactor operators are **thread-agnostic** by design — they run on whatever thread calls `onNext`. The two operators that explicitly switch threads are `subscribeOn` and `publishOn`.

In a Spring WebFlux application, incoming requests arrive on **Netty's event-loop threads**. These threads are precious — there are only as many as CPU cores, and **they must never block**. If you call a blocking database or HTTP library on the event loop, your entire server stalls.

```
  Netty Event Loop Thread (tiny fixed pool — NEVER BLOCK!)
        │
        ▼
  @GetMapping("/product/{id}")
  public Mono<Product> getProduct(@PathVariable String id) {
      return Mono.fromCallable(() -> jdbcRepo.findById(id)) // ← BLOCKS THE EVENT LOOP ❌
                 .map(Product::from);
  }
```

The classic pre-virtual-threads fix was to offload to `Schedulers.boundedElastic()`, which uses a capped pool of platform threads. This worked, but was still bounded (capped at 10 × CPU cores by default) and added OS thread overhead. Virtual threads remove the cap entirely.

---

## 4. The Bridge: Creating a Virtual-Thread Scheduler

The central integration pattern is wrapping `Executors.newVirtualThreadPerTaskExecutor()` in a Reactor `Scheduler`:

```java
import reactor.core.scheduler.Scheduler;
import reactor.core.scheduler.Schedulers;
import java.util.concurrent.Executors;

// Wrap a virtual-thread executor as a Reactor Scheduler
Scheduler virtualThreadScheduler =
    Schedulers.fromExecutorService(
        Executors.newVirtualThreadPerTaskExecutor(),
        "virtual-thread"   // name shown in thread dumps and metrics
    );
```

This single object is reusable across your entire application. Every time Reactor needs to schedule work on this scheduler, it submits a task to `newVirtualThreadPerTaskExecutor()`, which spawns a brand-new virtual thread — free to block, cheap to create, and automatically cleaned up.

### Application-Wide Shared Scheduler

Define it as a shared constant or Spring bean:

```java
@Configuration
public class SchedulerConfig {

    // Virtual threads for all blocking I/O offloading
    @Bean(destroyMethod = "dispose")
    public Scheduler virtualThreadScheduler() {
        return Schedulers.fromExecutorService(
            Executors.newVirtualThreadPerTaskExecutor(),
            "vt-scheduler"
        );
    }

    // Keep a separate parallel scheduler for CPU-bound reactive work
    @Bean
    public Scheduler cpuScheduler() {
        return Schedulers.newParallel("cpu-work",
            Runtime.getRuntime().availableProcessors());
    }
}
```

### Reactor 3.6+ Global Switch (Optional)

Starting with reactor-core 3.6, you can make `Schedulers.boundedElastic()` itself use virtual threads globally by setting a JVM system property — no code changes required:

```bash
-Dreactor.schedulers.defaultBoundedElasticOnVirtualThreads=true
```

When running on Java 21+, this activates the `BoundedElasticThreadPerTaskScheduler` implementation, which uses virtual threads for all `boundedElastic()` work across your whole application. This is the lowest-friction migration path for existing codebases.

---

## 5. Step 1 — Replacing `boundedElastic` with Virtual Threads

The most common Reactor pattern for blocking code is:

```java
// Before: blocking call offloaded to boundedElastic (platform threads, capped)
Mono.fromCallable(() -> jdbcRepository.findById(id))
    .subscribeOn(Schedulers.boundedElastic());
```

Replace with your virtual-thread scheduler:

```java
// After: same pattern, virtual threads — no cap, no OS thread waste
Mono.fromCallable(() -> jdbcRepository.findById(id))
    .subscribeOn(virtualThreadScheduler);
```

The API call is identical. The difference is that `virtualThreadScheduler` creates a fresh virtual thread per task, while `boundedElastic` borrows from a capped platform-thread pool. Under high concurrency, the virtual-thread version scales linearly where `boundedElastic` would queue tasks and introduce latency.

### Direct Comparison

```java
// ❌ Old approach — platform threads, pool bounded by 10 × CPUs
Mono<User> userMono = Mono.fromCallable(() -> userRepo.findByEmail(email))
    .subscribeOn(Schedulers.boundedElastic());

// ✅ New approach — virtual threads, effectively unlimited
Mono<User> userMono = Mono.fromCallable(() -> userRepo.findByEmail(email))
    .subscribeOn(virtualThreadScheduler);

// ✅ Simplest migration — set JVM flag and touch nothing
// -Dreactor.schedulers.defaultBoundedElasticOnVirtualThreads=true
Mono<User> userMono = Mono.fromCallable(() -> userRepo.findByEmail(email))
    .subscribeOn(Schedulers.boundedElastic()); // now backed by virtual threads
```

---

## 6. Step 2 — Wrapping Blocking Calls with `Mono.fromCallable`

`Mono.fromCallable` is the canonical way to safely wrap any blocking or checked-exception-throwing call into a reactive `Mono`. It is **lazy** — the lambda is only executed when someone subscribes, and only once per subscription.

### The Lazy Subscription Rule

```java
// ❌ BAD: This executes the blocking call at ASSEMBLY time — before anyone subscribes!
Mono<User> eager = Mono.just(userRepo.findById(id)); // blocks NOW

// ✅ GOOD: Lazy — executes only when subscribed
Mono<User> lazy = Mono.fromCallable(() -> userRepo.findById(id));
```

Always pair `Mono.fromCallable` with `subscribeOn(virtualThreadScheduler)` to push the execution off the current thread:

```java
Mono<User> userMono = Mono
    .fromCallable(() -> userRepo.findById(userId))   // lazy blocking call
    .subscribeOn(virtualThreadScheduler)              // execute on a virtual thread
    .map(User::sanitize)                              // transforms on same VT
    .switchIfEmpty(Mono.error(
        () -> new NotFoundException("User " + userId + " not found")));
```

### Wrapping Checked Exceptions

`Mono.fromCallable` handles checked exceptions automatically — they are propagated as `onError` signals:

```java
// No try-catch needed — IOException becomes an onError signal
Mono<String> fileContent = Mono
    .fromCallable(() -> Files.readString(Path.of("/data/config.txt")))
    .subscribeOn(virtualThreadScheduler)
    .onErrorMap(IOException.class, e ->
        new ServiceException("Config file unavailable", e));
```

### Wrapping a Multi-Step Blocking Sequence

Since virtual threads can block freely, you can write multiple blocking steps in a single `fromCallable` lambda instead of chaining many operators:

```java
// ✅ Multiple blocking steps in one fromCallable — simple and readable
Mono<Invoice> invoiceMono = Mono.fromCallable(() -> {
        Order    order    = orderRepo.findById(orderId);        // blocking
        Customer customer = customerRepo.findById(order.customerId()); // blocking
        Template template = templateService.getTemplate(order.locale()); // blocking
        return Invoice.generate(order, customer, template);
    })
    .subscribeOn(virtualThreadScheduler);
```

This is both simpler and more readable than chaining three separate `flatMap` operations.

---

## 7. Step 3 — `subscribeOn` vs `publishOn` with Virtual Threads

Understanding the difference between these two operators is critical when introducing virtual-thread schedulers.

### `subscribeOn(scheduler)` — Affects the Source

`subscribeOn` sets the scheduler used for the **subscription phase** — from the source upward. Only one `subscribeOn` per chain is effective (the one closest to the source wins). It affects the source and all upstream operators, and its placement in the chain does not matter.

```java
// subscribeOn is for the SOURCE — the blocking call runs on the virtual thread
Mono.fromCallable(() -> blockingDb.query())   // runs on virtualThreadScheduler
    .map(row -> transform(row))               // also runs on virtualThreadScheduler
    .subscribeOn(virtualThreadScheduler)      // placement here or anywhere — same result
    .map(result -> format(result));           // still virtualThreadScheduler
```

### `publishOn(scheduler)` — Affects Downstream Operators

`publishOn` **switches the thread** for all operators that come **after** it in the chain. It is placement-sensitive. Use it to hop back to a different scheduler mid-pipeline.

```java
Flux.fromIterable(ids)                          // assembles on caller thread
    .flatMap(id ->
        Mono.fromCallable(() -> db.findById(id))
            .subscribeOn(virtualThreadScheduler))  // DB call on virtual thread
    .publishOn(Schedulers.parallel())              // ← switch downstream to parallel
    .map(entity -> cpuIntensiveTransform(entity))  // runs on parallel scheduler
    .publishOn(virtualThreadScheduler)             // ← switch back for more I/O
    .flatMap(r -> Mono.fromCallable(() -> db.save(r))
                      .subscribeOn(virtualThreadScheduler));
```

### Visual Summary

```
  subscribeOn(vt)           publishOn(vt)
       │                         │
  ┌────▼────┐                    │
  │ SOURCE  │                    │
  │ (VT)    │                    │
  └────┬────┘           ┌────────▼───────┐
       │                │  Switches all  │
       ▼                │  DOWNSTREAM    │
  All upstream &        │  operators to  │
  source run on VT      │  virtual thread│
                        └────────────────┘

Rule: subscribeOn = WHERE the source runs
      publishOn   = WHERE the rest runs from that point forward
```

---

## 8. Step 4 — Fan-Out with `Flux.flatMap` and Virtual Threads

`flatMap` is Reactor's concurrent fan-out operator. It subscribes to inner publishers **eagerly and concurrently**. When each inner publisher uses `subscribeOn(virtualThreadScheduler)`, every item's blocking work runs in its own virtual thread simultaneously.

### Concurrent Blocking Fan-Out

```java
Flux<ProductSummary> summaries = Flux.fromIterable(productIds)
    .flatMap(id ->
        Mono.fromCallable(() -> catalogService.fetchDetails(id))  // blocking
            .subscribeOn(virtualThreadScheduler),                  // own virtual thread per item
        64  // max concurrency (default is 256 — tune to match downstream capacity)
    );
```

### Choosing the Right Fan-Out Operator

| Operator | Execution | Output Order | Use When |
|---|---|---|---|
| `flatMap` | Concurrent | Unordered | I/O calls, order doesn't matter |
| `concatMap` | Sequential | Ordered | Must process items in order, DB with ordered writes |
| `flatMapSequential` | Concurrent | Ordered | Parallel execution, but output must be ordered |

```java
// flatMap — concurrent, results in any order (fastest for pure I/O)
flux.flatMap(id -> fetchAsync(id).subscribeOn(vts));

// concatMap — one at a time, strictly ordered (slowest but simplest semantics)
flux.concatMap(id -> fetchAsync(id).subscribeOn(vts));

// flatMapSequential — parallel fetch, but emits in original order
flux.flatMapSequential(id -> fetchAsync(id).subscribeOn(vts));
```

### Limiting Concurrency to Protect Downstream

```java
// Without concurrency limit — could spawn 10,000 virtual threads for 10,000 IDs
// Fine for virtual threads themselves, but might overload the DB connection pool
Flux.fromIterable(tenThousandIds)
    .flatMap(id ->
        Mono.fromCallable(() -> db.findById(id)).subscribeOn(vts));

// With concurrency limit — at most 50 DB calls at a time
Flux.fromIterable(tenThousandIds)
    .flatMap(id ->
        Mono.fromCallable(() -> db.findById(id)).subscribeOn(vts),
        50  // maxConcurrency argument
    );
```

---

## 9. Step 5 — Bridging Virtual Threads into the Reactive Pipeline

### From a Virtual Thread Result to `Mono`

When you have logic that is easier to express imperatively (fan-out, branching, loops), run it on a virtual thread and wrap the result as a `Mono`:

```java
// Complex multi-step logic — easier as imperative code on a virtual thread
Mono<DashboardData> dashboard = Mono.fromCallable(() -> {
    // All of this runs on a virtual thread — blocking is free
    User user         = userService.getUser(userId);
    List<Order> orders = orderService.getOrders(userId);
    List<Rec> recs    = recEngine.getRecommendations(userId);
    return DashboardData.from(user, orders, recs);
})
.subscribeOn(virtualThreadScheduler);
```

### From `CompletableFuture` (Bridging Legacy Code)

```java
// Legacy service returns CompletableFuture — bridge it lazily
Mono<String> result = Mono.fromFuture(() ->
    CompletableFuture.supplyAsync(
        () -> legacyService.heavyComputation(input),
        Executors.newVirtualThreadPerTaskExecutor()
    )
);
```

> **Use `Mono.fromFuture(Supplier<CF>)` not `Mono.fromFuture(CF)`**. The `Supplier` form is lazy — the `CompletableFuture` is only created on subscription. The direct form is eager and may execute before anyone subscribes.

### From a WebFlux Controller — Offloading Blocking JDBC

```java
@RestController
@RequiredArgsConstructor
public class ProductController {

    private final ProductRepository productRepository; // blocking JDBC repo
    private final Scheduler vts; // injected virtual thread scheduler

    @GetMapping("/products/{id}")
    public Mono<Product> getProduct(@PathVariable String id) {
        // Safe: JDBC call runs on a virtual thread, not the Netty event loop
        return Mono.fromCallable(() -> productRepository.findById(id))
                   .subscribeOn(vts)
                   .switchIfEmpty(Mono.error(
                       () -> new ResponseStatusException(NOT_FOUND)));
    }

    @GetMapping("/products")
    public Flux<Product> getAllProducts() {
        // Wrap the blocking list call, then turn it into a Flux
        return Mono.fromCallable(productRepository::findAll)
                   .subscribeOn(vts)
                   .flatMapIterable(list -> list); // List<Product> → Flux<Product>
    }
}
```

---

## 10. Step 6 — Backpressure and Streaming: Where Reactor Stays King

This is the scenario where Project Reactor has **no equivalent** in the virtual-thread world, and you should always lean on it: **streaming large or unbounded datasets with backpressure**.

Backpressure means the **consumer controls how fast data flows** from the producer. This prevents a fast producer from overwhelming a slow consumer with data, causing out-of-memory errors or dropped events.

Virtual threads have no native concept of backpressure — they just block until the next item is available, which can cause unbounded accumulation in memory.

### Streaming with Backpressure — Reactor's Domain

```java
// Stream 1 million database rows to a client with backpressure
// The DB is only queried as fast as the HTTP client can consume
@GetMapping(value = "/products/export", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<Product> exportAllProducts() {
    return r2dbcProductRepository    // reactive R2DBC — truly non-blocking
        .findAll()                   // Flux<Product> with backpressure
        .filter(Product::isActive)
        .map(Product::toDto);
    // HTTP client's TCP buffer controls the pull rate — no OOM risk
}
```

### Buffering and Windowing — Only in Reactor

```java
// Process events in batches of 100 or every 5 seconds, whichever comes first
Flux<Event> eventStream = kafkaReceiver.receive();

eventStream
    .bufferTimeout(100, Duration.ofSeconds(5))   // backpressure-aware batching
    .flatMap(batch ->
        Mono.fromCallable(() -> batchProcessor.process(batch))
            .subscribeOn(virtualThreadScheduler), // batch processing on VT
        8  // at most 8 concurrent batch processors
    )
    .subscribe();
```

### `onBackpressureBuffer`, `onBackpressureDrop`, `onBackpressureLatest`

When a producer is inherently hot (Kafka, WebSocket events), Reactor operators let you define what happens when the consumer is slower than the producer:

```java
hotEventSource
    .onBackpressureBuffer(1000,         // buffer up to 1000 events
        dropped -> log.warn("Dropped: {}", dropped),
        BufferOverflowStrategy.DROP_OLDEST)
    .flatMap(event ->
        Mono.fromCallable(() -> processEvent(event))
            .subscribeOn(virtualThreadScheduler));
```

> **Rule:** Whenever data volume, flow rate, or memory pressure is a concern, keep Reactor in the loop. Virtual threads alone will happily buffer unlimited data and crash.

---

## 11. Step 7 — Error Handling Across the Boundary

Error handling follows the same Reactor patterns regardless of which scheduler runs the work. Errors thrown inside `Mono.fromCallable` or `subscribeOn` blocks surface as `onError` signals in the reactive pipeline.

### `onErrorResume` — Fallback Publisher

```java
Mono<Product> product = Mono
    .fromCallable(() -> productRepository.findById(id))
    .subscribeOn(virtualThreadScheduler)
    .onErrorResume(DataAccessException.class, ex ->
        Mono.fromCallable(() -> cacheService.getCachedProduct(id))
            .subscribeOn(virtualThreadScheduler));   // fallback also on VT
```

### `onErrorMap` — Translate Exceptions

```java
Mono<User> user = Mono
    .fromCallable(() -> userRepository.findByEmail(email))
    .subscribeOn(virtualThreadScheduler)
    .onErrorMap(SQLException.class, ex ->
        new ServiceException("Database unavailable", ex));
```

### `retry` and `retryWhen` — Resilience

```java
Mono<String> resilient = Mono
    .fromCallable(() -> unreliableExternalApi.call())
    .subscribeOn(virtualThreadScheduler)
    .retryWhen(Retry.backoff(3, Duration.ofMillis(200))
                   .filter(ex -> ex instanceof IOException));
```

### `timeout` — Deadline for Blocking Work

```java
Mono<Result> timedResult = Mono
    .fromCallable(() -> slowBlockingService.compute(input))
    .subscribeOn(virtualThreadScheduler)
    .timeout(Duration.ofSeconds(3))
    .onErrorResume(TimeoutException.class,
        ex -> Mono.just(Result.defaultResult()));
```

---

## 12. Step 8 — Context Propagation and Tracing

### The Problem: `ThreadLocal` vs Reactor `Context`

Reactor stores per-pipeline metadata in its own `Context` object, not in `ThreadLocal`. When you hop to a virtual thread via `subscribeOn`, the `ThreadLocal` values of the event-loop thread are not automatically carried over.

This matters for tracing (MDC, trace IDs) and security context (Spring Security's `SecurityContextHolder`).

### Solution A: Reactor 3.6+ Automatic Context Propagation

Reactor 3.6 introduced automatic `ThreadLocal` restoration from `Context` using the Micrometer Context Propagation library:

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>context-propagation</artifactId>
</dependency>
```

```java
// Register your ThreadLocal with the global ContextRegistry (done once at startup)
static final ThreadLocal<String> TRACE_ID = ThreadLocal.withInitial(() -> "");

static {
    ContextRegistry.getInstance()
        .registerThreadLocalAccessor("traceId", TRACE_ID);
}

// Enable automatic propagation globally
Hooks.enableAutomaticContextPropagation();
```

Once enabled, Reactor automatically saves and restores your registered `ThreadLocal` values across scheduler hops — including when work moves to virtual threads:

```java
TRACE_ID.set("req-abc-123");

Mono.fromCallable(() -> {
        // TRACE_ID is available here even though we're on a virtual thread
        log.info("[{}] Querying DB", TRACE_ID.get());
        return db.query();
    })
    .subscribeOn(virtualThreadScheduler)
    .doOnNext(result ->
        log.info("[{}] Got result", TRACE_ID.get())) // still propagated downstream
    .block();
```

### Solution B: Reactor `Context` API (Manual, No Library)

For manual context passing without the Micrometer library:

```java
Mono<String> pipeline = Mono
    .deferContextual(ctx -> {
        String traceId = ctx.get("traceId");
        return Mono.fromCallable(() -> {
            MDC.put("traceId", traceId);  // restore in the virtual thread
            try {
                return db.query();
            } finally {
                MDC.remove("traceId");
            }
        }).subscribeOn(virtualThreadScheduler);
    });

// Populate the Context at the subscription site
pipeline
    .contextWrite(Context.of("traceId", "req-abc-123"))
    .subscribe();
```

### Spring Boot 3.2+ with Micrometer Tracing

When using Spring Boot 3.2+ with Micrometer Tracing, `Hooks.enableAutomaticContextPropagation()` is called automatically. Trace context flows through `Mono`/`Flux` pipelines and across `subscribeOn` hops to virtual threads without any extra configuration.

---

## 13. Step 9 — Spring WebFlux + Virtual Threads Together

WebFlux uses a Netty event loop for HTTP handling. Within that, you can mix Reactor's non-blocking operators with virtual-thread-backed blocking calls.

### Configuration: Register the Virtual Thread Scheduler

```java
@Configuration
public class WebFluxConfig {

    @Bean(destroyMethod = "dispose")
    public Scheduler virtualThreadScheduler() {
        return Schedulers.fromExecutorService(
            Executors.newVirtualThreadPerTaskExecutor(),
            "webflux-vt"
        );
    }
}
```

### Controller Pattern: WebFlux + Blocking JDBC + Reactive Assembly

```java
@RestController
@RequiredArgsConstructor
public class OrderController {

    private final OrderRepository    orderRepo;     // blocking JDBC
    private final InventoryService   inventoryService; // blocking
    private final NotificationClient notifyClient;  // reactive WebClient
    private final Scheduler          vts;           // virtual thread scheduler

    @GetMapping("/orders/{id}/details")
    public Mono<OrderDetails> getOrderDetails(@PathVariable Long id) {

        // Offload two independent blocking calls to virtual threads, combine results
        Mono<Order>     orderMono    = Mono.fromCallable(() -> orderRepo.findById(id))
                                          .subscribeOn(vts);

        Mono<Inventory> inventoryMono = Mono.fromCallable(
                                              () -> inventoryService.checkStock(id))
                                          .subscribeOn(vts);

        // Reactor zip runs both concurrently, then combines results on event loop
        return Mono.zip(orderMono, inventoryMono)
                   .flatMap(tuple -> {
                       Order     order = tuple.getT1();
                       Inventory inv   = tuple.getT2();
                       // Switch to non-blocking reactive WebClient for notification
                       return notifyClient.sendConfirmation(order.customerId())
                                          .thenReturn(OrderDetails.from(order, inv));
                   });
    }

    @GetMapping(value = "/orders/stream", produces = TEXT_EVENT_STREAM_VALUE)
    public Flux<OrderEvent> streamOrders() {
        // Reactive streaming with backpressure — pure Reactor, no VT needed here
        return orderEventRepository.findAllByStatus("PENDING")
                                   .delayElements(Duration.ofMillis(100));
    }
}
```

### The `Mono.zip` Pattern for Parallel Blocking Calls

`Mono.zip` subscribes to all its inputs **concurrently**. When each input uses `subscribeOn(vts)`, they all run in parallel on separate virtual threads:

```java
return Mono.zip(
    Mono.fromCallable(() -> service1.get()).subscribeOn(vts),
    Mono.fromCallable(() -> service2.get()).subscribeOn(vts),
    Mono.fromCallable(() -> service3.get()).subscribeOn(vts)
)
.map(tuple -> combine(tuple.getT1(), tuple.getT2(), tuple.getT3()));
// All three blocking calls happen in parallel — total time ≈ max(t1, t2, t3)
```

---

## 14. Pitfalls to Avoid

### ❌ Pitfall 1: Calling `.block()` inside a Reactive Pipeline

This is the most dangerous mistake in WebFlux. `block()` forces a synchronous wait on the current thread — if that thread is a Netty event-loop thread, it deadlocks or stalls the server.

```java
// ❌ FATAL: block() on the event-loop thread causes deadlock or severe starvation
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable String id) {
    User user = userRepository.findById(id).block(); // NEVER do this in WebFlux
    return Mono.just(user);
}

// ✅ CORRECT: keep the pipeline reactive, offload blocking work to virtual threads
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable String id) {
    return Mono.fromCallable(() -> userRepository.findById(id))
               .subscribeOn(virtualThreadScheduler);
}
```

### ❌ Pitfall 2: Using `map` for Blocking Operations

`map` runs synchronously **on the current scheduler thread**. If that thread is the Netty event-loop, a blocking call inside `map` stalls everything.

```java
// ❌ BAD: blocking call inside map runs on the event-loop
flux.map(id -> jdbcRepo.findById(id));  // blocks the event-loop

// ✅ GOOD: use flatMap + fromCallable + subscribeOn
flux.flatMap(id ->
    Mono.fromCallable(() -> jdbcRepo.findById(id))
        .subscribeOn(virtualThreadScheduler));
```

### ❌ Pitfall 3: Creating a New Scheduler Per Request

Creating a new `Schedulers.fromExecutorService(...)` on every request creates and destroys executor services at high frequency, which is expensive.

```java
// ❌ BAD: new executor + scheduler created for every request
@GetMapping("/data")
public Mono<Data> getData() {
    Scheduler s = Schedulers.fromExecutorService(  // DON'T do this per-request!
        Executors.newVirtualThreadPerTaskExecutor());
    return Mono.fromCallable(() -> db.query()).subscribeOn(s);
}

// ✅ GOOD: inject a shared, application-scoped scheduler
@GetMapping("/data")
public Mono<Data> getData() {
    return Mono.fromCallable(() -> db.query()).subscribeOn(this.vts);
}
```

### ❌ Pitfall 4: Ignoring Backpressure for Unbounded Streams

Virtual threads have no backpressure mechanism. If you turn a virtual-thread result into a `Flux` without controlling the rate, you risk OOM.

```java
// ❌ BAD: generates all results at once, no backpressure
Flux<Record> records = Mono
    .fromCallable(() -> db.findAll())   // loads ENTIRE table into memory
    .subscribeOn(vts)
    .flatMapIterable(list -> list);      // then converts to Flux

// ✅ GOOD: use a reactive/streaming DB driver or paginate
Flux<Record> records = Flux.range(0, totalPages)
    .concatMap(page ->
        Mono.fromCallable(() -> db.findPage(page, PAGE_SIZE))
            .subscribeOn(vts)
            .flatMapIterable(list -> list));
```

### ❌ Pitfall 5: Thread Pinning Inside `fromCallable` (JDK 21–23)

On JDK 21–23, if your blocking code inside `fromCallable` uses `synchronized` blocks with I/O, the virtual thread gets pinned. Replace with `ReentrantLock` for critical sections that contain blocking operations:

```java
// ❌ JDK 21-23: synchronized + blocking I/O = thread pinning
Mono.fromCallable(() -> {
    synchronized (lock) {
        return db.query(); // blocks → pins OS thread ❌
    }
}).subscribeOn(vts);

// ✅ JDK 21-23: ReentrantLock is VT-safe
private final ReentrantLock lock = new ReentrantLock();

Mono.fromCallable(() -> {
    lock.lock();
    try { return db.query(); }  // blocks → safe, VT yields carrier ✅
    finally { lock.unlock(); }
}).subscribeOn(vts);
```

> On **JDK 24+** (JEP 491), `synchronized` no longer causes pinning. This workaround is unnecessary on modern JDKs.

### ❌ Pitfall 6: Using `Mono.fromFuture(cf)` Eagerly

```java
// ❌ The CF is created and started BEFORE anyone subscribes
CompletableFuture<User> cf = CompletableFuture.supplyAsync(() -> db.getUser(id), vte);
Mono<User> mono = Mono.fromFuture(cf); // computation already running!

// ✅ Lazy: CF is created only on subscription
Mono<User> mono = Mono.fromFuture(() ->
    CompletableFuture.supplyAsync(() -> db.getUser(id), vte));
```

---

## 15. Decision Guide: Reactor vs Virtual Threads vs Both

```
Does your scenario involve STREAMING or BACKPRESSURE?
│  (unbounded data, Kafka, WebSocket, SSE, large DB exports)
│
├─ YES ──► Project Reactor / Flux is essential — virtual threads alone cannot do this
│
└─ NO — proceed to next question


Does your scenario involve COMPLEX ASYNC COMPOSITION?
│  (zip, mergeWith, groupBy, window, retryWhen, timeout chains)
│
├─ YES ──► Use Reactor operators for the orchestration
│
└─ NO — proceed to next question


Does your code BLOCK (JDBC, legacy HTTP, file I/O)?
│
├─ YES ──► Wrap in Mono.fromCallable().subscribeOn(virtualThreadScheduler)
│         (use virtual threads for the blocking work)
│
└─ NO — continue on Reactor's non-blocking schedulers (parallel, event-loop)


Are you inside a Spring WebFlux app?
│
├─ YES ──► Never call block(). Use Mono.fromCallable + subscribeOn(vts) for
│         all blocking calls. Keep event-loop threads free.
│
└─ NO (standalone app, Spring MVC, batch job)
   └──► Consider whether you need Reactor at all. For pure request/response
        with blocking I/O, virtual threads alone may be sufficient.


Do you need BACKPRESSURE on virtual-thread-generated data?
│
├─ YES ──► Wrap virtual thread output in Flux.create() or paginate,
│         letting Reactor manage the flow rate
│
└─ NO ──► Mono.fromCallable is sufficient
```

---

## 16. Full Real-World Example

Below is a complete Spring WebFlux endpoint that:
- Fetches product details from a blocking JDBC repository (virtual threads).
- Concurrently calls an inventory service (virtual thread, parallel with DB).
- Enriches results from a non-blocking reactive HTTP call (WebClient, event-loop).
- Streams the enriched results with Reactor operators and backpressure.
- Handles errors with retries and fallbacks at each layer.

```java
@RestController
@RequiredArgsConstructor
public class ProductCatalogController {

    private final ProductJdbcRepository  productRepo;    // blocking JDBC
    private final InventoryService       inventoryService; // blocking RPC
    private final PricingWebClient       pricingClient;  // reactive WebClient
    private final ReviewWebClient        reviewClient;   // reactive WebClient
    private final Scheduler              vts;            // virtual thread scheduler

    // ─── Single product detail ────────────────────────────────────────────────

    @GetMapping("/catalog/{id}")
    public Mono<ProductDetail> getProductDetail(@PathVariable String id) {

        // 1. Two independent blocking calls — run in parallel on virtual threads
        Mono<Product>   productMono   = Mono
            .fromCallable(() -> productRepo.findById(id))
            .subscribeOn(vts)
            .switchIfEmpty(Mono.error(() -> new ResponseStatusException(NOT_FOUND)));

        Mono<Inventory> inventoryMono = Mono
            .fromCallable(() -> inventoryService.check(id))
            .subscribeOn(vts)
            .onErrorReturn(Inventory.UNKNOWN); // never fail for inventory issues

        // 2. Reactive HTTP calls — run on Netty event-loop (truly non-blocking)
        Mono<Price>  priceMono  = pricingClient.getPrice(id)
            .retryWhen(Retry.backoff(2, Duration.ofMillis(100)));

        Mono<Double> ratingMono = reviewClient.getAverageRating(id)
            .timeout(Duration.ofSeconds(2))
            .onErrorReturn(0.0); // graceful degradation

        // 3. Zip all four — run concurrently, combine when all complete
        return Mono.zip(productMono, inventoryMono, priceMono, ratingMono)
                   .map(t -> ProductDetail.from(t.getT1(), t.getT2(),
                                                t.getT3(), t.getT4()));
    }

    // ─── Streaming catalog export with backpressure ───────────────────────────

    @GetMapping(value = "/catalog/export", produces = TEXT_EVENT_STREAM_VALUE)
    public Flux<ProductDetail> streamCatalog(
            @RequestParam(defaultValue = "0") int startPage,
            @RequestParam(defaultValue = "50") int pageSize) {

        return Flux.range(startPage, Integer.MAX_VALUE)
            // Fetch one page at a time from JDBC (paginated, VT-backed)
            .concatMap(page ->
                Mono.fromCallable(() -> productRepo.findPage(page, pageSize))
                    .subscribeOn(vts)
                    .flatMapIterable(list -> list))

            // Stop when a page comes back empty (last page)
            .takeWhile(product -> product != null)

            // Concurrently enrich each product with live pricing (non-blocking)
            .flatMap(product ->
                pricingClient.getPrice(product.id())
                    .map(price -> product.withPrice(price))
                    .onErrorReturn(product),  // keep the product even if pricing fails
                32  // max 32 concurrent pricing calls
            )

            // Reactor handles backpressure: Netty will pull only as fast as
            // the HTTP client can consume
            .doOnNext(p -> log.debug("Streaming product {}", p.id()));
    }

    // ─── Batch update endpoint ────────────────────────────────────────────────

    @PutMapping("/catalog/batch-update")
    public Mono<BatchResult> batchUpdate(@RequestBody List<ProductUpdate> updates) {

        return Flux.fromIterable(updates)
            // Process 20 updates concurrently, each on its own virtual thread
            .flatMap(update ->
                Mono.fromCallable(() -> productRepo.update(update))
                    .subscribeOn(vts)
                    .map(ok -> new UpdateResult(update.id(), ok))
                    .onErrorReturn(new UpdateResult(update.id(), false)),
                20
            )
            .collectList()
            .map(BatchResult::from);
    }
}
```

### What this demonstrates

| Concern | Approach |
|---|---|
| Blocking JDBC calls | `Mono.fromCallable` + `subscribeOn(vts)` |
| Parallel independent fetches | `Mono.zip` with separate `subscribeOn` |
| Non-blocking HTTP | Reactive `WebClient` on event-loop |
| Graceful degradation | `.onErrorReturn`, `.onErrorResume`, `.retryWhen` |
| Paginated streaming with backpressure | `Flux.range` + `concatMap` + `takeWhile` |
| Concurrent enrichment with rate limiting | `flatMap` with `maxConcurrency` argument |
| Event-stream output to client | `TEXT_EVENT_STREAM_VALUE` with Flux |

---

## 17. Summary Cheat Sheet

```java
// ── Create the virtual-thread Reactor Scheduler (once, application-scoped) ───
Scheduler vts = Schedulers.fromExecutorService(
    Executors.newVirtualThreadPerTaskExecutor(), "vt");

// ── Enable globally (Reactor 3.6+ on JDK 21+, replaces boundedElastic) ───────
// -Dreactor.schedulers.defaultBoundedElasticOnVirtualThreads=true

// ── Wrap a blocking call as a lazy Mono ──────────────────────────────────────
Mono<T> m = Mono.fromCallable(() -> blockingRepo.find(id))  // lazy
               .subscribeOn(vts);                            // execute on VT

// ── subscribeOn = where the SOURCE runs ─────────────────────────────────────
// ── publishOn  = where DOWNSTREAM operators run from that point ─────────────
mono.subscribeOn(vts)           // source on VT
    .publishOn(Schedulers.parallel())  // switch to CPU pool for transforms
    .map(heavyTransform::apply);

// ── Concurrent fan-out (parallel blocking calls) ─────────────────────────────
Flux.fromIterable(ids)
    .flatMap(id ->
        Mono.fromCallable(() -> db.find(id)).subscribeOn(vts),
        32);  // max 32 concurrent VTs

// ── Parallel independent Monos ───────────────────────────────────────────────
Mono.zip(
    Mono.fromCallable(() -> svc1.get()).subscribeOn(vts),
    Mono.fromCallable(() -> svc2.get()).subscribeOn(vts)
).map(t -> combine(t.getT1(), t.getT2()));

// ── Lazy CompletableFuture bridge ────────────────────────────────────────────
Mono.fromFuture(() -> CompletableFuture.supplyAsync(() -> op(), vte));

// ── Error handling ────────────────────────────────────────────────────────────
mono.onErrorResume(ex -> fallbackMono)
    .onErrorMap(SomeEx.class, e -> new MyEx(e))
    .retryWhen(Retry.backoff(3, Duration.ofMillis(100)))
    .timeout(Duration.ofSeconds(5));

// ── Context propagation (Reactor 3.6+) ───────────────────────────────────────
Hooks.enableAutomaticContextPropagation();  // one-time at startup
ContextRegistry.getInstance()
    .registerThreadLocalAccessor("traceId", TRACE_ID);

// ── Streaming with backpressure (Reactor's unique strength) ──────────────────
flux.onBackpressureBuffer(1000, dropped -> log.warn("dropped"), DROP_OLDEST)
    .flatMap(item ->
        Mono.fromCallable(() -> process(item)).subscribeOn(vts));
```

### The Golden Rules

1. **Never call `.block()`** inside a WebFlux endpoint or on an event-loop thread.
2. **Always use `Mono.fromCallable`** (not `Mono.just`) to wrap blocking calls — it's lazy.
3. **Prefer `subscribeOn(vts)` over `publishOn(vts)`** for sources; use `publishOn` to hop mid-pipeline.
4. **Use `Mono.zip`** for parallel independent blocking calls — all subscribed concurrently.
5. **Use `flatMap` with a `maxConcurrency` arg** to protect downstream resources.
6. **Never create a new Scheduler per request** — share a single application-scoped instance.
7. **Use `Mono.fromFuture(Supplier<CF>)`**, not `Mono.fromFuture(CF)`, for lazy bridging.
8. **Keep Reactor for streaming + backpressure** — virtual threads alone cannot model this.
9. **Enable `Hooks.enableAutomaticContextPropagation()`** for transparent trace/MDC propagation.
10. On **JDK 21–23**, replace `synchronized` + I/O with `ReentrantLock` inside `fromCallable`. On **JDK 24+** (JEP 491), this is no longer required.

---

*References: Project Reactor Reference Guide · Spring Blog: reactor-core 3.6.0 · Spring Blog: Embracing Virtual Threads · JEP 444 (Virtual Threads) · JEP 491 (Synchronized Pinning Fix) · Reactor Schedulers API Docs*
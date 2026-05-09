# Structured Concurrency in Java: A Step-by-Step Tutorial

> **JEP Coverage:** JEP 428 (JDK 19 Incubator) · JEP 437 (JDK 20 Re-incubator) · JEP 453 (JDK 21 Preview) · JEP 525 (JDK 26 Sixth Preview)

---

## Table of Contents

1. [What is Structured Concurrency?](#1-what-is-structured-concurrency)
2. [The Problem: Unstructured Concurrency](#2-the-problem-unstructured-concurrency)
3. [Evolution Across JEPs](#3-evolution-across-jeps)
4. [Core Concepts](#4-core-concepts)
5. [Getting Started: Your First StructuredTaskScope](#5-getting-started-your-first-structuredtaskscope)
6. [The `fork()` and `join()` Lifecycle](#6-the-fork-and-join-lifecycle)
7. [Completion Policies with `Joiner`](#7-completion-policies-with-joiner)
8. [Error Handling](#8-error-handling)
9. [Timeouts](#9-timeouts)
10. [Scoped Values and Inheritance](#10-scoped-values-and-inheritance)
11. [Nested Scopes](#11-nested-scopes)
12. [Custom Joiners](#12-custom-joiners)
13. [Observability and Thread Dumps](#13-observability-and-thread-dumps)
14. [When NOT to Use Structured Concurrency](#14-when-not-to-use-structured-concurrency)
15. [Full Real-World Example](#15-full-real-world-example)
16. [Summary](#16-summary)

---

## 1. What is Structured Concurrency?

Structured Concurrency is a concurrency programming paradigm introduced in Project Loom that **treats groups of related tasks running in different threads as a single unit of work**. It is the concurrent equivalent of structured control flow (like `if`, `for`, `try`) applied to threads.

The core principle:

> **If a task splits into concurrent subtasks, they all return to the same place — the task's code block.**

This means:
- A thread that opens a scope *owns* it.
- Subtasks *cannot outlive* the scope that created them.
- The scope *cannot close* until all subtasks have finished.

### Goals (as stated across JEPs 428, 437, 453, 525)

- Simplify multithreaded programming.
- Eliminate thread leaks and cancellation delays.
- Improve reliability and error propagation.
- Enhance observability via structured thread dumps.

---

## 2. The Problem: Unstructured Concurrency

Before structured concurrency, combining multiple async subtasks using `ExecutorService` and `Future` was error-prone:

```java
// ❌ UNSTRUCTURED — fragile, hard to maintain
Invoice createInvoice(int orderId, int customerId, String language)
    throws InterruptedException, ExecutionException {

    Future<Order>    orderFuture    = executor.submit(() -> orderService.getOrder(orderId));
    Future<Customer> customerFuture = executor.submit(() -> customerService.getCustomer(customerId));
    Future<Template> templateFuture = executor.submit(() -> templateService.getTemplate(language));

    // ⚠️ If orderFuture.get() throws, customerFuture and templateFuture become ORPHANED threads!
    Order    order    = orderFuture.get();
    Customer customer = customerFuture.get();
    Template template = templateFuture.get();

    return Invoice.generate(order, customer, template);
}
```

### Problems with this approach:

| Problem | Description |
|---|---|
| **Thread Leaks** | If one `Future.get()` throws, sibling tasks keep running uncontrolled |
| **No Cancellation Propagation** | Cancelling the parent does not cancel children |
| **Poor Observability** | Thread dumps show `pool-1-thread-3` with no link to the parent task |
| **Complex Error Handling** | Checking multiple futures for exceptions requires boilerplate |

---

## 3. Evolution Across JEPs

| JEP | JDK | Status | Key Changes |
|---|---|---|---|
| **JEP 428** | 19 | Incubator (`jdk.incubator.concurrent`) | Initial API with `ShutdownOnFailure`, `ShutdownOnSuccess` subclasses |
| **JEP 437** | 20 | Re-Incubator | Added scoped value inheritance from parent thread to subtasks |
| **JEP 453** | 21 | Preview (`java.util.concurrent`) | `fork()` now returns `Subtask<T>` instead of `Future<T>` |
| **JEP 525** | 26 | Sixth Preview | New `Joiner.onTimeout()` method; `join()` returns `List`; `anySuccessfulResultOrThrow()` renamed |

> **Note:** JEP 505 (JDK 25) introduced the largest overhaul: public constructors of `StructuredTaskScope` were replaced with static factory methods (`StructuredTaskScope.open(...)`), and `ShutdownOnFailure`/`ShutdownOnSuccess` were replaced by the flexible `Joiner` interface.

---

## 4. Core Concepts

### `StructuredTaskScope`

The central class. It is an `AutoCloseable` resource that manages a group of subtasks. It is opened with `StructuredTaskScope.open(...)` and used in a `try-with-resources` block.

Key rules:
- The scope is **owned by the thread that opened it** — only that thread can call `fork()`, `join()`, and `close()`.
- `close()` waits for all running subtasks to complete before returning (it acts as an implicit join if `join()` was not called explicitly).

### `Subtask<T>`

A handle returned by `scope.fork(callable)`. Represents a subtask running concurrently. After `join()` completes, you can retrieve the result via `subtask.get()`.

### `Joiner<T, R>`

An interface (introduced to replace `ShutdownOnFailure`/`ShutdownOnSuccess`) that controls:
- **Completion policy** — what to do when a subtask finishes or fails.
- **Result type** — what `scope.join()` returns.

---

## 5. Getting Started: Your First StructuredTaskScope

### Prerequisites

- **JDK 21+** with `--enable-preview` flag (for preview APIs).
- For JDK 25+, use `StructuredTaskScope.open()` (static factory method).

### Compiling with Preview Features

```bash
# Compile
javac --enable-preview --source 25 MyClass.java

# Run
java --enable-preview MyClass
```

### Minimal Example (JDK 25+)

```java
import java.util.concurrent.StructuredTaskScope;

void main() throws InterruptedException {
    try (var scope = StructuredTaskScope.open()) {

        var taskA = scope.fork(() -> {
            Thread.sleep(200);
            return "Result from Task A";
        });

        var taskB = scope.fork(() -> {
            Thread.sleep(100);
            return "Result from Task B";
        });

        scope.join(); // Wait for ALL subtasks to complete

        System.out.println(taskA.get()); // Result from Task A
        System.out.println(taskB.get()); // Result from Task B
    }
}
```

**What happens here:**
1. A new `StructuredTaskScope` is opened.
2. Two subtasks are forked — each runs in its own **virtual thread**.
3. `scope.join()` blocks until **all** subtasks complete.
4. Results are retrieved via `subtask.get()`.
5. The `try-with-resources` block calls `scope.close()`, which ensures cleanup.

---

## 6. The `fork()` and `join()` Lifecycle

```
Thread (owner)
    │
    ├── open scope ──────────────────────────────────────────┐
    │                                                         │ StructuredTaskScope
    ├── fork(task1) ──► [virtual thread 1 runs task1]        │
    ├── fork(task2) ──► [virtual thread 2 runs task2]        │
    ├── fork(task3) ──► [virtual thread 3 runs task3]        │
    │                                                         │
    ├── join() ◄──────── waits for all threads to finish     │
    │                                                         │
    ├── use results                                           │
    │                                                         │
    └── close scope ─────────────────────────────────────────┘
```

### `Subtask<T>` States

After `join()` returns, each subtask will be in one of these states:

| State | Meaning | `subtask.get()` behaviour |
|---|---|---|
| `SUCCESS` | Task completed normally | Returns the result |
| `FAILED` | Task threw an exception | Throws `IllegalStateException` (wraps cause) |
| `UNAVAILABLE` | Task was cancelled before completion | Throws `IllegalStateException` |

---

## 7. Completion Policies with `Joiner`

The `Joiner` interface is the heart of the policy system. You pass a `Joiner` to `StructuredTaskScope.open(joiner)` to control behavior.

### Built-in Joiners

#### 7.1 Default Policy (wait for all, fail on any failure)

```java
// open() with no argument uses the default policy:
// wait for ALL subtasks; if any fails, join() throws FailedException
try (var scope = StructuredTaskScope.open()) {
    var u = scope.fork(() -> fetchUser(id));
    var o = scope.fork(() -> fetchOrder(id));

    scope.join(); // throws StructuredTaskScope.FailedException if any subtask failed

    return new Response(u.get(), o.get());
}
```

#### 7.2 `Joiner.allSuccessfulOrThrow()` — All Must Succeed

```java
// Returns a List<T> of results from all subtasks
// Throws if ANY subtask fails
try (var scope = StructuredTaskScope.open(
        Joiner.<String>allSuccessfulOrThrow())) {

    for (String url : urls) {
        scope.fork(() -> fetchContent(url));
    }

    List<String> results = scope.join(); // List of all results
    results.forEach(System.out::println);
}
```

#### 7.3 `Joiner.anySuccessfulOrThrow()` — First Success Wins

Cancels remaining subtasks as soon as **one** succeeds. Throws if **all** fail.

```java
// Returns the FIRST successful result; cancels the rest
try (var scope = StructuredTaskScope.open(
        Joiner.<String>anySuccessfulOrThrow())) {

    scope.fork(() -> queryPrimaryServer());
    scope.fork(() -> querySecondaryServer());
    scope.fork(() -> queryTertiaryServer());

    String fastestResult = scope.join(); // returns first winner
    System.out.println("Got: " + fastestResult);
}
```

#### 7.4 `Joiner.awaitAll()` — Wait for All, No Exception

```java
// Waits for ALL subtasks regardless of outcome
// Returns a Stream<Subtask<T>> for manual inspection
try (var scope = StructuredTaskScope.open(
        Joiner.<String>awaitAll())) {

    var t1 = scope.fork(() -> mightFail("task1"));
    var t2 = scope.fork(() -> mightFail("task2"));

    Stream<Subtask<String>> subtasks = scope.join();

    subtasks.forEach(st -> {
        if (st.state() == Subtask.State.SUCCESS) {
            System.out.println("OK: " + st.get());
        } else {
            System.out.println("FAILED: " + st.exception().getMessage());
        }
    });
}
```

---

## 8. Error Handling

### The Default Policy: `FailedException`

With the default `StructuredTaskScope.open()`, if any subtask throws an exception, `scope.join()` wraps and rethrows it as a `StructuredTaskScope.FailedException`:

```java
try (var scope = StructuredTaskScope.open()) {
    var user  = scope.fork(() -> findUser(userId));
    var order = scope.fork(() -> fetchOrder(orderId));

    try {
        scope.join();
    } catch (StructuredTaskScope.FailedException e) {
        // e.getCause() is the original exception from the subtask
        System.err.println("A subtask failed: " + e.getCause().getMessage());
        throw e; // re-throw or handle
    }

    return new Result(user.get(), order.get());
}
```

### Checking Subtask State Manually

When using `Joiner.awaitAll()`, you must check each subtask yourself:

```java
if (subtask.state() == Subtask.State.SUCCESS) {
    processResult(subtask.get());
} else if (subtask.state() == Subtask.State.FAILED) {
    logError(subtask.exception());
}
```

---

## 9. Timeouts

### Using `Joiner.onTimeout()` (New in JEP 525 / JDK 26)

JEP 525 added `Joiner.onTimeout()` to allow clean timeout handling without a custom `Joiner` implementation:

```java
import java.time.Duration;

try (var scope = StructuredTaskScope.open(
        Joiner.<String>anySuccessfulOrThrow()
              .onTimeout(Duration.ofSeconds(5), () -> "default-fallback"))) {

    scope.fork(() -> callSlowService());
    scope.fork(() -> callAnotherService());

    // If no subtask succeeds within 5 seconds, returns "default-fallback"
    String result = scope.join();
    System.out.println("Result: " + result);
}
```

### Using `scope.joinUntil()` (JDK 21–24)

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var task = scope.fork(() -> fetchDataWithRetry());

    // Deadline-based join (throws TimeoutException if exceeded)
    scope.joinUntil(Instant.now().plusSeconds(3));
    scope.throwIfFailed();

    return task.get();
}
```

---

## 10. Scoped Values and Inheritance

Added in **JEP 437** (JDK 20): when a `StructuredTaskScope` is created inside the scope of a `ScopedValue`, all forked subtasks **automatically inherit** the scoped value binding.

This is useful for passing contextual data (e.g., request IDs, user principals) without threading it through every method signature.

```java
// Define a scoped value (finalized in Java 25)
static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

void handleRequest(String reqId) throws InterruptedException {
    ScopedValue.runWhere(REQUEST_ID, reqId, () -> {
        try (var scope = StructuredTaskScope.open()) {

            scope.fork(() -> {
                // subtask automatically inherits REQUEST_ID = reqId
                String id = REQUEST_ID.get(); // "req-42"
                return fetchData(id);
            });

            scope.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });
}
```

**Key benefit:** No need to explicitly pass the request ID — scoped values flow naturally down the call tree.

---

## 11. Nested Scopes

Scopes can be nested. Each inner scope must complete before the outer scope can close. This enforces the "tree of tasks" structure.

```java
try (var outerScope = StructuredTaskScope.open()) {

    outerScope.fork(() -> {
        // This subtask itself opens an inner scope
        try (var innerScope = StructuredTaskScope.open()) {
            var a = innerScope.fork(() -> stepA());
            var b = innerScope.fork(() -> stepB());
            innerScope.join(); // inner must complete first
            return combine(a.get(), b.get());
        }
    });

    outerScope.fork(() -> parallelWork());

    outerScope.join();
}
// Both inner and outer scopes have closed cleanly here
```

This nesting mirrors structured control flow and makes the task hierarchy explicit and inspectable.

---

## 12. Custom Joiners

For advanced use cases, you can implement the `Joiner` interface directly. The interface requires:

- `onFork(Subtask<? extends T>)` — called when a subtask is forked; return `true` to cancel the scope immediately.
- `onComplete(Subtask<? extends T>)` — called when a subtask finishes; return `true` to cancel remaining tasks.
- `result()` — called after `join()` to produce the final value.

### Example: Collect only successful results, ignore failures

```java
class CollectSuccessfulJoiner<T> implements StructuredTaskScope.Joiner<T, List<T>> {

    private final List<T> results = new CopyOnWriteArrayList<>();

    @Override
    public boolean onFork(Subtask<? extends T> subtask) {
        return false; // don't cancel on fork
    }

    @Override
    public boolean onComplete(Subtask<? extends T> subtask) {
        if (subtask.state() == Subtask.State.SUCCESS) {
            results.add(subtask.get());
        }
        return false; // don't cancel scope; let all tasks run
    }

    @Override
    public List<T> result() {
        return Collections.unmodifiableList(results);
    }
}
```

Usage:

```java
try (var scope = StructuredTaskScope.open(new CollectSuccessfulJoiner<String>())) {
    for (String endpoint : endpoints) {
        scope.fork(() -> callEndpoint(endpoint));
    }
    List<String> successfulResults = scope.join();
    successfulResults.forEach(System.out::println);
}
```

> **Important:** Always create a **new `Joiner` instance** per scope. Never share or reuse Joiner objects across scopes.

---

## 13. Observability and Thread Dumps

One of the key advantages of structured concurrency is how it improves thread dump readability.

### Unstructured Thread Dump (confusing)

```
"pool-1-thread-1" — waiting on Future.get()
"pool-1-thread-2" — executing fetchUser()
"pool-1-thread-3" — executing fetchOrder()
```
There is no visible link between these threads in the dump.

### Structured Thread Dump (clear hierarchy)

```
"main" — createInvoice()
  └── "scope-1" — waiting on scope.join()
        ├── "virtual-thread-1" — fetchUser()      [child of main]
        ├── "virtual-thread-2" — fetchOrder()     [child of main]
        └── "virtual-thread-3" — fetchTemplate()  [child of main]
```

Structured concurrency enables tools and JVM diagnostics to show **which subtasks belong to which parent task**, making production debugging dramatically easier.

---

## 14. When NOT to Use Structured Concurrency

Structured Concurrency is not meant to replace all concurrent primitives. Here is a quick guide:

| Scenario | Right Tool |
|---|---|
| Forking multiple subtasks that **must all complete** for a single result | ✅ `StructuredTaskScope` |
| Race two services, take the **first to succeed** | ✅ `anySuccessfulOrThrow()` |
| Long-running background services or daemons | ❌ Use `ExecutorService` |
| Producer-consumer pipelines with queues | ❌ Use `BlockingQueue` + threads |
| Complex async chains with transformations | ❌ `CompletableFuture` may fit better |
| Fire-and-forget tasks not tied to a parent | ❌ Executor or virtual threads directly |

---

## 15. Full Real-World Example

Below is a complete, practical example: fetching a stock portfolio summary by concurrently calling three services — holdings, market price, and company details — with a timeout and clean error handling.

```java
import java.util.concurrent.*;
import java.time.Duration;

public class StockPortfolioService {

    record StockSummary(String symbol, String company, double price, double holdingValue) {}

    public StockSummary getStockSummary(String symbol)
            throws InterruptedException, StructuredTaskScope.FailedException {

        try (var scope = StructuredTaskScope.open(
                Joiner.<Object>allSuccessfulOrThrow()
                      .onTimeout(Duration.ofSeconds(5), () -> null))) {

            // Fork three independent I/O-bound subtasks
            Subtask<Double>  priceTask   = scope.fork(() -> marketDataService.getPrice(symbol));
            Subtask<String>  companyTask = scope.fork(() -> companyService.getName(symbol));
            Subtask<Double>  holdingTask = scope.fork(() -> portfolioService.getHolding(symbol));

            scope.join(); // Blocks until all succeed, any fails, or timeout fires

            double price        = priceTask.get();
            String companyName  = companyTask.get();
            double holding      = holdingTask.get();

            return new StockSummary(symbol, companyName, price, price * holding);

        }
        // If the try-with-resources block exits, all subtasks are guaranteed to have stopped
    }
}
```

### What this demonstrates:

- **Parallelism:** All three service calls happen concurrently in virtual threads.
- **Lifecycle control:** No subtask outlives the `try` block.
- **Timeout:** After 5 seconds, the scope is cancelled automatically.
- **Failure propagation:** If any service throws, `scope.join()` re-throws immediately.
- **Readability:** The control flow reads like sequential code.

---

## 16. Summary

| Concept | Description |
|---|---|
| `StructuredTaskScope.open()` | Opens a new scope owned by the current thread (JDK 25+) |
| `scope.fork(callable)` | Spawns a subtask in a new virtual thread; returns `Subtask<T>` |
| `scope.join()` | Blocks until subtasks finish per the joiner's policy; returns the joiner's result |
| `Joiner` | Pluggable interface controlling completion policy and the return type of `join()` |
| `Joiner.allSuccessfulOrThrow()` | All subtasks must succeed; returns `List<T>` |
| `Joiner.anySuccessfulOrThrow()` | First subtask to succeed wins; cancels others |
| `Joiner.awaitAll()` | Wait for all, no exception; inspect results manually via `Stream<Subtask<T>>` |
| `Joiner.onTimeout(duration, supplier)` | Produce a fallback result if timeout expires (new in JEP 525) |
| Scoped Value inheritance | Subtasks automatically inherit `ScopedValue` bindings from the owning thread |
| Observability | Thread dumps show subtask hierarchy linked to parent scope |

### The Key Principle

Structured Concurrency enforces a simple rule through the API itself:

> **A subtask thread cannot outlive the scope that forked it.**

This single constraint eliminates thread leaks, simplifies error propagation, and makes concurrent code as readable and maintainable as sequential code.

---
# The Future is Now: Asynchronous Programming with CompletableFuture

Welcome back to **our concurrency series**! In our first discussion, we likely touched on the traditional models of threading. Today, we’re taking a giant leap forward into the world of modern, non-blocking asynchronous programming in Java with `CompletableFuture`.

---

## Introduction: The Problem with “Waiting”
In traditional synchronous (blocking) code, when you call a method — say, to fetch data from a database or a remote API — your thread **stops and waits**. It’s stuck, unable to do anything else until the result comes back.

### The “Before” (Synchronous):
```java
// Our thread is blocked here for 2 seconds, doing nothing.
String data = traditionalDatabaseCall(); 
println("Got data: " + data);

public String traditionalDatabaseCall() {
    try {
        Thread.sleep(2000); // Simulating a slow network call
    } catch (InterruptedException e) { }
    return "Some data";
}
```

In a high-traffic application, this is a disaster. Every blocked thread is a resource wasted. You’d need thousands of threads to handle thousands of concurrent users, which is not scalable.

Java 5 introduced the `Future` interface, which was a good first step. It represented a value that **would eventually** exist. But it had a major flaw: the only way to get the value was `future.get()`, which... **blocks!**

Enter `CompletableFuture` (Java 8). It's a "future" you can **react** to. Instead of asking "Are you done yet?" (blocking), you tell it, "When **you are done, then** do this..." This enables a non-blocking, event-driven, and compositional style of programming that is incredibly powerful.

---

## The Basics: Creating a CompletableFuture
You’ll primarily create `CompletableFutures` in two ways, both of which (by default) run your task in the global `ForkJoinPool.commonPool()`.

### runAsync()
Use this when you want to run a task asynchronously but don’t need a return value (like a `Runnable`).
**Use Case:** Sending an email, logging an event, firing off a “fire-and-forget” task.

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

println("Main thread: Sending a notification.");
CompletableFuture<Void> notificationFuture = CompletableFuture.runAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(2); // Simulating work
        println("Async thread: Notification sent!");
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
});
println("Main thread: Continuing with other work.");
// Wait for the future to complete just for this demo
notificationFuture.join(); 
println("Main thread: Done.");
```

### supplyAsync()
This is the workhorse. Use it when your async task needs to return a value (like a `Callable`).
**Use Case:** Fetching data from an API, querying a database, performing a complex calculation.

```java
println("Main thread: Fetching user data...");

CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(2); // Simulating I/O
        System.out.println("Async thread: Got user data!");
        return "User-123-Data";
    } catch (InterruptedException e) {
        return "Error";
    }
});

println("Main thread: Doing other work...");
// .join() is a blocking call to get the result.
// We only use it here to end the demo. In real code,
// you'd chain tasks instead of joining.
String userData = userFuture.join(); 
println("Main thread: Received data: " + userData);
```

### Pro-Tip: Using Custom Executors
The default `ForkJoinPool.commonPool()` is great for **CPU-bound** tasks. But for **I/O-bound** tasks (like API calls or database queries), you risk starving the pool. It's best practice to provide your own `Executor` specifically for I/O.

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;

// Create a thread pool optimized for our I/O tasks
ExecutorService ioExecutor = Executors.newFixedThreadPool(10); 
CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> {
    // This task now runs on a thread from ioExecutor,
    // not the common ForkJoinPool.
    println("Async (custom pool): Fetching user...");
    // ... simulate API call ...
    return "User Data";
}, ioExecutor); // <-- Pass the custom executor here
```

---

## Chaining Tasks: thenApply(), thenAccept(), thenRun()
This is where `CompletableFuture` shines. You can build a pipeline of operations that execute automatically when the previous stage completes.

* **thenApply(Function<T, U>):** **Transforms** the result. Takes the result, does something with it, and returns a **new** result. (Like `Stream.map()`).
* **thenAccept(Consumer<T>):** **Consumes** the result. Takes the result and performs an action (e.g., prints it, saves it). Returns `CompletableFuture<Void>`. (Like `Stream.forEach()`).
* **thenRun(Runnable):** **Runs after**. Doesn't care about the result, just runs a `Runnable` after the stage completes. Returns `CompletableFuture<Void>`.

```java
ExecutorService ioExecutor = Executors.newFixedThreadPool(10);

println("Main: Kicking off the pipeline...");

CompletableFuture<Void> pipeline = CompletableFuture.supplyAsync(() -> {
    // 1. Fetches a user ID
    System.out.println("Async 1: Getting user ID...");
    sleep(1);
    return 123L;
}, ioExecutor)
.thenApply(userId -> {
    // 2. Uses the ID to fetch a name (transform)
    println("Async 2: Getting username for ID " + userId);
    sleep(1);
    return "User:Alice";
})
.thenApply(username -> {
    // 3. Transforms the name to uppercase
    println("Async 3: Capitalizing " + username);
    sleep(1);
    return username.toUpperCase();
})
.thenAccept(uppercasedName -> {
    // 4. Consumes the final result
    println("Async 4: Saving " + uppercasedName + " to database.");
    // ... saveToDb(uppercasedName) ...
    sleep(1);
})
.thenRun(() -> {
    // 5. Runs after everything is done
    println("Async 5: Pipeline complete. Logging event.");
});

println("Main: Pipeline is running. I can do other things!");
pipeline.join(); // Wait for the whole pipeline to finish
println("Main: All done.");

// Helper sleep method
private static void sleep(int seconds) {
    try { TimeUnit.SECONDS.sleep(seconds); } 
    catch (InterruptedException e) { }
}
```

---

## Combining Futures: Building Complex Workflows
What if you need to run tasks in parallel? Or run one task **after** another (but the second task is **also** async)?



### thenCombine()
Use this when you have **two independent** futures and want to do something with **both** of their results when they are **both** complete.
**Use Case:** Get user info + get user permissions, then combine them into a `UserContext` object.

```java
CompletableFuture<String> userInfo = CompletableFuture.supplyAsync(() -> {
    sleep(2);
    println("Async: Got user info.");
    return "Alice, 30";
});

CompletableFuture<String> userPerms = CompletableFuture.supplyAsync(() -> {
    sleep(3);
    println("Async: Got user permissions.");
    return "ROLE_ADMIN";
});

println("Main: Fired off both futures.");

// thenCombine waits for BOTH userInfo and userPerms to finish
CompletableFuture<String> userContext = userInfo.thenCombine(userPerms,
    (info, perms) -> {
        // This BiFunction runs only after both are done
        println("Async: Combining results.");
        return "UserContext[" + info + ", " + perms + "]";
    }
);

println("Main: Waiting for combined result...");
println(userContext.join()); // Blocks for ~3 seconds total (not 2+3=5)
```

### thenCompose()
Use this for **dependent** tasks, where the second async task **needs the result** from the first to even begin. This is the asynchronous equivalent of `Stream.flatMap()`.

```java
// WRONG way (using thenApply)
CompletableFuture<Long> userIdFuture = CompletableFuture.supplyAsync(() -> 123L);

// This gives you a nested CompletableFuture... awkward!
CompletableFuture<CompletableFuture<String>> nestedFuture = 
    userIdFuture.thenApply(id -> {
        // We return a *new* future here
        return CompletableFuture.supplyAsync(() -> "User:" + id);
    });

// RIGHT way (using thenCompose)
CompletableFuture<String> userFuture = userIdFuture.thenCompose(id -> {
    // thenCompose expects you to return a future
    // and automatically unwraps it for the chain.
    println("Async: Got ID " + id + ", fetching details...");
    return CompletableFuture.supplyAsync(() -> "UserDetails:" + id);
});

println(userFuture.join()); // Prints "UserDetails:123"
```

### allOf() / anyOf()
Use these to coordinate multiple futures at once.

* **CompletableFuture.allOf(f1, f2, f3):** Creates a new future that completes only when **all** of the provided futures are done.
* **CompletableFuture.anyOf(f1, f2, f3):** Creates a new future that completes as soon as **any one** of the provided futures finishes.

```java
// anyOf(): "Race" multiple services
CompletableFuture<String> serviceA = CompletableFuture.supplyAsync(() -> {
    sleep(3); return "Result from A";
});
CompletableFuture<String> serviceB = CompletableFuture.supplyAsync(() -> {
    sleep(1); return "Result from B";
});
CompletableFuture<String> serviceC = CompletableFuture.supplyAsync(() -> {
    sleep(2); return "Result from C";
});

CompletableFuture<Object> fastestResult = CompletableFuture.anyOf(serviceA, serviceB, serviceC);
println("Fastest response: " + fastestResult.join()); // Prints "Result from B"

// allOf(): Wait for a group of tasks
CompletableFuture<Void> allDone = CompletableFuture.allOf(serviceA, serviceB, serviceC);
allDone.join(); // Waits for all 3 to finish (~3 seconds total)
println("All services are finished.");
```

---

## Error Handling: Building Resilient Pipelines
In an async chain, a traditional `try-catch` block won't work. `CompletableFuture` **propagates exceptions** down the chain.



### exceptionally()
This is your **catch** block. It lets you "recover" the pipeline by providing a default value if an exception occurs.

```java
CompletableFuture<String> riskyFuture = CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) {
        println("Async: Simulating a failure!");
        throw new RuntimeException("Oops, DB is down!");
    }
    return "Successful Data";
})
.exceptionally(ex -> {
    println("Async: Caught error: " + ex.getMessage());
    return "Default Fallback Data"; // Provide a default to recover
});
```

### handle()
This is your **finally** block, but more powerful. It receives **both** the result and the exception.

```java
CompletableFuture<String> processedFuture = CompletableFuture.supplyAsync(() -> {
    if (Math.random() < 0.5) throw new RuntimeException("Failure!");
    return "Success";
})
.handle((result, exception) -> {
    if (exception != null) {
        println("Async: Handling error: " + exception.getMessage());
        return "Recovered";
    }
    return "Transformed:" + result;
});
```

### whenComplete()
This is a true **finally** block for side-effects (like logging). It **cannot change the outcome**.

---

## Real-World Example: Parallel API Calls
Let’s fetch data for a user dashboard. We need User Details, Order History, and Payment Info.

**The Synchronous (Slow) Way:** Total time = **~6 seconds** (1 + 3 + 2).
**The CompletableFuture (Fast) Way:** Total time = **~3 seconds** (the time of the longest call).

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

public void loadDashboardAsync() {
    long start = System.currentTimeMillis();
    
    // 1. Fire off all requests in parallel
    var userFuture = CompletableFuture.supplyAsync(this::getUserDetails, executor);
    var ordersFuture = CompletableFuture.supplyAsync(this::getOrderHistory, executor);
    var paymentFuture = CompletableFuture.supplyAsync(this::getPaymentInfo, executor);
    
    // 2. Wait for ALL of them to complete
    CompletableFuture<Void> allFutures = CompletableFuture.allOf(userFuture, ordersFuture, paymentFuture);
    allFutures.join(); 
    
    // 3. Get the results
    String user = userFuture.join();
    String orders = ordersFuture.join();
    String payment = paymentFuture.join();
    
    println("Dashboard: " + user + ", " + orders + ", " + payment);
    long end = System.currentTimeMillis();
    println("Total time: " + (end - start) + "ms");
}
```

---

## Conclusion
`CompletableFuture` represents a critical advancement in Java, enabling a shift from blocking threads to a **non-blocking, asynchronous paradigm.**
```markdown
```

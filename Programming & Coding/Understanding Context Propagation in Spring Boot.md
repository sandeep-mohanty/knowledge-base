# Why Your Async Threads Don’t Know Who You Are — Understanding Context Propagation in Spring Boot

![Header Image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*aCSZfXokBctiCTyghJsXmg.png)

## 1. What is “Request Context Propagation”?
In Spring Boot, every HTTP request handled by the application has a **request context** — this includes things like:

* The **HTTP headers**
* The **request attributes**
* The **SecurityContext** (like logged-in user)
* **MDC logging context** (for correlation IDs or request IDs)

When you use asynchronous processing — for example with:
`@Async`
`public void someAsyncMethod() { ... }`

Spring executes that method in a **different thread** (from a thread pool).
But by default, that **new thread doesn’t carry the parent request context** (like headers, MDC, or security info).
So, if your async thread tries to log or use authentication info — it won’t find it.

## 2. Why Context Is Lost
When you call a method annotated with `@Async`, Spring delegates execution to a thread in a **TaskExecutor**.
The new thread is not aware of:
* `RequestContextHolder` (web request context)
* `SecurityContextHolder` (authentication)
* `MDC` (logging context)

So, the context does **not propagate automatically** to new threads.

## 3. Real-World Problem Example
Imagine this microservice: **OrderService**

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @Autowired
    private OrderProcessingService orderProcessingService;

    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody OrderRequest order) {
        orderProcessingService.processOrderAsync(order);
        return ResponseEntity.ok("Order processing started!");
    }
}
```

Async service:
```java
@Service
public class OrderProcessingService {

    @Async
    public void processOrderAsync(OrderRequest order) {
        String user = SecurityContextHolder.getContext().getAuthentication().getName();
        System.out.println("Processing order for user: " + user);
    }
}
```

**Output:**
`Processing order for user: null`

Because the **SecurityContext** (which stores the authenticated user) was **not propagated** to the new async thread.

## 4. Solution: Context Propagation Techniques

### Option 1: Use Spring’s DelegatingSecurityContextAsyncTaskExecutor
This ensures that **SecurityContext** travels with your async thread.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor delegateExecutor = new ThreadPoolTaskExecutor();
        delegateExecutor.setCorePoolSize(5);
        delegateExecutor.setMaxPoolSize(10);
        delegateExecutor.setQueueCapacity(100);
        delegateExecutor.initialize();

        // Wrap with DelegatingSecurityContextAsyncTaskExecutor
        return new DelegatingSecurityContextAsyncTaskExecutor(delegateExecutor);
    }
}
```

Now, your **SecurityContext** will propagate to async threads correctly.
**Output:**
`Processing order for user: praveencodes`

### Option 2: Propagate Request Attributes using RequestContextHolder
If you need **HTTP headers**, **request params**, etc.

```java
@Async
public void processOrderAsync(OrderRequest order) {
    ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    HttpServletRequest request = attrs.getRequest();
    System.out.println("Header: " + request.getHeader("X-Request-ID"));
}
```

But this still doesn’t propagate automatically. So you can **manually capture and set** it before calling async method.

```java
@PostMapping
public ResponseEntity<String> createOrder(@RequestBody OrderRequest order) {
    ServletRequestAttributes attributes =
        (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();

    orderProcessingService.processOrderAsync(order, attributes);
    return ResponseEntity.ok("Order processing started!");
}
```

And inside async method:
```java
@Async
public void processOrderAsync(OrderRequest order, RequestAttributes attributes) {
    RequestContextHolder.setRequestAttributes(attributes);
    System.out.println("Header: " + ((ServletRequestAttributes) attributes).getRequest().getHeader("X-Request-ID"));
}
```
**Output:**
`Header: 12345-trace-id`

### Option 3: MDC (Logging Context) Propagation
If you use **MDC** for logging correlation IDs (like Sleuth or custom filters):

```java
MDC.put("traceId", "abc123");
asyncService.callSomething();
```

Use a **TaskDecorator** in the executor to propagate MDC context:

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("Async-");
        executor.setTaskDecorator(new ContextCopyingDecorator());
        executor.initialize();
        return executor;
    }

    public static class ContextCopyingDecorator implements TaskDecorator {
        @Override
        public Runnable decorate(Runnable runnable) {
            Map<String, String> contextMap = MDC.getCopyOfContextMap();
            return () -> {
                if (contextMap != null) {
                    MDC.setContextMap(contextMap);
                }
                try {
                    runnable.run();
                } finally {
                    MDC.clear();
                }
            };
        }
    }
}
```

This way your logs remain traceable across async calls.

## 5. Real-Time Use Case Example
**Scenario:**
Microservice A (OrderService) receives a REST API call with:
1. JWT Token in header (Authorization)
2. Trace ID header (X-Correlation-ID)

Inside the request, you:
* Save the order in DB
* Then call a downstream service (like EmailService) asynchronously

**Problem:**
Without context propagation:
* Async email thread doesn’t know who the user is
* Logging doesn’t include **X-Correlation-ID**
* Distributed tracing tools like Zipkin/Sleuth can’t connect logs

**With context propagation:**
* The async thread gets the same **SecurityContext** (user info)
* Same MDC trace ID → logs easily correlated
* Microservice logs and traces remain consistent

**Result:** Easier debugging, monitoring, and observability across async tasks.

---

## 1. The Problem Recap
In older versions of Spring Boot (before 3.2), we used:
* `DelegatingSecurityContextAsyncTaskExecutor` → for **SecurityContext**
* `RequestContextHolder` → for web request attributes
* `TaskDecorator` → for MDC logging context

That meant you had to write **different mechanisms** for each kind of context. This was repetitive and error-prone.

Spring Boot 3.2 introduced **ContextPropagator API** — one unified approach to handle **all kinds of contexts** automatically.

## 2. What Is ContextPropagator?
A **ContextPropagator** is a new SPI (Service Provider Interface) introduced in Spring Framework 6.1 / Boot 3.2.

It tells Spring **what context to capture**, **how to restore it** in another thread, and **how to reset it** after execution.
Each propagator is responsible for one type of context (e.g. Security, Request, MDC).

![Context Propagator Diagram](https://miro.medium.com/v2/resize:fit:720/format:webp/1*UAUXd2v95didfwVZHGm8VQ.png)

## 4. How It Works Internally
When Spring runs an async task or reactive flow, it checks the list of registered **ContextPropagators**.
Each propagator does 3 things:

```java
interface ContextPropagator<C> {
    C capture();            // Capture context from current thread
    void restore(C context); // Set context in new thread
    void reset(C context);   // Cleanup after task
}
```

Spring then wraps your **Runnable** or **Callable** with these steps automatically.
So — you don’t need to write manual code to copy MDC, security, or request data anymore.

## 5. Example: Auto Propagation in Async Threads

**build.gradle**
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-aop'
}
```

**AsyncConfig.java**
```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public ThreadPoolTaskExecutor taskExecutor(ApplicationContext context) {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("async-");
        executor.initialize();

        // Enable automatic context propagation
        executor.setTaskDecorator(
                ContextPropagators.defaultContextPropagator(context)
        );

        return executor;
    }
}
```
`ContextPropagators.defaultContextPropagator(context)` automatically registers **Security**, **Request**, and **MDC** propagators if those libraries are present.

## 6. Demo Flow
```java
@RestController
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping("/orders")
    public String placeOrder(@RequestHeader("X-Trace-ID") String traceId) {
        MDC.put("traceId", traceId);
        orderService.processOrder();
        return "Order placed!";
    }
}

@Service
public class OrderService {

    @Async
    public void processOrder() {
        String traceId = MDC.get("traceId");
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        System.out.println("Async processing by " + username + " with TraceID=" + traceId);
    }
}
```

**Output:**
`Async processing by praveencodes with TraceID=abc-123`
All contexts (Security, MDC, and Request) automatically propagated. No need for manual **TaskDecorator**, **RequestContextHolder**, or wrapping executors.

## 7. Custom Context Propagator Example
Let’s say you have your own thread-local called **TenantContext** (for multi-tenant apps):

```java
public class TenantContext {
    private static final ThreadLocal<String> tenant = new ThreadLocal<>();
    public static void setTenant(String id) { tenant.set(id); }
    public static String getTenant() { return tenant.get(); }
    public static void clear() { tenant.remove(); }
}
```

You can create a custom propagator:
```java
@Component
public class TenantContextPropagator implements ContextPropagator<String> {

    @Override
    public String capture() {
        return TenantContext.getTenant();
    }

    @Override
    public void restore(String context) {
        if (context != null) TenantContext.setTenant(context);
    }

    @Override
    public void reset(String context) {
        TenantContext.clear();
    }
}
```
Spring will automatically detect and apply it to async tasks. Now your **TenantContext** also flows seamlessly across threads.

![Thread Context Flow Diagram](https://miro.medium.com/v2/resize:fit:720/format:webp/1*fKsjzDaZqMLsANpOHZd-wA.png)

## 9. Real-World Use Case
**Scenario:**
Your microservice handles user requests and spawns background threads to:
* Send notifications asynchronously
* Audit the operation
* Push updates to Kafka

Each async task needs:
1. The logged-in user (from **SecurityContext**)
2. The tenant (from custom context)
3. The trace ID for logs (from MDC)

With **ContextPropagator**, all this travels automatically — no extra wiring.

**Benefit:**
* Traceability in distributed logging
* Simplified async architecture
* Zero manual context management
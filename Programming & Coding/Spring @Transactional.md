# @Transactional Almost Killed Our Database. Here’s What I Learned.

## _The Spring Boot annotation everyone uses and nobody understands — until production melts down at 3 AM_

Look, I need to tell you about the worst on-call shift of my life.

It was 2:47 AM on a Tuesday. My phone exploded with PagerDuty alerts. Database connections maxed out. API response times climbing from 200ms to 8 seconds. Customer support Slack channel filling with angry messages.

I SSHed into the production server, hands shaking, coffee cold.

The logs showed something I’d seen a hundred times in development but never thought twice about:

HikariPool-1 - Connection is not available, request timed out after 30000ms

Every. Single. Request. Timing out.

Our PostgreSQL connection pool was exhausted. All 50 connections locked. Queries piling up. Users getting timeout errors. Money being lost with every passing second.

And the root cause? One fucking annotation I’d copy-pasted from Stack Overflow six months earlier: `@Transactional`.

## The Annotation Everyone Uses (And Nobody Understands)

Here’s what I thought `@Transactional` did:

“It wraps my method in a database transaction. If something fails, it rolls back. Easy.”

Here’s what it actually does:

It holds a database connection for the **entire duration** of your method. Not just for database operations. For everything. HTTP calls. File I/O. Complex business logic. All of it.

Let me show you the code that killed us:

```java
@Service  
public class OrderService {

        @Transactional  
    public OrderResponse createOrder(OrderRequest request) {  
        // Step 1: Save order to database (20ms)  
        Order order = orderRepository.save(request.toEntity());

                // Step 2: Call payment gateway API (1200ms)  
        PaymentResponse payment = paymentGateway.charge(  
            order.getTotalAmount(),   
            request.getPaymentMethod()  
        );

                // Step 3: Call inventory service API (800ms)  
        inventoryService.reserveItems(order.getItems());

                // Step 4: Send confirmation email (400ms)  
        emailService.sendOrderConfirmation(order);

                // Step 5: Update order status (10ms)  
        order.setStatus(OrderStatus.CONFIRMED);  
        orderRepository.save(order);

                return OrderResponse.from(order);  
    }  
}
```
Looks reasonable, right?

Wrong. Dead wrong.

That `@Transactional` annotation means this method holds a database connection for **2,430 milliseconds**.

But only **30ms** of that time actually touches the database.

The other 2,400ms? We’re holding a precious database connection hostage while waiting for external APIs and sending emails.

## The Math That Should’ve Terrified Me

Our production setup:

-   Database connection pool: 50 connections (HikariCP default)
-   Average requests per second: 40
-   Connection hold time per request: 2.4 seconds

Do the math:

40 requests/second × 2.4 seconds = 96 concurrent connections needed

We had 50. We needed 96.

Disaster was inevitable. I just didn’t know it yet.

## When Production Finally Broke

The system held up fine for months. We were averaging 20–30 requests per second. Plenty of headroom.

Then we ran a marketing campaign.

Traffic spiked to 60 requests per second. Within 3 minutes, every database connection was locked. New requests couldn’t get connections. Timeouts everywhere.

The worst part? The database itself was fine. CPU at 15%. Memory at 40%. Barely stressed.

The bottleneck was our idiotic use of `@Transactional`.

## What I Learned the Hard Way

## Mistake #1: Transactions Spanning External Calls

Never, ever hold a database transaction open while calling external services.

**Why?** External calls can take seconds. Transactions should take milliseconds.

**The fix:**

```java
@Service  
public class OrderService {

        // Only database operations inside @Transactional  
    @Transactional  
    public Order saveOrder(OrderRequest request) {  
        return orderRepository.save(request.toEntity());  
    }

        // Orchestration outside transaction  
    public OrderResponse createOrder(OrderRequest request) {  
        // Step 1: Save order (transaction held for 20ms)  
        Order order = saveOrder(request);

                // Step 2-4: External calls (no transaction)  
        PaymentResponse payment = paymentGateway.charge(...);  
        inventoryService.reserveItems(...);  
        emailService.sendOrderConfirmation(...);

                // Step 5: Update order (transaction held for 10ms)  
        confirmOrder(order.getId());

                return OrderResponse.from(order);  
    }

        @Transactional  
    public void confirmOrder(Long orderId) {  
        Order order = orderRepository.findById(orderId)  
            .orElseThrow();  
        order.setStatus(OrderStatus.CONFIRMED);  
        orderRepository.save(order);  
    }  
}
```
Now we hold database connections for 30ms total instead of 2,400ms.

That’s an 80x improvement in connection utilization.

## Mistake #2: @Transactional on the Wrong Methods

I had `@Transactional` on methods that didn't even touch the database.

```java
@Transactional  // WHY???  
public OrderSummary calculateOrderSummary(Order order) {  
    BigDecimal subtotal = order.getItems().stream()  
        .map(item -> item.getPrice().multiply(item.getQuantity()))  
        .reduce(BigDecimal.ZERO, BigDecimal::add);

        BigDecimal tax = subtotal.multiply(TAX\_RATE);  
    BigDecimal total = subtotal.add(tax);

        return new OrderSummary(subtotal, tax, total);  
}
```

This is pure computation. No database. No side effects. No reason for a transaction.

But every time this method ran, it grabbed a database connection and held it for no reason.

**The fix:** Remove `@Transactional` from methods that don't need it. Obvious in hindsight. Invisible when you're coding fast.

## Mistake #3: Not Understanding Propagation

This one almost broke me.

```java
@Service  
public class UserService {

        @Autowired  
    private EmailService emailService;

        @Transactional  
    public void registerUser(UserRequest request) {  
        User user = userRepository.save(request.toEntity());  
        emailService.sendWelcomeEmail(user);  
    }  
}

@Service  
public class EmailService {

        @Transactional  // Problem here  
    public void sendWelcomeEmail(User user) {  
        Email email = Email.builder()  
            .to(user.getEmail())  
            .subject("Welcome!")  
            .build();

                emailRepository.save(email);  
        externalEmailService.send(email);  // External API call  
    }  
}
```

Both methods have `@Transactional`. What happens?

By default, `@Transactional` uses `Propagation.REQUIRED`. This means:

-   If a transaction exists, join it
-   If not, create a new one

So `sendWelcomeEmail()` **joins** the transaction from `registerUser()`.

Result? The external email API call (which takes 400ms) happens **inside** the user registration transaction.

If the email service is down? The entire user registration rolls back. Even though the user was successfully saved to the database.

**The fix:**

```java
@Service  
public class EmailService {

        @Transactional(propagation = Propagation.REQUIRES\_NEW)  
    public void sendWelcomeEmail(User user) {  
        // This runs in its own transaction  
        // Independent of the caller's transaction  
    }  
}
```
Or better yet, don’t use `@Transactional` for operations involving external services. Use async processing instead.

## Mistake #4: Read-Only Operations Locking Tables

I had `@Transactional` on read operations. Seemed harmless.

```java
@Transactional  
public List<Order> getUserOrders(Long userId) {  
    return orderRepository.findByUserId(userId);  
}
```

In PostgreSQL, even `SELECT` queries hold locks if they're in a transaction.

With default `READ COMMITTED` isolation, it's usually fine. But if your transaction is long-running (remember mistake #1?), you're holding locks on those rows.

**The fix:**

```java
@Transactional(readOnly = true)  
public List<Order> getUserOrders(Long userId) {  
    return orderRepository.findByUserId(userId);  
}
```

The `readOnly = true` flag tells Spring (and Hibernate) that this is a read-only transaction. Hibernate can optimize. The database can optimize. Everyone wins.

Even better? Remove `@Transactional` entirely from read-only operations that don't span multiple queries.

## Mistake #5: Not Handling Checked Exceptions

`@Transactional` only rolls back on **unchecked exceptions** by default.

```java
@Transactional  
public void processPayment(Order order) throws PaymentException {  
    // Save order  
    orderRepository.save(order);

        // Charge payment (throws checked exception)  
    paymentGateway.charge(order);  // throws PaymentException  
}
```

If `PaymentException` is thrown, the transaction **does not roll back**. The order is saved even though payment failed.

**Why?** Because `PaymentException` is a checked exception, and Spring only rolls back on `RuntimeException` by default.

**The fix:**

```java
@Transactional(rollbackFor = Exception.class)  
public void processPayment(Order order) throws PaymentException {  
    orderRepository.save(order);  
    paymentGateway.charge(order);  
}
```
Or better, convert checked exceptions to runtime exceptions:

```java
@Transactional  
public void processPayment(Order order) {  
    orderRepository.save(order);

        try {  
        paymentGateway.charge(order);  
    } catch (PaymentException e) {  
        throw new PaymentProcessingException("Payment failed", e);  
    }  
}
```

## The Production Fix That Saved Us

At 3:30 AM, with the system still melting down, I made an emergency fix:

1.  **Increased connection pool size** from 50 to 100 (bought us time)
2.  **Added** `**@Async**` **to external service calls** (removed them from transactions)
3.  **Wrapped only database operations in** `**@Transactional**` (reduced connection hold time)

```java
@Service  
public class OrderService {

        @Autowired  
    private AsyncService asyncService;

        @Transactional  
    public Order createOrder(OrderRequest request) {  
        // Only database work here  
        Order order = orderRepository.save(request.toEntity());

                // Trigger async operations (not in transaction)  
        asyncService.processOrderAsync(order.getId());

                return order;  
    }  
}

@Service  
public class AsyncService {

        @Async  
    public void processOrderAsync(Long orderId) {  
        // These run outside any transaction  
        paymentGateway.charge(...);  
        inventoryService.reserve(...);  
        emailService.send(...);  
    }  
}
```
System stabilized within 10 minutes.

Average connection hold time dropped from 2,400ms to 35ms.

We could handle 60+ requests/second with the same 50-connection pool.

## What I Wish I’d Known Earlier

After this incident, I spent two weeks studying transaction management. Here’s what I learned:

**Rule 1: Transactions should be short**

-   Milliseconds, not seconds
-   Database operations only
-   No external API calls
-   No file I/O
-   No complex business logic

**Rule 2: Use** `**@Transactional**` **sparingly**

-   Not every method needs it
-   Read-only operations usually don’t need it
-   Pure functions definitely don’t need it

**Rule 3: Understand propagation**

-   `REQUIRED` (default): Join existing or create new
-   `REQUIRES_NEW`: Always create new transaction
-   `MANDATORY`: Must have existing transaction
-   `SUPPORTS`: Use transaction if exists, otherwise don't

**Rule 4: Monitor connection pool usage**

We added metrics:

```java
@Component  
public class HikariMetricsExporter {

        @Autowired  
    private HikariDataSource dataSource;

        @Scheduled(fixedRate = 5000)  
    public void exportMetrics() {  
        HikariPoolMXBean pool = dataSource.getHikariPoolMXBean();

                log.info("Active connections: {}", pool.getActiveConnections());  
        log.info("Idle connections: {}", pool.getIdleConnections());  
        log.info("Threads waiting: {}", pool.getThreadsAwaitingConnection());  
        log.info("Total connections: {}", pool.getTotalConnections());  
    }  
}
```

Now we get paged **before** the connection pool is exhausted.

**Rule 5: Test under load**

Our staging environment had 10 users. Production had 1000.

Load testing would’ve caught this. We didn’t do load testing.

Expensive lesson.

I’ve seen too many backend systems fail for the same reasons — and too many teams learn the hard way.

So I turned those incidents into a practical field manual: real failures, root causes, fixes, and prevention systems.

No theory. No fluff. Just production.

👉 **The Backend Failure Playbook** — How real systems break and how to fix them:

## The Resources That Actually Helped

After this disaster, I became obsessed with understanding Spring transaction management properly.

These resources saved me:

**Grokking the Spring Boot Interview** was the first place I found transaction propagation explained clearly — not just what the options are, but when to actually use each one:

**Spring Boot Troubleshooting Cheatsheet** covers the connection pool exhaustion patterns I hit. Wish I’d read it six months earlier:

For general Java concurrency issues (which transaction management absolutely is), **Grokking the Java Interview** filled in gaps about thread safety and locking I didn’t even know I had:

And if you’re preparing for interviews where they’ll grill you on this stuff, **250+ Spring Certification Practice Questions**has an entire section on transaction management with scenarios like the ones that broke my production:

If you’re dealing with Spring Boot in production and want to avoid the mistakes I made, I keep a running checklist of what actually matters — not theory, just the stuff that breaks:  
**Spring Boot Production Checklist** (free):

All my production engineering resources are organized here if you want to explore what fits your situation:

## The Uncomfortable Truth

Most Spring Boot developers don’t understand `@Transactional`.

They know it “does transactions.” They copy-paste it from examples. It works in development.

Then production happens.

High load happens. External service latency happens. Database connection limits happen.

And suddenly that innocent-looking annotation is the root cause of a 3 AM incident.

I learned this the hard way so you don’t have to.

## What to Do Right Now

If you’re using `@Transactional` in production:

1.  **Audit your codebase** for `@Transactional` annotations
2.  **Check if methods with** `**@Transactional**` **make external API calls** (they shouldn't)
3.  **Monitor your connection pool usage** (add metrics if you don’t have them)
4.  **Test under realistic load** (staging with 10 users doesn’t count)
5.  **Add** `**readOnly = true**` **to read-only operations** (or remove `@Transactional` entirely)

Don’t wait for a 3 AM incident to learn these lessons.

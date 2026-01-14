# 10 Costly Retry Mistakes in Spring Boot Microservices (And How to Avoid Them)

![Retry Mistakes Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*uJz-yyY_eAwcoA-wPqmJHQ.png)

## 1. What is Retry in Spring Boot & Microservices?
**Retry** means:
When a request fails due to a **temporary problem**, the system automatically tries again instead of failing immediately.

**Simple meaning:**
* Network glitch
* Temporary DB lock
* Downstream service restarting

Retry gives the system another chance.

## 2. Why Retry Is REQUIRED in Microservices (Not Optional)
In **monoliths**, everything is local.
In **microservices**, everything is **remote**.

Remote calls can fail due to:
* Network latency
* DNS issues
* Service overload
* Container restarts
* Cloud infra issues

**Failure is NORMAL in distributed systems.** So retry is **defensive programming**.

## 3. Real-Time Production Use Case
**Payment System Example (Very Common)**

> Order Service → Payment Service → Bank API

**What can fail?**
* Bank API timeout
* Payment service slow
* Temporary 500 error

**What should NOT happen?**
* Order marked as FAILED immediately
* User charged twice
* Manual intervention

**Correct behavior**
* Retry payment **2–3 times**
* Wait between retries
* If still fails → move to fallback / manual queue

---

## 4. When Should You Retry? (VERY IMPORTANT)
**Retry only if:**
* Network timeout
* 5xx errors
* Temporary unavailability
* Idempotent operations

**NEVER retry if:**
* Validation error (400)
* Business error
* Authentication error
* Non-idempotent calls without safeguards

---

## 5. Spring Boot Retry — Official Way
Spring provides **Spring Retry**.

### Add Dependency
```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 6. Enable Retry in Spring Boot
```java
@SpringBootApplication
@EnableRetry
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

## 7. Basic Retry Example (Simple)
```java
@Service
public class PaymentService {

    @Retryable(
        value = { RuntimeException.class },
        maxAttempts = 3,
        backoff = @Backoff(delay = 2000)
    )
    public String processPayment() {
        System.out.println("Calling payment service...");
        throw new RuntimeException("Payment service down");
    }
}
```

**What happens internally?**
1. Attempt 1 → fails
2. Wait 2 seconds
3. Attempt 2 → fails
4. Wait 2 seconds
5. Attempt 3 → fails
6. Exception thrown



## 8. Add Recovery Logic (VERY IMPORTANT)
Without recovery = bad design.

```java
@Service
public class PaymentService {

    @Retryable(
        value = RuntimeException.class,
        maxAttempts = 3,
        backoff = @Backoff(delay = 2000)
    )
    public String processPayment() {
        System.out.println("Trying payment...");
        throw new RuntimeException("Service down");
    }

    @Recover
    public String recover(RuntimeException e) {
        System.out.println("Payment failed after retries");
        return "PAYMENT_PENDING";
    }
}
```

**Real-world meaning:**
* Payment failed
* Order kept in **PENDING**
* Background job retries later
* User not blocked

## 9. Retry with REST Call (REAL MICROSERVICE EXAMPLE)
**Order Service → Payment Service**
```java
@Service
public class PaymentClient {

    private final RestTemplate restTemplate = new RestTemplate();

    @Retryable(
        value = { ResourceAccessException.class },
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public String callPaymentService() {
        return restTemplate.getForObject(
            "http://payment-service/pay",
            String.class
        );
    }

    @Recover
    public String recover(ResourceAccessException ex) {
        return "PAYMENT_SERVICE_UNAVAILABLE";
    }
}
```

**Backoff explanation**
1. 1st retry → 1s
2. 2nd retry → 2s
3. 3rd retry → 4s
This avoids **thundering herd problem**.

## 10. Retry + Idempotency (CRITICAL IN PAYMENTS)
**Why?**
Retrying payment can **double charge users**.

**Solution: Idempotency Key**
```java
@PostMapping("/pay")
public ResponseEntity<String> pay(
    @RequestHeader("Idempotency-Key") String key
) {
    if (alreadyProcessed(key)) {
        return ResponseEntity.ok("ALREADY_PROCESSED");
    }
    processPayment();
    saveKey(key);
    return ResponseEntity.ok("SUCCESS");
}
```
Retry + idempotency = **safe retries**

---

## Common Mistakes to Avoid When Implementing Retry in Spring Boot & Microservices
Retry is powerful — but **misusing it can bring down production systems**.
Below are the **most common real-world mistakes** engineers make and **how to avoid them**.

### 1. Retrying Everything Blindly
**Mistake**
Retrying **all** failures, including:
* Validation errors (400)
* Business logic errors
* Authentication failures

> `@Retryable(Exception.class) // too broad`

**Why this is dangerous**
* Wastes resources
* Increases latency
* Never succeeds no matter how many retries

**Correct approach**
Retry **only transient** failures:
* Network timeouts
* 5xx server errors
* Temporary unavailability

> `@Retryable(ResourceAccessException.class)`

### 2. Retry Without Recovery Logic
**Mistake**
Using `@Retryable` without `@Recover`.

**Why this is bad design**
* After retries fail, exception bubbles up
* Order state becomes inconsistent
* User sees 500 error
* No business decision made

**Correct approach**
Always define recovery logic:
```java
@Recover
public String recover(Exception e) {
    return "PAYMENT_PENDING";
}
```
**Retry handles failures.**
**Recovery handles business decisions.**

### 3. Setting High `maxAttempts`
**Mistake**

> `maxAttempts = 10 // ❌`

**Why this kills systems**
* Threads blocked
* CPU spikes
* Cascading failures
* Downstream services collapse

**Best practice**
* 2–3 retries max
* Small, controlled attempts

> `maxAttempts = 3`
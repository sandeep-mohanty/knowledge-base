# The Toxic Side of Spring Boot: 10 Anti-Patterns That Ruin Your Application Architecture

Spring Boot makes it easy to build applications fast. But moving fast without the right design choices can quietly hurt performance, scalability, and long-term maintainability.

In this article, we’ll break down **real-world Spring Boot anti-patterns**, explain **why they’re dangerous**, and show **how to fix them** using clean, performant code.

---

## 1. Anti-Pattern: Business Logic in Controllers

```java
@PostMapping("/users")
public ResponseEntity<?> create(@RequestBody UserDTO dto) {
    if (dto.getAge() < 18) return ResponseEntity.badRequest().build();
    userRepository.save(new User(dto.getName(), dto.getAge()));
    return ResponseEntity.ok().build();
}
```

### What’s Wrong Here?
Putting business logic, validation, and repository calls inside controllers causes several problems:
* Business rules are mixed with HTTP-related logic.
* The controller becomes responsible for too many things (**Violates SRP**).
* Harder to test without starting the web layer.

### Correct Approach: Move Logic to a Service Layer
```java
@Service
public class UserService {
    public User createUser(UserDTO dto) {
        if (dto.getAge() < 18) {
            throw new IllegalArgumentException("Underage");
        }
        return userRepository.save(new User(dto.getName(), dto.getAge()));
    }
}
```
**Controllers should coordinate, not think.** Business rules belong in the service layer.



---

## 2. Anti-Pattern: No Centralized Exception Handling

### What’s the Problem?
Relying on default error pages or manual `try-catch` blocks in every controller method leads to:
* **Inconsistent error responses:** Different formats across the API.
* **Code duplication:** Repeated logic.
* **Security risks:** Leaking stack traces to clients.

### Correct Approach: Centralized Exception Handling
Use `@RestControllerAdvice` to handle exceptions globally.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return new ResponseEntity<>(
            new ErrorResponse(LocalDateTime.now(), 404, ex.getMessage()),
            HttpStatus.NOT_FOUND
        );
    }
}
```

---

## 3. Anti-Pattern: Overusing @Transactional Everywhere

```java
@Transactional
public List<User> getAllUsers() {
    return userRepository.findAll();
}
```

### What’s the Problem?
Using `@Transactional` on simple read operations:
* Opens a transaction unnecessarily.
* Holds database resources longer than required.
* Reduces performance under high load.

### Better Approach
Use a **read-only transaction** for queries that don’t modify data:

```java
@Transactional(readOnly = true)
public List<User> getAllUsers() {
    return userRepository.findAll();
}
```

---

## 4. Anti-Pattern: Unsecured Actuator Endpoints

### Why This Is Dangerous
Exposing `management.endpoints.web.exposure.include=*` in production allows attackers to access:
* `/env`: Exposes secrets and credentials.
* `/heapdump`: Leaks sensitive memory content.

### Correct Approach
Expose **only** what you need and secure it.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info
  endpoint:
    env:
      enabled: false
    heapdump:
      enabled: false
```

---

## 5. Anti-Pattern: Using @Cacheable Without Expiry

### What’s the Problem?
Caching without limits leads to **unbounded memory usage** and stale data, eventually causing `OutOfMemoryError`.

### Correct Approach: Configure Cache Expiration
Use a provider like **Caffeine**.

```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager cacheManager = new CaffeineCacheManager("products");
    cacheManager.setCaffeine(
        Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(1000)
    );
    return cacheManager;
}
```

---

## 6. Anti-Pattern: Blocking Calls in Reactive Code

```java
@GetMapping("/pdf")
public Mono<String> generatePdf() {
    return Mono.just(createHeavyPdf()); // Blocking call!
}
```

### What’s the Problem?
Blocking the Netty event loop reduces throughput and causes thread starvation.

### Correct Approach: Offload Blocking Work
```java
@GetMapping("/pdf")
public Mono<String> generatePdf() {
    return Mono.fromCallable(this::createHeavyPdf)
               .subscribeOn(Schedulers.boundedElastic());
}
```

---

## 7. Anti-Pattern: Scattered @Value Injections

### Why This Is a Problem
* **No Type Safety:** Raw values lead to runtime errors.
* **Hard to Test:** Requires complex setup for mocking.

### Correct Approach: Use @ConfigurationProperties
Define related configuration in a single, validated class.

```java
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    @NotNull
    private String name;
    @Min(100)
    private int timeout;
}
```

---

## 8. Anti-Pattern: Overlapping Scheduled Jobs

```java
@Scheduled(fixedRate = 5000)
public void sendEmails() {
    // Takes 10 seconds -> jobs overlap
}
```

### Correct Approach: Use ShedLock


```java
@Scheduled(fixedRate = 5000)
@SchedulerLock(name = "emailJob", lockAtLeastFor = "PT5S")
public void sendEmails() {
    // Runs only once at a time across clusters
}
```

---

## 9. Anti-Pattern: Relying on Default Tomcat Thread Settings
The default `max-threads=200` is often not optimized for high-traffic workloads.

### Correct Approach: Tune Thread Pool Settings
```properties
server.tomcat.max-threads=500
server.tomcat.accept-count=100
server.connection-timeout=10s
```

---

## 10. Anti-Pattern: No Observability

### Correct Approach: Add Observability
Use **Micrometer + Prometheus + Grafana** for visibility.



```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**Good architecture isn’t about writing more code — it’s about making the right decisions early.**
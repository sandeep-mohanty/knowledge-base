# 🧠 The Senior Java Engineer's Mental Model
### A Complete Tutorial: 10 Years of Production Lessons — Compressed, Expanded & Made Actionable

> *"I've made every one of these mistakes already, in roughly this order."*
> — A Principal Engineer who spotted bugs in 4 seconds

---

## 📋 Table of Contents

1. [Introduction: What is a Production Mental Model?](#introduction)
2. [The Five Root Causes of Production Problems](#five-root-causes)
3. [The Three Questions for Every Piece of State](#three-questions)
4. [The Performance Trilemma](#performance-trilemma)
5. [The Heuristics for Reliability](#reliability-heuristics)
6. [Anti-Patterns: Red Flags in Code Review](#anti-patterns)
7. [The Design Review Toolkit](#design-review)
8. [Skills That Compound Over Time](#compound-skills)
9. [The Synthesis: What Production Engineering Actually Is](#synthesis)
10. [Day One Wisdom](#day-one)

---

## 1. Introduction: What is a Production Mental Model? {#introduction}

A **production mental model** is an internalized map of how systems fail, degrade, and surprise you — built from years of debugging, incident reviews, and code reviews. It's the difference between a junior engineer who hunts a bug for an hour and a senior engineer who spots it in four seconds.

This tutorial distills a decade of Java production experience into teachable patterns, with real code examples, diagrams, and use cases. It's not about memorizing rules — it's about building **reflexes**.

````mermaid
mindmap
  root((Senior Engineer<br/>Mental Model))
    Production Patterns
      5 Root Causes
      State Management
      Performance Trilemma
    Code Quality
      Anti-Patterns
      Design Reviews
      Testing Strategy
    System Reliability
      Timeouts
      Circuit Breakers
      Bulkheads
      Backpressure
    Career Skills
      Thread Dumps
      Heap Dumps
      GC Logs
      Runbooks
      Postmortems
````

### Why Build a Mental Model?

Most production problems aren't new. They're variations of known failure patterns. A well-trained mental model lets you:

- Map symptoms to root causes in minutes, not hours
- Ask the right questions during design reviews
- Spot anti-patterns in code review before they reach production
- Build systems that are resilient by design, not by accident

---

## 2. The Five Root Causes of Production Problems {#five-root-causes}

> After enough incident reviews, production problems stop being surprising. They fall into the same five categories, again and again.

````mermaid
flowchart TD
    INCIDENT([🚨 Production Incident])
    INCIDENT --> C1
    INCIDENT --> C2
    INCIDENT --> C3
    INCIDENT --> C4
    INCIDENT --> C5

    C1["❶ Something Unbounded\n━━━━━━━━━━━━━━━━\nCaches, queues, retries,\nlogs grew past capacity"]
    C2["❷ Unanticipated Failure Mode\n━━━━━━━━━━━━━━━━\nDownstream timeout,\nDB failover, broker down"]
    C3["❸ False Assumption\n━━━━━━━━━━━━━━━━\nStale config, wrong capacity\nestimates, changed deps"]
    C4["❹ Unintended Side Effect\n━━━━━━━━━━━━━━━━\nRefactor changed behavior,\nconfig touched more than logging"]
    C5["❺ Violated Invariant\n━━━━━━━━━━━━━━━━\nUnexpected data shape,\nillegal state transition"]

    C1 --> S1["Strategy: Impose bounds;\nask 'what bounds this?'"]
    C2 --> S2["Strategy: Chaos testing;\nlist failure modes explicitly"]
    C3 --> S3["Strategy: Document assumptions;\nvalidate at startup"]
    C4 --> S4["Strategy: Better test coverage;\nfeature flags for risky changes"]
    C5 --> S5["Strategy: Validate invariants;\nuse types and assertions"]

    style INCIDENT fill:#e74c3c,color:#fff
    style C1 fill:#e67e22,color:#fff
    style C2 fill:#e67e22,color:#fff
    style C3 fill:#e67e22,color:#fff
    style C4 fill:#e67e22,color:#fff
    style C5 fill:#e67e22,color:#fff
    style S1 fill:#27ae60,color:#fff
    style S2 fill:#27ae60,color:#fff
    style S3 fill:#27ae60,color:#fff
    style S4 fill:#27ae60,color:#fff
    style S5 fill:#27ae60,color:#fff
````

---

### Cause 1: Something Unbounded

**The Pattern:** Any component whose growth is controlled only by hope will eventually exhaust resources.

**Real-World Examples:**

````java
// ❌ DANGEROUS: Unbounded static cache — a memory leak in slow motion
public class UserCache {
    private static final Map<Long, User> cache = new HashMap<>(); // No limit!

    public static void put(Long id, User user) {
        cache.put(id, user); // Grows forever
    }
}

// ✅ SAFE: Bounded cache using LinkedHashMap as LRU
public class UserCache {
    private static final int MAX_SIZE = 10_000;
    private static final Map<Long, User> cache = new LinkedHashMap<>(MAX_SIZE, 0.75f, true) {
        @Override
        protected boolean removeEldestEntry(Map.Entry<Long, User> eldest) {
            return size() > MAX_SIZE; // Evict oldest when full
        }
    };
}

// ✅ EVEN BETTER: Caffeine — production-grade bounded cache
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;

public class UserCache {
    private final Cache<Long, User> cache = Caffeine.newBuilder()
        .maximumSize(10_000)           // Bound by entry count
        .expireAfterWrite(5, MINUTES)  // Evict stale entries
        .recordStats()                 // Expose cache hit rates
        .build();
}
````

**Unbounded Queue Example:**

````java
// ❌ DANGEROUS: Unbounded queue — explodes under slow consumers
ExecutorService executor = Executors.newFixedThreadPool(10);
// This uses LinkedBlockingQueue internally — grows to Integer.MAX_VALUE

// ✅ SAFE: Bounded queue with rejection policy
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    10, 20,                           // core/max threads
    60, TimeUnit.SECONDS,             // keepAlive
    new ArrayBlockingQueue<>(1000),   // bounded queue
    new ThreadPoolExecutor.CallerRunsPolicy() // backpressure on caller
);
````

**Use Case:** An e-commerce platform saw OOM errors at peak sale hours. A static `Map<String, List<Order>>` for "recently viewed" was growing unboundedly as order volume spiked. Adding a Caffeine cache with a 10,000-entry max eliminated the issue.

---

### Cause 2: Unanticipated Failure Modes

**The Pattern:** Code is tested for happy paths and obvious sad paths — but not for the exact way production surprises you.

````mermaid
flowchart LR
    A[Your Service] -->|HTTP call| B[Downstream Service]
    B -->|Normal| C[200 OK ✅]
    B -->|Anticipated| D[500 Error ⚠️]
    B -->|Unanticipated| E[Timeout after 30s 💥]
    B -->|Unanticipated| F[Partial Response 💥]
    B -->|Unanticipated| G[DNS failure 💥]
    B -->|Unanticipated| H[SSL cert expired 💥]

    style E fill:#e74c3c,color:#fff
    style F fill:#e74c3c,color:#fff
    style G fill:#e74c3c,color:#fff
    style H fill:#e74c3c,color:#fff
````

**Failure Mode Checklist (use in design reviews):**

````
For every downstream dependency, explicitly answer:
□ What happens if it times out for the first time in production?
□ What happens if it returns a 5xx error?
□ What happens if it returns a partial/malformed response?
□ What happens during its deployment window?
□ What happens if the network between us becomes lossy?
□ What happens during a DB failover (10–30 second gap)?
````

**Example: Resilience4j for circuit breaking + timeout:**

````java
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)           // Open after 50% failure
    .waitDurationInOpenState(Duration.ofSeconds(30))
    .slidingWindowSize(10)
    .build();

CircuitBreaker breaker = CircuitBreaker.of("paymentService", config);
TimeLimiter timeLimiter = TimeLimiter.of(Duration.ofSeconds(2));

Supplier<CompletableFuture<PaymentResult>> futureSupplier = () ->
    CompletableFuture.supplyAsync(() -> paymentClient.charge(request));

Callable<PaymentResult> decorated =
    Decorators.ofSupplier(futureSupplier)
        .withTimeLimiter(timeLimiter, scheduledExecutorService)
        .withCircuitBreaker(breaker)
        .decorate();
````

---

### Cause 3: An Assumption Became False

**The Pattern:** Configuration, capacity estimates, and dependency behaviors change over time. Code has implicit assumptions baked in.

````java
// ❌ IMPLICIT ASSUMPTION: "We'll never have more than 1000 users per request"
public List<User> fetchUsers(List<Long> ids) {
    return jdbcTemplate.query(
        "SELECT * FROM users WHERE id IN (" + ids.stream()...join(",") + ")",
        userMapper
    );
    // What happens when ids.size() = 100,000? Query planner breaks. DB connection times out.
}

// ✅ EXPLICIT AND BOUNDED: Validate assumptions at entry points
public List<User> fetchUsers(List<Long> ids) {
    if (ids.size() > 1000) {
        throw new IllegalArgumentException(
            "Batch size " + ids.size() + " exceeds maximum of 1000"
        );
    }
    // Or: partition and batch
    return Lists.partition(ids, 500).stream()
        .flatMap(batch -> fetchBatch(batch).stream())
        .collect(toList());
}
````

**Startup Validation Pattern:**

````java
@Component
public class AssumptionValidator implements ApplicationListener<ApplicationReadyEvent> {
    @Value("${max.batch.size}")
    private int maxBatchSize;

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        Preconditions.checkState(maxBatchSize > 0 && maxBatchSize <= 10_000,
            "max.batch.size must be between 1 and 10000, got: %s", maxBatchSize);
        // Fail fast on startup rather than silently misbehave later
    }
}
````

---

### Cause 4: Unintended Consequences of a Change

**The Pattern:** Refactors, deploys, and config changes that "shouldn't affect behavior" do.

````mermaid
sequenceDiagram
    participant Dev as Developer
    participant PR as Pull Request
    participant CI as CI Pipeline
    participant Prod as Production

    Dev->>PR: "Safe refactor - just renaming"
    PR->>CI: Tests pass ✅
    CI->>Prod: Deploy
    Prod-->>Dev: 🚨 PagerDuty: Serialization broken!
    Note over Dev,Prod: Renamed field changed JSON output.<br/>Consumers expected old field name.
````

**Defense Strategies:**

````java
// 1. Use @JsonProperty to decouple field names from wire format
public class UserDto {
    @JsonProperty("user_id")       // Wire name is stable even if Java field changes
    private Long userId;

    @JsonProperty("created_at")
    private Instant createdAt;
}

// 2. Use consumer-driven contract tests (Pact)
// provider-test/UserApiContractTest.java
@Provider("user-service")
@PactFolder("pacts")
class UserApiContractTest {
    @TestTemplate
    void verifyPact(PactVerificationContext context) {
        context.verifyInteraction(); // Fails if your change breaks a consumer contract
    }
}

// 3. Feature flags for risky changes
if (featureFlags.isEnabled("new-serialization-format", userId)) {
    return newSerializer.serialize(user);
} else {
    return legacySerializer.serialize(user);
}
````

---

### Cause 5: An Invariant Was Violated

**The Pattern:** Systems rely on implicit invariants that production eventually breaks.

````java
// ❌ IMPLICIT INVARIANT: "Order total is always positive"
public class Order {
    private BigDecimal total; // What if it's negative? null?
}

// ✅ EXPLICIT INVARIANT: Enforce at construction time
public class Order {
    private final BigDecimal total;

    public Order(BigDecimal total) {
        Objects.requireNonNull(total, "total must not be null");
        if (total.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Order total cannot be negative: " + total);
        }
        this.total = total;
    }

    // Use sealed classes + records for state machines
}

// State machine invariant — only valid transitions allowed
public sealed interface OrderState
    permits OrderState.Draft, OrderState.Submitted, OrderState.Fulfilled, OrderState.Cancelled {

    record Draft() implements OrderState {}
    record Submitted(Instant submittedAt) implements OrderState {}
    record Fulfilled(Instant fulfilledAt) implements OrderState {}
    record Cancelled(String reason) implements OrderState {}
}

public class OrderService {
    public Order submit(Order order) {
        if (!(order.state() instanceof OrderState.Draft)) {
            throw new IllegalStateException(
                "Can only submit a Draft order, got: " + order.state()
            );
        }
        return order.withState(new OrderState.Submitted(Instant.now()));
    }
}
````

---

## 3. The Three Questions for Every Piece of State {#three-questions}

> This single heuristic has prevented more bugs than any other. Ask it every time you introduce state.

````mermaid
flowchart TD
    STATE([📦 New State Introduced\ne.g., Field, Cache, Pool,\nSession, Queue])

    STATE --> Q1

    Q1{"❶ What bounds\nthis state?"}
    Q1 -->|Bounded ✅| Q2
    Q1 -->|Unbounded ❌| FIX1["Add size limit,\nexpiry policy,\nor max depth"]
    FIX1 --> Q2

    Q2{"❷ Who owns\nthe lifecycle?"}
    Q2 -->|Clear owner ✅| Q3
    Q2 -->|Unclear ❌| FIX2["Assign owner,\ndefine cleanup path,\nuse try-with-resources"]
    FIX2 --> Q3

    Q3{"❸ Where does it\nlive across instances?"}
    Q3 -->|Single pod only\nand that's OK ✅| SAFE
    Q3 -->|Needs cross-pod\ncoordination ❌| FIX3["Move to Redis/DB,\nuse distributed lock,\nor sticky sessions"]
    FIX3 --> SAFE

    SAFE(["✅ State is safe\nto ship"])

    style STATE fill:#3498db,color:#fff
    style Q1 fill:#8e44ad,color:#fff
    style Q2 fill:#8e44ad,color:#fff
    style Q3 fill:#8e44ad,color:#fff
    style SAFE fill:#27ae60,color:#fff
    style FIX1 fill:#e74c3c,color:#fff
    style FIX2 fill:#e74c3c,color:#fff
    style FIX3 fill:#e74c3c,color:#fff
````

### Question 1: What Bounds This State?

````java
// ❌ Q1 FAIL: Session store with no bound
Map<String, UserSession> sessions = new ConcurrentHashMap<>();

// ✅ Q1 PASS: Bounded with eviction
Cache<String, UserSession> sessions = Caffeine.newBuilder()
    .maximumSize(100_000)
    .expireAfterAccess(30, MINUTES)
    .build();
````

### Question 2: Who Owns the Lifecycle?

````java
// ❌ Q2 FAIL: ThreadLocal leak — set but never cleaned
public class RequestContext {
    private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

    public static void setUser(User user) {
        currentUser.set(user); // Thread from pool keeps this forever!
    }
}

// ✅ Q2 PASS: Explicit cleanup in finally block
public class RequestFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        try {
            RequestContext.setUser(resolveUser(req));
            chain.doFilter(req, res);
        } finally {
            RequestContext.clear(); // Always cleans up, even on exception
        }
    }
}

// ✅ EVEN BETTER: AutoCloseable scope
public class UserScope implements AutoCloseable {
    private static final ThreadLocal<User> current = new ThreadLocal<>();

    public UserScope(User user) {
        current.set(user);
    }

    public static User get() { return current.get(); }

    @Override
    public void close() { current.remove(); } // try-with-resources ensures cleanup
}

// Usage:
try (var scope = new UserScope(resolvedUser)) {
    processRequest(request);
} // Automatically cleaned up
````

### Question 3: Where Does It Live Across Instances?

````java
// ❌ Q3 FAIL: Rate limiter in local memory — each pod allows the full rate
@Service
public class RateLimiter {
    private final Map<String, Long> requestCounts = new ConcurrentHashMap<>();
    // 3 pods × 100 req/s limit = effectively 300 req/s allowed. Oops.
}

// ✅ Q3 PASS: Rate limiter backed by Redis — shared across all pods
@Service
public class DistributedRateLimiter {
    private final RedisTemplate<String, Long> redis;

    public boolean tryAcquire(String clientId, int limitPerSecond) {
        String key = "rate:" + clientId + ":" + (System.currentTimeMillis() / 1000);
        Long count = redis.opsForValue().increment(key);
        if (count == 1) redis.expire(key, 2, SECONDS); // TTL slightly over 1s
        return count <= limitPerSecond;
    }
}
````

**Use Case Matrix:**

| State Type | Multi-Instance Safe? | Solution |
|---|---|---|
| Immutable config | ✅ Yes | Local field |
| Read-only cache | ✅ Yes (with staleness tolerance) | Local Caffeine cache |
| Rate limiting | ❌ No | Redis counter |
| Session data | ❌ No | Redis / sticky sessions |
| Deduplication | ❌ No | Redis SET with TTL |
| Distributed lock | ❌ No | Redisson / ZooKeeper |

---

## 4. The Performance Trilemma {#performance-trilemma}

> You cannot optimize for latency, throughput, and cost simultaneously. The best teams pick deliberately.

````mermaid
graph TD
    subgraph TRILEMMA["⚖️ The Performance Trilemma"]
        L["🎯 Low Latency\n━━━━━━━━━━\nP99 / P99.9 matters\nZGC / Shenandoah\nGenerous budgets\nFail-fast timeouts\n━━━━━━━━━━\nCost: 💰💰💰"]
        T["📈 High Throughput\n━━━━━━━━━━\nTotal work / resource\nG1 / Parallel GC\nBatch processing\nTolerate tail latency\n━━━━━━━━━━\nCost: Occasional slow P99"]
        C["💲 Low Cost\n━━━━━━━━━━\nRun close to capacity\nAggressive sizing\nMinimize allocation\nAccept headroom limits\n━━━━━━━━━━\nCost: Risk during spikes"]

        L <-->|"Trade-off: Infrastructure\nvs Response Time"| T
        T <-->|"Trade-off: Throughput\nvs Budget"| C
        L <-->|"Trade-off: Speed\nvs Spend"| C
    end

    style L fill:#3498db,color:#fff
    style T fill:#e67e22,color:#fff
    style C fill:#27ae60,color:#fff
````

### Choosing the Right GC for Your Trilemma Position

````java
// JVM flags for Latency-optimized (ZGC — sub-millisecond pauses)
// java -XX:+UseZGC -Xmx16g -XX:MaxGCPauseMillis=5

// JVM flags for Throughput-optimized (G1GC)
// java -XX:+UseG1GC -Xmx8g -XX:G1HeapRegionSize=16m

// JVM flags for Cost-optimized (SerialGC on small containers)
// java -XX:+UseSerialGC -Xmx512m -XX:+TieredCompilation

// MEASURE: How to verify which dimension you're hitting
// Add to your app:
@Scheduled(fixedRate = 60_000)
public void logGcStats() {
    ManagementFactory.getGarbageCollectorMXBeans().forEach(gc -> {
        log.info("GC [{}]: count={}, time={}ms",
            gc.getName(), gc.getCollectionCount(), gc.getCollectionTime());
    });
}
````

### Service Classification Examples

| Service Type | Primary Dimension | Why |
|---|---|---|
| Payment authorization | **Latency** | User experience; SLA penalties |
| Search indexing pipeline | **Throughput** | Process millions of docs/hour |
| Batch reporting job | **Cost** | Runs nightly; SLA is loose |
| User-facing API | **Latency + Cost balance** | SLA required; budget constrained |
| ML inference (real-time) | **Latency** | Sub-100ms required |
| ML inference (batch) | **Throughput** | Maximize GPUs / hour |

---

## 5. The Heuristics for Reliability {#reliability-heuristics}

### The Full Reliability Stack

````mermaid
flowchart TB
    subgraph OUTER["🛡️ Reliability Defense in Depth"]
        T["⏱️ Timeouts\nEvery call, propagating downward\nFail fast rather than wait forever"]
        R["🔁 Retries + Idempotency\nJittered backoff prevents storms\nIdempotency makes retries safe"]
        CB["⚡ Circuit Breakers\nTrip on measured failure rate\nNot too sensitive, not too slow"]
        BH["🚢 Bulkheads\nSeparate thread pools per dep\nIsolate failures to compartments"]
        BP["🌊 Backpressure\nReject > Buffer when overloaded\nMakes overload visible"]
        GD["🔄 Graceful Degradation\nFallback paths per dependency\nProduct decision, not code decision"]
    end

    T --> R --> CB --> BH --> BP --> GD

    style T fill:#2980b9,color:#fff
    style R fill:#8e44ad,color:#fff
    style CB fill:#c0392b,color:#fff
    style BH fill:#27ae60,color:#fff
    style BP fill:#d35400,color:#fff
    style GD fill:#16a085,color:#fff
````

---

### Heuristic 1: Timeouts Everywhere, Propagating Downward

````java
// Timeout budget propagation: API → Service → DB
// Each layer's timeout must be shorter than the layer above.

// API Gateway: 10s timeout
// Service A: 8s timeout to Service B (2s for its own overhead)
// Service B: 5s timeout to the DB (3s for its own overhead)

// HTTP client with timeout:
HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(2))
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(url))
    .timeout(Duration.ofSeconds(5)) // Total request timeout
    .build();

// Database timeout:
@Query(value = "SELECT * FROM orders WHERE user_id = :userId",
       nativeQuery = true,
       timeout = 3) // 3 seconds — shorter than service-level timeout
List<Order> findByUserId(@Param("userId") Long userId);

// Don't forget: JDBC-level timeout too
DataSource dataSource = DataSourceBuilder.create()
    .build();
((HikariDataSource) dataSource).setConnectionTimeout(3000);  // ms
((HikariDataSource) dataSource).setQueryTimeout(3);          // seconds
````

---

### Heuristic 2: Retries Are Dangerous Without Idempotency

````mermaid
sequenceDiagram
    participant Client
    participant Service
    participant DB

    Client->>Service: POST /charge $100
    Service->>DB: INSERT payment...
    DB-->>Service: 💥 Timeout!
    Service-->>Client: 503 Error

    Note over Client: Should I retry?

    alt Without Idempotency
        Client->>Service: POST /charge $100 (retry)
        Service->>DB: INSERT payment...
        DB-->>Service: ✅ OK
        Note over Client,DB: 😱 User charged TWICE!
    end

    alt With Idempotency Key
        Client->>Service: POST /charge (idempotency-key: abc-123)
        Service->>DB: INSERT IF NOT EXISTS (key: abc-123)
        DB-->>Service: ✅ OK (or "already exists")
        Service-->>Client: 200 OK (safe to retry)
    end
````

````java
// Idempotent payment service
@PostMapping("/charge")
public ResponseEntity<PaymentResult> charge(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @RequestBody ChargeRequest request) {

    // Check if we've seen this key before
    Optional<PaymentResult> existing = idempotencyStore.get(idempotencyKey);
    if (existing.isPresent()) {
        return ResponseEntity.ok(existing.get()); // Return cached result
    }

    PaymentResult result = paymentGateway.charge(request);

    // Store result with key (expires after 24h)
    idempotencyStore.set(idempotencyKey, result, Duration.ofHours(24));

    return ResponseEntity.ok(result);
}

// Jittered exponential backoff
public <T> T executeWithRetry(Supplier<T> operation, int maxAttempts) {
    int attempt = 0;
    while (true) {
        try {
            return operation.get();
        } catch (RetryableException e) {
            if (++attempt >= maxAttempts) throw e;
            long delay = (long) (Math.pow(2, attempt) * 100) // base: 100ms
                         + ThreadLocalRandom.current().nextLong(0, 100); // jitter
            Thread.sleep(Math.min(delay, 30_000)); // cap at 30s
        }
    }
}
````

---

### Heuristic 3: Circuit Breakers — The Correct Configuration

````mermaid
stateDiagram-v2
    [*] --> Closed : Service starts
    Closed --> Open : Failure rate > threshold\n(over sliding window)
    Open --> HalfOpen : Wait duration elapsed\n(e.g., 30 seconds)
    HalfOpen --> Closed : Test requests succeed
    HalfOpen --> Open : Test requests fail

    state Closed {
        [*] --> Tracking
        Tracking : Tracking success/failure rate
        note right of Tracking: All requests pass through\nMonitoring failure rate\ne.g., 5/10 = 50% → OPEN
    }

    state Open {
        [*] --> Rejecting
        Rejecting : Instantly rejecting all calls
        note right of Rejecting: Fast-fail: return fallback\nor throw CircuitBreakerOpenException
    }

    state HalfOpen {
        [*] --> Probing
        Probing : Allowing limited test traffic
        note right of Probing: e.g., 5 test requests\nDecide to close or re-open
    }
````

````java
// Correctly configured circuit breaker
CircuitBreakerConfig cbConfig = CircuitBreakerConfig.custom()
    // Only open if 50%+ of calls fail — but only after 20+ calls
    // (avoid tripping on 1 failure out of 2)
    .failureRateThreshold(50f)
    .minimumNumberOfCalls(20)
    .slidingWindowSize(50)

    // Stay open for 30s, then probe
    .waitDurationInOpenState(Duration.ofSeconds(30))
    .permittedNumberOfCallsInHalfOpenState(5)

    // Don't count timeouts separately from failures
    .slowCallRateThreshold(80f)
    .slowCallDurationThreshold(Duration.ofSeconds(3))

    // Don't trip on business exceptions (only infrastructure errors)
    .recordExceptions(IOException.class, TimeoutException.class)
    .ignoreExceptions(BusinessValidationException.class)
    .build();
````

---

### Heuristic 4: Bulkheads — Isolate Failures

````mermaid
flowchart LR
    subgraph YOUR_SERVICE["Your Service"]
        TP1["Thread Pool\nfor Auth Service\n⚙️ 10 threads"]
        TP2["Thread Pool\nfor Payment Service\n⚙️ 10 threads"]
        TP3["Thread Pool\nfor Notification Service\n⚙️ 5 threads"]
    end

    TP1 -->|calls| AUTH["🔐 Auth Service"]
    TP2 -->|calls| PAY["💳 Payment Service\n🐢 Slow!"]
    TP3 -->|calls| NOTIF["📧 Notification Service"]

    PAY -.->|"All 10 threads\nblocked waiting"| TP2
    note1["❌ Without Bulkhead:\nPayment slowness\nblocks ALL threads"]
    note2["✅ With Bulkhead:\nPayment slowness\nonly blocks its pool.\nAuth & Notifications\nstill work!"]

    style PAY fill:#e74c3c,color:#fff
    style TP2 fill:#e74c3c,color:#fff
````

````java
// Bulkhead pattern with separate thread pools per dependency
@Configuration
public class HttpClientConfig {

    @Bean("authServiceExecutor")
    public ExecutorService authServiceExecutor() {
        return new ThreadPoolExecutor(
            5, 10, 60, SECONDS,
            new ArrayBlockingQueue<>(50),
            new ThreadFactoryBuilder().setNameFormat("auth-pool-%d").build(),
            new ThreadPoolExecutor.AbortPolicy() // Reject if full, don't block caller
        );
    }

    @Bean("paymentServiceExecutor")
    public ExecutorService paymentServiceExecutor() {
        return new ThreadPoolExecutor(
            10, 20, 60, SECONDS,
            new ArrayBlockingQueue<>(100),
            new ThreadFactoryBuilder().setNameFormat("payment-pool-%d").build(),
            new ThreadPoolExecutor.AbortPolicy()
        );
    }
}

@Service
public class AuthService {
    @Autowired @Qualifier("authServiceExecutor")
    private ExecutorService executor;

    public CompletableFuture<AuthResult> verifyToken(String token) {
        return CompletableFuture.supplyAsync(
            () -> authClient.verify(token), // Uses isolated pool
            executor
        );
    }
}
````

---

### Heuristic 5: Backpressure Over Buffering

````java
// ❌ DANGEROUS: Unbounded buffer — queue hides overload until OOM
BlockingQueue<Request> queue = new LinkedBlockingQueue<>(); // No limit!

// What happens: queue fills with 500k requests, GC thrashes, OOM, service dies.
// Worse: users think requests are queued and will eventually succeed. They won't.

// ✅ SAFE: Bounded queue + rejection
BlockingQueue<Request> queue = new ArrayBlockingQueue<>(1000);

// Rejection makes overload visible (returns 429 to caller immediately)
// Caller can retry later or route to another instance
// System doesn't degrade silently

// ✅ REACTIVE: Use Project Reactor's backpressure operators
Flux.fromIterable(hugeOrderList)
    .onBackpressureBuffer(1000,        // Max 1000 buffered
        dropped -> log.warn("Dropped order: {}", dropped),
        BufferOverflowStrategy.DROP_LATEST)
    .flatMap(order -> processOrder(order), 10) // 10 concurrent
    .subscribe();
````

---

## 6. Anti-Patterns: Red Flags in Code Review {#anti-patterns}

````mermaid
flowchart TD
    CR(["👁️ Code Review"])

    CR --> AP1
    CR --> AP2
    CR --> AP3
    CR --> AP4
    CR --> AP5
    CR --> AP6
    CR --> AP7
    CR --> AP8

    AP1["🚨 private static final Map\nwritten at runtime"]
    AP2["🚨 Empty catch blocks\n or swallowed exceptions"]
    AP3["🚨 ThreadLocal.set\nwithout .remove in finally"]
    AP4["🚨 Sync HTTP/DB calls\ninside loops"]
    AP5["🚨 Shared mutable state\nwithout synchronization"]
    AP6["🚨 Config baked\ninto source code"]
    AP7["🚨 Tests with no\nmeaningful assertions"]
    AP8["🚨 Magic numbers\nand strings"]

    AP1 --> FIX1["Use Caffeine\nwith maximumSize + TTL"]
    AP2 --> FIX2["Log + rethrow or\nwrap in domain exception"]
    AP3 --> FIX3["try-with-resources\nor finally {tl.remove()}"]
    AP4 --> FIX4["Batch requests;\nuse IN clause or\nasync parallel calls"]
    AP5 --> FIX5["Use AtomicXxx,\nCopyOnWriteXxx,\nor synchronized blocks"]
    AP6 --> FIX6["@Value / @ConfigurationProperties\n+ environment variables"]
    AP7 --> FIX7["AssertJ: assertThat(...)\n.isEqualTo(expected)"]
    AP8 --> FIX8["Extract to named constants\nor enums"]

    style CR fill:#2c3e50,color:#fff
    style AP1 fill:#e74c3c,color:#fff
    style AP2 fill:#e74c3c,color:#fff
    style AP3 fill:#e74c3c,color:#fff
    style AP4 fill:#e74c3c,color:#fff
    style AP5 fill:#e74c3c,color:#fff
    style AP6 fill:#e74c3c,color:#fff
    style AP7 fill:#e74c3c,color:#fff
    style AP8 fill:#e74c3c,color:#fff
    style FIX1 fill:#27ae60,color:#fff
    style FIX2 fill:#27ae60,color:#fff
    style FIX3 fill:#27ae60,color:#fff
    style FIX4 fill:#27ae60,color:#fff
    style FIX5 fill:#27ae60,color:#fff
    style FIX6 fill:#27ae60,color:#fff
    style FIX7 fill:#27ae60,color:#fff
    style FIX8 fill:#27ae60,color:#fff
````

### Anti-Pattern Deep Dives with Fix Examples

**Anti-Pattern 4: N+1 Query in a Loop**

````java
// ❌ N+1 QUERIES: 1 query for orders + N queries for users
List<Order> orders = orderRepo.findAll(); // 1 query
for (Order order : orders) {
    User user = userRepo.findById(order.getUserId()); // N queries!
    sendEmail(user, order);
}
// 1000 orders = 1001 DB roundtrips

// ✅ BATCH LOAD: 2 queries total
List<Order> orders = orderRepo.findAll();
Set<Long> userIds = orders.stream().map(Order::getUserId).collect(toSet());
Map<Long, User> userMap = userRepo.findAllById(userIds).stream()
    .collect(toMap(User::getId, identity()));
for (Order order : orders) {
    User user = userMap.get(order.getUserId()); // Map lookup, no DB call
    sendEmail(user, order);
}

// ✅ JPA: Use JOIN FETCH in JPQL
@Query("SELECT o FROM Order o JOIN FETCH o.user WHERE o.status = :status")
List<Order> findWithUsersByStatus(@Param("status") Status status);
````

**Anti-Pattern 5: Shared Mutable State**

````java
// ❌ RACE CONDITION: Two threads can read-increment-write concurrently
public class RequestCounter {
    private long count = 0; // NOT thread-safe

    public void increment() { count++; } // Read, increment, write — 3 ops!
    public long get() { return count; }
}

// ✅ ATOMIC: Single CAS operation
public class RequestCounter {
    private final AtomicLong count = new AtomicLong(0);

    public void increment() { count.incrementAndGet(); }
    public long get() { return count.get(); }
}

// ✅ FOR MAPS: ConcurrentHashMap with atomic operations
ConcurrentHashMap<String, AtomicLong> counters = new ConcurrentHashMap<>();

public void increment(String key) {
    counters.computeIfAbsent(key, k -> new AtomicLong(0))
            .incrementAndGet();
}
````

**Anti-Pattern 7: Tests Without Assertions**

````java
// ❌ USELESS TEST: Doesn't fail if behavior changes
@Test
void testCalculateDiscount() {
    DiscountService service = new DiscountService();
    service.calculate(order); // What if it returns 0? What if it throws? We don't know!
}

// ✅ MEANINGFUL TEST: Using AssertJ for readable, specific assertions
@Test
void calculateDiscount_forGoldCustomer_appliesCorrectPercentage() {
    // Arrange
    Customer customer = Customer.builder().tier(GOLD).totalSpend(BigDecimal.valueOf(5000)).build();
    Order order = Order.builder().subtotal(BigDecimal.valueOf(100)).customer(customer).build();

    // Act
    BigDecimal discount = discountService.calculate(order);

    // Assert — specific, tells you exactly what broke
    assertThat(discount)
        .as("Gold customers should get 15% discount on orders over $100")
        .isEqualByComparingTo(BigDecimal.valueOf(15.00));
}

// ✅ EDGE CASE TESTS MATTER TOO
@ParameterizedTest
@CsvSource({
    "BRONZE, 50.00, 0.00",   // No discount below threshold
    "SILVER, 100.00, 10.00", // 10% for silver
    "GOLD,   100.00, 15.00"  // 15% for gold
})
void calculateDiscount_respectsTierRules(Tier tier, String subtotal, String expectedDiscount) {
    // ...
}
````

---

## 7. The Design Review Toolkit {#design-review}

````mermaid
flowchart TD
    DR(["📋 Design Review"])

    DR --> Q1
    DR --> Q2
    DR --> Q3
    DR --> Q4
    DR --> Q5
    DR --> Q6
    DR --> Q7
    DR --> Q8

    Q1["📏 Scope\nWhat are we deliberately\nNOT doing?"]
    Q2["🔗 Dependencies\nWhat happens when each\ndownstream is unavailable?"]
    Q3["📊 Data\nWhat is the max size of\nevery collection/queue/cache?"]
    Q4["♻️ Lifecycle\nDeployment? Rollback?\nPod restart? Node failure?"]
    Q5["👁️ Observability\nWhat signals prove\nit's working or failing?"]
    Q6["🔄 Evolution\nHow do we add fields\nwithout breaking consumers?"]
    Q7["🚨 Recovery\nWhen it fails in production,\nwhat does the runbook say?"]
    Q8["🧪 Testing\nWhat bugs will the\ntest suite NOT catch?"]

    Q1 --> R1["Clear exclusions prevent\nscope creep and overengineering"]
    Q2 --> R2["Explicit failure modes\nbecome explicit fallbacks"]
    Q3 --> R3["'Unbounded' is a\ndesign issue, not an answer"]
    Q4 --> R4["Transition states hide\nmost incidents"]
    Q5 --> R5["Observability without\npurpose is noise"]
    Q6 --> R6["Consumer-driven contracts;\nversioning strategy"]
    Q7 --> R7["Runbook review should\nhappen in design, not incident"]
    Q8 --> R8["The honest answer\npredicts your incident profile"]

    style DR fill:#2c3e50,color:#fff
    style Q1 fill:#8e44ad,color:#fff
    style Q2 fill:#8e44ad,color:#fff
    style Q3 fill:#8e44ad,color:#fff
    style Q4 fill:#8e44ad,color:#fff
    style Q5 fill:#8e44ad,color:#fff
    style Q6 fill:#8e44ad,color:#fff
    style Q7 fill:#8e44ad,color:#fff
    style Q8 fill:#8e44ad,color:#fff
````

### Design Review Question: Observability Template

````java
// Observability is a design property. Build it in from day one.

@Component
public class OrderService {
    private final Counter ordersProcessed;
    private final Counter ordersFailed;
    private final Timer orderProcessingTime;
    private final Gauge pendingOrders;

    public OrderService(MeterRegistry registry) {
        this.ordersProcessed = Counter.builder("orders.processed")
            .tag("service", "order-service")
            .description("Total successfully processed orders")
            .register(registry);

        this.ordersFailed = Counter.builder("orders.failed")
            .tag("service", "order-service")
            .description("Total failed order attempts")
            .register(registry);

        this.orderProcessingTime = Timer.builder("orders.processing.time")
            .description("Order processing latency")
            .publishPercentiles(0.5, 0.95, 0.99) // p50, p95, p99
            .register(registry);

        this.pendingOrders = Gauge.builder("orders.pending",
            orderQueue, Queue::size)
            .description("Current pending order queue depth")
            .register(registry);
    }

    public Order processOrder(OrderRequest request) {
        return orderProcessingTime.record(() -> {
            try {
                Order result = doProcess(request);
                ordersProcessed.increment();
                return result;
            } catch (Exception e) {
                ordersFailed.increment(Tags.of("error", e.getClass().getSimpleName()));
                throw e;
            }
        });
    }
}
````

---

## 8. Skills That Compound Over Time {#compound-skills}

````mermaid
timeline
    title Engineering Skills Progression
    Year 1-2 : Write working code
             : Learn basic debugging
             : Understand unit tests
    Year 2-4 : Read thread dumps
             : Understand GC basics
             : Write useful runbooks
    Year 4-7 : Read heap dumps
             : Analyze GC logs deeply
             : Produce real postmortems
             : Design for failure
    Year 7-10 : Spot patterns in 4 seconds
              : Build team mental models
              : Teach and multiply
              : System-level thinking
````

### Reading Thread Dumps — A Practical Guide

````
# Generate a thread dump:
kill -3 <pid>          # Linux (prints to stdout/stderr)
jstack <pid>           # JDK tool (better formatted)
jcmd <pid> Thread.print  # JDK tool (most complete)

# What to look for:
# 1. BLOCKED threads — contention on a monitor (synchronized block)
# 2. WAITING threads — calling Object.wait() or LockSupport.park()
# 3. TIMED_WAITING — Thread.sleep() or waiting with timeout

"http-nio-8080-exec-1" #42 daemon prio=5
  java.lang.Thread.State: BLOCKED (on object monitor)
    at com.example.OrderService.processOrder(OrderService.java:87)
    - waiting to lock <0x00000007b8020a50> (a com.example.OrderService)
    at ...

# This tells you: exec-1 is blocked because exec-2 holds the lock on OrderService.
# Likely a synchronized method that's taking too long, blocking other request threads.
````

### Reading GC Logs — Key Signals

````bash
# Enable detailed GC logging (JVM 11+):
# -Xlog:gc*:file=/var/log/app/gc.log:time,uptime,level,tags:filecount=5,filesize=20m

# Key patterns to look for:

# ✅ HEALTHY: Short, infrequent GC pauses
[0.512s][info][gc] GC(3) Pause Young (Normal) 45M->22M(256M) 8.234ms

# ⚠️ WARNING: Frequent GCs (high allocation rate)
[0.100s][info][gc] GC(1) Pause Young ...
[0.200s][info][gc] GC(2) Pause Young ...   # GC every 100ms = allocation problem
[0.301s][info][gc] GC(3) Pause Young ...

# 🚨 DANGER: Full GCs (everything pauses)
[5.234s][info][gc] GC(47) Pause Full (Ergonomics) 245M->230M(256M) 2345.123ms
# 2.3 SECOND pause. All threads frozen. Heap almost full after GC = OOM incoming.

# Tool: Use GCEasy.io to upload and visualize GC logs automatically
````

### Writing Runbooks That Actually Help

````markdown
# Runbook: High Memory Usage Alert (orders-service)

## Audience
This runbook is for the on-call engineer who may not have prior context on this service.

## Alert: orders-service-memory-usage > 85%

## Step 1: Verify (2 minutes)
- Open Grafana → orders-service → Memory dashboard
- Is this a sudden spike or gradual growth?
  - Spike: likely a specific large request; go to Step 3
  - Gradual: likely a leak; go to Step 2

## Step 2: Take a Heap Dump (5 minutes)
```bash
kubectl exec -it $(kubectl get pod -l app=orders-service -o name | head -1) \
  -- jcmd 1 GC.heap_dump /tmp/heap-$(date +%s).hprof
kubectl cp orders-service-xxx:/tmp/heap-xxx.hprof ./heap-analysis.hprof
```
Open in VisualVM or Eclipse MAT.
Look at: Dominator Tree → What object is using the most memory?

## Step 3: Immediate Mitigation
If heap > 90% and service is degraded: rolling restart
```bash
kubectl rollout restart deployment/orders-service
```

## Step 4: Root Cause (post-stabilization)
- Check recent deploys: `kubectl rollout history deployment/orders-service`
- Check allocation rate in GC logs
- Contact [orders-team Slack channel]

## Known Past Incidents
- 2024-03-15: Unbounded Caffeine cache for product metadata. Fixed in PR #4521.
````

---

## 9. The Synthesis: What Production Engineering Actually Is {#synthesis}

````mermaid
flowchart LR
    subgraph PROD["🏭 Production Engineering"]
        direction TB
        OT["⏳ Works Over Time\n━━━━━━━━━━━━━━━━\n• No unbounded growth\n• Dependencies updated\n• Technical debt addressed\n• Memory stable"]
        UL["📈 Works Under Load\n━━━━━━━━━━━━━━━━\n• Concurrency handled\n• Performance measured\n• Throughput proven\n• Tail latency controlled"]
        DF["💥 Works Despite Failures\n━━━━━━━━━━━━━━━━\n• Circuit breakers active\n• Fallbacks implemented\n• Incidents recovered\n• Runbooks maintained"]
    end

    OT <--> UL
    UL <--> DF
    OT <--> DF

    RESULT(["🌟 Reliable System\nat Scale"])

    OT --> RESULT
    UL --> RESULT
    DF --> RESULT

    style OT fill:#3498db,color:#fff
    style UL fill:#e67e22,color:#fff
    style DF fill:#e74c3c,color:#fff
    style RESULT fill:#27ae60,color:#fff
````

### The Compound Habits of Attention

The principal engineer who spotted bugs in 4 seconds wasn't smarter. They had built habits of attention over 10 years. These habits can be cultivated deliberately:

````mermaid
flowchart TD
    NOTICE["🔍 Practice Noticing"]

    NOTICE --> H1["In every code review:\nAsk the 3 State Questions"]
    NOTICE --> H2["In every design:\nMap to the 5 Failure Causes"]
    NOTICE --> H3["In every incident:\nProduce a real postmortem"]
    NOTICE --> H4["In every deploy:\nRun a 60-second allocation profile"]
    NOTICE --> H5["In every alert:\nRead the GC log, not just the metric"]

    H1 --> COMPOUND["📈 Mental Model Compounds"]
    H2 --> COMPOUND
    H3 --> COMPOUND
    H4 --> COMPOUND
    H5 --> COMPOUND

    COMPOUND --> EXPERT["🧠 Expert Intuition:\nSpot the bug in 4 seconds"]

    style NOTICE fill:#2c3e50,color:#fff
    style COMPOUND fill:#8e44ad,color:#fff
    style EXPERT fill:#27ae60,color:#fff
````

---

## 10. Day One Wisdom — If I Could Start Over {#day-one}

| Wisdom | Why It Matters |
|---|---|
| **Learn the JVM deeply** | JOL, GC logs, heap dumps — investment compounds for 20+ years |
| **Bugs are in state, not algorithms** | Optimize your test suite for the bugs that actually hurt production |
| **Performance is system-wide** | Optimizing one component in isolation often hurts the whole |
| **Build observability from day one** | Retrofitting it later is 10× more expensive |
| **Read code more than you write** | JDK source, library source, postmortems — patterns repeat |
| **Be kind and patient** | Technical excellence and basic decency compound together |
| **Never take outages personally** | Every senior engineer has caused incidents; growth comes from learning |
| **Learn to write clearly** | Influence above mid-level happens through writing, not code |
| **Hold strong opinions loosely** | The system you're in has constraints you can't fully see |

---

## 📌 Quick Reference Card

````mermaid
mindmap
  root((Production\nEngineering\nCheat Sheet))
    5 Root Causes
      Unbounded state
      Unanticipated failure
      False assumption
      Unintended consequence
      Violated invariant
    3 State Questions
      What bounds it?
      Who owns lifecycle?
      Single vs multi-pod?
    Performance Trilemma
      Latency vs Throughput vs Cost
      Pick deliberately, not by drift
    Reliability Stack
      Timeouts everywhere
      Idempotent retries
      Circuit breakers
      Bulkheads
      Backpressure
    Instant Red Flags
      Static map written at runtime
      Empty catch blocks
      ThreadLocal without remove
      Loops with sync calls
      Shared mutable state
    Design Review Must-Asks
      What are we NOT doing?
      What if dep is down?
      Max size of every collection?
      Observability signals?
````

---

## 🎓 Summary

The gap between engineers whose systems you'd want to inherit and those whose systems you'd rather not isn't made of single brilliant decisions. It's made of thousands of small habits of attention:

- **Asking "what bounds this?"** before every piece of state ships
- **Mapping symptoms to the 5 root causes** before debugging
- **Choosing deliberately on the trilemma** rather than drifting
- **Flagging anti-patterns immediately** instead of letting them accumulate
- **Running design reviews with the right questions** before the code is written

The principal engineer who spotted your bug in 4 seconds didn't have a special gift. They practiced noticing for ten more years than you had. The mental model is teachable — and every code review, every incident, every design discussion is a chance to build it.

**Practice noticing. Everything else follows.**

---
*Based on "The Senior Java Engineer's Mental Model" — expanded with production code examples, architectural diagrams, and real-world use cases.*
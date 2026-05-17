# HikariCP Internals: What Every Spring Boot Developer Should Know Before the 2 AM Incident

> *You use HikariCP on every project. Most developers don't understand it until production forces them to. Here's the complete breakdown — before that happens to you.*

---

## The Incident That Started This

At 2:13 AM, PagerDuty fires. The checkout service — the core of the revenue pipeline — is returning 504 Gateway Timeout errors. Customers can't complete payments. Grafana shows the horror in real time:

- CPU across all pods: **98%**
- Active database connections: **maxed out**
- Pending connection requests: **climbing every second**
- `idle in transaction` sessions in PostgreSQL: **everywhere**

A thread dump from any pod tells the whole story immediately:

```
"http-nio-8080-exec-74" #74 daemon prio=5 os_prio=0 WAITING (parking)
   at sun.misc.Unsafe.park(Native Method)
   - parking to wait for <0x00000006c2e1f1b8> (a java.util.concurrent.Semaphore$NonfairSync)
   at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
   at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:182)
```

180+ threads. All blocked. All waiting for a database connection that isn't coming.

This is HikariCP pool exhaustion, and it's one of the most common and most misunderstood production failures in Java microservices. The root cause in this case turned out to be three lines of code — a `@Transactional` annotation on a method that called an external payment API. But to understand *why* that caused 98% CPU, you need to understand what HikariCP is actually doing under the hood.

---

## Why HikariCP Exists

Every database connection involves:
- A TCP handshake
- TLS negotiation
- Database authentication
- Session initialization

That's 50–100ms of overhead, per connection, per request. At any meaningful scale, that's catastrophic.

A connection pool maintains a set of pre-established, pre-authenticated connections. Your application *borrows* a connection from the pool, uses it, and *returns* it. The borrow/return cycle costs nanoseconds, not milliseconds.

HikariCP is Spring Boot's default pool (since 2.x) and consistently outperforms every alternative — C3P0, Apache DBCP2, Tomcat Pool — in throughput and latency benchmarks. It achieves this through a small set of genuinely clever engineering decisions.

---

## How HikariCP Works Internally

### The Core Data Structures

Every HikariCP pool is built around three components:

**`ConcurrentBag`** — a custom, lock-free collection holding all `PoolEntry` objects. This is HikariCP's most important innovation and what sets it apart from other pools.

**`PoolEntry`** — a wrapper around a raw `java.sql.Connection`. Each entry tracks its state: `NOT_IN_USE`, `IN_USE`, `RESERVED`, or `REMOVED`.

**`HouseKeeper`** — a background thread running every 30 seconds that evicts idle connections, enforces `maxLifetime`, and runs keepalive queries.

### The Connection Borrow Path

When your code — directly or through Spring's transaction manager — calls `dataSource.getConnection()`, HikariCP executes this sequence:

```
1. Check thread-local handoff list (fast path)
  └─ If the same thread recently used a connection, return it immediately
     No locking. No CAS. Nanosecond-level.

2. Scan ConcurrentBag for NOT_IN_USE entries (CAS-based)
  └─ Atomically mark one as IN_USE and return it

3. Pool is full? Park on a Semaphore until connectionTimeout
  └─ Thread state becomes WAITING
  └─ If timeout expires: SQLTransientConnectionException
```

The fast path (step 1) is why HikariCP can borrow a connection in under 50 nanoseconds in warm JVM scenarios. The same thread repeatedly acquiring and releasing a connection — the typical pattern in a single request — never touches any shared data structure.

### ConcurrentBag: The Secret Sauce

Most connection pools use a shared queue or stack protected by a lock. Under high concurrency, every thread contends on the same lock. Throughput plateaus.

ConcurrentBag uses a fundamentally different design: a thread-local handoff list combined with a shared list scanned via lock-free CAS operations. When a thread borrows a connection:

1. It checks its own handoff list first — zero contention
2. If empty, it scans the shared list using Compare-And-Swap — no blocking
3. Other threads can simultaneously scan and succeed independently

The result: HikariCP scales near-linearly with thread count up to pool saturation, while traditional pools exhibit lock contention long before that.

### The Return Path

When `connection.close()` is called on a HikariCP proxy (note: this doesn't actually close the connection — it returns it to the pool):

1. The `PoolEntry` state is set back to `NOT_IN_USE`
2. The `Semaphore` releases one permit
3. The next parked thread waiting in step 3 above wakes up and receives the connection

This is the critical link: every waiting thread is blocked on `Semaphore.acquire()`. When a connection is returned, exactly one waiting thread unblocks. When no threads are waiting, the connection goes back to the shared list for the next borrow.

---

## Why Pool Exhaustion Causes CPU Spikes

This surprises most people. Threads are *waiting* — why would CPU spike to 98%?

The answer is **context switching**.

When 180 threads are all in `WAITING` state and connections start returning, the JVM and OS must:

1. Wake each thread as a permit becomes available
2. Schedule it onto a CPU core
3. Let it try to acquire the semaphore
4. If another thread got there first, park it again

At 180 threads competing for 20 connections returning one at a time, the scheduler is constantly cycling threads on and off CPU cores — doing almost no real work. This is a **context-switch storm**, and it's why CPU pegs at near-100% while simultaneously doing almost nothing useful.

The thread dump signature is unmistakable: hundreds of `WAITING (parking)` threads, all on the same semaphore address.

```bash
# Confirm it: count threads waiting on HikariCP
grep -A5 "HikariPool.getConnection" threaddump.txt | grep -c "WAITING"
```

If that count is in the double or triple digits, you have pool exhaustion.

---

## The Root Cause Taxonomy: What Actually Causes Exhaustion

### Cause 1: `@Transactional` on Methods That Call External Services

This is the most common cause and the hardest to spot in code review.

```java
// DANGEROUS: Transaction holds a DB connection open for the entire method
@Transactional
public OrderResult processPayment(Order order) {
   orderRepository.save(order);              // 2ms — connection acquired here
   PaymentResult result = paymentGateway
       .charge(order.getCard(), order.getTotal()); // 20 SECONDS — connection still held
   orderRepository.updateStatus(order, result);   // 2ms
   return new OrderResult(result);
}
```

The database connection is acquired when the transaction opens — before the external call — and held until the transaction commits. For 20 seconds per request, each thread holds a connection completely idle while waiting for a network response.

**The fix:**

```java
// Split the transaction boundary
public OrderResult processPayment(Order order) {
   orderService.saveOrder(order);                    // @Transactional — connection held ~2ms

   PaymentResult result = paymentGateway             // No transaction — no connection held
       .charge(order.getCard(), order.getTotal());

   orderService.updateOrderStatus(order, result);    // @Transactional — connection held ~2ms
   return new OrderResult(result);
}

@Transactional
public void saveOrder(Order order) {
   orderRepository.save(order);
}

@Transactional
public void updateOrderStatus(Order order, PaymentResult result) {
   orderRepository.updateStatus(order, result);
}
```

The connection is now held for 4ms total instead of 20 seconds. Your pool can handle 500x more concurrent requests with the same configuration.

### Cause 2: Connection Leaks

A connection leak happens when a connection is borrowed and never returned.

```java
// LEAK: Exception path skips close()
public List<Product> getProducts() throws SQLException {
   Connection conn = dataSource.getConnection();
   Statement stmt = conn.createStatement();
   ResultSet rs = stmt.executeQuery("SELECT * FROM products");
   List<Product> products = mapResults(rs); // throws RuntimeException
   conn.close(); // never reached
   return products;
}

// LEAK: Stream not closed
public void processOrders() {
   jdbcTemplate.queryForStream("SELECT * FROM orders", rowMapper)
       .forEach(this::process); // Stream not closed → connection held indefinitely
}
```

Leaked connections don't return to the pool. Over hours, the effective pool size shrinks to zero. Enable leak detection to catch these early:

```yaml
spring:
 datasource:
   hikari:
     leak-detection-threshold: 10000  # Log any connection held > 10 seconds
```

This produces a log entry with the full stack trace of where the connection was acquired:

```
WARN  c.z.h.p.ProxyLeakTask - Connection leak detection triggered for
     conn=ProxyConnection wrapping conn0: url=jdbc:postgresql://...
     on thread http-nio-8080-exec-45
     java.lang.Exception: Apparent connection leak detected
         at com.example.ProductRepository.getProducts(ProductRepository.java:34)
```

Always use try-with-resources or Spring's `JdbcTemplate` (which handles connection lifecycle automatically):

```java
// SAFE: try-with-resources
try (Connection conn = dataSource.getConnection();
    PreparedStatement stmt = conn.prepareStatement(sql);
    ResultSet rs = stmt.executeQuery()) {
   return mapResults(rs);
}

// SAFE: JdbcTemplate manages the connection
return jdbcTemplate.query(sql, rowMapper);

// SAFE: close the stream
try (Stream<Order> orders = jdbcTemplate.queryForStream(sql, rowMapper)) {
   orders.forEach(this::process);
}
```

### Cause 3: Misconfigured Pool Size

Two opposite mistakes are equally dangerous:

**Too small** — obvious exhaustion under load.

**Too large** — counterintuitively, also causes problems. More connections means more concurrent queries hitting the database. The database has its own resource limits (CPU, I/O, lock contention). Throwing 200 connections at a PostgreSQL instance optimized for 50 concurrent queries doesn't increase throughput — it increases database CPU, extends query durations, and makes exhaustion worse.

A useful starting formula (from HikariCP's own documentation):

```
maxPoolSize = (number_of_cores * 2) + effective_spindle_count
```

For most cloud database instances, this lands between 10 and 30 per application instance. The right number depends on your query profile and database capacity — measure, don't guess.

---

## Configuration Reference

### Development / Low Traffic

```yaml
spring:
 datasource:
   hikari:
     maximum-pool-size: 5
     minimum-idle: 2
     connection-timeout: 5000    # 5 seconds — generous for dev
     idle-timeout: 300000        # 5 minutes
     max-lifetime: 1800000       # 30 minutes
```

### Production Microservice

```yaml
spring:
 datasource:
   hikari:
     maximum-pool-size: 20
     minimum-idle: 20            # Keep pool fully warm — avoid cold starts under load
     connection-timeout: 2000    # 2 seconds — fail fast, don't queue forever
     idle-timeout: 600000        # 10 minutes
     max-lifetime: 1800000       # 30 minutes — must be < DB connection timeout
     keepalive-time: 60000       # 1 minute — prevent firewall idle drops
     leak-detection-threshold: 10000
     pool-name: checkout-db-pool  # Named pools make logs and metrics readable
```

### Critical: `maxLifetime` Must Be Less Than Your Database's `wait_timeout`

If your PostgreSQL `tcp_keepalives_idle` or MySQL `wait_timeout` is 1800 seconds and your `maxLifetime` is also 1800 seconds, there's a race condition: the database drops the connection a fraction of a second before HikariCP retires it, causing `Connection reset` errors under load.

Set `maxLifetime` to at least 30 seconds *less* than the database's connection timeout.

```yaml
# If DB wait_timeout = 1800s
max-lifetime: 1740000  # 29 minutes — safely inside the DB's limit
```

---

## Monitoring: The Metrics That Matter

Enable Micrometer metrics via Spring Boot Actuator:

```yaml
management:
 endpoints:
   web:
     exposure:
       include: health, metrics, prometheus
 metrics:
   tags:
     application: ${spring.application.name}
```

### Essential HikariCP Metrics

| Metric | What It Tells You |
|---|---|
| `hikaricp_connections_active` | Connections currently in use |
| `hikaricp_connections_idle` | Connections available to borrow |
| `hikaricp_connections_pending` | Threads waiting for a connection |
| `hikaricp_connections_timeout_total` | Cumulative connection timeout count |
| `hikaricp_connections_acquire_seconds` | Time to acquire a connection (latency) |
| `hikaricp_connections_creation_seconds` | Time to create a new physical connection |

The most important operational signal is `hikaricp_connections_pending`. Under normal conditions, this should be zero. Any sustained non-zero value means your pool is running at capacity. A rising value means you're heading toward the 2 AM incident.

### Prometheus Alert Rules

```yaml
groups:
 - name: hikaricp
   rules:
     - alert: HikariPoolNearExhaustion
       expr: hikaricp_connections_pending > 5
       for: 30s
       annotations:
         summary: "{{ $labels.pool }} has {{ $value }} threads waiting for connections"

     - alert: HikariConnectionTimeouts
       expr: rate(hikaricp_connections_timeout_total[5m]) > 0
       for: 1m
       annotations:
         summary: "Connection timeouts occurring in {{ $labels.pool }}"

     - alert: HikariAcquireLatencyHigh
       expr: histogram_quantile(0.95, hikaricp_connections_acquire_seconds_bucket) > 0.5
       for: 2m
       annotations:
         summary: "p95 connection acquire time > 500ms in {{ $labels.pool }}"
```

### Grafana Dashboard

A useful pool health panel stacks three metrics against the pool size limit:

```
active connections (filled)
+ idle connections (filled)
+ pending connections (filled, alarming color)
─────────────────────────────────
maximum-pool-size (horizontal threshold line)
```

When active + pending approaches `maximum-pool-size`, you're close to exhaustion. When pending is visible at all, you're already there.

---

## Thread Dump Analysis

When the system is misbehaving, a thread dump tells you exactly what's happening. Collect a series rather than a single snapshot:

```bash
# Five dumps, two seconds apart — reveals whether threads are stuck or moving
for i in {1..5}; do
   jstack <pid> > dump_$i.txt
   sleep 2
done
```

### Signature 1: Pool Exhaustion

```
"http-nio-8080-exec-97" #97 daemon WAITING (parking)
   at sun.misc.Unsafe.park(Native Method)
   - parking to wait for <0x00000006c2e1f1b8> (a java.util.concurrent.Semaphore$NonfairSync)
   at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
   at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:182)
```

If this pattern appears for dozens or hundreds of threads, you have pool exhaustion. Count them:

```bash
grep -c "HikariPool.getConnection" dump_1.txt
```

### Signature 2: Connection Leak

```
"http-nio-8080-exec-45" #45 daemon TIMED_WAITING (parking)
   at sun.misc.Unsafe.park(Native Method)
   at com.example.PaymentService.processPayment(PaymentService.java:67)
   at com.zaxxer.hikari.pool.ProxyConnection.<held by thread>
```

A thread holding a `ProxyConnection` while parked in application code is holding a database connection while waiting for something else — an external API, a sleep, a lock.

### Signature 3: Slow Queries

```
"http-nio-8080-exec-12" #12 daemon RUNNABLE
   at java.net.SocketInputStream.socketRead0(Native Method)
   at com.mysql.cj.protocol.FullReadInputStream.readFully(...)
   at com.zaxxer.hikari.pool.ProxyStatement.executeQuery(...)
   at com.example.ReportRepository.generateReport(ReportRepository.java:134)
```

Threads in `RUNNABLE` state inside database driver code are executing queries. If many threads are here, queries are slow — investigate query plans, not pool configuration.

---

## Anti-Patterns to Eliminate

**`@Transactional` at the controller level.** Every HTTP request holds a connection for the entire duration of request processing, including serialization overhead, validation, and any non-database work.

```java
@Transactional  // Never do this on a controller
@GetMapping("/orders")
public ResponseEntity<List<Order>> getOrders() { ... }
```

**Separate pools for the same database without reason.** Multiple `DataSource` beans pointing at the same PostgreSQL instance multiply connection count from the database's perspective but don't increase capacity.

**Treating `maximum-pool-size` as "more is better."** PostgreSQL's `max_connections` is typically 100. Four pods each with a pool of 30 = 120 connections, which exceeds the default database limit. You'll get `FATAL: sorry, too many clients already` errors under full deployment.

**Ignoring `idle-timeout` and `max-lifetime`.** Connections don't live forever. Network devices drop idle connections. Database servers close stale sessions. Without `max-lifetime` set below your database's timeout, you'll get sporadic `Connection reset` errors in production that are nearly impossible to reproduce locally.

**No connection name or pool name.** When you have five microservices all reporting `HikariPool-1` in logs and metrics, diagnosing which service is the problem under pressure is painful. Name your pools:

```yaml
spring.datasource.hikari.pool-name: payment-service-db
```

---

## What Fixed the 2 AM Incident

The complete set of changes that brought CPU from 98% to 12%:

1. **Removed `@Transactional` from the payment processing method** — split it into two thin transactional calls around the external API call. Connection hold time dropped from ~20 seconds to ~4 milliseconds.

2. **Added `leak-detection-threshold: 5000`** — immediately revealed two other slow methods holding connections longer than expected.

3. **Reduced `connection-timeout` from 30 seconds to 2 seconds** — instead of 180 threads piling up waiting 30 seconds each, threads now fail fast after 2 seconds, triggering the circuit breaker and shedding load.

4. **Added a circuit breaker around the payment API** — so that a slow payment gateway degrades gracefully rather than cascading into database connection exhaustion.

5. **Added Prometheus alerts on `hikaricp_connections_pending`** — next time the pending count rises above zero, an alert fires before threads reach the exhaustion point.

The root cause of 98% CPU was never the database. It was a transaction boundary that was three lines too wide.

---

## Summary

HikariCP is transparent until it isn't. When it breaks, it breaks hard and fast — and the failure mode (CPU spike from thread context switching) is counterintuitive enough that most engineers lose time investigating the wrong layer.

The mental model that prevents most incidents:

> **A database connection is held from the moment your transaction opens to the moment it commits. Every millisecond of non-database work inside that boundary is a millisecond of wasted pool capacity.**

Keep transactions narrow. Detect leaks early. Monitor pending connections, not just active ones. Name your pools. Set `maxLifetime` below your database's idle timeout. And when something goes wrong at 2 AM, read the thread dump first — it will tell you exactly what's happening.
~~~markdown~~~
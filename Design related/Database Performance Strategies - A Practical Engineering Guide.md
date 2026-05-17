# Database Performance Strategies: A Practical Engineering Guide

> *Performance problems don't announce themselves during development. They arrive in production, at scale, under load. Understanding these six strategies before you need them is the difference between a planned optimization and a 2 AM emergency.*

![](https://substackcdn.com/image/fetch/$s_!J1Or!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F76a7bcae-46a0-4c6f-a297-a033f7564d20_2250x2624.png)
---

## Start Here: What Actually Affects Database Performance

Before reaching for any optimization technique, it's worth being precise about what you're actually optimizing for. Database performance is not a single dial. It's a multi-dimensional problem shaped by at least eight independent factors, and the right strategy depends entirely on which factors are pressuring your system.

**Item Size** — How large is each row? A table of user profiles with a few varchar columns behaves completely differently from a table storing JSON blobs or binary data. Larger rows mean fewer rows fit in a database page, which means more I/O for the same number of rows fetched.

**Item Type** — Are you storing structured relational data, documents, time-series measurements, geospatial coordinates, or full-text content? The data type dictates which storage engine and index structures are appropriate. Using a relational database for time-series data, or a document store for heavily relational data, creates a structural mismatch no amount of tuning can fully overcome.

**Dataset Size** — A query plan that works perfectly at 100,000 rows can collapse at 100 million. Dataset size determines whether indexes fit in memory (fast) or require disk I/O (slow), and whether query plans remain optimal or need rethinking.

**Concurrency** — How many clients are reading and writing simultaneously? High concurrency introduces lock contention, connection pool exhaustion, and deadlock risk. A system that runs fine with 10 concurrent users can exhibit entirely different failure modes at 10,000.

**Consistency Expectations** — Do you need strict ACID guarantees, or can you tolerate eventual consistency? Strict consistency requires locking and coordination mechanisms that limit throughput. Relaxing consistency requirements (where the domain allows) is often the highest-leverage performance improvement available.

**Geographic Distribution** — Are your users in one region or distributed globally? A database in us-east-1 serving users in Southeast Asia adds 200–300ms of network latency to every query, which no index or cache can eliminate. Geographic distribution requires architectural solutions.

**High Availability Expectations** — What's your acceptable downtime? High availability requires replication and failover mechanisms, which add write overhead and operational complexity.

**Workload Variability** — Is your traffic steady or bursty? A system sized for average load will be overwhelmed by peak load. A system sized for peak load wastes resources at average load. Variability favors elastic, horizontally scalable architectures.

**The diagnostic implication:** before applying any of the strategies below, identify which of these factors is your actual bottleneck. Adding indexes doesn't help a system bottlenecked by geographic latency. Sharding doesn't help a system bottlenecked by lock contention on a single row. Diagnose first, optimize second.

---

## Strategy 1: Indexing — Making the Database Do Less Work

An index is a separate data structure that maps column values to row locations, allowing the database to jump directly to matching rows instead of scanning the entire table.

Without an index on `email`:
```sql
SELECT * FROM customers WHERE email = 'alice@example.com';
-- Database reads every row, one by one, until it finds a match
-- O(n) — performance degrades linearly as the table grows
```

With a B-Tree index on `email`, the database maintains a sorted structure where each entry maps a value to a row pointer:

```
Index (sorted by email):        Table (stored by insertion order):
alice@example.com  → row 2      row 1: john@example.com
bob@example.com    → row 3      row 2: alice@example.com
jane@example.com   → row 5      row 3: bob@example.com
john@example.com   → row 1      row 4: mark@example.com
mark@example.com   → row 4      row 5: jane@example.com
```

The index is sorted; the table is not. The database traverses the index in `O(log n)` using binary search, finds the row pointer, and fetches directly. At 50 million rows, this is the difference between examining 26 index entries versus 50 million table rows.

### What to Index and What to Skip

Index columns that appear in `WHERE` clauses, `JOIN` conditions, and `ORDER BY` clauses with high cardinality (many distinct values). High-cardinality columns like `email`, `user_id`, and `transaction_id` benefit enormously. Low-cardinality columns like `status` (3 possible values) or `is_active` (2 possible values) often don't — if 40% of rows match the filter, a full scan may be faster than an index lookup plus row fetches.

**Composite indexes respect column order.** An index on `(customer_id, status, created_at)` accelerates queries filtering on `customer_id` alone, or `customer_id + status`, or all three — but not `status` alone or `created_at` alone. Design composite indexes to match your query patterns, leftmost column first.

### The Write Tax

Every index must be updated on every INSERT, UPDATE, and DELETE. A table with 10 indexes performs 11 write operations per row change. For write-heavy systems, index proliferation degrades write throughput significantly. Audit indexes regularly — an index that's never scanned is pure overhead.

```sql
-- PostgreSQL: find indexes that have never been used
SELECT indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND schemaname = 'public';
```

### Covering Indexes: Eliminating the Row Lookup

A covering index includes all columns a query needs, so the database never has to fetch the actual row:

```sql
-- This query can be answered entirely from the index
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id, status)
INCLUDE (total_amount, created_at);

SELECT status, total_amount, created_at
FROM orders
WHERE customer_id = 42;
-- Index only scan — no table access at all
```

For read-heavy, latency-sensitive queries, covering indexes are one of the highest-leverage optimizations available.

---

## Strategy 2: Denormalization — Trading Storage for Query Speed

Normalization reduces data redundancy by decomposing data into separate tables joined by foreign keys. This is the right default for data integrity and consistency. But joins have a cost — and at scale, that cost becomes significant.

Consider a reporting query that needs product name, customer name, segment name, order ID, and order amount. In a fully normalized schema, this requires four joins across four tables:

```sql
SELECT
   p.name AS product_name,
   s.name AS segment_name,
   c.name AS customer_name,
   o.id AS order_id,
   o.amount AS order_amount
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN segments s ON c.segment_id = s.id
JOIN products p ON o.product_id = p.id
WHERE c.id = 12345;
```

At scale, this query requires the database to traverse four separate table's indexes, perform four lookups or scans, and join the results. For a high-frequency reporting endpoint, this is expensive.

**Denormalization collapses the joins into a single table:**

```sql
-- A denormalized customer_orders table
CREATE TABLE customer_orders (
   id BIGINT PRIMARY KEY,
   order_id BIGINT,
   order_amount DECIMAL(10,2),
   customer_name VARCHAR(200),
   segment_name VARCHAR(100),
   product_name VARCHAR(200)
);

-- The same query, now a single table scan with one index lookup
SELECT product_name, segment_name, customer_name, order_id, order_amount
FROM customer_orders
WHERE customer_id = 12345;
```

The multi-table join disappears. Query complexity collapses. Read performance improves dramatically.

### The Denormalization Trade-Off

Denormalization is never free. The costs are real and must be weighed deliberately:

**Data redundancy.** If a customer's name changes, it must be updated in every row of `customer_orders` that references them — not just in the `customers` table. Failing to do this creates inconsistency.

**Write complexity.** Every write that touches source data must also update denormalized copies, often requiring application-level coordination or database triggers.

**Storage overhead.** Duplicated data occupies more disk space. At significant scale, this is a non-trivial cost.

**When denormalization is justified:** read-heavy workloads where the same joins run thousands of times per second, reporting and analytics tables where data is written infrequently but read constantly, and materialized views where the database handles the synchronization automatically.

```sql
-- PostgreSQL materialized view: denormalization managed by the database
CREATE MATERIALIZED VIEW customer_orders_mv AS
SELECT
   o.id, o.amount,
   c.name AS customer_name,
   s.name AS segment_name,
   p.name AS product_name
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN segments s ON c.segment_id = s.id
JOIN products p ON o.product_id = p.id;

-- Refresh on a schedule or after bulk updates
REFRESH MATERIALIZED VIEW CONCURRENTLY customer_orders_mv;
```

Materialized views give you the read performance of denormalization with the consistency benefits of managing the source data in normalized form.

---

## Strategy 3: Optimistic Locking — Concurrency Without the Contention

When two users read and then update the same row concurrently, you have a race condition. The naive approach — pessimistic locking — prevents this by locking the row the moment someone reads it:

```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- Row is locked until commit
```

Pessimistic locking is safe but serializes all access to that row. Under high concurrency, threads queue up waiting for locks, throughput collapses, and deadlock risk grows.

**Optimistic locking takes a different approach:** don't lock on read. Instead, detect conflicts on write using a version number.

```sql
-- Table includes a version column
CREATE TABLE accounts (
   id BIGINT PRIMARY KEY,
   amount DECIMAL(10,2),
   version INT NOT NULL DEFAULT 1
);
```

The sequence for a safe concurrent update:

```sql
-- Step 1: Sarah reads the account (no lock)
SELECT id, amount, version FROM accounts WHERE id = 1;
-- Returns: id=1, amount=40, version=1

-- Step 2: John also reads the account (no lock, concurrent)
SELECT id, amount, version FROM accounts WHERE id = 1;
-- Returns: id=1, amount=40, version=1

-- Step 3: John updates first, incrementing the version
UPDATE accounts
SET amount = 20, version = 2
WHERE id = 1 AND version = 1;  -- WHERE clause checks version
-- 1 row affected — John's update succeeds

-- Step 4: Sarah tries to update, but version has changed
UPDATE accounts
SET amount = 20, version = 2
WHERE id = 1 AND version = 1;  -- version is now 2, not 1
-- 0 rows affected — Sarah's update fails
-- Application detects 0 rows affected, retries with fresh data
```

The `WHERE id = 1 AND version = 1` clause is the conflict detection mechanism. If the version has been incremented since you read the row, your update affects zero rows — a signal to the application that a concurrent modification occurred and a retry is needed.

This pattern eliminates lock contention entirely for the common case where conflicts are rare. Reads never block. Writes only retry on actual conflicts. For most applications, where the same row is rarely updated by two users simultaneously, optimistic locking dramatically outperforms pessimistic locking.

### When to Use Each

**Optimistic locking** — low write contention, reads are much more frequent than writes, short update operations. Most web applications.

**Pessimistic locking** — high write contention on specific rows, operations that must not be retried (financial transactions requiring exact once-semantics), long-running operations where retrying is expensive.

Most ORMs support optimistic locking natively:

```java
// JPA/Hibernate
@Entity
public class Account {
   @Id
   private Long id;

   private BigDecimal amount;

   @Version  // Hibernate manages the version column automatically
   private Integer version;
}
```

---

## Strategy 4: Replication — Scaling Reads and Improving Availability

A single database node is both a performance bottleneck and a single point of failure. Replication solves both problems by maintaining synchronized copies of the data on multiple nodes.

```
Write traffic  →  Leader Node  →  Replication Stream  →  Follower Node 1
                                                     →  Follower Node 2

Read traffic   →  Follower Node 1  (read-only queries)
              →  Follower Node 2  (read-only queries)
```

The leader accepts all writes and streams changes to followers in near real-time. Followers accept read-only queries. Most production applications are read-heavy — 80–95% of database operations are reads. Replication lets you scale read capacity horizontally by adding followers, without any changes to your write path.

### Practical Configuration in Spring Boot

```yaml
spring:
 datasource:
   # Primary datasource — all writes
   primary:
     url: jdbc:postgresql://leader:5432/mydb
     hikari:
       maximum-pool-size: 20

   # Read replica — queries routed here explicitly
   readonly:
     url: jdbc:postgresql://follower:5432/mydb
     hikari:
       maximum-pool-size: 40  # More connections for read-heavy load
```

```java
@Service
public class ProductService {

   @Autowired
   @Qualifier("primaryDataSource")
   private DataSource writeDataSource;

   @Autowired
   @Qualifier("readonlyDataSource")
   private DataSource readDataSource;

   @Transactional  // Uses primary — write path
   public Product createProduct(CreateProductRequest request) { /* ... */ }

   @Transactional(readOnly = true)  // Route to replica
   public List<Product> searchProducts(SearchQuery query) { /* ... */ }
}
```

### The Replication Lag Problem

Followers are not instantaneously consistent with the leader. Replication is asynchronous — there's a small window (typically milliseconds, occasionally seconds under heavy load) where a follower hasn't yet received the latest writes.

This creates a class of subtle bugs: a user creates a record, the write goes to the leader, the read for confirmation is routed to a follower that hasn't replicated the write yet, and the record appears to not exist.

**Mitigations:**
- Route reads to the leader for operations where the user just performed a write (read-your-writes consistency)
- Use synchronous replication for critical data (at the cost of write latency)
- Accept eventual consistency where the domain permits (displaying a count that's a few seconds stale is usually fine)
- Monitor replication lag as a first-class metric — alert when it exceeds acceptable thresholds

```sql
-- PostgreSQL: check replication lag on the leader
SELECT client_addr, state,
      pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;
```

---

## Strategy 5: Sharding — Horizontal Partitioning for Scale Beyond One Machine

Every single-node database has hard limits: maximum storage, maximum connection count, maximum write throughput. When you hit those limits, vertical scaling (bigger hardware) has diminishing returns and an eventual ceiling. Sharding is the solution.

**Sharding partitions a dataset across multiple independent database nodes**, called shards. Each shard holds a subset of the data and is fully independent — its own storage, its own connection pool, its own write path.

```
Monolithic Database (1 node, all data)
       ↓
Shard 1: users 1–10M       (node: db-shard-1)
Shard 2: users 10M–20M     (node: db-shard-2)
Shard 3: users 20M–30M     (node: db-shard-3)
```

Write throughput scales linearly with shard count. Storage capacity scales linearly. Connection limits scale linearly. Sharding converts a vertical scaling problem into a horizontal one.

### Shard Key Selection: The Critical Decision

Every sharding design lives or dies by the shard key — the column used to determine which shard a row belongs to. A poor shard key creates hotspots: one shard receives the majority of traffic while others sit idle, eliminating the benefit of sharding.

**Common shard key strategies:**

```sql
-- Hash-based sharding: uniform distribution, but range queries span shards
shard = hash(user_id) % num_shards

-- Range-based sharding: efficient range queries, but risk of hotspots
-- Shard 1: created_at < 2023-01-01
-- Shard 2: 2023-01-01 <= created_at < 2024-01-01
-- (recent data hotspot is common with time-based sharding)

-- Directory-based sharding: flexible routing via a lookup table
-- Requires an additional lookup service but allows resharding
shard = lookup_table[tenant_id]
```

For multi-tenant SaaS applications, `tenant_id` is often the natural shard key — it provides natural isolation between tenants and ensures all of one tenant's data lives on one shard, making cross-table joins within a tenant efficient.

### The Costs of Sharding

Sharding is the most operationally complex strategy in this list. Before committing to it, exhaust other options:

**Cross-shard queries become application-level joins.** A query like "find all orders over $500 across all users" must be sent to every shard and the results merged in the application. There's no database-level optimizer to help you here.

**Resharding is painful.** If you start with 4 shards and need 8, moving data between shards while the system remains live requires careful, multi-phase migrations. Planning for resharding ahead of time — consistent hashing, directory-based routing — is essential.

**Transactions don't span shards.** ACID guarantees apply within a shard. Operations touching data on multiple shards require distributed transaction protocols (two-phase commit) or eventual consistency patterns. Both add complexity.

**Operational overhead multiplies.** Backups, monitoring, schema migrations, and query debugging all become N times more complex where N is your shard count.

A pragmatic rule of thumb: exhaust indexing, query optimization, connection pooling, caching, and read replicas before considering sharding. Most applications never need it. When they do, it's usually because of write throughput or storage limits — and sharding addresses both directly.

---

## Putting It All Together: A Decision Framework

These six strategies aren't alternatives — they're layers that work together. A mature database architecture typically combines several of them.

| Problem | First Strategy to Try | When to Escalate |
|---|---|---|
| Slow reads on specific columns | Indexing | If reads are still slow after indexing, consider denormalization or caching |
| Slow multi-table join queries | Indexing on join columns | If joins are unavoidably complex and frequent, consider denormalization |
| Concurrent write conflicts | Optimistic locking | If conflict rate is high, consider pessimistic locking or serializable isolation |
| Read throughput ceiling | Read replicas | If write throughput is also bottlenecked, consider sharding |
| Storage or write throughput ceiling | Sharding | If cross-shard queries become too complex, reconsider data model |
| Geographic latency | Regional replicas or multi-region deployment | Requires careful consistency planning |

The sequence that applies to most production systems as they scale:

1. **Index strategically** — almost always the first intervention. High impact, low risk.
2. **Tune queries and connection pools** — eliminate unnecessary work before adding infrastructure.
3. **Add read replicas** — scale read throughput horizontally once indexing is solid.
4. **Denormalize selectively** — for specific high-frequency read paths where joins are the bottleneck.
5. **Implement optimistic locking** — for any rows with concurrent write access.
6. **Shard** — only when you've genuinely hit the limits of a single-node write path.

The engineers who build fast, reliable databases aren't the ones who applied every technique — they're the ones who applied the right technique for the actual bottleneck, in the right order, with measurement at every step.

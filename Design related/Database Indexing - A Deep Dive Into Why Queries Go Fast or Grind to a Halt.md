# Database Indexing: A Deep Dive Into Why Queries Go Fast or Grind to a Halt

> *Indexing is one of the highest-leverage tools in a backend engineer's toolkit. Understanding it beyond "add an index on WHERE columns" is what separates engineers who guess at performance from engineers who diagnose it.*

---

## The Problem Statement

Your application launches. Queries run in milliseconds. Six months later, the same queries are timing out. Nothing in the code changed. The database is the same. The hardware is the same.

The difference is data volume — and the absence of a thoughtful indexing strategy.

This isn't a rare edge case. It's the normal trajectory of every application that grows. A full table scan on 10,000 rows is imperceptible. The same scan on 50 million rows can take seconds and saturate your database CPU while doing it. Indexing is how you keep query performance from degrading as your data scales.

But indexing isn't free, and "add more indexes" is bad advice. Every index is a write tax — a data structure the database must maintain on every INSERT, UPDATE, and DELETE. A poorly considered index strategy can make writes slower, waste storage, and confuse the query optimizer into making worse decisions.

This guide covers how indexes actually work internally, what types exist and when each is appropriate, how to read execution plans to verify your indexes are being used, and the mistakes that turn a good idea into a performance liability.

---

## What an Index Actually Is

The book analogy is accurate but incomplete. An index is a separate data structure, stored on disk, that maps column values to the physical location of rows. The database maintains this structure in parallel with the table itself.

The dominant index structure in relational databases is the **B-Tree** (Balanced Tree). Understanding why requires a brief look at what it actually does.

### B-Tree Internals

A B-Tree is a self-balancing tree where every leaf node is at the same depth. For a database index on an `email` column, it looks roughly like this:

```
                   [M]
                  /   \
         [D - H]        [R - W]
        /   |   \      /   |   \
     [A-C] [E-G] [I-L] [N-Q] [S-V] [X-Z]
       |     |     |     |     |     |
    (row   (row  (row  (row  (row  (row
    ptrs)  ptrs) ptrs) ptrs) ptrs) ptrs)
```

Each leaf node contains the indexed value and a pointer to the physical row location (a row ID, page number, or primary key, depending on the database). The internal nodes contain separator keys that guide the search.

To find `email = 'bob@example.com'`:
1. Start at the root — go right (B > M? No, go left)
2. Traverse one internal node
3. Reach a leaf node with the matching value and its row pointer
4. Follow the pointer to fetch the actual row

This is `O(log n)` — a B-Tree with 50 million rows has a depth of roughly 25. That's at most 25 comparisons and a handful of disk I/O operations, regardless of table size. A full table scan is `O(n)` — potentially 50 million row reads.

The practical difference: milliseconds versus minutes at scale.

### What "Clustered" vs. "Non-Clustered" Means

Most developers encounter these terms and find the distinction murky.

**A clustered index** determines the physical order of rows on disk. The table *is* the index — rows are stored in index order. In MySQL/InnoDB, the primary key is always the clustered index. There can only be one per table (rows can only be in one physical order).

**A non-clustered index** is a separate structure that points back to the table. When you create an index on `email`, the database builds a B-Tree of email values with row pointers. Following a pointer from this index to the actual row is called a **bookmark lookup** or **key lookup** — an extra hop that has a cost.

This distinction has important practical implications:

```sql
-- These use the clustered index directly — single lookup
SELECT * FROM users WHERE id = 42;

-- This uses a non-clustered index, then does a key lookup for remaining columns
SELECT * FROM users WHERE email = 'bob@example.com';

-- This can avoid the key lookup entirely if the index covers all needed columns
SELECT id, email FROM users WHERE email = 'bob@example.com';
```

The last query is a **covering index scan** — all columns needed are in the index itself, no row lookup required. This is one of the most impactful optimizations available for read-heavy queries.

---

## Index Types: When to Use Each

### B-Tree Index (The Default)

Used for equality filters, range queries, sorting, and joins. This is what `CREATE INDEX` creates by default in every major database.

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

Effective for:
- `WHERE customer_id = 101`
- `WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'`
- `ORDER BY created_at DESC`
- `JOIN ON customer_id`

Not effective for:
- `WHERE email LIKE '%gmail.com'` — leading wildcard defeats B-Tree traversal
- `WHERE LOWER(email) = 'bob@example.com'` — function on the column defeats index use (more on this below)

### Unique Index

A B-Tree index that also enforces uniqueness at the database level. Creates an implicit constraint — the database rejects duplicate values on INSERT and UPDATE.

```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

Use this for natural unique identifiers: email addresses, usernames, national ID numbers, transaction IDs. Don't use a unique constraint at the application layer alone — the database is the last line of defense against race conditions.

### Composite Index (Multi-Column)

A single B-Tree index over multiple columns. The critical concept here is **column order**, which most developers get wrong.

```sql
CREATE INDEX idx_orders_customer_status ON orders(customer_id, status, created_at);
```

A composite index can be used by queries that filter on the **leftmost prefix** of the indexed columns:

```sql
-- Uses the index (leftmost column)
WHERE customer_id = 101

-- Uses the index (leftmost two columns)
WHERE customer_id = 101 AND status = 'PENDING'

-- Uses the index (all three columns)
WHERE customer_id = 101 AND status = 'PENDING' AND created_at > '2024-01-01'

-- Cannot use this index (skips customer_id)
WHERE status = 'PENDING'

-- Cannot use this index efficiently (skips status)
WHERE customer_id = 101 AND created_at > '2024-01-01'
```

This is the **leftmost prefix rule**. Design composite indexes by ordering columns from most selective to least selective, or by matching the exact column order of your most frequent query patterns.

A useful mental model: a composite index on `(A, B, C)` is like a phone book sorted by last name, then first name, then street. You can look up by last name alone, or last + first, or all three — but you can't look up by first name alone without scanning the whole book.

### Partial Index

An index over a *subset* of rows, defined by a WHERE condition. Available in PostgreSQL, SQLite, and some other databases (MySQL calls them "filtered indexes").

```sql
-- Index only unprocessed orders — the common query target
CREATE INDEX idx_orders_pending
ON orders(created_at)
WHERE status = 'PENDING';

-- Index only active users
CREATE INDEX idx_users_active_email
ON users(email)
WHERE is_active = true;
```

If 95% of your queries for orders filter by `status = 'PENDING'`, this index covers 5% of the rows but handles 95% of the queries. It's dramatically smaller, faster to maintain, and more selective than a full index on `created_at` alone.

This is an underused pattern. Any time you have a boolean or status filter that appears in the vast majority of your queries, a partial index is worth considering.

### Hash Index

Stores a hash of the indexed value rather than the value itself. Lookup is `O(1)` for exact equality — faster than B-Tree for point lookups. But hash indexes cannot be used for range queries, sorting, or prefix matching.

```sql
-- PostgreSQL
CREATE INDEX idx_sessions_token ON sessions USING HASH (token);
```

Use hash indexes only for exact equality lookups on columns that are never used in range queries or ORDER BY. Session tokens, API keys, and UUID lookups are good candidates.

### Full-Text Index

Built for searching natural language text. A B-Tree on a `description` column won't help you find all products that contain the word "wireless" — full-text indexes use inverted indexes (word → list of rows containing that word) designed specifically for this.

```sql
-- MySQL / MariaDB
CREATE FULLTEXT INDEX idx_products_description ON products(description);
SELECT * FROM products WHERE MATCH(description) AGAINST('wireless headphones' IN BOOLEAN MODE);

-- PostgreSQL (using tsvector)
CREATE INDEX idx_articles_content ON articles USING GIN(to_tsvector('english', content));
SELECT * FROM articles WHERE to_tsvector('english', content) @@ plainto_tsquery('database indexing');
```

Don't use `LIKE '%keyword%'` for text search on large tables. It defeats every index. Use full-text indexes, or a dedicated search engine (Elasticsearch, Typesense) for complex search requirements.

---

## Reading Execution Plans

Creating an index is not enough. The query optimizer may choose not to use it. You need to verify.

`EXPLAIN` (or `EXPLAIN ANALYZE` in PostgreSQL) shows you the query plan the optimizer chose:

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 101 AND status = 'PENDING';
```

### PostgreSQL Output: What to Look For

```
Index Scan using idx_orders_customer_status on orders
 (cost=0.43..8.45 rows=12 width=156)
 (actual time=0.021..0.035 rows=12 loops=1)
 Index Cond: ((customer_id = 101) AND (status = 'PENDING'))
Planning Time: 0.3 ms
Execution Time: 0.1 ms
```

**Good signs:**
- `Index Scan` or `Index Only Scan` (the latter means a covering index — no table lookup)
- Low `actual time`
- `rows` estimate close to `actual rows` (optimizer has accurate statistics)

**Warning signs:**
- `Seq Scan` on a large table — full table scan, likely missing an index
- `rows` estimate wildly off from actual rows — stale statistics, run `ANALYZE`
- `Bitmap Heap Scan` with high row counts — may benefit from a more selective index

### MySQL Output

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 101;
```

```
+----+-------------+--------+------+---------------------------+---------------------------+---------+-------+------+-------+
| id | select_type | table  | type | possible_keys             | key                       | key_len | ref   | rows | Extra |
+----+-------------+--------+------+---------------------------+---------------------------+---------+-------+------+-------+
|  1 | SIMPLE      | orders | ref  | idx_orders_customer_id    | idx_orders_customer_id    | 4       | const |   47 |       |
+----+-------------+--------+------+---------------------------+---------------------------+---------+-------+------+-------+
```

The `type` column is the most important signal. From best to worst:

| Type | Meaning |
|---|---|
| `const` / `eq_ref` | Single row lookup by unique key — optimal |
| `ref` | Index lookup returning multiple rows |
| `range` | Index range scan |
| `index` | Full index scan (better than table scan, still potentially slow) |
| `ALL` | Full table scan — investigate this on large tables |

---

## The Write Tax: What Indexes Cost

Every index is a data structure the database must keep synchronized with the table. On every write operation:

```
INSERT INTO orders (...) VALUES (...);
```

The database doesn't just write one row. It writes the row *and* updates every index defined on that table. Five indexes on a table = six write operations per insert.

For write-heavy workloads, this overhead is significant. A table with 20 indexes can have insert throughput 3–5x slower than the same table with 3 targeted indexes.

The practical implication: audit your indexes regularly. An unused index has zero read benefit and a real write cost.

```sql
-- PostgreSQL: Find unused indexes
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
 AND indexname NOT LIKE 'pg_%'
ORDER BY tablename;

-- MySQL: Find unused indexes (requires performance_schema)
SELECT object_schema, object_name, index_name
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE index_name IS NOT NULL
 AND count_star = 0
 AND object_schema NOT IN ('mysql', 'performance_schema', 'information_schema')
ORDER BY object_name;
```

Any index with zero scans since the last statistics reset is a candidate for removal — but verify by checking if it might be used during infrequent operations (monthly reports, batch jobs) before dropping it.

---

## Common Indexing Mistakes

### Indexing a Column That Has a Function Applied to It

```sql
-- Index on email exists, but this query won't use it:
SELECT * FROM users WHERE LOWER(email) = 'bob@example.com';

-- Neither will this:
SELECT * FROM orders WHERE YEAR(created_at) = 2024;
```

When you wrap a column in a function, the database can't use a standard B-Tree index on that column — it would need to apply the function to every entry in the index to find matches.

**Fix option 1:** Store the data in the canonical form you query in (store emails already lowercased).

**Fix option 2:** Use a functional index (PostgreSQL supports this natively):

```sql
-- PostgreSQL: Index the expression itself
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- Now this query uses the index:
SELECT * FROM users WHERE LOWER(email) = 'bob@example.com';
```

### Leading Wildcard LIKE

```sql
-- Cannot use a B-Tree index — leading wildcard defeats prefix traversal
SELECT * FROM products WHERE name LIKE '%wireless%';

-- CAN use a B-Tree index — leading characters allow prefix traversal
SELECT * FROM products WHERE name LIKE 'wireless%';
```

For the first pattern, use a full-text index or a dedicated search engine.

### Redundant Indexes

```sql
-- These two indexes are redundant — the composite covers the single-column use case
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_customer_status ON orders(customer_id, status);
```

The composite index on `(customer_id, status)` already handles queries that filter only on `customer_id` (leftmost prefix rule). The single-column index adds write overhead with no read benefit. Drop the narrower one.

### Ignoring Index Statistics

Query optimizers use table statistics to estimate row counts and choose plans. If statistics are stale after a large data load, the optimizer may make poor decisions — choosing a full table scan when an index would be dramatically faster.

```sql
-- PostgreSQL: Update statistics
ANALYZE orders;

-- MySQL: Update statistics
ANALYZE TABLE orders;
```

Run ANALYZE after bulk inserts or major data changes. Most databases do this automatically on a schedule, but high-churn tables may benefit from more frequent manual updates.

### The High-Cardinality Trap (Indexing Low-Cardinality Columns)

A column with only two possible values (`true`/`false`, `M`/`F`) has very low cardinality. An index on such a column rarely helps — if 50% of rows match the filter, the optimizer may correctly decide a full scan is cheaper than the index + key lookup overhead.

```sql
-- Usually not worth indexing alone
CREATE INDEX idx_users_is_active ON users(is_active);
```

The exception: use a **partial index** to index only the selective subset:

```sql
-- Index only active users — now highly selective
CREATE INDEX idx_users_active ON users(email) WHERE is_active = true;
```

---

## A Practical Indexing Workflow

Rather than guessing what to index, follow this process:

**1. Identify slow queries.** Enable slow query logging and find the worst offenders:

```sql
-- PostgreSQL: queries with most total time
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;

-- MySQL slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- Log queries > 1 second
```

**2. Run EXPLAIN ANALYZE on the worst offenders.** Look for full table scans on large tables.

**3. Design indexes based on actual query patterns.** Not based on what seems logical in the schema — based on the specific WHERE, JOIN, ORDER BY, and GROUP BY clauses in the slow queries.

**4. Create the index and measure.** Run EXPLAIN ANALYZE again. Confirm the optimizer chose the new index. Benchmark the query before and after with realistic data volumes.

**5. Monitor for unused indexes over time.** Queries change. Indexes that were useful six months ago may be dead weight today.

---

## Summary

| Scenario | Recommended Index |
|---|---|
| Exact lookup by primary key | Clustered (automatic) |
| Equality filter on unique column | Unique index |
| Frequent multi-column filter | Composite index (most selective column first) |
| Range queries or ORDER BY | B-Tree on the range/sort column |
| Query targeting a small subset of rows | Partial index |
| Exact equality only, no ranges | Hash index |
| Text search (contains, natural language) | Full-text index |
| Function applied to column in WHERE | Functional/expression index |
| All needed columns already in the index | Covering index (include all SELECT columns) |

The principle underlying all of it is the same: **an index is only useful if the query planner chooses it, and the planner only chooses it if it's more selective than a full scan**. Design indexes around real query patterns. Verify with EXPLAIN. Audit for unused indexes. Measure everything.

The engineers who master indexing aren't the ones who add indexes to every column — they're the ones who know exactly which indexes to add, which to remove, and how to verify the difference.
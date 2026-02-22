# SQL Query Optimization: How I Reduced Query Time from 3 Seconds to 50ms



![SQL Query Optimization Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*9erIl-fSJRDAmkhmCdOttA.png)

### SQL Query Optimization: A Practical Guide for Production Databases
Welcome! I’m glad you want to learn this properly. As someone who’s spent years optimizing queries for systems handling hundreds of millions of records, let me share what actually matters in production.

---

### First, Let’s Set Up Our Realistic Schema
-- We're building an e-commerce platform with millions of records

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL,
    username VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    date_of_birth DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    is_active BOOLEAN DEFAULT true,
    account_balance DECIMAL(10,2) DEFAULT 0.00
);
-- 10M+ users

CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    sku VARCHAR(50) UNIQUE NOT NULL,
    product_name VARCHAR(500) NOT NULL,
    description TEXT,
    category_id INT,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_available BOOLEAN DEFAULT true
);
-- 500K+ products

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'processing', 'completed', 'cancelled') DEFAULT 'pending',
    shipping_address TEXT,
    payment_method VARCHAR(50),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
-- 50M+ orders

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
-- 200M+ order items
```

---

### Understanding How Databases Actually Work
Before we dive into optimization, let’s understand what happens when you run a query:

**The Query Execution Process**
1.  **Parser**: Checks syntax and builds a parse tree.
2.  **Query Optimizer**: This is the brain — it generates multiple execution plans.
3.  **Query Executor**: Runs the chosen plan.
4.  **Storage Engine**: Actually fetches the data.



The optimizer uses statistics about your data to make decisions. This is where our optimization comes in.

---

### 1. Indexes: Your Primary Weapon

#### How B-Tree Indexes Work Internally
Imagine a library with millions of books. Without an index (card catalog), you’d have to walk through every aisle. That’s a full table scan.

A B-Tree index is like a well-organized card catalog:
* **Root node**: Points to ranges (“A-M” and “N-Z”)
* **Branch nodes**: More specific ranges (“A-F”, “G-M”)
* **Leaf nodes**: Actual pointers to book locations



For a b-tree index on email:
```text
Root: ['a@' to 'm@', 'n@' to 'z@']
      /           \
Branch: a@ to f@   g@ to m@
    /      \      /      \
Leaf: pointers to actual rows
```
Search complexity: $O(\log n)$ instead of $O(n)$. With 10 million records, that’s ~24 steps instead of 10 million!

#### Single Column Index
**Scenario**: Users logging into our e-commerce site

**Bad Query**:
```sql
-- No index on email
SELECT user_id, password_hash, first_name 
FROM users 
WHERE email = 'john.doe@example.com';
```
**What happens internally**: Full table scan — database reads every row (10M+ rows) checking the email.

**Optimized**:
```sql
-- First, create the index
CREATE INDEX idx_users_email ON users(email);

-- Same query, now FAST
SELECT user_id, password_hash, first_name 
FROM users 
WHERE email = 'john.doe@example.com';
```
**Internal behavior**: Database navigates the B-tree to find exactly the rows with that email in ~24 operations.

**Real-world impact**: At Netflix, login API response time dropped from 2 seconds to 50ms after indexing email.

#### Composite Index (Multi-column)
**Scenario**: Finding orders for a specific user within a date range

**Bad Query**:
```sql
-- Separate indexes on user_id and order_date
SELECT order_id, total_amount, status 
FROM orders 
WHERE user_id = 12345 
  AND order_date >= '2024-01-01';
```
**What happens**: Database might use one index (probably user_id) then filter millions of that user’s orders in memory.

**Optimized**:
```sql
-- Create composite index matching query conditions
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date);

-- Same query, now index can filter both conditions
SELECT order_id, total_amount, status 
FROM orders 
WHERE user_id = 12345 
  AND order_date >= '2024-01-01';
```
**Internal behavior**: The B-tree is organized first by `user_id`, then by `order_date` within each user. Database can jump directly to user 12345’s section, then to Jan 1, 2024.

**Real-world impact**: Amazon’s order history loads in milliseconds instead of seconds.

#### Covering Index (The Holy Grail)
**Scenario**: Dashboard showing user’s email and order count

**Bad Query**:
```sql
-- Database reads index, then fetches rows
SELECT email, COUNT(*) 
FROM users 
GROUP BY email;
```
**Optimized**:
```sql
-- Create covering index
CREATE INDEX idx_users_email_covering ON users(email) 
INCLUDE (user_id);  -- or in some DBs: CREATE INDEX ... USING (email) INCLUDE (user_id)

-- Or if your DB doesn't support INCLUDE, include needed columns in index
CREATE INDEX idx_users_email_id ON users(email, user_id);
```
**Internal behavior**: All needed data is in the index itself. Database never touches the actual table rows (avoiding “lookups”).

**Why it’s powerful**: Index pages are smaller and more cacheable than table pages. I/O is reduced dramatically.

---

### 2. SELECT * — The Performance Killer
**Scenario**: API endpoint returning user profiles

**Bad Query**:
```sql
-- Don't do this
SELECT * FROM users WHERE user_id = 12345;
```
**What’s wrong**:
1.  Returns all columns (including password_hash, sensitive data)
2.  Prevents covering index usage
3.  More network transfer
4.  More memory on application server

**Optimized**:
```sql
-- Select only needed columns
SELECT user_id, first_name, last_name, email 
FROM users 
WHERE user_id = 12345;
```
**Internal behavior**: With `SELECT *`, database might need to read the full row even if indexed. With specific columns, can potentially use covering index.

**Real-world impact**: At LinkedIn, changing `SELECT *` to specific columns reduced API response payload by 70% and improved cache hit rates.

---

### 3. WHERE Clause Efficiency

#### Cardinality and Selectivity
Key concepts:
* **Cardinality**: Number of distinct values in a column
* **Selectivity**: cardinality / total rows (higher is better for indexing)
* **High selectivity**: email (almost unique) — great for indexing
* **Low selectivity**: status (only 4 values) — poor for indexing

**Scenario**: Filtering active users in a city

**Bad Query**:
```sql
-- Low selectivity conditions first
SELECT * FROM users 
WHERE is_active = true  -- Only 2 values
  AND city = 'New York' -- Many values
  AND created_at > '2024-01-01';
```
**Optimized**:
```sql
-- Put high selectivity conditions first
SELECT * FROM users 
WHERE city = 'New York'  -- High selectivity
  AND created_at > '2024-01-01'
  AND is_active = true;  -- Low selectivity last

-- Create appropriate index
CREATE INDEX idx_users_city_date ON users(city, created_at);
```
**Internal behavior**: Optimizer uses statistics to estimate which filter eliminates most rows first.

---

### 4. JOINs vs Subqueries

#### When Subqueries Are Actually Better
**Scenario**: Find users who haven’t ordered in 2024

**Bad Approach**:
```sql
-- Correlated subquery - runs for each user
SELECT user_id, email 
FROM users u 
WHERE NOT EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.user_id = u.user_id 
      AND o.order_date >= '2024-01-01'
);
```
**Better with JOIN**:
```sql
-- JOIN with anti-join pattern
SELECT u.user_id, u.email 
FROM users u 
LEFT JOIN orders o ON u.user_id = o.user_id 
    AND o.order_date >= '2024-01-01'
WHERE o.order_id IS NULL;
```
**But sometimes subqueries win**:
**Scenario**: Get users with their latest order

```sql
-- Subquery with LIMIT 1 can be efficient
SELECT u.user_id, u.email,
    (SELECT order_date 
     FROM orders o 
     WHERE o.user_id = u.user_id 
     ORDER BY order_date DESC 
     LIMIT 1) as last_order_date
FROM users u
WHERE u.is_active = true;
```
**Internal behavior**: With proper index on `orders(user_id, order_date)`, the subquery becomes an efficient index lookup per user.

---

### 5. EXISTS vs IN Performance
**Myth**: `EXISTS` is always faster than `IN`

**Truth**: Modern optimizers often generate the same plan

**Scenario**: Find users with completed orders
```sql
-- These often perform identically with modern optimizers
SELECT * FROM users 
WHERE user_id IN (SELECT user_id FROM orders WHERE status = 'completed');

SELECT * FROM users 
WHERE EXISTS (SELECT 1 FROM orders WHERE orders.user_id = users.user_id AND status = 'completed');
```
**The exception**: When dealing with `NULL` values or large `IN` lists.

**Real production tip**: Use `IN` for small static lists, `EXISTS` for correlated subqueries.

---

### 6. LIMIT Optimization
**Scenario**: Paginating through user list

**Bad Query**:
```sql
-- Still slow even with LIMIT
SELECT * FROM users 
ORDER BY created_at DESC 
LIMIT 100 OFFSET 10000;
```
**What’s wrong**: Database still sorts all users (millions) before taking a slice.

**Optimized with Keyset Pagination**:
```sql
-- First page
SELECT user_id, username, created_at 
FROM users 
ORDER BY created_at DESC, user_id DESC 
LIMIT 100;

-- Next page (pass last_seen_created_at and last_seen_id from previous page)
SELECT user_id, username, created_at 
FROM users 
WHERE (created_at < last_seen_created_at) 
   OR (created_at = last_seen_created_at AND user_id < last_seen_id)
ORDER BY created_at DESC, user_id DESC 
LIMIT 100;
```
**Internal behavior**: With index on `(created_at, user_id)`, database only reads exactly the rows needed, no scanning.

**Real-world impact**: Facebook feed pagination uses this pattern to handle billions of posts.

---

### 7. Functions on Indexed Columns — DON’T
**Scenario**: Search users by year of birth

**Bad Query**:
```sql
-- Function on indexed column - index useless
SELECT * FROM users 
WHERE YEAR(date_of_birth) = 1990;
```
**Optimized**:
```sql
-- Rewrite to preserve index usage
SELECT * FROM users 
WHERE date_of_birth >= '1990-01-01' 
  AND date_of_birth < '1991-01-01';
```
**Internal behavior**: First query forces full table scan. Second uses index on `date_of_birth`.

---

### 8. EXPLAIN Plan — Your Best Friend
Always do this:
```sql
EXPLAIN (ANALYZE, BUFFERS) -- PostgreSQL
-- or
EXPLAIN FORMAT=JSON -- MySQL
SELECT ... your query ...
```
What to look for:
* **type**: `ALL` = full table scan (bad), `ref/range` = index used (good)
* **rows**: Estimated rows examined (lower is better)
* **Extra**: “Using index” = covering index (great), “Using where” = filtering after storage (ok)

---

### 9. Batch Operations
**Scenario**: Updating user balances

**Bad (Row-by-row)**:
```python
# N+1 queries problem
for user_id in user_ids:
    db.execute("UPDATE users SET account_balance = account_balance + 10 WHERE user_id = %s", (user_id,))
```
**Optimized**:
```sql
-- Single batch operation
UPDATE users 
SET account_balance = account_balance + 10 
WHERE user_id IN (1, 2, 3, 4, 5, ...);

-- Or for bulk inserts
INSERT INTO orders (user_id, total_amount, ...) VALUES 
(1, 100.00, ...),
(2, 200.00, ...),
(3, 150.00, ...);
```
**Internal behavior**: Single query execution plan, one network round trip, atomic operation.

---

### 10. Pagination Optimization Deep Dive
The `OFFSET` problem: `OFFSET 1000000 LIMIT 10` still reads and discards 1,000,000 rows.

**Keyset Pagination (Seek Method)**:
```sql
-- Instead of OFFSET
SELECT * FROM orders 
WHERE user_id = 123 
ORDER BY order_id DESC 
LIMIT 10 OFFSET 1000;

-- Do this (keyset pagination)
SELECT * FROM orders 
WHERE user_id = 123 
  AND order_id < last_seen_order_id  -- from previous page
ORDER BY order_id DESC 
LIMIT 10;
```
**Real-world impact**: Instagram’s infinite scroll uses this pattern.

---

### Real Production Scenarios

#### Scenario 1: Optimizing Slow Login Query
**Problem**: Login API taking 3 seconds

**Original Query**:
```sql
SELECT * FROM users 
WHERE email = 'user@example.com' 
  AND password_hash = 'hashed_pw' 
  AND is_active = true;
```
`EXPLAIN` reveals: No index on email

**Solution**:
```sql
CREATE INDEX idx_users_email_active ON users(email, is_active) 
INCLUDE (user_id, password_hash, first_name, last_name);

-- Now query uses covering index
SELECT user_id, password_hash, first_name, last_name 
FROM users 
WHERE email = 'user@example.com' 
  AND is_active = true;
```
**Result**: 3 seconds → 50ms

#### Scenario 2: E-commerce Order History
**Problem**: Order history page timing out

**Original**:
```sql
SELECT * FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.user_id = 12345
ORDER BY o.order_date DESC
LIMIT 20;
```
**Issues**:
1.  `SELECT *` returns too much
2.  No proper index
3.  Joining all items even for 20 orders

**Solution**:
```sql
-- First, create composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date DESC);

-- Optimized query
SELECT o.order_id, o.order_date, o.total_amount, o.status,
       oi.product_id, p.product_name, oi.quantity, oi.unit_price
FROM orders o
LEFT JOIN LATERAL (
    SELECT oi.product_id, oi.quantity, oi.unit_price
    FROM order_items oi
    WHERE oi.order_id = o.order_id
    LIMIT 5  -- Reasonable number of items per order
) oi ON true
LEFT JOIN products p ON oi.product_id = p.product_id
WHERE o.user_id = 12345
ORDER BY o.order_date DESC
LIMIT 20;
```
**Result**: Page loads in 200ms instead of timing out.

---

### Advanced Concepts Deep Dive

#### Cost-Based Optimizer
The optimizer assigns costs to operations:
* Sequential page read: Cost 1.0
* Random page read: Cost 4.0
* Processing a row: Cost 0.01
* Index tuple processing: Cost 0.005
It chooses the plan with lowest total cost.



#### When NOT to Use Indexes
* **Small tables**: Full table scan might be faster
* **Low cardinality columns**: Index on ‘status’ with 4 values — still reads 25% of table
* **Frequently updated columns**: Index maintenance overhead
* **In WHERE clauses with functions**: Unless you create functional indexes

#### Partitioning Large Tables
For orders table with 50M+ rows:
```sql
-- Partition by date range
CREATE TABLE orders_2024_01 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- Query automatically prunes partitions
SELECT * FROM orders 
WHERE order_date BETWEEN '2024-01-15' AND '2024-01-20';
-- Only scans Jan 2024 partition, not all 50M rows
```

---

### Design Time
* Use appropriate data types (INT for IDs, TIMESTAMP for dates, etc.)
* Index foreign key columns
* Consider partitioning for time-series data
* Plan indexes based on query patterns

### Query Writing
* Never use `SELECT *` in production
* Put most selective conditions first in `WHERE`
* Use `EXISTS` instead of `IN` for correlated subqueries
* Avoid functions on indexed columns
* Use `UNION ALL` instead of `UNION` when possible (avoids distinct sort)
* Batch operations instead of row-by-row

### Indexing Strategy
* Create composite indexes matching query patterns
* Use covering indexes for frequent queries
* Consider index column order (equality first, then range)
* Drop unused indexes (they slow down writes)
* Monitor index usage with database stats

### Performance Monitoring
* Always check `EXPLAIN` plan for new queries
* Set up slow query logging (threshold: 200ms)
* Monitor index hit rates (>99% is good)
* Regular vacuum/analyze (PostgreSQL) or optimize tables (MySQL)

### Pagination
* Use keyset pagination for deep pages
* Avoid large `OFFSET` values
* Consider cursor-based pagination for APIs

---

### Real-World Company Practices

**Netflix**:
* Use covering indexes extensively
* Partition by time for logs and events
* Denormalize for read performance

**Amazon**:
* Composite indexes on `(user_id, order_date)` for order history
* Avoid joins in critical paths (denormalize)
* Use read replicas for reporting

**LinkedIn**:
* Keyset pagination for feeds
* Covering indexes for profile views
* Batch updates for engagement metrics

**Google**:
* Massive partitioning
* Materialized views for aggregates
* Query rewriting at proxy level

---

### Final Production Tips
1.  **Start with the query, then design indexes** — Don’t index everything; index based on actual queries.
2.  **Monitor real query performance** — Use tools like `pg_stat_statements` (PostgreSQL) or `performance_schema` (MySQL).
3.  **Test with production-like data volume** — 1M vs 100M rows behave very differently.
4.  **Consider read replicas** — Move reporting/analytics queries away from primary.
5.  **Cache aggressively** — Redis/memcached for frequent, identical queries.
6.  **Regular maintenance** — Update statistics, rebuild fragmented indexes.
7.  **Use connection pooling** — Reduce connection overhead.

**Remember**: The goal isn’t to optimize every query, but to identify and fix the ones that matter — the 20% of queries that cause 80% of the performance problems.

Start with your slowest queries (check slow query log), analyze their `EXPLAIN` plans, and apply these techniques iteratively. Each optimization should have a measurable impact on response times or system resource usage.
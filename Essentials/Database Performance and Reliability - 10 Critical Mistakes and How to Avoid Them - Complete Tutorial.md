# Database Performance and Reliability: 10 Critical Mistakes That Will Wreck Your Application (And How to Avoid Them)

> **_tutorial-complete deep-dive_** | **Difficulty:** Intermediate | **⏱️ Estimated Reading Time:** 45–55 minutes | **Last Updated:** August 22, 2026

---

<img src="images/database-mindmap.png" alt="Database Mistakes Mind Map" width="100%" style="border-radius:8px" />
<p align="center"><em>Visual: Database mistakes clustered by Performance, Integrity, Design, and Operations</em></p>

---

## Table of Contents

1. [Introduction: Why Databases "Die Slowly"](#introduction-why-databases-die-slowly)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [How to Use This Tutorial](#how-to-use-this-tutorial)
5. [Mistake #1: Not Adding Indexes (or Adding Too Many)](#mistake-1-not-adding-indexes-or-adding-too-many)
6. [Mistake #2: Using `SELECT *` in Production Queries](#mistake-2-using-select--in-production-queries)
7. [Mistake #3: Not Using Transactions for Atomic Operations](#mistake-3-not-using-transactions-for-atomic-operations)
8. [Mistake #4: Ignoring Proper Schema Design](#mistake-4-ignoring-proper-schema-design)
9. [Mistake #5: Over-Normalizing Everything](#mistake-5-over-normalizing-everything)
10. [Mistake #6: Ignoring Query Execution Plans](#mistake-6-ignoring-query-execution-plans)
11. [Mistake #7: Not Handling the N+1 Query Problem](#mistake-7-not-handling-the-n1-query-problem)
12. [Mistake #8: Storing Everything in JSON Columns](#mistake-8-storing-everything-in-json-columns)
13. [Mistake #9: Skipping Backup and Restore Testing](#mistake-9-skipping-backup-and-restore-testing)
14. [Mistake #10: Not Understanding Isolation Levels and Concurrency](#mistake-10-not-understanding-isolation-levels-and-concurrency)
15. [Putting It All Together: A Decision Framework](#putting-it-all-together-a-decision-framework)
16. [Migration Guide: Auditing an Existing Codebase](#migration-guide-auditing-an-existing-codebase)
17. [Testing Strategies for Database Decisions](#testing-strategies-for-database-decisions)
18. [Security Considerations](#security-considerations)
19. [Performance Benchmarks Reference](#performance-benchmarks-reference)
20. [Best Practices Summary](#best-practices-summary)
21. [Anti-Patterns Summary](#anti-patterns-summary)
22. [Troubleshooting Guide](#troubleshooting-guide)
23. [Quick Recap](#quick-recap)
24. [Practice Exercises (with Solutions)](#practice-exercises-with-solutions)
25. [Hands-On Lab: E-Commerce Database Audit](#hands-on-lab-e-commerce-database-audit)
26. [Pro Tips](#pro-tips)
27. [Test Your Understanding](#test-your-understanding)
28. [Common Interview Questions](#common-interview-questions)
29. [Question Bank (50+ Questions)](#question-bank-50-questions)
30. [Self-Assessment Checklist](#self-assessment-checklist)
31. [Further Reading & Resources](#further-reading--resources)
32. [Learning Path Recommendations](#learning-path-recommendations)
33. [Next Steps](#next-steps)
34. [Final Thoughts](#final-thoughts)

---

## Introduction: Why Databases "Die Slowly"

> **💡 Key Insight:** Applications rarely crash because of a database explosion. Instead, they suffer a slow, creeping death.

Databases are not just storage engines — they are **consistency engines**, **performance engines**, and **scalability engines**. Yet 99% of database problems are not caused by the database engine itself (PostgreSQL, MySQL, SQL Server, MongoDB are all extremely capable). They are caused by **decisions made by developers** who didn't fully understand how the database works underneath the query.

### The Symptoms of "Death by a Thousand Cuts"

| Symptom | Root Cause Category |
|---|---|
| Queries that used to take 5ms now take 5 seconds | ⚡ Performance (Missing Indexes) |
| Random deadlocks under load | 🔒 Integrity (Concurrency / Isolation) |
| Data that doesn't add up between tables | 🏗️ Design (Schema Integrity) |
| A "simple" scaling task that turns into a multi-week migration | ⚙️ Operations (Backups, Testing) |

This tutorial expands on **10 real-world database mistakes**, why they happen, the underlying mechanics that cause them, and concrete, production-ready fixes — with diagrams, multiple examples, and real use cases for each one.

```mermaid
mindmap
  root((Database Mistakes))
    Performance
      Missing Indexes
      Too Many Indexes
      SELECT *
      N+1 Queries
    Integrity
      No Transactions
      Poor Isolation Handling
    Design
      Bad Schema
      Over-Normalization
      JSON Abuse
    Operations
      Untested Backups
      Ignoring Execution Plans
```

> **💬 Story Time:** A startup built a social platform that worked beautifully with 1,000 users. At 100,000 users, their login endpoint went from 5ms to 5 seconds. The culprit? A missing index on `email`. A one-line fix (`CREATE INDEX`) saved them from a full architectural rewrite. This is the power — and peril — of getting databases right.

---

## Prerequisites

Before diving in, you should have:

### Technical Requirements

| Skill | Minimum Level |
|---|---|
| **SQL fundamentals** | Comfortable writing `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **Relational databases** | Understand tables, rows, primary keys, foreign keys |
| **A programming language** | Familiarity with ORM concepts (Python/Django, Java/JPA, Node/TypeScript, etc.) |
| **Command-line usage** | Basic terminal/navigation skills |
| **Version control** | Basic `git` operations |

### Tools You'll Encounter (No Installation Required to Read)

- **Database engines:** PostgreSQL, MySQL, SQLite (examples use standard SQL that generalizes)
- **ORMs:** Sequelize (JavaScript), SQLAlchemy (Python), JPA/Hibernate (Java), Django ORM (Python)
- **CLI tools:** `psql`, `mysql`, `sqlite3`, `EXPLAIN` utilities

> **📝 Note:** All SQL examples are written in standard SQL and tested conceptually. Minor engine-specific syntax differences (e.g., `LIMIT` vs `TOP`, `SERIAL` vs `AUTO_INCREMENT`) are noted where relevant.

---

## Learning Objectives

By the end of this tutorial, you will be able to:

### 🎯 Core Competencies

1. **Diagnose index problems** — determine when indexes are missing, excessive, or misused using `EXPLAIN`
2. **Write efficient queries** — avoid `SELECT *` and other silent performance killers
3. **Implement transactional safety** — wrap multi-step operations in ACID transactions to prevent data corruption
4. **Design robust schemas** — model relationships correctly with foreign keys and proper normalization
5. **Balance normalization vs. denormalization** — know when to denormalize for read performance
6. **Read and interpret execution plans** — use `EXPLAIN ANALYZE` to verify query strategies
7. **Identify and fix N+1 query problems** — refactor ORM patterns using JOINs and eager loading
8. **Make informed JSON column decisions** — know when to use JSON vs. real columns
9. **Implement backup and restore testing** — verify backups work, not just run
10. **Handle concurrency correctly** — understand isolation levels, race conditions, and locking strategies

### 🚀 Professional Outcomes

- Audit any existing database schema and identify the top 3 immediate improvements
- Write production-ready queries that scale to millions of rows
- Design transaction boundaries that prevent data corruption under load
- Pass senior-level database interviews with confidence
- Lead database-related incident post-mortems effectively

---

## How to Use This Tutorial

This is a **comprehensive deep-dive**, not a quick reference. Here's the recommended approach:

1. **Read linearly** — Each mistake builds concepts used in later ones
2. **Try the exercises** — The practice problems at the end reinforce key concepts
3. **Bookmark the checklists** — The quick recap and self-assessment are useful for code reviews
4. **Use the question bank** — 50+ questions for interview prep or team training
5. **Apply the migration guide** — Audit your own codebase using the provided framework

> **🔔 Callout:** Each mistake section includes:
> - **The Problem** — What goes wrong and why
> - **Incorrect Implementation** — An anti-pattern example (❌)
> - **Correct Implementation** — A best-practice example (✅)
> - **Multiple Approaches** — Alternative solutions where they exist
> - **Real-World Use Case** — A concrete scenario from production
> - **Performance Impact** — Benchmarks or estimates
> - **Security Implications** — When security is at stake
> - **Testing Strategy** — How to verify the fix works
> > **Troubleshooting** — Common pitfalls and how to avoid them
> - **Pro Tip** — An advanced insight

---

## Mistake #1: Not Adding Indexes (or Adding Too Many)

### The Core Problem

An index is essentially a **sorted lookup structure** (usually a B-Tree) that lets the database jump directly to the rows it needs instead of scanning the entire table.

Without an index, a query like:

```sql
SELECT * FROM users WHERE email = 'john@example.com';
```

forces the database engine to check every single row, one by one, until it finds a match (or exhausts the table).

```mermaid
flowchart LR
    subgraph "Without Index — O(n)"
        A[Start Scan] --> B[Row 1]
        B --> C[Row 2]
        C --> D[Row 3]
        D --> E[...]
        E --> F[Row 1,000,000]
        F --> G[Match Found or Not]
    end
```

```mermaid
flowchart LR
    subgraph "With Index — O(log n)"
        H[Query] --> I[B-Tree Root]
        I --> J[Branch Node]
        J --> K[Leaf Node]
        K --> L[Direct Pointer to Row]
    end
```

> **🔍 Why This Matters:** On a table with 5 million rows, a sequential scan might touch 150MB of data. An index lookup touches ~30 pages (B-Tree depth of ~3). The difference is hundreds of milliseconds vs. sub-millisecond.

### Why It Happens: The Root Cause

Developers and DBAs make two opposite mistakes:

1. **Never adding indexes** — "I'll add them later" (later never comes)
2. **Adding too many indexes** — "Index everything, just in case" (write amplification)

Both stem from the same misunderstanding: **indexes are not free**. They cost storage, slow down writes, and must be maintained.

### Example 1: The Missing Index Disaster ❌

Imagine a `users` table with 5 million rows and a login endpoint that runs:

```sql
SELECT id, password_hash FROM users WHERE email = 'jane@example.com';
```

Without an index on `email`, this query does a **sequential scan** — reading all 5 million rows on every single login attempt. At 1,000 logins per minute, this alone can saturate your database CPU.

#### ❌ Incorrect Implementation

```sql
-- No index exists on email
-- Query plan:
-- Seq Scan on users  (cost=0.00..18584.00 rows=1 width=64)
--   Filter: (email = 'jane@example.com'::text)
```

**Problems:**
- Every login scans 5M rows
- CPU-bound under load
- No way to scale without hardware

#### ✅ Correct Implementation

```sql
-- Add the index
CREATE INDEX idx_users_email ON users(email);

-- Query plan now:
-- Index Scan using idx_users_email on users  (cost=0.43..8.45 rows=1 width=64)
--   Index Cond: (email = 'jane@example.com'::text)
```

Now the same query goes from potentially hundreds of milliseconds to sub-millisecond.

### Example 2: The Over-Indexing Trap ❌

A common overcorrection is indexing everything "just in case":

```sql
CREATE INDEX idx1 ON orders(status);
CREATE INDEX idx2 ON orders(created_at);
CREATE INDEX idx3 ON orders(is_deleted);
CREATE INDEX idx4 ON orders(customer_name);
CREATE INDEX idx5 ON orders(notes);
```

Every one of these indexes must be **updated on every INSERT and UPDATE**. If your `orders` table receives 500 writes/second, you're now paying an indexing tax five times over on every write, and your storage footprint balloons.

```mermaid
flowchart TD
    A[INSERT INTO orders] --> B[Write Row Data]
    B --> C[Update Index 1]
    C --> D[Update Index 2]
    D --> E[Update Index 3]
    E --> F[Update Index 4]
    F --> G[Update Index 5]
    G --> H[Commit]
    style H fill:#f96
```

> **⚠️ Write Amplification:** Each write now does 6x the work (1 row write + 5 index updates). On a table receiving 10K writes/sec, this can drop throughput from 10K to 2K writes/sec.

#### ✅ Correct Approach: Strategic Indexing

```sql
-- Only index columns actually used in WHERE, JOIN, or ORDER BY
-- for high-frequency queries
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_status_created ON orders(status, created_at) 
    WHERE is_deleted = false;  -- Partial index
```

### Decision Framework: Should I Index This Column?

| Question | If Yes → Index It |
|---|---|
| Is it used in `WHERE` clauses frequently? | ✅ |
| Is it used in `JOIN` conditions? | ✅ |
| Is it used in `ORDER BY` on large result sets? | ✅ |
| Is it a foreign key? | ✅ |
| Is it a boolean or low-cardinality field (e.g., `is_active`)? | ❌ (low selectivity — index won't help much) |
| Is it rarely queried, mostly just stored? | ❌ |

> **💡 Pro Tip #1:** A **composite index** on `(status, created_at)` can serve queries filtering by `status` alone, by `created_at` alone, or both — but only if the columns are in the right order (most selective first for single-column lookups, but both work for combined lookups).

### Multiple Approaches: Index Types

Different database engines offer specialized index types. Here's a comparison:

| Index Type | Best For | Trade-offs |
|---|---|---|
| **B-Tree** (default) | Equality, range queries, `ORDER BY` | Good all-rounder, moderate write cost |
| **Hash** | Exact equality only | Very fast reads, no range support, no ordering |
| **GIN** (PostgreSQL) | Arrays, JSON/JSONB, full-text search | Slower writes, excellent for complex data types |
| **BRIN** (PostgreSQL) | Very large tables with natural ordering | Minimal storage, poor for random access |
| **Bitmap** (MySQL/MariaDB) | Low-cardinality columns (e.g., `gender`) | Best combined with other indexes |

### Real-World Use Case

> **🏢 Case Study: Shopify** — An e-commerce platform noticed checkout was slow. `EXPLAIN ANALYZE` revealed a sequential scan on `inventory` filtering by `sku`. Adding a single index on `sku` cut checkout query time from 800ms to 2ms — a **400x improvement**, with zero code changes.

### Golden Rule

> **🔑 Always run `EXPLAIN` (or `EXPLAIN ANALYZE`) before and after adding an index. Never guess.**

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Index on wrong column order in composite index | Index not used for query | Reorder columns (most selective first, or match query prefix) |
| Index on low-cardinality column | `EXPLAIN` shows `Seq Scan` despite index | Remove index, use it only if it filters to < 5% of rows |
| Index on a column never used in queries | Wasted storage, slow writes | `DROP INDEX` after verifying no queries use it |
| Too many indexes on write-heavy table | Writes slow down dramatically | Audit with `pg_stat_user_indexes` (PostgreSQL) |
| Forgetting to `ANALYZE` after creating index | Planner doesn't know about new index | Run `ANALYZE table_name` |

### 🔒 Security Considerations

Indexes themselves don't create direct security vulnerabilities, but:
- **Information leakage**: An index on `ssn` allows an attacker who finds a SQL injection to enumerate all SSNs much faster
- **Sensitive data indexing**: Never index columns you wouldn't want searchable (e.g., don't index `password_reset_tokens`)

### 🧪 Testing Strategy

```sql
-- 1. Before adding index: capture baseline
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT id, password_hash FROM users WHERE email = 'jane@example.com';

-- 2. Add index
CREATE INDEX idx_users_email ON users(email);

-- 3. After adding index: verify improvement
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) 
SELECT id, password_hash FROM users WHERE email = 'jane@example.com';

-- 4. Monitor write impact (if table is write-heavy)
-- Run a write benchmark before/after
```

### ⚡ Performance Benchmarks

| Scenario | Sequential Scan | Indexed Lookup | Improvement |
|---|---|---|---|
| 1M rows, exact match on `email` | ~120ms | ~0.3ms | 400x faster |
| 10M rows, range query on `created_at` | ~1,200ms | ~2ms | 600x faster |
| Write-heavy table (10K writes/sec) with 5 indexes | Baseline | ~20% slower writes | Trade-off accepted for read gains |

### 🎯 Quick Recap — Mistake #1

- **Problem**: Missing indexes cause O(n) scans; too many indexes cause write amplification
- **Fix**: Add indexes on high-frequency `WHERE`/`JOIN`/`ORDER BY` columns; remove unused indexes
- **Verify**: Use `EXPLAIN` before and after
- **Rule**: Index strategically, not reactively

---

## Mistake #2: Using `SELECT *` in Production Queries

### Why It Feels Harmless But Isn't

```sql
SELECT * FROM orders WHERE user_id = 42;
```

This is quick to type, but it silently does several expensive things:

1. Pulls **every column**, even ones you'll never use in code
2. Increases **network payload size**, especially with `TEXT`, `JSON`, or `BLOB` columns
3. Slows down **serialization/deserialization** on the application side
4. Breaks **caching efficiency**, since cache keys/values become bloated
5. Makes schema changes riskier (adding a column silently changes what your app receives)

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: SELECT * FROM orders WHERE user_id = 42
    DB-->>App: id, total, status, notes, metadata_json, shipping_json, audit_log...
    Note over App,DB: App only needed 3 of these 7 fields
    App->>App: Deserialize unused data (wasted CPU)
```

> **💡 Key Insight:** The cost of `SELECT *` compounds. It's not just CPU — it's memory, network, cache, and maintainability.

### Why It Happens

- **Rapid prototyping**: "I'll refine the columns later" (later never comes)
- **ORM convenience**: Many ORMs default to selecting all columns
- **Fear of missing out**: "What if I need this field later?"

### Example 1: The Hidden Cost of Large Columns ❌

Suppose `orders` has a `shipping_label_pdf_base64` column storing a base64-encoded PDF (a real anti-pattern, but it happens). Every `SELECT *` on that table now drags megabytes of unnecessary data across the network — even for a simple "get order status" check.

#### ❌ Incorrect Implementation

```python
# Django ORM - selects ALL columns
order = Order.objects.get(user_id=42)
# Transfers: id, total, status, shipping_address, 
#   notes, metadata_json, shipping_label_pdf_base64 (2MB!), 
#   audit_log, created_at, updated_at
return JsonResponse({'status': order.status, 'total': order.total})
# ^ 95% of data transferred is unused
```

#### ✅ Correct Implementation

```python
# Django ORM - explicit column selection
order = Order.objects.values('id', 'total', 'status').get(user_id=42)
return JsonResponse({'status': order['status'], 'total': order['total']})
# ^ Minimal payload, fast serialization
```

```javascript
// Sequelize ORM - using attributes
const order = await Order.findOne({
  where: { user_id: 42 },
  attributes: ['id', 'total', 'status']  // Only these 3 columns
});
```

```java
// JPA - using constructor expression
@Query("SELECT new com.example.OrderSummary(o.id, o.total, o.status) " +
       "FROM Order o WHERE o.userId = ?1")
List<OrderSummary> findOrderSummaries(Long userId);
```

### Example 2: The Better Pattern ✅

```sql
SELECT id, total_amount, status
FROM orders
WHERE user_id = 42;
```

This is:
- **Smaller over the wire** — only 3 columns instead of 10+
- **Faster to parse** — less JSON/BSON deserialization
- **More stable** — adding new columns to the table won't silently break assumptions
- **Easier to cache** — predictable, minimal payload

### Multiple Approaches: Column Selection Patterns

| ORM/Framework | Explicit Selection Pattern |
|---|---|
| **Django** | `Model.objects.values('col1', 'col2')` |
| **SQLAlchemy** | `session.query(Model.col1, Model.col2)` |
| **Sequelize** | `{ attributes: ['col1', 'col2'] }` |
| **JPA/Hibernate** | `SELECT new DTO(...)` constructor expression |
| **ActiveRecord** | `Model.where(...).pluck(:col1, :col2)` |
| **Prisma** | `select: { col1: true, col2: true }` |
| **Raw SQL** | `SELECT col1, col2 FROM ...` |

### Use Case: High-Traffic API Endpoint

> **🏢 Case Study: Twitter** — An internal API endpoint serving a "feed" query used `SELECT *` on a `posts` table that included a `raw_html_render` column (large text blob). At 10,000 requests/minute, this alone consumed significant bandwidth. Switching to explicit column selection reduced payload size by 70% and noticeably cut p95 latency.

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Accidentally selecting `TEXT`/`JSON`/`BLOB` columns | High latency, large payloads | Audit column data types with `DESCRIBE table_name` |
| ORM auto-selecting all columns | Hidden in framework defaults | Use explicit field/attribute selection in ORM |
| Schema changes breaking app silently | New column breaks assumptions | Pin column lists in queries, add integration tests |
| Cache bloat from large payloads | Redis costs explode | Store only needed fields in cache |

### 🔒 Security Considerations

`SELECT *` can expose sensitive data:
- If you later add a `password_hash` or `ssn` column, every existing `SELECT *` query now leaks it
- API responses based on `SELECT *` may include fields you didn't intend to expose

**Best Practice:** Always use explicit column lists, and apply **field-level allowlisting** in your API layer.

### 🧪 Testing Strategy

```sql
-- 1. Identify tables with large columns
SELECT 
  table_name, 
  column_name, 
  data_type,
  character_maximum_length
FROM information_schema.columns 
WHERE table_schema = 'public'
  AND data_type IN ('text', 'json', 'jsonb', 'blob')
ORDER BY table_name;

-- 2. For each query path, measure payload size before/after
-- In application: log response sizes in staging
-- After fix: expect 30-70% reduction
```

### ⚡ Performance Benchmarks

| Scenario | `SELECT *` | Explicit Columns | Improvement |
|---|---|---|---|
| Orders table with 10 columns, 3 requested | 15KB per row | 4KB per row | 73% smaller payload |
| Posts table with `raw_html` (500KB avg) | 500KB per row | 2KB per row | 99.6% smaller |
| 10,000 req/min with `SELECT *` | 50MB/min transferred | 15MB/min transferred | 70% less bandwidth |

### When `SELECT *` Is Fine

- Ad-hoc debugging queries in a database console
- Small lookup tables with few columns (< 5 columns, < 1KB per row)
- One-off scripts, not production code paths

> **🔔 Rule:** `SELECT *` in production query paths = code smell. Treat it as such.

### 🎯 Quick Recap — Mistake #2

- **Problem**: `SELECT *` transfers unnecessary data, slows serialization, risks data exposure
- **Fix**: Always specify explicit columns in production queries
- **Verify**: Measure payload sizes before/after the change
- **Rule**: If you can't name every column you need, you're relying on luck

---

## Mistake #3: Not Using Transactions for Atomic Operations

### The Core Concept: ACID and Atomicity

A **transaction** groups multiple statements into a single all-or-nothing unit. If any step fails, everything rolls back — leaving the data exactly as it was before.

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: BEGIN
    App->>DB: UPDATE accounts SET balance = balance - 100 WHERE id = 1
    App->>DB: INSERT INTO transactions (...)
    alt Both succeed
        App->>DB: COMMIT
        DB-->>App: Data consistent ✅
    else Any step fails
        App->>DB: ROLLBACK
        DB-->>App: No partial changes ✅
    end
```

> **💡 Why This Matters:** Without transactions, a crash mid-operation leaves your data in an inconsistent state. The database has no way to know which statements were "meant" to happen together.

### Why It Happens: The Root Cause

1. **ORM convenience**: Some ORMs auto-commit each query individually
2. **Misunderstanding scope**: "These are just reads, no need for a transaction" — but you might be reading inconsistent data
3. **Fear of locking**: "Transactions are slow" — true if misused, but essential for correctness
4. **Assumption of reliability**: "My app code handles failures" — but the database can fail independently of your app

### ACID Properties Recap

| Property | Meaning | Why It Matters |
|---|---|---|
| **Atomicity** | All statements in a transaction succeed or all fail | Prevents partial updates |
| **Consistency** | Transaction brings DB from one valid state to another | Enforces business rules |
| **Isolation** | Concurrent transactions don't interfere | Prevents race conditions |
| **Durability** | Once committed, changes survive crashes | Data isn't lost |

### Example 1: The Bank Transfer Bug ❌

Without a transaction:

```sql
-- ❌ DANGEROUS: No transaction wrapper
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Server crashes here, or network drops
INSERT INTO transactions (account_id, amount, type) VALUES (1, -100, 'debit');
```

Result: the customer's balance decreased, but there's **no audit trail**. Support tickets, accounting mismatches, and angry customers follow.

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: UPDATE accounts SET balance = balance - 100 WHERE id = 1
    DB-->>App: Success ✅
    Note over App: Network drops, app crashes, or server reboots
    App->>DB: INSERT INTO transactions... (never reaches DB)
    Note over App: Balance changed, but no audit trail ❌
```

#### ✅ Correct Implementation

```sql
-- ✅ Wrap both operations in a transaction
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

INSERT INTO transactions (account_id, amount, type)
VALUES (1, -100, 'debit');

COMMIT;
```

If anything fails between `BEGIN` and `COMMIT`, the database automatically rolls back — the balance update never actually happens.

#### ✅ Application-Level Transaction (Java / Spring)

```java
@Service
@Transactional  // Spring manages BEGIN/COMMIT/ROLLBACK automatically
public class TransferService {
    
    public void transfer(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        Account from = accountRepository.findById(fromAccountId);
        Account to = accountRepository.findById(toAccountId);
        
        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
        
        accountRepository.save(from);
        accountRepository.save(to);
        
        transactionRepository.save(new Transaction(fromAccountId, -amount, "debit"));
        transactionRepository.save(new Transaction(toAccountId, amount, "credit"));
        
        // If any operation throws, @Transactional auto-rolls-back
    }
}
```

#### ✅ Application-Level Transaction (Python / Django)

```python
from django.db import transaction

@transaction.atomic  # Django manages BEGIN/COMMIT/ROLLBACK
def transfer(from_account_id, to_account_id, amount):
    from_account = Account.objects.get(id=from_account_id)
    to_account = Account.objects.get(id=to_account_id)
    
    from_account.balance -= amount
    to_account.balance += amount
    
    from_account.save()
    to_account.save()
    
    Transaction.objects.create(account_id=from_account_id, 
                               amount=-amount, type='debit')
    Transaction.objects.create(account_id=to_account_id, 
                               amount=amount, type='credit')
    
    # If any line throws, the decorator auto-rolls-back
```

#### ✅ Application-Level Transaction (Node.js / Sequelize)

```javascript
const transfer = async (fromId, toId, amount) => {
  const t = await sequelize.transaction();
  try {
    await Account.update(
      { balance: sequelize.literal(`balance - ${amount}`) },
      { where: { id: fromId }, transaction: t }
    );
    await Account.update(
      { balance: sequelize.literal(`balance + ${amount}`) },
      { where: { id: toId }, transaction: t }
    );
    await Transaction.create(
      { account_id: fromId, amount: -amount, type: 'debit' },
      { transaction: t }
    );
    await Transaction.create(
      { account_id: toId, amount: amount, type: 'credit' },
      { transaction: t }
    );
    await t.commit();
  } catch (error) {
    await t.rollback();
    throw error;
  }
};
```

### Example 2: Multi-Table Signup Flow ❌→✅

#### ❌ Incorrect Implementation

```sql
-- ❌ Each statement commits independently
INSERT INTO users (email, password_hash) VALUES ('a@b.com', '...');
INSERT INTO user_profiles (user_id, display_name) VALUES (currval('users_id_seq'), 'Alex');
INSERT INTO user_settings (user_id, theme) VALUES (currval('users_id_seq'), 'dark');
-- If the 3rd statement fails, we have a user with no profile or settings
```

#### ✅ Correct Implementation

```sql
BEGIN;

INSERT INTO users (email, password_hash) VALUES ('a@b.com', '...');
INSERT INTO user_profiles (user_id, display_name) VALUES (currval('users_id_seq'), 'Alex');
INSERT INTO user_settings (user_id, theme) VALUES (currval('users_id_seq'), 'dark');

COMMIT;
```

If the `user_settings` insert fails (e.g., a constraint violation), you don't want a "half-created" user with no profile or settings. The transaction guarantees this can't happen.

### Multiple Approaches: Transaction Patterns

| Pattern | When to Use | Trade-offs |
|---|---|---|
| **Declarative (`@Transactional`)** | Most business operations | Automatic, clean, but less control |
| **Programmatic (`BEGIN`/`COMMIT`)** | Fine-grained control needed | More boilerplate, explicit |
| **Savepoints** | Need to rollback part of a transaction | Adds complexity, use sparingly |
| **Compensating Transactions** | Distributed systems (Sagas) | Eventual consistency, not ACID |

> **🔔 Callout:** Not every query needs a transaction. Read-only queries at `READ COMMITTED` isolation are fine without explicit transaction blocks in most frameworks. But **multi-statement writes that must be consistent always need one.**

### Use Case: E-Commerce Checkout

A checkout process typically involves:
1. Deducting inventory
2. Creating an order record
3. Charging payment
4. Creating an invoice

If step 3 fails and steps 1–2 already committed independently, you've sold inventory that was never actually paid for. Wrapping all database writes in a transaction (with payment handled carefully, often via a two-phase approach with an external payment gateway) prevents this class of bug.

```mermaid
flowchart TD
    A[Checkout Process] --> B[Deduct Inventory]
    B --> C[Create Order Record]
    C --> D[Process Payment]
    D --> E[Create Invoice]
    E --> F{All Succeeded?}
    F -->|Yes| G[COMMIT — everything is consistent]
    F -->|No| H[ROLLBACK — no partial changes]
    style G fill:#9f6
    style H fill:#f66
```

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Forgetting `COMMIT` | Transaction hangs, locks held indefinitely | Set a timeout or always use framework-managed transactions |
| Transaction too large | Lock contention, deadlocks | Keep transactions short — do heavy computation outside |
| Mixing transaction boundaries with ORM sessions | Stale data, unexpected commits | Be explicit about transaction scope |
| Nested transactions | Unexpected behavior, performance issues | Use savepoints or refactor to single transaction |
| Auto-commit mode | Each statement is its own transaction | Explicitly manage transactions for multi-step ops |

### 🔒 Security Considerations

Transactions protect against data corruption, which is a security concern:
- **Financial integrity**: Unwrapped multi-step writes can lead to money being lost or created
- **Audit trail completeness**: Partial commits can create gaps in audit logs
- **Authorization consistency**: A user might gain access to something they shouldn't if a transaction partially commits

### 🧪 Testing Strategy

```java
// Integration test: verify rollback on failure
@Test
void transfer_rollsBackOnFailure() {
    // Given: insufficient funds
    Account from = new Account(1L, BigDecimal.valueOf(50));  // Only $50
    Account to = new Account(2L, BigDecimal.valueOf(0));
    BigDecimal transferAmount = BigDecimal.valueOf(100);     // $100 > $50

    // When: attempt transfer (should fail)
    assertThrows(InsufficientFundsException.class, () -> 
        transferService.transfer(1L, 2L, transferAmount)
    );

    // Then: balances are unchanged
    assertThat(accountRepository.findById(1L).getBalance())
        .isEqualTo(BigDecimal.valueOf(50));
    assertThat(accountRepository.findById(2L).getBalance())
        .isEqualTo(BigDecimal.valueOf(0));
    
    // And: no transaction record was created
    assertThat(transactionRepository.count()).isEqualTo(0);
}
```

### ⚡ Performance Benchmarks

| Scenario | Non-Transactional | Transactional | Trade-off |
|---|---|---|---|
| 2-statement transfer, low contention | 0.5ms | 0.6ms | +20% overhead (negligible) |
| 5-statement signup, low contention | 2.5ms | 2.8ms | +12% overhead |
| High contention (100 concurrent) | Deadlocks frequent | 1 deadlock, auto-retried | Correctness > speed |

> **💡 Pro Tip #2:** Keep transactions **short** — hold locks for the minimum time possible. Do validation, API calls, and heavy computation outside the transaction block; only include the actual database writes.

### 🎯 Quick Recap — Mistake #3

- **Problem**: Multi-step writes without transactions leave data inconsistent on failure
- **Fix**: Wrap related writes in `BEGIN`/`COMMIT` (or framework equivalent)
- **Verify**: Write integration tests that simulate failures mid-transaction
- **Rule**: If two or more writes must both succeed or both fail together, they belong inside a transaction. No exceptions.

---

## Mistake #4: Ignoring Proper Schema Design

### Why Schema Design Is Architecture, Not an Afterthought

A bad schema doesn't just look ugly — it actively fights every future feature you try to build.

```mermaid
erDiagram
    USERS {
        int id PK
        string name
        string email
    }
    ROLES {
        int id PK
        string name
    }
    USER_ROLES {
        int user_id FK
        int role_id FK
    }
    USERS ||--o{ USER_ROLES : "has"
    ROLES ||--o{ USER_ROLES : "assigned_to"
```

### Why It Happens: The Root Cause

1. **Rushing to ship**: "I'll fix the schema later" (schema refactor > app code refactor)
2. **Treating DB as a dumb key-value store**: "Just put the data in, I'll parse it in app code"
3. **Copy-pasting from tutorials**: No understanding of what each column type implies
4. **Fear of constraints**: "Constraints slow things down" — false, they prevent corruption

### Anti-Pattern Example: Comma-Separated Values ❌

```sql
-- ❌ DON'T DO THIS
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  roles VARCHAR(200)  -- stores: 'admin,editor,viewer'
);

INSERT INTO users (name, roles) VALUES ('Alice', 'admin,editor,viewer');
INSERT INTO users (name, roles) VALUES ('Bob', 'admin,viewer');
INSERT INTO users (name, roles) VALUES ('Carol', 'viewer');
```

This looks convenient until you try to answer: *"Give me all users with the 'editor' role."*

```sql
-- ❌ Terrible query — slow, fragile, wrong
SELECT * FROM users WHERE roles LIKE '%editor%';
```

**Problems:**
- **Performance**: Can't use an index (full table scan)
- **Accuracy**: `%editor%` matches "co-editor", "sub-editor", etc.
- **Integrity**: Nothing prevents `roles = 'admin,editor,editor,viewer'` (duplicates)
- **Maintenance**: Adding/removing a role requires string manipulation in app code

```mermaid
flowchart TD
    A[Query: Find editors] --> B[LIKE '%editor%' scan]
    B --> C[False match: co-editor ❌]
    B --> D[False match: sub-editor ❌]
    B --> E[Real match: editor ✅]
    B --> F[False miss: editor (trailing)]
```

### The Fix: Proper Relational Modeling ✅

```sql
-- ✅ CORRECT: Normalized relational design
CREATE TABLE roles (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE user_roles (
  user_id INT REFERENCES users(id),
  role_id INT REFERENCES roles(id),
  PRIMARY KEY (user_id, role_id)
);

-- Now the query is simple, indexable, and fast:
SELECT u.name
FROM users u
JOIN user_roles ur ON u.id = ur.user_id
JOIN roles r ON ur.role_id = r.id
WHERE r.name = 'editor';
```

### Example 2: Missing Foreign Keys ❌

Without foreign key constraints, nothing stops this:

```sql
-- ❌ No FK constraint — orphaned records
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT,  -- No FK constraint!
  total DECIMAL(10,2)
);

INSERT INTO orders (user_id, total) VALUES (999999, 50.00);
-- user_id 999999 doesn't exist! Silent data corruption.
```

You now have an **orphaned record** — an order with no valid owner. With a foreign key constraint:

```sql
-- ✅ Foreign key enforces referential integrity
ALTER TABLE orders
ADD CONSTRAINT fk_orders_user
FOREIGN KEY (user_id) REFERENCES users(id);

-- Now this fails with a clear error:
-- ERROR: insert or update on table "orders" violates foreign key constraint
INSERT INTO orders (user_id, total) VALUES (999999, 50.00);
```

> **💡 Pro Tip #3:** Foreign keys add a small overhead (checking the constraint on write), but they **cannot be bypassed by application bugs**. They are your last line of defense against data corruption.

### Multiple Approaches: Schema Design Patterns

| Pattern | When to Use | Trade-offs |
|---|---|---|
| **Fully Normalized** (3NF+) | Data integrity is paramount, write-heavy | More joins needed for reads |
| **Denormalized** | Read-heavy, pre-computed aggregates | Write complexity, potential inconsistency |
| **Entity-Attribute-Value (EAV)** | Highly variable schemas | Very slow queries, hard to validate |
| **Single Table** (all fields nullable) | Prototyping, unknown schema | Wasted space, no constraints |
| **JSON/HStore Columns** | Semi-structured data | No type safety, harder to query |

### Checklist for Good Schema Design

| Check | Why It Matters |
|---|---|
| ✅ Every relationship modeled with foreign keys | Prevents orphaned records |
| ✅ Consistent naming (`snake_case` or `camelCase`, pick one) | Reduces cognitive load |
| ✅ No repeated/duplicated data across tables without reason | Prevents update anomalies |
| ✅ Appropriate data types (don't store numbers as `VARCHAR`) | Type safety, performance |
| ✅ Constraints (`NOT NULL`, `UNIQUE`, `CHECK`) enforced at DB level | Defense in depth |
| ✅ Indexes match query patterns | Performance |
| ✅ Column names are descriptive, not abbreviated | Maintainability |

### Use Case: Multi-Tenant SaaS

> **🏢 Case Study: A SaaS product stored `company_name` as a plain text field on every table instead of a `company_id` foreign key. When a company renamed itself, engineers had to run risky bulk `UPDATE` statements across a dozen tables, with no guarantee of catching every row. A proper foreign key design would have made this a single-row update.**

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| CSV in a column instead of junction table | `LIKE` queries, data duplication | Normalize into separate tables with FK |
| No foreign key constraints | Orphaned records, no data integrity | Add `REFERENCES` constraints |
| Storing numbers as strings | Can't do arithmetic, wrong sort order | Use proper numeric types |
| No `NOT NULL` on required fields | `NULL` pollution, app crashes | Add `NOT NULL` constraints |
| Inconsistent naming | Confusion, bugs | Establish and enforce a naming convention |
| Over-abbreviated column names | Unreadable queries | Use descriptive names |

### 🔒 Security Considerations

Schema design affects security:
- **Data leakage**: Storing `company_name` as text on every table means a partial breach exposes all tenant names
- **Privilege escalation**: Without proper constraints, users can create relationships that bypass ownership checks
- **Injection surface**: Poor schema design (e.g., dynamic column names from user input) increases SQL injection risk

### 🧪 Testing Strategy

```sql
-- 1. Test foreign key enforcement
INSERT INTO orders (user_id, total) VALUES (999999, 50.00);
-- Should fail: ERROR: violates foreign key constraint

-- 2. Test data types
INSERT INTO orders (total) VALUES ('not-a-number');
-- Should fail: ERROR: invalid input for type numeric

-- 3. Test constraints
INSERT INTO users (email) VALUES (NULL);
-- Should fail if email has NOT NULL constraint

-- 4. Integration test: verify orphaned records can't exist
-- After deleting a user, verify their orders are also removed (ON DELETE CASCADE)
-- or blocked (ON DELETE RESTRICT)
```

### ⚡ Performance Benchmarks

| Schema Design | Query Complexity | Performance at 1M Rows |
|---|---|---|
| Normalized (proper joins, indexed FKs) | 2-3 joins | ~5ms |
| CSV in column (LIKE queries) | 1 table scan | ~5,000ms |
| Missing FK constraints | N/A (data integrity issue) | Risk of corruption |
| Denormalized (pre-computed) | 0 joins | ~1ms (but write overhead) |

### 🎯 Quick Recap — Mistake #4

- **Problem**: Poor schema design creates data integrity issues, slow queries, and maintenance nightmares
- **Fix**: Model relationships with foreign keys, use proper data types, enforce constraints at the DB level
- **Verify**: Write tests that verify constraint enforcement
- **Rule**: Your schema is your data's DNA — design it with care

---

## Mistake #5: Over-Normalizing Everything

### The Trade-Off

Normalization (breaking data into smaller, non-redundant tables) is great for **data integrity** but can hurt **read performance** when taken too far.

```mermaid
flowchart TD
    A[orders] --> B[order_items]
    B --> C[item_details]
    C --> D[item_categories]
    D --> E[item_category_mappings]
    E --> F[item_tags]
    F --> G[item_tag_mappings]
    style A fill:#bbf
    style G fill:#f99
```

A single "show me this order" query now requires 6 joins.

### Why It Happens: The Root Cause

1. **Academic influence**: "Normalization is always good" — taught without context of read patterns
2. **Premature optimization in the wrong direction**: "I'll normalize for flexibility" — but read performance suffers
3. **Fear of data inconsistency**: "If I duplicate, it might get stale" — true, but often acceptable for read models

### Example: The Painful Query ❌

```sql
-- ❌ 6-table join for a simple order display
SELECT o.id, oi.quantity, id.name, ic.category_name, it.tag_name
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN item_details id ON oi.item_id = id.id
JOIN item_categories ic ON id.category_id = ic.id
JOIN item_category_mappings icm ON ic.id = icm.category_id
JOIN item_tags it ON icm.tag_id = it.id;
```

Every join adds computational overhead. On large tables, this compounds quickly.

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: SELECT ... FROM orders JOIN order_items JOIN item_details JOIN ... (6 joins)
    Note over DB: Planner evaluates 6 tables, picks join order
    Note over DB: If any table lacks an index, performance degrades exponentially
    DB-->>App: Rows materialized, filtered, joined
    Note over App: High latency, CPU on join computation
```

### The Fix: Strategic Denormalization ✅

For **read-heavy** workloads (like an order summary page viewed thousands of times per minute), maintain a **denormalized summary table** updated whenever the underlying data changes:

```sql
-- ✅ Denormalized read model
CREATE TABLE order_summary (
  order_id INT PRIMARY KEY,
  item_names TEXT[],       -- pre-computed array
  category_names TEXT[],
  total_amount DECIMAL,
  updated_at TIMESTAMP
);
```

```mermaid
flowchart LR
    subgraph "Write Path (normalized, safe)"
        A[orders] --> B[order_items]
        B --> C[item_details]
    end
    subgraph "Read Path (denormalized, fast)"
        D[order_summary]
    end
    B -.trigger/update.-> D
    D --> E[Fast Single-Query Reads]
```

```sql
-- ✅ Read is now a single, fast query
SELECT * FROM order_summary WHERE order_id = 42;
```

### Multiple Approaches: Read Pattern Optimization

| Approach | When to Use | Trade-offs |
|---|---|---|
| **Materialized View** | Complex joins, acceptable staleness | Auto-refresh, but can be stale |
| **Summary Table + Trigger** | Real-time data needed | Write overhead, maintenance |
| **CQRS (Read/Write Model Separation)** | Complex domains, different read/write needs | Architectural complexity |
| **Query Result Caching** | Infrequently changing data | Cache invalidation complexity |
| **Full Denormalization** | Simple domain, high read volume | Significant write complexity |

### When to Denormalize — Decision Matrix

| Factor | Denormalize? | Reason |
|---|---|---|
| Data changes rarely | ✅ Yes | Stale reads acceptable |
| Read volume is 100x+ write volume | ✅ Yes | Read performance matters more |
| Data changes frequently and must be consistent | ❌ No | Risk of stale reads |
| Strong consistency required | ❌ No | Use normalized or materialized views with short refresh |
| Simple domain (1-3 tables) | ❌ No | Joins are cheap, denormalization unnecessary |
| Complex domain (6+ tables) | ✅ Consider | Join overhead significant |

### Use Case: Dashboard Analytics

> **🏢 Case Study: A reporting dashboard that aggregates sales by category was running 6-table joins on every page load, taking 4+ seconds. The team introduced a nightly (or event-driven) materialized summary table, dropping load time to under 100ms — at the cost of slightly stale data (acceptable for a dashboard, unacceptable for a live inventory count).**

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Over-normalizing without measuring | Slow reads, complex queries | Measure query performance, denormalize read hot paths |
| Under-normalizing | Data inconsistency, update anomalies | Add constraints, normalize critical data |
| No strategy for keeping denormalized data fresh | Stale reads, confusion | Use triggers, events, or scheduled jobs |
| Denormalizing everything | Maintenance nightmare | Denormalize selectively, for specific read patterns |
| Forgetting to index join columns | Joins are slow despite denormalization | Index all foreign key columns |

### 🔒 Security Considerations

Over-normalization can create security blind spots:
- **Access control complexity**: With 6 tables, it's harder to ensure every query applies the right tenant filter
- **Data leakage**: A missing join can expose data across tenants
- **Audit trail fragmentation**: Changes spread across many tables make auditing harder

### 🧪 Testing Strategy

```sql
-- 1. Measure baseline query performance
EXPLAIN (ANALYZE, BUFFERS) 
SELECT o.id, oi.quantity, id.name 
FROM orders o 
JOIN order_items oi ON o.id = oi.order_id 
JOIN item_details id ON oi.item_id = id.id;

-- 2. After introducing summary table, verify correctness
-- Compare results from normalized query vs. denormalized query
SELECT * FROM order_summary WHERE order_id = 42;
-- Should match the 6-table join result

-- 3. Test staleness
-- Insert/update in normalized tables, verify summary table updates
-- within acceptable time window
```

### ⚡ Performance Benchmarks

| Approach | Query Time at 1M Rows | Maintenance Overhead |
|---|---|---|
| 6-table normalized join | 4,200ms | None |
| Denormalized summary table | 2ms | Trigger + write overhead |
| Materialized view (daily refresh) | 1ms | Daily batch job |
| CQRS read model | 1ms | Event handler complexity |

> **💡 Pro Tip #4:** **Normalize for correctness. Denormalize deliberately, and only where you've measured a real performance need.** Never denormalize "just in case."

### 🎯 Quick Recap — Mistake #5

- **Problem**: Over-normalization causes excessive joins, slowing read-heavy workloads
- **Fix**: Denormalize strategic read paths into summary tables, views, or materialized views
- **Verify**: Measure query times before and after denormalization
- **Rule**: Denormalize only after measuring, and always have a strategy to keep denormalized data fresh

---

## Mistake #6: Ignoring Query Execution Plans

### Why "It Looks Fine" Isn't Enough

A query can look perfectly reasonable in SQL and still be catastrophically slow underneath. The only way to know for sure is to look at the **execution plan** — the database's actual strategy for running your query.

```sql
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'john@example.com';
```

```mermaid
flowchart TD
    A[Run EXPLAIN ANALYZE] --> B{Seq Scan on users?}
    B -->|Yes| C[⚠️ Full Table Scan Detected]
    B -->|No| D[✅ Index Scan / Index-Only Scan]
    C --> E[Add or Fix Index]
    D --> F[Query Is Optimized]
```

> **💡 Why This Matters:** The query planner doesn't always pick the "obvious" plan. Factors like stale statistics, outdated indexes, and join cardinality can lead it to make poor choices. `EXPLAIN` is your window into the planner's decision-making.

### Why It Happens: The Root Cause

1. **Lack of training**: Many developers never learned to read execution plans
2. **"It works in dev" syndrome**: Small dev datasets hide plan problems that explode at scale
3. **Fear of complexity**: `EXPLAIN` output looks intimidating (especially JSON format)
4. **No CI/CD gating**: No automated checks for query plan regressions

### Understanding EXPLAIN Output

Let's break down a typical PostgreSQL `EXPLAIN ANALYZE` output:

```sql
-- Query:
SELECT u.name, o.total FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.email = 'john@example.com';

-- EXPLAIN ANALYZE output:
Index Scan using idx_users_email on users u  (cost=0.43..8.45 rows=1 width=64) (actual time=0.025..0.026 rows=1 loops=1)
  Index Cond: (email = 'john@example.com'::text)
  ->  Nested Loop  (cost=0.43..16.73 rows=5 width=72) (actual time=0.040..0.060 rows=5 loops=1)
        ->  Index Scan using idx_orders_user_id on orders o  (cost=0.42..8.44 rows=5 width=8) (actual time=0.015..0.035 rows=5 loops=1)
              Index Cond: (user_id = u.id)
        ->  Seq Scan on shipping_addresses sa  (cost=0.00..1.00 rows=1 width=4) (actual time=0.005..0.006 rows=1 loops=5)
              Filter: (sa.order_id = o.id)
              Rows Removed by Filter: 995
```

> **🔑 Reading the Plan:** Read bottom-up and right-to-left. The innermost operations execute first.

### Reading a Bad Plan ❌

```
Seq Scan on users  (cost=0.00..18584.00 rows=1 width=64)
  Filter: (email = 'john@example.com'::text)
```

`Seq Scan` on a table with millions of rows means every row is being checked. This is your signal to add an index.

```mermaid
flowchart TD
    A[Bad Plan: Seq Scan] --> B{Table has >1M rows?}
    B -->|Yes| C[Missing Index]
    B -->|No| D[Acceptable for small tables]
    C --> E[CREATE INDEX on WHERE column]
```

### Reading a Good Plan ✅

```
Index Scan using idx_users_email on users
  (cost=0.43..8.45 rows=1 width=64)
  Index Cond: (email = 'john@example.com'::text)
```

`Index Scan` with a low cost estimate means the database is using your index efficiently.

### What to Watch For: Execution Plan Red Flags

| Red Flag | Meaning | Action |
|---|---|---|
| `Seq Scan` on large tables (>100K rows) | Missing or unused index | Add index on `WHERE`/`JOIN` column |
| `Nested Loop` with high row counts | Inefficient join strategy | Check if join column is indexed; consider `Hash Join` or `Merge Join` |
| Large gap between estimated vs. actual rows | Stale statistics | Run `ANALYZE table_name` |
| `Sort` operations on large datasets | Consider an index matching `ORDER BY` | Add composite index or pre-sort |
| `Bitmap Heap Scan` with high `Recheck Cond` | Many rows match, but not enough for full table scan | Consider a more selective index |
| `Parallelism` not used (DOP=1) | Missed opportunity for large scans | Enable parallel query execution |
| `Buffers: shared hit` very high | Data already in memory, good | Monitor hit ratio; ensure adequate RAM |
| `Buffers: shared read` very high | Disk I/O bottleneck | Add more RAM, tune `shared_buffers` |

### Multiple Approaches: Reading Plans Across Engines

| Engine | Command | Output Format | Notes |
|---|---|---|---|
| **PostgreSQL** | `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` | Tree + JSON | Most detailed; shows actual vs. estimated |
| **MySQL** | `EXPLAIN FORMAT=JSON` | JSON tree | Shows cost estimates; older `EXPLAIN` shows tabular |
| **SQL Server** | Include Actual Execution Plan (Ctrl+M) | Graphical + XML | Shows I/O stats, missing index hints |
| **SQLite** | `EXPLAIN QUERY PLAN` | Simple text | Lightweight but less detail |

### Example 1: The Nested Loop Problem ❌

```sql
-- Query that looks fine:
SELECT c.name, o.total, oi.quantity, p.name as product_name
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE c.email = 'customer@example.com';

-- Bad plan:
Nested Loop  (cost=1000..50000) (actual time=10..5000ms)
  -> Seq Scan on customers c  (cost=0.00..1000..18584.00)
       Filter: (email = 'customer@example.com')
  -> Seq Scan on orders o  (cost=0.00..1000..50000.00)
  -> Seq Scan on order_items oi  (cost=0.00..1000..50000.00)
  -> Seq Scan on products p  (cost=0.00..1000..50000.00)
```

**Root cause:** Missing indexes on `customer.email`, `orders.customer_id`, `order_items.order_id`, `products.id`. The planner falls back to sequential scans, and with 4 tables cross-joined, the row count explodes.

### Example 1 Fix: Add Missing Indexes ✅

```sql
CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_products_id ON products(id);  -- usually PK already indexed

-- New plan (good):
Hash Join  (cost=10..50) (actual time=0.1..0.5ms)
  -> Index Scan using idx_customers_email on customers c
  -> Hash  (cost=5..20)
       -> Hash Join on orders o, order_items oi
```

### Example 2: The Stale Statistics Problem ❌

After a bulk data import that doubled the table size, the planner still thinks the table has the old row count:

```
Index Scan on orders  (cost=0.43..8.45 rows=1 width=64) (actual time=50..5000ms)
  Index Cond: (status = 'pending')
  -- Planner estimated 1 row, but 50,000 rows matched!
```

**Root cause:** Statistics not refreshed after bulk import.

### Example 2 Fix: Refresh Statistics ✅

```sql
ANALYZE orders;  -- or VACUUM ANALYZE for PostgreSQL

-- After ANALYZE, new plan:
Bitmap Heap Scan on orders  (cost=10..500 rows=50000 width=64) (actual time=10..50ms)
  Recheck Cond: (status = 'pending')
```

### Real-World Use Case: The "Mystery Slowdown"

> **🏢 Case Study:** A team noticed a query that ran fast in staging (small dataset) but timed out in production (large dataset). `EXPLAIN ANALYZE` revealed the planner was choosing a nested loop join because table statistics were stale after a large bulk import. Running `ANALYZE users;` let the planner make a better decision, fixing the issue without any code change.

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Reading plans top-down instead of bottom-up | Misunderstanding execution order | Remember: read bottom-up, right-to-left |
| Ignoring cost vs. actual time | Missing real bottlenecks | Compare `cost=` vs. `actual time=` |
| Not checking buffer hit/miss ratio | Missing I/O issues | Look at `Buffers:` line for hits vs. reads |
| Running EXPLAIN without ANALYZE | Getting estimates, not reality | Use `EXPLAIN (ANALYZE)` for actual runtime |
| Not refreshing statistics | Planner makes bad choices | Schedule regular `ANALYZE` or use `auto_analyze` |
| Overlooking parallel query opportunities | Wasting CPU cores | Check if `Workers Planned` = 0; tune `max_parallel_workers` |

### 🔒 Security Considerations

Execution plans can leak information:
- **Timing attacks**: Slow queries on certain data can reveal existence of records
- **Plan instability**: An attacker can craft queries that force bad plans (e.g., `LIKE '%term%'` on unindexed columns), causing DoS
- **Plan caching**: Reused plans for queries with wildly different parameters can cause performance issues

**Best Practice:** Use parameterized queries, monitor plan regressions, and implement query timeout limits.

### 🧪 Testing Strategy

```sql
-- 1. Capture baseline plan for critical queries
CREATE TABLE query_plan_baseline AS
SELECT 'user_login' as query_name,
       pg_get_expr(query, 0) as sql_text,
       -- Store the EXPLAIN output as text
       (EXPLAIN (BUBUFFERS, FORMAT JSON) 
        SELECT id, password_hash FROM users WHERE email = 'john@example.com')
       as plan
FROM pg_prepared_statements 
WHERE statement LIKE '%users WHERE email%';

-- 2. In CI/CD: compare new plan against baseline
-- (This requires custom infrastructure, but tools like pgreplay exist)

-- 3. Manual regression check (PostgreSQL)
PREPARE user_login AS SELECT id FROM users WHERE email = $1;
EXPLAIN (ANALYZE, BUFFERS) EXECUTE user_login('test@example.com');
-- Compare against last week's plan output
```

### ⚡ Performance Benchmarks

| Query Type | Bad Plan Time | Optimized Plan Time | Improvement |
|---|---|---|---|
| Customer orders lookup (1M rows) | 5,000ms | 5ms | 1000x |
| Dashboard aggregation | 8,000ms | 200ms | 40x |
| Bulk import verification | 30,000ms | 8,000ms | 3.75x |
| Ad-hoc report | 15,000ms | 500ms | 30x |

> **💡 Pro Tip #5:** Use `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` in PostgreSQL to get machine-readable output. You can then diff two plan outputs to detect regressions automatically.

### 🎯 Quick Recap — Mistake #6

- **Problem**: Queries that look fine in SQL can have catastrophic execution plans
- **Fix**: Always run `EXPLAIN ANALYZE` on slow queries; look for `Seq Scan` on large tables, poor join strategies, stale statistics
- **Verify**: Compare actual vs. estimated row counts; check buffer hit/miss ratios
- **Rule**: If you can't explain why your query plan looks the way it does, optimize until you can

---

## Mistake #7: Not Handling the N+1 Query Problem

### What N+1 Looks Like

This is one of the most common ORM pitfalls:

```javascript
const users = await db.users.findAll();       // 1 query

for (const user of users) {
  const orders = await db.orders.findAll({     // N queries (one per user)
    userId: user.id
  });
}
```

With 100 users, this executes **101 separate database round-trips**.

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: SELECT * FROM users
    DB-->>App: 100 users
    loop For each of the 100 users
        App->>DB: SELECT * FROM orders WHERE user_id = ?
        DB-->>App: orders for that user
    end
    Note over App,DB: Total: 101 queries instead of 1-2
```

> **💡 Why This Matters:** Each round-trip adds network latency (even on localhost, ~0.1ms; on cloud DBs, 1-5ms). With 100 round-trips at 2ms each, that's 200ms of pure latency overhead — before any actual work is done.

### Why It Happens: The Root Cause

1. **ORM defaults**: Many ORMs lazy-load by default, making it the path of least resistance
2. **Iterative development**: "Make it work, then make it fast" — N+1 creeps in during feature development
3. **Object-oriented thinking**: "I need a User object with its Orders" — but relational queries don't work that way
4. **No query count monitoring**: No alerts on N+1 patterns in dev or staging

### Example 1: The Naive Loop ❌

```python
# Django ORM - classic N+1 pattern
users = User.objects.all()  # 1 query: SELECT * FROM users

for user in users:
    print(f"{user.name}: {user.orders.count()} orders")  # N queries
    for order in user.orders.all():  # N more queries
        print(f"  - {order.total}")
# With 1000 users, each with 5 orders:
# 1 + 1000 + 5000 = 6,001 queries!
```

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: SELECT * FROM users
    DB-->>App: 1,000 users
    loop 1000 times
        App->>DB: SELECT COUNT(*) FROM orders WHERE user_id = ?
    end
    loop 5000 times
        App->>DB: SELECT * FROM orders WHERE user_id = ?
    end
    Note over App,DB: 6,001 total queries 😱
```

### Example 1 Fix: JOIN-Based Query ✅

```python
# Django ORM - using select_related and prefetch_related
users = User.objects.prefetch_related('orders')  # 2 queries total

for user in users:
    for order in user.orders.all():  # Served from prefetched cache
        print(f"{user.name}: {order.total}")
# 2 queries instead of 6,001!
```

### Example 1 Fix: Single JOIN Query ✅

```sql
-- Single efficient query with JOIN
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
ORDER BY u.id;

-- Results can be grouped in application code:
-- User 1: [Order A, Order B]
-- User 2: [Order C]
-- etc.
```

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: SELECT u.*, o.* FROM users u LEFT JOIN orders o ON u.id = o.user_id
    DB-->>App: All users with their orders in one response
    Note over App: 1 query total ✅
```

### Example 2: ORM Eager Loading ✅

Most ORMs support this pattern natively. Here are examples across frameworks:

```javascript
// Sequelize - eager loading
const users = await User.findAll({
  include: [{ model: Order }]  // Generates a JOIN, fetches all at once
});
```

```java
// JPA/Hibernate - JOIN FETCH
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllUsersWithOrders();
```

```python
# SQLAlchemy - joinedload or selectinload
users = session.query(User).options(
    joinedload(User.orders)  # JOIN-based eager loading
).all()

# Or selectinload (separate query with IN clause - often better for many related)
users = session.query(User).options(
    selectinload(User.orders)  # SELECT ... WHERE user_id IN (1,2,3,...)
).all()
```

```ruby
# ActiveRecord (Ruby on Rails) - eager loading
User.includes(:orders).each do |user|
  user.orders.each { |order| puts order.total }
end
```

```go
// GORM (Go) - Preload
var users []User
db.Preload("Orders").Find(&users)
```

### Multiple Approaches: Solving N+1

| Approach | How It Works | When to Use | Trade-offs |
|---|---|---|---|
| **JOIN query** | Combine tables in single SQL query | Simple parent-child relationships | Cartesian product can blow up row count |
| **Eager loading** (`include`/`prefetch`) | ORM generates optimized queries | Most ORM use cases | May still generate N+1 if misused |
| **`IN` clause batching** | `WHERE id IN (1,2,3,...)` | When JOIN would produce too many rows | Need to batch to avoid SQL parameter limits |
| **Dataloader pattern** | Batch and cache by key at runtime | GraphQL, complex nested queries | Adds middleware complexity |
| **Connection pool + prepared statements** | Reduce per-query overhead | Complementary to query optimization | Doesn't solve N+1, just reduces cost |

### Real-World Use Case: The Slow Admin Dashboard

> **🏢 Case Study: An internal admin tool listing 500 customers, each showing their "last order date," was making 501 queries per page load, taking 12+ seconds. Switching to a single `JOIN`-based query (or eager loading) brought load time down to under 200ms.**

### How to Detect N+1 in the Wild

1. **Enable query logging** in development — count queries per request
2. **Use APM tools** (Datadog, New Relic, Sentry) that flag "N+1 query" patterns automatically
3. **Code review habit**: any `.findAll()` or `.find()` inside a loop is a red flag
4. **Database-side monitoring**: look for the same query repeating with different parameters

```yaml
# Example: PostgreSQL query log showing N+1
LOG: statement: SELECT * FROM users (1 time)
LOG: statement: SELECT * FROM orders WHERE user_id = 1 (1 time)
LOG: statement: SELECT * FROM orders WHERE user_id = 2 (1 time)
LOG: statement: SELECT * FROM orders WHERE user_id = 3 (1 time)
... 1000 more times
# This pattern = N+1 detected!
```

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| ORM with default lazy loading | N queries inside a loop | Use `SelectRelated`/`Include`/`FetchJoin` |
| Pagination with N+1 | Slow page loads | Eager load within pagination slice |
| Deeply nested eager loading | Cartesian explosion | Use batch loading, not deep JOINs |
| Not monitoring query counts | N+1 goes undetected in production | Add query count alerts to CI/CD |
| Using `LIMIT` without `OFFSET` correctly | Missing data or wrong batch | Use cursor-based pagination for large datasets |

### 🔒 Security Considerations

N+1 queries can be a security and availability issue:
- **Denial of Service**: An endpoint with N+1 can be exploited by passing a large list, causing the database to execute thousands of queries
- **Rate limiting bypass**: If each sub-query bypasses rate limits (different parameter values), the total impact can be large
- **Information disclosure**: N+1 patterns sometimes occur because the ORM is fetching more data than intended

### 🧪 Testing Strategy

```python
# Python: Using pytest with query counting
from django.test.utils import CaptureQueriesContext

def test_no_n_plus_one():
    with CaptureQueriesContext(connection) as ctx:
        users = User.objects.prefetch_related('orders')
        for user in users:
            list(user.orders.all())  # Force evaluation
        
        # Assert: only 2 queries should have been executed
        assert len(ctx.captured_queries) == 2
```

```javascript
// JavaScript: Using a Sequelize test with query count assertion
const { sequelize } = require('./db');

test('user fetch does not cause N+1', async () => {
  const queryCountBefore = sequelize.queryInterface.queryGenerator._queryCount;
  
  const users = await User.findAll({ include: [Order] });
  users.forEach(u => u.orders.forEach(o => {})); // access orders
  
  const queryCountAfter = sequelize.queryInterface.queryGenerator._queryCount;
  expect(queryCountAfter - queryCountBefore).toBeLessThanOrEqual(2);
});
```

```java
// Java: JUnit with p6spy or datasource-proxy for query counting
@Test
void shouldNotCauseNPlusOne() {
    List<User> users = userRepository.findAllWithOrders();
    
    assertThat(queryCounter.getQueryCount()).isEqualTo(1); // Single query
}
```

### ⚡ Performance Benchmarks

| Scenario | N+1 Queries (1000 users) | Optimized (1-2 queries) | Improvement |
|---|---|---|---|
| Admin dashboard load | 12,000ms | 200ms | 60x faster |
| API endpoint (list with related) | 5,000ms | 150ms | 33x faster |
| GraphQL resolver chain | 8,000ms | 300ms | 27x faster |
| Mobile API (high latency) | 20,000ms | 500ms | 40x faster |

### 🎯 Quick Recap — Mistake #7

- **Problem**: ORM lazy loading creates N+1 query patterns, causing 100+ round-trips for what should be 1-2 queries
- **Fix**: Use eager loading (`JOIN`, `include`, `prefetch_related`, `JOIN FETCH`) to batch related data fetches
- **Verify**: Monitor query counts in development; set up alerting in CI/CD
- **Rule**: Any database call inside a loop is a suspect — investigate immediately

---

## Mistake #8: Storing Everything in JSON Columns

### Why JSON Feels Like a Shortcut

```sql
ALTER TABLE users ADD COLUMN metadata JSONB;

UPDATE users SET metadata = '{"subscription": {"plan": "pro", "renews_at": "2026-09-01"}, "preferences": {"theme": "dark", "notifications": true}, "flags": {"beta_tester": true}}';
```

It's flexible — you can add fields without a migration. But flexibility has a cost.

```mermaid
flowchart TD
    A[JSON Column] --> B[Harder to Index]
    A --> C[Slower Filtering]
    A --> D[No Column-Level Constraints]
    A --> E[Complicated Migrations]
    A --> F[App Must Validate Structure]
```

> **💡 Why This Matters:** JSON columns trade structure for flexibility. For frequently queried, typed data, this trade-off is usually not worth it.

### Why It Happens: The Root Cause

1. **Avoiding migrations**: "Adding a JSON field is faster than a schema migration"
2. **Document database mindset**: Coming from MongoDB, treating SQL columns as documents
3. **Prototyping**: "I'll structure it properly later" (later never comes)
4. **Developer convenience**: Easy to serialize/deserialize objects into JSON

### Example 1: The Filtering Problem ❌

If you frequently query:

```sql
-- ❌ Slow JSON query — can't use standard B-Tree indexes efficiently
SELECT * FROM users WHERE metadata->>'plan' = 'pro';
```

This is significantly slower than a real column comparison, unless you build a specialized JSON/GIN index (which adds its own complexity and isn't as efficient as a plain B-Tree on a real column).

```mermaid
sequenceDiagram
    participant App
    participant DB
    App->>DB: SELECT * FROM users WHERE metadata->>'plan' = 'pro'
    Note over DB: Extract 'plan' from JSON, then compare<br/>Can't use standard B-Tree index
    DB-->>App: Scan all rows, parse JSON, filter
    Note over App: Slow at scale
```

#### ✅ Correct Implementation: Real Column

```sql
-- ✅ Real column with proper B-Tree index
ALTER TABLE users ADD COLUMN subscription_plan VARCHAR(20);
ALTER TABLE users ADD COLUMN subscription_renews_at TIMESTAMP;

CREATE INDEX idx_users_subscription_plan ON users(subscription_plan);

-- Fast query:
SELECT * FROM users WHERE subscription_plan = 'pro';
```

### Example 2: The Constraint Problem ❌

```sql
-- ❌ No way to enforce that 'plan' must be one of 'free', 'pro', 'enterprise'
UPDATE users SET metadata = '{"subscription": {"plan": "ultra-mega-deal"}}';
-- This succeeds, even though "ultra-mega-deal" is not a valid plan!
```

#### ✅ Correct Implementation: CHECK Constraint

```sql
-- ✅ Enforce valid values at the database level
ALTER TABLE users ADD COLUMN subscription_plan VARCHAR(20);

ALTER TABLE users 
ADD CONSTRAINT chk_subscription_plan 
CHECK (subscription_plan IN ('free', 'pro', 'enterprise'));

-- Now this fails:
-- ERROR: new row for relation "users" violates 
-- check constraint "chk_subscription_plan"
INSERT INTO users (subscription_plan) VALUES ('ultra-mega-deal');
```

### Multiple Approaches: JSON vs. Real Columns

| Approach | Best For | Trade-offs |
|---|---|---|
| **Real Columns** | Frequently queried, filtered, sorted fields | Requires migrations to add fields |
| **JSON/JSONB** | Infrequently accessed metadata, debug data | No type safety, slower queries |
| **Hybrid** | Core fields as columns, optional metadata in JSON | Best of both worlds, some complexity |
| **EAV** (key-value in JSON) | Highly variable schemas | Very slow, hard to query, no constraints |

### When JSON Is the Right Tool ✅

- Storing optional, rarely-queried metadata (e.g., a `raw_webhook_payload` for debugging)
- Storing genuinely variable/user-defined schemas (e.g., custom form field responses)
- Fields you read as a whole blob but never filter/sort by individual keys
- Temporary storage during data migration

> **🔔 Rule:** If you filter, sort, join, or aggregate on a field regularly, it deserves its own column. JSON is for the exception, not the default.

### Use Case: User Preferences vs. Billing Data

> **🏢 Case Study: A startup stored both `theme_preference` (cosmetic, rarely queried) and `subscription_status` (critical, queried on every request for access control) inside the same JSON blob. Every authorization check required parsing JSON. Splitting `subscription_status` into a real indexed column improved auth-check latency dramatically, while `theme_preference` stayed safely in JSON since it's low-stakes and low-frequency.**

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Storing queryable data in JSON | Slow `WHERE` clauses, no standard indexes | Extract to real columns |
| No type checking | Invalid data silently accepted | Use `CHECK` constraints on real columns |
| No uniqueness constraints | Duplicate values in JSON | Use real columns with `UNIQUE` |
| Complex nested JSON | Hard to query, hard to validate | Flatten structure or use real columns |
| No partial indexes on JSON paths | All JSON data scanned on every query | Create GIN/GiST indexes on JSON operator paths |

### 🔒 Security Considerations

JSON columns present unique security challenges:

| Risk | Description | Mitigation |
|---|---|---|
| **Input validation** | No type/schema validation in JSON | Validate in application layer; use `CHECK` constraints where possible |
| **Injection via JSON** | JSON path expressions can be manipulated | Never build JSON path queries from user input |
| **Data masking** | JSON `SELECT *` exposes all fields | Always specify which JSON keys to extract |
| **PII in JSON** | Sensitive data in JSON blobs harder to audit | Apply the same data classification rules to JSON fields |
| **Deserialization** | JSON deserialization in app code can have vulnerabilities | Use safe JSON parsers, limit depth, validate types |

### 🧪 Testing Strategy

```sql
-- 1. Test that JSON queries are actually slow
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM users WHERE metadata->>'plan' = 'pro';
-- If this shows a Seq Scan, consider a real column

-- 2. If using JSON GIN index, verify it's being used
CREATE INDEX idx_users_metadata_gin ON users USING GIN (metadata);
-- Re-run EXPLAIN — should show Bitmap Index Scan

-- 3. Test data integrity
-- JSON has no built-in constraints, so test in application:
INSERT INTO users (metadata) 
VALUES ('{"subscription": {"plan": "INVALID_PLAN"}}');
-- This succeeds — application must validate!

-- 4. Compare performance: JSON vs. real column
-- Query 1 (JSON):
EXPLAIN ANALYZE SELECT id FROM users WHERE metadata->>'plan' = 'pro';
-- Query 2 (real column):
EXPLAIN ANALYZE SELECT id FROM users WHERE subscription_plan = 'pro';
-- Real column should be significantly faster
```

### ⚡ Performance Benchmarks

| Query Type | JSON Column (B-Tree) | Real Column (B-Tree) | JSON with GIN Index |
|---|---|---|---|
| Exact match on 1M rows | ~150ms | ~0.3ms | ~2ms |
| Range query | Not supported | ~1ms | ~50ms |
| Write overhead (INSERT) | Baseline | Baseline | +15% ( GIN index maintenance) |
| Storage overhead | JSON text | Minimal | JSON + GIN index (~2x storage) |

> **💡 Pro Tip #6:** In PostgreSQL, you can create an **expression index** on JSON paths as a middle ground: `CREATE INDEX idx_users_plan ON users ((metadata->>'plan'));`. This gives you B-Tree performance on a JSON-extracted value, but you still lose type safety and constraints.

### 🎯 Quick Recap — Mistake #8

- **Problem**: Storing frequently-queried, structured data in JSON columns makes queries slow and loses data integrity guarantees
- **Fix**: Use real columns for fields you filter/sort/join on; reserve JSON for truly optional, unstructured metadata
- **Verify**: Run `EXPLAIN` on JSON queries; if they show `Seq Scan`, extract to real columns
- **Rule**: If you query it, index it, and if you index it, make it a real column

---

## Mistake #9: Skipping Backup and Restore Testing

### The Mistake Behind the Mistake

Most teams *do* have backups. The real failure is **never testing that those backups can actually be restored**.

```mermaid
flowchart TD
    A[Backup Runs Nightly] --> B{Was Restore Ever Tested?}
    B -->|No| C[⚠️ Backup May Be Corrupted, Incomplete, or Incompatible]
    B -->|Yes| D[✅ Confidence That Recovery Actually Works]
    C --> E[Discovered Only During a Real Disaster]
    D --> F[Disaster Recovery Plan Is Trustworthy]
```

> **💡 Why This Matters:** Having a backup is an illusion of safety. Having a **verified restore** is actual safety. The difference is knowing whether your backup works before you need it.

### Why It Happens: The Root Cause

1. **Assumption bias**: "If the backup job succeeds, the backup works" — but success ≠ restorability
2. **Time pressure**: Testing restores is time-consuming and rarely prioritized
3. **Fear of testing**: "What if we accidentally corrupt the test environment?"
4. **No checklist**: Backup procedures are documented but testing steps are not

### Example: The Silent Failure ❌

A company ran automated nightly backups for two years. When a botched migration corrupted production data, they went to restore — and discovered the backup files had been truncated by a storage quota issue for the last 6 months. **The backups existed. They just didn't work.**

```mermaid
sequenceDiagram
    participant Backup as Backup Job (Nightly)
    participant Storage as Storage System
    participant Admin as Admin (2 Years Later)
    participant Disaster as Real Disaster Strikes
    Backup->>Storage: Write backup file
    Storage-->>Backup: OK (no error, but truncated at quota)
    loop 730 nights
        Backup->>Storage: Write backup file
        Storage-->>Backup: OK (silently truncated)
    end
    Disaster->>Admin: Production data corrupted!
    Admin->>Storage: Restore from backup...
    Storage-->>Admin: Corrupted/truncated backup file
    Note over Admin: Panic — backups haven't worked for 6 months!
```

### Best Practices Checklist

| Practice | Why It Matters |
|---|---|
| **Daily automated full backups** | Baseline recovery point |
| **Transaction log / WAL backups** | Enables point-in-time recovery, not just daily snapshots |
| **Offsite / cross-region storage** | Protects against regional outages or data center loss |
| **Quarterly restore drills** | The only way to *know* backups work |
| **Automated backup success monitoring/alerts** | Catch silent failures immediately, not months later |
| **Multiple retention periods** | Daily (7 days), Weekly (4 weeks), Monthly (12 months), Yearly (7 years) |

### Multiple Approaches: Backup Strategies

| Strategy | Recovery Point Objective (RPO) | Recovery Time Objective (RTO) | Best For |
|---|---|---|---|
| **Full backup only** | Up to 24 hours of data | Hours | Non-critical data, simple setups |
| **Full + Incremental** | Minutes to hours | Minutes to hours | Most production systems |
| **Full + Differential + Log** | Seconds to minutes | Minutes | Financial systems, high-value data |
| **Continuous Archiving (WAL/eWAL)** | Seconds | Minutes | Mission-critical, zero data loss |
| **Logical replication to standby** | Real-time | Seconds | Disaster recovery, read scaling |

### Real-World Use Case: Point-in-Time Recovery

> **🏢 Case Study: An e-commerce team accidentally ran an `UPDATE` without a `WHERE` clause, corrupting pricing data across the entire `products` table. Because they had transaction log backups and had tested point-in-time recovery before, they restored the database to the exact minute before the bad query ran — losing only a few minutes of legitimate data instead of days.**

### A Simple Restore Test Procedure

```mermaid
flowchart LR
    A[Take Backup] --> B[Spin Up Isolated Test Environment]
    B --> C[Restore Backup Into Test Environment]
    C --> D[Run Data Integrity Checks]
    D --> E[Document Restore Time]
    E --> F[Tear Down Test Environment]
```

> **🔔 Callout:** Schedule restore drills like you schedule fire drills. They're not optional.

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Restoring to wrong environment | Accidentally overwriting production | Use a dedicated DR environment; never restore to prod |
| Test restore only checks "files exist" | Data is corrupt but files are present | Run full application against restored data |
| No automation for restore | Takes hours under pressure | Script the restore; test the script |
| Single backup destination | One failure point | Use 3-2-1 rule (3 copies, 2 media, 1 offsite) |
| No monitoring on backup success | Silent failures | Alert on backup job failures |
| No point-in-time recovery plan | Limited granular recovery | Enable WAL/transaction log archiving |

### 🔒 Security Considerations

Backup security is often overlooked:

| Risk | Description | Mitigation |
|---|---|---|
| **Backup data exposure** | Backups contain full production data, often unencrypted | Encrypt backups at rest; use key management |
| **Backup access creep** | Too many people have restore access | Apply principle of least privilege; audit restore access |
| **Backup chain integrity** | Corrupted backup chain can go undetected | Verify checksums; test restore of incremental chains |
| **Backup retention** | Old backups may contain stale PII | Implement automated data retention policies |
| **Cross-region compliance** | Offsite backups may cross legal jurisdictions | Understand data sovereignty requirements |

### 🧪 Testing Strategy

```bash
# 1. Automate the restore test (PostgreSQL example)
#!/bin/bash
set -e

BACKUP_FILE="s3://my-backups/prod_$(date -d '1 day ago' +%Y%m%d).sql.gz"
TEST_DB="restore_test_$(date +%s)"

# Create test database
createdb $TEST_DB

# Restore backup
gunzip -c $BACKUP_FILE | psql $TEST_DB

# Run integrity checks
psql $TEST_URL -c "SELECT COUNT(*) FROM users;" 
psql $TEST_URL -c "SELECT COUNT(*) FROM orders;"

# Verify row counts match production (within acceptable margin)
USERS_IN_BACKUP=$(psql $TEST_DB -t -c "SELECT COUNT(*) FROM users;")
USERS_IN_PROD=$(psql $PROD_DB -t -c "SELECT COUNT(*) FROM users;")

if [ "$USERS_IN_BACKUP" -ne "$USERS_IN_PROD" ]; then
    echo "MISMATCH: Backup has $USERS_IN_BACKUP users, prod has $USERS_IN_PROD"
    exit 1
fi

# Document restore time
echo "Restore completed successfully in $(($(date +%s) - $START_TIME)) seconds"

# Clean up
dropdb $TEST_DB
```

### ⚡ Performance Benchmarks

| Backup Type | Backup Time | Restore Time | Storage Cost | Recovery Granularity |
|---|---|---|---|---|
| Full (cold) | 2 hours | 3 hours | 1x data size | Daily |
| Full + WAL archive | 2 hours + continuous | 10 min (to any point) | 1.2x data size | To the second |
| Logical dump | 4 hours | 6 hours | 0.3x data size | Daily |
| Physical base backup | 30 min | 20 min | 1x data size | With WAL, to the second |

> **💡 Pro Tip #8:** Automate restore drills in CI/CD. Use ephemeral cloud instances: spin up, restore, verify, tear down. This should run at least quarterly.

### 🎯 Quick Recap — Mistake #9

- **Problem**: Backups exist but are never tested for restorability
- **Fix**: Schedule quarterly restore drills in an isolated environment; automate the restore script
- **Verify**: Run data integrity checks against restored data; document restore time
- **Rule**: An untested backup is a false sense of security. An untested restore process is worse.

---

## Mistake #10: Not Understanding Isolation Levels and Concurrency

### The Classic Race Condition: Overselling

Imagine two users trying to buy the last item in stock at the same time.

```mermaid
sequenceDiagram
    participant User A
    participant User B
    participant DB
    User A->>DB: SELECT stock FROM products WHERE id = 1
    DB-->>User A: stock = 1
    User B->>DB: SELECT stock FROM products WHERE id = 1
    DB-->>User B: stock = 1
    User A->>DB: UPDATE stock = 0 (purchase)
    User B->>DB: UPDATE stock = 0 (purchase)
    Note over User A,User B: Both believe they succeeded — item oversold!
```

Both reads happened before either write — a classic **race condition** (also called a TOCTOU: Time of Check to Time of Use bug).

> **💡 Why This Matters:** At low traffic, race conditions are rare. At high traffic (100+ concurrent users), they become guaranteed bugs that corrupt data.

### Why It Happens: The Root Cause

1. **Assumption of serial execution**: "The database handles concurrency, right?" — Yes, but you must specify how
2. **Default isolation is insufficient**: `READ COMMITTED` prevents dirty reads but not race conditions
3. **Application-level check-then-act**: "Check stock, then decrement" — without a lock, two checks can happen before either decrements
4. **Misunderstanding "just add a transaction"**: A transaction alone doesn't prevent race conditions without proper locking

### Understanding Isolation Levels

| Isolation Level | Prevents | Common Use |
|---|---|---|
| **Read Uncommitted** | Nothing (dirty reads possible) | Rarely used in practice |
| **Read Committed** | Dirty reads | Default in PostgreSQL, MySQL (InnoDB); most apps |
| **Repeatable Read** | Dirty + non-repeatable reads | Reporting, consistent snapshots |
| **Serializable** | All anomalies (fully consistent) | Financial transactions, strict correctness |

```mermaid
flowchart LR
    A[Read Uncommitted] --> B[Read Committed] --> C[Repeatable Read] --> D[Serializable]
    A -.-> E[Least Safe, Fastest]
    D -.-> F[Safest, Slowest]
```

### Common Concurrency Problems

| Problem | Description | Isolation Level That Prevents It |
|---|---|---|
| **Dirty Read** | Read uncommitted data | Read Committed+ |
| **Non-Repeatable Read** | Same row read twice gives different values | Repeatable Read |
| **Phantom Read** | New rows appear between reads in same transaction | Serializable |
| **Write Skew** | Two transactions read, then both write based on what they read | Serializable |
| **Lost Update** | Two transactions overwrite each other's changes | All (with proper locking) |

### Fix 1: Pessimistic Locking ✅

```sql
-- ✅ Lock the row for the duration of the transaction
BEGIN;

SELECT stock
FROM products
WHERE id = 1
FOR UPDATE;   -- locks the row until COMMIT/ROLLBACK

-- Application checks stock > 0, then:
UPDATE products SET stock = stock - 1 WHERE id = 1;

COMMIT;
```

`FOR UPDATE` locks the row, forcing the second transaction to **wait** until the first one commits or rolls back.

```mermaid
sequenceDiagram
    participant User A
    participant User B
    participant DB
    User A->>DB: SELECT stock FOR UPDATE
    DB-->>User A: stock = 1 (row locked)
    User B->>DB: SELECT stock FOR UPDATE
    Note over User B,DB: User B blocks/waits...
    User A->>DB: UPDATE stock = 0, COMMIT
    DB-->>User B: Lock released, stock = 0
    User B->>DB: Sees stock = 0, purchase rejected
```

### Fix 2: Optimistic Locking with Version Columns ✅

```sql
-- ✅ Version column tracks changes
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5;  -- only update if version hasn't changed

-- Check: did the UPDATE affect any rows?
-- If 0 rows affected, someone else changed the row — application retries or fails
```

If another transaction already changed the row (and bumped the version), this `UPDATE` affects **0 rows**, and the application knows to retry or reject the purchase. This avoids locking overhead and works well for high-concurrency, low-conflict scenarios.

```mermaid
sequenceDiagram
    participant User A
    participant User B
    participant DB
    User A->>DB: UPDATE stock = 0, version = 6 WHERE version = 5
    DB-->>User A: 1 row affected ✅
    User B->>DB: UPDATE stock = 0, version = 6 WHERE version = 5
    DB-->>User B: 0 rows affected — version has changed! ❌
    Note over User B: Application sees conflict, rejects or retries
```

### Multiple Approaches: Concurrency Control

| Approach | How It Works | When to Use | Trade-offs |
|---|---|---|---|
| **Pessimistic Locking** (`FOR UPDATE`) | Locks rows until transaction ends | Low concurrency, high contention | Blocking, potential deadlocks |
| **Optimistic Locking** (version column) | Compares version, rejects on conflict | High concurrency, low conflict rate | Requires retry logic in app |
| **Atomic UPDATE** | Single `UPDATE ... WHERE` with condition | Simple stock/counters | Limited to single-row operations |
| **Serializable transactions** | Full serializability | Critical correctness | Lowest throughput |
| **Application-level locks** (Redis `SETNX`, etc.) | External lock before DB operation | Distributed systems | Additional infrastructure, single point of failure |

### Example 1: Atomic UPDATE for Stock Management ✅

For a simple "decrement if enough stock" scenario, a single atomic `UPDATE` is the simplest and most efficient solution:

```sql
-- ✅ Single atomic operation — no separate read needed
UPDATE products 
SET stock = stock - 1 
WHERE id = 1 AND stock >= 1;

-- Check how many rows were affected
-- 1 row = success, 0 rows = insufficient stock
```

This is faster than `FOR UPDATE` because it's a single statement with no explicit transaction or locking:

```sql
BEGIN;
UPDATE products SET stock = stock - 1 WHERE id = 1 AND stock >= 1;
-- In application: check affected row count
-- if 0, stock was insufficient — rollback
-- if 1, success — commit
COMMIT;
```

### Example 2: Application-Level Optimistic Locking ✅

```java
@Entity
public class Product {
    @Id private Long id;
    private int stock;
    
    @Version
    private Long version;  // JPA auto-increments this on each update
}

@Service
public class PurchaseService {
    @Transactional
    public boolean purchase(Long productId, int quantity) {
        try {
            Product product = productRepository.findById(productId);
            
            if (product.getStock() < quantity) {
                return false; // Not enough stock
            }
            
            product.setStock(product.getStock() - quantity);
            productRepository.save(product);
            
            return true;
        } catch (OptimisticLockException e) {
            // Another transaction modified the product concurrently
            // Application should retry or inform the user
            return false;
        }
    }
}
```

### Real-World Use Case: Ticket Booking System

> **🏢 Case Study: A concert ticketing platform used plain `SELECT` + `UPDATE` without locking and oversold seats during a high-demand on-sale event. Switching to `SELECT ... FOR UPDATE` inside a transaction (with a short timeout to avoid long lock waits) eliminated overselling, at a small cost to throughput — an acceptable trade-off for correctness in a scarce-resource scenario.**

### Common Pitfalls & Troubleshooting

| Pitfall | Symptom | Fix |
|---|---|---|
| Default isolation too low | Race conditions, lost updates | Raise isolation level or use explicit locking |
| Deadlock from `FOR UPDATE` | Transactions hang or fail | Set lock timeouts; order locks consistently |
| No retry logic for optimistic lock | User sees error | Implement exponential backoff retry |
| Long-running transactions | Locks held too long | Keep transactions short; do work outside |
| `NOWAIT` not used | Requests queue indefinitely | Use `FOR UPDATE NOWAIT` or `SKIP LOCKED` |
| Forgetting to check affected row count | Assumes update always succeeds | Always verify `UPDATE` row count |

### 🔒 Security Considerations

Concurrency bugs can have security implications:

| Risk | Description | Mitigation |
|---|---|---|
| **Authorization race condition** | Two requests change a user's permissions; one might access data they shouldn't | Use `FOR UPDATE` when modifying permissions |
| **Rate limit bypass** | Concurrent requests can exceed rate limits if counting is not atomic | Use atomic counter updates or Redis `INCR` with `EXPIRE` |
| **Double-spend in payment** | Two simultaneous payment requests for the same order | Use optimistic/pessimistic locking on payment records |
| **Inventory manipulation** | Attackers can exploit race conditions to oversell | Use atomic `UPDATE ... WHERE stock >= quantity` |

### 🧪 Testing Strategy

```java
// Integration test: verify no overselling under concurrent load
@Test
void shouldNotOversellUnderConcurrentLoad() throws InterruptedException {
    int threadCount = 50;
    CountDownLatch startLatch = new CountDownLatch(1);
    CountDownLatch doneLatch = new CountDownLatch(threadCount);
    AtomicInteger successCount = new AtomicInteger(0);
    AtomicInteger failureCount = new AtomicInteger(0);
    
    // Product has stock = 10
    Product product = new Product("Widget", 10);
    productRepository.save(product);
    
    ExecutorService executor = Executors.newFixedThreadPool(threadCount);
    
    for (int i = 0; i < threadCount; i++) {
        executor.submit(() -> {
            try {
                startLatch.await(); // Wait for all threads to be ready
                boolean purchased = purchaseService.purchase(product.getId(), 1);
                if (purchased) successCount.incrementAndGet();
                else failureCount.incrementAndGet();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                doneLatch.countDown();
            }
        });
    }
    
    startLatch.countDown(); // All threads start simultaneously
    doneLatch.await(); // Wait for all to complete
    
    // Exactly 10 should succeed (stock = 10)
    assertEquals(10, successCount.get());
    assertEquals(40, failureCount.get());
}
```

### ⚡ Performance Benchmarks

| Approach | Throughput (100 concurrent) | Overhead | Best For |
|---|---|---|---|
| No locking (race condition) | 5,000 ops/sec | 0% | Never (incorrect) |
| `FOR UPDATE` pessimistic lock | 500 ops/sec | -90% | Low-moderate concurrency |
| Optimistic locking (version) | 4,500 ops/sec | -10% | High concurrency, low conflict |
| Atomic `UPDATE ... WHERE` | 3,000 ops/sec | -40% | Simple stock/reservation |
| Serializable isolation | 300 ops/sec | -94% | Full correctness, low throughput |

> **💡 Pro Tip #7:** For high-concurrency scenarios, combine **optimistic locking** (for the common case) with **pessimistic locking** (for the critical section). Check the version column first; if there's a conflict, escalate to `FOR UPDATE`.

### When to Use Each Isolation Level

```mermaid
flowchart TD
    A[What level of consistency do you need?] --> B{Money or inventory?}
    B -->|Yes| C[Serializable or Pessimistic Locking]
    B -->|No| D{Analytics or reporting?}
    D -->|Yes| E[Repeatable Read]
    D -->|No| F[Read Committed (default)]
    F --> G{Many concurrent writes?}
    G -->|Yes| H[Consider optimistic locking]
    G -->|No| I[Default is fine]
```

### 🎯 Quick Recap — Mistake #10

- **Problem**: Default isolation levels don't prevent race conditions on read-then-act patterns
- **Fix**: Use `FOR UPDATE` (pessimistic), version columns (optimistic), or atomic `UPDATE` statements
- **Verify**: Write concurrent-load tests that verify exactly N operations succeed (not N+1)
- **Rule**: Any "check-then-act" pattern on shared data needs explicit concurrency control


---

## Putting It All Together: A Decision Framework

### The Database Decision Flowchart

```mermaid
flowchart TD
    A[New Database Feature] --> B{Does it involve multiple related writes?}
    B -->|Yes| C[Wrap in a Transaction]
    B -->|No| D[Continue]
    C --> D
    D --> E{Is this a frequently-run query?}
    E -->|Yes| F[Check EXPLAIN ANALYZE]
    E -->|No| G[Continue]
    F --> G
    G --> H{Is this in a loop calling the DB repeatedly?}
    H -->|Yes| I[Refactor to JOIN / eager loading — avoid N+1]
    H -->|No| J[Continue]
    I --> J
    J --> K{Are you selecting more columns than needed?}
    K -->|Yes| L[Trim to explicit columns]
    K -->|No| M[Continue]
    L --> M
    M --> N{Could two requests race on the same row?}
    N -->|Yes| O[Use locking or optimistic concurrency]
    N -->|No| P[Ship it 🚀]
    O --> P
```

### How to Use This Framework

Every time you write or modify a database operation, ask yourself these questions in order:

1. **Multiple writes?** → Wrap in a transaction (Mistake #3)
2. **Frequently-run query?** → Run `EXPLAIN ANALYZE` (Mistake #6)
3. **DB call in a loop?** → Refactor to JOIN/eager loading (Mistake #7)
4. **More columns than needed?** → Use explicit columns (Mistake #2)
5. **Concurrent access risk?** → Add locking or optimistic concurrency (Mistake #10)
6. **Filtering/sorting on JSON?** → Extract to real column (Mistake #8)
7. **Missing index on hot column?** → Add it (Mistake #1)
8. **Too many indexes on write-heavy table?** → Remove unused ones (Mistake #1)
9. **Denormalized data going stale?** → Add refresh strategy (Mistake #5)
10. **Backup never tested?** → Schedule a restore drill (Mistake #9)

> **🔑 Key Insight:** This framework is designed to be applied **early and often** — in code reviews, architecture design sessions, and incident post-mortems. The cost of catching a mistake here is near zero; the cost of catching it in production is enormous.

---

## Migration Guide: Auditing an Existing Codebase

### Phase 1: Assessment

#### Step 1: Identify Hot Queries

```sql
-- PostgreSQL: Top 10 slowest queries from pg_stat_statements
SELECT query, calls, total_time, mean_time, stddev_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

#### Step 2: Identify Unused Indexes

```sql
-- PostgreSQL: Indexes that are never scanned
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY schemaname, tablename;
```

#### Step 3: Find N+1 Patterns

Enable query logging in development and look for the same query repeated with different parameters:

```bash
# In PostgreSQL, check logs for repeated queries:
grep -c "SELECT .* FROM orders WHERE user_id" postgresql.log
# If count > 100, investigate for N+1
```

#### Step 4: Check for SELECT * Usage

```bash
# Grep your codebase for SELECT * in production query files
grep -rn "SELECT \*" --include="*.py" --include="*.js" --include="*.java" src/ | grep -v "test\|debug\|dev"
# Review each hit — is it in a production code path?
```

### Phase 2: Prioritized Remediation Plan

| Priority | Issue | Estimated Effort | Risk Level |
|---|---|---|---|
| 🔴 Critical | Missing indexes on hot queries | 1-2 hours per index | Low (test with EXPLAIN) |
| 🟡 High | N+1 queries in production endpoints | 2-8 hours per endpoint | Medium |
| 🟡 High | Missing foreign key constraints | 1-3 hours per constraint | Medium (data cleanup may be needed) |
| 🟢 Medium | SELECT * in production paths | 1-2 hours per query | Low |
| 🟢 Medium | Over-indexing on write-heavy tables | 1 hour to audit + clean up | Low |
| 🟢 Medium | Transaction boundaries missing | 2-4 hours per flow | Medium |
| 🔵 Low | JSON columns used for queryable data | 4-8 hours per migration | High |
| 🔵 Low | Stale table statistics | 30 min (run ANALYZE) | None |
| 🔵 Low | Never-tested backups | 4-8 hours (first drill) | Medium |

> **⚠️ Warning:** Before any migration, always take a full backup, test on a staging copy, and run during low-traffic hours with a rollback plan.

### Phase 3: Execution

#### Adding an Index Safely (PostgreSQL)

```sql
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
-- CONCURRENTLY avoids table locks but takes longer
```

#### Adding a Foreign Key with Data Cleanup

```sql
-- Step 1: Find orphaned records
SELECT o.id, o.user_id 
FROM orders o 
LEFT JOIN users u ON o.user_id = u.id 
WHERE u.id IS NULL;

-- Step 2: Clean up orphans
DELETE FROM orders WHERE user_id NOT IN (SELECT id FROM users);

-- Step 3: Add constraint without checking existing data
ALTER TABLE orders 
ADD CONSTRAINT fk_orders_user 
FOREIGN KEY (user_id) REFERENCES users(id) NOT VALID;

-- Step 4: Validate after confirming no new violations
ALTER TABLE orders VALIDATE CONSTRAINT fk_orders_user;
```

#### Refactoring N+1 to JOIN

```javascript
// Before (N+1):
const users = await User.findAll();
for (const user of users) {
  user.orders = await Order.findAll({ where: { userId: user.id } });
}

// After:
const users = await User.findAll({
  include: [{ model: Order, required: false }]  // LEFT JOIN
});
```

### Phase 4: Verification

```sql
-- After all changes, run EXPLAIN on critical queries
-- Verify: Index Scan instead of Seq Scan
-- Verify: No N+1 patterns remain
```

---

## Testing Strategies for Database Decisions

### The Testing Pyramid for Database Code

```mermaid
flowchart TD
    A[Unit Tests] --> B[Integration Tests]
    B --> C[CI/CD Checks]
    C --> D[Production Monitoring]
    A -.-> a[SQL syntax, constraint logic]
    B -.-> b[Transaction behavior, isolation levels]
    C -.-> c[Query plan regressions, N+1 detection]
    D -.-> d[Real-world perf, deadlock detection]
```

### 1. Unit Testing Database Constraints

```sql
-- Test: foreign key enforcement
INSERT INTO orders (user_id, total) VALUES (999999, 50.00);
-- Should fail with FK violation

-- Test: check constraint
INSERT INTO users (subscription_plan) VALUES ('invalid_plan');
-- Should fail with check constraint violation
```

### 2. Integration Testing with Query Counting (Python/Django)

```python
from django.test import TestCase
from django.test.utils import CaptureQueriesContext

class NoNPlusOneTest(TestCase):
    def test_user_list_with_orders(self):
        with CaptureQueriesContext(self.client) as ctx:
            response = self.client.get('/api/users/')
        # Assert: no N+1 (should be 2 queries, not 251)
        self.assertEqual(len(ctx.captured_queries), 2)
```

### 3. CI/CD Integration

```yaml
# .github/workflows/ci.yml
jobs:
  db-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
        options: --health-cmd pg_isready
    steps:
      - name: Run database tests
        run: npm test
      - name: Check for N+1 queries
        run: npm run test:nplusone
      - name: Validate schema migrations
        run: npm run migrate:validate
```

### 4. Concurrency Testing

```java
// Java: Verify no overselling under concurrent load
@Test
void shouldNotOversellUnderConcurrentLoad() throws InterruptedException {
    int threadCount = 50;
    AtomicInteger successCount = new AtomicInteger(0);
    AtomicInteger failureCount = new AtomicInteger(0);
    
    Product product = new Product("Widget", 10);
    productRepository.save(product);
    
    ExecutorService executor = Executors.newFixedThreadPool(threadCount);
    CountDownLatch startLatch = new CountDownLatch(1);
    CountDownLatch doneLatch = new CountDownLatch(threadCount);
    
    for (int i = 0; i < threadCount; i++) {
        executor.submit(() -> {
            try {
                startLatch.await();
                if (purchaseService.purchase(product.getId(), 1)) {
                    successCount.incrementAndGet();
                } else {
                    failureCount.incrementAndGet();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                doneLatch.countDown();
            }
        });
    }
    
    startLatch.countDown();
    doneLatch.await();
    
    assertEquals(10, successCount.get());
    assertEquals(40, failureCount.get());
}
```

---

## Security Considerations

> **⚠️ Security Alert:** Database security extends beyond firewalls. Many of the 10 mistakes directly create security vulnerabilities.

### Security Impact Matrix

| Mistake | Security Risk | Severity |
|---|---|---|
| #1: Missing/Excessive Indexes | Information leakage via SQL injection | Medium |
| #2: SELECT * | Sensitive data exposure in API responses | High |
| #3: No Transactions | Financial fraud, audit trail gaps | Critical |
| #4: Poor Schema Design | Privilege escalation, data leakage | High |
| #5: Over-Normalization | Access control bypass via missing JOINs | Medium |
| #6: Ignored Exec Plans | DoS via crafted queries | Medium |
| #7: N+1 Queries | DoS amplification via large lists | High |
| #8: JSON Abuse | Input validation bypass, injection | Medium |
| #9: Untested Backups | Data breach during recovery | High |
| #10: Poor Isolation | Race conditions, double-spend | Critical |

### Defense-in-Depth Strategies

#### 1. Principle of Least Privilege

```sql
-- Create a read-only user for API queries
CREATE USER api_reader WITH PASSWORD '...';
GRANT SELECT ON users, orders TO api_reader;

-- Create a write user for mutations
CREATE USER api_writer WITH PASSWORD '...';
GRANT SELECT, INSERT, UPDATE ON users, orders TO api_writer;
```

#### 2. Row-Level Security

```sql
-- PostgreSQL Row Level Security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON orders
FOR ALL TO api_writer
USING (tenant_id = current_setting('app.tenant_id')::INTEGER);
```

#### 3. Encryption

```sql
-- PostgreSQL: Column-level encryption
CREATE EXTENSION IF NOT EXISTS pgcrypto;
INSERT INTO users (email, password_hash) 
VALUES ('user@example.com', digest('password', 'sha256'));
```

#### 4. Database Auditing

```
# In postgresql.conf:
log_statement = 'all'
log_line_prefix = '%t %d %u %r'
log_min_duration_statement = 1000  # Log queries >1 second
```

---

## Performance Benchmarks Reference

### Cross-Mistake Performance Impact Summary

| Mistake | Typical Impact | When It Matters |
|---|---|---|
| #1: Missing Index | 100-1000x slower queries | Any query on >100K row table |
| #2: SELECT * | 2-10x larger payloads | Tables with TEXT/JSON/BLOB columns |
| #3: No Transaction | Data corruption | Multi-step financial operations |
| #4: Poor Schema | 10-1000x slower queries | Any non-trivial relationship |
| #5: Over-Normalization | 10-100x slower reads | 3+ table joins on >1M rows |
| #6: Ignored Exec Plan | 10-1000x slower | Complex queries, production scale |
| #7: N+1 Queries | 30-100x slower endpoints | ORM with related data |
| #8: JSON Columns | 50-500x slower filtering | Frequent filtering on JSON keys |
| #9: No Backup Tests | Total data loss | Any production system |
| #10: Poor Isolation | Data corruption | Any concurrent writes |

---

## Best Practices Summary

### The Essential Database Best Practices Checklist

```mermaid
mindmap
  root((Best Practices))
    Indexes
      Index hot WHERE/JOIN columns
      Remove unused indexes
      Use composite indexes wisely
      Avoid low-cardinality indexes
    Queries
      Never use SELECT * in prod
      Always EXPLAIN ANALYZE
      Fix N+1 patterns
      Use explicit column lists
    Transactions
      Wrap multi-step writes
      Keep transactions short
      Use appropriate isolation levels
      Test rollback behavior
    Schema
      Use foreign keys everywhere
      Enforce constraints at DB level
      Use proper data types
      Consistent naming conventions
    Concurrency
      Prevent race conditions
      Use FOR UPDATE or optimistic locking
      Test under concurrent load
      Set lock timeouts
    Operations
      Test backup restores quarterly
      Store backups offsite
      Encrypt at rest
      Monitor query performance
```

### 📋 Quick Reference: Every Query Should...

| ✅ Do This | ❌ Never Do This |
|---|---|
| Use explicit column names, not `SELECT *` | SELECT * in production code |
| Be tested with `EXPLAIN ANALYZE` | Assume query is fast without measuring |
| Be wrapped in a transaction if multi-step | Leave multi-step writes unwrapped |
| Use indexed columns in WHERE/JOIN | Filter on unindexed columns |
| Use eager loading in ORMs | Lazy load in loops (N+1) |
| Use real columns for queryable data | Store queryable data in JSON |
| Be protected by foreign keys | Allow orphaned records |
| Handle concurrency with locks | Assume reads are always consistent |
| Be backed by tested backups | Trust backups without testing |
| Have constraints enforced at DB level | Rely solely on application-level checks |

---

## Anti-Patterns Summary

### The Comprehensive Anti-Pattern Catalog

```mermaid
flowchart TD
    A[Anti-Patterns] --> B[Performance]
    A --> C[Integrity]
    A --> D[Design]
    A --> E[Operations]
    B --> B1[SELECT * everywhere]
    B --> B2[Indexing everything]
    B --> B3[Ignoring EXPLAIN]
    B --> B4[N+1 in every loop]
    C --> C1[Auto-commit per statement]
    C --> C2[No foreign keys]
    C --> C3[CSV in columns]
    C --> C4[Race conditions unprotected]
    D --> D1[JSON for queryable data]
    D --> D2[Over-normalization]
    D --> D3[No constraints]
    D --> D4[Inconsistent naming]
    E --> E1[Never tested backups]
    E --> E2[Silent backup failures]
    E --> E3[No monitoring alerts]
    
    style B1 fill:#f96
    style C2 fill:#f96
    style D1 fill:#f96
    style E1 fill:#f96
```

### Anti-Pattern Detail Table

| Anti-Pattern | What It Is | Why It's Bad | Correct Approach |
|---|---|---|---|
| **SELECT *** | Selecting all columns | Wasted bandwidth, CPU, security risk | Explicit column lists |
| **Index Everything** | Adding indexes "just in case" | Write amplification, storage waste | Index based on actual query patterns |
| **Auto-commit Mode** | Each statement commits independently | Partial data states, no rollback | Explicit transactions for multi-step ops |
| **CSV in a Column** | Storing lists as comma-separated strings | Unqueryable, error-prone | Proper junction tables |
| **No Foreign Keys** | Skipping referential integrity | Orphaned records, data corruption | Always use FK constraints |
| **JSON for Everything** | Storing structured data in JSON | Slow queries, no constraints | Real columns for queryable data |
| **No Concurrency Control** | Check-then-act without locking | Race conditions, overselling | FOR UPDATE or optimistic locking |
| **Untested Backups** | Backups run but never restored | Zero recovery confidence | Quarterly restore drills |
| **Ignoring EXPLAIN** | Guessing about query performance | Missed optimization opportunities | Always EXPLAIN ANALYZE |
| **N+1 in Loops** | DB call per item in a loop | 100+ unnecessary round-trips | JOIN or eager loading |
| **No Schema Constraints** | Relying on app code alone | Data quality issues | Enforce at database level |
| **Over-Normalization** | Too many tables, too many joins | Slow reads, complex queries | Strategic denormalization |

---

## Troubleshooting Guide

### Quick Diagnosis: My Query Is Slow — What Now?

```mermaid
flowchart TD
    A[Query is slow] --> B[Run EXPLAIN ANALYZE]
    B --> C{See Seq Scan on large table?}
    C -->|Yes| D[Add index on WHERE/JOIN column]
    C -->|No| E{See N+1 pattern?}
    E -->|Yes| F[Add JOIN or eager loading]
    E -->|No| G{Large payload?}
    G -->|Yes| H[Replace SELECT * with explicit columns]
    G -->|No| I{Stale statistics?}
    I -->|Yes| J[Run ANALYZE table_name]
    I -->|No| K{Concurrent writes blocking?}
    K -->|Yes| L[Check for deadlocks, review transaction scope]
    K -->|No| M[Seek expert review]
```

### Common Error Messages and Fixes

| Error | Cause | Fix |
|---|---|---|
| `deadlock detected` | Two transactions waiting on each other | Reduce transaction size, order locks consistently |
| `duplicate key value` | Violating UNIQUE constraint | Handle gracefully, check before insert |
| `violates foreign key constraint` | Referencing non-existent record | Add FK constraint, clean up orphaned data |
| `canceling statement due to statement timeout` | Query too slow | Add index, optimize query, increase timeout |
| `tuple concurrently updated` | Concurrent modification | Retry with exponential backoff |
| `out of memory` | Very large query result | Paginate results, use streaming cursor |
| `disk I/O error` | Storage failure | Check disk health, restore from backup |
| `too many connections` | Connection pool exhausted | Optimize connection usage, increase pool size |

### Performance Troubleshooting Checklist

1. **Run `EXPLAIN ANALYZE`** on the slow query
2. **Check for `Seq Scan`** on tables with >100K rows → add index
3. **Check for `Nested Loop`** with high row counts → verify join indexes
4. **Compare actual vs. estimated rows** → run `ANALYZE` if very different
5. **Check for N+1** → count queries, look for repeated patterns
6. **Replace `SELECT *`** → use explicit columns
7. **Review transaction scope** → keep it short, do computation outside
8. **Check lock waits** → `SELECT * FROM pg_locks WHERE granted = false;`
9. **Monitor connection pool** → are connections leaking?
10. **Check disk I/O** → is the disk saturated?

### 🔍 Diagnostic Queries (PostgreSQL)

```sql
-- Find blocking queries
SELECT blocked_locks.pid AS blocked_pid,
       blocked_activity.usename AS blocked_user,
       blocking_locks.pid AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query AS blocked_statement,
       blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
  ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
  ON blocking_locks.locktype = blocked_locks.locktype
  AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
  AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
JOIN pg_catalog.pg_stat_activity blocking_activity 
  ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Check index usage (find unused indexes)
SELECT schemaname, tablename, indexname, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_tup_fetch = 0
ORDER BY idx_tup_read DESC;

-- Check table bloat (wasted space due to updates/deletes)
SELECT schemaname, tablename, 
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
       pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

## Quick Recap: All 10 Mistakes

### Executive Summary

| # | Mistake | One-Line Fix | Key Tool |
|---|---|---|---|
| 1 | Not Adding Indexes | Index hot WHERE/JOIN columns | `EXPLAIN ANALYZE` |
| 2 | Using SELECT * | Always use explicit columns | Column allowlisting |
| 3 | No Transactions | Wrap multi-step writes | `BEGIN`/`COMMIT` or `@Transactional` |
| 4 | Poor Schema Design | Use FKs, constraints, proper types | ER diagrams, naming conventions |
| 5 | Over-Normalizing | Denormalize read hot paths | Materialized views, summary tables |
| 6 | Ignoring Exec Plans | Always EXPLAIN every query | `EXPLAIN ANALYZE` |
| 7 | N+1 Queries | Eager loading or JOIN | Query count monitoring |
| 8 | JSON Column Abuse | Real columns for queryable data | `CHECK` constraints |
| 9 | Untested Backups | Schedule restore drills | Automated restore scripts |
| 10 | Poor Isolation | Use locking or optimistic concurrency | `FOR UPDATE`, version columns |

### The Iron Triangle of Database Performance

```mermaid
flowchart TD
    A[Read Performance] --> D{Pick any two}
    B[Write Performance] --> D
    C[Consistency] --> D
    D -->|Fast reads + Strong consistency| E[More writes overhead]
    D -->|Fast writes + Strong consistency| F[More reads overhead]
    D -->|Fast reads + Fast writes| G[Weaker consistency]
    style E fill:#ff6
    style F fill:#ff6
    style G fill:#f96
```

> **🔑 Golden Rule:** You can have any two of {fast reads, fast writes, strong consistency}. The database lets you choose — make the right choice for your use case.

---

## Practice Exercises (with Solutions)

### Exercise 1: Index Audit and Optimization

**Scenario:** You're a database administrator for an e-commerce platform. The `orders` table has 2 million rows. The following queries are running slowly:

```sql
-- Query A: Find orders by customer email (takes 3+ seconds)
SELECT o.id, o.total, o.status 
FROM orders o 
JOIN users u ON o.user_id = u.id 
WHERE u.email = 'customer@example.com';

-- Query B: Find pending orders by date range (takes 2+ seconds)
SELECT id, total, created_at 
FROM orders 
WHERE status = 'pending' AND created_at >= '2024-01-01' 
ORDER BY created_at DESC 
LIMIT 100;

-- Query C: Count orders per customer (takes 1+ seconds)
SELECT user_id, COUNT(*) as order_count 
FROM orders 
GROUP BY user_id 
HAVING COUNT(*) > 10;
```

**Tasks:**
1. Run `EXPLAIN ANALYZE` on each query and identify the problem
2. Identify what indexes are missing
3. Write the `CREATE INDEX` statements to fix each query
4. Explain why you chose those indexes

**Solution:**

```sql
-- Problem Analysis:
-- Query A: Likely missing index on users(email) 
-- Query B: Missing composite index on orders(status, created_at)
-- Query C: Missing index on orders(user_id)

-- Fix 1: Index on users for the JOIN + WHERE
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Fix 2: Composite index matching WHERE + ORDER BY + LIMIT
-- Column order matters: status first (filters), then created_at (sorts)
CREATE INDEX idx_orders_status_created ON orders(status, created_at DESC);

-- Fix 3: Index on the GROUP BY column
CREATE INDEX idx_orders_user_id_status ON orders(user_id);
-- The GROUP BY already benefits from idx_orders_user_id above

-- Verification:
EXPLAIN ANALYZE
SELECT o.id, o.total, o.status 
FROM orders o JOIN users u ON o.user_id = u.id 
WHERE u.email = 'customer@example.com';
-- Should show: Index Scan on users, Index Scan on orders (not Seq Scan)

EXPLAIN ANALYZE
SELECT id, total, created_at 
FROM orders 
WHERE status = 'pending' AND created_at >= '2024-01-01' 
ORDER BY created_at DESC LIMIT 100;
-- Should show: Index Scan using idx_orders_status_created (no Sort node)
```

> **💡 Key Learning:** Composite indexes serve multiple purposes — `(status, created_at)` handles both the `WHERE` filter and the `ORDER BY` sort. The column order in composite indexes matters: put the most selective filter first.

---

### Exercise 2: Transaction Boundary Design

**Scenario:** You're implementing a money transfer feature. The `transfer()` function must:
1. Deduct money from account A
2. Add money to account B
3. Record the transaction in an audit log
4. Update a running balance summary table

**Tasks:**
1. Identify which operations should be in a single transaction
2. Write the correct transactional implementation in SQL
3. Identify what happens if each step fails (and why a transaction is needed)
4. Write the application-level code using `@Transactional` (Java) or equivalent

**Solution:**

```sql
-- All 4 steps must be in a single transaction
BEGIN;

UPDATE accounts 
SET balance = balance - 100.00 
WHERE id = 1;  -- Account A

UPDATE accounts 
SET balance = balance + 100.00 
WHERE id = 2;  -- Account B

INSERT INTO transaction_audit (from_account, to_account, amount, timestamp)
VALUES (1, 2, 100.00, NOW());

UPDATE account_summary 
SET total_transferred = total_transferred + 100.00,
    last_updated = NOW()
WHERE account_id IN (1, 2);

COMMIT;
```

**What happens without a transaction:**
- If step 2 fails after step 1 succeeds: Money disappears from A but never reaches B
- If step 3 fails: Audit trail is incomplete, impossible to reconcile
- If step 4 fails: Summary is stale, reports are inaccurate

**Java Implementation:**

```java
@Service
@Transactional
public class TransferService {
    
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepository.findById(fromId);
        Account to = accountRepository.findById(toId);
        
        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
        
        accountRepository.save(from);
        accountRepository.save(to);
        
        auditLogRepository.save(new TransactionAudit(fromId, toId, amount));
        
        // Update summaries
        summaryRepository.incrementTransferred(fromId, amount);
        summaryRepository.incrementTransferred(toId, amount);
        
        // If ANY of these throw, the entire transaction rolls back
    }
}
```

> **💡 Key Learning:** The transaction boundary should encompass all operations that must succeed or fail together. A transaction that's too small (missing steps) leads to data inconsistency. One that's too large (including external API calls) leads to lock contention.

---

### Exercise 3: Fix N+1 and Concurrency Issues

**Scenario:** You have a REST API endpoint that returns a list of products with their category names and the number of reviews for each product. The current code has both an N+1 problem and a potential race condition.

```javascript
// Current (buggy) implementation
app.get('/api/products', async (req, res) => {
  const products = await Product.findAll();  // 1 query
  
  const result = [];
  for (const product of products) {
    const category = await Category.findByPk(product.categoryId);  // N queries!
    const reviewCount = await Review.count({ where: { productId: product.id } });  // N queries!
    result.push({
      id: product.id,
      name: product.name,
      category: category.name,
      reviewCount: reviewCount
    });
  }
  
  res.json(result);
});
```

**Tasks:**
1. Identify the N+1 pattern
2. Rewrite using a single JOIN query
3. Add a concurrency-safe way to increment the review count (when a new review is posted)
4. Explain why the original code is dangerous under concurrent load

**Solution:**

```javascript
// Fix 1 & 2: Eliminate N+1 with a single JOIN query
app.get('/api/products', async (req, res) => {
  const results = await sequelize.query(
    `SELECT p.id, p.name, c.name as category_name,
            COUNT(r.id) as review_count
     FROM products p
     LEFT JOIN categories c ON p.category_id = c.id
     LEFT JOIN reviews r ON p.id = r.product_id
     GROUP BY p.id, p.name, c.name
     ORDER BY p.id`,
    { type: sequelize.QueryTypes.SELECT }
  );
  
  res.json(results);
});

// Alternative using Sequelize eager loading:
app.get('/api/products', async (req, res) => {
  const products = await Product.findAll({
    attributes: ['id', 'name'],
    include: [
      { model: Category, attributes: ['name'] },
      { model: Review, attributes: [] }  // Don't fetch review data, just count
    ],
    group: ['product.id', 'category.name'],
    raw: true
  });
  
  // Sequelize with `include` and `group` generates efficient query
  res.json(products);
});

// Fix 3: Concurrency-safe review count increment
const addReview = async (productId, rating, comment) => {
  const transaction = await sequelize.transaction();
  try {
    // Use atomic UPDATE to avoid race conditions
    await Review.create(
      { product_id: productId, rating, comment },
      { transaction }
    );
    
    // Atomic increment — no separate read needed
    await Product.update(
      { review_count: sequelize.literal('review_count + 1') },
      { where: { id: productId }, transaction }
    );
    
    await transaction.commit();
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
};

// Even better — use an atomic UPDATE with a WHERE clause:
// This handles the case where the product might not exist
const addReviewAtomic = async (productId, rating, comment) => {
  return await sequelize.transaction(async (t) => {
    const newReview = await Review.create(
      { product_id: productId, rating, comment },
      { transaction: t }
    );
    
    const [updated] = await Product.update(
      { review_count: sequelize.literal('review_count + 1') },
      { 
        where: { id: productId },
        transaction: t 
      }
    );
    
    if (updated === 0) {
      throw new Error('Product not found — review count not updated');
    }
    
    return newReview;
  });
};
```

> **💡 Key Learning:** N+1 problems often co-occur with concurrency issues. When you have lazy loading in a loop (N+1), each individual query can also race with other requests. Fixing both simultaneously often means using JOIN queries (fixing N+1) and atomic UPDATEs (fixing concurrency).

---

## Hands-On Lab: E-Commerce Database Audit

### Lab Overview

In this lab, you'll audit a simulated e-commerce database schema, identify all 10 mistakes, and fix them. This mirrors a real production database audit.

### Setup

```sql
-- Download and load the sample schema
-- (In practice, you'd connect to a real database)

-- Sample schema with intentional mistakes:
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),
  name VARCHAR(100),
  password_hash VARCHAR(255),
  is_active BOOLEAN DEFAULT true,
  metadata JSONB  -- Mistake #8: storing queryable data in JSON
);

CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  price DECIMAL(10,2),
  category VARCHAR(100),  -- Mistake #4: CSV-like, no FK
  tags TEXT,              -- Mistake #4: CSV in column
  stock INTEGER,
  sku VARCHAR(50)         -- Mistake #1: no index on this hot column
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,        -- Mistake #4: no FK constraint
  total DECIMAL(10,2),
  status VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW(),
  -- Mistake #5: over-normalized — could be simpler
  billing_address_id INTEGER,
  shipping_address_id INTEGER,
  tax_amount DECIMAL(10,2),
  discount_amount DECIMAL(10,2),
  notes TEXT             -- Mistake #2: SELECT * includes this huge column
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER,       -- Mistake #4: no FK constraint
  product_id INTEGER,    -- Mistake #1: no FK constraint
  quantity INTEGER,
  price DECIMAL(10,2)
);

-- Mistake #9: No backup strategy documented or tested
```

### Step 1: Run EXPLAIN on Key Queries

```sql
-- Analyze the most critical queries
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
-- Expect: Seq Scan (no index on email)

EXPLAIN ANALYZE 
SELECT * FROM products WHERE sku = 'ABC-123';
-- Expect: Seq Scan (no index on sku)

EXPLAIN ANALYZE
SELECT o.id, o.total 
FROM orders o 
JOIN order_items oi ON o.id = oi.order_id 
WHERE o.user_id = 42;
-- Expect: Seq Scan on orders (no index on user_id)
```

### Step 2: Identify and Fix Indexes

```sql
-- Fix Missing Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

-- Verify improvements
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
-- Now: Index Scan using idx_users_email

-- Fix Over-Indexing: Check for unused indexes
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0;
```

### Step 3: Fix Schema Issues

```sql
-- Add missing foreign keys
ALTER TABLE orders 
ADD CONSTRAINT fk_orders_user 
FOREIGN KEY (user_id) REFERENCES users(id);

ALTER TABLE order_items 
ADD CONSTRAINT fk_order_items_order 
FOREIGN KEY (order_id) REFERENCES orders(id);

ALTER TABLE order_items 
ADD CONSTRAINT fk_order_items_product 
FOREIGN KEY (product_id) REFERENCES products(id);

-- Extract JSON queryable data to real columns
ALTER TABLE users ADD COLUMN subscription_plan VARCHAR(20);
ALTER TABLE users ADD COLUMN subscription_renews_at TIMESTAMP;
ALTER TABLE users 
ADD CONSTRAINT chk_subscription_plan 
CHECK (subscription_plan IN ('free', 'pro', 'enterprise'));

-- Fix CSV column: create proper category table
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL
);

ALTER TABLE products 
ADD COLUMN category_id INTEGER REFERENCES categories(id);

-- Update products from category text
UPDATE products p SET category_id = (
  SELECT id FROM categories c WHERE c.name = p.category
);

-- Now products.category can be dropped in a future migration
```

### Step 4: Fix Transaction Boundaries

```sql
-- Before: No transaction wrapping
-- Fix: Wrap order creation in a transaction
BEGIN;
INSERT INTO orders (user_id, total, status) VALUES (42, 100.00, 'pending');
INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (currval('orders_id_seq'), 1, 2, 50.00);
UPDATE products SET stock = stock - 2 WHERE id = 1;
COMMIT;
```

### Step 5: Fix SELECT * and N+1 Patterns

```sql
-- Before: SELECT * pulls huge notes column
-- After: explicit columns
SELECT id, user_id, total, status, created_at FROM orders WHERE user_id = 42;

-- Before: N+1 pattern in application
-- After: single JOIN query
SELECT u.email, p.name, o.total, oi.quantity, oi.price
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE u.email = 'customer@example.com';
```

### Step 6: Concurrency Protection

```sql
-- Before: Race condition on stock update
-- After: Atomic update with condition check
UPDATE products 
SET stock = stock - 2 
WHERE id = 1 AND stock >= 2;
-- If 0 rows affected, not enough stock — handle in app
```

### Step 7: Verify Everything

```sql
-- Re-run all queries with EXPLAIN ANALYZE
-- Verify: Index Scans, no Seq Scans on large tables
-- Count queries in application — verify no N+1
-- Test transaction rollback behavior
```

### Lab Results

After completing this audit, you should see:
- **Query performance**: 50-500x improvement on hot queries
- **Data integrity**: FK constraints prevent orphaned records
- **Security**: No sensitive data leaked via SELECT *
- **Concurrency safety**: No race conditions on stock updates
- **Backup readiness**: Documented and tested restore procedure

---


---

## Pro Tips

> Advanced insights that experienced database engineers use in production. These tips assume you've mastered the 10 core mistakes.

### Pro Tip #1: Composite Index Column Order Matters

A composite index on `(A, B)` can serve:
- Queries filtering by `A` alone ✅
- Queries filtering by `A` AND `B` ✅
- Queries filtering by `B` alone ❌ (index is not used)

> **Rule of thumb:** Put the most selective column first, but always match the most common query pattern.

### Pro Tip #2: Keep Transactions Short — Do Work Outside

```java
@Transactional
public void processOrder(OrderRequest request) {
    // ✅ Do this OUTSIDE the transaction
    User user = userService.fetchUserDetails(request.getUserId());
    
    // ✅ Minimal transaction — only the DB writes
    orderService.createOrder(request, user);
}
```

Long transactions hold locks longer, increasing the chance of deadlocks.

### Pro Tip #3: Foreign Keys Are Your Safety Net, Not Your Bottleneck

The overhead of FK checks is typically <1% even on write-heavy tables. The protection they provide is worth 10x their cost.

### Pro Tip #4: Normalize for Correctness, Denormalize for Performance

> **Normalize** to eliminate update anomalies. **Denormalize** only after measuring a real read performance problem, and always with a refresh strategy.

### Pro Tip #5: Automate EXPLAIN Regressions in CI/CD

```sql
-- Save this as a baseline query plan test
PREPARE critical_query AS
SELECT u.*, o.* FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE u.email = $1;

-- In CI: run EXPLAIN and compare against stored baseline
-- Tools: pgreplay (PostgreSQL), pt-query-digest (MySQL)
```

### Pro Tip #6: Expression Indexes for JSON (Middle Ground)

```sql
-- Instead of moving the column or full GIN index:
CREATE INDEX idx_users_plan ON users ((metadata->>'plan'));
-- This gives B-Tree performance on a JSON-extracted value
-- But you still lose type safety and constraints
```

> **Use this sparingly** — it's a temporary bridge, not a long-term solution.

### Pro Tip #7: Combine Optimistic + Pessimistic Locking

For high-concurrency, high-contention scenarios:
1. Try optimistic locking first (check version column)
2. On conflict, escalate to `FOR UPDATE` (pessimistic lock)
3. This gives you the speed of optimistic locking with the safety of pessimistic locking

### Pro Tip #8: Automate Restore Drills

```bash
# Use ephemeral cloud instances for restore tests
# Spin up → Restore → Verify → Tear down
# This should run at least quarterly

# Example with AWS RDS + EC2:
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier prod-db \
    --target-db-instance-identifier restore-test-$(date +%s) \
    --restore-time "2024-01-01T12:00:00"
```

### Pro Tip #9: Monitor Index Health

```sql
-- PostgreSQL: Track index bloat
SELECT schemaname, tablename, indexname,
       pg_size_pretty(pg_relation_size(i.indexrelid)) as index_size,
       pg_size_pretty((pg_stat_get_tuples_hot_updated(i.indexrelid) + 
                       pg_stat_get_tuples_inserted(i.indexrelid)) * 
                       pg_relation_size(i.indexrelid) / 
                       NULLIF(pg_stat_get_tuples_inserted(i.indexrelid), 0)) as estimated_bloat
FROM pg_stat_user_indexes i
JOIN pg_class c ON c.oid = i.indexrelid
WHERE pg_stat_get_tuples_hot_updated(i.indexrelid) > 0;
```

### Pro Tip #10: Use Partial Indexes for Common Filters

```sql
-- Instead of indexing all rows:
CREATE INDEX idx_orders_pending ON orders(created_at DESC) 
WHERE status = 'pending';
-- This index is smaller, faster to maintain, and used for the most common query
```

---

## Test Your Understanding

> **Test yourself!** These 15 questions cover the core concepts. Answers are provided at the bottom.

### Questions

1. **What does an index do?** (Beginner)
   a) Stores a copy of the entire table
   b) Creates a sorted lookup structure for faster row retrieval
   c) Encrypts sensitive data
   d) Automatically optimizes queries

2. **When is `SELECT *` acceptable?** (Beginner)
   a) Always in production code
   b) In ad-hoc debugging queries only
   c) In all API responses
   d) Always in production

3. **What ACID property ensures all statements in a transaction succeed or fail together?** (Beginner)
   a) Consistency
   b) Isolation
   c) Atomicity
   d) Durability

4. **Which of these is NOT a valid reason to denormalize?** (Intermediate)
   a) Read-heavy workload with slow joins
   b) Rarely-changed data that's frequently displayed
   c) "I'll normalize later" for a prototype
   d) Acceptable staleness for read performance

5. **What does `EXPLAIN ANALYZE` show that `EXPLAIN` does not?** (Intermediate)
   a) Estimated costs
   b) Actual execution time and row counts
   c) SQL syntax errors
   d) Missing indexes

6. **In the N+1 problem, what does N represent?** (Beginner)
   a) The number of database tables
   b) The number of rows in the result set
   c) The number of related queries run per parent row
   d) The number of columns selected

7. **When should you use a JSON column instead of a real column?** (Intermediate)
   a) When you filter on the value frequently
   b) When you sort by the value regularly
   c) When the value is optional and rarely queried
   d) When you need type constraints

8. **What is the minimum frequency for testing backup restores?** (Intermediate)
   a) Daily
   b) Weekly
   c) Monthly
   d) Quarterly

9. **Which isolation level prevents dirty reads but allows non-repeatable reads?** (Advanced)
   a) Read Uncommitted
   b) Read Committed
   c) Repeatable Read
   d) Serializable

10. **What does `FOR UPDATE` do?** (Intermediate)
    a) Deletes the locked row
    b) Locks the row until the transaction ends
    c) Marks the row for archival
    d) Forces a re-read of the row

11. **Which is the BEST strategy for adding a foreign key to a table with existing data?** (Advanced)
    a) `ALTER TABLE ... ADD FOREIGN KEY ...` directly
    b) `NOT VALID` constraint, clean data, then `VALIDATE CONSTRAINT`
    c) Drop and recreate the table
    d) Skip the constraint — handle it in application code

12. **What is "write amplification" caused by over-indexing?** (Intermediate)
    a) Queries return too much data
    b) Each write must update multiple indexes, slowing down writes
    c) Memory usage increases
    d) Queries become non-deterministic

13. **In PostgreSQL, which command refreshes table statistics?** (Intermediate)
    a) `REFRESH STATISTICS`
    b) `UPDATE STATISTICS`
    c) `ANALYZE`
    d) `VACUUM`

14. **What is the primary security risk of `SELECT *`?** (Intermediate)
    a) It's too slow
    b) It selects all columns including potentially sensitive ones not intended for exposure
    c) It uses too much memory
    d) It causes deadlocks

15. **For a high-concurrency ticket booking system, which concurrency control is BEST?** (Advanced)
    a) No locking at all
    b) `FOR UPDATE` pessimistic locking
    c) Optimistic locking with version column and retry
    d) Serializable isolation

### Answers

1. **b** — Indexes create sorted lookup structures (B-Trees) for O(log n) access
2. **b** — Only for ad-hoc debugging; never in production query paths
3. **c** — Atomicity ensures all-or-nothing execution
4. **c** — "I'll normalize later" is a common excuse but denormalization should be measured, not guessed
5. **b** — `ANALYZE` shows actual runtime, row counts, and buffer usage
6. **c** — N queries for N related items (e.g., N orders for N users)
7. **c** — JSON is for optional, rarely-queried data
8. **d** — Quarterly restore drills are the industry standard minimum
9. **b** — Read Committed prevents dirty reads but allows non-repeatable reads
10. **b** — `FOR UPDATE` locks rows until COMMIT/ROLLBACK
11. **b** — Add `NOT VALID`, clean data, then `VALIDATE CONSTRAINT` to avoid downtime
12. **b** — Over-indexing causes each write to update multiple indexes
13. **c** — `ANALYZE` refresh table statistics for the query planner
14. **b** — Sensitive columns (password_hash, ssn) can be silently exposed
15. **c** — Optimistic locking with retry balances correctness and throughput

---

## Common Interview Questions

> These questions are commonly asked in senior-level database, backend, and system design interviews.

### Database Fundamentals

**Q1: What's the difference between a B-Tree index and a Hash index?**
A: B-Tree indexes support equality AND range queries, and maintain sorted order. Hash indexes support ONLY exact equality lookups. B-Tree is the default because it's more versatile. Hash indexes are faster for exact lookups but can't handle ranges or sorting.

**Q2: Explain the difference between `EXPLAIN` and `EXPLAIN ANALYZE`.**
A: `EXPLAIN` shows the query plan the planner *would* use (estimates). `EXPLAIN ANALYZE` actually *executes* the query and shows real runtime statistics — actual time, row counts, and buffer usage. Always use `ANALYZE` for real performance insights.

**Q3: What are the ACID properties and why do they matter?**
A: Atomicity (all-or-nothing), Consistency (valid state transitions), Isolation (concurrent transaction safety), Durability (committed changes survive crashes). They matter because they're the foundation of data reliability — without them, financial systems, e-commerce, and any data-critical application would be impossible.

### Schema Design

**Q4: When would you choose to denormalize?**
A: When read performance significantly outweighs write performance, when data changes infrequently, or when a specific read query requires joining 3+ tables on large datasets. Always measure first — denormalization is a trade-off, not a default.

**Q5: What's a foreign key and why is it important?**
A: A foreign key is a column (or set of columns) that references the primary key of another table. It enforces referential integrity — preventing orphaned records. It's important because application bugs can't bypass it, making it your last line of defense against data corruption.

**Q6: What are the N+1 query problem and how do you fix it?**
A: N+1 is when an application makes 1 query to fetch N parent records, then N queries to fetch related data for each — resulting in N+1 total queries. Fix it with eager loading (`JOIN`, `include`, `prefetch_related`) or by writing a single JOIN query.

### Concurrency and Transactions

**Q7: What is a race condition, and how can databases prevent it?**
A: A race condition occurs when two concurrent operations read data, then both write based on what they read, causing lost updates or inconsistent state. Databases prevent this with locking (`FOR UPDATE`), optimistic concurrency (version columns), or atomic conditional updates.

**Q8: Explain the difference between pessimistic and optimistic locking.**
A: Pessimistic locking (`FOR UPDATE`) locks rows immediately, blocking other transactions until the lock is released. Optimized for high-contention scenarios. Optimistic locking uses a version column — if the version changed between read and write, the update fails and the application retries. Optimized for high-concurrency, low-contention scenarios.

**Q9: What isolation level would you use for a financial transaction?**
A: `SERIALIZABLE` for maximum correctness, or `REPEATABLE READ` with explicit `FOR UPDATE` locking for a balance of correctness and performance. The choice depends on the acceptable trade-off between consistency and throughput.

### Performance Optimization

**Q10: How do you identify a missing index?**
A: Run `EXPLAIN ANALYZE` on slow queries. Look for `Seq Scan` on tables with many rows (>100K). Add an index on the `WHERE` or `JOIN` column. After adding the index, re-run `EXPLAIN ANALYZE` to verify the plan changed to `Index Scan`.

**Q11: What is query plan regression and how do you detect it?**
A: Query plan regression is when a query that was fast suddenly becomes slow because the database's query planner chose a different (worse) execution plan. Detect it by storing `EXPLAIN ANALYZE` output for critical queries and comparing against baselines in CI/CD.

**Q12: What's the difference between `LIKE '%term'` and `ILIKE '%term'`?**
A: `LIKE` is case-sensitive, `ILIKE` is case-insensitive (PostgreSQL). Both perform full table scans because the leading wildcard prevents index usage. For case-insensitive prefix matching, use a trigram index (`CREATE EXTENSION pg_trgm; CREATE INDEX ... USING gin(column gin_trgm_ops)`).

### Operations and Reliability

**Q13: How do you test that your backups actually work?**
A: Schedule quarterly restore drills in an isolated environment. Restore the backup, run data integrity checks (row counts, checksums), verify application functionality against the restored data, document restore time, and tear down the test environment. An untested backup is a false sense of security.

**Q14: What is point-in-time recovery and when would you use it?**
A: Point-in-time recovery (PITR) lets you restore a database to a specific timestamp, using base backups plus transaction logs/WAL. Use it when you need to recover from a bad deploy, accidental `DELETE` without `WHERE`, or data corruption — without losing more recent legitimate data.

**Q15: How do you prevent the N+1 problem from reaching production?**
A: Enable query logging in development and count queries per request. Use APM tools (Datadog, New Relic) that flag N+1 patterns automatically. Add query count assertions in integration tests. Make it a code review checklist item.

---

## Question Bank (50+ Questions)

### Beginner Level (Questions 1-17)

**Q1: What is the primary purpose of a database index?**
A: To provide a sorted lookup structure (typically a B-Tree) that allows the database to find rows quickly without scanning the entire table, reducing lookup time from O(n) to O(log n).

**Q2: What does O(n) vs O(log n) mean for database queries?**
A: O(n) means the database scans every row (sequential scan). O(log n) means it uses an index to jump to the correct row. For 1 million rows, O(n) scans 1M rows while O(log n) scans ~20 rows.

**Q3: When should you NOT create an index?**
A: When the column has low cardinality (e.g., boolean), when the table is small (<1000 rows), when the column is rarely used in WHERE/JOIN/ORDER BY, or when the table is write-heavy and the index overhead outweighs read benefits.

**Q4: What is the N+1 query problem?**
A: An anti-pattern where an application makes 1 query to fetch a list of items, then N queries to fetch related data for each item, resulting in 1+N total queries instead of 1-2.

**Q5: What are the ACID properties?**
A: Atomicity (all-or-nothing), Consistency (valid state), Isolation (concurrent safety), Durability (survives crashes).

**Q6: What is a foreign key constraint?**
A: A constraint that enforces referential integrity between tables by ensuring that values in a column reference valid primary keys in another table.

**Q7: What's wrong with `SELECT *` in production?**
A: It transfers unnecessary columns (wasting bandwidth, CPU, memory), risks exposing sensitive data, breaks caching efficiency, and makes schema changes risky (silent behavior changes).

**Q8: What is the difference between a primary key and a unique key?**
A: A primary key uniquely identifies each row and cannot be NULL. A unique key also enforces uniqueness but can have NULL values (in standard SQL, multiple NULLs are allowed in a UNIQUE column).

**Q9: What does `EXPLAIN ANALYZE` do?**
A: It executes the query and displays the actual execution plan — including real runtime, row counts, and buffer usage — allowing you to compare actual vs. estimated performance.

**Q10: What is a sequential scan?**
A: A scan that reads every row in a table sequentially. It's the slowest possible access method for large tables, but may be the only option if no suitable index exists.

**Q11: What are the two most common index types?**
A: B-Tree (default, supports equality and range queries) and Hash (supports only exact equality lookups).

**Q12: What is a composite index?**
A: An index on multiple columns. It can serve queries that filter on the leftmost column(s), making it efficient for multi-column WHERE clauses.

**Q13: What happens if two transactions try to write to the same row simultaneously?**
A: The database uses locking to serialize access. One transaction gets the lock and proceeds; the other waits. If they deadlock, the database detects and kills one transaction, which then rolls back.

**Q14: What is data type coercion?**
A: The implicit conversion of one data type to another by the database. It can cause performance issues if it prevents index usage (e.g., comparing a string column to an integer).

**Q15: What is the purpose of a `CHECK` constraint?**
A: To enforce domain integrity by restricting the values that can be inserted into a column (e.g., `CHECK (price > 0)`).

**Q16: What is the difference between `DELETE` and `TRUNCATE`?**
A: `DELETE` removes rows one by one (can be rolled back, fires triggers, slower). `TRUNCATE` removes all rows at once (DDL operation, faster, cannot be rolled back in some databases, doesn't fire row-level triggers).

**Q17: What is a view in SQL?**
A: A virtual table defined by a stored query. It doesn't store data itself but presents the results of a query as a table. Useful for simplifying complex queries, hiding sensitive columns, or enforcing row-level security.

### Intermediate Level (Questions 18-34)

**Q18: How does the order of columns in a composite index matter?**
A: A composite index `(A, B)` can be used for queries filtering on `A` alone, or on `A` AND `B`. But it CANNOT be used for queries filtering on `B` alone — the index is ordered by `A` first.

**Q19: What is a partial index and when would you use one?**
A: A partial index indexes only a subset of rows (e.g., `WHERE is_deleted = false`). Use it when most queries filter on a specific condition, reducing index size and maintenance overhead.

**Q20: Explain the difference between `SERIALIZABLE` and `REPEATABLE READ` isolation levels.**
A: `REPEATABLE READ` prevents dirty reads and non-repeatable reads but allows phantom reads. `SERIALIZABLE` prevents all three (dirty, non-repeatable, phantom) by effectively serializing transactions — the highest isolation level.

**Q21: What is a "covering index" (or "index-only scan")?**
A: An index that contains all the columns needed by a query, allowing the database to satisfy the query entirely from the index without touching the table data. This eliminates expensive table lookups.

**Q22: What is the difference between `INNER JOIN` and `LEFT JOIN`?**
A: `INNER JOIN` returns only rows that match in both tables. `LEFT JOIN` returns all rows from the left table, plus matching rows from the right table (NULL if no match).

**Q23: What is a "deadlock" and how does the database handle it?**
A: A deadlock occurs when two or more transactions are waiting for each other to release locks. The database detects deadlocks and kills one transaction (the "victim"), which rolls back. The application should retry the failed transaction.

**Q24: What is the purpose of table statistics?**
A: Statistics help the query planner estimate how many rows a query will return, enabling it to choose the best execution plan. Outdated statistics lead to poor plan choices.

**Q25: What is the difference between `CREATE INDEX` and `CREATE INDEX CONCURRENTLY`?**
A: `CREATE INDEX` locks the table, blocking writes. `CREATE INDEX CONCURRENTLY` builds the index without locking the table, allowing concurrent writes — but it's slower and can't be used inside a transaction block.

**Q26: What is a "covering index" and how do you create one?**
A: A covering index includes all columns a query needs (data columns + filter columns). In SQL Server: `CREATE INDEX ... INCLUDE (col1, col2)`. In PostgreSQL, composite indexes can serve as covering indexes.

**Q27: What is the "lazy loading" vs "eager loading" pattern?**
A: Lazy loading defers loading related data until it's accessed (causing N+1). Eager loading fetches related data upfront (JOIN or separate batched query), avoiding N+1.

**Q28: What is WAL (Write-Ahead Logging)?**
A: WAL is a PostgreSQL (and other databases') durability mechanism where every change is written to a log before being applied to the data files. It enables crash recovery and point-in-time recovery.

**Q29: What is the 3-2-1 backup rule?**
A: 3 copies of data, on 2 different media, with 1 copy offsite. This ensures protection against hardware failure, data corruption, and site-level disasters.

**Q30: What is `ANALYZE` in PostgreSQL and when should you run it?**
A: `ANALYZE` collects statistics about table contents for the query planner. Run it after large data changes (bulk imports, massive updates/deletes) or rely on autovacuum to do it automatically.

**Q31: What is the difference between a "materialized view" and a regular view?**
A: A regular view is a saved query — re-executed every time it's accessed. A materialized view stores the query result physically — faster to read but can become stale (requires refresh).

**Q32: What is "row-level security" (RLS)?**
A: A PostgreSQL feature that filters rows based on policies. It enforces tenant isolation or data access rules at the database level, preventing data leakage even if application code has bugs.

**Q33: What are "triggers" and when would you use them?**
A: Triggers are stored procedures that automatically execute on table events (INSERT, UPDATE, DELETE). Use them for audit logging, maintaining denormalized summary tables, or enforcing complex constraints.

**Q34: What is the difference between `DELETE` and `DROP`?**
A: `DELETE` removes rows from a table but keeps the table structure. `DROP` removes the entire table (structure + data). `DELETE` can be rolled back; `DROP` typically cannot (depends on database).

### Advanced Level (Questions 35-50+)

**Q35: What is "write amplification" and how is it related to indexing?**
A: Write amplification is when a single logical write operation causes multiple physical writes. Over-indexing is a major cause: each INSERT/UPDATE must update the table row AND every index, multiplying write I/O.

**Q36: What is the "optimizer" and what does it do?**
A: The query optimizer is the database component that takes a SQL query and produces an execution plan. It evaluates different possible plans (join orders, access methods) and picks the one with the lowest estimated cost.

**Q37: How do you identify and fix a "nested loop" that's running too slowly?**
A: First, check if the join columns are indexed — a Nested Loop on unindexed columns is expensive. If the table is large, consider whether a Hash Join or Merge Join would be better. Run `ANALYZE` to ensure the planner has accurate statistics.

**Q38: What is "lock escalation"?**
A: Lock escalation is when a database converts many fine-grained locks (row-level) into fewer coarse-grained locks (table-level) to reduce lock overhead. This can cause unexpected blocking.

**Q39: What is the difference between "pessimistic" and "optimistic" concurrency control?**
A: Pessimistic (e.g., `FOR UPDATE`) assumes conflicts will happen and prevents them with locks. Optimistic (version column) assumes conflicts are rare and checks at commit time, retrying on conflict.

**Q40: What is a "covering index" and how do you design one?**
A: A covering index contains all columns needed by a query, enabling an "index-only scan." Design it by including the WHERE columns first, then the SELECT columns as included columns (if the database supports it, like SQL Server `INCLUDE`).

**Q41: What is "cursor" usage and why is it dangerous?**
A: Cursors process rows one at a time, which is extremely slow (like N+1 but in SQL). They're often a sign of procedural thinking applied to a declarative language. Use set-based operations instead.

**Q42: What is "partitioning" and when should you use it?**
A: Partitioning splits a large table into smaller, independent pieces based on a key (range, list, hash). Use it for very large tables (>100M rows) where queries can target specific partitions, improving both query and maintenance performance.

**Q43: What is the "CAP theorem" and how does it relate to databases?**
A: The CAP theorem states that in a distributed system, you can only have two of three: Consistency, Availability, and Partition tolerance. It influences database choices — e.g., choosing eventual consistency (AP) over strong consistency (CP).

**Q44: What is "MVCC" (Multi-Version Concurrency Control)?**
A: MVCC is a concurrency control method where each transaction sees a snapshot of the database at a specific point in time. It allows readers and writers to proceed without blocking each other. PostgreSQL and Oracle use MVCC.

**Q45: What is the difference between a "clustered" and "non-clustered" index?**
A: A clustered index determines the physical order of rows in the table (can only have one per table). A non-clustered index is a separate structure pointing to rows. In PostgreSQL, the `CLUSTER` command physically reorders a table by an index.

**Q46: What is "query cost estimation" and why can it be wrong?**
A: The query planner estimates the cost of each possible execution plan based on table statistics. It can be wrong when statistics are stale (after bulk operations), leading to suboptimal plan choices.

**Q47: What is "predicate pushdown"?**
A: An optimization where the database pushes parts of the query (predicates) down to the storage layer, so the storage engine only returns matching rows. This reduces the amount of data transferred between storage and the database engine.

**Q48: What is the "slow query log" and how do you use it?**
A: A log of all queries that exceed a specified duration threshold. Use it to identify slow queries in production, analyze their patterns, and prioritize optimization efforts.

**Q49: What is "connection pooling" and why is it important?**
A: Connection pooling reuses established database connections instead of creating new ones per request. It's important because database connections are expensive to create — too many concurrent connections exhaust database resources and degrade performance.

**Q50: What is "transaction log" (WAL/redo log) and why is it critical?**
A: The transaction log records all changes before they're applied to data files. It's critical for durability (surviving crashes) and for point-in-time recovery — without it, you can only restore to the last full backup.

**Q51: Explain the "double-write buffer" problem.**
A: A "torn page" occurs when a crash interrupts a partial page write, leaving the data file in an inconsistent state. Databases use a double-write buffer (write to sequential log first, then to data pages) to prevent this.

**Q52: What is a "query plan hint"?**
A: A directive given to the optimizer to force a specific execution strategy (e.g., `USE INDEX`, `JOIN` order). Hints should be used sparingly as a last resort — fixing the root cause (indexes, statistics) is always better.

**Q53: What is "schema migration" and what are the risks?**
A: Schema migration is changing database structure (columns, tables, indexes). Risks include: downtime (table rebuilds), data loss (dropping columns), lock contention (blocking writes), and backward compatibility issues (deploying changes to running applications).

**Q54: What is "read replicas" and when should you use them?**
A: Read replicas are copies of the database that can handle read queries, offloading the primary database. Use them when read traffic exceeds what the primary can handle, or when you need geographic proximity to readers.

**Q55: What is "sharding" and how does it differ from replication?**
A: Sharding splits data across multiple database instances (horizontal partitioning across servers), increasing write capacity and storage. Replication copies the same data across instances (for redundancy and read scaling, but doesn't increase write capacity).

---

## Self-Assessment Checklist

### Before You Ship: The Database Quality Checklist

Mark each item as ✅ (done) or ❌ (needs work):

#### Query Quality
- [ ] All production queries use explicit column names (no `SELECT *`)
- [ ] All frequently-run queries are tested with `EXPLAIN ANALYZE`
- [ ] No `Seq Scan` on tables with >100K rows (unless intentional)
- [ ] All N+1 patterns have been identified and fixed
- [ ] Query result sizes are reasonable (no unnecessary large BLOB/TEXT columns)
- [ ] Pagination is implemented for large result sets

#### Index Health
- [ ] All high-frequency `WHERE` columns have indexes
- [ ] All `JOIN` columns have indexes (foreign keys are indexed)
- [ ] No unused indexes exist (checked with `pg_stat_user_indexes`)
- [ ] Composite indexes have correct column order for query patterns
- [ ] Partial indexes exist for common filter conditions
- [ ] Low-cardinality columns are NOT indexed (unless combined with others)

#### Transaction & Concurrency
- [ ] All multi-step write operations are wrapped in transactions
- [ ] Transactions are kept short (no external API calls inside)
- [ ] Concurrency-critical paths use `FOR UPDATE` or optimistic locking
- [ ] Stock/price updates use atomic `UPDATE ... WHERE` conditions
- [ ] Deadlock detection and retry logic exists in the application
- [ ] Lock timeouts are configured (prevent infinite waits)

#### Schema Design
- [ ] All relationships are modeled with foreign key constraints
- [ ] `NOT NULL` constraints are on all required fields
- [ ] `UNIQUE` constraints on unique identifiers (email, etc.)
- [ ] `CHECK` constraints enforce business rules at the database level
- [ ] Data types are appropriate (no numbers stored as strings)
- [ ] JSON columns are only used for genuinely unstructured/optional data
- [ ] Column names are consistent and descriptive

#### Security
- [ ] Sensitive columns are not indexed (no SSN, password_hash indexes)
- [ ] `SELECT *` is not used in any API response paths
- [ ] Field-level allowlisting is in place for API responses
- [ ] Database users follow the principle of least privilege
- [ ] Row-level security is enabled for multi-tenant applications
- [ ] Backups are encrypted at rest

#### Operations
- [ ] Backups run automatically and have been verified restored
- [ ] Backups are stored offsite/cross-region
- [ ] Point-in-time recovery is possible (WAL/transaction logs enabled)
- [ ] A documented disaster recovery plan exists
- [ ] Query performance is monitored in production
- [ ] Database metrics are part of alerting dashboards
- [ ] Table statistics are refreshed regularly (or auto-analyze is enabled)

#### Testing
- [ ] Integration tests verify transaction rollback behavior
- [ ] Concurrency tests verify no race conditions under load
- [ ] Query count tests detect N+1 patterns (assert < N queries)
- [ ] Schema constraint tests verify FKs, CHECKs, UNIQUEs are enforced
- [ ] Backup restore tests run at least quarterly
- [ ] Performance regression tests run in CI/CD

---

## Further Reading & Resources

### Official Documentation

| Database | Documentation | Key Sections |
|---|---|---|
| **PostgreSQL** | [postgresql.org/docs](https://www.postgresql.org/docs/) | Performance Tips, Locking, Backup/Restore, WAL, Indexing |
| **MySQL** | [dev.mysql.com/doc](https://dev.mysql.com/doc/) | Optimization, InnoDB, Backup, Replication |
| **SQL Server** | [learn.microsoft.com/sql](https://learn.microsoft.com/sql/) | Query Store, Indexing, Transaction Isolation |
| **SQLite** | [sqlite.org/docs](https://www.sqlite.org/docs.html) | Query Optimization, Transactions |

### Books

| Title | Author | Why Read It |
|---|---|---|
| **Designing Data-Intensive Applications** | Martin Kleppmann | The definitive guide to databases, distributed systems, and reliability |
| **High Performance MySQL** | Baron Schwartz et al. | Deep dive into MySQL optimization, indexing, and scaling |
| **Effective PostgreSQL** | Hans-Jürgen Schönig | Advanced PostgreSQL techniques and best practices |
| **SQL Antipatterns** | Bill Karwin | 25 common database design antipatterns and how to avoid them |
| **The Art of SQL** | Stéphane Faroult | How to think about SQL queries, optimization, and design |

### Tools

| Tool | Purpose | Links |
|---|---|---|
| **EXPLAIN ANALYZE** | Query plan analysis | Built into all major DBs |
| **pgBadger** | PostgreSQL log analyzer | [pganalyze.com/pgbadger](https://pganalyze.com/pgbadger) |
| **pt-query-digest** | MySQL query analysis | [Percona Toolkit](https://www.percona.com/doc/percona-toolkit/LATEST/) |
| **Datadog Database Monitoring** | APM for databases | [datadoghq.com](https://www.datadoghq.com/) |
| **pgAdmin** | PostgreSQL GUI | [pgadmin.org](https://www.pgadmin.org/) |
| **TablePlus** | Multi-DB GUI | [tableplus.com](https://tableplus.com/) |

### Online Courses

| Course | Platform | Focus |
|---|---|---|
| **PostgreSQL for Beginners to Expert** | Udemy | Full PostgreSQL from basics to advanced |
| **SQL Optimization for Beginners** | GoTime | Query performance, indexing, EXPLAIN |
| **Database Design & Modeling** | Coursera | Schema design, normalization, relationships |
| **MySQL for Database Administrators** | Oracle University | Operations, backups, performance tuning |

### blogs & Websites

| Resource | URL | Highlights |
|---|---|---|
| **Use The Index, Luke** | [use-the-index-luke.com](https://use-the-index-luke.com/) | The best resource on SQL indexing and performance |
| **High Scalability** | [highscalability.com](http://highscalability.com/) | Case studies from Netflix, Twitter, Facebook, etc. |
| **Percona Blog** | [percona.com/blog](https://www.percona.com/blog) | MySQL/PostgreSQL performance, scaling |
| **Cybertec Blog** | [cybertec.tv](https://www.cybertec.tv/) | PostgreSQL internals and optimization |
| **AWS Database Blog** | [aws.amazon.com/blogs/database](https://aws.amazon.com/blogs/database/) | Cloud database best practices |

### GitHub Repositories

| Repo | Description |
|---|---|
| **killmast/abomination** | A deliberately bad database schema to learn from |
| **HeapSpy/Java-Multi-Threading** | Concurrency testing examples |
| **pgexcellent/pg_stat_statements** | PostgreSQL query analysis |

---

## Learning Path Recommendations

### For Junior Developers (0-2 years)

```mermaid
flowchart TD
    A[Learn SQL Basics] --> B[Understand Indexes]
    B --> C[Write EXPLAIN]
    C --> D[Basic Transactions]
    D --> E[ORM Patterns]
    E --> F[Simple Schema Design]
    F --> G[Mistakes #1, #2, #6]
    
    style A fill:#bbf
    style G fill:#dfd
```

**Resources:**
- Practice writing SQL queries on [HackerRank](https://www.hackerrank.com/domains/tutorials/10-days-of-javascript) or [LeetCode Database](https://leetcode.com/problemset/database/)
- Read: "SQL in 10 Minutes" by Ben Forta
- Read: Use The Index, Luke (full website)
- Project: Build a simple CRUD app with proper indexes and EXPLAIN verification

**Focus Mistakes:** #1 (Indexes), #2 (SELECT *), #6 (EXPLAIN), #3 (Basic Transactions)

### For Mid-Level Developers (2-5 years)

```mermaid
flowchart TD
    A[Master Transaction Isolation] --> B[Concurrency Control]
    B --> C[Advanced Schema Design]
    C --> D[N+1 Detection]
    D --> E[Backup/Restore]
    E --> F[Migration Strategies]
    F --> G[Mistakes #3-10]
    
    style A fill:#bbf
    style G fill:#ffd
```

**Resources:**
- Read: "Designing Data-Intensive Applications" (Chapters 7, 9, 10)
- Practice: Concurrency testing with JMeter or custom scripts
- Project: Audit an existing codebase using the migration guide from this tutorial
- Take: Database performance course on your platform of choice

**Focus Mistakes:** #3 (Transactions), #4 (Schema), #5 (Normalization), #7 (N+1), #8 (JSON), #9 (Backups), #10 (Concurrency)

### For Senior Engineers (5+ years)

```mermaid
flowchart TD
    A[Database Internals] --> B[Query Planner Deep Dive]
    B --> C[Distributed Databases]
    C --> D[Scaling Strategies]
    D --> E[Incident Response]
    E --> F[Mastery of All 10 Mistakes]
    F --> G[Lead Team Database Standards]
    
    style A fill:#bbf
    style G fill:#fdd
```

**Resources:**
- Read: "Database Internals" by Alex Petrov
- Study: PostgreSQL/MySQL source code for query planner
- Take: Advanced PostgreSQL/MySQL performance courses
- Practice: Simulating and debugging deadlock scenarios
- Lead: Database architecture decisions for your team

**Focus:** All 10 mistakes + advanced scaling, distributed systems, and incident response

### Suggested Learning Sequence

| Week | Topic | Key Mistake(s) | Outcome |
|---|---|---|---|
| 1 | SQL Basics + Indexing | #1 | Write efficient queries with proper indexes |
| 2 | Query Analysis + EXPLAIN | #6 | Read and interpret execution plans |
| 3 | Transactions + ACID | #3 | Ensure data consistency under all conditions |
| 4 | Schema Design | #4 | Design robust, normalized schemas with constraints |
| 5 | Query Optimization | #2, #7 | Eliminate N+1 and SELECT * issues |
| 6 | Concurrency Control | #10 | Prevent race conditions and deadlocks |
| 7 | JSON + Normalization Trade-offs | #5, #8 | Know when to denormalize or use JSON |
| 8 | Backup + Recovery | #9 | Ensure reliable disaster recovery |
| 9 | CI/CD Integration | All | Automate database quality checks |
| 10 | Comprehensive Audit | All 10 | Audit and fix a real codebase |

---

## Next Steps

### After Completing This Tutorial

1. **Audit one of your projects** using the [Migration Guide](#migration-guide-auditing-an-existing-codebase)
   - Run `EXPLAIN ANALYZE` on the top 10 slowest queries
   - Search for `SELECT *` and N+1 patterns
   - Check for unused indexes and missing foreign keys

2. **Set up monitoring** if you haven't already
   - Enable `pg_stat_statements` (PostgreSQL) or slow query log (MySQL)
   - Set up alerts for query regressions
   - Monitor index usage and table bloat

3. **Schedule a backup restore drill**
   - Read [Mistake #9](#mistake-9-skipping-backup-and-restore-testing)
   - Document the process
   - Schedule it quarterly in your calendar

4. **Write tests for database decisions**
   - Add query count assertions to prevent N+1 regressions
   - Write concurrency tests for critical operations
   - Add transaction rollback tests for multi-step operations

5. **Share this knowledge with your team**
   - Run a lunch-and-learn session
   - Add database best practices to your code review checklist
   - Implement a "no N+1" rule for new code

### Recommended Follow-Up Tutorials

| Tutorial | Why Next? |
|---|---|
| **Database Indexing - A Deep Dive Into Why Queries Go Fast or Grind to a Halt** | Deepens Mistake #1 with B-Tree internals, BRIN, GIN, and index scan types |
| **Solving the N+1 Problem in Spring Data JPA - The Complete Production Guide** | Practical, production-focused guide to N+1 in the most popular Java ORM |
| **Top 10 Database Scaling Strategies** | What comes after fixing these 10 mistakes — scaling to millions of users |
| **PostgreSQL Row Level Security - Baking Tenant Isolation Into the Database** | Deep dive into the security technique mentioned in Mistake #10 |
| **The Outbox Pattern - Bulletproof Message Delivery in Microservices** | How transactions (Mistake #3) enable reliable distributed systems |

---

## Final Thoughts

Every backend system — no matter how well-architected the application layer is — eventually becomes a **database problem**. Frameworks change, languages change, but the fundamentals of indexing, schema design, transactions, and concurrency remain the foundation everything else is built on.

### The Journey Ahead

> **The difference between a junior and a senior engineer is not that seniors write fewer bugs. It's that they know which bugs are expensive enough to prevent upfront, and which tradeoffs are worth making.**

These 10 mistakes are not theoretical — they're battle scars from real production systems. The e-commerce platform that lost $50,000 to overselling. The SaaS company that went down for 3 hours because a backup never worked. The startup that couldn't scale because every query did a sequential scan.

### Your Call to Action

1. **Bookmark this tutorial** — you'll reference it in code reviews
2. **Share it with your team** — make database quality a team responsibility
3. **Apply one fix this week** — start with the easiest: replace `SELECT *` with explicit columns
4. **Verify your backups** — schedule a restore drill before you forget this reading
5. **Measure, don't guess** — always run `EXPLAIN ANALYZE`

> **Databases are not just storage engines. They are consistency engines. They are performance engines. They are scalability engines. Treat every schema and query decision as an architectural choice — not an afterthought.**

---

✅ **Tutorial Complete!** 

**Total mistakes covered:** 10
**Mermaid diagrams:** 15+
**Code examples:** 40+ across 4 languages
**Practice exercises:** 3 with full solutions
**Question bank:** 55 questions
**Test your understanding:** 15 questions with answers
**Common interview questions:** 15 questions with answers
**Self-assessment checklist:** 30+ items
**Estimated reading time:** 45-55 minutes

**Last Updated:** August 22, 2026
**Version:** 1.0

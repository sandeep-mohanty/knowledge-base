# PostgreSQL JSON vs JSONB: Which One Will Ruin Your Speed?

It started as a harmless data structure.

A developer stored logs as JSON in PostgreSQL. Everything worked fine for months — until suddenly, the queries slowed down. A single dashboard load took **approximately 18 seconds**. But it wasn’t the hardware. The culprit? A tiny, five-letter difference: **JSON vs JSONB**.

![PostgreSQL JSON vs JSONB Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*T6chgoSKfX25nmMOEHIWuw.png)

## ⚙️ The Hidden War Between JSON and JSONB
JSON and JSONB appear identical at a glance. Both let you store semi-structured data. Both support nested fields. Both are great for flexibility. But under the hood, they are completely different beasts.


![Storage Mechanics Comparison](https://miro.medium.com/v2/resize:fit:720/format:webp/1*1gtzmJnWdstF08HM5g0QLw.png)

## 🚀 The Real-World Impact
Here’s a quick benchmark example tested on PostgreSQL 16:

```sql
CREATE TABLE logs_json (data JSON);
36+CREATE TABLE logs_jsonb (data JSONB);

-- Insert 100k records
INSERT INTO logs_json 
SELECT json_build_object('user', i, 'status', 'ok') 
FROM generate_series(1, 100000) AS s(i);

INSERT INTO logs_jsonb 
SELECT jsonb_build_object('user', i, 'status', 'ok') 
FROM generate_series(1, 100000) AS s(i);

-- Query test
EXPLAIN ANALYZE SELECT * FROM logs_json WHERE data->>'user' = '99999';
EXPLAIN ANALYZE SELECT * FROM logs_jsonb WHERE data->>'user' = '99999';
```

**Result (approximate):**
* **JSON** → 350ms
* **JSONB** → 5ms (with GIN index)

That’s a **70x performance difference** — not because of the server specs, but because of the storage format.

## 💡 Practical Tip: When to Use Each
Choosing between JSON and JSONB shouldn’t be based on habit. It should be based on **intent**.

### ✅ Use JSONB when:
* You query inside JSON fields (e.g., filtering by keys or values).
* You need indexing (**GIN** or **GiST** indexes).
* You frequently update JSON fields.
* You store logs, analytics, or metadata.

**Example:**
```sql
CREATE INDEX idx_user_data ON logs_jsonb USING gin ((data->'user'));
```
Now PostgreSQL can search nested values almost as fast as native columns.

### ⚠️ Use JSON when:
* You rarely query inside the JSON (just store and retrieve).
* You need to preserve key order or duplicates.
* You’re handling large writes where insert speed matters.

**Example:**
Use JSON for storing raw webhook payloads, system logs, or audit trails that won’t be queried deeply.

## 🧠 Common Mistake: The “Everything JSONB” Syndrome
Many developers switch all JSON to JSONB assuming it’s always faster. But JSONB adds CPU cost during inserts — PostgreSQL must parse and convert it into binary form.

A developer once shared a story on the PostgreSQL mailing list where bulk inserts of 5 million JSONB records took **3× longer** than JSON because of the conversion overhead. For pure storage, JSON wins.

**The key lesson:** Don’t optimize blindly. Measure.

## 🔍 Real-World Example: Metadata at Scale
A SaaS company stored user preferences, UI settings, and feature flags inside a `preferences` column. Initially stored as JSON, queries to find users with a particular flag were painfully slow.

After migrating to JSONB and adding this index:
```sql
CREATE INDEX idx_flags ON users USING gin ((preferences jsonb_path_ops));
```
Query latency dropped from **800ms to 12ms**.

That single schema change saved them **$1,200/month** in database costs — no hardware upgrade needed.

## 🧰 Solution Framework: Choosing the Right Type

![Selection Decision Matrix](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*5t_y_6Y0o1H5E6D8A2U3-g.png)

**Rule of thumb:**
👉 **JSON for storage, JSONB for search.**

## 🧩 Indexing That Actually Works
To speed up JSONB, use GIN or GiST indexes wisely:

```sql
CREATE INDEX idx_jsonb_gin ON logs_jsonb USING gin (data);
```

For targeted queries, use path operators:
```sql
SELECT * FROM logs_jsonb WHERE data @> '{"status": "ok"}';
```

And for complex filters:
```sql
SELECT * FROM logs_jsonb WHERE data->'metrics'->>'latency'::int > 200;
```

PostgreSQL’s JSONB engine efficiently handles these through its binary structure — something plain JSON can’t do.

## ⚡ Final Thoughts
The JSON vs JSONB debate isn’t about which one is “better.” It’s about **trade-offs**.

* **JSON** is like raw text — light, fast to write, hard to search.
* **JSONB** is like compiled code — slower to write, lightning-fast to read.

Choose JSONB when performance and querying matter. Choose JSON when simplicity and raw ingestion speed matter. Pick wrong — and you’ll ruin your speed.
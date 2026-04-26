# 10 Regular Expressions in SQL That Instantly Solve Data Cleaning Problems
**High-impact regex patterns to standardize, validate, and sanitize messy data directly in SQL**

SQL regular expressions to solve common data cleaning challenges efficiently. These patterns standardize formats, extract values, and remove inconsistencies quickly.

---

### 1. Extract Emails Instantly From Large Logs
Most basic patterns fail on edge cases like `user+tag@gmail.com`. This extraction method instead.

**REGEX Pattern :-** `([a-zA-Z0–9._%+-]+@[a-zA-Z0–9.-]+\.[a-zA-Z]{2,})`

Works for 99% of valid email formats, unlike naive `LIKE '%@%.%'`. It prevents broken extractions from logs, forms, or scraped data.

**Pros:** High accuracy for complex email formats. The `WHERE` clause acts as a filter, saving wasted extraction runs and compute costs.  
**Cons:** Computationally expensive if run on every row of a massive dataset without filtering first. Fails on extremely obscure RFC 5322 edge cases.

```sql
-- Filter before extracting to save compute
SELECT REGEXP_SUBSTR(text_column, '([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})') AS extracted_email
FROM user_logs
WHERE REGEXP_LIKE(text_column, '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}');
```

---

### 2. Clean Phone Numbers in One Step
Phone data formats vary wildly: `(123) 456–7890`, `123.456.7890`, `+1 123–456–7890`. Standardize them instantly using a two-step approach.



**Why it matters:** Eliminates the need for dozens of nested `REPLACE()` functions, cutting pipeline validation time significantly.

**Pros:** Highly robust against unexpected characters. Simplifies downstream formatting logic.  
**Cons:** Blindly strips international country codes if not handled conditionally beforehand.

```sql
WITH Cleaned AS (
  SELECT REGEXP_REPLACE(phone_column, '[^0-9]', '') AS num 
  FROM customers
)
SELECT CONCAT(SUBSTR(num, 1, 3), '-', SUBSTR(num, 4, 3), '-', SUBSTR(num, 7, 4)) AS formatted_phone
FROM Cleaned
WHERE LENGTH(num) = 10;
```

---

### 3. Fix Capitalization Without UDFs
Need “McDonald” instead of “MCDONALD”? Many SQL dialects lack an `INITCAP()` function, but regex handles Title Case conversion directly.

**Why it matters:** Standardizing names and addresses is a prerequisite for accurate group-bys and entity resolution.

**Pros:** Avoids the overhead of writing and executing Python or Java User Defined Functions. Keeps logic entirely within SQL.  
**Cons:** Dialect-dependent syntax. Complex names like “O’Connor” or “McDonald” require more intricate pattern matching to get perfect casing.

```sql
-- Converts "NEW YORK CITY" to "New York City"
-- Note: Syntax varies by dialect. This uses backreferences.
SELECT REGEXP_REPLACE(
  LOWER(ugly_column),
  '\b([a-z])',
  UPPER('\1')
) AS title_case_name
FROM user_inputs;
```

---

### 4. Validate URLs Like a Pro
Simple checks miss complex URLs with queries or paths like `https://sub.domain.co.uk/path?query=123`. Use a comprehensive matcher.

Catches malformed URLs early before they break downstream reporting pipelines or scraping scripts.

**Pros:** Highly accurate for standard web traffic data. Filters out noise effectively.  
**Cons:** Regex for perfect URI validation is notoriously massive. This practical version might miss rare top-level domains or specialized protocols.

```sql
SELECT url_column
FROM web_traffic
WHERE REGEXP_LIKE(
  url_column,
  '^(https?://)?([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$'
);
```

---

### 5. Extract Hashtags & Mentions
Parsing social media data requires pulling all `#tags` and `@mentions` without aggressively splitting strings.

**Why it matters:** Transforms unstructured text into array formats ready for immediate unnesting and analytical counting.

**Pros:** Extracts multiple instances per row cleanly into an array.  
**Cons:** Requires dialects supporting array extraction like `REGEXP_EXTRACT_ALL`. Standard `REGEXP_SUBSTR` only pulls the first match.

```sql
-- BigQuery/Presto array extraction example
SELECT
  text_column,
  REGEXP_EXTRACT_ALL(text_column, '(#[a-zA-Z0-9_]+)') AS hashtags,
  REGEXP_EXTRACT_ALL(text_column, '(@[a-zA-Z0-9_]+)') AS mentions
FROM social_feed;
```

---

### 6. Split Strings Without Libraries
Exploding delimited string like `"apple, banana, cherry"` with regular `SPLIT()` fails when spaces are inconsistent. Regex handles dirty delimiters.

**Why it matters:** Crucial for parsing dirty CSV dumps with irregular spacing directly in the database.

**Pros:** Much faster than pushing data to a Python pandas environment for pre-processing. Cleanly handles leading/trailing whitespace.  
**Cons:** Can be memory-intensive on massive text blocks. Array unnesting syntax varies heavily between SQL dialects.

```sql
-- Trims spaces around commas automatically during the split
SELECT unnested_items
FROM raw_data,
UNNEST(REGEXP_SPLIT_TO_ARRAY(csv_column, '\s*,\s*')) AS unnested_items;
```

---

### 7. Find Typos in Product Codes
SKUs follow strict patterns like “A123-B456”, but users input “A123B456” or “A123 B456”. Enforce format constraints at the query level.

**Why it matters:** Identifies data entry errors immediately. Over 90% of databases contain silent SKU formatting errors.

**Pros:** Perfect for data quality dashboards and `CHECK` constraints in table definitions.  
**Cons:** Rigid logic. If the business introduces a new SKU format, the regex must be updated immediately or valid data gets flagged.

```sql
SELECT sku_id, raw_input
FROM inventory_logs
WHERE NOT REGEXP_LIKE(raw_input, '^[A-Z]\d{3}-[A-Z]\d{3}$');
```

---

### 8. Remove Duplicate Words in Text
User comments often contain typos like “free free shipping shipping”. Condense repeated words dynamically.



**Why it matters:** Cleans up text for natural language processing pipelines or sentiment analysis without needing Python text-processing libraries.

**Pros:** Handles repetitive typos elegantly with backreferences.  
**Cons:** Engine support for backreferences and global flags varies. It might accidentally remove valid grammatical repetitions.

```sql
-- Before: "Final sale sale ends today today!"
-- After: "Final sale ends today!"
SELECT REGEXP_REPLACE(
  user_comment,
  '\b(\w+)\s+\1\b',
  '\1',
  'ig' -- Case insensitive, global replacement flags
) AS cleaned_comment
FROM feedback;
```

---

### 9. Parse JSON Keys Without json_decode()
If your SQL environment lacks native JSON support, or the JSON is technically invalid, extract values using regex.

**Why it matters:** Serves as a reliable fallback for parsing semi-structured logs or API responses when native JSON functions fail on malformed strings.

**Pros:** Bypasses strict JSON validation errors. Highly flexible.  
**Cons:** Brittle mapping. Changes in JSON serialization like unexpected line breaks or key renaming will break the extraction.

```sql
-- Extracts 29.99 from '{"item": "book", "price": 29.99}'
SELECT REGEXP_SUBSTR(
  json_text,
  '"price":\s*([0-9.]+)',
  1, 1, 'e', 1 -- Extract the first capture group
) AS extracted_price
FROM unstructured_logs;
```

---

### 10. Scrub Personal Data Safely
Mask sensitive information like emails for GDPR compliance while maintaining the structural format for analytics.



**Why it matters:** Manual masking is slow and error-prone. This automates PII scrubbing directly in the views exposed to analysts.

**Pros:** Ensures compliance while retaining domain information for aggregate analytics.  
**Cons:** Complex regex replacements can be slow on billion-row tables. Dedicated database masking policies or UDFs might be more performant at scale.

```sql
-- Converts rohan.dutt@company.com to r***@company.com
SELECT REGEXP_REPLACE(
  email_column,
  '^([a-zA-Z0-9])[^@]+(@.*)$',
  '\1***\2'
) AS masked_email
FROM user_profiles;
```
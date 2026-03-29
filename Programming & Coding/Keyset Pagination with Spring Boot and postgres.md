# javaPagination That Scales: Keyset Pagination with Spring Boot + Postgres

### Introduction
Offset pagination is easy:

```sql
SELECT * FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 50000;
```

…but it’s **not scalable**. Postgres has to walk past 50,000 rows before returning 20. As your table grows (audits, events, logs, orders), response times creep up and CPU burns.

**Keyset pagination** flips the model: instead of “go to page N”, the client asks for “the next 20 items after this last item I saw.” No skipping. No linear cost. Stable performance.

This article gives you a clean, copy-paste path to **seek pagination** in Spring Boot 3 + Postgres: schema, indexes, SQL, Spring Data code, cursors (next/prev), and gotchas.

### When to use keyset pagination
Use it for feeds, timelines, admin tables, logs, or anything you scroll **in chronological or natural order**. If your users must jump to **page 187**, offset is simpler; for everything else (99% of browsing), keyset is faster and safer.

### The idea in one picture (mental model)


1.  Choose a **stable, unique order** (e.g., `created_at DESC, id DESC`).
2.  Remember the **last row’s sort values** as a **cursor**.
3.  Next page: `WHERE (created_at, id) < (:lastCreatedAt, :lastId)` + `ORDER BY created_at DESC, id DESC LIMIT 20`.
4.  Postgres supports **tuple comparisons**: `(a,b) < (:x,:y)` follows your `ORDER BY (a DESC, b DESC)` semantics when you flip the operator accordingly.

### Table, sample data & indexes
```sql
-- orders table
CREATE TABLE orders (
  id           BIGSERIAL PRIMARY KEY,
  customer_id  BIGINT      NOT NULL,
  total_cents  BIGINT      NOT NULL,
  created_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Critical index for the sort order
CREATE INDEX idx_orders_created_id_desc
  ON orders (created_at DESC, id DESC);
```
**Tip:** Make sure your `ORDER BY` columns are covered by a matching index (same order & direction).

---

### SQL: offset vs keyset

**Offset (don’t do this at scale):**
```sql
SELECT id, customer_id, total_cents, created_at
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 100000;
```

**Keyset “next page” (fast):**
```sql
-- cursor (last row on the previous page)
-- :c_created_at TIMESTAMPTZ, :c_id BIGINT

SELECT id, customer_id, total_cents, created_at
FROM orders
WHERE (created_at, id) < (:c_created_at, :c_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```
**First page** has no cursor; you omit the `WHERE`.

**Previous page** (scroll up) reverses the direction:
```sql
SELECT id, customer_id, total_cents, created_at
FROM orders
WHERE (created_at, id) > (:c_created_at, :c_id)
ORDER BY created_at ASC, id ASC
LIMIT 20;    -- fetch “forward” then reverse in app
```
Return those results reversed in the service so the client still sees **DESC**.

---

### Spring Boot implementation

#### 1) DTOs for API
```java
// Page request/response with cursor strings
public record CursorPageRequest(int size, String after, String before) {}

public record OrderView(long id, long customerId, long totalCents, java.time.Instant createdAt) {}

public record CursorPageResponse<T>(
        java.util.List<T> data,
        String nextCursor,
        String prevCursor,
        boolean hasNext,
        boolean hasPrev
) {}
```

#### 2) A tiny cursor codec (Base64 JSON)
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.Base64;

public final class CursorCodec {
    private static final ObjectMapper MAPPER = new ObjectMapper();

    public static String encode(java.time.Instant createdAt, long id) {
        try {
            var json = MAPPER.writeValueAsBytes(java.util.Map.of(
                    "ts", createdAt.toString(),
                    "id", id
            ));
            return Base64.getUrlEncoder().withoutPadding().encodeToString(json);
        } catch (Exception e) { throw new IllegalStateException(e); }
    }

    public static java.util.AbstractMap.SimpleEntry<java.time.Instant, Long> decode(String cursor) {
        try {
            var bytes = Base64.getUrlDecoder().decode(cursor);
            var node = MAPPER.readTree(bytes);
            return new java.util.AbstractMap.SimpleEntry<>(
                    java.time.Instant.parse(node.get("ts").asText()),
                    node.get("id").asLong()
            );
        } catch (Exception e) { throw new IllegalArgumentException("Bad cursor"); }
    }

    private CursorCodec() {}
}
```

#### 3) Repository with keyset queries (Spring Data + native)
```java
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.Repository;
import org.springframework.data.repository.query.Param;
import java.time.Instant;
import java.util.List;

public interface OrderKeysetRepository extends Repository<Order, Long> {

    // first page (no cursor)
    @Query(value = """
        SELECT id, customer_id, total_cents, created_at
        FROM orders
        ORDER BY created_at DESC, id DESC
        LIMIT :limit
    """, nativeQuery = true)
    List<Object[]> first(@Param("limit") int limit);

    // next page (after)
    @Query(value = """
        SELECT id, customer_id, total_cents, created_at
        FROM orders
        WHERE (created_at, id) < (:createdAt, :id)
        ORDER BY created_at DESC, id DESC
        LIMIT :limit
    """, nativeQuery = true)
    List<Object[]> next(@Param("createdAt") Instant createdAt,
                        @Param("id") long id,
                        @Param("limit") int limit);

    // prev page (before) – note ASC order to find earlier rows, reverse later
    @Query(value = """
        SELECT id, customer_id, total_cents, created_at
        FROM orders
        WHERE (created_at, id) > (:createdAt, :id)
        ORDER BY created_at ASC, id ASC
        LIMIT :limit
    """, nativeQuery = true)
    List<Object[]> prevAsc(@Param("createdAt") Instant createdAt,
                           @Param("id") long id,
                           @Param("limit") int limit);
}
```
*(We return `Object[]` from native queries for brevity; map to DTO below.)*

#### 4) Service: mapping, cursors, prev/next
```java
import org.springframework.stereotype.Service;
import java.util.List;
import static java.util.stream.Collectors.toList;

@Service
public class OrderFeedService {

    private final OrderKeysetRepository repo;

    public OrderFeedService(OrderKeysetRepository repo) { this.repo = repo; }

    public CursorPageResponse<OrderView> list(CursorPageRequest req) {
        int size = req.size() > 0 && req.size() <= 100 ? req.size() : 20;

        List<OrderView> rows;
        boolean forward = req.after() != null && !req.after().isBlank();
        boolean backward = req.before() != null && !req.before().isBlank();

        if (forward) {
            var c = CursorCodec.decode(req.after());
            rows = toViews(repo.next(c.getKey(), c.getValue(), size + 1));
        } else if (backward) {
            var c = CursorCodec.decode(req.before());
            // fetch ASC then reverse to keep DESC in the response
            rows = toViews(repo.prevAsc(c.getKey(), c.getValue(), size + 1));
            java.util.Collections.reverse(rows);
        } else {
            rows = toViews(repo.first(size + 1));
        }

        boolean hasMore = rows.size() > size;
        if (hasMore) rows = rows.subList(0, size);

        String nextCursor = rows.isEmpty() ? null
                : CursorCodec.encode(rows.get(rows.size() - 1).createdAt(), rows.get(rows.size() - 1).id());

        String prevCursor = rows.isEmpty() ? null
                : CursorCodec.encode(rows.get(0).createdAt(), rows.get(0).id());

        return new CursorPageResponse<>(
                rows,
                hasMore ? nextCursor : null,
                prevCursor,                 // clients can use this to page “up”
                hasMore,
                (forward || backward)       // heuristic: we have a previous page if not on the very first
        );
    }

    private List<OrderView> toViews(List<Object[]> rs) {
        return rs.stream()
                .map(a -> new OrderView(
                        ((Number) a[0]).longValue(),
                        ((Number) a[1]).longValue(),
                        ((Number) a[2]).longValue(),
                        ((java.sql.Timestamp) a[3]).toInstant()))
                .collect(toList());
    }
}
```

#### 5) REST controller
```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/orders")
public class OrderFeedController {

    private final OrderFeedService service;

    public OrderFeedController(OrderFeedService service) { this.service = service; }

    @GetMapping
    public CursorPageResponse<OrderView> feed(
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String after,
            @RequestParam(required = false) String before) {

        return service.list(new CursorPageRequest(size, after, before));
    }
}
```

### Client usage
* **First page:** `/api/orders?size=20`
* **Next page:** `/api/orders?size=20&after=<nextCursor_from_previous_response>`
* **Previous:** `/api/orders?size=20&before=<prevCursor_from_previous_response>`

### Handling filters & searches
Keyset still works with filters — just include them both in the **index** and the **WHERE**:

```sql
-- Example: filter by customer_id
CREATE INDEX idx_orders_customer_created_id_desc
  ON orders (customer_id, created_at DESC, id DESC);

SELECT id, customer_id, total_cents, created_at
FROM orders
WHERE customer_id = :cust
  AND (created_at, id) < (:c_created_at, :c_id)
ORDER BY created_at DESC, id DESC
LIMIT :limit;
```
For text search, store **rank/sort keys** you can index (e.g., `created_at, id`) and keep the search term filter separate. Avoid sorting on a non-indexed expression.

### Tie-breakers and correctness
Always ensure a **unique sort**; otherwise pages can duplicate/skip rows.

* **Recommended order:** `created_at DESC, id DESC`
* If you sort by `created_at` only, two rows with the same timestamp can appear twice or be skipped between requests.

### Next vs Previous pages (UX notes)
* **Next:** easy — use `<` with `DESC`.
* **Previous:** query with `>` and `ASC LIMIT N`, then reverse in the app.
* Disable “Previous” on the very first page (no `before` cursor).

### Why Postgres loves this approach
* Uses an **index range scan** from the cursor onward — no scanning/throwing away earlier rows.
* Latency is **O(page_size)**, not O(offset).
* Works beautifully with **covering indexes** (only the columns you need).

### Common pitfalls (and fixes)
* **Unstable sorting** → add a **unique tie-breaker** (`id`).
* **Missing index** → add `(filter columns…, sort columns…)` in the right order & direction.
* **Changing filters mid-stream** → new filters invalidate the old cursor. Start fresh.
* **Clock skew across writers** → if `created_at` comes from app nodes, prefer DB timestamps (`DEFAULT now()`) or add a tie-breaker.
* **Cursor leakage** → cursors are just state; don’t put sensitive info inside. Use encoded values only.

### Conclusion
Offset pagination is fine for page 1 — but it breaks down as your dataset grows. **Keyset pagination** keeps queries fast by using a stable sort and a lightweight cursor. With the tuple comparison trick and a matching index, Postgres returns only what you need. In Spring Boot, a tiny Base64 cursor + three repository methods (first, next, prev) give you a user-friendly, production-ready feed that **stays fast forever**.
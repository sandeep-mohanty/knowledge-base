# Build Your Own API Rate Limiter in Spring Boot (Without Redis): A Complete Guide

### Introduction
Every backend developer eventually faces this problem:
> “Some client is hitting my API 1000 times per second, and my server is dying. How do I stop this?”

Most tutorials immediately push you toward Redis or API Gateways. But what if you want a **simple, fast, in-memory** rate limiter?
* No Redis
* No external dependencies
* No complicated infrastructure

Just pure **Spring Boot + Java**, a few smart algorithms, and clean code. This guide covers everything you need to build a production-grade rate limiter, including:
* Sliding Window algorithm
* Token Bucket algorithm
* Fixed Window (bonus)
* In-memory LRU cache
* Limiting per-user, per-IP, per-API-key
* Thread-safe data structures
* Real implementation you can directly copy

---

### Why Rate Limiting Matters
Imagine you’re running a food delivery app. One day, some script sends **10,000 requests/min** to your `/restaurants` API. Suddenly:
* CPU jumps to 100%
* Response times slow
* Your boss calls
* Everyone blames the backend developer (yes, you 😅)

Rate limiting prevents this. It allows you to define:
* Max 10 requests per second per user **OR**
* Max 100 requests per minute per IP **OR**
* Max 500 requests per day per API key

Basically: **Good clients stay fast. Bad clients get blocked. Your server stays healthy.**

---

### Basic Architecture (In-Memory Rate Limiter)
We’ll store limits in a `ConcurrentHashMap` like:
`Map<String, RateLimitInfo>`

Where **key** can be:
* User ID
* IP Address
* API Key
* Combination of all three (e.g., `"IP:103.44.2.1|API:/login"`)

For each key, we store timestamps, counters, or tokens — depending on the algorithm.

---

### 1. Sliding Window Rate Limiter (Most Accurate)
#### How it works
If limit = **5 requests per 10 seconds**, then for every request:
1. Look at timestamps from last 10 seconds.
2. Remove old timestamps.
3. Count remaining.
4. If > 5 → block.
5. Else → allow and add current timestamp.



#### Code: Sliding Window Rate Limiter
**RateLimiterService.java**
```java
@Service
public class SlidingWindowRateLimiter {
    private final int maxRequests = 5;
    private final long windowSizeInMillis = 10_000;
    private final ConcurrentHashMap<String, Deque<Long>> requestLog = new ConcurrentHashMap<>();

    public boolean isAllowed(String key) {
        long now = System.currentTimeMillis();
        requestLog.putIfAbsent(key, new LinkedList<>());
        Deque<Long> timestamps = requestLog.get(key);

        synchronized (timestamps) {
            while (!timestamps.isEmpty() && (now - timestamps.peekFirst() > windowSizeInMillis)) {
                timestamps.pollFirst();
            }
            if (timestamps.size() >= maxRequests) {
                return false;
            }
            timestamps.addLast(now);
        }
        return true;
    }
}
```

#### Filter: Enforcing Rate Limit
```java
@Component
public class SlidingWindowFilter extends OncePerRequestFilter {
    @Autowired
    private SlidingWindowRateLimiter limiter;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String key = request.getRemoteAddr() + ":" + request.getRequestURI();
        if (!limiter.isAllowed(key)) {
            response.setStatus(429);
            response.getWriter().write("Too many requests - Slow down!");
            return;
        }
        chain.doFilter(request, response);
    }
}
```

---

### 2. Token Bucket Algorithm (Used by Google, AWS, Netflix)
#### How it works
Think of a bucket:
* It has **N tokens**.
* Every request consumes **1 token**.
* Tokens refill at a fixed rate.
* If bucket is empty → block.



#### Code: Token Bucket Rate Limiter
```java
public class TokenBucket {
    private final int capacity;
    private final int refillRatePerSecond;
    private double tokens;
    private long lastRefillTimestamp;

    public TokenBucket(int capacity, int refillRatePerSecond) {
        this.capacity = capacity;
        this.refillRatePerSecond = refillRatePerSecond;
        this.tokens = capacity;
        this.lastRefillTimestamp = System.currentTimeMillis();
    }

    public synchronized boolean allowRequest() {
        refillTokens();
        if (tokens >= 1) {
            tokens -= 1;
            return true;
        }
        return false;
    }

    private void refillTokens() {
        long now = System.currentTimeMillis();
        double tokensToAdd = ((now - lastRefillTimestamp) / 1000.0) * refillRatePerSecond;
        tokens = Math.min(capacity, tokens + tokensToAdd);
        lastRefillTimestamp = now;
    }
}
```

#### Keyed Token Buckets (per IP/user)
```java
private final ConcurrentHashMap<String, TokenBucket> buckets = new ConcurrentHashMap<>();

public boolean isAllowed(String key) {
    buckets.putIfAbsent(key, new TokenBucket(10, 2));
    return buckets.get(key).allowRequest();
}
```

---

### 3. In-Memory LRU Cache for Cleanup
#### Why we need this?
With 1 million unique IPs, your hashmap will swell rapidly and start eating memory like crazy. Solution: use `LinkedHashMap` with eviction.

**LRUCache.java**
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int maxSize;

    public LRUCache(int maxSize) {
        super(16, 0.75f, true);
        this.maxSize = maxSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxSize;
    }
}
```
Now attach rate-limiters inside this cache so old keys auto-remove.

---

### 4. Rate Limiting Per User / IP / API Key / Role
Rate limiting becomes far more effective when you apply different limits based on **who** is calling your API and **how** they’re accessing it. Separate limits by:
* **IP address** → protects from bots and anonymous abuse.
* **User ID** → stops one logged-in user from overloading the system.
* **API key** → lets you offer different quotas (Free vs. Paid).
* **Role** → admins often need higher limits.

#### Get key from request
```java
String ip = request.getRemoteAddr();
String user = request.getHeader("X-USER-ID");
String apiKey = request.getHeader("X-API-KEY");
String api = request.getRequestURI();
String key = ip + "|" + user + "|" + apiKey + "|" + api;
```
This creates a unique signature like: `103.22.12.5|user123|apikey-ABCD|/login`

---

### Test It Live
Try calling any API 10+ times in Postman → You’ll get:  
`HTTP 429 Too Many Requests`

Congratulations!!! You built a real rate limiter.

---

### Security Best Practices
| Always Prefer | Avoid |
| :--- | :--- |
| ✔ Per-IP + Per-User combination | ✘ Fixed window (easy to burst) |
| ✔ Filters instead of interceptors | ✘ Storing timestamps for millions of users |
| ✔ Token Bucket for production load | ✘ Using `synchronized` everywhere (use Concurrent maps) |
| ✔ Sliding Window for accuracy | |
| ✔ LRU Cache for memory cleanup | |

---

### Real-World Use Cases
|Scenario |	Suggested Algorithm |
| :--- | :--- |
| Prevent brute force login	| Sliding Window (tight control) |
| Protect public API	| Token Bucket |
| APIs used by mobile apps | Token Bucket |
| Expensive AI/ML APIs | Sliding Window |
| Internal admin tools | Simple Fixed Window |

---

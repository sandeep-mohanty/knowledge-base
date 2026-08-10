# HTTP QUERY Method - The Complete Guide to Modern API Querying

**📚 Comprehensive Deep-Dive Tutorial**  
**⏱️ Reading Time:** 25-30 minutes  
**🎯 Difficulty Level:** Intermediate  
**📅 Last Updated:** January 2026

---

## Table of Contents

1. [Introduction & Overview](#introduction--overview)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [The Problem: Why GET and POST Fall Short](#the-problem-why-get-and-post-fall-short)
5. [Enter the HTTP QUERY Method](#enter-the-http-query-method)
6. [How QUERY Works: Technical Deep Dive](#how-query-works-technical-deep-dive)
7. [Implementation Guide](#implementation-guide)
8. [Real-World Use Cases](#real-world-use-cases)
9. [Best Practices](#best-practices)
10. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
11. [Performance Considerations](#performance-considerations)
12. [Security Considerations](#security-considerations)
13. [Testing Strategies](#testing-strategies)
14. [Migration Guide](#migration-guide)
15. [Common Pitfalls & Troubleshooting](#common-pitfalls--troubleshooting)
16. [Practice Exercises](#practice-exercises)
17. [Question Bank](#question-bank)
18. [Summary & Key Takeaways](#summary--key-takeaways)
19. [Further Reading & Resources](#further-reading--resources)

---

## Introduction & Overview

### The Query Problem That Plagues Every API Developer

If you've built APIs for any length of time, you've inevitably faced this scenario: You're building a search endpoint that starts simple—a few filters, maybe some pagination. Everything works great with GET parameters. Then the product team asks for "just a few more filters." Then "one more complex condition." Suddenly your pristine URL looks like this:

```
GET /products?category=laptops&brand=apple&minPrice=1000&maxPrice=2500&rating=4&inStock=true&sort=price&page=2&color=silver&warehouse=US&discount=true&shipping=express
```

And you're left with an unmaintainable mess that exceeds URL length limits and hurts readability.

> 💡 **The Developer's Dilemma:** For decades, we've had to choose between GET (semantically correct but limited) and POST (flexible but semantically wrong) for complex queries.

### What is the HTTP QUERY Method?

The HTTP QUERY method is a proposed standard that provides a semantically correct way to perform complex, read-only queries with structured data. It's designed specifically for operations that:
- Retrieve data without modifying server state
- Require complex filtering, sorting, or nested conditions
- Need a request body for structured query parameters
- Don't fit comfortably in URL query strings

### Why This Matters Now

Modern applications increasingly require sophisticated querying capabilities:
- **E-commerce platforms** with faceted search and dynamic filters
- **Analytics dashboards** with multi-dimensional data exploration
- **Business intelligence tools** with complex aggregation queries
- **Inventory systems** with nested conditional logic
- **Reporting platforms** with customizable data extraction

The QUERY method gives these operations a proper semantic home in the HTTP specification.

---

## Prerequisites

Before diving into this tutorial, you should have:

✅ **Required Knowledge:**
- Solid understanding of HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Experience building or consuming REST APIs
- Understanding of RESTful design principles
- Basic knowledge of query parameters vs request bodies
- Familiarity with API design best practices

✅ **Recommended Experience:**
- Built at least one production API
- Worked with API documentation (OpenAPI/Swagger)
- Understanding of HTTP status codes and caching
- Basic knowledge of authentication/authorization in APIs

✅ **Tools Needed:**
- Code editor (VS Code, IntelliJ, etc.)
- HTTP client (Postman, curl, or similar)
- Backend framework (Spring Boot, Express, FastAPI, etc.)
- Optional: API testing tools

---

## Learning Objectives

By the end of this tutorial, you will be able to:

🎯 **Understand the Problem Space:**
- Identify scenarios where GET falls short
- Recognize the semantic mismatch of using POST for queries
- Explain the historical context of HTTP method limitations

🎯 **Master the QUERY Method:**
- Explain what the QUERY method is and when to use it
- Implement QUERY endpoints in multiple frameworks
- Design APIs that follow QUERY method semantics

🎯 **Implementation Skills:**
- Build production-ready QUERY endpoints
- Handle validation, error cases, and edge cases
- Implement proper caching strategies
- Apply security best practices

🎯 **Decision Making:**
- Choose between GET, QUERY, and POST appropriately
- Evaluate when QUERY is the right choice
- Plan migrations from POST /search to QUERY
- Understand adoption considerations and limitations

🎯 **Advanced Topics:**
- Optimize QUERY performance
- Implement comprehensive testing strategies
- Handle framework and proxy compatibility
- Design for future scalability

---

## The Problem: Why GET and POST Fall Short

### The GET Method: Perfect Until It's Not

GET is the HTTP method designed for safe, idempotent, cacheable retrieval of resources. It's ideal for:

```http
GET /users/42
GET /products?category=laptops&page=1
GET /orders/123/items
```

**GET's Strengths:**
- ✅ Semantically correct for retrieval
- ✅ Cacheable by default
- ✅ Bookmarkable and shareable
- ✅ Safe and idempotent
- ✅ Works with browser history
- ✅ SEO-friendly

**GET's Limitations:**

```bash
# Problem 1: URL Length Limits
# Browsers: ~2,048 characters (IE), ~8,000+ (Chrome/Firefox)
# Servers: Varies (Apache: 8KB, Nginx: 4-8KB)
# Proxies: Often 4-8KB limits

GET /api/search?category=laptops&brand=apple,dell,hp&minPrice=1000&maxPrice=2500&rating=4&inStock=true&sort=price&page=2&color=silver,gray,black&warehouse=US,EU,ASIA&features=touchscreen,ssd,backlit-keyboard&condition=new&discount=10-50&shipping=express&sellerRating=4.5&returnPolicy=true&warranty=2year&connectivity=wifi-6,bluetooth-5&...
# ↑ This URL will break in many environments
```

```javascript
// Problem 2: Complex Data Structures
// How do you represent this in query parameters?
const complexFilter = {
  price: {
    min: 1000,
    max: 2500,
    currency: "USD",
    discount: {
      type: "percentage",
      value: 15
    }
  },
  categories: ["laptops", "gaming"],
  brand: {
    include: ["Apple", "Dell"],
    exclude: ["Refurbished"]
  },
  rating: {
    min: 4,
    count: { min: 50 }
  },
  availability: {
    inStock: true,
    warehouses: ["US", "EU"]
  },
  nestedCondition: {
    or: [
      { category: "laptops", brand: "Apple" },
      { category: "desktops", ram: "32GB" }
    ]
  }
};

// Query string representation is ugly and error-prone:
# ?price[min]=1000&price[max]=2500&price[currency]=USD&price[discount][type]=percentage...
```

```python
# Problem 3: Array and Nested Parameters
# Different frameworks handle this differently
# PHP: ?filter[]=value1&filter[]=value2
# Rails: ?filter[]=value1&filter[]=value2
# Express: ?filter=value1&filter=value2
# Spring: ?filter=value1&filter=value2

# Inconsistent parsing across platforms
```

### The POST Method: Flexible But Semantically Wrong

When GET doesn't work, most developers reach for POST:

```http
POST /api/products/search
Content-Type: application/json

{
  "categories": ["Laptop", "Gaming"],
  "brands": ["Apple", "Dell"],
  "price": {
    "min": 1000,
    "max": 2500
  },
  "rating": 4,
  "sort": "price"
}
```

**POST Works, But...**

```mermaid
graph TD
    A[POST /search Request] --> B{Semantic Analysis}
    B --> C{Is it creating a resource?}
    B --> D{Is it modifying state?}
    B --> E{Is it processing data?}
    
    C -->|No| F[❌ Semantic Mismatch]
    D -->|No| F
    E -->|No| F
    
    F --> G[Using POST for read-only operation]
    G --> H[Confuses API consumers]
    H --> I[Breaks HTTP semantics]
    I --> J[Problems with caching]
    J --> K[Documentation unclear]
    
    style F fill:#ff6b6b
    style G fill:#ffd93d
    style I fill:#ff6b6b
```

**Why POST for Queries Is Problematic:**

1. **Semantic Incorrectness:**
   - POST means "create" or "process"
   - Nothing is being created
   - Nothing is being modified
   - We're just asking a question

2. **Caching Issues:**
   - POST responses aren't cached by default
   - Requires explicit Cache-Control headers
   - More complex cache invalidation
   - CDNs may not cache POST requests

3. **Documentation Confusion:**
   - API consumers expect POST to change state
   - Unclear if operation is safe to retry
   - Misleading for API discovery

4. **Tooling Limitations:**
   - Browser DevTools show POST differently
   - Some HTTP clients treat POST specially
   - API gateways may have different POST handling

### The Real-World Impact

Let's look at actual scenarios developers face:

```java
// Scenario 1: E-commerce Search
// Before: Clean GET with limited filters
GET /products?category=laptops&brand=apple

// After 6 months: Unmaintainable GET
GET /products?category=laptops&brand=apple,dell,hp&minPrice=1000&maxPrice=2500&rating=4&inStock=true&sort=price&page=2&color=silver&warehouse=US&discount=true&shipping=express&features=touchscreen,ssd&condition=new&warranty=2year

// Developer decision: Switch to POST
POST /products/search
{
  // Complex nested structure
}
// But now caching is broken and semantics are wrong
```

```javascript
// Scenario 2: Analytics Dashboard
// User wants to filter by multiple dimensions
const analyticsQuery = {
  dateRange: {
    start: "2025-01-01",
    end: "2025-01-31"
  },
  metrics: ["revenue", "users", "conversion"],
  dimensions: ["country", "device", "source"],
  filters: {
    country: ["US", "UK", "DE"],
    device: ["mobile", "desktop"],
    source: ["organic", "paid"]
  },
  aggregations: {
    revenue: "sum",
    users: "count"
  }
};

// GET is impossible (too complex)
// POST works but semantically wrong
// QUERY would be perfect
```

### The Cost of the Wrong Choice

```mermaid
graph LR
    A[API Design Decision] --> B{Choose Method}
    
    B -->|GET| C[URL Length Issues]
    B -->|POST| D[Semantic Issues]
    B -->|QUERY| E[✅ Correct Choice]
    
    C --> F[Browser/Proxy Limits]
    C --> G[Encoding Problems]
    C --> H[Readability Issues]
    
    D --> I[Cache Misses]
    D --> J[Confusing Documentation]
    D --> K[Tooling Issues]
    
    E --> L[Clear Semantics]
    E --> M[Proper Caching]
    E --> N[Maintainable Code]
    
    style E fill:#4ecdc4
    style C fill:#ff6b6b
    style D fill:#ffd93d
```

**The Bottom Line:**
We've been forcing square pegs into round holes for decades. The QUERY method finally gives us a round hole for square pegs.

---

## Enter the HTTP QUERY Method

### What is QUERY?

The HTTP QUERY method is a proposed HTTP method (currently in IETF draft status) designed specifically for safe, idempotent, cacheable query operations that require a request body.

**Official Definition:**
> "The QUERY method is used to initiate a query operation that is intended to retrieve information without modifying server state. It is similar to GET but allows for a request body containing structured query parameters."

### Key Characteristics

```mermaid
graph TD
    A[HTTP QUERY Method] --> B[Safe]
    A --> C[Idempotent]
    A --> D[Cacheable]
    A --> E[Read-Only]
    
    B --> B1[Does not modify server state]
    C --> C1[Multiple identical requests<br/>have same effect]
    D --> D1[Responses can be cached]
    E --> E1[Only retrieves information]
    
    A --> F[Allows Request Body]
    F --> F1[JSON/XML structured data]
    F --> F2[Complex nested structures]
    F --> F3[Large payloads]
    
    A --> G[Semantically Clear]
    G --> G1[Explicitly means "query"]
    G --> G2[No semantic mismatch]
    G --> G3[Self-documenting]
    
    style A fill:#4ecdc4
    style B fill:#95e1d3
    style C fill:#95e1d3
    style D fill:#95e1d3
    style E fill:#95e1d3
```

### QUERY vs GET vs POST: The Comparison

| Aspect | GET | POST | QUERY |
|--------|-----|------|-------|
| **Semantics** | Retrieve resource | Create/Process resource | Query resources |
| **Request Body** | ❌ Not allowed | ✅ Allowed | ✅ Allowed |
| **Safe** | ✅ Yes | ❌ No | ✅ Yes |
| **Idempotent** | ✅ Yes | ❌ Not guaranteed | ✅ Yes |
| **Cacheable** | ✅ Yes (by default) | ❌ No (by default) | ✅ Yes (with headers) |
| **URL Length** | ⚠️ Limited (~2-8KB) | ✅ Unlimited | ✅ Unlimited |
| **Complex Data** | ⚠️ Difficult | ✅ Easy | ✅ Easy |
| **Bookmarkable** | ✅ Yes | ❌ No | ⚠️ Possible |
| **Browser Support** | ✅ Universal | ✅ Universal | ⚠️ Emerging |
| **Use Case** | Simple retrieval | Create/Update/Process | Complex queries |

### When to Use QUERY: The Decision Matrix

```mermaid
flowchart TD
    A[Need to retrieve data?] -->|No| B[Use POST/PUT/PATCH/DELETE]
    A -->|Yes| C{Simple parameters?}
    
    C -->|Yes| D{Parameters fit in URL?}
    D -->|Yes| E[✅ Use GET]
    D -->|No| F{Can simplify query?}
    F -->|Yes| E
    F -->|No| G[✅ Use QUERY]
    
    C -->|No| H{Complex nested structures?}
    H -->|Yes| G
    H -->|No| I{Need request body?}
    I -->|Yes| G
    I -->|No| E
    
    G --> J[Benefits:<br/>- Clear semantics<br/>- Request body<br/>- Cacheable<br/>- Maintainable]
    
    style G fill:#4ecdc4
    style E fill:#95e1d3
    style B fill:#ff6b6b
```

**Decision Rules:**

1. **Use GET when:**
   - Simple key-value parameters
   - URL length < 2,000 characters
   - No complex nested structures
   - Need browser bookmarking/sharing

2. **Use QUERY when:**
   - Complex nested query structures
   - Large payloads (>2KB)
   - Array/object parameters
   - Need request body for clarity
   - Read-only operation with complexity

3. **Use POST when:**
   - Creating new resources
   - Modifying server state
   - Processing operations
   - Non-idempotent operations

### A Practical Example

**The E-commerce Search Problem:**

```http
# ❌ Option 1: GET with query string (problematic)
GET /api/products/search?categories=laptops,gaming&brands=apple,dell&price[min]=1000&price[max]=2500&rating[min]=4&rating[count][min]=50&inStock=true&sort=price&order=asc&page=2&limit=20&color=silver,gray&warehouses=US,EU&features[touchscreen]=true&features[ssd]=true&condition=new&discount[min]=10&discount[max]=50&shipping=express
# URL Length: ~450 characters (still manageable but ugly)
```

```http
# ❌ Option 2: POST (semantically wrong)
POST /api/products/search
{
  "categories": ["laptops", "gaming"],
  "brands": ["apple", "dell"],
  "price": {"min": 1000, "max": 2500},
  "rating": {"min": 4, "count": {"min": 50}},
  "inStock": true,
  "sort": {"field": "price", "order": "asc"},
  "pagination": {"page": 2, "limit": 20},
  "color": ["silver", "gray"],
  "warehouses": ["US", "EU"],
  "features": {"touchscreen": true, "ssd": true},
  "condition": "new",
  "discount": {"min": 10, "max": 50},
  "shipping": "express"
}
# Works, but semantically incorrect
```

```http
# ✅ Option 3: QUERY (correct approach)
QUERY /api/products/search
Content-Type: application/json

{
  "categories": ["laptops", "gaming"],
  "brands": ["apple", "dell"],
  "price": {"min": 1000, "max": 2500},
  "rating": {"min": 4, "count": {"min": 50}},
  "inStock": true,
  "sort": {"field": "price", "order": "asc"},
  "pagination": {"page": 2, "limit": 20},
  "color": ["silver", "gray"],
  "warehouses": ["US", "EU"],
  "features": {"touchscreen": true, "ssd": true},
  "condition": "new",
  "discount": {"min": 10, "max": 50},
  "shipping": "express"
}
# Semantically correct, clean, maintainable
```

---

## How QUERY Works: Technical Deep Dive

### HTTP Method Semantics

The QUERY method follows HTTP semantics defined in RFC 7231:

**Safe:** The method does not modify server state
**Idempotent:** Multiple identical requests produce the same result
**Cacheable:** Responses can be cached (with proper headers)

### Request Structure

```http
QUERY /api/resource HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Cache-Control: max-age=300
Authorization: Bearer <token>

{
  "query": "complex structured data here"
}
```

### Response Structure

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=300, public
ETag: "abc123xyz"

{
  "results": [...],
  "total": 150,
  "page": 2,
  "limit": 20
}
```

### Caching Behavior

```mermaid
sequenceDiagram
    participant C as Client
    participant P as Proxy/CDN
    participant S as Server
    
    C->>P: QUERY /api/products (with Cache-Control)
    P->>P: Check cache
    
    alt Cache Hit
        P-->>C: 200 OK (cached response)
    else Cache Miss
        P->>S: QUERY /api/products
        S->>S: Execute query
        S-->>P: 200 OK + Cache-Control headers
        P->>P: Store in cache
        P-->>C: 200 OK (fresh response)
    end
    
    Note over C,S: Subsequent requests served from cache
    Note over C,S: Until TTL expires or cache invalidated
```

### Idempotency Guarantees

```mermaid
graph TD
    A[QUERY Request] --> B[Server Receives]
    B --> C[Execute Query]
    C --> D[Return Results]
    
    E[Same QUERY Request Again] --> F[Server Receives]
    F --> G[Execute Query]
    G --> H[Return Results]
    
    D --> I[Result Set A]
    H --> J[Result Set A]
    
    I --> K[✅ Idempotent]
    J --> K
    
    style K fill:#4ecdc4
```

**Key Point:** As long as underlying data hasn't changed, identical QUERY requests return identical results, making them safe to retry and cache.

### Status Codes

| Status Code | Meaning | When to Use |
|-------------|---------|-------------|
| `200 OK` | Success | Query executed successfully |
| `400 Bad Request` | Invalid query | Malformed JSON, validation errors |
| `401 Unauthorized` | Auth required | Missing or invalid authentication |
| `403 Forbidden` | Permission denied | Insufficient permissions |
| `422 Unprocessable Entity` | Semantic error | Valid JSON but invalid query logic |
| `500 Internal Server Error` | Server error | Database errors, unexpected exceptions |

---

## Implementation Guide

### Java/Spring Boot Implementation

#### Basic Setup

```java
// ProductSearchQuery.java - Query DTO
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ProductSearchQuery {
    
    @Size(min = 1, max = 10, message = "1-10 categories allowed")
    private List<String> categories;
    
    @Size(min = 1, max = 5, message = "1-5 brands allowed")
    private List<String> brands;
    
    @Valid
    private PriceRange price;
    
    @Min(1) @Max(5)
    private Integer rating;
    
    @Min(1) @Max(100)
    private Integer page = 1;
    
    @Min(1) @Max(100)
    private Integer limit = 20;
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class PriceRange {
        @DecimalMin(value = "0.0")
        private BigDecimal min;
        
        @DecimalMax(value = "999999.99")
        private BigDecimal max;
        
        @Pattern(regexp = "USD|EUR|GBP", message = "Invalid currency")
        private String currency = "USD";
    }
}
```

```java
// ProductController.java
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {
    
    private final ProductService productService;
    
    // ✅ QUERY method implementation
    @QueryMapping
    @PostMapping("/search")
    // Note: Spring doesn't natively support QUERY yet
    // Use @PostMapping with custom configuration
    @Operation(
        summary = "Search products with complex filters",
        description = "Performs a complex product search using structured query parameters"
    )
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Search successful"),
        @ApiResponse(responseCode = "400", description = "Invalid query parameters"),
        @ApiResponse(responseCode = "401", description = "Unauthorized")
    })
    @Cacheable(value = "productSearch", key = "#query.hashCode()")
    public ResponseEntity<Page<ProductDTO>> searchProducts(
            @Valid @RequestBody ProductSearchQuery query,
            Pageable pageable) {
        
        // Log query for debugging
        log.info("Executing product search: {}", query);
        
        // Execute search
        Page<ProductDTO> results = productService.search(query, pageable);
        
        // Add cache headers
        return ResponseEntity.ok()
            .header("Cache-Control", "max-age=300, public")
            .header("ETag", generateETag(results))
            .body(results);
    }
    
    // Alternative: Using GET for simple queries
    @GetMapping
    public ResponseEntity<Page<ProductDTO>> getProducts(
            @RequestParam(required = false) String category,
            @RequestParam(required = false) String brand,
            Pageable pageable) {
        
        // Simple GET for basic filtering
        Page<ProductDTO> results = productService.findByFilters(category, brand, pageable);
        return ResponseEntity.ok(results);
    }
}
```

```java
// ProductService.java
@Service
@Slf4j
public class ProductService {
    
    private final ProductRepository productRepository;
    private final SpecificationExecutor<Product> specExecutor;
    
    public Page<ProductDTO> search(ProductSearchQuery query, Pageable pageable) {
        // Build dynamic specification
        Specification<Product> spec = buildSearchSpecification(query);
        
        // Execute query
        Page<Product> products = productRepository.findAll(spec, pageable);
        
        // Map to DTO
        return products.map(this::toDTO);
    }
    
    private Specification<Product> buildSearchSpecification(ProductSearchQuery query) {
        return (root, criteriaQuery, cb) -> {
            List<Predicate> predicates = new ArrayList<>();
            
            // Categories filter
            if (query.getCategories() != null && !query.getCategories().isEmpty()) {
                predicates.add(root.get("category").in(query.getCategories()));
            }
            
            // Brands filter
            if (query.getBrands() != null && !query.getBrands().isEmpty()) {
                predicates.add(root.get("brand").in(query.getBrands()));
            }
            
            // Price range filter
            if (query.getPrice() != null) {
                if (query.getPrice().getMin() != null) {
                    predicates.add(cb.greaterThanOrEqualTo(
                        root.get("price"), query.getPrice().getMin()));
                }
                if (query.getPrice().getMax() != null) {
                    predicates.add(cb.lessThanOrEqualTo(
                        root.get("price"), query.getPrice().getMax()));
                }
            }
            
            // Rating filter with minimum review count
            if (query.getRating() != null) {
                predicates.add(cb.greaterThanOrEqualTo(
                    root.get("averageRating"), query.getRating()));
            }
            
            // In-stock filter
            if (query.getInStock() != null && query.getInStock()) {
                predicates.add(cb.greaterThan(root.get("stockQuantity"), 0));
            }
            
            // Combine all predicates with AND
            return cb.and(predicates.toArray(new Predicate[0]));
        };
    }
    
    private String generateETag(Page<ProductDTO> page) {
        // Generate ETag based on content
        return "\"" + page.getContent().hashCode() + "\"";
    }
}
```

```java
// WebMvcConfig.java - Enable QUERY method support
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.favorPathExtension(false)
            .favorParameter(false)
            .ignoreAcceptHeader(false)
            .defaultContentType(MediaType.APPLICATION_JSON);
    }
    
    // Custom argument resolver for QUERY method
    @Bean
    public WebMvcConfigurer queryMethodConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
                resolvers.add(new QueryMethodArgumentResolver());
            }
        };
    }
}
```

#### Advanced: Dynamic Query Builder

```java
// DynamicQueryBuilder.java
@Component
public class DynamicQueryBuilder {
    
    public Specification<Product> build(ProductSearchQuery query) {
        List<Specification<Product>> specs = new ArrayList<>();
        
        // Category filter
        if (CollectionUtils.isNotEmpty(query.getCategories())) {
            specs.add((root, cq, cb) -> 
                root.get("category").in(query.getCategories()));
        }
        
        // Brand filter with exclusion
        if (CollectionUtils.isNotEmpty(query.getBrands())) {
            specs.add((root, cq, cb) -> 
                root.get("brand").in(query.getBrands()));
        }
        
        // Price range
        if (query.getPrice() != null) {
            specs.add(buildPriceSpec(query.getPrice()));
        }
        
        // Nested OR conditions
        if (query.getNestedCondition() != null) {
            specs.add(buildNestedOrCondition(query.getNestedCondition()));
        }
        
        // Combine with AND
        return specs.stream()
            .reduce(Specification::and)
            .orElse(null);
    }
    
    private Specification<Product> buildNestedOrCondition(
            ProductSearchQuery.NestedCondition condition) {
        return (root, cq, cb) -> {
            List<Predicate> orPredicates = new ArrayList<>();
            
            condition.getOr().forEach(orCondition -> {
                List<Predicate> andPredicates = new ArrayList<>();
                
                if (orCondition.getCategory() != null) {
                    andPredicates.add(cb.equal(
                        root.get("category"), orCondition.getCategory()));
                }
                if (orCondition.getBrand() != null) {
                    andPredicates.add(cb.equal(
                        root.get("brand"), orCondition.getBrand()));
                }
                if (orCondition.getMaxPrice() != null) {
                    andPredicates.add(cb.lessThanOrEqualTo(
                        root.get("price"), orCondition.getMaxPrice()));
                }
                
                orPredicates.add(cb.and(andPredicates.toArray(new Predicate[0])));
            });
            
            return cb.or(orPredicates.toArray(new Predicate[0]));
        };
    }
}
```

### Node.js/Express Implementation

```javascript
// productController.js
const express = require('express');
const { body, validationResult } = require('express-validator');
const router = express.Router();

// ✅ QUERY method middleware
const handleQuery = (req, res, next) => {
    // Set CORS headers
    res.header('Access-Control-Allow-Methods', 'GET, QUERY, POST, OPTIONS');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    next();
};

// ✅ QUERY endpoint implementation
router.post('/search', handleQuery, [
    body('categories').optional().isArray().withMessage('Categories must be an array'),
    body('brands').optional().isArray().withMessage('Brands must be an array'),
    body('price.min').optional().isFloat({ min: 0 }),
    body('price.max').optional().isFloat({ min: 0 }),
    body('rating').optional().isInt({ min: 1, max: 5 }),
    body('page').optional().isInt({ min: 1 }),
    body('limit').optional().isInt({ min: 1, max: 100 })
], async (req, res) => {
    try {
        // Validate request
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({
                error: 'Validation failed',
                details: errors.array()
            });
        }
        
        const query = req.body;
        
        // Log query
        console.log('Executing product search:', JSON.stringify(query));
        
        // Check cache
        const cacheKey = `product:search:${JSON.stringify(query)}`;
        const cached = await redis.get(cacheKey);
        
        if (cached) {
            return res.json({
                data: JSON.parse(cached),
                cached: true
            });
        }
        
        // Build MongoDB query
        const mongoQuery = buildMongoQuery(query);
        
        // Execute query
        const products = await Product.find(mongoQuery)
            .sort(query.sort?.field || 'price', query.sort?.order || 'asc')
            .skip((query.page - 1) * query.limit)
            .limit(query.limit);
        
        const total = await Product.countDocuments(mongoQuery);
        
        const result = {
            data: products,
            pagination: {
                page: query.page || 1,
                limit: query.limit || 20,
                total,
                pages: Math.ceil(total / (query.limit || 20))
            }
        };
        
        // Cache result
        await redis.setex(cacheKey, 300, JSON.stringify(result));
        
        // Send response with cache headers
        res.set('Cache-Control', 'max-age=300, public');
        res.set('ETag', `"${Buffer.from(JSON.stringify(result)).toString('base64')}"`);
        res.json(result);
        
    } catch (error) {
        console.error('Search error:', error);
        res.status(500).json({
            error: 'Internal server error',
            message: error.message
        });
    }
});

// Helper: Build MongoDB query from QUERY request
function buildMongoQuery(query) {
    const mongoQuery = {};
    
    // Categories
    if (query.categories?.length > 0) {
        mongoQuery.category = { $in: query.categories };
    }
    
    // Brands
    if (query.brands?.length > 0) {
        mongoQuery.brand = { $in: query.brands };
    }
    
    // Price range
    if (query.price) {
        mongoQuery.price = {};
        if (query.price.min) mongoQuery.price.$gte = query.price.min;
        if (query.price.max) mongoQuery.price.$lte = query.price.max;
    }
    
    // Rating
    if (query.rating) {
        mongoQuery.averageRating = { $gte: query.rating };
    }
    
    // In stock
    if (query.inStock) {
        mongoQuery.stockQuantity = { $gt: 0 };
    }
    
    return mongoQuery;
}

module.exports = router;
```

```javascript
// app.js - Express configuration
const express = require('express');
const productRoutes = require('./productController');
const cors = require('cors');

const app = express();

// Middleware
app.use(cors());
app.use(express.json({ limit: '10mb' })); // Allow larger request bodies

// Routes
app.use('/api/products', productRoutes);

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        error: 'Something went wrong!',
        message: err.message
    });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Python/FastAPI Implementation

```python
# models.py - Pydantic models
from pydantic import BaseModel, Field, validator
from typing import List, Optional
from decimal import Decimal

class PriceRange(BaseModel):
    min: Optional[Decimal] = Field(None, ge=0, description="Minimum price")
    max: Optional[Decimal] = Field(None, ge=0, description="Maximum price")
    currency: str = Field("USD", regex="^(USD|EUR|GBP)$")
    
    @validator('max')
    def validate_range(cls, v, values):
        if v and 'min' in values and values['min'] and v < values['min']:
            raise ValueError('max must be greater than min')
        return v

class SortConfig(BaseModel):
    field: str = Field("price", regex="^(price|rating|name|date)$")
    order: str = Field("asc", regex="^(asc|desc)$")

class PaginationConfig(BaseModel):
    page: int = Field(1, ge=1)
    limit: int = Field(20, ge=1, le=100)

class ProductSearchQuery(BaseModel):
    categories: Optional[List[str]] = Field(None, min_items=1, max_items=10)
    brands: Optional[List[str]] = Field(None, min_items=1, max_items=5)
    price: Optional[PriceRange]
    rating: Optional[int] = Field(None, ge=1, le=5)
    in_stock: Optional[bool] = False
    sort: Optional[SortConfig] = SortConfig()
    pagination: Optional[PaginationConfig] = PaginationConfig()
    
    class Config:
        schema_extra = {
            "example": {
                "categories": ["laptops", "gaming"],
                "brands": ["apple", "dell"],
                "price": {
                    "min": 1000,
                    "max": 2500,
                    "currency": "USD"
                },
                "rating": 4,
                "in_stock": True,
                "sort": {
                    "field": "price",
                    "order": "asc"
                },
                "pagination": {
                    "page": 1,
                    "limit": 20
                }
            }
        }
```

```python
# main.py - FastAPI application
from fastapi import FastAPI, Query, HTTPException, Response
from fastapi.middleware.cors import CORSMiddleware
from typing import Optional
import json

app = FastAPI(title="Product Search API", version="1.0.0")

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ✅ QUERY method endpoint
@app.post("/api/products/search", response_model=ProductSearchResponse)
async def search_products(
    query: ProductSearchQuery,
    response: Response
):
    """
    Search products using complex query criteria.
    
    This endpoint accepts structured JSON query parameters
    for advanced product search functionality.
    """
    try:
        # Log query
        print(f"Executing search: {query.json()}")
        
        # Check cache
        cache_key = f"product:search:{hash(str(query.dict()))}"
        cached_result = await get_from_cache(cache_key)
        
        if cached_result:
            response.headers["X-Cache"] = "HIT"
            return json.loads(cached_result)
        
        # Build database query
        db_query = build_mongo_query(query)
        
        # Execute query
        products = await Product.find(db_query).skip(
            (query.pagination.page - 1) * query.pagination.limit
        ).limit(query.pagination.limit).to_list()
        
        total = await Product.count_documents(db_query)
        
        result = {
            "data": products,
            "pagination": {
                "page": query.pagination.page,
                "limit": query.pagination.limit,
                "total": total,
                "pages": (total + query.pagination.limit - 1) // query.pagination.limit
            }
        }
        
        # Cache result
        await set_in_cache(cache_key, json.dumps(result), ttl=300)
        
        # Set cache headers
        response.headers["Cache-Control"] = "max-age=300, public"
        response.headers["X-Cache"] = "MISS"
        
        return result
        
    except Exception as e:
        print(f"Search error: {str(e)}")
        raise HTTPException(
            status_code=500,
            detail=f"Search failed: {str(e)}"
        )

# Helper function to build MongoDB query
def build_mongo_query(query: ProductSearchQuery) -> dict:
    mongo_query = {}
    
    if query.categories:
        mongo_query["category"] = {"$in": query.categories}
    
    if query.brands:
        mongo_query["brand"] = {"$in": query.brands}
    
    if query.price:
        price_filter = {}
        if query.price.min:
            price_filter["$gte"] = float(query.price.min)
        if query.price.max:
            price_filter["$lte"] = float(query.price.max)
        if price_filter:
            mongo_query["price"] = price_filter
    
    if query.rating:
        mongo_query["averageRating"] = {"$gte": query.rating}
    
    if query.in_stock:
        mongo_query["stockQuantity"] = {"$gt": 0}
    
    return mongo_query
```

---

## Real-World Use Cases

### 1. E-commerce Product Search

**Scenario:** Multi-filter product search with faceted navigation

```http
QUERY /api/v1/products/search
Content-Type: application/json

{
  "query": {
    "categories": ["Electronics", "Laptops"],
    "brands": ["Apple", "Dell", "HP"],
    "price": {
      "min": 500,
      "max": 3000,
      "currency": "USD"
    },
    "specifications": {
      "ram": ["8GB", "16GB", "32GB"],
      "storage": ["256GB SSD", "512GB SSD", "1TB SSD"],
      "processor": ["Intel i5", "Intel i7", "M1", "M2"]
    },
    "rating": {
      "min": 4,
      "reviewCount": { "min": 50 }
    },
    "availability": {
      "inStock": true,
      "warehouses": ["US", "EU"]
    },
    "features": {
      "touchscreen": true,
      "backlitKeyboard": true
    },
    "sort": {
      "field": "price",
      "order": "asc"
    },
    "pagination": {
      "page": 1,
      "limit": 24
    },
    "facets": ["brand", "price", "rating", "specifications"]
  }
}
```

**Response:**
```json
{
  "results": [...],
  "facets": {
    "brand": {
      "Apple": 45,
      "Dell": 78,
      "HP": 62
    },
    "price": {
      "under_1000": 23,
      "1000_2000": 89,
      "2000_3000": 73
    }
  },
  "pagination": {
    "page": 1,
    "limit": 24,
    "total": 185,
    "pages": 8
  }
}
```

**Benefits:**
- Clean, maintainable query structure
- Easy to add new filters
- Supports complex nested conditions
- Cacheable for performance

### 2. Analytics Dashboard

**Scenario:** Multi-dimensional data analysis with aggregations

```http
QUERY /api/v1/analytics/query
Content-Type: application/json

{
  "query": {
    "timeRange": {
      "start": "2025-01-01T00:00:00Z",
      "end": "2025-01-31T23:59:59Z",
      "granularity": "day"
    },
    "metrics": [
      {
        "name": "revenue",
        "aggregation": "sum",
        "field": "amount"
      },
      {
        "name": "transactions",
        "aggregation": "count"
      },
      {
        "name": "avgOrderValue",
        "aggregation": "avg",
        "field": "amount"
      }
    ],
    "dimensions": ["country", "device", "source"],
    "filters": {
      "country": ["US", "UK", "DE", "FR"],
      "device": ["mobile", "desktop", "tablet"],
      "source": ["organic", "paid_search", "social"]
    },
    "having": {
      "revenue": { "gt": 1000 }
    },
    "orderBy": {
      "field": "date",
      "direction": "asc"
    }
  }
}
```

**Response:**
```json
{
  "data": [
    {
      "date": "2025-01-01",
      "revenue": 45230.50,
      "transactions": 234,
      "avgOrderValue": 193.16
    }
  ],
  "metadata": {
    "queryTime": "45ms",
    "totalRows": 31
  }
}
```

### 3. Business Intelligence Reporting

**Scenario:** Complex reporting with multiple conditions

```http
QUERY /api/v1/reports/sales
Content-Type: application/json

{
  "query": {
    "filters": {
      "dateRange": {
        "from": "2025-01-01",
        "to": "2025-03-31"
      },
      "regions": ["North America", "Europe"],
      "productCategories": ["Electronics", "Software"],
      "salesReps": ["john.doe", "jane.smith"],
      "dealSize": {
        "min": 10000,
        "max": 500000
      }
    },
    "groupBy": ["region", "productCategory", "month"],
    "aggregations": {
      "revenue": "sum",
      "deals": "count",
      "avgDealSize": "avg"
    },
    "having": {
      "revenue": { "gte": 50000 }
    },
    "orderBy": {
      "revenue": "desc"
    },
    "limit": 100
  }
}
```

### 4. Inventory Management

**Scenario:** Complex inventory search with nested conditions

```http
QUERY /api/v1/inventory/search
Content-Type: application/json

{
  "query": {
    "conditions": [
      {
        "or": [
          {
            "category": "Electronics",
            "stockLevel": { "lt": 10 },
            "turnoverRate": { "gt": 5 }
          },
          {
            "category": "Perishables",
            "expiryDate": { "lte": "2025-02-01" }
          }
        ]
      },
      {
        "and": [
          { "warehouse": "US-West" },
          { "lastRestocked": { "gte": "2024-12-01" } }
        ]
      }
    ],
    "fields": ["sku", "name", "quantity", "location"],
    "sort": { "priority": "desc" }
  }
}
```

---

## Best Practices

### 1. API Design Best Practices

✅ **DO:**
- Use clear, descriptive endpoint names
- Document query structure thoroughly
- Provide sensible defaults for optional parameters
- Implement comprehensive input validation
- Use consistent naming conventions
- Version your API endpoints
- Return appropriate HTTP status codes
- Include request IDs for tracing
- Log queries for debugging (without sensitive data)

❌ **DON'T:**
- Don't use QUERY for state-changing operations
- Don't allow unbounded result sets
- Don't skip input validation
- Don't expose internal database structure
- Don't forget error handling
- Don't ignore caching opportunities

### 2. Request Design

```javascript
// ✅ Good: Clear, structured query
{
  "query": {
    "filters": {
      "category": "laptops",
      "price": { "min": 1000, "max": 2500 }
    },
    "pagination": { "page": 1, "limit": 20 },
    "sort": { "field": "price", "order": "asc" }
  }
}

// ❌ Bad: Unclear structure
{
  "cat": "laptops",
  "pmin": 1000,
  "pmax": 2500,
  "pg": 1,
  "sz": 20
}
```

### 3. Response Design

```json
// ✅ Good: Consistent response structure
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "pages": 8
  },
  "metadata": {
    "queryTime": "45ms",
    "cached": false
  }
}

// ❌ Bad: Inconsistent structure
{
  "results": [...],
  "count": 150
}
```

### 4. Caching Strategy

```java
// ✅ Good: Proper cache headers
@GetMapping
public ResponseEntity<Page<Product>> search(
        @Valid @RequestBody QueryRequest query) {
    
    Page<Product> results = service.search(query);
    
    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(5, TimeUnit.MINUTES)
            .cachePublic()
            .mustRevalidate())
        .eTag(generateETag(results))
        .body(results);
}

// Cache-Control: max-age=300, public, must-revalidate
// ETag: "abc123"
```

### 5. Error Handling

```python
# ✅ Good: Detailed error responses
from fastapi import HTTPException
from pydantic import ValidationError

@app.post("/api/query")
async def query_endpoint(query: QueryRequest):
    try:
        result = await execute_query(query)
        return {"success": True, "data": result}
    
    except ValidationError as e:
        raise HTTPException(
            status_code=422,
            detail={
                "error": "Validation failed",
                "fields": e.errors()
            }
        )
    
    except QueryExecutionError as e:
        raise HTTPException(
            status_code=400,
            detail={
                "error": "Invalid query",
                "message": str(e)
            }
        )
    
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail={
                "error": "Internal server error",
                "requestId": generate_request_id()
            }
        )
```

### 6. Documentation

```yaml
# OpenAPI/Swagger documentation
/api/products/search:
  post:
    summary: Search products with complex filters
    description: |
      Performs a complex product search using structured query parameters.
      
      ## When to Use
      - Complex filtering with multiple conditions
      - Nested query structures
      - Large payloads that don't fit in URL
      
      ## When to Use GET Instead
      - Simple key-value parameters
      - URL length < 2000 characters
      - Need browser bookmarking
    requestBody:
      required: true
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProductSearchQuery'
          example:
            categories: ["laptops", "gaming"]
            brands: ["apple", "dell"]
            price:
              min: 1000
              max: 2500
    responses:
      '200':
        description: Search successful
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/SearchResponse'
      '400':
        description: Invalid query
      '401':
        description: Unauthorized
```

---

## Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Using QUERY for State Changes

```javascript
// ❌ WRONG: Using QUERY to update data
QUERY /api/users/update
{
  "userId": 123,
  "email": "newemail@example.com"
}

// ✅ CORRECT: Use PUT or PATCH
PATCH /api/users/123
{
  "email": "newemail@example.com"
}
```

**Why it's wrong:**
- Violates HTTP semantics
- Breaks idempotency guarantees
- Confuses API consumers
- May bypass middleware that checks for safe methods

### ❌ Anti-Pattern 2: Unbounded Queries

```python
# ❌ WRONG: No limit on results
@app.post("/api/query")
async def search(query: QueryRequest):
    results = await db.find(query.filters)  # Could return millions
    return {"results": results}

# ✅ CORRECT: Enforce limits
@app.post("/api/query")
async def search(query: QueryRequest):
    # Enforce maximum limit
    limit = min(query.limit or 20, 100)
    page = max(query.page or 1, 1)
    
    results = await db.find(query.filters)\
        .skip((page - 1) * limit)\
        .limit(limit)
    
    return {"results": results, "total": total}
```

**Why it's wrong:**
- Can exhaust server memory
- Slow response times
- Database overload
- Poor user experience

### ❌ Anti-Pattern 3: Ignoring Caching

```java
// ❌ WRONG: No cache headers
@PostMapping("/search")
public List<Product> search(@RequestBody Query query) {
    return service.search(query);
}

// ✅ CORRECT: Add cache headers
@PostMapping("/search")
public ResponseEntity<List<Product>> search(
        @RequestBody Query query,
        HttpServletRequest request) {
    
    List<Product> results = service.search(query);
    
    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(5, TimeUnit.MINUTES))
        .etag(generateETag(results))
        .body(results);
}
```

**Why it's wrong:**
- Unnecessary database load
- Slower response times
- Higher infrastructure costs
- Poor scalability

### ❌ Anti-Pattern 4: Poor Error Messages

```javascript
// ❌ WRONG: Generic error
{
  "error": "Bad request"
}

// ✅ CORRECT: Detailed error
{
  "error": "Validation failed",
  "message": "Query parameters are invalid",
  "details": [
    {
      "field": "price.min",
      "message": "Must be less than price.max",
      "value": 2500
    },
    {
      "field": "rating",
      "message": "Must be between 1 and 5",
      "value": 6
    }
  ],
  "requestId": "abc-123-xyz"
}
```

### ❌ Anti-Pattern 5: Over-Engineering Simple Queries

```python
# ❌ WRONG: Using QUERY for simple GET
# Client needs: GET /products?category=laptops

# Instead they do:
QUERY /api/products/search
{
  "query": {
    "filters": {
      "category": "laptops"
    }
  }
}

# ✅ CORRECT: Use GET for simple queries
GET /api/products?category=laptops
```

**When to use QUERY:**
- Complex nested structures
- Multiple array parameters
- Large payloads
- Need request body for clarity

**When to use GET:**
- Simple key-value pairs
- URL length < 2000 chars
- Need bookmarkability
- SEO requirements

### ❌ Anti-Pattern 6: No Input Validation

```javascript
// ❌ WRONG: Trusting user input
router.post('/search', async (req, res) => {
    const results = await db.query(req.body);  // Dangerous!
    res.json(results);
});

// ✅ CORRECT: Validate everything
const { body, validationResult } = require('express-validator');

router.post('/search', 
    [
        body('categories').optional().isArray(),
        body('price.min').optional().isFloat(),
        body('page').optional().isInt({ min: 1 })
    ],
    async (req, res) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });
        }
        
        const results = await db.query(req.body);
        res.json(results);
    }
);
```

### ❌ Anti-Pattern 7: Exposing Internal Structure

```python
# ❌ WRONG: Exposing database fields
{
  "query": {
    "_id": {"$oid": "..."},
    "created_at": {"$gte": "2025-01-01"},
    "internal_status": "active"
  }
}

# ✅ CORRECT: Use business-level abstractions
{
  "query": {
    "dateRange": {
      "from": "2025-01-01",
      "to": "2025-01-31"
    },
    "status": "published"
  }
}
```

---

## Performance Considerations

### 1. Database Query Optimization

```java
// ✅ Good: Indexed queries
@Document
public class Product {
    @Indexed
    private String category;
    
    @Indexed
    private String brand;
    
    @Indexed
    private BigDecimal price;
    
    @Indexed
    private Double averageRating;
}

// Compound index for common queries
@CompoundIndex(name = "category_brand_price", 
              def = "{'category': 1, 'brand': 1, 'price': 1}")

// Query execution plan
Specification<Product> spec = buildSearchSpec(query);
Page<Product> results = productRepository.findAll(spec, pageable);

// Monitor query performance
long startTime = System.currentTimeMillis();
Page<Product> results = productRepository.findAll(spec, pageable);
long duration = System.currentTimeMillis() - startTime;
log.info("Query executed in {}ms", duration);
```

### 2. Caching Strategies

```python
# Multi-level caching strategy
class CachedQueryService:
    async def search(self, query: QueryRequest) -> SearchResponse:
        # Level 1: In-memory cache (fastest)
        cache_key = self.generate_key(query)
        result = await self.memory_cache.get(cache_key)
        if result:
            return result
        
        # Level 2: Redis cache (fast)
        result = await self.redis.get(cache_key)
        if result:
            await self.memory_cache.set(cache_key, result, ttl=60)
            return json.loads(result)
        
        # Level 3: Database (slowest)
        result = await self.execute_query(query)
        
        # Populate caches
        await self.redis.setex(cache_key, 300, json.dumps(result))
        await self.memory_cache.set(cache_key, result, ttl=60)
        
        return result
```

### 3. Pagination Best Practices

```javascript
// ✅ Good: Cursor-based pagination for large datasets
{
  "query": {
    "filters": {...},
    "cursor": "eyJpZCI6MTIzNDU2fQ==",  // Base64 encoded cursor
    "limit": 20
  }
}

// Response
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTIzNDU3fQ==",
    "hasMore": true
  }
}

// Implementation
async function searchWithCursor(query) {
    const limit = query.limit || 20;
    const cursor = query.cursor ? JSON.parse(Buffer.from(cursor, 'base64')) : null;
    
    let mongoQuery = buildQuery(query.filters);
    
    if (cursor) {
        mongoQuery._id = { $gt: cursor.id };
    }
    
    const results = await Product.find(mongoQuery)
        .limit(limit + 1)  // Fetch one extra to check if more exist
        .sort('_id');
    
    const hasMore = results.length > limit;
    const data = hasMore ? results.slice(0, -1) : results;
    const nextCursor = hasMore ? 
        Buffer.from(JSON.stringify({ id: data[data.length - 1]._id })).toString('base64') :
        null;
    
    return { data, nextCursor, hasMore };
}
```

### 4. Query Performance Monitoring

```java
// Performance monitoring aspect
@Aspect
@Component
@Slf4j
public class QueryPerformanceAspect {
    
    @Around("@annotation(org.springframework.web.bind.annotation.PostMapping)")
    public Object monitorQueryPerformance(ProceedingJoinPoint pjp) throws Throwable {
        long startTime = System.currentTimeMillis();
        String method = pjp.getSignature().getName();
        
        try {
            Object result = pjp.proceed();
            long duration = System.currentTimeMillis() - startTime;
            
            // Log slow queries
            if (duration > 1000) {
                log.warn("Slow query detected: {} took {}ms", method, duration);
            } else {
                log.debug("Query {} completed in {}ms", method, duration);
            }
            
            // Add performance header
            if (result instanceof ResponseEntity) {
                ((ResponseEntity<?>) result).getHeaders().add(
                    "X-Query-Time", duration + "ms"
                );
            }
            
            return result;
        } catch (Exception e) {
            log.error("Query failed: {}", method, e);
            throw e;
        }
    }
}
```

### 5. Connection Pooling

```yaml
# application.yml - Database connection pool
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      leak-detection-threshold: 60000
```

### Performance Benchmarks

| Operation | GET (simple) | QUERY (complex) | POST (search) |
|-----------|--------------|-----------------|---------------|
| **Latency (p50)** | 45ms | 78ms | 82ms |
| **Latency (p95)** | 120ms | 195ms | 210ms |
| **Latency (p99)** | 250ms | 380ms | 410ms |
| **Cache Hit Rate** | 85% | 72% | 0% (default) |
| **Throughput (req/s)** | 1,200 | 850 | 800 |

**Key Insights:**
- QUERY adds ~30ms overhead vs GET (acceptable for complex queries)
- Caching significantly improves performance
- QUERY outperforms POST due to better caching

---

## Security Considerations

### 1. Input Validation

```java
// ✅ Comprehensive validation
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ProductSearchQuery {
    
    @NotBlank(message = "Query cannot be empty")
    @Size(max = 1000, message = "Query too long")
    private String query;
    
    @Valid
    @NotNull
    @Schema(description = "Search filters")
    private SearchFilters filters;
    
    @Min(1)
    @Max(100)
    private Integer limit = 20;
    
    @Min(1)
    @Max(1000)
    private Integer page = 1;
}

// Custom validator
@Component
public class QueryDepthValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return ProductSearchQuery.class.equals(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        ProductSearchQuery query = (ProductSearchQuery) target;
        
        // Prevent deeply nested queries (DoS protection)
        int depth = calculateDepth(query);
        if (depth > 5) {
            errors.reject("query.depth", "Query too complex");
        }
        
        // Prevent too many conditions (DoS protection)
        int conditions = countConditions(query);
        if (conditions > 50) {
            errors.reject("query.complexity", "Too many conditions");
        }
    }
    
    private int calculateDepth(Object obj, int currentDepth) {
        if (currentDepth > 10) return currentDepth;
        if (obj instanceof Map) {
            return ((Map<?, ?>) obj).values().stream()
                .mapToInt(v -> calculateDepth(v, currentDepth + 1))
                .max().orElse(currentDepth);
        }
        if (obj instanceof Collection) {
            return ((Collection<?>) obj).stream()
                .mapToInt(v -> calculateDepth(v, currentDepth + 1))
                .max().orElse(currentDepth);
        }
        return currentDepth;
    }
}
```

### 2. SQL Injection Prevention

```python
# ✅ Good: Parameterized queries
async def search_products(query: ProductSearchQuery):
    # Using parameterized queries
    sql = """
        SELECT * FROM products 
        WHERE category = ANY(:categories)
        AND price BETWEEN :min_price AND :max_price
        AND rating >= :rating
    """
    
    result = await db.execute(
        sql,
        {
            "categories": query.categories,
            "min_price": query.price.min,
            "max_price": query.price.max,
            "rating": query.rating
        }
    )
    
    return result

# ❌ Bad: String concatenation (SQL injection risk)
async def search_products_unsafe(query: ProductSearchQuery):
    sql = f"""
        SELECT * FROM products 
        WHERE category IN ({','.join(query.categories)})
    """  # Vulnerable to SQL injection!
```

### 3. Rate Limiting

```javascript
// ✅ Rate limiting middleware
const rateLimit = require('express-rate-limit');

const queryLimiter = rateLimit({
    windowMs: 60 * 1000,  // 1 minute
    max: 30,  // 30 requests per minute
    message: {
        error: 'Too many query requests',
        retryAfter: '60 seconds'
    },
    standardHeaders: true,
    legacyHeaders: false,
    // Custom key generator (per user)
    keyGenerator: (req) => {
        return req.user?.id || req.ip;
    }
});

// Apply to query endpoints
router.post('/search', queryLimiter, searchHandler);
```

### 4. Authentication & Authorization

```java
// ✅ Secure QUERY endpoint
@PreAuthorize("hasRole('USER') or hasRole('ADMIN')")
@PostMapping("/search")
public ResponseEntity<SearchResponse> search(
        @Valid @RequestBody QueryRequest request,
        Authentication authentication) {
    
    // Get current user
    User user = (User) authentication.getPrincipal();
    
    // Apply user-specific filters (data isolation)
    request.addFilter("tenantId", user.getTenantId());
    
    // Check permissions
    if (request.containsSensitiveFields() && 
        !user.hasPermission("VIEW_SENSITIVE_DATA")) {
        throw new AccessDeniedException(
            "Insufficient permissions for sensitive fields"
        );
    }
    
    SearchResponse response = service.search(request);
    return ResponseEntity.ok(response);
}
```

### 5. Data Exposure Prevention

```python
# ✅ Field-level security
class SecureQueryService:
    SENSITIVE_FIELDS = {
        'ssn', 'creditCard', 'password', 'salary', 
        'medicalInfo', 'personalEmail'
    }
    
    def apply_field_security(self, query: QueryRequest, user: User):
        """Remove sensitive fields based on user permissions"""
        if not user.has_permission('VIEW_SENSITIVE_DATA'):
            # Remove sensitive fields from query
            if 'fields' in query:
                query['fields'] = [
                    f for f in query['fields'] 
                    if f not in self.SENSITIVE_FIELDS
                ]
            
            # Add filter to exclude sensitive data
            if 'filters' not in query:
                query['filters'] = {}
            query['filters']['excludeSensitive'] = True
    
    def sanitize_output(self, data: dict, user: User) -> dict:
        """Remove sensitive fields from response"""
        if not user.has_permission('VIEW_SENSITIVE_DATA'):
            for field in self.SENSITIVE_FIELDS:
                self._remove_field(data, field)
        return data
```

### 6. Query Complexity Limits

```javascript
// ✅ Prevent DoS through complex queries
class QueryComplexityAnalyzer {
    MAX_DEPTH = 5;
    MAX_CONDITIONS = 50;
    MAX_FIELDS = 20;
    
    analyze(query) {
        const issues = [];
        
        // Check nesting depth
        const depth = this.calculateDepth(query);
        if (depth > this.MAX_DEPTH) {
            issues.push(`Query too complex: depth ${depth} > ${this.MAX_DEPTH}`);
        }
        
        // Check number of conditions
        const conditions = this.countConditions(query);
        if (conditions > this.MAX_CONDITIONS) {
            issues.push(`Too many conditions: ${conditions} > ${this.MAX_CONDITIONS}`);
        }
        
        // Check requested fields
        if (query.fields?.length > this.MAX_FIELDS) {
            issues.push(`Too many fields requested: ${query.fields.length} > ${this.MAX_FIELDS}`);
        }
        
        return issues;
    }
    
    calculateDepth(obj, currentDepth = 0) {
        if (currentDepth > 10) return currentDepth;
        
        if (Array.isArray(obj)) {
            return Math.max(...obj.map(
                item => this.calculateDepth(item, currentDepth + 1)
            ));
        }
        
        if (obj && typeof obj === 'object') {
            return Math.max(...Object.values(obj).map(
                value => this.calculateDepth(value, currentDepth + 1)
            ));
        }
        
        return currentDepth;
    }
}
```

---

## Testing Strategies

### 1. Unit Testing

```java
// ProductSearchControllerTest.java
@ExtendWith(MockitoExtension.class)
class ProductSearchControllerTest {
    
    @Mock
    private ProductService productService;
    
    @InjectMocks
    private ProductController controller;
    
    @Test
    void searchProducts_ValidQuery_ReturnsResults() throws Exception {
        // Arrange
        ProductSearchQuery query = new ProductSearchQuery();
        query.setCategories(List.of("laptops"));
        query.setBrands(List.of("apple"));
        query.setPrice(new PriceRange(new BigDecimal("1000"), new BigDecimal("2500")));
        
        Page<ProductDTO> expectedResults = new PageImpl<>(List.of(
            createProductDTO("MacBook Pro")
        ));
        
        when(productService.search(any(), any())).thenReturn(expectedResults);
        
        // Act
        ResponseEntity<Page<ProductDTO>> response = 
            controller.searchProducts(query, PageRequest.of(0, 20));
        
        // Assert
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).hasSize(1);
        assertThat(response.getHeaders().getCacheControl())
            .contains("max-age=300");
    }
    
    @Test
    void searchProducts_InvalidQuery_ReturnsBadRequest() throws Exception {
        // Arrange
        ProductSearchQuery query = new ProductSearchQuery();
        query.setPrice(new PriceRange(
            new BigDecimal("2500"),  // min > max
            new BigDecimal("1000")
        ));
        
        // Act & Assert
        assertThrows(MethodArgumentNotValidException.class, () ->
            controller.searchProducts(query, PageRequest.of(0, 20))
        );
    }
    
    @Test
    void searchProducts_Unauthorized_Returns401() throws Exception {
        // Arrange
        ProductSearchQuery query = new ProductSearchQuery();
        
        // Act
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setUserContext(null);  // No authentication
        
        // Assert
        assertThrows(AuthenticationException.class, () ->
            controller.searchProducts(query, PageRequest.of(0, 20))
        );
    }
}
```

### 2. Integration Testing

```python
# test_search_api.py
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

class TestProductSearch:
    
    def test_search_with_valid_query(self):
        """Test successful search with valid query"""
        query = {
            "categories": ["laptops"],
            "brands": ["apple"],
            "price": {"min": 1000, "max": 2500}
        }
        
        response = client.post("/api/products/search", json=query)
        
        assert response.status_code == 200
        assert "data" in response.json()
        assert "pagination" in response.json()
        assert response.headers["Cache-Control"] == "max-age=300, public"
    
    def test_search_with_invalid_query(self):
        """Test validation of invalid query"""
        query = {
            "price": {"min": 2500, "max": 1000}  # Invalid range
        }
        
        response = client.post("/api/products/search", json=query)
        
        assert response.status_code == 422
        assert "Validation failed" in response.json()["detail"]
    
    def test_search_with_too_many_filters(self):
        """Test query complexity limits"""
        query = {
            "categories": ["cat" + str(i) for i in range(100)]  # Too many
        }
        
        response = client.post("/api/products/search", json=query)
        
        assert response.status_code == 400
        assert "too many" in response.json()["detail"].lower()
    
    def test_search_caching(self):
        """Test that caching works correctly"""
        query = {"categories": ["laptops"]}
        
        # First request (cache miss)
        response1 = client.post("/api/products/search", json=query)
        assert response1.headers["X-Cache"] == "MISS"
        
        # Second request (cache hit)
        response2 = client.post("/api/products/search", json=query)
        assert response2.headers["X-Cache"] == "HIT"
        assert response2.json() == response1.json()
```

### 3. Performance Testing

```javascript
// performance-test.js
const autocannon = require('autocannon');

async function runPerformanceTest() {
    const query = {
        categories: ["laptops", "gaming"],
        brands: ["apple", "dell", "hp"],
        price: { min: 1000, max: 2500 },
        rating: 4
    };
    
    const result = await autocannon({
        url: 'http://localhost:3000',
        method: 'POST',
        path: '/api/products/search',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(query),
        connections: 100,
        duration: 30,
        amount: 10000
    });
    
    console.log('Performance Results:');
    console.log('===================');
    console.log(`Total requests: ${result.requests.total}`);
    console.log(`Requests/sec: ${result.requests.average}`);
    console.log(`Latency (avg): ${result.latency.mean}ms`);
    console.log(`Latency (p95): ${result.latency.p95}ms`);
    console.log(`Latency (p99): ${result.latency.p99}ms`);
    console.log(`Throughput: ${(result.throughput.average / 1024 / 1024).toFixed(2)} MB/s`);
}

runPerformanceTest();
```

**Expected Performance Metrics:**
```
Performance Results:
===================
Total requests: 10000
Requests/sec: 333
Latency (avg): 45ms
Latency (p95): 120ms
Latency (p99): 250ms
Throughput: 2.45 MB/s
```

### 4. Security Testing

```python
# test_security.py
class TestSecurity:
    
    def test_sql_injection_prevention(self):
        """Test SQL injection attempts are blocked"""
        malicious_queries = [
            {"categories": ["'; DROP TABLE products; --"]},
            {"brands": ["' OR '1'='1"]},
            {"query": {"$where": "function() { return true; }"}}
        ]
        
        for malicious_query in malicious_queries:
            response = client.post("/api/products/search", json=malicious_query)
            
            # Should return 400 or sanitize input
            assert response.status_code in [400, 422]
    
    def test_query_complexity_limits(self):
        """Test DoS protection through complex queries"""
        # Deeply nested query
        deep_query = {"filters": {"a": {"b": {"c": {"d": {"e": "value"}}}}}}
        
        response = client.post("/api/products/search", json=deep_query)
        assert response.status_code == 400
    
    def test_rate_limiting(self):
        """Test rate limiting works"""
        # Send 100 requests rapidly
        for i in range(100):
            response = client.post("/api/products/search", json={})
            if i >= 50:  # After rate limit
                assert response.status_code == 429
    
    def test_authentication_required(self):
        """Test that authentication is enforced"""
        response = client.post("/api/products/search", json={})
        assert response.status_code == 401
```

### 5. Contract Testing

```yaml
# pact-contract.yml
---
consumer:
  name: WebApp
  dependencies:
    - name: ProductAPI
      version: "1.0.0"
interactions:
  - description: Search products with complex filters
    request:
      method: POST
      path: /api/products/search
      headers:
        Content-Type: application/json
      body:
        query:
          categories: ["laptops"]
          brands: ["apple"]
          price:
            min: 1000
            max: 2500
    response:
      status: 200
      headers:
        Content-Type: application/json
        Cache-Control: max-age=300
      body:
        data: []
        pagination:
          page: 1
          limit: 20
          total: 0
          pages: 0
```

---

## Migration Guide

### From POST /search to QUERY

#### Step 1: Assessment

```bash
# Identify all POST /search endpoints
grep -r "POST.*search" --include="*.java" --include="*.ts" --include="*.py"

# Document current usage
# - Request/response formats
# - Client libraries
# - Documentation
# - Monitoring/analytics
```

#### Step 2: Create QUERY Endpoint

```java
// Phase 1: Add QUERY endpoint alongside POST
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // Keep old POST endpoint for backward compatibility
    @PostMapping("/search")
    public ResponseEntity<SearchResponse> searchPost(
            @Valid @RequestBody QueryRequest request) {
        return search(request);
    }
    
    // Add new QUERY endpoint
    @PostMapping("/search")
    public ResponseEntity<SearchResponse> searchQuery(
            @Valid @RequestBody QueryRequest request) {
        return search(request);
    }
    
    // Shared implementation
    private ResponseEntity<SearchResponse> search(QueryRequest request) {
        SearchResponse response = service.search(request);
        return ResponseEntity.ok(response);
    }
}
```

#### Step 3: Update Clients

```javascript
// client-migration.js

// Phase 1: Feature flag
const USE_QUERY_METHOD = process.env.USE_QUERY_METHOD === 'true';

async function searchProducts(query) {
    const url = '/api/products/search';
    
    if (USE_QUERY_METHOD) {
        // Use new QUERY method
        return fetch(url, {
            method: 'QUERY',  // Note: May need custom header for now
            headers: {
                'Content-Type': 'application/json',
                'X-HTTP-Method-Override': 'QUERY'  // Fallback
            },
            body: JSON.stringify({ query })
        });
    } else {
        // Use old POST method
        return fetch(url, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(query)
        });
    }
}

// Phase 2: Gradual rollout
// - 10% traffic → QUERY
// - 50% traffic → QUERY
// - 100% traffic → QUERY
```

#### Step 4: Monitor and Validate

```java
// Migration monitoring
@Component
public class MigrationMonitor {
    
    @EventListener
    public void onSearchCompleted(SearchEvent event) {
        // Track which method was used
        String method = event.getHttpMethod();
        long duration = event.getDuration();
        boolean success = event.isSuccess();
        
        // Send to monitoring
        metrics.counter("search.requests", 
            "method", method,
            "success", String.valueOf(success)
        ).increment();
        
        metrics.timer("search.duration",
            "method", method
        ).record(duration, TimeUnit.MILLISECONDS);
    }
}
```

#### Step 5: Deprecate Old Endpoint

```java
// Phase 3: Add deprecation warning
@PostMapping("/search")
@Deprecated(since = "2.0", forRemoval = true)
@Operation(
    deprecated = true,
    description = "Deprecated: Use QUERY /search instead"
)
public ResponseEntity<SearchResponse> searchPost(
        @Valid @RequestBody QueryRequest request,
        @RequestHeader("User-Agent") String userAgent) {
    
    // Add deprecation header
    return ResponseEntity.ok()
        .header("Sunset", "Sat, 01 Jul 2026 00:00:00 GMT")
        .header("Deprecation", "true")
        .body(service.search(request));
}

// Phase 4: Remove old endpoint (after deprecation period)
// @DeleteMapping("/search")  // Remove in version 3.0
```

### Framework Support

#### Spring Boot (Current Status)

```java
// Spring Boot doesn't natively support QUERY yet
// Workaround: Use custom configuration

@Configuration
public class QueryMethodConfig {
    
    @Bean
    public WebMvcConfigurer queryMethodSupport() {
        return new WebMvcConfigurer() {
            @Override
            public void addArgumentResolvers(
                    List<HandlerMethodArgumentResolver> resolvers) {
                resolvers.add(new QueryMethodArgumentResolver());
            }
        };
    }
}

// Alternative: Use HTTP method override header
// X-HTTP-Method-Override: QUERY
```

#### Express.js (Current Status)

```javascript
// Express.js - Add QUERY method support
const express = require('express');
const app = express();

// Middleware to handle QUERY method
app.use((req, res, next) => {
    if (req.method === 'POST' && 
        req.headers['x-http-method-override'] === 'QUERY') {
        req.method = 'QUERY';
    }
    next();
});

// Now you can use QUERY
app.QUERY('/api/search', (req, res) => {
    res.json({ data: req.body });
});
```

#### FastAPI (Current Status)

```python
# FastAPI - Custom method support
from fastapi import FastAPI, Request

app = FastAPI()

@app.api_route("/api/search", methods=["QUERY", "POST"])
async def search(request: Request, query: QueryRequest):
    # Handle both QUERY and POST
    method = request.method
    if method == "QUERY":
        # QUERY-specific logic
        pass
    else:
        # POST fallback
        pass
    
    return {"data": results}
```

### Backward Compatibility Strategy

```mermaid
graph LR
    A[Version 1.0] --> B[POST /search]
    B --> C[Version 2.0]
    C --> D[POST /search + QUERY /search]
    D --> E[Version 2.5]
    E --> F[QUERY /search (primary)]
    F --> G[POST /search (deprecated)]
    G --> H[Version 3.0]
    H --> I[QUERY /search only]
    
    style D fill:#ffd93d
    style F fill:#95e1d3
    style I fill:#4ecdc4
```

**Migration Timeline:**
- **Month 1-2:** Add QUERY endpoint, feature flag
- **Month 3-4:** Gradual rollout (10% → 50% → 100%)
- **Month 5-6:** Monitor, fix issues
- **Month 7:** Add deprecation warning to POST
- **Month 12:** Remove POST endpoint

---

## Common Pitfalls & Troubleshooting

### Pitfall 1: Framework Doesn't Support QUERY

**Problem:** Your framework doesn't recognize the QUERY method

**Solutions:**

```javascript
// Solution 1: HTTP Method Override
// Client sends POST with override header
POST /api/search
X-HTTP-Method-Override: QUERY

// Server-side handling
app.use((req, res, next) => {
    if (req.headers['x-http-method-override']) {
        req.method = req.headers['x-http-method-override'];
    }
    next();
});

// Solution 2: Custom routing
app.QUERY = function(path, handler) {
    this.post(path, (req, res) => {
        req.method = 'QUERY';
        return handler(req, res);
    });
};

// Solution 3: Use middleware
const queryMethod = require('express-http-method-override');
app.use(queryMethod({
    methods: ['QUERY']
}));
```

### Pitfall 2: Caching Not Working

**Problem:** QUERY responses aren't being cached

**Diagnosis:**
```bash
# Check response headers
curl -v -X POST http://localhost:3000/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "test"}'

# Look for:
# Cache-Control: max-age=300, public
# ETag: "abc123"
```

**Solution:**
```java
// ✅ Proper cache configuration
@PostMapping("/search")
public ResponseEntity<SearchResponse> search(@RequestBody QueryRequest request) {
    SearchResponse response = service.search(request);
    
    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(5, TimeUnit.MINUTES)
            .cachePublic()
            .mustRevalidate())
        .eTag(generateETag(response))
        .body(response);
}

// Client-side: Respect cache headers
fetch('/api/search', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Cache-Control': 'max-age=300'
    },
    body: JSON.stringify(query)
});
```

### Pitfall 3: Proxy/Gateway Rejects QUERY

**Problem:** API gateway or proxy doesn't allow QUERY method

**Solution:**

```nginx
# Nginx configuration
server {
    location /api/ {
        # Allow QUERY method
        limit_except GET POST QUERY OPTIONS {
            deny all;
        }
        
        proxy_pass http://backend;
    }
}

# Apache configuration
<Location "/api">
    <LimitExcept GET POST QUERY OPTIONS>
        Require all denied
    </LimitExcept>
</Location>
```

```yaml
# AWS API Gateway
# Add QUERY to allowed methods
/apis:
  /search:
    post:
      x-amazon-apigateway-integration:
        httpMethod: POST
        type: aws_proxy
    x-amazon-apigateway-any-method:
      x-amazon-apigateway-integration:
        httpMethod: POST
        type: aws_proxy
```

### Pitfall 4: Browser Limitations

**Problem:** Browser fetch API doesn't support QUERY method

**Solution:**

```javascript
// ✅ Use method override
async function queryAPI(endpoint, data) {
    const response = await fetch(endpoint, {
        method: 'POST',  // Use POST
        headers: {
            'Content-Type': 'application/json',
            'X-HTTP-Method-Override': 'QUERY'  // Override header
        },
        body: JSON.stringify(data)
    });
    
    return response.json();
}

// ✅ Or use custom fetch wrapper
class APIClient {
    async query(endpoint, data) {
        return this.request(endpoint, data, 'QUERY');
    }
    
    async request(endpoint, data, method) {
        const response = await fetch(endpoint, {
            method: method === 'QUERY' ? 'POST' : method,
            headers: {
                'Content-Type': 'application/json',
                ...(method === 'QUERY' && {
                    'X-HTTP-Method-Override': method
                })
            },
            body: method !== 'GET' ? JSON.stringify(data) : undefined
        });
        
        return response.json();
    }
}
```

### Pitfall 5: Idempotency Violations

**Problem:** QUERY operation accidentally modifies data

**Prevention:**

```java
// ✅ Use database transaction with read-only mode
@Transactional(readOnly = true)
public SearchResponse search(QueryRequest request) {
    // Database enforces read-only
    return service.search(request);
}

// ✅ Use separate read replica
@Configuration
public class DatabaseConfig {
    @Bean
    @Primary
    @ConfigurationProperties("app.datasource.read")
    public DataSource readDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("app.datasource.write")
    public DataSource writeDataSource() {
        return DataSourceBuilder.create().build();
    }
}

// Route QUERY to read replica
@QueriesReplica
@PostMapping("/search")
public SearchResponse search(@RequestBody QueryRequest request) {
    return service.search(request);
}
```

### Pitfall 6: Large Request Bodies

**Problem:** Request body too large, causing performance issues

**Solution:**

```javascript
// ✅ Limit request body size
app.use(express.json({ 
    limit: '1mb',
    verify: (req, res, buf) => {
        // Log request size
        console.log(`Request size: ${buf.length} bytes`);
        
        // Reject if too large
        if (buf.length > 1024 * 1024) {  // 1MB
            throw new Error('Request body too large');
        }
    }
}));

// ✅ Implement pagination for large queries
{
  "query": {
    "filters": {...},
    "batchSize": 100,
    "batchNumber": 1
  }
}
```

### Troubleshooting Checklist

```markdown
## QUERY Method Troubleshooting Checklist

### Request Issues
- [ ] Request body is valid JSON
- [ ] Content-Type header is set to application/json
- [ ] Request size is within limits
- [ ] Authentication token is valid
- [ ] User has required permissions

### Response Issues
- [ ] Response includes Cache-Control headers
- [ ] Response includes ETag for caching
- [ ] Status code is appropriate (200, 400, 401, etc.)
- [ ] Response format matches schema
- [ ] Error messages are clear

### Performance Issues
- [ ] Database queries are indexed
- [ ] Caching is enabled
- [ ] Connection pooling is configured
- [ ] Query complexity is limited
- [ ] Pagination is implemented

### Security Issues
- [ ] Input validation is enabled
- [ ] SQL injection prevention is active
- [ ] Rate limiting is configured
- [ ] Authentication is required
- [ ] Authorization checks are in place
- [ ] Sensitive fields are filtered

### Framework Issues
- [ ] QUERY method is supported or overridden
- [ ] Proxy/gateway allows QUERY method
- [ ] CORS is configured correctly
- [ ] Request size limits are set
- [ ] Error handling is implemented
```

---

## Practice Exercises

### Exercise 1: Basic QUERY Implementation

**Difficulty:** ⭐ Beginner  
**Time:** 30 minutes

**Task:** Implement a basic QUERY endpoint for searching blog posts

**Requirements:**
1. Create a QUERY endpoint `/api/posts/search`
2. Accept filters: category, author, tags, date range
3. Implement pagination (page, limit)
4. Add sorting (field, order)
5. Return results with pagination metadata

**Solution:**

```python
# models.py
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import datetime

class DateRange(BaseModel):
    from_date: Optional[datetime]
    to_date: Optional[datetime]

class SortConfig(BaseModel):
    field: str = "published_at"
    order: str = "desc"

class PaginationConfig(BaseModel):
    page: int = Field(1, ge=1)
    limit: int = Field(20, ge=1, le=100)

class PostSearchQuery(BaseModel):
    categories: Optional[List[str]] = None
    author: Optional[str] = None
    tags: Optional[List[str]] = None
    date_range: Optional[DateRange] = None
    sort: SortConfig = SortConfig()
    pagination: PaginationConfig = PaginationConfig()

# main.py
from fastapi import FastAPI, Response
from motor.motor_asyncio import AsyncIOMotorClient
from bson import ObjectId

app = FastAPI()
client = AsyncIOMotorClient("mongodb://localhost:27017")
db = client.blog

@app.post("/api/posts/search")
async def search_posts(
    query: PostSearchQuery,
    response: Response
):
    # Build MongoDB query
    mongo_query = {}
    
    if query.categories:
        mongo_query["category"] = {"$in": query.categories}
    
    if query.author:
        mongo_query["author"] = query.author
    
    if query.tags:
        mongo_query["tags"] = {"$in": query.tags}
    
    if query.date_range:
        date_filter = {}
        if query.date_range.from_date:
            date_filter["$gte"] = query.date_range.from_date
        if query.date_range.to_date:
            date_filter["$lte"] = query.date_range.to_date
        mongo_query["published_at"] = date_filter
    
    # Execute query with pagination
    skip = (query.pagination.page - 1) * query.pagination.limit
    
    posts = await db.posts.find(mongo_query)\
        .sort(query.sort.field, 1 if query.sort.order == "asc" else -1)\
        .skip(skip)\
        .limit(query.pagination.limit)\
        .to_list(length=query.pagination.limit)
    
    total = await db.posts.count_documents(mongo_query)
    
    # Set cache headers
    response.headers["Cache-Control"] = "max-age=300, public"
    
    return {
        "data": posts,
        "pagination": {
            "page": query.pagination.page,
            "limit": query.pagination.limit,
            "total": total,
            "pages": (total + query.pagination.limit - 1) // query.pagination.limit
        }
    }
```

**Test the Solution:**

```bash
# Test with curl
curl -X POST http://localhost:8000/api/posts/search \
  -H "Content-Type: application/json" \
  -d '{
    "categories": ["technology", "programming"],
    "tags": ["python", "fastapi"],
    "date_range": {
      "from_date": "2025-01-01T00:00:00",
      "to_date": "2025-12-31T23:59:59"
    },
    "sort": {
      "field": "published_at",
      "order": "desc"
    },
    "pagination": {
      "page": 1,
      "limit": 10
    }
  }'
```

**Expected Response:**
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

---

### Exercise 2: Advanced Filtering with Nested Conditions

**Difficulty:** ⭐⭐ Intermediate  
**Time:** 45 minutes

**Task:** Implement advanced filtering with OR conditions and nested logic

**Requirements:**
1. Support OR conditions between different filter groups
2. Support nested AND conditions within groups
3. Implement price range filtering
4. Add rating filters with minimum review count
5. Support multiple warehouse locations

**Solution:**

```java
// models/ProductSearchQuery.java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ProductSearchQuery {
    
    private List<String> categories;
    private List<String> brands;
    private PriceRange price;
    private RatingFilter rating;
    private Boolean inStock;
    private List<String> warehouses;
    private List<OrCondition> orConditions;
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class PriceRange {
        private BigDecimal min;
        private BigDecimal max;
    }
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class RatingFilter {
        private Integer min;
        private Integer reviewCountMin;
    }
    
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class OrCondition {
        private String category;
        private String brand;
        private BigDecimal maxPrice;
        private Integer minRating;
    }
}

// service/ProductSearchService.java
@Service
public class ProductSearchService {
    
    public Page<Product> search(ProductSearchQuery query, Pageable pageable) {
        Specification<Product> spec = buildSpecification(query);
        return productRepository.findAll(spec, pageable);
    }
    
    private Specification<Product> buildSpecification(ProductSearchQuery query) {
        List<Specification<Product>> specs = new ArrayList<>();
        
        // Basic filters (AND conditions)
        if (CollectionUtils.isNotEmpty(query.getCategories())) {
            specs.add((root, cq, cb) -> 
                root.get("category").in(query.getCategories()));
        }
        
        if (CollectionUtils.isNotEmpty(query.getBrands())) {
            specs.add((root, cq, cb) -> 
                root.get("brand").in(query.getBrands()));
        }
        
        if (query.getPrice() != null) {
            specs.add(buildPriceSpec(query.getPrice()));
        }
        
        if (query.getRating() != null) {
            specs.add(buildRatingSpec(query.getRating()));
        }
        
        if (Boolean.TRUE.equals(query.getInStock())) {
            specs.add((root, cq, cb) -> 
                cb.greaterThan(root.get("stockQuantity"), 0));
        }
        
        if (CollectionUtils.isNotEmpty(query.getWarehouses())) {
            specs.add((root, cq, cb) -> 
                root.get("warehouse").in(query.getWarehouses()));
        }
        
        // OR conditions
        if (CollectionUtils.isNotEmpty(query.getOrConditions())) {
            specs.add(buildOrSpec(query.getOrConditions()));
        }
        
        // Combine all with AND
        return specs.stream()
            .reduce(Specification::and)
            .orElse(null);
    }
    
    private Specification<Product> buildOrSpec(
            List<ProductSearchQuery.OrCondition> conditions) {
        return (root, cq, cb) -> {
            List<Predicate> orPredicates = new ArrayList<>();
            
            for (ProductSearchQuery.OrCondition condition : conditions) {
                List<Predicate> andPredicates = new ArrayList<>();
                
                if (condition.getCategory() != null) {
                    andPredicates.add(cb.equal(
                        root.get("category"), condition.getCategory()));
                }
                
                if (condition.getBrand() != null) {
                    andPredicates.add(cb.equal(
                        root.get("brand"), condition.getBrand()));
                }
                
                if (condition.getMaxPrice() != null) {
                    andPredicates.add(cb.lessThanOrEqualTo(
                        root.get("price"), condition.getMaxPrice()));
                }
                
                if (condition.getMinRating() != null) {
                    andPredicates.add(cb.greaterThanOrEqualTo(
                        root.get("averageRating"), condition.getMinRating()));
                }
                
                orPredicates.add(cb.and(andPredicates.toArray(new Predicate[0])));
            }
            
            return cb.or(orPredicates.toArray(new Predicate[0]));
        };
    }
    
    private Specification<Product> buildPriceSpec(ProductSearchQuery.PriceRange price) {
        return (root, cq, cb) -> {
            List<Predicate> predicates = new ArrayList<>();
            
            if (price.getMin() != null) {
                predicates.add(cb.greaterThanOrEqualTo(
                    root.get("price"), price.getMin()));
            }
            
            if (price.getMax() != null) {
                predicates.add(cb.lessThanOrEqualTo(
                    root.get("price"), price.getMax()));
            }
            
            return cb.and(predicates.toArray(new Predicate[0]));
        };
    }
}
```

**Test the Solution:**

```json
POST /api/products/search
{
  "categories": ["Electronics"],
  "inStock": true,
  "orConditions": [
    {
      "category": "Laptops",
      "brand": "Apple",
      "maxPrice": 2500
    },
    {
      "category": "Smartphones",
      "minRating": 4
    }
  ],
  "warehouses": ["US", "EU"]
}
```

**Explanation:**
This query finds:
- Electronics that are in stock
- AND (either Apple laptops under $2500 OR smartphones with rating ≥ 4)
- AND available in US or EU warehouses

---

### Exercise 3: Performance Optimization and Caching

**Difficulty:** ⭐⭐⭐ Advanced  
**Time:** 60 minutes

**Task:** Optimize QUERY endpoint with multi-level caching and performance monitoring

**Requirements:**
1. Implement multi-level caching (memory → Redis → database)
2. Add query result caching with TTL
3. Implement cache invalidation strategy
4. Add performance monitoring and metrics
5. Optimize database queries with indexes

**Solution:**

```javascript
// services/CachedQueryService.js
class CachedQueryService {
    constructor() {
        // Level 1: In-memory cache (LRU)
        this.memoryCache = new Map();
        this.memoryCacheMaxSize = 1000;
        
        // Level 2: Redis cache
        this.redis = redis.createClient();
        
        // Metrics
        this.metrics = {
            hits: 0,
            misses: 0,
            dbQueries: 0
        };
    }
    
    async search(query) {
        const cacheKey = this.generateCacheKey(query);
        
        // Level 1: Check memory cache
        const memoryResult = this.checkMemoryCache(cacheKey);
        if (memoryResult) {
            this.metrics.hits++;
            return { ...memoryResult, cached: true, cacheLevel: 'memory' };
        }
        
        // Level 2: Check Redis cache
        const redisResult = await this.checkRedisCache(cacheKey);
        if (redisResult) {
            this.metrics.hits++;
            // Populate memory cache
            this.setMemoryCache(cacheKey, redisResult);
            return { ...redisResult, cached: true, cacheLevel: 'redis' };
        }
        
        // Level 3: Query database
        this.metrics.misses++;
        this.metrics.dbQueries++;
        
        const dbResult = await this.queryDatabase(query);
        
        // Populate caches
        this.setMemoryCache(cacheKey, dbResult);
        await this.setRedisCache(cacheKey, dbResult, 300);  // 5 min TTL
        
        return { ...dbResult, cached: false, cacheLevel: 'database' };
    }
    
    checkMemoryCache(key) {
        if (this.memoryCache.has(key)) {
            const value = this.memoryCache.get(key);
            // Move to end (LRU)
            this.memoryCache.delete(key);
            this.memoryCache.set(key, value);
            return value;
        }
        return null;
    }
    
    setMemoryCache(key, value) {
        // LRU eviction
        if (this.memoryCache.size >= this.memoryCacheMaxSize) {
            const firstKey = this.memoryCache.keys().next().value;
            this.memoryCache.delete(firstKey);
        }
        
        this.memoryCache.set(key, value);
    }
    
    async checkRedisCache(key) {
        try {
            const cached = await this.redis.get(key);
            return cached ? JSON.parse(cached) : null;
        } catch (error) {
            console.error('Redis cache error:', error);
            return null;
        }
    }
    
    async setRedisCache(key, value, ttl) {
        try {
            await this.redis.setex(key, ttl, JSON.stringify(value));
        } catch (error) {
            console.error('Redis cache set error:', error);
        }
    }
    
    generateCacheKey(query) {
        // Create deterministic cache key
        const sortedQuery = JSON.stringify(query, Object.keys(query).sort());
        return `query:${crypto.createHash('md5').update(sortedQuery).digest('hex')}`;
    }
    
    getMetrics() {
        const total = this.metrics.hits + this.metrics.misses;
        const hitRate = total > 0 ? (this.metrics.hits / total * 100).toFixed(2) : 0;
        
        return {
            ...this.metrics,
            hitRate: `${hitRate}%`,
            memoryCacheSize: this.memoryCache.size
        };
    }
}

// Performance monitoring middleware
class PerformanceMiddleware {
    constructor(cachedQueryService) {
        this.queryService = cachedQueryService;
    }
    
    middleware() {
        return (req, res, next) => {
            const startTime = Date.now();
            
            // Add metrics to response
            res.on('finish', () => {
                const duration = Date.now() - startTime;
                const metrics = this.queryService.getMetrics();
                
                res.setHeader('X-Query-Time', `${duration}ms`);
                res.setHeader('X-Cache-Hit-Rate', metrics.hitRate);
                res.setHeader('X-DB-Queries', metrics.dbQueries);
            });
            
            next();
        };
    }
}

// Database optimization
async function optimizeDatabase() {
    // Create indexes
    await db.collection('products').createIndex({ category: 1 });
    await db.collection('products').createIndex({ brand: 1 });
    await db.collection('products').createIndex({ price: 1 });
    await db.collection('products').createIndex({ 
        category: 1, brand: 1, price: 1 
    });  // Compound index
    
    // Analyze query performance
    const explain = await db.collection('products')
        .find({ category: 'laptops', price: { $lt: 2500 } })
        .explain('executionStats');
    
    console.log('Query plan:', explain.executionStats);
}
```

**Usage:**

```javascript
// app.js
const cachedQueryService = new CachedQueryService();
const performanceMiddleware = new PerformanceMiddleware(cachedQueryService);

app.use('/api', performanceMiddleware.middleware());

app.post('/api/products/search', async (req, res) => {
    const result = await cachedQueryService.search(req.body);
    
    res.json({
        ...result,
        metrics: cachedQueryService.getMetrics()
    });
});

// Metrics endpoint
app.get('/api/metrics', (req, res) => {
    res.json(cachedQueryService.getMetrics());
});
```

**Expected Output:**
```json
{
  "data": [...],
  "pagination": {...},
  "cached": true,
  "cacheLevel": "redis",
  "metrics": {
    "hits": 1523,
    "misses": 234,
    "dbQueries": 234,
    "hitRate": "86.68%",
    "memoryCacheSize": 847
  }
}
```

---

## Question Bank

### Test Your Understanding (10 Questions)

1. **What is the primary purpose of the HTTP QUERY method?**
   - A) To create new resources
   - B) To update existing resources
   - C) To perform complex read-only queries with request bodies
   - D) To delete resources
   
   **Answer: C** - The QUERY method is designed for complex read-only queries that require a request body, filling the gap between GET (no body) and POST (semantically incorrect for queries).

2. **Which of the following is NOT a characteristic of the QUERY method?**
   - A) Safe
   - B) Idempotent
   - C) Cacheable
   - D) State-changing
   
   **Answer: D** - QUERY is safe, idempotent, and cacheable. It does NOT change server state.

3. **When should you use GET instead of QUERY?**
   - A) When you need a request body
   - B) When parameters fit in URL and are simple
   - C) When performing complex nested queries
   - D) When you need to send arrays
   
   **Answer: B** - Use GET for simple parameters that fit comfortably in the URL (< 2000 characters).

4. **What is the main problem with using POST for search operations?**
   - A) POST is too slow
   - B) Semantic mismatch - POST implies state change
   - C) POST doesn't support JSON
   - D) POST can't be cached
   
   **Answer: B** - The main issue is semantic incorrectness - POST implies creating/processing resources, not querying.

5. **Which HTTP method is idempotent?**
   - A) POST
   - B) QUERY
   - C) Both GET and QUERY
   - D) Neither
   
   **Answer: C** - Both GET and QUERY are idempotent. Multiple identical requests produce the same result.

6. **What cache header should you include in QUERY responses?**
   - A) Cache-Control: no-cache
   - B) Cache-Control: max-age=300, public
   - C) Cache-Control: private
   - D) No cache headers needed
   
   **Answer: B** - QUERY responses should include Cache-Control headers like `max-age=300, public` to enable caching.

7. **Which status code indicates a semantically invalid query (valid JSON but invalid logic)?**
   - A) 400 Bad Request
   - B) 401 Unauthorized
   - C) 422 Unprocessable Entity
   - D) 500 Internal Server Error
   
   **Answer: C** - 422 Unprocessable Entity is used when the request is well-formed but semantically invalid.

8. **What is a key benefit of using QUERY over POST for searches?**
   - A) Faster execution
   - B) Better caching support
   - C) Simpler implementation
   - D) More browser support
   
   **Answer: B** - QUERY enables better caching because it's semantically a safe, idempotent operation.

9. **Which is an anti-pattern when using QUERY?**
   - A) Using QUERY for read-only complex queries
   - B) Using QUERY to update user data
   - C) Implementing caching
   - D) Adding input validation
   
   **Answer: B** - Using QUERY for state-changing operations violates HTTP semantics.

10. **What should you do if your framework doesn't support QUERY natively?**
    - A) Don't use QUERY at all
    - B) Use X-HTTP-Method-Override header
    - C) Switch to GET
    - D) Use PUT instead
    
    **Answer: B** - Use the X-HTTP-Method-Override header to indicate QUERY method when the framework doesn't support it natively.

---

### Common Interview Questions (15 Questions)

11. **Explain the difference between GET and QUERY methods.**
    
    **Answer:** GET is for simple resource retrieval with parameters in the URL, limited by URL length (~2-8KB). QUERY is for complex queries with request bodies, allowing structured data without URL length limits. Both are safe, idempotent, and cacheable, but QUERY handles complex nested structures better.

12. **Why is using POST for search operations considered a semantic mismatch?**
    
    **Answer:** POST semantically means "create" or "process" - it implies the server will create a new resource or process data. Search operations are read-only and don't modify state, making POST semantically incorrect. This causes confusion in API documentation, breaks caching expectations, and violates HTTP semantics.

13. **How does QUERY enable better caching compared to POST?**
    
    **Answer:** QUERY is semantically a safe, idempotent operation, making it cacheable by default (with proper headers). POST is not cacheable by default because it's assumed to modify state. QUERY responses can include Cache-Control and ETag headers, enabling CDN and browser caching.

14. **What are the idempotency guarantees of QUERY?**
    
    **Answer:** QUERY is idempotent - making the same request multiple times produces the same result (assuming underlying data hasn't changed). This makes it safe to retry, enables caching, and allows clients to re-issue requests without side effects.

15. **How would you handle QUERY in a framework that doesn't support it natively?**
    
    **Answer:** Use the X-HTTP-Method-Override header. The client sends a POST request with `X-HTTP-Method-Override: QUERY` header. The server reads this header and processes the request as a QUERY operation. This provides backward compatibility while using the correct semantics.

16. **What security considerations are specific to QUERY endpoints?**
    
    **Answer:** Input validation (prevent injection attacks), query complexity limits (prevent DoS), rate limiting, authentication/authorization, field-level security (prevent data exposure), SQL injection prevention, and monitoring for suspicious query patterns.

17. **How do you prevent DoS attacks through complex QUERY requests?**
    
    **Answer:** Implement query complexity limits: maximum nesting depth (e.g., 5 levels), maximum number of conditions (e.g., 50), maximum requested fields (e.g., 20), request body size limits, and rate limiting per user/IP.

18. **What is the current adoption status of HTTP QUERY?**
    
    **Answer:** QUERY is currently an IETF draft specification, not yet an official standard. Framework support is limited/emerging. Most implementations use workarounds like X-HTTP-Method-Override. Adoption is growing but not yet widespread.

19. **How does QUERY compare to GraphQL?**
    
    **Answer:** Both allow complex queries with structured data. GraphQL is a complete query language with schema, type system, and single endpoint. QUERY is an HTTP method that works with existing REST patterns. QUERY is simpler to adopt in existing REST APIs, while GraphQL offers more flexibility but requires more infrastructure.

20. **When would you choose QUERY over GraphQL?**
    
    **Answer:** Choose QUERY when: you have an existing REST API, need simple query capabilities, want HTTP caching benefits, don't need GraphQL's type system, or want minimal infrastructure changes. Choose GraphQL when: you need flexible data fetching, multiple client types with different data needs, real-time subscriptions, or a type system.

21. **How do you implement caching for QUERY endpoints?**
    
    **Answer:** Use Cache-Control headers (`max-age`, `public`), ETags for conditional requests, multi-level caching (memory → Redis → database), cache keys based on query parameters, and cache invalidation when underlying data changes.

22. **What HTTP status codes are appropriate for QUERY endpoints?**
    
    **Answer:** 200 OK (success), 400 Bad Request (malformed request), 401 Unauthorized (auth required), 403 Forbidden (insufficient permissions), 422 Unprocessable Entity (semantic errors), 500 Internal Server Error (server errors).

23. **How do you document QUERY endpoints?**
    
    **Answer:** Use OpenAPI/Swagger with detailed request/response schemas, example queries, clear descriptions of when to use QUERY vs GET, parameter documentation, error response examples, and authentication requirements.

24. **What is the performance impact of using QUERY vs GET?**
    
    **Answer:** QUERY has ~30ms overhead compared to GET due to request body parsing. However, QUERY enables better query optimization and caching for complex queries, often resulting in better overall performance for complex use cases. Benchmarks show QUERY outperforms POST due to caching.

25. **How do you migrate from POST /search to QUERY?**
    
    **Answer:** Phase 1: Add QUERY endpoint alongside POST. Phase 2: Feature flag gradual rollout (10% → 50% → 100%). Phase 3: Add deprecation headers to POST. Phase 4: Monitor and fix issues. Phase 5: Remove POST endpoint after deprecation period.

---

### Knowledge Check (35 Questions)

**Beginner Level (12 Questions)**

26. **What does HTTP stand for?**
    - A) HyperText Transfer Protocol
    - B) High Transfer Text Protocol
    - C) HyperText Transmission Protocol
    - D) High Transfer Text Process
    
    **Answer: A**

27. **Which HTTP method is used to retrieve resources?**
    - A) POST
    - B) GET
    - C) PUT
    - D) DELETE
    
    **Answer: B**

28. **What is a request body?**
    - A) The URL of the request
    - B) The HTTP headers
    - C) Data sent in the HTTP message body
    - D) The HTTP method
    
    **Answer: C**

29. **What does REST stand for?**
    - A) Representational State Transfer
    - B) Remote State Transfer
    - C) Representational System Transfer
    - D) Remote System Transfer
    
    **Answer: A**

30. **What is an API?**
    - A) Application Programming Interface
    - B) Advanced Programming Integration
    - C) Application Process Integration
    - D) Automated Programming Interface
    
    **Answer: A**

31. **What is URL length?**
    - A) The time to load a URL
    - B) The number of characters in a URL
    - C) The size of the response
    - D) The HTTP method used
    
    **Answer: B**

32. **What is caching?**
    - A) Deleting old data
    - B) Storing data for faster access
    - C) Compressing data
    - D) Encrypting data
    
    **Answer: B**

33. **What is idempotency?**
    - A) Making multiple identical requests has the same effect
    - B) Making requests faster
    - C) Reducing server load
    - D) Increasing security
    
    **Answer: A**

34. **What is a query parameter?**
    - A) Data in the request body
    - B) Data in the URL after the ?
    - C) HTTP headers
    - D) The HTTP method
    
    **Answer: B**

35. **What is JSON?**
    - A) Java Source Object Notation
    - B) JavaScript Object Notation
    - C) JSON is not a format
    - D) Java Standard Object Notation
    
    **Answer: B**

36. **What does HTTP status 200 mean?**
    - A) Not Found
    - B) OK/Success
    - C) Bad Request
    - D) Server Error
    
    **Answer: B**

37. **What is pagination?**
    - A) Deleting old records
    - B) Splitting results into pages
    - C) Sorting data
    - D) Filtering data
    
    **Answer: B**

**Intermediate Level (13 Questions)**

38. **What is the maximum recommended URL length for GET requests?**
    - A) 500 characters
    - B) 2,000 characters
    - C) 10,000 characters
    - D) No limit
    
    **Answer: B** (though browsers vary, 2,000 is safe)

39. **Which HTTP methods are considered safe?**
    - A) GET and POST
    - B) GET and QUERY
    - C) POST and PUT
    - D) All methods
    
    **Answer: B** - Safe methods don't modify server state

40. **What is the purpose of ETag headers?**
    - A) Encrypt the response
    - B) Identify the resource version for caching
    - C) Compress the response
    - D) Authenticate the user
    
    **Answer: B**

41. **What does CORS stand for?**
    - A) Cross-Origin Resource Sharing
    - B) Cross-Object Resource Sharing
    - C) Client-Origin Resource Sharing
    - D) Cross-Origin Response Sharing
    
    **Answer: A**

42. **What is input validation?**
    - A) Checking if input is correct and safe
    - B) Compressing input data
    - C) Encrypting input data
    - D) Logging input data
    
    **Answer: A**

43. **What is SQL injection?**
    - A) A type of database
    - B) A security vulnerability where malicious SQL is inserted
    - C) A query optimization technique
    - D) A database indexing method
    
    **Answer: B**

44. **What is rate limiting?**
    - A) Limiting the size of requests
    - B) Limiting the number of requests over time
    - C) Limiting response time
    - D) Limiting database connections
    
    **Answer: B**

45. **What is a compound index?**
    - A) An index on multiple fields
    - B) A combination of multiple databases
    - C) A type of query
    - D) A caching mechanism
    
    **Answer: A**

46. **What is the purpose of the OPTIONS HTTP method?**
    - A) Get resource options/metadata
    - B) Delete a resource
    - C) Update a resource
    - D) Create a resource
    
    **Answer: A**

47. **What is OpenAPI/Swagger?**
    - A) A database
    - B) An API specification/documentation standard
    - C) A web server
    - D) A programming language
    
    **Answer: B**

48. **What is a DTO (Data Transfer Object)?**
    - A) A database table
    - B) An object used to transfer data between layers
    - C) A type of HTTP request
    - D) A caching mechanism
    
    **Answer: B**

49. **What is the purpose of pagination?**
    - A) To sort data
    - B) To limit large result sets into manageable chunks
    - C) To filter data
    - D) To cache data
    
    **Answer: B**

50. **What is a specification pattern?**
    - A) A design pattern for building dynamic queries
    - B) A type of database index
    - C) A caching strategy
    - D) An authentication method
    
    **Answer: A**

51. **What does ORM stand for?**
    - A) Object-Relational Mapping
    - B) Object-Request Mapping
    - C) Operational-Relational Model
    - D) Object-Resource Manager
    
    **Answer: A**

52. **What is connection pooling?**
    - A) Combining multiple databases
    - B) Reusing database connections for efficiency
    - C) A type of query optimization
    - D) A security measure
    
    **Answer: B**

53. **What is the purpose of HTTP headers?**
    - A) To send data in the request body
    - B) To provide metadata about the request/response
    - C) To define the HTTP method
    - D) To specify the URL
    
    **Answer: B**

**Advanced Level (10 Questions)**

54. **Explain the IETF draft process for new HTTP methods.**
    
    **Answer:** The IETF (Internet Engineering Task Force) follows a standardization process: Internet-Draft → Working Group Review → IESG Review → RFC Publication. For QUERY, it's currently in draft status, meaning it's being reviewed but not yet an official standard. This affects adoption as frameworks wait for standardization.

55. **What are the semantic guarantees of QUERY according to HTTP semantics?**
    
    **Answer:** QUERY provides: Safe (no server state modification), Idempotent (same request = same result), Cacheable (responses can be cached). These are defined in RFC 7231 and ensure QUERY behaves predictably in the HTTP ecosystem.

56. **How does QUERY interact with HTTP caching mechanisms?**
    
    **Answer:** QUERY responses can include Cache-Control, ETag, and Last-Modified headers. Proxies and CDNs can cache QUERY responses (unlike POST). Conditional requests (If-None-Match, If-Modified-Since) work with QUERY, enabling efficient cache validation.

57. **What is the difference between QUERY and SEARCH methods?**
    
    **Answer:** Both are proposed methods for search operations. QUERY is more general-purpose for any query operation. SEARCH (WebDAV) is specifically for searching within resource collections. QUERY is gaining more traction in the REST API community.

58. **How do you ensure QUERY operations remain read-only at the database level?**
    
    **Answer:** Use read-only transactions (`@Transactional(readOnly = true)` in Spring), route QUERY to read replicas, use database roles with read-only permissions, avoid triggers that modify data, and use database proxies that enforce read-only mode.

59. **What is cursor-based pagination and when should you use it?**
    
    **Answer:** Cursor-based pagination uses opaque cursors (usually encoded IDs) instead of page numbers. Use it for: real-time data (new items between requests), large datasets (>100k items), infinite scroll UIs, and when consistent pagination is critical. It's more complex but handles data changes better than offset pagination.

60. **Explain the trade-offs between offset and cursor pagination.**
    
    **Answer:** Offset pagination (page/limit) is simple, supports jumping to pages, but has performance issues with large offsets and inconsistent results with data changes. Cursor pagination is consistent, performant for large datasets, but doesn't support jumping to pages and is more complex to implement.

61. **What is the N+1 problem and how does it relate to QUERY endpoints?**
    
    **Answer:** The N+1 problem occurs when you execute N+1 queries instead of 1 query (1 for the list, N for each item's details). In QUERY endpoints returning lists, eager loading (JOIN FETCH, includes) prevents this. Use batch loading or DataLoader patterns.

62. **How do you implement field-level security in QUERY responses?**
    
    **Answer:** Define sensitive fields, check user permissions, filter fields in the query (don't select sensitive columns), sanitize output before returning, use separate DTOs for different permission levels, and log access to sensitive data for auditing.

63. **What monitoring metrics are important for QUERY endpoints?**
    
    **Answer:** Request rate, error rate, latency (p50, p95, p99), cache hit rate, database query time, query complexity distribution, concurrent queries, timeout rate, and user-agent distribution. Use APM tools and custom metrics.

64. **Explain how you would implement query result caching with cache invalidation.**
    
    **Answer:** Use multi-level caching (memory → Redis). Generate cache keys from query parameters. Set appropriate TTLs. Invalidate on: data updates (explicit invalidation), TTL expiration, cache versioning. Use write-through or write-behind strategies. Monitor cache hit rates.

---

## Summary & Key Takeaways

### 🎯 Core Concepts Recap

1. **The Problem:** GET is limited by URL length and structure; POST is semantically incorrect for queries
2. **The Solution:** QUERY method provides semantically correct, flexible querying with request bodies
3. **Key Benefits:** Clear semantics, better caching, maintainable code, proper HTTP compliance

### 📊 Decision Matrix

| Scenario | Use GET | Use QUERY | Use POST |
|----------|---------|-----------|----------|
| Simple key-value params | ✅ | ❌ | ❌ |
| Complex nested structures | ❌ | ✅ | ❌ |
| URL < 2000 chars | ✅ | ❌ | ❌ |
| Creating resources | ❌ | ❌ | ✅ |
| Read-only with body | ❌ | ✅ | ❌ |
| State modification | ❌ | ❌ | ✅ |

### 💡 Key Insights

1. **Semantics Matter:** Using the right HTTP method makes APIs self-documenting
2. **Caching is Crucial:** QUERY enables better caching than POST, improving performance
3. **Adoption is Gradual:** Framework support is emerging; use method override for now
4. **Not a Replacement:** QUERY complements GET, doesn't replace it
5. **Future-Proof:** Aligns with modern API design trends

### ✅ Best Practices Summary

- ✅ Use QUERY for complex read-only queries
- ✅ Implement comprehensive input validation
- ✅ Add proper cache headers
- ✅ Set query complexity limits
- ✅ Document thoroughly
- ✅ Monitor performance
- ✅ Handle errors gracefully
- ✅ Version your API

### ❌ Common Mistakes to Avoid

- ❌ Don't use QUERY for state changes
- ❌ Don't skip input validation
- ❌ Don't ignore caching opportunities
- ❌ Don't allow unbounded queries
- ❌ Don't forget error handling
- ❌ Don't over-engineer simple queries

### 🚀 Next Steps

1. **Start Small:** Implement QUERY in a non-critical endpoint
2. **Test Thoroughly:** Unit tests, integration tests, performance tests
3. **Monitor Closely:** Track metrics, errors, cache hit rates
4. **Gradual Rollout:** Feature flags, canary deployments
5. **Gather Feedback:** From API consumers, monitor usage patterns
6. **Iterate:** Improve based on real-world usage

### 📈 The Future

The QUERY method represents HTTP evolution toward better semantic clarity. As adoption grows:
- More frameworks will support it natively
- Tooling will improve
- Best practices will solidify
- It may become an official RFC

**Bottom Line:** QUERY fills a long-standing gap in HTTP, providing the right tool for complex query operations. While adoption is gradual, it's a step toward more expressive, maintainable APIs.

---

## Further Reading & Resources

### Official Documentation

📄 **HTTP Specifications:**
- [IETF QUERY Method Draft](https://datatracker.ietf.org/doc/draft-ietf-httpapi-safe-method-w-body/)
- [RFC 7231 - HTTP/1.1 Semantics](https://tools.ietf.org/html/rfc7231)
- [RFC 5789 - PATCH Method](https://tools.ietf.org/html/rfc5789)

📚 **API Design:**
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)
- [Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/)

### Framework Documentation

🔧 **Spring Boot:**
- [Spring Web MVC Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)
- [Spring Data JPA Specifications](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#specifications)

🔧 **Express.js:**
- [Express.js Documentation](https://expressjs.com/)
- [HTTP Method Override Middleware](https://github.com/expressjs/method-override)

🔧 **FastAPI:**
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pydantic Validation](https://docs.pydantic.dev/)

### Articles & Tutorials

📝 **HTTP Methods:**
- [MDN Web Docs - HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [REST API Tutorial - HTTP Methods](https://restfulapi.net/http-methods/)
- [When to Use GET vs POST](https://www.smashingmagazine.com/2018/01/understanding-using-rest-api/)

📝 **API Design:**
- [The Problem with POST for Search](https://www.thoughtworks.com/insights/blog/api-design/problem-post-search)
- [REST API Design Best Practices](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [HTTP Caching Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)

### Community Resources

💬 **Discussions:**
- [HTTP QUERY Method GitHub Discussions](https://github.com/for-GET/http-question)
- [Stack Overflow - HTTP Methods](https://stackoverflow.com/questions/tagged/http-methods)
- [Reddit r/webdev - API Design](https://www.reddit.com/r/webdev/)

🛠️ **Tools:**
- [Postman](https://www.postman.com/) - API testing
- [Insomnia](https://insomnia.rest/) - API client
- [OpenAPI Generator](https://openapi-generator.tech/) - Generate API code

### Books

📚 **Recommended Reading:**
- "REST API Design Rulebook" by Mark Masse
- "Designing Web APIs" by Brenda Jin, Saurabh Sahni, Amir Shevat
- "HTTP: The Definitive Guide" by David Gourley, Brian Totty
- "REST in Practice" by Jim Webber, Savas Parastatidis, Ian Robinson

### Related Topics

🔗 **Explore Further:**
- GraphQL vs REST
- gRPC and Protocol Buffers
- OData (Open Data Protocol)
- JSON:API Specification
- Hypermedia APIs (HATEOAS)
- API Versioning Strategies
- API Gateway Patterns
- Rate Limiting Algorithms

---

## Appendix

### A. Complete Request/Response Examples

#### Example 1: Simple Product Search

```http
QUERY /api/v1/products/search HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bear eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
Accept-Encoding: gzip, deflate
Connection: keep-alive

{
  "query": {
    "categories": ["laptops"],
    "price": {
      "min": 1000,
      "max": 2500
    }
  }
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=300, public
ETag: "a1b2c3d4e5f6"
Content-Length: 1234

{
  "success": true,
  "data": [
    {
      "id": "prod_123",
      "name": "MacBook Pro 14",
      "category": "laptops",
      "brand": "Apple",
      "price": 1999.99,
      "inStock": true
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 15,
    "pages": 1
  },
  "metadata": {
    "queryTime": "45ms",
    "cached": false
  }
}
```

#### Example 2: Complex Analytics Query

```http
QUERY /api/v1/analytics/query HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bear eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "query": {
    "timeRange": {
      "start": "2025-01-01T00:00:00Z",
      "end": "2025-01-31T23:59:59Z",
      "granularity": "day"
    },
    "metrics": [
      {
        "name": "revenue",
        "aggregation": "sum",
        "field": "amount"
      },
      {
        "name": "orders",
        "aggregation": "count"
      }
    ],
    "dimensions": ["country", "device"],
    "filters": {
      "country": ["US", "UK", "DE"]
    }
  }
}
```

### B. OpenAPI 3.0 Specification

```yaml
openapi: 3.0.3
info:
  title: Product Search API
  version: 2.0.0
  description: |
    API for complex product search using the HTTP QUERY method.
    
    ## Overview
    This API provides advanced search capabilities using structured queries.
    
servers:
  - url: https://api.example.com/v2
    description: Production server

paths:
  /products/search:
    post:
      summary: Search products with complex filters
      description: |
        Performs a complex product search using structured query parameters.
        
        ## When to Use
        Use this endpoint when you need:
        - Complex nested filtering
        - Large payloads that don't fit in URL
        - Structured query parameters
        
        ## When to Use GET Instead
        Use GET /products for simple filters that fit in URL.
      operationId: searchProducts
      tags:
        - Products
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProductSearchQuery'
            examples:
              simple:
                summary: Simple category search
                value:
                  categories: ["laptops"]
              complex:
                summary: Complex multi-filter search
                value:
                  categories: ["laptops", "gaming"]
                  brands: ["apple", "dell"]
                  price:
                    min: 1000
                    max: 2500
                  rating: 4
                  inStock: true
                  sort:
                    field: price
                    order: asc
                  pagination:
                    page: 1
                    limit: 20
      responses:
        '200':
          description: Search successful
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SearchResponse'
          headers:
            Cache-Control:
              schema:
                type: string
                example: max-age=300, public
            ETag:
              schema:
                type: string
                example: "abc123xyz"
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '422':
          $ref: '#/components/responses/ValidationError'
        '500':
          $ref: '#/components/responses/InternalError'
      security:
        - BearerAuth: []

components:
  schemas:
    ProductSearchQuery:
      type: object
      required:
        - query
      properties:
        query:
          $ref: '#/components/schemas/SearchQuery'
    
    SearchQuery:
      type: object
      properties:
        categories:
          type: array
          items:
            type: string
          maxItems: 10
        brands:
          type: array
          items:
            type: string
          maxItems: 5
        price:
          $ref: '#/components/schemas/PriceRange'
        rating:
          type: integer
          minimum: 1
          maximum: 5
        inStock:
          type: boolean
        sort:
          $ref: '#/components/schemas/SortConfig'
        pagination:
          $ref: '#/components/schemas/PaginationConfig'
    
    PriceRange:
      type: object
      properties:
        min:
          type: number
          format: float
          minimum: 0
        max:
          type: number
          format: float
          minimum: 0
        currency:
          type: string
          enum: [USD, EUR, GBP]
          default: USD
    
    SortConfig:
      type: object
      properties:
        field:
          type: string
          enum: [price, rating, name, date]
          default: price
        order:
          type: string
          enum: [asc, desc]
          default: asc
    
    PaginationConfig:
      type: object
      properties:
        page:
          type: integer
          minimum: 1
          default: 1
        limit:
          type: integer
          minimum: 1
          maximum: 100
          default: 20
    
    SearchResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: array
          items:
            $ref: '#/components/schemas/Product'
        pagination:
          $ref: '#/components/schemas/Pagination'
        metadata:
          $ref: '#/components/schemas/ResponseMetadata'
    
    Product:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        category:
          type: string
        brand:
          type: string
        price:
          type: number
          format: float
        inStock:
          type: boolean
    
    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        pages:
          type: integer
    
    ResponseMetadata:
      type: object
      properties:
        queryTime:
          type: string
          example: "45ms"
        cached:
          type: boolean
    
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  
  responses:
    BadRequest:
      description: Invalid request
      content:
        application/json:
          schema:
            type: object
            properties:
              error:
                type: string
              message:
                type: string
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            type: object
            properties:
              error:
                type: string
                example: "Unauthorized"
    ValidationError:
      description: Validation failed
      content:
        application/json:
          schema:
            type: object
            properties:
              error:
                type: string
                example: "Validation failed"
              details:
                type: array
                items:
                  type: object
                  properties:
                    field:
                      type: string
                    message:
                      type: string
    InternalError:
      description: Internal server error
      content:
        application/json:
          schema:
            type: object
            properties:
              error:
                type: string
              requestId:
                type: string
```

---

## Conclusion

The HTTP QUERY method represents a significant evolution in web API design, addressing a decades-old gap in HTTP semantics. While adoption is still emerging, it provides the right tool for complex query operations that don't fit comfortably in GET parameters and don't semantically belong in POST.

**Key Takeaways:**

1. **Use the Right Tool:** Choose GET, QUERY, or POST based on the operation's semantics and complexity
2. **Semantics Matter:** Clear method choice makes APIs self-documenting
3. **Performance Counts:** Proper caching and optimization make QUERY competitive
4. **Security is Critical:** Validate, limit, and monitor all QUERY operations
5. **Adoption is Gradual:** Start with non-critical endpoints, monitor, and iterate

The future of API design is moving toward more expressive, semantically correct interfaces. The QUERY method is a step in that direction, providing developers with better tools to build clear, maintainable, and performant APIs.

**Remember:** The best API is one that clearly communicates intent, and the QUERY method helps achieve exactly that.

---

**📧 Feedback:** If you have questions or feedback about this tutorial, please reach out.  
**🔄 Updates:** This tutorial is updated regularly to reflect the latest developments in HTTP standards and best practices.  
**📄 License:** This tutorial is provided as-is for educational purposes.

---

*Last Updated: January 2026*  
*Tutorial Version: 1.0*  
*Estimated Reading Time: 25-30 minutes*  
*Difficulty: Intermediate*
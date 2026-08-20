# Jakarta NoSQL 1.1: Advancing Polyglot Persistence for Jakarta EE 12

**Last Updated:** January 2026  
**Difficulty Level:** Intermediate  
**Estimated Reading Time:** 45-60 minutes  
**Version:** Jakarta NoSQL 1.1 (Jakarta EE 12)

---

## Table of Contents

1. [Introduction & Overview](#introduction--overview)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [The Polyglot Persistence Landscape](#the-polyglot-persistence-landscape)
5. [Jakarta NoSQL Architecture](#jakarta-nosql-architecture)
6. [Entity Mapping Deep Dive](#entity-mapping-deep-dive)
7. [Template API Operations](#template-api-operations)
8. [Jakarta NoSQL 1.1 New Features](#jakarta-nosql-11-new-features)
9. [Real-World Use Cases](#real-world-use-cases)
10. [Implementation Approaches](#implementation-approaches)
11. [Best Practices](#best-practices)
12. [Anti-Patterns](#anti-patterns)
13. [Performance Considerations](#performance-considerations)
14. [Security Considerations](#security-considerations)
15. [Testing Strategies](#testing-strategies)
16. [Common Pitfalls & Troubleshooting](#common-pitfalls--troubleshooting)
17. [Practice Exercises](#practice-exercises)
18. [Test Your Understanding](#test-your-understanding)
19. [Common Interview Questions](#common-interview-questions)
20. [Question Bank](#question-bank)
21. [Summary & Key Takeaways](#summary--key-takeaways)
22. [Further Reading & Resources](#further-reading--resources)
23. [Migration Guide: JPA to Jakarta NoSQL](#migration-guide-jpa-to-jakarta-nosql)

---

## Introduction & Overview

Modern applications rarely rely on a single data model. Relational databases remain essential for transactional consistency and structured business data. However, document, key-value, column-oriented, graph, and vector databases are now critical for workloads that require flexible schemas, horizontal scalability, low-latency access, or specialized queries. As a result, **polyglot persistence** — selecting the most appropriate database model for each use case — has become a standard architectural strategy rather than an exception.

The rise of artificial intelligence further supports this trend. Retrieval-augmented generation (RAG), semantic search, recommendation systems, and autonomous agents often rely on embeddings and vector similarity searches to access contextual information. As a result, vector databases and multimodel NoSQL platforms are becoming integral to the modern enterprise data landscape.

In this context, **Jakarta NoSQL** offers Jakarta EE developers a standardized and extensible programming model for working with various NoSQL technologies, while minimizing direct dependence on specific database vendors.

### What Makes Jakarta NoSQL 1.1 Special?

Jakarta NoSQL 1.1 represents a significant milestone as the **first specification developed within the Jakarta EE ecosystem** (rather than inherited from Java EE). It addresses the critical need for enterprise applications to embrace NoSQL databases while supporting polyglot persistence strategies. Its goal is to offer a simple, vendor-neutral programming model for document, key-value, column, and graph databases, so developers don't need to learn a separate API for each provider.

> 💡 **Key Insight:** Jakarta NoSQL 1.1 builds upon the foundation of version 1.0 and introduces integration with Jakarta Query, projections, fluent update operations, and automatic converters — making it more expressive and better aligned with the broader Jakarta EE data ecosystem.

---

## Prerequisites

### Required Knowledge
- ✅ **Java 17+** (records, sealed classes, text blocks)
- ✅ **Jakarta EE 9+** fundamentals
- ✅ **Basic understanding of NoSQL databases** (document, key-value, column, graph)
- ✅ **Familiarity with Jakarta Persistence (JPA)** concepts
- ✅ **Maven or Gradle** build tools
- ✅ **Basic understanding of polyglot persistence** concepts

### Required Tools
- ✅ **JDK 17 or higher**
- ✅ **Maven 3.8+** or **Gradle 7.5+**
- ✅ **IDE** (IntelliJ IDEA, Eclipse, or VS Code with Java extensions)
- ✅ **NoSQL Database** (MongoDB, Oracle NoSQL, ArangoDB, or similar)
- ✅ **Docker** (optional, for running databases locally)

### Recommended (But Not Required)
- 📚 Experience with JPA/Hibernate
- 📚 Understanding of microservices architecture
- 📚 Basic knowledge of vector databases and embeddings
- 📚 Familiarity with reactive programming concepts

---

## Learning Objectives

By the end of this tutorial, you will be able to:

- ✅ Understand the concept of polyglot persistence and when to use it
- ✅ Explain Jakarta NoSQL's role in the Jakarta EE ecosystem
- ✅ Map entities using Jakarta NoSQL annotations (`@Entity`, `@Id`, `@Column`)
- ✅ Use Java records for immutable entity definitions
- ✅ Perform CRUD operations using the Template API
- ✅ Write queries using both fluent API and Jakarta Query Language
- ✅ Implement projections for efficient data retrieval
- ✅ Use fluent update operations introduced in version 1.1
- ✅ Apply custom type converters with auto-apply functionality
- ✅ Choose the right NoSQL database for specific use cases
- ✅ Avoid common pitfalls and anti-patterns
- ✅ Test Jakarta NoSQL applications effectively
- ✅ Migrate existing JPA code to Jakarta NoSQL

---

## The Polyglot Persistence Landscape

### Why Multiple Database Models?

Different data models excel at different tasks. Understanding when to use each is crucial for architecting modern applications.

```mermaid
graph TB
    A[Application Requirements] --> B{Data Characteristics?}
    B -->|Structured, ACID| C[Relational DB<br/>PostgreSQL, MySQL]
    B -->|Flexible Schema| D[Document DB<br/>MongoDB, Couchbase]
    B -->|Key-Value Pairs| E[Key-Value DB<br/>Redis, Oracle NoSQL]
    B -->|Wide Columnar| F[Column DB<br/>Cassandra, HBase]
    B -->|Connected Data| G[Graph DB<br/>Neo4j, ArangoDB]
    B -->|Vector Search| H[Vector DB<br/>Pinecone, Milvus]
    
    C --> I[Polyglot Persistence]
    D --> I
    E --> I
    F --> I
    G --> I
    H --> I
    
    I --> J[Optimal Performance<br/>& Scalability]
```

### NoSQL Database Types Comparison

| Database Type | Best For | Examples | Strengths | Weaknesses |
|---------------|----------|----------|-----------|------------|
| **Document** | Content management, catalogs, user profiles | MongoDB, Couchbase | Flexible schema, rich queries | Limited transactions |
| **Key-Value** | Caching, sessions, real-time data | Redis, Oracle NoSQL | Ultra-fast, simple API | Limited query capabilities |
| **Column** | Time-series, analytics, logging | Cassandra, HBase | Horizontal scalability, fast writes | Complex data modeling |
| **Graph** | Social networks, recommendations | Neo4j, ArangoDB | Relationship traversal | Steep learning curve |
| **Vector** | AI/ML, semantic search, RAG | Pinecone, Milvus | Similarity search | Specialized use cases |

> ⚠️ **Important:** Choosing the right database is not about following trends — it's about matching the database's strengths to your application's specific requirements.

### The Rise of Vector Databases

With the explosion of AI applications, vector databases have become essential for:
- **Retrieval-Augmented Generation (RAG)** - Providing context to LLMs
- **Semantic Search** - Finding similar content by meaning, not just keywords
- **Recommendation Systems** - Suggesting items based on user preferences
- **Image/Video Search** - Finding similar media content
- **Anomaly Detection** - Identifying outliers in data

Jakarta NoSQL 1.1 positions itself to support these emerging patterns through its flexible, vendor-neutral approach.

---

## Jakarta NoSQL Architecture

### Where Jakarta NoSQL Fits in Jakarta EE

```mermaid
graph TB
    subgraph "Jakarta EE Ecosystem"
        A[Jakarta NoSQL] --> B[Document Databases]
        A --> C[Key-Value Databases]
        A --> D[Column Databases]
        A --> E[Graph Databases]
        
        F[Jakarta Persistence] --> G[Relational Databases]
        H[Jakarta Data] --> I[Repository Pattern]
        J[Jakarta Query] --> K[Unified Query Language]
        
        A -.-> J
        H -.-> A
        H -.-> F
    end
    
    subgraph "Application Layer"
        L[Enterprise Application] --> A
        L --> F
        L --> H
    end
    
    subgraph "Database Layer"
        B --> M[MongoDB]
        C --> N[Oracle NoSQL]
        D --> O[Cassandra]
        E --> P[Neo4j]
        G --> Q[PostgreSQL]
    end
```

### Core Components

**1. Entity Mapping Layer**
- Annotations: `@Entity`, `@Id`, `@Column`
- Similar to JPA but NoSQL-specific
- Explicit attribute marking (unlike JPA's implicit mapping)

**2. Template API**
- Similar to Spring's `MongoTemplate` or `JdbcTemplate`
- Provides fluent interface for CRUD operations
- Database-agnostic operations

**3. Query Engine**
- Fluent query API
- Jakarta Query Language support (version 1.1+)
- Type-safe queries with projections

**4. Type Conversion System**
- Custom converters with `@Converter`
- Auto-apply functionality (version 1.1+)

### How Jakarta NoSQL Differs from JPA

| Aspect | Jakarta NoSQL | Jakarta Persistence (JPA) |
|--------|---------------|---------------------------|
| **Database Type** | NoSQL (document, key-value, column, graph) | Relational (SQL) |
| **Schema** | Flexible/Schema-less | Fixed schema |
| **Attribute Mapping** | Explicit (`@Id` or `@Column` required) | Implicit (all fields mapped by default) |
| **Query Language** | Fluent API + Jakarta Query | JPQL/HQL + Criteria API |
| **Transactions** | Limited/vendor-specific | Full ACID support |
| **Joins** | Not supported | Supported |
| **Primary Goal** | Polyglot persistence | Relational data access |

---

## Entity Mapping Deep Dive

Entity mapping is the foundation of Jakarta NoSQL. Its annotations use terminology familiar from Jakarta Persistence, lowering the learning curve for Java developers.

### Basic Entity Definition

Let's start with a practical example — modeling an investment:

```java
package expert.os.videos.nosql;

import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;

import java.math.BigDecimal;
import java.util.UUID;

@Entity
public class Investment {

    @Id
    private UUID id;

    @Column
    private String name;

    @Column
    private InvestmentType type;

    @Column
    private BigDecimal amount;

    // Constructor with all fields
    public Investment(UUID id, String name, InvestmentType type, BigDecimal amount) {
        this.id = id;
        this.name = name;
        this.type = type;
        this.amount = amount;
    }

    // Default constructor (required by Jakarta NoSQL)
    Investment() {
    }

    @Override
    public String toString() {
        return "Investment{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", type=" + type +
                ", amount=" + amount +
                '}';
    }
}

// Enum for investment types
public enum InvestmentType {
    STOCK,
    BOND,
    FUND,
    CRYPTO,
    REAL_ESTATE
}
```

> ✅ **Key Point:** Notice that every persistent field MUST be annotated with either `@Id` or `@Column`. Fields without these annotations are **ignored** by Jakarta NoSQL, making the persistence model explicit and preventing accidental storage.

### Using Java Records (Recommended)

Jakarta NoSQL supports Java records, enabling more concise and immutable entity definitions:

```java
import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;
import java.math.BigDecimal;
import java.util.UUID;

@Entity
public record Investment(
        @Id UUID id,
        @Column String name,
        @Column InvestmentType type,
        @Column BigDecimal amount) {
}
```

**Benefits of Using Records:**
- ✅ **Immutability** - Thread-safe by default
- ✅ **Conciseness** - Less boilerplate code
- ✅ **Built-in methods** - `equals()`, `hashCode()`, `toString()` auto-generated
- ✅ **Clear intent** - Data carrier pattern explicit

> 💡 **Pro Tip:** Use records for simple entities and traditional classes when you need mutability or complex business logic.

### Advanced Entity Mapping

#### Embedding Related Data

```java
import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;
import java.time.LocalDateTime;

@Entity
public class Investment {

    @Id
    private UUID id;

    @Column
    private String name;

    @Column
    private InvestmentType type;

    @Column
    private BigDecimal amount;

    @Column
    private LocalDateTime createdAt;

    @Column
    private LocalDateTime updatedAt;

    // Embedded object example
    @Column
    private InvestmentDetails details;

    // Constructors, getters, etc.
}

// Embedded entity (no @Entity annotation)
public class InvestmentDetails {
    private String riskLevel;
    private String sector;
    private String manager;
    
    // Constructors, getters, setters
}
```

#### Collections and Arrays

```java
import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;
import java.util.List;
import java.util.Set;

@Entity
public class Portfolio {

    @Id
    private UUID id;

    @Column
    private String ownerName;

    @Column
    private List<Investment> investments;  // List of investments

    @Column
    private Set<String> tags;  // Set of tags

    @Column
    private String[] categories;  // Array of categories
}
```

> ⚠️ **Warning:** Not all NoSQL databases support complex types. Check your provider's documentation for supported types.

### Entity Mapping Best Practices

```mermaid
graph LR
    A[Entity Design] --> B{Use Records?}
    B -->|Yes| C[Immutable, Concise]
    B -->|No| D[Traditional Class]
    
    C --> E[Simple Entities]
    D --> F[Complex Business Logic]
    
    E --> G["✅ Recommended"]
    F --> H["When Needed"]
    
    G --> I["Always annotate\nall fields"]
    H --> I
    
    I --> J["@Id for keys"]
    I --> K["@Column for attributes"]
```

---

## Template API Operations

The Template API provides a fluent, intuitive interface for database operations. Think of it as your primary interface for interacting with NoSQL databases.

### Basic CRUD Operations

#### 1. Create (Insert)

```java
import jakarta.nosql.template.Template;
import java.math.BigDecimal;
import java.util.UUID;

// Initialize template (typically via dependency injection)
Template template = ...;

// Create a new investment
UUID id = UUID.randomUUID();
Investment investment = new Investment(
        id,
        "Java Growth Fund",
        InvestmentType.FUND,
        new BigDecimal("1500.00")
);

// Insert into database
template.insert(investment);

System.out.println("Investment created with ID: " + id);
```

**Error Handling Example:**

```java
try {
    template.insert(investment);
    System.out.println("Success: Investment created");
} catch (ConstraintViolationException e) {
    System.err.println("Error: Duplicate ID - " + e.getMessage());
} catch (PersistenceException e) {
    System.err.println("Error: Database error - " + e.getMessage());
}
```

#### 2. Read (Find)

```java
// Find by ID
template.find(Investment.class, id)
        .ifPresentOrElse(
            investment -> System.out.println("Found: " + investment),
            () -> System.out.println("Investment not found")
        );

// With error handling
try {
    Optional<Investment> result = template.find(Investment.class, id);
    if (result.isPresent()) {
        Investment investment = result.get();
        // Process investment
    } else {
        // Handle not found case
    }
} catch (NoSQLException e) {
    System.err.println("Database error: " + e.getMessage());
}
```

#### 3. Update

```java
// Retrieve existing investment
Investment existing = template.find(Investment.class, id)
        .orElseThrow(() -> new EntityNotFoundException("Investment not found"));

// Modify (if using mutable entity)
// existing.setAmount(new BigDecimal("2000.00"));
// template.update(existing);

// Or use fluent update (Jakarta NoSQL 1.1+)
template.update(Investment.class)
        .set("amount")
        .to(new BigDecimal("2000.00"))
        .where("id")
        .eq(id)
        .execute();
```

#### 4. Delete

```java
// Delete by entity
template.delete(investment);

// Or delete by criteria
template.delete(Investment.class)
        .where("amount")
        .lt(new BigDecimal("100.00"))
        .execute();
```

### Advanced Query Operations

#### Fluent Query API

```java
// Select with condition
template.select(Investment.class)
        .where("amount")
        .gt(new BigDecimal("1000"))
        .result()
        .forEach(System.out::println);

// Multiple conditions
template.select(Investment.class)
        .where("type")
        .eq(InvestmentType.FUND)
        .and("amount")
        .gte(new BigDecimal("5000"))
        .result()
        .forEach(System.out::println);

// Ordering and pagination
template.select(Investment.class)
        .where("type")
        .eq(InvestmentType.STOCK)
        .orderBy("amount")
        .desc()
        .limit(10)
        .offset(20)
        .result()
        .forEach(System.out::println);
```

#### Jakarta Query Language (Version 1.1+)

```java
// String-based query
template.query("FROM Investment WHERE amount > 1000")
        .result()
        .forEach(System.out::println);

// With named parameters
template.query("FROM Investment WHERE amount > :amount AND type = :type")
        .bind("amount", new BigDecimal("1000"))
        .bind("type", InvestmentType.FUND)
        .result()
        .forEach(System.out::println);

// Complex queries
template.query("""
        FROM Investment 
        WHERE amount BETWEEN :min AND :max 
        AND type IN :types
        ORDER BY amount DESC
        """)
        .bind("min", new BigDecimal("1000"))
        .bind("max", new BigDecimal("10000"))
        .bind("types", List.of(InvestmentType.FUND, InvestmentType.STOCK))
        .result()
        .forEach(System.out::println);
```

---

## Jakarta NoSQL 1.1 New Features

Version 1.1 brings significant enhancements that make Jakarta NoSQL more powerful and integrated with the Jakarta EE ecosystem.

### 1. Jakarta Query Integration

Jakarta NoSQL 1.1 integrates with **Jakarta Query**, providing a unified query model across persistence technologies.

**What is Jakarta Query?**

Jakarta Query provides a unified query model for Java applications and diverse data sources. Its core language defines essential query concepts such as:
- Entities and attributes
- Comparisons and filtering
- Parameters and binding
- Ordering and pagination

It also offers the Jakarta Persistence Query Language (JPQL), enabling familiar syntax to be used across different persistence technologies.

**Example Usage:**

```java
// Simple query
List<Investment> results = template.query(
        "FROM Investment WHERE amount > 1000")
        .result()
        .stream()
        .toList();

// Parameterized query (prevents injection)
List<Investment> expensiveInvestments = template.query(
        "FROM Investment WHERE amount > :amount")
        .bind("amount", new BigDecimal("1000"))
        .result()
        .stream()
        .toList();

// Complex query with multiple parameters
List<Investment> funds = template.query("""
        FROM Investment 
        WHERE type = :type 
        AND amount >= :minAmount
        AND createdAt >= :date
        """)
        .bind("type", InvestmentType.FUND)
        .bind("minAmount", new BigDecimal("5000"))
        .bind("date", LocalDateTime.now().minusMonths(6))
        .result()
        .stream()
        .toList();
```

> ✅ **Security Benefit:** Using named parameters (`:paramName`) prevents NoSQL injection attacks by separating query structure from data values.

### 2. Projections

Projections enable queries to return only the information needed for a specific use case, rather than loading the entire entity. This improves performance and reduces memory usage.

**Defining a Projection:**

```java
import jakarta.nosql.Projection;

// Projection as a Java record
@Projection
public record InvestmentProjector(
        String name,
        BigDecimal amount) {
}

// Or as a traditional class
@Projection
public class InvestmentSummary {
    private String name;
    private BigDecimal amount;
    
    // Constructors, getters, setters
}
```

**Using Projections:**

```java
// Typed query with projection
List<InvestmentProjector> summaries = template.typedQuery(
        "FROM Investment WHERE amount > 1000",
        InvestmentProjector.class)
        .result()
        .stream()
        .toList();

// Display results
summaries.forEach(projector -> 
    System.out.println(projector.name() + ": " + projector.amount())
);
```

**When to Use Projections:**

| Scenario | Use Projection? | Benefit |
|----------|----------------|---------|
| API responses | ✅ Yes | Reduce payload size |
| Reports/Dashboards | ✅ Yes | Faster queries, less memory |
| Dropdown lists | ✅ Yes | Only need name/id |
| Full entity display | ❌ No | Need all fields |
| Entity modification | ❌ No | Need complete entity |

> 💡 **Performance Tip:** Projections can significantly improve query performance, especially for large entities with many fields. Use them whenever you don't need the complete entity.

### 3. Fluent Update Operations

Version 1.1 adds fluent update operations, completing the main set of data manipulation capabilities.

**Basic Update:**

```java
// Update a single field
template.update(Investment.class)
        .set("amount")
        .to(new BigDecimal("2000.00"))
        .where("id")
        .eq(id)
        .execute();
```

**Multiple Field Updates:**

```java
template.update(Investment.class)
        .set("amount")
        .to(new BigDecimal("2500.00"))
        .set("updatedAt")
        .to(LocalDateTime.now())
        .where("id")
        .eq(id)
        .execute();
```

**Conditional Updates:**

```java
// Only update if current amount is below threshold
long updatedCount = template.update(Investment.class)
        .set("amount")
        .to(new BigDecimal("2000.00"))
        .set("status")
        .to("UPGRADED")
        .where("amount")
        .lt(new BigDecimal("1500.00"))
        .and("type")
        .eq(InvestmentType.FUND)
        .execute();

System.out.println("Updated " + updatedCount + " investments");
```

**Increment Operations:**

```java
// Increment amount (if supported by database)
template.update(Investment.class)
        .set("amount")
        .inc(new BigDecimal("100.00"))
        .where("id")
        .eq(id)
        .execute();
```

> ✅ **Advantage:** Fluent updates execute directly on the database, eliminating the need to retrieve and modify entities in memory. This reduces network round-trips and improves performance.

### 4. Auto-Apply Converters

The `@Converter` annotation now supports an `autoApply` attribute, automatically applying converters to all mapped attributes of the supported Java type.

**Defining a Custom Converter:**

```java
import jakarta.nosql.Converter;
import jakarta.nosql.AttributeConverter;
import java.math.BigDecimal;
import java.util.Currency;

// Converter for Currency objects
@Converter(autoApply = true)  // Auto-applied to all Currency fields
public class CurrencyConverter implements AttributeConverter<Currency, String> {
    
    @Override
    public String convertToDatabaseColumn(Currency attribute) {
        return attribute != null ? attribute.getCurrencyCode() : null;
    }
    
    @Override
    public Currency convertToEntityAttribute(String dbData) {
        return dbData != null ? Currency.getInstance(dbData) : null;
    }
}

// Now use Currency in entities without explicit @Convert annotation
@Entity
public class Investment {
    @Id
    private UUID id;
    
    @Column
    private String name;
    
    @Column
    private Currency currency;  // Automatically converted!
    
    @Column
    private BigDecimal amount;
}
```

**Without Auto-Apply (Manual):**

```java
@Entity
public class Investment {
    @Id
    private UUID id;
    
    @Column
    @Convert(converter = CurrencyConverter.class)  // Must specify on each field
    private Currency currency;
}
```

> 💡 **Pro Tip:** Use `autoApply = true` for converters that should apply globally (e.g., `Currency`, `LocalDate`, custom types). Use manual conversion when you need different converters for the same type in different contexts.

---

## Real-World Use Cases

### Use Case 1: Investment Portfolio Management

**Scenario:** A fintech company needs to manage investment portfolios with real-time updates and complex queries.

```java
@Entity
public class Portfolio {
    @Id
    private UUID id;
    
    @Column
    private String userId;
    
    @Column
    private List<Investment> investments;
    
    @Column
    private BigDecimal totalValue;
    
    @Column
    private RiskProfile riskProfile;
}

// Service implementation
public class PortfolioService {
    
    private final Template template;
    
    public void rebalancePortfolio(UUID portfolioId) {
        // Find portfolio
        Portfolio portfolio = template.find(Portfolio.class, portfolioId)
                .orElseThrow(() -> new PortfolioNotFoundException(portfolioId));
        
        // Get high-value investments
        List<InvestmentProjector> highValue = template.typedQuery(
                "FROM Investment WHERE portfolioId = :id AND amount > :threshold",
                InvestmentProjector.class)
                .bind("id", portfolioId)
                .bind("threshold", new BigDecimal("10000"))
                .result()
                .stream()
                .toList();
        
        // Rebalancing logic...
    }
}
```

### Use Case 2: E-Commerce Product Catalog

**Scenario:** An e-commerce platform needs flexible product storage with varying attributes per category.

```java
@Entity
public class Product {
    @Id
    private String sku;
    
    @Column
    private String name;
    
    @Column
    private String category;
    
    @Column
    private Map<String, Object> attributes;  // Flexible schema
    
    @Column
    private List<String> tags;
    
    @Column
    private BigDecimal price;
}

// Different products have different attributes
{
    "sku": "LAPTOP-001",
    "name": "Gaming Laptop",
    "category": "Electronics",
    "attributes": {
        "cpu": "Intel i9",
        "ram": "32GB",
        "storage": "1TB SSD"
    }
}

{
    "sku": "SHIRT-001",
    "name": "Cotton T-Shirt",
    "category": "Clothing",
    "attributes": {
        "size": "L",
        "color": "Blue",
        "material": "Cotton"
    }
}
```

### Use Case 3: AI-Powered Recommendation System

**Scenario:** A content platform uses vector embeddings for personalized recommendations.

```java
@Entity
public class ContentEmbedding {
    @Id
    private UUID contentId;
    
    @Column
    private String contentType;
    
    @Column
    private List<Double> embedding;  // Vector representation
    
    @Column
    private List<String> tags;
    
    @Column
    private LocalDateTime createdAt;
}

// Store content embeddings
public void storeContentEmbedding(UUID contentId, List<Double> embedding) {
    ContentEmbedding entity = new ContentEmbedding(
        contentId,
        "article",
        embedding,
        List.of("technology", "java"),
        LocalDateTime.now()
    );
    
    template.insert(entity);
}

// Find similar content (requires vector database support)
public List<ContentEmbedding> findSimilarContent(List<Double> queryEmbedding, int limit) {
    // Implementation depends on vector database provider
    // This demonstrates the concept
    return template.query(
            "FROM ContentEmbedding WHERE VECTOR_SIMILARITY(embedding, :query) > 0.8")
            .bind("query", queryEmbedding)
            .limit(limit)
            .result()
            .stream()
            .toList();
}
```

### Use Case 4: IoT Time-Series Data

**Scenario:** An IoT platform collects sensor data with high write throughput.

```java
@Entity
public class SensorReading {
    @Id
    private String sensorId;
    
    @Column
    private LocalDateTime timestamp;
    
    @Column
    private Double temperature;
    
    @Column
    private Double humidity;
    
    @Column
    private Map<String, Double> metrics;
}

// Batch insert for high throughput
public void recordSensorData(List<SensorReading> readings) {
    template.insertAll(readings);  // Batch operation
}

// Query recent readings
public List<SensorReading> getRecentReadings(String sensorId, Duration duration) {
    return template.select(SensorReading.class)
            .where("sensorId")
            .eq(sensorId)
            .and("timestamp")
            .gte(LocalDateTime.now().minus(duration))
            .orderBy("timestamp")
            .desc()
            .result()
            .stream()
            .toList();
}
```

---

## Implementation Approaches

### Approach 1: Traditional Class-Based Entities

**Best for:** Complex business logic, mutable entities, legacy codebases

```java
@Entity
public class Investment {
    @Id
    private UUID id;
    
    @Column
    private String name;
    
    @Column
    private BigDecimal amount;
    
    // Business logic
    public BigDecimal calculateReturn(BigDecimal currentValue) {
        return currentValue.subtract(amount).divide(amount, 4, RoundingMode.HALF_UP);
    }
    
    // Getters and setters
    public UUID getId() { return id; }
    public void setId(UUID id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
}
```

**Pros:**
- ✅ Familiar pattern for JPA developers
- ✅ Supports complex business logic
- ✅ Mutable state when needed

**Cons:**
- ❌ More boilerplate code
- ❌ Thread-safety concerns
- ❌ Verbose

### Approach 2: Record-Based Entities (Recommended)

**Best for:** Simple data carriers, immutable data, modern Java applications

```java
@Entity
public record Investment(
        @Id UUID id,
        @Column String name,
        @Column BigDecimal amount) {
    
    // Compact constructor for validation
    public Investment {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or blank");
        }
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
    
    // Additional methods (records can have methods)
    public BigDecimal calculateReturn(BigDecimal currentValue) {
        return currentValue.subtract(amount).divide(amount, 4, RoundingMode.HALF_UP);
    }
}
```

**Pros:**
- ✅ Concise and immutable
- ✅ Thread-safe by default
- ✅ Less boilerplate
- ✅ Built-in `equals()`, `hashCode()`, `toString()`

**Cons:**
- ❌ Immutable (requires creating new instances for updates)
- ❌ Cannot extend other classes

### Approach 3: Repository Pattern with Jakarta Data

**Best for:** Complex queries, type safety, separation of concerns

```java
import jakarta.data.repository.DataRepository;
import jakarta.data.repository.Query;
import jakarta.data.repository.Repository;

@Repository
public interface InvestmentRepository extends DataRepository<Investment, UUID> {
    
    // Derived query methods
    List<Investment> findByType(InvestmentType type);
    
    List<Investment> findByAmountGreaterThan(BigDecimal amount);
    
    List<Investment> findByTypeAndAmountGreaterThan(
            InvestmentType type, 
            BigDecimal amount);
    
    // Custom query with Jakarta Query
    @Query("FROM Investment WHERE amount > :amount ORDER BY amount DESC")
    List<Investment> findHighValueInvestments(@Param("amount") BigDecimal amount);
    
    // Projection query
    @Query("SELECT name, amount FROM Investment WHERE type = :type")
    List<InvestmentProjector> findInvestmentSummaries(
            @Param("type") InvestmentType type);
}
```

**Usage:**

```java
// Inject repository
@Inject
InvestmentRepository investmentRepository;

// Use repository methods
List<Investment> funds = investmentRepository.findByType(InvestmentType.FUND);

List<Investment> highValue = investmentRepository
        .findByAmountGreaterThan(new BigDecimal("10000"));
```

**Pros:**
- ✅ Type-safe queries
- ✅ Less boilerplate
- ✅ Separation of concerns
- ✅ Easy testing with mocks

**Cons:**
- ❌ Less flexibility for complex queries
- ❌ Learning curve for query derivation

### Approach 4: Direct Template API

**Best for:** Dynamic queries, full control, performance-critical operations

```java
public class InvestmentService {
    
    private final Template template;
    
    public List<Investment> findInvestmentsWithCriteria(
            InvestmentType type, 
            BigDecimal minAmount,
            BigDecimal maxAmount) {
        
        // Build query dynamically
        var query = template.select(Investment.class)
                .where("type")
                .eq(type);
        
        if (minAmount != null) {
            query = query.and("amount")
                    .gte(minAmount);
        }
        
        if (maxAmount != null) {
            query = query.and("amount")
                    .lte(maxAmount);
        }
        
        return query.result()
                .stream()
                .toList();
    }
}
```

**Pros:**
- ✅ Full control over queries
- ✅ Dynamic query building
- ✅ No abstraction overhead

**Cons:**
- ❌ More code to maintain
- ❌ Less type safety
- ❌ Harder to test

### Comparison Matrix: Choosing an Approach

| Criteria | Traditional Class | Record | Repository | Template API |
|----------|------------------|--------|------------|--------------|
| **Complexity** | Medium | Low | Low | Medium |
| **Boilerplate** | High | Low | Low | Medium |
| **Flexibility** | High | Medium | Low | High |
| **Type Safety** | Medium | High | High | Low |
| **Testability** | Medium | High | High | Medium |
| **Performance** | Medium | High | High | High |
| **Learning Curve** | Low | Low | Medium | Medium |

> 💡 **Recommendation:** Use **records** for simple entities, **repositories** for standard CRUD operations, and **template API** for complex dynamic queries.

---

## Best Practices

### 1. Entity Design

✅ **DO:**
- Use records for simple, immutable entities
- Keep entities focused on a single responsibility
- Use meaningful field names
- Document complex field purposes with comments
- Use enums for fixed sets of values
- Implement validation in constructors

❌ **DON'T:**
- Don't include business logic in entities (keep them as data carriers)
- Don't use mutable fields with records
- Don't create deeply nested entity hierarchies
- Don't store large binary data (use references instead)

### 2. Query Optimization

✅ **DO:**
- Use projections to fetch only needed fields
- Use indexes for frequently queried fields
- Use parameterized queries to prevent injection
- Implement pagination for large result sets
- Cache frequently accessed data
- Use batch operations for bulk inserts/updates

❌ **DON'T:**
- Don't use `SELECT *` equivalent (fetch all fields when you need few)
- Don't execute queries in loops (batch instead)
- Don't ignore query performance monitoring
- Don't fetch more data than needed

### 3. Error Handling

✅ **DO:**
- Handle `EntityNotFoundException` when finding by ID
- Validate input before database operations
- Use specific exception types
- Log errors with context
- Implement retry logic for transient failures

❌ **DON'T:**
- Don't catch generic `Exception` without handling
- Don't ignore null values from queries
- Don't expose database errors to end users

### 4. Performance

✅ **DO:**
- Use connection pooling
- Implement caching strategies
- Monitor query performance
- Use batch operations
- Optimize indexes based on query patterns
- Consider read replicas for read-heavy workloads

❌ **DON'T:**
- Don't create N+1 query problems
- Don't fetch entire collections when you need few items
- Don't ignore database-specific optimizations

### 5. Security

✅ **DO:**
- Use parameterized queries (never string concatenation)
- Implement authentication and authorization
- Encrypt sensitive data
- Validate all input data
- Use TLS/SSL for database connections
- Apply principle of least privilege for database users

❌ **DON'T:**
- Don't store passwords in plain text
- Don't expose internal database errors
- Don't use admin credentials in application code
- Don't trust user input without validation

---

## Anti-Patterns

### Anti-Pattern 1: God Entity

❌ **Problem:** Creating entities with too many fields and responsibilities

```java
// ❌ Bad: Too many responsibilities
@Entity
public class User {
    @Id
    private UUID id;
    @Column private String name;
    @Column private String email;
    @Column private String password;
    @Column private String address;
    @Column private String phone;
    @Column private List<Order> orders;
    @Column private List<Payment> payments;
    @Column private List<Review> reviews;
    @Column private Preferences preferences;
    @Column private Settings settings;
    // ... 50 more fields
}
```

✅ **Solution:** Split into focused entities

```java
@Entity
public class User {
    @Id
    private UUID id;
    @Column private String name;
    @Column private String email;
    @Column private String password;
}

@Entity
public class UserProfile {
    @Id
    private UUID userId;
    @Column private String address;
    @Column private String phone;
}

@Entity
public class UserPreferences {
    @Id
    private UUID userId;
    @Column private Preferences preferences;
    @Column private Settings settings;
}
```

### Anti-Pattern 2: Anemic Domain Model with Rich Entities

❌ **Problem:** Mixing data carriers with complex business logic

```java
// ❌ Bad: Entity with too much logic
@Entity
public class Investment {
    @Id
    private UUID id;
    @Column private BigDecimal amount;
    
    // Business logic that doesn't belong here
    public BigDecimal calculateTax() { /* 100 lines */ }
    public void generateReport() { /* 200 lines */ }
    public void notifyUser() { /* 50 lines */ }
    public void validateCompliance() { /* 150 lines */ }
}
```

✅ **Solution:** Separate concerns

```java
// Entity: Data only
@Entity
public record Investment(@Id UUID id, @Column BigDecimal amount) {}

// Service: Business logic
public class InvestmentService {
    public BigDecimal calculateTax(Investment investment) {
        // Tax calculation logic
    }
    
    public void generateReport(Investment investment) {
        // Report generation logic
    }
}
```

### Anti-Pattern 3: Query in Loop (N+1 Problem)

❌ **Problem:** Executing queries inside loops

```java
// ❌ Bad: N+1 query problem
List<Portfolio> portfolios = template.select(Portfolio.class).result();
for (Portfolio portfolio : portfolios) {
    // Executes a query for EACH portfolio
    List<Investment> investments = template.select(Investment.class)
            .where("portfolioId")
            .eq(portfolio.getId())
            .result();
    // Process investments...
}
```

✅ **Solution:** Batch fetch or join

```java
// ✅ Good: Single query with projection
List<UUID> portfolioIds = portfolios.stream()
        .map(Portfolio::getId)
        .toList();

Map<UUID, List<Investment>> investmentsByPortfolio = 
    template.select(Investment.class)
        .where("portfolioId")
        .in(portfolioIds)
        .result()
        .stream()
        .collect(groupingBy(Investment::getPortfolioId));
```

### Anti-Pattern 4: Ignoring Database Specifics

❌ **Problem:** Treating all NoSQL databases the same

```java
// ❌ Bad: Assuming all databases support the same features
template.update(Investment.class)
        .set("amount")
        .inc(new BigDecimal("100"))  // May not work in all databases!
        .where("id")
        .eq(id)
        .execute();
```

✅ **Solution:** Check database capabilities

```java
// ✅ Good: Check database support
if (databaseSupportsIncrementOperations()) {
    template.update(Investment.class)
            .set("amount")
            .inc(new BigDecimal("100"))
            .where("id")
            .eq(id)
            .execute();
} else {
    // Fallback: Read, modify, update
    Investment investment = template.find(Investment.class, id)
            .orElseThrow();
    investment = new Investment(
        investment.id(),
        investment.name(),
        investment.type(),
        investment.amount().add(new BigDecimal("100"))
    );
    template.update(investment);
}
```

### Anti-Pattern 5: String-Based IDs Without Validation

❌ **Problem:** Using string concatenation for IDs

```java
// ❌ Bad: String concatenation for IDs
String id = "INV-" + System.currentTimeMillis();
Investment investment = new Investment(id, name, type, amount);
```

✅ **Solution:** Use proper ID generation

```java
// ✅ Good: UUID or database-generated IDs
UUID id = UUID.randomUUID();
// Or let database generate ID
Investment investment = new Investment(null, name, type, amount);
template.insert(investment);  // Database generates ID
```

---

## Performance Considerations

### 1. Query Performance

**Indexing Strategy:**

```mermaid
graph TB
    A[Query Patterns] --> B{Analyze Queries}
    B --> C[Identify Frequent Filters]
    B --> D[Identify Sort Fields]
    B --> E[Identify Join Fields]
    
    C --> F[Create Indexes]
    D --> F
    E --> F
    
    F --> G[Single-Field Indexes]
    F --> H[Composite Indexes]
    F --> I[Unique Indexes]
    
    G --> J[Monitor Performance]
    H --> J
    I --> J
    
    J --> K[Optimize Based on Metrics]
```

**Example: Creating Indexes**

```java
// MongoDB example (in application.properties)
spring.data.mongodb.auto-index-creation=true

// Or programmatically
MongoTemplate mongoTemplate = ...;
IndexOperations indexOps = mongoTemplate.indexOps(Investment.class);
Index index = new Index().on("amount", Direction.DESCENDING).on("type", Direction.ASCENDING);
indexOps.ensureIndex(index);
```

### 2. Connection Pooling

**Configuration Example (HikariCP):**

```yaml
# application.yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

### 3. Caching Strategies

**Level 1: Application-Level Caching**

```java
import jakarta.inject.Inject;
import jakarta.nosql.template.Template;
import java.util.Optional;
import java.util.concurrent.TimeUnit;

public class InvestmentService {
    
    private final Template template;
    private final Cache<UUID, Investment> cache;
    
    public InvestmentService(Template template) {
        this.template = template;
        this.cache = Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(1, TimeUnit.HOURS)
                .build();
    }
    
    public Optional<Investment> findById(UUID id) {
        return cache.get(id, key -> 
            template.find(Investment.class, key)
        );
    }
}
```

**Level 2: Database-Level Caching**

```yaml
# Redis configuration for caching
spring:
  data:
    redis:
      host: localhost
      port: 6379
      time-to-live: 3600000  # 1 hour
```

### 4. Batch Operations

**Batch Insert:**

```java
// ✅ Good: Batch insert
List<Investment> investments = generateInvestments(1000);
template.insertAll(investments);  // Single operation
```

**Batch Update:**

```java
// ✅ Good: Batch update
template.update(Investment.class)
        .set("status")
        .to("ACTIVE")
        .where("amount")
        .gte(new BigDecimal("1000"))
        .execute();
```

### 5. Performance Monitoring

**Key Metrics to Track:**

| Metric | Target | Tool |
|--------|--------|------|
| Query execution time | < 100ms | APM tools (New Relic, Datadog) |
| Connection pool utilization | < 80% | HikariCP metrics |
| Cache hit ratio | > 90% | Cache metrics |
| Error rate | < 0.1% | Application logs |
| Throughput | > 1000 ops/sec | Load testing |

**Example: Monitoring with Micrometer**

```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;

public class MonitoredInvestmentService {
    
    private final Template template;
    private final MeterRegistry meterRegistry;
    private final Timer findTimer;
    
    public MonitoredInvestmentService(Template template, MeterRegistry meterRegistry) {
        this.template = template;
        this.meterRegistry = meterRegistry;
        this.findTimer = Timer.builder("investment.find")
                .description("Time to find investments")
                .register(meterRegistry);
    }
    
    public Optional<Investment> findById(UUID id) {
        return findTimer.record(() -> 
            template.find(Investment.class, id)
        );
    }
}
```

---

## Security Considerations

### 1. NoSQL Injection Prevention

❌ **Vulnerable Code:**

```java
// ❌ Bad: String concatenation (vulnerable to injection)
String userId = request.getParameter("userId");
template.query("FROM Investment WHERE userId = '" + userId + "'")
        .result();
```

**Attack Example:**
```
userId = "123' OR '1'='1"
Query becomes: FROM Investment WHERE userId = '123' OR '1'='1'
```

✅ **Secure Code:**

```java
// ✅ Good: Parameterized query
String userId = request.getParameter("userId");
template.query("FROM Investment WHERE userId = :userId")
        .bind("userId", userId)
        .result();
```

### 2. Data Encryption

**At Rest:**

```java
import jakarta.nosql.Converter;
import jakarta.nosql.AttributeConverter;
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import java.util.Base64;

@Converter(autoApply = true)
public class EncryptedStringConverter implements AttributeConverter<String, String> {
    
    private static final String ALGORITHM = "AES";
    private static final SecretKey SECRET_KEY = generateKey();
    
    @Override
    public String convertToDatabaseColumn(String attribute) {
        if (attribute == null) return null;
        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, SECRET_KEY);
            return Base64.getEncoder().encodeToString(
                cipher.doFinal(attribute.getBytes())
            );
        } catch (Exception e) {
            throw new PersistenceException("Encryption failed", e);
        }
    }
    
    @Override
    public String convertToEntityAttribute(String dbData) {
        if (dbData == null) return null;
        try {
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, SECRET_KEY);
            return new String(cipher.doFinal(Base64.getDecoder().decode(dbData)));
        } catch (Exception e) {
            throw new PersistenceException("Decryption failed", e);
        }
    }
    
    private static SecretKey generateKey() {
        try {
            KeyGenerator keyGen = KeyGenerator.getInstance(ALGORITHM);
            keyGen.init(256);
            return keyGen.generateKey();
        } catch (Exception e) {
            throw new PersistenceException("Key generation failed", e);
        }
    }
}
```

**In Transit:**

```yaml
# Always use TLS/SSL
spring:
  data:
    mongodb:
      uri: mongodb://user:pass@host:27017/db?ssl=true
```

### 3. Authentication & Authorization

```java
import jakarta.annotation.security.RolesAllowed;
import jakarta.inject.Inject;
import jakarta.nosql.template.Template;

public class SecureInvestmentService {
    
    private final Template template;
    private final SecurityContext securityContext;
    
    @Inject
    public SecureInvestmentService(Template template, SecurityContext securityContext) {
        this.template = template;
        this.securityContext = securityContext;
    }
    
    @RolesAllowed("USER")
    public Optional<Investment> findById(UUID id) {
        String currentUser = securityContext.getCallerPrincipal().getName();
        
        // Ensure user can only access their own investments
        return template.select(Investment.class)
                .where("id")
                .eq(id)
                .and("owner")
                .eq(currentUser)
                .result()
                .findFirst();
    }
    
    @RolesAllowed("ADMIN")
    public List<Investment> findAll() {
        return template.select(Investment.class)
                .result()
                .stream()
                .toList();
    }
}
```

### 4. Input Validation

```java
import jakarta.validation.Valid;
import jakarta.validation.constraints.*;
import jakarta.nosql.Entity;
import jakarta.nosql.Column;
import jakarta.nosql.Id;
import java.math.BigDecimal;
import java.util.UUID;

@Entity
public class Investment {
    
    @Id
    private UUID id;
    
    @Column
    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 100, message = "Name must be 3-100 characters")
    private String name;
    
    @Column
    @NotNull(message = "Amount is required")
    @DecimalMin(value = "0.01", message = "Amount must be positive")
    @DecimalMax(value = "1000000000", message = "Amount exceeds maximum")
    private BigDecimal amount;
    
    @Column
    @NotNull(message = "Type is required")
    private InvestmentType type;
    
    // Constructor with validation
    public Investment(@NotBlank String name, 
                     @NotNull @DecimalMin("0.01") BigDecimal amount,
                     @NotNull InvestmentType type) {
        this.name = name;
        this.amount = amount;
        this.type = type;
    }
}
```

### 5. Security Checklist

- [ ] Use parameterized queries (never string concatenation)
- [ ] Encrypt sensitive data at rest
- [ ] Use TLS/SSL for database connections
- [ ] Implement authentication and authorization
- [ ] Validate all user input
- [ ] Use principle of least privilege for database users
- [ ] Never expose database errors to end users
- [ ] Implement rate limiting
- [ ] Log security events
- [ ] Regular security audits
- [ ] Keep dependencies updated
- [ ] Use secrets management (not hardcoded credentials)

---

## Testing Strategies

### 1. Unit Testing with Mocks

```java
import jakarta.nosql.template.Template;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import java.math.BigDecimal;
import java.util.Optional;
import java.util.UUID;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class InvestmentServiceTest {
    
    @Mock
    private Template template;
    
    @InjectMocks
    private InvestmentService investmentService;
    
    @Test
    void shouldFindInvestmentById() {
        // Arrange
        UUID id = UUID.randomUUID();
        Investment expected = new Investment(id, "Test Fund", InvestmentType.FUND, 
                                             new BigDecimal("1000"));
        when(template.find(Investment.class, id)).thenReturn(Optional.of(expected));
        
        // Act
        Optional<Investment> result = investmentService.findById(id);
        
        // Assert
        assertTrue(result.isPresent());
        assertEquals(expected, result.get());
        verify(template).find(Investment.class, id);
    }
    
    @Test
    void shouldThrowExceptionWhenInvestmentNotFound() {
        // Arrange
        UUID id = UUID.randomUUID();
        when(template.find(Investment.class, id)).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(InvestmentNotFoundException.class, 
            () -> investmentService.findByIdOrThrow(id));
    }
}
```

### 2. Integration Testing

```java
import jakarta.nosql.template.Template;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.math.BigDecimal;
import java.util.UUID;
import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
class InvestmentRepositoryIntegrationTest {
    
    @Autowired
    private Template template;
    
    @BeforeEach
    void cleanUp() {
        // Clean database before each test
        template.delete(Investment.class).execute();
    }
    
    @Test
    void shouldInsertAndFindInvestment() {
        // Arrange
        UUID id = UUID.randomUUID();
        Investment investment = new Investment(id, "Test Fund", InvestmentType.FUND, 
                                               new BigDecimal("1000"));
        
        // Act
        template.insert(investment);
        Optional<Investment> found = template.find(Investment.class, id);
        
        // Assert
        assertTrue(found.isPresent());
        assertEquals(investment, found.get());
    }
    
    @Test
    void shouldQueryInvestmentsByType() {
        // Arrange
        template.insert(new Investment(UUID.randomUUID(), "Fund 1", InvestmentType.FUND, 
                                       new BigDecimal("1000")));
        template.insert(new Investment(UUID.randomUUID(), "Stock 1", InvestmentType.STOCK, 
                                       new BigDecimal("2000")));
        template.insert(new Investment(UUID.randomUUID(), "Fund 2", InvestmentType.FUND, 
                                       new BigDecimal("3000")));
        
        // Act
        List<Investment> funds = template.select(Investment.class)
                .where("type")
                .eq(InvestmentType.FUND)
                .result()
                .stream()
                .toList();
        
        // Assert
        assertEquals(2, funds.size());
    }
}
```

### 3. Test Data Builders

```java
public class InvestmentTestDataBuilder {
    
    private UUID id = UUID.randomUUID();
    private String name = "Test Investment";
    private InvestmentType type = InvestmentType.FUND;
    private BigDecimal amount = new BigDecimal("1000.00");
    
    public static InvestmentTestDataBuilder anInvestment() {
        return new InvestmentTestDataBuilder();
    }
    
    public InvestmentTestDataBuilder withId(UUID id) {
        this.id = id;
        return this;
    }
    
    public InvestmentTestDataBuilder withName(String name) {
        this.name = name;
        return this;
    }
    
    public InvestmentTestDataBuilder withType(InvestmentType type) {
        this.type = type;
        return this;
    }
    
    public InvestmentTestDataBuilder withAmount(BigDecimal amount) {
        this.amount = amount;
        return this;
    }
    
    public Investment build() {
        return new Investment(id, name, type, amount);
    }
}

// Usage in tests
@Test
void shouldCreateInvestment() {
    Investment investment = anInvestment()
            .withName("High Value Fund")
            .withAmount(new BigDecimal("5000"))
            .build();
    
    template.insert(investment);
    // ... assertions
}
```

### 4. Performance Testing

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import java.util.ArrayList;
import java.util.List;
import static org.junit.jupiter.api.Assertions.*;

class InvestmentPerformanceTest {
    
    private Template template;
    
    @BeforeEach
    void setUp() {
        template = ...;
    }
    
    @Test
    void shouldInsert1000InvestmentsInUnder1Second() {
        // Arrange
        List<Investment> investments = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            investments.add(new Investment(
                UUID.randomUUID(),
                "Investment " + i,
                InvestmentType.FUND,
                new BigDecimal("1000")
            ));
        }
        
        // Act
        long start = System.currentTimeMillis();
        template.insertAll(investments);
        long duration = System.currentTimeMillis() - start;
        
        // Assert
        assertTrue(duration < 1000, 
            "Insert took " + duration + "ms, expected < 1000ms");
    }
}
```

---

## Common Pitfalls & Troubleshooting

### Pitfall 1: Missing Default Constructor

❌ **Problem:**

```java
@Entity
public record Investment(@Id UUID id, @Column String name) {
    // No default constructor - records don't have one!
}
```

**Error:** `No default constructor for entity: Investment`

✅ **Solution:**

```java
@Entity
public record Investment(@Id UUID id, @Column String name) {
    // Records work fine - Jakarta NoSQL supports them
    // The compact constructor is sufficient
    public Investment {
        if (name == null) throw new IllegalArgumentException("Name required");
    }
}
```

### Pitfall 2: Forgetting @Id or @Column Annotations

❌ **Problem:**

```java
@Entity
public class Investment {
    @Id
    private UUID id;
    
    private String name;  // ❌ Missing @Column - will be ignored!
    private BigDecimal amount;  // ❌ Missing @Column - will be ignored!
}
```

✅ **Solution:**

```java
@Entity
public class Investment {
    @Id
    private UUID id;
    
    @Column
    private String name;
    
    @Column
    private BigDecimal amount;
}
```

### Pitfall 3: Using Wrong Field Types

❌ **Problem:**

```java
@Entity
public class Investment {
    @Id
    private String id;  // ❌ String instead of UUID
    
    @Column
    private Date createdAt;  // ❌ java.util.Date instead of java.time.LocalDateTime
}
```

✅ **Solution:**

```java
@Entity
public class Investment {
    @Id
    private UUID id;  // ✅ Correct type
    
    @Column
    private LocalDateTime createdAt;  // ✅ Modern Java time API
}
```

### Pitfall 4: Not Handling Optional Values

❌ **Problem:**

```java
Optional<Investment> result = template.find(Investment.class, id);
Investment investment = result.get();  // ❌ NoSuchElementException if not found!
```

✅ **Solution:**

```java
Optional<Investment> result = template.find(Investment.class, id);
Investment investment = result.orElseThrow(() -> 
    new InvestmentNotFoundException(id)
);

// Or provide default
Investment investment = result.orElse(Investment.empty());
```

### Pitfall 5: Ignoring Database-Specific Features

❌ **Problem:**

```java
// ❌ Assuming all databases support transactions
template.insert(investment1);
template.insert(investment2);
// If second insert fails, first is already committed!
```

✅ **Solution:**

```java
// ✅ Use transactions if supported
template.beginTransaction();
try {
    template.insert(investment1);
    template.insert(investment2);
    template.commitTransaction();
} catch (Exception e) {
    template.rollbackTransaction();
    throw e;
}
```

### Troubleshooting Guide

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Entity not persisting** | No error, but data not in database | Check for `@Id` and `@Column` annotations |
| **Query returns empty** | Query executes but no results | Verify field names match database, check case sensitivity |
| **Slow queries** | High latency | Add indexes, use projections, check query plan |
| **Connection timeout** | Cannot connect to database | Check network, verify credentials, check connection pool |
| **Type conversion error** | `ClassCastException` | Verify converter registration, check field types |
| **Transaction not rolling back** | Partial data saved | Ensure exception is not caught and swallowed |

---

## Practice Exercises

### Exercise 1: Basic CRUD Operations

**Task:** Create a complete CRUD service for managing books in a library system.

**Requirements:**
1. Define a `Book` entity with the following fields:
   - `id` (UUID, primary key)
   - `title` (String)
   - `author` (String)
   - `isbn` (String)
   - `publishedDate` (LocalDate)
   - `genre` (enum: FICTION, NON_FICTION, SCIENCE, HISTORY)
   - `price` (BigDecimal)

2. Implement a `BookService` with methods:
   - `addBook(Book book)` - Insert a new book
   - `findBookById(UUID id)` - Find book by ID
   - `findBooksByGenre(Genre genre)` - Find all books in a genre
   - `updatePrice(UUID id, BigDecimal newPrice)` - Update book price
   - `deleteBook(UUID id)` - Delete a book

3. Add proper error handling and validation

**Solution:**

```java
// 1. Entity Definition
import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import jakarta.validation.constraints.Size;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.UUID;

@Entity
public record Book(
        @Id UUID id,
        @Column @NotBlank @Size(min = 1, max = 200) String title,
        @Column @NotBlank String author,
        @Column @NotBlank @Size(min = 10, max = 13) String isbn,
        @Column @NotNull LocalDate publishedDate,
        @Column @NotNull Genre genre,
        @Column @NotNull @Positive BigDecimal price) {
    
    public Book {
        if (id == null) {
            throw new IllegalArgumentException("ID cannot be null");
        }
    }
    
    public static Book of(String title, String author, String isbn, 
                         LocalDate publishedDate, Genre genre, BigDecimal price) {
        return new Book(UUID.randomUUID(), title, author, isbn, 
                       publishedDate, genre, price);
    }
}

// Enum
public enum Genre {
    FICTION,
    NON_FICTION,
    SCIENCE,
    HISTORY
}

// 2. Service Implementation
import jakarta.inject.Inject;
import jakarta.nosql.template.Template;
import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;
import java.util.UUID;
import java.util.stream.Collectors;

public class BookService {
    
    private final Template template;
    
    @Inject
    public BookService(Template template) {
        this.template = template;
    }
    
    // Add book
    public Book addBook(String title, String author, String isbn, 
                       LocalDate publishedDate, Genre genre, BigDecimal price) {
        Book book = new Book(UUID.randomUUID(), title, author, isbn, 
                            publishedDate, genre, price);
        template.insert(book);
        return book;
    }
    
    // Find by ID
    public Optional<Book> findBookById(UUID id) {
        return template.find(Book.class, id);
    }
    
    // Find by genre
    public List<Book> findBooksByGenre(Genre genre) {
        return template.select(Book.class)
                .where("genre")
                .eq(genre)
                .result()
                .stream()
                .collect(Collectors.toList());
    }
    
    // Update price
    public Book updatePrice(UUID id, BigDecimal newPrice) {
        template.update(Book.class)
                .set("price")
                .to(newPrice)
                .where("id")
                .eq(id)
                .execute();
        
        return template.find(Book.class, id)
                .orElseThrow(() -> new BookNotFoundException(id));
    }
    
    // Delete book
    public boolean deleteBook(UUID id) {
        Book book = template.find(Book.class, id)
                .orElseThrow(() -> new BookNotFoundException(id));
        
        template.delete(book);
        return true;
    }
    
    // Additional: Find books by price range
    public List<Book> findBooksInPriceRange(BigDecimal min, BigDecimal max) {
        return template.select(Book.class)
                .where("price")
                .gte(min)
                .and("price")
                .lte(max)
                .result()
                .stream()
                .collect(Collectors.toList());
    }
}

// Custom Exception
public class BookNotFoundException extends RuntimeException {
    public BookNotFoundException(UUID id) {
        super("Book not found with ID: " + id);
    }
}
```

**Usage Example:**

```java
// Initialize service
Template template = ...;
BookService bookService = new BookService(template);

// Add books
Book book1 = bookService.addBook(
    "Clean Code", 
    "Robert Martin", 
    "978-0132350884",
    LocalDate.of(2008, 8, 1),
    Genre.NON_FICTION,
    new BigDecimal("39.99")
);

Book book2 = bookService.addBook(
    "Dune",
    "Frank Herbert",
    "978-0441172719",
    LocalDate.of(1965, 8, 1),
    Genre.FICTION,
    new BigDecimal("24.99")
);

// Find books
Optional<Book> found = bookService.findBookById(book1.id());
System.out.println("Found: " + found.get());

// Find by genre
List<Book> fictionBooks = bookService.findBooksByGenre(Genre.FICTION);
System.out.println("Fiction books: " + fictionBooks.size());

// Update price
Book updated = bookService.updatePrice(book1.id(), new BigDecimal("34.99"));
System.out.println("Updated price: " + updated.price());

// Delete
bookService.deleteBook(book2.id());
```

### Exercise 2: Advanced Queries with Projections

**Task:** Implement a reporting system for investment analysis using projections and complex queries.

**Requirements:**
1. Create a projection `InvestmentSummary` with:
   - `name`
   - `type`
   - `amount`

2. Implement methods to:
   - Get total investment count by type
   - Get average investment amount by type
   - Get top 10 highest-value investments
   - Get investment summaries for a specific type

**Solution:**

```java
// 1. Projection Definition
import jakarta.nosql.Projection;

@Projection
public record InvestmentSummary(
        String name,
        InvestmentType type,
        BigDecimal amount) {
}

// 2. Service Implementation
import jakarta.inject.Inject;
import jakarta.nosql.template.Template;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class InvestmentReportService {
    
    private final Template template;
    
    @Inject
    public InvestmentReportService(Template template) {
        this.template = template;
    }
    
    // Get total count by type
    public Map<InvestmentType, Long> getInvestmentCountByType() {
        List<Investment> allInvestments = template.select(Investment.class)
                .result()
                .stream()
                .toList();
        
        return allInvestments.stream()
                .collect(Collectors.groupingBy(
                    Investment::type,
                    Collectors.counting()
                ));
    }
    
    // Get average amount by type
    public Map<InvestmentType, BigDecimal> getAverageAmountByType() {
        List<Investment> allInvestments = template.select(Investment.class)
                .result()
                .stream()
                .toList();
        
        return allInvestments.stream()
                .collect(Collectors.groupingBy(
                    Investment::type,
                    Collectors.averagingBigDecimal(Investment::amount)
                ))
                .entrySet()
                .stream()
                .collect(Collectors.toMap(
                    Map.Entry::getKey,
                    e -> e.getValue().setScale(2, RoundingMode.HALF_UP)
                ));
    }
    
    // Get top 10 highest-value investments
    public List<InvestmentSummary> getTop10Investments() {
        return template.typedQuery(
                "FROM Investment ORDER BY amount DESC",
                InvestmentSummary.class)
                .limit(10)
                .result()
                .stream()
                .toList();
    }
    
    // Get summaries by type
    public List<InvestmentSummary> getSummariesByType(InvestmentType type) {
        return template.typedQuery(
                "FROM Investment WHERE type = :type ORDER BY amount DESC",
                InvestmentSummary.class)
                .bind("type", type)
                .result()
                .stream()
                .toList();
    }
    
    // Get total value by type
    public Map<InvestmentType, BigDecimal> getTotalValueByType() {
        List<Investment> allInvestments = template.select(Investment.class)
                .result()
                .stream()
                .toList();
        
        return allInvestments.stream()
                .collect(Collectors.groupingBy(
                    Investment::type,
                    Collectors.reducing(BigDecimal.ZERO, Investment::amount, BigDecimal::add)
                ));
    }
    
    // Get investment statistics
    public InvestmentStatistics getStatistics() {
        List<Investment> allInvestments = template.select(Investment.class)
                .result()
                .stream()
                .toList();
        
        BigDecimal totalValue = allInvestments.stream()
                .map(Investment::amount)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        BigDecimal averageValue = allInvestments.stream()
                .map(Investment::amount)
                .reduce(BigDecimal.ZERO, BigDecimal::add)
                .divide(BigDecimal.valueOf(allInvestments.size()), 2, RoundingMode.HALF_UP);
        
        return new InvestmentStatistics(
            allInvestments.size(),
            totalValue,
            averageValue
        );
    }
}

// Statistics record
public record InvestmentStatistics(
        long totalCount,
        BigDecimal totalValue,
        BigDecimal averageValue) {
}

// Usage Example
public class ReportGenerator {
    
    private final InvestmentReportService reportService;
    
    public void generateReport() {
        // Get statistics
        InvestmentStatistics stats = reportService.getStatistics();
        System.out.println("Total Investments: " + stats.totalCount());
        System.out.println("Total Value: $" + stats.totalValue());
        System.out.println("Average Value: $" + stats.averageValue());
        
        // Get count by type
        Map<InvestmentType, Long> countByType = reportService.getInvestmentCountByType();
        countByType.forEach((type, count) -> 
            System.out.println(type + ": " + count + " investments")
        );
        
        // Get top 10
        List<InvestmentSummary> top10 = reportService.getTop10Investments();
        System.out.println("\nTop 10 Investments:");
        top10.forEach(summary -> 
            System.out.println(summary.name() + ": $" + summary.amount())
        );
    }
}
```

### Exercise 3: Custom Type Converter with Auto-Apply

**Task:** Implement a custom type converter for handling monetary values with currency, and use it across multiple entities.

**Requirements:**
1. Create a `Money` value object that holds amount and currency
2. Implement a converter that serializes `Money` to a database-friendly format
3. Use `autoApply = true` to automatically apply the converter
4. Create entities that use the `Money` type
5. Demonstrate the converter working across different entities

**Solution:**

```java
// 1. Money Value Object
import java.math.BigDecimal;
import java.util.Currency;

public record Money(BigDecimal amount, Currency currency) {
    
    public Money {
        if (amount == null) {
            throw new IllegalArgumentException("Amount cannot be null");
        }
        if (currency == null) {
            throw new IllegalArgumentException("Currency cannot be null");
        }
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
    }
    
    public static Money of(BigDecimal amount, Currency currency) {
        return new Money(amount, currency);
    }
    
    public static Money of(String amount, String currencyCode) {
        return new Money(
            new BigDecimal(amount),
            Currency.getInstance(currencyCode)
        );
    }
    
    public Money add(Money other) {
        validateSameCurrency(other);
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    public Money multiply(BigDecimal multiplier) {
        return new Money(this.amount.multiply(multiplier), this.currency);
    }
    
    private void validateSameCurrency(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException(
                "Currency mismatch: " + this.currency + " vs " + other.currency
            );
        }
    }
    
    @Override
    public String toString() {
        return currency.getSymbol() + " " + amount;
    }
}

// 2. Custom Converter
import jakarta.nosql.Converter;
import jakarta.nosql.AttributeConverter;
import java.math.BigDecimal;
import java.util.Currency;

@Converter(autoApply = true)  // Auto-applied to all Money fields
public class MoneyConverter implements AttributeConverter<Money, String> {
    
    private static final String SEPARATOR = ":";
    
    @Override
    public String convertToDatabaseColumn(Money attribute) {
        if (attribute == null) {
            return null;
        }
        return attribute.amount().toPlainString() + SEPARATOR + 
               attribute.currency().getCurrencyCode();
    }
    
    @Override
    public Money convertToEntityAttribute(String dbData) {
        if (dbData == null || dbData.isBlank()) {
            return null;
        }
        
        String[] parts = dbData.split(SEPARATOR);
        if (parts.length != 2) {
            throw new IllegalArgumentException(
                "Invalid Money format: " + dbData
            );
        }
        
        BigDecimal amount = new BigDecimal(parts[0]);
        Currency currency = Currency.getInstance(parts[1]);
        
        return new Money(amount, currency);
    }
}

// 3. Entities Using Money
import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;
import java.math.BigDecimal;
import java.util.Currency;
import java.util.UUID;

@Entity
public class Investment {
    @Id
    private UUID id;
    
    @Column
    private String name;
    
    @Column
    private Money principalAmount;  // Automatically converted!
    
    @Column
    private Money currentValue;  // Automatically converted!
    
    @Column
    private Currency currency;  // Also automatically converted!
    
    // Constructor
    public Investment(UUID id, String name, Money principalAmount, Money currentValue) {
        this.id = id;
        this.name = name;
        this.principalAmount = principalAmount;
        this.currentValue = currentValue;
        this.currency = principalAmount.currency();
    }
    
    // Calculate returns
    public Money calculateReturns() {
        return currentValue.subtract(principalAmount);
    }
    
    public BigDecimal calculateReturnPercentage() {
        return currentValue.amount()
                .subtract(principalAmount.amount())
                .divide(principalAmount.amount(), 4, RoundingMode.HALF_UP)
                .multiply(new BigDecimal("100"));
    }
}

@Entity
public class Transaction {
    @Id
    private UUID id;
    
    @Column
    private String description;
    
    @Column
    private Money amount;  // Automatically converted!
    
    @Column
    private LocalDateTime timestamp;
    
    @Column
    private TransactionType type;
}

// 4. Usage Example
public class MoneyConverterDemo {
    
    private final Template template;
    
    public void demonstrateConverter() {
        // Create investment with Money objects
        Money principal = Money.of("10000.00", "USD");
        Money current = Money.of("12500.00", "USD");
        
        Investment investment = new Investment(
            UUID.randomUUID(),
            "Tech Growth Fund",
            principal,
            current
        );
        
        // Insert - converter automatically handles serialization
        template.insert(investment);
        
        // Retrieve - converter automatically handles deserialization
        Optional<Investment> found = template.find(Investment.class, investment.id());
        
        if (found.isPresent()) {
            Investment inv = found.get();
            System.out.println("Principal: " + inv.principalAmount());
            System.out.println("Current: " + inv.currentAmount());
            System.out.println("Returns: " + inv.calculateReturns());
            System.out.println("Return %: " + inv.calculateReturnPercentage() + "%");
        }
    }
}

// 5. Testing the Converter
public class MoneyConverterTest {
    
    @Test
    void shouldConvertMoneyToDatabaseColumn() {
        MoneyConverter converter = new MoneyConverter();
        Money money = Money.of("1000.50", "USD");
        
        String dbValue = converter.convertToDatabaseColumn(money);
        
        assertEquals("1000.50:USD", dbValue);
    }
    
    @Test
    void shouldConvertDatabaseColumnToMoney() {
        MoneyConverter converter = new MoneyConverter();
        
        Money money = converter.convertToEntityAttribute("1000.50:USD");
        
        assertEquals(new BigDecimal("1000.50"), money.amount());
        assertEquals(Currency.getInstance("USD"), money.currency());
    }
    
    @Test
    void shouldHandleNullValues() {
        MoneyConverter converter = new MoneyConverter();
        
        assertNull(converter.convertToDatabaseColumn(null));
        assertNull(converter.convertToEntityAttribute(null));
    }
}
```

---

## Test Your Understanding

### Questions 1-10: Fundamental Concepts

1. **What is polyglot persistence and why is it important in modern application development?**

2. **Explain the key difference between Jakarta NoSQL and Jakarta Persistence (JPA) in terms of attribute mapping.**

3. **What are the four main types of NoSQL databases supported by Jakarta NoSQL?**

4. **Why does Jakarta NoSQL require explicit `@Id` or `@Column` annotations on all persistent fields?**

5. **What are the benefits of using Java records for entity definitions in Jakarta NoSQL?**

6. **What is the Template API and what role does it play in Jakarta NoSQL?**

7. **Explain the concept of projections in Jakarta NoSQL 1.1 and when to use them.**

8. **What is Jakarta Query and how does it integrate with Jakarta NoSQL 1.1?**

9. **Describe the `autoApply` attribute in the `@Converter` annotation and its benefits.**

10. **What are the main components of the Jakarta NoSQL architecture?**

### Questions 11-20: Practical Implementation

11. **Write a complete entity definition for an `Order` entity with the following fields: order ID (UUID), customer ID (UUID), order date (LocalDateTime), total amount (BigDecimal), and status (enum: PENDING, PROCESSING, SHIPPED, DELIVERED).**

12. **How would you implement a query to find all investments with an amount greater than $5000 using both the fluent API and Jakarta Query Language?**

13. **Write code to insert multiple investments in a batch operation.**

14. **How do you handle the case when `template.find()` returns an empty Optional?**

15. **Implement a fluent update operation that sets the status to "ACTIVE" for all investments with an amount greater than $10,000.**

16. **Create a projection for an Investment entity that returns only the name and amount fields.**

17. **Write a parameterized query using Jakarta Query Language to find investments by type and minimum amount.**

18. **How would you implement error handling for database constraint violations?**

19. **Create a custom converter for a `PhoneNumber` value object and apply it to a field.**

20. **Write code to implement pagination for a query that returns large result sets.**

### Questions 21-30: Advanced Topics

21. **Explain the N+1 query problem and how to avoid it in Jakarta NoSQL applications.**

22. **What strategies can you use to optimize query performance in Jakarta NoSQL?**

23. **How does connection pooling improve application performance?**

24. **Describe different caching strategies for Jakarta NoSQL applications.**

25. **What security measures should be implemented to prevent NoSQL injection attacks?**

26. **How would you implement data encryption for sensitive fields?**

27. **Explain the principle of least privilege in the context of database security.**

28. **What are the trade-offs between using records vs traditional classes for entities?**

29. **How would you choose between using the Repository pattern vs direct Template API?**

30. **Describe how you would implement batch processing for importing large datasets.**

### Questions 31-40: Best Practices & Anti-Patterns

31. **List five best practices for entity design in Jakarta NoSQL.**

32. **What is a "God Entity" anti-pattern and how do you avoid it?**

33. **Why is it important to use parameterized queries instead of string concatenation?**

34. **Explain why you should not store large binary data directly in NoSQL databases.**

35. **What are the benefits of using projections for read-only operations?**

36. **How does the explicit attribute marking in Jakarta NoSQL improve code clarity?**

37. **Why should you check database-specific features before using them?**

38. **What is the recommended approach for handling transactions in Jakarta NoSQL?**

39. **How does using immutable records improve thread safety?**

40. **Why is input validation important even when using Jakarta NoSQL?**

### Questions 41-50: Real-World Scenarios

41. **You need to build a product catalog with flexible attributes per category. How would you design the entity structure?**

42. **Design a solution for storing and querying vector embeddings for a recommendation system.**

43. **How would you implement an audit trail for tracking changes to entities?**

44. **You need to store time-series IoT sensor data with high write throughput. What approach would you use?**

45. **Design a multi-tenant application where each tenant's data is isolated. How would you structure the entities?**

46. **How would you implement soft deletes in Jakarta NoSQL?**

47. **You need to support multiple currencies in a financial application. How would you implement this?**

48. **Design a caching strategy for frequently accessed investment data.**

49. **How would you implement data validation and business rules in entity definitions?**

50. **You need to migrate from JPA to Jakarta NoSQL. What steps would you take?**

---

## Common Interview Questions

### Questions 1-10: Conceptual

1. **What is polyglot persistence and when should you use it?**

2. **How does Jakarta NoSQL differ from MongoDB's native driver or other NoSQL client libraries?**

3. **What are the main advantages of using a specification like Jakarta NoSQL over vendor-specific APIs?**

4. **Explain the relationship between Jakarta NoSQL, Jakarta Data, and Jakarta Query.**

5. **What types of NoSQL databases does Jakarta NoSQL support?**

6. **How does Jakarta NoSQL handle schema flexibility?**

7. **What is the significance of Jakarta NoSQL being the first specification developed within the Jakarta EE ecosystem?**

8. **How does Jakarta NoSQL support the RAG (Retrieval-Augmented Generation) pattern?**

9. **What are projections and why are they useful?**

10. **How does the `autoApply` feature in `@Converter` improve developer productivity?**

### Questions 11-20: Technical

11. **Write a complete entity definition for a social media post with comments and likes.**

12. **How would you implement a query that returns distinct values for a field?**

13. **Explain how to use named parameters in Jakarta Query Language and why they're important.**

14. **What is the difference between `template.select()` and `template.query()`?**

15. **How do you handle transactions in Jakarta NoSQL?**

16. **Write code to implement a many-to-many relationship between entities.**

17. **How would you optimize a query that's performing slowly?**

18. **Explain how to use Java records with Jakarta NoSQL and their benefits.**

19. **What strategies would you use for testing Jakarta NoSQL applications?**

20. **How do you handle database-specific features while maintaining portability?**

### Questions 21-30: Design & Architecture

21. **Design a document management system using Jakarta NoSQL. What entities would you create?**

22. **How would you architect a real-time analytics system using Jakarta NoSQL?**

23. **Design a user preference system that supports flexible, user-defined attributes.**

24. **How would you implement a versioning system for entities?**

25. **Design a notification system that stores different types of notifications efficiently.**

26. **How would you structure a multi-language content management system?**

27. **Design an inventory management system for an e-commerce platform.**

28. **How would you implement a tagging system for content categorization?**

29. **Design a session management system for a web application.**

30. **How would you architect a system for storing and querying log data?**

### Questions 31-40: Troubleshooting

31. **Your query returns empty results but you're sure the data exists. What could be wrong?**

32. **You're getting a `ClassCastException` when retrieving entities. How would you debug this?**

33. **Your application is experiencing high latency. What performance issues should you check?**

34. **Batch inserts are failing. What could be the cause?**

35. **You're getting connection timeout errors. What are the possible causes and solutions?**

36. **Your application works in development but fails in production. What should you check?**

37. **You need to debug a slow query. What tools and techniques would you use?**

38. **Your converter is not being applied. What could be wrong?**

39. **You're getting duplicate key errors. How would you handle this?**

40. **Your application is using too much memory. What could be causing this?**

### Questions 41-50: Advanced Scenarios

41. **How would you implement eventual consistency in a distributed Jakarta NoSQL application?**

42. **Design a data migration strategy from one NoSQL database to another using Jakarta NoSQL.**

43. **How would you implement data archiving for old records?**

44. **Design a system for handling schema evolution in a NoSQL database.**

45. **How would you implement full-text search capabilities?**

46. **Design a data synchronization mechanism between multiple databases.**

47. **How would you implement data validation at both the application and database levels?**

48. **Design a backup and restore strategy for NoSQL data.**

49. **How would you implement data partitioning for large datasets?**

50. **Design a monitoring and alerting system for database performance issues.**

---

## Summary & Key Takeaways

### Key Concepts

1. **Polyglot Persistence** is the practice of using different database technologies for different use cases, and Jakarta NoSQL provides a standardized way to implement this in Jakarta EE applications.

2. **Jakarta NoSQL 1.1** is the first specification developed within the Jakarta EE ecosystem, offering vendor-neutral access to NoSQL databases.

3. **Entity Mapping** uses familiar annotations (`@Entity`, `@Id`, `@Column`) similar to JPA, but requires explicit marking of all persistent fields.

4. **Java Records** provide a concise, immutable way to define entities, reducing boilerplate and improving thread safety.

5. **Template API** offers a fluent interface for CRUD operations, making database interactions intuitive and type-safe.

6. **Jakarta Query Integration** (version 1.1) enables string-based queries with named parameters, preventing injection attacks.

7. **Projections** improve performance by fetching only required fields instead of entire entities.

8. **Fluent Update Operations** (version 1.1) enable direct database updates without retrieving entities first.

9. **Auto-Apply Converters** reduce repetitive configuration by automatically applying type conversions globally.

10. **Security** is paramount — always use parameterized queries, encrypt sensitive data, and implement proper authentication/authorization.

### When to Use Jakarta NoSQL

✅ **Use Jakarta NoSQL when:**
- You need polyglot persistence (multiple database types)
- You want vendor-neutral code that can switch databases easily
- Your data model is flexible or evolving
- You need horizontal scalability
- You're building microservices with diverse data requirements
- You want to leverage modern Java features (records, etc.)

❌ **Avoid Jakarta NoSQL when:**
- You need complex transactions across multiple entities
- Your data is highly relational with many joins
- You need strong consistency guarantees
- You're working exclusively with relational databases (use JPA instead)
- Your team lacks NoSQL database experience

### Quick Decision Matrix

| Scenario | Recommended Approach |
|----------|---------------------|
| Simple CRUD operations | Repository pattern |
| Complex dynamic queries | Template API |
| Read-only reports | Projections |
| Immutable data carriers | Records |
| Mutable business entities | Traditional classes |
| Batch operations | Template API with batch methods |
| Type-safe queries | Repository with derived queries |

---

## Further Reading & Resources

### Official Documentation
- [Jakarta NoSQL Specification](https://jakarta.ee/specifications/nosql/)
- [Jakarta NoSQL 1.1 Release Notes](https://github.com/eclipse-ee4j/nosql)
- [Jakarta Query Specification](https://jakarta.ee/specifications/query/)
- [Eclipse NoSQL Implementation](https://github.com/eclipse-ee4j/nosql)

### Books
- "Jakarta EE Cookbook" by Elder Moraes
- "Enterprise Java Microservices" by Kenneth Finnigan
- "NoSQL Distilled" by Pramod J. Sadalage & Martin Fowler
- "Designing Data-Intensive Applications" by Martin Kleppmann

### Online Resources
- [DZone Jakarta NoSQL Articles](https://dzone.com/articles/jakarta-nosql-10-a-standardized-way-to-bring-java)
- [Jakarta EE Tutorials](https://eclipse-ee4j.github.io/jakartaee-tutorial/)
- [NoSQL Database Comparison](https://db-engines.com/en/ranking)

### Community
- [Jakarta EE Slack](https://jakarta.ee/slack/)
- [Eclipse NoSQL GitHub](https://github.com/eclipse-ee4j/nosql)
- [Stack Overflow - Jakarta NoSQL](https://stackoverflow.com/questions/tagged/jakarta-nosql)

### Tools & Libraries
- **Eclipse NoSQL** - Reference implementation
- **MongoDB** - Document database
- **Oracle NoSQL** - Key-value database
- **ArangoDB** - Multi-model database
- **Neo4j** - Graph database
- **Milvus** - Vector database

---

## Migration Guide: JPA to Jakarta NoSQL

### When to Migrate

Consider migrating from JPA to Jakarta NoSQL when:
- ✅ You need to support multiple database types
- ✅ Your data model is flexible or semi-structured
- ✅ You need horizontal scalability
- ✅ You're experiencing performance issues with relational databases
- ✅ You want to reduce vendor lock-in

### Migration Steps

#### Step 1: Analyze Existing JPA Entities

```java
// JPA Entity
@Entity
@Table(name = "investments")
public class Investment {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id")
    private UUID id;
    
    @Column(name = "name", nullable = false, length = 100)
    private String name;
    
    @Column(name = "amount", precision = 19, scale = 2)
    private BigDecimal amount;
    
    @Column(name = "type")
    @Enumerated(EnumType.STRING)
    private InvestmentType type;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "portfolio_id")
    private Portfolio portfolio;
    
    // Getters and setters...
}
```

#### Step 2: Convert to Jakarta NoSQL Entity

```java
// Jakarta NoSQL Entity
import jakarta.nosql.Column;
import jakarta.nosql.Entity;
import jakarta.nosql.Id;

@Entity
public record Investment(
        @Id UUID id,
        @Column String name,
        @Column BigDecimal amount,
        @Column InvestmentType type,
        @Column UUID portfolioId) {  // Store reference as ID
    
    // Compact constructor for validation
    public Investment {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name is required");
        }
    }
}
```

**Key Changes:**
- Replace `@Entity` and `@Table` with `@Entity`
- Replace `@Id` and `@GeneratedValue` with `@Id`
- Replace `@Column` with `@Column` (similar but explicit)
- Remove `@Enumerated` (enums work directly)
- Replace relationships with foreign key IDs
- Remove `@ManyToOne`, `@OneToMany`, etc. (no joins in NoSQL)

#### Step 3: Update Repository/DAO Layer

**JPA Repository:**

```java
@Repository
public interface InvestmentRepository extends JpaRepository<Investment, UUID> {
    List<Investment> findByType(InvestmentType type);
    List<Investment> findByAmountGreaterThan(BigDecimal amount);
    @Query("SELECT i FROM Investment i WHERE i.portfolio.id = :portfolioId")
    List<Investment> findByPortfolioId(@Param("portfolioId") UUID portfolioId);
}
```

**Jakarta NoSQL Service:**

```java
public class InvestmentService {
    
    private final Template template;
    
    public List<Investment> findByType(InvestmentType type) {
        return template.select(Investment.class)
                .where("type")
                .eq(type)
                .result()
                .stream()
                .toList();
    }
    
    public List<Investment> findByAmountGreaterThan(BigDecimal amount) {
        return template.select(Investment.class)
                .where("amount")
                .gt(amount)
                .result()
                .stream()
                .toList();
    }
    
    public List<Investment> findByPortfolioId(UUID portfolioId) {
        return template.select(Investment.class)
                .where("portfolioId")
                .eq(portfolioId)
                .result()
                .stream()
                .toList();
    }
}
```

#### Step 4: Handle Relationships

**JPA Approach (with joins):**

```java
@Entity
public class Portfolio {
    @Id
    private UUID id;
    
    @OneToMany(mappedBy = "portfolio", cascade = CascadeType.ALL)
    private List<Investment> investments;
}

// Usage
Portfolio portfolio = portfolioRepository.findById(id).get();
List<Investment> investments = portfolio.getInvestments();  // Automatic join
```

**Jakarta NoSQL Approach (denormalization):**

```java
@Entity
public class Portfolio {
    @Id
    private UUID id;
    
    @Column
    private String name;
    
    @Column
    private List<InvestmentSummary> investments;  // Embedded data
}

// Or separate queries
Portfolio portfolio = template.find(Portfolio.class, id).get();
List<Investment> investments = template.select(Investment.class)
        .where("portfolioId")
        .eq(portfolio.id())
        .result()
        .stream()
        .toList();
```

#### Step 5: Update Queries

**JPA Queries:**

```java
// JPQL
@Query("SELECT i FROM Investment i WHERE i.amount > :amount AND i.type = :type")
List<Investment> findHighValueInvestments(@Param("amount") BigDecimal amount, 
                                          @Param("type") InvestmentType type);

// Criteria API
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<Investment> cq = cb.createQuery(Investment.class);
Root<Investment> root = cq.from(Investment.class);
cq.select(root).where(cb.and(
    cb.gt(root.get("amount"), amount),
    cb.equal(root.get("type"), type)
));
List<Investment> results = entityManager.createQuery(cq).getResultList();
```

**Jakarta NoSQL Queries:**

```java
// Jakarta Query Language
List<Investment> results = template.query(
        "FROM Investment WHERE amount > :amount AND type = :type")
        .bind("amount", amount)
        .bind("type", type)
        .result()
        .stream()
        .toList();

// Fluent API
List<Investment> results = template.select(Investment.class)
        .where("amount")
        .gt(amount)
        .and("type")
        .eq(type)
        .result()
        .stream()
        .toList();
```

#### Step 6: Testing Migration

```java
public class MigrationTest {
    
    @Test
    void shouldMigrateInvestmentEntity() {
        // Create JPA entity
        InvestmentJpa jpaInvestment = new InvestmentJpa();
        jpaInvestment.setId(UUID.randomUUID());
        jpaInvestment.setName("Test Fund");
        jpaInvestment.setAmount(new BigDecimal("1000"));
        jpaInvestment.setType(InvestmentType.FUND);
        
        // Convert to NoSQL entity
        Investment nosqlInvestment = new Investment(
            jpaInvestment.getId(),
            jpaInvestment.getName(),
            jpaInvestment.getAmount(),
            jpaInvestment.getType(),
            jpaInvestment.getPortfolio().getId()
        );
        
        // Insert and verify
        template.insert(nosqlInvestment);
        Optional<Investment> found = template.find(Investment.class, nosqlInvestment.id());
        
        assertTrue(found.isPresent());
        assertEquals(nosqlInvestment, found.get());
    }
}
```

### Migration Checklist

- [ ] Analyze existing JPA entities and relationships
- [ ] Convert entity definitions to Jakarta NoSQL
- [ ] Replace JPA repositories with services using Template API
- [ ] Handle relationships (denormalization or separate queries)
- [ ] Update queries from JPQL to Jakarta Query or fluent API
- [ ] Implement proper error handling
- [ ] Add validation to entities
- [ ] Update tests
- [ ] Performance test with production-like data
- [ ] Deploy gradually (can run both JPA and NoSQL side-by-side)
- [ ] Monitor performance and fix issues
- [ ] Decommission JPA code after validation

### Common Migration Challenges

| Challenge | Solution |
|-----------|----------|
| **Relationships** | Denormalize data or use separate queries with foreign keys |
| **Transactions** | Use database-specific transaction support or eventual consistency |
| **Joins** | Embed related data or perform multiple queries |
| **Schema Changes** | NoSQL is flexible, but plan for backward compatibility |
| **Query Complexity** | May need to restructure data or use multiple queries |
| **Performance** | Profile and optimize based on actual usage patterns |

---

## Conclusion

Jakarta NoSQL 1.1 represents a significant advancement in enterprise Java data access, providing a standardized, vendor-neutral approach to polyglot persistence. By embracing modern Java features like records, integrating with Jakarta Query, and introducing powerful features like projections and auto-apply converters, it enables developers to build flexible, scalable applications that leverage the right database technology for each use case.

The key to success with Jakarta NoSQL is understanding when and how to use it effectively. Use this tutorial as a reference, but always consider your specific requirements, database capabilities, and performance needs when designing your data access layer.

**Remember:** The best database is the one that fits your use case. Jakarta NoSQL gives you the flexibility to choose the right tool for each job while maintaining a consistent programming model across your application.

---

**Happy Coding! 🚀**

*This tutorial was created to help you master Jakarta NoSQL 1.1 and build modern, polyglot persistence applications with Jakarta EE 12.*
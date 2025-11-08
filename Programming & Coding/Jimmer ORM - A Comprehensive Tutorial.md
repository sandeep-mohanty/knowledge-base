# Jimmer ORM: A Comprehensive Tutorial

## Introduction

Jimmer is a revolutionary modern ORM framework for Java and Kotlin that fundamentally rethinks how we interact with databases. Unlike traditional ORMs like Hibernate, Jimmer is built on **immutable data structures** and uses **compile-time code generation** to provide type-safe, high-performance database operations.[1][2][3][4]

### Key Differences from Hibernate

Coming from Hibernate, you'll notice several paradigm shifts:[5][1]

- **Immutable Entities**: Entities are defined as interfaces, not classes, enforcing immutability
- **No Lazy Loading Proxies**: Eliminates LazyInitializationException issues
- **GraphQL-Style Fetching**: Fetch exactly what you need at the call site, not pre-defined in entity annotations
- **Automatic N+1 Problem Resolution**: Jimmer automatically batches queries by design
- **Dynamic Queries**: Powerful type-safe DSL without the complexity of JPA Criteria API
- **Compile-Time Safety**: Most magic happens at compile-time via APT (Java) or KSP (Kotlin)

## Project Setup

### Maven Dependencies

```xml
<dependencies>
    <!-- Jimmer Core -->
    <dependency>
        <groupId>org.babyfish.jimmer</groupId>
        <artifactId>jimmer-spring-boot-starter</artifactId>
        <version>0.8.51</version>
    </dependency>
    
    <!-- Database Driver (MySQL example) -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Annotation Processor for Code Generation -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.10.1</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.babyfish.jimmer</groupId>
                        <artifactId>jimmer-apt</artifactId>
                        <version>0.8.51</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Gradle Dependencies

```gradle
dependencies {
    implementation 'org.babyfish.jimmer:jimmer-spring-boot-starter:0.8.51'
    
    // Annotation processor for code generation
    annotationProcessor 'org.babyfish.jimmer:jimmer-apt:0.8.51'
    
    implementation 'mysql:mysql-connector-java:8.0.33'
}
```

### Application Configuration

**application.yml**:

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/jimmer_demo
    username: root
    password: your_password

jimmer:
  # Database dialect (required)
  dialect: org.babyfish.jimmer.sql.dialect.MySqlDialect
  
  # Show generated SQL (useful for debugging)
  show-sql: true
  
  # Pretty-print SQL
  pretty-sql: true
  
  # Database validation mode
  database-validation-mode: ERROR
```

**Important Note**: After adding dependencies, **compile the project first** before writing code. Jimmer generates code at compile-time, so you may see undefined types in your IDE until the first compilation completes.[6]

## Defining Entities: The Jimmer Way

### Basic Entity Definition

Unlike Hibernate where entities are classes, Jimmer entities are **immutable interfaces**:[4][7]

```java
package com.example.model;

import org.babyfish.jimmer.sql.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
public interface Book {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    long id();
    
    String name();
    
    int edition();
    
    BigDecimal price();
    
    LocalDateTime publishedDate();
    
    @ManyToOne
    BookStore store();
    
    @ManyToMany
    List<Author> authors();
}
```

**Key Observations vs Hibernate**:[5]

1. **Interface, not class**: No implementation needed; Jimmer generates it
2. **No field declarations**: Just method signatures
3. **No fetch strategies**: No `FetchType.LAZY` or `EAGER` annotations
4. **No cascade settings**: These are specified at operation time, not entity definition time
5. **No `@Column` insertable/updatable**: Specified when performing operations

### Related Entities

```java
@Entity
public interface Author {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    long id();
    
    String firstName();
    
    String lastName();
    
    Gender gender();
    
    @ManyToMany(mappedBy = "authors")
    List<Book> books();
}

// Enum type
public enum Gender {
    MALE, FEMALE
}
```

```java
@Entity
public interface BookStore {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    long id();
    
    @Key  // Business key - unique constraint
    String name();
    
    String website();
    
    @OneToMany(mappedBy = "store")
    List<Book> books();
}
```

### Understanding @Key Annotation

The `@Key` annotation marks properties as **business keys**. When saving entities without specifying IDs, Jimmer uses business keys to determine if a record already exists:[4]

```java
@Entity
public interface TreeNode {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    long id();
    
    @Key  // Business key
    String name();
    
    @Key  // Composite business key with parent
    @ManyToOne
    TreeNode parent();
    
    @OneToMany(mappedBy = "parent")
    List<TreeNode> childNodes();
}
```

## Working with SqlClient

The `JSqlClient` (or `KSqlClient` for Kotlin) is the entry point for all database operations:[8]

```java
import org.springframework.stereotype.Service;
import org.babyfish.jimmer.sql.JSqlClient;

@Service
public class BookService {
    
    private final JSqlClient sqlClient;
    
    // Spring Boot Starter auto-configures and injects SqlClient
    public BookService(JSqlClient sqlClient) {
        this.sqlClient = sqlClient;
    }
    
    // Use sqlClient for all database operations
}
```

## CRUD Operations

### Creating (Insert)

**Simple Insert**:

```java
public Book createBook(String name, BigDecimal price) {
    Book book = Objects.createBook(draft -> {
        draft.setName(name);
        draft.setEdition(1);
        draft.setPrice(price);
        draft.setPublishedDate(LocalDateTime.now());
    });
    
    return sqlClient.save(book);
}
```

**Insert with Associations**:

```java
public Book createBookWithStore(String bookName, long storeId) {
    Book book = Objects.createBook(draft -> {
        draft.setName(bookName);
        draft.setEdition(1);
        draft.setPrice(new BigDecimal("29.99"));
        
        // Associate with existing store by ID
        draft.setStore(
            Objects.createBookStore(store -> store.setId(storeId))
        );
    });
    
    return sqlClient.save(book);
}
```

**Deep Insert (Save Object Graph)**:

```java
public BookStore createStoreWithBooks() {
    BookStore store = Objects.createBookStore(draft -> {
        draft.setName("Tech Books Store");
        draft.setWebsite("https://techbooks.com");
        
        // Add books inline
        draft.addIntoBooks(book -> {
            book.setName("Spring Boot in Action");
            book.setEdition(1);
            book.setPrice(new BigDecimal("39.99"));
        });
        
        draft.addIntoBooks(book -> {
            book.setName("Effective Java");
            book.setEdition(3);
            book.setPrice(new BigDecimal("45.00"));
        });
    });
    
    // Jimmer automatically handles the entire object graph
    return sqlClient.save(store);
}
```

### Reading (Query)

**Find by ID**:

```java
public Book findById(long id) {
    return sqlClient.findById(Book.class, id);
}
```

**Simple Query**:

```java
public List<Book> findBooksByName(String keyword) {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .where(table.name().ilike("%" + keyword + "%"))
        .select(table)
        .execute();
}
```

**Query with Conditions**:

```java
public List<Book> findBooksByPriceRange(BigDecimal min, BigDecimal max) {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .where(table.price().between(min, max))
        .orderBy(table.price().desc())
        .select(table)
        .execute();
}
```

**Query with Joins**:

```java
public List<Book> findBooksWithStoreName(String storeName) {
    BookTable book = Tables.BOOK_TABLE;
    BookStoreTable store = Tables.BOOK_STORE_TABLE;
    
    return sqlClient
        .createQuery(book)
        .where(book.store().name().eq(storeName))
        .select(book)
        .execute();
}
```

### Updating

**Update Entity**:

```java
public Book updateBookPrice(long bookId, BigDecimal newPrice) {
    Book book = Objects.createBook(draft -> {
        draft.setId(bookId);  // Must specify ID
        draft.setPrice(newPrice);
    });
    
    // Only updates specified fields
    return sqlClient.update(book);
}
```

**Conditional Update**:

```java
public int updatePricesByStore(long storeId, BigDecimal increment) {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createUpdate(table)
        .set(table.price(), table.price().plus(increment))
        .where(table.store().id().eq(storeId))
        .execute();
}
```

### Deleting

**Delete by ID**:

```java
public void deleteBook(long id) {
    sqlClient.deleteById(Book.class, id);
}
```

**Conditional Delete**:

```java
public int deleteExpensiveBooks(BigDecimal threshold) {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createDelete(table)
        .where(table.price().gt(threshold))
        .execute();
}
```

## Object Fetchers: GraphQL-Style Data Shaping

One of Jimmer's most powerful features is **Object Fetchers**, which allow you to specify exactly what data to fetch at the call site:[2][3]

### Basic Fetcher

```java
public List<Book> findBooksWithScalarFields() {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .select(
            table.fetch(
                Fetchers.BOOK_FETCHER
                    .allScalarFields()  // All primitive fields
            )
        )
        .execute();
}
```

### Fetcher with Associations

```java
public List<Book> findBooksWithStore() {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .select(
            table.fetch(
                Fetchers.BOOK_FETCHER
                    .allScalarFields()
                    .store(  // Fetch associated store
                        Fetchers.BOOK_STORE_FETCHER
                            .allScalarFields()
                    )
            )
        )
        .execute();
}
```

**Generated SQL**: Jimmer automatically generates an optimized JOIN query, **eliminating the N+1 problem**.[1]

### Deep Nested Fetching

```java
public List<Book> findBooksWithStoreAndAuthors() {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .where(table.price().between(
            new BigDecimal("20"),
            new BigDecimal("50")
        ))
        .select(
            table.fetch(
                Fetchers.BOOK_FETCHER
                    .allScalarFields()
                    .store(
                        Fetchers.BOOK_STORE_FETCHER
                            .allScalarFields()
                    )
                    .authors(
                        Fetchers.AUTHOR_FETCHER
                            .firstName()
                            .lastName()
                    )
            )
        )
        .execute();
}
```

**This is fundamentally different from Hibernate**: In Hibernate, you'd define fetch strategies in entity annotations (`FetchType.LAZY/EAGER`, `@Fetch`). In Jimmer, **you specify what to fetch when you query**, giving you complete control.[5]

### Recursive Fetching

For self-referencing entities like tree structures:

```java
public TreeNode findNodeWithChildren(long id) {
    return sqlClient.findById(
        Fetchers.TREE_NODE_FETCHER
            .allScalarFields()
            .recursiveChildNodes(),  // Recursively fetch all children
        id
    );
}

// With depth limit
public TreeNode findNodeWithChildrenDepth(long id, int depth) {
    return sqlClient.findById(
        Fetchers.TREE_NODE_FETCHER
            .allScalarFields()
            .recursiveChildNodes(depth),  // Max depth
        id
    );
}
```

## Dynamic Queries

Jimmer's type-safe DSL makes building dynamic queries elegant:[2][1]

```java
public List<Book> searchBooks(
    String name,
    BigDecimal minPrice,
    BigDecimal maxPrice,
    Long storeId
) {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .where(table.name().ilikeIf(name))  // Only if name != null
        .where(table.price().geIf(minPrice))  // Only if minPrice != null
        .where(table.price().leIf(maxPrice))  // Only if maxPrice != null
        .where(table.store().id().eqIf(storeId))  // Only if storeId != null
        .select(table)
        .execute();
}
```

The `...If` methods automatically handle null values, making dynamic queries clean and safe.

### Complex Dynamic Queries

```java
public List<Book> advancedSearch(BookSearchCriteria criteria) {
    BookTable book = Tables.BOOK_TABLE;
    AuthorTable author = Tables.AUTHOR_TABLE;
    
    return sqlClient
        .createQuery(book)
        .where(book.name().ilikeIf(criteria.getName()))
        .where(book.price().betweenIf(criteria.getMinPrice(), criteria.getMaxPrice()))
        .whereIf(
            criteria.getAuthorName() != null,
            () -> book.id().in(
                sqlClient.createSubQuery(author)
                    .where(author.firstName().ilike(criteria.getAuthorName()))
                    .select(author.books().id())
            )
        )
        .orderBy(book.price().desc())
        .limit(criteria.getPageSize(), criteria.getOffset())
        .select(
            table.fetch(
                Fetchers.BOOK_FETCHER
                    .allScalarFields()
                    .store(Fetchers.BOOK_STORE_FETCHER.name())
            )
        )
        .execute();
}
```

## Pagination

```java
public Page<Book> findBooksPage(int pageNumber, int pageSize) {
    BookTable table = Tables.BOOK_TABLE;
    
    return sqlClient
        .createQuery(table)
        .orderBy(table.name())
        .select(table)
        .fetchPage(pageNumber, pageSize);  // Returns Page<Book>
}
```

The `Page` object contains:
- `rows`: List of results
- `totalRowCount`: Total count (separate COUNT query)
- `totalPageCount`: Calculated total pages

## Advanced Features

### Batch Loading (Solves N+1 Problem)

Jimmer **automatically** optimizes queries to prevent N+1 problems:[1]

```java
// This code does NOT cause N+1!
List<Book> books = sqlClient.findAll(Book.class);

for (Book book : books) {
    // Accessing store doesn't trigger N queries
    System.out.println(book.store().name());
}
```

Jimmer batches the store fetches into a single `WHERE id IN (...)` query.

### Save Commands (Advanced Saving)

Fine-grained control over save behavior:

```java
public Book saveBookWithControl(Book book) {
    return sqlClient
        .getEntities()
        .saveCommand(book)
        .setMode(SaveMode.UPSERT)  // Insert or update
        .setAssociatedModeAll(AssociatedSaveMode.REPLACE)  // Replace associations
        .execute()
        .getModifiedEntity();
}
```

### Caching

Jimmer supports multiple cache levels:[2]

```yaml
jimmer:
  cache:
    type: redis
    redis:
      host: localhost
      port: 6379
```

```java
@Configuration
public class CacheConfig {
    
    @Bean
    public CacheFactory cacheFactory(RedisConnectionFactory factory) {
        return new CacheFactory() {
            @Override
            public Cache<?, ?> createObjectCache(ImmutableType type) {
                return new ChainCacheBuilder<>()
                    .add(new CaffeineBinder<>(512, Duration.ofMinutes(1)))
                    .add(new RedisValueBinder<>(factory, type, Duration.ofHours(1)))
                    .build();
            }
            
            // ... other cache types
        };
    }
}
```

### Triggers (Audit Events)

Monitor and react to entity changes:

```java
@Component
public class BookTrigger implements Trigger<Book> {
    
    @Override
    public void onChange(EntityEvent<Book> e) {
        if (e.getEventType() == EntityEvent.INSERTED) {
            System.out.println("New book created: " + e.getNewEntity());
        } else if (e.getEventType() == EntityEvent.UPDATED) {
            System.out.println("Book updated from " + e.getOldEntity() 
                + " to " + e.getNewEntity());
        }
    }
}
```

## Complete Service Example

```java
@Service
@Transactional
public class BookService {
    
    private final JSqlClient sqlClient;
    
    public BookService(JSqlClient sqlClient) {
        this.sqlClient = sqlClient;
    }
    
    public Book createBook(CreateBookInput input) {
        Book book = Objects.createBook(draft -> {
            draft.setName(input.getName());
            draft.setEdition(input.getEdition());
            draft.setPrice(input.getPrice());
            draft.setPublishedDate(LocalDateTime.now());
            
            if (input.getStoreId() != null) {
                draft.setStore(
                    Objects.createBookStore(store -> 
                        store.setId(input.getStoreId())
                    )
                );
            }
            
            if (input.getAuthorIds() != null) {
                for (Long authorId : input.getAuthorIds()) {
                    draft.addIntoAuthors(author -> 
                        author.setId(authorId)
                    );
                }
            }
        });
        
        return sqlClient.save(book);
    }
    
    public List<Book> searchBooks(BookSearchInput input) {
        BookTable table = Tables.BOOK_TABLE;
        
        return sqlClient
            .createQuery(table)
            .where(table.name().ilikeIf(input.getName()))
            .where(table.price().betweenIf(
                input.getMinPrice(), 
                input.getMaxPrice()
            ))
            .where(table.store().id().eqIf(input.getStoreId()))
            .orderBy(table.name())
            .select(
                table.fetch(
                    Fetchers.BOOK_FETCHER
                        .allScalarFields()
                        .store(
                            Fetchers.BOOK_STORE_FETCHER
                                .name()
                                .website()
                        )
                        .authors(
                            Fetchers.AUTHOR_FETCHER
                                .firstName()
                                .lastName()
                        )
                )
            )
            .execute();
    }
    
    public Book updateBook(long id, UpdateBookInput input) {
        Book book = Objects.createBook(draft -> {
            draft.setId(id);
            
            if (input.getName() != null) {
                draft.setName(input.getName());
            }
            if (input.getPrice() != null) {
                draft.setPrice(input.getPrice());
            }
            if (input.getEdition() != null) {
                draft.setEdition(input.getEdition());
            }
        });
        
        return sqlClient.update(book);
    }
    
    public void deleteBook(long id) {
        sqlClient.deleteById(Book.class, id);
    }
    
    public Page<Book> findBooksPage(int page, int size) {
        BookTable table = Tables.BOOK_TABLE;
        
        return sqlClient
            .createQuery(table)
            .orderBy(table.publishedDate().desc())
            .select(
                table.fetch(
                    Fetchers.BOOK_FETCHER
                        .allScalarFields()
                        .store(Fetchers.BOOK_STORE_FETCHER.name())
                )
            )
            .fetchPage(page, size);
    }
}
```

## Jimmer vs Hibernate: Detailed Comparison

### Architecture Differences

| Aspect | Hibernate | Jimmer |
|--------|-----------|--------|
| **Entity Definition** | Mutable classes with fields | Immutable interfaces with methods |
| **Code Generation** | Runtime proxies | Compile-time code generation (APT/KSP) |
| **Data Structure** | Mutable, stateful | Immutable, functional |
| **Lazy Loading** | Runtime proxies (LazyInitializationException risk) | No proxies; explicit fetching |
| **Fetch Strategy** | Defined in entity annotations | Defined at query time (Object Fetchers) |
| **N+1 Problem** | Requires manual optimization (@EntityGraph, JOIN FETCH) | Automatically prevented by design |
| **Query DSL** | JPA Criteria API (complex) | Type-safe, fluent DSL (simple) |
| **Dynamic Queries** | Criteria API or QueryDSL | Built-in type-safe DSL with `...If` methods |
| **Session/Context** | Session-bound entities, detached state issues | Immutable entities, no session concept |

### Performance Comparison

**N+1 Query Example**:

**Hibernate** (without optimization):
```java
// Triggers N+1 queries
List<Book> books = session.createQuery("FROM Book", Book.class).list();
for (Book book : books) {
    book.getStore().getName();  // Each access = 1 query
}
```

**Jimmer**:
```java
// Automatically batched into 2 queries total
List<Book> books = sqlClient
    .createQuery(Tables.BOOK_TABLE)
    .select(
        table.fetch(
            Fetchers.BOOK_FETCHER
                .allScalarFields()
                .store(Fetchers.BOOK_STORE_FETCHER.name())
        )
    )
    .execute();
```

### State Management

**Hibernate**: 
- Entities have persistence context state (transient, persistent, detached, removed)
- "Dirty entities" blur persistence and business logic boundaries[1]
- Requires careful session management

**Jimmer**:
- Entities are always immutable
- No persistence context or state management
- Clean separation between data and operations

### When to Use Each

## When to Use Jimmer Over Hibernate

### Best Use Cases for Jimmer

**1. GraphQL APIs / REST APIs with Dynamic Responses**[3][2]

Jimmer's Object Fetchers are perfect when different API endpoints need different data shapes:

```java
// Different endpoints, different data shapes
// Endpoint 1: Books with just store names
GET /api/books/summary

// Endpoint 2: Books with full store details and authors
GET /api/books/details
```

With Jimmer, define the fetch shape per endpoint without multiple entity projections.

**2. High-Performance Requirements**[1]

Use Jimmer when:
- You need performance close to raw JDBC/JOOQ
- N+1 problems are a recurring issue in your codebase
- You want automatic query optimization

**3. Complex Object Graphs**[7]

When dealing with deeply nested relationships:
- Recursive structures (trees, hierarchies)
- Many-to-many relationships with complex fetching needs
- Dynamic nested data requirements

**4. Immutable Domain Models**[4]

If your architecture emphasizes:
- Functional programming principles
- Immutable data structures
- Thread-safe concurrent operations
- CQRS or Event Sourcing patterns

**5. Type-Safe Dynamic Queries**[1]

When you need:
- Complex search filters with many optional parameters
- Type-safe queries without strings
- Better alternative to JPA Criteria API

**6. Microservices with Varying Data Needs**

When different services need different views of the same data without creating multiple DTOs manually.

### When to Stick with Hibernate

**1. Existing Large Codebases**

Migration cost may be too high for established Hibernate applications.

**2. Team Familiarity**

If your team is deeply familiar with JPA/Hibernate patterns and the learning curve is a concern.

**3. JPA Portability Requirements**

When you need true JPA provider portability (Hibernate, EclipseLink, etc.).

**4. Enterprise Framework Dependencies**

Some enterprise frameworks are tightly coupled to JPA (e.g., Spring Data JPA repositories).

**5. Simple CRUD Applications**

For basic CRUD without complex querying or object graph requirements, Hibernate's maturity and ecosystem may be sufficient.

## Migration Considerations

### Gradual Adoption

You can use Jimmer alongside Hibernate:

```java
@Service
public class HybridService {
    
    private final EntityManager entityManager;  // Hibernate
    private final JSqlClient sqlClient;  // Jimmer
    
    // Use Hibernate for simple CRUD
    // Use Jimmer for complex queries
}
```

### Learning Curve

**Estimated Learning Time**:
- Basic usage: 1-2 days (if familiar with ORMs)
- Advanced features: 1-2 weeks
- Proficiency: 1-2 months

**Key Concepts to Master**:
1. Immutable entities and the Draft API
2. Object Fetchers
3. Type-safe query DSL
4. Compile-time code generation setup

## Summary

Jimmer represents a paradigm shift in JVM ORM development:[2][1]

- **Immutable by design**: Eliminates entire classes of bugs
- **GraphQL-style fetching**: Fetch exactly what you need, when you need it
- **Automatic optimization**: N+1 problems solved by design
- **Type-safe DSL**: Dynamic queries without sacrificing type safety
- **High performance**: Comparable to raw SQL frameworks like JOOQ

For new projects with complex querying needs, dynamic data requirements, or performance concerns, Jimmer offers compelling advantages over traditional JPA/Hibernate. However, for simple applications or teams deeply invested in JPA, Hibernate remains a solid choice.

The future of Jimmer looks promising as it continues to gain adoption in the Java community, particularly in scenarios where immutability, type safety, and performance are critical.[9][1]

## Additional Resources

- **Official Documentation**: https://babyfish-ct.github.io/jimmer-doc/
- **GitHub Repository**: https://github.com/babyfish-ct/jimmer
- **Official Website**: https://jimmer.org
- **Discord Community**: https://discord.gg/PmgR5mpY3E
- **Example Projects**: Available in the GitHub repository

***

**Note**: Remember to always compile your project after defining entities to generate the necessary code (Tables, Fetchers, Draft APIs, etc.).[6]

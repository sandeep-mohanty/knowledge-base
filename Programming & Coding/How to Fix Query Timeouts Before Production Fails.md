# Why Query Timeouts Happen in Java And How to Fix Them Before Production Fails

![Query Timeout Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*vRVASgGNLica_RFWkyIZzA.png)

## 1️ What is Query Timeout?
A **Query Timeout** occurs when a database query takes longer than the maximum allowed time to execute, so the system **automatically stops (kills) the query** and throws an exception.

### Simple Definition
Query Timeout = Query took too long → system cancels it → exception thrown.

### Why systems have timeouts?
Without a timeout:
* Slow queries would block threads forever
* Application performance would drop
* Thread pools could exhaust
* Users would see infinite loading
* Database would overload

So **timeouts protect both the application and the DB**.

## 2️ When does a Query Timeout happen?
Typical cases:
1. **Slow SQL queries**
   Large joins, missing indexes, too much data scanning.
2. **Database server is slow**
   High CPU / low memory / too many connections.
3. **Network latency**
4. **Locking issues in DB**
   Another query is holding a row/table lock.
5. **Application timeout settings too small**

## 3️ What exception do we get?
* **Hibernate/JPA**
  `javax.persistence.QueryTimeoutException`
* **Spring JDBC**
  `org.springframework.dao.QueryTimeoutException`
* **Java JDBC**
  `java.sql.SQLTimeoutException`

## 4️ Query Timeout in Java JDBC (Plain Java Example)
Set timeout using JDBC Statement
```java
import java.sql.*;

public class QueryTimeoutExample {
    public static void main(String[] args) throws Exception {

        Connection connection = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/testdb",
                "root",
                "password"
        );

        Statement stmt = connection.createStatement();

        stmt.setQueryTimeout(5); // Timeout = 5 seconds

        try {
            ResultSet rs = stmt.executeQuery("SELECT * FROM big_table");  // Slow query
        } catch (SQLTimeoutException e) {
            System.out.println("Query Timeout Occurred: " + e.getMessage());
        }
    }
}
```

### Explanation
* `setQueryTimeout(5)` → JDBC will wait only **5 seconds**
* If the query is still running → JDBC throws `SQLTimeoutException`

## 5️ Query Timeout in Spring Boot (JPA/Hibernate)
### Option 1: Set Timeout in @Query
```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Query(value = "SELECT * FROM product p WHERE p.category = :category", 
           nativeQuery = true,
           timeout = 3) // Timeout in seconds
    List<Product> findByCategory(String category);
}
```

### Option 2: Set Timeout using Transactional
```java
@Transactional(timeout = 5) // 5 seconds
public List<Product> getAllProducts() {
    return productRepository.findAll();
}
```
If query takes more than 5 seconds → Spring throws `QueryTimeoutException`.

### Option 3: Set Global Hibernate Timeout
Add to `application.properties`:
```properties
spring.jpa.properties.hibernate.jdbc.timeout=5
spring.jpa.properties.hibernate.query.timeout=5
```

## 6️ Query Timeout using Spring JdbcTemplate
```java
@Autowired
private JdbcTemplate jdbcTemplate;

public void getRecords() {
    jdbcTemplate.setQueryTimeout(4); // seconds

    try {
        jdbcTemplate.queryForList("SELECT * FROM big_table");
    } catch (QueryTimeoutException e) {
        System.out.println("Query timed out: " + e.getMessage());
    }
}
```

## 7️ Real-Time Example: Why Query Timeout Is Important?
### Scenario: E-commerce Order System
You have a table:
`orders` — 10 million rows

If a customer searches for their order history:
`SELECT * FROM orders WHERE customer_id = 123;`
This query **must be fast**.

### Problems that can cause timeouts:
* No index on `customer_id`
* DB high load
* Query scanning entire table
* Too many users running queries at same time

### To protect system
`@Transactional(timeout = 3)` // fail after 3 seconds
`public List<Order> getOrders() { ... }`
* Prevents thread blocking
* Prevents server overload
* Prevents customer waiting forever

## 8️ How to Prevent Query Timeout? (Important for Interviews)
1. **Optimize SQL queries**
   * Add missing indexes
   * Avoid `SELECT *`
   * Avoid heavy joins
   * Use pagination (LIMIT)
2. **Increase timeout if query is valid but slow**
3. **Make database faster**
   * Tune database buffers
   * Upgrade CPU or RAM
4. **Use async queries (CompletableFuture, @Async)**
5. **Reduce thread blocking using connection pooling**
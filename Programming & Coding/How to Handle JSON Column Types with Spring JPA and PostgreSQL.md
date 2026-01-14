# How to Handle JSON Column Types with Spring JPA and PostgreSQL: A Complete Guide

In modern applications, JSON columns have become a popular choice for storing semi-structured data such as user preferences, dynamic configurations, and nested attributes. They offer great flexibility without requiring frequent schema changes.

However, working with JSON columns in Java using Spring JPA isn’t always straightforward. If you’ve ever found yourself manually converting JSON strings to Java objects using libraries like `ObjectMapper` or `Gson`, there’s a much simpler approach.

In this article, we’ll explore how to efficiently map and query JSON columns in PostgreSQL using **Spring JPA** and the **Hypersistence Utils** library.

### Why Use JSON Columns?
Imagine you have a box to store all kinds of items — books, pens, or cables — without labeling specific compartments. That’s how JSON columns work: they let you store flexible data without changing your database table structure every time.

**Real-life use:** Using `UserSettings` to store user settings (language, darkMode, dashboardLayout) — Saving API responses in the database — Handling dynamic configurations for each user.

**PostgreSQL supports:**
* **JSON:** stores text exactly as given
* **JSONB:** optimized for searching and indexing (we’ll use JSONB because it’s faster for queries)

---

### Getting Started

#### Step 1: Create a Spring Boot Project
Creating a web application with Spring Initializr is very easy. It’s a helpful tool to quickly set up Spring Boot projects.

![Spring Initializr Configuration](https://miro.medium.com/v2/resize:fit:720/format:webp/1*YkGoglEV1ieUnK9IONSEdg.png)

#### Step 2: Add Required Dependencies
Add the following dependencies in your `pom.xml` inside `<dependencies>` section.

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>io.hypersistence</groupId>
        <artifactId>hypersistence-utils-hibernate-60</artifactId>
        <version>3.9.0</version>
    </dependency>

    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

#### Step 3: Configure Database Connection
In `src/main/resources/application.yml` (or application.properties), add your PostgreSQL details:

```yaml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/postgres
    username: postgres
    password: admin@123
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      ddl-auto: update
    show-sql: true
```
**Note:** Change `username` and `password` to your PostgreSQL credentials.

#### Step 4: Create UserSettings POJO
Create a Java record (or class) to represent your JSON structure:

```java
public record UserSettings(
    String language,
    boolean darkMode,
    Map<String, Object> dashboardLayout
) {}
```

#### Step 5: Create JPA Entity with JSONB Column
Now, let’s define the `CustomerEntity` and map the `user_settings` column as a JSONB type:

```java
import io.hypersistence.utils.hibernate.type.json.JsonBinaryType;
import org.hibernate.annotations.Type;
import org.hibernate.annotations.JdbcTypeCode;
import org.hibernate.type.SqlTypes;

import jakarta.persistence.*;

@Entity
@Table(name = "customer")
public class CustomerEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id_customer")
    private Long id;

    @Column(nullable = false, unique = true)
    private String name;

    @Type(JsonBinaryType.class)
    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "user_settings", columnDefinition = "jsonb")
    private UserSettings userSettings;

    // Getters and setters
    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public UserSettings getUserSettings() {
        return userSettings;
    }

    public void setUserSettings(UserSettings userSettings) {
        this.userSettings = userSettings;
    }
}
```

#### Step 6: Create JPA Repository
Define the repository interface to handle database operations:

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CustomerRepository extends JpaRepository<CustomerEntity, Long> {
}
```

#### Step 7: Create REST Controller
Expose endpoints to save and retrieve customers with JSON-based settings:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Optional;

@RestController
@RequestMapping("/api/customers")
public class CustomerController {

    @Autowired
    private CustomerRepository customerRepository;

    @PostMapping
    public CustomerEntity createCustomer(@RequestBody CustomerEntity customer) {
        return customerRepository.save(customer);
    }

    @GetMapping("/{id}")
    public ResponseEntity<CustomerEntity> getCustomer(@PathVariable Long id) {
        Optional<CustomerEntity> customer = customerRepository.findById(id);
        return customer.map(ResponseEntity::ok)
                       .orElseGet(() -> ResponseEntity.notFound().build());
    }
}
```

#### Step 8: Test Your Application
Run your Spring Boot app and use **Postman** or **curl** to test the endpoints.

**POST** `/api/customers` with JSON payload including preferences.

```json
// Sample JSON request
{
  "name": "Sunil",
  "userSettings": {
    "language": "en",
    "darkMode": true,
    "dashboardLayout": {
      "widgets": ["sales", "activity", "notifications"],
      "refreshInterval": 5
    }
  }
}
```

**GET** `/api/customers/{id}` to fetch saved data.

```json
// GET http://localhost:8080/api/customers/1
{
  "id": 1,
  "name": "Sunil",
  "userSettings": {
    "language": "en",
    "darkMode": true,
    "dashboardLayout": {
      "widgets": ["sales", "activity", "notifications"],
      "refreshInterval": 5
    }
  }
}
```

---

Using **PostgreSQL JSONB** columns with **Spring JPA** is a great way to store flexible and nested data like user settings without changing your database structure often. With the **Hypersistence Utils** library, you can easily map **JSON** data to Java objects such as **UserSettings**, without writing manual converters.

This approach helps you build simple yet powerful applications that can handle changing data structures smoothly. Just remember to use JSONB columns wisely and keep your main tables organized for better performance.
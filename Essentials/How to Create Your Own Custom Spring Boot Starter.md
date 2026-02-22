# How to Create Your Own Custom Spring Boot Starter

The Spring ecosystem has transformed Java development by offering powerful tools that simplify application setup and configuration. One of its most useful features is Spring Starters — pre-configured libraries that remove the need for manual setup when adding new technologies or functionalities. With Spring Starters, developers can spend less time managing configurations and more time building features.

Custom Spring Starters take this idea further. They allow you to package reusable components designed for your organization’s or project’s specific needs. By creating a custom starter, you can make onboarding easier, enforce best practices, and simplify integration with tools or services such as AWS Secret Manager, databases, or logging systems.

In this article, we’ll explore the benefits of Spring Starters, build a custom Spring Starter for AWS Secret Manager, and demonstrate how to use it in another application.

## Key Advantages of Spring Starters
Spring Starters make working with new technologies in a Spring Boot project much easier. They save time, reduce mistakes, and create consistency across applications.

Main benefits:

- Ease of use: Simplifies the integration of technologies, reducing boilerplate code and configuration.
- Standardization: Enforces consistent project structure and best practices.
- Reusability: Packages common configurations into reusable dependencies.
- Reduced errors: Handles complex setups internally to avoid misconfigurations.
- Faster onboarding: New developers can start quickly with minimal learning curve.
- Easy maintenance: Updates made in one starter automatically apply across all projects using it.

## Examples from the Spring Ecosystem
Spring Boot provides many built-in starters to integrate popular technologies easily:

- `spring-boot-starter-web`: Used to build web applications, including RESTful services. It includes an embedded Tomcat server and other essential dependencies.
- `spring-boot-starter-data-jpa`: Simplifies database access with JPA and Hibernate.
- `spring-boot-starter-security`: Adds authentication and authorization features for securing applications.
- `spring-boot-starter-test`: Includes testing libraries like JUnit and Mockito.
- `spring-boot-starter-actuator`: Adds production-ready features like health checks, monitoring, and metrics.
- `spring-boot-starter-thymeleaf`: Helps in developing server-side rendered web pages using the Thymeleaf templating engine.
- `spring-boot-starter-cloud-aws`: Helps connect AWS services easily. Facilitates integration with AWS services such as S3, SNS, and SQS.

These starters demonstrate the flexibility and power of the Spring ecosystem, enabling developers to seamlessly integrate a wide range of functionalities with minimal setup.

---

## Creating a Custom Spring Starter for Database Connection Pooling

### Use Case
Imagine you have multiple microservices that each need to connect to a database like PostgreSQL or MySQL. Every service typically requires:

- Adding a database driver dependency
- Configuring connection pooling (e.g., with HikariCP)
- Managing properties such as URL, username, password, and pool size

Doing this setup repeatedly in each project leads to duplicated code, inconsistent configurations, and complicated maintenance.

A custom Spring Starter solves this by packaging the entire database connection pool setup into one reusable module. This ensures consistency, simplifies onboarding new projects or developers, and centralizes maintenance.

---

### Solution: db-connection-starter

#### Setting Up the Project
Let’s start by creating a new Maven project for the custom Spring Starter.

```
db-connection-starter/
 ├── src/
 │   └── main/
 │       ├── java/
 │       │   └── com/example/dbstarter/
 │       │       ├── config/
 │       │       │   └── DatabaseAutoConfiguration.java
 │       │       └── properties/
 │       │           └── DatabaseProperties.java
 │       └── resources/
 │           └── META-INF/
 │               └── spring.factories
 └── pom.xml
```

Add this to your `pom.xml` for dependencies:

```xml
<project ...>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>db-connection-starter</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <properties>
    <java.version>21</java.version>
    <spring.boot.version>3.4.1</spring.boot.version>
  </properties>

  <dependencies>
    <!-- Spring Boot auto-configure -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
      <version>${spring.boot.version}</version>
    </dependency>

    <!-- Configuration processor for property metadata -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <version>${spring.boot.version}</version>
      <optional>true</optional>
    </dependency>

    <!-- HikariCP connection pool -->
    <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>5.1.0</version>
    </dependency>

    <!-- Logging API -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>2.0.16</version>
    </dependency>

    <!-- Lombok for boilerplate -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.36</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
</project>
```

---

#### 2. Creating the Properties Class

```java
// DatabaseProperties.java
package com.example.dbstarter.properties;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Data
@ConfigurationProperties(prefix = "custom.db")
public class DatabaseProperties {
    private boolean enabled = true;
    private String url;
    private String username;
    private String password;
    private int maxPoolSize = 10;
}
```

---

#### 3. Creating the Auto-Configuration Class

```java
// DatabaseAutoConfiguration.java
package com.example.dbstarter.config;

import com.example.dbstarter.properties.DatabaseProperties;
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import jakarta.annotation.PostConstruct;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Slf4j
@Configuration
@RequiredArgsConstructor
@EnableConfigurationProperties(DatabaseProperties.class)
@ConditionalOnProperty(prefix = "custom.db", name = "enabled", havingValue = "true", matchIfMissing = true)
public class DatabaseAutoConfiguration {

    private final DatabaseProperties props;

    @PostConstruct
    public void init() {
        log.info("Database Connection Starter initialized. URL={}", props.getUrl());
    }

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(props.getUrl());
        config.setUsername(props.getUsername());
        config.setPassword(props.getPassword());
        config.setMaximumPoolSize(props.getMaxPoolSize());
        return new HikariDataSource(config);
    }
}
```
#### 4. Registering the Auto-Configuration

Create a `spring.factories` file at `src/main/resources/META-INF/spring.factories` with this content:

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.dbstarter.config.DatabaseAutoConfiguration
```

---

#### 5. Publishing to Maven Local

```bash
mvn clean install
```

This will allow you to use the starter in any other Spring Boot project on your machine.

---

## Using the Custom Starter in Another Application

#### 1. Create a New Spring Boot Application
In your microservice or test project, include the dependency:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>db-connection-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

---

#### 2. Configure Properties
Add database configuration in `application.yml`:

```yaml
spring:
  application:
    name: example-db-app

custom:
  db:
    enabled: true
    url: jdbc:postgresql://localhost:5432/testdb
    username: postgres
    password: secret
    max-pool-size: 20
```

---

#### 3. Using the DataSource
Now, simply inject and use the configured `DataSource` in your Spring Boot app:

```java
package com.example.dbapp;

import jakarta.annotation.PostConstruct;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import javax.sql.DataSource;
import java.sql.Connection;

@Slf4j
@SpringBootApplication
@RequiredArgsConstructor
public class ExampleDbAppApplication {

    private final DataSource dataSource;

    public static void main(String[] args) {
        SpringApplication.run(ExampleDbAppApplication.class, args);
    }

    @PostConstruct
    public void testConnection() throws Exception {
        try (Connection connection = dataSource.getConnection()) {
            log.info("Database connection established successfully: {}", connection.getMetaData().getURL());
        }
    }
}
```

---

### Expected Console Output

```
2025-01-24 10:15:03 INFO  ExampleDbAppApplication - Database Connection Starter initialized. URL=jdbc:postgresql://localhost:5432/testdb
2025-01-24 10:15:03 INFO  ExampleDbAppApplication - Database connection established successfully: jdbc:postgresql://localhost:5432/testdb
```

---

## Conclusion
Creating a custom Spring Starter for database connection pooling is an effective way to standardize and centralize your data access configuration across microservices.

With this setup, you:

- Remove repetitive configuration  
- Ensure consistent setup across all microservices  
- Simplify maintenance and onboarding  

While this example uses HikariCP and PostgreSQL, the same approach works for any database or connection pool. By leveraging Spring Boot’s auto-configuration and modular design, you build reusable starters that keep your ecosystem clean and manageable.

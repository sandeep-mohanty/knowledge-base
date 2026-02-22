# How to Configure Multiple Databases in Spring Boot Using Spring JPA

**Code With Sunil** | Code Smarter, not harder  
Follow  
5 min read · 2 days ago  

Learn how to set up multiple data sources, entity managers, and repositories with Spring Data JPA.



If you are not a Member — [Read for free here](https://medium.com/@sunil.code/how-to-configure-multiple-databases-in-spring-boot-using-spring-jpa-0f3a2b3c4d5e)

Sometimes, we build applications that need to work with **multiple databases**. This usually happens when different types of data have different purposes, performance needs, or ownership. Instead of storing everything in a single database, we may choose to separate the data across multiple databases.

In this article, we will build a **simple Spring Boot application** to demonstrate how to configure **Spring Data JPA with multiple databases**.

We will create:
* A database named **book-database** to store information related to Books.
* A separate database named **author-database** to store information related to Authors.

By the end of this article, you will understand how to configure multiple data sources, entity managers, and repositories in Spring Boot using Spring Data JPA.

---

## Steps to Configure Multiple DataSources in a Spring Boot Application

Below are the steps to configure **multiple DataSources** in a Spring Boot application using **Spring Data JPA** and **PostgreSQL**.

### Step 1: Configure the application.properties File
First, we need to configure the database connection properties for both databases.

**application.properties**
```properties
# ===============================
# Primary Database (Book Database)
# ===============================
spring.datasource.url=jdbc:postgresql://localhost:5432/book_database
spring.datasource.username=postgres
spring.datasource.password=your_password
spring.datasource.driver-class-name=org.postgresql.Driver

# ===============================
# Secondary Database (Author Database)
# ===============================
spring.author-datasource.url=jdbc:postgresql://localhost:5432/author_database
spring.author-datasource.username=postgres
spring.author-datasource.password=your_password
spring.author-datasource.driver-class-name=org.postgresql.Driver

# ===============================
# JPA Configuration
# ===============================
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### Step 2: Configure Dependencies in pom.xml
Now, we will add the required dependencies to the `pom.xml` file.

**pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="https://maven.apache.org/POM/4.0.0"
         xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.8</version>
        <relativePath/>
    </parent>

   <groupId>com.sunil</groupId>
   <artifactId>Springboot_multipledatasource_demo</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>Springboot_multipledatasource_demo</name>
   <description>Multiple datasource example using Spring Boot and JPA</description>

    <properties>
        <java.version>17</java.version>
    </properties>

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
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 3: Create Entity Classes for Both Domains
Now, we will create entity classes for both domains: **Book** and **Author**. Each entity is placed in a **separate package**, which helps Spring Boot map them to different databases later.

**Book Entity Class**
```java
package com.sunil.springboot_multipledatasource_demo.model.book;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "books")
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer bookId;

    private String title;
    private String author;
    private String publisher;
    private double price;

    // getters and setters
}
```

**Author Entity Class**
```java
package com.sunil.springboot_multipledatasource_demo.model.author;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "authors")
public class Author {

    @Id
    private Integer authorId;

    private String name;
    private String email;

    // getters and setters
}
```

### Step 4: Create JPA Repositories
Now let’s create JPA repositories for both entities in their respective repository packages.

**BookRepository**
```java
package com.sunil.springboot_multipledatasource_demo.repository.book;

import org.springframework.data.jpa.repository.JpaRepository;
import com.sunil.springboot_multipledatasource_demo.model.book.Book;

public interface BookRepository extends JpaRepository<Book, Integer> {
}
```

**AuthorRepository**
```java
package com.sunil.springboot_multipledatasource_demo.repository.author;

import org.springframework.data.jpa.repository.JpaRepository;
import com.sunil.springboot_multipledatasource_demo.model.author.Author;

public interface AuthorRepository extends JpaRepository<Author, Integer> {
}
```

---

### Step 5: Book Database Configuration
This class handles the configuration for the **book database**, designated as the `@Primary` data source.

```java
package com.sunil.springboot_multipledatasource_demo.config;

import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
@EnableJpaRepositories(
        basePackages = "com.sunil.springboot_multipledatasource_demo.repository.book",
        entityManagerFactoryRef = "bookEntityManagerFactory",
        transactionManagerRef = "bookTransactionManager"
)
public class BookDataSourceConfig {

    @Primary
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource bookDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean bookEntityManagerFactory() {
        LocalContainerEntityManagerFactoryBean em =
                new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(bookDataSource());
        em.setPackagesToScan(
                "com.sunil.springboot_multipledatasource_demo.model.book"
        );

        HibernateJpaVendorAdapter vendorAdapter =
                new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);

        Map<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto", "update");
        properties.put("hibernate.dialect",
                "org.hibernate.dialect.PostgreSQLDialect");
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Primary
    @Bean
    public PlatformTransactionManager bookTransactionManager(
            @Qualifier("bookEntityManagerFactory")
            LocalContainerEntityManagerFactoryBean emf) {
        return new JpaTransactionManager(emf.getObject());
    }
}
```

### Step 6: Author Database Configuration
This class handles the secondary configuration for the **author database**.

```java
package com.sunil.springboot_multipledatasource_demo.config;

import java.util.HashMap;
import java.util.Map;
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
@EnableJpaRepositories(
        basePackages = "com.sunil.springboot_multipledatasource_demo.repository.author",
        entityManagerFactoryRef = "authorEntityManagerFactory",
        transactionManagerRef = "authorTransactionManager"
)
public class AuthorDataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.author-datasource")
    public DataSource authorDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean authorEntityManagerFactory() {
        LocalContainerEntityManagerFactoryBean em =
                new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(authorDataSource());
        em.setPackagesToScan(
                "com.sunil.springboot_multipledatasource_demo.model.author"
        );

        HibernateJpaVendorAdapter vendorAdapter =
                new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);

        Map<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto", "update");
        properties.put("hibernate.dialect",
                "org.hibernate.dialect.PostgreSQLDialect");
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Bean
    public PlatformTransactionManager authorTransactionManager(
            @Qualifier("authorEntityManagerFactory")
            LocalContainerEntityManagerFactoryBean emf) {
        return new JpaTransactionManager(emf.getObject());
    }
}
```



### Logic and Service Implementation
The class is annotated with `@Configuration`, which tells Spring that this class contains bean definitions. Spring Boot creates these beans at application startup. We define separate DataSources for each database using properties from `application.properties`. 

Then, we configure a `LocalContainerEntityManagerFactoryBean` to scan specific entity packages. This ensures that all Book-related entities are mapped to the book database and Author-related entities to the author database. Finally, we define a `TransactionManager` for each. 

All CRUD operations remain the same as a normal Spring Data JPA application.

**BookController**
```java
package com.sunil.springboot_multipledatasource_demo.book_database.controller;

import com.sunil.springboot_multipledatasource_demo.model.book.Book;
import com.sunil.springboot_multipledatasource_demo.service.book.BookService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @PostMapping
    public ResponseEntity<Book> saveBook(@RequestBody Book book) {
        Book savedBook = bookService.saveBook(book);
        return new ResponseEntity<>(savedBook, HttpStatus.CREATED);
    }
}
```

**BookService**
```java
package com.sunil.springboot_multipledatasource_demo.service.book;

import com.sunil.springboot_multipledatasource_demo.model.book.Book;
import com.sunil.springboot_multipledatasource_demo.repository.book.BookRepository;
import org.springframework.stereotype.Service;

@Service
public class BookService {

    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public Book saveBook(Book book) {
        return bookRepository.save(book);
    }
}
```

---

In this way, we can configure multiple DataSources in a single Spring Boot application. Each DataSource bean is created using the `@ConfigurationProperties` annotation with a specific property prefix mapping to `application.properties`. Spring Boot’s `DataSourceBuilder` automatically handles the heavy lifting, allowing us to connect multiple databases without writing low-level JDBC code.

---
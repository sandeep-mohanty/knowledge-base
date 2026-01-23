# How to Use Spring Boot Profiles for Environment Configuration

![Spring Boot Profiles Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*vlJSRDeCprJZk4dn9G7jRw.png)

Spring Boot is a popular framework built on top of Spring. It is widely used for the rapid development of Java-based applications. By using Spring Boot, developers can focus more on implementing business logic instead of spending time on boilerplate setup and configuration.

When working on real-time projects, we typically maintain multiple environments for the same application. This is necessary because different stages of the development lifecycle often require different infrastructures. Common environments include development (DEV), testing, user acceptance testing (UAT), and production (PROD).

However, these environments aren’t fixed. Some projects might require additional stages, while others might have fewer — depending entirely on the stakeholders’ decisions and the specific project requirements.

To handle this efficiently, the Spring Framework provides a powerful feature called **profiles**, which allows developers to configure and manage environment-specific settings with minimal effort.

In this article, we’ll explore how to use **Spring Boot Profiles for Environment Configuration** to simplify switching between environments.

---

## Spring Profile?
A **Spring Profile** is a logical grouping of configuration properties associated with a specific environment. For example, if a set of properties is intended for the development environment, it forms the **development profile**. Similarly, other profiles like **test**, **UAT**, or **production** can be created based on environment-specific configurations.



Spring Profiles allow developers to isolate and manage environment-specific parts of an application’s configuration. This helps ensure that the right settings are applied in the right environment — making the application easier to configure, maintain, and deploy across different stages.

## Why Use Spring Boot Profiles?
* **Manage environment-specific settings:** Configure different databases, API keys, logging levels, and more — all based on the active environment.
* **Switch configurations without modifying code:** Seamlessly move between **DEV**, **UAT**, and **PROD** environments just by activating the appropriate profile.
* **Use different beans for different environments:** Leverage the `@Profile` annotation to load environment-specific beans only when required.
* **Avoid hardcoding environment variables:** Keep configurations externalized, reducing the risk of hardcoded values in your codebase.

---

## What Is the Default Profile in Spring Boot?
Spring Boot uses the `application.properties` or `application.yml` file as the default profile. This file is always active and loaded first.

If a property is not defined in a specific profile (like `application-prod.properties`), Spring Boot will use the value from the default profile. This is useful for setting common values that apply to all environments.

---

## How to Create a Spring Profile
Instead of changing values in the default `application.properties` file every time you switch environments, it’s better to create separate configuration files for each environment. You can then activate the one you need with a single setting.

### Step 1: Identify Your Environments
Let’s say your project has three environments:
1.  Development
2.  Testing
3.  Production

You’ll create a separate profile (property file) for each of them.

### Step 2: Create Profile-Specific Property Files
Place these files in the same location as your `application.properties`:
* `application-dev.properties`
* `application-test.properties`
* `application-prod.properties`

### Step 3: Add Environment-Specific Settings
For example, different database configs for each environment:

**Development (`application-dev.properties`)**
PostgreSQL configuration
```properties
app.info=Dev Env Property file
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/springTestDB
spring.datasource.username=postgres
spring.datasource.password=postgres123
```

**Testing (`application-test.properties`)**
MySQL configuration
```properties
app.info= Test Env property file
spring.datasource.url=jdbc:mysql://localhost:3306/springTestDB
spring.datasource.username=mysql
spring.datasource.password=mysql123
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

**Production (`application-prod.properties`)**
Oracle configuration
```properties
app.message = Prod Env property file
spring.datasource.url=jdbc:oracle:thin:@localhost:1521:springTestDB
spring.datasource.username=oracle
spring.datasource.password=oracle123
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle10gDialect
```

Each profile file contains only the settings relevant to that specific environment.

---

## How to Activate a Particular Spring Boot Profile
There are several ways to activate a specific Spring Boot profile. Let’s look at the most common methods:

### 1. Set spring.profiles.active in application.properties
This is the simplest and most common way. Add the following line in your default `application.properties` file to activate a profile (e.g., dev):
```properties
spring.profiles.active=dev
```
// This tells Spring Boot to use the dev profile when starting the application.

### 2. Pass as a JVM system parameter
You can also activate a profile when starting your app by passing it as a JVM argument:
```bash
-Dspring.profiles.active=dev
```

### 3. Set in web.xml (for web apps using web.xml)
If you use a web.xml file, add this context parameter:
```xml
<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
```

### 4. Programmatically via WebApplicationInitializer
In web applications, you can set the active profile in code by implementing `WebApplicationInitializer`:
```java
@Configuration
public class MyWebApplicationInitializer implements WebApplicationInitializer {
   @Override
   public void onStartup(ServletContext servletContext) throws ServletException {
      servletContext.setInitParameter("spring.profiles.active", "dev");
   }
}
```

---

## How to Use @Profile in Spring
In Spring, you can use the `@Profile` annotation to load beans only for specific environments. This allows you to define environment-specific beans that get activated based on the active profile.

**Basic Syntax:**
```java
@Profile("profile-name")
public class MyBean {
    // Bean definition here
}
```
When Spring detects that the `profile-name` is active, it will load the associated bean. Otherwise, it will ignore the bean.

### Create Profile-Specific Beans
Here’s an example of how to create profile-specific data source configurations for Development, UAT, and Production environments.

```java
import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

// Dev Profile → PostgreSQL
@Configuration
@Profile("dev")
public class DatabaseConfig {

    @Bean
    public DataSource dataSource() {
        System.out.println("Using PostgreSQL database for Dev environment");
        return new PostgresDataSource(); // Example class, replace with actual DataSource
    }
}

// Test Profile → MySQL
@Configuration
@Profile("test")
public class TestDatabaseConfig {

    @Bean
    public DataSource dataSource() {
        System.out.println("Using MySQL database for Test environment");
        return new MySQLDataSource(); // Example class, replace with actual DataSource
    }
}

// Prod Profile → Oracle
@Configuration
@Profile("prod")
public class ProdDatabaseConfig {

    @Bean
    public DataSource dataSource() {
        System.out.println("Using Oracle database for Prod environment");
        return new OracleDataSource(); // Example class, replace with actual DataSource
    }
}
```

Furthermore, we can associate multiple profiles with a bean using the `@Profile` annotation, as shown below:
```java
@Configuration 
public class AppConfig { 

  @Profile({"prod", "dev"}) 
  @Bean 
  public MyBean getBean() {
    return new MyBean(); 
  } 
}
```

---

## How to Work with Profiles using .yml in Spring Boot
Spring Boot lets us use YAML (`.yml`) files instead of `.properties` files for configuration. The good thing is, we can create profile-specific settings in `.yml` in two different ways:

### 1. Separate .yml Files for Each Profile
Just like `.properties`, we can create separate `.yml` files for each environment:

**`application.yml`**: Default config
```yaml
spring:
  profiles:
    active: dev   # sets dev as the active profile
app:
  name: Primary Application Config
```

**`application-dev.yml`**: Dev profile config
```yaml
spring:
  profiles: dev
  datasource:
    username: postgres
    password: postgres123
    url: jdbc:postgresql://localhost:5432/springTestDB
    driver-class-name: org.postgresql.Driver
```

**`application-test.yml`**: Test profile config
```yaml
spring:
  profiles: test
  datasource:
    username: root
    password: root123
    url: jdbc:mysql://localhost:3306/springTestDB
    driver-class-name: com.mysql.cj.jdbc.Driver
```

**`application-prod.yml`**: Prod profile config
```yaml
spring:
  profiles: prod
  datasource:
    username: oracle
    password: oracle@123
    url: jdbc:oracle:thin:@localhost:1521:springTestDB
    driver-class-name: oracle.jdbc.OracleDriver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.Oracle10gDialect
```

### 2. All Profiles in One .yml File
Instead of multiple files, we can put everything in one `application.yml`.
We separate each profile section with `---` (three dashes).

```yaml
spring:
  profiles:
    active: dev   # Set active profile
---
spring:
  profiles: dev
  datasource:
    username: postgres
    password: postgres123
    url: jdbc:postgresql://localhost:5432/springTestDB
    driver-class-name: org.postgresql.Driver
---
spring:
  profiles: test
  datasource:
    username: mysql
    password: mysql123
    url: jdbc:mysql://localhost:3306/springTestDB
    driver-class-name: com.mysql.cj.jdbc.Driver
---
spring:
  profiles: prod
  datasource:
    username: oracle
    password: oracle@123
    url: jdbc:oracle:thin:@localhost:1521:springTestDB
    driver-class-name: oracle.jdbc.OracleDriver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.Oracle10gDialect
```

---

## How to Check Which Profile is Active in Spring Boot
In Spring Boot, we can easily check which profile is currently active.
There are **two simple ways** to do this:

### 1. Using Environment Object
Spring provides an `Environment` object which we can use to get active profiles.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProfileController {

    @Autowired
    private Environment env;

    public void getActiveProfiles() {
        for (String profile : env.getActiveProfiles()) {
            System.out.println("Active Profile: " + profile);
        }
    }
}
```

### 2. Using spring.profiles.active Property
We can directly inject the property `spring.profiles.active` into a variable.

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProfileController {

    @Value("${spring.profiles.active:}")
    private String activeProfiles;

    public void getActiveProfiles() {
        for (String profile : activeProfiles.split(",")) {
            System.out.println("Active Profile: " + profile);
        }
    }
}
```
`activeProfiles` will hold the active profile name.
If there are multiple profiles, they will be separated by commas.

---

## Best Practices for Using Spring Profiles
* **Keep configurations separate:** Create different configuration classes or `application-<profile>.properties` files for each environment.
* **Use clear profile names:** Choose simple and descriptive names like `dev`, `test`, `stage`, `prod`.
* **Don’t hardcode profiles in code:** Always activate profiles through environment variables or properties files instead of hardcoding them in your Java code.
* **Always test your application in all environments** (dev, test, prod, etc.) so you can catch configuration problems early.

In this article, we saw how **Spring Profiles** help manage different environments. They let us **switch configurations** (databases, settings, beans) without changing code. We learned to **create profile-specific beans** and **activate profiles** in various ways. We also explored using **.properties or .yml files** for profile-based configurations.
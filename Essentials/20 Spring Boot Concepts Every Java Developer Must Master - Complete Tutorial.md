# 20 Spring Boot Concepts Every Java Developer Must Master

**A Complete Hands-On Guide to Understanding Spring Boot Beyond Just Writing Code**
---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [Spring IoC Container: The Brain Behind Every Spring Boot Application](#1-spring-ioc-container-the-brain-behind-every-spring-boot-application)
5. [Dependency Injection: Let Spring Connect Your Components](#2-dependency-injection-let-spring-connect-your-components)
6. [Auto Configuration: Spring Boot's Biggest Superpower](#3-auto-configuration-spring-boots-biggest-superpower)
7. [Starter Dependencies: Simplifying Project Setup](#4-starter-dependencies-simplifying-project-setup)
8. [Spring Boot Project Structure: Organizing Applications Properly](#5-spring-boot-project-structure-organizing-applications-properly)
9. [Bean Lifecycle: Understanding What Happens Behind the Scenes](#6-bean-lifecycle-understanding-what-happens-behind-the-scenes)
10. [Configuration Properties: Making Applications Flexible](#7-configuration-properties--making-applications-flexible)
11. [REST Controllers: The Entry Point of Every API](#8-rest-controllers-the-entry-point-of-every-api)
12. [Request Mapping: Connecting URLs to Java Methods](#9-request-mapping-connecting-urls-to-java-methods)
13. [Validation: Protecting Your Application from Invalid Data](#10-validation-protecting-your-application-from-invalid-data)
14. [Global Exception Handling: Managing Errors Gracefully](#11-global-exception-handling-managing-errors-gracefully)
15. [Spring Data JPA: Making Database Access Easier](#12-spring-data-jpa-making-database-access-easier)
16. [Entity Relationships: Modeling Real-World Data](#13-entity-relationships-modeling-real-world-data)
17. [Transactions: Keeping Data Consistent](#14-transactions-keeping-data-consistent)
18. [Spring Security Basics: Protecting Your Application](#15-spring-security-basics-protecting-your-application)
19. [Building a Typical Spring Boot Request Flow](#16-building-a-typical-spring-boot-request-flow)
20. [JWT Authentication: Building Stateless Security](#17-jwt-authentication-building-stateless-security)
21. [Profiles and Environment Configuration](#18-profiles-and-environment-configuration)
22. [Logging and Monitoring: Understanding Your Application in Production](#19-logging-and-monitoring-understanding-your-application-in-production)
23. [Testing in Spring Boot: Writing Software You Can Trust](#20-testing-in-spring-boot-writing-software-you-can-trust)
24. [Building Production-Ready Spring Boot Applications](#21-building-production-ready-spring-boot-applications)
25. [Common Mistakes Spring Boot Developers Make](#common-mistakes-spring-boot-developers-make)
26. [Best Practices Every Spring Boot Developer Should Follow](#best-practices-every-spring-boot-developer-should-follow)
27. [Practice Exercises](#practice-exercises)
28. [Question Bank](#question-bank)
29. [Summary and Key Takeaways](#summary-and-key-takeaways)
30. [Further Reading and Resources](#further-reading-and-resources)

---

## Introduction

Java has remained one of the most trusted programming languages for decades. It powers enterprise applications, banking systems, healthcare platforms, e-commerce websites, government portals, cloud services, and countless business applications used by millions of people every day. While Java itself provides a strong foundation for building software, developing modern web applications from scratch using only core Java can quickly become overwhelming.

### The Problem with Plain Java

Imagine building an online shopping platform without any framework. You would need to manually configure servers, create database connections, manage object creation, implement authentication, handle HTTP requests, configure security, process exceptions, manage dependencies, and perform dozens of repetitive tasks before even writing your business logic.

```mermaid
graph TD
    A[Plain Java Development] --> B[Manual Server Config]
    A --> C[Manual Object Creation]
    A --> D[Manual DB Connections]
    A --> E[Manual HTTP Handling]
    A --> F[Manual Security Setup]
    A --> G[Manual Exception Handling]
    A --> H[Manual Dependency Management]
    
    B --> I[Weeks of Configuration]
    C --> I
    D --> I
    E --> I
    F --> I
    G --> I
    H --> I
    
    I --> J[Finally Write Business Logic]
    
    style A fill:#ff6b6b
    style I fill:#ffa500
    style J fill:#4ecdc4
```

**Figure 1: The Complexity of Plain Java Development**

While it's certainly possible, it's neither efficient nor practical for modern software development. This is exactly why Spring was created.

### The Spring Revolution

The Spring Framework revolutionized Java development by introducing concepts such as Dependency Injection and Inversion of Control, making applications more modular, maintainable, and testable. However, as powerful as Spring was, setting up a new project often involved writing hundreds of lines of configuration files before developers could even begin building features.

**Enter Spring Boot.**

Spring Boot takes the best features of the Spring Framework and removes much of the complexity associated with project setup and configuration. Instead of spending hours configuring XML files and dependencies, developers can create production-ready applications in minutes.

### Why This Guide Matters

Many developers can build CRUD applications but struggle to explain what actually happens behind the scenes. They know which annotation to use, but not why it exists or how Spring processes it internally.

> **💡 Key Insight**
> 
> Companies don't simply hire developers who can copy code from tutorials. They look for engineers who understand the framework's architecture, know how components communicate, and can troubleshoot problems when applications become large and complex.

In this guide, we'll explore twenty essential Spring Boot concepts that every Java developer should master. Whether you're preparing for technical interviews, building enterprise applications, or simply trying to become a better backend developer, these concepts will help you understand how Spring Boot truly works under the hood.

---

## Prerequisites

Before diving into this tutorial, ensure you have:

### Required Knowledge
- ✅ **Java Fundamentals:** Strong understanding of Java OOP concepts (classes, objects, inheritance, interfaces)
- ✅ **Java 8+ Features:** Familiarity with lambdas, streams, and functional interfaces
- ✅ **Basic Web Concepts:** Understanding of HTTP protocol, REST APIs, request/response cycle
- ✅ **Database Basics:** Knowledge of SQL and relational database concepts
- ✅ **Maven or Gradle:** Basic understanding of build tools

### Recommended Tools
- ☕ **JDK 17 or 21** (LTS versions recommended)
- 🛠️ **IDE:** IntelliJ IDEA, Eclipse, or VS Code with Java extensions
- 🔧 **Build Tool:** Maven 3.6+ or Gradle 7+
- 🗄️ **Database:** MySQL, PostgreSQL, or H2 for testing
- 🌐 **Postman** or similar API testing tool

### Nice to Have
- Basic understanding of Spring Framework
- Familiarity with annotations in Java
- Exposure to MVC architecture patterns

---

## Learning Objectives

By the end of this tutorial, you will be able to:

### Core Concepts
- ✅ Explain the Spring IoC Container and its role in application architecture
- ✅ Implement Dependency Injection using constructor, setter, and field injection
- ✅ Understand and leverage Auto Configuration in Spring Boot
- ✅ Organize Spring Boot projects using layered architecture
- ✅ Manage Bean lifecycles and initialization/cleanup

### Data & Persistence
- ✅ Configure Spring Data JPA repositories
- ✅ Model entity relationships (One-to-One, One-to-Many, Many-to-Many)
- ✅ Implement transaction management with @Transactional
- ✅ Optimize database queries and avoid N+1 problems

### Security & APIs
- ✅ Build REST controllers with proper request mapping
- ✅ Implement validation for incoming requests
- ✅ Handle exceptions globally with @RestControllerAdvice
- ✅ Secure applications with Spring Security and JWT
- ✅ Configure profiles for different environments

### Production & Testing
- ✅ Write unit and integration tests for Spring Boot applications
- ✅ Implement logging and monitoring with Actuator
- ✅ Apply best practices for production-ready applications
- ✅ Identify and avoid common anti-patterns

---

## 1. Spring IoC Container: The Brain Behind Every Spring Boot Application

### Understanding Inversion of Control

One of the biggest differences between traditional Java programming and Spring Boot lies in **who controls object creation**.

#### Traditional Java Approach
```java
// ❌ Manual object creation - tightly coupled
public class UserController {
    private UserService userService = new UserService();
    
    public void handleRequest() {
        userService.processUser();
    }
}

public class UserService {
    private UserRepository userRepository = new UserRepository();
    
    public void processUser() {
        userRepository.save();
    }
}
```

**Problems with this approach:**
- Tight coupling between classes
- Difficult to test (can't easily mock dependencies)
- Hard to change implementations
- Manual memory management
- No centralized control

#### Spring Boot Approach
```java
// ✅ Let Spring manage object creation
@Service
public class UserService {
    private final UserRepository userRepository;
    
    // Constructor injection - Spring provides the dependency
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public void processUser() {
        userRepository.save();
    }
}

@RestController
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.getAllUsers();
    }
}
```

### The IoC Container Architecture

```mermaid
graph TB
    subgraph "IoC Container (The Manager)"
        A[Component Scanning] --> B[Bean Definition Registry]
        B --> C[Bean Factory]
        C --> D[Bean Creation]
        D --> E[Dependency Injection]
        E --> F[Bean Lifecycle Management]
    end
    
    subgraph "Application Components"
        G[@Component] --> H[UserService]
        I[@Service] --> J[OrderService]
        K[@Repository] --> L[UserRepository]
        M[@Controller] --> N[UserController]
        O[@Configuration] --> P[AppConfig]
    end
    
    A --> G
    A --> I
    A --> K
    A --> M
    A --> O
    
    F --> H
    F --> J
    F --> L
    F --> N
    F --> P
    
    style A fill:#4ecdc4
    style F fill:#95e1d3
    style H fill:#ffe66d
    style J fill:#ffe66d
    style L fill:#ffe66d
    style N fill:#ffe66d
    style P fill:#ffe66d
```

**Figure 2: Spring IoC Container Architecture**

### How It Works: Step by Step

1. **Component Scanning:** Spring scans your application for classes annotated with `@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController`, and `@Configuration`

2. **Bean Definition Creation:** For each eligible class, Spring creates a bean definition (metadata about the bean)

3. **Bean Instantiation:** When needed, Spring creates instances of these beans

4. **Dependency Injection:** Spring automatically injects required dependencies into each bean

5. **Bean Lifecycle Management:** Spring manages the complete lifecycle of beans from creation to destruction

### Component Stereotypes

Spring provides several stereotype annotations to categorize your beans:

| Annotation | Purpose | Layer | Example |
|------------|---------|-------|---------|
| `@Component` | Generic component | Any | Utility classes |
| `@Service` | Business logic | Service layer | `UserService`, `OrderService` |
| `@Repository` | Data access | Persistence layer | `UserRepository` |
| `@Controller` | Web controller (returns views) | Presentation layer | MVC controllers |
| `@RestController` | REST API controller | Presentation layer | REST APIs |
| `@Configuration` | Configuration class | Configuration | `AppConfig`, `SecurityConfig` |

### Practical Example: Complete IoC Flow

```java
// Step 1: Define repository for data access
@Repository
public class UserRepository {
    public User findById(Long id) {
        // Database access logic
        return new User(id, "John Doe");
    }
}

// Step 2: Define service for business logic
@Service
public class UserService {
    private final UserRepository userRepository;
    
    // Spring automatically injects UserRepository
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
}

// Step 3: Define controller for HTTP handling
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    
    // Spring automatically injects UserService
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        return ResponseEntity.ok(user);
    }
}

// Step 4: Main application class
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        // Spring Boot starts IoC Container here
        SpringApplication.run(Application.class, args);
    }
}
```

### Advantages of IoC Container

✅ **Centralized Object Management:** All object creation happens in one place
✅ **Loose Coupling:** Components don't create their dependencies
✅ **Easy Testing:** Dependencies can be easily mocked or replaced
✅ **Lifecycle Management:** Spring handles initialization and cleanup
✅ **Configuration Externalization:** Easy to change implementations without modifying code
✅ **Better Memory Management:** Spring controls bean lifecycles efficiently

> **⚠️ Important Note**
> 
> Without understanding the IoC Container, it's difficult to fully appreciate how Spring Boot works behind the scenes. Every other concept builds upon this foundation.

### Common Pitfalls

❌ **Mistake 1: Creating Objects Manually**
```java
// Don't do this
@Service
public class BadService {
    private UserRepository repository = new UserRepository(); // ❌ Manual creation
}
```

✅ **Correct Approach:**
```java
@Service
public class GoodService {
    private final UserRepository repository;
    
    public GoodService(UserRepository repository) { // ✅ Let Spring inject
        this.repository = repository;
    }
}
```

❌ **Mistake 2: Using @Autowired on Fields**
```java
@Service
public class BadPractice {
    @Autowired // ❌ Hidden dependency, hard to test
    private UserRepository repository;
}
```

✅ **Correct Approach: Constructor Injection**
```java
@Service
public class GoodPractice {
    private final UserRepository repository;
    
    public GoodPractice(UserRepository repository) { // ✅ Explicit dependency
        this.repository = repository;
    }
}
```

---

## 2. Dependency Injection: Let Spring Connect Your Components

### What is Dependency Injection?

Dependency Injection (DI) is the mechanism through which the IoC Container supplies dependencies to application components. Instead of components creating their dependencies, Spring provides them.

### The Dependency Problem

Consider a simple user management system:

```
UserController → needs → UserService → needs → UserRepository → needs → DataSource
```

Without DI, each component would manually create the next one, creating tightly coupled code.

### Three Types of Dependency Injection

#### 1. Constructor Injection ⭐ (Recommended)

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final NotificationService notificationService;
    
    // All dependencies provided at construction time
    public OrderService(
            OrderRepository orderRepository,
            PaymentService paymentService,
            NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }
    
    public Order createOrder(OrderRequest request) {
        // Business logic using injected dependencies
        Order order = new Order(request);
        orderRepository.save(order);
        paymentService.processPayment(order);
        notificationService.sendConfirmation(order);
        return order;
    }
}
```

**Advantages:**
- ✅ Immutable dependencies (can use `final`)
- ✅ Clear what's required (explicit dependencies)
- ✅ Easy to test (no reflection needed)
- ✅ No hidden dependencies
- ✅ Works well with `@ConfigurationProperties`

#### 2. Setter Injection

```java
@Service
public class ReportService {
    private EmailService emailService;
    private PdfGenerator pdfGenerator;
    
    // Optional dependencies can be set later
    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    @Autowired
    public void setPdfGenerator(PdfGenerator pdfGenerator) {
        this.pdfGenerator = pdfGenerator;
    }
    
    public void generateReport(Report report) {
        byte[] pdf = pdfGenerator.generate(report);
        if (emailService != null) {
            emailService.send(report.getEmail(), pdf);
        }
    }
}
```

**When to use:**
- Optional dependencies
- Circular dependencies (rare, but sometimes necessary)
- Legacy code integration

#### 3. Field Injection ⚠️ (Not Recommended)

```java
@Service
public class BadPractice {
    @Autowired // ❌ Hidden dependency
    private UserRepository userRepository;
    
    @Autowired // ❌ Hard to test
    private EmailService emailService;
    
    public void process() {
        userRepository.save(new User());
    }
}
```

**Disadvantages:**
- ❌ Hidden dependencies (not visible in API)
- ❌ Can't use `final` (mutable)
- ❌ Hard to test (requires reflection)
- ❌ Violates Single Responsibility Principle
- ❌ Can lead to NullPointerException if bean not found

### Dependency Injection Flow

```mermaid
sequenceDiagram
    participant A as Application Start
    participant B as IoC Container
    participant C as Bean Factory
    participant D as UserController
    participant E as UserService
    participant F as UserRepository
    
    A->>B: Initialize Container
    B->>B: Scan for Components
    B->>C: Register Bean Definitions
    
    Note over C: Bean Creation Phase
    
    C->>F: Create UserRepository
    F-->>C: Instance Created
    
    C->>E: Create UserService
    Note over C: Inject UserRepository
    C->>F: Get UserRepository instance
    F-->>C: Return instance
    C->>E: Inject dependency
    E-->>C: Instance Created
    
    C->>D: Create UserController
    Note over C: Inject UserService
    C->>E: Get UserService instance
    E-->>C: Return instance
    C->>D: Inject dependency
    D-->>C: Instance Created
    
    C-->>B: All Beans Ready
    B-->>A: Application Started
    
    Note over D,F: Application Running
```

**Figure 3: Dependency Injection Sequence Diagram**

### Comparison: Injection Methods

| Aspect | Constructor | Setter | Field |
|--------|-------------|--------|-------|
| **Immutability** | ✅ Yes (can use `final`) | ❌ No | ❌ No |
| **Testability** | ✅ Easy (no reflection) | ✅ Good | ❌ Hard (needs reflection) |
| **Dependency Visibility** | ✅ Explicit | ✅ Visible | ❌ Hidden |
| **Optional Dependencies** | ❌ Not suitable | ✅ Yes | ✅ Yes |
| **Circular Dependencies** | ❌ Fails fast | ✅ Can handle | ✅ Can handle |
| **Code Clarity** | ✅ Very clear | ✅ Clear | ❌ Unclear |
| **Spring Recommendation** | ⭐ Preferred | Acceptable | ⚠️ Avoid |

### Practical Example: Testing with DI

```java
// Service to test
@Service
public class PaymentService {
    private final PaymentRepository paymentRepository;
    private final NotificationService notificationService;
    
    public PaymentService(PaymentRepository paymentRepository, 
                         NotificationService notificationService) {
        this.paymentRepository = paymentRepository;
        this.notificationService = notificationService;
    }
    
    public Payment processPayment(PaymentRequest request) {
        // Business logic
        Payment payment = new Payment(request);
        paymentRepository.save(payment);
        notificationService.sendPaymentConfirmation(payment);
        return payment;
    }
}

// Unit Test - Easy because of constructor injection
class PaymentServiceTest {
    
    private PaymentRepository mockRepository;
    private NotificationService mockNotification;
    private PaymentService paymentService;
    
    @BeforeEach
    void setUp() {
        mockRepository = mock(PaymentRepository.class);
        mockNotification = mock(NotificationService.class);
        
        // Easy to inject mocks
        paymentService = new PaymentService(mockRepository, mockNotification);
    }
    
    @Test
    void shouldProcessPaymentSuccessfully() {
        // Given
        PaymentRequest request = new PaymentRequest(100.0, "USD");
        Payment savedPayment = new Payment(1L, 100.0, "USD", "COMPLETED");
        
        when(mockRepository.save(any(Payment.class))).thenReturn(savedPayment);
        
        // When
        Payment result = paymentService.processPayment(request);
        
        // Then
        assertThat(result.getStatus()).isEqualTo("COMPLETED");
        verify(mockRepository).save(any(Payment.class));
        verify(mockNotification).sendPaymentConfirmation(savedPayment);
    }
}
```

### Benefits of Dependency Injection

✅ **Loose Coupling:** Components don't know how their dependencies are created
✅ **Easy Testing:** Dependencies can be easily replaced with mocks
✅ **Flexibility:** Easy to swap implementations
✅ **Maintainability:** Changes to dependencies don't affect component code
✅ **Reusability:** Components can be used in different contexts

> **💡 Pro Tip**
> 
> Constructor injection is not just a Spring recommendation—it's a best practice that aligns with SOLID principles, particularly the Dependency Inversion Principle. Always prefer constructor injection for required dependencies.

### Real-World Scenario

Imagine you're building an e-commerce platform:

```java
@Service
public class OrderProcessingService {
    private final PaymentGateway paymentGateway;
    private final InventoryService inventoryService;
    private final EmailService emailService;
    private final AnalyticsService analyticsService;
    
    // In production, Spring injects real implementations
    // In testing, you can inject mocks
    // In different environments, you can inject different implementations
    public OrderProcessingService(
            PaymentGateway paymentGateway,
            InventoryService inventoryService,
            EmailService emailService,
            AnalyticsService analyticsService) {
        this.paymentGateway = paymentGateway;
        this.inventoryService = inventoryService;
        this.emailService = emailService;
        this.analyticsService = analyticsService;
    }
    
    public OrderResult processOrder(Order order) {
        // All dependencies are ready to use
        PaymentResult payment = paymentGateway.charge(order);
        inventoryService.reserve(order.getItems());
        emailService.sendOrderConfirmation(order);
        analyticsService.trackOrder(order);
        
        return new OrderResult(payment, order);
    }
}
```

This design allows you to:
- Switch payment gateways without changing business logic
- Test with mock implementations
- Add new features (analytics) without modifying core logic
- Maintain clean, readable code

---

## 3. Auto Configuration: Spring Boot's Biggest Superpower

### What is Auto Configuration?

One of Spring Boot's greatest strengths is something many developers rarely think about: **Auto Configuration**.

Imagine creating a new backend application. Without auto configuration, you would need to configure:
- Embedded server (Tomcat, Jetty, Undertow)
- DispatcherServlet
- Database connections
- JSON converters (Jackson)
- Error handling
- Logging frameworks
- Static resource handling
- Security defaults
- Property loading

This could easily require **hundreds of lines** of configuration code.

### How Auto Configuration Works

Spring Boot automates much of this process. When the application starts, Spring Boot:

1. **Scans the classpath** for available libraries
2. **Analyzes configuration properties** from `application.properties`/`application.yml`
3. **Checks existing beans** to avoid conflicts
4. **Configures components** based on what it finds

```mermaid
graph LR
    A[Application Start] --> B[Scan Classpath]
    B --> C{Found Spring Web?}
    C -->|Yes| D[Configure Tomcat]
    C -->|No| E{Found Spring Data JPA?}
    
    D --> F[Configure DispatcherServlet]
    F --> G[Configure JSON Mapping]
    G --> H[Configure Static Resources]
    
    E -->|Yes| I[Configure DataSource]
    E -->|No| J{Found Spring Security?}
    
    I --> K[Configure EntityManager]
    K --> L[Configure Transaction Manager]
    
    J -->|Yes| M[Configure Security Filters]
    J -->|No| N[Application Ready]
    
    M --> N
    L --> N
    H --> N
    
    style A fill:#4ecdc4
    style N fill:#95e1d3
    style C fill:#ffe66d
    style E fill:#ffe66d
    style J fill:#ffe66d
```

**Figure 4: Spring Boot Auto Configuration Decision Tree**

### Practical Examples

#### Example 1: Spring Web Auto Configuration

When you add `spring-boot-starter-web` dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot **automatically** configures:
- Embedded Tomcat server (port 8080 by default)
- Spring MVC with DispatcherServlet
- Jackson for JSON serialization/deserialization
- Validation support
- Error handling (BasicErrorController)
- Static resource serving from `/static`, `/public`, `/resources`, `/META-INF/resources`

**You get all this without writing a single line of configuration!**

#### Example 2: Spring Data JPA Auto Configuration

When you add `spring-boot-starter-data-jpa` and a database driver:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

Spring Boot **automatically** configures:
- DataSource connection pool (HikariCP by default)
- EntityManagerFactory
- TransactionManager
- JPA vendor adapter (Hibernate)
- Database initialization scripts

#### Example 3: Spring Security Auto Configuration

When you add `spring-boot-starter-security`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Spring Boot **automatically** configures:
- Basic authentication
- Default user with generated password
- Security filter chain
- CSRF protection
- Session management
- Default security headers

### Customizing Auto Configuration

You can override auto-configured beans by defining your own:

```java
@Configuration
public class CustomTomcatConfiguration {
    
    @Bean
    public TomcatServletWebServerFactory tomcatFactory() {
        return new TomcatServletWebServerFactory() {
            @Override
            protected void customizeConnector(Connector connector) {
                // Customize Tomcat connector
                super.customizeConnector(connector);
                connector.setPort(9090); // Custom port
                connector.setRedirectPort(9443);
            }
        };
    }
}

// Or simpler approach using properties
// application.properties
server.port=9090
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
```

### Conditional Configuration

Spring Boot uses conditions to determine when to apply auto-configuration:

```java
@Configuration
@ConditionalOnClass(DataSource.class) // Only if DataSource is on classpath
@ConditionalOnProperty(prefix = "app", name = "database", matchIfMissing = true)
public class DatabaseConfiguration {
    // Configuration logic
}

@ConditionalOnMissingBean // Only if no other bean of same type exists
@Bean
public DataSource dataSource() {
    return new HikariDataSource();
}
```

### Common Auto-Configuration Classes

| Auto-Configuration Class | Triggered When | Configures |
|-------------------------|----------------|------------|
| `DataSourceAutoConfiguration` | Database driver on classpath | DataSource, connection pool |
| `HibernateJpaAutoConfiguration` | JPA + DataSource present | EntityManager, transactions |
| `WebMvcAutoConfiguration` | Spring Web on classpath | DispatcherServlet, message converters |
| `SecurityAutoConfiguration` | Spring Security on classpath | Security filter chain |
| `RedisAutoConfiguration` | Redis client on classpath | Redis connection factory |
| `MailSenderAutoConfiguration` | JavaMail on classpath | Mail sender |
| `CacheAutoConfiguration` | Cache on classpath | Cache manager |

### Debugging Auto Configuration

Enable debug logging to see what Spring Boot is configuring:

```properties
# application.properties
debug=true

# Or use actuator endpoint
management.endpoints.web.exposure.include=*
```

Visit `http://localhost:8080/actuator/conditions` to see a detailed report of auto-configuration decisions.

### Advantages of Auto Configuration

✅ **Rapid Development:** Start building features immediately
✅ **Best Practices:** Spring Boot applies proven configurations
✅ **Reduced Boilerplate:** Less code to write and maintain
✅ **Consistency:** Standard configurations across projects
✅ **Flexibility:** Easy to override when needed

> **⚠️ Important Consideration**
> 
> Auto Configuration is a double-edged sword. While it accelerates development, it can also hide important details. Always understand what Spring Boot is configuring for you. Enable debug mode when learning or troubleshooting.

### Real-World Impact

**Without Spring Boot:**
```java
// Hundreds of lines of configuration
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { RootConfig.class };
    }
    
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { WebConfig.class };
    }
    
    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}

// Plus XML or Java config for:
// - View resolvers
// - Message converters
// - Transaction managers
// - Entity managers
// - Security
// - And much more...
```

**With Spring Boot:**
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

That's it! Everything else is auto-configured.

---

## 4. Starter Dependencies: Simplifying Project Setup

### The Dependency Management Problem

Managing dependencies manually used to be one of the most frustrating parts of Java development. A simple web application might require dozens of individual libraries, each depending on other libraries, leading to version conflicts and compatibility issues.

### What are Starter Dependencies?

Spring Boot introduced **Starter Dependencies** to solve this problem. Instead of adding twenty individual libraries, developers include a single starter that contains a carefully selected collection of compatible libraries.

### Common Starters

| Starter | Purpose | Includes |
|---------|---------|----------|
| `spring-boot-starter-web` | Build web applications | Spring MVC, Tomcat, Jackson, Validation |
| `spring-boot-starter-data-jpa` | Database access with JPA | Spring Data JPA, Hibernate, HikariCP |
| `spring-boot-starter-security` | Security features | Spring Security, authentication, authorization |
| `spring-boot-starter-validation` | Validation support | Hibernate Validator, Bean Validation |
| `spring-boot-starter-test` | Testing framework | JUnit, Mockito, Spring Test |
| `spring-boot-starter-mail` | Email functionality | JavaMail, Spring Email |
| `spring-boot-starter-data-redis` | Redis integration | Spring Data Redis, Jedis/Lettuce |
| `spring-boot-starter-actuator` | Monitoring and metrics | Actuator endpoints, metrics |
| `spring-boot-starter-amqp` | AMQP messaging | Spring AMQP, RabbitMQ |
| `spring-boot-starter-oauth2-client` | OAuth2 client | OAuth2 client support |

### Practical Example: Building a REST API

**Without Starters (Old Way):**
```xml
<dependencies>
    <!-- Spring MVC -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.21</version>
    </dependency>
    
    <!-- Embedded Tomcat -->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-core</artifactId>
        <version>9.0.68</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-el</artifactId>
        <version>9.0.68</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-websocket</artifactId>
        <version>9.0.68</version>
    </dependency>
    
    <!-- JSON Processing -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.14.2</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.14.2</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.14.2</version>
    </dependency>
    
    <!-- Validation -->
    <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>2.0.1.Final</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate.validator</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>6.2.5.Final</version>
    </dependency>
    
    <!-- Logging -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.11</version>
    </dependency>
    
    <!-- And many more... -->
</dependencies>
```

**With Spring Boot Starters (Modern Way):**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**That's it!** One starter includes all necessary dependencies with compatible versions.

### What's Inside a Starter?

Let's examine `spring-boot-starter-web`:

```xml
<!-- spring-boot-starter-web/pom.xml -->
<dependencies>
    <!-- Core Spring MVC -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
    </dependency>
    
    <!-- Embedded Tomcat (default) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
    </dependency>
    
    <!-- Jackson for JSON -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    
    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Logging -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
    </dependency>
    
    <!-- Spring Boot core -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

### Creating a Complete Application

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
    </parent>
    
    <dependencies>
        <!-- Web API -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

With just these 6 starters, you have a complete, production-ready application stack!

### Benefits of Starter Dependencies

✅ **Version Compatibility:** All dependencies work together seamlessly
✅ **Reduced Configuration:** No need to research compatible versions
✅ **Faster Setup:** Create projects in minutes, not hours
✅ **Best Practices:** Spring Boot team curates optimal combinations
✅ **Easy Maintenance:** Update parent version to update all starters
✅ **Less Errors:** Avoid version conflicts and dependency hell

> **💡 Pro Tip**
> 
> Always use `spring-boot-starter-parent` as your Maven/Gradle parent. It manages all Spring Boot dependency versions, ensuring compatibility across your entire project.

### Custom Starters

You can create your own starters for organization-specific functionality:

```xml
<!-- company-starter-security/pom.xml -->
<project>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
    </parent>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <!-- Company-specific security libraries -->
        <dependency>
            <groupId>com.company</groupId>
            <artifactId>company-ldap</artifactId>
            <version>1.0.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.company</groupId>
            <artifactId>company-mfa</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</project>
```

Now all company projects can use `company-starter-security` for consistent security implementation.

---

## 5. Spring Boot Project Structure: Organizing Applications Properly

### Why Project Structure Matters

As applications grow, organization becomes increasingly important. A project containing hundreds of files can quickly become confusing without a clear structure. Good organization:

- Makes code easy to navigate
- Improves collaboration among team members
- Reduces merge conflicts
- Enhances maintainability
- Follows industry standards

### Standard Layered Architecture

Spring Boot doesn't enforce a specific structure, but most professional applications follow **layered architecture**:

```
src/main/java/com/example/demo/
├── controllers/          # Presentation Layer - Handle HTTP requests
│   ├── UserController.java
│   ├── OrderController.java
│   └── ProductController.java
│
├── services/             # Business Logic Layer - Core logic
│   ├── UserService.java
│   ├── OrderService.java
│   └── ProductService.java
│
├── repositories/         # Data Access Layer - Database operations
│   ├── UserRepository.java
│   ├── OrderRepository.java
│   └── ProductRepository.java
│
├── entities/             # Domain Models - Database entities
│   ├── User.java
│   ├── Order.java
│   └── Product.java
│
├── dtos/                 # Data Transfer Objects - API contracts
│   ├── UserDTO.java
│   ├── OrderRequest.java
│   └── ProductResponse.java
│
├── config/               # Configuration Classes
│   ├── SecurityConfig.java
│   ├── WebConfig.java
│   └── AppConfig.java
│
├── security/             # Security Components
│   ├── JwtAuthenticationFilter.java
│   ├── JwtTokenProvider.java
│   └── CustomUserDetailsService.java
│
├── exceptions/           # Exception Handling
│   ├── GlobalExceptionHandler.java
│   ├── ResourceNotFoundException.java
│   └── ErrorResponse.java
│
├── utils/                # Utility Classes
│   ├── DateUtils.java
│   ├── ValidationUtils.java
│   └── ApiResponseBuilder.java
│
└── DemoApplication.java  # Main Application Class

src/main/resources/
├── application.properties
├── application-dev.properties
├── application-prod.properties
├── application-test.properties
└── db/migration/
    ├── V1__create_users_table.sql
    ├── V2__create_orders_table.sql
    └── V3__create_products_table.sql

src/test/java/com/example/demo/
├── controllers/
├── services/
├── repositories/
└── integration/
```

### Layer Responsibilities

```mermaid
graph TB
    subgraph "Client"
        A[HTTP Request]
    end
    
    subgraph "Presentation Layer"
        B[Controller]
        B1[Request Validation]
        B2[Response Formatting]
    end
    
    subgraph "Business Layer"
        C[Service]
        C1[Business Logic]
        C2[Transaction Management]
        C3[Security Checks]
    end
    
    subgraph "Data Layer"
        D[Repository]
        D1[CRUD Operations]
        D2[Query Methods]
        D3[Database Access]
    end
    
    subgraph "Database"
        E[(Database)]
    end
    
    A --> B
    B --> B1
    B1 --> B2
    B2 --> C
    C --> C1
    C1 --> C2
    C2 --> C3
    C3 --> D
    D --> D1
    D1 --> D2
    D2 --> D3
    D3 --> E
    
    style A fill:#4ecdc4
    style B fill:#ffe66d
    style C fill:#ff6b6b
    style D fill:#95e1d3
    style E fill:#dda0dd
```

**Figure 5: Spring Boot Layered Architecture**

### Detailed Layer Breakdown

#### 1. Controller Layer (Presentation)
```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    private final UserService userService;
    
    // Constructor injection
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable @Min(1) Long id) {
        UserDTO user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDTO user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

**Responsibilities:**
- Handle HTTP requests/responses
- Validate incoming data
- Call appropriate service methods
- Return appropriate HTTP status codes
- Format responses (DTOs)

#### 2. Service Layer (Business Logic)
```java
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final PasswordEncoder passwordEncoder;
    
    public UserService(UserRepository userRepository, 
                      EmailService emailService,
                      PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        this.passwordEncoder = passwordEncoder;
    }
    
    public UserDTO getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        return UserDTO.fromEntity(user);
    }
    
    public UserDTO createUser(CreateUserRequest request) {
        // Business logic
        User user = new User();
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setName(request.getName());
        
        User savedUser = userRepository.save(user);
        
        // Side effects
        emailService.sendWelcomeEmail(savedUser.getEmail());
        
        return UserDTO.fromEntity(savedUser);
    }
}
```

**Responsibilities:**
- Implement business rules
- Orchestrate multiple repositories
- Manage transactions
- Handle security checks
- Coordinate with external services

#### 3. Repository Layer (Data Access)
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived query methods
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    List<User> findByNameContainingIgnoreCase(String name);
    
    // Custom query with @Query
    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findRecentUsers(@Param("date") LocalDateTime date);
    
    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateInactiveUsers(@Param("date") LocalDateTime date);
}
```

**Responsibilities:**
- Database CRUD operations
- Query execution
- Data mapping
- Transaction participation

#### 4. Entity Layer (Domain Models)
```java
@Entity
@Table(name = "users")
@EntityListeners(AuditingEntityListener.class)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @Column(nullable = false)
    private String name;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @Column(name = "is_active")
    private Boolean active = true;
    
    // Constructors, getters, setters
    // ...
}
```

**Responsibilities:**
- Represent database tables
- Define relationships
- Specify constraints
- Map to database schema

#### 5. DTO Layer (Data Transfer)
```java
// Request DTO
public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    String name,
    
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    String email,
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    String password
) {}

// Response DTO
public record UserDTO(
    Long id,
    String name,
    String email,
    LocalDateTime createdAt,
    Boolean active
) {
    public static UserDTO fromEntity(User user) {
        return new UserDTO(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.getCreatedAt(),
            user.getActive()
        );
    }
}
```

**Responsibilities:**
- Define API contracts
- Control data exposure
- Validate input data
- Format output data

### Package by Layer vs Package by Feature

#### Package by Layer (Traditional)
```
com.example.demo/
├── controller/
├── service/
├── repository/
├── entity/
└── config/
```

**Pros:**
- Clear separation of concerns
- Easy to find specific layer types
- Familiar to most developers

**Cons:**
- Feature changes require touching multiple packages
- Can lead to large packages as app grows

#### Package by Feature (Modern)
```
com.example.demo/
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   ├── User.java
│   └── UserDTO.java
├── order/
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   ├── Order.java
│   └── OrderDTO.java
└── product/
    ├── ProductController.java
    ├── ProductService.java
    ├── ProductRepository.java
    ├── Product.java
    └── ProductDTO.java
```

**Pros:**
- Feature cohesion (all related code together)
- Easier to understand feature implementation
- Better for microservices migration
- Reduces cross-package dependencies

**Cons:**
- Can duplicate some patterns
- Less familiar to beginners

> **💡 Recommendation**
> 
> For small to medium applications, **Package by Layer** works well. For large applications with many features, consider **Package by Feature** for better modularity.

### Best Practices

✅ **DO:**
- Follow consistent naming conventions
- Group related functionality together
- Keep controllers thin (delegate to services)
- Use interfaces for services when appropriate
- Separate configuration classes by concern
- Use meaningful package names

❌ **DON'T:**
- Mix layers in the same package
- Put business logic in controllers
- Create circular dependencies between layers
- Use generic names like `util` for everything
- Skip the DTO layer (expose entities directly)

### Complete Example Structure

```java
// Main Application
@SpringBootApplication
public class EcommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(EcommerceApplication.class, args);
    }
}

// Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // Security configuration
}

@Configuration
@EnableJpaAuditing
public class JpaConfig {
    // JPA configuration
}

// Exception Handling
@RestControllerAdvice
public class GlobalExceptionHandler {
    // Centralized exception handling
}

// Custom Exceptions
public class ResourceNotFoundException extends RuntimeException {
    // Custom exception
}
```

---

## 6. Bean Lifecycle: Understanding What Happens Behind the Scenes

### What is Bean Lifecycle?

Every object managed by Spring is called a **Bean**. But beans don't simply appear. They follow a lifecycle managed entirely by the IoC Container.

### Bean Lifecycle Phases

```mermaid
stateDiagram-v2
    [*] --> Instantiation: Bean Definition Found
    Instantiation --> PropertyPopulation: Instance Created
    PropertyPopulation --> DependencyInjection: Properties Set
    DependencyInjection --> AwareCallbacks: Dependencies Injected
    AwareCallbacks --> BeanPostProcessing: Aware Interfaces Called
    BeanPostProcessing --> Initialization: PostProcessBeforeInitialization
    Initialization --> CustomInit: @PostConstruct / afterPropertiesSet
    CustomInit --> Ready: PostProcessAfterInitialization
    Ready --> InUse: Bean Available for Use
    InUse --> Destruction: Context Closed
    Destruction --> Cleanup: @PreDestroy / DisposableBean
    Cleanup --> [*]: Bean Destroyed
    
    note right of Instantiation
        Spring creates bean instance
        using constructor
    end note
    
    note right of DependencyInjection
        Spring injects dependencies
        (constructor, setter, field)
    end note
    
    note right of CustomInit
        Custom initialization logic
        runs here
    end note
    
    note right of Cleanup
        Resource cleanup
        Connection closing
    end note
```

**Figure 6: Spring Bean Lifecycle State Diagram**

### Detailed Lifecycle Steps

#### 1. Instantiation
Spring creates an instance of the bean using the constructor.

```java
@Component
public class DataService {
    // Spring calls the constructor
    public DataService() {
        System.out.println("1. Bean instantiated");
    }
}
```

#### 2. Population of Properties
Spring sets all properties on the bean.

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    private String name;
    private int timeout;
    
    // Spring sets these properties
    public void setName(String name) {
        this.name = name;
    }
    
    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }
}
```

#### 3. Dependency Injection
Spring injects dependencies into the bean.

```java
@Component
public class UserService {
    private final UserRepository userRepository;
    
    // Dependency injected here
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

#### 4. Aware Callbacks
If the bean implements Aware interfaces, Spring calls the appropriate methods.

```java
@Component
public class CustomBean implements 
    ApplicationContextAware,
    BeanNameAware {
    
    private ApplicationContext context;
    private String beanName;
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.context = applicationContext;
        System.out.println("4. ApplicationContext set");
    }
    
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("4. Bean name set: " + name);
    }
}
```

**Common Aware Interfaces:**
- `ApplicationContextAware` - Get access to ApplicationContext
- `BeanNameAware` - Get the bean name
- `BeanFactoryAware` - Get access to BeanFactory
- `EnvironmentAware` - Get access to environment properties
- `ResourceLoaderAware` - Get access to resource loading

#### 5. BeanPostProcessor
`BeanPostProcessor` implementations can modify beans before and after initialization.

```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("5. Before initialization: " + beanName);
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("5. After initialization: " + beanName);
        return bean;
    }
}
```

#### 6. Initialization
Spring calls initialization callbacks.

```java
@Component
public class CacheService {
    
    // Option 1: @PostConstruct annotation (recommended)
    @PostConstruct
    public void init() {
        System.out.println("6. @PostConstruct - Initialize cache");
        initializeCache();
    }
    
    // Option 2: InitializingBean interface
    @Override
    public void afterPropertiesSet() {
        System.out.println("6. afterPropertiesSet - Initialize cache");
        initializeCache();
    }
    
    // Option 3: Custom init-method in @Bean
    // @Bean(initMethod = "init")
    
    private void initializeCache() {
        // Load cache data
        // Establish connections
        // Prepare resources
    }
}
```

#### 7. Ready for Use
The bean is now fully initialized and ready to be used by the application.

#### 8. Destruction
When the application context is closed, Spring destroys beans.

```java
@Component
public class ConnectionPool {
    private DataSource dataSource;
    
    // Option 1: @PreDestroy annotation (recommended)
    @PreDestroy
    public void cleanup() {
        System.out.println("8. @PreDestroy - Closing connections");
        closeConnections();
    }
    
    // Option 2: DisposableBean interface
    @Override
    public void destroy() {
        System.out.println("8. destroy - Closing connections");
        closeConnections();
    }
    
    // Option 3: Custom destroy-method in @Bean
    // @Bean(destroyMethod = "cleanup")
    
    private void closeConnections() {
        // Close database connections
        // Release resources
        // Stop background threads
    }
}
```

### Practical Example: Complete Lifecycle

```java
@Component
public class EmailService implements InitializingBean, DisposableBean {
    
    private JavaMailSender mailSender;
    private String fromAddress;
    private Connection connection;
    
    // 1. Constructor
    public EmailService(JavaMailSender mailSender) {
        this.mailSender = mailSender;
        System.out.println("1. EmailService instantiated");
    }
    
    // 2. Setter for properties
    public void setFromAddress(String fromAddress) {
        this.fromAddress = fromAddress;
    }
    
    // 3. Dependency injection (already done via constructor)
    
    // 4. Aware callbacks (if implemented)
    
    // 5. BeanPostProcessor (if configured)
    
    // 6. Initialization - Option 1: @PostConstruct
    @PostConstruct
    public void init() {
        System.out.println("6. @PostConstruct called");
        validateConfiguration();
    }
    
    // 6. Initialization - Option 2: InitializingBean
    @Override
    public void afterPropertiesSet() {
        System.out.println("6. afterPropertiesSet called");
        establishConnection();
    }
    
    private void validateConfiguration() {
        if (fromAddress == null || fromAddress.isEmpty()) {
            throw new IllegalStateException("From address is required");
        }
    }
    
    private void establishConnection() {
        // Connect to SMTP server
        this.connection = mailSender.createConnection();
        System.out.println("   Connection established");
    }
    
    // 7. Bean is ready to use
    public void sendEmail(String to, String subject, String body) {
        // Send email using connection
        System.out.println("   Sending email...");
    }
    
    // 8. Destruction - Option 1: @PreDestroy
    @PreDestroy
    public void destroyConnection() {
        System.out.println("8. @PreDestroy called");
        if (connection != null && connection.isOpen()) {
            connection.close();
            System.out.println("   Connection closed");
        }
    }
    
    // 8. Destruction - Option 2: DisposableBean
    @Override
    public void destroy() {
        System.out.println("8. destroy() called");
        destroyConnection();
    }
}
```

### Lifecycle Annotations Comparison

| Approach | Initialization | Destruction | Pros | Cons |
|----------|---------------|-------------|------|------|
| `@PostConstruct` / `@PreDestroy` | ✅ | ✅ | Simple, standard, no Spring dependency | Requires JSR-250 |
| `InitializingBean` / `DisposableBean` | ✅ | ✅ | Spring-native, more control | Tightly couples to Spring |
| Custom init/destroy method | ✅ | ✅ | Flexible, no interface needed | String-based, error-prone |
| `@Bean(initMethod, destroyMethod)` | ✅ | ✅ | Declarative, type-safe | Only works with @Bean methods |

> **💡 Best Practice**
> 
> Use `@PostConstruct` and `@PreDestroy` for most cases. They're part of the Java standard (JSR-250) and keep your code clean. Use `InitializingBean`/`DisposableBean` only when you need access to bean metadata.

### Scopes and Lifecycle

Bean scope affects lifecycle:

| Scope | Lifecycle | Use Case |
|-------|-----------|----------|
| **Singleton** (default) | Created once, lives for entire context | Most beans |
| **Prototype** | Created each time requested | Stateful beans |
| **Request** | Created per HTTP request | Web request beans |
| **Session** | Created per HTTP session | User session data |
| **Application** | Created once per ServletContext | Application-wide beans |

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class PrototypeBean {
    // New instance created each time
    // No @PreDestroy called (Spring doesn't track prototypes)
}

@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean {
    // New instance per HTTP request
    // Destroyed after request completes
}
```

### Practical Use Cases

#### Use Case 1: Cache Initialization
```java
@Component
public class ProductCache {
    
    private final ProductRepository productRepository;
    private final Map<Long, Product> cache = new ConcurrentHashMap<>();
    
    public ProductCache(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    @PostConstruct
    public void loadCache() {
        System.out.println("Loading products into cache...");
        List<Product> products = productRepository.findAll();
        products.forEach(product -> cache.put(product.getId(), product));
        System.out.println("Cache loaded with " + cache.size() + " products");
    }
    
    @PreDestroy
    public void clearCache() {
        System.out.println("Clearing cache...");
        cache.clear();
    }
    
    public Product getProduct(Long id) {
        return cache.get(id);
    }
}
```

#### Use Case 2: Database Connection Pool
```java
@Component
public class ConnectionPoolManager {
    
    private HikariDataSource dataSource;
    
    @PostConstruct
    public void initialize() {
        System.out.println("Initializing connection pool...");
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(env.getProperty("db.url"));
        config.setUsername(env.getProperty("db.username"));
        config.setPassword(env.getProperty("db.password"));
        config.setMaximumPoolSize(10);
        
        this.dataSource = new HikariDataSource(config);
        System.out.println("Connection pool initialized");
    }
    
    @PreDestroy
    public void shutdown() {
        System.out.println("Shutting down connection pool...");
        if (dataSource != null) {
            dataSource.close();
        }
    }
    
    public DataSource getDataSource() {
        return dataSource;
    }
}
```

#### Use Case 3: Scheduled Task Initialization
```java
@Component
public class ReportScheduler {
    
    private final ReportService reportService;
    private ScheduledExecutorService executor;
    
    public ReportScheduler(ReportService reportService) {
        this.reportService = reportService;
    }
    
    @PostConstruct
    public void startScheduler() {
        System.out.println("Starting report scheduler...");
        executor = Executors.newScheduledThreadPool(5);
        
        // Schedule daily reports
        executor.scheduleAtFixedRate(
            this::generateDailyReports,
            0, 24, TimeUnit.HOURS
        );
    }
    
    @PreDestroy
    public void stopScheduler() {
        System.out.println("Stopping report scheduler...");
        if (executor != null) {
            executor.shutdown();
        }
    }
    
    private void generateDailyReports() {
        reportService.generateDailyReport();
    }
}
```

### Common Pitfalls

❌ **Mistake 1: Heavy Logic in @PostConstruct**
```java
@Component
public class BadExample {
    @PostConstruct
    public void init() {
        // ❌ Don't do heavy processing here
        // This blocks application startup
        processMillionsOfRecords();
    }
}
```

✅ **Better Approach:**
```java
@Component
public class GoodExample {
    @PostConstruct
    public void init() {
        // ✅ Quick initialization only
        initializeCache();
    }
    
    @EventListener(ApplicationReadyEvent.class)
    public void loadData() {
        // ✅ Heavy processing after startup
        // Application is ready to serve requests
        processMillionsOfRecords();
    }
}
```

❌ **Mistake 2: Forgetting Cleanup**
```java
@Component
public class ResourceLeak {
    private FileInputStream stream;
    
    @PostConstruct
    public void init() {
        stream = new FileInputStream("data.txt");
        // ❌ Never closed!
    }
}
```

✅ **Correct Approach:**
```java
@Component
public class ResourceManager {
    private FileInputStream stream;
    
    @PostConstruct
    public void init() {
        stream = new FileInputStream("data.txt");
    }
    
    @PreDestroy
    public void cleanup() {
        // ✅ Always clean up resources
        if (stream != null) {
            try {
                stream.close();
            } catch (IOException e) {
                log.error("Error closing stream", e);
            }
        }
    }
}
```

---

## 7. Configuration Properties — Making Applications Flexible

### The Problem with Hardcoding

Hardcoding values inside source code creates maintenance problems. Imagine storing database URLs, passwords, API keys, server ports, and email credentials inside Java classes. Every environment would require modifying source code before deployment.

### External Configuration in Spring Boot

Spring Boot uses external configuration files to separate configuration from code:

**Primary Configuration Files:**
- `application.properties`
- `application.yml` (or `application.yaml`)

### Configuration File Formats

#### Properties Format
```properties
# application.properties
server.port=8080
server.servlet.context-path=/api

spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=admin
spring.datasource.password=secret

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

app.name=My Application
app.version=1.0.0
app.features.enable-notifications=true
app.features.enable-analytics=false
```

#### YAML Format (More Readable)
```yaml
# application.yml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: admin
    password: secret
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

app:
  name: My Application
  version: 1.0.0
  features:
    enable-notifications: true
    enable-analytics: false
```

### Accessing Configuration Values

#### Method 1: @Value Annotation
```java
@Component
public class EmailService {
    
    @Value("${app.email.from-address}")
    private String fromAddress;
    
    @Value("${app.email.smtp.host:localhost}") // Default value
    private String smtpHost;
    
    @Value("${app.email.smtp.port:25}")
    private int smtpPort;
    
    public void sendEmail(String to, String subject, String body) {
        // Use configuration values
        System.out.println("Sending from: " + fromAddress);
        System.out.println("Using SMTP: " + smtpHost + ":" + smtpPort);
    }
}
```

#### Method 2: @ConfigurationProperties (Recommended)
```java
@Component
@ConfigurationProperties(prefix = "app.email")
public class EmailConfig {
    // Fields must match property names
    private String fromAddress;
    private String replyToAddress;
    private SmtpConfig smtp;
    
    // Nested configuration class
    public static class SmtpConfig {
        private String host;
        private int port;
        private boolean auth;
        private boolean starttls;
        
        // Getters and setters
        public String getHost() { return host; }
        public void setHost(String host) { this.host = host; }
        public int getPort() { return port; }
        public void setPort(int port) { this.port = port; }
        public boolean isAuth() { return auth; }
        public void setAuth(boolean auth) { this.auth = auth; }
        public boolean isStarttls() { return starttls; }
        public void setStarttls(boolean starttls) { this.starttls = starttls; }
    }
    
    // Getters and setters
    public String getFromAddress() { return fromAddress; }
    public void setFromAddress(String fromAddress) { this.fromAddress = fromAddress; }
    public String getReplyToAddress() { return replyToAddress; }
    public void setReplyToAddress(String replyToAddress) { this.replyToAddress = replyToAddress; }
    public SmtpConfig getSmtp() { return smtp; }
    public void setSmtp(SmtpConfig smtp) { this.smtp = smtp; }
}

// Usage
@Service
public class EmailService {
    private final EmailConfig emailConfig;
    
    public EmailService(EmailConfig emailConfig) {
        this.emailConfig = emailConfig;
    }
    
    public void sendEmail(String to, String subject, String body) {
        String from = emailConfig.getFromAddress();
        String host = emailConfig.getSmtp().getHost();
        int port = emailConfig.getSmtp().getPort();
        
        // Use configuration
    }
}
```

**Configuration file:**
```yaml
app:
  email:
    from-address: noreply@example.com
    reply-to-address: support@example.com
    smtp:
      host: smtp.gmail.com
      port: 587
      auth: true
      starttls: true
```

#### Method 3: Using Records (Java 16+)
```java
@Component
@ConfigurationProperties(prefix = "app.email")
public record EmailConfig(
    String fromAddress,
    String replyToAddress,
    SmtpConfig smtp
) {
    public record SmtpConfig(
        String host,
        int port,
        boolean auth,
        boolean starttls
    ) {}
}

// Spring automatically binds properties to this record
```

### Environment-Specific Configuration

Spring Boot supports multiple configuration files for different environments:

```
application.properties          # Default (all environments)
application-dev.properties      # Development
application-test.properties     # Testing
application-staging.properties  # Staging
application-prod.properties     # Production
```

**Activate profiles:**
```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Environment variable
SPRING_PROFILES_ACTIVE=prod java -jar app.jar

# application.properties
spring.profiles.active=prod
```

**Profile-specific properties:**
```yaml
# application.yml (common)
server:
  port: 8080

spring:
  datasource:
    driver-class-name: org.postgresql.Driver

---
# application-dev.yml
spring:
  config:
    activate:
      on-profile: dev
  
  datasource:
    url: jdbc:postgresql://localhost:5432/devdb
    username: dev_user
    password: dev_pass
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

---
# application-prod.yml
spring:
  config:
    activate:
      on-profile: prod
  
  datasource:
    url: jdbc:postgresql://prod-server:5432/proddb
    username: ${DB_USERNAME} # From environment variable
    password: ${DB_PASSWORD}
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

### Configuration Priority

Spring Boot loads configuration from multiple sources with this priority (highest to lowest):

1. **Command line arguments**
2. **OS environment variables**
3. **Java System properties** (`-D` arguments)
4. **Application properties outside JAR**
5. **Application properties inside JAR**
6. **@PropertySource annotations**
7. **Default properties**

```bash
# Command line overrides everything
java -jar app.jar --server.port=9090

# Environment variables (use underscores)
SERVER_PORT=9090 java -jar app.jar

# System properties
java -Dserver.port=9090 -jar app.jar
```

### Type-Safe Configuration

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    private String name;
    private String version;
    private int maxUploadSize;
    private boolean enableCache;
    private List<String> allowedOrigins;
    private Map<String, String> apiKeys;
    
    // Getters and setters
}

// application.yml
app:
  name: MyApp
  version: 1.0.0
  max-upload-size: 10485760
  enable-cache: true
  allowed-origins:
    - https://example.com
    - https://app.example.com
  api-keys:
    service1: key123
    service2: key456
```

### Validation of Configuration

```java
@Component
@ConfigurationProperties(prefix = "app")
@Data // Lombok for getters/setters
@Valid
public class AppConfig {
    @NotBlank
    private String name;
    
    @Min(1)
    @Max(65535)
    private int port;
    
    @Email
    private String adminEmail;
    
    @Pattern(regexp = "^(dev|test|prod)$")
    private String environment;
}

// If validation fails, application won't start
```

### External Configuration Sources

Spring Boot can load configuration from various sources:

```java
@PropertySource("classpath:custom.properties")
@PropertySource("file:${config.location}/app.properties")
@PropertySource(value = "https://example.com/config.properties")
public class AppConfig {
    // Configuration beans
}
```

### Best Practices

✅ **DO:**
- Use `@ConfigurationProperties` for complex configurations
- Use profiles for environment-specific settings
- Externalize all environment-specific values
- Use environment variables for sensitive data
- Provide sensible defaults
- Document all configuration options

❌ **DON'T:**
- Hardcode values in Java classes
- Commit sensitive data to version control
- Use the same configuration for all environments
- Create deeply nested configuration structures
- Forget to validate configuration

### Security Considerations

```yaml
# ❌ Never do this in version control
spring:
  datasource:
    password: SuperSecret123!

# ✅ Use environment variables
spring:
  datasource:
    password: ${DB_PASSWORD}

# ✅ Or use external configuration
spring.config.import: optional:configtree:/etc/app/config/
```

### Real-World Example: Complete Configuration

```yaml
# application.yml
server:
  port: 8080
  error:
    include-message: always
    include-binding-errors: always

spring:
  application:
    name: ecommerce-api
  
  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/ecommerce}
    username: ${DATABASE_USERNAME:postgres}
    password: ${DATABASE_PASSWORD:postgres}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
  
  jpa:
    hibernate:
      ddl-auto: validate
    open-in-view: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
  
  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    time-to-live: 3600000
  
  mail:
    host: ${MAIL_HOST:smtp.gmail.com}
    port: ${MAIL_PORT:587}
    username: ${MAIL_USERNAME}
    password: ${MAIL_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true

app:
  jwt:
    secret: ${JWT_SECRET:defaultSecretKeyChangeInProduction}
    expiration-ms: 86400000 # 24 hours
  
  cors:
    allowed-origins:
      - https://example.com
      - https://app.example.com
    allowed-methods:
      - GET
      - POST
      - PUT
      - DELETE
  
  upload:
    max-file-size: 10MB
    allowed-types:
      - image/jpeg
      - image/png
      - application/pdf

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
```

---

## 8. REST Controllers: The Entry Point of Every API

### What is a REST Controller?

Every Spring Boot application that exposes APIs begins with a controller. A controller acts as the **communication bridge** between the outside world and your application.

### The Request Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant L as Load Balancer
    participant S as Security Filter
    participant Ctrl as Controller
    participant Svc as Service
    participant Repo as Repository
    participant DB as Database
    
    C->>L: HTTP Request
    L->>S: Forward Request
    S->>S: Authenticate & Authorize
    S->>Ctrl: Authorized Request
    
    Ctrl->>Ctrl: Validate Input
    Ctrl->>Svc: Call Business Logic
    Svc->>Repo: Data Operation
    Repo->>DB: SQL Query
    DB-->>Repo: Result
    Repo-->>Svc: Entity
    Svc-->>Ctrl: DTO
    Ctrl-->>C: HTTP Response
    
    Note over Ctrl: Thin layer -<br/>no business logic
    Note over Svc: Business logic<br/>lives here
```

**Figure 7: REST Controller Request Flow**

### Creating a REST Controller

```java
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {
    
    private final ProductService productService;
    
    // Constructor injection
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    /**
     * Get all products
     * GET /api/products
     */
    @GetMapping
    public ResponseEntity<List<ProductResponse>> getAllProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String category) {
        
        List<ProductResponse> products = productService.getAllProducts(page, size, category);
        return ResponseEntity.ok(products);
    }
    
    /**
     * Get product by ID
     * GET /api/products/{id}
     */
    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProductById(
            @PathVariable @Min(1) Long id) {
        
        ProductResponse product = productService.getProductById(id);
        return ResponseEntity.ok(product);
    }
    
    /**
     * Create new product
     * POST /api/products
     */
    @PostMapping
    public ResponseEntity<ProductResponse> createProduct(
            @Valid @RequestBody CreateProductRequest request) {
        
        ProductResponse product = productService.createProduct(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(product.id())
            .toUri();
        
        return ResponseEntity.created(location).body(product);
    }
    
    /**
     * Update product
     * PUT /api/products/{id}
     */
    @PutMapping("/{id}")
    public ResponseEntity<ProductResponse> updateProduct(
            @PathVariable @Min(1) Long id,
            @Valid @RequestBody UpdateProductRequest request) {
        
        ProductResponse product = productService.updateProduct(id, request);
        return ResponseEntity.ok(product);
    }
    
    /**
     * Delete product
     * DELETE /api/products/{id}
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable @Min(1) Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Controller Best Practices

#### ✅ Keep Controllers Thin

```java
// ❌ BAD: Business logic in controller
@RestController
public class BadController {
    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest request) {
        // ❌ Business logic in controller
        Order order = new Order();
        order.setCustomerId(request.getCustomerId());
        order.setTotal(calculateTotal(request)); // ❌ Business logic
        
        for (OrderItem item : request.getItems()) {
            // ❌ Business logic
            if (item.getQuantity() > 100) {
                throw new ValidationException("Quantity too high");
            }
            order.addItem(item);
        }
        
        // ❌ Database logic in controller
        Order saved = orderRepository.save(order);
        
        // ❌ Email logic in controller
        emailService.sendOrderConfirmation(saved);
        
        return saved;
    }
}

// ✅ GOOD: Controller delegates to service
@RestController
public class GoodController {
    private final OrderService orderService;
    
    public GoodController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping("/orders")
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody OrderRequest request) {
        
        OrderResponse order = orderService.createOrder(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(order.id())
            .toUri();
        
        return ResponseEntity.created(location).body(order);
    }
}
```

#### ✅ Use DTOs, Not Entities

```java
// ❌ BAD: Exposing entity directly
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id); // Returns entity with password!
}

// ✅ GOOD: Return DTO
@GetMapping("/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
    UserResponse user = userService.getUserById(id);
    return ResponseEntity.ok(user);
}

// DTO excludes sensitive fields
public record UserResponse(
    Long id,
    String name,
    String email,
    LocalDateTime createdAt
    // No password field!
) {
    public static UserResponse fromEntity(User user) {
        return new UserResponse(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.getCreatedAt()
        );
    }
}
```

### HTTP Status Codes

Always return appropriate HTTP status codes:

| Operation | Success Status | Error Status | Example |
|-----------|---------------|--------------|---------|
| **GET** (retrieve) | 200 OK | 404 Not Found | Get user by ID |
| **POST** (create) | 201 Created | 400 Bad Request | Create new user |
| **PUT** (update) | 200 OK | 404 Not Found | Update user |
| **PATCH** (partial update) | 200 OK | 404 Not Found | Update email only |
| **DELETE** | 204 No Content | 404 Not Found | Delete user |
| **Validation Error** | - | 400 Bad Request | Invalid input data |
| **Unauthorized** | - | 401 Unauthorized | Missing/invalid token |
| **Forbidden** | - | 403 Forbidden | Insufficient permissions |
| **Server Error** | - | 500 Internal Server Error | Unexpected error |

### Response Entity Builder

```java
@GetMapping("/{id}")
public ResponseEntity<ProductResponse> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);
    
    if (product == null) {
        return ResponseEntity.notFound().build(); // 404
    }
    
    if (!product.isActive()) {
        return ResponseEntity.status(HttpStatus.GONE).build(); // 410
    }
    
    return ResponseEntity.ok(ProductResponse.fromEntity(product)); // 200
}

@PostMapping
public ResponseEntity<ProductResponse> createProduct(@Valid @RequestBody CreateProductRequest request) {
    ProductResponse product = productService.create(request);
    
    // 201 Created with Location header
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(product.id());
    
    return ResponseEntity.created(location).body(product);
}
```

### Common Annotations

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@RestController` | Marks class as REST controller | `@RestController` |
| `@RequestMapping` | Base URL mapping | `@RequestMapping("/api/users")` |
| `@GetMapping` | Handle GET requests | `@GetMapping("/{id}")` |
| `@PostMapping` | Handle POST requests | `@PostMapping` |
| `@PutMapping` | Handle PUT requests | `@PutMapping("/{id}")` |
| `@PatchMapping` | Handle PATCH requests | `@PatchMapping("/{id}")` |
| `@DeleteMapping` | Handle DELETE requests | `@DeleteMapping("/{id}")` |
| `@PathVariable` | Extract path variable | `@PathVariable Long id` |
| `@RequestParam` | Extract query parameter | `@RequestParam String name` |
| `@RequestBody` | Bind request body | `@RequestBody UserRequest request` |
| `@RequestHeader` | Extract header value | `@RequestHeader("Authorization")` |
| `@Valid` | Trigger validation | `@Valid @RequestBody` |
| `@Validated` | Enable validation | `@Validated` on class |

### Complete Example: Product API

```java
@RestController
@RequestMapping("/api/v1/products")
@Validated
@Slf4j
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    /**
     * Get all products with pagination and filtering
     * GET /api/v1/products?page=0&size=20&category=electronics&minPrice=100&maxPrice=1000
     */
    @GetMapping
    public ResponseEntity<Page<ProductResponse>> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) @DecimalMin("0.0") BigDecimal minPrice,
            @RequestParam(required = false) @DecimalMax("999999.99") BigDecimal maxPrice,
            @RequestParam(required = false) @Pattern(regexp = "name|price|createdAt") String sortBy,
            @RequestParam(required = false) @Pattern(regexp = "asc|desc") String sortDir) {
        
        log.info("Fetching products - page: {}, size: {}, category: {}", 
            page, size, category);
        
        Pageable pageable = PageRequest.of(
            page, 
            size, 
            Sort.by(Sort.Direction.fromString(sortDir != null ? sortDir : "asc"), 
                    sortBy != null ? sortBy : "id")
        );
        
        Page<ProductResponse> products = productService.findAll(category, minPrice, maxPrice, pageable);
        return ResponseEntity.ok(products);
    }
    
    /**
     * Get product by ID
     * GET /api/v1/products/{id}
     */
    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProductById(
            @PathVariable @Positive(message = "Product ID must be positive") Long id) {
        
        log.info("Fetching product with ID: {}", id);
        
        ProductResponse product = productService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found with id: " + id));
        
        return ResponseEntity.ok(product);
    }
    
    /**
     * Search products
     * GET /api/v1/products/search?q=laptop
     */
    @GetMapping("/search")
    public ResponseEntity<List<ProductResponse>> searchProducts(
            @RequestParam @NotBlank @Size(min = 2, max = 100) String q) {
        
        log.info("Searching products with query: {}", q);
        
        List<ProductResponse> results = productService.search(q);
        return ResponseEntity.ok(results);
    }
    
    /**
     * Create new product
     * POST /api/v1/products
     */
    @PostMapping
    public ResponseEntity<ProductResponse> createProduct(
            @Valid @RequestBody CreateProductRequest request,
            Authentication authentication) {
        
        String currentUser = authentication.getName();
        log.info("Creating product by user: {}", currentUser);
        
        ProductResponse product = productService.create(request, currentUser);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(product.id());
        
        return ResponseEntity.created(location).body(product);
    }
    
    /**
     * Update product
     * PUT /api/v1/products/{id}
     */
    @PutMapping("/{id}")
    public ResponseEntity<ProductResponse> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody UpdateProductRequest request) {
        
        log.info("Updating product with ID: {}", id);
        
        ProductResponse product = productService.update(id, request);
        return ResponseEntity.ok(product);
    }
    
    /**
     * Partial update product
     * PATCH /api/v1/products/{id}
     */
    @PatchMapping("/{id}")
    public ResponseEntity<ProductResponse> partialUpdateProduct(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        
        log.info("Partially updating product with ID: {}", id);
        
        ProductResponse product = productService.partialUpdate(id, updates);
        return ResponseEntity.ok(product);
    }
    
    /**
     * Delete product
     * DELETE /api/v1/products/{id}
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        log.info("Deleting product with ID: {}", id);
        
        productService.delete(id);
        return ResponseEntity.noContent().build(); // 204 No Content
    }
    
    /**
     * Upload product image
     * POST /api/v1/products/{id}/images
     */
    @PostMapping(value = "/{id}/images", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<ImageResponse> uploadImage(
            @PathVariable Long id,
            @RequestParam("file") MultipartFile file) {
        
        log.info("Uploading image for product ID: {}", id);
        
        if (file.isEmpty()) {
            throw new ValidationException("File cannot be empty");
        }
        
        if (file.getSize() > 10 * 1024 * 1024) {
            throw new ValidationException("File size must be less than 10MB");
        }
        
        ImageResponse image = productService.uploadImage(id, file);
        return ResponseEntity.ok(image);
    }
}
```

### Exception Handling in Controllers

```java
@RestController
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        log.error("Resource not found: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.NOT_FOUND.value(),
            "Resource Not Found",
            ex.getMessage(),
            null
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        log.error("Validation failed: {}", ex.getMessage());
        
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .toList();
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            "Input validation failed",
            errors
        );
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        log.error("Access denied: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.FORBIDDEN.value(),
            "Access Denied",
            "You don't have permission to access this resource",
            null
        );
        
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error: {}", ex.getMessage(), ex);
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal Server Error",
            "An unexpected error occurred",
            null
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}

// Error response DTO
public record ErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    String message,
    List<String> details
) {}
```

---

## 9. Request Mapping: Connecting URLs to Java Methods

### How Request Mapping Works

Whenever a request reaches your application, Spring compares the requested URL and HTTP method against mappings defined inside controllers.

### HTTP Methods and Their Purpose

| Method | Purpose | Idempotent | Safe | Example Use Case |
|--------|---------|------------|------|------------------|
| **GET** | Retrieve resources | ✅ Yes | ✅ Yes | Get user, list products |
| **POST** | Create resources | ❌ No | ❌ No | Create user, place order |
| **PUT** | Replace entire resource | ✅ Yes | ❌ No | Update user profile |
| **PATCH** | Partial update | ❌ No | ❌ No | Update email only |
| **DELETE** | Remove resources | ✅ Yes | ❌ No | Delete user, cancel order |

### Basic Request Mapping

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // GET /api/users
    @GetMapping
    public List<UserResponse> getAllUsers() {
        return userService.findAll();
    }
    
    // GET /api/users/123
    @GetMapping("/{id}")
    public UserResponse getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // POST /api/users
    @PostMapping
    public UserResponse createUser(@RequestBody CreateUserRequest request) {
        return userService.create(request);
    }
    
    // PUT /api/users/123
    @PutMapping("/{id}")
    public UserResponse updateUser(@PathVariable Long id, 
                                   @RequestBody UpdateUserRequest request) {
        return userService.update(id, request);
    }
    
    // DELETE /api/users/123
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Path Variables

Path variables extract dynamic values from the URL:

```java
@RestController
@RequestMapping("/api")
public class MultiResourceController {
    
    // GET /api/users/123/orders/456
    @GetMapping("/users/{userId}/orders/{orderId}")
    public OrderResponse getOrder(
            @PathVariable Long userId,
            @PathVariable Long orderId) {
        
        return orderService.findByUserIdAndOrderId(userId, orderId);
    }
    
    // GET /api/products/electronics/123
    @GetMapping("/products/{category}/{id}")
    public ProductResponse getProduct(
            @PathVariable String category,
            @PathVariable Long id) {
        
        return productService.findByCategoryAndId(category, id);
    }
}
```

### Request Parameters

Query parameters for filtering, sorting, and pagination:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // GET /api/products?category=electronics&minPrice=100&maxPrice=1000&page=0&size=20
    @GetMapping
    public Page<ProductResponse> getProducts(
            @RequestParam(required = false) String category,
            @RequestParam(required = false) @DecimalMin("0.0") BigDecimal minPrice,
            @RequestParam(required = false) @DecimalMax("999999.99") BigDecimal maxPrice,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        
        return productService.filter(category, minPrice, maxPrice, page, size);
    }
    
    // GET /api/products/search?q=laptop&inStock=true
    @GetMapping("/search")
    public List<ProductResponse> search(
            @RequestParam @NotBlank String q,
            @RequestParam(defaultValue = "false") boolean inStock) {
        
        return productService.search(q, inStock);
    }
}
```

### Request Headers

```java
@RestController
@RequestMapping("/api")
public class HeaderController {
    
    // GET /api/profile
    @GetMapping("/profile")
    public UserProfile getProfile(
            @RequestHeader("Authorization") String authorizationHeader,
            @RequestHeader(value = "X-Request-ID", required = false) String requestId) {
        
        String token = authorizationHeader.replace("Bearer ", "");
        String trackingId = requestId != null ? requestId : UUID.randomUUID().toString();
        
        return userService.getProfileFromToken(token, trackingId);
    }
    
    // POST /api/data
    @PostMapping("/data")
    public ResponseEntity<DataResponse> processData(
            @RequestBody DataRequest request,
            @RequestHeader("Content-Type") String contentType,
            @RequestHeader(value = "X-API-Key", required = false) String apiKey) {
        
        // Process based on headers
        return ResponseEntity.ok(dataService.process(request, contentType, apiKey));
    }
}
```

### Request Body

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    // POST /api/orders
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        
        // Spring automatically deserializes JSON to CreateOrderRequest
        OrderResponse order = orderService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
    
    // PUT /api/orders/123
    @PutMapping("/{id}")
    public ResponseEntity<OrderResponse> updateOrder(
            @PathVariable Long id,
            @Valid @RequestBody UpdateOrderRequest request) {
        
        OrderResponse order = orderService.update(id, request);
        return ResponseEntity.ok(order);
    }
}

// Request DTO with validation
public record CreateOrderRequest(
    @NotNull(message = "Customer ID is required")
    Long customerId,
    
    @NotEmpty(message = "Order must contain at least one item")
    List<@Valid OrderItemRequest> items,
    
    @Size(max = 500, message = "Notes cannot exceed 500 characters")
    String notes
) {}

public record OrderItemRequest(
    @NotNull(message = "Product ID is required")
    Long productId,
    
    @Min(value = 1, message = "Quantity must be at least 1")
    @Max(value = 100, message = "Quantity cannot exceed 100")
    Integer quantity
) {}
```

### Advanced Mapping Patterns

#### Multiple Mappings for Same Method

```java
@RestController
@RequestMapping("/api")
public class FlexibleController {
    
    // Handle both GET /api/users and GET /api/customers
    @GetMapping(value = {"/users", "/customers"})
    public List<UserResponse> getUsers() {
        return userService.findAll();
    }
    
    // Multiple paths
    @GetMapping(value = {"/products", "/items", "/catalog"})
    public List<ProductResponse> getProducts() {
        return productService.findAll();
    }
}
```

#### Conditional Request Mapping

```java
@RestController
@RequestMapping("/api")
public class ConditionalController {
    
    // Only handle GET requests with Accept: application/json
    @GetMapping(value = "/data", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<DataResponse> getJsonData() {
        return ResponseEntity.ok(dataService.getData());
    }
    
    // Only handle GET requests with Accept: application/xml
    @GetMapping(value = "/data", produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<DataResponse> getXmlData() {
        return ResponseEntity.ok(dataService.getData());
    }
    
    // Only for POST requests with Content-Type: application/json
    @PostMapping(value = "/data", consumes = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<DataResponse> processJson(@RequestBody DataRequest request) {
        return ResponseEntity.ok(dataService.process(request));
    }
}
```

#### Custom Request Conditions

```java
@RestController
@RequestMapping("/api")
public class CustomConditionController {
    
    // Only for requests with specific header
    @GetMapping(value = "/special", headers = "X-Custom-Header=true")
    public ResponseEntity<String> getSpecialData() {
        return ResponseEntity.ok("Special data");
    }
    
    // Only for specific parameter
    @GetMapping(value = "/filtered", params = "filter=true")
    public ResponseEntity<List<DataResponse>> getFilteredData() {
        return ResponseEntity.ok(dataService.getFiltered());
    }
    
    // Only for specific content type
    @PostMapping(value = "/upload", consumes = "multipart/form-data")
    public ResponseEntity<String> uploadFile(@RequestParam MultipartFile file) {
        return ResponseEntity.ok("File uploaded");
    }
}
```

### URI Templates

```java
@RestController
@RequestMapping("/api")
public class UriTemplateController {
    
    // GET /api/users/123
    @GetMapping("/users/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // GET /api/users/123/orders/456
    @GetMapping("/users/{userId}/orders/{orderId}")
    public OrderResponse getOrder(
            @PathVariable Long userId,
            @PathVariable Long orderId) {
        return orderService.findByUserIdAndOrderId(userId, orderId);
    }
    
    // GET /api/files/documents/report.pdf
    @GetMapping("/files/{folder}/{filename:.+}")
    public ResponseEntity<Resource> getFile(
            @PathVariable String folder,
            @PathVariable String filename) {
        
        Path file = storageService.getFile(folder, filename);
        Resource resource = new UrlResource(file.toUri());
        
        return ResponseEntity.ok()
            .contentType(MediaType.APPLICATION_PDF)
            .body(resource);
    }
    
    // GET /api/users?name=John&age=30
    @GetMapping("/users")
    public List<UserResponse> searchUsers(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) Integer age) {
        
        return userService.search(name, age);
    }
}
```

### Matrix Variables

Matrix variables allow you to send multiple parameters for a single path variable:

```java
@RestController
@RequestMapping("/api")
public class MatrixController {
    
    // GET /api/cars;color=red;color=blue;make=toyota
    @GetMapping(value = "/cars", produces = MediaType.APPLICATION_JSON_VALUE)
    public List<Car> getCars(
            @MatrixVariable Map<String, List<String>> colors,
            @MatrixVariable(required = false) String make) {
        
        List<String> colorList = colors.getOrDefault("color", List.of());
        return carService.findByColorsAndMake(colorList, make);
    }
}

// URL: /api/cars;color=red;color=blue;make=toyota
// Requires: spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

### Consumable and Producible Media Types

```java
@RestController
@RequestMapping("/api")
public class MediaTypeController {
    
    // Only accepts JSON
    @PostMapping(value = "/data", consumes = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<DataResponse> createJson(@RequestBody DataRequest request) {
        return ResponseEntity.ok(dataService.create(request));
    }
    
    // Only accepts XML
    @PostMapping(value = "/data", consumes = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<DataResponse> createXml(@RequestBody DataRequest request) {
        return ResponseEntity.ok(dataService.create(request));
    }
    
    // Returns JSON
    @GetMapping(value = "/data/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<DataResponse> getJson(@PathVariable Long id) {
        return ResponseEntity.ok(dataService.findById(id));
    }
    
    // Returns XML
    @GetMapping(value = "/data/{id}", produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<DataResponse> getXml(@PathVariable Long id) {
        return ResponseEntity.ok(dataService.findById(id));
    }
}
```

### Best Practices

✅ **DO:**
- Use nouns for resource names (`/users`, not `/getUsers`)
- Use plural nouns (`/products`, not `/product`)
- Use HTTP methods correctly (GET for read, POST for create, etc.)
- Version your APIs (`/api/v1/users`)
- Use path variables for resource identifiers
- Use query parameters for filtering, sorting, pagination
- Return appropriate status codes
- Document your API endpoints

❌ **DON'T:**
- Use verbs in URLs (`/createUser`, `/getAllProducts`)
- Mix REST and RPC styles
- Use GET for operations that modify data
- Expose internal implementation details in URLs
- Forget to validate path variables

### REST API Design Example

```
✅ Good REST API Design:

GET    /api/v1/products              # List all products
GET    /api/v1/products/123          # Get product 123
POST   /api/v1/products              # Create product
PUT    /api/v1/products/123          # Update product 123
PATCH  /api/v1/products/123          # Partial update product 123
DELETE /api/v1/products/123          # Delete product 123
GET    /api/v1/products?category=electronics&page=0&size=20
GET    /api/v1/products/search?q=laptop
POST   /api/v1/orders                 # Create order
GET    /api/v1/orders/456             # Get order 456
GET    /api/v1/users/123/orders       # Get user's orders

❌ Bad REST API Design:

POST   /api/getProducts              # Verb in URL
GET    /api/v1/getProduct/123        # Verb in URL
POST   /api/v1/createProduct         # Verb in URL
GET    /api/v1/deleteProduct/123     # GET for deletion
/api/v1/product                      # Singular
/api/v1/products/list                # Redundant
```

---

## 10. Validation: Protecting Your Application from Invalid Data

### Why Validation is Critical

One of the easiest ways to create software bugs is by trusting user input. Users may accidentally—or intentionally—submit invalid information. Without validation, invalid data reaches your database, causing corruption and security vulnerabilities.

### Bean Validation in Spring Boot

Spring Boot integrates with **Jakarta Bean Validation** (formerly Java Bean Validation) to provide declarative validation.

### Built-in Validation Annotations

```java
public record UserRegistrationRequest(
    // Not null/empty
    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50, message = "First name must be 2-50 characters")
    String firstName,
    
    @NotBlank(message = "Last name is required")
    @Size(min = 2, max = 50)
    String lastName,
    
    // Email validation
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    @Size(max = 100)
    String email,
    
    // Password with multiple constraints
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$",
             message = "Password must contain uppercase, lowercase, digit, and special character")
    String password,
    
    // Age validation
    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Must be at least 18 years old")
    @Max(value = 120, message = "Age cannot exceed 120")
    Integer age,
    
    // Phone number
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", 
             message = "Invalid phone number format")
    String phoneNumber,
    
    // URL validation
    @Pattern(regexp = "^(https?://)?([\\da-z.-]+)\\.([a-z.]{2,6})([/\\w .-]*)*/?$",
             message = "Invalid website URL")
    String website,
    
    // Date validation
    @Past(message = "Birth date must be in the past")
    @NotNull(message = "Birth date is required")
    LocalDate birthDate,
    
    // Collection validation
    @NotEmpty(message = "At least one role must be selected")
    List<@NotBlank String> roles,
    
    // Nested object validation
    @Valid
    AddressRequest address
) {}

public record AddressRequest(
    @NotBlank String street,
    @NotBlank String city,
    @NotBlank String state,
    @NotBlank @Pattern(regexp = "^\\d{5}(-\\d{4})?$") String zipCode,
    @NotBlank String country
) {}
```

### Applying Validation

#### Method 1: @Valid on @RequestBody

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @PostMapping("/register")
    public ResponseEntity<UserResponse> register(
            // @Valid triggers validation
            @Valid @RequestBody UserRegistrationRequest request) {
        
        // If validation fails, MethodArgumentNotValidException is thrown
        // and handled by GlobalExceptionHandler
        
        UserResponse user = authService.register(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

#### Method 2: @Validated on Class Level

```java
@RestController
@RequestMapping("/api/users")
@Validated // Enable validation for all methods
public class UserController {
    
    @GetMapping("/{id}")
    public UserResponse getUser(
            @PathVariable @Positive(message = "ID must be positive") Long id) {
        return userService.findById(id);
    }
    
    @GetMapping("/search")
    public List<UserResponse> search(
            @RequestParam @NotBlank @Size(min = 2, max = 50) String name,
            @RequestParam @Min(18) @Max(120) Integer minAge) {
        return userService.search(name, minAge);
    }
}
```

### Custom Validation Annotations

Create your own validation rules:

```java
// Step 1: Define annotation
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordStrengthValidator.class)
public @interface StrongPassword {
    String message() default "Password does not meet strength requirements";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Step 2: Implement validator
public class PasswordStrengthValidator implements ConstraintValidator<StrongPassword, String> {
    
    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (password == null || password.isEmpty()) {
            return true; // @NotBlank handles empty check
        }
        
        // At least 8 characters
        if (password.length() < 8) return false;
        
        // At least one uppercase
        if (!password.matches(".*[A-Z].*")) return false;
        
        // At least one lowercase
        if (!password.matches(".*[a-z].*")) return false;
        
        // At least one digit
        if (!password.matches(".*[0-9].*")) return false;
        
        // At least one special character
        if (!password.matches(".*[@#$%^&+=!].*")) return false;
        
        return true;
    }
}

// Step 3: Use it
public record RegisterRequest(
    @NotBlank String username,
    @StrongPassword String password
) {}
```

### Validation Groups

Different validation rules for different use cases:

```java
// Define validation groups
public interface OnCreate {}
public interface OnUpdate {}

// Apply groups to constraints
public record UserRequest(
    @NotBlank(groups = OnCreate.class)
    @Size(min = 3, max = 50, groups = {OnCreate.class, OnUpdate.class})
    String username,
    
    @NotBlank(groups = OnCreate.class)
    @Email(groups = {OnCreate.class, OnUpdate.class})
    String email,
    
    @NotNull(groups = OnCreate.class)
    @Min(18, groups = OnCreate.class)
    Integer age
) {}

// Use in controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<UserResponse> create(
            // Validate with OnCreate group
            @Validated(OnCreate.class) @RequestBody UserRequest request) {
        return ResponseEntity.ok(userService.create(request));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> update(
            @PathVariable Long id,
            // Validate with OnUpdate group
            @Validated(OnUpdate.class) @RequestBody UserRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }
}
```

### Cross-Field Validation

Validate relationships between fields:

```java
public record DateRangeRequest(
    @NotNull @PastOrPresent LocalDateTime startDate,
    @NotNull @Future LocalDateTime endDate,
    
    @AssertTrue(message = "End date must be after start date")
    boolean isEndDateAfterStartDate() {
        return endDate != null && startDate != null && endDate.isAfter(startDate);
    }
) {}

public record PasswordChangeRequest(
    @NotBlank String currentPassword,
    @NotBlank @StrongPassword String newPassword,
    
    @AssertTrue(message = "New password must be different from current password")
    boolean isNewPasswordDifferent() {
        return !currentPassword.equals(newPassword);
    }
) {}
```

### Programmatic Validation

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request,
            Errors errors) {
        
        // Check for validation errors
        if (errors.hasErrors()) {
            List<String> errorMessages = errors.getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .toList();
            
            throw new ValidationException("Validation failed", errorMessages);
        }
        
        // Custom business validation
        if (!productService.areProductsAvailable(request.items())) {
            throw new ValidationException("Some products are out of stock");
        }
        
        OrderResponse order = orderService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}
```

### Validation Error Response

```java
@RestControllerAdvice
public class ValidationExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidationException(
            MethodArgumentNotValidException ex) {
        
        List<FieldError> fieldErrors = ex.getBindingResult().getFieldErrors();
        
        Map<String, String> errors = fieldErrors.stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                fieldError -> fieldError.getDefaultMessage() != null 
                    ? fieldError.getDefaultMessage() 
                    : "Invalid value"
            ));
        
        ValidationErrorResponse response = new ValidationErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            errors
        );
        
        return ResponseEntity.badRequest().body(response);
    }
}

public record ValidationErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    Map<String, String> fieldErrors
) {}
```

### Validation Best Practices

✅ **DO:**
- Validate all incoming data
- Provide clear, specific error messages
- Use constraint annotations on DTOs
- Validate at multiple layers (controller, service)
- Use validation groups for different scenarios
- Create custom validators for complex rules

❌ **DON'T:**
- Trust client-side validation alone
- Use generic error messages like "Invalid input"
- Skip validation for "internal" endpoints
- Expose internal implementation details in error messages
- Validate entities directly (use DTOs)

### Real-World Example: Complete Validation

```java
// Request DTO
public record CreateOrderRequest(
    @NotNull(message = "Customer ID is required")
    @Positive(message = "Customer ID must be positive")
    Long customerId,
    
    @NotEmpty(message = "Order must contain at least one item")
    @Size(min = 1, max = 50, message = "Order cannot have more than 50 items")
    List<@Valid OrderItemRequest> items,
    
    @Size(max = 1000, message = "Notes cannot exceed 1000 characters")
    String notes,
    
    @NotNull(message = "Delivery date is required")
    @Future(message = "Delivery date must be in the future")
    LocalDateTime deliveryDate,
    
    @AssertTrue(message = "Must agree to terms and conditions")
    boolean agreedToTerms
) {}

public record OrderItemRequest(
    @NotNull(message = "Product ID is required")
    @Positive(message = "Product ID must be positive")
    Long productId,
    
    @NotNull(message = "Quantity is required")
    @Min(value = 1, message = "Quantity must be at least 1")
    @Max(value = 100, message = "Quantity cannot exceed 100")
    Integer quantity,
    
    @DecimalMin(value = "0.0", message = "Price cannot be negative")
    BigDecimal unitPrice
) {}

// Controller
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        
        OrderResponse order = orderService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}

// Error Response
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 400,
  "error": "Validation Failed",
  "fieldErrors": {
    "customerId": "Customer ID is required",
    "items[0].quantity": "Quantity must be at least 1",
    "deliveryDate": "Delivery date must be in the future",
    "agreedToTerms": "Must agree to terms and conditions"
  }
}
```

---

## 11. Global Exception Handling: Managing Errors Gracefully

### The Problem with Scattered Exception Handling

Without centralized exception handling, every controller would need repetitive try-catch blocks. Applications quickly become cluttered, and error responses become inconsistent.

### Global Exception Handling with @RestControllerAdvice

Spring Boot provides `@RestControllerAdvice` (or `@ControllerAdvice` for MVC) to handle exceptions globally:

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    // Handle specific exception
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex,
            WebRequest request) {
        
        log.error("Resource not found: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.NOT_FOUND.value(),
            "Resource Not Found",
            ex.getMessage(),
            request.getDescription(false)
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // Handle validation exceptions
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidationExceptions(
            MethodArgumentNotValidException ex) {
        
        log.error("Validation failed: {}", ex.getMessage());
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> {
            String fieldName = error.getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        ValidationErrorResponse response = new ValidationErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            errors
        );
        
        return ResponseEntity.badRequest().body(response);
    }
    
    // Handle method argument type mismatch
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(
            MethodArgumentTypeMismatchException ex) {
        
        log.error("Type mismatch: {}", ex.getMessage());
        
        String message = String.format("Invalid value '%s' for parameter '%s'",
            ex.getValue(), ex.getName());
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Invalid Parameter",
            message,
            null
        );
        
        return ResponseEntity.badRequest().body(error);
    }
    
    // Handle access denied
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        log.error("Access denied: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.FORBIDDEN.value(),
            "Access Denied",
            "You don't have permission to access this resource",
            null
        );
        
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
    
    // Handle authentication exceptions
    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthentication(AuthenticationException ex) {
        log.error("Authentication failed: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.UNAUTHORIZED.value(),
            "Authentication Failed",
            "Invalid or missing authentication credentials",
            null
        );
        
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);
    }
    
    // Handle all other exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex,
            WebRequest request) {
        
        log.error("Unexpected error: {}", ex.getMessage(), ex);
        
        // Don't expose internal details in production
        String message = "An unexpected error occurred";
        
        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal Server Error",
            message,
            request.getDescription(false)
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### Custom Exception Classes

Create meaningful exception hierarchies:

```java
// Base exception class
public abstract class BaseException extends RuntimeException {
    private final HttpStatus status;
    private final String errorCode;
    
    protected BaseException(HttpStatus status, String errorCode, String message) {
        super(message);
        this.status = status;
        this.errorCode = errorCode;
    }
    
    public HttpStatus getStatus() { return status; }
    public String getErrorCode() { return errorCode; }
}

// Specific exceptions
public class ResourceNotFoundException extends BaseException {
    public ResourceNotFoundException(String resourceName, Long id) {
        super(HttpStatus.NOT_FOUND, "RESOURCE_NOT_FOUND",
            String.format("%s not found with id: %d", resourceName, id));
    }
    
    public ResourceNotFoundException(String message) {
        super(HttpStatus.NOT_FOUND, "RESOURCE_NOT_FOUND", message);
    }
}

public class ValidationException extends BaseException {
    public ValidationException(String message) {
        super(HttpStatus.BAD_REQUEST, "VALIDATION_ERROR", message);
    }
    
    public ValidationException(String message, List<String> details) {
        super(HttpStatus.BAD_REQUEST, "VALIDATION_ERROR", message);
        this.details = details;
    }
    
    private List<String> details;
    
    public List<String> getDetails() { return details; }
}

public class UnauthorizedException extends BaseException {
    public UnauthorizedException(String message) {
        super(HttpStatus.UNAUTHORIZED, "UNAUTHORIZED", message);
    }
}

public class ForbiddenException extends BaseException {
    public ForbiddenException(String message) {
        super(HttpStatus.FORBIDDEN, "FORBIDDEN", message);
    }
}

public class BusinessException extends BaseException {
    public BusinessException(String message) {
        super(HttpStatus.UNPROCESSABLE_ENTITY, "BUSINESS_RULE_VIOLATION", message);
    }
}
```

### Error Response DTOs

```java
// Standard error response
public record ErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    String message,
    String path
) {
    public static ErrorResponse of(HttpStatus status, String message, String path) {
        return new ErrorResponse(
            LocalDateTime.now(),
            status.value(),
            status.getReasonPhrase(),
            message,
            path
        );
    }
}

// Validation error response
public record ValidationErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    Map<String, String> fieldErrors
) {
    public static ValidationErrorResponse of(Map<String, String> fieldErrors) {
        return new ValidationErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            fieldErrors
        );
    }
}

// Detailed error response for debugging
public record DetailedErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    String message,
    String path,
    String errorCode,
    Map<String, Object> details
) {}
```

### Exception Handling Flow

```mermaid
graph TD
    A[Exception Thrown] --> B{Exception Type?}
    
    B -->|ResourceNotFoundException| C[404 Not Found]
    B -->|ValidationException| D[400 Bad Request]
    B -->|UnauthorizedException| E[401 Unauthorized]
    B -->|AccessDeniedException| F[403 Forbidden]
    B -->|BusinessException| G[422 Unprocessable Entity]
    B -->|MethodArgumentNotValidException| D
    B -->|Other Exceptions| H[500 Internal Server Error]
    
    C --> I[Log Error]
    D --> I
    E --> I
    F --> I
    G --> I
    H --> I
    
    I --> J[Create ErrorResponse]
    J --> K[Return JSON Response]
    
    style A fill:#ff6b6b
    style K fill:#4ecdc4
    style H fill:#ff0000
```

**Figure 8: Global Exception Handling Flow**

### Practical Example: Complete Error Handling

```java
// Service layer throwing exceptions
@Service
public class OrderService {
    
    public OrderResponse createOrder(CreateOrderRequest request) {
        // Validate business rules
        Customer customer = customerRepository.findById(request.customerId())
            .orElseThrow(() -> new ResourceNotFoundException("Customer", request.customerId()));
        
        if (!customer.isActive()) {
            throw new BusinessException("Cannot create order for inactive customer");
        }
        
        // Check stock
        for (OrderItemRequest item : request.items()) {
            Product product = productRepository.findById(item.productId())
                .orElseThrow(() -> new ResourceNotFoundException("Product", item.productId()));
            
            if (!product.isInStock()) {
                throw new BusinessException(
                    String.format("Product %s is out of stock", product.getName())
                );
            }
        }
        
        // Create order
        Order order = new Order(request);
        orderRepository.save(order);
        
        return OrderResponse.fromEntity(order);
    }
}

// Controller using service
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        
        // No try-catch needed! Global handler takes care of it
        OrderResponse order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}

// Global exception handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex,
            WebRequest request) {
        
        log.error("Resource not found: {}", ex.getMessage());
        
        ErrorResponse error = ErrorResponse.of(
            ex.getStatus(),
            ex.getMessage(),
            request.getDescription(false)
        );
        
        return new ResponseEntity<>(error, ex.getStatus());
    }
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(
            BusinessException ex,
            WebRequest request) {
        
        log.error("Business rule violation: {}", ex.getMessage());
        
        ErrorResponse error = ErrorResponse.of(
            ex.getStatus(),
            ex.getMessage(),
            request.getDescription(false)
        );
        
        return new ResponseEntity<>(error, ex.getStatus());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        
        log.error("Validation failed");
        
        Map<String, String> errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                error -> error.getDefaultMessage() != null ? error.getDefaultMessage() : "Invalid value"
            ));
        
        return ResponseEntity.badRequest()
            .body(ValidationErrorResponse.of(errors));
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(
            Exception ex,
            WebRequest request) {
        
        log.error("Unexpected error: {}", ex.getMessage(), ex);
        
        // In production, don't expose internal details
        String message = "An unexpected error occurred. Please try again later.";
        
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.INTERNAL_SERVER_ERROR,
            message,
            request.getDescription(false)
        );
        
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### Error Response Examples

**Validation Error:**
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 400,
  "error": "Validation Failed",
  "fieldErrors": {
    "email": "Invalid email format",
    "password": "Password must contain uppercase, lowercase, digit, and special character",
    "age": "Must be at least 18 years old"
  }
}
```

**Not Found Error:**
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "User not found with id: 123",
  "path": "/api/users/123"
}
```

**Business Rule Violation:**
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 422,
  "error": "Unprocessable Entity",
  "message": "Product Laptop is out of stock",
  "path": "/api/orders"
}
```

### Best Practices

✅ **DO:**
- Create custom exception classes for different error types
- Return appropriate HTTP status codes
- Provide clear, actionable error messages
- Log exceptions with context
- Don't expose sensitive information in error messages
- Use consistent error response format
- Include error codes for client-side handling

❌ **DON'T:**
- Return 200 OK for errors
- Expose stack traces to clients
- Use generic error messages
- Catch exceptions and swallow them
- Log sensitive data (passwords, tokens)
- Return different error formats for different errors

---

## 12. Spring Data JPA: Making Database Access Easier

### The Problem with Manual Database Access

Before frameworks like Spring Data JPA, developers wrote SQL queries manually for almost every database operation. Although SQL remains extremely important, repetitive CRUD operations became tedious and error-prone.

### What is Spring Data JPA?

Spring Data JPA dramatically simplifies database access by automatically generating repository implementations. Instead of writing boilerplate code for common operations, developers create repository interfaces and Spring generates the implementation behind the scenes.

### Creating a Repository

```java
// Step 1: Define entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String name;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    // Getters, setters, constructors
}

// Step 2: Create repository interface
public interface UserRepository extends JpaRepository<User, Long> {
    // That's it! Spring Data JPA provides:
    // - save(user) - Insert or update
    // - findById(id) - Find by primary key
    // - findAll() - Get all users
    // - deleteById(id) - Delete by ID
    // - count() - Count users
    // - existsById(id) - Check if exists
    // And many more...
}

// Step 3: Use in service
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User createUser(User user) {
        return userRepository.save(user); // INSERT query generated
    }
    
    public Optional<User> getUser(Long id) {
        return userRepository.findById(id); // SELECT query generated
    }
    
    public List<User> getAllUsers() {
        return userRepository.findAll(); // SELECT * query generated
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id); // DELETE query generated
    }
}
```

### Derived Query Methods

Spring Data JPA can derive queries from method names:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Find by single field
    Optional<User> findByEmail(String email);
    
    // Find by multiple fields (AND)
    Optional<User> findByEmailAndName(String email, String name);
    
    // Find by multiple fields (OR)
    List<User> findByEmailOrName(String email, String name);
    
    // Comparison operators
    List<User> findByIdGreaterThan(Long id);
    List<User> findByIdLessThan(Long id);
    List<User> findByIdGreaterThanEqual(Long id);
    List<User> findByIdBetween(Long start, Long end);
    
    // LIKE queries
    List<User> findByNameContaining(String keyword);
    List<User> findByNameStartingWith(String prefix);
    List<User> findByNameEndingWith(String suffix);
    List<User> findByEmailLike(String pattern);
    
    // Ignore case
    List<User> findByNameIgnoreCase(String name);
    List<User> findByNameContainingIgnoreCase(String keyword);
    
    // Ordering
    List<User> findByNameOrderByCreatedAtDesc(String name);
    List<User> findByNameOrderByCreatedAtDescNameAsc(String name);
    
    // Pagination and sorting
    Page<User> findByNameContaining(String name, Pageable pageable);
    List<User> findByNameContaining(String name, Sort sort);
    
    // Exists
    boolean existsByEmail(String email);
    boolean existsByEmailAndActive(String email, boolean active);
    
    // Count
    long countByActive(boolean active);
    long countByCreatedAtAfter(LocalDateTime date);
    
    // Delete
    void deleteByEmail(String email);
    long deleteByActive(boolean active);
}
```

### Custom Queries with @Query

For complex queries, use the `@Query` annotation:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // JPQL query (uses entity names, not table names)
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
    
    // Multiple parameters
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.active = :active")
    Optional<User> findByEmailAndStatus(
            @Param("email") String email,
            @Param("active") boolean active);
    
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
    Optional<User> findByEmailNative(String email);
    
    // Projection - select specific columns
    @Query("SELECT new com.example.dto.UserSummary(u.id, u.name, u.email) " +
           "FROM User u WHERE u.active = true")
    List<UserSummary> findActiveUserSummaries();
    
    // Update query
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateInactiveUsers(@Param("date") LocalDateTime date);
    
    // Delete query
    @Modifying
    @Query("DELETE FROM User u WHERE u.active = false AND u.createdAt < :date")
    int deleteInactiveUsers(@Param("date") LocalDateTime date);
}
```

### Pagination and Sorting

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public Page<User> getUsers(int page, int size, String sortBy, String direction) {
        Pageable pageable = PageRequest.of(
            page,
            size,
            Sort.by(Sort.Direction.fromString(direction), sortBy)
        );
        
        return userRepository.findAll(pageable);
    }
    
    public Page<User> searchUsers(String keyword, int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findByNameContainingIgnoreCase(keyword, pageable);
    }
}

// Controller
@GetMapping("/users")
public ResponseEntity<Page<UserResponse>> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "asc") String sortDir) {
    
    Page<User> users = userService.getUsers(page, size, sortBy, sortDir);
    return ResponseEntity.ok(users.map(UserResponse::fromEntity));
}
```

### Specifications (Dynamic Queries)

For dynamic queries based on optional parameters:

```java
public interface UserRepository extends JpaRepository<User, Long>, 
                                        JpaSpecificationExecutor<User> {
}

@Service
public class UserService {
    private final UserRepository userRepository;
    
    public Page<User> searchUsers(String name, String email, Boolean active, 
                                   LocalDateTime createdAfter, Pageable pageable) {
        
        Specification<User> spec = Specification.where(null);
        
        if (name != null && !name.isEmpty()) {
            spec = spec.and((root, query, cb) -> 
                cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%")
            );
        }
        
        if (email != null && !email.isEmpty()) {
            spec = spec.and((root, query, cb) -> 
                cb.equal(root.get("email"), email)
            );
        }
        
        if (active != null) {
            spec = spec.and((root, query, cb) -> 
                cb.equal(root.get("active"), active)
            );
        }
        
        if (createdAfter != null) {
            spec = spec.and((root, query, cb) -> 
                cb.greaterThan(root.get("createdAt"), createdAfter)
            );
        }
        
        return userRepository.findAll(spec, pageable);
    }
}
```

### Projections

Select specific fields for better performance:

```java
// Interface-based projection
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

// Use in repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findBy(); // Returns projections
    List<UserSummary> findByActiveTrue();
}

// DTO projection
public record UserSummaryDTO(Long id, String name, String email) {}

public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT new com.example.dto.UserSummaryDTO(u.id, u.name, u.email) " +
           "FROM User u WHERE u.active = true")
    List<UserSummaryDTO> findActiveUserSummaries();
}
```

### Auditing

Automatically track creation and modification timestamps:

```java
// Step 1: Enable auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}

// Step 2: Add auditing annotations to entity
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
}

// Step 3: Provide auditor information
@Component
public class AuditorAwareImpl implements AuditorAware<String> {
    
    @Override
    public Optional<String> getCurrentAuditor() {
        // Get current user from security context
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        
        if (authentication == null || !authentication.isAuthenticated()) {
            return Optional.of("system");
        }
        
        return Optional.of(authentication.getName());
    }
}
```

### Transactions

```java
@Service
@Transactional
public class UserService {
    
    @Transactional(readOnly = true)
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }
    
    @Transactional
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    @Transactional
    public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
        User fromUser = userRepository.findById(fromId)
            .orElseThrow(() -> new ResourceNotFoundException("User", fromId));
        User toUser = userRepository.findById(toId)
            .orElseThrow(() -> new ResourceNotFoundException("User", toId));
        
        fromUser.setBalance(fromUser.getBalance().subtract(amount));
        toUser.setBalance(toUser.getBalance().add(amount));
        
        userRepository.save(fromUser);
        userRepository.save(toUser);
        // Both saves succeed or both fail (transaction rollback)
    }
}
```

### Best Practices

✅ **DO:**
- Use repository interfaces for data access
- Leverage derived query methods when possible
- Use `@Query` for complex queries
- Apply pagination for large datasets
- Use projections to fetch only needed columns
- Enable auditing for created/updated tracking
- Use transactions appropriately

❌ **DON'T:**
- Put business logic in repositories
- Return entities directly from repositories to controllers
- Use native queries unless necessary
- Fetch entire tables when you need only a few columns
- Ignore N+1 query problems
- Forget to add indexes for frequently queried columns

---

## 13. Entity Relationships: Modeling Real-World Data

### Understanding Entity Relationships

Real-world information is interconnected. A customer places multiple orders, each order contains several products, and each product belongs to a category. Representing these relationships correctly is crucial for database design.

### Relationship Types

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "ordered in"
    PRODUCT }|--|| CATEGORY : belongs_to
    CUSTOMER ||--|| ADDRESS : has
    
    CUSTOMER {
        bigint id PK
        varchar email UK
        varchar name
        timestamp created_at
    }
    
    ORDER {
        bigint id PK
        bigint customer_id FK
        decimal total
        varchar status
        timestamp created_at
    }
    
    ORDER_ITEM {
        bigint id PK
        bigint order_id FK
        bigint product_id FK
        int quantity
        decimal price
    }
    
    PRODUCT {
        bigint id PK
        varchar name
        text description
        decimal price
        bigint category_id FK
    }
    
    CATEGORY {
        bigint id PK
        varchar name
        varchar description
    }
    
    ADDRESS {
        bigint id PK
        bigint customer_id FK
        varchar street
        varchar city
        varchar country
    }
```

**Figure 9: E-Commerce Database Schema**

### One-to-One Relationship

```java
// One customer has one profile
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "customer", cascade = CascadeType.ALL)
    private CustomerProfile profile;
}

@Entity
public class CustomerProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String bio;
    private String avatarUrl;
    
    @OneToOne
    @JoinColumn(name = "customer_id", referencedColumnName = "id")
    private Customer customer;
}

// Usage
Customer customer = new Customer();
customer.setName("John Doe");

CustomerProfile profile = new CustomerProfile();
profile.setBio("Software Developer");
profile.setCustomer(customer);

customer.setProfile(profile);

customerRepository.save(customer); // Saves both customer and profile
```

### One-to-Many Relationship

```java
// One customer has many orders
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    // Helper method to maintain bidirectional relationship
    public void addOrder(Order order) {
        orders.add(order);
        order.setCustomer(this);
    }
    
    public void removeOrder(Order order) {
        orders.remove(order);
        order.setCustomer(null);
    }
}

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private BigDecimal total;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    // Getters and setters
    public void setCustomer(Customer customer) {
        this.customer = customer;
    }
}

// Usage
Customer customer = new Customer();
customer.setName("John Doe");

Order order1 = new Order();
order1.setTotal(new BigDecimal("100.00"));

Order order2 = new Order();
order2.setTotal(new BigDecimal("200.00"));

customer.addOrder(order1);
customer.addOrder(order2);

customerRepository.save(customer); // Saves customer and both orders
```

### Many-to-One Relationship

```java
// Many orders belong to one customer
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private BigDecimal total;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "status_id", nullable = false)
    private OrderStatus status;
}

@Entity
public class OrderStatus {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name; // PENDING, PROCESSING, SHIPPED, DELIVERED
}
```

### Many-to-Many Relationship

```java
// Many students enroll in many courses
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
    
    public void enroll(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}

// Usage
Student student = new Student();
student.setName("Alice");

Course course1 = new Course();
course1.setTitle("Mathematics");

Course course2 = new Course();
course2.setTitle("Physics");

student.enroll(course1);
student.enroll(course2);

studentRepository.save(student); // Saves student, courses, and join table entries
```

### Fetch Strategies

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // EAGER: Load immediately with parent
    // Use for required relationships
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    // LAZY: Load only when accessed
    // Use for optional or large collections
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
}

// FetchType comparison
/*
EAGER:
✅ Data always available
❌ Always loaded (even if not needed)
❌ Can cause performance issues
❌ N+1 query problem

LAZY:
✅ Loaded only when needed
✅ Better performance
❌ Requires active session (or initialization)
❌ Can cause LazyInitializationException
*/
```

### Cascade Operations

```java
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany(
        mappedBy = "customer",
        cascade = CascadeType.ALL, // Cascade all operations
        orphanRemoval = true // Delete orphans
    )
    private List<Order> orders = new ArrayList<>();
}

// Cascade types:
// PERSIST: Save child when parent is saved
// MERGE: Update child when parent is updated
// REMOVE: Delete child when parent is deleted
// REFRESH: Refresh child when parent is refreshed
// DETACH: Detach child when parent is detached
// ALL: All of the above

// Usage
Customer customer = new Customer();
customer.setName("John");

Order order = new Order();
order.setTotal(new BigDecimal("100.00"));

customer.getOrders().add(order);

customerRepository.save(customer);
// Saves customer AND order (cascade)
```

### Orphan Removal

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany(
        mappedBy = "order",
        cascade = CascadeType.ALL,
        orphanRemoval = true // Delete items removed from collection
    )
    private List<OrderItem> items = new ArrayList<>();
    
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
    
    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
        // With orphanRemoval=true, this item will be deleted from database
    }
}

// Usage
Order order = orderRepository.findById(1L).orElseThrow();
OrderItem itemToRemove = order.getItems().get(0);

order.removeItem(itemToRemove);
orderRepository.save(order);
// Item is deleted from database (orphanRemoval)
```

### Avoiding N+1 Query Problem

The N+1 problem occurs when you fetch a list of entities and then access a lazy-loaded relationship for each entity, causing N+1 queries.

```java
// ❌ BAD: N+1 problem
@Service
public class BadOrderService {
    public List<OrderDTO> getOrders() {
        List<Order> orders = orderRepository.findAll(); // 1 query
        
        return orders.stream()
            .map(order -> {
                // N queries (one for each order)
                Customer customer = order.getCustomer().getName();
                return new OrderDTO(order, customer);
            })
            .toList();
    }
}

// ✅ GOOD: Use JOIN FETCH
public interface OrderRepository extends JpaRepository<Order, Long> {
    @Query("SELECT o FROM Order o JOIN FETCH o.customer")
    List<Order> findAllWithCustomer();
}

@Service
public class GoodOrderService {
    public List<OrderDTO> getOrders() {
        List<Order> orders = orderRepository.findAllWithCustomer(); // 1 query with JOIN
        
        return orders.stream()
            .map(order -> {
                // No additional queries - customer already loaded
                Customer customer = order.getCustomer().getName();
                return new OrderDTO(order, customer);
            })
            .toList();
    }
}

// ✅ BETTER: Use EntityGraph
public interface OrderRepository extends JpaRepository<Order, Long> {
    @EntityGraph(attributePaths = {"customer", "items", "items.product"})
    List<Order> findAll();
}
```

### Best Practices

✅ **DO:**
- Use `LAZY` fetch for collections and optional relationships
- Use `EAGER` fetch only for required relationships
- Use `JOIN FETCH` to avoid N+1 problems
- Use `orphanRemoval = true` for parent-child relationships
- Maintain bidirectional relationships properly
- Use `@Transactional` when loading lazy collections

❌ **DON'T:**
- Use `EAGER` for collections (causes performance issues)
- Access lazy collections outside transaction scope
- Forget to set both sides of bidirectional relationships
- Create circular dependencies
- Use many-to-many with extra columns (use intermediate entity instead)

### Real-World Example: Complete Entity Model

```java
@Entity
@Table(name = "orders")
@EntityListeners(AuditingEntityListener.class)
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String orderNumber;
    
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "status_id")
    private OrderStatus status;
    
    @OneToOne(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private Payment payment;
    
    @Column(nullable = false)
    private BigDecimal total;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderType type;
    
    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Helper methods
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
        recalculateTotal();
    }
    
    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
        recalculateTotal();
    }
    
    private void recalculateTotal() {
        this.total = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id", nullable = false)
    private Order order;
    
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @Column(nullable = false)
    private BigDecimal unitPrice;
    
    @Column(nullable = false)
    private BigDecimal subtotal;
    
    @PrePersist
    @PreUpdate
    private void calculateSubtotal() {
        if (unitPrice != null && quantity != null) {
            this.subtotal = unitPrice.multiply(BigDecimal.valueOf(quantity));
        }
    }
}

@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String name;
    
    @Column(name = "phone_number")
    private String phoneNumber;
    
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @OneToOne(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private CustomerProfile profile;
    
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Address> addresses = new ArrayList<>();
    
    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

---

## 14. Transactions: Keeping Data Consistent

### What are Transactions?

Imagine transferring money between two bank accounts. The application performs two operations:
1. Withdraw money from Account A
2. Deposit money into Account B

What happens if the first operation succeeds but the second fails? Money disappears. This is unacceptable.

**Transactions solve this problem** by grouping multiple database operations into a single logical unit. Either everything succeeds, or everything fails. There is no partial completion.

### ACID Properties

```mermaid
graph LR
    A[ACID Properties] --> B[Atomicity]
    A --> C[Consistency]
    A --> D[Isolation]
    A --> E[Durability]
    
    B --> B1[All or Nothing]
    C --> C1[Data Integrity]
    D --> D1[Concurrent Safety]
    E --> E1[Permanent Changes]
    
    style A fill:#4ecdc4
    style B fill:#ffe66d
    style C fill:#ffe66d
    style D fill:#ffe66d
    style E fill:#ffe66d
```

**Figure 10: ACID Properties of Transactions**

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All operations succeed or all fail | Money transfer: both debit and credit must succeed |
| **Consistency** | Data remains valid after transaction | Account balance can't go negative |
| **Isolation** | Concurrent transactions don't interfere | Two transfers don't corrupt data |
| **Durability** | Committed changes are permanent | After commit, data survives system failure |

### Transaction Management in Spring Boot

Spring Boot simplifies transaction management using the `@Transactional` annotation:

```java
@Service
public class BankService {
    
    @Transactional
    public void transferMoney(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        // Step 1: Withdraw from source account
        Account fromAccount = accountRepository.findById(fromAccountId)
            .orElseThrow(() -> new ResourceNotFoundException("Account not found"));
        
        if (fromAccount.getBalance().compareTo(amount) < 0) {
            throw new BusinessException("Insufficient funds");
        }
        
        fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
        accountRepository.save(fromAccount);
        
        // Step 2: Deposit to destination account
        Account toAccount = accountRepository.findById(toAccountId)
            .orElseThrow(() -> new ResourceNotFoundException("Account not found"));
        
        toAccount.setBalance(toAccount.getBalance().add(amount));
        accountRepository.save(toAccount);
        
        // If any exception occurs, both operations are rolled back
        // If both succeed, transaction commits
    }
}
```

### How @Transactional Works

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Service
    participant TM as Transaction Manager
    participant DB as Database
    
    C->>S: transferMoney(from, to, amount)
    S->>TM: Begin Transaction
    TM->>DB: START TRANSACTION
    
    Note over S,DB: Transaction Active
    
    S->>DB: UPDATE accounts SET balance = balance - 100 WHERE id = 1
    DB-->>S: Success
    
    S->>DB: UPDATE accounts SET balance = balance + 100 WHERE id = 2
    DB-->>S: Success
    
    S->>TM: Commit Transaction
    TM->>DB: COMMIT
    DB-->>TM: Committed
    TM-->>S: Success
    S-->>C: Transfer Complete
    
    Note over S,DB: If exception occurs:
    Note over S,DB: ROLLBACK instead of COMMIT
```

**Figure 11: Transaction Flow**

### Transaction Propagation

Define how transactions relate to each other:

```java
@Service
public class OrderService {
    
    // REQUIRED (default): Join existing transaction or create new one
    @Transactional(propagation = Propagation.REQUIRED)
    public void createOrder(Order order) {
        orderRepository.save(order);
        paymentService.processPayment(order); // Joins same transaction
    }
    
    // REQUIRES_NEW: Always create new transaction (suspends current)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderCreation(Order order) {
        auditLogRepository.save(new AuditLog("Order created", order));
        // This runs in separate transaction
    }
    
    // SUPPORTS: Join transaction if exists, otherwise non-transactional
    @Transactional(propagation = Propagation.SUPPORTS)
    public List<Order> getRecentOrders() {
        return orderRepository.findTop10ByOrderByCreatedAtDesc();
    }
    
    // NOT_SUPPORTED: Always run non-transactionally
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void exportOrders() {
        // Runs without transaction
        List<Order> orders = orderRepository.findAll();
        // Export logic
    }
    
    // MANDATORY: Must run within existing transaction
    @Transactional(propagation = Propagation.MANDATORY)
    public void updateInventory(Order order) {
        // Throws exception if no transaction exists
        inventoryService.updateStock(order);
    }
    
    // NEVER: Must NOT run within transaction
    @Transactional(propagation = Propagation.NEVER)
    public void sendNotification(Order order) {
        // Throws exception if transaction exists
        notificationService.send(order);
    }
}
```

### Transaction Isolation Levels

Control how transactions interact with each other:

```java
@Service
public class AccountService {
    
    // READ_UNCOMMITTED: Can read uncommitted changes (dirty reads)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public BigDecimal getBalance(Long accountId) {
        return accountRepository.findById(accountId)
            .get()
            .getBalance();
    }
    
    // READ_COMMITTED: Can only read committed changes (default for most DBs)
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void updateBalance(Long accountId, BigDecimal newBalance) {
        Account account = accountRepository.findById(accountId).get();
        account.setBalance(newBalance);
        accountRepository.save(account);
    }
    
    // REPEATABLE_READ: Same query returns same result within transaction
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void transferWithChecks(Long fromId, Long toId, BigDecimal amount) {
        // Multiple reads return consistent results
        BigDecimal balance1 = getBalance(fromId);
        // ... some time passes ...
        BigDecimal balance2 = getBalance(fromId);
        // balance1 == balance2 (within same transaction)
    }
    
    // SERIALIZABLE: Complete isolation (highest level, lowest performance)
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void criticalOperation() {
        // Complete isolation from other transactions
        // Prevents all concurrency issues
        // But significantly reduces performance
    }
}
```

### Transaction Rollback

By default, transactions roll back only on unchecked (runtime) exceptions:

```java
@Service
public class OrderService {
    
    // Rollback on RuntimeException (default)
    @Transactional
    public void createOrder(Order order) {
        orderRepository.save(order);
        
        if (order.getTotal().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Order total must be positive");
            // Transaction rolls back
        }
        
        paymentService.processPayment(order);
    }
    
    // Rollback on checked exception
    @Transactional(rollbackFor = Exception.class)
    public void createOrderWithCheckedException(Order order) throws Exception {
        orderRepository.save(order);
        
        if (order.getTotal().compareTo(BigDecimal.ZERO) <= 0) {
            throw new Exception("Order total must be positive");
            // Transaction rolls back (because of rollbackFor)
        }
    }
    
    // No rollback on specific exception
    @Transactional(noRollbackFor = IllegalArgumentException.class)
    public void createOrderNoRollback(Order order) {
        orderRepository.save(order);
        
        if (order.getTotal().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Order total must be positive");
            // Transaction commits despite exception
        }
    }
}
```

### Practical Example: Complete Transaction Flow

```java
@Service
@Slf4j
public class OrderProcessingService {
    
    private final OrderRepository orderRepository;
    private final PaymentRepository paymentRepository;
    private final InventoryRepository inventoryRepository;
    private final NotificationService notificationService;
    private final AuditLogRepository auditLogRepository;
    
    public OrderProcessingService(
            OrderRepository orderRepository,
            PaymentRepository paymentRepository,
            InventoryRepository inventoryRepository,
            NotificationService notificationService,
            AuditLogRepository auditLogRepository) {
        this.orderRepository = orderRepository;
        this.paymentRepository = paymentRepository;
        this.inventoryRepository = inventoryRepository;
        this.notificationService = notificationService;
        this.auditLogRepository = auditLogRepository;
    }
    
    @Transactional
    public OrderResult processOrder(OrderRequest request) {
        log.info("Processing order for customer: {}", request.customerId());
        
        // Step 1: Validate customer
        Customer customer = customerRepository.findById(request.customerId())
            .orElseThrow(() -> new ResourceNotFoundException("Customer", request.customerId()));
        
        if (!customer.isActive()) {
            throw new BusinessException("Customer account is not active");
        }
        
        // Step 2: Create order
        Order order = new Order();
        order.setOrderNumber(generateOrderNumber());
        order.setCustomer(customer);
        order.setStatus(OrderStatus.PENDING);
        
        // Step 3: Add items and check inventory
        for (OrderItemRequest itemRequest : request.items()) {
            Product product = productRepository.findById(itemRequest.productId())
                .orElseThrow(() -> new ResourceNotFoundException("Product", itemRequest.productId()));
            
            if (product.getStock() < itemRequest.quantity()) {
                throw new BusinessException(
                    String.format("Insufficient stock for product: %s", product.getName())
                );
            }
            
            // Reduce stock
            product.setStock(product.getStock() - itemRequest.quantity());
            productRepository.save(product);
            
            // Add item to order
            OrderItem item = new OrderItem();
            item.setProduct(product);
            item.setQuantity(itemRequest.quantity());
            item.setUnitPrice(product.getPrice());
            order.addItem(item);
        }
        
        order.calculateTotal();
        Order savedOrder = orderRepository.save(order);
        
        // Step 4: Process payment
        Payment payment = paymentService.processPayment(savedOrder);
        paymentRepository.save(payment);
        
        // Step 5: Update order status
        savedOrder.setStatus(OrderStatus.CONFIRMED);
        savedOrder.setPayment(payment);
        orderRepository.save(savedOrder);
        
        // Step 6: Send notification (in new transaction)
        notificationService.sendOrderConfirmation(savedOrder);
        
        // Step 7: Log audit (in new transaction)
        auditLogRepository.save(new AuditLog("ORDER_CREATED", savedOrder.getId()));
        
        log.info("Order processed successfully: {}", savedOrder.getOrderNumber());
        
        return OrderResult.fromEntity(savedOrder);
    }
}
```

### Transactional Event Publishing

```java
@Service
public class UserService {
    
    @Transactional
    public User createUser(User user) {
        User savedUser = userRepository.save(user);
        
        // Publish event after transaction commits
        eventPublisher.publishEvent(new UserCreatedEvent(savedUser));
        
        return savedUser;
    }
}

// Event listener
@Component
public class UserEventListener {
    
    @EventListener
    @Async // Run asynchronously after transaction commits
    public void handleUserCreated(UserCreatedEvent event) {
        // Send welcome email
        emailService.sendWelcomeEmail(event.getUser().getEmail());
        
        // Initialize user preferences
        preferenceService.createDefaults(event.getUser().getId());
    }
}
```

### Best Practices

✅ **DO:**
- Use `@Transactional` at service layer
- Keep transactions short
- Use appropriate isolation levels
- Handle exceptions properly
- Use rollback rules when needed
- Don't call @Transactional methods from same class (self-invocation)

❌ **DON'T:**
- Put @Transactional on controllers or repositories
- Run long-running operations in transactions
- Catch exceptions and swallow them
- Use transactions for read-only operations (use `@Transactional(readOnly = true)`)
- Access lazy collections outside transaction scope

### Common Pitfalls

❌ **Self-Invocation Problem:**
```java
@Service
public class BadService {
    
    @Transactional
    public void methodA() {
        // This works
        doSomething();
    }
    
    public void methodB() {
        // ❌ Transaction doesn't apply!
        // Self-invocation bypasses proxy
        methodA();
    }
}

// ✅ Solution: Use separate beans or self-inject
@Service
public class GoodService {
    private final GoodService self;
    
    public GoodService(GoodService self) {
        this.self = self;
    }
    
    @Transactional
    public void methodA() {
        doSomething();
    }
    
    public void methodB() {
        self.methodA(); // ✅ Works!
    }
}
```

---

## 15. Spring Security Basics: Protecting Your Application

### Why Security is Critical

Security is no longer optional. Every application connected to the internet becomes a potential target. Spring Security provides a comprehensive framework for securing Spring Boot applications.

### What Spring Security Provides

- **Authentication:** Verify user identity
- **Authorization:** Control access to resources
- **Password Encryption:** Secure password storage
- **Session Management:** Handle user sessions
- **Attack Protection:** CSRF, XSS, clickjacking protection
- **Request Filtering:** Intercept and filter requests
- **CORS Configuration:** Control cross-origin requests

### Basic Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**", "/auth/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/users/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/auth/login")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/auth/logout")
                .logoutSuccessUrl("/auth/login?logout")
                .permitAll()
            )
            .csrf(csrf -> csrf.disable()); // Disable for APIs (use JWT instead)
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Password Encoding

Never store passwords in plain text:

```java
@Service
public class UserService {
    private final PasswordEncoder passwordEncoder;
    
    public UserService(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }
    
    public User registerUser(RegisterRequest request) {
        User user = new User();
        user.setEmail(request.email());
        user.setName(request.name());
        
        // Encode password before saving
        String encodedPassword = passwordEncoder.encode(request.password());
        user.setPassword(encodedPassword);
        
        return userRepository.save(user);
    }
    
    public boolean validatePassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}

// Usage
PasswordEncoder encoder = new BCryptPasswordEncoder();
String rawPassword = "SecurePass123!";
String encodedPassword = encoder.encode(rawPassword);
// Store encodedPassword in database

// Later, when user logs in
boolean matches = encoder.matches("SecurePass123!", encodedPassword);
// Returns true
```

### Role-Based Access Control

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String email;
    private String password;
    private String name;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
}

@Entity
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    @Enumerated(EnumType.STRING)
    private RoleName name;
    
    public enum RoleName {
        ROLE_USER,
        ROLE_ADMIN,
        ROLE_MANAGER
    }
}

// Security configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/manager/**").hasAnyRole("ADMIN", "MANAGER")
                .requestMatchers("/users/**").hasAnyRole("USER", "ADMIN", "MANAGER")
                .anyRequest().authenticated()
            );
        
        return http.build();
    }
}

// Method-level security
@Service
@PreAuthorize("hasRole('ADMIN')")
public class AdminService {
    public void deleteUser(Long id) {
        // Only ADMIN can call this
    }
}

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    public User getUser(Long id) {
        // ADMIN can get any user
        // Regular users can only get their own data
        return userRepository.findById(id).orElseThrow();
    }
    
    @PostAuthorize("returnObject.customerId == authentication.principal.id")
    public Order getOrder(Long id) {
        // Check after method execution
        return orderRepository.findById(id).orElseThrow();
    }
}
```

### Authentication Providers

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz.anyRequest().authenticated())
            .authenticationProvider(authenticationProvider());
        
        return http.build();
    }
    
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService());
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        return username -> {
            User user = userRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
            
            return User.builder()
                .username(user.getEmail())
                .password(user.getPassword())
                .authorities(user.getRoles().stream()
                    .map(role -> new SimpleGrantedAuthority(role.getName().name()))
                    .toList())
                .build();
        };
    }
}
```

### Security Best Practices

✅ **DO:**
- Use BCrypt for password encoding
- Implement proper authentication (JWT, OAuth2)
- Use HTTPS in production
- Implement rate limiting
- Validate all inputs
- Keep dependencies updated
- Use security headers
- Implement proper session management

❌ **DON'T:**
- Store passwords in plain text
- Use weak password encoders (MD5, SHA-1)
- Expose sensitive information in error messages
- Disable CSRF protection for stateful applications
- Trust client-side validation
- Hardcode credentials

---

## 16. Building a Typical Spring Boot Request Flow

### Complete Request Lifecycle

Let's follow a typical request through a Spring Boot application:

```mermaid
sequenceDiagram
    participant C as Client
    participant F as Security Filters
    participant V as Validation
    participant Ctrl as Controller
    participant Svc as Service
    participant Tx as Transaction
    participant Repo as Repository
    participant DB as Database
    participant Exc as Exception Handler
    
    C->>F: HTTP Request with JWT
    F->>F: Authenticate JWT
    F->>F: Authorize Request
    F->>V: Forward to Controller
    
    V->>V: Validate Request Body
    V->>V: Validate Path Variables
    V->>V: Validate Query Params
    
    alt Validation Fails
        V->>Exc: Throw ValidationException
        Exc->>C: 400 Bad Request
    else Validation Passes
        V->>Ctrl: Valid Request
        Ctrl->>Svc: Call Service Method
        
        Svc->>Tx: Begin Transaction
        Tx->>Repo: Database Operations
        Repo->>DB: Execute Queries
        DB-->>Repo: Results
        Repo-->>Svc: Entities
        
        alt Business Logic Fails
            Svc->>Tx: Rollback Transaction
            Tx->>DB: ROLLBACK
            Svc->>Exc: Throw BusinessException
            Exc->>C: 422 Unprocessable Entity
        else Success
            Svc->>Tx: Commit Transaction
            Tx->>DB: COMMIT
            Svc-->>Ctrl: Return DTO
            Ctrl-->>C: 200 OK with JSON
        end
    end
```

**Figure 12: Complete Spring Boot Request Flow**

### Step-by-Step Breakdown

#### 1. Client Sends Request
```http
POST /api/orders HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "customerId": 123,
  "items": [
    {"productId": 456, "quantity": 2},
    {"productId": 789, "quantity": 1}
  ],
  "deliveryDate": "2024-01-20T10:00:00"
}
```

#### 2. Security Filters
```java
// JWT Authentication Filter
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {
        
        String token = extractToken(request);
        
        if (token != null && jwtTokenProvider.validateToken(token)) {
            String userId = jwtTokenProvider.getUserIdFromToken(token);
            
            UserDetails userDetails = userDetailsService.loadUserById(userId);
            UsernamePasswordAuthenticationToken authentication = 
                new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        
        filterChain.doFilter(request, response);
    }
}
```

#### 3. Validation
```java
@RestController
@RequestMapping("/api/orders")
@Validated
public class OrderController {
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        // Validation happens here
        // If validation fails, MethodArgumentNotValidException is thrown
        return ResponseEntity.ok(orderService.create(request));
    }
}

// Validation annotations
public record CreateOrderRequest(
    @NotNull(message = "Customer ID is required")
    @Positive(message = "Customer ID must be positive")
    Long customerId,
    
    @NotEmpty(message = "Order must contain items")
    List<@Valid OrderItemRequest> items,
    
    @Future(message = "Delivery date must be in the future")
    LocalDateTime deliveryDate
) {}
```

#### 4. Controller Processing
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request,
            Authentication authentication) {
        
        String currentUser = authentication.getName();
        log.info("Creating order for user: {}", currentUser);
        
        OrderResponse order = orderService.createOrder(request, currentUser);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(order.id());
        
        return ResponseEntity.created(location).body(order);
    }
}
```

#### 5. Service Layer (Business Logic)
```java
@Service
@Slf4j
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final CustomerRepository customerRepository;
    private final ProductRepository productRepository;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;
    
    public OrderService(
            OrderRepository orderRepository,
            CustomerRepository customerRepository,
            ProductRepository productRepository,
            InventoryService inventoryService,
            NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.customerRepository = customerRepository;
        this.productRepository = productRepository;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }
    
    @Transactional
    public OrderResponse createOrder(CreateOrderRequest request, String createdBy) {
        log.info("Creating order for customer: {}", request.customerId());
        
        // Business logic
        Customer customer = customerRepository.findById(request.customerId())
            .orElseThrow(() -> new ResourceNotFoundException("Customer", request.customerId()));
        
        Order order = new Order();
        order.setCustomer(customer);
        order.setCreatedBy(createdBy);
        order.setStatus(OrderStatus.PENDING);
        
        for (OrderItemRequest itemRequest : request.items()) {
            Product product = productRepository.findById(itemRequest.productId())
                .orElseThrow(() -> new ResourceNotFoundException("Product", itemRequest.productId()));
            
            OrderItem item = new OrderItem();
            item.setProduct(product);
            item.setQuantity(itemRequest.quantity());
            item.setUnitPrice(product.getPrice());
            order.addItem(item);
        }
        
        order.calculateTotal();
        Order savedOrder = orderRepository.save(order);
        
        // Update inventory (in same transaction)
        inventoryService.reserveItems(savedOrder);
        
        log.info("Order created successfully: {}", savedOrder.getId());
        
        return OrderResponse.fromEntity(savedOrder);
    }
}
```

#### 6. Repository Layer (Data Access)
```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @EntityGraph(attributePaths = {"customer", "items", "items.product"})
    List<Order> findByCustomerId(Long customerId);
    
    @Query("SELECT o FROM Order o WHERE o.customer.id = :customerId AND o.createdAt > :date")
    List<Order> findRecentOrdersByCustomer(
            @Param("customerId") Long customerId,
            @Param("date") LocalDateTime date);
}
```

#### 7. Database Operations
```sql
-- SQL generated by Spring Data JPA
INSERT INTO orders (customer_id, created_by, status, total, created_at, updated_at)
VALUES (123, 'john@example.com', 'PENDING', 299.99, NOW(), NOW());

INSERT INTO order_items (order_id, product_id, quantity, unit_price, subtotal)
VALUES (1, 456, 2, 99.99, 199.98), (1, 789, 1, 99.99, 99.99);

UPDATE products SET stock = stock - 2 WHERE id = 456;
UPDATE products SET stock = stock - 1 WHERE id = 789;
```

#### 8. Exception Handling
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex,
            WebRequest request) {
        
        log.error("Resource not found: {}", ex.getMessage());
        
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.NOT_FOUND,
            ex.getMessage(),
            request.getDescription(false)
        );
        
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(
            BusinessException ex,
            WebRequest request) {
        
        log.error("Business rule violation: {}", ex.getMessage());
        
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.UNPROCESSABLE_ENTITY,
            ex.getMessage(),
            request.getDescription(false)
        );
        
        return new ResponseEntity<>(error, HttpStatus.UNPROCESSABLE_ENTITY);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        
        Map<String, String> errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                error -> error.getDefaultMessage()
            ));
        
        return ResponseEntity.badRequest()
            .body(ValidationErrorResponse.of(errors));
    }
}
```

#### 9. Response to Client
```json
HTTP/1.1 201 Created
Location: /api/orders/1
Content-Type: application/json

{
  "id": 1,
  "orderNumber": "ORD-2024-001",
  "customerId": 123,
  "customerName": "John Doe",
  "items": [
    {
      "productId": 456,
      "productName": "Laptop",
      "quantity": 2,
      "unitPrice": 99.99,
      "subtotal": 199.98
    },
    {
      "productId": 789,
      "productName": "Mouse",
      "quantity": 1,
      "unitPrice": 99.99,
      "subtotal": 99.99
    }
  ],
  "total": 299.97,
  "status": "CONFIRMED",
  "createdAt": "2024-01-15T10:30:00"
}
```

### Performance Considerations

```java
// Use pagination for large datasets
@GetMapping("/orders")
public ResponseEntity<Page<OrderResponse>> getOrders(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    
    Pageable pageable = PageRequest.of(page, size);
    Page<Order> orders = orderRepository.findAll(pageable);
    
    return ResponseEntity.ok(orders.map(OrderResponse::fromEntity));
}

// Use projections to fetch only needed columns
public interface OrderSummary {
    Long getId();
    String getOrderNumber();
    BigDecimal getTotal();
    LocalDateTime getCreatedAt();
}

@Query("SELECT new com.example.dto.OrderSummary(o.id, o.orderNumber, o.total, o.createdAt) " +
       "FROM Order o WHERE o.customer.id = :customerId")
List<OrderSummary> findOrderSummariesByCustomerId(@Param("customerId") Long customerId);

// Use caching for frequently accessed data
@Service
public class ProductService {
    
    @Cacheable("products")
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElseThrow();
    }
    
    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }
}
```

---

## 17. JWT Authentication: Building Stateless Security

### What is JWT?

JSON Web Token (JWT) is an open standard (RFC 7519) for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed.

### JWT Structure

A JWT consists of three parts separated by dots:
```
header.payload.signature
```

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT Components

```java
// Header
{
  "alg": "HS256",  // Signing algorithm
  "typ": "JWT"     // Token type
}

// Payload (Claims)
{
  "sub": "1234567890",      // Subject (user ID)
  "name": "John Doe",       // Custom claim
  "email": "john@example.com", // Custom claim
  "roles": ["USER", "ADMIN"], // Custom claim
  "iat": 1516239022,        // Issued At
  "exp": 1516239022         // Expiration
}

// Signature
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secretKey
)
```

### JWT Authentication Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant A as Auth Controller
    participant S as Security Service
    participant J as JWT Provider
    participant DB as Database
    
    C->>A: POST /auth/login<br/>{email, password}
    A->>S: authenticate(credentials)
    S->>DB: findUserByEmail(email)
    DB-->>S: User entity
    S->>S: validatePassword(password, encodedPassword)
    
    alt Valid Credentials
        S->>J: generateToken(user)
        J->>J: Create JWT with claims
        J->>J: Sign with secret key
        J-->>S: JWT Token
        S-->>A: AuthenticationResponse(token)
        A-->>C: 200 OK + JWT Token
    else Invalid Credentials
        S-->>A: AuthenticationException
        A-->>C: 401 Unauthorized
    end
    
    Note over C: Store JWT (localStorage, etc.)
    
    C->>C: Subsequent Request with JWT
    C->>C: Add Authorization: Bearer <token>
    
    C->>A: GET /api/users/me<br/>Authorization: Bearer eyJhbG...
    A->>J: validateToken(token)
    J->>J: Verify signature
    J->>J: Check expiration
    J->>J: Extract claims
    J-->>A: User ID from token
    
    A->>DB: findUserById(userId)
    DB-->>A: User entity
    A-->>C: 200 OK + User Data
```

**Figure 13: JWT Authentication Flow**

### Implementing JWT Authentication

#### Step 1: JWT Utility Class

```java
@Component
public class JwtTokenProvider {
    
    @Value("${app.jwt.secret}")
    private String jwtSecret;
    
    @Value("${app.jwt.expiration-ms}")
    private long jwtExpirationMs;
    
    // Generate JWT token
    public String generateToken(String username, List<String> roles) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationMs);
        
        return Jwts.builder()
            .setSubject(username)
            .claim("roles", roles)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(Keys.hmacShaKeyFor(jwtSecret.getBytes()), SignatureAlgorithm.HS256)
            .compact();
    }
    
    // Get username from token
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(Keys.hmacShaKeyFor(jwtSecret.getBytes()))
            .build()
            .parseClaimsJws(token)
            .getBody();
        
        return claims.getSubject();
    }
    
    // Get roles from token
    public List<String> getRolesFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        List<String> roles = claims.get("roles", List.class);
        return roles != null ? roles : List.of();
    }
    
    // Validate token
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(Keys.hmacShaKeyFor(jwtSecret.getBytes()))
                .build()
                .parseClaimsJws(token);
            
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
            return false;
        }
    }
    
    // Get expiration date
    public Date getExpirationDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }
    
    private Claims getClaimsFromToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(Keys.hmacShaKeyFor(jwtSecret.getBytes()))
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    public boolean isTokenExpired(String token) {
        Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }
}
```

#### Step 2: JWT Authentication Filter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtTokenProvider tokenProvider;
    private final UserDetailsService userDetailsService;
    
    public JwtAuthenticationFilter(
            JwtTokenProvider tokenProvider,
            UserDetailsService userDetailsService) {
        this.tokenProvider = tokenProvider;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {
        
        String token = resolveToken(request);
        
        if (token != null && tokenProvider.validateToken(token)) {
            String username = tokenProvider.getUsernameFromToken(token);
            List<String> roles = tokenProvider.getRolesFromToken(token);
            
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            UsernamePasswordAuthenticationToken authentication = 
                new UsernamePasswordAuthenticationToken(
                    userDetails, 
                    null, 
                    userDetails.getAuthorities()
                );
            
            authentication.setDetails(
                new WebAuthenticationDetailsSource().buildDetails(request)
            );
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        
        return null;
    }
}
```

#### Step 3: Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    private final JwtTokenProvider tokenProvider;
    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;
    
    public SecurityConfig(
            JwtTokenProvider tokenProvider,
            UserDetailsService userDetailsService,
            PasswordEncoder passwordEncoder) {
        this.tokenProvider = tokenProvider;
        this.userDetailsService = userDetailsService;
        this.passwordEncoder = passwordEncoder;
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public JwtAuthenticationFilter jwtAuthenticationFilter() {
        return new JwtAuthenticationFilter(tokenProvider, userDetailsService);
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

#### Step 4: Authentication Controller

```java
@RestController
@RequestMapping("/api/auth")
@Validated
public class AuthController {
    
    private final AuthenticationService authenticationService;
    
    public AuthController(AuthenticationService authenticationService) {
        this.authenticationService = authenticationService;
    }
    
    @PostMapping("/login")
    public ResponseEntity<AuthenticationResponse> login(
            @Valid @RequestBody LoginRequest request) {
        
        AuthenticationResponse response = authenticationService.authenticate(request);
        return ResponseEntity.ok(response);
    }
    
    @PostMapping("/refresh")
    public ResponseEntity<AuthenticationResponse> refreshToken(
            @RequestHeader("Authorization") String authorizationHeader) {
        
        String token = authorizationHeader.substring(7);
        AuthenticationResponse response = authenticationService.refreshToken(token);
        return ResponseEntity.ok(response);
    }
    
    @PostMapping("/logout")
    public ResponseEntity<Void> logout() {
        // Invalidate token (if using token blacklist)
        authenticationService.logout();
        return ResponseEntity.ok().build();
    }
}

// Request/Response DTOs
public record LoginRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password
) {}

public record AuthenticationResponse(
    String accessToken,
    String refreshToken,
    long expiresIn,
    String tokenType
) {
    public static AuthenticationResponse of(String accessToken, String refreshToken, long expiresIn) {
        return new AuthenticationResponse(
            accessToken,
            refreshToken,
            expiresIn,
            "Bearer"
        );
    }
}
```

#### Step 5: Authentication Service

```java
@Service
@Slf4j
public class AuthenticationService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider tokenProvider;
    private final RefreshTokenRepository refreshTokenRepository;
    
    public AuthenticationService(
            UserRepository userRepository,
            PasswordEncoder passwordEncoder,
            JwtTokenProvider tokenProvider,
            RefreshTokenRepository refreshTokenRepository) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.tokenProvider = tokenProvider;
        this.refreshTokenRepository = refreshTokenRepository;
    }
    
    @Transactional
    public AuthenticationResponse authenticate(LoginRequest request) {
        log.info("Authentication attempt for email: {}", request.email());
        
        User user = userRepository.findByEmail(request.email())
            .orElseThrow(() -> new AuthenticationException("Invalid email or password"));
        
        if (!passwordEncoder.matches(request.password(), user.getPassword())) {
            log.warn("Invalid password for user: {}", request.email());
            throw new AuthenticationException("Invalid email or password");
        }
        
        if (!user.isActive()) {
            throw new AuthenticationException("Account is disabled");
        }
        
        // Generate tokens
        List<String> roles = user.getRoles().stream()
            .map(role -> role.getName().name())
            .toList();
        
        String accessToken = tokenProvider.generateToken(user.getEmail(), roles);
        String refreshToken = generateRefreshToken(user);
        
        // Save refresh token
        RefreshToken refreshTokenEntity = new RefreshToken();
        refreshTokenEntity.setToken(refreshToken);
        refreshTokenEntity.setUser(user);
        refreshTokenEntity.setExpiryDate(LocalDateTime.now().plusDays(7));
        refreshTokenRepository.save(refreshTokenEntity);
        
        long expiresIn = tokenProvider.getExpirationDateFromToken(accessToken).getTime() - System.currentTimeMillis();
        
        log.info("User authenticated successfully: {}", user.getEmail());
        
        return AuthenticationResponse.of(accessToken, refreshToken, expiresIn);
    }
    
    @Transactional
    public AuthenticationResponse refreshToken(String refreshToken) {
        RefreshToken token = refreshTokenRepository.findByToken(refreshToken)
            .orElseThrow(() -> new AuthenticationException("Invalid refresh token"));
        
        if (token.isExpired()) {
            refreshTokenRepository.delete(token);
            throw new AuthenticationException("Refresh token expired");
        }
        
        User user = token.getUser();
        List<String> roles = user.getRoles().stream()
            .map(role -> role.getName().name())
            .toList();
        
        String newAccessToken = tokenProvider.generateToken(user.getEmail(), roles);
        long expiresIn = tokenProvider.getExpirationDateFromToken(newAccessToken).getTime() - System.currentTimeMillis();
        
        return AuthenticationResponse.of(newAccessToken, refreshToken, expiresIn);
    }
    
    @Transactional
    public void logout(String refreshToken) {
        refreshTokenRepository.findByToken(refreshToken)
            .ifPresent(refreshTokenRepository::delete);
    }
    
    private String generateRefreshToken(User user) {
        return tokenProvider.generateToken(user.getEmail(), List.of());
    }
}
```

### Using JWT in Requests

```bash
# Login
POST /api/auth/login
{
  "email": "john@example.com",
  "password": "SecurePass123!"
}

# Response
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 86400000,
  "tokenType": "Bearer"
}

# Use access token in subsequent requests
GET /api/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Refresh access token
POST /api/auth/refresh
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### JWT Best Practices

✅ **DO:**
- Use strong, random secret keys (at least 256 bits for HS256)
- Set appropriate expiration times (short-lived access tokens)
- Use refresh tokens for long-lived sessions
- Store tokens securely (HttpOnly cookies, secure storage)
- Validate tokens on every request
- Implement token revocation/blacklisting when needed
- Use HTTPS in production

❌ **DON'T:**
- Store sensitive data in JWT payload (it's base64 encoded, not encrypted)
- Use weak signing algorithms (none, HS384, HS512 are okay; avoid none)
- Use long expiration times for access tokens
- Store JWT in localStorage vulnerable to XSS attacks
- Send JWT in URL parameters
- Use the same secret for all applications

### Security Considerations

```java
// Configuration
app.jwt.secret=MyVeryLongAndSecureSecretKeyThatIsAtLeast256BitsLongForHS256Algorithm123456789
app.jwt.expiration-ms=86400000  # 24 hours
app.jwt.refresh-token-expiration-ms=604800000  # 7 days

// Use different secrets for different environments
// Production: Use environment variables or vault
// Development: Use application-dev.properties
```

---

## 18. Profiles and Environment Configuration

### Why Use Profiles?

Software rarely runs in just one environment. During development, developers use local databases and debugging tools. Testing environments connect to separate servers. Production systems use secure databases, cloud storage, and optimized configurations.

### Profile-Specific Configuration

```yaml
# application.yml (common configuration)
server:
  port: 8080

spring:
  application:
    name: ecommerce-api
  
  datasource:
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10

---
# application-dev.yml (Development)
spring:
  config:
    activate:
      on-profile: dev
  
  datasource:
    url: jdbc:postgresql://localhost:5432/ecommerce_dev
    username: dev_user
    password: dev_pass
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    com.example: DEBUG
    org.hibernate.SQL: DEBUG

---
# application-test.yml (Testing)
spring:
  config:
    activate:
      on-profile: test
  
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password: password
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: false

---
# application-prod.yml (Production)
spring:
  config:
    activate:
      on-profile: prod
  
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

logging:
  level:
    com.example: INFO
  file:
    name: /var/log/ecommerce/app.log
```

### Activating Profiles

```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Multiple profiles
java -jar app.jar --spring.profiles.active=prod,aws

# Environment variable
SPRING_PROFILES_ACTIVE=prod java -jar app.jar

# application.properties
spring.profiles.active=prod
```

### Profile-Specific Beans

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/ecommerce_dev");
        config.setUsername("dev_user");
        config.setPassword("dev_pass");
        return new HikariDataSource(config);
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(env.getProperty("DATABASE_URL"));
        config.setUsername(env.getProperty("DATABASE_USERNAME"));
        config.setPassword(env.getProperty("DATABASE_PASSWORD"));
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }
}

// Or use @Profile on components
@Component
@Profile("dev")
public class DevEmailService implements EmailService {
    // Logs emails to console instead of sending
}

@Component
@Profile("prod")
public class ProdEmailService implements EmailService {
    // Actually sends emails
}
```

### Conditional Beans

```java
@Configuration
public class CacheConfig {
    
    @Bean
    @ConditionalOnProperty(prefix = "app", name = "cache-enabled", havingValue = "true")
    public CacheManager cacheManager() {
        return RedisCacheManager.builder(redisConnectionFactory)
            .build();
    }
    
    @Bean
    @ConditionalOnMissingBean(CacheManager.class)
    public CacheManager noOpCacheManager() {
        return new NoOpCacheManager();
    }
}
```

### Best Practices

✅ **DO:**
- Use profiles for environment-specific configuration
- Keep sensitive data in environment variables
- Use different databases for different environments
- Enable debug logging in development
- Disable debug features in production
- Document profile-specific settings

❌ **DON'T:**
- Commit production credentials to version control
- Use production profile for development
- Mix environment-specific code with business logic
- Forget to test all profiles

---

## 19. Logging and Monitoring: Understanding Your Application in Production

### Importance of Logging

Writing software is only part of the job. Keeping software healthy after deployment is equally important. Without logs, identifying issues becomes extremely difficult.

### Spring Boot Logging

Spring Boot uses SLF4J with Logback by default:

```properties
# application.properties

# Logging levels
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# Log file output
logging.file.name=logs/app.log
logging.file.max-size=10MB
logging.file.max-history=30

# Log pattern
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
```

### Structured Logging

```java
@Service
@Slf4j
public class OrderService {
    
    public Order createOrder(OrderRequest request) {
        // Use parameterized logging (avoids string concatenation if disabled)
        log.info("Creating order for customer: {}, items: {}", 
            request.customerId(), 
            request.items().size());
        
        try {
            Order order = new Order(request);
            Order savedOrder = orderRepository.save(order);
            
            log.info("Order created successfully: orderId={}, orderNumber={}", 
                savedOrder.getId(), 
                savedOrder.getOrderNumber());
            
            return savedOrder;
            
        } catch (Exception e) {
            log.error("Failed to create order for customer: {}", request.customerId(), e);
            throw e;
        }
    }
}
```

### Spring Boot Actuator

Spring Boot Actuator provides production-ready monitoring endpoints:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# application.properties
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=when-authorized
management.info.env.enabled=true
```

**Available Endpoints:**
- `/actuator/health` - Application health status
- `/actuator/info` - Application information
- `/actuator/metrics` - Application metrics
- `/actuator/prometheus` - Prometheus metrics
- `/actuator/env` - Environment properties
- `/actuator/beans` - Spring beans
- `/actuator/configprops` - Configuration properties
- `/actuator/threaddump` - Thread dump
- `/actuator/heapdump` - Heap dump

### Custom Health Indicators

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    private final DataSource dataSource;
    
    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            DatabaseMetaData metaData = connection.getMetaData();
            String databaseProduct = metaData.getDatabaseProductName();
            
            return Health.up()
                .withDetail("database", databaseProduct)
                .withDetail("version", metaData.getDatabaseProductVersion())
                .build();
        } catch (SQLException e) {
            return Health.down()
                .withException(e)
                .build();
        }
    }
}

@Component
public class CustomHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        // Check external service
        boolean externalServiceUp = checkExternalService();
        
        if (externalServiceUp) {
            return Health.up()
                .withDetail("externalService", "Available")
                .build();
        } else {
            return Health.down()
                .withDetail("externalService", "Unavailable")
                .build();
        }
    }
    
    private boolean checkExternalService() {
        // Check logic
        return true;
    }
}
```

### Custom Metrics

```java
@Component
public class OrderMetrics {
    
    private final Counter orderCounter;
    private final Timer orderTimer;
    private final DistributionSummary orderValueSummary;
    
    public OrderMetrics(MeterRegistry meterRegistry) {
        this.orderCounter = Counter.builder("orders.created")
            .description("Number of orders created")
            .register(meterRegistry);
        
        this.orderTimer = Timer.builder("orders.processing.time")
            .description("Time taken to process orders")
            .register(meterRegistry);
        
        this.orderValueSummary = DistributionSummary.builder("orders.value")
            .description("Order value distribution")
            .baseUnit("dollars")
            .register(meterRegistry);
    }
    
    public void recordOrderCreated(Order order) {
        orderCounter.increment();
        orderValueSummary.record(order.getTotal().doubleValue());
    }
    
    public Timer.Sample startOrderTimer() {
        return Timer.start(meterRegistry);
    }
}

// Usage
@Service
public class OrderService {
    private final OrderMetrics orderMetrics;
    
    public Order createOrder(OrderRequest request) {
        Timer.Sample sample = orderMetrics.startOrderTimer();
        
        Order order = new Order(request);
        Order savedOrder = orderRepository.save(order);
        
        orderMetrics.recordOrderCreated(savedOrder);
        
        sample.stop(orderMetrics.getOrderTimer());
        
        return savedOrder;
    }
}
```

### Monitoring with Prometheus and Grafana

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
# application.properties
management.endpoints.web.exposure.include=prometheus
management.metrics.export.prometheus.enabled=true
```

Visit `http://localhost:8080/actuator/prometheus` to see metrics in Prometheus format.

### Logging Best Practices

✅ **DO:**
- Use appropriate log levels (ERROR, WARN, INFO, DEBUG, TRACE)
- Include context in log messages (user ID, request ID, etc.)
- Use structured logging (JSON format for production)
- Log exceptions with stack traces
- Use correlation IDs for request tracing
- Implement log rotation
- Centralize logs (ELK, Splunk, Datadog)

❌ **DON'T:**
- Log sensitive data (passwords, credit cards, tokens)
- Use System.out/System.err
- Log at inappropriate levels (DEBUG in production)
- Create excessive log volume
- Forget to log errors

### Monitoring Best Practices

✅ **DO:**
- Monitor application health
- Track key metrics (response time, error rate, throughput)
- Set up alerts for critical issues
- Use distributed tracing (Jaeger, Zipkin)
- Monitor database performance
- Track business metrics (orders, users, revenue)

❌ **DON'T:**
- Monitor everything (focus on key metrics)
- Set too many alerts (alert fatigue)
- Ignore warning signs
- Forget to monitor external dependencies

---

## 20. Testing in Spring Boot: Writing Software You Can Trust

### Why Testing is Essential

Imagine deploying a new feature. Everything appears correct. Hours later, customers begin reporting broken functionality. The issue wasn't caused by the new feature itself—it accidentally broke an existing one.

**Testing prevents situations like this.**

### Types of Tests

```mermaid
graph TD
    A[Testing Pyramid] --> B[Unit Tests]
    A --> C[Integration Tests]
    A --> D[API Tests]
    A --> E[End-to-End Tests]
    
    B --> B1[Fast]
    B --> B2[Cheap]
    B --> B3[Many]
    
    C --> C1[Medium Speed]
    C --> C2[Medium Cost]
    C --> C3[Some]
    
    D --> D1[Slower]
    D --> D2[More Expensive]
    D --> D3[Few]
    
    E --> E1[Slow]
    E --> E2[Expensive]
    E --> E3[Very Few]
    
    style A fill:#4ecdc4
    style B fill:#95e1d3
    style C fill:#ffe66d
    style D fill:#ff6b6b
    style E fill:#ff0000
```

**Figure 14: Testing Pyramid**

### Unit Tests

Test individual components in isolation:

```java
// Service to test
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, 
                      PasswordEncoder passwordEncoder,
                      EmailService emailService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
    }
    
    public User createUser(CreateUserRequest request) {
        // Check if email exists
        if (userRepository.existsByEmail(request.email())) {
            throw new DuplicateResourceException("Email already exists");
        }
        
        // Create user
        User user = new User();
        user.setEmail(request.email());
        user.setPassword(passwordEncoder.encode(request.password()));
        user.setName(request.name());
        
        User savedUser = userRepository.save(user);
        
        // Send welcome email
        emailService.sendWelcomeEmail(savedUser.getEmail());
        
        return savedUser;
    }
}

// Unit test
class UserServiceTest {
    
    private UserRepository mockUserRepository;
    private PasswordEncoder mockPasswordEncoder;
    private EmailService mockEmailService;
    private UserService userService;
    
    @BeforeEach
    void setUp() {
        mockUserRepository = mock(UserRepository.class);
        mockPasswordEncoder = mock(PasswordEncoder.class);
        mockEmailService = mock(EmailService.class);
        
        userService = new UserService(
            mockUserRepository,
            mockPasswordEncoder,
            mockEmailService
        );
    }
    
    @Test
    void shouldCreateUserSuccessfully() {
        // Given
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "SecurePass123!",
            "John Doe"
        );
        
        User savedUser = new User(1L, "john@example.com", "encodedPassword", "John Doe");
        
        when(mockUserRepository.existsByEmail(request.email())).thenReturn(false);
        when(mockPasswordEncoder.encode(request.password())).thenReturn("encodedPassword");
        when(mockUserRepository.save(any(User.class))).thenReturn(savedUser);
        
        // When
        User result = userService.createUser(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        
        verify(mockUserRepository).existsByEmail(request.email());
        verify(mockPasswordEncoder).encode(request.password());
        verify(mockUserRepository).save(any(User.class));
        verify(mockEmailService).sendWelcomeEmail("john@example.com");
    }
    
    @Test
    void shouldThrowExceptionWhenEmailExists() {
        // Given
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "SecurePass123!",
            "John Doe"
        );
        
        when(mockUserRepository.existsByEmail(request.email())).thenReturn(true);
        
        // When/Then
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(DuplicateResourceException.class)
            .hasMessage("Email already exists");
        
        verify(mockUserRepository, never()).save(any());
        verify(mockEmailService, never()).sendWelcomeEmail(any());
    }
}
```

### Integration Tests

Test multiple components working together:

```java
@SpringBootTest
@AutoConfigureTestDatabase
class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    void shouldCreateUserSuccessfully() {
        // Given
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "SecurePass123!",
            "John Doe"
        );
        
        // When
        User result = userService.createUser(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isNotNull();
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        assertThat(result.getPassword()).isNotEqualTo("SecurePass123!"); // Encoded
        
        // Verify in database
        Optional<User> found = userRepository.findById(result.getId());
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("john@example.com");
    }
    
    @Test
    void shouldNotCreateUserWithDuplicateEmail() {
        // Given
        User existingUser = new User(null, "john@example.com", "password", "John Doe");
        userRepository.save(existingUser);
        
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "SecurePass456!",
            "John Smith"
        );
        
        // When/Then
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(DuplicateResourceException.class);
        
        // Verify only one user exists
        assertThat(userRepository.count()).isEqualTo(1);
    }
}
```

### API Tests (Controller Tests)

Test REST endpoints:

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void shouldCreateUserSuccessfully() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "SecurePass123!",
            "John Doe"
        );
        
        UserResponse response = new UserResponse(1L, "john@example.com", "John Doe", null, true);
        
        when(userService.createUser(any(CreateUserRequest.class))).thenReturn(response);
        
        // When/Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.email").value("john@example.com"))
            .andExpect(jsonPath("$.name").value("John Doe"));
        
        verify(userService).createUser(any(CreateUserRequest.class));
    }
    
    @Test
    void shouldReturnBadRequestWhenValidationFails() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest(
            "", // Invalid: empty email
            "123", // Invalid: too short
            "J" // Invalid: too short
        );
        
        // When/Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.fieldErrors.email").exists())
            .andExpect(jsonPath("$.fieldErrors.password").exists());
    }
    
    @Test
    void shouldReturnUserById() throws Exception {
        // Given
        Long userId = 1L;
        UserResponse response = new UserResponse(1L, "john@example.com", "John Doe", null, true);
        
        when(userService.getUserById(userId)).thenReturn(response);
        
        // When/Then
        mockMvc.perform(get("/api/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

### End-to-End Tests

Test complete user workflows:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase
@Testcontainers
class OrderE2ETest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private ProductRepository productRepository;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
        productRepository.deleteAll();
    }
    
    @Test
    void shouldCompleteOrderWorkflow() {
        // Step 1: Register user
        RegisterRequest registerRequest = new RegisterRequest(
            "john@example.com",
            "SecurePass123!",
            "John Doe"
        );
        
        ResponseEntity<UserResponse> registerResponse = restTemplate.postForEntity(
            "/api/auth/register",
            registerRequest,
            UserResponse.class
        );
        
        assertThat(registerResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        UserResponse user = registerResponse.getBody();
        assertThat(user).isNotNull();
        
        // Step 2: Login
        LoginRequest loginRequest = new LoginRequest("john@example.com", "SecurePass123!");
        
        ResponseEntity<AuthenticationResponse> loginResponse = restTemplate.postForEntity(
            "/api/auth/login",
            loginRequest,
            AuthenticationResponse.class
        );
        
        assertThat(loginResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        String token = loginResponse.getBody().accessToken();
        
        // Step 3: Create product
        Product product = new Product();
        product.setName("Laptop");
        product.setPrice(new BigDecimal("999.99"));
        product.setStock(10);
        Product savedProduct = productRepository.save(product);
        
        // Step 4: Create order
        CreateOrderRequest orderRequest = new CreateOrderRequest(
            user.id(),
            List.of(new OrderItemRequest(savedProduct.getId(), 2)),
            null,
            LocalDateTime.now().plusDays(7),
            true
        );
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<CreateOrderRequest> request = new HttpEntity<>(orderRequest, headers);
        
        ResponseEntity<OrderResponse> orderResponse = restTemplate.exchange(
            "/api/orders",
            HttpMethod.POST,
            request,
            OrderResponse.class
        );
        
        assertThat(orderResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        OrderResponse order = orderResponse.getBody();
        assertThat(order).isNotNull();
        assertThat(order.total()).isEqualByComparingTo(new BigDecimal("1999.98"));
        
        // Step 5: Get order
        ResponseEntity<OrderResponse> getOrderResponse = restTemplate.exchange(
            "/api/orders/{id}",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            OrderResponse.class,
            order.id()
        );
        
        assertThat(getOrderResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getOrderResponse.getBody()).isNotNull();
    }
}
```

### Test Configuration

```java
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    @Primary
    public EmailService emailService() {
        return mock(EmailService.class);
    }
}

// application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: false
```

### Testing Best Practices

✅ **DO:**
- Write tests for critical business logic
- Use descriptive test names
- Follow Arrange-Act-Assert pattern
- Test edge cases and error conditions
- Use mocking for external dependencies
- Keep tests independent
- Run tests in CI/CD pipeline

❌ **DON'T:**
- Test implementation details
- Write tests that depend on execution order
- Mock everything (integration tests are important too)
- Ignore failing tests
- Write tests without assertions

### Test Coverage

```xml
<dependency>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
</dependency>
```

```bash
# Run tests with coverage
mvn clean test jacoco:report

# View report
open target/site/jacoco/index.html
```

---

## 21. Building Production-Ready Spring Boot Applications

### Production Readiness Checklist

Building production-ready applications involves much more than functionality:

#### Security
- ✅ Use HTTPS
- ✅ Encrypt passwords with BCrypt
- ✅ Implement proper authentication and authorization
- ✅ Validate all inputs
- ✅ Use least-privilege principles
- ✅ Keep dependencies updated
- ✅ Implement rate limiting
- ✅ Use security headers

#### Performance
- ✅ Optimize database queries
- ✅ Use caching where appropriate
- ✅ Implement pagination
- ✅ Avoid N+1 query problems
- ✅ Use connection pooling
- ✅ Monitor response times
- ✅ Profile application regularly

#### Scalability
- ✅ Design stateless applications
- ✅ Use horizontal scaling
- ✅ Implement load balancing
- ✅ Use message queues for async operations
- ✅ Design for failure
- ✅ Implement circuit breakers

#### Maintainability
- ✅ Follow layered architecture
- ✅ Write clean, readable code
- ✅ Keep methods focused
- ✅ Avoid duplicated logic
- ✅ Document APIs
- ✅ Use consistent naming conventions

#### Observability
- ✅ Collect logs
- ✅ Monitor performance
- ✅ Track application health
- ✅ Implement distributed tracing
- ✅ Set up alerts
- ✅ Create dashboards

### Production Configuration

```yaml
# application-prod.yml
server:
  port: 8080
  error:
    include-message: never # Don't expose error messages
    include-stacktrace: never
    include-exception: false

spring:
  profiles:
    active: prod
  
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
  
  jpa:
    open-in-view: false # Prevent Open Session in View
    properties:
      hibernate:
        jdbc:
          batch_size: 20
          order_inserts: true
          order_updates: true
  
  cache:
    type: redis
    redis:
      time-to-live: 3600000

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    distribution:
      percentiles-histogram:
        http:
          server:
            requests: true

logging:
  level:
    com.example: INFO
  file:
    name: /var/log/ecommerce/app.log
  logback:
    rollingpolicy:
      max-file-size: 100MB
      max-history: 30
```

### Docker Configuration

```dockerfile
# Multi-stage build
FROM eclipse-temurin:17-jdk-jammy AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN apt-get update && apt-get install -y maven
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-jammy
WORKDIR /app

# Copy only the JAR file
COPY --from=builder /app/target/*.jar app.jar

# Run as non-root user
RUN addgroup --system spring && adduser --system spring
USER spring:spring

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-api
  labels:
    app: ecommerce-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ecommerce-api
  template:
    metadata:
      labels:
        app: ecommerce-api
    spec:
      containers:
      - name: ecommerce-api
        image: ecommerce-api:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        - name: DATABASE_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-api
spec:
  selector:
    app: ecommerce-api
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

### CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven
    
    - name: Run tests
      run: mvn clean test
    
    - name: Generate coverage report
      run: mvn jacoco:report
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./target/site/jacoco/jacoco.xml
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven
    
    - name: Build with Maven
      run: mvn clean package -DskipTests
    
    - name: Build Docker image
      run: docker build -t ecommerce-api:${{ github.sha }} .
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push ecommerce-api:${{ github.sha }}
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/ecommerce-api api=ecommerce-api:${{ github.sha }}
        kubectl rollout status deployment/ecommerce-api
```

### Monitoring and Alerting

```yaml
# Prometheus alert rules
groups:
  - name: ecommerce_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests per second"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_server_requests_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          description: "95th percentile response time is {{ $value }} seconds"
      
      - alert: ServiceDown
        expr: up{job="ecommerce-api"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"
```

---

## Common Mistakes Spring Boot Developers Make

### 1. Using Field Injection Everywhere

❌ **Bad:**
```java
@Service
public class BadService {
    @Autowired
    private UserRepository userRepository; // Hidden dependency
    
    @Autowired
    private EmailService emailService; // Hard to test
}
```

✅ **Good:**
```java
@Service
public class GoodService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public GoodService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

### 2. Placing Business Logic Inside Controllers

❌ **Bad:**
```java
@RestController
public class BadController {
    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest request) {
        // Business logic in controller
        if (request.total() <= 0) {
            throw new ValidationException("Invalid total");
        }
        // ... more business logic
        return orderRepository.save(order);
    }
}
```

✅ **Good:**
```java
@RestController
public class GoodController {
    private final OrderService orderService;
    
    public GoodController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping("/orders")
    public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody OrderRequest request) {
        OrderResponse order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}
```

### 3. Returning Entities Directly

❌ **Bad:**
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id); // Exposes password, internal fields
}
```

✅ **Good:**
```java
@GetMapping("/users/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
    UserResponse user = userService.getUserById(id);
    return ResponseEntity.ok(user);
}
```

### 4. Ignoring Exception Handling

❌ **Bad:**
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id); // Throws 500 if not found
}
```

✅ **Good:**
```java
@GetMapping("/users/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
    UserResponse user = userService.getUserById(id)
        .orElseThrow(() -> new ResourceNotFoundException("User", id));
    return ResponseEntity.ok(user);
}
```

### 5. Hardcoding Configuration Values

❌ **Bad:**
```java
@Service
public class BadService {
    private final String emailHost = "smtp.gmail.com"; // Hardcoded
    private final int emailPort = 587; // Hardcoded
}
```

✅ **Good:**
```java
@Component
@ConfigurationProperties(prefix = "app.email")
public class EmailConfig {
    private String host;
    private int port;
    // Getters and setters
}

@Service
public class GoodService {
    private final EmailConfig emailConfig;
    
    public GoodService(EmailConfig emailConfig) {
        this.emailConfig = emailConfig;
    }
}
```

### 6. Writing Huge Service Classes

❌ **Bad:**
```java
@Service
public class BadService {
    // 5000 lines of code doing everything
    public void createUser() { /* ... */ }
    public void updateUser() { /* ... */ }
    public void deleteUser() { /* ... */ }
    public void processOrder() { /* ... */ }
    public void sendEmail() { /* ... */ }
    public void generateReport() { /* ... */ }
    // ... 50 more methods
}
```

✅ **Good:**
```java
@Service
public class UserService { /* User-related operations */ }
@Service
public class OrderService { /* Order-related operations */ }
@Service
public class EmailService { /* Email operations */ }
@Service
public class ReportService { /* Report generation */ }
```

### 7. Skipping Validation

❌ **Bad:**
```java
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userRepository.save(user); // No validation
}
```

✅ **Good:**
```java
@PostMapping("/users")
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserResponse user = userService.createUser(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
}
```

### 8. Ignoring Database Performance

❌ **Bad:**
```java
@GetMapping("/orders")
public List<Order> getOrders() {
    return orderRepository.findAll(); // Fetches all orders with N+1 problem
}
```

✅ **Good:**
```java
@GetMapping("/orders")
public Page<OrderResponse> getOrders(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    
    Pageable pageable = PageRequest.of(page, size);
    return orderRepository.findAll(pageable).map(OrderResponse::fromEntity);
}
```

### 9. Forgetting About Security

❌ **Bad:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests().anyRequest().permitAll(); // No security!
        return http.build();
    }
}
```

✅ **Good:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**", "/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

### 10. Avoiding Automated Tests

❌ **Bad:**
```java
// No tests at all
// "I'll test it manually"
```

✅ **Good:**
```java
// Comprehensive test suite
class UserServiceTest { /* Unit tests */ }
class UserControllerTest { /* API tests */ }
class UserServiceIntegrationTest { /* Integration tests */ }
class OrderE2ETest { /* End-to-end tests */ }
```

---

## Best Practices Every Spring Boot Developer Should Follow

### Architecture & Design

1. **Follow Layered Architecture**
   - Controllers: Handle HTTP requests/responses
   - Services: Business logic
   - Repositories: Data access
   - Entities: Domain models
   - DTOs: Data transfer

2. **Use Constructor Injection**
   - Makes dependencies explicit
   - Enables immutability
   - Easier testing

3. **Keep Controllers Thin**
   - Controllers should only handle HTTP concerns
   - Delegate business logic to services
   - Return DTOs, not entities

4. **Single Responsibility Principle**
   - Each class should have one reason to change
   - Split large classes into smaller ones
   - Use interfaces for flexibility

### Security

5. **Never Store Plain Text Passwords**
   - Always use BCrypt or Argon2
   - Never log passwords
   - Implement password policies

6. **Implement Proper Authentication**
   - Use JWT or OAuth2 for APIs
   - Implement refresh tokens
   - Use HTTPS in production

7. **Validate All Inputs**
   - Use Bean Validation annotations
   - Validate at multiple layers
   - Never trust client-side validation

8. **Use Security Headers**
   - Implement CORS properly
   - Use CSRF protection where needed
   - Add security headers (CSP, X-Frame-Options)

### Data Management

9. **Use Transactions Appropriately**
   - Keep transactions short
   - Use appropriate isolation levels
   - Handle rollback scenarios

10. **Avoid N+1 Queries**
    - Use JOIN FETCH
    - Use EntityGraph
    - Monitor query performance

11. **Use DTOs**
    - Don't expose entities directly
    - Control data exposure
    - Version your APIs

### Performance

12. **Implement Caching**
    - Cache frequently accessed data
    - Use appropriate cache eviction policies
    - Monitor cache hit rates

13. **Use Pagination**
    - Never return unbounded result sets
    - Use reasonable page sizes
    - Provide sorting options

14. **Optimize Database Queries**
    - Add indexes for frequently queried columns
    - Use query profiling
    - Avoid SELECT *

### Code Quality

15. **Write Tests**
    - Unit tests for business logic
    - Integration tests for data access
    - API tests for endpoints
    - Aim for 80%+ coverage

16. **Use Meaningful Names**
    - Class names: nouns (UserService, OrderRepository)
    - Method names: verbs (createUser, findById)
    - Variables: descriptive (userList, not list)

17. **Handle Exceptions Properly**
    - Use global exception handler
    - Return appropriate HTTP status codes
    - Provide meaningful error messages

18. **Log Appropriately**
    - Use appropriate log levels
    - Include context (user ID, request ID)
    - Don't log sensitive data

### Configuration

19. **Externalize Configuration**
    - Use application.properties/yml
    - Use environment variables for sensitive data
    - Use profiles for different environments

20. **Document Your API**
    - Use OpenAPI/Swagger
    - Document request/response formats
    - Provide examples

### Deployment

21. **Use Docker**
    - Containerize applications
    - Use multi-stage builds
    - Run as non-root user

22. **Implement Health Checks**
    - Use Actuator health endpoints
    - Monitor dependencies
    - Set up alerts

23. **Use CI/CD**
    - Automate testing
    - Automate deployment
    - Use blue-green or canary deployments

---

## Practice Exercises

### Exercise 1: IoC Container and Dependency Injection

**Difficulty:** ⭐ Easy

**Task:** Create a simple book management system with proper dependency injection.

**Requirements:**
1. Create a `Book` entity with fields: id, title, author, isbn, publishedDate
2. Create a `BookRepository` interface extending JpaRepository
3. Create a `BookService` with methods: createBook, getAllBooks, getBookById, deleteBook
4. Create a `BookController` with REST endpoints
5. Use constructor injection everywhere

**Solution:**
```java
// Book.java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String author;
    @Column(unique = true)
    private String isbn;
    private LocalDate publishedDate;
    // Getters, setters, constructors
}

// BookRepository.java
public interface BookRepository extends JpaRepository<Book, Long> {
    Optional<Book> findByIsbn(String isbn);
    boolean existsByIsbn(String isbn);
}

// BookService.java
@Service
public class BookService {
    private final BookRepository bookRepository;
    
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
    
    public Book createBook(Book book) {
        if (bookRepository.existsByIsbn(book.getIsbn())) {
            throw new DuplicateResourceException("Book with ISBN already exists");
        }
        return bookRepository.save(book);
    }
    
    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }
    
    public Book getBookById(Long id) {
        return bookRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Book", id));
    }
    
    public void deleteBook(Long id) {
        if (!bookRepository.existsById(id)) {
            throw new ResourceNotFoundException("Book", id);
        }
        bookRepository.deleteById(id);
    }
}

// BookController.java
@RestController
@RequestMapping("/api/books")
public class BookController {
    private final BookService bookService;
    
    public BookController(BookService bookService) {
        this.bookService = bookService;
    }
    
    @PostMapping
    public ResponseEntity<Book> createBook(@Valid @RequestBody Book book) {
        Book created = bookService.createBook(book);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @GetMapping
    public List<Book> getAllBooks() {
        return bookService.getAllBooks();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        return ResponseEntity.ok(bookService.getBookById(id));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        bookService.deleteBook(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Exercise 2: Validation and Exception Handling

**Difficulty:** ⭐⭐ Medium

**Task:** Add validation and global exception handling to the book management system.

**Requirements:**
1. Create a `BookRequest` DTO with validation annotations
2. Add validation to all controller methods
3. Create custom exceptions: `BookNotFoundException`, `DuplicateISBNException`
4. Create a global exception handler
5. Return consistent error responses

**Solution:**
```java
// BookRequest.java
public record BookRequest(
    @NotBlank(message = "Title is required")
    @Size(min = 1, max = 200, message = "Title must be 1-200 characters")
    String title,
    
    @NotBlank(message = "Author is required")
    @Size(min = 1, max = 100, message = "Author must be 1-100 characters")
    String author,
    
    @NotBlank(message = "ISBN is required")
    @Pattern(regexp = "^(?:ISBN(?:-1[03])?:? )?(?=[0-9X]{10}$|(?=(?:[0-9]+[- ]){3})[- 0-9X]{13}$|97[89][0-9]{10}$|(?=(?:[0-9]+[- ]){4})[- 0-9]{17}$)(?:97[89][- ]?)?[0-9]{1,5}[- ]?[0-9]+[- ]?[0-9]+[- ]?[0-9X]$",
             message = "Invalid ISBN format")
    String isbn,
    
    @NotNull(message = "Published date is required")
    @Past(message = "Published date must be in the past")
    LocalDate publishedDate
) {}

// Custom exceptions
public class BookNotFoundException extends RuntimeException {
    public BookNotFoundException(Long id) {
        super("Book not found with id: " + id);
    }
}

public class DuplicateISBNException extends RuntimeException {
    public DuplicateISBNException(String isbn) {
        super("Book with ISBN " + isbn + " already exists");
    }
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleBookNotFound(BookNotFoundException ex) {
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.NOT_FOUND,
            ex.getMessage(),
            null
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(DuplicateISBNException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateISBN(DuplicateISBNException ex) {
        ErrorResponse error = ErrorResponse.of(
            HttpStatus.CONFLICT,
            ex.getMessage(),
            null
        );
        return new ResponseEntity<>(error, HttpStatus.CONFLICT);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                FieldError::getDefaultMessage
            ));
        return ResponseEntity.badRequest().body(ValidationErrorResponse.of(errors));
    }
}

// Updated BookService
@Service
public class BookService {
    private final BookRepository bookRepository;
    
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
    
    public Book createBook(BookRequest request) {
        if (bookRepository.existsByIsbn(request.isbn())) {
            throw new DuplicateISBNException(request.isbn());
        }
        
        Book book = new Book();
        book.setTitle(request.title());
        book.setAuthor(request.author());
        book.setIsbn(request.isbn());
        book.setPublishedDate(request.publishedDate());
        
        return bookRepository.save(book);
    }
    
    public Book getBookById(Long id) {
        return bookRepository.findById(id)
            .orElseThrow(() -> new BookNotFoundException(id));
    }
}
```

### Exercise 3: JWT Authentication

**Difficulty:** ⭐⭐⭐ Hard

**Task:** Implement JWT authentication for the book management system.

**Requirements:**
1. Create User entity with roles
2. Implement JWT token generation and validation
3. Create authentication endpoints (login, register)
4. Protect endpoints with JWT
5. Implement role-based access control

**Solution:**
```java
// User entity and repository
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(unique = true, nullable = false)
    private String email;
    @Column(nullable = false)
    private String password;
    @Column(nullable = false)
    private String name;
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
}

// JWT filter (see section 17 for complete implementation)
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    // Implementation from section 17
}

// Security configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/books/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}

// Authentication controller
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    private final AuthenticationService authService;
    
    public AuthController(AuthenticationService authService) {
        this.authService = authService;
    }
    
    @PostMapping("/register")
    public ResponseEntity<UserResponse> register(@Valid @RequestBody RegisterRequest request) {
        UserResponse user = authService.register(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
    
    @PostMapping("/login")
    public ResponseEntity<AuthenticationResponse> login(@Valid @RequestBody LoginRequest request) {
        AuthenticationResponse response = authService.authenticate(request);
        return ResponseEntity.ok(response);
    }
}
```

---

## Question Bank

### Section 1: IoC Container

1. **What is the Spring IoC Container and what is its primary responsibility?**
   
   **Answer:** The Spring IoC (Inversion of Control) Container is the core of Spring Boot that manages object creation, configuration, and lifecycle. Instead of developers creating objects manually with `new`, Spring creates and manages objects (beans), injects dependencies, and controls their lifecycle.

2. **What are the main stereotype annotations in Spring and when do you use each?**
   
   **Answer:**
   - `@Component`: Generic component for any Spring-managed bean
   - `@Service`: Business logic layer
   - `@Repository`: Data access layer (also provides exception translation)
   - `@Controller`: Web controller returning views (MVC)
   - `@RestController`: REST API controller (combines @Controller + @ResponseBody)
   - `@Configuration`: Configuration classes

3. **What is Dependency Injection and why is it important?**
   
   **Answer:** Dependency Injection is a design pattern where dependencies are provided to a class rather than the class creating them itself. It promotes loose coupling, makes testing easier, improves code maintainability, and follows the Dependency Inversion Principle.

### Section 2: Dependency Injection

4. **What are the three types of Dependency Injection in Spring? Which is recommended and why?**
   
   **Answer:** The three types are:
   - **Constructor Injection (Recommended):** Dependencies provided via constructor. Advantages: immutable dependencies, explicit requirements, easy testing, no hidden dependencies.
   - **Setter Injection:** Dependencies provided via setter methods. Use for optional dependencies.
   - **Field Injection:** Using @Autowired on fields. Not recommended due to hidden dependencies and testing difficulties.

5. **What is the difference between @Autowired and constructor injection?**
   
   **Answer:** `@Autowired` can be used on constructors, fields, or setter methods. Constructor injection (using `@Autowired` on constructor or single constructor without annotation in Spring 4.3+) is preferred because it makes dependencies explicit, allows immutable fields (final), and is easier to test without reflection.

6. **Can you explain the difference between tight coupling and loose coupling with examples?**
   
   **Answer:**
   - **Tight Coupling:** Class creates its own dependencies. Hard to change, hard to test.
     ```java
     public class BadService {
         private UserRepository repo = new UserRepository(); // Tightly coupled
     }
     ```
   - **Loose Coupling:** Dependencies are injected. Easy to change, easy to test.
     ```java
     public class GoodService {
         private final UserRepository repo;
         public GoodService(UserRepository repo) { // Loosely coupled
             this.repo = repo;
         }
     }
     ```

### Section 3: Auto Configuration

7. **What is Spring Boot Auto Configuration and how does it work?**
   
   **Answer:** Auto Configuration automatically configures Spring applications based on the libraries present in the classpath. Spring Boot scans for libraries, analyzes configuration, and automatically configures beans. For example, if H2 is on the classpath, it auto-configures an in-memory database.

8. **How can you disable a specific auto-configuration class?**
   
   **Answer:** You can disable auto-configuration using:
   ```java
   @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
   public class Application { }
   ```
   Or in application.properties:
   ```properties
   spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
   ```

9. **What is the difference between @Component, @Service, @Repository, and @Controller?**
   
   **Answer:** All are stereotype annotations that mark classes as Spring beans. The difference is semantic:
   - `@Component`: Generic stereotype
   - `@Service`: Indicates business logic layer
   - `@Repository`: Indicates data access layer and provides automatic exception translation (PersistenceException to DataAccessException)
   - `@Controller/@RestController`: Indicates web controller layer

### Section 4: Starter Dependencies

10. **What are Spring Boot Starter Dependencies and why are they useful?**
    
    **Answer:** Starter dependencies are curated sets of dependencies that work well together. For example, `spring-boot-starter-web` includes Spring MVC, Tomcat, Jackson, and validation. They eliminate dependency management complexity, ensure version compatibility, and speed up project setup.

11. **What is included in spring-boot-starter-web?**
    
    **Answer:** spring-boot-starter-web includes:
    - Spring MVC
    - Embedded Tomcat
    - Jackson for JSON processing
    - Validation support
    - Spring Boot starter (logging, core)
    - Spring Web

### Section 5: Project Structure

12. **What is the standard layered architecture in Spring Boot?**
    
    **Answer:** The standard layers are:
    - **Controller Layer:** Handles HTTP requests/responses
    - **Service Layer:** Contains business logic
    - **Repository Layer:** Data access and database operations
    - **Entity Layer:** Domain models representing database tables
    - **DTO Layer:** Data transfer objects for API contracts

13. **Why should you use DTOs instead of exposing entities directly?**
    
    **Answer:** DTOs provide:
    - Security: Hide sensitive fields (passwords, internal IDs)
    - Flexibility: Different API contracts without changing database schema
    - Versioning: Support multiple API versions
    - Performance: Fetch only needed fields
    - Validation: Separate validation rules for API vs database

### Section 6: Bean Lifecycle

14. **What is the Spring Bean lifecycle?**
    
    **Answer:** The lifecycle consists of:
    1. Instantiation (constructor called)
    2. Population of properties
    3. Dependency Injection
    4. Aware callbacks (if implementing Aware interfaces)
    5. BeanPostProcessor before initialization
    6. @PostConstruct / afterPropertiesSet()
    7. BeanPostProcessor after initialization
    8. Bean is ready to use
    9. @PreDestroy / destroy() when context closes
    10. Bean destruction

15. **What is the difference between @PostConstruct and @PreDestroy?**
    
    **Answer:**
    - `@PostConstruct`: Called after dependency injection, used for initialization logic (loading caches, establishing connections)
    - `@PreDestroy`: Called before bean destruction, used for cleanup (closing connections, releasing resources)

### Section 7: Configuration Properties

16. **What is the difference between @Value and @ConfigurationProperties?**
    
    **Answer:**
    - `@Value`: Injects individual property values. Good for few properties.
     ```java
     @Value("${app.name}")
     private String appName;
     ```
    - `@ConfigurationProperties`: Binds entire configuration objects. Better for complex configurations, supports type safety, validation, and nested properties.

17. **How do you externalize configuration in Spring Boot?**
    
    **Answer:** Spring Boot supports multiple external configuration sources (in order of precedence):
    1. Command line arguments
    2. OS environment variables
    3. Java System properties
    4. application.properties/yml outside JAR
    5. application.properties/yml inside JAR
    6. @PropertySource annotations
    7. Default properties

### Section 8: REST Controllers

18. **What is the difference between @Controller and @RestController?**
    
    **Answer:**
    - `@Controller`: Used for MVC applications returning views (HTML). Requires @ResponseBody on methods returning JSON/XML.
    - `@RestController`: Combines @Controller + @ResponseBody. All methods return response body (JSON/XML) directly. Used for REST APIs.

19. **What HTTP methods are used in REST and what are they for?**
    
    **Answer:**
    - GET: Retrieve resources (safe, idempotent)
    - POST: Create new resources (not idempotent)
    - PUT: Update/replace entire resource (idempotent)
    - PATCH: Partial update (not idempotent)
    - DELETE: Remove resources (idempotent)

20. **When should you return different HTTP status codes?**
    
    **Answer:**
    - 200 OK: Successful GET, PUT, PATCH
    - 201 Created: Successful POST
    - 204 No Content: Successful DELETE
    - 400 Bad Request: Validation errors
    - 401 Unauthorized: Missing/invalid authentication
    - 403 Forbidden: Insufficient permissions
    - 404 Not Found: Resource doesn't exist
    - 500 Internal Server Error: Unexpected errors

### Section 9: Request Mapping

21. **What is the difference between @PathVariable and @RequestParam?**
    
    **Answer:**
    - `@PathVariable`: Extracts values from URI path. Used for resource identifiers.
      ```java
      @GetMapping("/users/{id}")  // /users/123
      public User getUser(@PathVariable Long id)
      ```
    - `@RequestParam`: Extvalues from query parameters. Used for filtering, sorting, pagination.
      ```java
      @GetMapping("/users")  // /users?name=John&age=30
      public List<User> search(@RequestParam String name, @RequestParam Integer age)
      ```

22. **What is the N+1 query problem and how do you solve it?**
    
    **Answer:** The N+1 problem occurs when you fetch a list of entities (1 query) and then access a lazy-loaded relationship for each entity (N queries). Solutions:
    - Use JOIN FETCH in queries
    - Use @EntityGraph
    - Use batch fetching
    - Use DTO projections

### Section 10: Validation

23. **How do you implement validation in Spring Boot?**
    
    **Answer:** Use Jakarta Bean Validation:
    1. Add `spring-boot-starter-validation` dependency
    2. Add `@Valid` annotation to request body
    3. Use validation annotations on DTOs (@NotBlank, @Email, @Min, etc.)
    4. Handle MethodArgumentNotValidException in global exception handler

24. **What are some common validation annotations?**
    
    **Answer:**
    - `@NotBlank/@NotNull/@NotEmpty`: Not null/empty
    - `@Size(min, max)`: String/collection size
    - `@Min/@Max/@DecimalMin/@DecimalMax`: Numeric ranges
    - `@Email`: Email format
    - `@Pattern(regexp)`: Regular expression
    - `@Past/@Future/@PastOrPresent/@FutureOrPresent`: Date validation
    - `@Valid`: Validate nested objects

### Section 11: Global Exception Handling

25. **What is @RestControllerAdvice and how does it work?**
    
    **Answer:** `@RestControllerAdvice` is a combination of `@ControllerAdvice` and `@ResponseBody`. It allows you to handle exceptions globally across all controllers. Methods annotated with `@ExceptionHandler` catch specific exception types and return consistent error responses.

26. **Why is centralized exception handling important?**
    
    **Answer:** Centralized exception handling:
    - Eliminates repetitive try-catch blocks
    - Provides consistent error responses
    - Separates error handling from business logic
    - Makes code cleaner and more maintainable
    - Allows proper logging and monitoring

### Section 12: Spring Data JPA

27. **What is Spring Data JPA and what are its benefits?**
    
    **Answer:** Spring Data JPA simplifies database access by:
    - Automatically generating repository implementations
    - Providing derived query methods from method names
    - Supporting pagination and sorting
    - Reducing boilerplate code
    - Supporting custom queries with @Query

28. **What is the difference between findById and getOne in Spring Data JPA?**
    
    **Answer:**
    - `findById(id)`: Returns Optional, executes query immediately, returns null if not found
    - `getOne(id)`: Returns entity reference, lazy loading, throws EntityNotFoundException if not found (deprecated in newer versions, use getReferenceById)

### Section 13: Entity Relationships

29. **What is the difference between FetchType.EAGER and FetchType.LAZY?**
    
    **Answer:**
    - **EAGER:** Loads the relationship immediately with the parent entity. Always loaded even if not used. Can cause performance issues.
    - **LAZY:** Loads the relationship only when accessed. Better performance but requires active session or initialization.

30. **What is the N+1 query problem?**
    
    **Answer:** The N+1 problem occurs when:
    - 1 query fetches a list of entities
    - N queries fetch related entities for each entity in the list
    - Total: N+1 queries instead of 1
    - Solution: Use JOIN FETCH, @EntityGraph, or batch fetching

### Section 14: Transactions

31. **What are the ACID properties of transactions?**
    
    **Answer:**
    - **Atomicity:** All operations succeed or all fail
    - **Consistency:** Data remains valid after transaction
    - **Isolation:** Concurrent transactions don't interfere
    - **Durability:** Committed changes are permanent

32. **What is the default propagation behavior of @Transactional?**
    
    **Answer:** The default propagation is `REQUIRED`, which means:
    - If a transaction exists, join it
    - If no transaction exists, create a new one

33. **When does a transaction roll back?**
    
    **Answer:** By default, transactions roll back on unchecked exceptions (RuntimeException and Error). For checked exceptions, you need to specify `rollbackFor`:
    ```java
    @Transactional(rollbackFor = Exception.class)
    ```

### Section 15: Spring Security

34. **What is the purpose of Spring Security?**
    
    **Answer:** Spring Security provides:
    - Authentication (verifying identity)
    - Authorization (controlling access)
    - Password encryption
    - Session management
    - Protection against common attacks (CSRF, XSS)
    - Integration with authentication providers (LDAP, OAuth2, JWT)

35. **What password encoder should you use?**
    
    **Answer:** Use `BCryptPasswordEncoder` (or Argon2 for stronger security). Never store passwords in plain text or use weak algorithms like MD5 or SHA-1.

### Section 16: Request Flow

36. **Can you describe the complete flow of a request in Spring Boot?**
    
    **Answer:** 
    1. Client sends HTTP request
    2. Security filters authenticate/authorize
    3. DispatcherServlet receives request
    4. Handler mapping finds appropriate controller method
    5. Validation occurs (if @Valid present)
    6. Controller processes request
    7. Service layer executes business logic
    8. Repository accesses database
    9. Response returns through layers
    10. Exception handler catches any errors
    11. HTTP response sent to client

### Section 17: JWT Authentication

37. **What is JWT and what are its advantages?**
    
    **Answer:** JWT (JSON Web Token) is a compact, URL-safe token for securely transmitting information. Advantages:
    - Stateless (no server-side session storage)
    - Scalable across multiple servers
    - Works well with mobile apps
    - Can contain user claims
    - Self-contained

38. **What should you store in a JWT payload?**
    
    **Answer:** Store non-sensitive claims:
    - Subject (user ID)
    - Username/email
    - Roles/permissions
    - Expiration time
    - Issued at time
    
    **Never store:** Passwords, sensitive personal data, large amounts of data

### Section 18: Profiles

39. **What are Spring Boot profiles and when would you use them?**
    
    **Answer:** Profiles allow different configurations for different environments (dev, test, prod). Use them to:
    - Use different databases per environment
    - Enable/disable features
    - Configure different logging levels
    - Use different API endpoints

40. **How do you activate a specific profile?**
    
    **Answer:**
    ```bash
    # Command line
    java -jar app.jar --spring.profiles.active=prod
    
    # Environment variable
    SPRING_PROFILES_ACTIVE=prod java -jar app.jar
    
    # application.properties
    spring.profiles.active=prod
    ```

### Section 19: Logging and Monitoring

41. **What is the purpose of Spring Boot Actuator?**
    
    **Answer:** Actuator provides production-ready monitoring endpoints for:
    - Health checks
    - Metrics
    - Application info
    - Environment properties
    - Thread dumps
    - Heap dumps

42. **What logging framework does Spring Boot use by default?**
    
    **Answer:** Spring Boot uses SLF4J with Logback by default. It supports Log4j2, JUL, and other frameworks through bridges.

### Section 20: Testing

43. **What are the different types of tests in Spring Boot?**
    
    **Answer:**
    - **Unit Tests:** Test individual classes in isolation (JUnit + Mockito)
    - **Integration Tests:** Test multiple components together (@SpringBootTest)
    - **Slice Tests:** Test specific layers (@WebMvcTest, @DataJpaTest)
    - **API Tests:** Test REST endpoints (MockMvc, TestRestTemplate)
    - **End-to-End Tests:** Test complete workflows

44. **What is the difference between @MockBean and @Mock?**
    
    **Answer:**
    - `@Mock` (Mockito): Creates a mock but doesn't add it to Spring context
    - `@MockBean` (Spring Test): Creates a mock and replaces any existing bean in the Spring context

### Section 21: Production Readiness

45. **What are the key aspects of production-ready applications?**
    
    **Answer:**
    - Security (HTTPS, authentication, authorization)
    - Performance (caching, optimization, monitoring)
    - Scalability (stateless design, load balancing)
    - Observability (logging, metrics, tracing)
    - Reliability (error handling, retries, circuit breakers)
    - Maintainability (clean code, documentation)

46. **What is the purpose of health checks in production?**
    
    **Answer:** Health checks:
    - Monitor application status
    - Check dependencies (database, external services)
    - Enable load balancers to route traffic
    - Trigger alerts when unhealthy
    - Support Kubernetes liveness/readiness probes

---

## Summary and Key Takeaways

### Core Concepts Mastered

Throughout this comprehensive guide, you've learned 20 essential Spring Boot concepts:

1. ✅ **IoC Container** - The foundation of Spring Boot that manages object creation and lifecycle
2. ✅ **Dependency Injection** - Reduces coupling and improves testability
3. ✅ **Auto Configuration** - Spring Boot's superpower that eliminates boilerplate configuration
4. ✅ **Starter Dependencies** - Simplifies dependency management
5. ✅ **Project Structure** - Organizes applications for maintainability
6. ✅ **Bean Lifecycle** - Understanding what happens behind the scenes
7. ✅ **Configuration Properties** - Makes applications flexible and environment-agnostic
8. ✅ **REST Controllers** - The entry point for APIs
9. ✅ **Request Mapping** - Connects URLs to Java methods
10. ✅ **Validation** - Protects against invalid data
11. ✅ **Global Exception Handling** - Manages errors gracefully
12. ✅ **Spring Data JPA** - Simplifies database access
13. ✅ **Entity Relationships** - Models real-world data
14. ✅ **Transactions** - Ensures data consistency
15. ✅ **Spring Security** - Protects applications
16. ✅ **Request Flow** - Complete lifecycle understanding
17. ✅ **JWT Authentication** - Modern stateless security
18. ✅ **Profiles** - Environment-specific configuration
19. ✅ **Logging and Monitoring** - Production observability
20. ✅ **Testing** - Ensures software quality
21. ✅ **Production Readiness** - Builds reliable applications

### Key Insights

> **💡 The Spring Boot Philosophy**
> 
> Spring Boot's greatest strength is not just convenience—it's the powerful ecosystem built around proven software engineering principles. Understanding these 20 concepts gives you the knowledge to build applications that are:
> - **Maintainable:** Clean architecture, proper separation of concerns
> - **Scalable:** Stateless design, efficient database access
> - **Secure:** Proper authentication, authorization, and validation
> - **Reliable:** Transaction management, error handling, testing
> - **Observable:** Logging, monitoring, health checks

### What Separates Beginners from Experts

| Beginners | Experts |
|-----------|---------|
| Know which annotation to use | Understand why it exists |
| Copy code from tutorials | Understand how it works internally |
| Build simple CRUD apps | Design complex enterprise systems |
| Struggle with debugging | Troubleshoot efficiently |
| Write code that works | Write production-ready code |
| Focus on functionality | Consider security, performance, scalability |

### Next Steps

To continue your Spring Boot journey:

1. **Build Projects:** Apply these concepts to real projects
2. **Read Source Code:** Explore Spring Boot source code on GitHub
3. **Learn Advanced Topics:**
   - Spring Cloud (microservices)
   - Spring Batch (batch processing)
   - Spring Integration (enterprise integration)
   - Spring WebFlux (reactive programming)
4. **Contribute:** Contribute to open-source Spring projects
5. **Stay Updated:** Follow Spring blog and release notes

### Final Thoughts

Becoming an excellent Spring Boot developer isn't about memorizing every annotation or API. Frameworks evolve, libraries change, and new technologies appear every year. What remains constant is a solid understanding of:

- **Architecture:** How components work together
- **Security:** Protecting applications and data
- **Data Management:** Efficient database operations
- **Testing:** Ensuring software quality
- **Clean Design:** Writing maintainable code

Master these concepts, and you'll not only write better Spring Boot applications—you'll develop the mindset of a professional software engineer who can design systems that are reliable, scalable, and ready for the challenges of the real world.

---

## Further Reading and Resources

### Official Documentation
- [Spring Boot Official Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Framework Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring Data JPA Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Spring Security Documentation](https://docs.spring.io/spring-security/docs/current/reference/html/)

### Books
- "Spring Boot in Action" by Craig Walls
- "Spring in Action" by Craig Walls
- "Clean Architecture" by Robert C. Martin
- "Domain-Driven Design" by Eric Evans
- "Building Microservices" by Sam Newman

### Online Courses
- [Spring Academy](https://spring.academy/)
- [Udemy - Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)
- [Pluralsight - Spring Boot Path](https://www.pluralsight.com/paths/spring-boot)

### Community Resources
- [Spring Blog](https://spring.io/blog)
- [Baeldung](https://www.baeldung.com/)
- [Spring Tips](https://www.youtube.com/c/SpringTips)
- [r/springboot](https://www.reddit.com/r/springboot/)

### Tools
- [Spring Initializr](https://start.spring.io/)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/actuator/html/)
- [Spring Boot DevTools](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.devtools)
- [Docker](https://www.docker.com/)
- [Kubernetes](https://kubernetes.io/)

### Practice Platforms
- [LeetCode](https://leetcode.com/) (Algorithm practice)
- [HackerRank](https://www.hackerrank.com/)
- [Spring Boot Projects](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-samples)

---

**Congratulations!** You've completed a comprehensive deep-dive into Spring Boot. You now have the knowledge to build professional, production-ready Spring Boot applications. Keep practicing, keep learning, and build amazing things! 🚀

---

**Last Updated:** July 2, 2026  
**Version:** 1.0  
**Author:** Gopi C K  
**Tutorial Created Using:** Knowledge Base Tutorial Preferences
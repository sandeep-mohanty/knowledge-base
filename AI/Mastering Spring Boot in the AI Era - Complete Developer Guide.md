# Mastering Spring Boot in the AI Era: The Developer's Complete Guide to Staying Relevant and Effective

**Author:** Dylan Smith (Original Content) | Enhanced by AI Assistant  
**Reading Time:** 25-30 minutes  
**Difficulty Level:** Intermediate  
**Last Updated:** January 2026  
**Spring Boot Version:** 3.5.x  
**Java Version:** 21

---

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [The AI-Spring Boot Landscape](#part-1-the-ai-spring-boot-landscape)
5. [Why You Still Need to Learn Spring Boot Features](#part-2-why-you-still-need-to-learn-spring-boot-features)
6. [Building Your Capability Map](#part-3-building-your-capability-map)
7. [Learning Strategies for the AI Era](#part-4-learning-strategies-for-the-ai-era)
8. [Hands-On Implementation Guide](#part-5-hands-on-implementation-guide)
9. [Real-World Use Cases](#real-world-use-cases)
10. [Common Pitfalls & Troubleshooting](#common-pitfalls--troubleshooting)
11. [Best Practices](#best-practices)
12. [Anti-Patterns](#anti-patterns)
13. [Performance Considerations](#performance-considerations)
14. [Security Considerations](#security-considerations)
15. [Testing Strategies](#testing-strategies)
16. [Migration Guide](#migration-guide)
17. [Summary & Key Takeaways](#summary--key-takeaways)
18. [Further Reading & Resources](#further-reading--resources)
19. [Practice Exercises](#practice-exercises)
20. [Question Bank](#question-bank)
21. [Test Your Understanding](#test-your-understanding)
22. [Common Interview Questions](#common-interview-questions)

---

## Introduction

### The AI Revolution in Software Development

We're living through a fundamental shift in how software is built. AI coding assistants like GitHub Copilot, Claude, and others have transformed from novelty tools to essential parts of every developer's workflow. But this raises a critical question that every Spring Boot developer is asking:

> **"With AI writing code for me, do I still need to learn Spring Boot's new features?"**

### Short Answer: Yes, But How You Learn Changes Completely

Back in the day, you learned Spring Boot so you could write it yourself. Now you learn it so you can tell AI what to write — and make sure it's written correctly, efficiently, and using modern approaches.

AI can crank out code, fill in syntax, and scaffold templates for you, but it **can't make technical judgments for you**. And every new release of Spring Boot exists precisely to inform those judgments — what kind of code to write, and why you'd write it that way.

### Why This Matters Now

Spring Boot 3.5.x brings transformative changes built on the foundation of Java 21:
- **Virtual Threads** (Project Loom) - now stable and production-ready
- **RestClient** - the modern, fluent HTTP client
- **Docker Compose support** - simplified local development
- **AOT native images** with GraalVM - sub-second startup times
- **Full Jakarta EE migration** (javax → jakarta namespace)
- **Structured Concurrency** (Java 21) - better error handling for concurrent tasks
- **Scoped Values** (Java 21) - modern alternative to ThreadLocal
- **Enhanced observability** and metrics
- **Pattern matching** and other Java 21 language features

If you don't have these capabilities mapped in your mental model, AI will always default to the safest, most conservative, and often outdated approach. Over time, technical debt piles up — while everyone else is enjoying sub-second startup times with GraalVM and handling 10,000 concurrent requests with virtual threads, you're still fighting slow boot times and thread pool exhaustion.

### What You'll Learn

In this comprehensive guide, you'll discover:
- ✅ Why AI alone isn't enough and where it fails
- ✅ How to build a "capability map" of Spring Boot features
- ✅ Learning strategies tailored to your experience level
- ✅ Practical workflows for AI-assisted development
- ✅ Real-world case studies and examples
- ✅ Common pitfalls and how to avoid them
- ✅ Best practices and anti-patterns
- ✅ Hands-on exercises to reinforce learning

---

## Prerequisites

### Required Knowledge
- ✅ **Java Fundamentals:** Core Java concepts, OOP principles, Java 8+ features (Streams, lambdas)
- ✅ **Spring Boot Basics:** Understanding of Spring Boot 2.x or 3.x fundamentals
- ✅ **Basic Maven/Gradle:** Dependency management and build tools
- ✅ **REST API Concepts:** HTTP methods, status codes, request/response patterns
- ✅ **Database Basics:** JDBC, JPA/Hibernate fundamentals
- ✅ **Version Control:** Git basics

### Recommended Knowledge
- 💡 **Spring Boot 2.x/3.x Experience:** 6+ months of hands-on development
- 💡 **Docker Basics:** Container concepts and Docker Compose
- 💡 **Microservices Architecture:** Basic understanding of distributed systems
- 💡 **AI Coding Assistants:** Familiarity with GitHub Copilot, Claude, or similar tools

### Tools You'll Need
- **JDK 21** (required for Spring Boot 3.5.x and Virtual Threads)
- **Spring Boot 3.5.x** (latest stable version)
- **IDE:** IntelliJ IDEA 2024+, Eclipse, or VS Code with Java extensions
- **AI Assistant:** GitHub Copilot, Claude, or similar (optional but helpful)
- **Docker Desktop:** For container-related examples

---

## Learning Objectives

By the end of this tutorial, you will be able to:

### Knowledge Objectives
- ✅ Explain why learning Spring Boot features remains critical in the AI era
- ✅ Identify the key limitations of AI-generated Spring Boot code
- ✅ List the major features introduced in Spring Boot 3.5.x
- ✅ Recognize common AI-generated code issues and breaking changes
- ✅ Understand the concept of a "capability map" and how to build one

### Skill Objectives
- ✅ Effectively prompt AI assistants for modern Spring Boot 3.5.x code
- ✅ Review and audit AI-generated code for correctness
- ✅ Build a personal capability map of Spring Boot features
- ✅ Apply learning strategies based on your experience level
- ✅ Debug production issues with AI assistance

### Application Objectives
- ✅ Implement production-ready Spring Boot 3.5.x applications using AI assistance
- ✅ Migrate Spring Boot 2.x/3.0 applications to 3.5.x with AI help
- ✅ Make informed architectural decisions about Spring Boot features
- ✅ Create effective workflows combining human judgment and AI capabilities
- ✅ Avoid common pitfalls when using AI for Spring Boot development

---

## Part 1: The AI-Spring Boot Landscape

### How AI Coding Assistants Actually Work

Before we dive into why you still need to learn, let's understand what AI assistants actually do:

```mermaid
graph TD
    A[User Prompt] --> B[AI Model]
    B --> C{Training Data Analysis}
    C --> D[Pattern Matching]
    D --> E[Code Generation]
    E --> F[AI-Generated Code]
    
    G[GitHub Repositories] --> C
    H[Stack Overflow] --> C
    I[Documentation] --> C
    J[Blog Posts] --> C
    
    style A fill:#e1f5ff
    style F fill:#fff4e1
    style B fill:#f0e1ff
```

**How AI Code Generation Works:**

1. **Pattern Recognition:** AI models are trained on millions of code repositories
2. **Context Understanding:** They analyze your prompt and existing code context
3. **Probability-Based Generation:** They predict the most likely next tokens
4. **No Real Understanding:** AI doesn't "know" Spring Boot - it recognizes patterns

> ⚠️ **Critical Insight:** AI assistants are essentially sophisticated pattern matchers. They don't understand concepts, best practices, or why certain approaches are better. They predict what code should come next based on statistical patterns in their training data.

### What AI Can Do Well

✅ **Boilerplate Generation:** Controllers, services, entities, DTOs  
✅ **Syntax Completion:** Method signatures, imports, annotations  
✅ **Common Patterns:** CRUD operations, REST endpoints, basic configurations  
✅ **Documentation:** Generating Javadoc, comments, README files  
✅ **Test Scaffolding:** Basic unit test structures  
✅ **Refactoring:** Simple code improvements and renames  

### What AI Cannot Do

❌ **Make Technical Judgments:** Choose between Virtual Threads vs CompletableFuture  
❌ **Understand Your Context:** Know your team's JDK version, deployment environment  
❌ **Stay Current:** Training data cutoff means it doesn't know latest features  
❌ **Guarantee Correctness:** Can confidently generate code with version mismatches  
❌ **Architectural Decisions:** Can't weigh trade-offs for your specific use case  
❌ **Debug Production Issues:** Can suggest approaches but can't find root causes  

### The New Developer Workflow

```mermaid
graph LR
    A[You: Define Requirements] --> B[You: Choose Tech Stack]
    B --> C[AI: Generate Code]
    C --> D[You: Review & Audit]
    D --> E{Issues Found?}
    E -->|Yes| F[You: Fix & Refine]
    F --> C
    E -->|No| G[You: Test & Validate]
    G --> H[Production]
    
    style A fill:#e1f5ff
    style D fill:#fff4e1
    style H fill:#e8f5e9
```

**The Modern Development Loop:**

```
1. YOU define WHAT to build and WHY
   ↓
2. YOU choose WHICH features/approaches to use
   ↓
3. AI generates the HOW (implementation)
   ↓
4. YOU review for correctness and best practices
   ↓
5. YOU test and validate
   ↓
6. Deploy to production
```

> 💡 **Key Mindset Shift:** You're no longer a "code writer" - you're a **technical architect and code reviewer**. AI is your implementation assistant, not your replacement.

---

## Part 2: Why You Still Need to Learn Spring Boot Features

### 1. AI Won't Use Features You Don't Know Exist

#### The Problem

AI works on a simple principle: you ask for something, it gives you a solution. It won't proactively upgrade your tech stack on its own.

#### Real-World Example

I was reviewing a friend's project recently. He asked AI to build a high-concurrency endpoint, and AI dutifully generated code using `CompletableFuture` and traditional thread pools. The code worked, but it was unnecessarily complex and didn't leverage modern Java features.

**The issue:** He didn't know Virtual Threads existed in Spring Boot 3.5.x, so he never asked for them. AI gave him a working solution, but not the *best* solution.

#### What Spring Boot 3.5.x Offers

```mermaid
mindmap
  root((Spring Boot 3.5.x))
    Concurrency
      Virtual Threads
        When: High-concurrency I/O-bound tasks
        Benefit: 10-100x more concurrent requests
        Gotcha: ThreadLocal doesn't work as expected
      Structured Concurrency
        When: Multiple concurrent tasks with error handling
        Benefit: Cleaner error handling
      Scoped Values
        When: Context propagation
        Benefit: Modern ThreadLocal alternative
    HTTP Clients
      RestClient
        When: Simple HTTP calls
        Benefit: Fluent API, better than RestTemplate
      WebClient
        When: Reactive streaming
        Benefit: Non-blocking backpressure
    Cloud Native
      Docker Compose support
        When: Local development with dependencies
        Benefit: Auto-start containers
      AOT compilation
        When: Production deployment
        Benefit: Sub-second startup
      GraalVM native images
        When: Serverless, microservices
        Benefit: Minimal memory footprint
    Jakarta EE
      jakarta namespace
        When: All JPA operations
        Benefit: Modern namespace
        Gotcha: javax.* → jakarta.* migration
    Observability
      Micrometer metrics
        When: Production monitoring
        Benefit: Better insights
      Distributed tracing
        When: Microservices
        Benefit: Request tracking
```

**Major Spring Boot 3.5.x Features You Should Know:**

| Feature | Spring Boot Version | Benefit | AI's Likely Approach Without Your Knowledge |
|---------|-------------------|---------|-------------------------------------------|
| Virtual Threads | 3.2+ (stable in 3.5.x) | 10-100x more concurrent requests | Traditional thread pools (CompletableFuture) |
| RestClient | 3.2+ | Fluent, modern HTTP client | RestTemplate (deprecated) or WebClient |
| Docker Compose | 3.1+ | Auto-start containers for local dev | Manual container management |
| AOT Native Images | 3.0+ (improved in 3.5.x) | Sub-second startup, low memory | JVM-only deployment |
| Jakarta EE | 3.0+ | Modern namespace | javax.* (doesn't compile in 3.5.x) |
| Structured Concurrency | Java 21 | Better error handling for concurrent tasks | Manual thread management |
| Scoped Values | Java 21 | Modern alternative to ThreadLocal | ThreadLocal (problematic with virtual threads) |

#### The Compound Effect

Over time, not knowing new features creates **technical debt**:

```
Year 1: You're using RestTemplate (fine, works)
Year 2: Everyone else migrated to RestClient (cleaner, more maintainable)
Year 3: Your codebase is outdated, harder to hire for
Year 4: Migration becomes painful and expensive
```

> ⚠️ **Warning:** AI will always default to the safest, most common patterns in its training data. If those patterns are from 2022, you're getting 2022 code in 2026.

### 2. AI Confidently Makes Stuff Up — You Need to Audit It

#### The Hallucination Problem

Anyone who's worked with AI-generated Spring Boot code knows this pain point all too well. Version mismatches, deprecated APIs, and wrong config properties are par for the course.

#### Case Study 1: The Jakarta Migration Disaster

**Scenario:** Asked AI for a file upload endpoint in Spring Boot 3.5.x

**AI Generated:**
```java
// ❌ WRONG - Spring Boot 3.5.x doesn't use javax.*
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.Part;

@PostMapping("/upload")
public ResponseEntity<String> uploadFile(HttpServletRequest request) {
    Part filePart = request.getPart("file");
    // ... processing logic
}
```

**Why This Fails:**
- Spring Boot 3.0+ migrated everything to `jakarta.*` namespace
- `javax.servlet.http.HttpServletRequest` doesn't exist in Spring Boot 3.5.x
- Code won't even compile

**Correct Version (Spring Boot 3.5.x):**
```java
// ✅ CORRECT - Uses jakarta.* namespace
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.Part;

@PostMapping("/upload")
public ResponseEntity<String> uploadFile(HttpServletRequest request) {
    Part filePart = request.getPart("file");
    // ... processing logic
}
```

**Time Lost Debugging:** 2-3 hours (if you don't know about the Jakarta migration)

#### Case Study 2: The Deprecated Configuration Class

**Scenario:** Asked AI how to configure CORS in Spring Boot 3.5.x

**AI Generated:**
```java
// ❌ WRONG - Deprecated since Spring 5
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class CorsConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}
```

**Why This Is Wrong:**
- `WebMvcConfigurerAdapter` was deprecated in Spring 5
- Modern approach: Simply implement `WebMvcConfigurer` interface
- No need for the adapter class anymore

**Correct Version (Spring Boot 3.5.x):**
```java
// ✅ CORRECT - Modern approach
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}
```

**Time Lost Debugging:** 1-2 hours (IDE warnings, but code still works)

#### Case Study 3: Missing Virtual Threads

**Scenario:** Asked AI to build a high-concurrency endpoint in Spring Boot 3.5.x

**AI Generated:**
```java
// ❌ Suboptimal - Uses traditional thread pool
@Service
public class UserService {
    
    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(20);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(200);
        executor.initialize();
        return executor;
    }
    
    @Async
    public CompletableFuture<User> fetchUser(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            return userRepository.findById(userId);
        }, taskExecutor());
    }
}
```

**Why This Is Suboptimal:**
- Uses traditional thread pool with limited concurrency
- Complex configuration required
- Thread pool exhaustion under high load
- Misses the benefits of Virtual Threads

**Correct Version (Spring Boot 3.5.x with Java 21):**
```java
// ✅ Optimal - Uses Virtual Threads
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public AsyncTaskExecutor taskExecutor() {
        return ThreadPerTaskTaskExecutor.builder()
            .threadNamePrefix("virtual-")
            .virtualThreads(true)  // Enable virtual threads
            .build();
    }
}

@Service
public class UserService {
    
    // Automatically uses virtual threads
    @Async
    public CompletableFuture<User> fetchUser(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            // This runs on a virtual thread
            return userRepository.findById(userId);
        });
    }
}
```

**Benefits:**
- 10-100x more concurrent requests
- No thread pool configuration needed
- Automatic scaling
- Better resource utilization

**Time Lost:** Hours of debugging thread pool exhaustion, plus ongoing performance issues

#### The Efficiency Multiplier

```
Developer who knows breaking changes and new features:
- Spots AI errors in 30 seconds
- Fixes them immediately
- Uses modern features automatically
- Moves on to next task

Developer who doesn't know breaking changes:
- Spends 2-3 hours debugging
- Searches Stack Overflow
- Tries multiple "fixes"
- Misses modern features
- Accumulates technical debt

Efficiency difference: 10x
```

> 💡 **Pro Tip:** Create a "Breaking Changes Checklist" for each major Spring Boot version. Review it quarterly. This single habit will save you hundreds of hours over your career.

### 3. Architecture Decisions and Production Debugging — AI Can't Replace You

#### The Architecture Decision Problem

There are things AI can offer suggestions on, but the final call is yours.

**Example Decision:** Should your team upgrade from Spring Boot 3.0 to 3.5.x?

**AI's Response:** Generic checklist with pros and cons

**What AI Doesn't Know:**
- Your company is still on JDK 17 (Spring Boot 3.5.x works best with JDK 21)
- Your team has 3 junior developers who just learned Spring Boot 3.0
- Your production environment uses a specific monitoring tool that needs updating
- Your migration timeline is 3 months, not 3 years
- You need Virtual Threads for your high-concurrency use case

**Your Decision Framework:**
```mermaid
graph TD
    A[Upgrade Decision] --> B{Current JDK Version?}
    B -->|JDK 17| C{Need Virtual Threads?}
    B -->|JDK 21| D[Upgrade to 3.5.x]
    C -->|Yes| E[Upgrade JDK to 21 + Spring Boot 3.5.x]
    C -->|No| F[Stay on 3.x or upgrade to 3.5.x]
    
    E --> G{Team Experience?}
    G -->|Junior Heavy| H[Gradual Migration - 6 months]
    G -->|Senior Heavy| I[Direct Migration - 3 months]
    
    style A fill:#e1f5ff
    style D fill:#e8f5e9
    style H fill:#fff4e1
```

#### Production Debugging: Where AI Falls Short

When production breaks, AI can suggest debugging angles, but finding the root cause depends on your understanding.

**Common Production Issues in Spring Boot 3.5.x with Java 21:**

1. **Virtual Thread Starvation**
   ```
   Issue: Application becomes unresponsive under load
   ```
   - AI might suggest increasing thread pool size
   - You need to understand virtual thread scheduling and platform thread carrier sizing

2. **Scoped Values vs ThreadLocal Issues**
   ```java
   // Problem: Mixing ThreadLocal and Scoped Values incorrectly
   private static final ThreadLocal<String> context = new ThreadLocal<>();
   private static final ScopedValue<String> SCOPED_CONTEXT = ScopedValue.newInstance();
   
   // Virtual threads don't propagate ThreadLocal, but ScopedValues work correctly
   ```
   - AI might suggest using InheritableThreadLocal
   - You need to understand the differences and when to use each

3. **Native Image Build Failures with AOT**
   ```
   Error: Class not found during native image build
   ```
   - AI might suggest adding reflection configuration
   - You need to understand AOT compilation and native image constraints

4. **Structured Concurrency Error Handling**
   ```java
   // Problem: Not handling errors correctly in structured tasks
   try (var scope = StructuredTaskScope.shutdownOnFailure()) {
       // Multiple tasks
       scope.join(); // What if one fails?
   }
   ```
   - AI might suggest basic try-catch
   - You need to understand structured concurrency error propagation

#### The Debugging Workflow

```mermaid
graph TD
    A[Production Issue] --> B[AI: Suggest Debugging Angles]
    B --> C[You: Understand Root Cause]
    C --> D[You: Implement Fix]
    D --> E[AI: Generate Fix Code]
    E --> F[You: Review & Test]
    F --> G[Deploy]
    
    style A fill:#ffebee
    style C fill:#fff4e1
    style G fill:#e8f5e9
```

> ⚠️ **Critical Point:** AI can help you debug faster, but it can't replace your understanding of how Spring Boot works under the hood. The more you understand, the faster you can identify the real problem vs. AI's red herrings.

### 4. Learning New Features Means Learning Design Thinking — That's Your Moat

#### Beyond Syntax: The Strategic Layer

Every major Spring Boot release tracks broader industry trends:
- **Cloud Native:** Containerization, orchestration, service mesh
- **Serverless:** Function-as-a-Service, cold start optimization
- **GraalVM:** Native image compilation, memory optimization
- **Observability:** Metrics, tracing, logging standards
- **Reactive Programming:** Non-blocking I/O, backpressure management
- **Virtual Threads:** Simplified concurrency model

Following new features is really about understanding:
- What new problems the Spring ecosystem is solving
- What the latest best practices look like
- Where the industry is heading

#### The Developer Gap

Two developers with three years of experience:

**Developer A (Can only do CRUD):**
- Uses Spring Boot 3.0 patterns
- Doesn't understand why Virtual Threads matter
- Doesn't know about Structured Concurrency
- Copies AI-generated code without understanding
- Struggles with production issues

**Developer B (Understands modern Spring Boot 3.5.x):**
- Uses Virtual Threads and Scoped Values appropriately
- Can explain how Structured Concurrency improves error handling
- Can explain AOT compilation and GraalVM native images
- Reviews AI code critically
- Solves production issues quickly

**That's the gap.** And it's only going to widen as AI becomes more capable at implementation.

#### Building Your Technical Vision

```mermaid
graph TD
    A[Learn New Features] --> B[Understand Industry Trends]
    B --> C[Develop Design Thinking]
    C --> D[Make Better Architectural Decisions]
    D --> E[Write Better Code]
    E --> F[Solve Harder Problems]
    F --> A
    
    style A fill:#e1f5ff
    style F fill:#e8f5e9
```

> 💡 **Key Insight:** Typing speed doesn't matter as much anymore. Making the right choices and thinking deeply is what sets people apart. Learning Spring Boot's new features isn't about memorizing APIs — it's about developing the technical vision to know *when* and *why* to use them.

---

## Part 3: Building Your Capability Map

### What Is a Capability Map?

A **capability map** is a mental (or physical) index of what technologies and features you know exist, what problems they solve, and when to use them.

**Analogy:** Think of it like a chef's spice rack. You don't need to know every recipe, but you need to know what spices are available and what dishes they enhance. When a customer asks for "something with an Indian flavor profile," you know to reach for cumin, coriander, and turmeric.

### Your Spring Boot 3.5.x Capability Map

```mermaid
mindmap
  root((Spring Boot 3.5.x<br/>Capability Map))
    Concurrency
      Virtual Threads
        When: High-concurrency I/O-bound tasks
        Benefit: 10-100x more concurrent requests
        Gotcha: ThreadLocal doesn't work as expected
        Use: REST APIs, microservices
      Structured Concurrency
        When: Multiple concurrent tasks
        Benefit: Cleaner error handling
        Use: Parallel API calls
      Scoped Values
        When: Context propagation
        Benefit: Modern ThreadLocal alternative
        Use: Request context, security context
    HTTP Clients
      RestClient
        When: Simple HTTP calls
        Benefit: Fluent API, better than RestTemplate
        Use: External API integration
      WebClient
        When: Reactive streaming
        Benefit: Non-blocking backpressure
        Use: Reactive applications
    Cloud Native
      Docker Compose
        When: Local development
        Benefit: Auto-start containers
        Use: Development environment
      AOT Compilation
        When: Production deployment
        Benefit: Sub-second startup
        Use: Serverless, microservices
      GraalVM Native Images
        When: Resource-constrained environments
        Benefit: Minimal memory footprint
        Use: Lambda, edge computing
    Persistence
      Jakarta Persistence
        When: All JPA operations
        Benefit: Modern namespace
        Gotcha: javax.* → jakarta.* migration
    Observability
      Micrometer Metrics
        When: Production monitoring
        Benefit: Performance insights
      Distributed Tracing
        When: Microservices
        Benefit: Request tracking
```

### Building Your Map: Step-by-Step

#### Step 1: Scan Major Version Features

For each major Spring Boot version, document:

```markdown
### Spring Boot 3.5.x (Latest)
**Major Features:**
- Enhanced Virtual Threads support (stable in Java 21)
- Improved RestClient with better error handling
- Advanced AOT compilation for GraalVM
- Structured Concurrency support (Java 21)
- Scoped Values for context propagation
- Enhanced observability with Micrometer 1.12+
- Improved Docker Compose integration

**Breaking Changes from 3.0:**
- Minimum JDK 21 recommended (17 still supported but 21+ for virtual threads)
- Some deprecated APIs removed
- Configuration property updates

**When to Use:**
- Virtual Threads: All new I/O-bound applications
- RestClient: All new HTTP client code
- AOT: Production deployments needing fast startup
- Structured Concurrency: Multiple parallel operations
```

#### Step 2: Create a Quick Reference Card

```markdown
## Spring Boot 3.5.x Quick Reference

### Concurrency (Java 21)
- **Virtual Threads:** High-concurrency I/O (use by default in Spring Boot 3.5.x)
- **Structured Concurrency:** Multiple tasks with error handling
- **Scoped Values:** Modern ThreadLocal alternative

### HTTP Clients
- **RestClient:** Synchronous HTTP (Spring Boot 3.2+) ← Use this
- **RestTemplate:** Legacy, deprecated ← Avoid
- **WebClient:** Reactive streaming ← Use for reactive apps

### Configuration
- **application.yml:** Primary config file
- **@ConfigurationProperties:** Type-safe config (Java 21 records)
- **@Value:** Simple property injection

### Database
- **Spring Data JPA:** ORM with jakarta.persistence.*
- **R2DBC:** Reactive database access
- **JDBC Template:** Simple SQL operations

### Cloud Native
- **Docker Compose:** Local development with dependencies
- **AOT Compilation:** Native image build
- **GraalVM:** Native image runtime
```

#### Step 3: Document Breaking Changes

Create a **Breaking Changes Checklist**:

```markdown
## Spring Boot 2.x/3.0 → 3.5.x Migration Checklist

### Package Changes
- [ ] javax.servlet.* → jakarta.servlet.*
- [ ] javax.persistence.* → jakarta.persistence.*
- [ ] javax.validation.* → jakarta.validation.*
- [ ] javax.annotation.* → jakarta.annotation.*

### API Changes
- [ ] WebMvcConfigurerAdapter → WebMvcConfigurer (interface)
- [ ] RestTemplate → RestClient (for new code)
- [ ] ThreadLocal → ScopedValues (for virtual threads)
- [ ] ThreadPoolTaskExecutor → ThreadPerTaskTaskExecutor (for virtual threads)

### Configuration Changes
- [ ] server.error.include-message → updated properties
- [ ] spring.mvc.format.date → updated format
- [ ] Jackson Java 8 module → built-in support

### Minimum Requirements
- [ ] JDK 17+ minimum (JDK 21+ recommended for Virtual Threads)
- [ ] Maven 3.6+ or Gradle 7.5+
- [ ] Updated third-party libraries for Jakarta compatibility
```

### Key Features to Know (Spring Boot 3.5.x with Java 21)

#### 1. Virtual Threads (Project Loom - Stable in Java 21)

**What It Is:** Lightweight threads that are cheap to create and can be created in large numbers (millions vs thousands).

**What It Does:** Eliminates the need for complex reactive programming for most I/O-bound use cases.

**When to Use It:**
- High-concurrency REST APIs
- Microservices with many concurrent requests
- I/O-bound operations (database calls, HTTP requests, file I/O)

**Code Example:**
```java
// ✅ Spring Boot 3.5.x with Java 21 - Virtual Threads
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public AsyncTaskExecutor taskExecutor() {
        return ThreadPerTaskTaskExecutor.builder()
            .threadNamePrefix("virtual-")
            .virtualThreads(true)  // Enable virtual threads
            .build();
    }
}

@Service
public class UserService {
    
    // Runs on virtual thread automatically
    @Async
    public CompletableFuture<User> fetchUser(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            // This runs on a virtual thread
            return userRepository.findById(userId);
        });
    }
    
    // Handle 10,000+ concurrent requests easily
    @Async
    public CompletableFuture<List<User>> fetchUsers(List<Long> userIds) {
        return CompletableFuture.supplyAsync(() -> {
            return userIds.parallelStream()
                .map(this::fetchUser)
                .flatMap(CompletableFuture::stream)
                .toList();
        });
    }
}
```

**What AI Might Generate (Without Your Knowledge):**
```java
// ❌ Traditional approach - still works but misses the point
@Configuration
public class AsyncConfig {
    
    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(20);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(200);
        executor.initialize();
        return executor;
    }
}
```

**Gotcha:** Virtual threads are cheap, but blocking operations (like `Thread.sleep()`) still block the virtual thread. Use `java.util.concurrent` locks carefully. For context propagation, use Scoped Values instead of ThreadLocal.

#### 2. RestClient

**What It Is:** A modern, fluent HTTP client introduced in Spring Boot 3.2 and enhanced in 3.5.x.

**What It Does:** Provides a more intuitive API than RestTemplate with better error handling.

**When to Use It:**
- Making HTTP calls to external APIs
- Replacing RestTemplate in new code
- Synchronous HTTP operations

**Code Example:**
```java
// ✅ Spring Boot 3.5.x - RestClient
@Service
public class GitHubService {
    
    private final RestClient restClient;
    
    public GitHubService() {
        this.restClient = RestClient.builder()
            .baseUrl("https://api.github.com")
            .defaultHeader("Accept", "application/json")
            .defaultHeader("User-Agent", "MyApp")
            .build();
    }
    
    public GitHubUser getUser(String username) {
        return restClient.get()
            .uri("/users/{username}", username)
            .retrieve()
            .body(GitHubUser.class);
    }
    
    public GitHubUser getUserWithErrorHandling(String username) {
        return restClient.get()
            .uri("/users/{username}", username)
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError, 
                (request, response) -> {
                    throw new GitHubApiException("User not found: " + username);
                })
            .onStatus(HttpStatusCode::is5xxServerError,
                (request, response) -> {
                    throw new GitHubApiException("GitHub API error: " + response.getStatusCode());
                })
            .body(GitHubUser.class);
    }
    
    // With timeout and retry
    public GitHubUser getUserWithResilience(String username) {
        return restClient.get()
            .uri("/users/{username}", username)
            .attribute("timeout", Duration.ofSeconds(5))
            .retrieve()
            .body(GitHubUser.class);
    }
}
```

**What AI Might Generate (Without Your Knowledge):**
```java
// ❌ RestTemplate - deprecated approach
@Service
public class GitHubService {
    
    private final RestTemplate restTemplate = new RestTemplate();
    
    public GitHubUser getUser(String username) {
        return restTemplate.getForObject(
            "https://api.github.com/users/{username}", 
            GitHubUser.class, 
            username
        );
    }
}
```

**Gotcha:** RestClient is synchronous. For reactive applications, use WebClient.

#### 3. Docker Compose Support

**What It Is:** Spring Boot 3.1+ can automatically start Docker Compose services during development, enhanced in 3.5.x.

**What It Does:** Eliminates manual container management for local development.

**When to Use It:**
- Local development with dependencies (databases, message brokers)
- Integration testing with real services
- Onboarding new developers

**Code Example:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  kafka:
    image: confluentinc/cp-kafka:latest
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

volumes:
  postgres-data:
```

```properties
# application.properties
spring.docker.compose.enabled=true
spring.docker.compose.file=docker-compose.yml
```

**What This Does:**
- Automatically starts PostgreSQL, Redis, and Kafka when you run the app
- Waits for services to be ready
- Shuts them down when the app stops
- Works seamlessly with Spring Boot 3.5.x

**Gotcha:** Only works in development profile by default. Don't use in production.

#### 4. AOT Native Images with GraalVM

**What It Is:** Ahead-of-Time compilation to native executables using GraalVM, significantly improved in Spring Boot 3.5.x.

**What It Does:** Produces applications that start in milliseconds and use minimal memory.

**When to Use It:**
- Serverless functions (AWS Lambda, Azure Functions)
- Microservices needing fast startup
- Resource-constrained environments
- Edge computing

**Code Example:**
```java
// Native image configuration for Spring Boot 3.5.x
@NativeHint(trigger = UserService.class, 
            types = { User.class, Address.class })
@AutoConfiguration
public class NativeConfig {
    
    @Bean
    public RuntimeHintsRegistrar hintsRegistrar() {
        return ( hints, context) -> {
            if (context.isNative()) {
                hints.reflection().registerType(User.class);
                hints.serialization().registerType(User.class);
            }
        };
    }
}
```

**Build Command:**
```bash
# Build native image
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=myapp

# Run native image
docker run -p 8080:8080 myapp
```

**Performance Comparison:**

| Metric | JVM | Native Image (3.5.x) | Improvement |
|--------|-----|----------------------|-------------|
| Startup Time | 3-5 seconds | 50-100ms | 50-100x faster |
| Memory Usage | 256-512MB | 20-50MB | 5-10x less |
| Throughput | 100% | 85-95% | Slight decrease |
| Build Time | 30 seconds | 2-3 minutes | Slower build |
| Image Size | ~100MB (JAR) | ~50MB (native) | 2x smaller |

**Gotcha:** Native images require more configuration (reflection, resources, proxies). Not all libraries support native images. Spring Boot 3.5.x has improved AOT support, but testing is essential.

#### 5. Structured Concurrency (Java 21)

**What It Is:** A Java 21 feature for managing multiple concurrent tasks with better error handling.

**What It Does:** Simplifies concurrent programming by treating multiple tasks as a single unit of work.

**When to Use It:**
- Making multiple parallel API calls
- Concurrent database operations
- Any scenario with multiple independent concurrent tasks

**Code Example:**
```java
@Service
public class OrderService {
    
    private final RestClient inventoryService;
    private final RestClient pricingService;
    private final RestClient shippingService;
    
    public OrderDetails getOrderDetails(Long orderId) {
        // Structured concurrency - all tasks must complete or all fail
        try (var scope = StructuredTaskScope.shutdownOnFailure()) {
            
            // Fork tasks - they run concurrently
            Future<Inventory> inventoryFuture = scope.fork(() -> 
                inventoryService.get()
                    .uri("/inventory/{id}", orderId)
                    .retrieve()
                    .body(Inventory.class)
            );
            
            Future<Pricing> pricingFuture = scope.fork(() ->
                pricingService.get()
                    .uri("/pricing/{id}", orderId)
                    .retrieve()
                    .body(Pricing.class)
            );
            
            Future<Shipping> shippingFuture = scope.fork(() ->
                shippingService.get()
                    .uri("/shipping/{id}", orderId)
                    .retrieve()
                    .body(Shipping.class)
            );
            
            // Wait for all tasks to complete
            scope.join();
            
            // If any task failed, this throws the exception
            scope.throwIfFailed();
            
            // All tasks succeeded, get results
            return new OrderDetails(
                inventoryFuture.get(),
                pricingFuture.get(),
                shippingFuture.get()
            );
            
        } catch (Exception e) {
            throw new OrderProcessingException("Failed to fetch order details", e);
        }
    }
}
```

**Benefits:**
- Cleaner error handling (all tasks fail or all succeed)
- Automatic resource management
- Better than CompletableFuture for multiple tasks
- Works seamlessly with Virtual Threads

**Gotcha:** Only available in Java 21+. Requires Spring Boot 3.5.x with JDK 21.

#### 6. Scoped Values (Java 21)

**What It Is:** A modern alternative to ThreadLocal, designed for virtual threads.

**What It Does:** Provides a way to share immutable data across threads without the problems of ThreadLocal.

**When to Use It:**
- Request context propagation
- Security context sharing
- Any scenario where you used ThreadLocal before

**Code Example:**
```java
// ✅ Modern approach with Scoped Values
public class RequestContext {
    // Define scoped value
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    private static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();
    
    // Method to run code with context
    public static <T> T withContext(String requestId, User user, Supplier<T> supplier) {
        // Bind scoped values for this scope
        try (var scope = ScopedValue.where(REQUEST_ID, requestId)
                .where(CURRENT_USER, user)) {
            return supplier.get();
        }
    }
    
    // Get current request ID
    public static String getRequestId() {
        return REQUEST_ID.orElse("unknown");
    }
    
    // Get current user
    public static User getCurrentUser() {
        return CURRENT_USER.orElse(null);
    }
}

// Usage in controller
@GetMapping("/data")
public ResponseEntity<Data> getData() {
    return RequestContext.withContext(
        UUID.randomUUID().toString(),
        getCurrentUser(),
        () -> {
            // REQUEST_ID.get() works here
            // CURRENT_USER.get() works here
            // Even in virtual threads!
            return service.getData();
        }
    );
}
```

**vs ThreadLocal:**
```java
// ❌ Old approach - problematic with virtual threads
private static final ThreadLocal<String> requestId = new ThreadLocal<>();

@GetMapping("/data")
public ResponseEntity<Data> getData() {
    requestId.set(UUID.randomUUID().toString());
    // Virtual thread unmounts here
    return service.getData(); // requestId.get() returns null!
}
```

**Benefits:**
- Works correctly with virtual threads
- Immutable by design
- Better performance
- Cleaner semantics

**Gotcha:** Only available in Java 21+. Requires Spring Boot 3.5.x with JDK 21.

### The Capability Map in Action

**Scenario:** You need to build a high-concurrency endpoint that calls an external API.

**Without Capability Map:**
```
You: "Write me an HTTP endpoint that calls an external API"
AI: Generates code with RestTemplate and traditional thread pools
Result: Works, but not optimal
```

**With Capability Map:**
```
You: "Write me a high-concurrency HTTP endpoint using Spring Boot 3.5.x's 
     RestClient, running on virtual threads with Structured Concurrency 
     for parallel calls, use Scoped Values for context propagation, 
     and add circuit breaker pattern with Resilience4j"
     
AI: Generates modern, optimized code
Result: Production-ready, uses best practices
```

> 💡 **The Power of Specificity:** The more specific you are about which features to use, the better AI's output will be. Your capability map gives you the vocabulary to be specific.

---

## Part 4: Learning Strategies for the AI Era

### For Beginners: Don't Go All-In on AI Right Away

#### The Foundation-First Approach

Relying on AI before you have a solid foundation turns you into a **copy-paste engineer** fast — the code runs, but you have no idea why, and you're completely lost when a bug shows up.

#### Learning Path for Beginners

```mermaid
graph TD
    A[Start Here] --> B[Learn Spring Boot Core]
    B --> C[Build 3-5 Projects Without AI]
    C --> D[Understand Annotations & Config]
    D --> E[Learn Common Patterns]
    E --> F[Start Using AI for Boilerplate]
    F --> G[Review AI Code Critically]
    G --> H[Gradually Increase AI Usage]
    H --> I[AI as Amplifier, Not Crutch]
    
    style A fill:#e1f5ff
    style I fill:#e8f5e9
```

**Phase 1: Foundation (Months 1-3)**
- Learn Spring Boot core concepts
- Build projects WITHOUT AI assistance
- Understand annotations, dependency injection, configuration
- Learn common patterns (CRUD, REST, validation)
- Master Java 21 features (records, pattern matching, virtual threads basics)

**Phase 2: AI Introduction (Months 3-6)**
- Use AI for boilerplate generation
- Always read and understand AI-generated code
- Make simple modifications on your own
- Learn to spot obvious errors
- Start using Virtual Threads and RestClient

**Phase 3: AI Amplification (Months 6+)**
- Use AI for repetitive tasks
- Focus on architecture and design
- Review AI code critically
- Make informed technology choices
- Master advanced features (Structured Concurrency, Scoped Values, AOT)

#### Minimum Viable Understanding

Before relying on AI, you should be able to:
- ✅ Read a Spring Boot controller and explain what it does
- ✅ Explain what @Autowired does and why
- ✅ Configure application.properties correctly
- ✅ Debug a simple NullPointerException
- ✅ Write a basic integration test
- ✅ Understand the request/response flow
- ✅ Explain what Virtual Threads are and when to use them
- ✅ Understand the difference between RestClient and RestTemplate

> ⚠️ **Warning:** If you can't read and understand AI-generated code, you're not ready to use AI as a primary tool. You'll create bugs you can't fix and accumulate technical debt you don't understand.

### For Experienced Developers: Build a "Capability Map"

#### The Scanning Approach

You don't need to read release notes word for word. But you should absolutely scan through each major version's headline features and breaking changes.

#### Quarterly Review Process

```mermaid
graph LR
    A[Quarterly Review] --> B[Scan Release Notes]
    B --> C[Identify New Features]
    C --> D[Document Breaking Changes]
    D --> E[Update Capability Map]
    E --> F[Try One New Feature]
    F --> G[Add to Toolkit]
    
    style A fill:#e1f5ff
    style G fill:#e8f5e9
```

**Monthly Time Investment:** 2-3 hours

**Process:**
1. **Scan** Spring Boot release notes (30 minutes)
2. **Identify** 2-3 features relevant to your work (30 minutes)
3. **Document** breaking changes (30 minutes)
4. **Try** one new feature in a small project (1-2 hours)

#### The Mental Index

What matters is having that instinct:

> "Wait, doesn't Spring Boot 3.5.x have a better way to solve this?"

**Example Thought Process:**
```
Problem: Need to make HTTP calls to external API
Old Thinking: "I'll use RestTemplate, it works fine"
New Thinking: "Wait, Spring Boot 3.2+ introduced RestClient. 
              Should I use that instead? And with Java 21, 
              should I run it on virtual threads?"
```

The dangerous part is not even knowing what to ask for.

### The "Product Manager" Mindset

#### Treat AI as Your Code Implementer

The right workflow looks like this:

```
YOU: "Implement an HTTP call with retry and timeout using Spring Boot 3.5.x's 
      RestClient, run it on virtual threads, use Structured Concurrency for 
      parallel calls, use Scoped Values for context propagation, and add 
      circuit breaker pattern with Resilience4j"
      
AI: [Generates comprehensive, modern code]

YOU: [Reviews for correctness, tweaks as needed]
```

**You define:**
- ✅ The tech choices
- ✅ The constraints
- ✅ The requirements
- ✅ The quality standards

**AI produces:**
- ✅ The implementation
- ✅ The boilerplate
- ✅ The syntax

#### The Prompt Engineering Mindset

**Bad Prompt:**
```
"Write me an HTTP client"
```
**Result:** AI gives you RestTemplate from 2015

**Good Prompt:**
```
"Write a Spring Boot 3.5.x HTTP client using RestClient with:
- Running on Java 21 virtual threads
- Connection timeout: 5 seconds
- Read timeout: 10 seconds
- Retry mechanism with exponential backoff (3 retries)
- Circuit breaker pattern using Resilience4j
- Proper error handling and logging
- Scoped Values for request context propagation
- OpenAPI documentation annotations"
```
**Result:** Modern, production-ready code

> 💡 **Pro Tip:** Always specify the Spring Boot version AND Java version in your prompts. This alone eliminates 50% of AI errors.

### Progressive Learning Path

#### Month 1-2: Core Features
- Master Spring Boot 3.5.x fundamentals
- Learn Jakarta namespace changes
- Understand new configuration properties
- Practice with Docker Compose
- Learn Java 21 features (records, pattern matching)

#### Month 3-4: Advanced Features
- Virtual Threads and concurrency (Java 21)
- RestClient and HTTP clients
- Structured Concurrency
- Scoped Values
- AOT compilation basics
- Observability and metrics

#### Month 5-6: Production Readiness
- Native image optimization with GraalVM
- Performance tuning
- Security best practices
- Monitoring and debugging
- Advanced AOT configuration

#### Ongoing: Stay Current
- Follow Spring Boot blog
- Attend Spring One/Spring I/O
- Read release notes quarterly
- Experiment with new features
- Contribute to Spring Boot projects

---

## Part 5: Hands-On Implementation Guide

### Exercise 1: AI Prompt Engineering for Spring Boot 3.5.x

#### Objective
Learn to write effective prompts that generate modern, production-ready Spring Boot 3.5.x code.

#### Scenario
You need to build a REST endpoint that fetches data from an external API with proper error handling.

#### Step-by-Step Implementation

**Step 1: Start with a Bad Prompt**

```markdown
Prompt: "Write a REST controller that calls an API"
```

**AI Output (Likely):**
```java
// ❌ Uses old patterns
@RestController
public class ApiController {
    
    @Autowired
    private RestTemplate restTemplate;  // Deprecated
    
    @GetMapping("/data")
    public String getData() {
        return restTemplate.getForObject(
            "https://api.example.com/data", 
            String.class
        );
    }
}
```

**Issues:**
- Uses deprecated RestTemplate
- No error handling
- No timeout configuration
- No version specification
- No modern Java 21 features

**Step 2: Improve with Version Specification**

```markdown
Prompt: "Write a Spring Boot 3.5.x REST controller that calls an API using Java 21"
```

**AI Output (Better):**
```java
// ✅ Better, uses RestClient
@RestController
public class ApiController {
    
    private final RestClient restClient;
    
    public ApiController() {
        this.restClient = RestClient.builder()
            .baseUrl("https://api.example.com")
            .build();
    }
    
    @GetMapping("/data")
    public String getData() {
        return restClient.get()
            .uri("/data")
            .retrieve()
            .body(String.class);
    }
}
```

**Improvements:**
- Uses RestClient (modern)
- Still missing error handling, timeouts, virtual threads

**Step 3: Perfect Prompt with All Requirements**

```markdown
Prompt: "Write a Spring Boot 3.5.x REST controller using Java 21 that:

1. Calls https://api.example.com/data endpoint using RestClient
2. Runs on virtual threads
3. Has connection timeout of 5 seconds and read timeout of 10 seconds
4. Implements retry with exponential backoff (3 retries, initial delay 1s)
5. Includes circuit breaker pattern using Resilience4j
6. Has proper error handling with custom exceptions
7. Returns ResponseEntity with appropriate status codes
8. Includes logging at INFO level for requests and ERROR level for failures
9. Uses Scoped Values for request context propagation
10. Includes OpenAPI documentation annotations
11. Uses Structured Concurrency for parallel operations where applicable"
```

**AI Output (Production-Ready):**
```java
import org.springframework.web.bind.annotation.*;
import org.springframework.web.reactive.function.client.*;
import org.springframework.http.*;
import org.springframework.stereotype.Component;
import io.github.resilience4j.circuitbreaker.*;
import io.github.resilience4j.retry.*;
import org.slf4j.*;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.Operation;
import java.time.Duration;
import java.util.concurrent.*;

@RestController
@RequestMapping("/api/v1")
@Tag(name = "External API", description = "External API integration")
public class ExternalApiController {
    
    private static final Logger logger = LoggerFactory.getLogger(ExternalApiController.class);
    
    // Scoped Value for request context
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    
    private final RestClient restClient;
    private final CircuitBreaker circuitBreaker;
    private final Retry retry;
    
    public ExternalApiController() {
        this.restClient = RestClient.builder()
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
            .build();
            
        // Configure circuit breaker
        this.circuitBreaker = CircuitBreaker.of("externalApi", 
            CircuitBreakerConfig.custom()
                .failureRateThreshold(50)
                .waitDurationInOpenState(Duration.ofSeconds(30))
                .slidingWindowSize(10)
                .build()
        );
        
        // Configure retry
        this.retry = Retry.of("externalApi",
            RetryConfig.custom()
                .maxAttempts(3)
                .waitDuration(Duration.ofMillis(1000))
                .build()
        );
    }
    
    @GetMapping("/data")
    @Operation(summary = "Fetch data from external API")
    public ResponseEntity<DataResponse> fetchData() {
        String requestId = UUID.randomUUID().toString();
        
        // Use Scoped Value for context propagation
        try (var scope = ScopedValue.where(REQUEST_ID, requestId)) {
            logger.info("[{}] Fetching data from external API", requestId);
            
            // Structured concurrency for parallel operations
            try (var taskScope = StructuredTaskScope.shutdownOnFailure()) {
                
                Future<DataResponse> dataFuture = taskScope.fork(() -> 
                    makeApiCall(requestId)
                );
                
                Future<Metadata> metadataFuture = taskScope.fork(() ->
                    fetchMetadata(requestId)
                );
                
                taskScope.join();
                taskScope.throwIfFailed();
                
                DataResponse data = dataFuture.get();
                Metadata metadata = metadataFuture.get();
                
                return ResponseEntity.ok(new DataResponse(data, metadata));
                
            } catch (Exception e) {
                logger.error("[{}] Failed to fetch data: {}", requestId, e.getMessage());
                return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                    .body(new DataResponse("Service temporarily unavailable"));
            }
        }
    }
    
    private DataResponse makeApiCall(String requestId) {
        logger.info("[{}] Making API call with timeout", requestId);
        
        return restClient.get()
            .uri("/data")
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError, 
                (request, response) -> {
                    throw new ExternalApiException("Client error: " + response.getStatusCode());
                })
            .onStatus(HttpStatusCode::is5xxServerError,
                (request, response) -> {
                    throw new ExternalApiException("Server error: " + response.getStatusCode());
                })
            .body(DataResponse.class);
    }
    
    private Metadata fetchMetadata(String requestId) {
        return restClient.get()
            .uri("/metadata")
            .retrieve()
            .body(Metadata.class);
    }
}

// DTOs using records (Java 16+)
record DataResponse(String data, Metadata metadata) {}
record DataResponse(String message) {}
record Metadata(String version, String timestamp) {}
```

**Key Improvements:**
- ✅ Uses RestClient (Spring Boot 3.5.x)
- ✅ Runs on virtual threads (Java 21)
- ✅ Structured Concurrency for parallel operations
- ✅ Scoped Values for context propagation
- ✅ Circuit breaker for fault tolerance
- ✅ Retry with exponential backoff
- ✅ Proper error handling
- ✅ Logging with request IDs
- ✅ OpenAPI documentation
- ✅ Timeout configuration

#### Step 4: Review and Refine

**Always review AI code for:**
1. ✅ Correct Spring Boot 3.5.x version features used
2. ✅ Java 21 features used appropriately (virtual threads, structured concurrency, scoped values)
3. ✅ Proper error handling
4. ✅ Security considerations (input validation, auth)
5. ✅ Performance implications
6. ✅ Best practices followed
7. ✅ Testability

### Exercise 2: Code Review Checklist for AI-Generated Spring Boot 3.5.x Code

Create a checklist to review AI-generated Spring Boot code:

```markdown
## AI-Generated Code Review Checklist for Spring Boot 3.5.x

### Version Compatibility
- [ ] Uses correct Spring Boot 3.5.x features
- [ ] No deprecated APIs (RestTemplate, WebMvcConfigurerAdapter, etc.)
- [ ] Correct namespace (jakarta.* not javax.* for Spring Boot 3.x)
- [ ] Minimum JDK 21 requirements met (for virtual threads, structured concurrency)
- [ ] Uses Java 21 features where appropriate (records, pattern matching)

### Code Quality
- [ ] Follows Spring Boot 3.5.x conventions
- [ ] Proper separation of concerns
- [ ] No code duplication
- [ ] Appropriate use of design patterns
- [ ] Clean, readable code
- [ ] Uses records for DTOs where appropriate

### Error Handling
- [ ] Try-catch blocks where needed
- [ ] Custom exceptions for domain errors
- [ ] Proper HTTP status codes
- [ ] Error messages don't expose sensitive info
- [ ] Logging at appropriate levels
- [ ] Structured Concurrency used for parallel tasks

### Concurrency (Java 21)
- [ ] Virtual Threads used for I/O-bound tasks
- [ ] ThreadLocal avoided (use Scoped Values instead)
- [ ] Structured Concurrency for multiple concurrent tasks
- [ ] No unnecessary synchronization
- [ ] Proper use of async/await

### Security
- [ ] Input validation present
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (output encoding)
- [ ] Authentication/authorization checks
- [ ] No hardcoded credentials
- [ ] Sensitive data encrypted

### Performance
- [ ] Connection pooling configured
- [ ] Timeouts set appropriately
- [ ] No N+1 query problems
- [ ] Appropriate use of caching
- [ ] Efficient data structures used
- [ ] RestClient used instead of RestTemplate

### Testing
- [ ] Testable code (dependency injection)
- [ ] No hardcoded values
- [ ] Mockable dependencies
- [ ] Edge cases considered
- [ ] Unit tests for business logic
- [ ] Integration tests for API endpoints

### Documentation
- [ ] Code is self-documenting
- [ ] Complex logic has comments
- [ ] OpenAPI annotations present
- [ ] README updated if needed
```

### Exercise 3: Migrate Spring Boot 3.0 Application to 3.5.x with Java 21

#### Scenario
You have a Spring Boot 3.0 application that needs to be migrated to 3.5.x with Java 21 features.

#### Step-by-Step Migration

**Step 1: Audit Current Codebase**

```bash
# Find all javax.* imports
grep -r "import javax\." src/

# Find deprecated APIs
grep -r "WebMvcConfigurerAdapter" src/
grep -r "RestTemplate" src/

# Check current Java version
java -version
```

**Step 2: Create Migration Prompt**

```markdown
Prompt: "Migrate this Spring Boot 3.0 code to Spring Boot 3.5.x with Java 21:

[PASTE CODE HERE]

Requirements:
1. Update to use Java 21 features where appropriate (virtual threads, structured concurrency, scoped values)
2. Change all javax.* imports to jakarta.*
3. Replace WebMvcConfigurerAdapter with WebMvcConfigurer interface
4. Replace RestTemplate with RestClient
5. Use records for DTOs
6. Add virtual threads for async operations
7. Update application.properties keys if needed
8. Add comments explaining changes
9. Ensure JDK 21 compatibility"
```

**Step 3: Review AI Output**

```java
// Before (Spring Boot 3.0)
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.validation.constraints.NotBlank;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Entity
public class User {
    @Id
    private Long id;
    
    @NotBlank
    private String name;
}

// After (Spring Boot 3.5.x with Java 21)
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.validation.constraints.NotBlank;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import java.util.concurrent.*;

@Entity
public class User {
    @Id
    private Long id;
    
    @NotBlank
    private String name;
    
    // Consider using records for DTOs
}

// Updated service with virtual threads
@Service
public class UserService {
    private final UserRepository userRepository;
    
    // Virtual thread executor
    private final ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
    
    public CompletableFuture<User> fetchUser(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            return userRepository.findById(userId);
        }, executor);
    }
}
```

**Step 4: Test Thoroughly**

```bash
# Compile with JDK 21
./mvnw clean compile

# Run tests
./mvnw test

# Run application
./mvnw spring-boot:run

# Check for errors in logs
```

**Step 5: Update Dependencies**

```xml
<!-- pom.xml -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.0</version>  <!-- Updated from 3.0.x -->
</parent>

<properties>
    <java.version>21</java.version>  <!-- Updated from 17 -->
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- For virtual threads and structured concurrency -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aot</artifactId>
    </dependency>
</dependencies>
```

---

## Real-World Use Cases

### Use Case 1: High-Concurrency E-Commerce API

**Scenario:** Build a product search API handling 10,000 concurrent requests.

**Traditional Approach (Without Knowledge of Virtual Threads):**
```java
@Service
public class ProductService {
    
    private final ExecutorService executor = Executors.newFixedThreadPool(100);
    
    public List<Product> searchProducts(String query) {
        // Complex CompletableFuture chains
        // Limited to 100 concurrent requests
        // Thread pool exhaustion under load
    }
}
```

**Modern Approach (With Virtual Threads in Spring Boot 3.5.x):**
```java
@Service
public class ProductService {
    
    // Java 21 virtual thread executor
    private final ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
    
    @Async
    public CompletableFuture<List<Product>> searchProducts(String query) {
        // Runs on virtual thread
        // Can handle 10,000+ concurrent requests
        // No thread pool configuration needed
        return CompletableFuture.supplyAsync(() -> {
            return productRepository.search(query);
        }, executor);
    }
}
```

**Performance Comparison:**

| Metric | Traditional Threads | Virtual Threads (Java 21) | Improvement |
|--------|--------------------|---------------------------|-------------|
| Max Concurrent Requests | 100 | 10,000+ | 100x |
| Memory Usage | 512MB | 128MB | 4x less |
| Response Time (p95) | 250ms | 120ms | 2x faster |
| Thread Creation Time | 10ms | 0.1ms | 100x faster |

### Use Case 2: Microservices Communication with Structured Concurrency

**Scenario:** Service A needs to call Service B and Service C in parallel.

**Without Knowledge of Modern Features:**
```java
// Complex WebClient setup
// Manual error handling
// Thread management
```

**With Modern Spring Boot 3.5.x and Java 21:**
```java
@Service
public class AggregationService {
    
    private final RestClient serviceBClient;
    private final RestClient serviceCClient;
    
    public AggregationResult aggregateData(Long id) {
        // Structured concurrency (Java 21)
        try (var scope = StructuredTaskScope.shutdownOnFailure()) {
            
            Future<DataB> futureB = scope.fork(() -> 
                serviceBClient.get().uri("/data/{id}", id).retrieve().body(DataB.class)
            );
            
            Future<DataC> futureC = scope.fork(() ->
                serviceCClient.get().uri("/data/{id}", id).retrieve().body(DataC.class)
            );
            
            scope.join();
            scope.throwIfFailed();
            
            return new AggregationResult(futureB.get(), futureC.get());
            
        } catch (Exception e) {
            throw new AggregationException("Failed to aggregate data", e);
        }
    }
}
```

**Benefits:**
- Cleaner error handling
- Automatic resource management
- Better performance with virtual threads
- More readable code
- All tasks succeed or all fail

### Use Case 3: Local Development Environment

**Scenario:** New developer joins team, needs to run application with PostgreSQL, Redis, and Kafka.

**Traditional Approach:**
```bash
# Manual steps:
# 1. Install PostgreSQL
# 2. Install Redis
# 3. Install Kafka
# 4. Create databases
# 5. Configure connections
# 6. Start services
# 7. Run application
# Time: 2-3 hours
```

**Modern Approach (Docker Compose in Spring Boot 3.5.x):**
```bash
# Single command:
./mvnw spring-boot:run
# Docker Compose automatically starts PostgreSQL, Redis, and Kafka
# Time: 5 minutes
```

**Configuration:**
```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp_dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  kafka:
    image: confluentinc/cp-kafka:latest
    ports:
      - "9092:9092"
```

```properties
# application-dev.properties
spring.docker.compose.enabled=true
spring.datasource.url=jdbc:postgresql://localhost:5432/myapp_dev
spring.data.redis.host=localhost
spring.kafka.bootstrap-servers=localhost:9092
```

### Use Case 4: Serverless Function with Native Image

**Scenario:** Deploy a Spring Boot 3.5.x application as an AWS Lambda function.

**Traditional Approach (JVM):**
- Cold start: 3-5 seconds
- Memory: 512MB
- Timeout issues with slow starts

**Modern Approach (Native Image with GraalVM):**
```bash
# Build native image
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=myapp

# Deploy to AWS Lambda
# Cold start: 50-100ms
# Memory: 64MB
# No timeout issues
```

**Performance Metrics:**

| Metric | JVM Lambda | Native Image Lambda | Improvement |
|--------|------------|---------------------|-------------|
| Cold Start | 3-5 seconds | 50-100ms | 50-100x faster |
| Memory Usage | 512MB | 64MB | 8x less |
| Cost | $0.20 per 1M requests | $0.10 per 1M requests | 2x cheaper |

---

## Common Pitfalls & Troubleshooting

### Pitfall 1: AI Generates Code with Version Mismatches

**Symptom:** Code doesn't compile or throws ClassNotFoundException

**Example:**
```java
// AI generates javax.servlet.* in Spring Boot 3.5.x project
import javax.servlet.http.HttpServletRequest;  // ❌ Doesn't exist
```

**Solution:**
1. Always specify Spring Boot version AND Java version in prompts
2. Maintain a breaking changes checklist
3. Review imports carefully
4. Use IDE to highlight missing classes

**Prevention:**
```markdown
Always include in your prompts:
"Use Spring Boot 3.5.x with JDK 21, jakarta.* namespace, not javax.*"
```

### Pitfall 2: AI Uses Deprecated APIs

**Symptom:** IDE warnings, future compatibility issues

**Example:**
```java
// AI generates deprecated WebMvcConfigurerAdapter
public class CorsConfig extends WebMvcConfigurerAdapter {  // ⚠️ Deprecated
}
```

**Solution:**
1. Enable IDE warnings for deprecated APIs
2. Consult Spring Boot migration guides
3. Use modern alternatives

**Correct Version:**
```java
public class CorsConfig implements WebMvcConfigurer {  // ✅ Modern
}
```

### Pitfall 3: Missing Error Handling

**Symptom:** Application crashes on unexpected input

**Example:**
```java
// AI generates code without error handling
public User getUser(Long id) {
    return restClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .body(User.class);  // ❌ What if 404? What if timeout?
}
```

**Solution:**
```java
// ✅ Proper error handling
public User getUser(Long id) {
    try {
        return restClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError, 
                (req, res) -> throw new UserNotFoundException(id))
            .body(User.class);
    } catch (RestClientException e) {
        throw new ExternalServiceException("Failed to fetch user", e);
    }
}
```

### Pitfall 4: ThreadLocal Issues with Virtual Threads

**Symptom:** Data loss or unexpected behavior in concurrent code

**Example:**
```java
// ❌ ThreadLocal doesn't work as expected with virtual threads
private static final ThreadLocal<String> context = new ThreadLocal<>();

@GetMapping("/process")
public String process() {
    context.set("value");
    // Virtual thread unmounts here
    return doWork();  // context.get() returns null!
}
```

**Solution:**
```java
// ✅ Use ScopedValue (Java 21) or pass context explicitly
private static final ScopedValue<String> CONTEXT = ScopedValue.newInstance();

@GetMapping("/process")
public String process() {
    try (var scope = ScopedValue.where(CONTEXT, "value")) {
        return doWork();  // CONTEXT.get() returns "value"
    }
}
```

### Pitfall 5: Native Image Configuration Issues

**Symptom:** Native image build fails with ClassNotFoundException

**Example:**
```java
// ❌ Missing reflection configuration
public class UserService {
    public User getUser(Long id) {
        return new ObjectMapper().readValue(json, User.class);
        // Fails in native image - ObjectMapper needs reflection config
    }
}
```

**Solution:**
```java
// ✅ Add native hint
@NativeHint(trigger = UserService.class, 
            types = { User.class })
@AutoConfiguration
public class NativeConfig {
    // Configuration for native image
}
```

### Pitfall 6: Not Using Java 21 Features

**Symptom:** Code works but misses modern Java 21 improvements

**Example:**
```java
// ❌ Not using records for DTOs
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    
    // Getters and setters
    // Constructor
    // toString, equals, hashCode
}
```

**Solution:**
```java
// ✅ Use records (Java 16+)
public record UserDTO(Long id, String name, String email) {}
```

### Troubleshooting Decision Tree

```mermaid
graph TD
    A[Problem] --> B{Compilation Error?}
    B -->|Yes| C[Check Imports]
    C --> D{Using javax.*?}
    D -->|Yes| E[Change to jakarta.*]
    D -->|No| F[Check Dependencies]
    
    B -->|No| G{Runtime Error?}
    G -->|Yes| H{ClassNotFoundException?}
    H -->|Yes| I[Check Native Image Config]
    H -->|No| J{NullPointerException?}
    J -->|Yes| K[Check ThreadLocal Usage]
    J -->|No| L[Check Logs]
    
    G -->|No| M{Performance Issue?}
    M -->|Yes| N[Check Virtual Threads Config]
    M -->|No| O[Check AI Prompt Specificity]
    
    style A fill:#ffebee
    style E fill:#e8f5e9
    style I fill:#fff4e1
```

---

## Best Practices

### 1. Always Specify Spring Boot and Java Versions in Prompts

```markdown
✅ Good: "Using Spring Boot 3.5.x with JDK 21..."
❌ Bad: "Write a Spring Boot controller..."
```

### 2. Maintain a Capability Map

- Review quarterly
- Document new features
- Note breaking changes
- Update your mental model

### 3. Review AI Code Critically

- Check for deprecated APIs
- Verify error handling
- Validate security practices
- Test edge cases

### 4. Start with Foundation, Then Amplify with AI

- Learn core concepts first
- Build projects without AI
- Gradually introduce AI
- Never rely on AI exclusively

### 5. Use AI for Boilerplate, Not Architecture

```markdown
✅ Good: "Generate CRUD controller for User entity"
❌ Bad: "Design my entire microservices architecture"
```

### 6. Test AI-Generated Code Thoroughly

- Unit tests
- Integration tests
- Edge case testing
- Performance testing

### 7. Keep Learning Continuously

- Follow Spring blog
- Attend conferences
- Read release notes
- Experiment with new features

### 8. Use Java 21 Features Where Appropriate

- Virtual Threads for I/O-bound tasks
- Structured Concurrency for parallel operations
- Scoped Values instead of ThreadLocal
- Records for DTOs
- Pattern matching for instanceof

### 9. Document Your AI Workflow

```markdown
## My AI-Assisted Development Workflow

1. Define requirements clearly
2. Choose technology stack (Spring Boot 3.5.x, JDK 21)
3. Write detailed prompt with version specs
4. Review AI output against checklist
5. Refine and test
6. Document learnings
```

### 10. Share Knowledge with Team

- Create internal capability map
- Conduct code review sessions
- Share AI prompt templates
- Document common pitfalls

### 11. Balance AI Usage with Deep Work

- Use AI for repetitive tasks
- Reserve deep work for architecture
- Don't let AI replace critical thinking
- Stay hands-on with complex problems

### 12. Leverage Spring Boot 3.5.x Features

- Use RestClient instead of RestTemplate
- Enable virtual threads for async operations
- Use Docker Compose for local development
- Consider AOT compilation for production
- Use Structured Concurrency for parallel tasks

---

## Anti-Patterns

### Anti-Pattern 1: Over-Reliance on AI

**Problem:** Using AI for everything without understanding

**Example:**
```java
// Developer has no idea what this code does
// AI generated it, they copied it, it works
@Service
public class ComplexService {
    // 500 lines of AI-generated code
    // Developer can't debug or modify it
}
```

**Consequences:**
- Can't debug production issues
- Can't modify code safely
- Accumulates technical debt
- Stops learning

**Solution:**
- Always understand AI-generated code
- Build foundation first
- Review before using
- Learn from AI output

### Anti-Pattern 2: Copy-Paste Without Understanding

**Problem:** Blindly copying AI code without comprehension

**Example:**
```markdown
Developer: "Write authentication with JWT"
AI: [Generates 200 lines of code]
Developer: [Copies without reading]
Result: Security vulnerability because they didn't notice 
        the token expiration wasn't validated
```

**Consequences:**
- Security vulnerabilities
- Performance issues
- Bugs in production
- Knowledge gaps

**Solution:**
- Read every line
- Ask AI to explain
- Review against best practices
- Test thoroughly

### Anti-Pattern 3: Ignoring Breaking Changes

**Problem:** Not learning about version changes

**Example:**
```java
// Developer upgrades Spring Boot 3.0 → 3.5.x
// Doesn't know about new Java 21 features
// Code doesn't use virtual threads
// Spends 3 days debugging performance issues
```

**Consequences:**
- Wasted time
- Frustration
- Delayed releases
- Technical debt

**Solution:**
- Read migration guides
- Maintain breaking changes checklist
- Test in isolation first
- Use AI to help migrate

### Anti-Pattern 4: Vague Prompts

**Problem:** Asking AI without enough context

**Example:**
```markdown
❌ "Write a REST API"
❌ "Make it secure"
❌ "Add error handling"
```

**Result:** Generic, non-production-ready code

**Solution:**
```markdown
✅ "Write a Spring Boot 3.5.x REST API for User management with:
    - JWT authentication
    - Input validation using Jakarta Validation
    - Global exception handler
    - Proper HTTP status codes
    - Virtual threads for async operations
    - OpenAPI documentation"
```

### Anti-Pattern 5: Not Testing AI Code

**Problem:** Assuming AI code is correct

**Example:**
```java
// AI generated this, looks good
public User createUser(User user) {
    return userRepository.save(user);
}
// But didn't check for null, validation, or duplicates
```

**Consequences:**
- Production bugs
- Data corruption
- Security issues
- Poor user experience

**Solution:**
- Write tests for all AI code
- Test edge cases
- Test error scenarios
- Test performance

### Anti-Pattern 6: Using Outdated Patterns

**Problem:** AI generates old patterns because that's in training data

**Example:**
```java
// ❌ AI generates this for Spring Boot 3.5.x
public class Config extends WebMvcConfigurerAdapter {  // Deprecated!
}
```

**Consequences:**
- Deprecated code
- Future compatibility issues
- Missed optimizations
- Technical debt

**Solution:**
- Specify version in prompts
- Review for deprecated APIs
- Consult official docs
- Stay current

### Anti-Pattern 7: AI for Architecture Decisions

**Problem:** Letting AI make architectural choices

**Example:**
```markdown
Developer: "Should I use microservices or monolith?"
AI: "Microservices are better because..."
Developer: [Follows advice without context]
Result: Wrong choice for their specific situation
```

**Consequences:**
- Wrong architecture
- Increased complexity
- Higher costs
- Team frustration

**Solution:**
- AI for suggestions only
- Make final decisions yourself
- Consider your context
- Consult team/experts

### Anti-Pattern 8: Neglecting Performance

**Problem:** AI generates working but slow code

**Example:**
```java
// AI generates N+1 query problem
public List<Order> getOrdersWithUsers() {
    List<Order> orders = orderRepository.findAll();
    for (Order order : orders) {
        order.setUser(userRepository.findById(order.getUserId()));
        // N+1 problem!
    }
    return orders;
}
```

**Consequences:**
- Poor performance
- Database overload
- Slow response times
- Scalability issues

**Solution:**
- Review for performance issues
- Use JOIN FETCH
- Enable query logging
- Profile regularly

### Anti-Pattern 9: Not Using Modern Java Features

**Problem:** Sticking to old Java patterns when using Spring Boot 3.5.x with JDK 21

**Example:**
```java
// ❌ Using old patterns
public class UserDTO {
    private Long id;
    private String name;
    // Getters, setters, constructor, toString, equals, hashCode
}

// ✅ Should use records
public record UserDTO(Long id, String name) {}
```

**Consequences:**
- More boilerplate code
- Missed immutability benefits
- Outdated codebase
- Harder to maintain

**Solution:**
- Learn Java 21 features
- Use records for DTOs
- Use pattern matching
- Use virtual threads

---

## Performance Considerations

### AI-Generated Code Performance Issues

AI often generates code that works but isn't optimized:

#### Issue 1: N+1 Query Problem

**AI Generated:**
```java
// ❌ N+1 queries
public List<OrderDTO> getOrders() {
    List<Order> orders = orderRepository.findAll();
    return orders.stream()
        .map(order -> {
            User user = userRepository.findById(order.getUserId()).get();
            return new OrderDTO(order, user);
        })
        .toList();
}
// Executes: 1 query for orders + N queries for users
```

**Optimized:**
```java
// ✅ Single query with JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.user")
public List<Order> findAllWithUser();

public List<OrderDTO> getOrders() {
    return orderRepository.findAllWithUser().stream()
        .map(OrderDTO::new)
        .toList();
}
// Executes: 1 query total
```

**Performance Impact:**
- N+1: 1000 orders = 1001 queries (2-3 seconds)
- Optimized: 1000 orders = 1 query (50ms)
- **Improvement: 40-60x faster**

#### Issue 2: Missing Connection Pooling

**AI Generated:**
```java
// ❌ No connection pooling
@Bean
public DataSource dataSource() {
    return DataSourceBuilder.create()
        .url("jdbc:postgresql://localhost:5432/mydb")
        .build();
}
```

**Optimized:**
```java
// ✅ With HikariCP (Spring Boot default)
@Bean
public DataSource dataSource() {
    HikariConfig config = new HikariConfig();
    config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
    config.setMaximumPoolSize(20);
    config.setMinimumIdle(5);
    config.setConnectionTimeout(20000);
    return new HikariDataSource(config);
}
```

**Performance Impact:**
- No pooling: 100-200 requests/second
- With pooling: 1000-2000 requests/second
- **Improvement: 10x throughput**

#### Issue 3: Not Using Virtual Threads

**AI Generated:**
```java
// ❌ Traditional thread pool
@Configuration
public class AsyncConfig {
    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(20);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(200);
        executor.initialize();
        return executor;
    }
}
```

**Optimized:**
```java
// ✅ Virtual threads (Java 21)
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public AsyncTaskExecutor taskExecutor() {
        return ThreadPerTaskTaskExecutor.builder()
            .threadNamePrefix("virtual-")
            .virtualThreads(true)
            .build();
    }
}
```

**Performance Impact:**
- Traditional: 100 concurrent requests, thread pool exhaustion at 200
- Virtual threads: 10,000+ concurrent requests, no exhaustion
- **Improvement: 100x concurrency**

### Performance Optimization Checklist

```markdown
## AI-Generated Code Performance Review

### Database
- [ ] No N+1 queries (use JOIN FETCH)
- [ ] Connection pooling configured (HikariCP)
- [ ] Proper indexing
- [ ] Batch operations where appropriate
- [ ] Query optimization

### Caching
- [ ] Appropriate caching strategy
- [ ] Cache invalidation strategy
- [ ] Cache key design
- [ ] TTL configured

### Concurrency (Java 21)
- [ ] Virtual threads used where appropriate
- [ ] Structured Concurrency for parallel tasks
- [ ] Scoped Values instead of ThreadLocal
- [ ] No unnecessary synchronization
- [ ] Thread-safe collections

### Memory
- [ ] No memory leaks
- [ ] Proper resource cleanup
- [ ] Efficient data structures (use records)
- [ ] Streaming for large datasets

### Network
- [ ] Connection pooling
- [ ] Timeout configuration
- [ ] Retry with backoff
- [ ] Circuit breaker for external calls
```

### Performance Benchmarks

**Virtual Threads vs Traditional Threads (Java 21):**

| Scenario | Traditional Threads | Virtual Threads | Improvement |
|----------|--------------------|-----------------|-------------|
| 100 concurrent requests | 100 threads, 512MB | 100 virtual threads, 64MB | 8x less memory |
| 1,000 concurrent requests | Thread pool exhaustion | 1,000 virtual threads | Works! |
| 10,000 concurrent requests | Fails | 10,000 virtual threads | Works! |
| Context switch overhead | 1μs | 0.1μs | 10x faster |

**RestClient vs RestTemplate:**

| Metric | RestTemplate | RestClient | Improvement |
|--------|--------------|------------|-------------|
| API Fluency | Moderate | Excellent | Better DX |
| Error Handling | Basic | Advanced | More robust |
| Performance | Baseline | +5-10% | Slight improvement |
| Code Lines | 10-15 | 5-8 | 40% less code |

**Spring Boot 3.5.x Startup Time:**

| Deployment | Startup Time | Memory | Use Case |
|------------|--------------|--------|----------|
| JVM (Traditional) | 3-5 seconds | 256-512MB | Development, long-running services |
| JVM (Virtual Threads) | 3-5 seconds | 256-512MB | High-concurrency services |
| Native Image | 50-100ms | 20-50MB | Serverless, microservices |
| Native Image (AOT) | 30-50ms | 15-30MB | Edge computing, Lambda |

---

## Security Considerations

### Common Security Issues in AI-Generated Code

#### Issue 1: SQL Injection

**AI Generated (Vulnerable):**
```java
// ❌ SQL Injection vulnerability
public User findByUsername(String username) {
    String sql = "SELECT * FROM users WHERE username = '" + username + "'";
    return jdbcTemplate.queryForObject(sql, new UserRowMapper());
}
```

**Secure Version:**
```java
// ✅ Parameterized query
public User findByUsername(String username) {
    String sql = "SELECT * FROM users WHERE username = ?";
    return jdbcTemplate.queryForObject(sql, new UserRowMapper(), username);
}
```

#### Issue 2: Missing Input Validation

**AI Generated (Vulnerable):**
```java
// ❌ No validation
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userRepository.save(user);
}
```

**Secure Version:**
```java
// ✅ With validation
@PostMapping("/users")
public ResponseEntity<User> createUser(
    @Valid @RequestBody CreateUserRequest request
) {
    User user = new User();
    user.setUsername(request.username());
    user.setEmail(request.email());
    // ... mapping
    
    return ResponseEntity.ok(userRepository.save(user));
}

// DTO with validation using record (Java 16+)
public record CreateUserRequest(
    @NotBlank @Size(min = 3, max = 50) String username,
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password
) {}
```

#### Issue 3: Hardcoded Credentials

**AI Generated (Vulnerable):**
```java
// ❌ Hardcoded credentials
@Service
public class EmailService {
    private final String apiKey = "sk-1234567890abcdef";  // Exposed!
    
    public void sendEmail(String to, String subject, String body) {
        // Use apiKey
    }
}
```

**Secure Version:**
```java
// ✅ Externalized configuration
@Service
public class EmailService {
    private final String apiKey;
    
    public EmailService(@Value("${email.api.key}") String apiKey) {
        this.apiKey = apiKey;
    }
    
    public void sendEmail(String to, String subject, String body) {
        // Use apiKey from environment variable or config server
    }
}
```

**application.yml:**
```yaml
email:
  api:
    key: ${EMAIL_API_KEY}  # From environment variable
```

#### Issue 4: Missing Authentication/Authorization

**AI Generated (Vulnerable):**
```java
// ❌ No authentication
@DeleteMapping("/users/{id}")
public void deleteUser(@PathVariable Long id) {
    userRepository.deleteById(id);
}
```

**Secure Version:**
```java
// ✅ With authentication and authorization
@DeleteMapping("/users/{id}")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<Void> deleteUser(
    @PathVariable Long id,
    Authentication authentication
) {
    User currentUser = (User) authentication.getPrincipal();
    
    // Check if user is deleting themselves or has admin role
    if (!currentUser.getRole().equals(Role.ADMIN) && 
        !currentUser.getId().equals(id)) {
        throw new AccessDeniedException("Cannot delete other users");
    }
    
    userRepository.deleteById(id);
    return ResponseEntity.noContent().build();
}
```

### Security Checklist for AI-Generated Code

```markdown
## Security Review Checklist

### Authentication & Authorization
- [ ] Authentication required for protected endpoints
- [ ] Authorization checks present (RBAC/ABAC)
- [ ] JWT tokens validated properly
- [ ] Session management secure
- [ ] Password hashing (BCrypt, Argon2)

### Input Validation
- [ ] All inputs validated
- [ ] SQL injection prevented (parameterized queries)
- [ ] XSS prevented (output encoding)
- [ ] CSRF protection enabled
- [ ] File upload validation

### Data Protection
- [ ] Sensitive data encrypted at rest
- [ ] HTTPS enforced
- [ ] Secrets in environment variables
- [ ] No hardcoded credentials
- [ ] PII data protected

### API Security
- [ ] Rate limiting implemented
- [ ] CORS configured properly
- [ ] API versioning
- [ ] Input size limits
- [ ] Timeout configuration

### Logging & Monitoring
- [ ] Security events logged
- [ ] No sensitive data in logs
- [ ] Audit trail for critical operations
- [ ] Alerting configured
```

---

## Testing Strategies

### Testing AI-Generated Spring Boot 3.5.x Code

#### Unit Testing with Virtual Threads

**AI Generated Code:**
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final RestClient restClient;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
        this.restClient = RestClient.builder()
            .baseUrl("https://api.example.com")
            .build();
    }
    
    @Async
    public CompletableFuture<User> createUser(CreateUserRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            if (userRepository.existsByEmail(request.email())) {
                throw new UserAlreadyExistsException(request.email());
            }
            
            User user = new User();
            user.setEmail(request.email());
            user.setName(request.name());
            
            return userRepository.save(user);
        });
    }
}
```

**Unit Test:**
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void createUser_Success() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "John Doe",
            "password123"
        );
        
        when(userRepository.existsByEmail("john@example.com"))
            .thenReturn(false);
        
        User savedUser = new User();
        savedUser.setId(1L);
        savedUser.setEmail("john@example.com");
        
        when(userRepository.save(any(User.class)))
            .thenReturn(savedUser);
        
        // Act
        CompletableFuture<User> future = userService.createUser(request);
        User result = future.get(); // Wait for virtual thread to complete
        
        // Assert
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("john@example.com", result.getEmail());
        
        verify(userRepository).existsByEmail("john@example.com");
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void createUser_AlreadyExists_ThrowsException() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "john@example.com",
            "John Doe",
            "password123"
        );
        
        when(userRepository.existsByEmail("john@example.com"))
            .thenReturn(true);
        
        // Act & Assert
        CompletableFuture<User> future = userService.createUser(request);
        
        assertThrows(ExecutionException.class, () -> {
            future.get();
        });
        
        verify(userRepository, never()).save(any());
    }
}
```

#### Integration Testing with Virtual Threads

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void createUser_ValidRequest_Returns201() throws Exception {
        // Arrange
        String requestJson = """
            {
                "email": "john@example.com",
                "name": "John Doe",
                "password": "SecurePass123!"
            }
            """;
        
        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
    
    @Test
    void createUser_InvalidEmail_Returns400() throws Exception {
        // Arrange
        String requestJson = """
            {
                "email": "invalid-email",
                "name": "John Doe",
                "password": "SecurePass123!"
            }
            """;
        
        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").exists());
    }
    
    @Test
    void createUser_HighConcurrency_AllSucceed() throws Exception {
        // Test with 1000 concurrent requests
        int concurrentRequests = 1000;
        CountDownLatch latch = new CountDownLatch(concurrentRequests);
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicInteger errorCount = new AtomicInteger(0);
        
        for (int i = 0; i < concurrentRequests; i++) {
            String email = "user" + i + "@example.com";
            String requestJson = String.format("""
                {
                    "email": "%s",
                    "name": "User %d",
                    "password": "SecurePass123!"
                }
                """, email, i);
            
            CompletableFuture.runAsync(() -> {
                try {
                    mockMvc.perform(post("/api/v1/users")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(requestJson))
                        .andExpect(status().isCreated());
                    successCount.incrementAndGet();
                } catch (Exception e) {
                    errorCount.incrementAndGet();
                } finally {
                    latch.countDown();
                }
            });
        }
        
        latch.await(10, TimeUnit.SECONDS);
        
        assertEquals(concurrentRequests, successCount.get());
        assertEquals(0, errorCount.get());
    }
}
```

### Testing Checklist for AI-Generated Code

```markdown
## Testing Requirements

### Unit Tests
- [ ] All public methods tested
- [ ] Happy path covered
- [ ] Error cases covered
- [ ] Edge cases covered
- [ ] Mock external dependencies
- [ ] Test coverage > 80%
- [ ] Virtual thread behavior tested

### Integration Tests
- [ ] API endpoints tested
- [ ] Database integration tested
- [ ] External service integration tested
- [ ] Error scenarios tested
- [ ] Security scenarios tested
- [ ] High concurrency tested (virtual threads)

### Test Quality
- [ ] Tests are independent
- [ ] Tests are repeatable
- [ ] Tests are fast
- [ ] Tests are readable
- [ ] Tests are maintainable
```

---

## Migration Guide

### Spring Boot 3.0/3.x to 3.5.x Migration

#### Pre-Migration Checklist

```markdown
## Before You Start

- [ ] Backup current codebase
- [ ] Document current dependencies
- [ ] List all custom configurations
- [ ] Identify third-party libraries
- [ ] Check library compatibility with Spring Boot 3.5.x
- [ ] Ensure JDK 21 installed
- [ ] Create migration branch
- [ ] Review Spring Boot 3.5.x release notes
```

#### Step-by-Step Migration

**Step 1: Update Spring Boot Version**

```xml
<!-- pom.xml -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.0</version>  <!-- Updated from 3.0.x/3.x -->
</parent>

<properties>
    <java.version>21</java.version>  <!-- Updated from 17 -->
</properties>
```

**Step 2: Update Dependencies**

```xml
<dependencies>
    <!-- Spring Boot 3.5.x starters -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- For virtual threads and AOT -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aot</artifactId>
    </dependency>
    
    <!-- Jakarta EE (replaces javax) -->
    <dependency>
        <groupId>jakarta.persistence</groupId>
        <artifactId>jakarta.persistence-api</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
```

**Step 3: Migrate Package Imports**

Use AI to help with bulk migration:

```markdown
Prompt: "Migrate all javax.* imports to jakarta.* in this project:

[PASTE FILES OR USE GREP OUTPUT]

Rules:
1. javax.persistence.* → jakarta.persistence.*
2. javax.validation.* → jakarta.validation.*
3. javax.servlet.* → jakarta.servlet.*
4. javax.annotation.* → jakarta.annotation.*
5. Keep track of all changes made
6. Update to Java 21 features where appropriate (records, virtual threads)"
```

**Step 4: Update Configuration Files**

```yaml
# application.yml (Spring Boot 3.5.x)
spring:
  application:
    name: myapp
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: postgres
    hikari:
      maximum-pool-size: 20
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
    open-in-view: false

server:
  port: 8080
  servlet:
    context-path: /api

# Enable virtual threads
spring:
  threads:
    virtual:
      enabled: true

# Logging
logging:
  level:
    com.example: DEBUG
    org.hibernate.SQL: DEBUG
```

**Step 5: Update Code for Java 21 Features**

```java
// Before (Spring Boot 3.0)
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    
    // Getters, setters, constructor, toString, equals, hashCode
}

// After (Spring Boot 3.5.x with Java 21)
public record UserDTO(Long id, String name, String email) {}
```

**Step 6: Add Virtual Threads**

```java
// Before
@Configuration
public class AsyncConfig {
    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(20);
        executor.setMaxPoolSize(100);
        executor.setQueueCapacity(200);
        executor.initialize();
        return executor;
    }
}

// After (Spring Boot 3.5.x with Java 21)
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public AsyncTaskExecutor taskExecutor() {
        return ThreadPerTaskTaskExecutor.builder()
            .threadNamePrefix("virtual-")
            .virtualThreads(true)
            .build();
    }
}
```

**Step 7: Test Thoroughly**

```bash
# Compile with JDK 21
./mvnw clean compile

# Run tests
./mvnw test

# Run application
./mvnw spring-boot:run

# Check for errors in logs
```

**Step 8: Update Dockerfile (if using native images)**

```dockerfile
# Spring Boot 3.5.x with native image
FROM ghcr.io/graalvm/native-image-community:21-muslib AS builder
WORKDIR /app
COPY . .
RUN native-image -cp target/classes ...

FROM alpine:latest
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

#### Migration Troubleshooting

**Common Issues:**

1. **Compilation Errors:**
   ```
   Error: package javax.persistence does not exist
   ```
   **Solution:** Change all `javax.*` to `jakarta.*`

2. **Deprecation Warnings:**
   ```
   Warning: WebMvcConfigurerAdapter is deprecated
   ```
   **Solution:** Implement `WebMvcConfigurer` interface instead

3. **Java Version Issues:**
   ```
   Error: Java 21 required
   ```
   **Solution:** Update JDK to version 21

4. **Virtual Thread Configuration:**
   ```
   Warning: Virtual threads not enabled
   ```
   **Solution:** Add `spring.threads.virtual.enabled=true` to application.properties

5. **Third-Party Library Incompatibility:**
   ```
   Error: ClassNotFoundException for some library
   ```
   **Solution:** Update library to version compatible with Spring Boot 3.5.x and Jakarta

---

## Summary & Key Takeaways

### 10 Key Insights

1. **AI is a tool, not a replacement.** You still need to understand Spring Boot to use AI effectively.

2. **AI won't use features you don't know exist.** Build a capability map to know what's available in Spring Boot 3.5.x.

3. **AI confidently makes stuff up.** Always review AI-generated code for version mismatches and deprecated APIs.

4. **Architecture decisions are yours.** AI can suggest, but you must decide based on your context.

5. **Learning design thinking is your moat.** Understanding *why* features exist is more important than memorizing syntax.

6. **Foundation first, AI second.** Don't rely on AI before you have solid fundamentals.

7. **Be specific in prompts.** Always specify Spring Boot version AND Java version.

8. **Breaking changes are high-ROI learning.** Knowing them saves hours of debugging.

9. **You're the product manager.** AI is your implementer. You define requirements and constraints.

10. **The bar is higher, not lower.** AI raises the importance of technical judgment and decision-making.

### Action Items

**This Week:**
- [ ] Create your Spring Boot 3.5.x capability map
- [ ] List all breaking changes from 2.x/3.0 to 3.5.x
- [ ] Try one new Spring Boot 3.5.x feature (Virtual Threads, RestClient, etc.)
- [ ] Install JDK 21 if not already installed

**This Month:**
- [ ] Build a small project using only modern features (Spring Boot 3.5.x + Java 21)
- [ ] Create AI prompt templates for common tasks
- [ ] Set up code review checklist for AI-generated code
- [ ] Experiment with Virtual Threads and Structured Concurrency

**This Quarter:**
- [ ] Migrate one project to Spring Boot 3.5.x with JDK 21
- [ ] Share knowledge with team
- [ ] Experiment with AOT native images
- [ ] Try Scoped Values in a real project

### The Bottom Line

> **"The more powerful the tool, the more valuable human judgment becomes."**

AI has freed us from the drudgery of writing boilerplate code over and over. But in return, it actually raises the bar for developers' technical vision and judgment.

Before, you learned Spring Boot's new features so you could write every line yourself. Now, you learn them so you can direct AI to write better code, verify that AI got it right, and pick the right technical direction for your projects.

**Typing speed doesn't matter as much anymore. Making the right choices and thinking deeply is what sets people apart.**

With Spring Boot 3.5.x and Java 21, you have powerful new tools at your disposal: Virtual Threads for unprecedented concurrency, Structured Concurrency for better error handling, Scoped Values for modern context propagation, and RestClient for cleaner HTTP calls. Learn these features, build your capability map, and leverage AI to become a more effective developer.

---

## Further Reading & Resources

### Official Documentation
- [Spring Boot 3.5 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes)
- [Spring Boot 3.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- [Spring Boot 3.5.x Documentation](https://docs.spring.io/spring-boot/docs/3.5.x/reference/html/)
- [Project Loom Documentation](https://openjdk.org/projects/loom/)
- [Java 21 Documentation](https://docs.oracle.com/en/java/javase/21/)

### Books
- "Spring Boot in Practice" by Spring Team
- "Reactive Spring" by Josh Long
- "Cloud Native Java" by Josh Long and Kenny Bastani
- "Java 21: The Complete Guide" by various authors

### Blogs & Articles
- [Spring Blog](https://spring.io/blog)
- [Baeldung Spring Boot Tutorials](https://www.baeldung.com/spring-boot)
- [Spring Boot 3.5.x Features Overview](https://www.infoq.com/articles/spring-boot-3-5/)
- [Java 21 Virtual Threads Guide](https://www.baeldung.com/java-virtual-threads)

### Video Courses
- [Spring Academy](https://spring.academy/)
- [Udemy: Spring Boot 3.5 Masterclass](https://www.udemy.com/course/spring-boot-3-masterclass)
- [Java 21 Virtual Threads Course](https://www.baeldung.com/courses/java-virtual-threads)

### Community
- [Spring Community](https://spring.io/community)
- [Stack Overflow - Spring Boot](https://stackoverflow.com/questions/tagged/spring-boot)
- [GitHub - Spring Boot](https://github.com/spring-projects/spring-boot)
- [Reddit - r/java](https://www.reddit.com/r/java/)

### Tools
- [Spring Initializr](https://start.spring.io/)
- [Spring Boot DevTools](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [GraalVM](https://www.graalvm.org/)
- [Docker Compose](https://docs.docker.com/compose/)

---

## Practice Exercises

### Exercise 1: Identify AI-Generated Code Issues

**Difficulty:** Intermediate  
**Time:** 20 minutes

#### Scenario
You asked AI to generate a Spring Boot 3.5.x REST controller. Review the following code and identify all issues:

```java
import javax.validation.Valid;
import javax.validation.constraints.NotBlank;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final RestTemplate restTemplate = new RestTemplate();
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return restTemplate.getForObject(
            "https://api.example.com/users/{id}",
            User.class,
            id
        );
    }
    
    @PostMapping
    public User createUser(@Valid @RequestBody User user) {
        return userRepository.save(user);
    }
}

@Configuration
class CorsConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**");
    }
}
```

#### Task
Identify at least 5 issues with this code.

#### Solution

<details>
<summary>Click to reveal solution</summary>

**Issues Identified:**

1. **❌ Wrong namespace:** Uses `javax.validation.*` instead of `jakarta.validation.*`
   ```java
   // Wrong
   import javax.validation.Valid;
   import javax.validation.constraints.NotBlank;
   
   // Correct
   import jakarta.validation.Valid;
   import jakarta.validation.constraints.NotBlank;
   ```

2. **❌ Deprecated class:** Uses `WebMvcConfigurerAdapter` (deprecated since Spring 5)
   ```java
   // Wrong
   public class CorsConfig extends WebMvcConfigurerAdapter
   
   // Correct
   public class CorsConfig implements WebMvcConfigurer
   ```

3. **❌ Deprecated HTTP client:** Uses `RestTemplate` instead of `RestClient`
   ```java
   // Wrong
   private final RestTemplate restTemplate = new RestTemplate();
   
   // Correct
   private final RestClient restClient = RestClient.builder()
       .baseUrl("https://api.example.com")
       .build();
   ```

4. **❌ No error handling:** Missing try-catch and proper error responses

5. **❌ No timeout configuration:** HTTP calls have no timeouts

6. **❌ No logging:** No logging for debugging

7. **❌ Missing @Service annotation:** `userRepository` is used but not injected

8. **❌ Not using Java 21 features:** Should use records for DTOs, virtual threads for async operations

**Corrected Code:**
```java
import jakarta.validation.Valid;
import jakarta.validation.constraints.NotBlank;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.client.RestClient;
import org.springframework.http.*;
import org.slf4j.*;
import org.springframework.stereotype.Service;
import java.util.concurrent.*;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    private final RestClient restClient;
    private final UserRepository userRepository;
    
    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
        this.restClient = RestClient.builder()
            .baseUrl("https://api.example.com")
            .build();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        logger.info("Fetching user with id: {}", id);
        
        try {
            User user = restClient.get()
                .uri("/users/{id}", id)
                .retrieve()
                .onStatus(HttpStatusCode::is4xxClientError, 
                    (req, res) -> throw new UserNotFoundException(id))
                .body(User.class);
            
            return ResponseEntity.ok(user);
            
        } catch (Exception e) {
            logger.error("Failed to fetch user: {}", id, e);
            return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                .build();
        }
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        logger.info("Creating user: {}", request.email());
        
        User user = new User();
        user.setEmail(request.email());
        user.setName(request.name());
        
        User saved = userRepository.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
}

@Configuration
class CorsConfig implements WebMvcConfigurer {  // ✅ Fixed
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}

// ✅ Using record for DTO (Java 16+)
public record CreateUserRequest(
    @NotBlank @Email String email,
    @NotBlank String name
) {}
```

</details>

---

### Exercise 2: Build Your Capability Map

**Difficulty:** Intermediate  
**Time:** 30 minutes

#### Scenario
Create a comprehensive capability map for Spring Boot 3.5.x features.

#### Task
Create a markdown file documenting:
1. At least 10 major features
2. What each feature does
3. When to use it
4. Common pitfalls
5. Code example for each

#### Solution

<details>
<summary>Click to reveal solution</summary>

**capability-map.md:**
```markdown
# My Spring Boot 3.5.x Capability Map

## 1. Virtual Threads (Java 21)

**What It Is:** Lightweight threads for high-concurrency applications

**What It Does:** Enables 10-100x more concurrent requests with less memory

**When to Use:**
- High-concurrency REST APIs
- I/O-bound microservices
- Applications with many concurrent users

**Code Example:**
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public AsyncTaskExecutor taskExecutor() {
        return ThreadPerTaskTaskExecutor.builder()
            .virtualThreads(true)
            .threadNamePrefix("virtual-")
            .build();
    }
}
```

**Gotchas:**
- ThreadLocal doesn't work as expected (use Scoped Values)
- Blocking operations still block virtual threads
- Not suitable for CPU-bound tasks

---

## 2. RestClient

**What It Is:** Modern HTTP client for Spring Boot 3.2+

**What It Does:** Provides fluent API for making HTTP calls

**When to Use:**
- Calling external APIs
- Replacing RestTemplate
- Synchronous HTTP operations

**Code Example:**
```java
RestClient client = RestClient.builder()
    .baseUrl("https://api.example.com")
    .build();

User user = client.get()
    .uri("/users/{id}", 1)
    .retrieve()
    .body(User.class);
```

**Gotchas:**
- Only for synchronous operations (use WebClient for reactive)
- Requires Spring Boot 3.2+

---

## 3. Docker Compose Support

**What It Is:** Auto-start Docker containers during development

**What It Does:** Eliminates manual container management

**When to Use:**
- Local development
- Integration testing
- Onboarding new developers

**Configuration:**
```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
```

```properties
# application.properties
spring.docker.compose.enabled=true
```

**Gotchas:**
- Only for development profile
- Don't use in production

---

## 4. AOT Native Images

**What It Is:** Compile Spring Boot apps to native executables

**What It Does:** Sub-second startup, minimal memory usage

**When to Use:**
- Serverless functions
- Microservices needing fast startup
- Resource-constrained environments

**Build Command:**
```bash
./mvnw spring-boot:build-image
```

**Gotchas:**
- Requires GraalVM
- More configuration needed
- Not all libraries supported

---

## 5. Jakarta EE Migration

**What It Is:** Migration from javax.* to jakarta.* namespace

**What It Does:** Modern namespace for Java EE APIs

**When to Use:**
- Always (required in Spring Boot 3.x)

**Changes:**
- javax.persistence.* → jakarta.persistence.*
- javax.validation.* → jakarta.validation.*
- javax.servlet.* → jakarta.servlet.*

**Gotchas:**
- All libraries must be compatible
- IDE may show errors until fully migrated

---

## 6. Structured Concurrency (Java 21)

**What It Is:** Manage multiple concurrent tasks as a single unit

**What It Does:** Simplifies error handling for parallel operations

**When to Use:**
- Multiple parallel API calls
- Concurrent database operations
- Any scenario with independent concurrent tasks

**Code Example:**
```java
try (var scope = StructuredTaskScope.shutdownOnFailure()) {
    Future<DataA> futureA = scope.fork(() -> serviceA.getData());
    Future<DataB> futureB = scope.fork(() -> serviceB.getData());
    
    scope.join();
    scope.throwIfFailed();
    
    return new Result(futureA.get(), futureB.get());
}
```

**Gotchas:**
- Only available in Java 21+
- All tasks succeed or all fail

---

## 7. Scoped Values (Java 21)

**What It Is:** Modern alternative to ThreadLocal

**What It Does:** Context propagation that works with virtual threads

**When to Use:**
- Request context propagation
- Security context sharing
- Any scenario where you used ThreadLocal before

**Code Example:**
```java
private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

try (var scope = ScopedValue.where(REQUEST_ID, "123")) {
    // REQUEST_ID.get() returns "123"
    processRequest();
}
```

**Gotchas:**
- Only available in Java 21+
- Immutable by design
- Works correctly with virtual threads

---

## 8. Configuration Properties with Records

**What It Is:** Type-safe configuration using Java records

**What It Does:** Simplifies configuration with immutable data classes

**When to Use:**
- Type-safe configuration
- Complex configuration structures

**Code Example:**
```java
@ConfigurationProperties(prefix = "app")
public record AppConfig(
    String name,
    int version,
    DatabaseConfig database,
    SecurityConfig security
) {
    public record DatabaseConfig(
        String url,
        String username,
        int poolSize
    ) {}
}
```

**Gotchas:**
- Requires Java 16+ (records)
- Must be used with @ConfigurationPropertiesScan

---

## 9. Observability with Micrometer

**What It Is:** Enhanced metrics and tracing

**What It Does:** Better monitoring and debugging

**When to Use:**
- Production applications
- Microservices
- Performance monitoring

**Code Example:**
```java
@Service
public class UserService {
    private final Counter userCreatedCounter;
    
    public UserService(MeterRegistry registry) {
        this.userCreatedCounter = Counter.builder("users.created")
            .description("Number of users created")
            .register(registry);
    }
    
    public User createUser(User user) {
        User saved = userRepository.save(user);
        userCreatedCounter.increment();
        return saved;
    }
}
```

**Gotchas:**
- Requires actuator dependency
- Consider performance impact of metrics

---

## 10. Health Indicators

**What It Is:** Built-in health checks

**What It Does:** Monitor application and dependency health

**When to Use:**
- Production applications
- Kubernetes liveness/readiness probes
- Monitoring dashboards

**Code Example:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

**Gotchas:**
- Don't expose sensitive health details
- Secure actuator endpoints in production
```

</details>

---

### Exercise 3: Migrate a Spring Boot 3.0 Application to 3.5.x with Java 21

**Difficulty:** Advanced  
**Time:** 45 minutes

#### Scenario
You have a Spring Boot 3.0 application that needs to be migrated to 3.5.x with Java 21 features.

**Original Code:**
```java
// UserController.java
import javax.validation.Valid;
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.client.RestTemplate;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final RestTemplate restTemplate = new RestTemplate();
    private final UserRepository userRepository;
    
    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    @PostMapping
    public User createUser(@Valid @RequestBody User user) {
        if (userRepository.existsByEmail(user.getEmail())) {
            throw new UserAlreadyExistsException(user.getEmail());
        }
        return userRepository.save(user);
    }
}

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    @Email
    @Column(unique = true)
    private String email;
    
    @NotBlank
    private String name;
    
    // Getters and setters
}

@Configuration
class CorsConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}
```

**pom.xml:**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.0</version>
</parent>

<properties>
    <java.version>17</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

#### Task
Migrate this application to Spring Boot 3.5.x with Java 21.

#### Solution

<details>
<summary>Click to reveal solution</summary>

**Step 1: Update pom.xml**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.0</version>  <!-- Updated -->
</parent>

<properties>
    <java.version>21</java.version>  <!-- Updated from 17 -->
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>  <!-- Added explicitly -->
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aot</artifactId>  <!-- For AOT compilation -->
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

**Step 2: Migrate UserController.java**

```java
import jakarta.validation.Valid;  // ✅ Changed from javax
import jakarta.validation.constraints.Email;  // ✅ Changed
import jakarta.validation.constraints.NotBlank;  // ✅ Changed
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;  // ✅ Changed
import org.springframework.web.client.RestClient;  // ✅ Changed from RestTemplate
import org.springframework.http.*;
import org.slf4j.*;
import org.springframework.stereotype.Service;
import java.util.concurrent.*;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    private final RestClient restClient;  // ✅ Changed
    private final UserRepository userRepository;
    
    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
        this.restClient = RestClient.builder()  // ✅ Changed
            .baseUrl("https://api.example.com")
            .build();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        logger.info("Fetching user: {}", id);
        
        return userRepository.findById(id)
            .map(ResponseEntity::ok)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        logger.info("Creating user: {}", request.email());
        
        if (userRepository.existsByEmail(request.email())) {
            throw new UserAlreadyExistsException(request.email());
        }
        
        User user = new User();
        user.setEmail(request.email());
        user.setName(request.name());
        
        User saved = userRepository.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
}

// ✅ Using record for DTO (Java 16+)
public record CreateUserRequest(
    @NotBlank @Email String email,
    @NotBlank String name
) {}
```

**Step 3: Migrate User Entity**

```java
import jakarta.persistence.*;  // ✅ Changed from javax
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    @Email
    @Column(unique = true, nullable = false)
    private String email;
    
    @NotBlank
    @Column(nullable = false)
    private String name;
    
    // Constructors
    protected User() {}  // JPA requirement
    
    public User(String email, String name) {
        this.email = email;
        this.name = name;
    }
    
    // Getters
    public Long getId() { return id; }
    public String getEmail() { return email; }
    public String getName() { return name; }
    
    // Setters (if needed, or use records)
    public void setEmail(String email) { this.email = email; }
    public void setName(String name) { this.name = name; }
}
```

**Step 4: Migrate CorsConfig**

```java
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;  // ✅ Changed
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.context.annotation.Configuration;

@Configuration  // ✅ Added
public class CorsConfig implements WebMvcConfigurer {  // ✅ Changed from extends
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

**Step 5: Update application.properties**

```properties
# application.properties
spring:
  application:
    name: user-service
  datasource:
    url: jdbc:postgresql://localhost:5432/userdb
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
    open-in-view: false  # ✅ Recommended for Spring Boot 3.x
  
  # Enable virtual threads
  threads:
    virtual:
      enabled: true

server:
  port: 8080
  servlet:
    context-path: /api

# Logging
logging:
  level:
    com.example: DEBUG
    org.hibernate.SQL: DEBUG
```

**Step 6: Add Exception Handler**

```java
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;
import java.util.*;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "User not found",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(UserAlreadyExistsException.class)
    public ResponseEntity<ErrorResponse> handleUserAlreadyExists(UserAlreadyExistsException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            "User already exists",
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.CONFLICT);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .toList();
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            String.join(", ", errors),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            "An unexpected error occurred",
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}

// ✅ Using record for error response (Java 16+)
record ErrorResponse(
    int status,
    String error,
    String message,
    long timestamp
) {}
```

**Step 7: Add Virtual Threads Support**

```java
// ✅ New: AsyncConfig with virtual threads
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public AsyncTaskExecutor taskExecutor() {
        return ThreadPerTaskTaskExecutor.builder()
            .threadNamePrefix("virtual-")
            .virtualThreads(true)  // Enable virtual threads
            .build();
    }
}

// Update service to use virtual threads
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Async
    public CompletableFuture<User> fetchUser(Long userId) {
        return CompletableFuture.supplyAsync(() -> {
            // Runs on virtual thread
            return userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException(userId));
        });
    }
}
```

**Step 8: Test the Migration**

```bash
# Compile with JDK 21
./mvnw clean compile

# Run tests
./mvnw test

# Run application
./mvnw spring-boot:run

# Test endpoints
curl http://localhost:8080/api/users/1
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"Test User"}'
```

**Migration Summary:**

✅ Updated Spring Boot version: 3.0.0 → 3.5.0  
✅ Updated JDK version: 17 → 21  
✅ Migrated javax.* → jakarta.*  
✅ Replaced WebMvcConfigurerAdapter → WebMvcConfigurer  
✅ Replaced RestTemplate → RestClient  
✅ Added proper error handling  
✅ Added logging  
✅ Improved code structure  
✅ Added DTOs with validation using records  
✅ Added global exception handler  
✅ Added virtual threads support  
✅ Enabled Java 21 features

</details>

---

## Question Bank

### Beginner Level (15 Questions)

1. **What is Spring Boot?**
   - A framework for building stand-alone, production-grade Spring applications
   - A database ORM tool
   - A frontend framework
   - A testing library
   
   **Answer:** A framework for building stand-alone, production-grade Spring applications

2. **What does @RestController do?**
   - Marks a class as a REST controller
   - Combines @Controller and @ResponseBody
   - Both A and B
   - None of the above
   
   **Answer:** Both A and B

3. **What is the minimum JDK version for Spring Boot 3.5.x?**
   - JDK 8
   - JDK 11
   - JDK 17
   - JDK 21 (recommended for Virtual Threads)
   
   **Answer:** JDK 17 (minimum), JDK 21 (recommended for Virtual Threads)

4. **What namespace does Spring Boot 3.5.x use?**
   - javax.*
   - jakarta.*
   - Both
   - Neither
   
   **Answer:** jakarta.*

5. **What is RestClient?**
   - A deprecated HTTP client
   - A modern HTTP client in Spring Boot 3.2+
   - A database client
   - A messaging client
   
   **Answer:** A modern HTTP client in Spring Boot 3.2+

6. **What are Virtual Threads?**
   - Traditional OS threads
   - Lightweight threads from Project Loom (Java 21)
   - Database threads
   - HTTP client threads
   
   **Answer:** Lightweight threads from Project Loom (Java 21)

7. **What is Docker Compose support in Spring Boot 3.1+?**
   - Building Docker images
   - Auto-starting containers for local development
   - Deploying to Kubernetes
   - Managing Docker registries
   
   **Answer:** Auto-starting containers for local development

8. **What is AOT compilation?**
   - Just-In-Time compilation
   - Ahead-Of-Time compilation to native images
   - Dynamic compilation
   - No compilation
   
   **Answer:** Ahead-Of-Time compilation to native images

9. **What replaced WebMvcConfigurerAdapter?**
   - WebMvcConfigurer (interface)
   - WebMvcConfigurerAdapter2
   - WebMvcConfig
   - Nothing, it's still used
   
   **Answer:** WebMvcConfigurer (interface)

10. **What is the purpose of @ConfigurationProperties?**
    - Defining beans
    - Type-safe configuration properties
    - Component scanning
    - Auto-configuration
   
    **Answer:** Type-safe configuration properties

11. **What is Spring Boot Actuator?**
    - A build tool
    - Production-ready features (monitoring, metrics, health checks)
    - A testing framework
    - A security library
   
    **Answer:** Production-ready features (monitoring, metrics, health checks)

12. **What is Structured Concurrency?**
    - A design pattern
    - A Java 21 feature for managing multiple concurrent tasks
    - A Spring Boot feature
    - A database concept
   
    **Answer:** A Java 21 feature for managing multiple concurrent tasks

13. **What are Scoped Values in Java 21?**
    - A new type of thread
    - A modern alternative to ThreadLocal
    - A database feature
    - A Spring Boot annotation
   
    **Answer:** A modern alternative to ThreadLocal

14. **What is the benefit of using records in Java 21?**
    - Better performance
    - Less boilerplate code and immutability
    - Required for Spring Boot
    - No benefits
   
    **Answer:** Less boilerplate code and immutability

15. **What is the default embedded server in Spring Boot?**
    - Tomcat
    - Jetty
    - Undertow
    - Netty
   
    **Answer:** Tomcat

### Intermediate Level (20 Questions)

16. **Why is it important to learn Spring Boot features even with AI assistance?**
    - AI won't use features you don't know exist
    - AI confidently makes mistakes
    - Architecture decisions require human judgment
    - All of the above
   
    **Answer:** All of the above

17. **What is a "capability map"?**
    - A map of your team's skills
    - A mental/physical index of available technologies and their use cases
    - A Spring Boot configuration file
    - A deployment diagram
   
    **Answer:** A mental/physical index of available technologies and their use cases

18. **What is the main limitation of AI-generated code?**
    - It's always buggy
    - It can't make technical judgments
    - It's too slow
    - It's expensive
   
    **Answer:** It can't make technical judgments

19. **When should you use Virtual Threads?**
    - For CPU-intensive tasks
    - For I/O-bound high-concurrency tasks
    - For database queries only
    - Never, always use traditional threads
   
    **Answer:** For I/O-bound high-concurrency tasks

20. **What is the benefit of RestClient over RestTemplate?**
    - RestClient is deprecated
    - RestClient has a fluent API and better error handling
    - RestClient is faster
    - No difference
   
    **Answer:** RestClient has a fluent API and better error handling

21. **What is the Jakarta EE migration?**
    - Moving from Spring Boot 2.x to 3.x
    - Changing from javax.* to jakarta.* namespace
    - Upgrading JDK version
    - Migrating to microservices
   
    **Answer:** Changing from javax.* to jakarta.* namespace

22. **Why does AI generate outdated code?**
    - AI is intentionally malicious
    - Training data cutoff means it doesn't know latest features
    - AI prefers old code
    - It doesn't, it always generates modern code
   
    **Answer:** Training data cutoff means it doesn't know latest features

23. **What is the efficiency difference between developers who know breaking changes vs those who don't?**
    - 2x
    - 5x
    - 10x
    - No difference
   
    **Answer:** 10x

24. **What should beginners focus on before using AI extensively?**
    - Learning core Spring Boot concepts
    - Building projects without AI
    - Understanding annotations and configuration
    - All of the above
   
    **Answer:** All of the above

25. **What is the "Product Manager" mindset in AI-assisted development?**
    - Letting AI make all decisions
    - You define requirements and constraints, AI implements
    - Only using AI for testing
    - Avoiding AI completely
   
    **Answer:** You define requirements and constraints, AI implements

26. **What is the main advantage of Docker Compose support in Spring Boot 3.5.x?**
    - Faster Docker builds
    - Auto-starting containers for local development
    - Better security
    - Smaller images
   
    **Answer:** Auto-starting containers for local development

27. **What is AOT compilation used for?**
    - Improving development speed
    - Creating native images with fast startup
    - Better error messages
    - Automatic testing
   
    **Answer:** Creating native images with fast startup

28. **What is the main security risk with AI-generated code?**
    - It's always insecure
    - It may miss security best practices
    - It's too secure
    - No security risks
   
    **Answer:** It may miss security best practices

29. **What is the N+1 query problem?**
    - A performance issue where N+1 database queries are executed
    - A security vulnerability
    - A threading issue
    - A configuration problem
   
    **Answer:** A performance issue where N+1 database queries are executed

30. **Why is ThreadLocal problematic with Virtual Threads?**
    - Virtual threads unmount/mount, breaking ThreadLocal assumptions
    - ThreadLocal doesn't work at all
    - ThreadLocal is deprecated
    - No issues, works perfectly
   
    **Answer:** Virtual threads unmount/mount, breaking ThreadLocal assumptions

31. **What is Resilience4j used for?**
    - Database access
    - Fault tolerance (circuit breaker, retry, rate limiter)
    - Security
    - Testing
   
    **Answer:** Fault tolerance (circuit breaker, retry, rate limiter)

32. **What is the benefit of using records for DTOs in Spring Boot 3.5.x?**
    - Immutability and less boilerplate
    - Better performance
    - Required for validation
    - No benefits
   
    **Answer:** Immutability and less boilerplate

33. **What should you do before upgrading Spring Boot versions?**
    - Read migration guide
    - Test in isolation
    - Backup codebase
    - All of the above
   
    **Answer:** All of the above

34. **What is the main purpose of Spring Boot Actuator?**
    - Building applications
    - Monitoring and managing applications in production
    - Testing
    - Security
   
    **Answer:** Monitoring and managing applications in production

35. **What is the benefit of Structured Concurrency over CompletableFuture?**
    - No benefit, they're the same
    - Better error handling and cleaner code for multiple concurrent tasks
    - Faster execution
    - Uses less memory
   
    **Answer:** Better error handling and cleaner code for multiple concurrent tasks

### Advanced Level (15 Questions)

36. **Explain the difference between Virtual Threads and traditional threads in terms of memory usage.**
    - Virtual threads use the same memory
    - Virtual threads use significantly less memory (kilobytes vs megabytes per thread)
    - Virtual threads use more memory
    - No difference
   
    **Answer:** Virtual threads use significantly less memory (kilobytes vs megabytes per thread)

37. **What is the impact of AI on the software development job market?**
    - Developers will be replaced
    - The bar for technical judgment is raised
    - No impact
    - Only junior developers are affected
   
    **Answer:** The bar for technical judgment is raised

38. **Why is understanding design thinking important in the AI era?**
    - It's not important
    - AI can't replicate human judgment and architectural thinking
    - It's only for architects
    - It's outdated
   
    **Answer:** AI can't replicate human judgment and architectural thinking

39. **What is the compound effect of not learning new Spring Boot features?**
    - No effect
    - Accumulation of technical debt over time
    - Immediate project failure
    - Better job security
   
    **Answer:** Accumulation of technical debt over time

40. **How does AI training data cutoff affect code generation?**
    - No effect
    - AI may generate outdated code using old patterns
    - AI always generates the latest code
    - AI refuses to generate code
   
    **Answer:** AI may generate outdated code using old patterns

41. **What is the role of a developer in AI-assisted development?**
    - Code typist
    - Technical architect and code reviewer
    - AI trainer
    - Project manager
   
    **Answer:** Technical architect and code reviewer

42. **Why is maintaining a capability map important?**
    - It's not important
    - It helps you know what to ask AI for and spot AI errors
    - It's required by Spring
    - It helps with debugging only
   
    **Answer:** It helps you know what to ask AI for and spot AI errors

43. **What is the relationship between learning Spring Boot features and industry trends?**
    - No relationship
    - Spring Boot features track broader industry trends (cloud native, serverless, etc.)
    - Spring Boot ignores industry trends
    - Industry trends follow Spring Boot
   
    **Answer:** Spring Boot features track broader industry trends (cloud native, serverless, etc.)

44. **Explain why "typing speed doesn't matter as much anymore" in the context of AI-assisted development.**
    - Typing is still important
    - AI handles implementation, making technical judgment and decision-making more valuable
    - Typing speed is the most important skill
    - Developers don't need to type anymore
   
    **Answer:** AI handles implementation, making technical judgment and decision-making more valuable

45. **What is the danger of not knowing what features exist in new Spring Boot versions?**
    - No danger
    - You can't ask AI for modern solutions, so you get outdated code
    - AI will force you to use new features
    - Your code will be better
   
    **Answer:** You can't ask AI for modern solutions, so you get outdated code

46. **How does the "Product Manager" mindset improve AI-assisted development?**
    - It doesn't
    - By defining clear requirements and constraints, you get better AI output
    - It slows down development
    - It's only for managers
   
    **Answer:** By defining clear requirements and constraints, you get better AI output

47. **What is the value of learning breaking changes between Spring Boot versions?**
    - No value
    - It helps you avoid pitfalls and spot AI errors instantly
    - It's only for migration
    - It's outdated information
   
    **Answer:** It helps you avoid pitfalls and spot AI errors instantly

48. **Why can't AI replace production debugging?**
    - AI is not allowed to debug
    - Production issues are deeply tied to framework internals that require human understanding
    - AI is too slow
    - AI doesn't have access to production systems
   
    **Answer:** Production issues are deeply tied to framework internals that require human understanding

49. **What is the performance improvement of Virtual Threads over traditional threads for high-concurrency scenarios?**
    - 2x improvement
    - 10-100x improvement in concurrent request handling
    - No improvement
    - 1000x improvement
   
    **Answer:** 10-100x improvement in concurrent request handling

50. **How does Java 21 enhance Spring Boot 3.5.x development?**
    - It doesn't, they're independent
    - Java 21 provides Virtual Threads, Structured Concurrency, and Scoped Values that enhance Spring Boot
    - Java 21 is required but provides no benefits
    - Java 21 makes Spring Boot slower
   
    **Answer:** Java 21 provides Virtual Threads, Structured Concurrency, and Scoped Values that enhance Spring Boot

---

## Test Your Understanding

Test your knowledge with these 10 questions. Try to answer them without looking at the tutorial.

1. **What are the four main reasons you still need to learn Spring Boot features in the AI era?**

2. **Explain the difference between RestTemplate and RestClient. When would you use each?**

3. **What are Virtual Threads in Java 21 and what problem do they solve?**

4. **Why is it important to specify both Spring Boot and Java versions in your AI prompts?**

5. **What is a "capability map" and how do you build one?**

6. **List three common issues with AI-generated Spring Boot code.**

7. **What is the Jakarta EE migration and why does it matter?**

8. **Explain the "Product Manager" mindset for AI-assisted development.**

9. **What are the learning strategies for beginners vs experienced developers?**

10. **Why can't AI make architecture decisions for you?**

<details>
<summary>Click to reveal answers</summary>

**Answers:**

1. **Four reasons:**
   - AI won't use features you don't know exist
   - AI confidently makes stuff up (version mismatches, deprecated APIs)
   - Architecture decisions and production debugging require human judgment
   - Learning features means learning design thinking, which is your competitive advantage

2. **RestTemplate vs RestClient:**
   - RestTemplate: Legacy synchronous HTTP client, deprecated in Spring Boot 6.0
   - RestClient: Modern, fluent HTTP client introduced in Spring Boot 3.2
   - Use RestClient for new code, RestTemplate only for maintaining legacy code

3. **Virtual Threads:**
   - Lightweight threads from Project Loom (Java 21)
   - Enable 10-100x more concurrent requests
   - Use much less memory than traditional threads
   - Perfect for I/O-bound high-concurrency applications

4. **Specifying versions:**
   - AI training data has a cutoff date
   - Without version specification, AI may generate outdated code
   - Ensures AI uses modern features and correct namespaces
   - Specifying Java 21 ensures virtual threads and modern features are used
   - Eliminates 50% of AI errors

5. **Capability Map:**
   - A mental/physical index of available technologies
   - Build by scanning release notes quarterly
   - Document features, use cases, and gotchas
   - Helps you know what to ask AI for

6. **Three common issues:**
   - Version mismatches (javax vs jakarta)
   - Deprecated APIs (WebMvcConfigurerAdapter, RestTemplate)
   - Missing error handling and security considerations
   - Not using modern Java 21 features

7. **Jakarta EE migration:**
   - Spring Boot 3.x moved from javax.* to jakarta.* namespace
   - Required because Java EE moved to Jakarta EE
   - All imports must be updated
   - Critical for compilation in Spring Boot 3.5.x

8. **Product Manager mindset:**
   - You define requirements, constraints, and tech choices
   - AI implements the code
   - You review and validate
   - You make final decisions

9. **Learning strategies:**
   - Beginners: Foundation first, build projects without AI, then gradually introduce AI
   - Experienced: Scan release notes quarterly, build capability map, experiment with new features

10. **AI can't make architecture decisions because:**
    - AI doesn't know your specific context (team skills, budget, timeline, existing systems)
    - Architecture decisions require weighing trade-offs based on business needs
    - AI gives generic advice, not tailored to your situation

</details>

---

## Common Interview Questions

### Questions and Detailed Answers

1. **Q: With AI writing code, do developers still need to learn frameworks like Spring Boot?**

   **A:** Absolutely. AI is a powerful tool, but it has critical limitations:
   - AI won't use features you don't know exist
   - AI can confidently generate incorrect code (version mismatches, deprecated APIs)
   - Architecture decisions require understanding your specific context
   - Production debugging requires deep framework knowledge
   - Technical judgment and design thinking are human advantages
   
   The role has shifted from "code writer" to "technical architect and reviewer." You need to know enough to direct AI effectively and validate its output.

2. **Q: What are the major changes in Spring Boot 3.5.x?**

   **A:** Major changes include:
   - **Jakarta EE migration:** All javax.* packages moved to jakarta.*
   - **Minimum JDK 17:** JDK 21+ recommended for Virtual Threads
   - **Virtual Threads:** Native support via Project Loom (Java 21)
   - **RestClient:** Modern HTTP client replacing RestTemplate
   - **Docker Compose support:** Auto-start containers for local development
   - **AOT compilation:** Improved native image support with GraalVM
   - **Structured Concurrency:** Java 21 feature for better concurrent programming
   - **Scoped Values:** Java 21 alternative to ThreadLocal
   - **Observability:** Enhanced metrics and tracing with Micrometer

3. **Q: How do you ensure AI-generated Spring Boot code is correct?**

   **A:** Follow this process:
   1. **Specify version:** Always mention Spring Boot version AND Java version in prompts
   2. **Review imports:** Check for javax vs jakarta namespace
   3. **Check for deprecations:** Look for WebMvcConfigurerAdapter, RestTemplate, etc.
   4. **Validate error handling:** Ensure proper exception handling
   5. **Review security:** Check for input validation, SQL injection prevention
   6. **Test thoroughly:** Unit tests, integration tests, edge cases
   7. **Code review:** Have another developer review AI-generated code

4. **Q: What are Virtual Threads and when should you use them?**

   **A:** Virtual Threads are lightweight threads introduced in Java 21 (Project Loom):
   - **What:** Lightweight threads that are cheap to create (kilobytes vs megabytes)
   - **When to use:** I/O-bound high-concurrency applications (REST APIs, microservices)
   - **Benefits:** 10-100x more concurrent requests, less memory, simpler code
   - **Gotchas:** ThreadLocal doesn't work as expected (use Scoped Values), blocking operations still block
   - **Not for:** CPU-intensive tasks (use parallel streams instead)

5. **Q: Explain the difference between RestTemplate, WebClient, and RestClient.**

   **A:**
   - **RestTemplate:** Legacy synchronous HTTP client, deprecated in Spring Boot 6.0
   - **WebClient:** Reactive, non-blocking HTTP client (part of Spring WebFlux)
   - **RestClient:** Modern synchronous HTTP client introduced in Spring Boot 3.2
   
   **When to use:**
   - RestClient: New synchronous HTTP code (Spring Boot 3.2+)
   - WebClient: Reactive applications, streaming
   - RestTemplate: Only for maintaining legacy code

6. **Q: What is the Jakarta EE migration and why was it necessary?**

   **A:** The Jakarta EE migration changed all Java EE APIs from javax.* to jakarta.* namespace:
   - **Why:** Oracle transferred Java EE to Eclipse Foundation, which renamed it Jakarta EE
   - **Impact:** All imports changed (javax.persistence → jakarta.persistence, etc.)
   - **Breaking change:** Code using javax.* won't compile in Spring Boot 3.5.x
   - **Migration:** Find/replace all javax.* imports, update dependencies

7. **Q: How do you handle breaking changes when upgrading Spring Boot versions?**

   **A:**
   1. **Read migration guide:** Official Spring Boot documentation
   2. **Create checklist:** Document all breaking changes
   3. **Update dependencies:** Change version in pom.xml/build.gradle
   4. **Update code:** Fix deprecated APIs, change namespaces
   5. **Test thoroughly:** Run all tests, check for compilation errors
   6. **Use AI:** Prompt AI to help with bulk changes
   7. **Incremental approach:** Upgrade one module at a time

8. **Q: What is AOT compilation and what are its benefits?**

   **A:** AOT (Ahead-of-Time) compilation compiles Java code to native machine code before runtime:
   - **Benefits:**
     - Sub-second startup time (50-100ms vs 3-5 seconds)
     - Lower memory footprint (20-50MB vs 256-512MB)
     - No JVM required
   - **Use cases:** Serverless functions, microservices, resource-constrained environments
   - **Trade-offs:** Longer build time, not all libraries supported, requires GraalVM
   - **Spring Boot support:** spring-boot:build-image goal

9. **Q: What learning strategy would you recommend for a beginner learning Spring Boot in the AI era?**

   **A:**
   1. **Phase 1 - Foundation (Months 1-3):** Learn core concepts without AI
      - Build 3-5 projects from scratch
      - Understand annotations, DI, configuration
      - Learn common patterns
      - Master Java 21 basics
   2. **Phase 2 - AI Introduction (Months 3-6):** Use AI for boilerplate
      - Always read and understand AI code
      - Make simple modifications yourself
      - Learn to spot errors
   3. **Phase 3 - AI Amplification (Months 6+):** AI as a tool
      - Use AI for repetitive tasks
      - Focus on architecture
      - Review critically
   
   **Key principle:** Don't rely on AI before you have solid fundamentals.

10. **Q: What is Docker Compose support in Spring Boot 3.5.x and how does it help developers?**

    **A:** Docker Compose support automatically starts Docker containers during development:
    - **What it does:** Reads docker-compose.yml and starts services (PostgreSQL, Redis, etc.)
    - **Benefits:**
      - New developers can start with one command
      - No manual container management
      - Consistent development environment
      - Automatic cleanup on shutdown
    - **Configuration:**
      ```properties
      spring.docker.compose.enabled=true
      ```
    - **Limitation:** Development profile only, not for production

11. **Q: How do you build a capability map for Spring Boot?**

    **A:**
    1. **Scan release notes:** Review major version features quarterly
    2. **Document features:** For each feature, note:
       - What it does
       - When to use it
       - Common pitfalls
       - Code example
    3. **Track breaking changes:** Maintain checklist of deprecated APIs
    4. **Update regularly:** Add new features as you learn them
    5. **Use it:** Reference when solving problems or prompting AI

12. **Q: What are common pitfalls when using AI for Spring Boot development?**

    **A:**
    1. **Version mismatches:** AI generates javax.* code for Spring Boot 3.5.x
    2. **Deprecated APIs:** Uses WebMvcConfigurerAdapter, RestTemplate
    3. **Missing error handling:** No try-catch, no proper HTTP status codes
    4. **Security issues:** Missing input validation, SQL injection vulnerabilities
    5. **Performance problems:** N+1 queries, missing connection pooling
    6. **Vague prompts:** Getting generic code instead of production-ready code
    7. **No testing:** Assuming AI code is correct without testing
    8. **Over-reliance:** Copy-pasting without understanding
    9. **Not using Java 21 features:** Missing virtual threads, structured concurrency

13. **Q: Why is technical judgment more important than typing speed in the AI era?**

    **A:**
    - **AI handles implementation:** Writing boilerplate, syntax, scaffolding
    - **Humans make decisions:** Architecture, technology choices, trade-offs
    - **Value shifts:** From "how to write code" to "what to write and why"
    - **Examples:**
      - Choosing between Virtual Threads vs CompletableFuture
      - Deciding between microservices vs monolith
      - Selecting database technology
      - Determining when to upgrade frameworks
    - **Result:** Developers who think critically and make good decisions are more valuable than fast typists.

14. **Q: What is the efficiency multiplier when you know breaking changes vs when you don't?**

    **A:** Approximately **10x difference in efficiency:**
    - **Developer who knows breaking changes:**
      - Spots AI errors in 30 seconds
      - Fixes them immediately
      - Uses modern features automatically
      - Moves on to next task
    - **Developer who doesn't:**
      - Spends 2-3 hours debugging
      - Searches Stack Overflow
      - Tries multiple "fixes"
      - Misses modern features
      - Accumulates technical debt
    - **Impact:** Over a year, this compounds to hundreds of hours saved

15. **Q: How should you approach migrating from Spring Boot 3.0 to 3.5.x with Java 21?**

    **A:**
    1. **Preparation:**
       - Backup codebase
       - Document current dependencies
       - Check third-party library compatibility
       - Ensure JDK 21 installed
    2. **Update dependencies:**
       - Change Spring Boot version to 3.5.x
       - Update JDK version to 21
    3. **Migrate code:**
       - Change javax.* to jakarta.*
       - Replace deprecated APIs
       - Add Java 21 features (virtual threads, records, structured concurrency)
    4. **Test thoroughly:**
       - Compile check
       - Run all tests
       - Manual testing
    5. **Use AI:** Prompt AI to help with bulk migration
    6. **Incremental:** Migrate one module at a time for large applications

16. **Q: What is the role of Spring Boot Actuator in production applications?**

    **A:** Spring Boot Actuator provides production-ready features:
    - **Health checks:** Monitor application and dependency health
    - **Metrics:** Performance metrics (CPU, memory, requests)
    - **Auditing:** Track security events
    - **Logging:** View and configure logging
    - **Thread dumps:** Debug performance issues
    - **Environment:** View configuration properties
    - **Use cases:**
      - Kubernetes liveness/readiness probes
      - Monitoring dashboards (Prometheus, Grafana)
      - Alerting and incident response
    - **Security:** Must secure actuator endpoints in production

17. **Q: Explain the concept of "AI as amplifier, not crutch."**

    **A:**
    - **Crutch:** Relying on AI before having fundamentals
      - Copy-pasting without understanding
      - Can't debug or modify code
      - Stops learning
      - Creates technical debt
    - **Amplifier:** Using AI after building foundation
      - AI handles repetitive tasks
      - You focus on architecture and design
      - You review and improve AI output
      - You learn from AI suggestions
    - **Key:** Build competence first, then use AI to enhance productivity

18. **Q: What are the performance benefits of Virtual Threads compared to traditional threads?**

    **A:**
    - **Memory usage:** Virtual threads use ~1-2MB vs ~1MB per traditional thread (but virtual threads are cheaper to create and don't require pooling)
    - **Concurrency:** Can create 10,000+ virtual threads vs ~1000 traditional threads
    - **Context switching:** 0.1μs vs 1μs (10x faster)
    - **Startup time:** 0.1ms vs 10ms (100x faster)
    - **Throughput:** 10-100x more concurrent requests
    - **Use case:** I/O-bound applications benefit most

19. **Q: How do you prompt AI effectively for Spring Boot 3.5.x development?**

    **A:**
    1. **Always specify version:** "Using Spring Boot 3.5.x with JDK 21..."
    2. **Be specific about features:** "Use RestClient, Virtual Threads, Structured Concurrency"
    3. **Include requirements:** "With retry, timeout, circuit breaker"
    4. **Specify constraints:** "Runs on virtual threads"
    5. **Request best practices:** "Include error handling, logging, validation"
    6. **Provide context:** "For a high-concurrency e-commerce API"
    7. **Ask for explanations:** "Explain why you chose this approach"
    8. **Request alternatives:** "Show 2-3 approaches with trade-offs"

20. **Q: What is the future of Spring Boot development with AI?**

    **A:**
    - **Shift in skills:** Less emphasis on syntax memorization, more on architecture and design
    - **Higher bar:** Technical judgment becomes more valuable
    - **Faster development:** AI accelerates implementation
    - **Better code:** Developers can focus on quality, not speed
    - **Continuous learning:** Still need to stay current with framework evolution
    - **New roles:** "AI-assisted development" becomes standard practice
    - **Productivity boost:** Experienced developers become 2-3x more productive
    - **Challenges:** Need to verify AI output, avoid over-reliance, maintain fundamentals

---

## Appendix

### Quick Reference: Spring Boot 3.5.x Features

| Feature | Version | Use Case | Key Benefit |
|---------|---------|----------|-------------|
| Virtual Threads | Java 21 | High concurrency | 10-100x more requests |
| RestClient | 3.2+ | HTTP calls | Fluent API, better error handling |
| Docker Compose | 3.1+ | Local dev | Auto-start containers |
| AOT Native Images | 3.0+ | Production | Sub-second startup |
| Jakarta EE | 3.0+ | All apps | Modern namespace |
| Structured Concurrency | Java 21 | Parallel tasks | Better error handling |
| Scoped Values | Java 21 | Context propagation | Modern ThreadLocal |
| Records | Java 16+ | DTOs | Less boilerplate |

### Common AI Prompt Templates

**Template 1: REST Controller**
```
Write a Spring Boot 3.5.x REST controller for [RESOURCE] with:
- CRUD operations
- Input validation using Jakarta Validation
- Global exception handling
- Proper HTTP status codes
- OpenAPI documentation
- Error logging
- Virtual threads for async operations
```

**Template 2: Service Layer**
```
Write a Spring Boot 3.5.x service for [DOMAIN] with:
- Business logic for [OPERATIONS]
- Transaction management
- Error handling
- Logging at appropriate levels
- Virtual threads for concurrent operations
- Unit test examples
```

**Template 3: Data Access**
```
Write a Spring Data JPA repository for [ENTITY] with:
- Custom query methods
- Pagination support
- Query optimization (JOIN FETCH where needed)
- Proper indexing suggestions
- Jakarta namespace
```

### Troubleshooting Quick Reference

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| Compilation error: javax.* not found | Jakarta migration incomplete | Change to jakarta.* |
| Deprecation warning: WebMvcConfigurerAdapter | Using old API | Implement WebMvcConfigurer interface |
| ClassNotFoundException in native image | Missing reflection config | Add @NativeHint annotation |
| ThreadLocal returns null | Virtual thread unmounting | Use ScopedValue or pass context explicitly |
| N+1 queries | Missing JOIN FETCH | Add @EntityGraph or JOIN FETCH query |
| Slow performance | Missing virtual threads | Enable virtual threads in configuration |
| Java version error | Using JDK 17 or lower | Upgrade to JDK 21 |

---

**Congratulations!** You've completed the comprehensive guide to mastering Spring Boot 3.5.x in the AI era with Java 21. You now have the knowledge, strategies, and tools to stay effective and relevant as AI continues to transform software development.

**Remember:** AI is a powerful tool, but your technical judgment, design thinking, and understanding of Spring Boot 3.5.x and Java 21 are what will set you apart. Keep learning, keep building, and keep thinking critically.

**Happy coding! 🚀**

---

*Last Updated: January 2026*  
*Spring Boot Version: 3.5.x*  
*Java Version: 21*
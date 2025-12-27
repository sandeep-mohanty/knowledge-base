# NoException: A Better Way to Handle Exceptions in Java

NoException is a lightweight Java library that removes boilerplate try-catch code and offers functional exception handling.

As a Java developer, I’ve always felt that try-catch blocks add a lot of unnecessary noise to the code. We often copy-paste the same exception handling patterns again and again, which leads to inconsistent and hard-to-maintain code.

---

### What is NoException?

NoException brings functional programming style to Java exception handling. In many real-world applications, we end up with thousands of try-catch blocks scattered across the codebase. These catch clauses are often verbose, repetitive, and hard to maintain or test.

NoException solves this by providing a set of **predefined, reusable exception handlers** — acting as clean replacements for try-catch blocks. You can also define your own handlers based on the exception-handling policy of your project.



One of the simplest handlers is `Exceptions.silence()`, which ignores the exception and returns a fallback value.

**Instead of writing this (Traditional Java code):**
```java
String value;
try {
    value = "hello".substring(10); // throws exception
} catch (Exception e) {
    value = "fallback";
}
System.out.println(value);
```

**With NoException, you can write:**
```java
String value = Exceptions
        .silence()
        .get(() -> "hello".substring(10))
        .orElse("fallback");

System.out.println(value);
```

---

### Using NoException in Java — A Step-by-Step

#### 1. Add Dependencies
To use NoException, include the following dependencies in your `pom.xml` file:

```xml
<dependency>
    <groupId>com.machinezoo.noexception</groupId>
    <artifactId>noexception</artifactId>
    <version>1.9.1</version>
</dependency>
<dependency>
    <groupId>com.machinezoo.noexception</groupId>
    <artifactId>noexception-slf4j</artifactId>
    <version>1.9.1</version>
</dependency>
```

#### 2. Why Use NoException?
In most applications, code becomes cluttered with boilerplate try-catch blocks. Because they are verbose, developers often avoid handling exceptions properly and end up propagating them instead. 



This leads to problems such as:
* **Fallbacks:** Exception severity is unnecessarily escalated.
* **Callbacks:** Exceptions bubble up to unrelated code.
* **Loops:** One failing job can stop others.
* **Executors:** Exceptions can get lost in background tasks.
* **Lambdas:** Checked exceptions cause compilation issues.

---

### Start Using NoException

NoException provides several predefined exception handlers that eliminate or simplify try-catch logic. They are short, readable, safe, and lambda-friendly.

#### 1. Catch-All Handlers (Ignoring Exceptions)
To ignore exceptions safely without cluttering code:
```java
String t1 = null;
Exceptions.silence().run(() -> System.out.println("This is a " + t1));
```
If an exception occurs, it is caught and ignored, allowing your program to continue.

#### 2. Using Fallbacks
If you want to return a fallback value in case of an exception, use `get(Supplier)`:
```java
System.out.println(
    Exceptions.silence()
        .get(() -> "hello".substring(5))
        .orElse("fallback")
);
```

#### 3. Predefined Exception Handlers
NoException provides several ready-to-use handlers in the `Exceptions` class:
* **silence():** Swallows exceptions silently
* **sneak():** Bypasses checked exception restrictions
* **wrap():** Wraps checked exceptions as unchecked
* **propagate():** Rethrows exceptions (acts as a no-op)

With **SLF4J**, you also get additional logging handlers via `ExceptionLogging`:
* **log():** Logs and swallows the exception
* **log(Logger):** Uses a custom logger
* **log(Logger, String):** Logs with a custom message

#### 4. Functional Interface Support
To handle exceptions inside functional interfaces (like `Function`, `Supplier`, `Runnable`, etc.), wrap them using NoException:
```java
Function<String, String> hello = x -> "Hello " + x;
System.out.println(
    Exceptions.silence().function(hello).apply(null).orElse("Who?")
);
```

If you need to pass it to APIs expecting a standard functional interface, use `orElse(defaultValue)`:
```java
Function<Object, String> format = x -> "[" + x.toString() + "]";
Function<Object, String> safe = Exceptions.silence().function(format).orElse("???");
System.out.println(safe.apply(null)); // Prints "???"
```

#### 5. Creating Custom Exception Handlers
You can define custom policies by extending `ExceptionHandler` and overriding the `handle()` method.

```java
public class ExceptionCounter extends ExceptionHandler {
    private long count;

    @Override
    public synchronized boolean handle(Throwable exception) {
        ++count;
        return true; // true means exception is handled
    }

    public synchronized long report() {
        return count;
    }
}
```

Use this in your project like this:
```java
public class ExceptionPolicy {
    private static final ExceptionCounter counter = new ExceptionCounter();
    public static ExceptionCounter count() {
        return counter;
    }
}

// Applying and tracking exceptions
ExceptionPolicy.count().run(() -> oftenFailingCode());
System.out.println(ExceptionPolicy.count().report());
```

---

### Advantages of Using NoException

1.  **Cleaner Code:** Removes repetitive try-catch blocks for better readability.
2.  **Consistent Policy:** Uniform handling across the entire project.
3.  **Greater Flexibility:** Different handlers for different contexts (silent, logging, or rethrowing).
4.  **Improved Testability:** Verification of exception management without duplicating logic.
5.  **Functional Programming Support:** Natural fit for Java Streams, lambdas, and Optional.

### Conclusion
Integrating NoException into real-world projects transforms how exception handling is approached in Java. It encourages a shift from scattered try-catch blocks to structured **exception policies**, treating error management as part of the architecture rather than just recovery.

For teams using modern Java (Java 8 and beyond) and prioritizing clean, functional code, adopting NoException is highly recommended.
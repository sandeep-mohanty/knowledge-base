# Railway Oriented Programming in Java: A Complete Tutorial

## Introduction

Railway Oriented Programming (ROP) is a functional programming pattern that treats the flow of operations like a railway track. Success cases continue on the main track, while errors switch to a parallel error track. This tutorial will show you how to implement ROP in Java using a custom Result type.

## Table of Contents

1. [Understanding the Concept](#understanding-the-concept)
2. [Creating the Result Type](#creating-the-result-type)
3. [Basic Operations](#basic-operations)
4. [Chaining Operations](#chaining-operations)
5. [Practical Examples](#practical-examples)
6. [Advanced Patterns](#advanced-patterns)
7. [Best Practices](#best-practices)

## Understanding the Concept

In traditional Java error handling, we often see nested try-catch blocks or multiple if-else statements checking for null or error conditions. ROP provides a cleaner alternative by encapsulating success and failure in a single type that can be composed.

## Creating the Result Type

Let's start by creating our core Result type:

### Basic Result Interface

```java
public sealed interface Result<T, E> permits Success, Failure {
    
    // Check if the result is successful
    boolean isSuccess();
    
    // Check if the result is a failure
    boolean isFailure();
    
    // Get the value if success, throw exception if failure
    T getValue();
    
    // Get the error if failure, throw exception if success
    E getError();
}
```

### Success Implementation

```java
public record Success<T, E>(T value) implements Result<T, E> {
    
    @Override
    public boolean isSuccess() {
        return true;
    }
    
    @Override
    public boolean isFailure() {
        return false;
    }
    
    @Override
    public T getValue() {
        return value;
    }
    
    @Override
    public E getError() {
        throw new IllegalStateException("Cannot get error from Success");
    }
}
```

### Failure Implementation

```java
public record Failure<T, E>(E error) implements Result<T, E> {
    
    @Override
    public boolean isSuccess() {
        return false;
    }
    
    @Override
    public boolean isFailure() {
        return true;
    }
    
    @Override
    public T getValue() {
        throw new IllegalStateException("Cannot get value from Failure");
    }
    
    @Override
    public E getError() {
        return error;
    }
}
```

### Factory Methods

```java
public interface Result<T, E> permits Success, Failure {
    // ... previous methods ...
    
    // Factory method for success
    static <T, E> Result<T, E> success(T value) {
        return new Success<>(value);
    }
    
    // Factory method for failure
    static <T, E> Result<T, E> failure(E error) {
        return new Failure<>(error);
    }
}
```

## Basic Operations

Now let's add essential operations to our Result type:

### Map Operation

The map operation transforms the success value while preserving failures:

```java
public interface Result<T, E> permits Success, Failure {
    // ... previous methods ...
    
    default <U> Result<U, E> map(Function<T, U> mapper) {
        if (isSuccess()) {
            return Result.success(mapper.apply(getValue()));
        }
        return Result.failure(getError());
    }
}
```

### FlatMap Operation

The flatMap operation allows chaining operations that return Results:

```java
public interface Result<T, E> permits Success, Failure {
    // ... previous methods ...
    
    default <U> Result<U, E> flatMap(Function<T, Result<U, E>> mapper) {
        if (isSuccess()) {
            return mapper.apply(getValue());
        }
        return Result.failure(getError());
    }
}
```

### MapError Operation

Transform error values:

```java
public interface Result<T, E> permits Success, Failure {
    // ... previous methods ...
    
    default <F> Result<T, F> mapError(Function<E, F> mapper) {
        if (isFailure()) {
            return Result.failure(mapper.apply(getError()));
        }
        return Result.success(getValue());
    }
}
```

### OrElse Operations

Provide fallback values:

```java
public interface Result<T, E> permits Success, Failure {
    // ... previous methods ...
    
    default T orElse(T defaultValue) {
        return isSuccess() ? getValue() : defaultValue;
    }
    
    default T orElseGet(Supplier<T> supplier) {
        return isSuccess() ? getValue() : supplier.get();
    }
    
    default <X extends Throwable> T orElseThrow(Function<E, X> exceptionMapper) throws X {
        if (isSuccess()) {
            return getValue();
        }
        throw exceptionMapper.apply(getError());
    }
}
```

## Chaining Operations

Let's create a more complete example showing how to chain operations:

### Complete Result Implementation

```java
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.function.Consumer;
import java.util.Optional;

public sealed interface Result<T, E> permits Success, Failure {
    
    boolean isSuccess();
    boolean isFailure();
    T getValue();
    E getError();
    
    static <T, E> Result<T, E> success(T value) {
        return new Success<>(value);
    }
    
    static <T, E> Result<T, E> failure(E error) {
        return new Failure<>(error);
    }
    
    default <U> Result<U, E> map(Function<T, U> mapper) {
        if (isSuccess()) {
            return Result.success(mapper.apply(getValue()));
        }
        return Result.failure(getError());
    }
    
    default <U> Result<U, E> flatMap(Function<T, Result<U, E>> mapper) {
        if (isSuccess()) {
            return mapper.apply(getValue());
        }
        return Result.failure(getError());
    }
    
    default <F> Result<T, F> mapError(Function<E, F> mapper) {
        if (isFailure()) {
            return Result.failure(mapper.apply(getError()));
        }
        return Result.success(getValue());
    }
    
    default Result<T, E> peek(Consumer<T> consumer) {
        if (isSuccess()) {
            consumer.accept(getValue());
        }
        return this;
    }
    
    default Result<T, E> peekError(Consumer<E> consumer) {
        if (isFailure()) {
            consumer.accept(getError());
        }
        return this;
    }
    
    default T orElse(T defaultValue) {
        return isSuccess() ? getValue() : defaultValue;
    }
    
    default T orElseGet(Supplier<T> supplier) {
        return isSuccess() ? getValue() : supplier.get();
    }
    
    default <X extends Throwable> T orElseThrow(Function<E, X> exceptionMapper) throws X {
        if (isSuccess()) {
            return getValue();
        }
        throw exceptionMapper.apply(getError());
    }
    
    default Optional<T> toOptional() {
        return isSuccess() ? Optional.of(getValue()) : Optional.empty();
    }
    
    default Result<T, E> recover(Function<E, T> recovery) {
        if (isFailure()) {
            return Result.success(recovery.apply(getError()));
        }
        return this;
    }
    
    default Result<T, E> recoverWith(Function<E, Result<T, E>> recovery) {
        if (isFailure()) {
            return recovery.apply(getError());
        }
        return this;
    }
}
```

## Practical Examples

### Example 1: User Registration Flow

```java
public class UserService {
    
    public Result<User, String> registerUser(String email, String password) {
        return validateEmail(email)
            .flatMap(validEmail -> validatePassword(password))
            .flatMap(validPassword -> checkEmailNotExists(email))
            .flatMap(this::createUser)
            .flatMap(this::sendWelcomeEmail)
            .peek(user -> System.out.println("User registered: " + user.getEmail()));
    }
    
    private Result<String, String> validateEmail(String email) {
        if (email == null || !email.contains("@")) {
            return Result.failure("Invalid email format");
        }
        return Result.success(email);
    }
    
    private Result<String, String> validatePassword(String password) {
        if (password == null || password.length() < 8) {
            return Result.failure("Password must be at least 8 characters");
        }
        return Result.success(password);
    }
    
    private Result<String, String> checkEmailNotExists(String email) {
        // Simulate database check
        if (emailExists(email)) {
            return Result.failure("Email already registered");
        }
        return Result.success(email);
    }
    
    private Result<User, String> createUser(String email) {
        try {
            User user = new User(email);
            // Save to database
            return Result.success(user);
        } catch (Exception e) {
            return Result.failure("Failed to create user: " + e.getMessage());
        }
    }
    
    private Result<User, String> sendWelcomeEmail(User user) {
        try {
            // Send email
            return Result.success(user);
        } catch (Exception e) {
            return Result.failure("Failed to send welcome email: " + e.getMessage());
        }
    }
    
    private boolean emailExists(String email) {
        // Database check simulation
        return false;
    }
}
```

### Example 2: Order Processing Pipeline

```java
public class OrderService {
    
    public Result<Order, OrderError> processOrder(OrderRequest request) {
        return validateOrderRequest(request)
            .flatMap(this::checkInventory)
            .flatMap(this::calculatePricing)
            .flatMap(this::applyDiscounts)
            .flatMap(this::processPayment)
            .flatMap(this::createOrder)
            .flatMap(this::updateInventory)
            .mapError(this::enrichError)
            .peek(order -> logSuccess(order))
            .peekError(error -> logError(error));
    }
    
    private Result<OrderRequest, OrderError> validateOrderRequest(OrderRequest request) {
        if (request.getItems().isEmpty()) {
            return Result.failure(new OrderError("EMPTY_ORDER", "Order has no items"));
        }
        return Result.success(request);
    }
    
    private Result<OrderRequest, OrderError> checkInventory(OrderRequest request) {
        for (OrderItem item : request.getItems()) {
            if (!hasStock(item)) {
                return Result.failure(new OrderError("OUT_OF_STOCK", 
                    "Item " + item.getProductId() + " is out of stock"));
            }
        }
        return Result.success(request);
    }
    
    private Result<PricedOrder, OrderError> calculatePricing(OrderRequest request) {
        try {
            double total = request.getItems().stream()
                .mapToDouble(item -> item.getQuantity() * getPrice(item))
                .sum();
            return Result.success(new PricedOrder(request, total));
        } catch (Exception e) {
            return Result.failure(new OrderError("PRICING_ERROR", 
                "Failed to calculate price: " + e.getMessage()));
        }
    }
    
    // Additional methods...
}
```

### Example 3: File Processing with Error Recovery

```java
public class FileProcessor {
    
    public Result<ProcessedFile, FileError> processFile(String filePath) {
        return readFile(filePath)
            .flatMap(this::validateFormat)
            .flatMap(this::parseContent)
            .flatMap(this::transform)
            .flatMap(this::save)
            .recoverWith(error -> {
                if (error.getCode().equals("PARSE_ERROR")) {
                    return tryAlternativeParser(filePath);
                }
                return Result.failure(error);
            });
    }
    
    private Result<String, FileError> readFile(String filePath) {
        try {
            String content = Files.readString(Paths.get(filePath));
            return Result.success(content);
        } catch (IOException e) {
            return Result.failure(new FileError("READ_ERROR", 
                "Cannot read file: " + e.getMessage()));
        }
    }
    
    private Result<String, FileError> validateFormat(String content) {
        if (content.startsWith("<?xml")) {
            return Result.success(content);
        }
        return Result.failure(new FileError("FORMAT_ERROR", "Invalid file format"));
    }
    
    // Additional methods...
}
```

## Advanced Patterns

### Combining Multiple Results

```java
public class ResultCombinators {
    
    public static <T1, T2, E> Result<Tuple2<T1, T2>, E> combine(
            Result<T1, E> result1, 
            Result<T2, E> result2) {
        return result1.flatMap(value1 ->
            result2.map(value2 -> new Tuple2<>(value1, value2))
        );
    }
    
    public static <T, E> Result<List<T>, E> sequence(List<Result<T, E>> results) {
        List<T> values = new ArrayList<>();
        for (Result<T, E> result : results) {
            if (result.isFailure()) {
                return Result.failure(result.getError());
            }
            values.add(result.getValue());
        }
        return Result.success(values);
    }
    
    public static <T, E> Result<List<T>, List<E>> traverse(List<Result<T, E>> results) {
        List<T> successes = new ArrayList<>();
        List<E> failures = new ArrayList<>();
        
        for (Result<T, E> result : results) {
            if (result.isSuccess()) {
                successes.add(result.getValue());
            } else {
                failures.add(result.getError());
            }
        }
        
        return failures.isEmpty() 
            ? Result.success(successes) 
            : Result.failure(failures);
    }
}
```

### Async Operations with CompletableFuture

```java
public class AsyncResult {
    
    public static <T, E> CompletableFuture<Result<T, E>> fromFuture(
            CompletableFuture<T> future, 
            Function<Throwable, E> errorMapper) {
        return future
            .thenApply(Result::<T, E>success)
            .exceptionally(throwable -> Result.failure(errorMapper.apply(throwable)));
    }
    
    public static <T, E> CompletableFuture<Result<T, E>> flatMapAsync(
            Result<T, E> result,
            Function<T, CompletableFuture<Result<T, E>>> asyncMapper) {
        if (result.isSuccess()) {
            return asyncMapper.apply(result.getValue());
        }
        return CompletableFuture.completedFuture(Result.failure(result.getError()));
    }
}
```

## Best Practices

### 1. Use Specific Error Types

Instead of using String for errors, create specific error types:

```
# Railway Oriented Programming in C#: A Complete Tutorial

## Introduction

Railway Oriented Programming (ROP) is a functional programming pattern that treats the flow of operations like a railway track. Success cases continue on the main track, while errors switch to a parallel error track. This tutorial will show you how to implement ROP in C# using a custom Result type.

## Table of Contents

1. [Understanding the Concept](#understanding-the-concept)
2. [Creating the Result Type](#creating-the-result-type)
3. [Basic Operations](#basic-operations)
4. [Chaining Operations](#chaining-operations)
5. [Practical Examples](#practical-examples)
6. [Advanced Patterns](#advanced-patterns)
7. [Best Practices](#best-practices)

## Understanding the Concept

In traditional C# error handling, we often see nested try-catch blocks or multiple if-else statements checking for null or error conditions. ROP provides a cleaner alternative by encapsulating success and failure in a single type that can be composed.

## Creating the Result Type

Let's start by creating our core Result type:

### Basic Result Structure

```csharp
public abstract class Result<TSuccess, TError>
{
    public abstract bool IsSuccess { get; }
    public abstract bool IsFailure { get; }
    public abstract TSuccess Value { get; }
    public abstract TError Error { get; }
    
    // Factory methods
    public static Result<TSuccess, TError> Success(TSuccess value) 
        => new SuccessResult<TSuccess, TError>(value);
    
    public static Result<TSuccess, TError> Failure(TError error) 
        => new FailureResult<TSuccess, TError>(error);
}
```

### Success Implementation

```csharp
internal sealed class SuccessResult<TSuccess, TError> : Result<TSuccess, TError>
{
    private readonly TSuccess _value;
    
    internal SuccessResult(TSuccess value)
    {
        _value = value ?? throw new ArgumentNullException(nameof(value));
    }
    
    public override bool IsSuccess => true;
    public override bool IsFailure => false;
    
    public override TSuccess Value => _value;
    
    public override TError Error 
        => throw new InvalidOperationException("Cannot get error from success result");
}
```

### Failure Implementation

```csharp
internal sealed class FailureResult<TSuccess, TError> : Result<TSuccess, TError>
{
    private readonly TError _error;
    
    internal FailureResult(TError error)
    {
        _error = error ?? throw new ArgumentNullException(nameof(error));
    }
    
    public override bool IsSuccess => false;
    public override bool IsFailure => true;
    
    public override TSuccess Value 
        => throw new InvalidOperationException("Cannot get value from failure result");
    
    public override TError Error => _error;
}
```

### Pattern Matching Support

```csharp
public abstract class Result<TSuccess, TError>
{
    // ... previous members ...
    
    public TResult Match<TResult>(
        Func<TSuccess, TResult> onSuccess,
        Func<TError, TResult> onFailure)
    {
        return IsSuccess ? onSuccess(Value) : onFailure(Error);
    }
    
    public void Match(
        Action<TSuccess> onSuccess,
        Action<TError> onFailure)
    {
        if (IsSuccess)
            onSuccess(Value);
        else
            onFailure(Error);
    }
}
```

## Basic Operations

Now let's add essential operations to our Result type:

### Map Operation

The Map operation transforms the success value while preserving failures:

```csharp
public abstract class Result<TSuccess, TError>
{
    // ... previous members ...
    
    public Result<TNewSuccess, TError> Map<TNewSuccess>(
        Func<TSuccess, TNewSuccess> mapper)
    {
        if (mapper == null) throw new ArgumentNullException(nameof(mapper));
        
        return IsSuccess 
            ? Result<TNewSuccess, TError>.Success(mapper(Value))
            : Result<TNewSuccess, TError>.Failure(Error);
    }
}
```

### Bind (FlatMap) Operation

The Bind operation allows chaining operations that return Results:

```csharp
public abstract class Result<TSuccess, TError>
{
    // ... previous members ...
    
    public Result<TNewSuccess, TError> Bind<TNewSuccess>(
        Func<TSuccess, Result<TNewSuccess, TError>> binder)
    {
        if (binder == null) throw new ArgumentNullException(nameof(binder));
        
        return IsSuccess ? binder(Value) : Result<TNewSuccess, TError>.Failure(Error);
    }
    
    // Alias for Bind
    public Result<TNewSuccess, TError> FlatMap<TNewSuccess>(
        Func<TSuccess, Result<TNewSuccess, TError>> mapper) => Bind(mapper);
}
```

### MapError Operation

Transform error values:

```csharp
public abstract class Result<TSuccess, TError>
{
    // ... previous members ...
    
    public Result<TSuccess, TNewError> MapError<TNewError>(
        Func<TError, TNewError> mapper)
    {
        if (mapper == null) throw new ArgumentNullException(nameof(mapper));
        
        return IsFailure 
            ? Result<TSuccess, TNewError>.Failure(mapper(Error))
            : Result<TSuccess, TNewError>.Success(Value);
    }
}
```

### OrElse Operations

Provide fallback values:

```csharp
public abstract class Result<TSuccess, TError>
{
    // ... previous members ...
    
    public TSuccess OrElse(TSuccess defaultValue)
    {
        return IsSuccess ? Value : defaultValue;
    }
    
    public TSuccess OrElse(Func<TSuccess> defaultValueFactory)
    {
        if (defaultValueFactory == null) throw new ArgumentNullException(nameof(defaultValueFactory));
        
        return IsSuccess ? Value : defaultValueFactory();
    }
    
    public TSuccess OrElseThrow(Func<TError, Exception> exceptionFactory)
    {
        if (exceptionFactory == null) throw new ArgumentNullException(nameof(exceptionFactory));
        
        return IsSuccess ? Value : throw exceptionFactory(Error);
    }
}
```

## Chaining Operations

Let's create a more complete example showing how to chain operations:

### Complete Result Implementation

```csharp
using System;
using System.Threading.Tasks;

public abstract class Result<TSuccess, TError>
{
    public abstract bool IsSuccess { get; }
    public abstract bool IsFailure { get; }
    public abstract TSuccess Value { get; }
    public abstract TError Error { get; }
    
    public static Result<TSuccess, TError> Success(TSuccess value) 
        => new SuccessResult<TSuccess, TError>(value);
    
    public static Result<TSuccess, TError> Failure(TError error) 
        => new FailureResult<TSuccess, TError>(error);
    
    public Result<TNewSuccess, TError> Map<TNewSuccess>(
        Func<TSuccess, TNewSuccess> mapper)
    {
        if (mapper == null) throw new ArgumentNullException(nameof(mapper));
        return IsSuccess 
            ? Success<TNewSuccess>(mapper(Value))
            : Failure<TNewSuccess>(Error);
    }
    
    public Result<TNewSuccess, TError> Bind<TNewSuccess>(
        Func<TSuccess, Result<TNewSuccess, TError>> binder)
    {
        if (binder == null) throw new ArgumentNullException(nameof(binder));
        return IsSuccess ? binder(Value) : Failure<TNewSuccess>(Error);
    }
    
    public Result<TSuccess, TNewError> MapError<TNewError>(
        Func<TError, TNewError> mapper)
    {
        if (mapper == null) throw new ArgumentNullException(nameof(mapper));
        return IsFailure 
            ? Failure<TSuccess, TNewError>(mapper(Error))
            : Success<TSuccess, TNewError>(Value);
    }
    
    public Result<TSuccess, TError> Tap(Action<TSuccess> action)
    {
        if (action == null) throw new ArgumentNullException(nameof(action));
        if (IsSuccess) action(Value);
        return this;
    }
    
    public Result<TSuccess, TError> TapError(Action<TError> action)
    {
        if (action == null) throw new ArgumentNullException(nameof(action));
        if (IsFailure) action(Error);
        return this;
    }
    
    public TResult Match<TResult>(
        Func<TSuccess, TResult> onSuccess,
        Func<TError, TResult> onFailure)
    {
        if (onSuccess == null) throw new ArgumentNullException(nameof(onSuccess));
        if (onFailure == null) throw new ArgumentNullException(nameof(onFailure));
        return IsSuccess ? onSuccess(Value) : onFailure(Error);
    }
    
    public TSuccess OrElse(TSuccess defaultValue)
    {
        return IsSuccess ? Value : defaultValue;
    }
    
    public TSuccess OrElse(Func<TSuccess> defaultValueFactory)
    {
        if (defaultValueFactory == null) throw new ArgumentNullException(nameof(defaultValueFactory));
        return IsSuccess ? Value : defaultValueFactory();
    }
    
    public Result<TSuccess, TError> Recover(Func<TError, TSuccess> recovery)
    {
        if (recovery == null) throw new ArgumentNullException(nameof(recovery));
        return IsFailure ? Success(recovery(Error)) : this;
    }
    
    public Result<TSuccess, TError> RecoverWith(
        Func<TError, Result<TSuccess, TError>> recovery)
    {
        if (recovery == null) throw new ArgumentNullException(nameof(recovery));
        return IsFailure ? recovery(Error) : this;
    }
    
    // Helper methods for creating typed results
    private static Result<T, TError> Success<T>(T value) 
        => Result<T, TError>.Success(value);
    
    private static Result<T, TError> Failure<T>(TError error) 
        => Result<T, TError>.Failure(error);
    
    private static Result<TSuccess, T> Success<TSuccess, T>(TSuccess value) 
        => Result<TSuccess, T>.Success(value);
    
    private static Result<TSuccess, T> Failure<TSuccess, T>(T error) 
        => Result<TSuccess, T>.Failure(error);
}
```

## Practical Examples

### Example 1: User Registration Flow

```csharp
public class UserService
{
    public Result<User, string> RegisterUser(string email, string password)
    {
        return ValidateEmail(email)
            .Bind(_ => ValidatePassword(password))
            .Bind(_ => CheckEmailNotExists(email))
            .Bind(_ => CreateUser(email, password))
            .Bind(SendWelcomeEmail)
            .Tap(user => Console.WriteLine($"User registered: {user.Email}"));
    }
    
    private Result<string, string> ValidateEmail(string email)
    {
        if (string.IsNullOrWhiteSpace(email) || !email.Contains("@"))
        {
            return Result<string, string>.Failure("Invalid email format");
        }
        return Result<string, string>.Success(email);
    }
    
    private Result<string, string> ValidatePassword(string password)
    {
        if (string.IsNullOrWhiteSpace(password) || password.Length < 8)
        {
            return Result<string, string>.Failure("Password must be at least 8 characters");
        }
        return Result<string, string>.Success(password);
    }
    
    private Result<string, string> CheckEmailNotExists(string email)
    {
        // Simulate database check
        if (EmailExists(email))
        {
            return Result<string, string>.Failure("Email already registered");
        }
        return Result<string, string>.Success(email);
    }
    
    private Result<User, string> CreateUser(string email, string password)
    {
        try
        {
            var user = new User 
            { 
                Email = email, 
                PasswordHash = HashPassword(password) 
            };
            // Save to database
            return Result<User, string>.Success(user);
        }
        catch (Exception ex)
        {
            return Result<User, string>.Failure($"Failed to create user: {ex.Message}");
        }
    }
    
    private Result<User, string> SendWelcomeEmail(User user)
    {
        try
        {
            // Send email logic
            return Result<User, string>.Success(user);
        }
        catch (Exception ex)
        {
            return Result<User, string>.Failure($"Failed to send welcome email: {ex.Message}");
        }
    }
    
    private bool EmailExists(string email)
    {
        // Database check simulation
        return false;
    }
    
    private string HashPassword(string password)
    {
        // Password hashing logic
        return password; // Simplified for example
    }
}

public class User
{
    public string Email { get; set; }
    public string PasswordHash { get; set; }
}
```

### Example 2: Order Processing Pipeline

```csharp
public class OrderService
{
    public Result<Order, OrderError> ProcessOrder(OrderRequest request)
    {
        return ValidateOrderRequest(request)
            .Bind(CheckInventory)
            .Bind(CalculatePricing)
            .Bind(ApplyDiscounts)
            .Bind(ProcessPayment)
            .Bind(CreateOrder)
            .Bind(UpdateInventory)
            .MapError(EnrichError)
            .Tap(order => LogSuccess(order))
            .TapError(error => LogError(error));
    }
    
    private Result<OrderRequest, OrderError> ValidateOrderRequest(OrderRequest request)
    {
        if (request?.Items == null || !request.Items.Any())
        {
            return Result<OrderRequest, OrderError>.Failure(
                new OrderError("EMPTY_ORDER", "Order has no items"));
        }
        return Result<OrderRequest, OrderError>.Success(request);
    }
    
    private Result<OrderRequest, OrderError> CheckInventory(OrderRequest request)
    {
        foreach (var item in request.Items)
        {
            if (!HasStock(item))
            {
                return Result<OrderRequest, OrderError>.Failure(
                    new OrderError("OUT_OF_STOCK", 
                        $"Item {item.ProductId} is out of stock"));
            }
        }
        return Result<OrderRequest, OrderError>.Success(request);
    }
    
    private Result<PricedOrder, OrderError> CalculatePricing(OrderRequest request)
    {
        try
        {
            var total = request.Items.Sum(item => item.Quantity * GetPrice(item));
            var pricedOrder = new PricedOrder 
            { 
                Request = request, 
                TotalAmount = total 
            };
            return Result<PricedOrder, OrderError>.Success(pricedOrder);
        }
        catch (Exception ex)
        {
            return Result<PricedOrder, OrderError>.Failure(
                new OrderError("PRICING_ERROR", 
                    $"Failed to calculate price: {ex.Message}"));
        }
    }
    
    private Result<PricedOrder, OrderError> ApplyDiscounts(PricedOrder pricedOrder)
    {
        try
        {
            // Apply discount logic
            if (pricedOrder.TotalAmount > 100)
            {
                pricedOrder.Discount = 10; // $10 off for orders over $100
            }
            
            if (IsLoyalCustomer(pricedOrder.Request.CustomerId))
            {
                pricedOrder.Discount += 5; // Additional $5 off for loyal customers
            }
            
            pricedOrder.FinalAmount = pricedOrder.TotalAmount - pricedOrder.Discount;
            
            return Result<PricedOrder, OrderError>.Success(pricedOrder);
        }
        catch (Exception ex)
        {
            return Result<PricedOrder, OrderError>.Failure(
                new OrderError("DISCOUNT_ERROR", 
                    $"Failed to apply discounts: {ex.Message}"));
        }
    }
    
    private Result<PaymentResult, OrderError> ProcessPayment(PricedOrder pricedOrder)
    {
        try
        {
            // Payment processing logic
            var paymentResult = new PaymentResult
            {
                Order = pricedOrder,
                TransactionId = Guid.NewGuid().ToString(),
                Status = "APPROVED"
            };
            
            return Result<PaymentResult, OrderError>.Success(paymentResult);
        }
        catch (Exception ex)
        {
            return Result<PaymentResult, OrderError>.Failure(
                new OrderError("PAYMENT_ERROR", 
                    $"Payment processing failed: {ex.Message}"));
        }
    }
    
    private Result<Order, OrderError> CreateOrder(PaymentResult paymentResult)
    {
        try
        {
            // Create and persist the order
            var order = new Order
            {
                OrderId = Guid.NewGuid().ToString(),
                CustomerId = paymentResult.Order.Request.CustomerId,
                Items = paymentResult.Order.Request.Items,
                TotalAmount = paymentResult.Order.TotalAmount,
                Discount = paymentResult.Order.Discount,
                FinalAmount = paymentResult.Order.FinalAmount,
                PaymentTransactionId = paymentResult.TransactionId,
                OrderStatus = "CONFIRMED",
                CreatedDate = DateTime.UtcNow
            };
            
            // Save to database
            
            return Result<Order, OrderError>.Success(order);
        }
        catch (Exception ex)
        {
            return Result<Order, OrderError>.Failure(
                new OrderError("ORDER_CREATION_ERROR", 
                    $"Failed to create order: {ex.Message}"));
        }
    }
    
    private Result<Order, OrderError> UpdateInventory(Order order)
    {
        try
        {
            // Update inventory for each item
            foreach (var item in order.Items)
            {
                UpdateProductStock(item.ProductId, item.Quantity);
            }
            
            return Result<Order, OrderError>.Success(order);
        }
        catch (Exception ex)
        {
            // Still return success but log error
            // We might want to handle this differently in a real system
            LogInventoryUpdateError(ex, order);
            return Result<Order, OrderError>.Success(order);
        }
    }
    
    private OrderError EnrichError(OrderError error)
    {
        // Add additional context to the error
        error.Timestamp = DateTime.UtcNow;
        error.TraceId = Guid.NewGuid().ToString();
        return error;
    }
    
    private bool HasStock(OrderItem item)
    {
        // Inventory check simulation
        return true;
    }
    
    private decimal GetPrice(OrderItem item)
    {
        // Price lookup simulation
        return 10.0m * item.ProductId;
    }
    
    private bool IsLoyalCustomer(int customerId)
    {
        // Customer status check simulation
        return customerId % 2 == 0;
    }
    
    private void UpdateProductStock(int productId, int quantity)
    {
        // Update inventory simulation
    }
    
    private void LogSuccess(Order order)
    {
        Console.WriteLine($"Order {order.OrderId} processed successfully");
    }
    
    private void LogError(OrderError error)
    {
        Console.WriteLine($"Error: [{error.Code}] {error.Message}");
    }
    
    private void LogInventoryUpdateError(Exception ex, Order order)
    {
        Console.WriteLine($"Warning: Inventory update failed for order {order.OrderId}: {ex.Message}");
    }
}

public class OrderRequest
{
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; } = new List<OrderItem>();
}

public class OrderItem
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}

public class PricedOrder
{
    public OrderRequest Request { get; set; }
    public decimal TotalAmount { get; set; }
    public decimal Discount { get; set; }
    public decimal FinalAmount { get; set; }
}

public class PaymentResult
{
    public PricedOrder Order { get; set; }
    public string TransactionId { get; set; }
    public string Status { get; set; }
}

public class Order
{
    public string OrderId { get; set; }
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
    public decimal TotalAmount { get; set; }
    public decimal Discount { get; set; }
    public decimal FinalAmount { get; set; }
    public string PaymentTransactionId { get; set; }
    public string OrderStatus { get; set; }
    public DateTime CreatedDate { get; set; }
}

public class OrderError
{
    public string Code { get; set; }
    public string Message { get; set; }
    public DateTime Timestamp { get; set; }
    public string TraceId { get; set; }
    
    public OrderError(string code, string message)
    {
        Code = code;
        Message = message;
        Timestamp = DateTime.UtcNow;
    }
}
```

### Example 3: File Processing with Error Recovery

```csharp
public class FileProcessor
{
    public Result<ProcessedFile, FileError> ProcessFile(string filePath)
    {
        return ReadFile(filePath)
            .Bind(ValidateFormat)
            .Bind(ParseContent)
            .Bind(Transform)
            .Bind(Save)
            .RecoverWith(error => 
            {
                if (error.Code == "PARSE_ERROR")
                {
                    return TryAlternativeParser(filePath);
                }
                return Result<ProcessedFile, FileError>.Failure(error);
            });
    }
    
    private Result<string, FileError> ReadFile(string filePath)
    {
        try
        {
            if (!File.Exists(filePath))
            {
                return Result<string, FileError>.Failure(
                    new FileError("FILE_NOT_FOUND", $"File not found: {filePath}"));
            }
            
            string content = File.ReadAllText(filePath);
            return Result<string, FileError>.Success(content);
        }
        catch (Exception ex)
        {
            return Result<string, FileError>.Failure(
                new FileError("READ_ERROR", $"Cannot read file: {ex.Message}"));
        }
    }
    
    private Result<string, FileError> ValidateFormat(string content)
    {
        if (content.StartsWith("<?xml"))
        {
            return Result<string, FileError>.Success(content);
        }
        
        if (content.StartsWith("{") || content.StartsWith("["))
        {
            return Result<string, FileError>.Success(content);
        }
        
        return Result<string, FileError>.Failure(
            new FileError("FORMAT_ERROR", "Invalid file format"));
    }
    
    private Result<FileData, FileError> ParseContent(string content)
    {
        try
        {
            // Parsing logic depends on file type
            if (content.StartsWith("<?xml"))
            {
                return ParseXml(content);
            }
            else
            {
                return ParseJson(content);
            }
        }
        catch (Exception ex)
        {
            return Result<FileData, FileError>.Failure(
                new FileError("PARSE_ERROR", $"Failed to parse content: {ex.Message}"));
        }
    }
    
    private Result<TransformedData, FileError> Transform(FileData data)
    {
        try
        {
            // Apply business transformations to the data
            var transformed = new TransformedData
            {
                Id = data.Id,
                Name = data.Name.ToUpper(), // Simple transformation example
                ProcessedDate = DateTime.UtcNow,
                Values = data.Values.Select(v => v * 2).ToList() // Double all values
            };
            
            return Result<TransformedData, FileError>.Success(transformed);
        }
        catch (Exception ex)
        {
            return Result<TransformedData, FileError>.Failure(
                new FileError("TRANSFORM_ERROR", $"Failed to transform data: {ex.Message}"));
        }
    }
    
    private Result<ProcessedFile, FileError> Save(TransformedData data)
    {
        try
        {
            // Save the transformed data
            var outputPath = $"processed_{data.Id}_{DateTime.UtcNow:yyyyMMddHHmmss}.json";
            
            // Convert to JSON and save
            // SaveDataToJson(data, outputPath);
            
            var processedFile = new ProcessedFile
            {
                OriginalId = data.Id,
                OutputPath = outputPath,
                ProcessedDate = data.ProcessedDate
            };
            
            return Result<ProcessedFile, FileError>.Success(processedFile);
        }
        catch (Exception ex)
        {
            return Result<ProcessedFile, FileError>.Failure(
                new FileError("SAVE_ERROR", $"Failed to save processed data: {ex.Message}"));
        }
    }
    
    private Result<ProcessedFile, FileError> TryAlternativeParser(string filePath)
    {
        Console.WriteLine($"Trying alternative parser for {filePath}");
        
        // Implement a more robust parsing strategy
        return ReadFile(filePath)
            .Bind(content => ParseWithFallbackStrategy(content))
            .Bind(Transform)
            .Bind(Save);
    }
    
    private Result<FileData, FileError> ParseWithFallbackStrategy(string content)
    {
        try
        {
            // Implement a more forgiving parser
            // ...
            
            // For this example, create some dummy data
            var fallbackData = new FileData
            {
                Id = Guid.NewGuid(),
                Name = "Recovered Data",
                Values = new List<int> { 1, 2, 3 }
            };
            
            return Result<FileData, FileError>.Success(fallbackData);
        }
        catch (Exception ex)
        {
            return Result<FileData, FileError>.Failure(
                new FileError("FALLBACK_PARSE_ERROR", 
                    $"Even fallback parsing failed: {ex.Message}"));
        }
    }
    
    private Result<FileData, FileError> ParseXml(string content)
    {
        // XML parsing logic
        // ...
        return Result<FileData, FileError>.Success(new FileData
        {
            Id = Guid.NewGuid(),
            Name = "XML Data",
            Values = new List<int> { 10, 20, 30 }
        });
    }
    
    private Result<FileData, FileError> ParseJson(string content)
    {
        // JSON parsing logic
        // ...
        return Result<FileData, FileError>.Success(new FileData
        {
            Id = Guid.NewGuid(),
            Name = "JSON Data",
            Values = new List<int> { 100, 200, 300 }
        });
    }
}

public class FileData
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public List<int> Values { get; set; } = new List<int>();
}

public class TransformedData
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public DateTime ProcessedDate { get; set; }
    public List<int> Values { get; set; } = new List<int>();
}

public class ProcessedFile
{
    public Guid OriginalId { get; set; }
    public string OutputPath { get; set; }
    public DateTime ProcessedDate { get; set; }
}

public class FileError
{
    public string Code { get; set; }
    public string Message { get; set; }
    
    public FileError(string code, string message)
    {
        Code = code;
        Message = message;
    }
}
```

## Advanced Patterns

### Combining Multiple Results

```csharp
public static class ResultCombinators
{
    public static Result<(T1, T2), TError> Combine<T1, T2, TError>(
        Result<T1, TError> result1,
        Result<T2, TError> result2)
    {
        if (result1.IsFailure)
            return Result<(T1, T2), TError>.Failure(result1.Error);
            
        if (result2.IsFailure)
            return Result<(T1, T2), TError>.Failure(result2.Error);
            
        return Result<(T1, T2), TError>.Success((result1.Value, result2.Value));
    }
    
    public static Result<(T1, T2, T3), TError> Combine<T1, T2, T3, TError>(
        Result<T1, TError> result1,
        Result<T2, TError> result2,
        Result<T3, TError> result3)
    {
        var combined = Combine(result1, result2);
        
        if (combined.IsFailure)
            return Result<(T1, T2, T3), TError>.Failure(combined.Error);
            
        if (result3.IsFailure)
            return Result<(T1, T2, T3), TError>.Failure(result3.Error);
            
        return Result<(T1, T2, T3), TError>.Success(
            (combined.Value.Item1, combined.Value.Item2, result3.Value));
    }
    
    public static Result<IEnumerable<T>, TError> Sequence<T, TError>(
        IEnumerable<Result<T, TError>> results)
    {
        var values = new List<T>();
        
        foreach (var result in results)
        {
            if (result.IsFailure)
                return Result<IEnumerable<T>, TError>.Failure(result.Error);
                
            values.Add(result.Value);
        }
        
        return Result<IEnumerable<T>, TError>.Success(values);
    }
    
    public static Result<IEnumerable<T>, IEnumerable<TError>> Traverse<T, TError>(
        IEnumerable<Result<T, TError>> results)
    {
        var successes = new List<T>();
        var failures = new List<TError>();
        
        foreach (var result in results)
        {
            if (result.IsSuccess)
                successes.Add(result.Value);
            else
                failures.Add(result.Error);
        }
        
        return failures.Any()
            ? Result<IEnumerable<T>, IEnumerable<TError>>.Failure(failures)
            : Result<IEnumerable<T>, IEnumerable<TError>>.Success(successes);
    }
}
```

### Async Operations with Task

```csharp
public static class AsyncResultExtensions
{
    public static async Task<Result<T, TError>> MapAsync<T, TError, U>(
        this Result<T, TError> result,
        Func<T, Task<U>> mapper)
    {
        if (result.IsFailure)
            return Result<U, TError>.Failure(result.Error);
            
        var newValue = await mapper(result.Value).ConfigureAwait(false);
        return Result<U, TError>.Success(newValue);
    }
    
    public static async Task<Result<U, TError>> BindAsync<T, U, TError>(
        this Result<T, TError> result,
        Func<T, Task<Result<U, TError>>> binder)
    {
        if (result.IsFailure)
            return Result<U, TError>.Failure(result.Error);
            
        return await binder(result.Value).ConfigureAwait(false);
    }
    
    public static async Task<Result<T, TError>> TapAsync<T, TError>(
        this Result<T, TError> result,
        Func<T, Task> action)
    {
        if (result.IsSuccess)
            await action(result.Value).ConfigureAwait(false);
            
        return result;
    }
    
    public static Task<Result<T, TError>> AsTask<T, TError>(
        this Result<T, TError> result)
    {
        return Task.FromResult(result);
    }
    
    public static async Task<Result<T, TError>> FromTask<T, TError>(
        Task<T> task,
        Func<Exception, TError> errorMapper)
    {
        try
        {
            var value = await task.ConfigureAwait(false);
            return Result<T, TError>.Success(value);
        }
        catch (Exception ex)
        {
            return Result<T, TError>.Failure(errorMapper(ex));
        }
    }
}
```

## Best Practices

### 1. Use Specific Error Types

Instead of using string for errors, create specific error types:

```csharp
public abstract class DomainError
{
    public string Message { get; }
    
    protected DomainError(string message)
    {
        Message = message;
    }
}

public class ValidationError : DomainError
{
    public string Field { get; }
    
    public ValidationError(string field, string message) 
        : base(message)
    {
        Field = field;
    }
}

public class NotFoundError : DomainError
{
    public string EntityType { get; }
    public string EntityId { get; }
    
    public NotFoundError(string entityType, string entityId) 
        : base($"{entityType} with ID '{entityId}' not found")
    {
        EntityType = entityType;
        EntityId = entityId;
    }
}
```

### 2. Create Domain-Specific Result Types

```csharp
public class DomainResult<T> : Result<T, DomainError>
{
    public static DomainResult<T> Success(T value) 
        => new SuccessResult<T, DomainError>(value) as DomainResult<T>;
        
    public static DomainResult<T> Failure(DomainError error) 
        => new FailureResult<T, DomainError>(error) as DomainResult<T>;
        
    public static DomainResult<T> NotFound(string entityType, string entityId) 
        => Failure(new NotFoundError(entityType, entityId));
        
    public static DomainResult<T> Invalid(string field, string message) 
        => Failure(new ValidationError(field, message));
}
```

### 3. Use Extension Methods for Domain-Specific Operations

```csharp
public static class DomainResultExtensions
{
    public static Result<T, DomainError> EnsureExists<T>(
        this Result<T, DomainError> result,
        Func<T, bool> predicate,
        string entityType,
        string entityId)
    {
        return result.Bind(value => 
            predicate(value) 
                ? Result<T, DomainError>.Success(value) 
                : DomainResult<T>.NotFound(entityType, entityId));
    }
    
    public static Result<T, DomainError> EnsureValid<T>(
        this Result<T, DomainError> result,
        Func<T, bool> predicate,
        string field,
        string message)
    {
        return result.Bind(value => 
            predicate(value) 
                ? Result<T, DomainError>.Success(value) 
                : DomainResult<T>.Invalid(field, message));
    }
}
```

### 4. Use Builder Pattern for Complex Flows

```csharp
public class UserRegistrationFlow
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService;
    
    public UserRegistrationFlow(IUserRepository userRepository, IEmailService emailService)
    {
        _userRepository = userRepository;
        _emailService = emailService;
    }
    
    public Result<User, DomainError> Register(string email, string password)
    {
        return Result<RegistrationRequest, DomainError>
            .Success(new RegistrationRequest(email, password))
            .Bind(ValidateRequest)
            .Bind(CheckUserDoesNotExist)
            .Bind(CreateUser)
            .Bind(SendWelcomeEmail);
    }
    
    private Result<RegistrationRequest, DomainError> ValidateRequest(RegistrationRequest request)
    {
        return Result<RegistrationRequest, DomainError>.Success(request)
            .EnsureValid(r => !string.IsNullOrEmpty(r.Email), "email", "Email is required")
            .EnsureValid(r => r.Email.Contains("@"), "email", "Invalid email format")
            .EnsureValid(r => !string.IsNullOrEmpty(r.Password), "password", "Password is required")
            .EnsureValid(r => r.Password.Length >= 8, "password", "Password must be at least 8 characters");
    }
    
    private Result<RegistrationRequest, DomainError> CheckUserDoesNotExist(RegistrationRequest request)
    {
        if (_userRepository.ExistsByEmail(request.Email))
        {
            return DomainResult<RegistrationRequest>.Invalid("email", "Email already registered");
        }
        
        return Result<RegistrationRequest, DomainError>.Success(request);
    }
    
    private Result<User, DomainError> CreateUser(RegistrationRequest request)
    {
        try
        {
            var user = new User
            {
                Email = request.Email,
                PasswordHash = HashPassword(request.Password)
            };
            
            _userRepository.Save(user);
            
            return Result<User, DomainError>.Success(user);
        }
        catch (Exception ex)
        {
            return DomainResult<User>.Failure(
                new ServiceError("Failed to create user", ex));
        }
    }
    
    private Result<User, DomainError> SendWelcomeEmail(User user)
    {
        try
        {
            _emailService.SendWelcomeEmail(user.Email);
            return Result<User, DomainError>.Success(user);
        }
        catch (Exception ex)
        {
            // Continue with success but log error
            Console.WriteLine($"Failed to send welcome email: {ex.Message}");
            return Result<User, DomainError>.Success(user);
        }
    }
    
    private string HashPassword(string password)
    {
        // Password hashing logic
        return Convert.ToBase64String(
            System.Security.Cryptography.SHA256.Create()
                .ComputeHash(System.Text.Encoding.UTF8.GetBytes(password))
        );
    }
    
    private record RegistrationRequest(string Email, string Password);
}

public class ServiceError : DomainError
{
    public Exception Exception { get; }
    
    public ServiceError(string message, Exception exception) 
        : base(message)
    {
        Exception = exception;
    }
}
```

### 5. Testing ROP Flows

```csharp
[TestClass]
public class UserRegistrationTests
{
    [TestMethod]
    public void ValidRegistration_ShouldSucceed()
    {
        // Arrange
        var mockRepo = new MockUserRepository();
        var mockEmail = new MockEmailService();
        var flow = new UserRegistrationFlow(mockRepo, mockEmail);
        
        // Act
        var result = flow.Register("test@example.com", "password123");
        
        // Assert
        Assert.IsTrue(result.IsSuccess);
        Assert.AreEqual("test@example.com", result.Value.Email);
        Assert.IsTrue(mockEmail.EmailSent);
    }
    
    [TestMethod]
    public void InvalidEmail_ShouldFail()
    {
        // Arrange
        var mockRepo = new MockUserRepository();
        var mockEmail = new MockEmailService();
        var flow = new UserRegistrationFlow(mockRepo, mockEmail);
        
        // Act
        var result = flow.Register("invalid-email", "password123");
        
        // Assert
        Assert.IsTrue(result.IsFailure);
        Assert.IsInstanceOfType(result.Error, typeof(ValidationError));
        var error = (ValidationError)result.Error;
        Assert.AreEqual("email", error.Field);
    }
    
    // Additional test methods...
}
```
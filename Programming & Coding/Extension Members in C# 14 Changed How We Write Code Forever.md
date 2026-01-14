# Extension Members in C# 14 Changed How We Write Code Forever

Extension members is my favorite feature in C# 14. They represent a modern evolution of extension methods that have been part of C# since version 3.0.

This new syntax makes it easier to add not just methods, but also properties to existing types and static members.

In this post, we will explore:

* How the extension keyword differs from traditional extension methods
* Creating instance extension properties
* Adding static extension members to types
* Working with generics and type constraints
* Real-world examples for using extension members
* Best practices for organizing extension code

Let's dive in!

---

## How The Extension Keyword Differs from Traditional Extension Methods

C# developers have used extension methods since C# 3.0 to add functionality to types without modifying their source code.

In the traditional approach, you create a static class with static methods.

Before C# 14, you created extensions by writing static methods with a `this` parameter:

```csharp
public static class StringExtensions 
{     
    public static bool IsNullOrEmpty(this string value)     
    {         
        return string.IsNullOrEmpty(value);     
    }          
    
    public static string Truncate(this string value, int maxLength)     
    {         
        if (string.IsNullOrEmpty(value) || value.Length <= maxLength)         
        {             
            return value;         
        }                      
        
        return value.Substring(0, maxLength);     
    } 
}
```

C# 14 introduces the `extension` keyword, bringing a modern approach to extending types.

The new syntax separates the receiver (the type you're extending) from the members you're adding. Instead of putting `this` on each method parameter, you declare an `extension` block that specifies the receiver once:

```csharp
public static class StringExtensions 
{     
    extension(string value)     
    {         
        public bool IsNullOrEmpty()         
        {             
            return string.IsNullOrEmpty(value);         
        }                  
        
        public string Truncate(int maxLength)         
        {             
            if (string.IsNullOrEmpty(value) || value.Length <= maxLength)             
            {                 
                return value;             
            }                              
            
            return value.Substring(0, maxLength);         
        }     
    } 
}
```

The `extension` block takes the receiver as a parameter. Inside the block, you write your methods and properties just like they were actual members of the type. The `value` parameter is available to all members within the block.

Both old and new syntax compile to identical code, so they work the same way. The new syntax is optional and designed to work alongside the traditional `this` parameter approach.

The new extension supports extending:
* Methods
* Properties
* Static methods
* Static properties

---

## Creating Instance Extension Properties

Extension properties make your code more readable and expressive. Instead of calling methods, you can use properties that feel more natural.

For example, when working with collections, you frequently check if they're empty. Instead of writing `!items.Any()` everywhere, you can create an `IsEmpty` property:

```csharp
public static class CollectionExtensions 
{     
    extension<T>(IEnumerable<T> source)     
    {         
        public bool IsEmpty => !source.Any();         
        public bool HasItems => source.Any();         
        public int Count => source.Count();     
    } 
} 

public void ProcessOrders(IEnumerable<Order> orders) 
{     
    if (orders.IsEmpty)     
    {         
        Console.WriteLine("No orders to process");         
        return;     
    }     
    
    foreach (var order in orders)     
    {         
        // Process order     
    } 
}
```

---

## Adding Static Extension Members to Types

Static extensions allow you to add factory methods or utility functions to a type, rather than to an instance. To create static extensions, use `extension` without naming the receiver parameter:

```csharp
public static class ProductExtensions 
{     
    extension(Product)     
    {         
        public static Product CreateDefault() =>             
            new Product             
            {                 
                Name = "Unnamed Product",                 
                Price = 0,                 
                StockQuantity = 0,                 
                Category = "Uncategorized",                 
                CreatedDate = DateTime.UtcNow             
            };         
            
        public static bool IsValidPrice(decimal price) =>             
            price >= 0 && price <= 1000000;         
            
        public static string DefaultCategory => "General";     
    } 
}
```

You can call these static members directly on the type:

```csharp
var product = Product.CreateDefault(); 

if (Product.IsValidPrice(999.99m)) 
{     
    product.Price = 999.99m; 
}
```

---

## Working with Generics and Type Constraints

Generic extensions let you write code that works with multiple types. This is useful when working with collections or interfaces.

Let's extend `IEnumerable<T>` to add filtering and transformation capabilities:

```csharp
public static class EnumerableExtensions 
{     
    extension<T>(IEnumerable<T> source)     
    {         
        public IEnumerable<T> WhereNotNull() =>             
            source.Where(item => item != null);                  
            
        public Dictionary<TKey, List<T>> GroupToDictionary<TKey>(             
            Func<T, TKey> keySelector) where TKey : notnull =>             
            source.GroupBy(keySelector)                   
                  .ToDictionary(g => g.Key, g => g.ToList());     
    } 
}
```

Here we add a `where TKey : notnull` constraint to the `GroupToDictionary` method. This ensures that the key selector returns a non-nullable property.

Now you can chain these extensions with LINQ methods:

```csharp
var products = _productService.GetAll(); 
var productsByCategory = products    
    .WhereNotNull()     
    .Where(p => p.IsAvailable)     
    .GroupToDictionary(p => p.Category); 

foreach (var category in productsByCategory) 
{     
    Console.WriteLine($"{category.Key}: {category.Value.Count} products"); 
}
```

You can also add type constraints to ensure your extensions only work with specific types. Here's an extension that works only with numeric types:

```csharp
public static class NumericExtensions 
{     
    extension<T>(IEnumerable<T> source)         
        where T : INumber<T>     
    {         
        public T Sum()         
        {             
            var total = T.Zero;             
            foreach (var item in source)             
            {                 
                total += item;             
            }             
            return total;         
        }                  
        
        public T Average()         
        {             
            var enumerable = source.ToList();             
            var sum = enumerable.Sum();             
            var count = T.CreateChecked(enumerable.Count);             
            return sum / count;         
        }                  
        
        public IEnumerable<T> GreaterThan(T threshold) =>             
            source.Where(x => x > threshold);     
    } 
}
```

This extension works with any numeric type that implements `INumber<T>`:

```csharp
var prices = new[] { 10.99m, 25.50m, 5.00m, 15.75m }; 
var expensiveItems = prices.GreaterThan(15.00m); 
var averagePrice = prices.Average();
```

---

## Real-World Examples for Using Extension Members

When building web APIs, you often need to extract information from `HttpContext`.

Instead of writing the same extraction code repeatedly, you can create extensions that make it cleaner:

```csharp
public static class ApiHttpContextExtensions 
{     
    extension(HttpContext context)     
    {         
        public string CorrelationId =>             
            context.Request.Headers["X-Correlation-ID"].FirstOrDefault()                 
                ?? Guid.NewGuid().ToString();         
        
        public string ClientIp =>             
            context.Request.Headers["X-Forwarded-For"].FirstOrDefault()                 
                ?? context.Connection.RemoteIpAddress?.ToString()                 
                ?? "Unknown";         
        
        public bool IsApiRequest =>             
            context.Request.Path.StartsWithSegments("/api");         
        
        public string? GetBearerToken()         
        {             
            var authHeader = context.Request.Headers["Authorization"].FirstOrDefault();             
            if (authHeader?.StartsWith("Bearer ") == true)             
            {                 
                return authHeader.Substring("Bearer ".Length).Trim();             
            }             
            return null;         
        }         
        
        public T? GetQueryParameter<T>(string key)         
        {             
            if (context.Request.Query.TryGetValue(key, out var value))             
            {                 
                try                 
                {                     
                    return (T?)Convert.ChangeType(value.ToString(), typeof(T));                 
                }                 
                catch                 
                {                     
                    return default;                 
                }             
            }             
            return default;         
        }         
        
        public void AddResponseHeader(string key, string value)         
        {             
            context.Response.Headers[key] = value;         
        }     
    }     
    
    extension(HttpContext)     
    {         
        public static bool IsValidPath(string path) =>             
            !string.IsNullOrWhiteSpace(path) && path.StartsWith("/");     
    } 
}
```

Now your middleware and controllers are much cleaner:

```csharp
public class RequestLoggingMiddleware(     
    RequestDelegate next,     
    ILogger<RequestLoggingMiddleware> logger) 
{     
    public async Task InvokeAsync(HttpContext context)     
    {         
        _logger.LogInformation(             
            "Request {Method} {Path} from {ClientIp} with CorrelationId {CorrelationId}",             
            context.Request.Method,             
            context.Request.Path,             
            context.ClientIp,             
            context.CorrelationId);                  
            
        context.AddResponseHeader("X-Correlation-ID", context.CorrelationId);                  
        
        await _next(context);     
    } 
} 

[ApiController] 
[Route("api/[controller]")] 
public class OrdersController : ControllerBase 
{     
    [HttpGet]     
    public IActionResult GetOrders(HttpContext httpContext)     
    {         
        var pageSize = httpContext.GetQueryParameter<int?>("pageSize") ?? 10;         
        var pageNumber = httpContext.GetQueryParameter<int?>("page") ?? 1;         
        var orders = orderService.GetPaged(pageNumber, pageSize);         
        return Ok(orders);     
    } 
}
```

---

## Best Practices for Organizing Extension Code

### How To Organize Extension Blocks

When using the `extension` keyword, you can group multiple extension members that apply to the same type in a single extension block. This reduces repetition and keeps related code together.

You can have multiple extension blocks in one static class if you need different receivers or different generic type parameters:

```csharp
public static class CollectionExtensions 
{     
    extension<T>(IEnumerable<T> source)     
    {         
        public bool IsEmpty => !source.Any();         
        public bool HasItems => source.Any();         
        public int Count => source.Count();     
    }     
    
    extension(IEnumerable<string> source)     
    {         
        public string JoinWithComma() => string.Join(", ", source);         
        public IEnumerable<string> NonEmpty() => source.Where(s => !string.IsNullOrEmpty(s));     
    }     
    
    extension<T>(List<T> list)     
    {         
        public void AddIfNotExists(T item)         
        {             
            if (!list.Contains(item))             
            {                 
                list.Add(item);             
            }         
        }     
    } 
}
```

You can also mix the traditional `this` parameter syntax with the new `extension` keyword syntax in the same class:

```csharp
public static class StringExtensions 
{     
    // Traditional syntax     
    public static bool IsEmail(this string value)     
    {         
        return value.Contains("@");     
    }          
    
    // New syntax     
    extension(string value)     
    {         
        public bool IsUrl =>              
            Uri.TryCreate(value, UriKind.Absolute, out _);                  
            
        public string ToTitleCase() =>             
            CultureInfo.CurrentCulture.TextInfo.ToTitleCase(value.ToLower());     
    } 
}
```

Avoid creating a single, giant extension class with extensions for dozens of types. Group related extensions together:

```csharp
public static class ProductExtensions 
{     
    extension(Product product)     
    {         
        // Product-specific extensions     
    } 
} 

public static class ValidationExtensions 
{     
    extension(Order value)     
    {         
        // Validation logic     
    }     
    
    extension(OrderItem value)     
    {         
        // Validation logic     
    } 
}
```

### Use Extension Properties for Calculated Values

If you're creating an extension method that takes no parameters and returns a value, consider making it a property instead:

```csharp
extension(Order order) 
{     
    public decimal TotalPrice =>          
        order.Items.Sum(item => item.Price * item.Quantity); 
}
```

### Use Meaningful Names

Extension properties should read like they're part of the original type. Avoid names that make it obvious they're extensions:

```csharp
// Good 
public bool IsEmpty => !source.Any(); 
public string DisplayPrice => price.ToString("C");
```

### Document Your Extensions

Add XML code summary on big projects to help other developers understand what your extensions do:

```csharp
extension(Product product) 
{     
    /// <summary>     
    /// Gets whether the product is currently available for purchase.     
    /// </summary>     
    /// <remarks>     
    /// A product is available if its stock quantity is greater than zero.     
    /// </remarks>     
    public bool IsAvailable => product.StockQuantity > 0; 
}
```

---

## Summary

The `extension` keyword in C# 14 is optional. Your existing extension methods continue to work without changes. However, when you need to add properties, static members or want cleaner syntax for grouping related extensions, the new approach provides a better developer experience.

As you work with C# 14 and .NET 10, experiment with extension properties in your projects. You'll find opportunities to replace methods with properties, making your code more intuitive and easier to use.

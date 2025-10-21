# Functional Programming in C# 11+: With Focus on Asynchronous Programming

## Table of Contents
1. [Introduction to Modern Functional C#](#introduction-to-modern-functional-c)
2. [C# 11+ Features for Functional Programming](#c-11-features-for-functional-programming)
3. [First-Class Functions with Modern Delegates](#first-class-functions-with-modern-delegates)
4. [Immutability in Modern C#](#immutability-in-modern-c)
5. [Pattern Matching Enhancements](#pattern-matching-enhancements)
6. [Records and Record Structs](#records-and-record-structs)
7. [Required Properties and Init-Only Setters](#required-properties-and-init-only-setters)
8. [Modern Function Composition](#modern-function-composition)
9. [Collection Expressions](#collection-expressions)
10. [Asynchronous Programming Fundamentals](#asynchronous-programming-fundamentals)
11. [Task-based Asynchronous Pattern](#task-based-asynchronous-pattern)
12. [ValueTask and Performance Optimization](#valuetask-and-performance-optimization)
13. [Combining Functional Patterns with Async](#combining-functional-patterns-with-async)
14. [Asynchronous Streams](#asynchronous-streams)
15. [Parallel Processing with PLINQ and PFX](#parallel-processing-with-plinq-and-pfx)
16. [Functional Error Handling with Tasks](#functional-error-handling-with-tasks)
17. [Real-World Applications](#real-world-applications)
18. [Best Practices](#best-practices)

## Introduction to Modern Functional C#

C# has evolved dramatically in recent versions, bringing powerful functional programming capabilities to the language. C# 11 and above include features that make functional programming more natural and expressive.

### Key Functional Programming Features in Modern C#:

- **Records and Record Structs**: Immutable reference and value types with value-based equality
- **Required Properties**: Ensuring object initialization without mutable setters
- **Pattern Matching**: Enhanced pattern matching for more expressive code
- **Raw String Literals**: Cleaner multi-line string declarations
- **Collection Expressions**: Concise syntax for creating collections
- **Static Abstract Members in Interfaces**: Enabling more functional abstractions
- **Async Streams**: Processing asynchronous data sequences functionally

### Evolution of C# for Functional Programming:
```csharp
    // C# 11+ allows for more concise, expressive functional code

    // Record types with primary constructor for immutability
    public record Person(string Name, int Age);

    // Record struct for immutable value types
    public readonly record struct Point(double X, double Y);

    // Required properties ensure initialization
    public class Configuration
    {
        public required string ApiKey { get; init; }
        public required string Endpoint { get; init; }
    }

    // Raw string literals for cleaner multi-line strings
    var query = """
        SELECT *
        FROM Users
        WHERE LastLogin > @lastLogin
        ORDER BY Name
        """;

    // Collection expressions (C# 12+)
    var numbers = [1, 2, 3, 4, 5];
    var words = ["hello", "world"];

    // Pattern matching enhancements
    public static string GetShapeDescription(Shape shape) => shape switch
    {
        Circle { Radius: > 10 } => "Large circle",
        Rectangle { Width: var w, Height: var h } when w == h => "Square",
        Rectangle { Width: > 10, Height: > 10 } => "Large rectangle",
        Triangle { Points: [var p1, var p2, var p3] } => "Triangle with 3 points",
        _ => "Other shape"
    };
```
## C# 11+ Features for Functional Programming

C# 11 and later versions introduce several features that enhance functional programming capabilities.

### Raw String Literals

```csharp
Raw string literals make it easier to represent strings with special characters without escaping:

    // Raw string literals with triple quotes
    var json = """
    {
        "name": "John Doe",
        "age": 30,
        "skills": ["C#", "F#", "JavaScript"]
    }
    """;

    // Raw interpolated strings
    var name = "Alice";
    var greeting = $$"""
    Hello, {{name}}!
    Today is: {{DateTime.Now:yyyy-MM-dd}}
    """;
```
### List Patterns

```csharp
List patterns allow matching against sequence elements:

    public static string AnalyzeSequence<T>(T[] sequence) => sequence switch
    {
        [] => "Empty sequence",
        [var first] => $"Single item: {first}",
        [var first, var second] => $"Two items: {first}, {second}",
        [var first, _, var last] => $"At least three items, starting with {first} and ending with {last}",
        [var first, .. var middle, var last] => $"Sequence with first={first}, last={last}, and {middle.Length} items in between"
    };

    // Usage
    Console.WriteLine(AnalyzeSequence(new[] { 1, 2, 3, 4, 5 })); 
    // "Sequence with first=1, last=5, and 3 items in between"

```
### Static Abstract Members in Interfaces

C# 11 introduces static abstract members in interfaces, enabling more functional abstractions:
```csharp
    public interface IAddable<T> where T : IAddable<T>
    {
        static abstract T operator +(T left, T right);
        static abstract T Zero { get; }
    }

    public readonly record struct ComplexNumber(double Real, double Imaginary) : IAddable<ComplexNumber>
    {
        public static ComplexNumber operator +(ComplexNumber left, ComplexNumber right) =>
            new(left.Real + right.Real, left.Imaginary + right.Imaginary);
            
        public static ComplexNumber Zero => new(0, 0);
    }

    // Generic sum function using the static interface
    public static T Sum<T>(IEnumerable<T> items) where T : IAddable<T>
    {
        T sum = T.Zero;
        foreach (var item in items)
        {
            sum += item;
        }
        return sum;
    }

    // Usage
    var numbers = new[]
    {
        new ComplexNumber(1, 2),
        new ComplexNumber(3, 4),
        new ComplexNumber(5, 6)
    };

    var total = Sum(numbers); // ComplexNumber(9, 12)
```
### Required Members

The `required` modifier ensures properties must be initialized:
```csharp
    public class ApiClient
    {
        public required string ApiKey { get; init; }
        public required Uri BaseUrl { get; init; }
        public TimeSpan Timeout { get; init; } = TimeSpan.FromSeconds(30);
        
        public async Task<HttpResponseMessage> SendRequestAsync(HttpRequestMessage request)
        {
            using var client = new HttpClient { BaseAddress = BaseUrl };
            client.DefaultRequestHeaders.Add("X-API-Key", ApiKey);
            client.Timeout = Timeout;
            
            return await client.SendAsync(request);
        }
    }

    // Usage with object initializer - compiler ensures required properties are set
    var client = new ApiClient
    {
        ApiKey = "your-api-key",
        BaseUrl = new Uri("https://api.example.com")
    };
```
### UTF-8 String Literals

C# 11 supports UTF-8 string literals for more efficient encoding:
```csharp
    // UTF-8 string literal
    ReadOnlySpan<byte> utf8Hello = "Hello, world!"u8;

    // Useful for APIs that require UTF-8 encoded bytes
    await File.WriteAllBytesAsync("message.txt", "Hello, world!"u8.ToArray());

    // HTTP client with UTF-8 JSON
    using var client = new HttpClient();
    using var request = new HttpRequestMessage(HttpMethod.Post, "https://api.example.com");
    request.Content = new StringContent("""{"message":"Hello"}"""u8.ToArray(), 
        null, "application/json");
```
## First-Class Functions with Modern Delegates

C# treats functions as first-class citizens through delegates, lambdas, and recent enhancements.

### Modern Lambda Expressions
```csharp
    // Lambda with inferred parameters and return type
    var add = (int a, int b) => a + b;

    // Lambda with explicit return type
    var divide = double (double a, double b) => a / b;

    // Lambda with attributes
    var validate = [NotNull] (string s) => !string.IsNullOrEmpty(s);

    // Lambda with ref parameters (C# 12+)
    var swap = (ref int x, ref int y) =>
    {
        (y, x) = (x, y);
    };

    // Optional parameters in lambdas (C# 12+)
    var format = (string s, bool uppercase = false) =>
        uppercase ? s.ToUpper() : s;

    // Static lambdas
    var square = static (int x) => x * x;

    // Natural delegates
    var naturals = (int x, int y) => x + y;
    int result = naturals(5, 3); // 8
```
### Modern Function Types
```csharp
    // Newer patterns prefer Func/Action over custom delegates
    public class FunctionalProcessor
    {
        // Function that takes and returns functions
        public static Func<T, R2> Compose<T, R1, R2>(
            Func<T, R1> first,
            Func<R1, R2> second) => x => second(first(x));
        
        // Higher-order function with delegate inference
        public static void ProcessItems<T>(
            IEnumerable<T> items, 
            Action<T> processor)
        {
            foreach (var item in items)
            {
                processor(item);
            }
        }
        
        // Passing function to function
        public static List<R> Map<T, R>(
            IEnumerable<T> items, 
            Func<T, R> mapper)
        {
            var result = new List<R>();
            foreach (var item in items)
            {
                result.Add(mapper(item));
            }
            return result;
        }
    }

    // Usage
    var numbers = [1, 2, 3, 4, 5];
    var doubled = FunctionalProcessor.Map(numbers, x => x * 2);
```
### Function Type Inference

C# 11+ provides better type inference for delegates:
```csharp
    // Better lambda inference
    var people = new List<Person>();

    // The type of "adult" is inferred to be Func<Person, bool>
    var adult = p => p.Age >= 18;

    // Query with inferred delegate types
    var adultNames = people
        .Where(adult)
        .Select(p => p.Name)
        .ToList();

    // Inferred generic delegate types
    public static TResult Apply<T, TResult>(T value, Func<T, TResult> func) => func(value);

    // Type is inferred without specifying generic parameters
    var result = Apply(42, x => x.ToString());
```
## Immutability in Modern C#

Immutability (unchangeable state) is central to functional programming and well-supported in modern C#.

### Records for Immutable Reference Types
```csharp
    // Basic record with primary constructor
    public record Person(string Name, int Age);

    // Creating and using a record
    var alice = new Person("Alice", 30);
    Console.WriteLine(alice); // Person { Name = Alice, Age = 30 }

    // Records have value-based equality
    var alice2 = new Person("Alice", 30);
    Console.WriteLine(alice == alice2); // True

    // Non-destructive mutation with 'with' expressions
    var olderAlice = alice with { Age = 31 };
    Console.WriteLine(alice.Age);     // 30 (original unchanged)
    Console.WriteLine(olderAlice.Age); // 31 (new instance)
```
### Record Types with Additional Features
```csharp
    // Record with additional members and validation
    public record User(string Username, string Email)
    {
        // Init-only property
        public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
        
        // Method
        public string GetDisplayName() => Username;
        
        // Constructor with validation
        public User(string Username, string Email) : this(Username, Email)
        {
            if (string.IsNullOrEmpty(Username))
                throw new ArgumentException("Username cannot be empty", nameof(Username));
                
            if (!Email.Contains('@'))
                throw new ArgumentException("Invalid email format", nameof(Email));
        }
    }

    // Record inheritance
    public record AdminUser(string Username, string Email, string[] Permissions)
        : User(Username, Email);
```
### Record Structs
```csharp
Record structs are value types with record-like features:

    // Basic record struct
    public readonly record struct Point(double X, double Y)
    {
        // Add a method to calculate distance
        public double DistanceTo(Point other)
        {
            var dx = X - other.X;
            var dy = Y - other.Y;
            return Math.Sqrt(dx * dx + dy * dy);
        }
    }

    // Usage
    var p1 = new Point(0, 0);
    var p2 = new Point(3, 4);
    Console.WriteLine(p2.DistanceTo(p1)); // 5

    // With-expressions work on record structs too
    var p3 = p2 with { X = 5 };
```
### Immutable Collections
```csharp
    using System.Collections.Immutable;

    // Create immutable collections
    var immutableList = ImmutableList<int>.Empty.AddRange([1, 2, 3, 4, 5]);
    var immutableDict = ImmutableDictionary<string, int>.Empty
        .Add("one", 1)
        .Add("two", 2);

    // "Modifications" create new instances
    var newList = immutableList.Add(6); // Original is unchanged
    var newDict = immutableDict.SetItem("two", 22); // Original is unchanged

    // More concise creation with collection expressions (C# 12+)
    ImmutableList<int> list = [1, 2, 3, 4, 5].ToImmutableList();
    ImmutableDictionary<string, int> dict = new Dictionary<string, int>
    {
        ["one"] = 1,
        ["two"] = 2
    }.ToImmutableDictionary();
```
## Pattern Matching Enhancements

C# 11+ includes powerful pattern matching features that enable more functional code styles.

### Extended Property Patterns
```csharp
    public record Address(string Street, string City, string Country);
    public record Customer(string Name, Address Address, bool IsVip);

    // Nested property pattern matching
    public static string AnalyzeCustomer(Customer customer) => customer switch
    {
        // Property patterns with nested properties
        { IsVip: true, Address.Country: "USA" } => "VIP US customer",
        { Address: { City: "London" } } => "London-based customer",
        { Address.Country: "France" or "Italy" or "Spain" } => "European customer",
        { Name: var name } when name.StartsWith("A") => "Customer with name starting with A",
        _ => "Regular customer"
    };
```
### List and Array Patterns
```csharp
    // Pattern matching on arrays and lists
    public static string DescribeList<T>(List<T> items) => items switch
    {
        [] => "Empty list",
        [var only] => $"List with single item: {only}",
        [var first, var second] => $"List with two items: {first} and {second}",
        [var first, _, var last] => $"List with at least 3 items, starting with {first} and ending with {last}",
        [_, _, .. var middle, _] => $"List with at least 3 items and {middle.Count} items in the middle"
    };

    // Pattern matching on array types
    public static string GetArrayType<T>(T[] array) => array switch
    {
        int[] => "Integer array",
        string[] { Length: 0 } => "Empty string array",
        string[] [var first, ..] when first.Length > 5 => "String array starting with a long string",
        double[] => "Double array",
        _ => "Other array type"
    };
```
### Extended Type Patterns
```csharp
    // Type patterns with additional constraints
    public static string ProcessValue(object value) => value switch
    {
        null => "Null value",
        string { Length: 0 } => "Empty string",
        string s => $"String: {s}",
        int n when n < 0 => "Negative integer",
        int n => $"Integer: {n}",
        IEnumerable<int> list when !list.Any() => "Empty integer list",
        IEnumerable<int> list => $"Integer list with {list.Count()} elements",
        System.IO.Stream => "Stream object",
        _ => "Unknown type"
    };
```
### Relational Patterns
```csharp
    // Relational patterns for numeric comparisons
    public static string GradeScore(int score) => score switch
    {
        >= 90 => "A",
        >= 80 => "B",
        >= 70 => "C",
        >= 60 => "D",
        _ => "F"
    };

    // Combining relational and logical patterns
    public static string ClassifyTemperature(double temp) => temp switch
    {
        < 0 => "Freezing",
        >= 0 and < 15 => "Cold",
        >= 15 and < 25 => "Comfortable",
        >= 25 and < 35 => "Hot",
        >= 35 => "Very hot"
    };
```
## Records and Record Structs

Records and record structs are ideal for functional programming as they enable immutable data modeling.

### Advanced Record Features
```csharp
    // Positional and nominal record syntax
    public record Product(int Id, string Name, decimal Price)
    {
        // Computed property
        public decimal PriceWithTax => Price * 1.2m;
        
        // Custom formatting
        public override string ToString() => $"{Name} (${Price:F2})";
        
        // Deconstruct custom types
        public void Deconstruct(out string Name, out decimal Price)
        {
            Name = this.Name;
            Price = this.Price;
        }
    }

    // Usage
    var product = new Product(1, "Laptop", 1200);

    // Value equality
    var sameProduct = new Product(1, "Laptop", 1200);
    Console.WriteLine(product == sameProduct); // True

    // Deconstruction
    var (name, price) = product;
    Console.WriteLine($"{name}: ${price}"); // Laptop: $1200.00

    // Positional matching
    string GetProductCategory(Product p) => p switch
    {
        (_, "Laptop", > 1000) => "Premium laptop",
        (_, "Laptop", _) => "Regular laptop",
        (_, _, < 50) => "Budget item",
        _ => "Regular product"
    };
```
### Sealed and Unsealed Records
```csharp
    // Base record
    public record Vehicle(string Make, string Model, int Year);

    // Derived records
    public record Car(string Make, string Model, int Year, int Doors) 
        : Vehicle(Make, Model, Year);
        
    public record ElectricCar(string Make, string Model, int Year, int Doors, int BatteryCapacity) 
        : Car(Make, Model, Year, Doors);

    // Sealed record prevents inheritance
    public sealed record ApiCredential(string Key, string Secret);

    // Usage with pattern matching
    string DescribeVehicle(Vehicle v) => v switch
    {
        ElectricCar { BatteryCapacity: > 75 } => "Long-range electric car",
        ElectricCar e => $"Electric car with {e.BatteryCapacity} kWh battery",
        Car { Doors: 2 } => "Coupe",
        Car { Doors: 4 } => "Sedan",
        _ => "Other vehicle"
    };
```
### Performance-Focused Record Structs
```csharp
    // Regular record struct
    public record struct Coordinate(double X, double Y);

    // Readonly record struct for better performance
    public readonly record struct Vector3D(double X, double Y, double Z)
    {
        // Methods
        public double Length => Math.Sqrt(X * X + Y * Y + Z * Z);
        
        // Operator overloading for functional operations
        public static Vector3D operator +(Vector3D a, Vector3D b) =>
            new(a.X + b.X, a.Y + b.Y, a.Z + b.Z);
        
        public static Vector3D operator *(Vector3D v, double scalar) =>
            new(v.X * scalar, v.Y * scalar, v.Z * scalar);
    }

    // Usage
    var v1 = new Vector3D(1, 2, 3);
    var v2 = new Vector3D(4, 5, 6);
    var v3 = v1 + v2 * 2; // Vector3D(9, 12, 15)
```
## Required Properties and Init-Only Setters

Modern C# enables immutability while ensuring proper initialization of objects.

### Init-Only Properties
```csharp
    // Class with init-only properties
    public class ImmutableConfiguration
    {
        // Init-only properties can only be set during initialization
        public string ServerUrl { get; init; }
        public int Port { get; init; }
        public TimeSpan Timeout { get; init; }
        
        // Regular constructor
        public ImmutableConfiguration(string serverUrl, int port)
        {
            ServerUrl = serverUrl;
            Port = port;
            Timeout = TimeSpan.FromSeconds(30);
        }
    }

    // Usage with object initializers
    var config = new ImmutableConfiguration("https://api.example.com", 443)
    {
        Timeout = TimeSpan.FromSeconds(60)
    };

    // Cannot modify after initialization
    // config.Timeout = TimeSpan.FromSeconds(90); // Compilation error
```
### Required Properties
```csharp
    // Class with required properties
    public class ApiConfiguration
    {
        // These properties must be initialized
        public required string ApiKey { get; init; }
        public required Uri Endpoint { get; init; }
        
        // Optional property with default
        public TimeSpan Timeout { get; init; } = TimeSpan.FromSeconds(30);
        
        // Method using the properties
        public HttpClient CreateClient()
        {
            var client = new HttpClient
            {
                BaseAddress = Endpoint,
                Timeout = Timeout
            };
            client.DefaultRequestHeaders.Add("X-API-Key", ApiKey);
            return client;
        }
    }

    // Usage - compiler ensures all required properties are set
    var config = new ApiConfiguration
    {
        ApiKey = "your-api-key",
        Endpoint = new Uri("https://api.example.com")
    };

    // SatisfiesRequiredMembers attribute for custom constructors
    public class DatabaseConfig
    {
        public required string ConnectionString { get; init; }
        public required string DatabaseName { get; init; }
        
        [SetsRequiredMembers]
        public DatabaseConfig(string server, string database, string user, string password)
        {
            ConnectionString = $"Server={server};Database={database};User={user};Password={password}";
            DatabaseName = database;
        }
    }
```
### Validation with Required Properties
```csharp
    // Combining required properties with validation
    public class EmailMessage
    {
        private string _to = string.Empty;
        private string _subject = string.Empty;
        
        public required string To
        {
            get => _to;
            init
            {
                if (!value.Contains('@'))
                    throw new ArgumentException("Invalid email format", nameof(To));
                _to = value;
            }
        }
        
        public required string Subject
        {
            get => _subject;
            init
            {
                if (string.IsNullOrEmpty(value))
                    throw new ArgumentException("Subject cannot be empty", nameof(Subject));
                _subject = value;
            }
        }
        
        public string Body { get; init; } = string.Empty;
    }

    // Usage with validation
    try
    {
        var message = new EmailMessage
        {
            To = "user@example.com",
            Subject = "Hello from C# 11"
        };
    }
    catch (ArgumentException ex)
    {
        Console.WriteLine($"Validation error: {ex.Message}");
    }
```
## Modern Function Composition

Functional programming emphasizes composing small functions to build complex operations.

### Pipe Operator Simulation
```csharp
    // Pipeline extension method
    public static class PipelineExtensions
    {
        // Simple pipe method
        public static TOutput Pipe<TInput, TOutput>(
            this TInput input, Func<TInput, TOutput> func)
        {
            return func(input);
        }
        
        // Async pipe method
        public static async Task<TOutput> PipeAsync<TInput, TOutput>(
            this Task<TInput> inputTask, Func<TInput, Task<TOutput>> func)
        {
            var input = await inputTask;
            return await func(input);
        }
    }

    // Using the pipeline
    int result = 5
        .Pipe(x => x * 2)   // 10
        .Pipe(x => x + 1)   // 11
        .Pipe(x => x * x);  // 121

    // Multi-step pipeline with different types
    string greeting = "hello"
        .Pipe(s => s.ToUpper())           // "HELLO"
        .Pipe(s => s + ", WORLD!")        // "HELLO, WORLD!"
        .Pipe(s => s.Length)              // 13
        .Pipe(n => $"Length is {n}");     // "Length is 13"
```
### Functional Composition with Generic Extensions
```csharp
    // Generic composition utilities
    public static class FunctionalExtensions
    {
        // Compose two functions: f(g(x))
        public static Func<T1, T3> Compose<T1, T2, T3>(
            this Func<T2, T3> f, Func<T1, T2> g)
        {
            return x => f(g(x));
        }
        
        // Curry a function: f(a, b) -> f(a)(b)
        public static Func<T1, Func<T2, TResult>> Curry<T1, T2, TResult>(
            this Func<T1, T2, TResult> func)
        {
            return a => b => func(a, b);
        }
        
        // Partial application: fix first parameter
        public static Func<T2, TResult> PartialApply<T1, T2, TResult>(
            this Func<T1, T2, TResult> func, T1 first)
        {
            return second => func(first, second);
        }
    }

    // Using the composition utilities
    Func<int, int> double = x => x * 2;
    Func<int, int> increment = x => x + 1;

    // Compose: first double, then increment
    Func<int, int> doubleAndIncrement = increment.Compose(double);
    Console.WriteLine(doubleAndIncrement(5));  // 11 (5*2 + 1)

    // Curry a function
    Func<int, int, int> add = (a, b) => a + b;
    Func<int, Func<int, int>> curriedAdd = add.Curry();
    Console.WriteLine(curriedAdd(5)(3));  // 8

    // Partial application
    Func<int, int> add5 = add.PartialApply(5);
    Console.WriteLine(add5(10));  // 15
```
### Practical Pipeline Example
```csharp
    // Processing pipeline for data transformation
    public static class UserProcessor
    {
        // Chain of functions with pipe
        public static List<UserDto> ProcessUsers(List<User> users)
        {
            return users
                .Pipe(FilterActive)
                .Pipe(FilterAdults)
                .Pipe(SortByName)
                .Pipe(MapToDto)
                .Pipe(LimitResults);
        }

        // Individual pipeline steps
        private static List<User> FilterActive(List<User> users) =>
            users.Where(u => u.IsActive).ToList();

        private static List<User> FilterAdults(List<User> users) =>
            users.Where(u => u.Age >= 18).ToList();

        private static List<User> SortByName(List<User> users) =>
            users.OrderBy(u => u.Name).ToList();

        private static List<UserDto> MapToDto(List<User> users) =>
            users.Select(u => new UserDto
            {
                Id = u.Id,
                Name = u.Name,
                Email = u.Email
            }).ToList();

        private static List<UserDto> LimitResults(List<UserDto> dtos) =>
            dtos.Take(10).ToList();
    }

    // Record types for the pipeline
    public record User(int Id, string Name, string Email, int Age, bool IsActive);
    public record UserDto
    {
        public required int Id { get; init; }
        public required string Name { get; init; }
        public required string Email { get; init; }
    }
```
## Collection Expressions

C# 12 introduces collection expressions, a concise syntax for creating and initializing collections.

### Basic Collection Expressions
```csharp
    // Create arrays with collection expressions
    int[] numbers = [1, 2, 3, 4, 5];
    string[] words = ["hello", "world"];

    // Create lists
    List<int> numberList = [1, 2, 3, 4, 5];
    List<string> wordList = ["hello", "world"];

    // Create spans and readonly spans
    Span<int> span = [1, 2, 3, 4, 5];
    ReadOnlySpan<byte> bytes = [0x01, 0x02, 0x03, 0x04];

    // Mix collection expressions
    int[] combined = [.. numbers, 6, 7, .. [8, 9, 10]];
    // Result: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
### Functional Patterns with Collection Expressions
```csharp
    // Function that generates a range
    public static int[] Range(int start, int count)
    {
        int[] result = new int[count];
        for (int i = 0; i < count; i++)
        {
            result[i] = start + i;
        }
        return result;
    }

    // Using spread operator with functions
    int[] sequence = [1, .. Range(2, 3), 5, 6];
    // Result: [1, 2, 3, 4, 5, 6]

    // Conditional collection construction with spreads
    bool includeExtras = true;
    int[] allNumbers = [
        1, 2, 3,
        .. (includeExtras ? [4, 5, 6] : [])
    ];

    // Return collections from expressions
    public static List<string> GetUsernames(bool includeAdmins = false) => [
        "user1", 
        "user2", 
        "user3",
        .. (includeAdmins ? ["admin1", "admin2"] : [])
    ];
```
### Creating Custom Collection Types
```csharp
    // Create a custom collection type
    public class OrderedSet<T> : ICollection<T>
    {
        private readonly List<T> _items = [];
        private readonly HashSet<T> _set = [];
        
        // Implement collection initialization
        public void Add(T item)
        {
            if (_set.Add(item))
            {
                _items.Add(item);
            }
        }
        
        // Other ICollection<T> members
        public void Clear()
        {
            _items.Clear();
            _set.Clear();
        }
        
        public bool Contains(T item) => _set.Contains(item);
        
        public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
        
        public bool Remove(T item)
        {
            if (_set.Remove(item))
            {
                _items.Remove(item);
                return true;
            }
            return false;
        }
        
        public int Count => _items.Count;
        
        public bool IsReadOnly => false;
        
        public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
        
        System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => 
            _items.GetEnumerator();
    }

    // Using custom collection with collection expression
    OrderedSet<string> uniqueNames = ["Alice", "Bob", "Alice", "Charlie"];
    // Contains: "Alice", "Bob", "Charlie" (duplicates removed while preserving order)
```
## Asynchronous Programming Fundamentals

Asynchronous programming is essential in modern C# applications for better resource utilization.

### Modern Async/Await Pattern
```csharp
    // Basic async method pattern
    public async Task<string> DownloadWebpageAsync(string url)
    {
        using var client = new HttpClient();
        Console.WriteLine("Starting download...");
        string result = await client.GetStringAsync(url);
        Console.WriteLine("Download completed.");
        return result;
    }

    // Calling the async method
    public async Task ProcessDataAsync()
    {
        try
        {
            string data = await DownloadWebpageAsync("https://example.com");
            Console.WriteLine($"Downloaded {data.Length} characters");
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Download failed: {ex.Message}");
        }
    }
```
### Async Method Return Types
```csharp
    // Task - for asynchronous operations without a return value
    public async Task SaveLogAsync(string message)
    {
        await Task.Delay(100);  // Simulate I/O
        await File.AppendAllTextAsync("log.txt", $"{DateTime.Now}: {message}{Environment.NewLine}");
    }

    // Task<T> - for asynchronous operations that return a value
    public async Task<int> CountWordsAsync(string url)
    {
        string content = await DownloadWebpageAsync(url);
        return content.Split(new[] { ' ', '\n', '\r', '\t' }, 
            StringSplitOptions.RemoveEmptyEntries).Length;
    }

    // ValueTask<T> - for potentially synchronous operations (optimization)
    public ValueTask<int> GetCachedValueAsync(string key)
    {
        // Check if value is in cache
        if (_cache.TryGetValue(key, out var value))
        {
            return new ValueTask<int>(value);  // Return immediately without allocation
        }
        
        // Fall back to asynchronous operation
        return new ValueTask<int>(FetchValueSlowlyAsync(key));
    }

    private async Task<int> FetchValueSlowlyAsync(string key)
    {
        await Task.Delay(1000);  // Simulate slow operation
        var value = new Random().Next(100);
        _cache[key] = value;
        return value;
    }
```
### Modern Error Handling with Async
```csharp
    // Multiple exception handling with async/await
    public async Task<string> FetchUserDataAsync(string userId)
    {
        try
        {
            // Try multiple operations
            var user = await _userRepository.GetByIdAsync(userId);
            var settings = await _settingsRepository.GetForUserAsync(userId);
            var permissions = await _permissionsService.GetUserPermissionsAsync(userId);
            
            return JsonSerializer.Serialize(new 
            {
                User = user,
                Settings = settings,
                Permissions = permissions
            });
        }
        catch (NotFoundException ex)
        {
            // Handle not found
            _logger.LogWarning(ex, "Resource not found for user {UserId}", userId);
            throw new UserNotFoundException($"User data not found: {ex.Message}", ex);
        }
        catch (UnauthorizedException ex)
        {
            // Handle unauthorized
            _logger.LogWarning(ex, "Unauthorized access for user {UserId}", userId);
            throw new UserAccessDeniedException("Access denied to user data", ex);
        }
        catch (Exception ex) when (ex is HttpRequestException or TimeoutException)
        {
            // Handle network issues with pattern matching
            _logger.LogError(ex, "Network error for user {UserId}", userId);
            throw new UserDataFetchException("Network error while fetching user data", ex);
        }
    }
```
### Task Continuation
```csharp
    // Modern task continuation with async/await
    public async Task ProcessOrderAsync(Order order)
    {
        // Sequential async operations with await
        await ValidateOrderAsync(order);
        await ProcessPaymentAsync(order);
        await UpdateInventoryAsync(order);
        await SendConfirmationEmailAsync(order);
    }

    // When needed, use WhenAll for parallel operations
    public async Task ProcessMultipleOrdersAsync(List<Order> orders)
    {
        var tasks = orders.Select(ProcessOrderAsync);
        await Task.WhenAll(tasks);
    }

    // WhenAny for completing when any task finishes
    public async Task<string> SearchWithTimeoutAsync(string query)
    {
        var databaseTask = SearchDatabaseAsync(query);
        var timeoutTask = Task.Delay(2000).ContinueWith(_ => (string)null);
        
        var completedTask = await Task.WhenAny(databaseTask, timeoutTask);
        
        if (completedTask == timeoutTask)
        {
            return "Search timed out";
        }
        
        return await databaseTask;
    }
```
## Task-based Asynchronous Pattern

The Task-based Asynchronous Pattern (TAP) is the modern approach to asynchrony in C#.

### Task Creation and Composition
```csharp
    // Creating tasks
    public async Task DemonstrateTaskCreationAsync()
    {
        // From a running operation
        Task<int> calculationTask = Task.Run(() => 
        {
            Console.WriteLine("Calculating...");
            Thread.Sleep(100);  // Simulate work
            return 42;
        });
        
        // From an already known value
        Task<string> completedTask = Task.FromResult("Instant result");
        
        // Creating a failed task
        Task failedTask = Task.FromException(new InvalidOperationException("Task failed"));
        
        // Creating a canceled task
        var cts = new CancellationTokenSource();
        cts.Cancel();
        Task canceledTask = Task.FromCanceled(cts.Token);
        
        // Wait for the running task and get result
        int result = await calculationTask;
        Console.WriteLine($"Result: {result}");
    }
```
### Modern Task Combinators
```csharp
    // Task.WhenAll - run multiple tasks in parallel
    public async Task<IEnumerable<string>> DownloadMultiplePagesAsync(IEnumerable<string> urls)
    {
        var downloadTasks = urls.Select(url => DownloadWebpageAsync(url));
        return await Task.WhenAll(downloadTasks);
    }

    // Task.WhenAny - get the first completed task
    public async Task<string> GetFastestResultAsync()
    {
        var task1 = SlowOperationAsync("Source 1", 500);
        var task2 = SlowOperationAsync("Source 2", 1000);
        var task3 = SlowOperationAsync("Source 3", 300);
        
        var firstCompletedTask = await Task.WhenAny(task1, task2, task3);
        return await firstCompletedTask;
    }

    // Simulation methods
    private async Task<string> SlowOperationAsync(string source, int delay)
    {
        await Task.Delay(delay);
        return $"Result from {source}";
    }
```
### Functional Error Handling with Tasks
```csharp
    // Result type for functional error handling
    public class Result<TSuccess, TFailure>
    {
        private readonly TSuccess _success;
        private readonly TFailure _failure;
        private readonly bool _isSuccess;

        private Result(TSuccess success, TFailure failure, bool isSuccess)
        {
            _success = success;
            _failure = failure;
            _isSuccess = isSuccess;
        }

        public static Result<TSuccess, TFailure> Success(TSuccess success) => 
            new(success, default, true);
            
        public static Result<TSuccess, TFailure> Failure(TFailure failure) => 
            new(default, failure, false);

        public bool IsSuccess => _isSuccess;
        public bool IsFailure => !_isSuccess;
        public TSuccess SuccessValue => IsSuccess ? _success : throw new InvalidOperationException("Cannot access success value of a failure result");
        public TFailure FailureValue => IsFailure ? _failure : throw new InvalidOperationException("Cannot access failure value of a success result");
        
        public TResult Match<TResult>(
            Func<TSuccess, TResult> onSuccess, 
            Func<TFailure, TResult> onFailure) =>
            IsSuccess ? onSuccess(_success) : onFailure(_failure);
    }

    // Async method with Result return type
    public async Task<Result<User, string>> GetUserAsync(int userId)
    {
        try
        {
            var user = await _userRepository.GetByIdAsync(userId);
            
            if (user == null)
                return Result<User, string>.Failure($"User with ID {userId} not found");
                
            return Result<User, string>.Success(user);
        }
        catch (Exception ex)
        {
            return Result<User, string>.Failure($"Failed to get user: {ex.Message}");
        }
    }

    // Usage with pattern matching
    public async Task DisplayUserAsync(int userId)
    {
        var result = await GetUserAsync(userId);
        
        string message = result.Match(
            user => $"Found user: {user.Name}",
            error => $"Error: {error}"
        );
        
        Console.WriteLine(message);
    }
```
## ValueTask and Performance Optimization

`ValueTask<T>` optimizes asynchronous code by eliminating allocations in specific scenarios.

### When to Use ValueTask
```csharp
    // Task<T> creates an allocation even when result is immediate
    public Task<int> GetValueTaskAsync()
    {
        // Already have the result, but still allocates a Task<int>
        return Task.FromResult(42);
    }

    // ValueTask<T> avoids allocation when result is immediate
    public ValueTask<int> GetValueEfficientAsync()
    {
        // No allocation for the immediate result
        return new ValueTask<int>(42);
    }

    // Conditional allocation with ValueTask
    public ValueTask<string> GetCachedDataAsync(string key)
    {
        // Check if data is in cache
        if (_cache.TryGetValue(key, out string value))
        {
            // Return immediately without allocation
            return new ValueTask<string>(value);
        }
        
        // Fall back to Task-based asynchronous operation
        return new ValueTask<string>(FetchFromDatabaseAsync(key));
    }

    private async Task<string> FetchFromDatabaseAsync(string key)
    {
        await Task.Delay(100);  // Simulate database access
        var result = $"Data for {key}";
        _cache[key] = result;
        return result;
    }
```
### ValueTask Performance Considerations
```csharp
    // IMPORTANT: ValueTask should only be awaited once
    public async Task DemonstrateValueTaskUsageAsync()
    {
        ValueTask<int> valueTask = GetValueAsync();
        
        // CORRECT: Await once
        int result = await valueTask;
        
        // INCORRECT: Multiple awaits on same ValueTask
        // int result1 = await valueTask;  // Works first time
        // int result2 = await valueTask;  // May throw or return incorrect result
        
        // INCORRECT: Accessing Result without checking IsCompleted
        // int directResult = valueTask.Result;  // May deadlock
        
        // CORRECT: Check IsCompleted before accessing Result
        if (valueTask.IsCompleted)
        {
            int completedResult = valueTask.Result;
        }
    }

    private async ValueTask<int> GetValueAsync()
    {
        await Task.Delay(100);
        return 42;
    }
```
### Combining ValueTask with Other Optimizations
```csharp
    // Optimized caching service
    public class CachingService
    {
        private readonly Dictionary<string, object> _cache = new();
        private readonly SemaphoreSlim _lock = new(1);
        
        // Optimized async caching method
        public async ValueTask<T> GetOrCreateAsync<T>(
            string key, 
            Func<Task<T>> factory,
            TimeSpan? expiration = null)
        {
            // Fast path - cache hit
            if (_cache.TryGetValue(key, out var cachedItem))
            {
                var entry = (CacheEntry<T>)cachedItem;
                if (!entry.IsExpired)
                {
                    return new ValueTask<T>(entry.Value);
                }
            }
            
            // Slow path - cache miss, acquire lock
            await _lock.WaitAsync();
            try
            {
                // Double check after acquiring lock
                if (_cache.TryGetValue(key, out cachedItem))
                {
                    var entry = (CacheEntry<T>)cachedItem;
                    if (!entry.IsExpired)
                    {
                        return new ValueTask<T>(entry.Value);
                    }
                }
                
                // Create new value
                T value = await factory();
                
                // Store in cache
                _cache[key] = new CacheEntry<T>(
                    value, 
                    expiration.HasValue ? DateTime.UtcNow.Add(expiration.Value) : (DateTime?)null
                );
                
                return new ValueTask<T>(value);
            }
            finally
            {
                _lock.Release();
            }
        }
        
        private class CacheEntry<T>
        {
            public T Value { get; }
            public DateTime? ExpiresAt { get; }
            
            public bool IsExpired => ExpiresAt.HasValue && DateTime.UtcNow > ExpiresAt;
            
            public CacheEntry(T value, DateTime? expiresAt)
            {
                Value = value;
                ExpiresAt = expiresAt;
            }
        }
    }
```
## Combining Functional Patterns with Async

Modern C# allows elegant combinations of functional programming and asynchronous execution.

### Async Function Composition
```csharp
    // Extension methods for async function composition
    public static class AsyncFunctionalExtensions
    {
        // Async pipe method
        public static async Task<TOutput> PipeAsync<TInput, TOutput>(
            this Task<TInput> input, 
            Func<TInput, Task<TOutput>> func)
        {
            return await func(await input);
        }
        
        // Async compose method
        public static Func<T1, Task<T3>> ComposeAsync<T1, T2, T3>(
            this Func<T2, Task<T3>> f, 
            Func<T1, Task<T2>> g)
        {
            return async x => await f(await g(x));
        }
    }

    // Using async function composition
    public async Task ProcessUserOrderAsync(int userId)
    {
        // Define async functions
        Func<int, Task<User>> getUser = async id => 
            await _userRepository.GetByIdAsync(id);
            
        Func<User, Task<Order[]>> getOrders = async user => 
            await _orderRepository.GetOrdersForUserAsync(user.Id);
            
        Func<Order[], Task<OrderSummary>> createSummary = async orders =>
        {
            await Task.Delay(10); // Simulate processing
            return new OrderSummary
            {
                OrderCount = orders.Length,
                TotalAmount = orders.Sum(o => o.Amount),
                LastOrderDate = orders.Length > 0 
                    ? orders.Max(o => o.OrderDate) 
                    : (DateTime?)null
            };
        };
        
        // Compose async functions
        var getUserOrderSummary = getUser
            .ComposeAsync(getOrders)
            .ComposeAsync(createSummary);
        
        // Execute the composed async function
        var summary = await getUserOrderSummary(userId);
        
        Console.WriteLine($"User has {summary.OrderCount} orders totaling {summary.TotalAmount:C}");
    }

    public class OrderSummary
    {
        public int OrderCount { get; init; }
        public decimal TotalAmount { get; init; }
        public DateTime? LastOrderDate { get; init; }
    }
```
### Railway-Oriented Programming with Async
```csharp
    // Extension methods for Railway-oriented async programming
    public static class AsyncResultExtensions
    {
        // Bind for Task<Result<>>
        public static async Task<Result<TNewSuccess, TFailure>> BindAsync<TSuccess, TNewSuccess, TFailure>(
            this Task<Result<TSuccess, TFailure>> task,
            Func<TSuccess, Task<Result<TNewSuccess, TFailure>>> func)
        {
            var result = await task;
            if (result.IsSuccess)
                return await func(result.SuccessValue);
            return Result<TNewSuccess, TFailure>.Failure(result.FailureValue);
        }
        
        // Map for Task<Result<>>
        public static async Task<Result<TNewSuccess, TFailure>> MapAsync<TSuccess, TNewSuccess, TFailure>(
            this Task<Result<TSuccess, TFailure>> task,
            Func<TSuccess, TNewSuccess> func)
        {
            var result = await task;
            if (result.IsSuccess)
                return Result<TNewSuccess, TFailure>.Success(func(result.SuccessValue));
            return Result<TNewSuccess, TFailure>.Failure(result.FailureValue);
        }
    }

    // Example services
    public class UserService
    {
        public async Task<Result<User, string>> GetUserAsync(int userId)
        {
            try
            {
                await Task.Delay(100); // Simulate DB call
                
                if (userId <= 0)
                    return Result<User, string>.Failure("Invalid user ID");
                    
                if (userId == 999) // Simulate user not found
                    return Result<User, string>.Failure("User not found");
                    
                return Result<User, string>.Success(new User
                {
                    Id = userId,
                    Name = $"User {userId}",
                    Email = $"user{userId}@example.com"
                });
            }
            catch (Exception ex)
            {
                return Result<User, string>.Failure($"Error retrieving user: {ex.Message}");
            }
        }
        
        public async Task<Result<Order[], string>> GetOrdersAsync(User user)
        {
            try
            {
                await Task.Delay(100); // Simulate DB call
                
                if (user.Id % 3 == 0) // Simulate no orders
                    return Result<Order[], string>.Failure("No orders found");
                    
                return Result<Order[], string>.Success(new[]
                {
                    new Order { Id = 1, UserId = user.Id, Amount = 100.00m },
                    new Order { Id = 2, UserId = user.Id, Amount = 200.00m }
                });
            }
            catch (Exception ex)
            {
                return Result<Order[], string>.Failure($"Error retrieving orders: {ex.Message}");
            }
        }
    }

    // Usage of Railway-oriented async programming
    public async Task ProcessUserOrdersAsync(int userId)
    {
        var userService = new UserService();
        
        var result = await userService.GetUserAsync(userId)
            .BindAsync(user => userService.GetOrdersAsync(user))
            .MapAsync(orders => new
            {
                OrderCount = orders.Length,
                TotalAmount = orders.Sum(o => o.Amount)
            });
        
        string message = result.Match(
            summary => $"User has {summary.OrderCount} orders totaling {summary.TotalAmount:C}",
            error => $"Processing failed: {error}"
        );
        
        Console.WriteLine(message);
    }

    // Models
    public class User
    {
        public required int Id { get; init; }
        public required string Name { get; init; }
        public required string Email { get; init; }
    }

    public class Order
    {
        public required int Id { get; init; }
        public required int UserId { get; init; }
        public required decimal Amount { get; init; }
        public DateTime OrderDate { get; init; } = DateTime.UtcNow;
    }
```
## Asynchronous Streams

C# 8.0+ supports async streams for handling asynchronous sequences of data efficiently.

### Basic Async Streams
```csharp
    // Generate an async stream of data
    public static async IAsyncEnumerable<int> GenerateNumbersAsync(
        int count, 
        int delay = 100)
    {
        for (int i = 0; i < count; i++)
        {
            await Task.Delay(delay);  // Simulate async work
            yield return i;
        }
    }

    // Consume an async stream
    public static async Task ConsumeNumbersAsync()
    {
        await foreach (var number in GenerateNumbersAsync(5))
        {
            Console.WriteLine($"Received: {number}");
        }
    }
```
### LINQ-Style Operations on Async Streams
```csharp
    // Extension methods for IAsyncEnumerable<T>
    public static class AsyncEnumerableExtensions
    {
        // Where for async streams
        public static async IAsyncEnumerable<T> WhereAsync<T>(
            this IAsyncEnumerable<T> source,
            Func<T, bool> predicate)
        {
            await foreach (var item in source)
            {
                if (predicate(item))
                {
                    yield return item;
                }
            }
        }
        
        // Where with async predicate
        public static async IAsyncEnumerable<T> WhereAsync<T>(
            this IAsyncEnumerable<T> source,
            Func<T, Task<bool>> predicate)
        {
            await foreach (var item in source)
            {
                if (await predicate(item))
                {
                    yield return item;
                }
            }
        }
        
        // Select for async streams
        public static async IAsyncEnumerable<TResult> SelectAsync<TSource, TResult>(
            this IAsyncEnumerable<TSource> source,
            Func<TSource, TResult> selector)
        {
            await foreach (var item in source)
            {
                yield return selector(item);
            }
        }
        
        // Select with async selector
        public static async IAsyncEnumerable<TResult> SelectAsync<TSource, TResult>(
            this IAsyncEnumerable<TSource> source,
            Func<TSource, Task<TResult>> selector)
        {
            await foreach (var item in source)
            {
                yield return await selector(item);
            }
        }
        
        // ToListAsync
        public static async Task<List<T>> ToListAsync<T>(
            this IAsyncEnumerable<T> source)
        {
            var result = new List<T>();
            await foreach (var item in source)
            {
                result.Add(item);
            }
            return result;
        }
    }

    // Using the extensions
    public async Task ProcessDataStreamAsync()
    {
        var numbers = GenerateNumbersAsync(100)
            .WhereAsync(n => n % 2 == 0)              // Only even numbers
            .SelectAsync(n => n * n)                  // Square them
            .WhereAsync(async n => {
                await Task.Delay(10);                 // Simulate async check
                return n < 1000;                      // Only values less than 1000
            });
        
        await foreach (var number in numbers)
        {
            Console.WriteLine($"Processed value: {number}");
        }
    }
```
### Real-World Async Stream Example
```csharp
    // Database repository with async streams
    public class ProductRepository
    {
        private readonly string _connectionString;
        
        public ProductRepository(string connectionString)
        {
            _connectionString = connectionString;
        }
        
        // Stream data from database without loading everything into memory
        public async IAsyncEnumerable<Product> GetProductsStreamAsync(
            string category = null,
            decimal? minPrice = null)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.OpenAsync();
            
            string query = "SELECT Id, Name, Category, Price FROM Products";
            List<string> whereConditions = new();
            
            if (!string.IsNullOrEmpty(category))
            {
                whereConditions.Add("Category = @Category");
            }
            
            if (minPrice.HasValue)
            {
                whereConditions.Add("Price >= @MinPrice");
            }
            
            if (whereConditions.Count > 0)
            {
                query += " WHERE " + string.Join(" AND ", whereConditions);
            }
            
            using var command = new SqlCommand(query, connection);
            
            if (!string.IsNullOrEmpty(category))
            {
                command.Parameters.AddWithValue("@Category", category);
            }
            
            if (minPrice.HasValue)
            {
                command.Parameters.AddWithValue("@MinPrice", minPrice.Value);
            }
            
            using var reader = await command.ExecuteReaderAsync();
            
            while (await reader.ReadAsync())
            {
                yield return new Product
                {
                    Id = reader.GetInt32(0),
                    Name = reader.GetString(1),
                    Category = reader.GetString(2),
                    Price = reader.GetDecimal(3)
                };
            }
        }
    }

    // Product model
    public class Product
    {
        public required int Id { get; init; }
        public required string Name { get; init; }
        public required string Category { get; init; }
        public required decimal Price { get; init; }
    }

    // Using the async stream repository
    public async Task ProcessProductsAsync()
    {
        var repository = new ProductRepository("connection_string_here");
        
        // Process products one at a time without loading all into memory
        decimal totalValue = 0;
        int count = 0;
        
        await foreach (var product in repository.GetProductsStreamAsync(
            category: "Electronics", 
            minPrice: 100))
        {
            totalValue += product.Price;
            count++;
            
            // Process each product
            Console.WriteLine($"Processing: {product.Name} - ${product.Price}");
        }
        
        Console.WriteLine($"Processed {count} products with total value: ${totalValue}");
    }
```
## Parallel Processing with PLINQ and PFX

For CPU-bound operations, combining functional programming with parallelism is powerful.

### PLINQ for Parallel Data Processing
```csharp
    // Basic PLINQ
    public void DemonstratePlinq()
    {
        var numbers = Enumerable.Range(1, 1000000);
        
        // Sequential LINQ
        var evenSquaresLinq = numbers
            .Where(n => n % 2 == 0)
            .Select(n => n * n)
            .Take(10)
            .ToList();
        
        // Parallel LINQ
        var evenSquaresPlinq = numbers
            .AsParallel()                 // Process in parallel
            .Where(n => n % 2 == 0)
            .Select(n => n * n)
            .OrderBy(n => n)              // Ensure ordering (parallel operations may change order)
            .Take(10)                     // Take is order-dependent
            .ToList();
        
        // Control degree of parallelism
        var customPlinq = numbers
            .AsParallel()
            .WithDegreeOfParallelism(Environment.ProcessorCount)
            .Where(n => n % 2 == 0);
    }
```
### Parallel ForEach with Functional Style
```csharp
    // Parallel processing with functional approach
    public async Task ProcessImagesInParallelAsync(string[] imagePaths)
    {
        // Define image processing pipeline
        Func<string, Task<byte[]>> loadImage = async path => 
            await File.ReadAllBytesAsync(path);
            
        Func<byte[], Task<byte[]>> applyFilter = async imageData => 
        {
            await Task.Delay(100);  // Simulate CPU-intensive operation
            // Apply filter to image...
            return imageData;
        };
        
        Func<byte[], string, Task> saveImage = async (imageData, path) => 
            await File.WriteAllBytesAsync(
                Path.Combine("processed", Path.GetFileName(path)), 
                imageData);
        
        // Process images in parallel
        var options = new ParallelOptions 
        { 
            MaxDegreeOfParallelism = Environment.ProcessorCount 
        };
        
        await Parallel.ForEachAsync(
            imagePaths, 
            options,
            async (path, token) => 
            {
                try
                {
                    var imageData = await loadImage(path);
                    var filteredData = await applyFilter(imageData);
                    await saveImage(filteredData, path);
                    
                    Console.WriteLine($"Processed: {path}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Error processing {path}: {ex.Message}");
                }
            });
    }
```
### Task Parallel Library with Functional Approaches
```csharp
    // Parallel processing with TPL Dataflow
    public async Task ProcessDataWithDataflowAsync(IEnumerable<string> inputs)
    {
        var options = new ExecutionDataflowBlockOptions
        {
            MaxDegreeOfParallelism = Environment.ProcessorCount
        };
        
        // Create pipeline blocks
        var loadBlock = new TransformBlock<string, Data>(
            async input => 
            {
                Console.WriteLine($"Loading: {input}");
                await Task.Delay(100);  // Simulate I/O
                return new Data { Source = input, Value = input.Length };
            }, 
            options);
        
        var processBlock = new TransformBlock<Data, Result>(
            async data => 
            {
                Console.WriteLine($"Processing: {data.Source}");
                await Task.Delay(200);  // Simulate CPU work
                return new Result { Source = data.Source, Value = data.Value * 2 };
            }, 
            options);
        
        var saveBlock = new ActionBlock<Result>(
            async result => 
            {
                Console.WriteLine($"Saving: {result.Source}, Value: {result.Value}");
                await Task.Delay(100);  // Simulate I/O
            }, 
            options);
        
        // Link the blocks
        loadBlock.LinkTo(processBlock, new DataflowLinkOptions { PropagateCompletion = true });
        processBlock.LinkTo(saveBlock, new DataflowLinkOptions { PropagateCompletion = true });
        
        // Post input data
        foreach (var input in inputs)
        {
            await loadBlock.SendAsync(input);
        }
        
        // Mark completion
        loadBlock.Complete();
        
        // Wait for completion
        await saveBlock.Completion;
    }

    // Data models
    public record Data
    {
        public required string Source { get; init; }
        public required int Value { get; init; }
    }

    public record Result
    {
        public required string Source { get; init; }
        public required int Value { get; init; }
    }
```
## Functional Error Handling with Tasks

Functional approaches to error handling are particularly valuable in asynchronous code.

### Option Pattern with Tasks
```csharp
    // Option type for nullable values
    public class Option<T>
    {
        private readonly T _value;
        private readonly bool _hasValue;

        private Option(T value, bool hasValue)
        {
            _value = value;
            _hasValue = hasValue;
        }

        public static Option<T> Some(T value) => new(value, true);
        public static Option<T> None() => new(default, false);

        public bool HasValue => _hasValue;
        public T Value => _hasValue ? _value : throw new InvalidOperationException("Option has no value");

        public TResult Match<TResult>(Func<T, TResult> some, Func<TResult> none) =>
            _hasValue ? some(_value) : none();
    }

    // Extensions for async operations with Option
    public static class OptionAsyncExtensions
    {
        // Map/Select
        public static async Task<Option<TResult>> MapAsync<T, TResult>(
            this Task<Option<T>> optionTask,
            Func<T, TResult> mapper)
        {
            var option = await optionTask;
            return option.Match(
                value => Option<TResult>.Some(mapper(value)),
                () => Option<TResult>.None()
            );
        }
        
        // Bind/SelectMany
        public static async Task<Option<TResult>> BindAsync<T, TResult>(
            this Task<Option<T>> optionTask,
            Func<T, Task<Option<TResult>>> binder)
        {
            var option = await optionTask;
            return option.Match(
                async value => await binder(value),
                () => Task.FromResult(Option<TResult>.None())
            ).Result;
        }
    }

    // Using Option with async operations
    public class UserRepository
    {
        public async Task<Option<User>> FindUserAsync(int id)
        {
            await Task.Delay(100);  // Simulate database query
            
            if (id <= 0)
                return Option<User>.None();
                
            if (id == 999)  // Simulate not found
                return Option<User>.None();
                
            return Option<User>.Some(new User
            {
                Id = id,
                Name = $"User {id}",
                Email = $"user{id}@example.com"
            });
        }
    }

    // Using the Option with async
    public async Task ProcessUserAsync(int userId)
    {
        var repository = new UserRepository();
        
        var userNameOption = await repository.FindUserAsync(userId)
            .MapAsync(user => user.Name);
        
        string message = userNameOption.Match(
            name => $"Found user: {name}",
            () => "User not found"
        );
        
        Console.WriteLine(message);
    }
```
### Either/Result Pattern with Tasks
```csharp
    // Result type for operations that might fail
    public class Result<TSuccess, TFailure>
    {
        private readonly TSuccess _success;
        private readonly TFailure _failure;
        private readonly bool _isSuccess;

        private Result(TSuccess success, TFailure failure, bool isSuccess)
        {
            _success = success;
            _failure = failure;
            _isSuccess = isSuccess;
        }

        public static Result<TSuccess, TFailure> Success(TSuccess success) => 
            new(success, default, true);
            
        public static Result<TSuccess, TFailure> Failure(TFailure failure) => 
            new(default, failure, false);

        public bool IsSuccess => _isSuccess;
        public bool IsFailure => !_isSuccess;
        
        public TSuccess SuccessValue => IsSuccess 
            ? _success 
            : throw new InvalidOperationException("Cannot access success value of a failure result");
            
        public TFailure FailureValue => IsFailure 
            ? _failure 
            : throw new InvalidOperationException("Cannot access failure value of a success result");
        
        public TResult Match<TResult>(
            Func<TSuccess, TResult> onSuccess, 
            Func<TFailure, TResult> onFailure) =>
            IsSuccess ? onSuccess(_success) : onFailure(_failure);
    }

    // Extensions for async operations with Result
    public static class ResultAsyncExtensions
    {
        // Map the success value
        public static async Task<Result<TNewSuccess, TFailure>> MapAsync<TSuccess, TNewSuccess, TFailure>(
            this Task<Result<TSuccess, TFailure>> resultTask,
            Func<TSuccess, TNewSuccess> mapper)
        {
            var result = await resultTask;
            return result.IsSuccess
                ? Result<TNewSuccess, TFailure>.Success(mapper(result.SuccessValue))
                : Result<TNewSuccess, TFailure>.Failure(result.FailureValue);
        }
        
        // Bind/SelectMany
        public static async Task<Result<TNewSuccess, TFailure>> BindAsync<TSuccess, TNewSuccess, TFailure>(
            this Task<Result<TSuccess, TFailure>> resultTask,
            Func<TSuccess, Task<Result<TNewSuccess, TFailure>>> binder)
        {
            var result = await resultTask;
            return result.IsSuccess
                ? await binder(result.SuccessValue)
                : Result<TNewSuccess, TFailure>.Failure(result.FailureValue);
        }
        
        // Map the failure value
        public static async Task<Result<TSuccess, TNewFailure>> MapFailureAsync<TSuccess, TFailure, TNewFailure>(
            this Task<Result<TSuccess, TFailure>> resultTask,
            Func<TFailure, TNewFailure> mapper)
        {
            var result = await resultTask;
            return result.IsSuccess
                ? Result<TSuccess, TNewFailure>.Success(result.SuccessValue)
                : Result<TSuccess, TNewFailure>.Failure(mapper(result.FailureValue));
        }
    }

    // Using Result with async operations
    public class OrderService
    {
        // Return Result type from async method
        public async Task<Result<Order, string>> CreateOrderAsync(OrderRequest request)
        {
            await Task.Delay(100);  // Simulate processing
            
            // Validate request
            if (request.Items.Count == 0)
                return Result<Order, string>.Failure("Order must contain at least one item");
                
            if (string.IsNullOrEmpty(request.CustomerName))
                return Result<Order, string>.Failure("Customer name is required");
            
            // Create order
            var order = new Order
            {
                Id = new Random().Next(1000, 9999),
                CustomerName = request.CustomerName,
                Items = request.Items.ToList(),
                TotalAmount = request.Items.Sum(i => i.Price * i.Quantity),
                CreatedAt = DateTime.UtcNow
            };
            
            // Simulate saving to database
            await Task.Delay(100);
            
            return Result<Order, string>.Success(order);
        }
        
        public async Task<Result<PaymentResult, string>> ProcessPaymentAsync(Order order)
        {
            await Task.Delay(200);  // Simulate payment processing
            
            if (order.TotalAmount > 1000)
                return Result<PaymentResult, string>.Failure("Amount exceeds maximum payment limit");
                
            var paymentResult = new PaymentResult
            {
                OrderId = order.Id,
                TransactionId = Guid.NewGuid().ToString(),
                Amount = order.TotalAmount,
                Status = "Completed"
            };
            
            return Result<PaymentResult, string>.Success(paymentResult);
        }
    }

    // Usage with railway-oriented programming
    public async Task PlaceOrderAsync(OrderRequest request)
    {
        var orderService = new OrderService();
        
        var result = await orderService.CreateOrderAsync(request)
            .BindAsync(order => orderService.ProcessPaymentAsync(order))
            .MapAsync(payment => new OrderConfirmation
            {
                OrderId = payment.OrderId,
                TransactionId = payment.TransactionId,
                Amount = payment.Amount,
                Status = "Confirmed"
            });
        
        string message = result.Match(
            confirmation => $"Order {confirmation.OrderId} confirmed with transaction {confirmation.TransactionId}",
            error => $"Order failed: {error}"
        );
        
        Console.WriteLine(message);
    }

    // Models
    public class OrderRequest
    {
        public required string CustomerName { get; init; }
        public required List<OrderItem> Items { get; init; } = new();
    }

    public class OrderItem
    {
        public required string ProductId { get; init; }
        public required string Name { get; init; }
        public required decimal Price { get; init; }
        public required int Quantity { get; init; }
    }

    public class Order
    {
        public required int Id { get; init; }
        public required string CustomerName { get; init; }
        public required List<OrderItem> Items { get; init; }
        public required decimal TotalAmount { get; init; }
        public required DateTime CreatedAt { get; init; }
    }

    public class PaymentResult
    {
        public required int OrderId { get; init; }
        public required string TransactionId { get; init; }
        public required decimal Amount { get; init; }
        public required string Status { get; init; }
    }

    public class OrderConfirmation
    {
        public required int OrderId { get; init; }
        public required string TransactionId { get; init; }
        public required decimal Amount { get; init; }
        public required string Status { get; init; }
    }
```
## Real-World Applications

Putting everything together in real-world scenarios.

### Functional API Client
```csharp
    // Functional REST API client
    public class FunctionalApiClient
    {
        private readonly HttpClient _httpClient;
        
        public FunctionalApiClient(string baseUrl)
        {
            _httpClient = new HttpClient { BaseAddress = new Uri(baseUrl) };
        }
        
        // Generic GET method that returns a Result
        public async Task<Result<T, ApiError>> GetAsync<T>(string endpoint)
        {
            try
            {
                var response = await _httpClient.GetAsync(endpoint);
                
                if (!response.IsSuccessStatusCode)
                {
                    var errorContent = await response.Content.ReadAsStringAsync();
                    return Result<T, ApiError>.Failure(new ApiError 
                    {
                        StatusCode = (int)response.StatusCode,
                        Message = errorContent
                    });
                }
                
                var content = await response.Content.ReadAsStringAsync();
                var result = JsonSerializer.Deserialize<T>(content, new JsonSerializerOptions
                {
                    PropertyNameCaseInsensitive = true
                });
                
                if (result == null)
                    return Result<T, ApiError>.Failure(new ApiError 
                    { 
                        StatusCode = 0, 
                        Message = "Deserialization returned null" 
                    });
                    
                return Result<T, ApiError>.Success(result);
            }
            catch (Exception ex)
            {
                return Result<T, ApiError>.Failure(new ApiError 
                {
                    StatusCode = 0,
                    Message = ex.Message
                });
            }
        }
        
        // Generic POST method that returns a Result
        public async Task<Result<TResponse, ApiError>> PostAsync<TRequest, TResponse>(
            string endpoint, TRequest data)
        {
            try
            {
                var json = JsonSerializer.Serialize(data);
                var content = new StringContent(json, Encoding.UTF8, "application/json");
                
                var response = await _httpClient.PostAsync(endpoint, content);
                
                if (!response.IsSuccessStatusCode)
                {
                    var errorContent = await response.Content.ReadAsStringAsync();
                    return Result<TResponse, ApiError>.Failure(new ApiError 
                    {
                        StatusCode = (int)response.StatusCode,
                        Message = errorContent
                    });
                }
                
                var responseContent = await response.Content.ReadAsStringAsync();
                var result = JsonSerializer.Deserialize<TResponse>(responseContent, new JsonSerializerOptions
                {
                    PropertyNameCaseInsensitive = true
                });
                
                if (result == null)
                    return Result<TResponse, ApiError>.Failure(new ApiError 
                    { 
                        StatusCode = 0, 
                        Message = "Deserialization returned null" 
                    });
                    
                return Result<TResponse, ApiError>.Success(result);
            }
            catch (Exception ex)
            {
                return Result<TResponse, ApiError>.Failure(new ApiError 
                {
                    StatusCode = 0,
                    Message = ex.Message
                });
            }
        }
    }

    public record ApiError
    {
        public required int StatusCode { get; init; }
        public required string Message { get; init; }
    }

    // Example usage
    public class UserApiService
    {
        private readonly FunctionalApiClient _client;
        
        public UserApiService(string baseUrl)
        {
            _client = new FunctionalApiClient(baseUrl);
        }
        
        public async Task<Result<User, ApiError>> GetUserAsync(int userId)
        {
            return await _client.GetAsync<User>($"users/{userId}");
        }
        
        public async Task<Result<User, ApiError>> CreateUserAsync(CreateUserRequest request)
        {
            return await _client.PostAsync<CreateUserRequest, User>("users", request);
        }
        
        public async Task ProcessUserAsync(int userId)
        {
            var result = await GetUserAsync(userId)
                .BindAsync(async user => {
                    var ordersResult = await _client.GetAsync<Order[]>($"users/{user.Id}/orders");
                    return ordersResult.MapAsync(orders => (User: user, Orders: orders));
                })
                .MapAsync(data => {
                    Console.WriteLine($"User {data.User.Name} has {data.Orders.Length} orders");
                    return "Processing complete";
                });
                
            result.Match(
                success => Console.WriteLine(success),
                error => Console.WriteLine($"Error: {error.StatusCode} - {error.Message}")
            );
        }
    }

    public record CreateUserRequest(string Name, string Email);
```
### Functional Event Processing System
```csharp
    // Event processing pipeline using functional patterns
    public class EventProcessor
    {
        // Define event types
        public record UserEvent
        {
            public required int UserId { get; init; }
            public required string EventType { get; init; }
            public required DateTime Timestamp { get; init; }
            public Dictionary<string, string> Data { get; init; } = new();
        }
        
        // Pipeline step type
        public delegate Task<Result<UserEvent, string>> EventPipelineStep(UserEvent evt);
        
        // Define pipeline steps as pure functions
        public static class EventPipeline
        {
            // Validate event data
            public static async Task<Result<UserEvent, string>> ValidateEvent(UserEvent evt)
            {
                await Task.Delay(10); // Simulate validation logic
                
                if (evt.UserId <= 0)
                    return Result<UserEvent, string>.Failure("Invalid user ID");
                    
                if (string.IsNullOrEmpty(evt.EventType))
                    return Result<UserEvent, string>.Failure("Event type is required");
                    
                return Result<UserEvent, string>.Success(evt);
            }
            
            // Enrich event with additional data
            public static async Task<Result<UserEvent, string>> EnrichEvent(UserEvent evt)
            {
                try
                {
                    await Task.Delay(20); // Simulate enrichment logic
                    
                    // Create a new event with enriched data (immutability)
                    var enrichedEvent = evt with 
                    { 
                        Data = new Dictionary<string, string>(evt.Data)
                    };
                    
                    // Add additional context data
                    enrichedEvent.Data["ProcessedBy"] = "EventPipeline";
                    enrichedEvent.Data["ProcessedAt"] = DateTime.UtcNow.ToString("o");
                    
                    return Result<UserEvent, string>.Success(enrichedEvent);
                }
                catch (Exception ex)
                {
                    return Result<UserEvent, string>.Failure($"Enrichment failed: {ex.Message}");
                }
            }
            
            // Transform event for a specific purpose
            public static async Task<Result<UserEvent, string>> TransformEvent(UserEvent evt)
            {
                await Task.Delay(15); // Simulate transformation logic
                
                // Create a new event with transformed data (immutability)
                var transformedEvent = evt with 
                { 
                    Data = evt.Data.ToDictionary(
                        kv => kv.Key.ToLowerInvariant(),
                        kv => kv.Value
                    )
                };
                
                return Result<UserEvent, string>.Success(transformedEvent);
            }
            
            // Persist event to storage
            public static async Task<Result<string, string>> PersistEvent(UserEvent evt)
            {
                try
                {
                    await Task.Delay(50); // Simulate database operation
                    
                    // In a real app, this would save to a database
                    string eventId = Guid.NewGuid().ToString();
                    
                    return Result<string, string>.Success(
                        $"Event {eventId} for user {evt.UserId} persisted successfully");
                }
                catch (Exception ex)
                {
                    return Result<string, string>.Failure($"Persistence failed: {ex.Message}");
                }
            }
        }
        
        // Combine pipeline steps using function composition
        public static EventPipelineStep CreatePipeline(params EventPipelineStep[] steps)
        {
            return async (UserEvent evt) =>
            {
                var result = Result<UserEvent, string>.Success(evt);
                
                foreach (var step in steps)
                {
                    result = await result.BindAsync(e => step(e));
                    
                    if (result.IsFailure)
                        break;
                }
                
                return result;
            };
        }
        
        // Process an event through the entire pipeline
        public static async Task<Result<string, string>> ProcessEventAsync(UserEvent evt)
        {
            // Option 1: Chain with Railway pattern
            return await EventPipeline.ValidateEvent(evt)
                .BindAsync(EventPipeline.EnrichEvent)
                .BindAsync(EventPipeline.TransformEvent)
                .BindAsync(EventPipeline.PersistEvent);
                
            // Option 2: Use the pipeline builder
            // var pipeline = CreatePipeline(
            //     EventPipeline.ValidateEvent,
            //     EventPipeline.EnrichEvent,
            //     EventPipeline.TransformEvent
            // );
            // 
            // return await pipeline(evt)
            //     .BindAsync(EventPipeline.PersistEvent);
        }
        
        // Process multiple events concurrently
        public static async Task<IEnumerable<Result<string, string>>> ProcessEventsAsync(
            IEnumerable<UserEvent> events)
        {
            var tasks = events.Select(ProcessEventAsync);
            return await Task.WhenAll(tasks);
        }
    }

    // Usage of the event processing pipeline
    public async Task DemonstrateEventProcessingAsync()
    {
        var events = new[]
        {
            new EventProcessor.UserEvent
            {
                UserId = 1,
                EventType = "LOGIN",
                Timestamp = DateTime.UtcNow,
                Data = new Dictionary<string, string> { ["IpAddress"] = "192.168.1.1" }
            },
            new EventProcessor.UserEvent
            {
                UserId = 2,
                EventType = "PURCHASE",
                Timestamp = DateTime.UtcNow,
                Data = new Dictionary<string, string> { ["Amount"] = "99.99", ["ItemId"] = "PROD-123" }
            },
            new EventProcessor.UserEvent
            {
                UserId = 0,  // Invalid user ID
                EventType = "LOGOUT",
                Timestamp = DateTime.UtcNow
            }
        };
        
        var results = await EventProcessor.ProcessEventsAsync(events);
        
        foreach (var result in results)
        {
            string message = result.Match(
                success => $"Success: {success}",
                failure => $"Failure: {failure}"
            );
            
            Console.WriteLine(message);
        }
    }
```
## Best Practices

### 1. Embrace Immutability
```csharp
    // Bad: Mutable state
    public class MutableOrder
    {
        public int Id { get; set; }
        public string CustomerName { get; set; }
        public List<OrderItem> Items { get; set; } = new();
        public decimal TotalAmount { get; set; }
        
        public void AddItem(OrderItem item)
        {
            Items.Add(item);
            TotalAmount += item.Price * item.Quantity;
        }
    }

    // Good: Immutable approach with records
    public record Order
    {
        public required int Id { get; init; }
        public required string CustomerName { get; init; }
        public required IReadOnlyList<OrderItem> Items { get; init; }
        public decimal TotalAmount => Items.Sum(i => i.Price * i.Quantity);
        
        // Non-destructive mutation returns a new instance
        public Order AddItem(OrderItem newItem)
        {
            return this with
            {
                Items = Items.Concat(new[] { newItem }).ToList()
            };
        }
    }

    public record OrderItem
    {
        public required string ProductId { get; init; }
        public required string Name { get; init; }
        public required decimal Price { get; init; }
        public required int Quantity { get; init; }
    }
```
### 2. Prefer Expression-Bodied Members
```csharp
    // Traditional methods and properties
    public class TraditionalClass
    {
        private readonly int _value;
        
        public TraditionalClass(int value)
        {
            _value = value;
        }
        
        public int GetDoubled()
        {
            return _value * 2;
        }
        
        public int Square
        {
            get
            {
                return _value * _value;
            }
        }
    }

    // Modern expression-bodied members
    public class ModernClass
    {
        private readonly int _value;
        
        public ModernClass(int value) => _value = value;
        
        public int GetDoubled() => _value * 2;
        
        public int Square => _value * _value;
        
        // Expression-bodied property with init
        public string Description { get; init; } = "Default description";
        
        // Expression-bodied method with multiple statements
        public string GetDescription() => $"{_value} squared is {Square}";
    }
```
### 3. Use Pattern Matching for Control Flow
```csharp
    // Traditional approach with if/else
    public string DescribePersonTraditional(Person person)
    {
        if (person == null)
        {
            return "No person provided";
        }
        
        if (person.Age < 18)
        {
            return $"{person.Name} is a minor";
        }
        else if (person.Age >= 65)
        {
            return $"{person.Name} is a senior";
        }
        else
        {
            return $"{person.Name} is an adult";
        }
    }

    // Modern approach with pattern matching
    public string DescribePerson(Person? person) => person switch
    {
        null => "No person provided",
        { Age: < 18 } => $"{person.Name} is a minor",
        { Age: >= 65 } => $"{person.Name} is a senior",
        _ => $"{person.Name} is an adult"
    };

    // Using pattern matching for more complex logic
    public string GetDiscountLevel(Customer customer) => customer switch
    {
        { IsVip: true } => "VIP discount",
        { Orders: { Count: > 10 } } => "Loyalty discount",
        { TotalSpend: > 1000 } => "High spender discount",
        { FirstPurchaseDate: var d } when d < DateTime.Now.AddYears(-1) => "Anniversary discount",
        _ => "No discount"
    };
```
### 4. Compose Async Functions for Clarity
```csharp
    // Bad: Nested callbacks create "callback hell"
    public async Task ProcessUserBadAsync(int userId)
    {
        var user = await _userRepository.GetByIdAsync(userId);
        
        if (user == null)
        {
            Console.WriteLine("User not found");
            return;
        }
        
        var orders = await _orderRepository.GetOrdersForUserAsync(user.Id);
        
        if (orders.Count == 0)
        {
            Console.WriteLine("No orders found");
            return;
        }
        
        var latestOrder = orders.OrderByDescending(o => o.OrderDate).First();
        var payment = await _paymentService.GetPaymentForOrderAsync(latestOrder.Id);
        
        if (payment == null)
        {
            Console.WriteLine("Payment not found");
            return;
        }
        
        await _emailService.SendReceiptAsync(user.Email, payment);
    }

    // Good: Railway-oriented programming with async
    public async Task ProcessUserGoodAsync(int userId)
    {
        var result = await _userRepository.GetByIdAsync(userId)
            .PipeAsync(async user => 
                user != null 
                    ? Result<User, string>.Success(user) 
                    : Result<User, string>.Failure("User not found"))
            .BindAsync(async user => {
                var orders = await _orderRepository.GetOrdersForUserAsync(user.Id);
                
                if (orders.Count == 0)
                    return Result<(User User, Order Order), string>.Failure("No orders found");
                    
                var latestOrder = orders.OrderByDescending(o => o.OrderDate).First();
                return Result<(User User, Order Order), string>.Success((user, latestOrder));
            })
            .BindAsync(async data => {
                var payment = await _paymentService.GetPaymentForOrderAsync(data.Order.Id);
                
                if (payment == null)
                    return Result<(User User, Payment Payment), string>.Failure("Payment not found");
                    
                return Result<(User User, Payment Payment), string>.Success((data.User, payment));
            })
            .BindAsync(async data => {
                await _emailService.SendReceiptAsync(data.User.Email, data.Payment);
                return Result<string, string>.Success("Receipt sent successfully");
            });
        
        result.Match(
            success => Console.WriteLine(success),
            error => Console.WriteLine($"Error: {error}")
        );
    }
```
### 5. Use Async Streams for Large Datasets
```csharp
    // Bad: Loading everything into memory
    public async Task<List<OrderSummary>> GetAllOrdersSummaryBadAsync()
    {
        var orders = await _orderRepository.GetAllOrdersAsync();  // Potentially millions
        var results = new List<OrderSummary>();
        
        foreach (var order in orders)
        {
            var items = await _orderItemRepository.GetItemsForOrderAsync(order.Id);
            var customer = await _customerRepository.GetByIdAsync(order.CustomerId);
            
            results.Add(new OrderSummary
            {
                OrderId = order.Id,
                CustomerName = customer.Name,
                TotalItems = items.Count,
                TotalAmount = items.Sum(i => i.Price * i.Quantity)
            });
        }
        
        return results;
    }

    // Good: Using async streams
    public async IAsyncEnumerable<OrderSummary> GetAllOrdersSummaryGoodAsync()
    {
        await foreach (var order in _orderRepository.GetAllOrdersAsyncStream())
        {
            var items = await _orderItemRepository.GetItemsForOrderAsync(order.Id);
            var customer = await _customerRepository.GetByIdAsync(order.CustomerId);
            
            yield return new OrderSummary
            {
                OrderId = order.Id,
                CustomerName = customer.Name,
                TotalItems = items.Count,
                TotalAmount = items.Sum(i => i.Price * i.Quantity)
            };
        }
    }

    // Usage
    public async Task ProcessOrderSummariesAsync()
    {
        // Process one at a time without loading all into memory
        await foreach (var summary in GetAllOrdersSummaryGoodAsync())
        {
            Console.WriteLine($"Order {summary.OrderId}: {summary.CustomerName}, " + 
                             $"{summary.TotalItems} items, ${summary.TotalAmount}");
        }
    }
```
### 6. Implement Async Interfaces Correctly
```csharp
    // Defining async interfaces
    public interface IUserRepository
    {
        // Define async methods
        Task<User?> GetByIdAsync(int id);
        Task<IEnumerable<User>> GetAllAsync();
        Task<User> CreateAsync(User user);
        Task<bool> UpdateAsync(User user);
        Task<bool> DeleteAsync(int id);
        
        // Async stream methods
        IAsyncEnumerable<User> GetActiveUsersAsync();
    }

    // Implementing async interfaces
    public class UserRepository : IUserRepository
    {
        private readonly DatabaseContext _context;
        
        public UserRepository(DatabaseContext context)
        {
            _context = context;
        }
        
        public async Task<User?> GetByIdAsync(int id)
        {
            return await _context.Users.FindAsync(id);
        }
        
        public async Task<IEnumerable<User>> GetAllAsync()
        {
            return await _context.Users.ToListAsync();
        }
        
        public async Task<User> CreateAsync(User user)
        {
            _context.Users.Add(user);
            await _context.SaveChangesAsync();
            return user;
        }
        
        public async Task<bool> UpdateAsync(User user)
        {
            _context.Users.Update(user);
            int affected = await _context.SaveChangesAsync();
            return affected > 0;
        }
        
        public async Task<bool> DeleteAsync(int id)
        {
            var user = await GetByIdAsync(id);
            if (user == null) return false;
            
            _context.Users.Remove(user);
            int affected = await _context.SaveChangesAsync();
            return affected > 0;
        }
        
        public async IAsyncEnumerable<User> GetActiveUsersAsync()
        {
            var query = _context.Users.Where(u => u.IsActive);
            
            await foreach (var user in query.AsAsyncEnumerable())
            {
                yield return user;
            }
        }
    }
```
### 7. Avoid Async Void Methods
```csharp
    // Bad: async void method
    public async void ProcessDataVoid(string filePath)
    {
        try
        {
            // If this throws, the exception cannot be caught by caller
            var data = await File.ReadAllTextAsync(filePath);
            await ProcessTextAsync(data);
        }
        catch (Exception ex)
        {
            // Exception is not propagated properly
            Console.WriteLine($"Error: {ex.Message}");
        }
    }

    // Good: async Task method
    public async Task ProcessDataAsync(string filePath)
    {
        try
        {
            var data = await File.ReadAllTextAsync(filePath);
            await ProcessTextAsync(data);
        }
        catch (Exception ex)
        {
            // Exception can be caught by caller
            Console.WriteLine($"Error: {ex.Message}");
            throw; // Re-throw to propagate to caller
        }
    }

    // Usage
    public async Task RunAsync()
    {
        try
        {
            await ProcessDataAsync("data.txt");
            Console.WriteLine("Processing completed");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Processing failed: {ex.Message}");
        }
    }
```
### 8. Use Functional Error Handling
```csharp
    // Traditional exception-based approach
    public async Task<Order> PlaceOrderTraditionalAsync(OrderRequest request)
    {
        try
        {
            // Validate request
            if (request.Items.Count == 0)
                throw new ArgumentException("Order must contain at least one item");
                
            // Create order
            var order = new Order
            {
                Id = Guid.NewGuid(),
                CustomerName = request.CustomerName,
                Items = request.Items.ToList(),
                TotalAmount = request.Items.Sum(i => i.Price * i.Quantity),
                CreatedAt = DateTime.UtcNow
            };
            
            // Process payment
            var paymentResult = await _paymentService.ProcessPaymentAsync(order);
            
            if (!paymentResult.Success)
                throw new PaymentException(paymentResult.ErrorMessage);
                
            // Save order
            await _orderRepository.SaveAsync(order);
            
            return order;
        }
        catch (ArgumentException ex)
        {
            _logger.LogWarning(ex, "Invalid order request");
            throw;
        }
        catch (PaymentException ex)
        {
            _logger.LogError(ex, "Payment failed");
            throw;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error");
            throw new OrderProcessingException("Failed to process order", ex);
        }
    }

    // Functional approach with Result type
    public async Task<Result<Order, OrderError>> PlaceOrderFunctionalAsync(OrderRequest request)
    {
        // Validate request
        if (request.Items.Count == 0)
            return Result<Order, OrderError>.Failure(new OrderError(
                ErrorType.Validation, 
                "Order must contain at least one item"));
                
        // Create order
        var order = new Order
        {
            Id = Guid.NewGuid(),
            CustomerName = request.CustomerName,
            Items = request.Items.ToList(),
            TotalAmount = request.Items.Sum(i => i.Price * i.Quantity),
            CreatedAt = DateTime.UtcNow
        };
        
        // Process payment
        try
        {
            var paymentResult = await _paymentService.ProcessPaymentAsync(order);
            
            if (!paymentResult.Success)
                return Result<Order, OrderError>.Failure(new OrderError(
                    ErrorType.Payment, 
                    paymentResult.ErrorMessage));
        }
        catch (Exception ex)
        {
            return Result<Order, OrderError>.Failure(new OrderError(
                ErrorType.System, 
                $"Payment processing error: {ex.Message}"));
        }
        
        // Save order
        try
        {
            await _orderRepository.SaveAsync(order);
        }
        catch (Exception ex)
        {
            return Result<Order, OrderError>.Failure(new OrderError(
                ErrorType.Database, 
                $"Failed to save order: {ex.Message}"));
        }
        
        return Result<Order, OrderError>.Success(order);
    }

    // Error model
    public record OrderError(ErrorType Type, string Message);

    public enum ErrorType
    {
        Validation,
        Payment,
        Database,
        System
    }
    ```
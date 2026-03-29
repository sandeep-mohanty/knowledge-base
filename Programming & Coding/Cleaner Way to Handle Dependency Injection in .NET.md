# A Cleaner Way to Handle Dependency Injection in .NET
A practical look at how source generation removes dependency injection boilerplate and improves reliability in modern .NET applications.

If we have worked with dependency injection in .NET for any amount of time, the pattern is familiar.
A new dependency is introduced.
We:
1. Add a private field
2. Add a constructor parameter
3. Assign the parameter to the field
4. Open Program.cs
5. Register the service
6. Repeat again and again

At first, it feels fine.
Then the class grows.

Two dependencies become five.
Five becomes ten.
The constructor becomes longer than the actual logic.
Eventually, Program.cs turns into a long list of registrations that nobody enjoys maintaining.
Most of us accept this as “just how DI works in .NET”.
But it does not have to be this way anymore.

### The Real Problem With Traditional DI
The issue is not dependency injection itself.
The issue is the manual work around it.
In real systems, we usually face:
* Constructors that keep growing
* Boilerplate repeated in every class
* Registration code scattered across projects
* Runtime crashes when something is forgotten

The pain compounds as the system grows.
Most of this work exists purely to wire things together.
None of it delivers business value.

### What If Constructors and Registrations Were Generated?
Instead of writing constructors manually and registering every service ourselves, we can let the compiler do it for us.
With source generators, dependencies can be injected directly into fields.
The constructor is created automatically.
Service registrations are generated automatically.
All at compile time.
No reflection.
No runtime scanning.
No performance cost.
Just real C# code produced during build.



### Injecting Dependencies Without Writing Constructors
Here is what a service can look like with source generation:

```csharp
public partial class OrderService : IOrderService
{
    [Inject] private readonly IOrderRepository _repository;
    [Inject] private readonly IEmailService _emailService;
    [Inject] private readonly ILogger<OrderService> _logger;

    public async Task ProcessAsync(Order order)
    {
        _logger.LogInformation("Processing order");
        await _repository.SaveAsync(order);
        await _emailService.SendConfirmationAsync(order);
    }
}
```

No constructor.
No assignments.
Behind the scenes, the generator creates:
```csharp
public OrderService(
    IOrderRepository repository,
    IEmailService emailService,
    ILogger<OrderService> logger)
{
    _repository = repository;
    _emailService = emailService;
    _logger = logger;
}
```
The code is real and visible in generated files.
This keeps classes focused on behavior instead of wiring.

### Automatic Service Registration
Manual registration is another constant source of friction.

Instead of writing:
```csharp
services.AddScoped<IOrderService, OrderService>();
services.AddScoped<IOrderRepository, OrderRepository>();
services.AddScoped<IEmailService, EmailService>();
```
We can decorate services directly:
```csharp
[Register(typeof(IOrderService), ServiceLifetime.Scoped)]
public partial class OrderService : IOrderService
{
    [Inject] private readonly IOrderRepository _repository;
}
```
The generator creates an extension method:
`builder.Services.AddMyProject();`
One line.
Everything registered.
This keeps Program.cs clean and removes entire classes of runtime errors.

### Running Initialization Logic After Injection
Sometimes a service needs setup logic after dependencies are available.

Instead of placing logic awkwardly in constructors, a post-construction hook can be used:
```csharp
public partial class CacheService
{
    [Inject] private readonly IConfiguration _configuration;
    
    private TimeSpan _expiration;
    [PostConstruct]
    private void Initialize()
    {
        _expiration = TimeSpan.FromMinutes(
            _configuration.GetValue<int>("Cache:Expiration"));
    }
}
```
The generated constructor assigns dependencies and calls Initialize() automatically.
This keeps setup logic explicit and readable.

### Inheritance Without Constructor Headaches
One of the more painful parts of traditional DI is inheritance.

Base classes have dependencies.
Derived classes add more.
Constructors must forward everything manually.
With source generation, dependencies across inheritance chains are discovered and wired automatically.
The generator:
* Collects injected fields from base classes
* Generates the correct constructor
* Forwards parameters properly
This removes a lot of fragile plumbing.

### Multiple Implementations Become Natural
In many systems, we register several implementations of the same interface.

Notification handlers.
Validators.
Strategies.
With generated registration, this becomes straightforward.
Each class registers itself:
```csharp
[Register(typeof(INotificationHandler), ServiceLifetime.Scoped)]
public partial class EmailNotificationHandler : INotificationHandler { }

[Register(typeof(INotificationHandler), ServiceLifetime.Scoped)]
public partial class SmsNotificationHandler : INotificationHandler { }
```
Then inject them as:
`[Inject] private readonly IEnumerable<INotificationHandler> _handlers;`
The DI container handles everything.
The code remains clean.

### Configuration Without Injecting IConfiguration Everywhere
Another small but useful improvement is injecting configuration values directly:

```csharp
public partial class FeatureService
{
    [Config("App:MaxItems")] 
    private readonly int _maxItems;

    [ConnectionString("MainDb")] 
    private readonly string _connectionString;
}
```
This removes a lot of boilerplate while keeping configuration strongly typed.

### Why Source Generation Is Better Than Reflection Scanning
Some teams already use assembly scanning libraries.

They work, but come with tradeoffs:
* Startup overhead
* Trimming issues
* NativeAOT incompatibility
* Runtime failures
Source generation avoids all of that.
Everything is:
* Generated at compile time
* Type safe
* Visible in code
* Friendly to trimming and AOT
This aligns with the direction modern .NET is heading.

### What This Changes in Real Projects
After moving to the generated DI:

* Constructors shrink or disappear
* Program.cs becomes minimal
* Runtime registration errors vanish
* Startup improves slightly
* Code becomes easier to navigate
Most importantly, developers spend less time wiring and more time building.

### Final Thoughts
Traditional dependency injection in .NET works well.

But the surrounding boilerplate has always been a tax we accepted.
Source generation removes that tax.
By generating constructors and registrations at compile time, we get:
* Cleaner classes
* Fewer bugs
* Better performance
* Future-proof design
DI becomes invisible again, which is exactly how infrastructure should feel.
Once teams adopt this approach, going back to manual constructors feels unnecessary.
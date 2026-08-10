# Mastering .NET Architecture Through Real-World Codebases

**A Comprehensive Study Guide for Intermediate .NET Developers**

![Difficulty Level: Intermediate](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Estimated Reading Time: 20 minutes](https://img.shields.io/badge/Reading_Time-20_min-blue)
![Last Updated: July 23, 2026](https://img.shields.io/badge/Last_Updated-July_2026-green)

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [Why Production Code Matters](#why-production-code-matters)
5. [The 10 Essential .NET Codebases](#the-10-essential-net-codebases)
   - [1. fullstackhero .NET Starter Kit](#1-fullstackhero-net-starter-kit)
   - [2. fullstackhero Blazor WebAssembly Boilerplate](#2-fullstackhero-blazor-webassembly-boilerplate)
   - [3. fullstackhero Blazor Starter Kit](#3-fullstackhero-blazor-starter-kit)
   - [4. Dotnet Boxed Templates](#4-dotnet-boxed-templates)
   - [5. Jason Taylor's Clean Architecture](#5-jason-taylors-clean-architecture)
   - [6. Ardalis Clean Architecture](#6-ardalis-clean-architecture)
   - [7. CleanArchitecture.WebApi by Mukesh Murugan](#7-cleanarchitecturewebapi-by-mukesh-murugan)
   - [8. Onion Architecture by Mukesh Murugan](#8-onion-architecture-by-mukesh-murugan)
   - [9. Event Reminder by Milan Jovanović](#9-event-reminder-by-milan-jovanović)
   - [10. .NET eShop](#10-net-eshop)
6. [Architectural Patterns Comparison](#architectural-patterns-comparison)
7. [How to Study a Repository Properly](#how-to-study-a-repository-properly)
8. [Practical Implementation Guide](#practical-implementation-guide)
9. [Common Pitfalls and Anti-Patterns](#common-pitfalls-and-anti-patterns)
10. [Best Practices](#best-practices)
11. [Practice Exercises with Solutions](#practice-exercises-with-solutions)
12. [Test Your Understanding](#test-your-understanding)
13. [Common Interview Questions](#common-interview-questions)
14. [Question Bank](#question-bank)
15. [Troubleshooting Guide](#troubleshooting-guide)
16. [Performance Considerations](#performance-considerations)
17. [Security Considerations](#security-considerations)
18. [Summary and Key Takeaways](#summary-and-key-takeaways)
19. [Further Reading and Resources](#further-reading-and-resources)
20. [Self-Assessment Checklist](#self-assessment-checklist)

---

## Introduction

> **💡 Key Insight:** Tutorial projects teach syntax. Production-ready projects teach decisions.

If you've ever opened a tutorial project and thought "This looks nothing like the real applications I work on," you're not alone. There's a fundamental gap between the simplified examples we use to learn and the complex, production-ready systems that power businesses worldwide.

### The Problem with Tutorial Code

Most tutorials focus on:
- ✅ Teaching specific syntax
- ✅ Demonstrating isolated features
- ✅ Showing "happy path" scenarios
- ❌ Ignoring real-world complexity
- ❌ Skipping architectural decisions
- ❌ Missing production concerns

### What Production Code Teaches

Real-world codebases reveal:
- 🎯 How experienced developers organize code
- 🎯 How to manage dependencies and coupling
- 🎯 How to secure applications properly
- 🎯 How to handle failures gracefully
- 🎯 How to test important behavior
- 🎯 How to configure deployment and observability
- 🎯 How to prepare systems for growth

### Why This Guide Matters

This comprehensive study guide will transform how you approach .NET development. You'll learn to:

1. **Analyze** production-ready codebases effectively
2. **Understand** the architectural decisions behind them
3. **Apply** proven patterns to your own projects
4. **Avoid** common pitfalls and anti-patterns
5. **Make** informed decisions about architecture choices

---

## Prerequisites

Before diving into this guide, ensure you have:

### Technical Requirements
- ✅ **.NET 6.0+** or **.NET 8.0+** SDK installed
- ✅ **Visual Studio 2022** or **Visual Studio Code** with C# extensions
- ✅ **Git** for cloning repositories
- ✅ **Docker Desktop** (for containerized applications)
- ✅ Basic understanding of **C#** and **ASP.NET Core**
- ✅ Familiarity with **Entity Framework Core**
- ✅ Knowledge of **REST API** fundamentals

### Knowledge Requirements
- ✅ Understanding of **SOLID principles**
- ✅ Basic familiarity with **design patterns**
- ✅ Experience with **dependency injection**
- ✅ Understanding of **HTTP and web fundamentals**
- ✅ Basic knowledge of **databases and ORMs**

### Recommended Prior Knowledge
- 📚 Exposure to **Clean Architecture** concepts
- 📚 Understanding of **CQRS** (Command Query Responsibility Segregation)
- 📚 Basic knowledge of **Domain-Driven Design** (DDD)
- 📚 Familiarity with **testing frameworks** (xUnit, NUnit, MSTest)

> **⚠️ Note:** If you're new to .NET, I recommend completing a foundational course first. This guide assumes intermediate knowledge and focuses on architectural patterns and production concerns.

---

## Learning Objectives

By the end of this comprehensive guide, you will be able to:

### Knowledge Objectives
1. ✅ **Identify** key architectural patterns used in production .NET applications
2. ✅ **Compare** Clean Architecture, Onion Architecture, and Hexagonal Architecture
3. ✅ **Explain** the purpose and benefits of each architectural layer
4. ✅ **Understand** how cross-cutting concerns are implemented (authentication, logging, validation)
5. ✅ **Recognize** the difference between good abstractions and unnecessary complexity

### Practical Objectives
6. ✅ **Analyze** a real-world codebase systematically
7. ✅ **Trace** a request from endpoint to database and back
8. ✅ **Implement** proper dependency injection and inversion
9. ✅ **Apply** CQRS and MediatR patterns in projects
10. ✅ **Set up** multitenancy and modular organization

### Decision-Making Objectives
11. ✅ **Choose** the right architecture for your project size and complexity
12. ✅ **Avoid** over-engineering small applications
13. ✅ **Identify** when to introduce abstractions vs. keep things simple
14. ✅ **Plan** migration paths from simpler to more complex architectures
15. ✅ **Evaluate** starter kits and templates critically

---

## Why Production Code Matters

### The Tutorial Code Trap

Most learning resources follow this pattern:

```csharp
// Tutorial Example - Simplified
[HttpGet("{id}")]
public async Task<ActionResult<Product>> Get(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null)
        return NotFound();
    
    return Ok(product);
}
```

This code works, but it raises critical questions:
- ❓ Where's the validation?
- ❓ What if the database is down?
- ❓ How do we handle authorization?
- ❓ Where's the logging?
- ❓ How do we test this?
- ❓ What if we need to change the database?

### Production-Ready Code

Here's how production code addresses these concerns:

```csharp
// Production-Ready Example
[HttpGet("{id}")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
public async Task<ActionResult<ProductDto>> Get(int id, CancellationToken cancellationToken)
{
    _logger.LogInformation("Fetching product with ID: {ProductId}", id);
    
    // Validation
    if (id <= 0)
    {
        _logger.LogWarning("Invalid product ID: {ProductId}", id);
        return BadRequest(new { Error = "Invalid product ID" });
    }
    
    try
    {
        // Authorization check
        if (!await _authorizationService.AuthorizeAsync(User, "Products.Read"))
        {
            _logger.LogWarning("Unauthorized access attempt for product: {ProductId}", id);
            return Forbid();
        }
        
        // Business logic in application layer
        var query = new GetProductByIdQuery(id);
        var result = await _mediator.Send(query, cancellationToken);
        
        if (result == null)
        {
            _logger.LogInformation("Product not found: {ProductId}", id);
            return NotFound();
        }
        
        _logger.LogInformation("Successfully retrieved product: {ProductId}", id);
        return Ok(result);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error fetching product: {ProductId}", id);
        return StatusCode(500, new { Error = "An error occurred while processing your request" });
    }
}
```

### What Changed?

| Aspect | Tutorial Code | Production Code |
|--------|---------------|-----------------|
| **Logging** | ❌ None | ✅ Structured logging with context |
| **Validation** | ❌ None | ✅ Input validation |
| **Authorization** | ❌ None | ✅ Permission checks |
| **Error Handling** | ❌ Basic | ✅ Comprehensive try-catch |
| **Documentation** | ❌ None | ✅ XML comments and metadata |
| **Testability** | ❌ Hard to test | ✅ Uses MediatR for easy mocking |
| **Separation of Concerns** | ❌ All in controller | ✅ Business logic in handlers |
| **Observability** | ❌ None | ✅ Logs, metrics, traces |

### The Learning Gap

```
┌─────────────────────────────────────────────────────────────┐
│                    THE LEARNING GAP                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Tutorial Code          Production Code                     │
│  ─────────────          ──────────────                      │
│                                                             │
│  ✓ Syntax              ✓ Architecture decisions            │
│  ✓ Basic features      ✓ Cross-cutting concerns            │
│  ✓ Happy path          ✓ Error handling                    │
│  ✓ Single file         ✓ Multi-layer organization          │
│  ✓ No dependencies     ✓ Dependency management             │
│  ✓ Simple examples     ✓ Real business rules               │
│                                                             │
│              ╔═══════════════════════╗                      │
│              ║   THE GAP IN BETWEEN  ║                      │
│              ╚═══════════════════════╝                      │
│                                                             │
│  This is what you'll learn from studying                    │
│  production-ready codebases!                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The 10 Essential .NET Codebases

Let's explore each codebase in detail, understanding what makes it valuable and what you should focus on when studying it.

---

### 1. fullstackhero .NET Starter Kit

**Repository:** [fullstackhero/dotnet-starter-kit](https://github.com/fullstackhero/dotnet-starter-kit)

#### Overview

The fullstackhero .NET Starter Kit is a **production-oriented ASP.NET Core foundation** designed for building modular business applications. It's not just a template—it's a comprehensive foundation that addresses real-world concerns from day one.

#### Key Features

```
┌──────────────────────────────────────────────────────────────┐
│              fullstackhero Starter Kit Features              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🔐 Authentication & Authorization                           │
│     • JWT Bearer Authentication                              │
│     • Role-based Authorization                               │
│     • Policy-based Access Control                            │
│                                                              │
│  🏢 Multitenancy                                             │
│     • Tenant Resolution Strategies                           │
│     • Per-Tenant Database Isolation                          │
│     • Tenant-Specific Configuration                          │
│                                                              │
│  🗄️  Persistence                                             │
│     • Entity Framework Core                                  │
│     • Repository Pattern                                     │
│     • Unit of Work                                           │
│                                                              │
│  📦 Modular Organization                                     │
│     • Feature-based Folder Structure                         │
│     • Dependency Isolation                                   │
│     • Cross-cutting Concern Handling                         │
│                                                              │
│  ⚡ CQRS Implementation                                      │
│     • MediatR Integration                                    │
│     • Command Handlers                                       │
│     • Query Handlers                                         │
│     • Pipeline Behaviors                                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 1.1 Module Separation

The starter kit demonstrates how to organize code by feature rather than by layer:

```
Traditional Layer-Based Structure:
├── Controllers/
├── Services/
├── Repositories/
└── Models/

Feature-Based Structure (Better!):
├── Features/
│   ├── Products/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   ├── Product.cs
│   │   └── ProductProfile.cs
│   ├── Orders/
│   └── Customers/
```

**Why This Matters:**
- ✅ Related code stays together
- ✅ Easier to understand feature boundaries
- ✅ Simpler to extract features into microservices later
- ✅ Reduced merge conflicts in team environments

##### 1.2 Cross-Cutting Concerns Registration

Study how the kit registers services without tight coupling:

```csharp
// Program.cs - Clean dependency registration
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices();
builder.Services.AddWebServices();

// Extension method pattern
public static IServiceCollection AddApplicationServices(this IServiceCollection services)
{
    // MediatR - CQRS
    services.AddMediatR(cfg => 
        cfg.RegisterServicesFromAssembly(typeof(ApplicationAssemblyMarker).Assembly));
    
    // Validators - FluentValidation
    services.AddValidatorsFromAssembly(typeof(ApplicationAssemblyMarker).Assembly);
    
    // Behaviors - Pipeline
    services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
    services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
    
    return services;
}
```

**Key Takeaway:** Extension methods keep `Program.cs` clean and organize registration by concern.

##### 1.3 CQRS Organization

Commands and queries are organized by feature:

```csharp
// Features/Products/Commands/CreateProduct/
public class CreateProductCommand : IRequest<Guid>
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
}

public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Guid>
{
    private readonly IApplicationDbContext _context;
    
    public async Task<Guid> Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        var product = new Product(request.Name, request.Price, request.Stock);
        
        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);
        
        return product.Id;
    }
}
```

**Benefits:**
- ✅ Each operation is isolated
- ✅ Easy to test individually
- ✅ Clear separation of read and write operations
- ✅ Natural fit for complex business logic

##### 1.4 Multitenancy Implementation

The kit shows how multitenancy affects data access:

```csharp
// Tenant resolution
public interface ITenantProvider
{
    Guid? GetTenantId();
    string GetTenantIdentifier();
}

// Database context with tenant filtering
public class ApplicationDbContext : DbContext
{
    private readonly ITenantProvider _tenantProvider;
    
    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // Automatically set TenantId on new entities
        foreach (var entry in ChangeTracker.Entries<ITenantEntity>())
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.TenantId = _tenantProvider.GetTenantId() ?? Guid.Empty;
            }
        }
        
        return base.SaveChangesAsync(cancellationToken);
    }
}

// Global query filter for tenant isolation
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>()
        .HasQueryFilter(p => p.TenantId == _tenantProvider.GetTenantId());
}
```

**Critical Security Lesson:** Never rely solely on UI hiding for multitenancy. Always enforce tenant isolation at the database level.

#### Best For

- ✅ Intermediate developers building **SaaS platforms**
- ✅ Multi-tenant systems requiring data isolation
- ✅ Modular business applications
- ✅ Teams wanting production-ready foundations

#### Important Lesson

> **⚠️ Warning:** Do not copy the entire solution before understanding its conventions.

Starter kits save time only when the team understands what was started. Otherwise, they simply deliver someone else's complexity faster.

#### Practical Exercise

**Exercise:** Clone the repository and:
1. Identify how authentication is configured
2. Trace a request from controller to database
3. Add a new feature following the existing patterns
4. Document the module structure

---

### 2. fullstackhero Blazor WebAssembly Boilerplate

**Repository:** [fullstackhero/blazor-wasm-boilerplate](https://github.com/fullstackhero/blazor-wasm-boilerplate)

#### Overview

This project demonstrates how a **Blazor WebAssembly frontend** can communicate with a structured .NET backend. It's valuable because it shows more than isolated Razor components—it demonstrates how the client application fits into a larger authenticated system.

#### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    System Architecture                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐      HTTPS       ┌──────────────┐       │
│  │   Browser    │ ◄──────────────► │  ASP.NET     │       │
│  │              │                  │  Core API    │       │
│  │  Blazor      │                  │              │       │
│  │  WebAssembly │                  │  Backend     │       │
│  └──────────────┘                  └──────────────┘       │
│        │                                  │                 │
│        │                                  │                 │
│        │                          ┌───────▼────────┐        │
│        │                          │   Database     │        │
│        │                          │  (PostgreSQL)  │        │
│        │                          └────────────────┘        │
│        │                                                  │
│  Client-Side:                    Server-Side:              │
│  • UI Components                 • API Controllers         │
│  • State Management              • Business Logic          │
│  • Authentication Token          • Data Access             │
│  • API Communication             • Authorization            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 2.1 Client-Side Authentication

```csharp
// Program.cs - Authentication setup
builder.Services.AddApiAuthorization(options =>
{
    options.UserOptions.RoleClaim = "role";
});

// Authentication state provider
public class CustomAuthenticationStateProvider : AuthenticationStateProvider
{
    private readonly IAccessTokenProvider _tokenProvider;
    
    public async override Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        var result = await _tokenProvider.RequestAccessToken();
        
        if (result.TryGetToken(out var token))
        {
            // Parse and validate token
            var identity = new ClaimsIdentity(ParseClaimsFromJwt(token.Value), "Bearer");
            var user = new ClaimsPrincipal(identity);
            
            return new AuthenticationState(user);
        }
        
        return new AuthenticationState(new ClaimsPrincipal(new ClaimsIdentity()));
    }
}
```

**Key Points:**
- ✅ Tokens stored securely (not in localStorage by default)
- ✅ Automatic token refresh
- ✅ Integration with ASP.NET Core Identity

##### 2.2 API Communication

```csharp
// ApiService.cs - Centralized API communication
public class ApiService
{
    private readonly HttpClient _httpClient;
    private readonly IAccessTokenProvider _tokenProvider;
    
    public async Task<Result<T>> GetAsync<T>(string url)
    {
        try
        {
            var tokenResult = await _tokenProvider.RequestAccessToken();
            
            if (tokenResult.TryGetToken(out var token))
            {
                _httpClient.DefaultRequestHeaders.Authorization = 
                    new AuthenticationHeaderValue("Bearer", token.Value);
            }
            
            var response = await _httpClient.GetAsync(url);
            
            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadFromJsonAsync<T>();
                return Result<T>.Success(content);
            }
            
            return Result<T>.Failure($"Error: {response.StatusCode}");
        }
        catch (Exception ex)
        {
            return Result<T>.Failure(ex.Message);
        }
    }
}
```

**Pattern Benefits:**
- ✅ Centralized error handling
- ✅ Consistent response format
- ✅ Automatic token injection
- ✅ Type-safe responses

##### 2.3 State Management

```csharp
// ApplicationState.cs - Global state management
public class ApplicationState
{
    private readonly ApiService _apiService;
    
    // Event for state changes
    public event Action OnChange;
    
    // State properties
    private List<Product> _products = new();
    public List<Product> Products
    {
        get => _products;
        set
        {
            _products = value;
            NotifyStateChanged();
        }
    }
    
    private bool _isLoading;
    public bool IsLoading
    {
        get => _isLoading;
        set
        {
            _isLoading = value;
            NotifyStateChanged();
        }
    }
    
    // Methods to modify state
    public async Task LoadProducts()
    {
        IsLoading = true;
        var result = await _apiService.GetAsync<List<Product>>("api/products");
        
        if (result.IsSuccess)
        {
            Products = result.Data;
        }
        
        IsLoading = false;
    }
    
    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

**Usage in Components:**

```razor
@inject ApplicationState AppState

@if (AppState.IsLoading)
{
    <p>Loading...</p>
}
else
{
    @foreach (var product in AppState.Products)
    {
        <ProductCard Product="product" />
    }
}

@code {
    protected override async Task OnInitializedAsync()
    {
        await AppState.LoadProducts();
    }
}
```

#### Important Lesson

> **⚠️ Critical Security Warning:** Hiding a button in the frontend is NOT authorization.

The client may improve user experience, but the backend must remain responsible for enforcing security rules.

**Example of Insecure Frontend-Only "Authorization":**

```razor
<!-- ❌ WRONG - Security through obscurity -->
@if (User.IsInRole("Admin"))
{
    <button @onclick="DeleteProduct">Delete Product</button>
}

<!-- ✅ CORRECT - Backend still validates -->
@code {
    private async Task DeleteProduct()
    {
        // Backend will still check permissions!
        await ApiService.DeleteAsync($"api/products/{productId}");
    }
}
```

**Backend Must Always Validate:**

```csharp
[HttpDelete("{id}")]
[Authorize(Policy = "AdminOnly")]
public async Task<IActionResult> Delete(Guid id)
{
    // Even if frontend hides the button,
    // this endpoint still checks authorization
    await _productService.DeleteAsync(id);
    return NoContent();
}
```

#### Best For

- ✅ Developers evaluating **Blazor WebAssembly** for dashboards
- ✅ Internal tools and administration panels
- ✅ Business portals requiring rich UI
- ✅ Teams familiar with .NET backend exploring frontend options

---

### 3. fullstackhero Blazor Starter Kit

**Repository:** [fullstackhero/blazor-starter-kit](https://github.com/fullstackhero/blazor-starter-kit)

#### Overview

The fullstackhero Blazor Starter Kit focuses on creating a **practical Blazor Server application** with common functionality already available. This makes it useful for studying how a component-based .NET UI can be organized beyond small demonstration projects.

#### Key Features

```
┌──────────────────────────────────────────────────────────────┐
│              Blazor Starter Kit Features                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🎨 UI Components                                            │
│     • Reusable Component Library                             │
│     • MudBlazor Integration                                  │
│     • Responsive Layouts                                     │
│                                                              │
│  🔐 Authentication Flow                                      │
│     • Login/Logout                                           │
│     • Registration                                           │
│     • Password Reset                                         │
│     • External Authentication                                │
│                                                              │
│  📱 Navigation                                               │
│     • Protected Routes                                       │
│     • Role-Based Navigation                                  │
│     • Breadcrumbs                                            │
│                                                              │
│  ✅ Form Validation                                          │
│     • FluentValidation Integration                           │
│     • Client + Server Validation                             │
│     • Custom Validation Rules                                │
│                                                              │
│  🏗️  Application Layout                                      │
│     • Main Layout                                            │
│     • Login Layout                                           │
│     • Error Layout                                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 3.1 Component Organization

```csharp
// Shared/Components/BaseComponent.cs
public class BaseComponent : ComponentBase
{
    [Inject] protected NavigationManager Navigation { get; set; }
    [Inject] protected ILogger Logger { get; set; }
    
    protected override void OnInitialized()
    {
        Logger.LogInformation("{Component} initialized", GetType().Name);
    }
    
    protected void NavigateTo(string url)
    {
        Navigation.NavigateTo(url);
    }
}

// Usage in components
public class ProductListComponent : BaseComponent
{
    [Inject] protected IProductService ProductService { get; set; }
    
    private List<Product> products;
    
    protected override async Task OnInitializedAsync()
    {
        await LoadProducts();
    }
    
    private async Task LoadProducts()
    {
        products = await ProductService.GetAllAsync();
    }
}
```

**Benefits:**
- ✅ Consistent base functionality across components
- ✅ Centralized logging
- ✅ Easy navigation helpers
- ✅ Reduced code duplication

##### 3.2 Authentication Flow

```csharp
// Authentication.razor
@inject IAuthenticationService AuthService
@inject NavigationManager Navigation

<EditForm Model="Model" OnValidSubmit="HandleLogin">
    <DataAnnotationsValidator />
    <ValidationSummary />
    
    <MudTextField @bind-Value="Model.Email"
                  Label="Email"
                  Required="true"
                  RequiredError="Email is required!" />
    
    <MudTextField @bind-Value="Model.Password"
                  Label="Password"
                  InputType="InputType.Password"
                  Required="true"
                  RequiredError="Password is required!" />
    
    <MudButton ButtonType="ButtonType.Submit"
               Variant="Variant.Filled"
               Color="Color.Primary">
        Login
    </MudButton>
</EditForm>

@code {
    public LoginRequest Model { get; set; } = new();
    
    private async Task HandleLogin()
    {
        var result = await AuthService.Login(Model);
        
        if (result.Success)
        {
            Navigation.NavigateTo("/");
        }
        else
        {
            // Show error message
        }
    }
}
```

##### 3.3 Reusable UI Components

```csharp
// Shared/Components/ConfirmDialog.razor
<MudDialog>
    <DialogContent>
        <MudText>@Message</MudText>
    </DialogContent>
    <DialogActions>
        <MudButton OnClick="Cancel">Cancel</MudButton>
        <MudButton Color="Color.Error" OnClick="Confirm">Confirm</MudButton>
    </DialogActions>
</MudDialog>

@code {
    [CascadingParameter] MudDialogInstance MudDialog { get; set; }
    [Parameter] public string Message { get; set; }
    
    private void Confirm() => MudDialog.Close(DialogResult.Ok(true));
    private void Cancel() => MudDialog.Close(DialogResult.Cancel());
}

// Usage
private async Task DeleteProduct(int id)
{
    var confirmed = await DialogService.ShowConfirm("Are you sure?");
    
    if (confirmed)
    {
        await ProductService.Delete(id);
    }
}
```

#### Important Lesson

> **⚠️ Critical Principle:** Do not place business logic directly inside UI components.

**❌ Wrong Approach:**

```csharp
// ProductPage.razor - BAD!
private async Task DeleteProduct(int id)
{
    // Business logic in UI component!
    var product = await DbContext.Products.FindAsync(id);
    
    if (product.Orders.Any())
    {
        // More business logic!
        Snackbar.Add("Cannot delete product with orders", Severity.Error);
        return;
    }
    
    product.IsDeleted = true;
    await DbContext.SaveChangesAsync();
}
```

**✅ Correct Approach:**

```csharp
// ProductPage.razor - GOOD!
private async Task DeleteProduct(int id)
{
    // UI coordination only
    var command = new DeleteProductCommand(id);
    var result = await Mediator.Send(command);
    
    if (result.IsSuccess)
    {
        Snackbar.Add("Product deleted successfully", Severity.Success);
        await LoadProducts();
    }
    else
    {
        Snackbar.Add(result.Error, Severity.Error);
    }
}

// DeleteProductCommandHandler.cs - Business logic here
public class DeleteProductCommandHandler : IRequestHandler<DeleteProductCommand, Result>
{
    private readonly IProductRepository _repository;
    
    public async Task<Result> Handle(DeleteProductCommand request, CancellationToken cancellationToken)
    {
        var product = await _repository.GetByIdAsync(request.Id);
        
        if (product.Orders.Any())
        {
            return Result.Failure("Cannot delete product with existing orders");
        }
        
        product.SoftDelete();
        await _repository.SaveChangesAsync(cancellationToken);
        
        return Result.Success();
    }
}
```

**Why This Matters:**
- ✅ Business logic can be tested independently
- ✅ UI can change without affecting business rules
- ✅ Multiple UI entry points can use same logic
- ✅ Business rules are centralized and consistent

#### Best For

- ✅ ASP.NET Core developers moving into **Blazor**
- ✅ Teams with strong backend skills building internal tools
- ✅ Developers preferring C# over JavaScript
- ✅ Enterprise applications requiring rich UI

---

### 4. Dotnet Boxed Templates

**Repository:** [Dotnet-Boxed/Templates](https://github.com/Dotnet-Boxed/Templates)

#### Overview

Dotnet Boxed provides **opinionated `dotnet new` templates** for APIs and other .NET application types. The repository demonstrates how good defaults can reduce the amount of production work developers accidentally forget.

#### Template Types

```
┌──────────────────────────────────────────────────────────────┐
│              Dotnet Boxed Template Categories                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🌐 Web APIs                                                 │
│     • dotnet new boxed-api                                   │
│     • Minimal APIs                                           │
│     • Controller-based APIs                                  │
│                                                              │
│  🖥️  Blazor Applications                                     │
│     • Server-side Blazor                                     │
│     • WebAssembly Blazor                                     │
│     • Hosted Blazor                                          │
│                                                              │
│  ⚡ Console Applications                                     │
│     • Worker Service                                         │
│     • Console with DI                                        │
│                                                              │
│  🧪 Testing Projects                                         │
│     • xUnit Test Project                                     │
│     • Integration Test Project                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 4.1 Secure Defaults

```csharp
// Program.cs - Security headers
builder.Services.AddHsts(options =>
{
    options.Preload = true;
    options.IncludeSubDomains = true;
    options.MaxAge = TimeSpan.FromDays(365);
});

// Security headers middleware
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
    
    await next();
});
```

**Why This Matters:**
- ✅ Prevents clickjacking attacks
- ✅ Blocks MIME type sniffing
- ✅ Enables XSS protection
- ✅ Security headers applied consistently

##### 4.2 Health Checks

```csharp
// Program.cs - Comprehensive health checks
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy())
    .AddDbContextCheck<ApplicationDbContext>("database")
    .AddRedis(redisConnectionString, "redis")
    .AddUrlGroup(new Uri("https://api.example.com/health"), "external-api");

// Health checks endpoint
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description
            })
        });
        
        await context.Response.WriteAsync(result);
    }
});
```

**Health Check Benefits:**
- ✅ Monitor application health
- ✅ Detect database connectivity issues
- ✅ Verify external dependencies
- ✅ Kubernetes readiness/liveness probes

##### 4.3 Docker Support

```dockerfile
# Multi-stage Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy csproj and restore
COPY ["Api/Api.csproj", "Api/"]
RUN dotnet restore "Api/Api.csproj"

# Copy everything else and build
COPY . .
WORKDIR "/src/Api"
RUN dotnet build "Api.csproj" -c Release -o /app/build

# Publish
FROM build AS publish
RUN dotnet publish "Api.csproj" -c Release -o /app/publish

# Final stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Non-root user for security
RUN adduser --disabled-password --gecos '' appuser && \
    chown -R appuser /app
USER appuser

COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Api.dll"]
```

**Docker Best Practices:**
- ✅ Multi-stage builds reduce image size
- ✅ Non-root user for security
- ✅ Separate build and runtime images
- ✅ Proper layer caching

##### 4.4 OpenAPI Configuration

```csharp
// Program.cs - OpenAPI/Swagger setup
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "A comprehensive API example",
        Contact = new OpenApiContact
        {
            Name = "Support Team",
            Email = "support@example.com"
        }
    });
    
    // Include XML comments
    var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFilename));
    
    // Add JWT authentication
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using the Bearer scheme",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey
    });
    
    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});

// Enable Swagger UI
app.UseSwagger();
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "API V1");
    options.RoutePrefix = "api-docs";
});
```

#### Important Lesson

> **💡 Key Insight:** The main lesson is not one specific architecture. It is that good defaults reduce forgotten work.

Security headers, health checks, validation, API documentation, and deployment configuration should not be rediscovered in every project.

#### Production Checklist from Templates

Study how the templates include:

- ✅ Security headers (HSTS, CSP, X-Frame-Options)
- ✅ Health checks (database, external services)
- ✅ Request validation and sanitization
- ✅ API documentation (OpenAPI/Swagger)
- ✅ Docker configuration
- ✅ CI/CD pipeline examples
- ✅ Logging configuration (Serilog)
- ✅ Exception handling middleware
- ✅ CORS configuration
- ✅ Rate limiting

#### Best For

- ✅ Developers wanting to understand what a **polished application template** should include
- ✅ Teams starting new projects
- ✅ Learning production-ready defaults
- ✅ Understanding project structure best practices

---

### 5. Jason Taylor's Clean Architecture

**Repository:** [jasontaylordev/CleanArchitecture](https://github.com/jasontaylordev/CleanArchitecture)

#### Overview

Jason Taylor's Clean Architecture template is one of the **best-known architectural templates** in the .NET community. It demonstrates how dependencies can point toward application and domain concerns while infrastructure remains replaceable.

#### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Clean Architecture Layers                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────┐         │
│  │  Presentation Layer                           │         │
│  │  ┌─────────────────────────────────────────┐  │         │
│  │  │  • API Controllers                      │  │         │
│  │  │  • Minimal API Endpoints                 │  │         │
│  │  │  • SignalR Hubs                          │  │         │
│  │  │  • gRPC Services                         │  │         │
│  │  └─────────────────────────────────────────┘  │         │
│  └───────────────────────────────────────────────┘         │
│                      ▲                                      │
│                      │ Depends on                           │
│  ┌───────────────────────────────────────────────┐         │
│  │  Application Layer                           │         │
│  │  ┌─────────────────────────────────────────┐  │         │
│  │  │  • Use Cases (Commands/Queries)         │  │         │
│  │  │  • DTOs                                   │  │         │
│  │  │  • Interfaces (Repositories, Services)   │  │         │
│  │  │  • Validation (FluentValidation)         │  │         │
│  │  │  • Behaviors (MediatR Pipeline)          │  │         │
│  │  └─────────────────────────────────────────┘  │         │
│  └───────────────────────────────────────────────┘         │
│                      ▲                                      │
│                      │ Depends on                           │
│  ┌───────────────────────────────────────────────┐         │
│  │  Domain Layer (CORE)                         │         │
│  │  ┌─────────────────────────────────────────┐  │         │
│  │  │  • Entities                              │  │         │
│  │  │  • Value Objects                          │  │         │
│  │  │  • Domain Events                          │  │         │
│  │  │  • Aggregates                             │  │         │
│  │  │  • Domain Services                        │  │         │
│  │  │  • Repository Interfaces                  │  │         │
│  │  │  • Domain Exceptions                      │  │         │
│  │  └─────────────────────────────────────────┘  │         │
│  └───────────────────────────────────────────────┘         │
│                      ▲                                      │
│                      │ Implements                           │
│  ┌───────────────────────────────────────────────┐         │
│  │  Infrastructure Layer                        │         │
│  │  ┌─────────────────────────────────────────┐  │         │
│  │  │  • EF Core DbContext                     │  │         │
│  │  │  • Repository Implementations            │  │         │
│  │  │  • Identity Implementation               │  │         │
│  │  │  • Email Service                          │  │         │
│  │  │  • File Storage                           │  │         │
│  │  │  • Third-party Integrations               │  │         │
│  │  └─────────────────────────────────────────┘  │         │
│  └───────────────────────────────────────────────┘         │
│                                                             │
│  Dependency Rule: Dependencies point INWARD                 │
│  Domain has NO dependencies on outer layers                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 5.1 Dependency Direction

The most critical aspect of Clean Architecture:

```
┌────────────────────────────────────────────────────────────┐
│              Dependency Direction Rule                      │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ❌ WRONG:                                                 │
│  Domain → Application → Infrastructure → Presentation      │
│                                                            │
│  ✅ CORRECT:                                               │
│  Presentation → Application → Domain                       │
│       ↓              ↓                                    │
│  Infrastructure ───┘                                       │
│                                                            │
│  Rule: Source code dependencies point INWARD               │
│  Inner circles know nothing about outer circles            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Code Example:**

```csharp
// Domain Layer - NO dependencies on outer layers!
namespace CleanArchitecture.Domain.Entities;

public class Product : BaseEntity
{
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    public int Stock { get; private set; }
    
    // Domain logic - pure C#, no infrastructure
    public void UpdatePrice(decimal newPrice)
    {
        if (newPrice <= 0)
            throw new DomainException("Price must be greater than zero");
        
        Price = newPrice;
        
        // Domain event
        AddDomainEvent(new ProductPriceUpdatedEvent(this.Id, newPrice));
    }
    
    public void ReduceStock(int quantity)
    {
        if (quantity > Stock)
            throw new DomainException("Insufficient stock");
        
        Stock -= quantity;
    }
}

// Application Layer - Depends on Domain, defines interfaces
namespace CleanArchitecture.Application.Interfaces;

public interface IProductRepository
{
    Task<Product> GetByIdAsync(Guid id, CancellationToken cancellationToken);
    Task<IReadOnlyList<Product>> GetAllAsync(CancellationToken cancellationToken);
    Task AddAsync(Product product, CancellationToken cancellationToken);
    Task UpdateAsync(Product product, CancellationToken cancellationToken);
    Task DeleteAsync(Product product, CancellationToken cancellationToken);
}

// Infrastructure Layer - Implements interfaces
namespace CleanArchitecture.Infrastructure.Persistence.Repositories;

public class ProductRepository : IProductRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<Product> GetByIdAsync(Guid id, CancellationToken cancellationToken)
    {
        return await _context.Products
            .FirstOrDefaultAsync(p => p.Id == id, cancellationToken);
    }
    
    // Other implementations...
}
```

##### 5.2 Application Use Cases

```csharp
// Application Layer - Use cases (Commands/Queries)

// Command - Changes state
public record CreateProductCommand(
    string Name,
    string Description,
    decimal Price,
    int Stock
) : IRequest<Result<Guid>>;

public class CreateProductCommandHandler 
    : IRequestHandler<CreateProductCommand, Result<Guid>>
{
    private readonly IProductRepository _repository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task<Result<Guid>> Handle(
        CreateProductCommand request, 
        CancellationToken cancellationToken)
    {
        // Validation
        var validation = await ValidateAsync(request);
        if (!validation.IsValid)
            return Result<Guid>.Failure(validation.Errors);
        
        // Create entity
        var product = new Product(
            request.Name,
            request.Description,
            request.Price,
            request.Stock);
        
        // Persist
        await _repository.AddAsync(product, cancellationToken);
        await _unitOfWork.SaveChangesAsync(cancellationToken);
        
        // Return result
        return Result<Guid>.Success(product.Id);
    }
}

// Query - Returns data
public record GetProductByIdQuery(Guid Id) 
    : IRequest<Result<ProductDto>>;

public class GetProductByIdQueryHandler 
    : IRequestHandler<GetProductByIdQuery, Result<ProductDto>>
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;
    
    public async Task<Result<ProductDto>> Handle(
        GetProductByIdQuery request, 
        CancellationToken cancellationToken)
    {
        var product = await _repository.GetByIdAsync(
            request.Id, 
            cancellationToken);
        
        if (product == null)
            return Result<ProductDto>.Failure("Product not found");
        
        var dto = _mapper.Map<ProductDto>(product);
        return Result<ProductDto>.Success(dto);
    }
}
```

##### 5.3 Validation

```csharp
// FluentValidation validators
public class CreateProductCommandValidator 
    : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .MaximumLength(100)
            .WithMessage("Product name is required and must be under 100 characters");
        
        RuleFor(x => x.Price)
            .GreaterThan(0)
            .WithMessage("Price must be greater than zero");
        
        RuleFor(x => x.Stock)
            .GreaterThanOrEqualTo(0)
            .WithMessage("Stock cannot be negative");
    }
}

// Validation behavior (pipeline)
public class ValidationBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;
    
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        var context = new ValidationContext<TRequest>(request);
        
        var validationResults = await _validators
            .Select(v => v.ValidateAsync(context, cancellationToken))
            .WhenAll();
        
        var failures = validationResults
            .SelectMany(r => r.Errors)
            .Where(f => f != null)
            .ToList();
        
        if (failures.Any())
        {
            throw new ValidationException(failures);
        }
        
        return await next();
    }
}
```

##### 5.4 Testing Boundaries

```csharp
// Domain Tests - No infrastructure dependencies
[Test]
public void UpdatePrice_WithNegativeValue_ThrowsException()
{
    // Arrange
    var product = new Product("Test", "Description", 100, 10);
    
    // Act & Assert
    Assert.Throws<DomainException>(() => product.UpdatePrice(-10));
}

// Application Tests - Mock repositories
[Test]
public async Task CreateProductCommand_WithValidData_ReturnsSuccess()
{
    // Arrange
    var mockRepo = new Mock<IProductRepository>();
    var mockUow = new Mock<IUnitOfWork>();
    var handler = new CreateProductCommandHandler(mockRepo.Object, mockUow.Object);
    
    var command = new CreateProductCommand(
        "Test Product",
        "Description",
        100,
        10);
    
    // Act
    var result = await handler.Handle(command, CancellationToken.None);
    
    // Assert
    Assert.IsTrue(result.IsSuccess);
    Assert.IsTrue(result.Data != Guid.Empty);
}

// Integration Tests - Real database
[Test]
public async Task CreateProduct_WithValidData_PersistsToDatabase()
{
    // Arrange
    await using var context = await DbContextHelper.CreateDbContextAsync();
    var handler = new CreateProductCommandHandler(context, context);
    
    // Act
    var result = await handler.Handle(command, CancellationToken.None);
    
    // Assert
    var product = await context.Products.FindAsync(result.Data);
    Assert.IsNotNull(product);
    Assert.AreEqual("Test Product", product.Name);
}
```

#### Important Lesson

> **⚠️ Important:** Do not assume every application needs the same number of projects, layers, abstractions, and handlers.

A large application may benefit from these boundaries. A small CRUD application may only gain additional ceremony.

**Use the principles, not the folder count.**

#### When to Use Clean Architecture

| Scenario | Use Clean Architecture? | Reason |
|----------|------------------------|--------|
| Large enterprise application (50+ entities) | ✅ Yes | Clear boundaries help manage complexity |
| Medium business application (10-50 entities) | ⚠️ Consider | Benefits may not outweigh ceremony |
| Small CRUD application (<10 entities) | ❌ Probably not | Over-engineering |
| Long-lived application (5+ years) | ✅ Yes | Maintainability crucial |
| Short-term project (<1 year) | ❌ Probably not | Speed matters more |
| Multiple teams working | ✅ Yes | Clear boundaries reduce conflicts |
| Solo developer/small team | ⚠️ Consider | May be overkill |

#### Best For

- ✅ Intermediate developers learning **layered architecture**
- ✅ Understanding **dependency inversion**
- ✅ Learning **separation of concerns**
- ✅ Building maintainable business applications

---

### 6. Ardalis Clean Architecture

**Repository:** [ardalis/CleanArchitecture](https://github.com/ardalis/CleanArchitecture)

#### Overview

Steve Smith's (Ardalis) template presents another respected interpretation of Clean Architecture. It places **strong emphasis on domain logic**, maintainability, use cases, and keeping infrastructure concerns outside the application core.

#### Comparison with Jason Taylor's Template

```
┌──────────────────────────────────────────────────────────────┐
│         Jason Taylor vs Ardalis Clean Architecture           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Aspect              │ Jason Taylor    │ Ardalis             │
│  ────────────────────┼────────────────┼───────────────────── │
│  Folder Structure    │ Feature-based   │ Layer-based          │
│  ────────────────────┼────────────────┼───────────────────── │
│  CQRS Implementation │ MediatR heavy   │ Lighter MediatR     │
│  ────────────────────┼────────────────┼───────────────────── │
│  Validation          │ FluentValidation│ FluentValidation     │
│  ────────────────────┼────────────────┼───────────────────── │
│  Testing Approach    │ Unit + Integr. │ Strong unit focus    │
│  ────────────────────┼────────────────┼───────────────────── │
│  Domain Modeling     │ DDD-oriented    │ DDD-oriented         │
│  ────────────────────┼────────────────┼───────────────────── │
│  Documentation       │ Extensive       │ Concise              │
│  ────────────────────┼────────────────┼───────────────────── │
│  Community Adoption  │ Very High       │ High                 │
│  ────────────────────┼────────────────┼───────────────────── │
│  Learning Curve      │ Moderate        │ Moderate             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 6.1 Domain Entities and Aggregates

```csharp
// Domain/Entities/Order.cs
public class Order : BaseEntity, IAggregateRoot
{
    private readonly List<OrderItem> _orderItems = new();
    
    public Guid CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    public DateTime OrderDate { get; private set; }
    public decimal TotalAmount { get; private set; }
    
    // Read-only collection
    public IReadOnlyCollection<OrderItem> OrderItems => _orderItems.AsReadOnly();
    
    // Private constructor - enforce creation through factory method
    private Order()
    {
        // Required for EF Core
    }
    
    // Factory method
    public static Order Create(Guid customerId, List<OrderItem> items)
    {
        var order = new Order
        {
            Id = Guid.NewGuid(),
            CustomerId = customerId,
            Status = OrderStatus.Pending,
            OrderDate = DateTime.UtcNow
        };
        
        // Business rule: Order must have at least one item
        if (items == null || !items.Any())
            throw new DomainException("Order must contain at least one item");
        
        foreach (var item in items)
        {
            order.AddOrderItem(item);
        }
        
        order.CalculateTotal();
        
        return order;
    }
    
    // Business logic encapsulated in entity
    public void AddOrderItem(OrderItem item)
    {
        // Business rule: Cannot add items to completed orders
        if (Status == OrderStatus.Completed)
            throw new DomainException("Cannot modify completed order");
        
        _orderItems.Add(item);
        CalculateTotal();
    }
    
    public void RemoveOrderItem(Guid itemId)
    {
        if (Status == OrderStatus.Completed)
            throw new DomainException("Cannot modify completed order");
        
        var item = _orderItems.FirstOrDefault(i => i.Id == itemId);
        if (item != null)
        {
            _orderItems.Remove(item);
            CalculateTotal();
        }
    }
    
    public void Submit()
    {
        // Business rule: Cannot submit empty order
        if (!_orderItems.Any())
            throw new DomainException("Cannot submit empty order");
        
        // Business rule: Cannot submit already submitted order
        if (Status != OrderStatus.Pending)
            throw new DomainException("Order is not in pending status");
        
        Status = OrderStatus.Submitted;
        OrderDate = DateTime.UtcNow;
        
        // Domain event
        AddDomainEvent(new OrderSubmittedEvent(this));
    }
    
    private void CalculateTotal()
    {
        TotalAmount = _orderItems.Sum(item => item.Price * item.Quantity);
    }
}

// Value Object - Immutable, no identity
public class Money : ValueObject
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new DomainException("Amount cannot be negative");
        
        if (string.IsNullOrWhiteSpace(currency) || currency.Length != 3)
            throw new DomainException("Invalid currency code");
        
        Amount = amount;
        Currency = currency.ToUpper();
    }
    
    // Value objects compared by value, not identity
    protected override bool EqualsCore(Money other)
    {
        return Amount == other.Amount && Currency == other.Currency;
    }
    
    protected override int GetHashCodeCore()
    {
        return HashCode.Combine(Amount, Currency);
    }
    
    // Operations return new instances (immutable)
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot add different currencies");
        
        return new Money(Amount + other.Amount, Currency);
    }
}
```

**Key Principles:**
- ✅ Entities enforce business rules
- ✅ Private setters prevent external modification
- ✅ Factory methods control creation
- ✅ Domain events communicate state changes
- ✅ Value objects are immutable
- ✅ Aggregates protect invariants

##### 6.2 Specification Pattern

```csharp
// Specification pattern for reusable business rules
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T entity);
    Expression<Func<T, bool>> Criteria { get; }
    List<Expression<Func<T, object>>> Includes { get; }
    List<string> IncludeStrings { get; }
}

// Base specification
public abstract class BaseSpecification<T> : ISpecification<T>
{
    public abstract Expression<Func<T, bool>> Criteria { get; }
    
    public List<Expression<Func<T, object>>> Includes { get; } = new();
    public List<string> IncludeStrings { get; } = new();
    
    protected virtual void AddInclude(Expression<Func<T, object>> includeExpression)
    {
        Includes.Add(includeExpression);
    }
    
    protected virtual void AddInclude(string includeString)
    {
        IncludeStrings.Add(includeString);
    }
}

// Concrete specifications
public class ProductInStockSpecification : BaseSpecification<Product>
{
    public ProductInStockSpecification()
    {
        Criteria = p => p.Stock > 0;
    }
}

public class ActiveProductsByCategorySpecification : BaseSpecification<Product>
{
    public ActiveProductsByCategorySpecification(Guid categoryId)
    {
        Criteria = p => p.CategoryId == categoryId 
                     && p.IsActive 
                     && p.Stock > 0;
        
        AddInclude(p => p.Category);
        AddInclude(p => p.Images);
    }
}

// Usage
public async Task<IReadOnlyList<Product>> GetActiveProductsAsync(Guid categoryId)
{
    var spec = new ActiveProductsByCategorySpecification(categoryId);
    return await _repository.ListAsync(spec);
}

// Repository with specification support
public async Task<IReadOnlyList<T>> ListAsync(ISpecification<T> spec)
{
    // Apply criteria
    var query = _context.Set<T>().Where(spec.Criteria);
    
    // Apply includes
    query = spec.Includes.Aggregate(query, (current, include) => current.Include(include));
    
    // Apply include strings
    query = spec.IncludeStrings.Aggregate(query, (current, include) => current.Include(include));
    
    return await query.ToListAsync();
}
```

**Benefits:**
- ✅ Reusable business rules
- ✅ Composable specifications
- ✅ Type-safe queries
- ✅ Centralized query logic

##### 6.3 Domain-Focused Modeling

```csharp
// Rich domain model with behavior
public class BankAccount : BaseEntity, IAggregateRoot
{
    public Guid AccountNumber { get; private set; }
    public decimal Balance { get; private set; }
    public AccountStatus Status { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    // Domain events
    private readonly List<IDomainEvent> _domainEvents = new();
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    // Business operations
    public void Deposit(decimal amount)
    {
        // Business rules
        if (amount <= 0)
            throw new DomainException("Deposit amount must be positive");
        
        if (Status != AccountStatus.Active)
            throw new DomainException("Cannot deposit to inactive account");
        
        Balance += amount;
        
        AddDomainEvent(new MoneyDepositedEvent(AccountNumber, amount, Balance));
    }
    
    public void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new DomainException("Withdrawal amount must be positive");
        
        if (Status != AccountStatus.Active)
            throw new DomainException("Cannot withdraw from inactive account");
        
        if (amount > Balance)
            throw new InsufficientFundsException(Balance, amount);
        
        Balance -= amount;
        
        AddDomainEvent(new MoneyWithdrawnEvent(AccountNumber, amount, Balance));
    }
    
    public void Close()
    {
        if (Balance != 0)
            throw new DomainException("Cannot close account with non-zero balance");
        
        Status = AccountStatus.Closed;
        
        AddDomainEvent(new AccountClosedEvent(AccountNumber));
    }
    
    private void AddDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}
```

#### Important Lesson

Studying both Jason Taylor's and Ardalis's repositories is more useful than treating either one as the only correct implementation.

**Two respected projects can follow similar architectural principles while choosing different:**
- Folder structures
- Project boundaries
- Naming conventions
- Abstractions
- Testing approaches
- Application-flow patterns

> **💡 Key Insight:** Architecture is not about finding one sacred diagram. It is about choosing boundaries that suit the problem.

#### Best For

- ✅ Developers building **business applications** with meaningful domain rules
- ✅ Learning **domain-driven design** principles
- ✅ Understanding **specification pattern**
- ✅ Studying different Clean Architecture interpretations

---

### 7. CleanArchitecture.WebApi by Mukesh Murugan

**Repository:** [iammukeshm/CleanArchitecture.WebApi](https://github.com/iammukeshm/CleanArchitecture.WebApi)

#### Overview

This project provides an **approachable Clean Architecture implementation** for ASP.NET Core Web APIs. It can be easier to follow for developers moving from controllers, services, and repositories toward a more structured application design.

#### Project Structure

```
CleanArchitecture.WebApi/
├── src/
│   ├── Core/                          # Domain & Application
│   │   ├── Domain/
│   │   │   ├── Entities/
│   │   │   ├── Interfaces/
│   │   │   └── Common/
│   │   └── Application/
│   │       ├── Common/
│   │       │   ├── Behaviors/
│   │       │   ├── Exceptions/
│   │       │   └── Mappings/
│   │       ├── Features/
│   │       └── Interfaces/
│   │
│   ├── Infrastructure/                # External concerns
│   │   ├── Persistence/
│   │   ├── Identity/
│   │   └── Services/
│   │
│   └── API/                           # Presentation
│       ├── Controllers/
│       ├── Middleware/
│       └── Extensions/
│
└── tests/
    ├── UnitTests/
    ├── IntegrationTests/
    └── FunctionalTests/
```

#### What to Study

##### 7.1 Project Separation

```csharp
// Core/Domain/Entities/Product.cs
public class Product
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public bool IsActive { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Core/Application/Features/Products/Commands/CreateProduct/
public record CreateProductCommand : IRequest<ApiResponse<Guid>>
{
    public string Name { get; init; }
    public string Description { get; init; }
    public decimal Price { get; init; }
    public int Stock { get; init; }
}

public class CreateProductCommandHandler 
    : IRequestHandler<CreateProductCommand, ApiResponse<Guid>>
{
    private readonly IApplicationDbContext _context;
    
    public async Task<ApiResponse<Guid>> Handle(
        CreateProductCommand request, 
        CancellationToken cancellationToken)
    {
        var product = new Product
        {
            Id = Guid.NewGuid(),
            Name = request.Name,
            Description = request.Description,
            Price = request.Price,
            Stock = request.Stock,
            IsActive = true,
            CreatedAt = DateTime.UtcNow
        };
        
        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);
        
        return new ApiResponse<Guid>
        {
            Data = product.Id,
            Message = "Product created successfully"
        };
    }
}

// API/Controllers/ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ISender _mediator;
    
    public ProductsController(ISender mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<ActionResult<ApiResponse<Guid>>> Create(
        CreateProductCommand command, 
        CancellationToken cancellationToken)
    {
        var result = await _mediator.Send(command, cancellationToken);
        return Ok(result);
    }
}
```

##### 7.2 API Response Wrapper

```csharp
// Common/ApiResponse.cs
public class ApiResponse<T>
{
    public ApiResponse()
    {
    }
    
    private ApiResponse(T data, string message = null)
    {
        Succeeded = true;
        Message = message;
        Data = data;
    }
    
    private ApiResponse(string message)
    {
        Succeeded = false;
        Message = message;
    }
    
    public bool Succeeded { get; set; }
    public string Message { get; set; }
    public T Data { get; set; }
    
    public static ApiResponse<T> Success(T data, string message = null)
    {
        return new ApiResponse<T>(data, message);
    }
    
    public static ApiResponse<T> Failure(string message)
    {
        return new ApiResponse<T>(message);
    }
}

// Usage
return ApiResponse<Guid>.Success(product.Id, "Product created successfully");
return ApiResponse<Guid>.Failure("Product not found");
```

**Benefits:**
- ✅ Consistent API response format
- ✅ Easy client-side handling
- ✅ Clear success/failure indication
- ✅ Standardized error messages

##### 7.3 Validation Pipeline

```csharp
// Behaviors/ValidationBehavior.cs
public class ValidationBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;
    
    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }
    
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        if (!_validators.Any())
        {
            return await next();
        }
        
        var context = new ValidationContext<TRequest>(request);
        
        var validationResults = await Task.WhenAll(
            _validators.Select(v => v.ValidateAsync(context, cancellationToken)));
        
        var failures = validationResults
            .SelectMany(r => r.Errors)
            .Where(f => f != null)
            .ToList();
        
        if (failures.Count != 0)
        {
            var response = new ApiResponse<TResponse>
            {
                Succeeded = false,
                Message = string.Join(", ", failures.Select(f => f.ErrorMessage))
            };
            
            // Cast to response type
            return (TResponse)Convert.ChangeType(response, typeof(TResponse));
        }
        
        return await next();
    }
}
```

#### Important Lesson

> **⚠️ Warning:** Do not introduce abstractions merely because they appear in a template.

Every interface should protect a useful boundary, improve testability, or isolate change. An interface with one implementation and no meaningful boundary may only add another file to maintain.

**Example of Unnecessary Abstraction:**

```csharp
// ❌ WRONG - Unnecessary abstraction
public interface IProductService
{
    Task<Product> GetByIdAsync(Guid id);
}

public class ProductService : IProductService
{
    public async Task<Product> GetByIdAsync(Guid id)
    {
        return await _context.Products.FindAsync(id);
    }
}

// Only one implementation, no boundary to protect
// Just adds maintenance burden

// ✅ CORRECT - Only abstract when needed
public interface IProductRepository
{
    Task<Product> GetByIdAsync(Guid id);
}

public class ProductRepository : IProductRepository
{
    private readonly DbContext _context;
    
    public async Task<Product> GetByIdAsync(Guid id)
    {
        return await _context.Products.FindAsync(id);
    }
}

// Abstraction justified because:
// 1. Multiple implementations possible (EF Core, Dapper, In-Memory)
// 2. Easy to mock for testing
// 3. Clear boundary between application and infrastructure
```

#### Best For

- ✅ Developers moving from **traditional layered applications** toward Clean Architecture
- ✅ Those who find other templates too complex
- ✅ Learning practical implementation patterns
- ✅ Understanding request flow across layers

---

### 8. Onion Architecture by Mukesh Murugan

**Repository:** [iammukeshm/OnionArchitecture](https://github.com/iammukeshm/OnionArchitecture)

#### Overview

Onion Architecture places the **domain near the center** of the application and pushes infrastructure concerns outward. Frameworks, databases, and external services should depend on the application's core abstractions rather than controlling the business model.

#### Onion Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Onion Architecture                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌───────────────┐                        │
│                    │  UI Layer     │                        │
│                    │ (Controllers) │                        │
│                    └───────┬───────┘                        │
│                            │                                │
│                    ┌───────▼───────┐                        │
│                    │  API Layer    │                        │
│                    │  (Services)   │                        │
│                    └───────┬───────┘                        │
│                            │                                │
│              ┌─────────────▼─────────────┐                  │
│              │   Application Layer       │                  │
│              │   (Use Cases)             │                  │
│              └─────────────┬─────────────┘                  │
│                            │                                │
│              ┌─────────────▼─────────────┐                  │
│              │    Domain Layer           │ ◄── CORE         │
│              │  (Entities & Rules)       │                  │
│              └─────────────┬─────────────┘                  │
│                            │                                │
│              ┌─────────────▼─────────────┐                  │
│              │ Infrastructure Layer      │                  │
│              │ (DB, External APIs, etc)  │                  │
│              └───────────────────────────┘                  │
│                                                             │
│  Direction: Outer layers depend on inner layers             │
│  Domain has ZERO dependencies on outer layers               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 8.1 Dependency Inversion

```csharp
// Domain Layer - NO infrastructure dependencies!
namespace OnionArchitecture.Domain.Entities;

public class Order : BaseEntity
{
    public Guid CustomerId { get; private set; }
    public decimal Total { get; private set; }
    public OrderStatus Status { get; private set; }
    
    // Domain logic only - pure C#
    public void AddItem(Product product, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Can only add items to draft orders");
        
        var orderItem = new OrderItem(product.Id, product.Price, quantity);
        _items.Add(orderItem);
        
        CalculateTotal();
    }
    
    private void CalculateTotal()
    {
        Total = _items.Sum(item => item.Price * item.Quantity);
    }
}

// Domain Layer - Defines interfaces
namespace OnionArchitecture.Domain.Interfaces;

public interface IOrderRepository
{
    Task<Order> GetByIdAsync(Guid id);
    Task AddAsync(Order order);
    Task UpdateAsync(Order order);
}

public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
}

// Application Layer - Uses domain interfaces
namespace OnionArchitecture.Application.Services;

public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IEmailService _emailService;
    
    public OrderService(IOrderRepository orderRepository, IEmailService emailService)
    {
        _orderRepository = orderRepository;
        _emailService = emailService;
    }
    
    public async Task SubmitOrderAsync(Guid orderId)
    {
        var order = await _orderRepository.GetByIdAsync(orderId);
        
        order.Submit();
        
        await _orderRepository.UpdateAsync(order);
        await _emailService.SendEmailAsync(
            order.CustomerEmail,
            "Order Confirmed",
            $"Your order {orderId} has been confirmed");
    }
}

// Infrastructure Layer - Implements interfaces
namespace OnionArchitecture.Infrastructure.Repositories;

public class OrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<Order> GetByIdAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
    }
    
    public async Task UpdateAsync(Order order)
    {
        _context.Orders.Update(order);
    }
}

// Infrastructure Layer - Email implementation
namespace OnionArchitecture.Infrastructure.Services;

public class EmailService : IEmailService
{
    private readonly SmtpClient _smtpClient;
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        var message = new MailMessage("noreply@example.com", to, subject, body);
        await _smtpClient.SendMailAsync(message);
    }
}
```

**Key Principle:** The domain layer defines WHAT the application does. Infrastructure defines HOW it's implemented.

##### 8.2 Core-Domain Independence

```csharp
// Domain layer has NO external dependencies
// <Project Sdk="Microsoft.NET.Sdk">

// <PropertyGroup>
//   <TargetFramework>net8.0</TargetFramework>
// </PropertyGroup>

// NO PackageReference to:
// - Entity Framework Core
// - MediatR
// - FluentValidation
// - Any external library

// Pure C# domain logic only!

// This ensures:
// 1. Domain can be tested without infrastructure
// 2. Domain logic is framework-agnostic
// 3. Domain can be reused in different applications
// 4. Domain is the most stable part of the system
```

**Benefits:**
- ✅ Domain logic is pure and testable
- ✅ No framework lock-in
- ✅ Can evolve independently
- ✅ Most stable part of the system

##### 8.3 Infrastructure Adapters

```csharp
// Infrastructure/Persistence/Configurations/OrderConfiguration.cs
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders");
        
        builder.HasKey(o => o.Id);
        
        builder.Property(o => o.Total)
            .HasPrecision(18, 2);
        
        builder.HasMany(o => o.Items)
            .WithOne()
            .HasForeignKey(oi => oi.OrderId)
            .OnDelete(DeleteBehavior.Cascade);
        
        builder.HasQueryFilter(o => !o.IsDeleted);
    }
}

// Infrastructure/Persistence/Repositories/OrderRepository.cs
public class OrderRepository : Repository<Order>, IOrderRepository
{
    public OrderRepository(ApplicationDbContext context) : base(context)
    {
    }
    
    public async Task<Order> GetByIdWithItemsAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
}

// Base repository with common operations
public abstract class Repository<T> where T : BaseEntity
{
    protected readonly ApplicationDbContext Context;
    
    protected Repository(ApplicationDbContext context)
    {
        Context = context;
    }
    
    public async Task<IReadOnlyList<T>> GetAllAsync()
    {
        return await Context.Set<T>().ToListAsync();
    }
    
    public async Task<T> GetByIdAsync(Guid id)
    {
        return await Context.Set<T>().FindAsync(id);
    }
    
    public async Task<T> AddAsync(T entity)
    {
        await Context.Set<T>().AddAsync(entity);
        await Context.SaveChangesAsync();
        return entity;
    }
    
    public async Task UpdateAsync(T entity)
    {
        Context.Set<T>().Update(entity);
        await Context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(T entity)
    {
        Context.Set<T>().Remove(entity);
        await Context.SaveChangesAsync();
    }
}
```

#### Important Lesson

> **💡 Key Insight:** The terminology differs, but the central principle is similar across Clean Architecture, Hexagonal Architecture, and Onion Architecture:

**Business rules should not depend directly on databases, user-interface frameworks, or third-party services.**

That does not mean infrastructure is unimportant. It means infrastructure should remain replaceable without rewriting the business model.

#### Comparison: Clean vs Onion vs Hexagonal

```
┌─────────────────────────────────────────────────────────────┐
│      Architecture Comparison Matrix                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Aspect          │ Clean Arch  │ Onion Arch │ Hexagonal     │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Core Focus      │ Use Cases   │ Domain     │ Ports/Adapt.  │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Outer Layer     │ Infrastruct.│ Infrastruct│ Adapters      │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Inner Layer     │ Domain      │ Domain     │ Domain        │
│  ──────────────  ┼─────────────┼────────────┼────────────── │
│  Dependency      │ Inward      │ Inward     │ Inward        │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Key Pattern     │ CQRS        │ Repository │ Port/Adapter  │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Complexity      │ High        │ Medium     │ Medium        │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Learning Curve  │ Steep       │ Moderate   │ Moderate      │
│  ────────────────┼─────────────┼────────────┼────────────── │
│  Best For        │ Large Apps  │ Bus. Rules │ Integration   │
│                                                             │
│  All three share:                                           │
│  • Dependency rule (inward)                                 │
│  • Domain independence                                      │
│  • Infrastructure replaceability                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Best For

- ✅ Developers comparing **Clean Architecture, Hexagonal Architecture, and Onion Architecture**
- ✅ Understanding **dependency inversion** deeply
- ✅ Learning **domain-focused modeling**
- ✅ Studying different architectural interpretations

---

### 9. Event Reminder by Milan Jovanović

**Repository:** [m-jovanovic/event-reminder](https://github.com/m-jovanovic/event-reminder)

#### Overview

Event Reminder demonstrates how **Domain-Driven Design, CQRS, Clean Architecture, and practical business workflows** can work together inside an application. This makes it particularly useful for developers who already understand architectural diagrams but want to see how those ideas connect with actual behavior.

#### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              Event Reminder System Architecture              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐                                          │
│  │   Client     │                                          │
│  │  (Blazor)    │                                          │
│  └──────┬───────┘                                          │
│         │                                                   │
│         │ HTTP/HTTPS                                        │
│         ▼                                                   │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │  API Layer   │─────►│ Application  │                    │
│  │ (Minimal API)│      │    Layer     │                    │
│  └──────────────┘      └──────┬───────┘                    │
│                               │                             │
│                               │                             │
│                    ┌──────────▼──────────┐                  │
│                    │   Domain Layer      │                  │
│                    │  ┌──────────────┐   │                  │
│                    │  │  Entities    │   │                  │
│                    │  │  (Event,     │   │                  │
│                    │  │  Reminder)   │   │                  │
│                    │  └──────────────┘   │                  │
│                    │  ┌──────────────┐   │                  │
│                    │  │   Events     │   │                  │
│                    │  │ (Domain)     │   │                  │
│                    │  └──────────────┘   │                  │
│                    └──────────┬──────────┘                  │
│                               │                             │
│         ┌─────────────────────┼─────────────────────┐      │
│         │                     │                     │      │
│         ▼                     ▼                     ▼      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ Persistence  │    │ Background   │    │ Integration  │  │
│  │   (EF Core)  │    │   Jobs       │    │   (Email)    │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 9.1 Domain Modeling

```csharp
// Domain/Entities/Event.cs
public class Event : BaseEntity, IAggregateRoot
{
    public string Title { get; private set; }
    public string Description { get; private set; }
    public DateTime StartDate { get; private set; }
    public DateTime EndDate { get; private set; }
    public Guid OrganizerId { get; private set; }
    public EventStatus Status { get; private set; }
    
    private readonly List<Reminder> _reminders = new();
    public IReadOnlyList<Reminder> Reminders => _reminders.AsReadOnly();
    
    // Private constructor for EF Core
    private Event() { }
    
    // Factory method
    public static Event Create(
        string title, 
        string description, 
        DateTime startDate, 
        DateTime endDate, 
        Guid organizerId)
    {
        // Business rules
        if (string.IsNullOrWhiteSpace(title))
            throw new DomainException("Event title is required");
        
        if (startDate >= endDate)
            throw new DomainException("Start date must be before end date");
        
        if (endDate < DateTime.UtcNow)
            throw new DomainException("Cannot create event in the past");
        
        return new Event
        {
            Id = Guid.NewGuid(),
            Title = title,
            Description = description,
            StartDate = startDate,
            EndDate = endDate,
            OrganizerId = organizerId,
            Status = EventStatus.Draft
        };
    }
    
    public void Publish()
    {
        if (Status != EventStatus.Draft)
            throw new DomainException("Only draft events can be published");
        
        Status = EventStatus.Published;
        
        AddDomainEvent(new EventPublishedEvent(this.Id, this.StartDate));
    }
    
    public void AddReminder(Reminder reminder)
    {
        if (Status != EventStatus.Published)
            throw new DomainException("Can only add reminders to published events");
        
        if (reminder.RemindAt >= StartDate)
            throw new DomainException("Reminder must be before event start");
        
        _reminders.Add(reminder);
    }
}

// Domain/Entities/Reminder.cs
public class Reminder : BaseEntity
{
    public Guid EventId { get; private set; }
    public Guid UserId { get; private set; }
    public DateTime RemindAt { get; private set; }
    public ReminderChannel Channel { get; private set; }
    public bool IsSent { get; private set; }
    public DateTime? SentAt { get; private set; }
    
    public void MarkAsSent()
    {
        if (IsSent)
            throw new DomainException("Reminder already sent");
        
        IsSent = true;
        SentAt = DateTime.UtcNow;
    }
}

// Domain/Events/EventPublishedEvent.cs
public class EventPublishedEvent : IDomainEvent
{
    public Guid EventId { get; }
    public DateTime EventStart { get; }
    public DateTime OccurredOn { get; }
    
    public EventPublishedEvent(Guid eventId, DateTime eventStart)
    {
        EventId = eventId;
        EventStart = eventStart;
        OccurredOn = DateTime.UtcNow;
    }
}
```

##### 9.2 Domain Events

```csharp
// Domain/Common/DomainEvent.cs
public interface IDomainEvent
{
    DateTime OccurredOn { get; }
}

// Domain/Common/Entity.cs
public abstract class BaseEntity
{
    private readonly List<IDomainEvent> _domainEvents = new();
    
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    protected void AddDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}

// Application/Behaviors/DomainEventDispatcherBehavior.cs
public class DomainEventDispatcherBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IDomainEventDispatcher _dispatcher;
    
    public DomainEventDispatcherBehavior(IDomainEventDispatcher dispatcher)
    {
        _dispatcher = dispatcher;
    }
    
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        // Execute the handler
        var response = await next();
        
        // Dispatch domain events
        await _dispatcher.DispatchEventsAsync(cancellationToken);
        
        return response;
    }
}

// Infrastructure/DomainEventDispatcher.cs
public class DomainEventDispatcher : IDomainEventDispatcher
{
    private readonly ApplicationDbContext _context;
    
    public async Task DispatchEventsAsync(CancellationToken cancellationToken)
    {
        var entities = _context.ChangeTracker
            .Entries<BaseEntity>()
            .Where(e => e.Entity.DomainEvents.Any())
            .Select(e => e.Entity);
        
        var domainEvents = entities
            .SelectMany(e => e.DomainEvents)
            .ToList();
        
        foreach (var domainEvent in domainEvents)
        {
            // Handle the event
            await HandleAsync(domainEvent, cancellationToken);
        }
        
        // Clear events
        foreach (var entity in entities)
        {
            entity.ClearDomainEvents();
        }
    }
    
    private async Task HandleAsync(IDomainEvent domainEvent, CancellationToken cancellationToken)
    {
        switch (domainEvent)
        {
            case EventPublishedEvent e:
                await HandleEventPublished(e, cancellationToken);
                break;
            
            case ReminderDueEvent e:
                await HandleReminderDue(e, cancellationToken);
                break;
        }
    }
    
    private async Task HandleEventPublished(
        EventPublishedEvent e, 
        CancellationToken cancellationToken)
    {
        // Create reminders for event attendees
        // Send notifications
        // Update calendar
    }
}
```

##### 9.3 Background Processing

```csharp
// Application/Jobs/ProcessRemindersJob.cs
public class ProcessRemindersJob : IProcessRemindersJob
{
    private readonly ApplicationDbContext _context;
    private readonly IEmailService _emailService;
    private readonly ISmsService _smsService;
    private readonly ILogger<ProcessRemindersJob> _logger;
    
    public async Task ExecuteAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Processing reminders at {Time}", DateTime.UtcNow);
        
        // Get reminders that are due
        var dueReminders = await _context.Reminders
            .Include(r => r.Event)
            .Where(r => !r.IsSent 
                     && r.RemindAt <= DateTime.UtcNow 
                     && r.Event.Status == EventStatus.Published)
            .ToListAsync(cancellationToken);
        
        _logger.LogInformation("Found {Count} reminders to process", dueReminders.Count);
        
        foreach (var reminder in dueReminders)
        {
            try
            {
                await SendReminderAsync(reminder, cancellationToken);
                reminder.MarkAsSent();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to send reminder {ReminderId}", reminder.Id);
                // Continue processing other reminders
            }
        }
        
        await _context.SaveChangesAsync(cancellationToken);
    }
    
    private async Task SendReminderAsync(Reminder reminder, CancellationToken cancellationToken)
    {
        var message = $"Reminder: {reminder.Event.Title} starts at {reminder.Event.StartDate}";
        
        switch (reminder.Channel)
        {
            case ReminderChannel.Email:
                await _emailService.SendEmailAsync(
                    reminder.UserEmail,
                    $"Reminder: {reminder.Event.Title}",
                    message);
                break;
            
            case ReminderChannel.Sms:
                await _smsService.SendSmsAsync(reminder.UserPhone, message);
                break;
            
            case ReminderChannel.Push:
                await _pushService.SendNotificationAsync(reminder.UserId, message);
                break;
        }
    }
}

// Register as hosted service
builder.Services.AddHostedService<ProcessRemindersJob>();
```

##### 9.4 Integration Boundaries

```csharp
// Infrastructure/Email/SendGridEmailService.cs
public class SendGridEmailService : IEmailService
{
    private readonly SendGridClient _client;
    private readonly ILogger<SendGridEmailService> _logger;
    
    public async Task SendEmailAsync(string to, string subject, string body)
    {
        var message = new SendGridMessage
        {
            From = new EmailAddress("noreply@eventreminder.com", "Event Reminder"),
            Subject = subject,
            PlainTextContent = body,
            HtmlContent = $"<p>{body}</p>"
        };
        
        message.AddTo(new EmailAddress(to));
        
        try
        {
            var response = await _client.SendEmailAsync(message);
            
            if (response.StatusCode != HttpStatusCode.Accepted)
            {
                _logger.LogWarning("Email send returned status: {StatusCode}", response.StatusCode);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to send email to {Recipient}", to);
            throw;
        }
    }
}

// Application/Interfaces/IEmailService.cs (Domain defines interface)
public interface IEmailService
{
    Task SendEmailAsync(string to, string subject, string body);
}
```

#### Important Lesson

> **💡 Key Insight:** A domain model becomes valuable when it expresses business rules.

A class containing properties that mirror a database table is not automatically a rich domain model. The model should protect invariants, control state changes, and make invalid operations difficult.

**❌ Anemic Domain Model (Anti-Pattern):**

```csharp
// Anemic - just properties, no behavior
public class Order
{
    public Guid Id { get; set; }
    public decimal Total { get; set; }
    public string Status { get; set; }
    public List<OrderItem> Items { get; set; }
}

// Business logic scattered in services
public class OrderService
{
    public void SubmitOrder(Order order)
    {
        if (order.Items.Count == 0)
            throw new Exception("Empty order");
        
        if (order.Total <= 0)
            throw new Exception("Invalid total");
        
        order.Status = "Submitted";
    }
}
```

**✅ Rich Domain Model:**

```csharp
// Rich - behavior encapsulated in entity
public class Order : BaseEntity
{
    private readonly List<OrderItem> _items = new();
    
    public decimal Total { get; private set; }
    public OrderStatus Status { get; private set; }
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();
    
    public void Submit()
    {
        // Business rules enforced HERE
        if (Status != OrderStatus.Draft)
            throw new DomainException("Only draft orders can be submitted");
        
        if (!_items.Any())
            throw new DomainException("Cannot submit empty order");
        
        if (Total <= 0)
            throw new DomainException("Order total must be positive");
        
        Status = OrderStatus.Submitted;
        
        AddDomainEvent(new OrderSubmittedEvent(this.Id));
    }
    
    public void AddItem(Product product, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Can only modify draft orders");
        
        var item = new OrderItem(product.Id, product.Price, quantity);
        _items.Add(item);
        
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        Total = _items.Sum(i => i.Price * i.Quantity);
    }
}
```

#### Best For

- ✅ Intermediate and senior developers studying how **architecture connects with domain behavior**
- ✅ Learning **Domain-Driven Design** in practice
- ✅ Understanding **domain events** and **background processing**
- ✅ Seeing complete business workflows

---

### 10. .NET eShop

**Repository:** [dotnet/eShop](https://github.com/dotnet/eShop)

#### Overview

Microsoft's .NET eShop reference application demonstrates a **distributed application** built using modern .NET technologies. It covers problems that appear when an application is divided across multiple services and processes.

#### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    .NET eShop Architecture                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              .NET Aspire Orchestration                │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Service Discovery & Configuration             │  │  │
│  │  │  • Automatic service registration              │  │  │
│  │  │  • Health monitoring                            │  │  │
│  │  │  • Distributed tracing                          │  │  │
│  │  │  • Logging aggregation                         │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│                            │                                │
│         ┌──────────────────┼──────────────────┐            │
│         │                  │                  │            │
│         ▼                  ▼                  ▼            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Catalog    │  │   Basket     │  │   Ordering   │     │
│  │   Service    │  │   Service    │  │   Service    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │            │
│         └──────────────────┼──────────────────┘            │
│                            │                                │
│                    ┌───────▼────────┐                       │
│                    │  SQL Database  │                       │
│                    │  (PostgreSQL)  │                       │
│                    └───────────────┘                       │
│                                                             │
│  Additional Services:                                       │
│  • Identity Service (authentication)                        │
│  • Marketing Service (campaigns)                            │
│  • Payment Service (transactions)                           │
│  • Web Status (monitoring dashboard)                        │
│  • Web SPA (Angular frontend)                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### What to Study

##### 10.1 Service Boundaries

```csharp
// Catalog Service - Product catalog management
namespace Microsoft.eShopOnContainers.Services.Catalog.API;

[ApiController]
[Route("api/v1/[controller]")]
public class CatalogController : ControllerBase
{
    private readonly ICatalogService _catalogService;
    
    [HttpGet]
    [ProducesResponseType(typeof(PaginatedItemsViewModel<CatalogItem>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PaginatedItemsViewModel<CatalogItem>>> GetItems(
        [FromQuery] int pageSize = 10, 
        [FromQuery] int pageIndex = 0)
    {
        var items = await _catalogService.GetCatalogItemsAsync(pageSize, pageIndex);
        
        return Ok(items);
    }
    
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    public async Task<ActionResult<CatalogItem>> CreateItem([FromBody] CatalogItem item)
    {
        await _catalogService.CreateCatalogItemAsync(item);
        
        return CreatedAtAction(nameof(GetItem), new { id = item.Id }, item);
    }
}

// Basket Service - Shopping cart management
namespace Microsoft.eShopOnContainers.Services.Basket.API;

[ApiController]
[Route("api/v1/[controller]")]
public class BasketController : ControllerBase
{
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<CustomerBasket>> GetBasketById(string id)
    {
        var basket = await _basketRepository.GetBasketAsync(id);
        return Ok(basket ?? new CustomerBasket(id));
    }
    
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<IActionResult> UpdateBasket([FromBody] CustomerBasket basket)
    {
        await _basketRepository.UpdateBasketAsync(basket);
        return Ok();
    }
}

// Ordering Service - Order processing
namespace Microsoft.eShopOnContainers.Services.Ordering.API;

[ApiController]
[Route("api/v1/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    public async Task<ActionResult<Order>> CreateOrder([FromBody] OrderDto orderDto)
    {
        var order = await _orderingService.CreateOrderAsync(orderDto);
        
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
}
```

##### 10.2 .NET Aspire Orchestration

```csharp
// AppHost/Program.cs - Service orchestration
var builder = DistributedApplication.CreateBuilder(args);

// Catalog Service
var catalogDb = builder.AddPostgres("catalogdb")
    .AddDatabase("catalog");
var catalogCache = builder.AddRedis("catalogcache");
var catalogService = builder.AddProject<Projects.Catalog_Service>("catalog-service")
    .WithReference(catalogDb)
    .WithReference(catalogCache);

// Basket Service
var basketDb = builder.AddPostgres("basketdb")
    .AddDatabase("basket");
var basketCache = builder.AddRedis("basketcache");
var basketService = builder.AddProject<Projects.Basket_Service>("basket-service")
    .WithReference(basketDb)
    .WithReference(basketCache);

// Ordering Service
var orderingDb = builder.AddPostgres("orderingdb")
    .AddDatabase("ordering");
var orderingService = builder.AddProject<Projects.Ordering_Service>("ordering-service")
    .WithReference(orderingDb)
    .WithReference(catalogService)
    .WithReference(basketService);

// Identity Service
var identityDb = builder.AddPostgres("identitydb");
var identityService = builder.AddProject<Projects.Identity_Service>("identity-service")
    .WithReference(identityDb);

// Web Frontend
var webApp = builder.AddProject<Projects.Web_App>("webapp")
    .WithReference(catalogService)
    .WithReference(basketService)
    .WithReference(orderingService)
    .WithReference(identityService);

builder.Build().Run();
```

**Benefits of .NET Aspire:**
- ✅ Automatic service discovery
- ✅ Configuration management
- ✅ Health monitoring
- ✅ Distributed tracing
- ✅ Simplified local development
- ✅ Easy deployment

##### 10.3 Messaging

```csharp
// Ordering/Application/IntegrationEvents/EventHandling/
public class OrderCreatedIntegrationEventHandler : 
    IIntegrationEventHandler<OrderCreatedIntegrationEvent>
{
    private readonly ILogger<OrderCreatedIntegrationEventHandler> _logger;
    
    public async Task Handle(OrderCreatedIntegrationEvent @event)
    {
        _logger.LogInformation(
            "Handling OrderCreated event: {OrderId}", 
            @event.OrderId);
        
        // Update read model
        await _orderSummaryRepository.AddAsync(new OrderSummary
        {
            OrderId = @event.OrderId,
            UserName = @event.UserName,
            Total = @event.Total,
            OrderDate = @event.OrderDate
        });
        
        // Send notification
        await _notificationService.SendOrderConfirmationAsync(
            @event.UserId, 
            @event.OrderId);
    }
}

// Publishing events
public class Order : BaseEntity, IAggregateRoot
{
    // ... entity properties ...
    
    public async Task<bool> ShipAsync()
    {
        if (Status != OrderStatus.Confirmed)
            return false;
        
        Status = OrderStatus.Shipped;
        
        // Publish integration event
        var orderShippedEvent = new OrderShippedIntegrationEvent
        {
            OrderId = Id,
            ShippedDate = DateTime.UtcNow,
            UserId = UserId
        };
        
        await _integrationEventService.PublishAsync(orderShippedEvent);
        
        return true;
    }
}
```

##### 10.4 Observability

```csharp
// Program.cs - Distributed tracing
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddEntityFrameworkCoreInstrumentation()
            .AddJaegerExporter(options =>
            {
                options.AgentHost = builder.Configuration["Jaeger:Host"];
                options.AgentPort = int.Parse(builder.Configuration["Jaeger:Port"]);
            });
    })
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddPrometheusExporter();
    });

// Health checks with detailed information
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy())
    .AddPostgres(builder.Configuration.GetConnectionString("DefaultConnection"))
    .AddRedis(builder.Configuration.GetConnectionString("Redis"))
    .AddRabbitMQ(builder.Configuration.GetConnectionString("MessageBus"));

// Structured logging
builder.Logging.AddOpenTelemetry(options =>
{
    options.IncludeFormattedMessage = true;
    options.IncludeScopes = true;
});
```

##### 10.5 Resilience

```csharp
// Resilience/Polly policies
public static class ResiliencePolicies
{
    public static AsyncRetryPolicy<HttpResponseMessage> HttpRetryPolicy =>
        HttpPolicyExtensions
            .HandleTransientHttpError()
            .OrResult(msg => !msg.IsSuccessStatusCode)
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: attempt => 
                    TimeSpan.FromSeconds(Math.Pow(2, attempt)),
                onRetry: (outcome, timespan, retryAttempt, context) =>
                {
                    Log.Warning(
                        "Retry {RetryAttempt} for {RequestUri} due to {Reason}",
                        retryAttempt,
                        context["requestUri"],
                        outcome.Exception?.Message ?? outcome.Result.StatusCode.ToString());
                });
    
    public static AsyncCircuitBreakerPolicy<HttpResponseMessage> CircuitBreakerPolicy =>
        HttpPolicyExtensions
            .HandleTransientHttpError()
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(30));
    
    public static AsyncBulkheadPolicy<HttpResponseMessage> BulkheadPolicy =>
        Policy<HttpResponseMessage>
            .BulkheadAsync(
                maxParallelization: 10,
                maxQueuingActions: 20);
}

// Usage
var httpClient = new HttpClient();
httpClient.AddPolicyHandler(ResiliencePolicies.HttpRetryPolicy);
httpClient.AddPolicyHandler(ResiliencePolicies.CircuitBreakerPolicy);
httpClient.AddPolicyHandler(ResiliencePolicies.BulkheadPolicy);
```

#### Important Warning

> **⚠️ Critical Warning:** Do not use a microservices reference application as the default template for a small business system.

Microservices introduce real costs:

| Cost | Description |
|------|-------------|
| **Network Failures** | Services communicate over network, introducing latency and failure points |
| **Distributed Transactions** | ACID transactions across services are complex |
| **Message Consistency** | Eventual consistency requires careful design |
| **Deployment Coordination** | Multiple services need orchestrated deployments |
| **Observability Requirements** | Need distributed tracing, centralized logging |
| **More Infrastructure** | Service discovery, API gateways, message brokers |
| **Difficult Local Development** | Running multiple services locally is complex |
| **More Operational Responsibility** | Monitoring, scaling, and maintaining multiple services |

**Study the patterns, but apply only what your problem genuinely requires.**

> **💡 Key Insight:** A well-structured monolith is often a better starting point.

#### When to Use Microservices

| Scenario | Recommendation |
|----------|----------------|
| Small team (<5 developers) | ❌ Start with monolith |
| Simple domain (<10 entities) | ❌ Monolith is sufficient |
| Rapid prototyping | ❌ Monolith for speed |
| Well-understood domain | ❌ Monolith |
| Large team (20+ developers) | ✅ Consider microservices |
| Complex domain with clear boundaries | ✅ Microservices may help |
| Need independent scaling | ✅ Microservices beneficial |
| Different technology requirements | ✅ Microservices allow polyglot |

#### Best For

- ✅ Senior developers and teams exploring **distributed systems**
- ✅ Learning **cloud-native development**
- ✅ Understanding **service-based architectures**
- ✅ Studying **.NET Aspire** and modern orchestration

---

## Architectural Patterns Comparison

### Side-by-Side Comparison

```
┌─────────────────────────────────────────────────────────────┐
│         Architectural Patterns Comparison                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Pattern          │ Complexity │ Learning │ Testability │   │
│  ─────────────────┼────────────┼──────────┼───────────── │   │
│  Clean Arch       │ High       │ Steep    │ Excellent   │   │
│  ─────────────────┼────────────┼──────────┼───────────── │   │
│  Onion Arch       │ Medium     │ Moderate │ Excellent   │   │
│  ─────────────────┼────────────┼──────────┼───────────── │   │
│  Hexagonal Arch   │ Medium     │ Moderate │ Excellent   │   │
│  ─────────────────┼────────────┼──────────┼───────────── │   │
│  Vertical Slice   │ Low        │ Easy     │ Good        │   │
│  ─────────────────┼────────────┼──────────┼───────────── │   │
│  Modular Monolith │ Medium     │ Moderate │ Good        │   │
│  ─────────────────┼────────────┼──────────┼───────────── │   │
│  Layered Arch     │ Low        │ Easy     │ Fair        │   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Decision Tree

```
                    ┌─────────────────┐
                    │  Start Here     │
                    └────────┬────────┘
                             │
                    What's your team size?
                             │
            ┌────────────────┼────────────────┐
            │                │                │
     Solo/2-3 devs      4-10 devs       10+ devs
            │                │                │
            ▼                ▼                ▼
    ┌───────────────┐  ┌───────────────┐  ┌───────────────┐
    │  Simple       │  │  Consider     │  │  Likely need  │
    │  Layered or   │  │  Clean/Onion  │  │  microservices│
    │  Vertical     │  │  Architecture │  │  or modular   │
    │  Slice        │  │               │  │  monolith     │
    └───────────────┘  └───────────────┘  └───────────────┘
                             │
                    What's the domain complexity?
                             │
            ┌────────────────┼────────────────┐
            │                │                │
     Simple CRUD      Complex business    Long-lived
                     rules & workflows    application
            │                │                │
            ▼                ▼                ▼
    ┌───────────────┐  ┌───────────────┐  ┌───────────────┐
    │  Vertical     │  │  Clean/Onion  │  │  Clean Arch   │
    │  Slice works  │  │  Architecture │  │  with DDD     │
    │               │  │  with DDD     │  │               │
    └───────────────┘  └───────────────┘  └───────────────┘
```

### Pattern Comparison Matrix

| Aspect | Clean Architecture | Onion Architecture | Vertical Slice | Modular Monolith |
|--------|-------------------|-------------------|----------------|------------------|
| **Dependency Rule** | ✅ Strict inward | ✅ Strict inward | ⚠️ Feature-based | ✅ Module boundaries |
| **Testability** | ✅ Excellent | ✅ Excellent | ✅ Good | ✅ Good |
| **Learning Curve** | ⚠️ Steep | ⚠️ Moderate | ✅ Easy | ⚠️ Moderate |
| **Boilerplate** | ⚠️ High | ⚠️ Medium | ✅ Low | ⚠️ Medium |
| **Flexibility** | ✅ High | ✅ High | ⚠️ Medium | ✅ High |
| **Team Scalability** | ✅ Excellent | ✅ Good | ⚠️ Fair | ✅ Good |
| **Overhead** | ⚠️ High | ⚠️ Medium | ✅ Low | ⚠️ Medium |
| **Best For** | Large enterprise | Business logic | Simple features | Growing systems |

---

## How to Study a Repository Properly

> **💡 Key Insight:** Do not begin by reading every file. Large repositories become overwhelming when explored without a specific question.

### The Right Approach

```
┌─────────────────────────────────────────────────────────────┐
│          Repository Study Methodology                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Step 1: Read the README                                    │
│  Step 2: Run the application                                │
│  Step 3: Identify the main user journey                     │
│  Step 4: Trace one successful request                       │
│  Step 5: Trace one failure                                  │
│  Step 6: Read the relevant tests                            │
│  Step 7: Review dependency registration                     │
│  Step 8: Review configuration files                         │
│  Step 9: Change one small feature                           │
│  Step 10: Explain the architecture in your own words        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Detailed Steps

#### Step 1: Read the README

```markdown
Look for:
- Project purpose and goals
- Architecture overview
- Technology stack
- Setup instructions
- Key features
- Contributing guidelines
- License information
```

**Questions to Answer:**
- What problem does this project solve?
- What architectural patterns does it use?
- What's the technology stack?
- How is the project organized?

#### Step 2: Run the Application

```bash
# Clone the repository
git clone https://github.com/example/project.git
cd project

# Install dependencies
dotnet restore

# Run the application
dotnet run

# Or use Docker
docker-compose up
```

**What to Observe:**
- Startup time
- Configuration requirements
- Database setup
- External dependencies
- Default routes/endpoints

#### Step 3: Identify the Main User Journey

Choose one core feature and understand it completely:

**Example: Creating an Order**

```
User Journey:
1. User browses products
2. User adds items to cart
3. User proceeds to checkout
4. User enters shipping information
5. User selects payment method
6. User confirms order
7. System processes payment
8. System creates order
9. System sends confirmation email
10. User sees order confirmation
```

#### Step 4: Trace One Successful Request

Follow the complete flow:

```
HTTP Request
    ↓
Endpoint/Controller
    ↓
Validation
    ↓
Application Command/Query
    ↓
Domain Behavior
    ↓
Persistence
    ↓
Events
    ↓
Response
```

**Example Trace:**

```csharp
// 1. HTTP Request
POST /api/orders
{
    "items": [
        { "productId": "guid", "quantity": 2 }
    ],
    "shippingAddress": "123 Main St"
}

// 2. Controller
[HttpPost]
public async Task<ActionResult<Guid>> Create(CreateOrderCommand command)
{
    var result = await _mediator.Send(command);
    return Ok(result);
}

// 3. Validation
public class CreateOrderCommandValidator : AbstractValidator<CreateOrderCommand>
{
    public CreateOrderCommandValidator()
    {
        RuleFor(x => x.Items).NotEmpty();
        RuleForEach(x => x.Items).ChildRules(items =>
        {
            items.RuleFor(i => i.Quantity).GreaterThan(0);
        });
    }
}

// 4. Command Handler
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken ct)
    {
        var order = Order.Create(customerId, request.Items);
        await _repository.AddAsync(order, ct);
        await _unitOfWork.SaveChangesAsync(ct);
        return order.Id;
    }
}

// 5. Domain Entity
public class Order : BaseEntity
{
    public static Order Create(Guid customerId, List<OrderItem> items)
    {
        // Business rules
        if (!items.Any())
            throw new DomainException("Order must have items");
        
        var order = new Order { /* ... */ };
        
        foreach (var item in items)
        {
            order.AddItem(item);
        }
        
        return order;
    }
}

// 6. Repository
public async Task AddAsync(Order order, CancellationToken ct)
{
    await _context.Orders.AddAsync(order, ct);
}

// 7. Domain Event
public class OrderCreatedEvent : IDomainEvent
{
    public Guid OrderId { get; }
    public DateTime OccurredOn { get; }
}

// 8. Response
{
    "orderId": "guid",
    "status": "created"
}
```

#### Step 5: Trace One Failure

```csharp
// Send invalid input
POST /api/orders
{
    "items": [],  // Empty - should fail
    "shippingAddress": ""
}

// Trace the failure:
// 1. Controller receives request
// 2. Validation pipeline catches empty items
// 3. ValidationBehavior throws ValidationException
// 4. Exception handling middleware catches it
// 5. Returns 400 Bad Request with error details

// Response:
{
    "errors": [
        "Order must contain at least one item",
        "Shipping address is required"
    ]
}
```

#### Step 6: Read the Relevant Tests

```csharp
// Unit test for command handler
[Test]
public async Task CreateOrder_WithValidData_ReturnsSuccess()
{
    // Arrange
    var mockRepo = new Mock<IOrderRepository>();
    var handler = new CreateOrderCommandHandler(mockRepo.Object);
    
    var command = new CreateOrderCommand(/* valid data */);
    
    // Act
    var result = await handler.Handle(command, CancellationToken.None);
    
    // Assert
    Assert.IsTrue(result.IsSuccess);
    Assert.IsNotNull(result.Data);
}

// Integration test
[Test]
public async Task CreateOrder_WithValidData_PersistsToDatabase()
{
    // Arrange
    await using var context = await TestDbContextHelper.CreateAsync();
    var handler = new CreateOrderCommandHandler(context);
    
    // Act
    var result = await handler.Handle(command, CancellationToken.None);
    
    // Assert
    var order = await context.Orders.FindAsync(result.Data);
    Assert.IsNotNull(order);
    Assert.AreEqual(2, order.Items.Count);
}
```

#### Step 7: Review Dependency Registration

```csharp
// Program.cs
builder.Services
    .AddApplicationServices()      // MediatR, Validators, Behaviors
    .AddInfrastructureServices()   // DbContext, Repositories, Email
    .AddWebServices();             // Controllers, Authentication

// Extension methods
public static IServiceCollection AddApplicationServices(this IServiceCollection services)
{
    // MediatR - CQRS
    services.AddMediatR(cfg => 
        cfg.RegisterServicesFromAssembly(typeof(ApplicationAssemblyMarker).Assembly));
    
    // Validation
    services.AddValidatorsFromAssembly(typeof(ApplicationAssemblyMarker).Assembly);
    
    // Pipeline behaviors
    services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
    services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
    services.AddTransient(typeof(IPipelineBehavior<,>), typeof(PerformanceBehavior<,>));
    
    return services;
}
```

#### Step 8: Review Configuration Files

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;..."
  },
  "JwtSettings": {
    "Secret": "your-secret-key",
    "ExpiryInMinutes": 60
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}

// appsettings.Development.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp_Dev;..."
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

#### Step 9: Change One Small Feature

```csharp
// Add a new validation rule
public class CreateProductCommandValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .MaximumLength(100);
        
        RuleFor(x => x.Price)
            .GreaterThan(0)
            .LessThan(1000000);  // NEW: Maximum price
        
        RuleFor(x => x.Stock)
            .GreaterThanOrEqualTo(0);
    }
}

// Add a new field to entity
public class Product : BaseEntity
{
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    public int Stock { get; private set; }
    public string Sku { get; private set; }  // NEW: SKU field
}

// Update DTO
public record CreateProductCommand(
    string Name,
    string Description,
    decimal Price,
    int Stock,
    string Sku  // NEW
) : IRequest<Result<Guid>>;
```

#### Step 10: Explain the Architecture

Write a summary document explaining:
- The architectural pattern used
- How layers interact
- Why certain decisions were made
- What you would change and why

### Practical Exercise: Complete Request Trace

**Exercise:** Choose the "Create Product" feature and trace it completely.

```
1. HTTP Request: POST /api/products
   ↓
2. Controller: ProductsController.Create()
   ↓
3. Validation: CreateProductCommandValidator
   ↓
4. MediatR: Send(CreateProductCommand)
   ↓
5. Pipeline: ValidationBehavior → LoggingBehavior → Handler
   ↓
6. Handler: CreateProductCommandHandler.Handle()
   ↓
7. Domain: Product.Create() factory method
   ↓
8. Repository: AddAsync(product)
   ↓
9. DbContext: SaveChangesAsync()
   ↓
10. Database: INSERT INTO Products
   ↓
11. Domain Event: ProductCreatedEvent
   ↓
12. Response: 201 Created with product ID
```

**Now trace a failure:**
- Invalid input → Validation fails → 400 Bad Request
- Duplicate product name → Domain exception → 409 Conflict
- Database error → Exception → 500 Internal Server Error

---

## Practical Implementation Guide

### When to Use Which Pattern

```
┌─────────────────────────────────────────────────────────────┐
│              Pattern Selection Guide                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Project Size              Recommended Pattern              │
│  ─────────────────────────┼──────────────────────────────── │
│  Small (< 10 entities)     │ Vertical Slice or Layered      │
│  ─────────────────────────┼──────────────────────────────── │
│  Medium (10-50 entities)   │ Clean/Onion Architecture       │
│  ─────────────────────────┼──────────────────────────────── │
│  Large (50+ entities)      │ Clean Architecture with DDD     │
│  ─────────────────────────┼──────────────────────────────── │
│  Distributed System        │ Microservices or Modular Mono   │
│                                                             │
│  Team Size                 Recommended Pattern              │
│  ─────────────────────────┼──────────────────────────────── │
│  Solo/2-3 developers       │ Vertical Slice                  │
│  ─────────────────────────┼──────────────────────────────── │
│  4-10 developers           │ Clean/Onion Architecture        │
│  ─────────────────────────┼──────────────────────────────── │
│  10+ developers            │ Microservices or Modular Mono   │
│                                                             │
│  Project Lifespan          Recommended Pattern              │
│  ─────────────────────────┼──────────────────────────────── │
│  Short-term (< 1 year)     │ Simple Layered                 │
│  ─────────────────────────┼──────────────────────────────── │
│  Medium-term (1-3 years)   │ Clean/Onion Architecture       │
│  ─────────────────────────┼──────────────────────────────── │
│  Long-term (3+ years)      │ Clean Architecture with DDD     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Implementation Checklist

#### Starting a New Project

- [ ] Choose appropriate architecture for project size
- [ ] Set up project structure
- [ ] Configure dependency injection
- [ ] Add logging framework
- [ ] Add health checks
- [ ] Configure security headers
- [ ] Set up database
- [ ] Add authentication/authorization
- [ ] Configure API documentation
- [ ] Add error handling middleware
- [ ] Set up testing framework
- [ ] Configure CI/CD pipeline
- [ ] Add Docker support
- [ ] Document architecture decisions

#### Migrating to More Complex Architecture

- [ ] Assess current architecture
- [ ] Identify pain points
- [ ] Plan migration phases
- [ ] Extract domain layer first
- [ ] Introduce application layer
- [ ] Move business logic to domain
- [ ] Add repository interfaces
- [ ] Implement infrastructure
- [ ] Update tests
- [ ] Deploy incrementally

---

## Common Pitfalls and Anti-Patterns

### Anti-Pattern 1: Over-Engineering

```
❌ WRONG - Over-Engineered
├── Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Events/
│   ├── Exceptions/
│   └── Interfaces/
├── Application/
│   ├── Commands/
│   ├── Queries/
│   ├── Validators/
│   ├── Behaviors/
│   ├── DTOs/
│   └── Interfaces/
├── Infrastructure/
│   ├── Persistence/
│   ├── Repositories/
│   ├── Services/
│   └── Configurations/
└── API/
    ├── Controllers/
    ├── Middleware/
    └── Extensions/

For a simple blog with 3 entities!
```

**When to Avoid:**
- Small projects (< 10 entities)
- Short-lived projects
- Solo developers
- Rapid prototyping

### Anti-Pattern 2: Copy-Paste Architecture

```
❌ WRONG - Copying without understanding

"Let's use Clean Architecture because the tutorial said so"

Result:
- Unnecessary complexity
- Team doesn't understand it
- Hard to maintain
- Slower development
```

**Solution:**
- Understand why each layer exists
- Start simple, add complexity when needed
- Refactor incrementally

### Anti-Pattern 3: Anemic Domain Model

```csharp
// ❌ WRONG - Anemic domain
public class Order
{
    public Guid Id { get; set; }
    public decimal Total { get; set; }
    public string Status { get; set; }
}

// Business logic in service
public class OrderService
{
    public void Submit(Order order)
    {
        if (order.Total <= 0)
            throw new Exception("Invalid total");
        
        order.Status = "Submitted";
    }
}

// ✅ CORRECT - Rich domain model
public class Order : BaseEntity
{
    public decimal Total { get; private set; }
    public OrderStatus Status { get; private set; }
    
    public void Submit()
    {
        if (Total <= 0)
            throw new DomainException("Invalid total");
        
        if (Status != OrderStatus.Draft)
            throw new DomainException("Only draft orders can be submitted");
        
        Status = OrderStatus.Submitted;
        AddDomainEvent(new OrderSubmittedEvent(Id));
    }
}
```

### Anti-Pattern 4: Repository Over-Use

```csharp
// ❌ WRONG - Repository for everything
public interface IEverythingRepository
{
    Task<object> GetData(string query);
}

// ✅ CORRECT - Use repository for aggregates
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(Guid id);
    Task AddAsync(Order order);
}

// For complex queries, use specifications or queries
public class GetOrderReportQuery : IRequest<List<OrderReportDto>>
{
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }
}
```

### Anti-Pattern 5: God Services

```csharp
// ❌ WRONG - Service does everything
public class ProductService
{
    public void Create() { }
    public void Update() { }
    public void Delete() { }
    public void Search() { }
    public void Report() { }
    public void Import() { }
    public void Export() { }
    // 50 more methods...
}

// ✅ CORRECT - Single responsibility
public class CreateProductCommandHandler { }
public class UpdateProductCommandHandler { }
public class SearchProductsQueryHandler { }
public class ProductReportGenerator { }
```

---

## Best Practices

### 1. Follow the Dependency Rule

```
✅ Inner layers never depend on outer layers
✅ Dependencies point inward only
✅ Domain layer has zero external dependencies
✅ Use interfaces to invert dependencies
```

### 2. Keep Domain Logic in Domain Layer

```
✅ Business rules in entities
✅ Domain events for side effects
✅ Value objects for immutable concepts
✅ Domain exceptions for business rule violations
```

### 3. Use CQRS for Complex Operations

```
✅ Commands for state changes
✅ Queries for data retrieval
✅ Separate read and write models when needed
✅ MediatR for decoupling
```

### 4. Implement Proper Validation

```
✅ FluentValidation for complex rules
✅ DataAnnotations for simple validation
✅ Validation in pipeline behaviors
✅ Domain validation in entities
```

### 5. Handle Errors Gracefully

```
✅ Global exception handling middleware
✅ Structured logging
✅ Meaningful error messages
✅ ProblemDetails for API errors
```

### 6. Write Tests at Every Layer

```
✅ Unit tests for domain logic
✅ Unit tests for application handlers
✅ Integration tests for repositories
✅ End-to-end tests for critical paths
```

### 7. Use Dependency Injection

```
✅ Constructor injection
✅ Interface-based design
✅ Avoid service locator pattern
✅ Register services by lifetime
```

### 8. Configure for Different Environments

```
✅ appsettings.json for defaults
✅ appsettings.Development.json for dev
✅ appsettings.Production.json for prod
✅ Environment variables for secrets
✅ User secrets for local development
```

---

## Practice Exercises with Solutions

### Exercise 1: Analyze a Codebase

**Task:** Clone the [fullstackhero/dotnet-starter-kit](https://github.com/fullstackhero/dotnet-starter-kit) and analyze its architecture.

**Requirements:**
1. Identify the project structure
2. Map out the dependency flow
3. Document how authentication is implemented
4. Explain the multitenancy approach
5. Identify 3 patterns you can apply to your projects

**Solution:**

```markdown
## Analysis Report

### 1. Project Structure

The project follows a feature-based structure:

```
src/
├── Core/                          # Domain & Application
│   ├── Domain/
│   │   ├── Common/
│   │   ├── Entities/
│   │   ├── Enums/
│   │   ├── Exceptions/
│   │   └── Interfaces/
│   └── Application/
│       ├── Common/
│       │   ├── Behaviors/
│       │   ├── Exceptions/
│       │   ├── Mappings/
│       │   └── Models/
│       ├── Contracts/
│       └── Features/
│
├── Infrastructure/                # External concerns
│   ├── Persistence/
│   ├── Security/
│   └── Services/
│
└── API/                           # Presentation
    ├── Endpoints/
    ├── Middleware/
    └── Program.cs
```

### 2. Dependency Flow

```
API → Application → Domain
Infrastructure → Application → Domain

Dependencies point INWARD ✓
```

### 3. Authentication Implementation

- Uses JWT Bearer Authentication
- Token generation in Infrastructure layer
- Claims-based authorization
- Policy-based access control

### 4. Multitenancy Approach

- ITenantProvider interface in Domain
- Tenant resolution in middleware
- Global query filters for data isolation
- Per-tenant database support

### 5. Applicable Patterns

1. **Pipeline Behaviors** - For cross-cutting concerns
2. **Feature-based Organization** - Better than layer-based
3. **Result Pattern** - For explicit success/failure handling
```

### Exercise 2: Refactor to Clean Architecture

**Task:** Refactor a simple controller-based API to Clean Architecture.

**Starting Code:**

```csharp
// ❌ Starting point - All in one place
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _context;
    
    [HttpPost]
    public async Task<ActionResult<Product>> Create(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(Get), new { id = product.Id }, product);
    }
}
```

**Solution:**

```csharp
// Step 1: Create Command
public record CreateProductCommand(string Name, decimal Price, int Stock) 
    : IRequest<Result<Guid>>;

// Step 2: Create Validator
public class CreateProductCommandValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(100);
        RuleFor(x => x.Price).GreaterThan(0);
        RuleFor(x => x.Stock).GreaterThanOrEqualTo(0);
    }
}

// Step 3: Create Handler
public class CreateProductCommandHandler 
    : IRequestHandler<CreateProductCommand, Result<Guid>>
{
    private readonly IProductRepository _repository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task<Result<Guid>> Handle(
        CreateProductCommand request, 
        CancellationToken ct)
    {
        var product = Product.Create(request.Name, request.Price, request.Stock);
        
        await _repository.AddAsync(product, ct);
        await _unitOfWork.SaveChangesAsync(ct);
        
        return Result<Guid>.Success(product.Id);
    }
}

// Step 4: Update Controller
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ISender _mediator;
    
    [HttpPost]
    public async Task<ActionResult<Result<Guid>>> Create(
        CreateProductCommand command, 
        CancellationToken ct)
    {
        var result = await _mediator.Send(command, ct);
        
        if (result.IsSuccess)
        {
            return CreatedAtAction(nameof(Get), new { id = result.Data }, result);
        }
        
        return BadRequest(result.Error);
    }
}
```

### Exercise 3: Add Cross-Cutting Concerns

**Task:** Add logging, validation, and error handling to an existing API.

**Solution:**

```csharp
// Step 1: Add Logging Behavior
public class LoggingBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;
    
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        _logger.LogInformation(
            "Handling {RequestName} with data: {@RequestData}",
            typeof(TRequest).Name,
            request);
        
        var response = await next();
        
        _logger.LogInformation(
            "Handled {RequestName} with response: {@Response}",
            typeof(TRequest).Name,
            response);
        
        return response;
    }
}

// Step 2: Add Validation Behavior (already shown in examples)

// Step 3: Add Global Exception Handler
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;
    
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext, 
        Exception exception, 
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "An error occurred: {Message}", exception.Message);
        
        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "An error occurred",
            Detail = exception.Message,
            Instance = httpContext.Request.Path
        };
        
        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);
        
        return true;
    }
}

// Step 4: Register in Program.cs
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
```

---

## Test Your Understanding

### Questions

1. **What is the main difference between tutorial code and production code?**
   
   **Answer:** Tutorial code focuses on teaching syntax and basic features, while production code addresses real-world concerns like error handling, logging, validation, security, and maintainability.

2. **Why should dependencies point inward in Clean Architecture?**
   
   **Answer:** This ensures that the domain layer (the most stable and important part) has no dependencies on outer layers. Infrastructure, UI, and frameworks can change without affecting business logic.

3. **What is the purpose of the Specification pattern?**
   
   **Answer:** The Specification pattern encapsulates business rules that can be combined and reused. It provides a way to query objects using business criteria rather than technical implementation details.

4. **When should you NOT use Clean Architecture?**
   
   **Answer:** For small CRUD applications (< 10 entities), short-term projects, rapid prototyping, or when the team doesn't understand the pattern. The overhead isn't justified.

5. **What's the difference between Clean Architecture and Onion Architecture?**
   
   **Answer:** They're very similar! Both emphasize dependency inversion and domain independence. The main difference is terminology and emphasis: Clean Architecture focuses on use cases, while Onion Architecture emphasizes the domain at the center.

6. **Why is an anemic domain model an anti-pattern?**
   
   **Answer:** Because business logic is scattered across services instead of being encapsulated in entities. This leads to duplication, inconsistency, and makes it hard to enforce business rules.

7. **What is the benefit of using MediatR and CQRS?**
   
   **Answer:** MediatR decouples controllers from business logic, making code more testable and maintainable. CQRS separates read and write operations, allowing each to be optimized independently.

8. **How does multitenancy work in the fullstackhero starter kit?**
   
   **Answer:** It uses a tenant provider to resolve the current tenant, applies global query filters to automatically filter data by tenant, and sets the tenant ID on new entities automatically.

9. **What are domain events and when should you use them?**
   
   **Answer:** Domain events represent something important that happened in the domain. Use them to trigger side effects (sending emails, updating caches) without coupling the domain to infrastructure.

10. **Why is studying production codebases important?**
    
    **Answer:** Production codebases show how experienced developers make architectural decisions, handle real-world complexity, and balance trade-offs. They reveal patterns and practices that tutorials often skip.

---

## Common Interview Questions

### Architecture Questions

1. **Explain Clean Architecture and its benefits.**

   **Answer:** Clean Architecture is a software design pattern that separates concerns into layers with dependencies pointing inward. The domain layer is at the center with no external dependencies. Benefits include testability, maintainability, framework independence, and clear separation of concerns.

2. **What is the Dependency Inversion Principle and why is it important?**

   **Answer:** DIP states that high-level modules should not depend on low-level modules; both should depend on abstractions. This enables swapping implementations (e.g., changing databases) without changing business logic.

3. **Compare Clean Architecture, Onion Architecture, and Hexagonal Architecture.**

   **Answer:** All three share the dependency inversion principle. Clean Architecture emphasizes use cases, Onion Architecture focuses on the domain at the center, and Hexagonal Architecture (Ports and Adapters) emphasizes application boundaries. Choose based on team preference and project needs.

4. **What is CQRS and when should you use it?**

   **Answer:** CQRS separates read and write operations into different models. Use it when read and write workloads have different requirements, need independent scaling, or when complexity justifies the separation.

5. **Explain the difference between a Value Object and an Entity.**

   **Answer:** Entities have identity and can change over time (e.g., Order with OrderId). Value Objects are immutable and defined by their properties (e.g., Money with Amount and Currency). Two Money objects with the same values are considered equal.

### Pattern Questions

6. **What is the Repository pattern and why use it?**

   **Answer:** The Repository pattern abstracts data access logic behind an interface. It enables testing with fake implementations, centralizes data access logic, and decouples the application from specific ORMs or databases.

7. **Explain the Specification pattern with an example.**

   **Answer:** The Specification pattern encapsulates business rules that can be combined. Example: `ActiveProductsInCategorySpecification` combines multiple criteria (is active, in category, in stock) into a reusable query.

8. **What are Domain Events and how do they work?**

   **Answer:** Domain Events represent something important that happened in the domain. Entities raise events, and handlers react to them. This enables loose coupling between domain logic and side effects like sending emails or updating caches.

9. **Explain the Mediator pattern and its benefits.**

   **Answer:** The Mediator pattern centralizes communication between components. In .NET, MediatR implements this for CQRS. Benefits include decoupling, centralized pipeline behaviors (validation, logging), and easier testing.

10. **What is the Unit of Work pattern?**

    **Answer:** Unit of Work maintains a list of objects affected by a business transaction and coordinates writing changes and concurrency problems. In .NET, DbContext implements this pattern.

### Scenario Questions

11. **You need to build a simple blog API. Which architecture would you choose and why?**

    **Answer:** For a simple blog with < 10 entities, I'd use Vertical Slice Architecture or simple Layered Architecture. Clean Architecture would be overkill. Start simple and refactor if complexity grows.

12. **How would you handle validation in a Clean Architecture application?**

    **Answer:** Use FluentValidation validators in the Application layer for command/query validation. Add a ValidationBehavior in the MediatR pipeline. Also add domain validation in entities for business rules.

13. **Your team wants to migrate from a monolithic controller-service-repository structure to Clean Architecture. How would you approach this?**

    **Answer:** 
    1. Extract domain entities first
    2. Create repository interfaces
    3. Move business logic from services to domain entities
    4. Create command/query handlers
    5. Update controllers to use MediatR
    6. Do this feature-by-feature, not all at once

14. **How do you test a Clean Architecture application?**

    **Answer:** 
    - Unit test domain entities (no dependencies)
    - Unit test handlers (mock repositories)
    - Integration test repositories (real database)
    - E2E test critical user journeys

15. **When would you choose a modular monolith over microservices?**

    **Answer:** Choose modular monolith when you have a small team, need faster development, the domain doesn't have clear boundaries, or you're not ready for the operational complexity of microservices.

---

## Question Bank

### Beginner Questions (1-15)

1. **What is software architecture?**
   
   **Answer:** Software architecture is the high-level structure of a software system, defining how components interact, how data flows, and how concerns are separated. It's the foundation that makes systems maintainable, scalable, and testable.

2. **What is the difference between a controller and a service?**
   
   **Answer:** A controller handles HTTP requests and responses (presentation layer), while a service contains business logic (application/domain layer). Controllers should be thin, delegating to services or handlers.

3. **What is dependency injection?**
   
   **Answer:** Dependency injection is a technique where objects receive their dependencies from an external source rather than creating them internally. It enables loose coupling, easier testing, and better maintainability.

4. **What is an interface and why is it useful?**
   
   **Answer:** An interface defines a contract without implementation. It's useful for abstraction, enabling multiple implementations, easier testing with mocks, and following the Dependency Inversion Principle.

5. **What is the difference between an abstract class and an interface?**
   
   **Answer:** Abstract classes can have implementation and state; interfaces cannot (in C# before default interface methods). Use abstract classes for shared base functionality, interfaces for contracts.

6. **What is a design pattern?**
   
   **Answer:** A design pattern is a reusable solution to a common problem in software design. Examples include Repository, Factory, Strategy, and Observer patterns.

7. **What is the Single Responsibility Principle?**
   
   **Answer:** SRP states that a class should have only one reason to change. Each class should have a single, well-defined responsibility.

8. **What is coupling in software design?**
   
   **Answer:** Coupling is the degree of interdependence between modules. High coupling means changes in one module affect many others. Good architecture minimizes coupling.

9. **What is cohesion?**
   
   **Answer:** Cohesion is how closely related the responsibilities within a module are. High cohesion means a module does one thing well. Good architecture maximizes cohesion.

10. **What is the DRY principle?**
    
    **Answer:** Don't Repeat Yourself. Avoid duplicating code by extracting common functionality into reusable components.

11. **What is the difference between synchronous and asynchronous code?**
    
    **Answer:** Synchronous code blocks until completion. Asynchronous code allows other work to continue while waiting. In .NET, use `async/await` for I/O-bound operations.

12. **What is Entity Framework Core?**
    
    **Answer:** EF Core is an Object-Relational Mapper (ORM) for .NET. It maps database tables to C# classes and provides LINQ queries, change tracking, and database migrations.

13. **What is a DbContext?**
    
    **Answer:** DbContext is the main class in EF Core that coordinates database operations. It tracks entity changes, manages database connections, and provides DbSet properties for querying.

14. **What is the difference between GET and POST HTTP methods?**
    
    **Answer:** GET retrieves data (idempotent, safe, cacheable). POST creates new resources (not idempotent, not safe). GET parameters are in URL; POST data is in the body.

15. **What is REST?**
    
    **Answer:** REST (Representational State Transfer) is an architectural style for web services using HTTP methods, stateless communication, resource-based URLs, and standard status codes.

### Intermediate Questions (16-35)

16. **What is Clean Architecture?**
    
    **Answer:** Clean Architecture is a design pattern that separates concerns into concentric circles (layers) with dependencies pointing inward. The domain layer is at the center with no external dependencies, ensuring business logic is independent of frameworks and UI.

17. **What are the layers in Clean Architecture?**
    
    **Answer:** Domain (entities, value objects), Application (use cases, interfaces), Infrastructure (data access, external services), and Presentation (controllers, UI).

18. **What is the Dependency Rule in Clean Architecture?**
    
    **Answer:** Source code dependencies can only point inward. Inner circles know nothing about outer circles. This ensures the domain layer remains independent.

19. **What is CQRS?**
    
    **Answer:** Command Query Responsibility Segregation separates read operations (queries) from write operations (commands). This allows each to be optimized independently and can improve performance and scalability.

20. **What is MediatR and why use it?**
    
    **Answer:** MediatR is a library implementing the Mediator pattern in .NET. It decouples controllers from handlers, enables pipeline behaviors (validation, logging), and simplifies CQRS implementation.

21. **What is a Value Object?**
    
    **Answer:** A Value Object is an immutable object defined by its properties rather than identity. Two Value Objects with the same properties are considered equal. Examples: Money, Address, Email.

22. **What is an Aggregate in DDD?**
    
    **Answer:** An Aggregate is a cluster of domain objects (entities and value objects) treated as a single unit. It has a root entity (Aggregate Root) that controls access. Example: Order (root) with OrderItems.

23. **What is the Repository pattern?**
    
    **Answer:** The Repository pattern abstracts data access behind an interface, providing collection-like access to aggregates. It decouples the application from the ORM and enables testing with fake implementations.

24. **What is the Unit of Work pattern?**
    
    **Answer:** Unit of Work maintains a list of objects affected by a transaction and coordinates writing changes. In EF Core, DbContext implements this pattern.

25. **What are Domain Events?**
    
    **Answer:** Domain Events represent something important that happened in the domain. They enable loose coupling between domain logic and side effects (sending emails, updating caches).

26. **What is the Specification pattern?**
    
    **Answer:** The Specification pattern encapsulates business rules as reusable objects. It allows combining criteria and provides a way to query objects using business language rather than technical queries.

27. **What is FluentValidation?**
    
    **Answer:** FluentValidation is a library for building validation rules using a fluent interface. It's more powerful than DataAnnotations and supports complex validation scenarios.

28. **What is the difference between transient, scoped, and singleton services?**
    
    **Answer:** Transient: new instance each time. Scoped: one instance per request. Singleton: one instance for the application lifetime. Choose based on state and thread-safety requirements.

29. **What is middleware in ASP.NET Core?**
    
    **Answer:** Middleware are components that form the request processing pipeline. Each middleware can handle or pass the request to the next. Examples: authentication, logging, error handling.

30. **What is the difference between authentication and authorization?**
    
    **Answer:** Authentication verifies identity (who you are). Authorization checks permissions (what you can do). Authentication comes first, then authorization.

31. **What is JWT?**
    
    **Answer:** JSON Web Token is a compact, URL-safe token format for securely transmitting information. Used for stateless authentication in APIs.

32. **What is the difference between AddDbContext and AddDbContextFactory?**
    
    **Answer:** AddDbContext registers a scoped DbContext. AddDbContextFactory registers a factory for creating DbContext instances, useful for background services or when you need multiple contexts per request.

33. **What is the Result pattern?**
    
    **Answer:** The Result pattern explicitly represents success or failure without exceptions. It contains IsSuccess, Data (on success), and Error (on failure). Makes error handling explicit and type-safe.

34. **What is the difference between MapGet, MapPost, etc. and MapControllers?**
    
    **Answer:** MapGet/MapPost are for minimal APIs (lightweight, no controllers). MapControllers enables controller-based APIs with full MVC features. Choose based on complexity.

35. **What is the purpose of CancellationToken?**
    
    **Answer:** CancellationToken enables cooperative cancellation of async operations. It's passed through the call chain and checked periodically, allowing operations to be cancelled gracefully.

### Advanced Questions (36-50)

36. **Explain the Dependency Inversion Principle in detail.**
    
    **Answer:** DIP has two parts: (1) High-level modules should not depend on low-level modules; both should depend on abstractions. (2) Abstractions should not depend on details; details should depend on abstractions. This enables swapping implementations and makes systems more flexible.

37. **What is the difference between Vertical Slice Architecture and Clean Architecture?**
    
    **Answer:** Vertical Slice organizes code by feature (all code for a feature in one place). Clean Architecture organizes by layer (all controllers, then all handlers). Vertical Slice is simpler and more cohesive; Clean Architecture provides clearer separation for large systems.

38. **How do you handle distributed transactions in microservices?**
    
    **Answer:** Use the Saga pattern (choreography or orchestration), compensating transactions, eventual consistency, and message queues. Avoid distributed transactions (2PC) due to complexity and performance issues.

39. **What is the Outbox pattern and why is it important?**
    
    **Answer:** The Outbox pattern ensures reliable message delivery in distributed systems. Instead of sending messages directly, save them to an outbox table in the same transaction as business data. A separate process sends messages from the outbox, ensuring atomicity.

40. **Explain the difference between eventual consistency and strong consistency.**
    
    **Answer:** Strong consistency guarantees that all nodes see the same data at the same time. Eventual consistency guarantees that if no new updates are made, eventually all nodes will have the same data. Eventual consistency is used in distributed systems for better availability.

41. **What is the Circuit Breaker pattern?**
    
    **Answer:** Circuit Breaker prevents cascading failures in distributed systems. It monitors for failures and "trips" (stops sending requests) when failures exceed a threshold. After a timeout, it allows limited requests to test if the service has recovered.

42. **How do you implement multitenancy in .NET?**
    
    **Answer:** Use a tenant resolver (from subdomain, header, or JWT), store tenant ID in a scoped service, apply global query filters in EF Core, and ensure tenant isolation at the database level (separate databases or schemas).

43. **What is the difference between a Domain Service and an Application Service?**
    
    **Answer:** Domain Service contains domain logic that doesn't naturally belong to an entity or value object. Application Service orchestrates use cases, coordinates domain objects, and handles transactions. Domain Service is part of the domain; Application Service is part of the application layer.

44. **Explain the concept of Bounded Context in DDD.**
    
    **Answer:** A Bounded Context defines the boundaries within which a particular domain model is valid. Different contexts may have different models for the same concept (e.g., "Product" in Sales vs. Shipping contexts). This prevents model pollution and confusion.

45. **What is the Anti-Corruption Layer pattern?**
    
    **Answer:** An ACL is a layer that translates between two different models or systems, preventing the external model from corrupting the internal domain model. It's used when integrating with legacy systems or external APIs.

46. **How do you handle background jobs in .NET?**
    
    **Answer:** Use IHostedService for long-running background tasks, BackgroundService as a base class, Hangfire for persistent jobs with retry, or Quartz.NET for scheduled jobs. Consider reliability, retry logic, and monitoring.

47. **What is the difference between integration tests and unit tests?**
    
    **Answer:** Unit tests test individual components in isolation (mocked dependencies). Integration tests test how components work together (real or test database). Unit tests are fast and isolated; integration tests are slower but test real interactions.

48. **Explain the concept of Event Sourcing.**
    
    **Answer:** Event Sourcing stores all state changes as events rather than current state. The current state is derived by replaying events. Benefits: complete audit trail, temporal queries, event replay. Challenges: event versioning, snapshotting, learning curve.

49. **What is the CQRS pattern and when should you use it?**
    
    **Answer:** CQRS separates read and write models. Use it when read and write workloads differ significantly, need independent scaling, or when complexity is justified. Don't use it for simple CRUD applications.

50. **How do you ensure security in a layered architecture?**
    
    **Answer:** Implement defense in depth: validate input at every layer, use authentication/authorization at the API layer, enforce business rules in the domain, sanitize data before display, use HTTPS, implement CORS properly, and never trust client-side validation alone.

---

## Troubleshooting Guide

### Common Issues When Studying Codebases

#### Issue 1: Analysis Paralysis

**Symptoms:**
- Can't decide where to start
- Overwhelmed by file count
- Reading without understanding

**Solution:**
```
1. Start with README only
2. Run the application
3. Pick ONE feature
4. Trace ONE request completely
5. Ignore everything else initially
```

#### Issue 2: Can't Understand the Architecture

**Symptoms:**
- Don't see the pattern
- Files seem randomly organized
- Unclear why certain decisions were made

**Solution:**
```
1. Look for documentation
2. Check commit history for architectural changes
3. Draw the architecture yourself
4. Compare with known patterns
5. Ask: "What problem does this solve?"
```

#### Issue 3: Too Many Abstractions

**Symptoms:**
- 10+ interfaces for simple operations
- Can't trace the actual implementation
- Unnecessary complexity

**Solution:**
```
1. Identify the actual implementation
2. Ask: "Does this abstraction protect a boundary?"
3. If no, it's over-engineering
4. Focus on understanding the core flow
5. Note it as an example of what NOT to do
```

#### Issue 4: Can't Run the Application

**Symptoms:**
- Missing dependencies
- Database connection errors
- Configuration issues

**Solution:**
```
1. Read setup instructions carefully
2. Check .NET SDK version
3. Verify database connection strings
4. Check appsettings.json
5. Look for Docker Compose files
6. Check GitHub issues for known problems
```

#### Issue 5: Tests Don't Make Sense

**Symptoms:**
- Tests are too complex
- Can't understand what's being tested
- Too many mocking frameworks

**Solution:**
```
1. Start with integration tests (easier to understand)
2. Look at test fixtures and helpers
3. Focus on one test class
4. Run tests and see what they do
5. Read test names carefully
```

### Tools for Code Exploration

#### Essential Tools

```bash
# Visual Studio
- Architecture diagrams
- Code maps
- Dependency graphs
- Live unit testing

# Visual Studio Code
- C# extensions
- GitLens for history
- Bookmarks for navigation

# Command Line
# Find all interfaces
grep -r "interface I" --include="*.cs"

# Find all implementations
grep -r "class.*:.*I[A-Z]" --include="*.cs"

# Find all usages
grep -r "IProductRepository" --include="*.cs"

# Count lines of code
find . -name "*.cs" -exec wc -l {} + | tail -1
```

#### Visual Studio Features

1. **Code Map** - Visualize dependencies
2. **Architecture Diagram** - Generate from code
3. **Call Hierarchy** - See who calls what
4. **Find All References** - Track usage
5. **Navigate To** - Quick file/class search

---

## Performance Considerations

### How Architecture Affects Performance

```
┌─────────────────────────────────────────────────────────────┐
│              Architecture Performance Impact                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Factor                    │ Impact │ Mitigation            │
│  ──────────────────────────┼────────┼────────────────────── │
│  Abstraction layers        │ Medium │ Use judiciously      │
│  ──────────────────────────┼────────┼────────────────────── │
│  Dependency injection      │ Low    │ Register properly    │
│  ──────────────────────────┼────────┼────────────────────── │
│  CQRS overhead             │ Low    │ Only when beneficial │
│  ──────────────────────────┼────────┼────────────────────── │
│  Domain events             │ Low    │ Async processing     │
│  ──────────────────────────┼────────┼────────────────────── │
│  Repository pattern        │ Low    │ Use specifications   │
│  ──────────────────────────┼────────┼────────────────────── │
│  Validation pipeline       │ Low    │ Cache when possible  │
│  ──────────────────────────┼────────┼────────────────────── │
│  Too many layers           │ High   │ Simplify             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Performance Best Practices

#### 1. Database Queries

```csharp
// ❌ BAD - N+1 query problem
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders)
{
    var customer = await _context.Customers.FindAsync(order.CustomerId);
    // N+1 queries!
}

// ✅ GOOD - Eager loading
var orders = await _context.Orders
    .Include(o => o.Customer)
    .ToListAsync();

// ✅ GOOD - Projection
var orders = await _context.Orders
    .Select(o => new OrderDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name
    })
    .ToListAsync();
```

#### 2. Caching Strategy

```csharp
// Cache frequently accessed data
public class GetProductQueryHandler 
    : IRequestHandler<GetProductQuery, Result<ProductDto>>
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;
    
    public async Task<Result<ProductDto>> Handle(
        GetProductQuery request, 
        CancellationToken ct)
    {
        // Try cache first
        if (_cache.TryGetValue(request.Id, out ProductDto cached))
        {
            return Result<ProductDto>.Success(cached);
        }
        
        // Cache miss - fetch from database
        var product = await _repository.GetByIdAsync(request.Id, ct);
        
        if (product == null)
            return Result<ProductDto>.Failure("Product not found");
        
        var dto = _mapper.Map<ProductDto>(product);
        
        // Cache for 5 minutes
        _cache.Set(request.Id, dto, TimeSpan.FromMinutes(5));
        
        return Result<ProductDto>.Success(dto);
    }
}
```

#### 3. Async/Await Best Practices

```csharp
// ✅ GOOD - Async all the way
public async Task<Result<List<Product>>> GetAllAsync(CancellationToken ct)
{
    return await _context.Products.ToListAsync(ct);
}

// ❌ BAD - Sync over async
public List<Product> GetAll()
{
    return _context.Products.ToListAsync().Result; // Deadlock risk!
}

// ❌ BAD - Fire and forget without tracking
public async Task ProcessData()
{
    _ = Task.Run(() => DoWork()); // Can't track or handle errors
}

// ✅ GOOD - Proper background processing
public async Task ProcessData(CancellationToken ct)
{
    await Task.Run(() => DoWork(), ct);
}
```

#### 4. Connection Pooling

```csharp
// Configure connection pooling in connection string
"ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;Pooling=true;Minimum Pool Size=5;Maximum Pool Size=100;"
}

// Benefits:
// - Reuses connections
// - Reduces connection overhead
// - Improves performance under load
```

### Performance Monitoring

```csharp
// Add performance logging
public class PerformanceBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly ILogger<PerformanceBehavior<TRequest, TResponse>> _logger;
    private readonly Stopwatch _timer;
    
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        _timer.Start();
        
        var response = await next();
        
        _timer.Stop();
        
        var elapsedMs = _timer.ElapsedMilliseconds;
        
        if (elapsedMs > 500) // Log slow requests
        {
            _logger.LogWarning(
                "Long running request: {Name} ({ElapsedMilliseconds}ms) {@Request}",
                typeof(TRequest).Name, 
                elapsedMs, 
                request);
        }
        
        return response;
    }
}
```

---

## Security Considerations

### Security in Layered Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              Defense in Depth                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer              Security Controls                       │
│  ───────────────────┼────────────────────────────────────── │
│  Presentation       │ Input validation, sanitization        │
│  ───────────────────┼────────────────────────────────────── │
│  Application        │ Authorization, business rule checks   │
│  ───────────────────┼────────────────────────────────────── │
│  Domain             │ Business rule enforcement             │
│  ───────────────────┼────────────────────────────────────── │
│  Infrastructure     │ Data encryption, secure storage       │
│                                                             │
│  Each layer enforces security!                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Essential Security Practices

#### 1. Authentication and Authorization

```csharp
// JWT Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

// Policy-based authorization
[Authorize(Policy = "RequireAdminRole")]
public IActionResult AdminOnly() { }

[Authorize(Policy = "CanEditProducts")]
public IActionResult EditProduct() { }
```

#### 2. Input Validation

```csharp
// Always validate input
public class CreateProductCommandValidator 
    : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .MaximumLength(100)
            .Matches("^[a-zA-Z0-9 ]+$") // Alphanumeric only
            .WithMessage("Product name contains invalid characters");
        
        RuleFor(x => x.Price)
            .GreaterThan(0)
            .LessThan(1000000)
            .WithMessage("Price out of valid range");
        
        RuleFor(x => x.Description)
            .MaximumLength(1000)
            .When(x => !string.IsNullOrEmpty(x.Description));
    }
}

// Sanitize HTML input
public class SanitizeHtmlAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext context)
    {
        var html = value as string;
        if (!string.IsNullOrEmpty(html))
        {
            var sanitized = HtmlSanitizer.Sanitize(html);
            // Set sanitized value
        }
        
        return ValidationResult.Success;
    }
}
```

#### 3. SQL Injection Prevention

```csharp
// ❌ VULNERABLE - SQL Injection
var sql = $"SELECT * FROM Products WHERE Name = '{productName}'";
var products = await _context.Products.FromSqlRaw(sql).ToListAsync();

// ✅ SAFE - Parameterized query
var products = await _context.Products
    .FromSqlInterpolated($"SELECT * FROM Products WHERE Name = {productName}")
    .ToListAsync();

// ✅ SAFE - LINQ (automatically parameterized)
var products = await _context.Products
    .Where(p => p.Name == productName)
    .ToListAsync();
```

#### 4. Sensitive Data Protection

```csharp
// Don't log sensitive data
public class LoggingBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request, 
        RequestHandlerDelegate<TResponse> next, 
        CancellationToken cancellationToken)
    {
        // Sanitize request before logging
        var sanitizedRequest = SanitizeSensitiveData(request);
        
        _logger.LogInformation("Handling {RequestName}", typeof(TRequest).Name);
        
        var response = await next();
        
        return response;
    }
    
    private object SanitizeSensitiveData(object request)
    {
        // Remove passwords, credit cards, etc.
        return request;
    }
}

// Use Data Protection API for encryption
public class CreditCardService
{
    private readonly IDataProtector _protector;
    
    public string EncryptCardNumber(string cardNumber)
    {
        return _protector.Protect(cardNumber);
    }
    
    public string DecryptCardNumber(string encryptedCardNumber)
    {
        return _protector.Unprotect(encryptedCardNumber);
    }
}
```

#### 5. Security Headers

```csharp
// Add security headers
app.UseHsts();
app.UseHttpsRedirection();

app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
    context.Response.Headers.Add("Content-Security-Policy", 
        "default-src 'self'; script-src 'self' 'unsafe-inline'");
    
    await next();
});
```

### Security Checklist

- [ ] Use HTTPS everywhere
- [ ] Implement authentication (JWT, OAuth, etc.)
- [ ] Implement authorization (policies, roles)
- [ ] Validate all input
- [ ] Sanitize output
- [ ] Use parameterized queries
- [ ] Hash passwords (bcrypt, Argon2)
- [ ] Encrypt sensitive data
- [ ] Add security headers
- [ ] Implement CORS properly
- [ ] Rate limit API endpoints
- [ ] Log security events
- [ ] Keep dependencies updated
- [ ] Use secrets management (not appsettings.json)
- [ ] Implement audit logging

---

## Summary and Key Takeaways

### 10 Key Insights

1. **Production code teaches decisions, not just syntax.** Tutorials show you how; production code shows you why.

2. **Architecture is about trade-offs, not perfection.** There's no "one true architecture." Choose based on your context.

3. **The dependency rule is non-negotiable.** Dependencies must point inward to keep the domain independent.

4. **Start simple, add complexity when needed.** Don't use Clean Architecture for a 3-entity CRUD app.

5. **Domain logic belongs in the domain layer.** Anemic domain models are an anti-pattern.

6. **Study repositories actively, not passively.** Trace requests, ask questions, make small changes.

7. **Good defaults prevent forgotten work.** Security headers, health checks, and logging should be standard.

8. **Abstractions should protect boundaries.** Don't create interfaces with one implementation.

9. **Microservices are not always the answer.** A well-structured monolith is often better.

10. **Understanding the "why" is more important than copying the "what".**

### Action Items

**This Week:**
- [ ] Clone one of the 10 repositories
- [ ] Run the application
- [ ] Trace one complete request
- [ ] Document the architecture

**This Month:**
- [ ] Study 2-3 repositories in depth
- [ ] Implement one pattern in your project
- [ ] Write tests for your domain logic
- [ ] Add cross-cutting concerns (logging, validation)

**This Quarter:**
- [ ] Refactor one project using learned patterns
- [ ] Create your own starter template
- [ ] Present architecture to your team
- [ ] Contribute to an open-source project

### Next Steps

1. **Choose a repository** that matches your current project
2. **Study it actively** using the methodology outlined
3. **Apply one pattern** to your work
4. **Share your learnings** with your team
5. **Iterate and improve** your architecture continuously

---

## Further Reading and Resources

### Books

1. **"Clean Architecture" by Robert C. Martin** - The definitive guide to clean architecture principles
2. **"Domain-Driven Design" by Eric Evans** - The foundational DDD book
3. **"Implementing Domain-Driven Design" by Vaughn Vernon** - Practical DDD implementation
4. **"Patterns of Enterprise Application Architecture" by Martin Fowler** - Classic patterns catalog
5. **"The Art of Unit Testing" by Roy Osherove** - Testing strategies and practices

### Official Documentation

- [Microsoft .NET Architecture Guides](https://learn.microsoft.com/en-us/dotnet/architecture/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [MediatR Documentation](https://github.com/jasontaylordev/CleanArchitecture)

### Online Courses

- [Clean Architecture by Jason Taylor](https://www.youtube.com/c/JasonTaylorDev)
- [Ardalis Courses on Pluralsight](https://www.pluralsight.com/authors/steve-smith)
- [Domain-Driven Design Fundamentals on Pluralsight](https://www.pluralsight.com/courses/domain-driven-design-fundamentals)

### Community Resources

- [.NET Foundation](https://dotnetfoundation.org/)
- [r/dotnet on Reddit](https://reddit.com/r/dotnet)
- [.NET Discord](https://discord.gg/dotnet)
- [C# Discord](https://discord.gg/csharp)

### Blogs and Articles

- [Steve Smith (Ardalis) Blog](https://ardalis.com/)
- [Jason Taylor's Blog](https://jasontaylor.dev/)
- [Milan Jovanović's Blog](https://milanjovanovic.tech/)
- [Andrew Lock's Blog](https://andrewlock.net/)

### Tools

- [Visual Studio](https://visualstudio.microsoft.com/) - IDE with architecture tools
- [ReSharper](https://www.jetbrains.com/resharper/) - Code analysis and refactoring
- [NDepend](https://www.ndepend.com/) - Code quality and architecture analysis
- [ArchUnitNET](https://github.com/TNG/ArchUnitNET) - Architecture testing

---

## Self-Assessment Checklist

Use this checklist to evaluate your understanding:

### Knowledge Assessment

- [ ] I can explain the difference between Clean, Onion, and Hexagonal Architecture
- [ ] I understand the dependency rule and can apply it
- [ ] I know when to use (and when NOT to use) Clean Architecture
- [ ] I can explain CQRS and when it's appropriate
- [ ] I understand the difference between entities, value objects, and aggregates
- [ ] I can explain domain events and their benefits
- [ ] I know how to implement multitenancy
- [ ] I understand the Specification pattern
- [ ] I can explain the Result pattern
- [ ] I know how to structure a modular monolith

### Practical Skills

- [ ] I can clone and run a production codebase
- [ ] I can trace a request from controller to database
- [ ] I can identify architectural patterns in existing code
- [ ] I can refactor a simple API to use CQRS
- [ ] I can implement proper validation
- [ ] I can write unit tests for domain logic
- [ ] I can write integration tests for repositories
- [ ] I can configure dependency injection properly
- [ ] I can add cross-cutting concerns (logging, validation)
- [ ] I can explain my architecture decisions to others

### Decision-Making

- [ ] I can choose the right architecture for a project
- [ ] I know when to introduce abstractions
- [ ] I can identify over-engineering
- [ ] I understand the trade-offs of different patterns
- [ ] I can plan a migration from simple to complex architecture
- [ ] I know when to use microservices vs. monolith
- [ ] I can evaluate starter kits critically
- [ ] I understand the costs of architectural decisions

### Next Steps Based on Score

**0-40%:** Review the fundamentals, study simpler codebases first
**41-70%:** Practice implementing patterns in small projects
**71-100%:** Start applying to real projects, mentor others

---

## Conclusion

Studying production-ready .NET codebases is one of the fastest ways to level up as a developer. These 10 repositories represent thousands of hours of combined experience, battle-tested patterns, and real-world problem solving.

**Remember:**
- 🎯 Focus on understanding the "why" behind decisions
- 🎯 Start with one repository and study it deeply
- 🎯 Apply patterns incrementally to your projects
- 🎯 Share your learnings with your team
- 🎯 Continuously refine your architecture skills

**The goal is not to copy code.** The goal is to understand the principles, recognize the trade-offs, and make informed decisions for your specific context.

Now go forth and build better software! 🚀

---

**Last Updated:** July 23, 2026  
**Author:** Comprehensive Study Guide based on Muhammad Waseem's article  
**License:** Educational use  
**Feedback:** This tutorial was created following comprehensive tutorial preferences with deep-dive content, practical exercises, and extensive question banks.

---

## Appendix: Quick Reference

### Architecture Decision Matrix

| Scenario | Recommended Pattern | Complexity | Learning Curve |
|----------|-------------------|------------|----------------|
| Simple CRUD (< 10 entities) | Vertical Slice | Low | Easy |
| Medium business app (10-50 entities) | Clean/Onion | Medium | Moderate |
| Large enterprise (50+ entities) | Clean + DDD | High | Steep |
| Distributed system | Microservices | Very High | Steep |
| Rapid prototype | Layered | Low | Easy |
| Long-lived application | Clean Architecture | Medium-High | Moderate |

### Common Abbreviations

- **CQRS** - Command Query Responsibility Segregation
- **DDD** - Domain-Driven Design
- **EF Core** - Entity Framework Core
- **ORM** - Object-Relational Mapper
- **SOLID** - Single responsibility, Open-closed, Liskov substitution, Interface segregation, Dependency inversion
- **CRUD** - Create, Read, Update, Delete
- **DTO** - Data Transfer Object
- **DI** - Dependency Injection
- **ACID** - Atomicity, Consistency, Isolation, Durability

### Essential NuGet Packages

```xml
<!-- MediatR for CQRS -->
<PackageReference Include="MediatR" Version="12.0.0" />

<!-- FluentValidation -->
<PackageReference Include="FluentValidation.DependencyInjectionExtensions" Version="11.0.0" />

<!-- Entity Framework Core -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />

<!-- AutoMapper -->
<PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.0" />

<!-- Serilog for logging -->
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
<PackageReference Include="Serilog.Sinks.Seq" Version="8.0.0" />

<!-- Health checks -->
<PackageReference Include="Microsoft.AspNetCore.Diagnostics.HealthChecks" Version="8.0.0" />
<PackageReference Include="AspNetCore.HealthChecks.UI" Version="8.0.0" />

<!-- OpenAPI/Swagger -->
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />
```

---

**End of Tutorial**

*This comprehensive guide has covered 10 production-ready .NET codebases, architectural patterns, practical implementation strategies, and extensive learning resources. Use this as a reference throughout your .NET architecture journey.*
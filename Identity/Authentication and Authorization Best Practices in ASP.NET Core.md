# Authentication and Authorization Best Practices in ASP.NET Core

Authentication and authorization are two pillars of application security. **Authentication** verifies the identity of a user, while **Authorization** determines what that authenticated user is allowed to do.

Understanding and applying proven tools is critical to prevent common vulnerabilities such as unauthorized access and data leaks.

We will walk you through the best practices for implementing both authentication and authorization in ASP.NET Core. We will cover:

-   JWT-based Authentication
-   Role-Based Authorization (RBAC)
-   Claim-Based Authorization
-   Attribute-Based Authorization (ABAC)

Let's dive in!

[](#implementing-authentication-with-jwt-tokens)

## Implementing Authentication with JWT Tokens

One of the most popular and secure ways to implement authentication is by using JSON Web Tokens (JWT). JWT enables stateless authentication and simplifies scaling.

Let's explore how you can configure authentication in ASP.NET Core.

First, add the following configuration to your `appsettings.json`:

```json
{   
    "AuthConfiguration": {     
        "Key": "your_secret_key_here_change_it_please",     
        "Issuer": "DevTips",     
        "Audience": "DevTips"   
    } 
}
```

Next, configure JWT token validation in your Program.cs using `AuthConfiguration`:

```csharp
builder.Services
    .AddAuthentication(options => {  
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme; 
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme; 
        }) 
        .AddJwtBearer(options => { 
            options.TokenValidationParameters = new TokenValidationParameters
                {         
                    ValidateIssuer = true,         
                    ValidateAudience = true,         
                    ValidateLifetime = true,         
                    ValidateIssuerSigningKey = true,         
                    ValidIssuer = builder.Configuration["AuthConfiguration:Issuer"],         
                    ValidAudience = builder.Configuration["AuthConfiguration:Audience"],         
                    IssuerSigningKey = new SymmetricSecurityKey( Encoding.UTF8.GetBytes(builder.Configuration["AuthConfiguration:Key"]!))     
                }; 
            }
    ); 
    builder.Services.AddAuthorization();
```

Finally, register the authentication and authorization middleware:

```csharp
var app = builder.Build(); 
if (app.Environment.IsDevelopment()) 
{     
    app.UseSwagger();     
    app.UseSwaggerUI(); 
} 
app.UseAuthentication(); 
app.UseAuthorization();
```

Consider these token security best practices:

-   **Expiration:** Set a short lifespan for JWT tokens and use refresh tokens for continuous authentication.
-   **Signature:** Use a strong secret key with a robust algorithm (typically HMAC-SHA256 or HMAC-SHA512).
-   **Validation:** Strictly enforce token validation parameters like issuer, audience, lifetime, and signing key.

[](#authentication-users-with-jwt-tokens-and-aspnet-core-identity)

## Authentication Users with JWT tokens and ASP.NET Core Identity

When handling user credentials, avoid storing passwords in plain text. Instead, use a robust hashing algorithm (e.g., bcrypt, or ASP.NET Core Identity's built-in password hasher) to securely store password hashes.

Let's explore the endpoint that performs user login with ASP.NET Core Identity:

```csharp
public void AddRoutes(IEndpointRouteBuilder app) 
{     
    app.MapPost("/api/users/login", Handle); 
} 

private static async Task<IResult> Handle( [FromBody] LoginUserRequest request, IOptions<AuthConfiguration> authOptions, UserManager<User> userManager, SignInManager<User> signInManager,  CancellationToken cancellationToken) 
{     
    var user = await userManager.FindByEmailAsync(request.Email);     
    if (user is null)     
    {         
        return Results.NotFound("User not found");     
    }     
    var result = await signInManager.CheckPasswordSignInAsync(user, request.Password, false);     
    if (!result.Succeeded)     
    {         
        return Results.Unauthorized();     
    }     
    var token = GenerateJwtToken(user, authOptions.Value);     
    return Results.Ok(new { Token = token }); 
}
```

And the helper method to generate the JWT token:

```csharp
private static string GenerateJwtToken(User user, AuthConfiguration authConfiguration) 
{     
    var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(authConfiguration.Key));     
    var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);     
    var claims = new[] { new Claim(JwtRegisteredClaimNames.Sub, user.Email!), new Claim("userid", user.Id) };     
    var token = new JwtSecurityToken( issuer: authConfiguration.Issuer, audience: authConfiguration.Audience, claims: claims, expires: DateTime.Now.AddMinutes(30),         signingCredentials: credentials );     
    return new JwtSecurityTokenHandler().WriteToken(token); 
}
```

This token includes and `email` and `userid` claims and expires in 30 minutes. For longer sessions, consider implementing refresh tokens.

You can decode the created JWT token using [JWT IO web site](https://jwt.io/) to see what's inside.

For Minimal APIs, you can secure endpoints with `RequireAuthorization`:

```csharp
app.MapPost("/api/books", Handle).RequireAuthorization();
```

For controllers, simply decorate classes or methods with the `[Authorize]` attribute.

Role-Based Authorization (RBAC) restricts access based on the user roles.

RBAC simplifies permission management by grouping users into roles and defining access rights for these roles. This not only makes your application more secure but also simplifies authorization implementation.

For example, consider two roles: **Admin** and **Author**:

```csharp
var adminRole = new Role { Name = "Admin" }; 
var authorRole = new Role { Name = "Author" }; 
await roleManager.CreateAsync(adminRole); 
await roleManager.CreateAsync(authorRole);
```

Admins and Authors can create, update, and delete books, while regular users can only view them. Only Admins can manage users.

Here is how you can regiser Role-Based policies in the `AddAuthorization` method:

```csharp
builder.Services.AddAuthorization(options => {
        options.AddPolicy("Admin", policy => { policy.RequireRole("Admin");});     
        options.AddPolicy("Author", policy => { policy.RequireRole("Author");});     
        options.AddPolicy("BookEditor", policy => {         
                // Allow both Admin and Author roles to edit books         
                policy.RequireRole("Admin", "Author"); 
            }); 
    });
```
Here we define 3 roles: Author, Admin and BookEditor - that allows both Admin and Author roles to edit books.

When you define your Minimal API endpoint, you can specify the PolicyName in the `RequireAuthorization` method:

```csharp
public void AddRoutes(IEndpointRouteBuilder app) 
{     
    app.MapPost("/api/books", Handle).RequireAuthorization("BookEditor");
    app.MapPost("/api/users", Handle).RequireAuthorization("Admin");
}
```

What if you have more roles, and you need to have multiple combinations of roles to define access to various endpoints? As your application grows, managing multiple role combinations might become cumbersome.

[](#implementing-claimsbased-authorization)

## Implementing Claims-Based Authorization

A more flexible approach is to use claims-based authorization.

Instead of hardcoding roles, you can require specific claims for each endpoint. This method allows for fine-grained control and easier updates to user permissions.

Let's explore an example:

```csharp
var adminRole = new Role { Name = "Admin" }; 
var authorRole = new Role { Name = "Author" }; 
await roleManager.CreateAsync(adminRole); 
await roleManager.CreateAsync(authorRole); 
await roleManager.AddClaimAsync(adminRole, new Claim("users:create", "true")); 
await roleManager.AddClaimAsync(adminRole, new Claim("users:update", "true")); 
await roleManager.AddClaimAsync(adminRole, new Claim("users:delete", "true")); 
await roleManager.AddClaimAsync(adminRole, new Claim("books:create", "true")); 
await roleManager.AddClaimAsync(adminRole, new Claim("books:update", "true")); 
await roleManager.AddClaimAsync(adminRole, new Claim("books:delete", "true")); 
await roleManager.AddClaimAsync(authorRole, new Claim("books:create", "true")); 
await roleManager.AddClaimAsync(authorRole, new Claim("books:update", "true")); 
await roleManager.AddClaimAsync(authorRole, new Claim("books:delete", "true"));
```

Here I assign claims to each of the roles:

-   users: create, update, delete
-   books: create, update, delete

```csharp
public void AddRoutes(IEndpointRouteBuilder app) 
{     
    app.MapPost("/api/books", Handle).RequireAuthorization("books:create");              
    app.MapDelete("/api/books/{id}", Handle).RequireAuthorization("books:delete");              
    app.MapPost("/api/users", Handle).RequireAuthorization("users:create");              
    app.MapDelete("/api/users/{id}", Handle).RequireAuthorization("users:delete");    
}
```

For simpler management, you can combine multiple claims into a role and dynamically update set of claims for each role.

When issuing the JWT token, add the user's claims along with their basic information:

```csharp
var roleClaims = await userManager.GetClaimsAsync(user); 
List<Claim> claims = [ new(JwtRegisteredClaimNames.Sub, user.Email!), new("userid", user.Id), new("role", userRole) ]; 
foreach (var roleClaim in roleClaims) 
{     
    claims.Add(new Claim(roleClaim.Type, roleClaim.Value)); 
} 
var token = new JwtSecurityToken( issuer: authConfiguration.Issuer, audience: authConfiguration.Audience, claims: claims, expires: DateTime.Now.AddMinutes(30), signingCredentials: credentials );
```

When we log in, here is what the decoded JWT token looks like:

```json
{   
    "sub": "[[email protected]](/cdn-cgi/l/email-protection)",   
    "userid": "dc233fac-bace-4719-9a4f-853e199300d5",   
    "role": "Admin",   
    "users:create": "true",   
    "users:update": "true",   
    "users:delete": "true",   
    "books:create": "true",   
    "books:update": "true",   
    "books:delete": "true",   
    "exp": 1739481834,   
    "iss": "DevTips",   
    "aud": "DevTips" 
}
```

Here is how you can register all these claims:

```csharp
builder.Services.AddAuthorization(options => {     
    options.AddPolicy("books:create", policy => policy.RequireClaim("books:create"));     
    options.AddPolicy("books:update", policy => policy.RequireClaim("books:update"));     
    options.AddPolicy("books:delete", policy => policy.RequireClaim("books:delete"));     
    options.AddPolicy("users:create", policy => policy.RequireClaim("users:create", "true"));     
    options.AddPolicy("users:update", policy => policy.RequireClaim("users:update"));     
    options.AddPolicy("users:delete", policy => policy.RequireClaim("users:delete")); 
});
```

Notice, that we use `RequireClaim` instead of `RequireRole` for Claims-Based Authorization.

[](#implementing-attributebased-authorization)

## Implementing Attribute-Based Authorization

While RBAC and Claims-Based approaches work well, there are scenarios where access decisions must consider the attributes of both the user and the resource.

In the previous example, we allowed Author to update any books.

More practical is to allow an author to update and delete only his own books. Or allow a region manager to manage books: allowing to update and delete books based on the region.

Let's explore an example of how to allow an author to update only his own books.

You need to define a custom `IAuthorizationRequirement`:

```csharp
public class BookAuthorRequirement : IAuthorizationRequirement { }
```

And implement the `AuthorizationHandler` that checks whether the current user is the author of the book:

```csharp
public class BookAuthorHandler : AuthorizationHandler<BookAuthorRequirement, Author> 
{     
    protected override Task HandleRequirementAsync( AuthorizationHandlerContext context,  BookAuthorRequirement requirement, Author resource) 
    {         
        var userId = context.User.FindFirst("userid")?.Value;         
        if (userId is not null && userId == resource.UserId)         
        {             
            context.Succeed(requirement);         
        }         
        return Task.CompletedTask;     
    } 
}
```

Register the authorization handler in the DI container:

```csharp
builder.Services.AddScoped<IAuthorizationHandler, BookAuthorHandler>();
```

In the API endpoint you can call the `IAuthorizationService` and check the `BookAuthorRequirement`:

```csharp 
app.MapPut("/api/books/{id}", Handle).RequireAuthorization("books:update");
```

```csharp
private static async Task<IResult> Handle( [FromRoute] Guid id, [FromBody] UpdateBookRequest request, IBookRepository repository, IAuthorizationService authService, ClaimsPrincipal user,  CancellationToken cancellationToken) 
{     
    var book = await repository.GetByIdAsync(id, cancellationToken);     
    if (book is null)     
    {         
        return Results.NotFound($"Book with id {id} not found");     
    }     
    var requirement = new BookAuthorRequirement();     
    var authResult = await authService.AuthorizeAsync(user, book.Author, requirement);     
    if (!authResult.Succeeded)     
    {         
        return Results.Forbid();     
    }          
    book.Title = request.Title;     
    book.Year = request.Year;          
    await repository.UpdateAsync(book, cancellationToken);     
    return Results.NoContent(); 
}
```

In this example, only the author of the book is allowed to update it. We determined this by comparing the user's "userid" claim with the book's author ID column.

[](#summary)

## Summary

We explored how to implement JWT-based authentication and 3 types of Authorization:

-   Role-Based
-   Claims-Based
-   Attribute-Based

When to use each type of Authorization?

-   **Role-Based Authorization:** when you need to control access based on predefined roles, such as Admin or Editor, for simplicity and centralized permission management.
-   **Claims-Based Authorization:** when finer-grained control is required, allowing access based on specific properties or permissions assigned to a user, such as "books:create" or "users:delete". You can group claims into roles and dynamically update set of permissions for each role.
-   **Attribute-Based Authorization (ABAC):** when access decisions depend on dynamic attributes of the user, resource, or environment, such as allowing users to edit only their own data or managing context-based permissions.

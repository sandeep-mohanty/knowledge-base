# Configurable Sliding Window Rate Limiting for SCIM APIs

This guide builds upon the previous implementation by making the **SlidingWindowRateLimiterOptions** configurable. This allows you to adjust rate-limiting parameters without modifying the code, using configuration files (e.g., `appsettings.json`).

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1: Update `appsettings.json`](#step-1-update-appsettingsjson)
3. [Step 2: Create a Configuration Model](#step-2-create-a-configuration-model)
4. [Step 3: Bind Configuration in `Program.cs`](#step-3-bind-configuration-in-programcs)
5. [Step 4: Use Configurable Options in Rate Limiting](#step-4-use-configurable-options-in-rate-limiting)
6. [Step 5: Test the Implementation](#step-5-test-the-implementation)

---

## Prerequisites

- .NET 9 SDK installed on your machine.
- A working SCIM API project with the `Microsoft.AspNetCore.RateLimiting` package installed.

---

## Step 1: Update `appsettings.json`

Add a section in your `appsettings.json` file to define the rate-limiting options. This makes it easy to modify the settings without changing the code.

```json
{
  "RateLimiting": {
    "PermitLimit": 10,
    "WindowInSeconds": 60,
    "QueueProcessingOrder": "OldestFirst",
    "QueueLimit": 2
  }
}
```  
---  

## Step 2: Create a Configuration Model  
Create a class to represent the rate-limiting configuration. This class will be used to bind the settings from `appsettings.json`.  
```csharp
public class RateLimitingOptions
{
    public int PermitLimit { get; set; }
    public int WindowInSeconds { get; set; }
    public string QueueProcessingOrder { get; set; } = "OldestFirst";
    public int QueueLimit { get; set; }
}
```  
Create a class to represent SCIM error response:  

```csharp  
public class ScimErrorResponse
{
    public List<string> Schemas { get; set; } = new() { "urn:ietf:params:scim:api:messages:2.0:Error" };
    public string Detail { get; set; }
    public string Status { get; set; }
}
```
---
## Step 3: Bind Configuration in `Program.cs`  

In `Program.cs`, bind the configuration from `appsettings.json` to the `RateLimitingOptions` class and use it to configure the rate limiter.  

```csharp
using Microsoft.AspNetCore.RateLimiting;
using System.Time;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();

// Bind rate limiting configuration
builder.Services.Configure<RateLimitingOptions>(builder.Configuration.GetSection("RateLimiting"));

// Add rate limiting services
builder.Services.AddRateLimiter(options =>
{
    var rateLimitingOptions = builder.Configuration.GetSection("RateLimiting").Get<RateLimitingOptions>();

    options.AddPolicy("SlidingWindowPolicy", context =>
        RateLimitPartition.GetSlidingWindowLimiter(
            partitionKey: context.User.Identity?.Name ?? context.Request.Headers.Host.ToString(),
            options => new SlidingWindowRateLimiterOptions
            {
                PermitLimit = rateLimitingOptions.PermitLimit,
                Window = TimeSpan.FromSeconds(rateLimitingOptions.WindowInSeconds),
                QueueProcessingOrder = Enum.Parse<QueueProcessingOrder>(rateLimitingOptions.QueueProcessingOrder),
                QueueLimit = rateLimitingOptions.QueueLimit
            }))
        .OnRejected(async (context, cancellationToken) =>
        {
            context.HttpContext.Response.StatusCode = StatusCodes.Status429TooManyRequests;
            // Explicitly set Content-Type to application/scim+json
            context.HttpContext.Response.ContentType = "application/scim+json";
            var errorResponse = new ScimErrorResponse
            {
                Detail = "Too many requests. Please try again later.",
                Status = StatusCodes.Status429TooManyRequests.ToString()
            };
            await context.HttpContext.Response.WriteAsync(
                JsonSerializer.Serialize(errorResponse, new JsonSerializerOptions
                {
                    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
                }),
                cancellationToken);
        });
});

var app = builder.Build();

// Use rate limiting middleware
app.UseRateLimiter();
app.MapControllers();
app.Run();
```  
---  
## Step 4: Use Configurable Options in Rate Limiting  

The `RateLimitingOptions` class is now bound to the configuration, and its values are used to configure the `SlidingWindowRateLimiterOptions`. This ensures that changes to `appsettings.json` are reflected in the rate-limiting behavior without requiring a code change.  

For example, if you update `appsettings.json` to:  
```json  
{
  "RateLimiting": {
    "PermitLimit": 20,
    "WindowInSeconds": 120,
    "QueueProcessingOrder": "NewestFirst",
    "QueueLimit": 5
  }
}
```  
The rate limiter will automatically enforce these new settings when the application restarts.  

---  

## Step 5: Test the Implementation  

- Use tools like Postman , cURL , or Swagger to send multiple requests to your SCIM API endpoints.
- Exceed the rate limit to trigger the `.OnRejected` callback.
- Verify that the response has:
    - HTTP status code `429` Too Many Requests.
    - Content-Type header set to `application/scim+json`.
    - Response body in the SCIM error schema format.

Example cURL command:  
```bash
curl -X GET http://localhost:5000/scim/v2/Users
``` 

Expected response when rate-limited:  
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
  "detail": "Too many requests. Please try again later.",
  "status": "429"
}
```  
Additionally, verify the headers:  

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/scim+json
```

--- 
## Optional: Environment-Specific Configuration  
If you want different `rate-limiting` settings for different environments (e.g., `development`, `staging`, `production`), you can use environment-specific configuration files like `appsettings.Development.json` or `appsettings.Production.json`.

For example, in `appsettings.Production.json`:  
```json
{
  "RateLimiting": {
    "PermitLimit": 50,
    "WindowInSeconds": 60,
    "QueueProcessingOrder": "OldestFirst",
    "QueueLimit": 10
  }
}
```  

## Optional: Customize the Error Message  

You can customize the `Detail` field of the SCIM error response based on the context. For example, include information about when the client can retry:

```csharp  
var retryAfter = TimeSpan.FromSeconds(10); // Example: Retry after 10 seconds
context.HttpContext.Response.Headers.Append("Retry-After", ((int)retryAfter.TotalSeconds).ToString());

var errorResponse = new ScimErrorResponse
{
    Detail = $"Too many requests. Please try again after {retryAfter.TotalSeconds} seconds.",
    Status = StatusCodes.Status429TooManyRequests.ToString()
};
```  
This adds a Retry-After header to the response and updates the error message accordingly.  

---

## Conclusion

By following these steps, you can ensure that your SCIM API returns a compliant SCIM error response with the correct `Content-Type` header (`application/scim+json`) when rate limiting is triggered. This approach enhances the usability and interoperability of your API while adhering to the SCIM specification. It also allows you the flexibility to configure the rete-limiting options for different environments.



# Implementing Global Rate Limiting with `SemaphoreSlim` in ASP.NET Core

In this tutorial, we’ll implement **rate limiting** for SCIM APIs (e.g., `/scim/users`) using `SemaphoreSlim`.  
The goal is to enforce a **global concurrency limit** across all clients and return a **SCIM-compliant error response (429)** when requests exceed the allowed limit.  

We’ll also make the concurrency and retry-after values **configurable via `appsettings.json`**, not hardcoded.  

Finally, we’ll add **unit tests** to validate the middleware.

---

## 1. Why `SemaphoreSlim`?

- `SemaphoreSlim` is ideal for controlling concurrency because it allows us to limit **simultaneous** requests.
- Unlike request-per-second throttling, this ensures only **N concurrent requests** can be processed at a time, queueing or rejecting the rest.

---

## 2. Configuration (`appsettings.json`)

We define the concurrency and retry-after values under a `RateLimiting` section.

```json
{
  "RateLimiting": {
    "ScimUsersMaxConcurrentRequests": 2,
    "RetryAfterSeconds": 30
  }
}
```

- **ScimUsersMaxConcurrentRequests** → Max number of requests processed concurrently for `/scim/users`.  
- **RetryAfterSeconds** → The suggested wait time before retrying, included in the `Retry-After` response header.

---

## 3. Constants (`ScimConstants.cs`)

To keep SCIM error formatting clean, we define constants in one place.

```csharp
namespace MyApi.Scim
{
    public static class ScimConstants
    {
        public const string ErrorSchema = "urn:ietf:params:scim:api:messages:2.0:Error";

        public const string TooManyRequestsDetail =
            "Too Many Requests - The SCIM /users endpoint is rate limited.";
    }
}
```

---

## 4. Middleware (`PathRateLimitMiddleware.cs`)

This middleware enforces concurrency limits for `/scim/users`.

```csharp
using System.Text.Json;
using Microsoft.AspNetCore.Http;

namespace MyApi.Middleware
{
    public class PathRateLimitMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly ILogger<PathRateLimitMiddleware> _logger;
        private readonly SemaphoreSlim _semaphore;
        private readonly int _retryAfterSeconds;

        public PathRateLimitMiddleware(RequestDelegate next, 
                                       IConfiguration configuration,
                                       ILogger<PathRateLimitMiddleware> logger)
        {
            _next = next;
            _logger = logger;

            var maxConcurrent = configuration.GetValue<int>("RateLimiting:ScimUsersMaxConcurrentRequests", 5);
            _retryAfterSeconds = configuration.GetValue<int>("RateLimiting:RetryAfterSeconds", 30);

            _semaphore = new SemaphoreSlim(maxConcurrent, maxConcurrent);
        }

        public async Task InvokeAsync(HttpContext context)
        {
            if (context.Request.Path.StartsWithSegments("/scim/users", StringComparison.OrdinalIgnoreCase))
            {
                if (!await _semaphore.WaitAsync(0))
                {
                    await WriteScimErrorResponse(context);
                    return;
                }

                try
                {
                    await _next(context);
                }
                finally
                {
                    _semaphore.Release();
                }
            }
            else
            {
                await _next(context);
            }
        }

        private async Task WriteScimErrorResponse(HttpContext context)
        {
            context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
            context.Response.ContentType = "application/scim+json";
            context.Response.Headers["Retry-After"] = _retryAfterSeconds.ToString();

            var error = new
            {
                schemas = new[] { ScimConstants.ErrorSchema },
                detail = ScimConstants.TooManyRequestsDetail,
                status = StatusCodes.Status429TooManyRequests.ToString()
            };

            var json = JsonSerializer.Serialize(error);
            await context.Response.WriteAsync(json);
        }
    }
}
```

---

## 5. Register Middleware (`Program.cs`)

Add the middleware early in the request pipeline:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();

var app = builder.Build();

// Register middleware before endpoints
app.UseMiddleware<MyApi.Middleware.PathRateLimitMiddleware>();

app.MapControllers();

app.Run();
```

---

## 6. Tests (`PathRateLimitMiddlewareTests.cs`)

We’ll use **xUnit** with `[Theory]` to parameterize test cases.  

### Test Class

```csharp
using System.Net;
using System.Text.Json;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.TestHost;
using Xunit;
using FluentAssertions;

public class PathRateLimitMiddlewareTests
{
    private const string TooManyRequestsDetail = 
        "Too Many Requests - The SCIM /users endpoint is rate limited.";

    private HttpClient CreateClient(int maxConcurrent, int retryAfterSeconds)
    {
        var builder = new WebHostBuilder()
            .ConfigureAppConfiguration((context, config) =>
            {
                var dict = new Dictionary<string, string?>
                {
                    ["RateLimiting:ScimUsersMaxConcurrentRequests"] = maxConcurrent.ToString(),
                    ["RateLimiting:RetryAfterSeconds"] = retryAfterSeconds.ToString()
                };
                config.AddInMemoryCollection(dict!);
            })
            .Configure(app =>
            {
                app.UseMiddleware<MyApi.Middleware.PathRateLimitMiddleware>();
                app.Run(async ctx =>
                {
                    await Task.Delay(200);
                    await ctx.Response.WriteAsync("OK");
                });
            });

        var server = new TestServer(builder);
        return server.CreateClient();
    }

    [Theory]
    [InlineData(1, 10, HttpStatusCode.TooManyRequests)]
    [InlineData(2, 5, HttpStatusCode.OK)]
    public async Task Enforces_RateLimit_With_Configurable_RetryAfter(
        int maxConcurrent, int retryAfter, HttpStatusCode expectedStatus)
    {
        var client = CreateClient(maxConcurrent, retryAfter);

        // First request takes the slot
        var req1 = client.GetAsync("/scim/users");

        // Second request decides the test outcome
        var req2 = await client.GetAsync("/scim/users");

        req2.StatusCode.Should().Be(expectedStatus);

        if (expectedStatus == HttpStatusCode.TooManyRequests)
        {
            // Validate Retry-After header
            req2.Headers.Should().ContainKey("Retry-After");
            req2.Headers.GetValues("Retry-After")
                .Should().ContainSingle()
                .Which.Should().Be(retryAfter.ToString());

            // Validate SCIM error response
            var body = await req2.Content.ReadAsStringAsync();
            var json = JsonDocument.Parse(body);

            json.RootElement.GetProperty("schemas")[0].GetString()
                .Should().Be("urn:ietf:params:scim:api:messages:2.0:Error");
            json.RootElement.GetProperty("detail").GetString()
                .Should().Be(TooManyRequestsDetail);
            json.RootElement.GetProperty("status").GetString()
                .Should().Be(((int)HttpStatusCode.TooManyRequests).ToString());
        }
    }

    [Fact]
    public async Task Ignores_NonMatching_Path()
    {
        var client = CreateClient(maxConcurrent: 1, retryAfterSeconds: 5);

        var response = await client.GetAsync("/health");

        response.StatusCode.Should().Be(HttpStatusCode.OK);
        (await response.Content.ReadAsStringAsync()).Should().Be("OK");
    }
}
```

---

## 7. Run the Tests

From the project root:

```bash
dotnet test
```

Expected output: All tests pass ✅

---

## 8. Summary

- We implemented a **global concurrency-based rate limiter** using `SemaphoreSlim`.  
- Limits are **configurable via appsettings.json**.  
- When over the limit, a **SCIM-compliant 429 response** is returned with `Retry-After`.  
- Unit tests validate both **allowed** and **rejected** cases, plus **bypass for non-matching paths**.

This approach is lightweight, config-driven, and SCIM-compliant, making it a good fit for **customer identity APIs** like SCIM.

---

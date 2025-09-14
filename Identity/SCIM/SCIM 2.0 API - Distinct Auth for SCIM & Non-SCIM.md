## SCIM 2.0 API: Distinct Auth for SCIM & Non-SCIM

This tutorial explains how to implement a .NET application that exposes SCIM-compliant user endpoints alongside other REST API endpoints. The solution ensures that:

1. SCIM endpoints are authenticated via a specific issuer, while other REST endpoints use a different issuer (or the same issuer with different error responses).
2. SCIM endpoints return SCIM-compliant error responses for both authentication and authorization failures.
3. Non-SCIM endpoints return generic HTTP error responses.

---

### **1. Define Multiple Authentication Schemes**

To handle separate issuers and error responses, we define multiple authentication schemes in `Program.cs`. Each scheme corresponds to a specific issuer, and custom middleware is used to handle SCIM-specific error responses.

#### Updated `Program.cs`

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers()
    .AddNewtonsoftJson();

// SCIM-specific authentication scheme
builder.Services.AddAuthentication()
    .AddJwtBearer("ScimAuth", options =>
    {
        options.Authority = "https://sts.windows.net/{scim-tenant-id}/"; // Replace with SCIM tenant ID
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidIssuer = "https://sts.windows.net/{scim-tenant-id}/",
            ValidateAudience = false,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            IssuerSigningKeyResolver = (token, securityToken, kid, validationParameters) =>
            {
                // Resolve the signing key from Microsoft's public keys
                return validationParameters.IssuerSigningKeys;
            }
        };
        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                // Handle SCIM-specific authentication failure
                context.NoResult();
                context.Response.StatusCode = 401; // Return 401 for authentication failure
                context.Response.ContentType = "application/scim+json";
                var scimError = new ScimError
                {
                    Detail = "Authentication failed.",
                    Status = "401"
                };
                var json = System.Text.Json.JsonSerializer.Serialize(scimError);
                context.Response.WriteAsync(json);
                return System.Threading.Tasks.Task.CompletedTask;
            },
            OnChallenge = context =>
            {
                // Handle SCIM-specific authorization failure
                context.HandleResponse();
                context.Response.StatusCode = 403; // Return 403 for authorization failure
                context.Response.ContentType = "application/scim+json";
                var scimError = new ScimError
                {
                    Detail = "Authorization failed.",
                    Status = "403"
                };
                var json = System.Text.Json.JsonSerializer.Serialize(scimError);
                context.Response.WriteAsync(json);
                return System.Threading.Tasks.Task.CompletedTask;
            }
        };
    })
    // Default authentication scheme for other API endpoints
    .AddJwtBearer("DefaultAuth", options =>
    {
        options.Authority = "https://sts.windows.net/{default-tenant-id}/"; // Replace with default tenant ID
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidIssuer = "https://sts.windows.net/{default-tenant-id}/",
            ValidateAudience = false,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            IssuerSigningKeyResolver = (token, securityToken, kid, validationParameters) =>
            {
                // Resolve the signing key from Microsoft's public keys
                return validationParameters.IssuerSigningKeys;
            }
        };
    });

// Add authorization policies
builder.Services.AddAuthorization(options =>
{
    // Policy for SCIM endpoints
    options.AddPolicy("ScimPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("ScimAuth");
        policy.RequireAuthenticatedUser();
    });

    // Policy for other API endpoints
    options.AddPolicy("DefaultPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("DefaultAuth");
        policy.RequireAuthenticatedUser();
    });
});

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

// Map controllers
app.MapControllers();

app.Run();
```

### **2. SCIM Models**

`Models/ScimError.cs`  

The `ScimError` model defines the structure of SCIM-compliant error responses.

```csharp
namespace ScimUserManagement.Models
{
    public class ScimError
    {
        public string[] Schemas { get; set; } = new[] { "urn:ietf:params:scim:api:messages:2.0:Error" };
        public string Detail { get; set; }
        public string Status { get; set; }
    }
}
```

### **3. Apply Policies to Controllers**  

**SCIM Endpoints (`UsersController`)**

Apply the `ScimPolicy` to the SCIM endpoints. The `JwtBearerEvents` in the `ScimAuth` scheme ensure that SCIM-compliant error responses are returned for authentication and authorization failures.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using ScimUserManagement.Models;
using System;
using System.Collections.Generic;
using System.Linq;

namespace ScimUserManagement.Controllers
{
    [Route("scim/v2/Users")]
    [ApiController]
    [Authorize(Policy = "ScimPolicy")] // Use the SCIM-specific policy
    public class UsersController : ControllerBase
    {
        private static readonly List<ScimUser> _users = new List<ScimUser>();
        private static int _userIdCounter = 1;

        // GET /scim/v2/Users
        [HttpGet]
        public IActionResult GetUsers([FromQuery] string filter = null, [FromQuery] int startIndex = 1, [FromQuery] int count = 100)
        {
            IEnumerable<ScimUser> filteredUsers = _users;

            if (!string.IsNullOrEmpty(filter))
            {
                if (filter.Contains("userName eq"))
                {
                    var userName = filter.Split("\"")[1];
                    filteredUsers = filteredUsers.Where(u => u.UserName == userName);
                }
                else if (filter.Contains("externalId eq"))
                {
                    var externalId = filter.Split("\"")[1];
                    filteredUsers = filteredUsers.Where(u => u.ExternalId == externalId);
                }
            }

            var paginatedUsers = filteredUsers.Skip(startIndex - 1).Take(count);

            return Ok(new ScimListResponse<ScimUser>
            {
                TotalResults = filteredUsers.Count(),
                ItemsPerPage = count,
                StartIndex = startIndex,
                Resources = paginatedUsers.ToList()
            });
        }

        // Other SCIM methods remain unchanged...
    }
}
```
**Other API Endpoints**  

Apply the `DefaultPolicy` to non-SCIM endpoints. These endpoints will return generic HTTP error responses.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace ScimUserManagement.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    [Authorize(Policy = "DefaultPolicy")] // Use the default policy
    public class OtherApiController : ControllerBase
    {
        [HttpGet]
        public IActionResult GetData()
        {
            return Ok(new { Message = "This is a non-SCIM endpoint." });
        }
    }
}
```

### **4. Middleware Order**  

Ensure that the middleware order in `Program.cs` is correct:

1. `app.UseHttpsRedirection();`
2. `app.UseAuthentication();`
3. `app.UseAuthorization();`
4. `app.MapControllers();`  

The `UseAuthenticatio`n middleware must come before `UseAuthorization` to ensure that authentication is performed before enforcing policies.

### **5. Testing the Configuration**

1. **SCIM Endpoints:**  

    - **Authentication Failure**  
        - Send a request to `/scim/v2/Users` with an invalid or missing token.
        - The response should be a `401 Unauthorized` with a SCIM-compliant error body:
        ```json
        {
            "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
            "detail": "Authentication failed.",
            "status": "401"
        }
        ```
    - **Authorization Failure**  
        - Send a request to `/scim/v2/Users` with a valid token but insufficient permissions.
        - The response should be a `403 Forbidden` with a SCIM-compliant error body:  
        ```json
        {
            "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
            "detail": "Authorization failed.",
            "status": "403"
        }

2. **Other API Endpoints:**  

    - **Authentication Failure**  
        - Send requests to `/api/OtherApi` with an invalid or missing token.
        - The response should be a generic `401 Unauthorized` without a SCIM-compliant error body.

    - **Authorization Failure**  
        - Send a request to `/api/OtherApi` with a valid token but insufficient permissions.
        - The response should be a `403 Forbidden` without a SCIM-compliant error body.  

### **6. Key Points**  

- **Correct Use of Status Codes**  
    - `401 Unauthorized` is returned for authentication failures.
    - `403 Forbidden` is returned for authorization failures.

- **SCIM-Compliant Error Responses :** Both `401` and `403` responses for SCIM endpoints include SCIM-compliant error bodies.
- **Separation of Concerns :** Non-SCIM endpoints return generic HTTP error responses, while SCIM endpoints adhere to SCIM standards.

### Conclusion  
The implementation ensures that SCIM endpoints return the correct HTTP status codes (`401` for authentication failures and `403` for authorization failures) along with `SCIM-compliant` error responses. Non-SCIM endpoints continue to return generic HTTP error responses. This approach adheres to HTTP standards and SCIM 2.0 specifications, ensuring a robust and compliant API design.
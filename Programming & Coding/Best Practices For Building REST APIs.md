# Best Practices For Building REST APIs

Building REST APIs seems straightforward until you face real-world challenges.

A well-designed REST API is predictable, maintainable, and easy to consume.

It follows consistent patterns that developers can understand without needing to read extensive documentation.

When building REST APIs with ASP.NET Core, these practical guidelines are used in production systems handling millions of requests daily.

Here, we will explore:

-   Understanding REST Maturity Levels
-   How to name resources correctly for clean API design
-   Choosing the correct HTTP methods
-   Choosing the correct status codes
-   Implementing API versioning strategies that work
-   Structuring requests and responses with proper standards

Let's dive in.

[](#understanding-rest-maturity-levels)

## Understanding REST Maturity Levels

Leonard Richardson created a maturity model that breaks down REST adoption into four levels.

**Level 0: The Swamp of POX (Plain Old XML)**

At this level, you're using HTTP as a transport mechanism, nothing more. You have a single URI endpoint that typically uses only POST requests, and the request body contains all the information about what operation to perform. This is essentially RPC (Remote Procedure Call) over HTTP. SOAP APIs often operate at this level.

Endpoint example: `/api/service`

**Level 1: Resources**

At this level, you start breaking down your single endpoint into multiple resource-based URIs. Instead of having a single endpoint handle everything, you have different URIs for different resources.

Endpoint examples: `/api/orders`, `/api/customers`, `/api/products`

**Level 2: HTTP Verbs**

Now you're using HTTP methods correctly. GET for retrieving data, POST for creating, PUT for updating, and DELETE for removing. You also use HTTP status codes meaningfully.

Endpoint examples:

-   GET `/api/orders/123` - retrieve an order
-   POST `/api/orders` - create a new order
-   PUT `/api/orders/123` - update an order
-   DELETE `/api/orders/123` - delete an order

Proper status codes: 200 OK, 201 Created, 404 Not Found, etc.

Most APIs stop at Level 2, and it works. Such APIs are predictable; they follow HTTP standards, and developers know how to build them.

Here is a typical API at Level 2, implemented with ASP.NET Core and Controllers:

```csharp
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }
    [HttpGet("{id}")]
    public ActionResult<Order> GetOrder(int id)
    {
        var order = _orderService.GetById(id);
        
        if (order is null)
        {
            return NotFound();
        }
        
        return Ok(order);
    }
    [HttpPost]
    public ActionResult<Order> CreateOrder([FromBody] CreateOrderRequest request)
    {
        var order = _orderService.Create(request);
        
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
    [HttpPut("{id}")]
    public ActionResult<Order> UpdateOrder(int id, [FromBody] UpdateOrderRequest request)
    {
        if (id != request.Id)
        {
            return BadRequest("Id mismatch");
        }
        
        var order = _orderService.Update(request);
        
        if (order is null)
        {
            return NotFound();
        }
        
        return Ok(order);
    }
    [HttpDelete("{id}")]
    public IActionResult DeleteOrder(int id)
    {
        var success = _orderService.Delete(id);
        
        if (!success)
        {
            return NotFound();
        }
        
        return NoContent();
    }
}
```

**Level 3: Hypermedia Controls (HATEOAS)**

This is true REST. Your API responses include links to related resources and available actions. The client doesn't need to know your URL structure upfront. The server tells the client what it can do next.

You can read my article on [HATEOAS](https://antondevtips.com/blog/90-of-apis-are-not-restful-what-youre-missing-and-when-it-matters) to learn when you can benefit from HATEOAS and when it's an overkill.

[](#how-to-name-resources-correctly-for-clean-api-design)

## How to Name Resources Correctly for Clean API Design

Resource naming is one of the most visible aspects of your API design. Poor naming creates confusion and makes your API harder to use. Good naming makes your API intuitive and self-documenting.

[](#1-use-nouns-not-verbs)

### 1\. Use Nouns, Not Verbs

Resources should represent things instead of actions. The HTTP method (GET, POST, PUT, DELETE) indicates the action.

```csharp
// Bad - Using verbs in URLs
[HttpGet("api/getProducts")]
[HttpPost("api/createProduct")]
[HttpPut("api/updateProduct")]
[HttpDelete("api/deleteProduct")]
// Good - Using nouns with HTTP methods
[HttpGet("api/products")]
[HttpPost("api/products")]
[HttpPut("api/products/{id}")]
[HttpDelete("api/products/{id}")]
```

[](#2-use-plural-nouns-for-collections)

### 2\. Use Plural Nouns for Collections

Collections should always use plural nouns for consistency:

```csharp
// Bad - Mixing singular and plural
[Route("api/product")]  // Singular
[Route("api/orders")]   // Plural
// Good - Always plural
[Route("api/products")]
[Route("api/orders")]
[Route("api/customers")]
```

This creates a predictable pattern. Even when getting a single item, you still use the plural form:

```csharp
[HttpGet("api/products/{id}")]      // Get single product
[HttpGet("api/products")]           // Get all products
```

The only exception is nouns that are always singular, such as software and localization.

[](#3-handling-multiword-resources)

### 3\. Handling Multi-Word Resources

Choose one naming convention and stick to it consistently throughout your entire API.

**Lower kebab-case** is recommended for multi-word resources, it's URL-friendly, easy to read, and widely adopted:

```csharp
// Good - Kebab-case (lowercase with hyphens)
[Route("api/product-categories")]
[Route("api/shopping-carts")]
[Route("api/customer-addresses")]
[Route("api/order-items")]
[ApiController]
[Route("api/product-categories")]
public class ProductCategoriesController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<ProductCategory>> GetAll()
    {
        var categories = _categoryService.GetAll();
        return Ok(categories);
    }
}
```

[](#4-resource-hierarchies-and-nested-resources)

### 4\. Resource Hierarchies and Nested Resources

When resources have clear parent-child relationships, reflect that in the URL structure:

```csharp
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    // Get all orders
    [HttpGet]
    public ActionResult<IEnumerable<Order>> GetOrders()
    {
        var orders = _orderService.GetAll();
        return Ok(orders);
    }
    // Get specific order
    [HttpGet("{orderId}")]
    public ActionResult<Order> GetOrder(int orderId)
    {
        var order = _orderService.GetById(orderId);
        
        if (order is null)
            return NotFound();
        
        return Ok(order);
    }
    // Get items for specific order
    [HttpGet("{orderId}/items")]
    public ActionResult<IEnumerable<OrderItem>> GetOrderItems(int orderId)
    {
        var order = _orderService.GetById(orderId);
        
        if (order is null)
            return NotFound();
        
        var items = _orderService.GetItems(orderId);
        return Ok(items);
    }
    // Get specific item from specific order
    [HttpGet("{orderId}/items/{itemId}")]
    public ActionResult<OrderItem> GetOrderItem(int orderId, int itemId)
    {
        var item = _orderService.GetItem(orderId, itemId);
        
        if (item is null)
            return NotFound();
        
        return Ok(item);
    }
    // Add item to order
    [HttpPost("{orderId}/items")]
    public ActionResult<OrderItem> AddOrderItem(
        int orderId, 
        [FromBody] CreateOrderItemRequest request)
    {
        var order = _orderService.GetById(orderId);
        
        if (order is null)
            return NotFound();
        
        var item = _orderService.AddItem(orderId, request);
        
        return CreatedAtAction(
            nameof(GetOrderItem),
            new { orderId = orderId, itemId = item.Id },
            item);
    }
}
```

[](#5-when-not-to-nest-resources)

### 5\. When Not to Nest Resources

Avoid nesting more than two levels deep. Deep nesting creates long, complex URLs:

```csharp
// Bad - Too much nesting
[HttpGet("api/customers/{customerId}/orders/{orderId}/items/{itemId}/reviews/{reviewId}")]
// Good - Flatten the hierarchy
[HttpGet("api/reviews/{reviewId}")]
[HttpGet("api/order-items/{itemId}/reviews")]
```

If a resource can exist independently, give it its own top-level endpoint.

Here is a complete example showing consistent resource naming:

```csharp
// Products and categories
[Route("api/products")]
[Route("api/product-categories")]
// Customer management
[Route("api/customers")]
[Route("api/customers/{customerId}/addresses")]
[Route("api/customers/{customerId}/payment-methods")]
// Orders and shopping
[Route("api/orders")]
[Route("api/orders/{orderId}/items")]
[Route("api/shopping-carts")]
[Route("api/shopping-carts/{cartId}/items")]
// Inventory
[Route("api/warehouses")]
[Route("api/warehouses/{warehouseId}/inventory")]
// Reviews
[Route("api/products/{productId}/reviews")]
[Route("api/reviews/{reviewId}")]  // Direct access when you have the ID
```

[](#6-query-parameters-for-filtering-and-operations)

### 6\. Query Parameters for Filtering and Operations

Use query parameters for filtering:

```csharp
// Good - Query parameters for filtering
[HttpGet("api/products?category=electronics&inStock=true")]
[HttpGet("api/orders?status=pending&customerId=123")]
[HttpGet("api/products?search=laptop")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetProducts(
        [FromQuery] string category = null,
        [FromQuery] bool? inStock = null,
        [FromQuery] string search = null)
    {
        var products = _productService.GetFiltered(category, inStock, search);
        return Ok(products);
    }
}
// Bad - Query parameters for actions
[HttpGet("api/products?action=delete&id=123")]
[HttpPost("api/orders?operation=cancel")]
```

Don't use query parameters for actions.

Query parameters refine which resources you want. The HTTP method and path determine the action. Good resource naming makes your API predictable and easy to understand. Stick to these patterns consistently across your entire API.

[](#choosing-the-right-http-methods)

## Choosing the Right HTTP Methods

HTTP methods and status codes are the foundation of RESTful communication. Using them correctly makes your API predictable and allows clients to handle responses appropriately. Using them incorrectly creates confusion and bugs.

[](#get-retrieve-resources)

### GET - Retrieve Resources

GET retrieves data without modifying anything. It must be safe and idempotent.

**GET requests:**

-   Should never modify data
-   Can be cached
-   Are idempotent
-   Are safe to retry

[](#post-create-new-resources)

### POST - Create New Resources

POST creates new resources.

**POST requests:**

-   Create new resources
-   Are not idempotent (multiple calls create multiple resources)
-   Should return 201 Created with the new resource location
-   Include the created resource in the response body

[](#put-replace-entire-resource)

### PUT - Replace Entire Resource

PUT replaces the entire resource with the provided data. It is idempotent.

**PUT requests:**

-   Replace the entire resource
-   Are idempotent (multiple identical calls have the same effect)
-   Require all fields in the request
-   Can return 204 No Content or 200 OK with the updated resource

[](#patch-partial-update)

### PATCH - Partial Update

PATCH updates only specific fields. It is idempotent when designed properly.

**PATCH requests:**

-   Update only specified fields
-   Are idempotent
-   Use nullable types to indicate optional fields
-   Return 204 No Content

**DELETE - Remove Resources**

DELETE removes a resource. It is idempotent.

**DELETE requests:**

-   Remove the resource
-   Are idempotent (deleting twice has the same effect as deleting once)
-   Return 204 No Content on success
-   Return 404 Not Found if the resource does not exist

[](#choosing-the-right-status-codes)

## Choosing the Right Status Codes

[](#success-status-codes)

### Success Status Codes

**200 OK - Standard Success**

Use for successful `GET`, `PUT`, and `PATCH` operations:

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _productService.GetById(id);
    return Ok(product);  // 200 OK
}
```

**201 Created - Resource Created**

Use for successful POST operations:

```csharp
[HttpPost]
public ActionResult<Order> CreateOrder([FromBody] CreateOrderRequest request)
{
    var order = _orderService.Create(request);
    
    // 201 Created with Location header
    return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
}
```

**202 Accepted - Asynchronous Processing**

Use when the request is accepted, but processing will be completed asynchronously:

```csharp
[HttpPost("process-bulk-import")]
public IActionResult ProcessBulkImport([FromBody] BulkImportRequest request)
{
    var jobId = _importService.QueueImport(request);
    
    return AcceptedAtAction(
        nameof(GetImportStatus),
        new { jobId = jobId },
        new { jobId = jobId, status = "processing" });  // 202 Accepted
}

[HttpGet("import-status/{jobId}")]
public ActionResult<ImportStatus> GetImportStatus(string jobId)
{
    var status = _importService.GetStatus(jobId);
    return Ok(status);
}
```

**204 No Content - Success With No Response Body**

Use for successful `DELETE` operations or `PUT`, `PATCH` that do not return data:

```csharp
[HttpDelete("{id}")]
public IActionResult DeleteProduct(int id)
{
    var deleted = _productService.Delete(id);
    
    if (!deleted)
    {
        return NotFound();
    }
    
    return NoContent();  // 204 No Content
}
```

[](#client-error-status-codes)

### Client Error Status Codes

-   **400 Bad Request:** use when the request is malformed or contains invalid data.
-   **401 Unauthorized:** use when authentication is missing or invalid
-   **403 Forbidden:** use when the user is authenticated but lacks permission
-   **404 Not Found:** use when the requested resource does not exist
-   **409 Conflict:** use when the request conflicts with the current state of the resource
-   **422 Unprocessable Entity:** use when the request is well-formed but fails business validation
-   **429 Too Many Requests:** use when the client is sending too many requests in a given amount of time (Rate Limiting)

Example when the user is authenticated but lacks permission (403 Forbidden):

```csharp
[HttpDelete("{id}")]
public IActionResult DeleteOrder(int id)
{
    var order = _orderService.GetById(id);
    
    if (order is null)
    {
        return NotFound();
    }
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    
    // User can only delete their own orders
    if (order.CustomerId.ToString() != userId)
    {
        return Forbid();  // 403 Forbidden
    }
    _orderService.Delete(id);
    return NoContent();
}
```

Example when the request is well-formed but fails business validation (422 Unprocessable Entity):

```csharp
[HttpPost("checkout")]
public ActionResult<Order> Checkout([FromBody] CheckoutRequest request)
{
    var cart = _cartService.GetById(request.CartId);
    
    if (cart is null)
    {
        return NotFound();
    }
    // Business validation
    var validation = _orderService.ValidateCheckout(cart);
    
    if (!validation.IsValid)
    {
        return UnprocessableEntity(validation.Errors);  // 422
    }
    var order = _orderService.CompleteCheckout(cart, request);
    return Ok(order);
}
```

[](#server-error-status-codes)

### Server Error Status Codes

-   **500 Internal Server Error:** use when a server-side error occurs (ASP.NET Core handles this automatically for unhandled exceptions).
-   **503 Service Unavailable:** use when a dependent service is unavailable

[](#implementing-api-versioning-strategies-that-work)

## Implementing API Versioning Strategies That Work

APIs evolve over time, and you need a way to introduce changes without breaking existing clients. API versioning allows you to maintain backward compatibility while adding new features or fixing design issues.

If you can avoid versioning, do it:

-   Add non-required fields to existing resources
-   Add new endpoints

If you need to introduce breaking changes, use versioning.

[](#uri-versioning)

### URI Versioning

URI versioning puts the version number directly in the URL path. This is the most common and straightforward approach:

```csharp
// Version 1
[ApiController]
[Route("api/v1/products")]
public class ProductsV1Controller : ControllerBase
{
    private readonly IProductService _productService;
    public ProductsV1Controller(IProductService productService)
    {
        _productService = productService;
    }
    [HttpGet("{id}")]
    public ActionResult<ProductV1> GetProduct(int id)
    {
        var product = _productService.GetById(id);
        
        if (product is null)
        {
            return NotFound();
        }
        
        return Ok(product);
    }
}
public class ProductV1
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

```csharp
// Version 2 - Added more fields and changed structure
[ApiController]
[Route("api/v2/products")]
public class ProductsV2Controller : ControllerBase
{
    private readonly IProductService _productService;
    public ProductsV2Controller(IProductService productService)
    {
        _product_service = productService;
    }
    [HttpGet("{id}")]
    public ActionResult<ProductV2> GetProduct(int id)
    {
        var product = _productService.GetById(id);
        
        if (product is null)
        {
            return NotFound();
        }
        
        return Ok(product);
    }
}
public class ProductV2
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public PriceInfo Pricing { get; set; }
    public InventoryInfo Inventory { get; set; }
}
public class PriceInfo
{
    public decimal BasePrice { get; set; }
    public decimal? DiscountPrice { get; set; }
    public string Currency { get; set; }
}
public class InventoryInfo
{
    public int StockQuantity { get; set; }
    public bool InStock { get; set; }
}
```

**Benefits of URI versioning:**

-   Immediately visible in the URL
-   Easy to test with a browser or tools
-   Simple to route in API gateways
-   Clear for documentation

```http
GET /api/v1/products
GET /api/v2/products
POST /api/v1/orders
POST /api/v2/orders
```

To configure versioning, add the `Asp.Versioning.Mvc` package to your project:

```csharp
dotnet add package Asp.Versioning.Mvc
```

And configure it in DI:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
// Add API versioning
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new UrlSegmentApiVersionReader();
});
var app = builder.Build();
app.MapControllers();
app.Run();
```

Header versioning uses a custom HTTP header to specify the version.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
// Configure header-based versioning
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new HeaderApiVersionReader("X-Api-Version");
});
var app = builder.Build();
app.MapControllers();
app.Run();
```

**Example API call with header:**

```http
GET /api/products
X-Api-Version: 2.0
```

**Downsides of header versioning:**

-   Not visible in the URL
-   Harder to test in browsers
-   Easy to forget the header

[](#media-type-versioning)

### Media Type Versioning

Media type versioning uses the `Accept` header to specify the version:

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new MediaTypeApiVersionReader();
});
```

**Example API call:**

```http
GET /api/products
Accept: application/vnd.myapi.v2+json
```

This approach follows REST principles closely but is more complex to implement and use.

[](#query-string-versioning)

### Query String Versioning

Query string versioning uses a query parameter to specify the version.

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new QueryStringApiVersionReader("version");
});
```

**Example API call:**

```http
GET /api/products?version=2
```

**Why to avoid this:**

-   Mixes versioning with filtering parameters
-   Easy to omit accidentally
-   Less clear than URI versioning

Note: When introducing a new version, plan to deprecate the old one.

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0", Deprecated = true)]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<ProductV1>> GetProducts()
    {
        // Add deprecation warning in response headers
        Response.Headers.Add("X-API-Deprecation-Warning", 
             "This API version is deprecated. Please migrate to v2 by 2025-12-31");
        
        var products = _productService.GetAll();
        return Ok(products);
    }
}
```

The API versioning package automatically adds deprecation information to the response headers when you mark a version as deprecated.

[](#structuring-requests-and-responses-with-proper-standards)

## Structuring Requests and Responses With Proper Standards

Consistent request and response structures make your API predictable and easy to integrate. Standard formats reduce confusion and allow clients to handle responses uniformly across all endpoints.

**1\. Prefer Using JSON:**

JSON is the standard format for REST APIs. It is readable, well-supported, and language-agnostic.

**2\. Use Consistent Property Naming:**

Use camelCase for JSON properties and stick to it consistently throughout your entire API.

**3\. Standardize Error Responses With RFC 9457 Problem Details:** RFC 9457 (previously RFC 7807) defines a standard format for error responses.

```csharp
using Microsoft.AspNetCore.Mvc;
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
// Enable Problem Details for all error responses
builder.Services.AddProblemDetails();
var app = builder.Build();
// Use Problem Details middleware
app.UseExceptionHandler();
app.UseStatusCodePages();
app.MapControllers();
app.Run();
```

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _productService.GetById(id);
        
        if (product == null)
        {
            return NotFound(new ProblemDetails
            {
                Type = "https://api.mystore.com/errors/product-not-found",
                Title = "Product not found",
                Status = 404,
                Detail = $"Product with ID {id} does not exist",
                Instance = $"/api/products/{id}"
            });
        }
        
        return Ok(product);
    }
}
```

Response example:

```json
{
  "type": "https://api.mystore.com/errors/product-not-found",
  "title": "Product not found",
  "status": 404,
  "detail": "Product with ID 123 does not exist",
  "instance": "/api/products/123"
}
```

Extend Problem Details for validation errors:

```csharp
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    public ActionResult<Order> CreateOrder([FromBody] CreateOrderRequest request)
    {
        var validationErrors = ValidateOrder(request);
        
        if (validationErrors.Any())
        {
            var problemDetails = new ValidationProblemDetails(validationErrors)
            {
                Type = "https://api.mystore.com/errors/validation-failed",
                Title = "One or more validation errors occurred",
                Status = 422,
                Detail = "Please correct the errors and try again",
                Instance = "/api/orders"
            };
            
            return UnprocessableEntity(problemDetails);
        }
        
        var order = _orderService.Create(request);
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
}
```

Validation error response:

```json
{
  "type": "https://api.mystore.com/errors/validation-failed",
  "title": "One or more validation errors occurred",
  "status": 422,
  "detail": "Please correct the errors and try again",
  "instance": "/api/orders",
  "errors": {
    "customerId": ["Customer ID must be greater than zero"],
    "items": ["Order must contain at least one item"],
    "items[0].quantity": ["Quantity must be greater than zero"]
  }
}
```

[](#summary)

## Summary

Consider this REST API checklist:

**Resource naming:**

✅ Use nouns: /users, /orders ❌ Avoid verbs: /getUsers, /createOrder

✅ Be consistent: user-profiles or product-carts ❌ Avoid: UserProfiles, userProfiles

**HTTP Methods & Status Codes:**

**Methods:**

-   GET → Read
-   POST → Create
-   PUT/PATCH → Update
-   DELETE → Remove

**Success Codes:**

-   200: Success
-   201: Created
-   202: Accepted (async)
-   204: No Content

**Error Codes (client):**

-   400: Bad Request
-   401: Unauthorized
-   403: Forbidden
-   404: Not Found
-   422: Validation Failed

**Error Codes (server):**

-   500: Internal Error on Server
-   503: Service Unavailable

**API Versioning:**

-   URI: /api/v1/users ✅
-   Header: X-Api-Version
-   Media Type: application/vnd.api.v1+json
-   Query String: ?version=1 (avoid)

**Request/Response Best Practices:**

-   Always use JSON
-   Standardize error responses
-   Support filtering & pagination
-   Document with OpenAPI/Swagger

Hope you find this newsletter useful. See you next time.

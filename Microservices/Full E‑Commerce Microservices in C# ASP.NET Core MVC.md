# 🛍️ End‑to‑End Tutorial: E‑Commerce Microservices in C# ASP.NET Core MVC

**Technologies Used:**
- **C# ASP.NET Core MVC** for microservices APIs  
- **Postgres** for relational storage (Products & Orders)  
- **Redis** for session & cart store  
- **RabbitMQ** for asynchronous event processing (cart checkout → orders → payments)  
- **Microsoft Entra ID (Azure AD)** for OAuth2 JWT authentication  
- **OpenTelemetry** for distributed tracing  
- **Grafana + Tempo + Loki + Prometheus** for observability dashboards  
- **RedisInsight GUI** for visualizing Redis key/values  
- **Docker Compose** to bring it all together locally  

---

## 1. Motivation

Distributed microservices systems need:  
- Separate services per domain (products, cart, orders, payments)  
- Authentication shared at the entry point (gateway)  
- Reliable storage (Postgres, Redis)  
- Event-driven async flows (RabbitMQ)  
- Observability (logs, metrics, traces)  

We’ll implement all this in **.NET Core Web API (MVC)** services, dockerized, observable, and demoable locally.

---

## 2. System Architecture

```
User ──► Gateway API (ASP.NET + Entra ID Auth)
           │
           ▼
  ┌─────────────────────────────────────┐
  │ Microservices                       │
  │                                     │
  │  Products API (EF Core + Postgres)  │
  │  Cart API (Redis)                   │
  │  Orders API (EF Core + Postgres)    │
  │  Payments API (Mock)                │
  └─────────────────────────────────────┘
           │
           ▼
 Event bus (RabbitMQ):
   Cart → CartCheckedOut → Orders
   Orders → OrderCreated → Payments

Infrastructure:  
- Postgres, Redis, RabbitMQ  
- RedisInsight for Redis GUI  
- Grafana + Tempo + Loki + Prometheus for monitoring  
```

---

## 3. Infrastructure with Docker Compose

`docker-compose.yml`

```yaml
version: "3.9"
services:

  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: ecommerce
    ports: ["5432:5432"]

  redis:
    image: redis:7
    ports: ["6379:6379"]

  rabbitmq:
    image: rabbitmq:3-management
    ports: ["5672:5672", "15672:15672"]
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest

  redisinsight:
    image: redislabs/redisinsight:latest
    ports: ["8001:8001"]
    depends_on: [redis]

  prometheus:
    image: prom/prometheus
    volumes: ["./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml"]
    ports: ["9090:9090"]

  loki:
    image: grafana/loki:2.9.0
    ports: ["3100:3100"]

  tempo:
    image: grafana/tempo:latest
    command: ["-config.file=/etc/tempo.yaml"]
    volumes: ["./monitoring/tempo.yaml:/etc/tempo.yaml"]
    ports: ["3200:3200"]

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    ports: ["3000:3000"]

  # Microservices containers (built locally)
  gateway:
    build: ./gateway
    ports: ["8080:80"]
    environment:
      - ENTRA_TENANT_ID=xxx
      - ENTRA_CLIENT_ID=xxx
      - ENTRA_CLIENT_SECRET=xxx
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
    depends_on: [postgres, redis, rabbitmq]

  products:
    build: ./products
    ports: ["8081:80"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
    depends_on: [postgres]

  cart:
    build: ./cart
    ports: ["8082:80"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
    depends_on: [redis, rabbitmq]

  orders:
    build: ./orders
    ports: ["8083:80"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
    depends_on: [postgres, rabbitmq]

  payments:
    build: ./payments
    ports: ["8084:80"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
    depends_on: [rabbitmq]
```

---

## 4. Common NuGet Packages

Each project adds:
```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.Identity.Web
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package StackExchange.Redis
dotnet add package RabbitMQ.Client
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

---

## 5. Gateway Service

- Validates JWTs from Microsoft Entra ID.  
- Proxies requests to Products service.  

**Program.cs**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer(options =>
    {
        options.Authority = $"https://login.microsoftonline.com/{builder.Configuration["ENTRA_TENANT_ID"]}/v2.0";
        options.Audience = builder.Configuration["ENTRA_CLIENT_ID"];
    });

builder.Services.AddHttpClient();
builder.Services.AddOpenTelemetry()
   .WithTracing(t => t.AddAspNetCoreInstrumentation()
                      .AddHttpClientInstrumentation()
                      .AddOtlpExporter(o => o.Endpoint = new Uri("http://tempo:4318")));

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**GatewayController.cs**
```csharp
[ApiController]
[Route("[controller]")]
public class GatewayController : ControllerBase
{
    private readonly HttpClient _client;
    public GatewayController(IHttpClientFactory factory) { _client = factory.CreateClient(); }

    [HttpGet("products")]
    [Authorize]
    public async Task<IActionResult> GetProducts()
    {
        var resp = await _client.GetStringAsync("http://products:80/products");
        return Content(resp, "application/json");
    }
}
```

---

## 6. Products Service (Postgres + EF Core)

**Models/Product.cs**
```csharp
public class Product { public int Id {get;set;} public string Name {get;set;} public double Price {get;set;} public int Stock {get;set;} }
```

**ProductsDbContext.cs**
```csharp
public class ProductsDbContext : DbContext
{
    public ProductsDbContext(DbContextOptions options):base(options){}
    public DbSet<Product> Products {get;set;}
}
```

**ProductsController.cs**
```csharp
[ApiController]
[Route("[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductsDbContext _db;
    public ProductsController(ProductsDbContext db){_db=db;}

    [HttpGet] public IEnumerable<Product> Get() => _db.Products.ToList();

    [HttpPost]
    public Product Add(Product p){ _db.Products.Add(p); _db.SaveChanges(); return p; }
}
```

---

## 7. Cart Service (Redis Publisher)

Publishes `CartCheckedOut` event to RabbitMQ.

**Program.cs**
```csharp
builder.Services.AddSingleton<IConnection>(sp => {
    var factory = new ConnectionFactory(){HostName="rabbitmq"};
    return factory.CreateConnection();
});
builder.Services.AddSingleton<IModel>(sp => {
    var c = sp.GetRequiredService<IConnection>().CreateModel();
    c.QueueDeclare("cart-checked-out", true, false, false);
    return c;
});
```

**CartController.cs**
```csharp
[ApiController]
[Route("[controller]")]
public class CartController : ControllerBase
{
    private readonly IDatabase _redis;
    private readonly IModel _channel;

    public CartController(IConnectionMultiplexer redis, IModel channel)
    {
        _redis = redis.GetDatabase();
        _channel = channel;
    }

    [HttpPost("add")]
    public async Task<IActionResult> Add(string userId, string productId)
    {
        await _redis.SetAddAsync($"cart:{userId}", productId);
        return Ok();
    }

    [HttpGet("{userId}")]
    public async Task<IActionResult> Get(string userId)
    {
        var items = await _redis.SetMembersAsync($"cart:{userId}");
        return Ok(items.Select(i => i.ToString()));
    }

    [HttpPost("checkout")]
    public IActionResult Checkout(string userId)
    {
        var msg = new { UserId = userId, Timestamp = DateTime.UtcNow };
        var body = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(msg));
        _channel.BasicPublish("", "cart-checked-out", null, body);
        return Accepted(new {Message="Checkout accepted!"});
    }
}
```

---

## 8. Orders Service (RabbitMQ Consumer + Postgres)

Consumes `cart-checked-out` messages.

**OrderWorker.cs**
```csharp
public class OrderWorker : BackgroundService
{
    private readonly ILogger<OrderWorker> _logger;
    private readonly IServiceProvider _provider;
    private readonly IModel _channel;

    public OrderWorker(IServiceProvider provider, ILogger<OrderWorker> logger, IConnection conn)
    {
        _provider=provider; _logger=logger;
        _channel=conn.CreateModel();
        _channel.QueueDeclare("cart-checked-out", true, false, false);
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var consumer = new EventingBasicConsumer(_channel);
        consumer.Received += async (m, ea) =>
        {
            var json = Encoding.UTF8.GetString(ea.Body.ToArray());
            var ev = JsonConvert.DeserializeObject<CartCheckoutEvent>(json);
            using var scope = _provider.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<OrdersDbContext>();
            var order = new Order { UserId=ev.UserId, Status="PENDING" };
            db.Orders.Add(order);
            await db.SaveChangesAsync();
            _logger.LogInformation("Created order {Id}", order.Id);
        };
        _channel.BasicConsume("cart-checked-out", true, consumer);
        return Task.CompletedTask;
    }
}

public class CartCheckoutEvent { public string UserId {get;set;} public DateTime Timestamp{get;set;} }
```

---

## 9. Payments Service (Optionally consumes `order-created`)

Similar to Orders, it can have a BackgroundService consuming an `order-created` queue and logs payment confirmation.

---

## 10. Observability Setup (OpenTelemetry)

Each service registers OpenTelemetry in `Program.cs`:

```csharp
builder.Services.AddOpenTelemetry()
   .WithTracing(t => t.AddAspNetCoreInstrumentation()
                      .AddHttpClientInstrumentation()
                      .AddOtlpExporter(o => o.Endpoint = new Uri("http://tempo:4318")));
```

Docker logs > Loki, metrics > Prometheus, traces > Tempo.

---

## 11. RedisInsight

Run at http://localhost:8001 to check keys like `cart:alice`.

---

## 12. Running & Demo

### Build
```bash
dotnet build
```

### Spin up infra + services
```bash
docker-compose up -d --build
```

### Verify
- Gateway → http://localhost:8080/products  
- Products → http://localhost:8081/products  
- Cart → http://localhost:8082/cart/alice  
- Orders → http://localhost:8083/orders  
- Payments → http://localhost:8084/pay  
- RabbitMQ mgmt → http://localhost:15672  
- Grafana → http://localhost:3000  
- Redis GUI → http://localhost:8001  

### Flow
1. Gateway requires JWT token via **Entra ID**.  
2. Add products → persisted in Postgres.  
3. Add to Cart (Redis) → see in RedisInsight.  
4. Checkout → event published to RabbitMQ.  
5. Orders worker consumes event → order persisted to Postgres.  
6. Payments can consume order events → confirm payment.  
7. See distributed traces across services in Grafana Tempo.  

### Teardown
```bash
docker-compose down -v
```

---

## 🎓 Conclusion

You’ve now built a **fully self‑contained event‑driven microservices demo** in **C# ASP.NET Core MVC**:  

- Gateway with **JWT auth via Microsoft Entra ID**.  
- Products with **Postgres + EF Core**.  
- Cart with **Redis + RabbitMQ publisher**.  
- Orders as a **RabbitMQ consumer + Postgres**.  
- Payments as optional **RabbitMQ consumer**.  
- Unified **observability: OpenTelemetry, Grafana, Tempo, Loki, Prometheus**.  
- Redis GUI via **RedisInsight**.  

Runs entirely on **Docker Compose** for local labs, but production‑ready patterns (async, event-driven, observable). 🚀
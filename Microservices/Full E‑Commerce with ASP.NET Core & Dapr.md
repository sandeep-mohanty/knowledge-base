# 🛍️ Comprehensive Tutorial: Event‑Driven E‑Commerce with ASP.NET Core & Dapr

---

## 1. What is Dapr?

**Dapr (Distributed Application Runtime)** is a CNCF project that provides **pluggable building blocks** for cloud‑native apps:

- **Service Invocation**: discover & call services via sidecars.  
- **State Management**: store/retrieve application state via state stores (Redis, Cosmos, Postgres).  
- **Pub/Sub Messaging**: publish events decoupled from messaging infra (RabbitMQ, Kafka, Azure Service Bus).  
- **Bindings**: trigger I/O from infra (DBs, queues).  
- **Observability**: integrated OpenTelemetry traces/logs/metrics.  

### Sidecar Pattern
Each service runs with a **Dapr sidecar** (container/process). The service calls its sidecar via HTTP/gRPC on `localhost:<port>`. The sidecar talks to Redis, RabbitMQ, Postgres, etc.

---

## 2. Architecture

### System Diagram

```
User → Gateway API (JWT Auth, Entra ID)
        │
        ├─► Products Service → Postgres (via EFCore) [or binding]
        ├─► Cart Service → Redis State (via Dapr state API)
        │       └─► Publishes CartCheckedOut → RabbitMQ (via Dapr pub/sub)
        ├─► Orders Service → Subscribes CartCheckedOut → Save Order Postgres
        │       └─► Publishes OrderCreated (RabbitMQ)
        └─► Payments Service → Subscribes OrderCreated → mark Paid
```

### Event‑Driven Checkout Flow

```
Cart → publish CartCheckedOut → RabbitMQ
  ↓
Orders (Dapr subscription)
  ↓
Persist Order (Postgres)
  ↓
Publish OrderCreated
  ↓
Payments consumes OrderCreated
```

---

## 3. Infrastructure (Docker Compose)

`docker-compose.yml`

```yaml
version: "3.9"
services:

  # Infra
  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: ecommerce
    ports: [ "5432:5432" ]

  redis:
    image: redis:7
    ports: [ "6379:6379" ]

  rabbitmq:
    image: rabbitmq:3-management
    ports: [ "5672:5672", "15672:15672" ]
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest

  redisinsight:
    image: redislabs/redisinsight:latest
    ports: [ "8001:8001" ]

  grafana:
    image: grafana/grafana:latest
    ports: [ "3000:3000" ]

  tempo:
    image: grafana/tempo:latest
    ports: [ "4318:4318" ]  # OTel OTLP endpoint

  loki:
    image: grafana/loki:2.9.0
    ports: ["3100:3100"]

  prometheus:
    image: prom/prometheus
    ports: ["9090:9090"]

  # Example microservice + dapr sidecar
  products:
    build: ./products
    ports: ["8081:80"]
  products-dapr:
    image: "daprio/daprd:latest"
    network_mode: "service:products"
    command: [
      "./daprd","-app-id","products",
      "-app-port","80",
      "-components-path","/components",
      "-config","/dapr/config.yaml"
    ]
    volumes:
      - ./components:/components
      - ./dapr/config.yaml:/dapr/config.yaml
```

⚠️ Repeat the pattern for `cart`, `orders`, `payments`, `gateway` → each service has a `-dapr` sidecar.

---

## 4. Dapr Components Configuration

Place in `./components/`.

**Redis State (statestore.yaml)**
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: redis:6379
  - name: redisPassword
    value: ""
```

**RabbitMQ Pub/Sub (pubsub.yaml)**
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: messagebus
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
  - name: host
    value: amqp://guest:guest@rabbitmq:5672
scopes:
- cart
- orders
- payments
```

**Postgres Binding (ordersdb.yaml)** (optional pattern)
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: ordersdb
spec:
  type: bindings.postgresql
  version: v1
  metadata:
  - name: connectionString
    value: Host=postgres;Port=5432;Database=ecommerce;Username=postgres;Password=postgres
```

---

## 5. Microservices (ASP.NET Core + Dapr)

### 5.1 Gateway

Validates JWT → uses Dapr service invocation.

```csharp
[ApiController]
[Route("[controller]")]
public class GatewayController : ControllerBase
{
    private readonly HttpClient _dapr;
    public GatewayController(IHttpClientFactory f){ _dapr=f.CreateClient(); _dapr.BaseAddress=new Uri("http://localhost:3500"); }

    [HttpGet("products")]
    [Authorize]   // Entra ID validated automatically
    public async Task<IActionResult> Products(){
        var resp=await _dapr.GetStringAsync("v1.0/invoke/products/method/products");
        return Content(resp,"application/json");
    }
}
```

---

### 5.2 Products API

Standard EF Core Postgres. Dapr optional here.  
Endpoint: `/products`.

---

### 5.3 Cart API (Dapr State + Pub/Sub)

```csharp
[ApiController]
[Route("[controller]")]
public class CartController : ControllerBase
{
    private readonly HttpClient _dapr;
    public CartController(IHttpClientFactory f){_dapr=f.CreateClient();_dapr.BaseAddress=new Uri("http://localhost:3500");}

    [HttpPost("add")]
    public async Task<IActionResult> Add(string userId,string product){
        var entry=new[]{new{key=$"cart:{userId}",value=product}};
        await _dapr.PostAsJsonAsync("v1.0/state/statestore", entry);
        return Ok();
    }

    [HttpPost("checkout")]
    public async Task<IActionResult> Checkout(string userId){
        var evt=new{UserId=userId,Timestamp=DateTime.UtcNow};
        await _dapr.PostAsJsonAsync("v1.0/publish/messagebus/cartCheckedOut", evt);
        return Accepted();
    }
}
```

---

### 5.4 Orders API (Consuming CartCheckedOut events)

Using **Dapr’s pub/sub subscription model**:

```csharp
[ApiController]
[Route("[controller]")]
public class OrdersController : ControllerBase
{
    private readonly OrdersDbContext _db;
    public OrdersController(OrdersDbContext db){_db=db;}

    [HttpPost("cartCheckedOut")]
    [Topic("messagebus","cartCheckedOut")] // Dapr attribute binds to pubsub
    public async Task<IActionResult> OnCartCheckedOut([FromBody] CartCheckoutEvent evt)
    {
        var order=new Order{UserId=evt.UserId,Status="PENDING"};
        _db.Orders.Add(order);
        await _db.SaveChangesAsync();
        // publish new event
        await DaprClient.CreateInvokeHttpClient("dapr").PostAsJsonAsync("v1.0/publish/messagebus/orderCreated",order);
        return Ok();
    }
}
public class CartCheckoutEvent{ public string UserId{get;set;} public DateTime Timestamp{get;set;} }
```

---

### 5.5 Payments API (Consumes OrderCreated)

```csharp
[ApiController]
[Route("[controller]")]
public class PaymentsController : ControllerBase
{
    [HttpPost("orderCreated")]
    [Topic("messagebus","orderCreated")]
    public IActionResult OnOrderCreated([FromBody]Order o){
        // Simulate payment
        return Ok(new {o.Id, Status="PAID"});
    }
}
```

---

## 6. Observability

- Dapr sidecars emit **OpenTelemetry traces/metrics/logs**.  
- Configure Grafana stack:  
  - Tempo receives OTLP traces from Dapr.  
  - Prometheus scrapes metrics.  
  - Loki collects logs (from containers).  
- View distributed traces directly in Grafana.

---

## 7. RedisInsight GUI

- Run at `http://localhost:8001`  
- Keys: `cart:<userid>` visible when add products.  

---

## 8. Running the App

### Step 1. Install Dapr
```bash
dapr init
```

### Step 2. Run Docker Compose
```bash
docker-compose up --build
```

This starts: infra, microservices, plus Dapr sidecars.

### Step 3. Verify
- Gateway → http://localhost:8080/products  
- Products → http://localhost:8081/products  
- Cart → http://localhost:8082/cart  
- Orders → http://localhost:8083/orders  
- Payments → http://localhost:8084/payments  
- RabbitMQ UI → http://localhost:15672  
- RedisInsight → http://localhost:8001  
- Grafana → http://localhost:3000  

---

## 9. Demo Flow

1. User logs in via Gateway (Entra ID JWT validated).  
2. Add products (`POST /products`).  
3. Add product to cart (`POST /cart/add`). → state stored via Dapr Redis.  
4. Checkout (`POST /cart/checkout`). → Dapr publishes pub/sub event.  
5. Orders service subscribes, creates PENDING order in Postgres. → publishes `orderCreated`.  
6. Payments service consumes, responds PAID.  
7. Grafana Tempo shows end‑to‑end distributed trace.  
8. RedisInsight shows cart data; RabbitMQ management shows pub/sub traffic.

---

## 10. Teardown

```bash
docker-compose down -v
```

---

## 🎓 Conclusion

You have successfully implemented the **E‑Commerce microservices application** using **Dapr** with:

- **ASP.NET Core MVC services**  
- **Dapr building blocks**:  
  - State management (Redis)  
  - Pub/Sub (RabbitMQ)  
  - Service invocation (Dapr HTTP/gRPC APIs)  
- **Infra**: Postgres, Redis, RabbitMQ  
- **Observability**: Grafana + Tempo + Loki + Prometheus  
- **GUI**: RedisInsight  

✅ The system runs cross‑platform (Linux/macOS/Windows) with **Docker Compose** and can later be **deployed to Kubernetes** seamlessly using Dapr’s same components.
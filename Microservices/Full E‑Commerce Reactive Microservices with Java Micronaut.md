# 🛍️ Full E‑Commerce Reactive Microservices with Java Micronaut  
**With Postgres (R2DBC), Redis, RabbitMQ, Microsoft Entra ID, OpenTelemetry, Grafana, Tempo, Loki, Prometheus, and Redis GUI**

---

## 1. Why Micronaut?

- **Ahead‑of‑time DI & AOP** → lightning fast startup, tiny memory footprint.  
- **Reactive HTTP** via Netty, built to scale.  
- **Micronaut Data R2DBC** → non‑blocking Postgres access.  
- **Micronaut Redis** → reactive Redis operations.  
- **Micronaut Security** → OAuth2 / JWT integration with Microsoft Entra ID.  
- **First‑class OpenTelemetry support** → instrumentation for distributed tracing.  

---

## 2. Architecture

(Same as before, different framework)

```
                           ┌──────────────┐
                           │   Frontend   │
                           └──────┬───────┘
                                  │
                        ┌─────────┴──────────┐
                        │ Gateway (Micronaut)│ ←→ Microsoft Entra ID (OAuth2/JWT)
                        └─────────┬──────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
 Products Svc (Micronaut + R2DBC) Cart Svc (Micronaut+Redis) Orders Svc (Micronaut+R2DBC)
                                  │                         │
                                  └───────── RabbitMQ ←─────┘
                                                            │
                                                    Payments Svc
Infrastructure: Postgres, Redis, RabbitMQ, RedisInsight  
Observability: OpenTelemetry → Tempo (traces), Loki (logs), Prometheus (metrics), Grafana dashboards  
```

---

## 3. Infrastructure: docker-compose

(same infra stack — Postgres, Redis, RabbitMQ, Grafana stack, RedisInsight). See earlier `docker-compose.yml` setup.

---

## 4. Micronaut Service Setup

Each service is a **Micronaut application**. Create via CLI:

```bash
mn create-app com.ecommerce.products --features=reactor,postgres-r2dbc,security-jwt,opentelemetry
```

---

### Shared Dependencies (build.gradle or pom.xml)

```xml
<dependencies>
  <!-- HTTP & JSON -->
  <dependency>
    <groupId>io.micronaut</groupId>
    <artifactId>micronaut-http-client</artifactId>
  </dependency>

  <!-- R2DBC Postgres -->
  <dependency>
    <groupId>io.micronaut.r2dbc</groupId>
    <artifactId>micronaut-r2dbc-core</artifactId>
  </dependency>
  <dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
  </dependency>

  <!-- Redis -->
  <dependency>
    <groupId>io.micronaut.redis</groupId>
    <artifactId>micronaut-redis-lettuce</artifactId>
  </dependency>

  <!-- RabbitMQ -->
  <dependency>
    <groupId>io.micronaut.rabbitmq</groupId>
    <artifactId>micronaut-rabbitmq</artifactId>
  </dependency>

  <!-- Security JWT/OAuth2 -->
  <dependency>
    <groupId>io.micronaut.security</groupId>
    <artifactId>micronaut-security-oauth2</artifactId>
  </dependency>

  <!-- OpenTelemetry -->
  <dependency>
    <groupId>io.micronaut.opentelemetry</groupId>
    <artifactId>micronaut-opentelemetry-exporter-otlp-http</artifactId>
  </dependency>
</dependencies>
```

---

## 5. Gateway Service (Micronaut + JWT Entra ID)

**application.yml**
```yaml
micronaut:
  application:
    name: gateway
  server:
    port: 8080
  security:
    enabled: true
    oauth2:
      clients:
        azure:
          client-id: ${ENTRA_CLIENT_ID}
          client-secret: ${ENTRA_CLIENT_SECRET}
          openid:
            issuer: https://login.microsoftonline.com/${ENTRA_TENANT_ID}/v2.0
      redirect-uri: http://localhost:8080/callback
```

**Controller**
```java
@Controller("/products")
public class GatewayController {
    private final RxHttpClient client;

    public GatewayController(@Client("http://products:8081") RxHttpClient client) {
        this.client = client;
    }

    @Get
    @Secured(SecurityRule.IS_AUTHENTICATED)
    public Publisher<String> listProducts(Authentication auth) {
        return client.retrieve(HttpRequest.GET("/products"), String.class);
    }
}
```

---

## 6. Products Service (Micronaut + R2DBC/Postgres)

**Entity**
```java
@MappedEntity("products")
public class Product {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private Double price;
    private Integer stock;
    // getters/setters
}
```

**Repository**
```java
@Repository
public interface ProductRepository extends ReactiveStreamsCrudRepository<Product, Long> {}
```

**Controller**
```java
@Controller("/products")
public class ProductController {
    private final ProductRepository repo;

    public ProductController(ProductRepository repo) { this.repo = repo; }

    @Get
    public Publisher<Product> all() { return repo.findAll(); }

    @Post
    public Publisher<Product> create(@Body Product p) { return repo.save(p); }
}
```

**application.yml**
```yaml
r2dbc:
  datasources:
    default:
      url: r2dbc:postgresql://postgres:5432/ecommerce
      username: postgres
      password: postgres
micronaut:
  server:
    port: 8081
```

---

## 7. Cart Service (Micronaut + Redis)

**Controller**
```java
@Controller("/cart")
public class CartController {
    private final RedisReactiveCommands<String, String> redis;
    private final HttpClient ordersClient;

    public CartController(RedisReactiveCommands<String,String> redis, 
                          @Client("http://orders:8083") HttpClient ordersClient) {
        this.redis = redis;
        this.ordersClient = ordersClient;
    }

    @Post("/add")
    public Publisher<String> add(@Body Map<String,String> body) {
        String key = "cart:" + body.get("userId");
        return redis.sadd(key, body.get("productId"))
                    .thenReturn("OK").toFlowable();
    }

    @Get("/{userId}")
    public Publisher<String> get(String userId) {
        return redis.smembers("cart:" + userId).toFlowable();
    }

    @Post("/checkout/{userId}")
    public Publisher<String> checkout(String userId) {
        return Flowable.fromPublisher(
            ordersClient.toBlocking().exchange(HttpRequest.POST("/cartCheckout?userId="+userId,""), String.class)
                         .getBody(String.class)
        );
    }
}
```

**application.yml**
```yaml
redis:
  uri: redis://redis:6379
micronaut:
  server:
    port: 8082
```

---

## 8. Orders Service (Micronaut + R2DBC)

**Entity**
```java
@MappedEntity("orders")
public class Order {
    @Id @GeneratedValue
    private Long id;
    private String userId;
    private String status;
}
```

**Repository**
```java
@Repository
public interface OrderRepository extends ReactiveStreamsCrudRepository<Order, Long> {}
```

**Controller**
```java
@Controller("/orders")
public class OrderController {
    private final OrderRepository repo;
    public OrderController(OrderRepository repo) { this.repo = repo; }

    @Post("/cartCheckout{?userId}")
    public Publisher<Order> checkout(String userId) {
        Order o = new Order();
        o.setUserId(userId);
        o.setStatus("PENDING");
        return repo.save(o);
    }

    @Get
    public Publisher<Order> list() { return repo.findAll(); }
}
```

**application.yml**
```yaml
r2dbc:
  datasources:
    default:
      url: r2dbc:postgresql://postgres:5432/ecommerce
      username: postgres
      password: postgres
micronaut:
  server:
    port: 8083
```

---

## 9. Payments Service (Micronaut)

**Controller**
```java
@Controller("/pay")
public class PaymentController {
    @Post
    public Map<String,Object> pay(@Body Map<String,Object> body) {
        return Map.of("orderId", body.get("orderId"), "status", "PAID");
    }
}
```

**application.yml**
```yaml
micronaut:
  server:
    port: 8084
```

---

## 10. Observability (Micronaut + OpenTelemetry)

Enable OTel exporter in application.yml for each service:

```yaml
otel:
  traces:
    exporter:
      otlp:
        endpoint: http://tempo:4318
```

Run with agent in Dockerfile:

```Dockerfile
FROM eclipse-temurin:17-jdk
WORKDIR /app
COPY build/libs/app-all.jar app.jar
COPY opentelemetry-javaagent.jar /opentelemetry-javaagent.jar
ENTRYPOINT ["java","-javaagent:/opentelemetry-javaagent.jar","-jar","/app/app.jar"]
```

---

## 11. Redis GUI

- Run RedisInsight → http://localhost:8001  
- Inspect keys: `cart:<userId>`.  

---

## 12. Running & Demo

```bash
./gradlew build
docker-compose up -d --build
```

Verify:
- Gateway http://localhost:8080/products  
- Products http://localhost:8081/products  
- Cart http://localhost:8082/cart/{userId}  
- Orders http://localhost:8083/orders  
- Payments http://localhost:8084/pay  
- Grafana http://localhost:3000 (admin/admin)  
- RabbitMQ UI http://localhost:15672  
- RedisInsight http://localhost:8001  

**Flow:**  
1. Authenticate user via Microsoft Entra ID → token validated by Gateway.  
2. Add products → `/products`.  
3. Add items to cart → Redis.  
4. Checkout → Orders persisted.  
5. Pay order → Payment returns `PAID`.  
6. Observe full traces in Tempo (Grafana), logs in Loki, metrics in Prometheus.  

Teardown:
```bash
docker-compose down -v
```

---

## 🎓 Conclusion

👏 You now have a **Micronaut‑based reactive microservice e‑commerce system**:  
- **Gateway**: JWT validation with Microsoft Entra ID  
- **Products**: R2DBC Postgres  
- **Cart**: Redis reactive client  
- **Orders**: R2DBC Postgres  
- **Payments**: Lightweight Micronaut controller  
- **Async MQ**: RabbitMQ (integration point)  
- **Observability**: OpenTelemetry instrumentation → Grafana, Tempo, Loki, Prometheus  
- **GUI**: RedisInsight for cart debugging  

This gives you **fast, lightweight, fully reactive microservices** with Micronaut, all observable and runnable in Docker 🚀.
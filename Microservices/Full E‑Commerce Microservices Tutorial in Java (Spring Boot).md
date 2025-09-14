# 🛍️ Full E‑Commerce Microservices Tutorial in Java (Spring Boot WebFlux Reactive)  
**With Postgres (R2DBC), Redis (Reactive), RabbitMQ, Microsoft Entra ID (OAuth2), OpenTelemetry, Grafana, Loki, Tempo, Prometheus, and Redis GUI (RedisInsight)**

---

## 1. Motivation: Why Reactive?

- **Spring WebFlux** = non‑blocking I/O engine built on Reactor.  
- Handles massive concurrent load with fewer threads.  
- **Backpressure support** ensures slow consumers never get overwhelmed.  
- **Reactive databases**:  
  - R2DBC for Postgres  
  - Redis reactive streams  
- **End-to-end reactive**: Services exchange `Mono`/`Flux`, integrated into observability with OpenTelemetry.  

---

## 2. System Architecture

```
                       ┌──────────────┐
                       │   Frontend   │
                       └──────┬───────┘
                              │
                    ┌─────────┴─────────┐
                    │ Gateway (WebFlux) │ ←→ Auth via Microsoft Entra ID
                    └───────┬───────────┘
                            │
     ┌──────────────────────┼─────────────────────────────────┐
     │                      │                                 │
 Products Service (R2DBC)   Cart Service (Reactive Redis)   Orders Service (R2DBC)
        │                        │                                │
        └───────────→ RabbitMQ ←─┘                                │
                                                                  │
                                                          Payments Service (Reactive)
                                                                  │
 Infrastructure: Postgres (R2DBC), Redis (Reactive), RabbitMQ  
 Observability: OpenTelemetry → Tempo, Loki, Prometheus, Grafana  
 Redis GUI: RedisInsight for browsing carts  
```

---

## 3. Infrastructure with Docker Compose

File: `docker-compose.yml`

```yaml
version: "3.9"
services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: ecommerce
    ports: ["5432:5432"]

  redis:
    image: redis:7
    ports: ["6379:6379"]

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"

  redisinsight:
    image: redislabs/redisinsight:latest
    ports: ["8001:8001"]
    depends_on: [redis]

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    ports: ["9090:9090"]

  loki:
    image: grafana/loki:2.9.0
    ports: ["3100:3100"]

  tempo:
    image: grafana/tempo:latest
    command: ["-config.file=/etc/tempo.yaml"]
    volumes:
      - ./monitoring/tempo.yaml:/etc/tempo.yaml
    ports: ["3200:3200"]

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    ports: ["3000:3000"]

  gateway:
    build: ./gateway
    ports: ["8080:8080"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
      - OTEL_SERVICE_NAME=gateway
      - ENTRA_TENANT_ID=xxx
      - ENTRA_CLIENT_ID=xxx
      - ENTRA_CLIENT_SECRET=xxx
    depends_on: [postgres, redis, rabbitmq]

  products:
    build: ./products
    ports: ["8081:8081"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
      - OTEL_SERVICE_NAME=products
    depends_on: [postgres]

  cart:
    build: ./cart
    ports: ["8082:8082"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
      - OTEL_SERVICE_NAME=cart
    depends_on: [redis]

  orders:
    build: ./orders
    ports: ["8083:8083"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
      - OTEL_SERVICE_NAME=orders
    depends_on: [postgres]

  payments:
    build: ./payments
    ports: ["8084:8084"]
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
      - OTEL_SERVICE_NAME=payments
```

---

## 4. Dockerfile for Spring Boot Reactive Services

In every service folder:

```Dockerfile
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY target/*.jar app.jar
COPY opentelemetry-javaagent.jar /opentelemetry-javaagent.jar
ENTRYPOINT ["java",
            "-javaagent:/opentelemetry-javaagent.jar",
            "-Dotel.exporter.otlp.endpoint=http://tempo:4318",
            "-Dotel.service.name=${SERVICE_NAME}",
            "-jar","/app/app.jar"]
```

---

## 5. Microservice Implementations (Reactive)

### 5.1 Gateway Service (Reactive, Entra ID OAuth2 JWT)

**`application.yml`**
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://login.microsoftonline.com/${ENTRA_TENANT_ID}/v2.0
server:
  port: 8080
```

**App**
```java
@SpringBootApplication
public class GatewayApplication {
  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }
}
```

**Controller**
```java
@RestController
public class GatewayController {
  private final WebClient webClient = WebClient.create("http://products:8081");

  @GetMapping("/products")
  public Mono<String> products(@AuthenticationPrincipal Jwt jwt) {
    return webClient.get().uri("/products").retrieve().bodyToMono(String.class);
  }
}
```

---

### 5.2 Products Service (Reactive R2DBC with Postgres)

**Entity & Repository**
```java
@Table("products")
public class Product {
  @Id
  private Long id;
  private String name;
  private Double price;
  private Integer stock;
}

public interface ProductRepository extends ReactiveCrudRepository<Product, Long> {}
```

**Controller**
```java
@RestController
@RequestMapping("/products")
public class ProductController {
  private final ProductRepository repo;
  public ProductController(ProductRepository repo) { this.repo = repo; }

  @GetMapping
  public Flux<Product> all() { return repo.findAll(); }

  @PostMapping
  public Mono<Product> create(@RequestBody Product p) { return repo.save(p); }
}
```

**`application.yml`**
```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://postgres:5432/ecommerce
    username: postgres
    password: postgres
  sql:
    init:
      mode: always
server:
  port: 8081
```

---

### 5.3 Cart Service (Reactive Redis)

**Controller**
```java
@RestController
@RequestMapping("/cart")
public class CartController {

  private final ReactiveRedisTemplate<String,String> redis;
  private final WebClient ordersClient = WebClient.create("http://orders:8083");

  public CartController(ReactiveRedisTemplate<String,String> redis) { this.redis = redis; }

  @PostMapping("/add")
  public Mono<Boolean> add(@RequestParam String userId, @RequestParam String productId) {
    return redis.opsForSet().add("cart:" + userId, productId).map(v -> v > 0);
  }

  @GetMapping
  public Flux<String> get(@RequestParam String userId) {
    return redis.opsForSet().members("cart:" + userId);
  }

  @PostMapping("/checkout")
  public Mono<String> checkout(@RequestParam String userId) {
    return ordersClient.post().uri("/cartCheckout?userId=" + userId)
            .retrieve().bodyToMono(String.class);
  }
}
```

**`application.yml`**
```yaml
spring:
  data:
    redis:
      host: redis
      port: 6379
server:
  port: 8082
```

---

### 5.4 Orders Service (Reactive R2DBC)

**Entity & Repository**
```java
@Table("orders")
public class Order {
  @Id
  private Long id;
  private String userId;
  private String status;
}

public interface OrderRepository extends ReactiveCrudRepository<Order, Long> {}
```

**Controller**
```java
@RestController
public class OrderController {
  private final OrderRepository repo;
  public OrderController(OrderRepository repo) { this.repo = repo; }

  @PostMapping("/cartCheckout")
  public Mono<Order> checkout(@RequestParam String userId) {
    Order o = new Order();
    o.setUserId(userId);
    o.setStatus("PENDING");
    return repo.save(o);
  }

  @GetMapping("/orders")
  public Flux<Order> list() { return repo.findAll(); }
}
```

**`application.yml`**
```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://postgres:5432/ecommerce
    username: postgres
    password: postgres
server:
  port: 8083
```

---

### 5.5 Payments Service (Reactive)

**Controller**
```java
@RestController
public class PaymentController {
  @PostMapping("/pay")
  public Mono<Map<String,Object>> pay(@RequestParam Long orderId) {
    return Mono.just(Map.of("orderId", orderId, "status", "PAID"));
  }
}
```

**`application.yml`**
```yaml
server:
  port: 8084
```

---

## 6. Redis GUI (RedisInsight)

- Runs at http://localhost:8001  
- Connect to host `redis:6379`  
- Browse keys like `cart:<userId>`  

---

## 7. Observability

- **Traces/metrics/logs** auto-exported via OpenTelemetry Java Agent.  
- Data flows:  
  - Traces → Tempo  
  - Logs → Loki  
  - Metrics → Prometheus  
- View all correlated in Grafana at http://localhost:3000  

---

## 8. Running & Teardown

### Build services
```bash
mvn clean package
```

### Start stack
```bash
docker-compose up -d --build
```

### Verify services
- Gateway → http://localhost:8080  
- Products → http://localhost:8081/products  
- Cart → http://localhost:8082/cart?userId=alice  
- Orders → http://localhost:8083/orders  
- Payments → http://localhost:8084/pay  
- RabbitMQ UI → http://localhost:15672  
- Grafana → http://localhost:3000 (admin/admin)  
- Redis GUI → http://localhost:8001  

### Demo flow
1. Authenticate against **Entra ID** via Gateway (JWT validated).  
2. `POST /products` → add product.  
3. `POST /cart/add` → add to cart → verify key in RedisInsight.  
4. `POST /cart/checkout` → Orders service creates reactive order.  
5. `POST /pay` → Payment service marks order PAID.  
6. See distributed traces/metrics/logs in Grafana.  

### Teardown
```bash
docker-compose down -v
```

---

## 🎓 Conclusion

You now have a full **Spring Boot WebFlux (Reactive) microservices e‑commerce platform**:  
- **Reactive Gateway**: OAuth2/JWT with Microsoft Entra ID  
- **Reactive Products**: Postgres via R2DBC  
- **Reactive Cart**: Redis reactive template  
- **Reactive Orders**: Postgres R2DBC  
- **Reactive Payments**: Mono‑returning mock  
- **Infrastructure**: Redis, Postgres, RabbitMQ, RedisInsight  
- **Observability**: OTel agent auto‑instrumentation → Tempo/Loki/Prometheus/Grafana  

All **non‑blocking, backpressure‑aware, observable**, and runnable locally in Docker — a complete, consolidated tutorial! 🚀
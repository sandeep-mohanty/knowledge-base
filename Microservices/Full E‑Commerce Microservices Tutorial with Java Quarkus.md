# 🛍️ Full E‑Commerce Microservices Tutorial with Java Quarkus  
**With Postgres (Reactive), Redis, RabbitMQ, Microsoft Entra ID (OIDC), OpenTelemetry, Grafana, Tempo, Loki, Prometheus, and Redis GUI**

---

## 1. Why Quarkus?

- **Built for Cloud & K8s** → low memory, quick startup, perfect for containers.  
- **Supersonic Subatomic Java**: runs great in JVM or compiled to native binary.  
- **Extensions** for reactive DB, Redis, messaging, security, telemetry.  
- **Kubernetes Ready** but also **Docker‑Compose Friendly** — perfect for our tutorials.  

---

## 2. Architecture

```
                           ┌──────────────┐
                           │   Frontend   │
                           └──────┬───────┘
                                  │
                        ┌─────────┴──────────┐
                        │ Gateway (Quarkus) │ ←→ Microsoft Entra ID (OIDC/JWT)
                        └─────────┬──────────┘
                                  │
        ┌─────────────────────────┼────────────────────────┐
        │                         │                        │
 Products Svc (Quarkus+PgReactive) Cart Svc (Quarkus+Redis) Orders Svc (Quarkus+PgReactive)
                                  │                        │
                                  └────── RabbitMQ ────────┘
                                                           │
                                                   Payments Svc
Infra: Postgres, Redis, RabbitMQ, Grafana, Tempo, Loki, Prometheus, RedisInsight
```

---

## 3. Infrastructure

👉 Exactly same `docker-compose.yml` file as Spring/Micronaut/Vert.x stack to ensure consistency.

---

## 4. Quarkus Project Setup

Generate projects with Quarkus CLI or code.quarkus.io.

```bash
quarkus create app com.ecommerce:gateway -x resteasy-reactive,oidc,smallrye-opentelemetry
quarkus create app com.ecommerce:products -x resteasy-reactive,hibernate-reactive-panache,reactive-pg-client,smallrye-opentelemetry
quarkus create app com.ecommerce:cart -x resteasy-reactive,quarkus-redis-client,smallrye-opentelemetry
quarkus create app com.ecommerce:orders -x resteasy-reactive,hibernate-reactive-panache,reactive-pg-client,smallrye-opentelemetry
quarkus create app com.ecommerce:payments -x resteasy-reactive,smallrye-opentelemetry
```

---

## 5. Gateway Service (Quarkus OIDC)

**application.properties**
```properties
quarkus.http.port=8080

quarkus.oidc.auth-server-url=https://login.microsoftonline.com/${ENTRA_TENANT_ID}/v2.0
quarkus.oidc.client-id=${ENTRA_CLIENT_ID}
quarkus.oidc.credentials.secret=${ENTRA_CLIENT_SECRET}
quarkus.oidc.application-type=web-app
quarkus.oidc.token.audience=${ENTRA_CLIENT_ID}
```

**GatewayResource.java**
```java
@Path("/products")
@Authenticated
public class GatewayResource {

    @RestClient
    ProductsService productsService;

    @GET
    public Uni<String> proxyProducts(@Context SecurityIdentity securityIdentity) {
        // Token is already verified by Quarkus OIDC
        return productsService.getProducts();
    }
}

@RegisterRestClient(baseUri = "http://products:8081")
interface ProductsService {
    @GET
    @Path("/products")
    Uni<String> getProducts();
}
```

---

## 6. Products Service (Quarkus Reactive Postgres)

**application.properties**
```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=postgres
quarkus.datasource.password=postgres
quarkus.datasource.reactive.url=postgresql://postgres:5432/ecommerce
quarkus.hibernate-orm.database.generation=update
quarkus.http.port=8081
```

**Entity**
```java
@Entity
public class Product extends PanacheEntity {
    public String name;
    public double price;
    public int stock;
}
```

**Resource**
```java
@Path("/products")
public class ProductResource {

    @GET
    public Uni<List<Product>> all() {
        return Product.listAll();
    }

    @POST
    public Uni<Product> add(Product p) {
        return Panache.withTransaction(p::persist).replaceWith(p);
    }
}
```

---

## 7. Cart Service (Quarkus Redis)

**application.properties**
```properties
quarkus.redis.hosts=redis://redis:6379
quarkus.http.port=8082
```

**Resource**
```java
@Path("/cart")
public class CartResource {

    @Inject
    RedisAPI redis;

    @POST
    @Path("/add")
    public Uni<String> add(@QueryParam("userId") String userId, @QueryParam("productId") String productId) {
        return redis.sadd(Arrays.asList("cart:" + userId, productId)).replaceWith("OK");
    }

    @GET
    public Uni<List<String>> get(@QueryParam("userId") String userId) {
        return redis.smembers(List.of("cart:" + userId))
                    .map(resp -> resp.stream().map(r -> r.toString()).collect(Collectors.toList()));
    }

    @POST
    @Path("/checkout")
    public Uni<String> checkout(@QueryParam("userId") String userId) {
        return WebClient.create().postAbs("http://orders:8083/cartCheckout?userId=" + userId)
                        .send().onItem().transform(Buffer::toString);
    }
}
```

---

## 8. Orders Service (Quarkus Reactive + Postgres)

**Entity**
```java
@Entity
public class Order extends PanacheEntity {
    public String userId;
    public String status;
}
```

**Resource**
```java
@Path("/orders")
public class OrderResource {

    @POST
    @Path("/cartCheckout")
    public Uni<Order> checkout(@QueryParam("userId") String userId) {
        Order o = new Order();
        o.userId = userId;
        o.status = "PENDING";
        return Panache.withTransaction(() -> o.persist()).replaceWith(o);
    }

    @GET
    public Uni<List<Order>> all() {
        return Order.listAll();
    }
}
```

**application.properties**
```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.reactive.url=postgresql://postgres:5432/ecommerce
quarkus.datasource.username=postgres
quarkus.datasource.password=postgres
quarkus.http.port=8083
```

---

## 9. Payments Service (Quarkus RESTEasy Reactive)

**Resource**
```java
@Path("/pay")
public class PaymentResource {
    @POST
    public Map<String,Object> pay(Map<String,Object> body) {
        return Map.of("orderId", body.get("orderId"), "status", "PAID");
    }
}
```

**application.properties**
```properties
quarkus.http.port=8084
```

---

## 10. Observability (Quarkus OTel)

Enable OpenTelemetry exporter:

**application.properties**
```properties
quarkus.opentelemetry.enabled=true
quarkus.opentelemetry.tracer.exporter.otlp.endpoint=http://tempo:4318
```

Every service auto-exports traces; logs go via Docker’s Loki driver; metrics exposed to Prometheus.

---

## 11. Redis GUI

- RedisInsight at http://localhost:8001  
- Inspect keys `cart:<userId>`  

---

## 12. Running & Demo

### Build
```bash
./mvnw package
```

### Start
```bash
docker-compose up -d --build
```

### Verify
- Gateway → http://localhost:8080/products  
- Products → http://localhost:8081/products  
- Cart → http://localhost:8082/cart?userId=alice  
- Orders → http://localhost:8083/orders  
- Payments → http://localhost:8084/pay  
- Grafana → http://localhost:3000  
- RabbitMQ UI → http://localhost:15672  
- Redis GUI → http://localhost:8001  

### Demo Flow
1. Authenticate with Microsoft Entra ID token at Gateway.  
2. Add products to catalog.  
3. Add product to cart → Redis (visible in RedisInsight).  
4. Checkout → Order persisted to Postgres.  
5. Pay → Payment service returns PAID.  
6. Observe distributed traces, logs, metrics in Grafana suite.  

### Teardown
```bash
docker-compose down -v
```

---

## 🎓 Conclusion

You now have the same **reactive microservice e‑commerce system** but built with **Quarkus**:  

- **Gateway**: OIDC + JWT validation with Entra ID.  
- **Products**: Reactive Postgres with Panache (R2DBC).  
- **Cart**: Quarkus Redis Client.  
- **Orders**: Reactive Postgres.  
- **Payments**: RESTEasy Reactive.  
- **Observability**: OpenTelemetry → Tempo, Loki, Prometheus, Grafana.  
- **Redis GUI**: RedisInsight.  
- **Deployable**: via Docker Compose locally, or easily to Kubernetes later.  

✅ Even though Quarkus is “Kubernetes‑native,” you can confidently run it in **plain Docker‑Compose** for local demos, just like Spring Boot or Micronaut. Later, scaling to Kubernetes is trivial.  
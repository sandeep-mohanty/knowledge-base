# 🛍️ Full E‑Commerce Microservices in Java with Akka (Actor Model)

**With Postgres, Redis, RabbitMQ, Microsoft Entra ID (JWT), OpenTelemetry, Grafana, Tempo, Loki, Prometheus, and RedisInsight**

---

## 1. Why Akka?

- **Actor model**: Each service’s logic encapsulated in actors → easier concurrency & resilience.  
- **Akka Streams**: Backpressure-aware streaming of data/events.  
- **Akka HTTP**: Lightweight REST endpoints with actor backend.  
- **Akka Persistence & Akka Typed**: Great for reliability (optional).  
- Natural fit for **event-driven microservices**, RabbitMQ → actors → DB.  

---

## 2. System Architecture

```
Client → Gateway (Akka HTTP) 
         │ Validates JWT with Entra ID
         ▼
   ┌───────────────────────────┐
   │         Actor System       │
   │   ProductsActor  (Postgres)
   │   CartActor      (Redis)
   │   OrdersActor    (Postgres)
   │   PaymentsActor  (Mock/RabbitMQ)
   └───────────────────────────┘
Infra: Postgres, Redis, RabbitMQ, Grafana stack, RedisInsight
```

---

## 3. Infrastructure  

👉 We reuse the **same docker-compose.yml** from previous tutorials (Postgres, Redis, RabbitMQ, Grafana, Tempo, Loki, Prometheus, RedisInsight + our Akka services in Docker).

---

## 4. Dependencies (Maven)

```xml
<dependencies>
  <dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor-typed_2.13</artifactId>
    <version>2.8.0</version>
  </dependency>
  <dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-stream_2.13</artifactId>
    <version>2.8.0</version>
  </dependency>
  <dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-http_2.13</artifactId>
    <version>10.5.0</version>
  </dependency>
  <dependency>
    <groupId>com.lightbend.akka</groupId>
    <artifactId>akka-http-jackson_2.13</artifactId>
    <version>10.5.0</version>
  </dependency>
  <!-- Postgres driver, Lettuce Redis, RabbitMQ Java client -->
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.6.0</version>
  </dependency>
  <dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>6.2.4</version>
  </dependency>
  <dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.17.0</version>
  </dependency>
  <!-- OpenTelemetry + Akka instrumentation -->
</dependencies>
```

---

## 5. Actor System

Each domain actor encapsulates state & logic.

### ProductActor
```java
public class ProductActor extends AbstractBehavior<ProductActor.Command> {
    interface Command {}
    public static final class GetProducts implements Command {
        public final ActorRef<List<Product>> replyTo;
        public GetProducts(ActorRef<List<Product>> r) { this.replyTo = r; }
    }
    public static final class AddProduct implements Command {
        public final Product product;
        public final ActorRef<Product> replyTo;
        public AddProduct(Product p, ActorRef<Product> r) { this.product = p; this.replyTo = r; }
    }

    public static Behavior<Command> create() {
        return Behaviors.setup(ProductActor::new);
    }

    private ProductActor(ActorContext<Command> ctx) { super(ctx); }

    @Override
    public Receive<Command> createReceive() {
        return newReceiveBuilder()
            .onMessage(GetProducts.class, this::onGet)
            .onMessage(AddProduct.class, this::onAdd)
            .build();
    }

    private Behavior<Command> onGet(GetProducts msg) {
        // query Postgres asynchronously (DAO call omitted)
        msg.replyTo.tell(fetchFromDb());
        return this;
    }

    private Behavior<Command> onAdd(AddProduct msg) {
        // insert into Postgres (DAO call omitted)
        saveToDb(msg.product);
        msg.replyTo.tell(msg.product);
        return this;
    }
}
```

### CartActor
```java
public class CartActor extends AbstractBehavior<CartActor.Command> {
    interface Command {}
    public static final class AddToCart implements Command {
        public final String userId, productId;
        public final ActorRef<String> replyTo;
        public AddToCart(String u, String p, ActorRef<String> r) { userId=u; productId=p; replyTo=r; }
    }
    public static final class GetCart implements Command {
        public final String userId;
        public final ActorRef<List<String>> replyTo;
        public GetCart(String u, ActorRef<List<String>> r) { userId=u; replyTo=r; }
    }

    public static Behavior<Command> create() { return Behaviors.setup(CartActor::new); }
    private CartActor(ActorContext<Command> ctx) { super(ctx); }

    @Override
    public Receive<Command> createReceive() {
        return newReceiveBuilder()
          .onMessage(AddToCart.class, this::onAdd)
          .onMessage(GetCart.class, this::onGet)
          .build();
    }

    private Behavior<Command> onAdd(AddToCart msg) {
        // use Redis lettuce async client
        // redis.sadd("cart:"+msg.userId, msg.productId);
        msg.replyTo.tell("OK");
        return this;
    }

    private Behavior<Command> onGet(GetCart msg) {
        List<String> items = fetchFromRedis(msg.userId);
        msg.replyTo.tell(items);
        return this;
    }
}
```

### OrdersActor
```java
public class OrdersActor extends AbstractBehavior<OrdersActor.Command> {
    interface Command {}
    public static final class Checkout implements Command {
        public final String userId;
        public final ActorRef<Order> replyTo;
        public Checkout(String u, ActorRef<Order> r) { userId=u; replyTo=r; }
    }

    public static Behavior<Command> create() { return Behaviors.setup(OrdersActor::new); }
    private OrdersActor(ActorContext<Command> ctx) { super(ctx); }

    @Override
    public Receive<Command> createReceive() {
        return newReceiveBuilder()
           .onMessage(Checkout.class, this::onCheckout)
           .build();
    }

    private Behavior<Command> onCheckout(Checkout msg) {
        Order o = new Order();
        o.userId = msg.userId;
        o.status = "PENDING";
        saveToPostgres(o);
        msg.replyTo.tell(o);
        return this;
    }
}
```

### PaymentsActor
```java
public class PaymentsActor extends AbstractBehavior<PaymentsActor.Command> {
    interface Command {}
    public static final class Pay implements Command {
        public final String orderId;
        public final ActorRef<String> replyTo;
        public Pay(String id, ActorRef<String> r){orderId=id; replyTo=r;}
    }

    public static Behavior<Command> create() { return Behaviors.setup(PaymentsActor::new); }
    private PaymentsActor(ActorContext<Command> ctx){super(ctx);}
    @Override
    public Receive<Command> createReceive() {
        return newReceiveBuilder()
          .onMessage(Pay.class, this::onPay)
          .build();
    }
    private Behavior<Command> onPay(Pay cmd){
        cmd.replyTo.tell("PAID:"+cmd.orderId);
        return this;
    }
}
```

---

## 6. Akka HTTP Routes

Each service binds an HTTP route, translates requests → messages to actors → responses.

Example (ProductsService):

```java
public class ProductRoutes {
    private final ActorRef<ProductActor.Command> productActor;
    private final Timeout timeout = Timeout.create(Duration.ofSeconds(5));

    public ProductRoutes(ActorRef<ProductActor.Command> ref){this.productActor=ref;}

    public Route routes() {
        return path("products", () ->
            concat(
                get(() ->
                    onSuccess(
                        AskPattern.ask(productActor,
                          replyTo -> new ProductActor.GetProducts(replyTo),
                          timeout, Scheduler.global),
                        products -> completeOK(products, Jackson.marshaller())
                    )
                ),
                post(() ->
                    entity(Jackson.unmarshaller(Product.class), p ->
                        onSuccess(
                          AskPattern.ask(productActor,
                            replyTo -> new ProductActor.AddProduct(p, replyTo),
                            timeout, Scheduler.global),
                          prod -> completeOK(prod, Jackson.marshaller())
                        )
                    )
                )
            )
        );
    }
}
```

Similarly, **CartRoutes**, **OrdersRoutes**, **PaymentsRoutes** map HTTP → actor communication.

---

## 7. Gateway Service (Akka HTTP)

- Add a route `/products` that validates Authorization header (JWT from Entra ID — validate signature against JWKS via jose4j).  
- Forward to Products service (Http().singleRequest).  
- For simplicity, local demo: skip full OIDC handshake.

---

## 8. Observability

- Attach OpenTelemetry SDK (io.opentelemetry API + Akka instrumentation).  
- Configure environment variables:  
```properties
OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318
OTEL_SERVICE_NAME=products
```
- Docker images include `-javaagent:opentelemetry-javaagent.jar` for automatic instrumentation.  

---

## 9. RedisInsight GUI

- Same as before → http://localhost:8001 → view user carts.

---

## 10. Running

### Build
```bash
mvn clean package
```

### Start
```bash
docker-compose up -d --build
```

---

## 11. Demo Flow

1. **Add Product** → `/products POST`. Saved in Postgres by ProductActor.  
2. **Add to Cart** → `/cart/add`. CartActor stores in Redis.  
3. **Get Cart** → `/cart?userId=alice`. Redis shows IDs; RedisInsight confirms.  
4. **Checkout** → `/orders/cartCheckout`. OrdersActor persists order → Postgres.  
5. **Pay** → `/pay`. PaymentsActor responds `PAID`.  
6. **Observe** → Logs to Loki, Traces to Tempo, Grafana dashboard aggregates.  

---

## 12. Conclusion

🎉 You now have an **Akka-based actor microservice system** implementing the same e‑commerce architecture:

- **Actors** as microservice domain logic (ProductsActor, CartActor, OrdersActor, PaymentsActor).  
- **Akka HTTP** exposes endpoints.  
- **Postgres (JDBC async)** via DAOs.  
- **Redis (Lettuce async)** via CartActor.  
- **RabbitMQ** possible via AMQP client (actors consume/publish messages).  
- **OpenTelemetry** instrumentation to Tempo & Grafana.  
- **Redis GUI** with RedisInsight.  

The **Actor Model** brings resilience, supervision strategies, and backpressure (via Akka Streams). Even though we simulate microservices, each bounded context can run in its own container/actor system.  

✅ This Akka version complements the Spring / Micronaut / Vert.x / Quarkus variants — now you’ve seen the full spectrum of Java microservice frameworks! 🚀
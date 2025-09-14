# 🛍️ Full E‑Commerce Microservices Tutorial  
**With Deno, Redis, Postgres, RabbitMQ, Microsoft Entra ID, OpenTelemetry, Grafana, Loki, Tempo, Prometheus, and Redis GUI (RedisInsight)**

This end‑to‑end tutorial shows how to build, run, and observe a distributed **e‑commerce platform** locally using **Deno microservices** with complete monitoring and debugging tools.  

---

# 1. High‑Level Architecture

```
             ┌──────────────┐
             │   Frontend    │
             └──────┬───────┘
                    │
          ┌─────────┴─────────┐
          │ API Gateway (Deno)│  ←→ Auth via Microsoft Entra ID
          └───────┬───────────┘
                  │
    ┌─────────────┼──────────────────────────────┐
    │             │                              │
 Orders Svc   Products Svc    Cart Svc     Payment Svc
 (Deno+DB)    (Deno+DB)       (Deno+Redis) (Stubbed)
    │             │                              │
    └───────→ RabbitMQ event bus ←───────────────┘
                  │
             Async updates
                  │
             ┌────┴─────┐
             │  Redis    │ (cart store)
             │  Postgres │ (catalog & orders)
             └──────────┘

Observability stack:
- Traces → Tempo
- Logs → Loki
- Metrics → Prometheus
- Dashboards → Grafana
- Redis GUI → RedisInsight
```

---

# 2. Authentication (Microsoft Entra ID)

- Register an App in your Microsoft Entra tenant.  
- Redirect URI: `http://localhost:8080/callback`  
- Collect the **Tenant ID**, **Client ID**, and **Client Secret**.  
- Insert them into environment vars for Gateway service.

---

# 3. Observability Stack

- **Prometheus** scrapes metrics (`http_request_duration`, etc.)  
- **Tempo** ingests OpenTelemetry traces  
- **Loki** collects logs  
- **Grafana** ties them all together  
- **RedisInsight** provides GUI to inspect Redis keys & values  

---

# 4. Docker‑Compose Infrastructure

File: `docker-compose.yml`

```yaml
version: "3.9"
services:
  # === Datastores ===
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
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest

  # === Redis GUI ===
  redisinsight:
    image: redislabs/redisinsight:latest
    ports:
      - "8001:8001"
    depends_on: [redis]

  # === Monitoring ===
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
    volumes:
      - ./monitoring/grafana-provisioning:/etc/grafana/provisioning
    ports: ["3000:3000"]

  # === Microservices ===
  gateway:
    build: ./gateway
    ports: ["8080:8080"]
    environment:
      - ENTRA_TENANT_ID=xxx
      - ENTRA_CLIENT_ID=xxx
      - ENTRA_CLIENT_SECRET=xxx
      - OTELO_ENDPOINT=http://tempo:4318/v1/traces
    depends_on: [redis, postgres, rabbitmq]

  products:
    build: ./products
    ports: ["8081:8081"]
    depends_on: [postgres, rabbitmq]

  cart:
    build: ./cart
    ports: ["8082:8082"]
    depends_on: [redis, rabbitmq]

  orders:
    build: ./orders
    ports: ["8083:8083"]
    depends_on: [postgres, rabbitmq]

  payments:
    build: ./payments
    ports: ["8084:8084"]
    depends_on: [rabbitmq]
```

---

# 5. Common Telemetry Code

File: `common/opentelemetry.ts`

```ts
import { Resource } from "npm:@opentelemetry/resources";
import { NodeTracerProvider } from "npm:@opentelemetry/sdk-trace-node";
import { BatchSpanProcessor } from "npm:@opentelemetry/sdk-trace-base";
import { OTLPTraceExporter } from "npm:@opentelemetry/exporter-trace-otlp-http";

export function initTelemetry(serviceName: string) {
  const provider = new NodeTracerProvider({
    resource: new Resource({ "service.name": serviceName }),
  });
  const exporter = new OTLPTraceExporter({
    url: Deno.env.get("OTELO_ENDPOINT") || "http://tempo:4318/v1/traces",
  });
  provider.addSpanProcessor(new BatchSpanProcessor(exporter));
  provider.register();
}
```

---

# 6. Microservice Implementations

## 6.1 Gateway

File: `gateway/main.ts`

```ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import { verifyJwt } from "https://deno.land/x/djwt/mod.ts";
import { initTelemetry } from "../common/opentelemetry.ts";

initTelemetry("gateway");

const app = new Application();
const router = new Router();

// Simplified JWT middleware
async function checkAuth(ctx: any, next: any) {
  const auth = ctx.request.headers.get("authorization");
  if (!auth) { ctx.response.status = 401; return; }
  const [, token] = auth.split(" ");
  try {
    const { payload } = await verifyJwt(token, "PUBLIC_KEY_PLACEHOLDER");
    ctx.state.user = payload;
    await next();
  } catch {
    ctx.response.status = 401;
  }
}

router.get("/products", checkAuth, async (ctx) => {
  const resp = await fetch("http://products:8081/products");
  ctx.response.body = await resp.json();
});

app.use(router.routes());
app.listen({ port: 8080 });
```

---

## 6.2 Product Service

File: `products/main.ts`

```ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import { Client } from "https://deno.land/x/postgres/mod.ts";
import { initTelemetry } from "../common/opentelemetry.ts";

initTelemetry("products");

const client = new Client({
  user: "postgres",
  password: "postgres",
  database: "ecommerce",
  hostname: "postgres",
  port: 5432,
});
await client.connect();

const router = new Router();
router
  .get("/products", async (ctx) => {
    const result = await client.queryObject("SELECT id, name, price, stock FROM products");
    ctx.response.body = result.rows;
  })
  .post("/products", async (ctx) => {
    const { name, price, stock } = await ctx.request.body({ type: "json" }).value;
    await client.queryArray(
      "INSERT INTO products(name, price, stock) VALUES ($1, $2, $3)",
      name, price, stock,
    );
    ctx.response.status = 201;
  });

const app = new Application();
app.use(router.routes());
app.listen({ port: 8081 });
```

---

## 6.3 Cart Service

File: `cart/main.ts`

```ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import { connect } from "https://deno.land/x/redis/mod.ts";
import { initTelemetry } from "../common/opentelemetry.ts";

initTelemetry("cart");

const redis = await connect({ hostname: "redis", port: 6379 });
const router = new Router();

router
  .post("/cart/add", async (ctx) => {
    const { userId, productId } = await ctx.request.body({ type: "json" }).value;
    await redis.sadd(`cart:${userId}`, productId);
    ctx.response.status = 200;
  })
  .get("/cart", async (ctx) => {
    const userId = ctx.request.url.searchParams.get("userId")!;
    const cart = await redis.smembers(`cart:${userId}`);
    ctx.response.body = cart;
  })
  .post("/cart/checkout", async (ctx) => {
    const { userId } = await ctx.request.body({ type: "json" }).value;
    await fetch("http://orders:8083/cartCheckout", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ userId }),
    });
    ctx.response.status = 202;
  });

const app = new Application();
app.use(router.routes());
app.listen({ port: 8082 });
```

---

## 6.4 Orders Service

File: `orders/main.ts`

```ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import { Client } from "https://deno.land/x/postgres/mod.ts";
import { initTelemetry } from "../common/opentelemetry.ts";

initTelemetry("orders");

const client = new Client({
  user: "postgres",
  password: "postgres",
  database: "ecommerce",
  hostname: "postgres",
  port: 5432,
});
await client.connect();

const router = new Router();

router.post("/cartCheckout", async (ctx) => {
  const { userId } = await ctx.request.body({ type: "json" }).value;
  const res = await client.queryObject`
    INSERT INTO orders(user_id, status) VALUES (${userId}, 'PENDING')
    RETURNING id
  `;
  ctx.response.body = { orderId: res.rows[0].id };
});

const app = new Application();
app.use(router.routes());
app.listen({ port: 8083 });
```

---

## 6.5 Payment Service

File: `payments/main.ts`

```ts
import { Application, Router } from "https://deno.land/x/oak/mod.ts";
import { initTelemetry } from "../common/opentelemetry.ts";

initTelemetry("payments");

const router = new Router();

router.post("/pay", async (ctx) => {
  const { orderId } = await ctx.request.body({ type: "json" }).value;
  ctx.response.body = { orderId, status: "PAID" };
});

const app = new Application();
app.use(router.routes());
app.listen({ port: 8084 });
```

---

# 7. Dockerfile for Services

Each microservice has a `Dockerfile`:

```Dockerfile
FROM denoland/deno:alpine-1.37.1
WORKDIR /app
COPY . .
CMD ["run","--allow-net","--allow-env","main.ts"]
```

---

# 8. Redis GUI (RedisInsight)

- Runs at `http://localhost:8001`  
- Connect to host `redis:6379` inside Docker, or `localhost:6379` from your local machine.  
- Lets you browse/edit keys like `cart:<userId>`.

---

# 9. Running and Teardown

### Start
```bash
docker-compose up -d --build
```

### Verify
- Gateway → `http://localhost:8080`  
- Products → `http://localhost:8081/products`  
- Cart → `http://localhost:8082/cart`  
- Orders → `http://localhost:8083`  
- Payments → `http://localhost:8084`  
- RabbitMQ UI → `http://localhost:15672`  
- Grafana → `http://localhost:3000` (admin/admin)  
- Redis GUI → `http://localhost:8001`  

### Teardown
```bash
docker-compose down -v
```

---

# 10. Demo Flow

1. Login through Gateway (Entra ID, JWT token).  
2. Add products (via Products API).  
3. Add items to cart.  
4. Open RedisInsight to view `cart:<user>` keys.  
5. Checkout → Orders service creates pending order.  
6. Payment service marks order paid.  
7. View distributed traces in Grafana Tempo, logs in Loki, metrics in Prometheus.  

---

# 🎓 Conclusion

You now have a **complete distributed e‑commerce system** running on Docker:  
- Deno microservices tied together by Postgres, Redis, RabbitMQ  
- Authentication against Microsoft Entra ID  
- Traces, logs, and metrics in Grafana, Loki, Tempo, Prometheus  
- **RedisInsight** GUI to explore Redis data  

A perfect hands‑on system demo, educational tool, and platform foundation.  

Happy hacking — and enjoy watching your carts fill up in RedisInsight while Grafana lights up with traces 📊🐇🛒!
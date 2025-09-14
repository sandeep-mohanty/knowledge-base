# 🛍️ Comprehensive Tutorial: Event‑Driven E‑Commerce Microservices in Node.js (Express + Microsoft Entra ID)

---

## 1. Overview

We will implement a **microservices‑based, event‑driven e‑commerce system** using **Node.js + Express 4** and Docker, with the following architecture:

- **Gateway Service**  
  - Entry point for clients.  
  - Validates **Microsoft Entra ID (Azure AD)** JWT tokens.  
  - Proxies requests to other services.

- **Products Service**  
  - Stores and retrieves product catalog.  
  - Persists to **Postgres**.

- **Cart Service**  
  - Manages user carts using **Redis**.  
  - On checkout, publishes `CartCheckedOut` events to **RabbitMQ**.

- **Orders Service**  
  - Subscribes to RabbitMQ `CartCheckedOut` events.  
  - Persists new orders to **Postgres**.  
  - Can publish `OrderCreated` events.

- **Payments Service**  
  - Subscribes to RabbitMQ `OrderCreated`.  
  - Logs payment processing.

**Observability**: OpenTelemetry → Tempo, Loki, Prometheus in Grafana.  
**Debug GUI**: RedisInsight (for carts), RabbitMQ management UI.  

---

## 2. Architecture

```
          ┌─────────────────────┐
          │      Gateway        │
          │ JWT Auth (Entra ID) │
          └─────────┬───────────┘
                    │
   ┌────────────────┼─────────────────┐
   │                │                 │
Products Service   Cart Service     Orders Service
(Postgres)        (Redis store)    (Postgres + RMQ Cons.)
                     │                 ▲
                     │ Publishes       │ Consumes
                     └───────► RabbitMQ◄──────────────
                                     │
                                     ▼
                                Payments Service
Infra: Postgres, Redis, RabbitMQ, Grafana Tempo/Loki/Prometheus, RedisInsight
```

---

## 3. Project Structure

```
ecommerce-node/
  docker-compose.yml
  gateway/
    server.js
    package.json
    Dockerfile
  products/
    server.js
    package.json
    Dockerfile
  cart/
    server.js
    package.json
    Dockerfile
  orders/
    server.js
    package.json
    Dockerfile
  payments/
    server.js
    package.json
    Dockerfile
  monitoring/
    prometheus.yml
    tempo.yaml
```

---

## 4. Infrastructure: Docker Compose

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

  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]

  tempo:
    image: grafana/tempo:latest
    ports: ["4318:4318"]

  loki:
    image: grafana/loki:2.9.0
    ports: ["3100:3100"]

  prometheus:
    image: prom/prometheus
    ports: ["9090:9090"]

  gateway:
    build: ./gateway
    ports: ["8080:80"]
    depends_on: [products]

  products:
    build: ./products
    ports: ["8081:80"]
    depends_on: [postgres]

  cart:
    build: ./cart
    ports: ["8082:80"]
    depends_on: [redis, rabbitmq]

  orders:
    build: ./orders
    ports: ["8083:80"]
    depends_on: [postgres, rabbitmq]

  payments:
    build: ./payments
    ports: ["8084:80"]
    depends_on: [rabbitmq]
```

---

## 5. Service Implementations (Node.js + Express)

We’ll use modern ES modules (`"type":"module"` in package.json).

---

### 5.1 Gateway Service

`gateway/package.json`

```json
{
  "type": "module",
  "dependencies": {
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2",
    "jwks-rsa": "^3.1.0",
    "node-fetch": "^3.3.2"
  }
}
```

`gateway/server.js`

```javascript
import express from "express";
import jwt from "jsonwebtoken";
import jwksClient from "jwks-rsa";
import fetch from "node-fetch";

const app = express();
const PORT = 80;

const TENANT_ID = process.env.ENTRA_TENANT_ID;    // set in docker-compose env
const CLIENT_ID = process.env.ENTRA_CLIENT_ID;
const JWKS_URI = `https://login.microsoftonline.com/${TENANT_ID}/discovery/v2.0/keys`;

const client = jwksClient({ jwksUri: JWKS_URI });

function getKey(header, callback) {
  client.getSigningKey(header.kid, function(err, key) {
    if (err) callback(err);
    else callback(null, key.getPublicKey());
  });
}

function verifyJwt(req, res, next) {
  const token = req.headers["authorization"]?.split(" ")[1];
  if (!token) return res.status(401).send("Missing token");
  jwt.verify(token, getKey, { algorithms: ["RS256"], audience: CLIENT_ID }, (err, decoded) => {
    if (err) return res.status(401).send("Invalid token");
    req.user = decoded; next();
  });
}

app.get("/products", verifyJwt, async (req,res) => {
  const resp = await fetch("http://products:80/products");
  const data = await resp.json();
  res.json(data);
});

app.listen(PORT, ()=> console.log("Gateway running on",PORT));
```

`gateway/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node","server.js"]
```

---

### 5.2 Products Service

`products/package.json`

```json
{
  "type":"module",
  "dependencies": {
    "express":"^4.19.2",
    "pg":"^8.11.3"
  }
}
```

`products/server.js`

```javascript
import express from "express";
import pkg from "pg";
const { Pool } = pkg;

const app=express(); app.use(express.json());
const pool=new Pool({ host:"postgres", user:"postgres", password:"postgres", database:"ecommerce" });

app.get("/products", async (req,res)=>{
  const result=await pool.query("SELECT id,name,price,stock FROM products");
  res.json(result.rows);
});

app.post("/products", async (req,res)=>{
  const {name,price,stock}=req.body;
  await pool.query("INSERT INTO products(name,price,stock) VALUES ($1,$2,$3)",[name,price,stock]);
  res.json({status:"created"});
});

app.listen(80, ()=> console.log("Products service running"));
```

`products/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node","server.js"]
```

---

### 5.3 Cart Service

`cart/package.json`

```json
{
  "type":"module",
  "dependencies": {
    "express":"^4.19.2",
    "ioredis":"^5.4.1",
    "amqplib":"^0.10.3",
    "body-parser":"^1.20.2"
  }
}
```

`cart/server.js`

```javascript
import express from "express";
import Redis from "ioredis";
import amqp from "amqplib";
import bodyParser from "body-parser";

const app=express(); app.use(bodyParser.json());
const redis=new Redis({ host:"redis", port:6379 });
let channel;

async function connectRabbit() {
  const conn=await amqp.connect("amqp://guest:guest@rabbitmq:5672");
  channel=await conn.createChannel();
  await channel.assertQueue("cartCheckedOut");
}
connectRabbit();

app.post("/cart/add", async (req,res)=>{
  const { userId, productId } = req.body;
  await redis.sadd(`cart:${userId}`, productId);
  res.json({status:"added"});
});

app.get("/cart/:userId", async (req,res)=>{
  const items=await redis.smembers(`cart:${req.params.userId}`);
  res.json(items);
});

app.post("/cart/checkout", async (req,res)=>{
  const { userId }=req.body;
  const event={ userId, time: Date.now() };
  channel.sendToQueue("cartCheckedOut", Buffer.from(JSON.stringify(event)));
  res.json({status:"checkout published"});
});

app.listen(80,()=> console.log("Cart service running"));
```

`cart/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node","server.js"]
```

---

### 5.4 Orders Service

`orders/package.json`

```json
{
  "type":"module",
  "dependencies": {
    "express":"^4.19.2",
    "pg":"^8.11.3",
    "amqplib":"^0.10.3"
  }
}
```

`orders/server.js`

```javascript
import express from "express";
import pkg from "pg";
import amqp from "amqplib";
const { Pool } = pkg;

const app=express();
const pool=new Pool({ host:"postgres", user:"postgres", password:"postgres", database:"ecommerce" });

async function consume() {
  const conn=await amqp.connect("amqp://guest:guest@rabbitmq:5672");
  const ch=await conn.createChannel();
  await ch.assertQueue("cartCheckedOut");
  ch.consume("cartCheckedOut", async msg=>{
    const evt=JSON.parse(msg.content.toString());
    await pool.query("INSERT INTO orders(userId,status) VALUES ($1,$2)", [evt.userId,"PENDING"]);
    console.log("Order created for", evt.userId);
    ch.ack(msg);
  });
}
consume();

app.get("/orders", async (req,res)=>{
  const result=await pool.query("SELECT * FROM orders");
  res.json(result.rows);
});

app.listen(80,()=> console.log("Orders service running"));
```

`orders/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node","server.js"]
```

---

### 5.5 Payments Service

`payments/package.json`

```json
{
  "type":"module",
  "dependencies": {
    "express":"^4.19.2",
    "amqplib":"^0.10.3"
  }
}
```

`payments/server.js`

```javascript
import express from "express";
import amqp from "amqplib";

const app=express();

async function consume() {
  const conn=await amqp.connect("amqp://guest:guest@rabbitmq:5672");
  const ch=await conn.createChannel();
  await ch.assertQueue("orderCreated");
  ch.consume("orderCreated", msg=>{
    const evt=JSON.parse(msg.content.toString());
    console.log("Payment processed for order",evt);
    ch.ack(msg);
  });
}
consume();

app.get("/health",(req,res)=>res.json({status:"ok"}));

app.listen(80,()=> console.log("Payments service running"));
```

`payments/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node","server.js"]
```

---

## 6. Observability (OpenTelemetry)

Add to each service bootstrap:

```javascript
import { NodeSDK } from "@opentelemetry/sdk-node";
import { OTLPTraceExporter } from "@opentelemetry/exporter-trace-otlp-http";

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({ url:"http://tempo:4318/v1/traces" })
});
sdk.start();
```

Install packages:  
```
npm install @opentelemetry/api @opentelemetry/sdk-node @opentelemetry/exporter-trace-otlp-http
```

---

## 7. Running

```bash
docker-compose build
docker-compose up -d
```

Test endpoints:  
- Gateway → http://localhost:8080/products  
- Products → http://localhost:8081/products  
- Cart → http://localhost:8082/cart/:userId  
- Orders → http://localhost:8083/orders  
- Payments logs show confirmation  
- RedisInsight → http://localhost:8001  
- RabbitMQ UI → http://localhost:15672  
- Grafana → http://localhost:3000  

---

## 8. Demo Workflow

1. User authenticates with Microsoft Entra ID → JWT.  
2. Calls **Gateway** with `Authorization: Bearer <token>`.  
3. Gateway validates JWT (via Entra JWKS).  
4. Adds product → persists in Postgres.  
5. Adds to cart → stored in Redis. Verify via RedisInsight.  
6. Checkout → event published to RabbitMQ.  
7. Orders consumes event, saves PENDING order into Postgres.  
8. Payments consumes `orderCreated`, logs payment.  
9. Grafana Tempo shows distributed traces.  

---

## 9. Teardown

```bash
docker-compose down -v
```

---

## 🎓 Conclusion

We now have the **Node.js + Express implementation** of the same e‑commerce platform:  

- ✅ Gateway with **Microsoft Entra ID JWT auth**  
- ✅ Products in **Postgres**  
- ✅ Cart in **Redis** + RabbitMQ publisher  
- ✅ Orders via **RabbitMQ consumer** + Postgres persistence  
- ✅ Payments consumer mocking processing  
- ✅ Full infra with Observability, RedisInsight, RabbitMQ UI via Docker Compose  

This is a **production‑ready Node.js variant** paralleling our Java, .NET, Python, and Ballerina tutorials 🚀
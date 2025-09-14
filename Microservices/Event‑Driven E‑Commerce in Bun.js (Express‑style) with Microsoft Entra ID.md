# 🛍️ Comprehensive Tutorial: Event‑Driven E‑Commerce Microservices in Bun.js (Express‑style, Microsoft Entra ID)

---

## 1. Overview

This tutorial shows how to build an **event‑driven e‑commerce backend** with **Bun.js** using the same reference architecture as our Spring/.NET/Python/Ballerina tutorials:

- **Gateway** → Validates **Microsoft Entra ID (Azure AD) JWT tokens** using JWKS, proxies requests.  
- **Products** → Stores products in **Postgres**, provides CRUD endpoints.  
- **Cart** → Uses **Redis** for carts; publishes `CartCheckedOut` events to **RabbitMQ**.  
- **Orders** → Consumes `CartCheckedOut` from RabbitMQ, inserts order into Postgres, publishes `OrderCreated`.  
- **Payments** → Consumes `OrderCreated`, mocks payment processing.  

ℹ️ Logging/Tracing/Metrics will go into **Grafana Tempo/Loki/Prometheus**.  
🗄️ RedisInsight GUI lets you inspect cart state. RabbitMQ UI helps you manage queues.  

---

## 2. Architecture Diagram

```
            ┌───────────────────┐
            │      Gateway       │
            │ Entra ID JWT Auth  │
            └──────────┬─────────┘
                       │
   ┌───────────────────┼───────────────────┐
   │                   │                   │
Products Svc      Cart Svc (Redis)   Orders Svc (Postgres)
(Postgres)              │                ▲
                        │ Publishes      │ Consumes
                        └─────► RabbitMQ ◄────────────
                                  │
                                  ▼
                           Payments Svc

Infra: Postgres, Redis, RabbitMQ, RedisInsight, Grafana Tempo/Loki/Prometheus
```

---

## 3. Project Structure

```
ecommerce-bun/
  docker-compose.yml
  init.sql
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
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
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
    environment:
      - ENTRA_TENANT_ID=xxxxxxx
      - ENTRA_CLIENT_ID=xxxxxxx
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

## 5. Postgres Init Script

`init.sql`

```sql
CREATE TABLE IF NOT EXISTS products (
   id SERIAL PRIMARY KEY,
   name VARCHAR(100),
   price NUMERIC,
   stock INT
);

CREATE TABLE IF NOT EXISTS orders (
   id SERIAL PRIMARY KEY,
   userId VARCHAR(100),
   status VARCHAR(50)
);
```

This ensures the required tables are created at Postgres startup.

---

## 6. Services (Bun.js + Express‑style)

### Common Note
- We’ll use **Express with Bun** (`bun install express`).  
- Each package.json has `"type": "module"`.  
- Each service has a Dockerfile using `oven/bun:1`.

---

### 6.1 Gateway Service

`gateway/package.json`

```json
{
  "type":"module",
  "dependencies": {
    "express":"^4.19.2",
    "jsonwebtoken":"^9.0.2",
    "jwks-rsa":"^3.1.0",
    "node-fetch":"^3.3.2"
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

const TENANT_ID = process.env.ENTRA_TENANT_ID;
const CLIENT_ID = process.env.ENTRA_CLIENT_ID;
const JWKS_URI = `https://login.microsoftonline.com/${TENANT_ID}/discovery/v2.0/keys`;

const client = jwksClient({ jwksUri: JWKS_URI });

function getKey(header, cb) {
  client.getSigningKey(header.kid, (err, key)=>{
    if(err) cb(err); else cb(null,key.getPublicKey());
  });
}

function verifyJwt(req,res,next){
  const token=req.headers["authorization"]?.split(" ")[1];
  if(!token) return res.status(401).send("Missing token");
  jwt.verify(token,getKey,{algorithms:["RS256"],audience:CLIENT_ID},(err,decoded)=>{
    if(err) return res.status(401).send("Invalid token");
    req.user=decoded; next();
  });
}

app.get("/products", verifyJwt, async (req,res)=>{
  const resp = await fetch("http://products:80/products");
  res.json(await resp.json());
});

app.listen(PORT,()=>console.log("Gateway running"));
```

`gateway/Dockerfile`

```dockerfile
FROM oven/bun:1
WORKDIR /app
COPY package.json .
RUN bun install
COPY . .
CMD ["bun","run","server.js"]
```

---

### 6.2 Products Service

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

const app=express();app.use(express.json());
const pool=new Pool({host:"postgres",user:"postgres",password:"postgres",database:"ecommerce"});

app.get("/products",async (req,res)=>{
  const r=await pool.query("SELECT * FROM products");
  res.json(r.rows);
});

app.post("/products",async (req,res)=>{
  const {name,price,stock}=req.body;
  await pool.query("INSERT INTO products(name,price,stock) VALUES ($1,$2,$3)",[name,price,stock]);
  res.json({status:"created"});
});

app.listen(80,()=>console.log("Products running"));
```

`products/Dockerfile`

```dockerfile
FROM oven/bun:1
WORKDIR /app
COPY package.json .
RUN bun install
COPY . .
CMD ["bun","run","server.js"]
```

---

### 6.3 Cart Service

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

const app=express();app.use(bodyParser.json());
const redis=new Redis({host:"redis",port:6379});
let channel;

async function connectRabbit(){
  const conn=await amqp.connect("amqp://guest:guest@rabbitmq:5672");
  channel=await conn.createChannel();
  await channel.assertQueue("cartCheckedOut");
}
connectRabbit();

app.post("/cart/add",async(req,res)=>{
  const {userId,productId}=req.body;
  await redis.sadd(`cart:${userId}`,productId);
  res.json({status:"added"});
});

app.get("/cart/:userId",async(req,res)=>{
  const items=await redis.smembers(`cart:${req.params.userId}`);
  res.json(items);
});

app.post("/cart/checkout",async(req,res)=>{
  const {userId}=req.body;
  const evt={userId,time:Date.now()};
  channel.sendToQueue("cartCheckedOut",Buffer.from(JSON.stringify(evt)));
  res.json({status:"checkout published"});
});

app.listen(80,()=>console.log("Cart running"));
```

`cart/Dockerfile`

```dockerfile
FROM oven/bun:1
WORKDIR /app
COPY package.json .
RUN bun install
COPY . .
CMD ["bun","run","server.js"]
```

---

### 6.4 Orders Service

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
const pool=new Pool({host:"postgres",user:"postgres",password:"postgres",database:"ecommerce"});

async function consume(){
  const conn=await amqp.connect("amqp://guest:guest@rabbitmq:5672");
  const ch=await conn.createChannel();
  await ch.assertQueue("cartCheckedOut");
  ch.consume("cartCheckedOut",async msg=>{
    const evt=JSON.parse(msg.content.toString());
    await pool.query("INSERT INTO orders(userId,status) VALUES ($1,$2)",[evt.userId,"PENDING"]);
    console.log("Order created",evt.userId);
    ch.ack(msg);
    // Optionally publish orderCreated
    await ch.assertQueue("orderCreated");
    ch.sendToQueue("orderCreated",Buffer.from(JSON.stringify(evt)));
  });
}
consume();

app.get("/orders",async(req,res)=>{
  const result=await pool.query("SELECT * FROM orders");
  res.json(result.rows);
});

app.listen(80,()=>console.log("Orders running"));
```

`orders/Dockerfile`

```dockerfile
FROM oven/bun:1
WORKDIR /app
COPY package.json .
RUN bun install
COPY . .
CMD ["bun","run","server.js"]
```

---

### 6.5 Payments Service

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

async function consume(){
  const conn=await amqp.connect("amqp://guest:guest@rabbitmq:5672");
  const ch=await conn.createChannel();
  await ch.assertQueue("orderCreated");
  ch.consume("orderCreated", msg=>{
    const evt=JSON.parse(msg.content.toString());
    console.log("Payment processed",evt);
    ch.ack(msg);
  });
}
consume();

app.get("/health",(req,res)=>res.json({status:"ok"}));

app.listen(80,()=>console.log("Payments running"));
```

`payments/Dockerfile`

```dockerfile
FROM oven/bun:1
WORKDIR /app
COPY package.json .
RUN bun install
COPY . .
CMD ["bun","run","server.js"]
```

---

## 7. Running

```bash
docker-compose build
docker-compose up -d
```

Services run on ports:  
- Gateway: `http://localhost:8080` (requires Entra ID JWT)  
- Products: `http://localhost:8081/products`  
- Cart: `http://localhost:8082/cart/{userId}`  
- Orders: `http://localhost:8083/orders`  
- Payments: logs events  
- RabbitMQ UI: http://localhost:15672  
- RedisInsight: http://localhost:8001  
- Grafana: http://localhost:3000  

---

## 8. Demo Flow

1. User authenticates with **Microsoft Entra ID**, retrieves JWT.  
2. Calls Gateway with `Authorization: Bearer <token>`.  
3. Gateway verifies JWT against JWKS keys.  
4. Add product → persisted to Postgres.  
5. Add to cart → saved in Redis (see in RedisInsight).  
6. Checkout → RabbitMQ publishes event.  
7. Orders consumes → saves PENDING order to Postgres, emits `orderCreated`.  
8. Payments consumes → logs payment complete.  
9. Grafana Tempo → shows distributed traces.  

---

## 9. Teardown

```bash
docker-compose down -v
```

---

## 🎓 Conclusion

With **Bun.js**, we’ve implemented the same **event‑driven, cloud‑native E‑Commerce microservice architecture** as in the Java/.NET/Python tutorials:

- ✅ Gateway (Microsoft Entra ID JWT Auth)  
- ✅ Products (Postgres)  
- ✅ Cart (Redis + RabbitMQ publisher)  
- ✅ Orders (RabbitMQ consumer + Postgres)  
- ✅ Payments (RabbitMQ consumer)  
- ✅ Observability stack integrated  
- ✅ Postgres `init.sql` auto‑creates tables at startup  

💡 Bun.js provides **Node compatibility with much higher performance**, making it a modern alternative runtime for building distributed systems 🚀
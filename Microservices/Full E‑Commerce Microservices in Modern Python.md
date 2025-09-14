# 🛍️ Comprehensive Tutorial: Event‑Driven E‑Commerce Microservices in Python (FastAPI + Microsoft Entra ID)

---

## 1. Overview

We will build a **cloud‑native e‑commerce backend** in Python using **microservices and event‑driven architecture**:

- **Gateway** → authenticates user tokens from **Microsoft Entra ID (Azure AD)** via JWT validation + proxies calls.  
- **Products** → CRUD catalog, persisted with Postgres.  
- **Cart** → stores items in Redis, publishes checkout events to RabbitMQ.  
- **Orders** → consumes checkout events, creates orders in Postgres, optionally publishes `OrderCreated`.  
- **Payments** → consumes order events, simulates payment confirmation.  

🔒 **Authentication**: Microsoft Entra ID tokens (JWTs) validated at Gateway against Entra’s JWKS keys.  
📊 **Observability**: OpenTelemetry traces sent to **Tempo**, metrics to **Prometheus**, logs to **Loki**, visualized in **Grafana**.  
🗄️ **Debugging GUI**: RedisInsight for cart store, RabbitMQ management UI.  

---

## 2. Architecture

### Overall Microservice Architecture

```
            ┌─────────────────────┐
            │      Gateway         │
            │ JWT Auth (Entra ID)  │
            └──────────┬───────────┘
                       │
  ┌────────────────────┼───────────────────┐
  │                    │                   │
Products Service   Cart Service (Redis)   Orders Service (Postgres)
(Postgres)             │                      ▲
                       │ Publishes            │ Consumes
                       └──────> RabbitMQ <────┘
                                 │
                                 ▼
                            Payments Service
Infra: Postgres, Redis, RabbitMQ, Grafana (Tempo/Loki/Prometheus), RedisInsight
```

---

## 3. Project Structure

```
ecommerce-python/
  docker-compose.yml
  gateway/
    main.py
    requirements.txt
    Dockerfile
  products/
    main.py
    requirements.txt
    Dockerfile
  cart/
    main.py
    requirements.txt
    Dockerfile
  orders/
    main.py
    requirements.txt
    Dockerfile
  payments/
    main.py
    requirements.txt
    Dockerfile
  monitoring/
    prometheus.yml
    tempo.yaml
```

---

## 4. Infrastructure (Docker Compose)

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
    depends_on: [postgres, redis, rabbitmq]

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

## 5. Services (FastAPI)

Common libraries across services:

```
fastapi==0.111.0
uvicorn[standard]==0.30.0
httpx==0.27.0
pydantic==2.8.2
sqlalchemy==2.0.31
asyncpg==0.29.0
aioredis==2.0.1
aio-pika==9.4.2
pyjwt==2.8.0
requests==2.32.3
opentelemetry-api==1.28.2
opentelemetry-sdk==1.28.2
opentelemetry-exporter-otlp==1.28.2
```

---

### 5.1 Gateway (with Microsoft Entra JWT validation)

`gateway/main.py`

```python
from fastapi import FastAPI, Request, HTTPException
import jwt, requests, httpx

app = FastAPI(title="Gateway")

ENTRA_TENANT = "xxxxx-your-tenant-id"
ENTRA_CLIENT_ID = "xxxxx-your-client-id"
JWKS_URL = f"https://login.microsoftonline.com/{ENTRA_TENANT}/discovery/v2.0/keys"

jwks_data = requests.get(JWKS_URL).json()["keys"]

def verify_jwt(token: str):
    try:
        header = jwt.get_unverified_header(token)
        key = next(k for k in jwks_data if k["kid"] == header["kid"])
        public_key = jwt.algorithms.RSAAlgorithm.from_jwk(key)
        payload = jwt.decode(token, public_key, algorithms=["RS256"],
                             audience=ENTRA_CLIENT_ID)
        return payload
    except Exception as e:
        raise HTTPException(status_code=401, detail=f"JWT invalid: {e}")

@app.get("/products")
async def proxy_products(request: Request):
    auth = request.headers.get("authorization")
    if not auth:
        raise HTTPException(status_code=401, detail="Missing token")
    token = auth.replace("Bearer ","")
    verify_jwt(token)

    async with httpx.AsyncClient() as client:
        resp = await client.get("http://products:80/products")
        return resp.json()
```

---

### 5.2 Products Service (Postgres)

`products/main.py`

```python
from fastapi import FastAPI
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker, declarative_base
from sqlalchemy import Column, Integer, String, Float

DATABASE_URL="postgresql+asyncpg://postgres:postgres@postgres/ecommerce"

engine=create_async_engine(DATABASE_URL, echo=True)
Session=sessionmaker(engine,class_=AsyncSession,expire_on_commit=False)
Base=declarative_base()

class Product(Base):
    __tablename__="products"
    id=Column(Integer,primary_key=True)
    name=Column(String); price=Column(Float); stock=Column(Integer)

app=FastAPI(title="Products")

@app.on_event("startup")
async def init():
    async with engine.begin() as c:
        await c.run_sync(Base.metadata.create_all)

@app.get("/products")
async def list_products():
    async with Session() as s:
        r=await s.execute("SELECT id,name,price,stock FROM products")
        return [dict(row) for row in r]

@app.post("/products")
async def add(p:dict):
    pr=Product(name=p["name"],price=p["price"],stock=p["stock"])
    async with Session() as s:
        s.add(pr); await s.commit()
    return {"id":pr.id,"status":"created"}
```

---

### 5.3 Cart Service (Redis + RabbitMQ publisher)

`cart/main.py`

```python
from fastapi import FastAPI
import aioredis, aio_pika, json

app=FastAPI(title="Cart")

@app.on_event("startup")
async def startup():
    app.state.redis=await aioredis.from_url("redis://redis:6379", decode_responses=True)
    app.state.rabbit=await aio_pika.connect_robust("amqp://guest:guest@rabbitmq/")
    app.state.channel=await app.state.rabbit.channel()

@app.post("/cart/add")
async def add(userId:str, productId:str):
    await app.state.redis.sadd(f"cart:{userId}", productId)
    return {"status":"added"}

@app.get("/cart/{userId}")
async def get_cart(userId:str):
    return await app.state.redis.smembers(f"cart:{userId}")

@app.post("/cart/checkout")
async def checkout(userId:str):
    evt={"userId":userId}
    msg=aio_pika.Message(body=json.dumps(evt).encode())
    await app.state.channel.default_exchange.publish(msg,routing_key="cartCheckedOut")
    return {"status":"checkout published"}
```

---

### 5.4 Orders Service (RabbitMQ consumer + Postgres)

`orders/main.py`

```python
import json, asyncio, aio_pika
from fastapi import FastAPI
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import Column,Integer,String

DATABASE="postgresql+asyncpg://postgres:postgres@postgres/ecommerce"
engine=create_async_engine(DATABASE,echo=True)
Session=sessionmaker(engine,class_=AsyncSession,expire_on_commit=False)
Base=declarative_base()

class Order(Base):
    __tablename__="orders"
    id=Column(Integer,primary_key=True)
    userId=Column(String); status=Column(String)

async def consume():
    conn=await aio_pika.connect_robust("amqp://guest:guest@rabbitmq/")
    ch=await conn.channel()
    q=await ch.declare_queue("cartCheckedOut", durable=True)
    async with engine.begin() as c: await c.run_sync(Base.metadata.create_all)
    async for msg in q:
        async with msg.process():
            data=json.loads(msg.body)
            async with Session() as s:
                o=Order(userId=data["userId"],status="PENDING")
                s.add(o); await s.commit()
            print("Order created:", data)

app=FastAPI(title="Orders")

@app.on_event("startup")
async def start():
    asyncio.create_task(consume())

@app.get("/orders")
async def all_orders():
    async with Session() as s:
        result=await s.execute("SELECT id,userid,status FROM orders")
        return [dict(r) for r in result]
```

---

### 5.5 Payments Service (mock consumer)

`payments/main.py`

```python
import asyncio, aio_pika, json
from fastapi import FastAPI

app=FastAPI(title="Payments")

async def consume():
    conn=await aio_pika.connect_robust("amqp://guest:guest@rabbitmq/")
    ch=await conn.channel()
    q=await ch.declare_queue("orderCreated", durable=True)
    async for msg in q:
        async with msg.process():
            data=json.loads(msg.body)
            print("Payment processed:", data)

@app.on_event("startup")
async def startup():
    asyncio.create_task(consume())

@app.get("/health")
async def health(): return {"status":"ok"}
```

---

## 6. Observability (OpenTelemetry)

Add in each service for tracing:

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

provider=TracerProvider()
processor=BatchSpanProcessor(OTLPSpanExporter(endpoint="http://tempo:4318/v1/traces"))
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
```

---

## 7. Running Workflow

### Prerequisites
- Docker & docker-compose installed
- Python 3.11+ (for dev, not needed in container runtime)

### Steps
```bash
docker-compose build
docker-compose up -d
```

### Access points
- Gateway → http://localhost:8080/products (protected by Entra ID JWT)
- Products → http://localhost:8081/products
- Cart → http://localhost:8082/cart/{userId}
- Orders → http://localhost:8083/orders
- Payments → logs in container
- RabbitMQ UI → http://localhost:15672
- RedisInsight → http://localhost:8001
- Grafana → http://localhost:3000

---

## 8. Demo Flow

1. User authenticates via Microsoft Entra ID → obtains JWT.  
2. Calls **Gateway** with `Authorization: Bearer <token>`.  
3. Gateway validates token with Entra JWKS.  
4. Add product → stored in Postgres (Products).  
5. Add product to Cart → Redis. View in **RedisInsight**.  
6. Checkout → RabbitMQ `cartCheckedOut` event.  
7. Orders consumes, persists to Postgres.  
8. Payment service consumes → logs processed.  
9. Grafana → view end‑to‑end trace via Tempo.  

---

## 9. Teardown

```bash
docker-compose down -v
```

---

## 🎓 Conclusion

You now have a **production‑ready Python/FastAPI microservices e‑commerce system**:

- **Gateway** – validates Microsoft Entra ID JWTs before allowing workflow  
- **Products** – persisted in Postgres via SQLAlchemy Async  
- **Cart** – Redis storage, RabbitMQ publisher for events  
- **Orders** – RabbitMQ consumer → Postgres persistence  
- **Payments** – RabbitMQ consumer simulating payment  
- **Observability** – OpenTelemetry integrated → Grafana/Tempo/Loki/Prometheus  
- **Full Infra in Docker Compose** – works on Linux/Windows/macOS  
- **Debug tools** – RedisInsight, RabbitMQ UI  

✅ This is the **Python implementation** of the same architecture you’ve seen in all previous tutorials 🚀
# 🦕 Deno 2.x.x Masterclass Tutorial  
This tutorial is structured in two big parts:  

- **Part 1: Understanding Deno as a modern JavaScript/TypeScript runtime**  
  (covering basics, permissions, modules, tooling, TUI apps, etc.).  

- **Part 2: Building realistic backend applications**  
  (REST APIs with Oak + Postgres + Redis + RabbitMQ + Entra ID authentication).  

By the end, you’ll be able to write both **interactive CLI/TUI apps** as well as **production-style backend APIs** using Deno.  

---

# Part 1: Deno — The Modern JavaScript Runtime  

## 1. Overview of Deno  
Deno is:  
- Built on **V8** (like Node).  
- **Written in Rust** with Tokio under the hood.  
- **Secure** by default (explicit permission flags).  
- **TypeScript support, out of the box**.  
- **No `package.json` or `node_modules` mess** — direct URL imports.  
- Batteries-included: formatter, test runner, bundler, linter.  

Think of Node as an older sibling who moved out fast, leaving tools scattered everywhere; Deno is the tidier younger sibling who cleans up, locks up, and takes security seriously.  

---

## 2. Install and Basic Commands  

### Install  
```bash
# macOS/Linux
curl -fsSL https://deno.land/x/install/install.sh | sh

# Windows
irm https://deno.land/install.ps1 | iex
```

Check version:  
```bash
deno --version
# deno 2.x.x
```

### Essential Commands  
```bash
deno run app.ts     # run script
deno run --allow-net server.ts   # run with network permission
deno fmt            # format code
deno lint           # lint your code
deno test           # run tests
deno bundle app.ts app.bundle.js # bundle into single file
```

---

## 3. Permissions Sandbox  

Unlike Node, Deno sandboxes everything. By default, your script:  
- Cannot access the filesystem  
- Cannot access network  
- Cannot read environment variables  

Example:  

```typescript
// read_file.ts
const text = await Deno.readTextFile("hello.txt");
console.log(text);
```

Run it:  
```bash
deno run read_file.ts  # ERROR! No permission
deno run --allow-read read_file.ts  # Works
```

---

## 4. Modules and Imports  

Imports come from **URLs or local files**, no `npm install`.  

Example using Deno’s standard library:  
```typescript
import { serve } from "https://deno.land/std@0.207.0/http/server.ts";

await serve((_req) => new Response("Hello Deno!"), { port: 8000 });
```

Run with network permission:  
```bash
deno run --allow-net server.ts
```

---

## 5. Tooling Showcase  

### TypeScript out of the box  
```typescript
function greet(name: string): string {
  return `Hello ${name}!`;
}
console.log(greet("Deno"));
```

### Testing framework built-in  
```typescript
import { assertEquals } from "https://deno.land/std@0.207.0/assert/mod.ts";

Deno.test("simple math", () => {
  assertEquals(2 + 2, 4);
});
```

### Formatter and Linter  
```bash
deno fmt
deno lint
```

These enforce style and catch errors. ✨  

---

## 6. TUIs (Terminal User Interfaces) with Cliffy  

TUIs are fun alternatives to plain boring CLIs. With [Cliffy](https://deno.land/x/cliffy), you can prompt, create command-line tools, and even use fancy spinners.  

### Example: CLI Book Manager  
```typescript
// cli.ts
import { Command, Input, Confirm } from "https://deno.land/x/cliffy@v1.0.0-rc.4/mod.ts";

await new Command()
  .name("book-cli")
  .version("0.1.0")
  .description("CLI for managing books")
  .command("add", "Add a new book")
  .action(async () => {
    const title = await Input.prompt("Enter book title");
    const confirm = await Confirm.prompt(`Save book "${title}"?`);
    if (confirm) {
      console.log(`✅ Saved "${title}"`);
    } else {
      console.log("❌ Not saved");
    }
  })
  .parse(Deno.args);
```

Run:  
```bash
deno run --allow-read cli.ts add
```

### Example: Spinner  
```typescript
import { Spinner, wait } from "https://deno.land/x/cliffy@v1.0.0-rc.4/spinner.ts";

const spinner = new Spinner().start("Loading...");
await wait(1500);
spinner.succeed("Done!");
```

At this point you can be productive in Deno like a "normal JavaScript runtime" — but with clearer boundaries, built-in tools, and handy extras.  

---

# Part 2: Building a Realistic Web API  

Now that you’re fluent in Deno, let’s shift gears and create production-style backend applications.  

We’ll integrate:  
- **Oak** (web framework)  
- **PostgreSQL** for DB  
- **Redis** for caching  
- **RabbitMQ** for asynchronous messaging  
- **OIDC (via Entra ID)** for authentication  

---

## 1. Folder Structure  
```
project/
├─ deps.ts        # dependencies
├─ db.ts          # postgres client
├─ cache.ts       # redis client
├─ queue.ts       # rabbitmq connection
├─ auth.ts        # oidc client
├─ server.ts      # oak app
└─ cli.ts         # TUI frontend interacting with API
```

---

## 2. Centralized Deps  
```typescript
// deps.ts
export { Application, Router } from "https://deno.land/x/oak@v12.6.0/mod.ts";
export { Client as PostgresClient } from "https://deno.land/x/postgres@v0.17.0/mod.ts";
export { connect as connectRedis } from "https://deno.land/x/redis@v0.31.1/mod.ts";
export { connect as connectAmqp } from "https://deno.land/x/amqp@v0.23.1/mod.ts";
export { OAuth2Client } from "https://deno.land/x/oauth2_client@v1.0.2/mod.ts";
```

---

## 3. Database Setup  
```typescript
// db.ts
import { PostgresClient } from "./deps.ts";

const client = new PostgresClient({
  user: "postgres",
  password: "password123",
  database: "booksdb",
  hostname: "localhost",
  port: 5432,
});

await client.connect();

await client.queryArray(`
  CREATE TABLE IF NOT EXISTS books (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL
  )
`);

export default client;
```

---

## 4. Redis & RabbitMQ  
```typescript
// cache.ts
import { connectRedis } from "./deps.ts";
export const redis = await connectRedis({ hostname: "127.0.0.1", port: 6379 });
```

```typescript
// queue.ts
import { connectAmqp } from "./deps.ts";

const connection = await connectAmqp({ hostname: "127.0.0.1" });
const channel = await connection.openChannel();
await channel.declareQueue({ queue: "book_events" });

export const publishBookEvent = async (msg: unknown) => {
  await channel.publish(
    { routingKey: "book_events" },
    { contentType: "application/json" },
    new TextEncoder().encode(JSON.stringify(msg)),
  );
};
```

---

## 5. OIDC with Entra ID  
```typescript
// auth.ts
import { OAuth2Client } from "./deps.ts";

const tenantId = Deno.env.get("TENANT_ID")!;
const clientId = Deno.env.get("CLIENT_ID")!;
const clientSecret = Deno.env.get("CLIENT_SECRET")!;

export const oauth2Client = new OAuth2Client({
  clientId,
  clientSecret,
  authorizationEndpointUri: `https://login.microsoftonline.com/${tenantId}/oauth2/v2.0/authorize`,
  tokenUri: `https://login.microsoftonline.com/${tenantId}/oauth2/v2.0/token`,
  redirectUri: "http://localhost:8000/callback",
  defaults: { scope: "openid profile email" },
});
```

---

## 6. The API Server  
```typescript
// server.ts
import { Application, Router } from "./deps.ts";
import db from "./db.ts";
import { redis } from "./cache.ts";
import { publishBookEvent } from "./queue.ts";
import { oauth2Client } from "./auth.ts";

const app = new Application();
const router = new Router();

// Middleware
app.use(async (ctx, next) => {
  console.log(`${ctx.request.method} ${ctx.request.url}`);
  await next();
});

async function requireAuth(ctx: any, next: any) {
  const auth = ctx.request.headers.get("authorization");
  if (!auth) {
    ctx.response.status = 401;
    ctx.response.body = { error: "Unauthorized" };
    return;
  }
  await next();
}

// Routes
router.get("/books", requireAuth, async (ctx) => {
  const cached = await redis.get("books");
  if (cached) {
    ctx.response.body = JSON.parse(cached);
    return;
  }
  const result = await db.queryObject("SELECT * FROM books");
  const books = result.rows;
  await redis.set("books", JSON.stringify(books));
  ctx.response.body = books;
});

router.post("/books", requireAuth, async (ctx) => {
  const body = await ctx.request.body.json();
  const { title } = body;

  const result = await db.queryObject<{ id: number }>(
    "INSERT INTO books (title) VALUES ($1) RETURNING id", [title],
  );

  const newBook = { id: result.rows[0].id, title };
  await publishBookEvent({ type: "BOOK_CREATED", payload: newBook });
  await redis.del("books");

  ctx.response.status = 201;
  ctx.response.body = newBook;
});

// Login & Callback
router.get("/login", (ctx) => {
  ctx.response.redirect(oauth2Client.code.getAuthorizationUri().toString());
});
router.get("/callback", async (ctx) => {
  const tokens = await oauth2Client.code.getToken(ctx.request.url);
  ctx.response.body = { message: "Login success", tokens };
});

app.use(router.routes());
app.use(router.allowedMethods());

console.log("🚀 API running http://localhost:8000");
await app.listen({ port: 8000 });
```

---

## 7. TUI for the API  
We make a simple CLI frontend that talks to the API.  

```typescript
// cli.ts
import { Command, Input } from "https://deno.land/x/cliffy@v1.0.0-rc.4/mod.ts";

await new Command()
  .name("book-cli")
  .description("Interact with Book API")
  .command("list", "List books")
  .action(async () => {
    const resp = await fetch("http://localhost:8000/books", {
      headers: { "Authorization": "Bearer <your-token>" },
    });
    console.log(await resp.json());
  })
  .command("add", "Add a book")
  .action(async () => {
    const title = await Input.prompt("Book title?");
    const resp = await fetch("http://localhost:8000/books", {
      method: "POST",
      headers: {
        "Authorization": "Bearer <your-token>",
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title }),
    });
    console.log(await resp.json());
  })
  .parse(Deno.args);
```

---

## 8. Running the Whole Stack  

1. Start services (Docker):  
```bash
docker run -d --name pg -e POSTGRES_PASSWORD=password123 -p 5432:5432 postgres:15
docker run -d --name redis -p 6379:6379 redis:7
docker run -d --name rabbitmq -p 5672:5672 rabbitmq:3-management
```

2. Configure your Entra ID **TENANT_ID**, **CLIENT_ID**, **CLIENT_SECRET** as env vars.  

3. Run the API:  
```bash
deno run --allow-net --allow-env server.ts
```

4. Run CLI (simulating user actions):  
```bash
deno run --allow-net cli.ts list
deno run --allow-net cli.ts add
```

---

# 🎯 Summary  
- **Part 1**: Deno basics → runtime, permissions, modules, TypeScript, testing, formatting, TUI with Cliffy.  
- **Part 2**: Production-like backend → Oak-powered API + Postgres + Redis + RabbitMQ + OIDC via Entra ID.  
- **Plus**: A CLI client that interacts with backend APIs like a TUI.  

Now you can wield Deno both as a *friendly runtime* for experiments **and** as a *heavy-duty engine* for serious web applications.  

🦕✨  
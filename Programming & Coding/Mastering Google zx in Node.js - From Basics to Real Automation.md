# ⚡ Mastering Google zx in Node.js: From Basics to Real Automation  

Google’s **[zx](https://github.com/google/zx)** is a Node.js library designed to make shell scripting as elegant as **async/await JavaScript**. It eliminates messy `child_process` boilerplate while still letting you bring in any Node.js package (like Postgres, Redis, or RabbitMQ).  

In this tutorial, we’ll start with a **gentle introduction to zx** and then dive into **real-world automation use cases**, including:  
- File reading and writing (text + binary)  
- Database integration with PostgreSQL  
- Caching and status info with Redis  
- Event-driven messaging with RabbitMQ  
- External API calls  

---

# Part 1: Generic zx Tutorial  

## 1. Installation  

Install globally so you can run scripts directly:  
```bash
npm install -g zx
```

Or in a project:  
```bash
npm install zx --save-dev
```

---

## 2. Writing Scripts with zx  

Create a script `script.mjs`:  

```js
#!/usr/bin/env zx

await $`echo "Hello from zx!"`;
```

Make it executable:  
```bash
chmod +x script.mjs
./script.mjs
```

Or run:  
```bash
zx script.mjs
```

---

## 3. Core Features  

### 3.1 `$` — Run Shell Commands  
```js
const whoami = await $`whoami`;
console.log("👤 You are:", whoami.stdout.trim());
```

- `$` is a tagged template literal that runs the command safely.  
- Variables (`${}` substitutions) are properly quoted to prevent injection.  

---

### 3.2 File System Helpers  

zx brings `fs-extra` out of the box (promise-based fs).  
```js
import { readFile, writeFile } from "zx/fs";

// Write text
await writeFile("message.txt", "Hello zx!\n");

// Read text
const txt = await readFile("message.txt", "utf8");
console.log("📄 Contents:", txt);
```

---

### 3.3 Chalk: Add Color  
```js
import chalk from "zx/chalk";
console.log(chalk.green("✅ Success"), chalk.red("❌ Error"));
```

---

### 3.4 Globs + Paths  
```js
import { glob } from "zx";
for await (const file of glob("*.mjs")) {
  console.log("Found script:", file);
}
```

---

### 3.5 Questions & Interaction  
```js
import { question } from "zx";
const name = await question("What is your name? ");
console.log("✨ Hello,", name);
```

---

# Part 2: Automation Use Cases  

Now that the basics are clear, let’s look at how zx can **orchestrate real workflows**.  

---

## Use Case 1: File Read & Write (Text + Binary)

### Text Example
```js
#!/usr/bin/env zx
import { readFile, writeFile } from "zx/fs";

// Write text
await writeFile("log.txt", "Automation log started...\n");

// Append some data
await writeFile("log.txt", "Append this line!\n", { flag: "a" });

// Read file back
const text = await readFile("log.txt", "utf8");
console.log("📄 Text file contents:", text);
```

---

### Binary Example (images, zips, etc.)
```js
#!/usr/bin/env zx
import { readFile, writeFile } from "zx/fs";

// Simulate binary data (a buffer)
const buffer = Buffer.from([0xde, 0xad, 0xbe, 0xef]);
await writeFile("data.bin", buffer);

// Read binary
const bin = await readFile("data.bin");
console.log("📦 Binary file length:", bin.length);
console.log("Hex dump:", bin.toString("hex"));
```

Practical use: automate downloading files and storing them.  
```js
const resp = await fetch("https://httpbin.org/image/png");
const arrayBuffer = await resp.arrayBuffer();
await writeFile("logo.png", Buffer.from(arrayBuffer));
console.log("📷 Image saved as logo.png");
```

---

## Use Case 2: PostgreSQL Integration  

```bash
npm install pg
```

```js
#!/usr/bin/env zx
import { Client } from "pg";

// Connect
const db = new Client({
  user: "postgres",
  password: "password123",
  host: "localhost",
  database: "automationdb",
});
await db.connect();

// Create table
await db.query(`
  CREATE TABLE IF NOT EXISTS tasks(
    id SERIAL PRIMARY KEY,
    description TEXT,
    created_at TIMESTAMP DEFAULT NOW()
  )
`);

// Insert task
await db.query("INSERT INTO tasks(description) VALUES($1)", ["Automate with zx"]);
console.log("✅ Inserted task");

// Query tasks
const res = await db.query("SELECT id, description, created_at FROM tasks");
console.table(res.rows);

await db.end();
```

---

## Use Case 3: Redis Integration  

```bash
npm install redis
```

```js
#!/usr/bin/env zx
import { createClient } from "redis";

const redis = createClient({ url: "redis://127.0.0.1:6379" });
redis.on("error", console.error);
await redis.connect();

// Cache something
await redis.set("last_run", new Date().toISOString());
console.log("🗝️ Last run:", await redis.get("last_run"));

// Work with counters
await redis.incr("run_counter");
console.log("Run count:", await redis.get("run_counter"));

await redis.quit();
```

Use: store ephemeral job results or status quickly.

---

## Use Case 4: RabbitMQ Integration  

```bash
npm install amqplib
```

```js
#!/usr/bin/env zx
import amqplib from "amqplib";

// Connect & prepare queue
const conn = await amqplib.connect("amqp://localhost");
const channel = await conn.createChannel();
await channel.assertQueue("automation_queue");

// Publish
channel.sendToQueue("automation_queue", Buffer.from("Deploy finished!"));
console.log("📤 Sent a message");

// Consume
channel.consume("automation_queue", (msg) => {
  if (msg !== null) {
    console.log("📥 Received:", msg.content.toString());
    channel.ack(msg);
  }
});

// Allow 3 seconds to consume then exit
await new Promise(r => setTimeout(r, 3000));
await conn.close();
```

Use: trigger downstream jobs, notify services, monitor deployments.

---

## Use Case 5: Combined Automation Workflow  

Imagine:  
- Script checks Git status.  
- Records a deployment in Postgres.  
- Caches metadata in Redis.  
- Pushes a RabbitMQ message.  
- Notifies external API.  

```js
#!/usr/bin/env zx
import { Client } from "pg";
import { createClient } from "redis";
import amqplib from "amqplib";
import chalk from "zx/chalk";

// 1. Git clean check
const status = await $`git status --porcelain`;
if (status.stdout.trim()) {
  console.log(chalk.red("❌ Repo dirty!"));
  process.exit(1);
}

// 2. Connect Postgres
const pg = new Client({ user:"postgres", password:"password123", database:"automationdb" });
await pg.connect();
await pg.query("CREATE TABLE IF NOT EXISTS deploy_log(id SERIAL, ts TIMESTAMP DEFAULT NOW(), branch TEXT)");
const branch = (await $`git rev-parse --abbrev-ref HEAD`).stdout.trim();
await pg.query("INSERT INTO deploy_log(branch) VALUES($1)", [branch]);
console.log("📦 Logged deploy in Postgres");

// 3. Connect Redis
const redis = createClient({ url: "redis://127.0.0.1:6379" });
await redis.connect();
await redis.set("last_deploy", branch);
console.log("🗝️ Cached last branch deploy in Redis");

// 4. Connect RabbitMQ
const conn = await amqplib.connect("amqp://localhost");
const ch = await conn.createChannel();
const q = "deploy_events";
await ch.assertQueue(q);
ch.sendToQueue(q, Buffer.from(JSON.stringify({ branch, ts: Date.now() })));
console.log("📤 RabbitMQ event published");

// 5. External API notify
await fetch("https://httpbin.org/post", {
  method:"POST",
  headers:{ "Content-Type": "application/json" },
  body: JSON.stringify({ app:"myapp", branch })
});
console.log(chalk.green("🌐 API notified"));

await pg.end();
await redis.quit();
await conn.close();
```

This script shows the real power of **zx** as automation tooling: seamlessly gluing OS shell, DB, cache, queue, and APIs into one place.

---

# 🎯 Summary  

- **Part 1** → zx basics: `$` for commands, `fs-extra`, `chalk`, `globs`.  
- **Part 2** → Real use cases:  
  - File Read/Write: both text & binary (e.g., logs + image download).  
  - PostgreSQL: persisting automation logs, queries.  
  - Redis: quick caching & counters.  
  - RabbitMQ: messaging for decoupled tasks.  
  - Combined automation pipelines with Git + DB + Cache + Messaging + API.  

With zx, Node.js transforms into a **super-charged scripting tool**: safer than Bash, more powerful than raw child process code, and fully integrated with npm’s ecosystem.  

⚡ Automate everything: from local system tasks to distributed service orchestration — all in clean async/await JavaScript.  
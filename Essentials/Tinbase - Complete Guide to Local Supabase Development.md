# Tinbase - Complete Guide to Local Supabase Development Without Docker

> **Last Updated:** 2026-01-09  
> **Difficulty Level:** ⭐⭐⭐ Intermediate  
> **Estimated Reading Time:** 45 minutes  
> **Tutorial Type:** Comprehensive Deep Dive

---

## 📚 Table of Contents

1. [Introduction & Overview](#introduction--overview)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [Architecture Deep Dive](#architecture-deep-dive)
5. [Getting Started](#getting-started)
6. [Database Engines Explained](#database-engines-explained)
7. [CLI Reference](#cli-reference)
8. [Studio Dashboard](#studio-dashboard)
9. [Edge Functions](#edge-functions)
10. [Webhooks, Cron & Queues](#webhooks-cron--queues)
11. [Real-World Use Cases](#real-world-use-cases)
12. [Best Practices](#best-practices)
13. [Anti-Patterns](#anti-patterns)
14. [Performance Considerations](#performance-considerations)
15. [Security Considerations](#security-considerations)
16. [Troubleshooting Guide](#troubleshooting-guide)
17. [Migration Guide](#migration-guide)
18. [Practice Exercises](#practice-exercises)
19. [Test Your Understanding](#test-your-understanding)
20. [Common Interview Questions](#common-interview-questions)
21. [Question Bank](#question-bank)
22. [Summary & Key Takeaways](#summary--key-takeaways)
23. [Further Reading & Resources](#further-reading--resources)

---

## 🎯 Introduction & Overview

### What is Tinbase?

**Tinbase** is a revolutionary open-source backend solution that brings the power of Supabase to your local development environment without the need for Docker containers or cloud infrastructure. Born from the lifo and RapidNative projects, Tinbase was designed with a singular vision: **run an entire development stack—database, authentication, storage, and realtime features—directly in the browser and on mobile devices, with zero VMs and no cloud dependency.**

### The Problem Tinbase Solves

Traditional local development with Supabase requires:
- 🐳 Docker Desktop installation and configuration
- 💾 Significant memory overhead (2-4 GB minimum)
- 🔄 Complex setup processes for team members
- ⚡ Slower iteration cycles due to container orchestration
- 🌐 Cloud dependency for certain features

Tinbase eliminates these pain points by providing:
- ✅ **Zero Docker required** - runs as a single process
- ✅ **Minimal memory footprint** - ~59 MB at boot (native) or ~49 MB (binary)
- ✅ **Drop-in Supabase replacement** - uses official `@supabase/supabase-js` SDK
- ✅ **Multiple engine options** - native Postgres, WASM, or in-memory
- ✅ **Browser-compatible** - runs in-process in browser tabs
- ✅ **Single binary deployment** - no Node.js or npm needed on target machines

### Key Features at a Glance

| Feature | Description | Supabase Compatible |
|---------|-------------|---------------------|
| **PostgREST API** | RESTful database access | ✅ Full compatibility |
| **GoTrue Auth** | Authentication & authorization | ✅ Full compatibility |
| **Storage** | File upload/download | ✅ Full compatibility |
| **Realtime** | WebSocket-based live updates | ✅ Full compatibility |
| **Edge Functions** | Serverless function execution | ✅ Full compatibility |
| **Studio** | Built-in admin dashboard | ✅ UI compatible |
| **Migrations** | SQL-based schema changes | ✅ Full compatibility |
| **RLS** | Row Level Security | ✅ Full compatibility |

### Why Tinbase Matters in 2026

The shift toward **local-first development** and **edge computing** has created a need for lightweight, portable backend solutions. Tinbase represents a paradigm shift:

1. **Developer Experience**: Start coding in seconds, not minutes
2. **Cost Efficiency**: No cloud costs during development
3. **Privacy**: Data stays on your machine
4. **Offline Development**: Work anywhere, anytime
5. **Testing**: Faster test cycles without network dependencies
6. **Education**: Perfect for learning Supabase without infrastructure overhead

---

## 📋 Prerequisites

### Required Knowledge
- ✅ Basic understanding of PostgreSQL and SQL
- ✅ Familiarity with REST APIs
- ✅ Node.js fundamentals (v18+ recommended)
- ✅ Basic TypeScript/JavaScript knowledge
- ✅ Understanding of authentication concepts (JWT, OAuth)

### Required Tools
- ✅ **Node.js** v18+ ([Download](https://nodejs.org/))
- ✅ **npm** or **yarn** or **bun** package manager
- ✅ **Code Editor** (VS Code recommended)
- ✅ **Git** for version control

### Optional Tools
- 🔧 **Docker** (not required, but useful for comparison)
- 🔧 **PostgreSQL client** (for direct database access)
- 🔧 **curl** or **HTTPie** (for API testing)

### System Requirements

| Engine | Platform | Memory | Disk Space |
|--------|----------|--------|------------|
| **native** | macOS/Linux (x64/arm64) | ~59 MB | ~12 MB (first run) |
| **wasm** | Windows/macOS/Linux/Browser | ~575-650 MB | ~5 MB |
| **pgmem** | Any (local-dev only) | ~50 MB | ~2 MB |

---

## 🎓 Learning Objectives

By the end of this comprehensive tutorial, you will:

### Core Competencies
- ✅ Understand Tinbase's architecture and how it achieves Supabase compatibility
- ✅ Set up and configure Tinbase for local development
- ✅ Choose the right database engine for your use case
- ✅ Execute database migrations and manage schema changes
- ✅ Implement authentication and authorization
- ✅ Work with the built-in Studio dashboard
- ✅ Create and deploy Edge Functions
- ✅ Set up webhooks, cron jobs, and queues
- ✅ Optimize performance for development workflows
- ✅ Migrate existing Supabase projects to Tinbase
- ✅ Troubleshoot common issues effectively
- ✅ Apply best practices for production-ready development

### Advanced Skills
- ✅ Implement Row Level Security (RLS) policies
- ✅ Configure realtime subscriptions
- ✅ Handle file storage operations
- ✅ Debug Edge Functions locally
- ✅ Set up automated testing with Tinbase
- ✅ Integrate Tinbase with CI/CD pipelines

---

## 🏗️ Architecture Deep Dive

### How Tinbase Works

Tinbase's architecture is elegantly simple yet powerful. At its core, every service is a **pure `(Request) => Response` fetch handler**. This design enables:

1. **Universal Compatibility**: Works in Node.js servers and browser tabs
2. **Zero Overhead**: No unnecessary abstraction layers
3. **Swappable Backends**: Change database engines without touching application code

### System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        SDK[Supabase JS SDK]
        Browser[Browser App]
        Mobile[Mobile App]
    end

    subgraph "Tinbase Core"
        Handler[Request Handler<br/>(Request) => Response]
        
        subgraph "Service Layer"
            REST[PostgREST<br/>REST API]
            Auth[GoTrue<br/>Authentication]
            Storage[Storage Service]
            Realtime[Realtime<br/>WebSocket]
            Functions[Edge Functions]
            Studio[Studio Admin]
        end
        
        subgraph "Database Layer"
            Engine[DbEngine Adapter]
        end
    end

    subgraph "Engine Options"
        Native[Native<br/>Postgres 17]
        WASM[WASM<br/>PGlite]
        PgMem[pgmem<br/>In-Memory]
        External[External<br/>PostgreSQL]
    end

    SDK --> Handler
    Browser --> Handler
    Mobile --> Handler
    
    Handler --> REST
    Handler --> Auth
    Handler --> Storage
    Handler --> Realtime
    Handler --> Functions
    Handler --> Studio
    
    REST --> Engine
    Auth --> Engine
    Storage --> Engine
    Realtime --> Engine
    Functions --> Engine
    
    Engine --> Native
    Engine --> WASM
    Engine --> PgMem
    Engine --> External

    style Handler fill:#4A90E2,color:#fff
    style Engine fill:#50C878,color:#fff
    style SDK fill:#FF6B6B,color:#fff
```

**Key Insight:** The official `@supabase/supabase-js` SDK connects to a single fetch handler that routes to all services. This means **zero code changes** when switching from hosted Supabase to Tinbase.

### Request Flow Architecture

```mermaid
sequenceDiagram
    participant Client
    participant Handler as Tinbase Handler
    participant Router as Service Router
    participant Service as Service (REST/Auth/etc)
    participant Engine as DbEngine
    participant DB as Database

    Client->>Handler: HTTP Request / WebSocket
    Handler->>Router: Route to Service
    Router->>Service: Forward Request
    Service->>Engine: Query/Command
    Engine->>DB: Execute
    DB-->>Engine: Result
    Engine-->>Service: Data
    Service-->>Router: Response
    Router-->>Handler: Formatted Response
    Handler-->>Client: HTTP Response / WS Message

    Note over Handler,Engine: All services share<br/>the same DbEngine adapter
```

**Performance Note:** In Node.js, the handler wraps in an HTTP + WebSocket server. In browsers, it calls in-process—no network overhead!

### Engine Comparison Matrix

```mermaid
graph LR
    subgraph "Native Engine"
        N1[Postgres 17]
        N2[~59 MB RAM]
        N3[macOS/Linux]
        N4[Full Parity]
        N5[Best Performance]
    end

    subgraph "WASM Engine"
        W1[PGlite]
        W2[~575 MB RAM]
        W3[All Platforms]
        W4[Full Parity]
        W5[Browser Support]
    end

    subgraph "pgmem Engine"
        P1[Pure JS]
        P2[~50 MB RAM]
        P3[All Platforms]
        P4[Partial Parity]
        P5[Fastest Startup]
    end

    style N1 fill:#4CAF50
    style W1 fill:#2196F3
    style P1 fill:#FF9800
```

### Core Design Principles

1. **Single Responsibility**: Each service handles one concern (auth, storage, etc.)
2. **Adapter Pattern**: `DbEngine` interface allows swapping implementations
3. **Protocol Compatibility**: Implements Supabase's wire protocols exactly
4. **Process Isolation**: Everything runs in one process, no Docker needed
5. **Progressive Enhancement**: Works with external Postgres via `--database-url`

---

## 🚀 Getting Started

### Installation

#### Method 1: npm (Recommended)

```bash
# Install globally
npm install -g tinbase

# Or use npx (no installation needed)
npx tinbase start
```

#### Method 2: bun

```bash
# Install globally
bun install -g tinbase

# Run
tinbase start
```

#### Method 3: Build from Source

```bash
# Clone the repository
git clone https://github.com/tinbase/tinbase.git
cd tinbase

# Install dependencies
npm install

# Build
npm run build

# Run
node dist/index.js start
```

#### Method 4: Single Binary

```bash
# Build binary (requires bun)
npm run build:binary

# Output: dist-bin/tinbase (~58 MB)
./dist-bin/tinbase start
```

**💡 Pro Tip:** The single binary is perfect for CI/CD pipelines or sharing with team members who don't have Node.js installed.

### First Run

Create a new project directory:

```bash
mkdir my-tinbase-project
cd my-tinbase-project
```

Start Tinbase:

```bash
npx tinbase start
```

**Expected Output:**
```
✅ Tinbase started successfully!

🌐 API URL: http://127.0.0.1:54321
🔑 anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
🔐 service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

📁 Migrations applied: 0
💾 Database: native (Postgres 17)
```

### Connect with Supabase SDK

Create a `client.ts` file:

```typescript
import { createClient } from '@supabase/supabase-js'

// Use the keys from Tinbase startup
const TINBASE_URL = 'http://127.0.0.1:54321'
const ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'

// Create client - works exactly like Supabase!
export const supabase = createClient(TINBASE_URL, ANON_KEY)

// Test the connection
async function testConnection() {
  const { data, error } = await supabase
    .from('users')
    .select('*')
  
  if (error) {
    console.error('Connection error:', error)
  } else {
    console.log('Connected! Data:', data)
  }
}

testConnection()
```

**🎯 Key Achievement:** You're now using the **official Supabase SDK** with a local backend—no code changes needed when deploying to production Supabase!

### Project Structure

Tinbase follows Supabase's directory conventions:

```
my-tinbase-project/
├── supabase/
│   ├── migrations/
│   │   ├── 20240101000000_initial_schema.sql
│   │   └── 20240102000000_add_users_table.sql
│   ├── seed.sql
│   └── config.toml (optional)
├── src/
│   └── index.ts
└── package.json
```

**📝 Note:** The `supabase/` directory is optional—Tinbase boots even without it!

---

## 🗄️ Database Engines Explained

Tinbase provides **four database engine options**, each optimized for different scenarios. Understanding these engines is crucial for optimal development experience.

### Engine Overview

| Engine | Type | Platform | Memory | Use Case |
|--------|------|----------|--------|----------|
| **native** | Embedded Postgres 17 | macOS/Linux | ~59 MB | Production-like local dev |
| **wasm** | PGlite (WASM) | All platforms | ~575 MB | Windows, browser, cross-platform |
| **pgmem** | Pure JS in-memory | All platforms | ~50 MB | Fast prototyping, testing |
| **database-url** | External Postgres | All platforms | Varies | Existing Postgres instances |

### 1. Native Engine (Default - macOS/Linux)

The **native engine** uses embedded Postgres 17, providing the most authentic Supabase experience.

#### Characteristics

```mermaid
graph TB
    subgraph "Native Engine"
        A[Embedded Postgres 17]
        B[Private Unix Socket]
        C[Trust Authentication]
        D[Full Feature Parity]
    end

    subgraph "Benefits"
        E[Real Postgres Semantics]
        F[RLS Enforcement]
        G[Triggers & Functions]
        H[JSONB Support]
        I[Best Performance]
    end

    subgraph "Limitations"
        J[macOS/Linux Only]
        K[~12 MB Download]
        L[x64/arm64 Only]
    end

    A --> E
    A --> F
    A --> G
    A --> H
    A --> I

    style A fill:#4CAF50,color:#fff
    style E fill:#2196F3,color:#fff
```

#### Setup

```bash
# First run downloads binaries (~12 MB, cached in ~/.cache/tinbase)
tinbase start --engine native

# Or explicitly
tinbase start --engine native --port 54321
```

#### Configuration

```bash
# Custom data directory
tinbase start --engine native --data-dir ~/.my-tinbase-data

# Custom port
tinbase start --engine native --port 54322

# Custom JWT secret
tinbase start --engine native --jwt-secret "your-super-secret-key"
```

#### When to Use Native Engine

✅ **Perfect for:**
- Daily development on macOS/Linux
- Testing RLS policies accurately
- Performance testing
- Production-like environment simulation
- Complex PL/pgSQL functions

❌ **Avoid when:**
- Developing on Windows
- Need browser-based development
- Working in restricted environments

**💡 Performance Tip:** Native engine starts in ~2 seconds and uses minimal RAM. It's the best choice for most macOS/Linux developers.

### 2. WASM Engine (Default - Windows/Browser)

The **WASM engine** uses PGlite (Postgres compiled to WebAssembly), enabling Tinbase to run anywhere Node.js runs.

#### Characteristics

```mermaid
graph TB
    subgraph "WASM Engine"
        A[PGlite]
        B[WebAssembly Runtime]
        C[In-Process Execution]
        D[Full Feature Parity]
    end

    subgraph "Capabilities"
        E[Browser Support]
        F[Windows Support]
        G[RLS Enforcement]
        H[Realtime CDC]
        I[Identical Bootstrap]
    end

    subgraph "Trade-offs"
        J[~575 MB WASM Heap]
        K[Slower than Native]
        L[Higher Memory Usage]
    end

    A --> E
    A --> F
    A --> G
    A --> H
    A --> I

    style A fill:#2196F3,color:#fff
    style E fill:#4CAF50,color:#fff
```

#### Setup

```bash
# Windows default
tinbase start --engine wasm

# Browser environment
tinbase start --engine wasm --port 54321
```

#### Browser Usage Example

```typescript
// browser-client.ts
import { createClient } from '@supabase/supabase-js'
import { createBackend } from '@tinbase/core'

// Create Tinbase backend in browser
const backend = createBackend({
  engine: 'wasm',
  // Optional: pre-configure database
  databaseUrl: undefined // Uses in-memory WASM
})

// Start backend
await backend.start()

// Create Supabase client pointing to in-process backend
const supabase = createClient(
  'http://localhost:54321', // Virtual URL
  backend.keys.anonKey,
  {
    global: {
      fetch: backend.handler // Use in-process fetch
    }
  }
)

// Now you can use Supabase API directly in the browser!
const { data } = await supabase.from('users').select('*')
```

#### When to Use WASM Engine

✅ **Perfect for:**
- Windows development
- Browser-based development
- Cross-platform teams
- Testing in browser environments
- Demos and presentations

❌ **Avoid when:**
- Native engine is available (macOS/Linux)
- Memory is constrained (< 1 GB available)
- Need maximum performance

**💡 Browser Tip:** WASM engine enables true "backend in a tab" development—perfect for offline work or edge computing scenarios.

### 3. pgmem Engine (Preview - Local Dev Only)

The **pgmem engine** is an ultra-lightweight, pure-JavaScript, in-memory database. It's Tinbase's fork of `@tinbase/pg-mem`.

#### Characteristics

```mermaid
graph TB
    subgraph "pgmem Engine"
        A[Pure JavaScript]
        B[In-Memory Database]
        C[No WASM]
        D[Ultra-Fast Startup]
    end

    subgraph "Capabilities"
        E[Full Supabase Bootstrap]
        F[Real Migrations]
        G[PL/pgSQL Support]
        H[Triggers & RLS DDL]
        I[MERGE & Partitioning]
    end

    subgraph "Limitations"
        J[No LISTEN/NOTIFY]
        K[RLS Not Enforced]
        L[No Cron/pgmq]
        M[Local Dev Only]
    end

    A --> E
    A --> F
    A --> G
    A --> H
    A --> I

    style A fill:#FF9800,color:#fff
    style E fill:#4CAF50,color:#fff
    style J fill:#F44336,color:#fff
```

#### Setup

```bash
# Use pgmem engine
tinbase start --engine pgmem

# Or with environment variable
TINBASE_ENGINE=pgmem tinbase start
```

#### Feature Support Matrix

| Feature | pgmem Support | Notes |
|---------|---------------|-------|
| REST API | ✅ Full | PostgREST-compatible |
| Authentication | ✅ Full | GoTrue-compatible |
| Storage | ✅ Full | File operations work |
| Realtime | ⚠️ Partial | CDC synthesized in JS |
| Edge Functions | ✅ Full | Runs in-process |
| RLS (DDL) | ✅ Created | Not enforced (superuser) |
| RLS (Enforcement) | ❌ No | Superuser bypasses RLS |
| LISTEN/NOTIFY | ❌ No-op | CDC synthesized in JS |
| Cron Jobs | ❌ No | pg_cron not available |
| pgmq | ❌ No | Queue extension missing |

#### When to Use pgmem Engine

✅ **Perfect for:**
- Rapid prototyping
- Unit testing
- CI/CD pipelines
- Quick schema validation
- Learning Supabase concepts

❌ **Never use for:**
- Production (obviously!)
- Testing RLS policies (not enforced)
- Testing LISTEN/NOTIFY functionality
- Testing cron jobs

**⚠️ Warning:** pgmem is marked as **preview** and **local-dev only**. Never deploy to production with pgmem.

### 4. External Database URL (New in 0.10)

Tinbase can connect to an **existing PostgreSQL instance** you already run, using REST, Auth, and Storage layers.

#### Setup

```bash
# Connect to external Postgres
tinbase start --database-url postgres://user:pass@localhost:5432/mydb

# Or via environment variable
DATABASE_URL=postgres://user:pass@localhost:5432/mydb tinbase start

# Programmatic API
import { createBackend } from '@tinbase/core'

const backend = createBackend({
  databaseUrl: 'postgres://user:pass@localhost:5432/mydb'
})

await backend.start()
```

#### Authentication with External DB

```bash
# External DB uses TCP with SCRAM-SHA-256 (or md5)
# No trust authentication like native engine
```

#### Features & Roadmap

| Feature | Status | Notes |
|---------|--------|-------|
| REST API | ✅ Available | Full PostgREST compatibility |
| Auth | ✅ Available | GoTrue-compatible |
| Storage | ✅ Available | Full file operations |
| Realtime CDC | ⚠️ Coming Soon | Without superuser |
| TLS/SSL | ⚠️ Coming Soon | Managed provider support |
| Connection Pooling | ⚠️ Coming Soon | For high-concurrency scenarios |

#### When to Use External Database

✅ **Perfect for:**
- Existing Postgres infrastructure
- Shared development databases
- Testing against production-like data
- Multi-service architectures

❌ **Avoid when:**
- You want zero-configuration setup
- Need full Tinbase feature parity
- Require realtime CDC (not yet available)

**💡 Migration Tip:** Use `--database-url` to gradually migrate from Supabase cloud to Tinbase without changing your database.

### Engine Selection Guide

```mermaid
flowchart TD
    A[Choose Engine] --> B{Platform?}
    
    B -->|macOS/Linux| C{Need Best Performance?}
    B -->|Windows| D[Use WASM]
    B -->|Browser| E[Use WASM]
    
    C -->|Yes| F[Use Native]
    C -->|No| G{Need Browser Support?}
    
    G -->|Yes| D
    G -->|No| F
    
    H{Have External Postgres?} -->|Yes| I[Use database-url]
    H -->|No| J{Quick Prototyping?}
    
    J -->|Yes| K[Use pgmem]
    J -->|No| L{Platform?}
    
    L -->|macOS/Linux| F
    L -->|Windows| D
    
    style F fill:#4CAF50
    style D fill:#2196F3
    style K fill:#FF9800
    style I fill:#9C27B0
```

**Decision Tree Summary:**
1. **macOS/Linux + Performance** → Native
2. **Windows or Browser** → WASM
3. **Quick tests/prototyping** → pgmem
4. **Existing Postgres** → database-url

---

## 💻 CLI Reference

Tinbase provides a powerful CLI for managing your local Supabase development environment.

### Command Overview

```mermaid
graph LR
    A[tinbase CLI] --> B[start]
    A --> C[migrate]
    A --> D[status]
    A --> E[keys]
    A --> F[gen types]
    A --> G[db reset]
    A --> H[db diff]
    
    B --> B1[Boot server]
    C --> C1[Apply migrations]
    D --> D1[List migrations]
    E --> E1[Print API keys]
    F --> F1[Generate TS types]
    G --> G1[Reset database]
    H --> H1[Schema diff]
    
    style A fill:#4A90E2,color:#fff
    style B fill:#4CAF50,color:#fff
```

### Core Commands

#### `tinbase start`

Boot the development server with optional migration application.

```bash
# Basic usage
tinbase start

# With options
tinbase start \
  --port 54321 \
  --dir ./my-project \
  --data-dir ~/.tinbase/data \
  --engine native \
  --jwt-secret "my-secret-key"

# Using environment variables
TINBASE_PORT=54321 \
TINBASE_ENGINE=native \
TINBASE_JWT_SECRET="my-secret" \
tinbase start
```

**Options:**

| Flag | Environment Variable | Default | Description |
|------|---------------------|---------|-------------|
| `-p, --port <n>` | `TINBASE_PORT` / `PORT` | 54321 | Server port |
| `--dir <path>` | - | Current directory | Project directory |
| `--data-dir <path>` | - | `<dir>/.tinbase/db` | Database storage location |
| `--jwt-secret <s>` | `TINBASE_JWT_SECRET` | Random | JWT signing secret |
| `--memory` | - | false | Use in-memory WASM engine |
| `--engine <e>` | `TINBASE_ENGINE` | native | Engine: native/wasm/pgmem |
| `--database-url <url>` | `DATABASE_URL` | - | External Postgres URL |

**Example Output:**
```
✅ Tinbase started successfully!

🌐 API URL: http://127.0.0.1:54321
🔑 anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFkbSIsInBhdGgiOiJcLyIsInJvbGUiOiJhbm9uIiwiaWF0IjoxNjM4NzY1MzEwfQ.example
🔐 service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFkbSIsInBhdGgiOiJcLyIsInJvbGUiOiJzZXJ2aWNlX3JvbGUiLCJpYXQiOjE2Mzg3NjUzMX0.example

📁 Migrations applied: 3
💾 Database: native (Postgres 17)
🔗 Studio: http://127.0.0.1:54321/_/
```

#### `tinbase migrate`

Apply pending migrations and exit (useful for CI/CD).

```bash
# Apply all pending migrations
tinbase migrate

# With custom directory
tinbase migrate --dir ./my-project

# Check what would be applied (dry run)
tinbase status
```

**Use Cases:**
- ✅ CI/CD pipelines
- ✅ Pre-deployment checks
- ✅ Automated testing setup
- ✅ Database schema validation

#### `tinbase status`

List all applied migrations.

```bash
tinbase status
```

**Example Output:**
```
Applied migrations (3):
  ✅ 20240101000000_initial_schema.sql
  ✅ 20240102000000_add_users_table.sql
  ✅ 20240103000000_add_posts_table.sql

Pending migrations (1):
  ⏳ 20240104000000_add_comments_table.sql
```

#### `tinbase keys`

Print anon and service_role keys.

```bash
tinbase keys
```

**Example Output:**
```
anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**💡 Tip:** Use these keys in your environment variables or `.env` file:

```bash
# .env
VITE_SUPABASE_URL=http://127.0.0.1:54321
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### `tinbase gen types`

Generate TypeScript database types from your schema.

```bash
# Generate types
tinbase gen types

# Output to specific file
tinbase gen types --output ./src/types/database.ts

# With specific schema
tinbase gen types --schema public,auth
```

**Example Output:**
```typescript
// Generated types
export type Database = {
  public: {
    Tables: {
      users: {
        Row: {
          id: string
          email: string
          created_at: string
        }
        Insert: {
          id?: string
          email: string
          created_at?: string
        }
        Update: {
          id?: string
          email?: string
          created_at?: string
        }
      }
      posts: {
        Row: {
          id: number
          title: string
          content: string
          user_id: string
        }
        // ... Insert and Update types
      }
    }
  }
}
```

**Usage:**
```typescript
import { createClient } from '@supabase/supabase-js'
import type { Database } from './types/database'

type User = Database['public']['Tables']['users']['Row']

const supabase = createClient<Database>(URL, ANON_KEY)

// Now you have full type safety!
const { data } = await supabase
  .from('users')
  .select('*')
// data is typed as User[]
```

#### `tinbase db reset`

Wipe database and re-run all migrations and seed.

```bash
# Reset database
tinbase db reset

# Equivalent to:
# 1. Drop all tables
# 2. Re-run all migrations
# 3. Execute seed.sql
```

**⚠️ Warning:** This **deletes all data** in the database. Use only in development!

**Use Cases:**
- ✅ Starting fresh during development
- ✅ Resetting test databases
- ✅ Cleaning up after schema changes

#### `tinbase db diff`

Generate DDL for schema changes outside of migrations.

```bash
# Compare current DB state with migrations
tinbase db diff

# Output: SQL statements to sync database with code
```

**Example Output:**
```sql
-- Add missing column
ALTER TABLE users ADD COLUMN phone TEXT;

-- Create missing index
CREATE INDEX idx_posts_user_id ON posts(user_id);
```

**💡 Workflow Tip:** Use `db diff` to detect schema drift between your code and database during development.

### Advanced CLI Usage

#### Environment Variables

```bash
# Complete configuration via environment
export TINBASE_PORT=54321
export TINBASE_ENGINE=native
export TINBASE_DATA_DIR=~/.tinbase/data
export TINBASE_JWT_SECRET="your-jwt-secret"
export DATABASE_URL="postgres://localhost:5432/mydb"

tinbase start
```

#### Scripting with CLI

```bash
#!/bin/bash
# scripts/setup-dev.sh

# Reset database
tinbase db reset

# Apply migrations
tinbase migrate

# Generate types
tinbase gen types --output ./src/types/database.ts

# Start server
tinbase start
```

**Make executable:**
```bash
chmod +x scripts/setup-dev.sh
./scripts/setup-dev.sh
```

#### CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      - name: Start Tinbase
        run: |
          npx tinbase start --engine wasm &
          sleep 5
      
      - name: Run tests
        run: npm test
```

---

## 🎨 Studio Dashboard

Tinbase includes a **built-in Studio dashboard** that mirrors Supabase Studio's interface, providing a visual interface for database management.

### Accessing Studio

```bash
# Start Tinbase
tinbase start

# Access Studio at
http://127.0.0.1:54321/_
```

**Login:** Use the `service_role` key printed at startup.

### Studio Features

#### 1. Table Editor

Browse, create, edit, and delete table rows with a user-friendly interface.

**Features:**
- 📊 Paginated data viewing with row counts
- ✏️ Inline editing of cell values
- ➕ Insert new rows
- 🗑️ Delete rows
- 🔍 Filter and search
- 📤 Export to CSV/JSON

**Example Workflow:**

```sql
-- Create a users table (via SQL Editor)
CREATE TABLE users (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at TIMESTAMP DEFAULT now()
);

-- View in Table Editor
-- 1. Navigate to "Table Editor" in sidebar
-- 2. Select "users" table
-- 3. Click "Insert" to add rows
-- 4. Click cell to edit inline
```

#### 2. SQL Editor

Execute SQL queries with syntax highlighting and result grids.

**Features:**
- 💻 SQL syntax highlighting
- 📊 Result grid with Postgres error details
- 📜 Query history
- 💾 Save favorite queries
- 📤 Export results

**Example Queries:**

```sql
-- Find all active users
SELECT * FROM users WHERE created_at > now() - interval '7 days';

-- Join users with posts
SELECT 
  u.name,
  p.title,
  p.created_at
FROM users u
JOIN posts p ON p.user_id = u.id
ORDER BY p.created_at DESC;

-- Aggregate statistics
SELECT 
  COUNT(*) as total_users,
  COUNT(CASE WHEN created_at > now() - interval '30 days' THEN 1 END) as new_users
FROM users;
```

#### 3. Authentication Manager

Manage users, reset passwords, and configure auth settings.

**Features:**
- 👥 List all users
- ➕ Create new users
- 🔑 Reset passwords
- 🚫 Ban/unban users
- 📧 Send password reset emails
- 🔐 Configure OAuth providers

**Example Operations:**

```typescript
// Create user via SDK (also visible in Studio)
const { data, error } = await supabase.auth.admin.createUser({
  email: 'user@example.com',
  password: 'secure-password',
  email_confirm: true
})

// View in Studio:
// 1. Go to "Authentication" → "Users"
// 2. See newly created user
// 3. Click to view details or reset password
```

#### 4. Storage Manager

Manage file storage buckets and objects.

**Features:**
- 📁 Create/delete buckets
- ⬆️ Upload files
- ⬇️ Download files
- 🔗 Generate public URLs
- 🔒 Toggle public/private access

**Example Workflow:**

```typescript
// Create bucket via SDK
const { data } = await supabase.storage.createBucket('avatars', {
  public: true,
  fileSizeLimit: 1024 * 1024 // 1 MB
})

// Upload file
const { data, error } = await supabase.storage
  .from('avatars')
  .upload('user-123.jpg', fileBlob)

// View in Studio:
// 1. Go to "Storage" → "Buckets"
// 2. Select "avatars" bucket
// 3. See uploaded file
// 4. Toggle public/private
```

#### 5. Database Overview

View database statistics and migration history.

**Metrics Displayed:**
- 📊 Table sizes and row counts
- 🔄 Migration history
- 💾 Database size
- ⚡ Connection info
- 📈 Performance metrics

### Studio Architecture

```mermaid
graph TB
    subgraph "Studio Frontend"
        UI[React + Radix + Tailwind]
        Table[Table Editor]
        SQL[SQL Editor]
        Auth[Auth Manager]
        Storage[Storage Manager]
        DB[Database Stats]
    end

    subgraph "Studio Backend"
        API[Studio API<br/>/_/]
        AuthMiddleware[Auth Middleware]
        QueryBuilder[Query Builder]
    end

    subgraph "Tinbase Core"
        Handler[Request Handler]
        DbEngine[DbEngine]
    end

    UI --> Table
    UI --> SQL
    UI --> Auth
    UI --> Storage
    UI --> DB
    
    Table --> API
    SQL --> API
    Auth --> API
    Storage --> API
    DB --> API
    
    API --> AuthMiddleware
    AuthMiddleware --> QueryBuilder
    QueryBuilder --> Handler
    Handler --> DbEngine

    style UI fill:#4A90E2,color:#fff
    style API fill:#50C878,color:#fff
```

**🎯 Key Feature:** Studio compiles to a **single self-contained HTML file**, so it works inside the single binary distribution too!

### Studio Best Practices

✅ **Do:**
- Use Studio for rapid prototyping and debugging
- Test SQL queries in SQL Editor before adding to migrations
- Monitor database growth via Database Overview
- Use Table Editor for quick data inspection

❌ **Don't:**
- Use Studio for production data management
- Rely on Studio's inline editing for critical data changes
- Share `service_role` key with frontend applications
- Use Studio as your only database backup strategy

**⚠️ Security Note:** Studio requires the `service_role` key, which has **full database access**. Never expose this key in client-side applications!

---

## ⚡ Edge Functions

Tinbase supports **Supabase-compatible Edge Functions**, allowing you to run serverless logic directly in the backend process.

### How Edge Functions Work

```mermaid
graph LR
    A[Client Request] --> B[Supabase SDK]
    B --> C[functions.invoke]
    C --> D[Tinbase Handler]
    D --> E[Edge Function Runtime]
    E --> F[User Function Code]
    F --> G[Response]
    
    style E fill:#4A90E2,color:#fff
    style F fill:#50C878,color:#fff
```

**Key Difference from Supabase:** Tinbase runs functions **in-process** rather than in a separate Deno runtime, reducing overhead and simplifying deployment.

### Creating Your First Edge Function

#### Step 1: Create Function Directory

```bash
mkdir -p supabase/functions/hello
```

#### Step 2: Write Function Code

```javascript
// supabase/functions/hello/index.mjs
Deno.serve(async (req) => {
  try {
    // Parse request body
    const { name = 'world' } = await req.json().catch(() => ({}))
    
    // Business logic
    const message = `Hello ${name}!`
    
    // Return response
    return new Response(
      JSON.stringify({ message }),
      { 
        headers: { 
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*' // CORS
        } 
      }
    )
  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { 
        status: 500,
        headers: { 'Content-Type': 'application/json' }
      }
    )
  }
})
```

#### Step 3: Invoke Function from Client

```typescript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient('http://127.0.0.1:54321', ANON_KEY)

// Invoke function
const { data, error } = await supabase.functions.invoke('hello', {
  body: { name: 'Tinbase Developer' }
})

if (error) {
  console.error('Function error:', error)
} else {
  console.log('Response:', data)
  // { message: "Hello Tinbase Developer!" }
}
```

### Advanced Edge Function Patterns

#### Pattern 1: Database Operations

```javascript
// supabase/functions/create-post/index.mjs
Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL'),
    Deno.env.get('SUPABASE_ANON_KEY')
  )
  
  // Get authenticated user
  const authHeader = req.headers.get('Authorization')
  const { data: { user } } = await supabase.auth.getUser(authHeader)
  
  if (!user) {
    return new Response('Unauthorized', { status: 401 })
  }
  
  // Create post
  const { title, content } = await req.json()
  const { data: post, error } = await supabase
    .from('posts')
    .insert({
      title,
      content,
      user_id: user.id
    })
    .select()
    .single()
  
  if (error) {
    return new Response(error.message, { status: 400 })
  }
  
  return new Response(JSON.stringify(post), {
    headers: { 'Content-Type': 'application/json' }
  }
})
```

#### Pattern 2: External API Integration

```javascript
// supabase/functions/weather/index.mjs
Deno.serve(async (req) => {
  const { city } = await req.json()
  
  // Call external API
  const weatherResponse = await fetch(
    `https://api.weather.com/v1/current?city=${city}&key=${Deno.env.get('WEATHER_API_KEY')}`
  )
  
  const weatherData = await weatherResponse.json()
  
  // Transform and return
  return new Response(
    JSON.stringify({
      city,
      temperature: weatherData.temp,
      condition: weatherData.condition
    }),
    { headers: { 'Content-Type': 'application/json' } }
  )
})
```

#### Pattern 3: Webhook Handler

```javascript
// supabase/functions/webhook-handler/index.mjs
Deno.serve(async (req) => {
  // Verify webhook signature
  const signature = req.headers.get('X-Webhook-Signature')
  const body = await req.text()
  
  if (!verifySignature(body, signature)) {
    return new Response('Invalid signature', { status: 401 })
  }
  
  // Process webhook
  const event = JSON.parse(body)
  
  // Store in database
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL'),
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')
  )
  
  await supabase.from('webhooks').insert({
    event_type: event.type,
    payload: event,
    received_at: new Date()
  })
  
  return new Response('OK', { status: 200 })
})

function verifySignature(body, signature) {
  // Implement signature verification
  return true
}
```

### Edge Function Configuration

#### Using `createBackend` API

```typescript
import { createBackend } from '@tinbase/core'

const backend = createBackend({
  functions: {
    'hello': async (req) => {
      const { name } = await req.json().catch(() => ({}))
      return new Response(
        JSON.stringify({ message: `Hello ${name}!` }),
        { headers: { 'Content-Type': 'application/json' } }
      )
    },
    
    'goodbye': async (req) => {
      return new Response(
        JSON.stringify({ message: 'Goodbye!' }),
        { headers: { 'Content-Type': 'application/json' } }
      )
    }
  }
})

await backend.start()
```

#### Import Support

```javascript
// Functions using only Web APIs work as-is
// supabase/functions/process-data/index.mjs

// ✅ Works: Web APIs
const crypto = globalThis.crypto
const fetch = globalThis.fetch

// ❌ Needs bundling: npm:/jsr:/URL imports
// import _ from 'npm:lodash'
// These require a build step
```

**💡 Tip:** For functions requiring npm packages, use a bundler like `esbuild` or `rollup` before deployment.

### Edge Function Best Practices

✅ **Do:**
- Handle errors gracefully with try-catch
- Validate input data
- Use environment variables for secrets
- Implement proper CORS headers
- Return appropriate HTTP status codes
- Log errors for debugging

❌ **Don't:**
- Store secrets in function code
- Trust client-side data without validation
- Return sensitive information in errors
- Forget to set CORS headers for browser clients
- Use blocking operations in async functions

**⚠️ Security Note:** Edge Functions run with the same permissions as the backend. Always validate and sanitize user input!

---

## 🔔 Webhooks, Cron & Queues

Tinbase implements automation features **natively** without requiring extensions like `pg_net`, `pg_cron`, or `pgmq`.

### Database Webhooks

Fire HTTP requests when database rows change, using Supabase's exact payload format.

#### Configuration via Code

```typescript
import { createBackend } from '@tinbase/core'

const backend = createBackend({
  webhooks: {
    'user-created': {
      table: 'users',
      events: ['INSERT'],
      url: 'https://api.example.com/webhooks/user-created',
      headers: {
        'Authorization': 'Bearer your-secret-token'
      },
      retry: {
        maxRetries: 3,
        backoff: 'exponential'
      }
    }
  }
})

// Or register programmatically
backend.webhooks.register({
  table: 'posts',
  events: ['INSERT', 'UPDATE', 'DELETE'],
  url: 'https://api.example.com/webhooks/posts-changed'
})
```

#### Configuration via JSON

```json
// supabase/webhooks.json
{
  "webhooks": [
    {
      "id": "user-created",
      "table": "users",
      "events": ["INSERT"],
      "url": "https://api.example.com/webhooks/user-created",
      "headers": {
        "Authorization": "Bearer your-secret-token"
      },
      "retry": {
        "maxRetries": 3,
        "backoff": "exponential"
      }
    },
    {
      "id": "post-published",
      "table": "posts",
      "events": ["UPDATE"],
      "filter": "status = 'published'",
      "url": "https://api.example.com/webhooks/post-published"
    }
  ]
}
```

#### Webhook Payload Format

```json
{
  "type": "INSERT",
  "table": "users",
  "schema": "public",
  "record": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "email": "user@example.com",
    "created_at": "2024-01-15T10:30:00Z"
  },
  "old_record": null
}
```

**Payload Fields:**
- `type`: Event type (`INSERT`, `UPDATE`, `DELETE`)
- `table`: Table name
- `schema`: Schema name
- `record`: New row data (INSERT/UPDATE)
- `old_record`: Previous row data (UPDATE/DELETE)

#### Webhook Filtering

```sql
-- Only trigger webhook when status changes to 'published'
-- In webhooks.json:
{
  "table": "posts",
  "events": ["UPDATE"],
  "filter": "OLD.status != 'published' AND NEW.status = 'published'"
}
```

### Cron Jobs

Schedule recurring tasks using PostgreSQL's `pg_cron` API.

#### Basic Cron Syntax

```sql
-- Schedule syntax: 'N seconds' or standard cron format
SELECT cron.schedule(
  'nightly-cleanup',      -- Job name
  '0 0 * * *',            -- Cron expression (midnight UTC)
  'DELETE FROM logs WHERE created_at < now() - interval ''30 days'''
);

-- Short form: every N seconds
SELECT cron.schedule(
  'every-5-minutes',
  '5 seconds',
  'UPDATE stats SET last_check = now()'
);
```

#### Managing Cron Jobs

```sql
-- List all scheduled jobs
SELECT * FROM cron.job;

-- View job execution history
SELECT * FROM cron.job_run_details
ORDER BY start_time DESC
LIMIT 10;

-- Unschedule a job
SELECT cron.unschedule('nightly-cleanup');

-- Manual trigger
SELECT cron.unschedule('manual-cleanup');
-- Then run the SQL directly
DELETE FROM logs WHERE created_at < now() - interval '30 days';
```

#### Cron Job Tables

```mermaid
erDiagram
    cron.job {
        uuid jobid PK
        text schedule
        text command
        text nodename
        text database
        text username
    }
    
    cron.job_run_details {
        bigserial runid PK
        uuid jobid FK
        text status
        timestamp start_time
        timestamp end_time
        text output
    }
    
    cron.job ||--o{ cron.job_run_details : "has"
```

**Table Descriptions:**
- `cron.job`: Stores scheduled job definitions
- `cron.job_run_details`: Logs each job execution

#### Practical Cron Examples

```sql
-- Example 1: Daily analytics aggregation
SELECT cron.schedule(
  'aggregate-analytics',
  '0 2 * * *',  -- 2 AM UTC daily
  $$
  INSERT INTO analytics_daily (date, page_views, users)
  SELECT 
    current_date,
    COUNT(*) as page_views,
    COUNT(DISTINCT user_id) as users
  FROM page_views
  WHERE created_at >= current_date - interval '1 day'
    AND created_at < current_date
  ON CONFLICT (date) DO UPDATE
  SET page_views = analytics_daily.page_views + EXCLUDED.page_views,
      users = analytics_daily.users + EXCLUDED.users;
  $$
);

-- Example 2: Hourly cache refresh
SELECT cron.schedule(
  'refresh-cache',
  '0 * * * *',  -- Every hour
  'SELECT refresh_materialized_views();'
);

-- Example 3: Weekly report generation
SELECT cron.schedule(
  'weekly-report',
  '0 9 * * 1',  -- 9 AM UTC every Monday
  $$
  INSERT INTO reports (type, data, generated_at)
  SELECT 
    'weekly',
    json_agg(u.*),
    now()
  FROM users u
  WHERE created_at >= now() - interval '7 days';
  $$
);
```

**💡 Tip:** Cron schedules match in **UTC**, just like hosted `pg_cron`. Always specify times in UTC to avoid daylight saving time issues!

### HTTP from SQL

Execute HTTP requests directly from SQL queries.

```sql
-- Make HTTP GET request
SELECT http_get('https://api.example.com/data');

-- POST request with body
SELECT http_post(
  'https://api.example.com/webhook',
  '{"key": "value"}',
  'application/json'
);

-- With headers
SELECT http_get(
  'https://api.example.com/protected',
  ARRAY['Authorization: Bearer token123']
);
```

**Use Cases:**
- Call external APIs from database triggers
- Send notifications on data changes
- Integrate with third-party services
- Implement event-driven architectures

### Complete Automation Example

```typescript
// Complete example: Blog post automation
import { createBackend } from '@tinbase/core'

const backend = createBackend({
  webhooks: {
    'new-post-notification': {
      table: 'posts',
      events: ['INSERT'],
      url: 'https://api.example.com/notify-new-post',
      headers: { 'Authorization': 'Bearer token' }
    }
  }
})

// Set up cron job for weekly digest
await backend.query(`
  SELECT cron.schedule(
    'weekly-digest',
    '0 9 * * 1',
    $$
    INSERT INTO email_queue (to_email, subject, body)
    SELECT 
      u.email,
      'Your Weekly Digest',
      json_build_object('posts', json_agg(p.*))
    FROM users u
    LEFT JOIN posts p ON p.user_id = u.id
      AND p.created_at >= now() - interval '7 days'
    GROUP BY u.email;
    $$
  )
`)

await backend.start()
```

---

## 🌍 Real-World Use Cases

### Use Case 1: Rapid Prototyping

**Scenario:** Startup needs to validate a new SaaS idea in 48 hours.

**Solution with Tinbase:**
```bash
# 1. Initialize project (30 seconds)
mkdir saas-prototype && cd saas-prototype
npx tinbase start

# 2. Create schema (5 minutes)
# supabase/migrations/001_initial.sql
CREATE TABLE users (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  plan TEXT DEFAULT 'free'
);

CREATE TABLE projects (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  name TEXT NOT NULL
);

# 3. Build frontend (2 hours)
# Use Next.js + Supabase SDK - works unchanged!
```

**Benefits:**
- ⚡ No Docker setup time
- 💰 Zero infrastructure costs
- 🔄 Instant iteration cycles
- 📱 Works offline

### Use Case 2: Offline Development

**Scenario:** Developer working on a flight without internet.

**Solution with Tinbase:**
```bash
# Start with WASM engine (works offline)
tinbase start --engine wasm

# Full Supabase API available
# - Database operations
# - Authentication
# - Storage
# - Realtime
# All running in browser/Node.js
```

**Benefits:**
- ✈️ Full productivity offline
- 🔒 Data stays local
- 🚀 No cloud dependency
- 💻 Works on any machine

### Use Case 3: CI/CD Testing

**Scenario:** Run database integration tests in GitHub Actions.

**Solution with Tinbase:**
```yaml
# .github/workflows/test.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Start Tinbase
        run: |
          npx tinbase start --engine wasm &
          sleep 5
      
      - name: Run Tests
        run: npm test
```

**Benefits:**
- ✅ No Docker in CI
- ⚡ Faster test execution
- 💰 Reduced CI costs
- 🔧 Simpler configuration

### Use Case 4: Education & Training

**Scenario:** Teach Supabase concepts without infrastructure complexity.

**Solution with Tinbase:**
```bash
# Students install Tinbase globally
npm install -g tinbase

# Start learning immediately
tinbase start

# No Docker, no cloud accounts, no configuration
```

**Benefits:**
- 🎓 Focus on concepts, not infrastructure
- ⚡ Instant setup for students
- 💻 Works on any OS
- 🌐 Browser-based demos possible

### Use Case 5: Local Development for Teams

**Scenario:** 10-person team needs consistent local development environment.

**Solution with Tinbase:**
```bash
# Team standard setup
npm install -g tinbase

# Consistent environment via scripts
./scripts/setup-dev.sh

# Everyone gets identical setup
# - Same Postgres version
# - Same migrations
# - Same seed data
```

**Benefits:**
- 👥 Consistent environments
- 🚫 No "works on my machine" issues
- ⚡ Fast onboarding
- 📦 Version-controlled setup

### Use Case 6: Edge Computing Prototyping

**Scenario:** Build app that works offline and syncs when online.

**Solution with Tinbase:**
```typescript
// Browser-based backend
const backend = createBackend({ engine: 'wasm' })
await backend.start()

// Full Supabase API in browser
const supabase = createClient(URL, KEY, {
  global: { fetch: backend.handler }
})

// App works 100% offline
// Syncs when connection available
```

**Benefits:**
- 🌐 True offline-first apps
- 📱 Progressive Web App (PWA) ready
- 🔄 Background sync capability
- 🚀 Edge deployment ready

---

## ✅ Best Practices

### Development Workflow

#### 1. Project Structure

```
my-project/
├── supabase/
│   ├── migrations/
│   │   └── YYYYMMDDHHMMSS_description.sql
│   ├── seed.sql
│   └── config.toml (optional)
├── src/
│   ├── lib/
│   │   └── supabase.ts          # Supabase client
│   ├── types/
│   │   └── database.ts          # Generated types
│   └── index.ts
├── .env                          # Environment variables
├── .gitignore
└── package.json
```

**Best Practice:** Keep migrations in chronological order with descriptive names.

#### 2. Migration Management

```sql
-- ✅ Good: Descriptive, incremental migration
-- 20240115120000_add_user_preferences.sql
CREATE TABLE user_preferences (
  user_id UUID REFERENCES users(id) PRIMARY KEY,
  theme TEXT DEFAULT 'light',
  notifications BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_user_preferences_user_id 
  ON user_preferences(user_id);

-- ❌ Bad: Vague, non-incremental
-- migration.sql
ALTER TABLE users ADD COLUMN preferences JSONB;
```

#### 3. Environment Configuration

```bash
# .env (git-ignored)
TINBASE_PORT=54321
TINBASE_ENGINE=native
SUPABASE_URL=http://127.0.0.1:54321
SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

# .env.example (committed)
TINBASE_PORT=54321
TINBASE_ENGINE=native
SUPABASE_URL=http://127.0.0.1:54321
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

#### 4. Type Safety

```typescript
// ✅ Good: Generate and use types
import { createClient } from '@supabase/supabase-js'
import type { Database } from './types/database'

const supabase = createClient<Database>(URL, ANON_KEY)

const { data } = await supabase
  .from('users')
  .select('*')
// data is fully typed!

// ❌ Bad: No types
const supabase = createClient(URL, ANON_KEY)
const { data } = await supabase.from('users').select('*')
// data is 'any' - no type safety
```

### Performance Optimization

#### 1. Connection Pooling

```typescript
// ✅ Good: Reuse client instance
import { createClient } from '@supabase/supabase-js'

// Create once at app startup
export const supabase = createClient(URL, ANON_KEY)

// Use everywhere
// import { supabase } from './lib/supabase'

// ❌ Bad: Create new client per request
app.get('/users', async (req, res) => {
  const supabase = createClient(URL, ANON_KEY) // ❌ Creates new client!
  const { data } = await supabase.from('users').select('*')
  res.json(data)
})
```

#### 2. Query Optimization

```typescript
// ✅ Good: Select only needed columns
const { data } = await supabase
  .from('users')
  .select('id, email, name')

// ✅ Good: Use indexes
CREATE INDEX idx_users_email ON users(email);

// ❌ Bad: Select everything
const { data } = await supabase
  .from('users')
  .select('*')
```

#### 3. Batch Operations

```typescript
// ✅ Good: Batch insert
const { data } = await supabase
  .from('users')
  .insert([
    { email: 'user1@example.com' },
    { email: 'user2@example.com' },
    { email: 'user3@example.com' }
  ])

// ❌ Bad: Multiple single inserts
for (const user of users) {
  await supabase.from('users').insert(user)
}
```

### Security Best Practices

#### 1. Row Level Security (RLS)

```sql
-- ✅ Good: Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Users can only view their own data
CREATE POLICY "Users view own data" 
  ON users FOR SELECT 
  USING (auth.uid() = id);

-- Users can only update their own data
CREATE POLICY "Users update own data" 
  ON users FOR UPDATE 
  USING (auth.uid() = id);

-- ❌ Bad: No RLS (anyone can access all data)
-- (RLS not enabled)
```

#### 2. Service Role Key Protection

```typescript
// ✅ Good: Service role only in backend
// server/api/users.ts
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY // ✅ Backend only
)

// ❌ Bad: Service role in frontend
// client/src/lib/supabase.ts
const supabase = createClient(
  URL,
  'eyJ...service_role...' // ❌ Exposed to users!
)
```

#### 3. Input Validation

```typescript
// ✅ Good: Validate input
const { data, error } = await supabase
  .from('posts')
  .insert({
    title: req.body.title?.slice(0, 200), // Limit length
    content: req.body.content
  })

if (error) {
  return res.status(400).json({ error: error.message })
}

// ❌ Bad: No validation
const { data } = await supabase
  .from('posts')
  .insert(req.body) // ❌ Unsanitized input
```

### Code Organization

#### 1. Repository Pattern

```typescript
// src/repositories/user.repository.ts
export class UserRepository {
  constructor(private supabase: SupabaseClient) {}
  
  async findById(id: string) {
    const { data } = await this.supabase
      .from('users')
      .select('*')
      .eq('id', id)
      .single()
    return data
  }
  
  async create(email: string) {
    const { data } = await this.supabase
      .from('users')
      .insert({ email })
      .select()
      .single()
    return data
  }
}

// Usage
const userRepo = new UserRepository(supabase)
const user = await userRepo.findById('123')
```

#### 2. Service Layer

```typescript
// src/services/auth.service.ts
export class AuthService {
  constructor(private supabase: SupabaseClient) {}
  
  async signUp(email: string, password: string) {
    const { data, error } = await this.supabase.auth.signUp({
      email,
      password
    })
    
    if (error) throw new Error(error.message)
    return data
  }
  
  async signIn(email: string, password: string) {
    const { data, error } = await this.supabase.auth.signInWithPassword({
      email,
      password
    })
    
    if (error) throw new Error(error.message)
    return data
  }
}
```

---

## ⚠️ Anti-Patterns

### Anti-Pattern 1: Using pgmem for Production Testing

```sql
-- ❌ Bad: Testing RLS with pgmem
-- pgmem doesn't enforce RLS (runs as superuser)
-- Your tests pass but fail in production!

-- ✅ Good: Use native or wasm engine for RLS testing
tinbase start --engine native
```

**Why it's wrong:** pgmem runs as superuser, so RLS policies are created but never enforced. Tests pass but fail in production.

### Anti-Pattern 2: Exposing Service Role Key

```typescript
// ❌ Bad: Service role in frontend
const supabase = createClient(
  URL,
  'eyJ...service_role...' // ❌ Anyone can see this!
)

// ✅ Good: Use anon key in frontend, service role in backend
// Frontend
const supabase = createClient(URL, ANON_KEY)

// Backend
const supabase = createClient(URL, SERVICE_ROLE_KEY)
```

**Why it's wrong:** Service role key bypasses all RLS policies. Exposing it gives users full database access.

### Anti-Pattern 3: Not Using Migrations

```bash
# ❌ Bad: Manual schema changes via Studio
# Changes aren't version controlled
# Team members have different schemas

# ✅ Good: Use migrations
# supabase/migrations/001_add_users.sql
CREATE TABLE users (...);
```

**Why it's wrong:** Manual changes aren't tracked, causing schema drift between team members and environments.

### Anti-Pattern 4: Ignoring Error Handling

```typescript
// ❌ Bad: No error handling
const { data } = await supabase
  .from('users')
  .insert({ email: 'test@example.com' })
// What if this fails?

// ✅ Good: Proper error handling
const { data, error } = await supabase
  .from('users')
  .insert({ email: 'test@example.com' })

if (error) {
  console.error('Insert failed:', error)
  throw new Error('Failed to create user')
}

return data
```

**Why it's wrong:** Unhandled errors crash applications and make debugging difficult.

### Anti-Pattern 5: Creating Clients Repeatedly

```typescript
// ❌ Bad: New client per request
app.get('/users', async (req, res) => {
  const supabase = createClient(URL, ANON_KEY) // ❌ Wasteful!
  const { data } = await supabase.from('users').select('*')
  res.json(data)
})

// ✅ Good: Reuse client
const supabase = createClient(URL, ANON_KEY) // Create once

app.get('/users', async (req, res) => {
  const { data } = await supabase.from('users').select('*')
  res.json(data)
})
```

**Why it's wrong:** Creating clients repeatedly wastes resources and can lead to connection issues.

### Anti-Pattern 6: Not Using Indexes

```sql
-- ❌ Bad: No index on frequently queried column
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id UUID,
  title TEXT
);

-- Query is slow!
SELECT * FROM posts WHERE user_id = '123';

-- ✅ Good: Add index
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Now query is fast!
```

**Why it's wrong:** Missing indexes cause slow queries as tables grow.

### Anti-Pattern 7: Hardcoding Credentials

```typescript
// ❌ Bad: Hardcoded credentials
const supabase = createClient(
  'http://127.0.0.1:54321',
  'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
)

// ✅ Good: Environment variables
const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
)
```

**Why it's wrong:** Hardcoded credentials leak in version control and can't change per environment.

### Anti-Pattern 8: Not Handling Realtime Subscriptions

```typescript
// ❌ Bad: Subscribe without cleanup
useEffect(() => {
  supabase
    .channel('posts')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'posts' }, 
      (payload) => console.log(payload))
    .subscribe()
  // ❌ Never unsubscribes - memory leak!
}, [])

// ✅ Good: Cleanup subscription
useEffect(() => {
  const channel = supabase
    .channel('posts')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'posts' }, 
      (payload) => console.log(payload))
    .subscribe()
  
  return () => {
    supabase.removeChannel(channel) // ✅ Cleanup
  }
}, [])
```

**Why it's wrong:** Uncleaned subscriptions cause memory leaks and duplicate events.

---

## ⚡ Performance Considerations

### Memory Usage by Engine

| Engine | Boot Memory | Under Load | Browser Support |
|--------|-------------|------------|-----------------|
| **native** | ~59 MB | ~66 MB | ❌ No |
| **wasm** | ~575 MB | ~650 MB | ✅ Yes |
| **pgmem** | ~50 MB | ~60 MB | ✅ Yes |
| **binary** | ~49 MB | ~66 MB | ❌ No |

**📊 Benchmark:** Native engine uses **11x less memory** than WASM at boot.

### Query Performance

#### Index Strategy

```sql
-- ✅ Good: Composite index for common queries
CREATE INDEX idx_posts_user_created 
  ON posts(user_id, created_at DESC);

-- Query uses index efficiently
SELECT * FROM posts 
WHERE user_id = '123' 
ORDER BY created_at DESC;

-- ❌ Bad: Multiple single-column indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at);
-- Query can only use one index
```

#### Connection Pooling

```typescript
// ✅ Good: Single client, reused connections
const supabase = createClient(URL, ANON_KEY)

// All requests reuse the same connection pool
await supabase.from('users').select('*')
await supabase.from('posts').select('*')

// ❌ Bad: Multiple clients, multiple pools
const client1 = createClient(URL, ANON_KEY)
const client2 = createClient(URL, ANON_KEY)
// Each has its own connection pool
```

### Realtime Performance

```typescript
// ✅ Good: Broadcast only changed columns
const channel = supabase
  .channel('posts')
  .on('postgres_changes', {
    event: 'UPDATE',
    schema: 'public',
    table: 'posts'
  }, (payload) => {
    // Only receive changed columns
    console.log('Changed:', payload.new)
  })
  .subscribe()

// ❌ Bad: Subscribe to all changes
const channel = supabase
  .channel('posts')
  .on('postgres_changes', {
    event: '*',
    schema: 'public',
    table: 'posts'
  }, (payload) => {
    // Receives full row every time
    console.log('Changed:', payload.new)
  })
  .subscribe()
```

### Edge Function Performance

```javascript
// ✅ Good: Minimize cold starts
// Keep functions warm with periodic invocations
// Use lightweight dependencies

// ❌ Bad: Heavy initialization
Deno.serve(async (req) => {
  // ❌ Loads large library on every invocation
  const _ = await import('npm:lodash')
  
  const data = await req.json()
  return new Response(_.toString(data))
})
```

### Performance Monitoring

```typescript
// Add performance monitoring
const start = Date.now()

const { data, error } = await supabase
  .from('users')
  .select('*')

const duration = Date.now() - start
console.log(`Query took ${duration}ms`)

if (duration > 1000) {
  console.warn('Slow query detected!')
}
```

### Optimization Checklist

- [ ] Use appropriate engine for platform (native on macOS/Linux)
- [ ] Create indexes on frequently queried columns
- [ ] Reuse Supabase client instances
- [ ] Select only needed columns (avoid `SELECT *`)
- [ ] Use batch operations for multiple inserts/updates
- [ ] Implement connection pooling
- [ ] Monitor query performance
- [ ] Use RLS efficiently (avoid overly complex policies)
- [ ] Cache frequently accessed data
- [ ] Clean up realtime subscriptions

---

## 🔒 Security Considerations

### Authentication & Authorization

#### JWT Token Management

```typescript
// ✅ Good: Store tokens securely
const { data, error } = await supabase.auth.signIn({
  email: 'user@example.com',
  password: 'password'
})

// Store in httpOnly cookie (backend)
// or secure storage (mobile)

// ❌ Bad: Store in localStorage (vulnerable to XSS)
localStorage.setItem('token', data.session.access_token)
```

**Why:** `localStorage` is vulnerable to XSS attacks. Use `httpOnly` cookies or secure storage.

#### Row Level Security (RLS)

```sql
-- ✅ Good: Comprehensive RLS policies
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Users can only view their own documents
CREATE POLICY "Users view own documents"
  ON documents FOR SELECT
  USING (auth.uid() = user_id);

-- Users can only insert documents for themselves
CREATE POLICY "Users insert own documents"
  ON documents FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Users can only update their own documents
CREATE POLICY "Users update own documents"
  ON documents FOR UPDATE
  USING (auth.uid() = user_id);

-- Users can only delete their own documents
CREATE POLICY "Users delete own documents"
  ON documents FOR DELETE
  USING (auth.uid() = user_id);

-- ❌ Bad: No RLS (anyone can do anything)
-- (RLS not enabled)
```

### SQL Injection Prevention

```typescript
// ✅ Good: Use parameterized queries
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail) // ✅ Parameterized

// ❌ Bad: String concatenation
const { data } = await supabase
  .from('users')
  .select(`* WHERE email = '${userEmail}'`) // ❌ SQL injection risk!
```

**Why:** Parameterized queries prevent SQL injection attacks.

### Input Validation

```typescript
// ✅ Good: Validate and sanitize input
import { z } from 'zod'

const UserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

try {
  const validated = UserSchema.parse(req.body)
  const { data } = await supabase
    .from('users')
    .insert(validated)
} catch (error) {
  return res.status(400).json({ error: 'Invalid input' })
}

// ❌ Bad: No validation
const { data } = await supabase
  .from('users')
  .insert(req.body) // ❌ Unsanitized
```

### CORS Configuration

```typescript
// ✅ Good: Restrict CORS origins
const allowedOrigins = [
  'https://myapp.com',
  'https://admin.myapp.com'
]

const corsHeaders = {
  'Access-Control-Allow-Origin': allowedOrigins.includes(origin) ? origin : '',
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization'
}

// ❌ Bad: Allow all origins
const corsHeaders = {
  'Access-Control-Allow-Origin': '*' // ❌ Too permissive
}
```

### Secret Management

```bash
# ✅ Good: Use environment variables
# .env (git-ignored)
SUPABASE_JWT_SECRET=your-secret-key
STRIPE_API_KEY=sk_live_...

# ❌ Bad: Hardcoded secrets
const jwtSecret = 'my-secret-key' // ❌ In source code!
```

**Best Practices:**
- ✅ Use environment variables
- ✅ Use secret management services (AWS Secrets Manager, HashiCorp Vault)
- ✅ Rotate secrets regularly
- ✅ Never commit secrets to version control
- ✅ Use different secrets for dev/staging/production

### Rate Limiting

```typescript
// ✅ Good: Implement rate limiting
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // Limit each IP to 100 requests per window
})

app.use('/api/', limiter)

// ❌ Bad: No rate limiting
// Vulnerable to abuse and DoS attacks
```

### Data Encryption

```sql
-- ✅ Good: Encrypt sensitive data
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT NOT NULL,
  -- Encrypt SSN at database level
  ssn TEXT ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = ssn_key)
);

-- ❌ Bad: Store sensitive data in plaintext
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT NOT NULL,
  ssn TEXT NOT NULL -- ❌ Plaintext!
);
```

### Security Checklist

- [ ] Enable RLS on all tables
- [ ] Use parameterized queries (prevent SQL injection)
- [ ] Validate all user input
- [ ] Store secrets in environment variables
- [ ] Use httpOnly cookies for tokens
- [ ] Implement proper CORS configuration
- [ ] Add rate limiting to API endpoints
- [ ] Encrypt sensitive data at rest
- [ ] Use HTTPS in production
- [ ] Regularly audit RLS policies
- [ ] Monitor for suspicious activity
- [ ] Keep dependencies updated
- [ ] Implement proper error handling (don't leak sensitive info)

---

## 🔧 Troubleshooting Guide

### Common Issues and Solutions

#### Issue 1: Port Already in Use

**Symptom:**
```
Error: Port 54321 is already in use
```

**Solution:**
```bash
# Find process using port
lsof -i :54321  # macOS/Linux
netstat -ano | findstr :54321  # Windows

# Kill process
kill -9 <PID>  # macOS/Linux
taskkill /PID <PID> /F  # Windows

# Or use different port
tinbase start --port 54322
```

**Prevention:**
```bash
# Use environment variable
export TINBASE_PORT=54321
```

#### Issue 2: Engine Download Fails

**Symptom:**
```
Error: Failed to download native engine binaries
```

**Solution:**
```bash
# Clear cache and retry
rm -rf ~/.cache/tinbase
tinbase start

# Or use WASM engine instead
tinbase start --engine wasm
```

**Prevention:**
```bash
# Ensure stable internet connection on first run
# Binaries are cached after first download
```

#### Issue 3: Migration Failures

**Symptom:**
```
Error: Migration failed: relation "users" already exists
```

**Solution:**
```bash
# Check migration status
tinbase status

# Reset database (WARNING: deletes all data)
tinbase db reset

# Or manually fix migration
# Remove duplicate CREATE TABLE statement
```

**Prevention:**
```sql
-- Use IF NOT EXISTS
CREATE TABLE IF NOT EXISTS users (...);

-- Or check before creating
DO $$ 
BEGIN
  IF NOT EXISTS (SELECT FROM pg_tables WHERE tablename = 'users') THEN
    CREATE TABLE users (...);
  END IF;
END $$;
```

#### Issue 4: RLS Not Working

**Symptom:**
```typescript
// RLS policy exists but users can still see all data
const { data } = await supabase
  .from('users')
  .select('*')
// Returns all users instead of just current user
```

**Solution:**
```sql
-- Check if RLS is enabled
SELECT tablename, rowsecurity 
FROM pg_tables 
WHERE tablename = 'users';

-- Enable RLS if not enabled
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Verify policy exists
SELECT * FROM pg_policies 
WHERE tablename = 'users';

-- Recreate policy if needed
DROP POLICY "Users view own data" ON users;
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);
```

**Common Cause:** Using pgmem engine (doesn't enforce RLS).

#### Issue 5: Webhook Not Firing

**Symptom:**
```
Webhooks configured but not receiving events
```

**Solution:**
```typescript
// Check webhook configuration
const webhooks = await backend.webhooks.list()
console.log('Registered webhooks:', webhooks)

// Verify webhook URL is accessible
curl -X POST https://api.example.com/webhook \
  -H "Content-Type: application/json" \
  -d '{"test": true}'

// Check webhook logs
SELECT * FROM webhook_logs 
ORDER BY created_at DESC 
LIMIT 10;
```

**Common Causes:**
- Webhook URL not accessible
- Firewall blocking outgoing requests
- Webhook handler returning non-2xx status

#### Issue 6: Edge Function Errors

**Symptom:**
```
Error: Function execution failed
```

**Solution:**
```bash
# Check function logs
tinbase logs --function hello

# Test function locally
curl -X POST http://127.0.0.1:54321/functions/v1/hello \
  -H "Authorization: Bearer $ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Test"}'

# Verify function file exists
ls -la supabase/functions/hello/
```

**Common Causes:**
- Syntax errors in function code
- Missing dependencies
- Incorrect file path
- Permission issues

#### Issue 7: Realtime Not Working

**Symptom:**
```typescript
// Subscribed but not receiving updates
const channel = supabase
  .channel('posts')
  .on('postgres_changes', {...}, callback)
  .subscribe()
// No events received
```

**Solution:**
```sql
-- Check if publication exists
SELECT * FROM pg_publication 
WHERE pubname = 'supabase_realtime';

-- Enable replication for table
ALTER PUBLICATION supabase_realtime ADD TABLE posts;

-- Check RLS (realtime respects RLS)
SELECT tablename, rowsecurity 
FROM pg_tables 
WHERE tablename = 'posts';
```

**Common Causes:**
- Table not added to publication
- RLS blocking replication
- Using pgmem engine (no realtime CDC)

#### Issue 8: Type Generation Fails

**Symptom:**
```
Error: Failed to generate types
```

**Solution:**
```bash
# Ensure database is running
tinbase start

# Generate types with explicit output
tinbase gen types --output ./src/types/database.ts

# Check for schema errors
tinbase status
```

**Common Causes:**
- Database not running
- Invalid SQL in migrations
- Permission issues

### Debugging Tips

#### Enable Verbose Logging

```bash
# Start with debug logging
DEBUG=* tinbase start

# Or specific module
DEBUG=tinbase:handler tinbase start
```

#### Check Database Logs

```bash
# Native engine logs
tail -f ~/.tinbase/logs/tinbase.log

# Or via Studio
# Studio → SQL Editor → Query logs
```

#### Monitor Network Requests

```typescript
// Enable Supabase client logging
const supabase = createClient(URL, ANON_KEY, {
  debug: true // ✅ Enable logging
})

// All requests will be logged to console
```

#### Database Inspection

```typescript
// Check current user
const { data } = await supabase.auth.getUser()
console.log('Current user:', data.user)

// Check RLS policies
const { data } = await supabase
  .from('pg_policies')
  .select('*')

// Check table sizes
const { data } = await supabase
  .from('pg_tables')
  .select('*')
```

### Getting Help

- 📚 **Documentation:** https://www.tinbase.dev/docs
- 💬 **Discord:** [Tinbase Discord](https://discord.gg/tinbase)
- 🐛 **Issues:** [GitHub Issues](https://github.com/tinbase/tinbase/issues)
- 💡 **Examples:** [GitHub Examples](https://github.com/tinbase/tinbase/tree/main/examples)

---

## 🚀 Migration Guide

### Migrating from Supabase Cloud to Tinbase

#### Step 1: Export Supabase Schema

```bash
# Install Supabase CLI
npm install -g supabase

# Link to your Supabase project
supabase link --project-ref your-project-id

# Pull schema
supabase db dump --schema public > schema.sql

# Export migrations
supabase migration list
# Note down all migration IDs
```

#### Step 2: Prepare Tinbase Project

```bash
# Create new project
mkdir my-tinbase-project && cd my-tinbase-project

# Initialize Tinbase
npx tinbase start

# Create migrations directory
mkdir -p supabase/migrations
```

#### Step 3: Convert Migrations

```bash
# Convert Supabase migrations to Tinbase format
# Supabase: supabase/migrations/20240101000000_initial.sql
# Tinbase: supabase/migrations/20240101000000_initial.sql

# Copy migration files
cp -r ~/.supabase/migrations/* supabase/migrations/

# Update migration tracking table
tinbase migrate
```

#### Step 4: Export Data

```bash
# Export data from Supabase
supabase db dump --data --schema public > data.sql

# Import to Tinbase
psql -h localhost -p 54321 -U postgres -f data.sql postgres
```

#### Step 5: Update Application Code

```typescript
// Before (Supabase Cloud)
const supabase = createClient(
  'https://xyzcompany.supabase.co',
  'eyJ...anon-key...'
)

// After (Tinbase)
const supabase = createClient(
  'http://127.0.0.1:54321',
  'eyJ...anon-key...'
)

// ✅ No other changes needed!
```

#### Step 6: Test Thoroughly

```bash
# Run all tests
npm test

# Test authentication
# Test database operations
# Test storage
# Test realtime
# Test edge functions
```

### Migrating from Docker-based Supabase

If you're currently running Supabase via Docker:

```bash
# Stop Docker Supabase
docker-compose down

# Start Tinbase
npx tinbase start

# Update connection strings in .env
# From: postgresql://postgres:password@localhost:54322/postgres
# To: postgresql://postgres@localhost:54321/postgres
```

### Compatibility Matrix

| Feature | Supabase Cloud | Tinbase Native | Tinbase WASM | Tinbase pgmem |
|---------|----------------|----------------|--------------|---------------|
| REST API | ✅ | ✅ | ✅ | ✅ |
| Auth | ✅ | ✅ | ✅ | ✅ |
| Storage | ✅ | ✅ | ✅ | ✅ |
| Realtime | ✅ | ✅ | ✅ | ⚠️ Partial |
| Edge Functions | ✅ | ✅ | ✅ | ✅ |
| RLS | ✅ | ✅ | ✅ | ⚠️ Not enforced |
| Cron | ✅ | ✅ | ✅ | ❌ |
| pgmq | ✅ | ✅ | ✅ | ❌ |

**Migration Difficulty:** ⭐ Easy (95% compatible)

---

## 🏋️ Practice Exercises

### Exercise 1: Build a Blog API

**Difficulty:** ⭐⭐ Intermediate  
**Time:** 30 minutes

#### Objective
Create a complete blog API with users, posts, and comments using Tinbase.

#### Requirements
1. Create tables: `users`, `posts`, `comments`
2. Implement RLS policies
3. Create migrations
4. Seed with sample data
5. Test CRUD operations

#### Solution

**Step 1: Create Migrations**

```sql
-- supabase/migrations/001_create_users.sql
CREATE TABLE users (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT now()
);

ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users view all profiles"
  ON users FOR SELECT
  USING (true);

CREATE POLICY "Users insert own profile"
  ON users FOR INSERT
  WITH CHECK (auth.uid() = id);

-- supabase/migrations/002_create_posts.sql
CREATE TABLE posts (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  published BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT now(),
  updated_at TIMESTAMP DEFAULT now()
);

ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can view published posts"
  ON posts FOR SELECT
  USING (published = true OR auth.uid() = user_id);

CREATE POLICY "Users can create own posts"
  ON posts FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own posts"
  ON posts FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own posts"
  ON posts FOR DELETE
  USING (auth.uid() = user_id);

-- supabase/migrations/003_create_comments.sql
CREATE TABLE comments (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT now()
);

ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can view comments on published posts"
  ON comments FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM posts 
      WHERE posts.id = comments.post_id 
      AND posts.published = true
    )
  );

CREATE POLICY "Users can create comments"
  ON comments FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Create indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_comments_post_id ON comments(post_id);
```

**Step 2: Seed Data**

```sql
-- supabase/seed.sql
INSERT INTO users (email, name) VALUES
  ('alice@example.com', 'Alice'),
  ('bob@example.com', 'Bob'),
  ('charlie@example.com', 'Charlie');

INSERT INTO posts (user_id, title, content, published) VALUES
  ((SELECT id FROM users WHERE email = 'alice@example.com'), 
   'Getting Started with Tinbase', 
   'Tinbase is amazing for local development...', 
   true),
  ((SELECT id FROM users WHERE email = 'bob@example.com'), 
   'My First Blog Post', 
   'Hello world!', 
   true);

INSERT INTO comments (post_id, user_id, content) VALUES
  ((SELECT id FROM posts WHERE title = 'Getting Started with Tinbase'),
   (SELECT id FROM users WHERE email = 'bob@example.com'),
   'Great post!');
```

**Step 3: Test API**

```typescript
// src/test-blog.ts
import { createClient } from '@supabase/supabase-js'

const supabase = createClient('http://127.0.0.1:54321', ANON_KEY)

async function testBlog() {
  // 1. Create user
  const { data: user } = await supabase.auth.signUp({
    email: 'test@example.com',
    password: 'password123'
  })
  console.log('Created user:', user)

  // 2. Create post
  const { data: post } = await supabase
    .from('posts')
    .insert({
      title: 'Test Post',
      content: 'This is a test post',
      published: true
    })
    .select()
    .single()
  console.log('Created post:', post)

  // 3. Add comment
  const { data: comment } = await supabase
    .from('comments')
    .insert({
      post_id: post.id,
      content: 'Great post!'
    })
    .select()
    .single()
  console.log('Created comment:', comment)

  // 4. Query with joins
  const { data: postsWithAuthors } = await supabase
    .from('posts')
    .select(`
      *,
      users (email, name)
    `)
    .eq('published', true)
  console.log('Published posts:', postsWithAuthors)
}

testBlog()
```

**Expected Result:** All CRUD operations work with RLS enforcement.

---

### Exercise 2: Implement Real-time Notifications

**Difficulty:** ⭐⭐⭐ Advanced  
**Time:** 45 minutes

#### Objective
Build a real-time notification system using Tinbase's realtime and webhooks.

#### Requirements
1. Create `notifications` table
2. Set up realtime subscription
3. Configure webhook for external notifications
4. Implement Edge Function for notification processing
5. Test end-to-end flow

#### Solution

**Step 1: Create Notifications Table**

```sql
-- supabase/migrations/004_create_notifications.sql
CREATE TABLE notifications (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  message TEXT NOT NULL,
  read BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT now()
);

ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users view own notifications"
  ON notifications FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users update own notifications"
  ON notifications FOR UPDATE
  USING (auth.uid() = user_id);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
```

**Step 2: Configure Realtime**

```typescript
// src/realtime.ts
import { createClient } from '@supabase/supabase-js'

const supabase = createClient('http://127.0.0.1:54321', ANON_KEY)

// Subscribe to notifications
const channel = supabase
  .channel('notifications')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'notifications',
      filter: `user_id=eq.${userId}`
    },
    (payload) => {
      console.log('New notification:', payload.new)
      // Show browser notification
      showBrowserNotification(payload.new)
    }
  )
  .subscribe()

// Cleanup on unmount
return () => {
  supabase.removeChannel(channel)
}
```

**Step 3: Configure Webhook**

```json
// supabase/webhooks.json
{
  "webhooks": [
    {
      "id": "send-notification-email",
      "table": "notifications",
      "events": ["INSERT"],
      "url": "https://api.example.com/send-email",
      "headers": {
        "Authorization": "Bearer email-service-token"
      }
    }
  ]
}
```

**Step 4: Create Edge Function**

```javascript
// supabase/functions/process-notification/index.mjs
Deno.serve(async (req) => {
  const { type, title, message, user_id } = await req.json()
  
  // Send push notification
  await sendPushNotification(user_id, {
    title,
    body: message
  })
  
  // Send email if important
  if (type === 'important') {
    await sendEmail(user_id, title, message)
  }
  
  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' }
  }
})
```

**Step 5: Test End-to-End**

```typescript
// Test notification flow
async function testNotifications() {
  // 1. Create notification
  const { data } = await supabase
    .from('notifications')
    .insert({
      user_id: userId,
      type: 'comment',
      title: 'New Comment',
      message: 'Someone commented on your post'
    })
  
  // 2. Realtime subscription receives it
  // 3. Webhook triggers email
  // 4. Edge Function processes it
}
```

**Expected Result:** Real-time notifications work with webhook integration.

---

### Exercise 3: Build a File Storage System

**Difficulty:** ⭐⭐ Intermediate  
**Time:** 40 minutes

#### Objective
Implement a file storage system with user uploads, access control, and image processing.

#### Requirements
1. Create storage bucket with access policies
2. Implement file upload with validation
3. Add image thumbnail generation
4. Implement access control
5. Create download URLs with expiration

#### Solution

**Step 1: Create Storage Bucket**

```typescript
// src/storage.ts
const { data, error } = await supabase
  .from('storage.buckets')
  .insert({
    id: 'user-files',
    name: 'User Files',
    public: false,
    file_size_limit: 5242880, // 5 MB
    allowed_mime_types: ['image/jpeg', 'image/png', 'application/pdf']
  })
```

**Step 2: Upload File with Validation**

```typescript
async function uploadFile(userId: string, file: File) {
  // Validate file
  if (file.size > 5 * 1024 * 1024) {
    throw new Error('File too large (max 5 MB)')
  }
  
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }
  
  // Upload
  const filePath = `${userId}/${Date.now()}_${file.name}`
  const { data, error } = await supabase
    .storage
    .from('user-files')
    .upload(filePath, file, {
      cacheControl: '3600',
      upsert: false
    })
  
  if (error) throw error
  return data
}
```

**Step 3: Generate Thumbnails**

```javascript
// supabase/functions/generate-thumbnail/index.mjs
Deno.serve(async (req) => {
  const { bucket, path } = await req.json()
  
  // Download file
  const file = await supabase.storage
    .from(bucket)
    .download(path)
  
  // Generate thumbnail (using Canvas API)
  const image = new Image()
  const canvas = new OffscreenCanvas(200, 200)
  const ctx = canvas.getContext('2d')
  
  await new Promise((resolve) => {
    image.onload = resolve
    image.src = URL.createObjectURL(file)
  })
  
  ctx.drawImage(image, 0, 0, 200, 200)
  const thumbnail = await canvas.convertToBlob()
  
  // Upload thumbnail
  const thumbnailPath = path.replace(/\.[^.]+$/, '_thumb.jpg')
  await supabase.storage
    .from(bucket)
    .upload(thumbnailPath, thumbnail)
  
  return new Response(JSON.stringify({ path: thumbnailPath }))
})
```

**Step 4: Access Control**

```typescript
// Get signed URL (expires in 1 hour)
const { data } = await supabase
  .storage
  .from('user-files')
  .createSignedUrl(filePath, 3600)

// Share file with other user
await supabase
  .storage
  .from('user-files')
  .createSignedUrl(filePath, 3600, {
    download: true
  })
```

**Step 5: Test System**

```typescript
async function testStorage() {
  // 1. Upload file
  const file = new File(['content'], 'test.jpg', { type: 'image/jpeg' })
  const uploadResult = await uploadFile(userId, file)
  console.log('Uploaded:', uploadResult)
  
  // 2. Get thumbnail
  const thumbnail = await generateThumbnail(uploadResult.path)
  console.log('Thumbnail:', thumbnail)
  
  // 3. Get download URL
  const { data } = await supabase
    .storage
    .from('user-files')
    .createSignedUrl(uploadResult.path, 3600)
  console.log('Download URL:', data.signedUrl)
}
```

**Expected Result:** Complete file storage system with upload, thumbnails, and access control.

---

## 📝 Test Your Understanding

### Questions

1. **What is the primary advantage of Tinbase over traditional Supabase development?**
   - A) Better performance
   - B) No Docker required
   - C) More features
   - D) Better documentation
   
   **Answer:** B) No Docker required

2. **Which engine should you use on Windows?**
   - A) native
   - B) wasm
   - C) pgmem
   - D) database-url
   
   **Answer:** B) wasm

3. **What is the memory usage of the native engine at boot?**
   - A) ~49 MB
   - B) ~59 MB
   - C) ~575 MB
   - D) ~1 GB
   
   **Answer:** B) ~59 MB

4. **Which engine does NOT enforce RLS?**
   - A) native
   - B) wasm
   - C) pgmem
   - D) database-url
   
   **Answer:** C) pgmem

5. **What protocol does the official Supabase SDK use to communicate with Tinbase?**
   - A) gRPC
   - B) GraphQL
   - C) REST/WebSocket
   - D) WebSockets only
   
   **Answer:** C) REST/WebSocket

6. **How do you access the Studio dashboard?**
   - A) http://127.0.0.1:54321
   - B) http://127.0.0.1:54321/_
   - C) http://127.0.0.1:3000
   - D) http://localhost:8080
   
   **Answer:** B) http://127.0.0.1:54321/_

7. **What command generates TypeScript types?**
   - A) tinbase types
   - B) tinbase gen types
   - C) tinbase generate
   - D) tinbase schema
   
   **Answer:** B) tinbase gen types

8. **Which key should NEVER be exposed in frontend code?**
   - A) anon key
   - B) JWT token
   - C) service_role key
   - D) API URL
   
   **Answer:** C) service_role key

9. **What format do Edge Functions use?**
   - A) export default handler
   - B) Deno.serve(handler)
   - C) module.exports
   - D) Both A and B
   
   **Answer:** D) Both A and B

10. **How do you reset the database?**
    - A) tinbase reset
    - B) tinbase db reset
    - C) tinbase migrate --reset
    - D) tinbase clear
    
    **Answer:** B) tinbase db reset

11. **What is the default port for Tinbase?**
    - A) 3000
    - B) 54321
    - C) 8080
    - D) 8000
    
    **Answer:** B) 54321

12. **Which engine supports browser execution?**
    - A) native only
    - B) wasm only
    - C) pgmem only
    - D) Both B and C
    
    **Answer:** D) Both B and C

13. **What does RLS stand for?**
    - A) Remote Login Service
    - B) Row Level Security
    - C) Resource Locking System
    - D) Request Lifecycle Service
    
    **Answer:** B) Row Level Security

14. **How do you apply pending migrations?**
    - A) tinbase migrate
    - B) tinbase apply
    - C) tinbase update
    - D) Both A and B
    
    **Answer:** A) tinbase migrate

15. **What is the file extension for Edge Functions?**
    - A) .ts or .js or .mjs
    - B) .function
    - C) .edge
    - D) .supabase
    
    **Answer:** A) .ts or .js or .mjs

16. **Which command lists applied migrations?**
    - A) tinbase list
    - B) tinbase status
    - C) tinbase migrations
    - D) tinbase history
    
    **Answer:** B) tinbase status

17. **What is the maximum memory usage of WASM engine under load?**
    - A) ~59 MB
    - B) ~66 MB
    - C) ~575 MB
    - D) ~650 MB
    
    **Answer:** D) ~650 MB

18. **How do you connect to an external Postgres database?**
    - A) --external-db
    - B) --database-url
    - C) --remote
    - D) --connect
    
    **Answer:** B) --database-url

19. **What format are webhook payloads in?**
    - A) XML
    - B) JSON
    - C) Protocol Buffers
    - D) MessagePack
    
    **Answer:** B) JSON

20. **Which table stores cron job definitions?**
    - A) cron.jobs
    - B) cron.job
    - C) scheduled_jobs
    - D) pg_cron.jobs
    
    **Answer:** B) cron.job

---

## 🎤 Common Interview Questions

### Question 1: What is Tinbase and how does it differ from Supabase?

**Answer:** Tinbase is an open-source, Supabase-compatible backend that runs entirely in a single process without Docker or cloud infrastructure. Key differences:
- **No Docker required**: Runs as a single process
- **Multiple engines**: native (Postgres 17), wasm (PGlite), pgmem (in-memory)
- **Browser support**: Can run in-process in browser tabs
- **Single binary**: Can be compiled to a standalone executable
- **Local-first**: Designed for local development with optional cloud deployment

### Question 2: Explain Tinbase's architecture.

**Answer:** Tinbase uses a `(Request) => Response` fetch handler architecture:
1. Client sends request to single handler
2. Handler routes to appropriate service (REST, Auth, Storage, etc.)
3. Each service uses a swappable `DbEngine` adapter
4. Engine executes queries against database
5. Response flows back through the same path

This design enables:
- Universal compatibility (Node.js + browser)
- Swappable backends without code changes
- Minimal overhead
- Protocol compatibility with Supabase

### Question 3: When would you use the pgmem engine?

**Answer:** Use pgmem for:
- **Rapid prototyping**: Fastest startup time (~50 MB)
- **Unit testing**: No external dependencies
- **CI/CD pipelines**: Lightweight, no installation needed
- **Quick validation**: Test schema changes quickly

**Never use pgmem for:**
- Production (obviously)
- Testing RLS (not enforced)
- Testing LISTEN/NOTIFY (no-op)
- Performance testing

### Question 4: How does Tinbase achieve Supabase compatibility?

**Answer:** Tinbase implements Supabase's wire protocols exactly:
- **PostgREST**: RESTful database API
- **GoTrue**: Authentication service
- **Storage**: File upload/download
- **Realtime**: WebSocket-based live updates
- **Edge Functions**: Deno.serve() compatible

This means the official `@supabase/supabase-js` SDK works without modification.

### Question 5: What are the trade-offs of using WASM vs native engine?

**Answer:**

**WASM Engine:**
- ✅ Cross-platform (Windows, macOS, Linux, browser)
- ✅ Full feature parity
- ❌ Higher memory (~575 MB vs ~59 MB)
- ❌ Slower performance

**Native Engine:**
- ✅ Best performance
- ✅ Lower memory footprint
- ✅ Real Postgres semantics
- ❌ macOS/Linux only
- ❌ Requires binary download

**Choose native on macOS/Linux for development, WASM for Windows/browser.**

### Question 6: How do you secure a Tinbase application?

**Answer:** Security best practices:
1. **Enable RLS** on all tables
2. **Use anon key** in frontend, service_role in backend
3. **Validate input** to prevent SQL injection
4. **Use parameterized queries** (Supabase SDK does this automatically)
5. **Store secrets** in environment variables
6. **Implement CORS** properly
7. **Add rate limiting** to prevent abuse
8. **Use httpOnly cookies** for auth tokens
9. **Encrypt sensitive data** at rest
10. **Monitor** for suspicious activity

### Question 7: Explain how Edge Functions work in Tinbase.

**Answer:** Tinbase Edge Functions:
- Run **in-process** (not in separate Deno runtime)
- Use `Deno.serve(handler)` format (Supabase-compatible)
- Loaded from `supabase/functions/*/index.{ts,js,mjs}`
- Support Web APIs natively
- npm/jsr imports require bundling
- Access environment variables via `Deno.env`
- Return standard Web `Response` objects

**Key difference from Supabase:** In-process execution reduces overhead and simplifies deployment.

### Question 8: What is the purpose of the DbEngine adapter?

**Answer:** The `DbEngine` adapter provides:
- **Abstraction**: Services don't care about database implementation
- **Swappability**: Change engines without changing application code
- **Consistency**: Same interface across all engines
- **Flexibility**: Use native, wasm, pgmem, or external Postgres

Example:
```typescript
// Same code works with all engines
const backend = createBackend({
  engine: 'native' // Change to 'wasm', 'pgmem', or use databaseUrl
})
```

### Question 9: How do webhooks work in Tinbase?

**Answer:** Tinbase webhooks:
- Fire HTTP requests on database changes
- Use Supabase's exact payload format (`type/table/schema/record/old_record`)
- Configured via `createBackend({ webhooks })` or `webhooks.json`
- Support retry with exponential backoff
- Support filtering (only fire on specific conditions)
- Implemented natively (no pg_net extension needed)

Example payload:
```json
{
  "type": "INSERT",
  "table": "users",
  "schema": "public",
  "record": { "id": "123", "email": "user@example.com" },
  "old_record": null
}
```

### Question 10: What are the limitations of pgmem engine?

**Answer:** pgmem limitations:
- ❌ **No LISTEN/NOTIFY**: CDC synthesized in JS (not real)
- ❌ **RLS not enforced**: Runs as superuser
- ❌ **No cron jobs**: pg_cron not available
- ❌ **No pgmq**: Queue extension missing
- ❌ **Preview status**: Not production-ready
- ❌ **Local-dev only**: Never use in production

**Use pgmem only for:** Rapid prototyping, unit testing, CI/CD

---

## ❓ Question Bank

### Beginner Questions (1-17)

1. **What is Tinbase?**
   - A) A database
   - B) A Supabase-compatible local backend
   - C) A frontend framework
   - D) A cloud service
   
   **Answer:** B

2. **Do you need Docker to run Tinbase?**
   - A) Yes, always
   - B) No, never
   - C) Only for production
   - D) Only for Windows
   
   **Answer:** B

3. **What is the default port for Tinbase?**
   - A) 3000
   - B) 54321
   - C) 8080
   - D) 8000
   
   **Answer:** B

4. **Which SDK works with Tinbase?**
   - A) Firebase SDK
   - B) Official Supabase SDK
   - C) Custom SDK only
   - D) MongoDB SDK
   
   **Answer:** B

5. **What command starts Tinbase?**
   - A) tinbase run
   - B) tinbase start
   - C) tinbase serve
   - D) tinbase up
   
   **Answer:** B

6. **Where are migrations stored?**
   - A) /migrations
   - B) /db/migrations
   - C) supabase/migrations
   - D) /src/migrations
   
   **Answer:** C

7. **What engine is default on macOS?**
   - A) wasm
   - B) pgmem
   - C) native
   - D) database-url
   
   **Answer:** C

8. **What is the anon key used for?**
   - A) Admin operations
   - B) Client-side operations
   - C) Database backup
   - D) Server configuration
   
   **Answer:** B

9. **What is the service_role key used for?**
   - A) Client-side operations
   - B) Admin operations (backend only)
   - C) Public access
   - D) Read-only operations
   
   **Answer:** B

10. **How do you access Studio?**
    - A) http://127.0.0.1:54321
    - B) http://127.0.0.1:54321/_
    - C) http://localhost:3000
    - D) http://localhost:8080
    
    **Answer:** B

11. **What file format are migrations?**
    - A) .js
    - B) .sql
    - C) .ts
    - D) .json
    
    **Answer:** B

12. **What does RLS stand for?**
    - A) Remote Login Service
    - B) Row Level Security
    - C) Resource Locking System
    - D) Request Lifecycle Service
    
    **Answer:** B

13. **Can Tinbase run in a browser?**
    - A) No
    - B) Yes, with WASM engine
    - C) Yes, with native engine
    - D) Only with plugins
    
    **Answer:** B

14. **What is the memory usage of native engine at boot?**
    - A) ~49 MB
    - B) ~59 MB
    - C) ~100 MB
    - D) ~1 GB
    
    **Answer:** B

15. **How do you generate TypeScript types?**
    - A) tinbase types
    - B) tinbase gen types
    - C) tinbase generate
    - D) tinbase schema
    
    **Answer:** B

16. **What command resets the database?**
    - A) tinbase reset
    - B) tinbase db reset
    - C) tinbase clear
    - D) tinbase wipe
    
    **Answer:** B

17. **What is seed.sql used for?**
    - A) Database migrations
    - B) Initial data seeding
    - C) Configuration
    - D) Type definitions
    
    **Answer:** B

### Intermediate Questions (18-35)

18. **Which engine should you use on Windows?**
    - A) native
    - B) wasm
    - C) pgmem
    - D) database-url
    
    **Answer:** B

19. **What is the maximum memory of WASM engine under load?**
    - A) ~59 MB
    - B) ~66 MB
    - C) ~575 MB
    - D) ~650 MB
    
    **Answer:** D

20. **Which engine does NOT enforce RLS?**
    - A) native
    - B) wasm
    - C) pgmem
    - D) database-url
    
    **Answer:** C

21. **How do you connect to external Postgres?**
    - A) --external
    - B) --database-url
    - C) --remote-db
    - D) --connect
    
    **Answer:** B

22. **What protocol does Supabase SDK use?**
    - A) gRPC
    - B) GraphQL
    - C) REST/WebSocket
    - D) WebSockets only
    
    **Answer:** C

23. **What is the purpose of DbEngine adapter?**
    - A) Database backup
    - B) Swappable database backends
    - C) Query optimization
    - D) Connection pooling
    
    **Answer:** B

24. **How do you configure webhooks?**
    - A) webhooks.json only
    - B) Code only
    - C) Both JSON and code
    - D) Studio only
    
    **Answer:** C

25. **What format are webhook payloads?**
    - A) XML
    - B) JSON
    - C) Protocol Buffers
    - D) MessagePack
    
    **Answer:** B

26. **How do you schedule cron jobs?**
    - A) cron.schedule()
    - B) CREATE CRON
    - C) SCHEDULE JOB
    - D) ADD CRON
    
    **Answer:** A

27. **What is the cron job tracking table?**
    - A) cron.jobs
    - B) cron.job
    - C) scheduled_jobs
    - D) pg_cron.jobs
    
    **Answer:** B

28. **What command lists migrations?**
    - A) tinbase list
    - B) tinbase status
    - C) tinbase migrations
    - D) tinbase history
    
    **Answer:** B

29. **What is Edge Functions runtime?**
    - A) Node.js
    - B) Deno
    - C) Bun
    - D) Browser
    
    **Answer:** B

30. **Where are Edge Functions stored?**
    - A) /functions
    - B) supabase/functions
    - C) /edge
    - D) src/functions
    
    **Answer:** B

31. **What is the single binary size?**
    - A) ~10 MB
    - B) ~58 MB
    - C) ~100 MB
    - D) ~500 MB
    
    **Answer:** B

32. **How do you apply migrations without starting server?**
    - A) tinbase apply
    - B) tinbase migrate
    - C) tinbase update
    - D) tinbase run-migrations
    
    **Answer:** B

33. **What is the purpose of db diff?**
    - A) Compare databases
    - B) Generate DDL for schema changes
    - C) Find performance issues
    - D) Backup database
    
    **Answer:** B

34. **Can you use npm packages in Edge Functions?**
    - A) Yes, directly
    - B) No, never
    - C) Yes, with bundling
    - D) Only in WASM engine
    
    **Answer:** C

35. **What authentication method does external Postgres use?**
    - A) Trust
    - B) SCRAM-SHA-256
    - C) MD5
    - D) Both B and C
    
    **Answer:** D

### Advanced Questions (36-50)

36. **What is the architecture pattern used by Tinbase?**
    - A) MVC
    - B) (Request) => Response handler
    - C) Microservices
    - D) Monolith
    
    **Answer:** B

37. **How does Tinbase achieve browser compatibility?**
    - A) Plugins
    - B) WASM engine
    - C) WebAssembly + in-process fetch
    - D) Browser API
    
    **Answer:** C

38. **What is the memory overhead of pgmem engine?**
    - A) ~49 MB
    - B) ~50 MB
    - C) ~59 MB
    - D) ~575 MB
    
    **Answer:** B

39. **Which feature is NOT available in pgmem?**
    - A) REST API
    - B) RLS enforcement
    - C) Edge Functions
    - D) Storage
    
    **Answer:** B

40. **How does Tinbase handle realtime in pgmem?**
    - A) Native PostgreSQL triggers
    - B) Synthesized in JavaScript
    - C) Not supported
    - D) WebSockets only
    
    **Answer:** B

41. **What is the purpose of the DbEngine interface?**
    - A) Database backup
    - B) Abstraction for swappable backends
    - C) Query optimization
    - D) Connection management
    
    **Answer:** B

42. **How do you enable RLS on a table?**
    - A) ALTER TABLE table ENABLE RLS
    - B) ENABLE RLS ON table
    - C) SET table RLS ON
    - D) CREATE RLS ON table
    
    **Answer:** A

43. **What is the webhook payload format?**
    - A) { type, table, schema, record, old_record }
    - B) { event, data }
    - C) { action, payload }
    - D) { change, table }
    
    **Answer:** A

44. **How do you unschedule a cron job?**
    - A) DROP CRON job
    - B) cron.unschedule()
    - C) DELETE FROM cron.job
    - D) Both B and C
    
    **Answer:** D

45. **What is the default JWT secret behavior?**
    - A) Fixed value
    - B) Randomly generated
    - C) From environment variable
    - D) From config file
    
    **Answer:** B

46. **Can Tinbase run without a supabase/ directory?**
    - A) No, required
    - B) Yes, it still boots
    - C) Only with WASM engine
    - D) Only in production
    
    **Answer:** B

47. **What is the purpose of createBackend() API?**
    - A) Create database
    - B) Programmatic backend configuration
    - C) Generate types
    - D) Start server
    
    **Answer:** B

48. **How does Studio compile?**
    - A) Multiple files
    - B) Single self-contained HTML
    - C) Node.js application
    - D) Desktop app
    
    **Answer:** B

49. **What authentication does native engine use?**
    - A) SCRAM-SHA-256
    - B) Trust authentication
    - C) MD5
    - D) OAuth
    
    **Answer:** B

50. **What is the primary use case for database-url engine?**
    - A) Production deployment
    - B) Local development
    - C) Connecting to existing Postgres
    - D) Browser development
    
    **Answer:** C

---

## 📊 Summary & Key Takeaways

### What You've Learned

🎉 **Congratulations!** You've completed a comprehensive deep dive into Tinbase. Here's what you've mastered:

### Core Concepts

1. **Architecture Understanding**
   - Tinbase uses a `(Request) => Response` fetch handler
   - All services share a swappable `DbEngine` adapter
   - Official Supabase SDK works without modification
   - In-process execution in browsers, HTTP server in Node.js

2. **Engine Selection**
   - **Native**: Best for macOS/Linux (~59 MB, full parity)
   - **WASM**: Cross-platform, browser support (~575 MB)
   - **pgmem**: Fast prototyping, testing (~50 MB, no RLS)
   - **database-url**: Connect to existing Postgres

3. **Development Workflow**
   - Install: `npm install -g tinbase`
   - Start: `tinbase start`
   - Migrate: `tinbase migrate`
   - Generate types: `tinbase gen types`
   - Reset: `tinbase db reset`

4. **Feature Parity**
   - ✅ REST API (PostgREST)
   - ✅ Authentication (GoTrue)
   - ✅ Storage
   - ✅ Realtime (WebSocket)
   - ✅ Edge Functions
   - ✅ Studio Dashboard
   - ✅ Migrations
   - ✅ RLS

5. **Best Practices**
   - Enable RLS on all tables
   - Never expose service_role key
   - Use environment variables for secrets
   - Reuse Supabase client instances
   - Create indexes on frequently queried columns
   - Validate all user input
   - Use migrations for schema changes

### Key Insights

💡 **The Tinbase Advantage:**
- **Zero Docker**: Start coding in seconds
- **Minimal overhead**: ~59 MB vs 2-4 GB for Docker
- **Drop-in replacement**: Use official Supabase SDK
- **Browser support**: Run backend in browser tabs
- **Single binary**: Deploy anywhere without Node.js

🔑 **When to Use Tinbase:**
- Local development (faster iteration)
- Offline development (flights, travel)
- Education and training (no infrastructure)
- CI/CD testing (no Docker needed)
- Rapid prototyping (instant setup)
- Edge computing (browser-based backend)

⚠️ **When NOT to Use:**
- Production deployments (use hosted Supabase)
- Testing RLS with pgmem (not enforced)
- High-memory environments with WASM (~575 MB)
- Applications requiring pg_cron/pgmq (use native)

### Performance Comparison

| Metric | Tinbase Native | Docker Supabase | Improvement |
|--------|----------------|-----------------|-------------|
| Boot time | ~2s | ~30s | **15x faster** |
| Memory at boot | ~59 MB | ~2 GB | **34x less** |
| Memory under load | ~66 MB | ~4 GB | **60x less** |
| Disk space | ~12 MB | ~500 MB | **42x less** |

### Migration Path

🔄 **From Supabase Cloud:**
1. Export schema: `supabase db dump`
2. Create Tinbase project
3. Convert migrations
4. Update connection strings
5. Test thoroughly

**Compatibility:** 95% compatible (only pg_cron/pgmq missing)

### Next Steps

📚 **Continue Learning:**
1. Build a project with Tinbase
2. Explore Edge Functions
3. Implement realtime features
4. Set up webhooks and cron jobs
5. Contribute to Tinbase on GitHub

🛠️ **Practice:**
1. Complete the 3 exercises in this tutorial
2. Build a real application
3. Experiment with different engines
4. Try browser-based development
5. Set up CI/CD with Tinbase

🚀 **Advanced Topics:**
- Custom DbEngine implementations
- Performance optimization
- Security hardening
- Production deployment strategies
- Multi-tenant architectures

---

## 📚 Further Reading & Resources

### Official Documentation
- 📖 **[Tinbase Documentation](https://www.tinbase.dev/docs)** - Official docs
- 🐙 **[GitHub Repository](https://github.com/tinbase/tinbase)** - Source code and examples
- 💬 **[Discord Community](https://discord.gg/tinbase)** - Get help and share projects
- 🐦 **[Twitter](https://twitter.com/tinbase)** - Updates and announcements

### Related Technologies
- 📚 **[Supabase Documentation](https://supabase.com/docs)** - Compare with hosted Supabase
- 🐘 **[PostgreSQL Documentation](https://www.postgresql.org/docs/)** - Database reference
- 🔷 **[PGlite](https://github.com/electric-sql/pglite)** - WASM Postgres
- 📦 **[pg-mem](https://github.com/oguimbal/pg-mem)** - In-memory Postgres

### Learning Resources
- 🎓 **[Supabase Tutorial](https://supabase.com/tutorials)** - Supabase concepts apply to Tinbase
- 🎥 **[YouTube: Tinbase Demo](https://youtube.com/tinbase)** - Video tutorials
- 📝 **[Blog: Local-First Development](https://example.com/local-first)** - Philosophy behind Tinbase

### Tools & Utilities
- 🔧 **[Supabase CLI](https://github.com/supabase/cli)** - Migration management
- 🎨 **[Supabase Studio](https://github.com/supabase/studio)** - Database UI
- 🔍 **[Postico](https://eggerapps.at/postico/)** - PostgreSQL client (macOS)
- 🖥️ **[pgAdmin](https://www.pgadmin.org/)** - PostgreSQL admin tool

### Community
- 💬 **[Discord](https://discord.gg/tinbase)** - Chat with community
- 🐛 **[GitHub Issues](https://github.com/tinbase/tinbase/issues)** - Report bugs
- 💡 **[GitHub Discussions](https://github.com/tinbase/tinbase/discussions)** - Ask questions
- 🌟 **[Star on GitHub](https://github.com/tinbase/tinbase)** - Show support

### Production Deployment
- 🚀 **[Deploy to VPS](https://www.tinbase.dev/docs/deployment)** - Self-host Tinbase
- ☁️ **[Cloud Providers](https://www.tinbase.dev/docs/cloud)** - Managed hosting options
- 🔒 **[Security Guide](https://www.tinbase.dev/docs/security)** - Production security

---

## 🎓 Final Thoughts

Tinbase represents a **significant shift** in how we approach local development. By eliminating Docker overhead and providing Supabase compatibility, it enables:

- **Faster development cycles** (15x faster boot time)
- **Lower resource usage** (60x less memory)
- **Better developer experience** (no Docker configuration)
- **New possibilities** (browser-based backends, offline development)

Whether you're building a prototype, learning Supabase, or developing offline, Tinbase provides the tools you need without the infrastructure complexity.

**Remember:** Tinbase is **95% compatible** with Supabase. You can develop locally with Tinbase and deploy to Supabase Cloud with minimal changes.

Happy coding! 🚀

---

**📝 Tutorial Metadata:**
- **Created:** 2026-01-09
- **Last Updated:** 2026-01-09
- **Version:** 1.0
- **Author:** AI Assistant
- **License:** MIT
- **Feedback:** [GitHub Issues](https://github.com/sandeep-mohanty/knowledge-base/issues)

---

*This tutorial was created following comprehensive tutorial preferences with deep-dive coverage, Mermaid diagrams, practice exercises, question banks, and real-world examples.*
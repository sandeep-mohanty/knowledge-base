# Aspire for JavaScript developers

Remember when Aspire was just for .NET folks? Yeah, those days are over. With Aspire 13, JavaScript and TypeScript developers get to join the party—and I’m not talking about some half-baked afterthought integration. This is first-class, full-featured support for orchestrating your JavaScript apps in distributed systems.

**Note**

To be clear, we’re not done yet, we have lots of work to do in fact, and the JavaScript support is being actively improved.

The [📦 `Aspire.Hosting.JavaScript`](https://www.nuget.org/packages/Aspire.Hosting.JavaScript) package (formerly `Aspire.Hosting.NodeJs`, because we renamed it to be less confusing—you’re welcome) brings comprehensive support for developing, debugging, and deploying JavaScript applications. Whether you’re building slick frontends with Vite, REST APIs with Express, or full-stack apps with Next.js, Aspire’s got your back.

## 🚀 Adding JavaScript applications to Aspire

Aspire 13 gives you three different ways to run JavaScript code, each tailored for specific scenarios. Think of them as different tools in your toolbox—you wouldn’t use a sledgehammer to hang a picture frame, right? (Well, you could, but results may vary 🤣!)

### 📦 JavaScript applications: The npm script runner

Got a `package.json` with scripts? `AddJavaScriptApp()` is your new best friend. It’s perfect for literally any JavaScript project that uses npm scripts—React, Angular, Vue, Express, Next.js, that weird side project you started at 2 AM, you name it.

```javascript
var builder = DistributedApplication.CreateBuilder(args);

var frontend = builder.AddJavaScriptApp("frontend", "../frontend");

builder.Build().Run();
```

Dead simple. By default, it runs your `"dev"` script during local development and your `"build"` script when you’re deploying. No fuss, no ceremony, just works.

#### 📥 Package manager support: Because one size doesn’t fit all

Look, I know the JavaScript ecosystem has opinions, lots of them, I get it…but Aspire supports them already. If your project has a `package.json`, npm is auto-configured as the default. But if you’re team Yarn or team pnpm (respect), switching is trivial:

```javascript
// npm (default) with custom arguments
var app = builder.AddJavaScriptApp("app", "../app")
    .WithNpm(installArgs: ["--legacy-peer-deps"]);

// Yarn
var yarnApp = builder.AddJavaScriptApp("yarn-app", "../yarn-app")
    .WithYarn();

// pnpm
var pnpmApp = builder.AddJavaScriptApp("pnpm-app", "../pnpm-app")
    .WithPnpm();

// Disable automatic installation (for pre-installed dependencies)
var noInstallApp = builder.AddJavaScriptApp("no-install", "../no-install")
    .WithNpm(install: false);
```

Here’s what’s happening under the hood (because details matter):

**npm’s got smarts:**

- Development: Uses `npm install` (the classic)
- Production: Automatically switches to `npm ci` if it spots a `package-lock.json` (faster, more reliable, chef’s kiss)

**Yarn plays nice:**

- Auto-detects your Yarn version by checking for `.yarnrc.yml` or the `.yarn` directory
- Yarn 2+: Uses `yarn install --immutable` in production when `yarn.lock` exists (no surprises)
- Yarn 1.x: Uses `yarn install --frozen-lockfile` in production (because legacy support matters)

**pnpm just works:**

- Uses `pnpm install --frozen-lockfile` in production when `pnpm-lock.yaml` exists
- Automatically enables pnpm via corepack in Docker builds (we’re not savages)

**Note**

The `install` parameter (default: `true`) controls whether packages are automatically installed before the application starts. Set to `false` to skip installation when dependencies are already in place.

#### 🔧 Script customization: Your package.json, your rules

Don’t like the default `"dev"` and `"build"` scripts? No problem. Override them:

```javascript
var app = builder.AddJavaScriptApp("app", "../app")
    .WithRunScript("local")    // Use "npm run local" in development
    .WithBuildScript("build:prod");  // Use "npm run build:prod" when publishing
```

Need to pass arguments? We got you:

```javascript
var app = builder.AddJavaScriptApp("app", "../app")
    .WithRunScript("dev", ["--port", "3000", "--host"]);
```

### ⚙️ Node.js applications: Direct execution, no package manager required

Sometimes you just want to run `node server.js` and call it a day. No npm scripts, no build steps, just pure Node.js execution. That’s where `AddNodeApp()` shines:

```javascript
var nodeApp = builder.AddNodeApp("node-app", "../node-app", "server.js")
    .WithHttpEndpoint(env: "PORT");
```

Perfect for simple Node.js scripts, microservices, background workers, or when you want direct control over the Node.js process without the npm script ceremony. (Sometimes less is more, you know?)

#### 📥 Adding dependencies to Node apps

Your Node app has dependencies? Of course it does. When there’s a `package.json`, npm kicks in automatically. You can mix and match package managers with npm scripts if that’s your jam:

```javascript
// Use Yarn with npm scripts
var nodeApp = builder.AddNodeApp("node-app", "../node-app", "server.js")
    .WithYarn()
    .WithRunScript("dev");  // Now uses "yarn run dev" instead of "node server.js"

// Use pnpm
var pnpmNode = builder.AddNodeApp("pnpm-node", "../pnpm-node", "server.js")
    .WithPnpm();
```

#### 🚀 What happens in production

When you publish `AddNodeApp()`, Aspire generates multi-stage Dockerfiles that are actually good (I know, shocking). Here’s what you get:

- **Build stage** installs dependencies and runs your build scripts
- **Runtime stage** copies only the built artifacts (smaller images, faster deploys, happier DevOps team)
- Runs as the non-privileged `node` user (because security isn’t optional)
- Sets proper entrypoint as `["node", "your-script.js"]` (no weird shell wrapping nonsense)

### ⚡ Vite applications: Frontend frameworks, but faster

If you’re building modern frontends with Vite (and you should be—it’s ridiculously fast), use `AddViteApp()` for Vite-specific optimizations:

```javascript
var viteApp = builder.AddViteApp("vite-app", "../vite-app");
```

Works beautifully with React, Vue, Svelte, Astro, or any Vite-based framework.

#### ⚙️ What Aspire does for you automatically

`AddViteApp()` is kind of magical. Behind the scenes, it:

- Configures an HTTP endpoint (port gets allocated dynamically—no more port conflicts!)
- Passes the `--port` argument to Vite with the allocated port
- Runs the “dev” script during development
- Runs the “build” script when publishing
- Handles the `--` separator correctly for pnpm (because pnpm is special and doesn’t strip it like npm/Yarn do)
- Generates production-ready Dockerfiles

All of this, without you lifting a finger. It’s almost too easy.

#### 📥 Package managers for Vite apps

Like `AddJavaScriptApp()`, Vite apps support all the package managers you’d expect:

```javascript
// npm (default)
var viteApp = builder.AddViteApp("vite-app", "../vite-app");

// Yarn
var yarnVite = builder.AddViteApp("yarn-vite", "../yarn-vite")
    .WithYarn();

// pnpm
var pnpmVite = builder.AddViteApp("pnpm-vite", "../pnpm-vite")
    .WithPnpm();
```

#### 📄 Custom Vite configs: For when defaults aren’t enough

Got multiple Vite configs for different environments? (Of course you do.) Point Aspire at the right one:

```javascript
var viteApp = builder.AddViteApp("vite-app", "../vite-app")
    .WithViteConfig("./vite.production.config.js");
```

The path is relative to your Vite app’s root directory. Pretty straightforward stuff.

#### 🔒 HTTPS configuration: Certificate trust without the trust issues

Here’s where things get clever. Aspire handles HTTPS for Vite without butchering your carefully crafted `vite.config.js`. When you configure HTTPS certificates, Aspire generates a _wrapper_ config that layers HTTPS settings on top:

```javascript
var viteApp = builder.AddViteApp("vite-app", "../vite-app")
    .WithHttpsEndpoint(env: "PORT")
    .WithHttpsDeveloperCertificate();
```

What happens under the hood:

1. Aspire finds your existing Vite config (or uses Vite’s default resolution)
2. Generates a wrapper config at `node_modules/.bin/aspire.{your-config}.js`
3. Passes cert paths through `TLS_CONFIG_PFX` and `TLS_CONFIG_PASSWORD` environment variables
4. Augments your config to use HTTPS if it’s not already set up

Your original `vite.config.js`? Untouched. Beautiful. This is how you do non-invasive tooling.

**Important**

Vite apps use HTTP by default. HTTPS is opt-in via `WithHttpsEndpoint()` and requires calling `WithHttpsDeveloperCertificate()` or providing a custom certificate.

## 🌐 Endpoints and networking: Ports without the pain

JavaScript apps love their environment variables for port configuration (it’s basically a universal constant). Aspire makes this dead simple:

```javascript
var app = builder.AddJavaScriptApp("app", "../app")
    .WithHttpEndpoint(port: 3000, env: "PORT");
```

What this does:

- Allocates port 3000 for your app
- Sets the `PORT` environment variable
- Registers the endpoint in Aspire’s service discovery (so other services can find you)
- Makes everything visible in the Aspire dashboard

Common port environment variables you’ll see in the wild:

- `PORT` – The universal standard. Express, Fastify, Koa, Next.js, basically everyone uses this
- `VITE_PORT` – Vite being Vite
- `HOST` – For binding to specific network interfaces (less common but still useful)

## 🔍 Service discovery: Finding your friends in the distributed system

This is where things get genuinely cool. JavaScript apps in Aspire automatically participate in service discovery. No configuration files, no service mesh complexity, just straightforward environment variables:

```javascript
var api = builder.AddProject<Projects.BackendApi>("api");

var frontend = builder.AddViteApp("frontend", "../frontend")
    .WithReference(api);
```

The frontend automatically receives environment variables like `API_HTTP` and `API_HTTPS`. Connect to your backend without hardcoding URLs (gasp, imagine that):

```typescript
// In your Vite/React application
const apiUrl = import.meta.env.API_HTTP || 'http://localhost:5000';

const response = await fetch(`${apiUrl}/api/data`);
const data = await response.json();
```

### 🔀 Using Vite proxy configuration

Alternatively, you can configure Vite to proxy API requests, eliminating the need to manage the API URL in your code. This approach is especially useful during development:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: process.env.APISERVICE_HTTP || 'http://localhost:5000',
        changeOrigin: true,
        secure: false,
      },
    },
  },
});
```

With this configuration, your frontend code becomes simpler:

```typescript
// No need to manage apiUrl - just make requests directly
const response = await fetch('/api/data');
const data = await response.json();
```

**Tip**

Using a Vite proxy configuration allows you to make API requests without explicitly managing URLs in your application code. The proxy automatically forwards requests to your backend service based on the environment variables provided by Aspire.

## 🗄️ Database connections: Connection strings that actually make sense

When you wire up databases in Aspire, you get connection information in multiple formats. Why? Because JavaScript’s Database library ecosystem is… diverse (that’s the polite way to put it). Some libraries want URIs, others want individual properties. Aspire gives you both:

```javascript
var postgres = builder.AddPostgres("postgres")
    .AddDatabase("appdb");

var api = builder.AddNodeApp("api", "../api")
    .WithReference(postgres);

builder.AddJavaScriptApp("client", "../client")
    .WithReference(api);
```

Your Node.js API app gets both URI format _and_ individual connection properties:

```typescript
// Option 1: Use the URI (perfect for Prisma, TypeORM, etc.)
const databaseUrl = process.env.APPDB_URI;
// postgresql://user:pass@host:port/dbname

// Option 2: Use individual properties (great for node-postgres)
const pool = new Pool({
  host: process.env.APPDB_HOST,
  port: process.env.APPDB_PORT,
  user: process.env.APPDB_USERNAME,
  password: process.env.APPDB_PASSWORD,
  database: process.env.APPDB_DATABASE
});
```

This flexibility means you can use Prisma, TypeORM, Sequelize, Knex, node-postgres, or whatever database library you prefer—no Aspire-specific adapters required. It just works with what you’re already using.

## 🏛️ Default behaviors and publishing: The magic behind the curtain

Okay, time for the deep dive. Understanding what Aspire does automatically helps you leverage its full power and tweak things when you need to. (And sometimes you will need to—we’re engineers, we love to tinker.)

### 🔧 What every JavaScript resource gets for free

Every JavaScript resource you add to Aspire—whether it’s `AddJavaScriptApp()`, `AddNodeApp()`, or `AddViteApp()`— automatically gets a bunch of sensible Node.js defaults through an internal `WithNodeDefaults()` method. You don’t call this yourself; it happens automatically. Here’s what you’re getting:

**OpenTelemetry integration (observability for free):**

Aspire automatically configures OpenTelemetry exporters for your JavaScript apps. Distributed tracing? Metrics collection? You get all of that out of the box. Your Node.js services automatically participate in Aspire’s observability story without you writing a single line of telemetry code. It’s glorious.

**Environment-aware NODE_ENV (because context matters):**

The `NODE_ENV` environment variable gets set automatically based on what you’re doing:

- `"development"` when you’re hacking away locally
- `"production"` when you’re shipping to prod

This ensures frameworks and libraries use appropriate optimizations. Express, for instance, enables template caching and serves less verbose error messages in production. These small optimizations add up.

**Automatic certificate trust (SSL/TLS without the headaches):**

Aspire handles certificate trust transparently so your Node.js apps can talk to other services over HTTPS during local development without angry certificate warnings:

- **Append mode (default)**: Sets `NODE_EXTRA_CA_CERTS` to point to Aspire’s certificate bundle. Node.js trusts Aspire-managed certificates _alongside_ system certificates. Best of both worlds.
- **Replace mode**: Modifies `NODE_OPTIONS` to include `--use-openssl-ca`, forcing Node.js to use OpenSSL’s certificate store instead.

Here’s the clever bit: if you’ve already set `NODE_OPTIONS` to something like `--max-old-space-size=4096` (because you’re dealing with large datasets or memory-hungry builds), Aspire _appends_ `--use-openssl-ca` instead of nuking your existing settings. It’s respectful like that.

### 🐳 Publishing and Dockerfile generation: Production-ready containers without the pain

When you publish your Aspire project (to Azure Container Apps, Kubernetes, wherever), Aspire auto-generates production-ready Dockerfiles through `PublishAsDockerFile()`. Smart detail: if you already have a Dockerfile in your app directory, Aspire backs off and uses yours. It’s not going to overwrite your carefully crafted custom setup.

**What Aspire generates for you:**

For `AddJavaScriptApp()` and `AddViteApp()`, you get single-stage Dockerfiles optimized for build tools that spit out static assets:

1. **Base image selection**: Defaults to `node:22-slim` (or detects version from `.nvmrc`, `package.json` engines, or `.node-version`)
2. **Working directory**: Sets up `/app` as home base
3. **Package manager setup**: Runs initialization commands (like `corepack enable pnpm` for pnpm users)
4. **Smart layer caching**: Copies package files first (`package.json`, lockfiles, etc.) _before_ copying source code. This means dependency installation gets cached unless your dependencies actually change. Docker layer caching FTW.
5. **Dependency installation**: Runs your package manager’s install command with BuildKit cache mounts (more on this in a sec)
6. **Build execution**: Runs your configured build script (default: “build”) to produce production assets
7. **Container files**: Automatically marks `/app/dist` as a container files source for sharing build outputs between containers

For `AddNodeApp()`, you get multi-stage Dockerfiles that separate build from runtime:

1. **Build stage**: Named “build”, uses the full Node.js image, installs deps, runs build scripts
2. **Runtime stage**: Named “runtime”, copies _only_ built artifacts from the build stage (smaller images, faster deploys)
3. **Security hardening**: Switches to the non-privileged `node` user before running anything
4. **Production environment**: Sets `NODE_ENV=production` explicitly
5. **Clean entrypoint**: Configures as `["node", "your-script.js"]` for efficient process execution

**BuildKit cache mount magic:**

Aspire leverages Docker BuildKit’s cache mount feature for package managers. This is a game-changer for build times:

- npm: `--mount=type=cache,target=/root/.npm`
- Yarn v1: `--mount=type=cache,target=/root/.cache/yarn`
- Yarn v2+: `--mount=type=cache,target=.yarn/cache`
- pnpm: `--mount=type=cache,target=/pnpm/store`

These cache mounts persist package manager caches _across builds_. Translation: you only download packages that actually changed. Subsequent builds are dramatically faster because you’re not re-downloading React for the 500th time.

**Smart build script execution:**

`AddJavaScriptApp()` and `AddViteApp()` automatically configure build scripts for you, so your apps build without extra configuration. For `AddNodeApp()`, you’ll need to explicitly call `WithBuildScript()` if you want a build step. If no build script is configured, the generated Dockerfile skips that step entirely. Perfect for apps that don’t need a compilation or bundling step—why waste build time on something unnecessary?

**Container files (build output sharing):**

JavaScript applications using `AddJavaScriptApp()` or `AddViteApp()` can share their build outputs with other containers through Aspire’s container files feature. By default, Aspire marks `/app/dist` as a container files source, meaning other resources can reference and copy these built assets. Super useful for scenarios where you build a frontend in one container and serve it from another (like when you’re using Nginx as a reverse proxy). No weird volume mounts or hacky workarounds.

Here’s a practical example where a Node.js backend serves the built Vite frontend:

```javascript
var builder = DistributedApplication.CreateBuilder(args);

var server = builder.AddNodeApp("server", "../server", "app.js")
    .WithHttpEndpoint(env: "PORT")
    .WithExternalHttpEndpoints();

var frontend = builder.AddViteApp("frontend", "../frontend")
    .WithReference(server)
    .WaitFor(server);

// The server will include the frontend's built assets in its public directory
server.PublishWithContainerFiles(frontend, "public");

builder.Build().Run();
```

What’s happening here:

1. The Vite app builds and exposes `/app/dist` as container files automatically
2. The Node.js server uses `PublishWithContainerFiles()` to pull in those built assets
3. Frontend assets get copied into the server’s `public` directory during publishing
4. In the generated Dockerfile, you’ll see `COPY --from=frontend /app/dist public`

This pattern is perfect for scenarios where your Node.js backend (Express, Fastify, etc.) serves static frontend assets—a common pattern for SPAs with API backends, or when you want a single deployable container for both frontend and backend.

### 🎨 Customizing Docker images: When you need more control

Aspire’s defaults work for most scenarios, but sometimes you need to do your own thing. That’s totally fine.

#### 🔢 Node.js version selection: Picking your Node

Aspire defaults to **Node.js 22**, but it’s smart enough to detect what you actually want. It checks your project files in this order:

1. **`.nvmrc` file** – The standard for nvm users
2. **`.node-version` file** – Alternative format supported by various Node version managers
3. **`package.json` engines.node field** – The npm-official way to specify Node requirements

Want Node.js 24? Create a `.nvmrc` file in your app directory:

```text
24
```

Or spec it in your `package.json`:

```json
{    
  "name": "my-app",
  "engines": {
    "node": ">=24.0.0"
  }
}
```

Aspire extracts the major version and uses it when building the Docker base image (e.g., `node:24-slim`).

**Note**

Aspire only detects the **major version** from these files. If you specify `24.11.0` or `>=24.0.0`, Aspire will use `node:24-slim` or `node:24-alpine` depending on the resource type.

#### 🎯 Override the base image completely

If auto-detection doesn’t cut it, or you want a specific image variant, use `WithDockerfileBaseImage()`.

**For client-side apps** (`AddJavaScriptApp()` and `AddViteApp()`), there’s only a `buildImage` parameter since these generate single-stage Dockerfiles:

```javascript
var app = builder.AddJavaScriptApp("app", "../app")
    .WithDockerfileBaseImage(buildImage: "node:22-alpine");

var viteApp = builder.AddViteApp("vite-app", "../vite-app")
    .WithDockerfileBaseImage(buildImage: "node:24-slim");
```

**For `AddNodeApp()`**, you can specify _both_ a build image (first parameter) and a runtime image (because multi-stage Dockerfiles let you use different images for building vs running):

```javascript
var nodeApp = builder.AddNodeApp("node-app", "../node-app", "server.js")
    .WithDockerfileBaseImage(
        buildImage: "node:22-bullseye",
        runtimeImage: "node:22-slim");
```

This lets you use a full-featured image for building (with Python, build tools, native dependencies, whatever) while keeping your runtime image minimal. Smaller images = faster deploys = reduced attack surface. Win-win-win.

## 💻 Development experience: Local dev that doesn’t suck

JavaScript development is actually pretty great these days! I do thoroughly enjoy how easy and quick it is to write JavaScript (especially with TypeScript). Aspire aims to enhance that experience further by smoothing out the rough edges of local development when services need to work together from different projects. And of course, taking your apps from local dev to production without the usual headaches.

### 🔥 Hot Module Replacement: Changes without waiting

Vite apps get fast Hot Module Replacement during development automatically. Change your code, see it instantly in the browser without full page reloads. If you’ve used Vite before, you know how addictive this is.

### 📦 What you get in those auto-generated Dockerfiles

**For `AddJavaScriptApp()` and `AddViteApp()`:**

- Multi-stage builds (or single stage when appropriate)
- Package files copied first for optimal Docker layer caching
- Package install commands with BuildKit cache mounts (faster builds are better builds)
- Build scripts executed to create production assets
- `NODE_ENV=production` set for runtime
- Defaults to Node.js 22 (or your specified version)

**For `AddNodeApp()`:**

- Separate `build` and `runtime` stages (smaller final images)
- Dependencies installed in build stage
- Build scripts run if you’ve configured them
- Built artifacts copied to minimal runtime image
- Proper entrypoint with the Node command and your script path
- Runs as non-root `node` user (because security isn’t optional)

### ⚡ Package manager-specific optimizations: The details that matter

**npm:**

- Copies `package*.json` for layer caching
- Uses BuildKit cache mount at `/root/.npm`
- Switches to `npm ci` in production mode when `package-lock.json` exists (faster, more deterministic)

**Yarn:**

- Copies `package.json`, `yarn.lock`, `.yarnrc.yml`, and `.yarn` directory
- Detects Yarn version and uses appropriate frozen lockfile flags
- Uses BuildKit cache mount at `/root/.cache/yarn` (v1) or `.yarn/cache` (v2+)

**pnpm:**

- Copies `package.json` and `pnpm-lock.yaml`
- Enables pnpm via `corepack enable pnpm` before installation
- Uses BuildKit cache mount at `/pnpm/store`
- Uses `--frozen-lockfile` in production mode when lockfile exists

## 📚 Working with TypeScript: It just works

Aspire fully supports TypeScript apps without any special configuration. Your build process (defined in your `package.json` scripts) handles TypeScript compilation:

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

Aspire respects your TypeScript configuration and build pipeline. Whether you’re using `tsc`, `tsx`, `esbuild`, `swc`, or any other TypeScript compiler, configure your scripts and Aspire handles the rest. TypeScript is a first-class citizen here.

## 🎉 The bottom line

Aspire 13 brings JavaScript and TypeScript into the fold as first-class citizens—not as an afterthought, but as a core part of the platform. Whether you’re orchestrating Vite frontends, Express APIs, or Node.js microservices alongside any other project you can imagine, Aspire gives you unified tooling, automatic service discovery, production-ready Dockerfiles, and a development experience that actually works.

**Massive shoutout to the [Aspire Community Toolkit](https://github.com/CommunityToolkit/Aspire)** 🙌—this JavaScript integration wouldn’t exist without them. They pioneered the Node.js support that became the foundation for what shipped in Aspire 13. The community saw what was missing, built it, refined it, and made it production-ready. This is community-driven development at its finest, and they deserve all the credit.

Get started by installing the [`Aspire.Hosting.JavaScript`](https://www.nuget.org/packages/Aspire.Hosting.JavaScript) NuGet package in your AppHost project, or try `aspire new aspire-ts-cs-starter` to see JavaScript and .NET working together. For complete docs, check out the [JavaScript support documentation on aspire.dev](https://aspire.dev/integrations/frameworks/javascript/).

Now go build something cool. 🚀

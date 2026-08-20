# The Complete Guide to `just`: A Comprehensive Tutorial to Modern Command Running

> **Difficulty Level:** Intermediate  
> **Estimated Reading Time:** 45 minutes  
> **Last Updated:** August 2026

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [What is `just` and Why Use It?](#what-is-just-and-why-use-it)
5. [Installation Guide](#installation-guide)
6. [Core Concepts](#core-concepts)
7. [Your First Justfile — Quick Start](#quick-start)
8. [Recipe Syntax Deep Dive](#recipe-syntax)
9. [Dependencies Between Recipes](#dependencies)
10. [Recipe Parameters (Arguments)](#recipe-parameters)
11. [Variables and Expressions](#variables)
12. [Built-in Functions](#functions)
13. [Recipes in Other Languages (Shebang Recipes)](#shebang)
14. [Settings, `.env` Files, and Shell Configuration](#settings)
15. [Aliases and Private Recipes](#aliases)
16. [Modules and Imports for Large Projects](#modules)
17. [Real-World Use Cases (20+ Examples)](#use-cases)
18. [Common Pitfalls and Troubleshooting](#troubleshooting)
19. [Best Practices](#best-practices)
20. [Anti-Patterns](#anti-patterns)
21. [Performance Considerations](#performance)
22. [Security Considerations](#security)
23. [Testing Strategies](#testing)
24. [Migration Guide: From `make` to `just`](#migration)
25. [Summary and Key Takeaways](#summary)
26. [Further Reading and Resources](#resources)
27. [Practice Exercises](#exercises)
28. [Question Bank](#question-bank)
29. [Test Your Understanding](#test-understanding)
30. [Common Interview Questions](#interview-questions)
31. [Self-Assessment Checklist](#checklist)
32. [Hands-On Labs](#labs)
33. [Pro Tips](#pro-tips)
33. [Quick Recap](#recap)

---

<a name="introduction"></a>
## 1. Introduction

### What You'll Learn

In this comprehensive tutorial, you'll learn how to use `just` — a modern, cross-platform command runner that transforms how development teams manage and execute project-specific tasks. Instead of remembering complex command sequences or maintaining scattered shell scripts, `just` provides a single, discoverable file (`justfile`) where all your project commands live.

From simple `build` and `test` recipes to complex multi-service deployment workflows, this guide covers everything you need to know to master `just` and integrate it into your development workflow.

### Why This Matters

Every development team faces the same problem: how do you consistently run the same commands across different environments, team members, and platforms? While `make` has been the traditional solution, its legacy constraints (file timestamp tracking, `.PHONY` boilerplate, cryptic errors) make it a poor fit for modern development workflows.

`just` solves these problems by providing a clean, intuitive syntax that focuses on what developers actually need: a simple, discoverable list of named commands that work consistently everywhere.

---

<a name="prerequisites"></a>
## 2. Prerequisites

Before diving into this tutorial, ensure you have the following:

- **Basic shell/command-line knowledge**: Comfort with bash, zsh, PowerShell, or similar
- **Fundamental understanding of build tools**: Familiarity with concepts like compilers, test runners, or package managers (Gradle, npm, pip, etc.)
- **A text editor or IDE**: VS Code, IntelliJ, Vim, or any editor you're comfortable with
- **Git installed**: For version control (recommended but not strictly required)
- **Optional - `just` installed**: You can follow along with examples even without installing `just` first, but installing it early will let you experiment
- **Optional - Docker**: Some use cases involve containerized workflows
- **Optional - Language runtimes**: Depending on which examples you explore (Java/Gradle, Python, Node.js, etc.)

> 💡 **No prior experience with `make` or `just` is required** — this tutorial starts from the basics.

---

<a name="learning-objectives"></a>
## 3. Learning Objectives

By completing this tutorial, you will be able to:

1. ✅ Install and configure `just` on any platform (Linux, macOS, Windows)
2. ✅ Create and structure `justfile`s with recipes, dependencies, and parameters
3. ✅ Use variables, expressions, and built-in functions effectively
4. ✅ Write recipes in multiple languages (Bash, Python, Node.js, Ruby)
5. ✅ Configure settings, `.env` files, and shell environments
6. ✅ Organize large projects using modules and imports
7. ✅ Implement complex workflows like CI/CD pipelines, Docker workflows, and multi-service deployments
8. ✅ Troubleshoot common issues and avoid anti-patterns
9. ✅ Apply best practices for maintainable, team-friendly justfiles
10. ✅ Migrate from `make` to `just` with minimal friction

---

<a name="what-is-just-and-why-use-it"></a>
## 4. What is `just` and Why Use It?

`just` is a **command runner** — a lightweight tool that lets you save and run project-specific commands from a file called a `justfile`. Think of it as `make`, but purpose-built for running tasks rather than building software from source dependencies.

Instead of remembering (or copy-pasting from Slack) a long string like:

```bash
docker build -t myapp:latest . && docker run --rm -p 8080:8080 myapp:latest
```

...you write it once in a `justfile`, give it a name, and from then on you just run:

```bash
just docker-run
```

### Why not just use `make`?

`make` was designed in 1976 to compile C programs by tracking file timestamps and rebuilding only what's stale. That's a great model for compilers, but a poor fit for tasks like "run the linter" or "deploy to staging" — tasks that don't produce a file. This forces `make` users to create empty `.PHONY` targets and fight with `make`'s quirky whitespace and escaping rules.

`just` strips away the file-dependency model and idiosyncrasies, and keeps what people actually loved about `make`: a simple, discoverable list of named commands.

### Key Advantages

| Feature | `just` |
|---|---|
| Cross-platform | Linux, macOS, Windows, no extra dependencies |
| Error messages | Clear, specific, with source context; static detection of unknown recipes and circular dependencies |
| Parameters | First-class support for arguments, flags, and options |
| Expression language | Variables, conditionals, built-in functions |
| `.env` support | Automatic loading of `.env` files |
| Discoverability | `just --list` shows all available commands |
| Multi-language | Recipes can be written in Python, Node.js, Bash, Ruby, etc. |
| Directory traversal | Callable from any subdirectory of your project |
| Modular design | Justfiles can be split across multiple files using modules and imports |

### Mental Model

```mermaid
flowchart LR
    A[Developer types<br/>just deploy] --> B[just searches upward<br/>for a justfile]
    B --> C{Recipe 'deploy'<br/>found?}
    C -->|No| D[Error: Unknown recipe<br/>+ suggestions]
    C -->|Yes| E[Resolve dependencies<br/>statically]
    E --> F{Dependencies<br/>exist & valid?}
    F -->|No| G[Error before<br/>anything runs]
    F -->|Yes| H[Run dependency recipes<br/>in order]
    H --> I[Run 'deploy' recipe lines<br/>in a shell]
    I --> J[Stop immediately<br/>if any line fails]
    J --> K[Exit 0 on success]
```

> 💡 **Aha Moment:** `just` is not a build system — it's a **task runner**. This fundamental shift in purpose is what makes it so much more intuitive than `make` for modern development workflows.

---

<a name="installation-guide"></a>
## 5. Installation Guide

`just` can be installed in three main ways:

| Method | Command | Best for |
|---|---|---|
| Package manager | `brew install just` (macOS), `winget install --id Casey.Just` (Windows), `apt`/`pacman`/etc. on Linux | Most users |
| Pre-built binary | Download from the [GitHub releases page](https://github.com/casey/just/releases) | CI systems, no package manager access |
| From source | `cargo install just` | Rust developers, latest features |

### Example: macOS / Linux with Homebrew

```console
$ brew install just
$ just --version
just 1.x.x
```

### Example: Cargo (any platform with Rust installed)

```console
$ cargo install just
$ just --version
```

### Example: Windows with winget

```console
> winget install --id Casey.Just --exact
> just --version
```

> ⚠️ **Important:** Always verify your install with `just --version` before moving on — a mismatched or missing install is the #1 cause of "recipe not found" confusion for beginners.

---

<a name="core-concepts"></a>
## 6. Core Concepts

Before writing any code, it helps to understand the vocabulary `just` uses:

| Term | Meaning |
|---|---|
| **justfile** | The file (case-insensitive: `justfile`, `Justfile`, `.justfile`) holding your commands |
| **recipe** | A named command (like a `make` target) — the core building block |
| **dependency** | A recipe that must run before another recipe |
| **parameter** | An argument a recipe accepts, similar to a function parameter |
| **variable** | A named value assigned with `:=`, reusable across recipes |
| **setting** | A directive (like `set shell := [...]`) that changes `just`'s behavior |
| **alias** | An alternate name for a recipe |
| **module** | A separate justfile imported as a namespaced sub-unit |

```mermaid
flowchart TD
    subgraph justfile
        S[Settings<br/>set shell, set dotenv-load]
        V[Variables<br/>version := '1.2.3']
        R1[Recipe: build]
        R2[Recipe: test]
        R3["Recipe: deploy target env="prod""]
        A[Aliases]
        M[Modules / Imports]
    end
    R3 -->|depends on| R2
    R2 -->|depends on| R1
    V -.->|used by| R1
    V -.->|used by| R3
    S -.->|configures execution of| R1
    S -.->|configures execution of| R2
    A -.->|shortcut for| R3
```

---

<a name="quick-start"></a>
## 7. Your First Justfile — Quick Start

Create a file named `justfile` in your project root:

```just
recipe-name:
    echo 'This is a recipe!'

# this is a comment
another-recipe:
    @echo 'This is another recipe.'
```

Run it:

```console
$ just
echo 'This is a recipe!'
This is a recipe!
```

Two important beginner facts baked into that example:

1. **Running `just` with no arguments runs the first recipe** in the file (or the one marked `[default]` — more on that later).
2. **`just` echoes each shell command before running it**, similar to `make`. Prefixing a line with `@` suppresses that echo — useful for `echo` statements where printing the command *and* its output would be redundant.

```console
$ just another-recipe
This is another recipe.
```

Notice `another-recipe` only printed its output, not the `echo` command itself, because of the `@`.

### Where does `just` look for the justfile?

`just` searches the **current directory, then upward through parent directories**, so you can run `just build` from deep inside `src/components/widgets/` and it will still find the `justfile` at your project root — a huge quality-of-life win over `make`, which only looks in the current directory.

```mermaid
flowchart BT
    D1["/project/src/components/widgets/<br/>(you run 'just' here)"] --> D2["/project/src/components/"]
    D2 --> D3["/project/src/"]
    D3 --> D4["/project/  ✅ justfile found here"]
    style D4 fill:#c8f7c8,stroke:#2e7d32
```

---

<a name="recipe-syntax"></a>
## 8. Recipe Syntax Deep Dive

A recipe looks like this:

```just
recipe-name: dependency1 dependency2
    shell command line 1
    shell command line 2
```

- The **name** comes first, followed by a colon.
- **Dependencies** (other recipes that must run first) go after the colon, space-separated.
- Each **body line must be indented** (with tabs or consistent spaces) — this is how `just` knows the recipe body has ended.

### Example 1 — Basic recipe with output suppression

```just
build:
    cc main.c foo.c bar.c -o main

test: build
    ./test

sloc:
    @echo "`wc -l *.c` lines of code"
```

Running `just test` builds first automatically, because `test` depends on `build`:

```console
$ just test
cc main.c foo.c bar.c -o main
./test
testing… all tests passed!
```

### Example 2 — Recipes stop on first failure

```just
publish:
    cargo test
    # tests passed, time to publish!
    cargo publish
```

If `cargo test` exits with a nonzero status, `cargo publish` **never runs**. This "fail fast" behavior protects you from accidentally shipping broken code — a subtle but crucial safety net.

### Example 3 — Multiple recipes on one command line

```console
$ just build sloc
cc main.c foo.c bar.c -o main
1337 lines of code
```

Recipes without dependencies between them run **in the order given on the command line**.

### Example 4 — Dependencies always run first, regardless of order

```console
$ just test build
cc main.c foo.c bar.c -o main
./test
testing… all tests passed!
```

Even though you typed `test build` (test first), `build` runs first because `test` depends on it. `just` resolves the dependency graph, not just your typed order.

```mermaid
sequenceDiagram
    participant You
    participant just
    participant Shell
    You->>just: just test build
    just->>just: Resolve dependency graph<br/>(test → build)
    just->>Shell: Run 'build' recipe
    Shell-->>just: cc main.c foo.c bar.c -o main
    just->>Shell: Run 'test' recipe
    Shell-->>just: ./test
    just->>Shell: Run 'build' recipe again? NO
    Note over just: build already satisfied<br/>this invocation
```

---

<a name="dependencies"></a>
## 9. Dependencies Between Recipes

Dependencies are the heart of `just`'s power — they let you compose small recipes into larger workflows without repeating yourself.

### Simple dependency chain

```just
build:
    cc main.c foo.c bar.c -o main

test: build
    ./test
```

### Recipes may depend on recipes in submodules

```Just
mod foo

baz: foo::bar
```

Here `baz` depends on the `bar` recipe defined inside the `foo` module (covered fully in [Modules](#modules)).

### Use case: a realistic CI pipeline

```Just
lint:
    eslint src/

typecheck:
    tsc --noEmit

test: lint typecheck
    jest --coverage

build: test
    vite build

deploy: build
    rsync -avz dist/ user@server:var/www/app
```

Running `just deploy` triggers the **entire chain**: `lint → typecheck → test → build → deploy`, each stopping the pipeline immediately if any step fails.

```mermaid
flowchart LR
    lint --> test
    typecheck --> test
    test --> build
    build --> deploy
    style deploy fill:#ffe0b2,stroke:#e65100
```

This mirrors exactly how CI systems like GitHub Actions structure jobs — which is why teams often use the *same* `justfile` both locally and inside CI, guaranteeing "it works on my machine" actually matches "it works in CI."

---

<a name="recipe-parameters"></a>
## 10. Recipe Parameters (Arguments)

Recipes can take parameters, just like functions.

### Basic parameter

```just
build target:
    @echo 'Building {{target}}…'
    cd {{target}} && make
```

```console
$ just build my-awesome-project
Building my-awesome-project…
cd my-awesome-project && make
```

### Passing arguments to a dependency

```just
default: (build "main")

build target:
    @echo 'Building {{target}}…'
    cd {{target}} && make
```

Or forward a caller's own argument to its dependency:

```Just
build target:
    @echo "Building {{target}}…"

push target: (build target)
    @echo 'Pushing {{target}}…'
```

### Default values

```Just
default := 'all'

test target tests=default:
    @echo 'Testing {{target}}:{{tests}}…'
    ./test --tests {{tests}} {{target}}
```

```console
$ just test server
Testing server:all…

$ just test server unit
Testing server:unit…
```

Parameters with defaults are optional; parameters without defaults are required.

### Variadic parameters — accept a list of arguments

Use `+` for **one-or-more**:

```Just
backup +FILES:
    scp {{FILES}} me@server.com:
```

```console
$ just backup FAQ.md GRAMMAR.md
scp FAQ.md GRAMMAR.md me@server.com:
```

Use `*` for **zero-or-more**:

```Just
commit MESSAGE *FLAGS:
    git commit {{FLAGS}} -m "{{MESSAGE}}"
```

```console
$ just commit "fix bug"
git commit  -m "fix bug"

$ just commit "fix bug" --amend --no-verify
git commit --amend --no-verify -m "fix bug"
```

### Environment-variable parameters

Prefix a parameter with `$` to export it as an environment variable instead of a `{{...}}` substitution:

```Just
foo $bar:
    echo $bar
```

### Flags and options (turning parameters into `--flag` style CLI args)

By default parameters are **positional**. You can opt into GNU-style flags with the `[arg(...)]` attribute:

```Just
[arg("bar", long="bar")]
foo bar:
    echo bar={{bar}}
```

```console
$ just foo --bar hello
bar=hello

$ just foo --bar=hello
bar=hello
```

Short options work the same way:

```Just
[arg("bar", short="b")]
foo bar:
    echo bar={{bar}}
```

```console
$ just foo -b hello
bar=hello
```

You can even create no-value "flags" (great for dangerous operations like `--force`):

```Just
[arg("bar", long="bar", value="hello")]
foo bar="goodbye":
    echo bar={{bar}}
```

```console
$ just foo --bar
bar=hello

$ just foo
bar=goodbye
```

### Use case: a single flexible `deploy` recipe

```Just
[arg("env", long="env")]
[arg("force", long="force", value="true")]
deploy env="staging" force="false":
    @echo "Deploying to {{env}} (force={{force}})"
    ./scripts/deploy.sh {{env}} {{force}}
```

```console
$ just deploy
Deploying to staging (force=false)

$ just deploy --env prod --force
Deploying to prod (force=true)
```

One recipe now safely covers every deployment scenario your team needs, with a discoverable, self-documenting CLI.

---

<a name="variables"></a>
## 11. Variables and Expressions

Variables are declared at the top level with `:=` and interpolated with `{{...}}`:

```Just
version := "1.4.2"
image_name := "myapp:" + version

build:
    docker build -t {{image_name}} .
```

### String concatenation and paths

```Just
arch := "wasm"

test triple=(arch + "-unknown-unknown") input=(arch / "input.dat"):
    ./test {{triple}}
```

> ⚠️ Expressions containing `+`, `&&`, `||`, or `/` **must be parenthesized** when used as default parameter values.

### Quoting substitutions to avoid word-splitting

```Just
search QUERY:
    lynx 'https://www.google.com/?q={{QUERY}}'
```

Without the quotes, `just search "cat toupee"` would be parsed by the shell as **two** separate arguments (`cat` and `toupee`) instead of one query string — a classic beginner gotcha.

### Conditional expressions

```Just
os := if os() == "windows" { "win" } else { "unix" }

build:
    @echo "Building for {{os}}"
```

---

<a name="functions"></a>
## 12. Built-in Functions

`just` ships with a rich standard library of functions for paths, strings, environment info, and shelling out.

| Category | Example function | Purpose |
|---|---|---|
| System | `os()`, `os_family()`, `arch()` | Cross-platform conditionals |
| Paths | `justfile()`, `justfile_directory()`, `invocation_directory()` | Locate files reliably regardless of where `just` was called from |
| Strings | `uppercase()`, `lowercase()`, `trim()`, `replace()` | Text manipulation |
| Shell | `shell(command, args...)` | Run a shell command and capture its stdout |

### Example: cross-platform justfile with `os_family()`

```Just
os := os_family()

install:
    {{ if os == "windows" { "choco install foo" } else { "brew install foo" } }}
```

### Example: capturing shell output into a variable

```Just
file := '/sys/class/power_supply/BAT0/status'
bat0stat := shell('cat $1', file)

battery:
    @echo "Battery status: {{bat0stat}}"
```

### Example: always run from the justfile's own directory

```Just
build:
    cd {{justfile_directory()}} && make
```

This guarantees `make` runs against the right directory even if a teammate calls `just build` from three folders deep.

---

<a name="shebang"></a>
## 13. Recipes in Other Languages (Shebang Recipes)

One of `just`'s most loved features: recipes aren't limited to shell one-liners. Add a shebang line and the recipe body becomes a script in *any* language.

### Python example

```Just
plot:
    #!/usr/bin/env python3
    import matplotlib.pyplot as plt
    plt.plot([1, 2, 3], [4, 5, 6])
    plt.savefig("plot.png")
    print("Saved plot.png")
```

### Node.js example

```Just
report:
    #!/usr/bin/env node
    const fs = require('fs');
    const data = JSON.parse(fs.readFileSync('data.json'));
    console.log(`Total records: ${data.length}`);
```

### Bash example with strict mode

```Just
release:
    #!/usr/bin/env bash
    set -euo pipefail
    echo "Tagging release..."
    git tag "v$(cat VERSION)"
    git push --tags
```

```mermaid
flowchart TD
    A["just release"] --> B{Recipe body has<br/>a shebang line?}
    B -->|No| C[Run each line via<br/>the configured shell<br/>e.g. sh -cu]
    B -->|Yes| D[Write body to a<br/>temp script file]
    D --> E[Make it executable]
    E --> F[Execute directly<br/>via its interpreter<br/>python3 / node / bash / ruby...]
```

**Use case:** a data-science project can keep its `train-model` recipe in Python, its `deploy` recipe in Bash, and its `generate-report` recipe in Node — all discoverable through the same `just --list`, without maintaining three separate wrapper scripts.

---

<a name="settings"></a>
## 14. Settings, `.env` Files, and Shell Configuration

Settings are declared with `set NAME := VALUE` (or just `set NAME` for booleans) at the top of the justfile.

```Just
set shell := ["bash", "-uc"]
set dotenv-load
set export
```

| Setting | What it does |
|---|---|
| `set shell := [...]` | Change the interpreter used to run recipe lines (default `sh -cu` on unix) |
| `set dotenv-load` | Automatically load variables from a `.env` file into recipe environments |
| `set export` | Export all `just` variables as environment variables for recipes |
| `set windows-shell := [...]` | Set a different shell specifically on Windows (e.g., PowerShell) |
| `set positional-arguments` | Pass recipe parameters as shell positional args (`$1`, `$2`, ...) |

### Example: loading secrets from `.env`

```
# .env
DATABASE_URL=postgres://localhost/mydb
API_KEY=supersecret
```

```Just
set dotenv-load

migrate:
    diesel migration run --database-url $DATABASE_URL
```

No more `source .env && ...` boilerplate — `just` handles it automatically.

### Example: using PowerShell on Windows, bash elsewhere

```Just
set windows-shell := ["powershell.exe", "-NoLogo", "-Command"]

greet:
    echo "Hello from just!"
```

This lets a **single justfile work identically for the whole team**, whether they're on macOS, Linux, or Windows.

---

<a name="aliases"></a>
## 15. Aliases and Private Recipes

### Aliases — shorter names for common recipes

```Just
alias b := build

build:
    cargo build
```

```console
$ just b
cargo build
```

### Private recipes — hide implementation details

Prefix a recipe name with `_` (or use the `[private]` attribute) to hide it from `just --list`, while still allowing it to be used as a dependency:

```Just
_compile:
    cc -c main.c

build: _compile
    cc main.o -o main
```

`_compile` won't clutter `just --list`, but `build` can still depend on it. This is a common pattern for breaking a workflow into small reusable steps without overwhelming your teammates with internal plumbing.

### The default recipe

```Just
[default]
help:
    @just --list
```

Now simply running `just` with no arguments shows the recipe list — a friendly, self-documenting entry point for new contributors.

---

<a name="modules"></a>
## 16. Modules and Imports for Large Projects

As a project grows, one giant `justfile` becomes unwieldy. `just` supports splitting recipes across files.

### Modules — namespaced sub-justfiles

```Just
# justfile
mod frontend
mod backend
```

```Just
# frontend.just
build:
    npm run build

test:
    npm test
```

```console
$ just frontend::build
$ just --list
Available recipes:
    frontend:
        build
        test
    backend:
        build
        test
```

### Imports — flatten another file's recipes into the current namespace

```Just
import 'tasks/docker.just'
import 'tasks/database.just'
```

```mermaid
flowchart TD
    root["justfile (root)"]
    root -->|mod| fe["frontend.just<br/>(namespaced: frontend::*)"]
    root -->|mod| be["backend.just<br/>(namespaced: backend::*)"]
    root -->|import| docker["tasks/docker.just<br/>(flattened into root)"]
    root -->|import| db["tasks/database.just<br/>(flattened into root)"]
```

**Use case:** a monorepo with a `frontend/`, `backend/`, and `infra/` directory can give each team ownership of its own `justfile`, while the root `justfile` composes them with `mod`, giving everyone one consistent entry point: `just frontend::deploy`, `just backend::test`, `just infra::plan`.

---

<a name="use-cases"></a>
## 17. Real-World Use Cases (20+ Examples)

This section showcases 20+ real-world scenarios where `just` provides significant value over manual command execution or traditional tools like `make`.

### Use Case 1: Java/Gradle Build Automation

```Just
set dotenv-load

# Variables
java_version := "17"
app_name := "my-enterprise-app"

# Default recipe
[default]
help:
    @just --list

# Build with Gradle
build:
    @echo "Building {{app_name}}..."
    ./gradlew build -x test

# Run tests
test:
    @./gradlew test

# Run the application
run:
    @./gradlew bootRun

# Run with specific profile
run-env env="dev":
    @./gradlew bootRun --args='--spring.profiles.active={{env}}'

# Clean build
clean:
    @./gradlew clean

# Full CI pipeline
ci: clean build test
    @echo "CI pipeline complete!"

# Generate Swagger docs
swagger:
    @./gradlew swagger-codegen-generate

# Database migrations with Flyway
db-migrate:
    @./gradlew flywayMigrate

# Package as JAR
jar: build
    @./gradlew bootJar

# Run integration tests
integration-test:
    @./gradlew integrationTest

# Static analysis with SpotBugs
analyze:
    @./gradlew spotbugsMain spotbugsTest

# Deploy to staging
deploy-staging: ci
    @./scripts/deploy-k8s.sh staging

# Deploy to production
deploy-prod:
    @echo "⚠️  Confirm: Deploying to production?"
    @read -r -p "Type 'PROD' to confirm: " confirm
    @if [ "$$confirm" = "PROD" ]; then \
        ./scripts/deploy-k8s.sh prod; \
    else \
        echo "Deployment cancelled."; \
        exit 1; \
    fi
```

### Use Case 2: Docker Development Environment

```Just
image := "myapp:dev"

build-image:
    docker build -t {{image}} .

shell: build-image
    docker run --rm -it -v $(pwd):/app {{image}} bash

up: build-image
    docker compose up -d

down:
    docker compose down

restart: down up

logs:
    docker compose logs -f

# Multi-service development
dev-all: build-image
    docker compose up -d db redis api frontend
```

### Use Case 3: Database Migration Workflows

```Just
set dotenv-load

db-reset:
    dropdb mydb --if-exists
    createdb mydb
    just db-migrate

db-migrate:
    diesel migration run

db-seed: db-migrate
    psql mydb -f seeds.sql

db-rollback:
    diesel migration revert
```

### Use Case 4: Multi-Language Monorepo Task Orchestrator

```Just
mod api
mod web
mod mobile

test-all: api::test web::test mobile::test

deploy-all: test-all
    just api::deploy
    just web::deploy
    just mobile::deploy
```

### Use Case 5: Kubernetes Deployment Pipeline

```Just
set dotenv-load

# Build Docker image
docker-build:
    docker build -t {{DOCKER_REGISTRY}}/{{IMAGE_NAME}}:{{VERSION}} .

# Push to registry
docker-push: docker-build
    docker push {{DOCKER_REGISTRY}}/{{IMAGE_NAME}}:{{VERSION}}

# Deploy to cluster
k8s-deploy env="staging":
    kubectl set image deployment/myapp myapp={{DOCKER_REGISTRY}}/{{IMAGE_NAME}}:{{VERSION}} -n {{env}}
    kubectl rollout status deployment/myapp -n {{env}}

# Rollback
k8s-rollback env="staging":
    kubectl rollout undo deployment/myapp -n {{env}}

# Full deployment pipeline
deploy env="staging": docker-push k8s-deploy
    @echo "✅ Deployed to {{env}}"
```

### Use Case 6: API Development and Testing Workflow

```Just
# Generate OpenAPI types
gen-types:
    openapi-typescript schema.yaml --output types/api.ts

# Run API in development mode
dev:
    nodemon --exec "ts-node src/index.ts"

# Run API tests
test:
    jest --testPathPattern=__tests__/api

# Test with Postman collection
test-postman:
    newman run collections/api-tests.postman_collection.json

# Lint and format
lint:
    eslint src/ --fix
    prettier --write src/

# Generate API documentation
docs:
    @redoc-cli bundle schema.yaml -o docs/api.html
```

### Use Case 7: Data Science Model Training Pipeline

```Just
# Install dependencies
install-deps:
    pip install -r requirements.txt

# Download dataset
download-data:
    python scripts/download_dataset.py

# Train model
train epochs=10 lr=0.001:
    python train.py --epochs {{epochs}} --lr {{lr}}

# Evaluate model
evaluate model_path="models/best.pth":
    python evaluate.py --model {{model_path}}

# Run full pipeline
pipeline: download-data train evaluate
    @echo "Pipeline complete!"
```

### Use Case 8: CI/CD Pipeline Standardization

```Just
# Local CI simulation
ci: lint test build
    @echo "✅ All CI checks passed!"

lint:
    eslint src/

test:
    npm run test -- --coverage

build:
    npm run build
```

### Use Case 9: GitHub Actions Local Testing

```Just
# Simulate CI locally
ci-local:
    just lint
    just test
    just build
    just security-scan

security-scan:
    npm audit
    trivy fs .
```

### Use Case 10: Go Application Build and Release

```Just
# Build binary
build-go:
    CGO_ENABLED=0 go build -o bin/app .

# Run tests
test-go:
    go test ./... -v -race -cover

# Run linting
lint-go:
    golangci-lint run

# Format code
fmt:
    gofmt -s -w .

# Cross-compile
build-windows:
    GOOS=windows GOARCH=amd64 go build -o bin/app.exe .

# Release
release version="v1.0.0":
    git tag {{version}}
    git push origin {{version}}
```

### Use Case 11: Rust Project Workflow

```Just
# Build with optimizations
release:
    cargo build --release

# Run tests
test-rust:
    cargo test --all

# Lint with clippy
clippy:
    cargo clippy -- -D warnings

# Format code
fmt:
    cargo fmt

# Run benchmarking
bench:
    cargo bench

# Generate documentation
docs:
    cargo doc --open

# Full quality check
check: fmt clippy test-rust
    @echo "✅ All checks passed!"
```

### Use Case 12: React/Frontend Development Workflow

```Just
# Start development server
dev:
    vite

# Build for production
build-react:
    vite build

# Run linter
lint:
    eslint src/

# Run tests
test:
    vitest run

# Preview production build
preview:
    vite preview

# Type checking
typecheck:
    tsc --noEmit

# Storybook
storybook:
    storybook dev -p 6006

# Full CI pipeline
ci-react: lint typecheck test build-react
    @echo "✅ Frontend CI complete!"
```

### Use Case 13: Django/Python Web Application

```Just
# Activate environment and run
run:
    python manage.py runserver

# Run migrations
migrate:
    python manage.py migrate

# Make migrations
makemigrations:
    python manage.py makemigrations

# Run tests
test-py:
    pytest

# Lint with ruff
lint:
    ruff check .

# Format code
format:
    ruff format .

# Collect static files
collectstatic:
    python manage.py collectstatic --noinput

# Create superuser
createsuperuser username="admin":
    python manage.py createsuperuser --username {{username}}
```

### Use Case 14: Spring Boot Application Lifecycle

```Just
# Run application
run-spring:
    ./gradlew bootRun

# Build JAR
jar:
    ./gradlew bootJar

# Run tests
test:
    ./gradlew test

# Run with specific profile
run-profile profile="dev":
    ./gradlew bootRun --args='--spring.profiles.active={{profile}}'

# Database migration
db-migrate:
    ./gradlew flywayMigrate

# Package and deploy
deploy version="1.0.0":
    ./gradlew bootJar
    scp build/libs/*.jar server:/opt/app/
```

### Use Case 15: Flask API Development

```Just
# Run development server
run-flask:
    FLASK_APP=app.py flask run

# Run with debug mode
debug:
    FLASK_APP=app.py FLASK_ENV=development flask run

# Run tests
test:
    pytest tests/

# Run with gunicorn
prod:
    gunicorn -w 4 -b 0.0.0.0:8000 app:app

# Lint
lint:
    flake8 app/
    black app/

# Database setup
db-init:
    flask db init
    flask db migrate
    flask db upgrade
```

### Use Case 16: Next.js Full-Stack Application

```Just
# Development server
dev-next:
    next dev

# Production build
build-next:
    next build

# Start production server
start:
    next start

# Lint and format
lint:
    next lint
    prettier --write .

# Run tests
test:
    jest

# Type check
typecheck:
    next build && rm -rf .next

# Deploy to Vercel
deploy:
    vercel --prod

# Full CI
ci: lint typecheck test build-next
    @echo "✅ Next.js CI complete!"
```

### Use Case 17: Terraform Infrastructure Management

```Just
# Initialize Terraform
tf-init:
    terraform init

# Validate configuration
tf-validate:
    terraform validate

# Plan changes
tf-plan:
    terraform plan

# Apply changes
tf-apply:
    terraform apply

# Destroy infrastructure
tf-destroy:
    terraform destroy

# Format Terraform files
tf-fmt:
    terraform fmt

# Full workflow
deploy-infra: tf-init tf-validate tf-plan tf-apply
    @echo "✅ Infrastructure deployed!"
```

### Use Case 18: Flutter Mobile App Development

```Just
# Run on connected device
run-ios:
    flutter run -d iPhone

run-android:
    flutter run -d Android

# Build release APK
build-apk:
    flutter build apk --release

# Run tests
test-flutter:
    flutter test

# Analyze code
analyze:
    flutter analyze

# Format code
format:
    dart format .

# Build for web
build-web:
    flutter build web

# Full pipeline
ci: analyze test-flutter format build-apk
    @echo "✅ Flutter CI complete!"
```

### Use Case 19: C++ Build and Test Workflow

```Just
# Build with CMake
configure:
    cmake -B build -DCMAKE_BUILD_TYPE=Release

build-cpp: configure
    cmake --build build --config Release

# Run tests
test-cpp: build-cpp
    ctest --test-dir build --output-on-failure

# Clean build
clean:
    rm -rf build

# Run with valgrind
valgrind: build-cpp
    valgrind --leak-check=full ./build/app

# Format code
format:
    clang-format -i src/*.cpp src/*.h

# Static analysis
analyze: build-cpp
    cppcheck --enable=all src/

# Install
install: build-cpp
    cmake --install build
```

### Use Case 20: PHP/Laravel Web Application

```Just
# Run development server
serve:
    php artisan serve

# Run migrations
migrate:
    php artisan migrate

# Seed database
seed:
    php artisan db:seed

# Run tests
test-php:
    php artisan test

# Clear caches
cache-clear:
    php artisan cache:clear
    php artisan config:clear
    php artisan route:clear

# Optimize for production
optimize:
    php artisan optimize

# Run queue worker
queue:
    php artisan queue:work

# Create new controller
make-controller name:
    php artisan make:controller {{name}}Controller

# Full CI pipeline
ci: test-php optimize
    @echo "✅ Laravel CI complete!"
```

### Use Case 21: Ansible Playbook Execution

```Just
# Run playbook
ansible-run playbook hosts="production":
    ansible-playbook -i inventory/{{hosts}}.ini {{playbook}}

# Run with tags
ansible-tags playbook tags="deploy,notify":
    ansible-playbook -i inventory/production.ini {{playbook}} --tags {{tags}}

# Run in check mode
ansible-check:
    ansible-playbook -i inventory/production.ini site.yml --check

# Syntax check
ansible-syntax:
    ansible-playbook --syntax-check site.yml

# Vault operations
vault-encrypt file:
    ansible-vault encrypt {{file}}

vault-decrypt file:
    ansible-vault decrypt {{file}}
```

### Use Case 22: Elixir/Phoenix Project

```Just
# Start Phoenix server
phx-server:
    mix phx.server

# Run tests
test-ex:
    mix test

# Format code
format:
    mix format

# Compile
compile:
    mix compile --warnings-as-errors

# Run linter
credo:
    mix credo

# Database setup
setup-db:
    mix ecto.create
    mix ecto.migrate

# Full quality check
quality: format credo test-ex compile
    @echo "✅ Elixir quality checks complete!"
```

### Use Case 23: Elixir/Phoenix Project (Alternate Naming)

```Just
# Start Phoenix server
phx-server:
    mix phx.server

# Run tests
test-ex:
    mix test

# Format code
format:
    mix format

# Compile
compile:
    mix compile --warnings-as-errors

# Run linter
credo:
    mix credo

# Database setup
setup-db:
    mix ecto.create
    mix ecto.migrate

# Full quality check
quality: format credo test-ex compile
    @echo "✅ Elixir quality checks complete!"
```

> 💡 **Pro Tip:** These 23 use cases demonstrate `just`'s versatility across virtually every technology stack. The same principles apply whether you're building containers, running tests, managing infrastructure, or orchestrating complex deployment pipelines.

---

<a name="troubleshooting"></a>
## 18. Common Pitfalls and Troubleshooting

### Problem 1: "Recipe not found" Error

**Cause:** You're not in a directory with a `justfile`, or you've misspelled the recipe name.

**Solution:**
```console
# Check if a justfile exists
$ find . -name "justfile" -o -name "Justfile" -o -name ".justfile"

# List available recipes
$ just --list

# Use a specific justfile
$ just --justfile /path/to/justfile recipe-name
```

### Problem 2: Indentation Errors

**Cause:** Mixing tabs and spaces inconsistently, or not indenting recipe body lines.

**Solution:**
```just
# ❌ Incorrect - mixed indentation
build:
    echo "line 1"
     echo "line 2"  # Different indentation

# ✅ Correct - consistent spaces
build:
    echo "line 1"
    echo "line 2"
# Or consistent tabs
build:
\techo "line 1"
\techo "line 2"
```

### Problem 3: Shell Command Failures

**Cause:** Commands failing silently or not propagating errors correctly.

**Solution:**
```just
set shell := ["bash", "-c"]

# Use explicit error handling
build:
    set -euo pipefail
    cc main.c -o main || { echo "Build failed"; exit 1; }
```

### Problem 4: Variable Expansion Issues

**Cause:** Variables not expanding as expected, especially with paths containing spaces.

**Solution:**
```just
# Always quote substitutions with potential spaces
search QUERY:
    lynx 'https://www.google.com/?q={{QUERY}}'
```

### Problem 5: Cross-Platform Compatibility

**Cause:** Commands that work on one OS but fail on another.

**Solution:**
```just
os := os_family()

build:
    {{ if os == "windows" { "build.bat" } else { "./build.sh" } }}
```

### Problem 6: Dotenv Not Loading

**Cause:** `.env` file missing or not configured.

**Solution:**
```Just
set dotenv-load
```

Make sure `.env` exists in your project root.

### Problem 7: Private Recipe Accidentally Hidden

**Cause:** Naming a recipe with `_` prefix when you actually want it visible.

**Solution:** Remove the `_` prefix or remove the `[private]` attribute.

---

<a name="best-practices"></a>
## 19. Best Practices

### 1. Start with a Default Help Recipe

```Just
[default]
help:
    @just --list --summary
    @echo ""
    @echo "Run 'just --list' to see all available recipes."
```

### 2. Group Related Recipes Logically

```Just
# === Database ===
db-setup:
    ...

db-migrate:
    ...

# === Build ===
build:
    ...

# === Test ===
test:
    ...
```

### 3. Use Aliases for Frequently Used Recipes

```Just
alias b := build
alias t := test
alias d := deploy
```

### 4. Leverage Dependencies for Pipeline Composition

```Just
ci: lint test build deploy
    @echo "✅ Full pipeline complete!"
```

### 5. Use Environment Variables for Configuration

```Just
set dotenv-load

image := "myapp:" + env("APP_VERSION", "latest")
```

### 6. Document Complex Recipes with Comments

```Just
# Deploys to the specified environment.
# Usage: just deploy env=staging --force
[arg("env", long="env")]
[arg("force", long="force", value="true")]
deploy env="staging" force="false":
    ...
```

### 7. Use Shebang Recipes for Complex Logic

```Just
complex-data-processing:
    #!/usr/bin/env python3
    # Complex multi-step data processing
    import pandas as pd
    ...
```

### 8. Keep Justfiles in Version Control

Always commit your `justfile` to version control so all team members share the same commands.

### 9. Test Your Justfile in CI

```Just
validate:
    just --list
    just --show build
    just --show test
```

---

<a name="anti-patterns"></a>
## 20. Anti-Patterns

### Anti-Pattern 1: Duplicating Commands Across Recipes

```Just
# ❌ Don't repeat the same setup
build:
    export NODE_ENV=production
    npm run build

test:
    export NODE_ENV=test
    npm run test
```

```Just
# ✅ Use a shared private recipe
_env-setup:
    @export NODE_ENV={{if env("NODE_ENV", "") == "" { "development" } else { env("NODE_ENV") }}}

build: _env-setup
    npm run build

test: _env-setup
    NODE_ENV=test npm run test
```

### Anti-Pattern 2: Hardcoding Secrets in Justfiles

```Just
# ❌ Never commit secrets
deploy:
    ./deploy.sh --key "super-secret-key"
```

```Just
# ✅ Use environment variables
set dotenv-load

deploy:
    ./deploy.sh --key "$DEPLOY_KEY"
```

### Anti-Pattern 3: Over-Nesting Dependencies

```Just
# ❌ Too many layers
a: b
b: c
c: d
d: e
```

```Just
# ✅ Flatter is better
a: d
b: d
c: d
d:
    echo "Shared setup"
```

### Anti-Pattern 4: Recipes That Do Too Much

```Just
# ❌ One giant recipe
deploy-everything:
    git pull
    npm ci
    npm run lint
    npm run test
    npm run build
    docker build -t app .
    docker push registry/app
    kubectl set image deploy/app app=registry/app:latest
```

```just
# ✅ Break into logical steps
git-pull:
    git pull

install:
    npm ci

ci: lint test build

lint:
    npm run lint

test:
    npm run test

build:
    npm run build

docker-build:
    docker build -t app .

docker-push: docker-build
    docker push registry/app

deploy: git-pull ci docker-push
    kubectl set image deploy/app app=registry/app:latest
```

### Anti-Pattern 5: Ignoring Platform Differences

```Just
# ❌ Won't work on Windows
path := "/usr/local/bin"
```

```Just
# ✅ Platform-aware
path := if os_family() == "windows" { "C:\\Program Files\\App" } else { "/usr/local/bin" }
```

---

<a name="performance"></a>
## 21. Performance Considerations

### 1. Minimize Shell Invocations

Avoid spawning unnecessary subprocesses:

```Just
# ❌ Multiple shell calls
build:
    mkdir -p dist
    cp -r src/* dist/
    minify dist/*.js
```

```Just
# ✅ Single shell invocation
build:
    @mkdir -p dist && cp -r src/* dist/ && minify dist/*.js
```

### 2. Cache Expensive Operations

```Just
# Only rebuild if source changed
build:
    @if [ -d target ]; then \
        cargo build; \
    else \
        cargo build --fresh; \
    fi
```

### 3. Use Parallel Execution Where Possible

```Just
# Run independent checks in parallel
test: lint format typecheck
    npm test
```

### 4. Optimize Justfile Parsing

Large justfiles with many imports can slow down parsing. Consider splitting rarely-used recipes into separate justfiles that are imported conditionally.

### 5. Leverage Static Analysis

`just` performs static analysis before execution. Take advantage of this to catch errors early:

```console
$ just --evaluate build
cc main.c foo.c bar.c -o main
```

---

<a name="security"></a>
## 22. Security Considerations

### 1. Protect Secrets

Never hardcode secrets in justfiles. Use `.env` files (which should be in `.gitignore`) or external secret managers:

```Just
set dotenv-load

# .env file (never committed)
DATABASE_URL=postgres://user:pass@localhost/db
API_KEY=supersecret
```

### 2. Validate User Input

When recipes accept parameters, validate them before use:

```Just
# Sanitize inputs to prevent injection
test path:
    @if echo "{{path}}" | grep -qE '^[a-zA-Z0-9_/]+$'; then \
        echo "Invalid path"; \
        exit 1; \
    fi
    ./test --path {{path}}
```

### 3. Use Read-Only Modes in Production

```Just
# Dry-run mode for testing
dry-run:
    cargo build --dry-run
```

### 4. Restrict Shell Access

```Just
set shell := ["sh", "-c"]
```

Avoid using bash-specific features unless necessary, as `sh` is more portable and has fewer attack surfaces.

### 5. Audit Dependencies

Regularly audit your project dependencies:

```Just
audit:
    npm audit
    cargo audit
    pip-audit -r requirements.txt
```

---

<a name="testing"></a>
## 23. Testing Strategies

### 1. Test Your Justfile Itself

```Just
# Validate justfile
validate:
    just --list
    just --show build
    just --show test
```

### 2. Unit Testing Recipes

```Just
# Test individual components
test-unit:
    python -m pytest tests/unit/

# Integration testing
test-integration:
    python -m pytest tests/integration/

# Full test suite
test: test-unit test-integration
    @echo "✅ All tests passed!"
```

### 3. Dry-Run for Safety

```console
$ just --dry-run deploy
```

This shows what would run without actually executing it.

### 4. CI Validation

```Just
# Run in CI to validate the justfile
ci-validate:
    just --list --evaluate
    just --summary
```

### 5. Test Cross-Platform Compatibility

```Just
test-platforms:
    {{ if os() == "linux" { "echo 'On Linux'" } else if os() == "macos" { "echo 'On macOS'" } else { "echo 'On Windows'" } }}
```

---

<a name="migration"></a>
## 24. Migration Guide: From `make` to `just`

### Step 1: Identify Your Makefile Targets

```makefile
# Makefile
.PHONY: build test deploy clean

build:
	gcc main.c -o app

test: build
	./app --test

deploy: build
	scp app server:/opt/

clean:
	rm -f app
```

### Step 2: Convert to Justfile

```Just
# justfile
build:
    gcc main.c -o app

test: build
    ./app --test

deploy: build
    scp app server:/opt/

clean:
    rm -f app
```

### Step 3: Add Enhancements

```Just
# justfile with enhancements
version := "1.0.0"
app_name := "myapp"

build:
    @echo "Building {{app_name}} v{{version}}..."
    gcc main.c -o {{app_name}}

test: build
    ./{{app_name}} --test

deploy env="staging":
    @echo "Deploying to {{env}}..."
    scp {{app_name}} server:/opt/

clean:
    rm -f {{app_name}}

[default]
help:
    @just --list
```

### Key Differences

| `make` | `just` |
|---|---|
| Needs `.PHONY` declaration | All recipes are phony by default |
| Tab-sensitive syntax | Flexible indentation |
| Complex argument parsing | First-class parameters |
| No built-in shell scripting | Shebang recipe support |
| Cryptic error messages | Clear, contextual errors |

---

<a name="summary"></a>
## 25. Summary and Key Takeaways

### What We Covered

1. **`just` is a modern command runner** designed for running named tasks, not building software from file dependencies
2. **Installation is straightforward** across all platforms (macOS, Linux, Windows)
3. **Justfiles are simple** — recipes with optional dependencies, parameters, and variables
4. **Cross-platform compatibility** is built-in with settings like `set windows-shell`
5. **Multi-language support** via shebang recipes enables Python, Node.js, Bash, and more
6. **Module system** allows splitting justfiles across files for large projects
7. **Real-world use cases** span from simple builds to complex CI/CD pipelines
8. **Best practices** include using dependencies, aliases, and private recipes
9. **Security considerations** involve protecting secrets and validating inputs
10. **Testing strategies** ensure your justfiles are reliable and predictable

### Key Insights

- `just` eliminates the boilerplate and complexity of `make` while keeping its discoverability
- Recipes can accept parameters, have default values, and support flags/options
- The module system enables monorepo-scale organization
- `.env` file support automates environment variable management
- Shebang recipes let you write complex logic in any language
- Static analysis catches errors before execution

---

<a name="resources"></a>
## 26. Further Reading and Resources

### Official Documentation
- [just Manual](https://just.systems/) — Official documentation
- [GitHub Repository](https://github.com/casey/just) — Source code and issues
- [Cheat Sheet](https://github.com/casey/just/blob/master/just.1.ron) — Quick reference

### Community Resources
- [just Examples](https://github.com/casey/just/tree/master/examples) — Official examples
- [Awesome just](https://github.com/casey/just#awesome-just) — Community-curated list

### Related Tools
- [`make`](https://www.gnu.org/software/make/) — The traditional build tool (for comparison)
- [`task`](https://taskfile.dev/) — Another modern task runner
- [`nmake`](https://learn.microsoft.com/en-us/cpp/build/nmake-reference) — Microsoft's make variant

### Books and Articles
- "just: A Command Runner" — Official documentation
- "Replacing Make with just" — Blog posts on migration
- "Task Runners in Modern Development" — Comparative analysis

---

<a name="exercises"></a>
## 27. Practice Exercises

### Exercise 1: Create a Basic Justfile

**Objective:** Create a justfile for a simple project with build, test, and clean recipes.

**Instructions:**
1. Create a new directory for a simple C project
2. Write a simple `main.c` file
3. Create a `justfile` with `build`, `test`, and `clean` recipes
4. Run each recipe to verify it works

**Solution:**

```just
# justfile for simple C project
build:
    cc main.c -o main

test: build
    ./main

clean:
    rm -f main
```

```c
// main.c
#include <stdio.h>

int main() {
    printf("Hello, just!\n");
    return 0;
}
```

```console
$ just build
cc main.c -o main
$ just test
./main
Hello, just!
$ just clean
rm -f main
```

### Exercise 2: Add Parameters and Environment Variables

**Objective:** Extend your justfile to accept parameters and use environment variables.

**Instructions:**
1. Add a `run` recipe that accepts a `--name` parameter
2. Add a `greet` recipe that uses an environment variable `GREETING`
3. Use `.env` file to set the `GREETING` variable
4. Test both recipes

**Solution:**

```just
set dotenv-load

greet:
    @echo "{{GREETING}}, World!"

run name="default":
    @echo "Running with name: {{name}}"
```

```
# .env
GREETING=Hello
```

```console
$ just greet
Hello, World!
$ just run --name Alice
Running with name: Alice
```

### Exercise 3: Create a Multi-Language Project Justfile

**Objective:** Create a justfile that runs recipes in different languages using shebang lines.

**Instructions:**
1. Create a justfile with three recipes
2. First recipe runs in Bash
3. Second recipe runs in Python
4. Third recipe runs in Node.js
5. Each recipe should print a message

**Solution:**

```just
hello-bash:
    #!/usr/bin/env bash
    echo "Hello from Bash!"
    echo "Current directory: $(pwd)"

hello-python:
    #!/usr/bin/env python3
    import sys
    print("Hello from Python!")
    print(f"Python version: {sys.version}")

hello-node:
    #!/usr/bin/env node
    console.log("Hello from Node.js!");
    console.log(`Node version: ${process.version}`);
```

```console
$ just hello-bash
Hello from Bash!
Current directory: /path/to/project
$ just hello-python
Hello from Python!
Python version: 3.x.x
$ just hello-node
Hello from Node.js!
Node version: v18.x.x
```

---

<a name="question-bank"></a>
## 28. Question Bank

### Beginner Questions (1-20)

1. What is `just` primarily designed for?
2. What is the name of the file `just` uses to store commands?
3. How do you run the first recipe in a justfile?
4. What does the `@` prefix do in a recipe line?
5. How does `just` search for a justfile?
6. What file naming conventions does `just` support?
7. How do you add a dependency to a recipe?
8. What happens if a recipe line fails?
9. How do you suppress command echoing in `just`?
10. What is a "private" recipe and how is it named?
11. How do you define a variable in a justfile?
12. What is the syntax for string interpolation in `just`?
13. How do you load `.env` files automatically?
14. What are aliases used for?
15. How do you change the shell used for recipes?
16. What is the default shell on Unix-like systems?
17. How do you list all available recipes?
18. What is a shebang recipe?
19. How do you run a specific justfile?
20. What is the `[default]` attribute used for?

### Intermediate Questions (21-40)

21. How do you define a parameter with a default value?
22. What is the difference between `+` and `*` variadic parameters?
23. How do you create a no-value flag parameter?
24. What does the `os_family()` function return?
25. How do you use conditional expressions in variable definitions?
26. What is the purpose of the `justfile_directory()` function?
27. How do modules differ from imports in `just`?
28. What happens when you pass arguments to multiple recipes?
29. How do you export parameters as environment variables?
30. What is the `set export` setting used for?
31. How do you pass arguments to a dependency recipe?
32. What is the difference between positional and named parameters?
33. How do you configure a different shell on Windows?
34. What does the `shell()` built-in function do?
35. How do you create a recipe that accepts any number of files?
36. What is word-splitting and how can it affect recipes?
37. How do you use the `uppercase()` function?
38. What is the purpose of the `invocation_directory()` function?
39. How do you make a recipe work on both Windows and Unix?
40. What is the `[private]` attribute?

### Advanced Questions (41-50)

41. How does `just` perform static analysis?
42. What is the difference between `os()` and `os_family()`?
43. How do you create a cross-platform deployment recipe?
44. What is the purpose of the `set positional-arguments` setting?
45. How do you use built-in functions in parameter defaults?
46. What happens if dependencies form a circular graph?
47. How do you create a recipe that only shows in help?
48. What is the difference between `mod` and `import`?
49. How do you use environment variables in conditionals?
50. What is the `--dry-run` flag used for?

### Additional Questions (51-75)

51. How does `just` handle errors in dependencies?
52. What is the syntax for the `[arg]` attribute?
53. How do you specify both `short` and `long` flag names?
54. What happens when a recipe with a shebang is executed?
55. How do you use the `replace()` string function?
56. What is the difference between `trim()` and other string functions?
57. How do you create a recipe that runs tests in parallel?
58. What is the purpose of the `--choose` flag?
59. How do you use the `shell()` function safely?
60. What security considerations apply to recipe parameters?
61. How do you handle sensitive data in justfiles?
62. What is the `--justfile` flag used for?
63. How do you specify a custom working directory?
64. What is the `--summary` flag useful for?
65. How do you evaluate a recipe without running it?
66. What is the `--show` flag used for?
67. How do you handle multiple justfiles in a project?
68. What is the difference between `just --show` and `just --list`?
69. How do you create a recipe that confirms before execution?
70. What is the `[no-cd]` attribute used for?
71. How do you use the `if-else` expression in recipes?
72. What is the purpose of the `--evaluate` flag?
73. How do you handle errors in shell scripts within recipes?
74. What is the difference between `set export` and `set dotenv-load`?
75. How do you create a recipe that conditionally loads dependencies?

---

<a name="test-understanding"></a>
## 29. Test Your Understanding

### Test Questions (1-10)

1. **Multiple Choice:** Running `just` with no arguments will:
   - a) Show an error
   - b) Run the first recipe
   - c) Run the `[default]` recipe
   - d) Run both b and c

2. **True/False:** `just` only searches the current directory for a justfile.
   - a) True
   - b) False

3. **Fill in the Blank:** The `@` prefix in a recipe line ______.

4. **Short Answer:** What is the difference between `os()` and `os_family()`?

5. **Multiple Choice:** Which syntax creates a private recipe?
   - a) `# private`
   - b) `_recipe-name:`
   - c) `[private] recipe-name:`
   - d) Both b and c

6. **True/False:** Variadic parameters with `+` accept zero or more arguments.
   - a) True
   - b) False

7. **Fill in the Blank:** The `set ______` setting is used to automatically load `.env` files.

8. **Short Answer:** How do you pass arguments to a dependency recipe?

9. **Multiple Choice:** Which function returns the directory containing the justfile?
   - a) `invocation_directory()`
   - b) `justfile_directory()`
   - c) `current_dir()`
   - d) `pwd()`

10. **True/False:** Shebang recipes are written to a temporary file and executed directly.
    - a) True
    - b) False

### Answers:
1. d) Both b and c
2. b) False (searches upward through parent directories)
3. Suppresses echoing of the command before execution
4. `os()` returns the OS name (e.g., "linux", "macos"), `os_family()` returns the OS family (e.g., "unix", "windows")
5. d) Both b and c
6. b) False (requires at least one argument)
7. `dotenv-load`
8. Using `(dependency param)` syntax in the recipe header
9. b) `justfile_directory()`
10. a) True

---

<a name="interview-questions"></a>
## 30. Common Interview Questions

### 10 Interview Questions

1. **"What problem does `just` solve that `make` doesn't?"**
   - `just` removes the file-dependency model and timestamp tracking that `make` was designed for, focusing purely on running named commands. It eliminates `.PHONY` boilerplate, provides better error messages, and supports modern features like first-class parameters and multi-language recipes.

2. **"How does `just` find the justfile in a project?"**
   - `just` searches the current directory first, then traverses upward through parent directories until it finds a justfile. This allows running `just` from any subdirectory within a project.

3. **"Explain the difference between `mod` and `import` in justfiles."**
   - `mod` creates a namespaced submodule (`module::recipe`), while `import` flattens another file's recipes into the current namespace (making them directly accessible).

4. **"How do you handle cross-platform compatibility in a justfile?"**
   - Use `os()` or `os_family()` functions to conditionally execute commands, and configure `set windows-shell` for Windows-specific shell behavior.

5. **"What are private recipes and when would you use them?"**
   - Private recipes (prefixed with `_` or marked `[private]`) are hidden from `just --list` but can still be used as dependencies. Use them for internal implementation steps that users shouldn't run directly.

6. **"How does `just` handle errors in recipe execution?"**
   - `just` stops execution immediately when any line in a recipe exits with a nonzero status. Dependencies are also resolved statically before execution, catching errors like unknown recipes or circular dependencies upfront.

7. **"What are shebang recipes and why are they useful?"**
   - Shebang recipes use a `#!` line to specify an interpreter (Python, Node.js, Bash, etc.), allowing complex logic in any language while remaining discoverable through `just --list`.

8. **"How do you pass arguments to a `just` recipe?"**
   - Parameters are defined after the recipe name (e.g., `build target:`), invoked with `just build my-target`. Flags use the `[arg(...)]` attribute for GNU-style `--flag` syntax.

9. **"What is the purpose of the `set export` directive?"**
   - `set export` automatically exports all `just` variables as environment variables, making them available to recipe commands without explicit interpolation.

10. **"How would you migrate a project from `make` to `just`?"**
    - Convert `.PHONY` targets directly (all `just` recipes are effectively phony). Replace tab indentation with spaces or consistent tabs. Add `[default]` attributes where needed. Replace manual argument parsing with `just`'s built-in parameter support.

---

<a name="checklist"></a>
## 31. Self-Assessment Checklist

After completing this tutorial, check your understanding:

- [ ] I can install `just` on my system
- [ ] I can create a basic justfile with recipes
- [ ] I understand how dependencies work
- [ ] I can use parameters (positional, optional, variadic)
- [ ] I know how to use variables and expressions
- [ ] I can write shebang recipes in different languages
- [ ] I understand settings (`set shell`, `set dotenv-load`, etc.)
- [ ] I know how to use aliases and private recipes
- [ ] I can split justfiles using modules and imports
- [ ] I understand error handling in recipes
- [ ] I can write cross-platform justfiles
- [ ] I know how to troubleshoot common issues
- [ ] I can apply best practices in my justfiles
- [ ] I understand security considerations
- [ ] I know how to test and validate justfiles
- [ ] I can migrate from `make` to `just`
- [ ] I understand when to use `just` vs other tools

---

<a name="labs"></a>
## 32. Hands-On Labs

### Lab 1: Justfile for a Personal Project

**Objective:** Create a justfile for a personal project (e.g., a blog, portfolio, or CLI tool).

**Tasks:**
1. Choose a personal project (or create a simple one)
2. Identify 5-10 commands you regularly run
3. Create a justfile with these commands as recipes
4. Add dependencies between related recipes
5. Add at least one parameter with a default value
6. Add comments documenting each recipe
7. Test all recipes to ensure they work

### Lab 2: Cross-Platform Justfile

**Objective:** Create a justfile that works on both macOS/Windows/Linux.

**Tasks:**
1. Create a justfile with a `setup` recipe
2. The recipe should install dependencies using the appropriate package manager
3. Use `os_family()` or `os()` to detect the platform
4. Test on different platforms (or simulate using conditionals)
5. Add a `build` recipe that uses platform-specific commands

### Lab 3: Module-Based Justfile for a Monorepo

**Objective:** Create a modular justfile structure for a monorepo.

**Tasks:**
1. Create a directory structure for a monorepo (e.g., `frontend/`, `backend/`)
2. Create separate justfiles in each subdirectory
3. Create a root justfile that uses `mod` to import them
4. Add at least 3 recipes in each sub-justfile
5. Create a root-level recipe that depends on submodule recipes
6. Test that `just frontend::build` and `just backend::build` work

---

<a name="pro-tips"></a>
## 33. Pro Tips

### Tip 1: Use `--dry-run` to Preview Execution

```console
$ just --dry-run deploy
```

This shows exactly what would run without executing it — invaluable for complex deployment recipes.

### Tip 2: Interactive Recipe Selection

```console
$ just --choose
```

Presents an interactive menu for selecting a recipe to run.

### Tip 3: Dynamic Recipe Discovery

Add a `help` recipe that serves as the default:

```just
[default]
help:
    @printf "Available recipes:\n\n"
    @just --list --format "{{recipe.name}} ({{recipe.description}})\n"
```

### Tip 4: Use `just --summary` for Automation

```console
$ just --summary
build test deploy clean
```

Output is space-separated, perfect for shell scripting or validation.

### Tip 5: Combine with Shell Aliases

```bash
# In your .bashrc or .zshrc
alias j="just"
```

Now you can type `j build` instead of `just build`.

### Tip 6: Version Control Your Justfile

Always include your justfile in version control:

```bash
$ git add justfile
$ git commit -m "Add justfile with project recipes"
```

### Tip 7: Use `--show` to Inspect Recipes

```console
$ just --show deploy
```

Displays the full recipe definition including body and dependencies.

### Tip 8: Set Up Global Justfile

You can create a personal justfile at `~/.user.justfile` and alias it:

```bash
alias jg="just --justfile ~/.user.justfile"
```

This lets you use `just` recipes as personal utilities across all projects.

---

<a name="recap"></a>
## 34. Quick Recap

### Key Concepts
- **justfile**: The file containing your recipes (case-insensitive name)
- **recipe**: A named command with optional dependencies and parameters
- **dependency**: A recipe that must run before another
- **parameter**: Arguments passed to recipes (with defaults, variadic support)
- **variable**: Named values assigned with `:=` and interpolated with `{{}}`
- **module**: Namespaced sub-justfiles for large projects
- **shebang recipe**: Recipes that run in any language via `#!` line

### Essential Commands
| Command | Purpose |
|---|---|
| `just` | Run default recipe |
| `just RECIPE` | Run a specific recipe |
| `just --list` | List all public recipes |
| `just --dry-run RECIPE` | Preview without executing |
| `just --choose` | Interactive recipe selection |
| `just --justfile PATH` | Use a specific justfile |
| `just --show RECIPE` | Display recipe definition |

### Next Steps
1. Add a `justfile` to your current project with 2-3 recipes (`build`, `test`, `run`)
2. Run `just --list` to confirm they're discoverable
3. Gradually migrate any `README.md` "how to run this" instructions into recipes
4. Explore the [official manual](https://just.systems/man/en/) for advanced topics

---

*This tutorial was created following comprehensive documentation standards. For the latest `just` features and syntax, always refer to the [official documentation](https://just.systems/).*
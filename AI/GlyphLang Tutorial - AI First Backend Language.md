# GlyphLang Tutorial: AI-First Backend Language

GlyphLang is a symbol-based backend programming language designed for efficient AI/LLM code generation, reducing token usage significantly compared to traditional languages like Python or Java. This tutorial covers installation, core syntax, and practical examples for building web APIs, database interactions, and async services. Use it to test rapid prototyping in AI-assisted workflows.

## Installation and Setup
Download the GlyphLang runtime from the official site or GitHub repository (https://github.com/GlyphLang/GlyphLang).

- Clone the repo: `git clone https://github.com/GlyphLang/GlyphLang`
- Build (requires Go): `go build`
- Run files: `./glyphlang yourfile.glyph`

The compiler achieves sub-microsecond times (e.g., 867 ns), with JIT support for fast execution.

Example hello world setup:
```bash
# Save as hello.glyph
echo 'print("Hello GlyphLang")' > hello.glyph  # Note: Use actual symbols in practice
./glyphlang hello.glyph
```

## Core Syntax Symbols
GlyphLang uses compact symbols for LLM efficiency:

| Symbol | Meaning | Python Equivalent |
|--------|---------|-------------------|
| @      | Route/decorator | @app.route() [3] |
| $      | Variable declaration | var / def param [3] |
| >      | Return | return [3] |
| { }    | Block scope | def: / if: [1] |
| =      | Assignment | = [4] |
| :      | Path param | <id> [3] |

No keywords like "def", "return", or "async"—symbols handle everything. Indentation optional; braces enforce structure.

## Basic Examples
### Hello World
```glyphlang
print("Hello GlyphLang!")
```
Outputs: `Hello GlyphLang!` Compiles in nanoseconds.

### Variable and Functions
```glyphlang
$ msg = "Welcome"

dec greet:func string(void) {
  > msg
}

print(greet())
```
- `dec` declares functions (minimal boilerplate).
- `$` prefixes variables.
- Runs: `Welcome`

## Web API Routing
Core strength: Token-efficient routes for AI generation.

### Simple GET Endpoint
```glyphlang
@import "http"

@ GET /users/:id {
  $ user = db.query("SELECT * FROM users WHERE id = ?", id)
  > user
}
```
Equivalent to Python Flask: `@app.route('/users/<id>') def get_user(id): ...` Saves ~45% tokens.

### POST with JSON
```glyphlang
@ POST /users {
  $ body = req.json()
  $ id = db.insert("users", body)
  > {id: id}
}
```
Handles JSON automatically; supports WebSockets natively.

Run server: `./glyphlang api.glyph --serve :8080`

## Database Integration
Built-in PostgreSQL support (Redis via libs).

```glyphlang
@import "postgres"

db = connect("postgres://user:pass@localhost/db")

@ GET /users {
  $ users = db.query("SELECT * FROM users")
  > users
}

@ POST /users {
  $ user = req.json()
  db.exec("INSERT INTO users (name, email) VALUES (?, ?)", user.name, user.email)
  > {success: true}
}
```
Query params bind automatically; generics for type safety.

## Async and Advanced Features
### Async Operations
```glyphlang
@import "async"

@ GET /heavy {
  async {
    $ data = await fetch("https://api.example.com")
    > data
  }
}
```
Native async/await with JIT optimization.

### Generics Example
```glyphlang
dec fetch<T>:func T(url:string) {
  $ res = http.get(url)
  > res.json<T>()
}

$ data = fetch<User>("https://api/users/1")
```
Type-safe generics reduce AI errors.

## Error Handling
```glyphlang
@ GET /users/:id {
  try {
    $ user = db.query("SELECT * FROM users WHERE id = ?", id)
    if (user == null) {
      > {error: "User not found"} 404
    }
    > user
  } catch (e) {
    > {error: e.message} 500
  }
}
```
Status codes via trailing numbers.

## Full Example: User API Server
Combine into `user-api.glyph`:
```glyphlang
@import "http"
@import "postgres"

db = connect("postgres://localhost/mydb")

@ GET /users {
  $ users = db.query("SELECT * FROM users")
  > users
}

@ GET /users/:id {
  $ user = db.query("SELECT * FROM users WHERE id = ?", id)
  > user
}

@ POST /users {
  $ user = req.json()
  $ id = db.insert("users", user)
  > {id: id}
}

@ PUT /users/:id {
  $ updates = req.json()
  db.exec("UPDATE users SET ? WHERE id = ?", updates, id)
  > {success: true}
}

@ DELETE /users/:id {
  db.exec("DELETE FROM users WHERE id = ?", id)
  > {success: true}
}
```
Start: `./glyphlang user-api.glyph --serve :8080`

Test with curl:
```bash
curl http://localhost:8080/users  # List
curl -X POST -d '{"name":"Alice"}' http://localhost:8080/users
```

## Testing and Deployment
- VS Code extension for syntax highlighting/intellisense.
- Docker: Official images available.
- Production: Bytecode export for edge deployment.

## AI Prompting Tips
Prompt LLMs: "Write a GlyphLang POST /orders endpoint with PostgreSQL insert and validation."
Efficient due to symbol density.

For more, check docs: <https://glyphlang.dev/docs>

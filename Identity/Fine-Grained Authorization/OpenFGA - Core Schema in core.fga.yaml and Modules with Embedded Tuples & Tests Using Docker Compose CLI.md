# Modular Authorization with OpenFGA: Core Schema in `core.fga.yaml` and Modules with Embedded Tuples & Tests Using Docker Compose CLI

***

This tutorial covers organizing your OpenFGA authorization model with a `core.fga.yaml` file for base schema types, and `.mod` files for modules that include schema extensions, related tuples, and tests together. It demonstrates running OpenFGA server and CLI with Docker Compose and managing modular schemas, tuples, and tests efficiently via the CLI using a consolidated `.fga.yaml` configuration.

***

## Table of Contents

1. Detailed Directory Structure and Project Layout (with diagrams and explanations)  
2. Setup OpenFGA Server and CLI with Docker Compose  
3. File Organization: core schema, modules, tuples, and tests  
4. Example core schema: `core.fga.yaml`  
5. Example module file: `module1.mod` (proper use of `extend` keyword + tuples + tests)  
6. `.fga.yaml` CLI configuration referencing core and modules  
7. Writing schema, tuples, and running tests via Docker Compose CLI  
8. Comprehensive OpenFGA CLI Commands with Examples and Explanations  
9. Benefits of this modular structure  
10. Simplifying CLI usage with ephemeral shell aliases  
11. Explanation: Empty Curly Brackets `{}` in Schema Definitions  

***

## 1. Detailed Directory Structure and Project Layout

Visualizing your project's directory and understanding where each file lives is crucial for both beginners and advanced users, especially when working with containerized environments like Docker Compose.

***

### Project Root Overview

```
your-project-root/
│
├── docker-compose.yml          # Orchestrates OpenFGA server, CLI, Playground containers
├── fga-workdir/                # Authorization model files, tuples, tests, CLI config
│   ├── core.fga.yaml           # Core schema defining base types and relations
│   ├── module1.mod             # Module extending core with schema, tuples, and tests
│   ├── module2.mod             # Additional modules structured similarly
│   ├── .fga.yaml               # CLI configuration referencing all model and test files
│   ├── tuples.yaml             # Optional if tuples are separate from modules
│   └── tests.yaml              # Optional if tests are separate from modules
├── README.md                   # Documentation, instructions (optional but recommended)
└── app-code/                   # Your application source code and configs
```

***

### Diagram: High-Level Directory Structure

```
+ your-project-root/
  |
  |-- docker-compose.yml
  |
  |-- fga-workdir/
  |     |-- core.fga.yaml
  |     |-- module1.mod
  |     |-- module2.mod
  |     |-- .fga.yaml
  |     |-- tuples.yaml (optional)
  |     |-- tests.yaml (optional)
  |
  |-- README.md
  |
  `-- app-code/
```

***

### Explanation

- **docker-compose.yml**: The main Docker Compose configuration, placed at the root to control container lifecycle.
- **fga-workdir/**: This folder is mounted into OpenFGA’s server and CLI containers at runtime. It houses **all** schema definitions, modular extensions, access tuples, and tests. Keeping all authorization model-related files here ensures version control and easy edits.
- **core.fga.yaml**: Defines the base types and relations foundational to your authorization model.
- **module1.mod, module2.mod**: Modular files that extend or add specific features on top of the core schema. Each module can include schema extensions, tuples, and tests, making the model modular and maintainable.
- **.fga.yaml**: Central CLI configuration file that references your core schema and all modular files, guiding the OpenFGA CLI in loading, applying, and testing your authorization logic.
- Optional **tuples.yaml** and **tests.yaml** can be used if you prefer separating tuples and tests outside modules.
- **README.md**: It is good practice to keep a project readme to explain usage.
- **app-code/**: Your application codes are stored here, outside the OpenFGA domain, preserving separation of concerns.

***

### Interaction Flow Diagram

```plaintext
+---------------------+
| User code / Client   |
+---------------------+
         |
         v
+---------------------+          Mount         +----------------------+
| docker-compose.yml   |---------------------->| fga-workdir/          |
| - openfga-server     |                       | - core.fga.yaml       |
| - openfga-cli        |                       | - module1.mod         |
| - playground UI      |                       | - .fga.yaml           |
+---------------------+                       +----------------------+
         |                                        ^
         | API and CLI calls                      |
         ------------------------------------------
```

- The server listens on ports 8080 (HTTP), 8081 (gRPC), and 3000 (Playground UI).
- The CLI container uses the `.fga.yaml` to load model files and interact via API URL pointing to the server.
- The Playground UI accessible on `localhost:3000` provides a web interface to visualize and interact with your models.

***

## 2. Setup OpenFGA Server and CLI Using Docker Compose

Create `docker-compose.yml` to run server and CLI containers:

```yaml
version: "3.8"

services:
  openfga-server:
    image: openfga/openfga:latest
    container_name: openfga-server
    ports:
      - "8080:8080"
      - "8081:8081"
      - "3000:3000"  # Playground UI
    command: ["run"]
    restart: unless-stopped

  openfga-cli:
    image: openfga/cli:latest
    container_name: openfga-cli
    depends_on:
      - openfga-server
    environment:
      - FGA_API_URL=http://openfga-server:8080
    volumes:
      - ./fga-workdir:/workdir
    working_dir: /workdir
    tty: true
    stdin_open: true
```

Start containers:

```bash
docker-compose up -d
```

***

## 3. File Organization

```
fga-workdir/
│
├── core.fga.yaml           # Core schema types only
├── module1.mod             # Module with schema extension, tuples, and tests
├── module2.mod             # Additional modules
└── .fga.yaml               # CLI config referencing core and modules
```

***

## 4. Example Core Schema: `core.fga.yaml`

```yaml
types:
  - type: user
  
  - type: document
    relations:
      define viewer: [user]
      define editor: [user]
```

Defines basic `user` and `document` types with relations permitting direct assignment from `user`.

***

## 5. Example Module File with Proper `extend` Usage: `module1.mod`

This module **extends** the `document` type defined in core schema and defines new types, tuples, and tests. The `extend` keyword is properly used to add new relations to existing types.

```yaml
types:
  - extend type document
      relations:
        define project_manager: [project#manager]

  - type: project
    relations:
      define manager: [user]
      define member: [user]

tuples:
  - user: user:alice
    relation: editor
    object: document:doc1

  - user: user:bob
    relation: member
    object: project:proj1

  - user: user:carol
    relation: manager
    object: project:proj1

tests:
  - description: "Alice can edit document doc1"
    query:
      user: user:alice
      relation: editor
      object: document:doc1
    expect:
      allowed: true

  - description: "Bob is a member of project proj1"
    query:
      user: user:bob
      relation: member
      object: project:proj1
    expect:
      allowed: true

  - description: "Carol is manager of project proj1"
    query:
      user: user:carol
      relation: manager
      object: project:proj1
    expect:
      allowed: true
```

***

## 6. `.fga.yaml` CLI Configuration File

```yaml
api-url: http://openfga-server:8080
store-id: "<your_store_id_here>"
model-files:
  - core.fga.yaml
  - module1.mod
  - module2.mod
```

The CLI merges core and module schemas, applies tuples, and executes tests accordingly.

***

## 7. Docker Compose CLI Commands

- Create a store and receive `store-id`:

```bash
docker-compose exec openfga-cli fga store create --name "my-store"
```

- Write combined model:

```bash
docker-compose exec openfga-cli fga model write --config /workdir/.fga.yaml
```

- Write tuples:

```bash
docker-compose exec openfga-cli fga tuple write --config /workdir/.fga.yaml
```

- Run tests:

```bash
docker-compose exec openfga-cli fga test run --config /workdir/.fga.yaml
```

***

## 8. Comprehensive OpenFGA CLI Commands with Examples and Explanations

### Store Commands

- **Create Store**

```bash
fga store create --name "store-name"
```

Creates a new authorization store and outputs its store ID.

- **List Stores**

```bash
fga store list
```

Lists all stores accessible by the CLI client.

***

### Model Commands

- **Write Model**

```bash
fga model write --file model.json     # or --config .fga.yaml referencing multiple files
```

Write or update the authorization schema model.

- **List Models**

```bash
fga model list --store-id <store_id>
```

List versions of models used in a store.

***

### Tuple Commands

- **Write Tuples**

```bash
fga tuple write --file tuples.yaml --store-id <store_id>
```

Add access control tuples (user-to-object relations).

- **Delete Tuples**

```bash
fga tuple delete --file tuples_to_delete.yaml --store-id <store_id>
```

Remove specific tuples.

- **Import Tuples**

```bash
fga tuple import --file large_tuples.yaml --store-id <store_id> --max-tuples-per-write 20 --max-parallel-requests 4
```

Efficient bulk tuple writing with concurrency controls.

***

### Query Commands

- **Check Permission**

```bash
fga query check --store-id <store_id> --user user:alice --relation editor --object document:doc1
```

Checks if a user has a particular relation on an object (returns allowed/denied).

- **List Objects**

```bash
fga query list-objects --store-id <store_id> --user user:bob --relation viewer document
```

Lists all documents the user can access with the specified relation.

- **List Relations**

```bash
fga query list-relations --store-id <store_id> --user user:alice document:doc1
```

Lists all relations a user has over a particular object.

***

### Test Commands

- **Run Tests**

```bash
fga test run --file tests.yaml --store-id <store_id>
```

Run defined test cases verifying schema behavioral expectations.

***

### Other Useful Commands

- **Get Store Metadata**

```bash
fga store get --store-id <store_id>
```

Retrieve metadata info about a store.

- **Version Info**

```bash
fga version
```

Checks the current CLI tool version.

***

## 9. Benefits of Proper `extend` Usage

- Allows adding relations to types defined in core without redefining the entire type.
- Encourages modularity and separation of concerns.
- Simplifies collaboration and scaling of authorization models.

***

## 10. Simplifying CLI Usage with Ephemeral Aliases

For ephemeral aliases valid for your current terminal session only, run:

```bash
alias fga='docker-compose exec openfga-cli fga'
alias fgash='docker-compose exec -it openfga-cli sh'
```

Use commands like:

```bash
fga store list
fga model write --config /workdir/.fga.yaml
fga tuple write --config /workdir/.fga.yaml
fga test run --config /workdir/.fga.yaml
fgash
```

***

## 11. Explanation: Empty Curly Brackets `{}`

- In relation definitions, `{}` after `"this"` means **direct assignment** of the relation on the object itself without computation or inheritance.
- Example:

```yaml
viewer:
  this: {}
```

This states that users can be directly related as viewers of the object.

***

This comprehensive tutorial equips you with a practical, modular approach to OpenFGA authorization model design, usage with Docker Compose, full CLI command reference with examples, and best practices for ease of development and scaling.

For official and evolving guidance, always check [OpenFGA documentation](https://openfga.dev/docs).


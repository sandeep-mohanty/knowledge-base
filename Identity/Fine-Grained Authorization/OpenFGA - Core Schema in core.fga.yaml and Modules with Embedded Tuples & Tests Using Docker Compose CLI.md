# Modular Authorization with OpenFGA: Core Schema in `core.fga.yaml` and Modules with Embedded Tuples & Tests Using Docker Compose CLI (Complete Detailed Tutorial)

***

This tutorial provides a complete and detailed guide for building modular OpenFGA authorization models, including:

- Core schema and modular extensions with tuples and tests
- Docker Compose setups with dedicated networks
- Two Docker Compose versions—with and without Postgres datastore
- CLI commands and best practices
- Full explanations and examples for a beginner-friendly experience

***

## Table of Contents

1. Project Root Overview with Directory Structure and Explanation  
2. Interaction Flow Diagram with Explanation  
3. Docker Compose Setup  
 a. Version 1 – Without Postgres (In-Memory)  
 b. Version 2 – With Postgres  
4. Core Schema Example (`core.fga.yaml`)  
5. Module 1 Example (`module1.mod`)  
6. Module 2 Example (`module2.mod`) with Detailed Explanation  
7. CLI Configuration (`.fga.yaml`)  
8. Docker Compose CLI Commands  
9. Comprehensive OpenFGA CLI Commands with Usage Examples  
10. Benefits of Modular Structure and Networking  
11. Ephemeral CLI Aliases  
12. Explanation of Empty Curly Brackets `{}` in Relations  

***

## 1. Project Root Overview & Directory Structure

```
your-project-root/
│
├── docker-compose.yml          
│  # Main file orchestrating all containers (OpenFGA server, CLI, optionally Postgres)
│
├── fga-workdir/                
│   ├── core.fga.yaml           # Base authorization schema types and relations
│   ├── module1.mod             # First module extending schema, tuples, and tests
│   ├── module2.mod             # Second module defining teams, extending project, plus tuples and tests
│   ├── .fga.yaml               # CLI configuration referencing all model and test files
│   ├── tuples.yaml             # Optional separate tuples file
│   └── tests.yaml              # Optional separate tests file
│
├── README.md                   # Documentation and instructions
│
└── app-code/                   # Your app source, configuration
```

**Explanation:**  
- The `docker-compose.yml` lives at project root to manage all container startup and networking.  
- `fga-workdir` holds your authorization model schemas and configuration files. It is mounted as a volume in server and CLI containers.  
- Separation of app source in `app-code/` keeps code and authorization configs independent.  
- Use `README.md` for project documentation and usage notes.

***

## 2. Interaction Flow Diagram

```plaintext
+------------------------+
| Application / Clients  |
+------------------------+
          |
          v
+--------------------------+               Mount                +----------------------------+
| docker-compose.yml        |----------------------------------->| fga-workdir/                |
| - openfga-server          |                                    | - core.fga.yaml             |
| - openfga-cli             |                                    | - module1.mod               |
| - postgres (optional)     |                                    | - module2.mod               |
| - playground UI           |                                    | - .fga.yaml                 |
+--------------------------+                                    +----------------------------+
          |                                                    ^
          | Internal API & CLI calls                            |
          ------------------------------------------------------
```

**Explanation:**  
- `openfga-server` exposes APIs on ports `8080` (HTTP), `8081` (gRPC), and the Playground UI on `3000`.  
- `openfga-cli` container communicates with `openfga-server` internally via Docker network, mounting `fga-workdir` for schema and test files.  
- Optional PostgreSQL service provides persistent storage for the server.  
- Playground UI provides interactive access for model testing without needing CLI.

***

## 3. Docker Compose Setup

### a. Version 1 – Without Postgres (In-Memory Datastore)

```yaml
version: "3.8"

services:
  openfga-server:
    image: openfga/openfga:latest
    container_name: openfga-server
    ports:
      - "8080:8080"
      - "8081:8081"
      - "3000:3000"
    command: ["run"]
    restart: unless-stopped
    volumes:
      - ./fga-workdir:/workdir
    networks:
      - openfga-network

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
    networks:
      - openfga-network

networks:
  openfga-network:
```

***

### b. Version 2 – With Postgres Persistence

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    container_name: openfga-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: openfga
      POSTGRES_USER: openfga_user
      POSTGRES_PASSWORD: openfga_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - openfga-network

  openfga-server:
    image: openfga/openfga:latest
    container_name: openfga-server
    depends_on:
      - postgres
    ports:
      - "8080:8080"
      - "8081:8081"
      - "3000:3000"
    command:
      - "run"
      - "--datastore-engine"
      - "postgres"
      - "--datastore-conn-string"
      - "postgres://openfga_user:openfga_password@postgres:5432/openfga?sslmode=disable"
    restart: unless-stopped
    volumes:
      - ./fga-workdir:/workdir
    networks:
      - openfga-network

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
    networks:
      - openfga-network

networks:
  openfga-network:

volumes:
  postgres-data:
```

***

## 4. Core Schema Example: `core.fga.yaml`

```yaml
schema_version: 1.1

types:
  - type: user
  
  - type: document
    relations:
      define viewer: [user]
      define editor: [user]
```

***

## 5. Module 1 Example: `module1.mod`

```yaml
schema_version: 1.1

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

## 6. Detailed Module 2 Example: `module2.mod`

```yaml
schema_version: 1.1

types:
  - type: team
    relations:
      define admin: [user]
      define member: [user, group#member]

  - extend type project
      relations:
        define team_admin: [team#admin]

tuples:
  - user: user:dave
    relation: admin
    object: team:team1

  - user: user:eve
    relation: member
    object: team:team1

  - user: user:dave
    relation: team_admin
    object: project:proj1

tests:
  - description: "Dave is admin of team1"
    query:
      user: user:dave
      relation: admin
      object: team:team1
    expect:
      allowed: true

  - description: "Eve is member of team1"
    query:
      user: user:eve
      relation: member
      object: team:team1
    expect:
      allowed: true

  - description: "Dave is team_admin of project proj1"
    query:
      user: user:dave
      relation: team_admin
      object: project:proj1
    expect:
      allowed: true
```

***

## 7. CLI Configuration: `.fga.yaml`

```yaml
schema_version: 1.1

api-url: http://openfga-server:8080
store-id: "<your_store_id_here>"
model-files:
  - core.fga.yaml
  - module1.mod
  - module2.mod
```

***

## 8. Docker Compose CLI Commands

```bash
docker-compose exec openfga-cli fga store create --name "my-store"

docker-compose exec openfga-cli fga model write --config /workdir/.fga.yaml

docker-compose exec openfga-cli fga tuple write --config /workdir/.fga.yaml

docker-compose exec openfga-cli fga test run --config /workdir/.fga.yaml
```

***

## 9. Comprehensive OpenFGA CLI Commands Overview

- **Store commands:**  
  `store create`, `store list`, `store get`

- **Model commands:**  
  `model write`, `model list`

- **Tuples commands:**  
  `tuple write`, `tuple delete`, `tuple import`

- **Query commands:**  
  `query check`, `query list-objects`, `query list-relations`

- **Test commands:**  
  `test run`

Example permission check:

```bash
fga query check --store-id <store_id> --user user:alice --relation editor --object document:doc1
```

***

## 10. Benefits and Best Practices

- Modular schema management facilitates collaborative development and scaling.
- Postgres enables persistent storage in production.
- Dedicated Docker network ensures isolated and controlled communication.
- Ephemeral CLI aliases simplify repeated command usage within sessions.

***

## 11. Simplifying CLI Usage with Ephemeral Aliases

```bash
alias fga='docker-compose exec openfga-cli fga'
alias fgash='docker-compose exec -it openfga-cli sh'
```

Use commands like `fga store list` or interact in shell via `fgash`.

***

## 12. Explanation: Empty Curly Brackets `{}` in Relation Definitions

Indicates a direct relation assignment on the object itself without intermediate computation.

Example:

```yaml
viewer:
  this: {}
```

Users assigned as `viewer` have direct access without derivation.

***

This concludes the fully detailed, inclusive OpenFGA tutorial with all requested content, files, and explanations suitable for users of all levels.

For official references on modeling and Docker setup, visit:

- [OpenFGA Modular Models Documentation](https://openfga.dev/docs/modeling/modular-models)  
- [OpenFGA Docker Setup](https://openfga.dev/docs/getting-started/setup-openfga/docker)


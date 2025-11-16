# OpenFGA Local Setup Guide: Complete Playground Environment

This comprehensive guide will help you set up a complete OpenFGA playground environment on your local machine using Docker Compose. The setup includes OpenFGA server, PostgreSQL database, and a dedicated CLI container - keeping your host machine completely clean.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [Setup Steps](#setup-steps)
4. [Working with the CLI Container](#working-with-the-cli-container)
5. [Testing and Experimentation](#testing-and-experimentation)
6. [Advanced Features](#advanced-features)
7. [Troubleshooting](#troubleshooting)
8. [Quick Reference](#quick-reference)

---

## Prerequisites

Before starting, ensure you have:

- **Docker Desktop** installed and running
  - [Download Docker Desktop](https://www.docker.com/products/docker-desktop)
  - Minimum version: Docker 20.10+ and Docker Compose 2.0+
- **Basic terminal/command line knowledge**
- **A code editor** (VS Code, IntelliJ, etc.)
- **At least 2GB free disk space**

### Verify Docker Installation

```bash
# Check Docker version
docker --version
# Should show: Docker version 20.10.x or higher

# Check Docker Compose version
docker compose version
# Should show: Docker Compose version v2.x.x or higher
```

***

## Project Structure

Create the following directory structure:

```
openfga-playground/
├── docker-compose.yml          # Docker orchestration
├── cli/
│   ├── Dockerfile              # CLI container image
│   ├── .bashrc                 # Shell configuration with aliases
│   └── scripts/                # Helper scripts
│       ├── help.sh             # Quick reference guide
│       ├── setup-store.sh      # Initial setup automation
│       ├── load-sample-data.sh # Sample data loader
│       └── manage.sh           # Interactive management console
├── models/                     # Authorization models (.fga files)
│   └── getting-started.fga     # Sample model
├── tests/                      # Test files
│   └── auth-tests.sh           # Authorization test suite
└── data/                       # Persistent data (auto-created by Docker)
```

***

## Setup Steps

### Step 1: Create Project Directory

```bash
# Create the main project directory
mkdir -p openfga-playground/cli/scripts
mkdir -p openfga-playground/models
mkdir -p openfga-playground/tests
cd openfga-playground
```

***

### Step 2: Create CLI Container Dockerfile

Create `cli/Dockerfile`:

```dockerfile
# Use Debian-based image for stability
FROM debian:bookworm-slim

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV FGA_VERSION=v0.2.3
ENV FGA_API_URL=http://openfga:8080

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    jq \
    vim \
    nano \
    git \
    ca-certificates \
    bash-completion \
    && rm -rf /var/lib/apt/lists/*

# Install OpenFGA CLI
RUN wget https://github.com/openfga/cli/releases/download/${FGA_VERSION}/fga_${FGA_VERSION#v}_linux_amd64.tar.gz \
    && tar -xzf fga_${FGA_VERSION#v}_linux_amd64.tar.gz \
    && chmod +x fga \
    && mv fga /usr/local/bin/ \
    && rm fga_${FGA_VERSION#v}_linux_amd64.tar.gz

# Create working directory
WORKDIR /workspace

# Create user for running CLI
RUN useradd -m -s /bin/bash openfga-user

# Copy configuration files
COPY .bashrc /home/openfga-user/.bashrc
COPY scripts /home/openfga-user/scripts

# Set ownership
RUN chown -R openfga-user:openfga-user /home/openfga-user /workspace

# Switch to non-root user
USER openfga-user

# Set default command
CMD ["/bin/bash"]
```

***

### Step 3: Create CLI Shell Configuration

Create `cli/.bashrc`:

```bash
# OpenFGA CLI Configuration

# Colors for better readability
export PS1='\[\033[01;32m\]openfga-cli\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# OpenFGA Configuration
export FGA_API_URL=${FGA_API_URL:-http://openfga:8080}
export OPENFGA_STORE_ID=${OPENFGA_STORE_ID:-}
export OPENFGA_MODEL_ID=${OPENFGA_MODEL_ID:-}

# Aliases for common commands
alias fga-version='fga version'
alias fga-stores='fga store list'
alias fga-models='fga model list --store-id $OPENFGA_STORE_ID'
alias fga-tuples='fga tuple read --store-id $OPENFGA_STORE_ID'
alias fga-health='curl -s http://openfga:8080/healthz | jq'

# Helper functions
function fga-check() {
    if [ -z "$1" ] || [ -z "$2" ] || [ -z "$3" ]; then
        echo "Usage: fga-check USER RELATION OBJECT"
        echo "Example: fga-check user:alice can_view document:doc1"
        return 1
    fi
    fga query check --store-id $OPENFGA_STORE_ID $1 $2 $3
}

function fga-write-tuple() {
    if [ -z "$1" ] || [ -z "$2" ] || [ -z "$3" ]; then
        echo "Usage: fga-write-tuple USER RELATION OBJECT"
        echo "Example: fga-write-tuple user:alice member organization:acme"
        return 1
    fi
    fga tuple write --store-id $OPENFGA_STORE_ID $1 $2 $3
}

function fga-list-objects() {
    if [ -z "$1" ] || [ -z "$2" ] || [ -z "$3" ]; then
        echo "Usage: fga-list-objects USER RELATION TYPE"
        echo "Example: fga-list-objects user:alice can_view document"
        return 1
    fi
    fga query list-objects --store-id $OPENFGA_STORE_ID --user $1 --relation $2 --type $3
}

# Welcome message
cat << "EOF"
┌─────────────────────────────────────────────────────────────┐
│              OpenFGA CLI Container                          │
│             Ready for experimentation!                      │
└─────────────────────────────────────────────────────────────┘

Available commands:
  fga                  - OpenFGA CLI
  fga-stores          - List all stores
  fga-models          - List models in current store
  fga-tuples          - Read tuples from current store
  fga-health          - Check OpenFGA server health
  fga-check           - Quick authorization check
  fga-write-tuple     - Quick tuple write
  fga-list-objects    - List objects user can access

Current configuration:
  API URL:    $FGA_API_URL
  Store ID:   ${OPENFGA_STORE_ID:-<not set>}
  Model ID:   ${OPENFGA_MODEL_ID:-<not set>}

Workspace: /workspace
Models:    /workspace/models
Scripts:   ~/scripts

Type 'scripts/help.sh' for more information.
Type 'scripts/manage.sh' for interactive console.
EOF

# Load bash completion if available
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi
```

***

### Step 4: Create Helper Scripts

#### Create `cli/scripts/help.sh`:

```bash
#!/bin/bash

cat << "EOF"
╔════════════════════════════════════════════════════════════╗
║              OpenFGA CLI Quick Reference                   ║
╚════════════════════════════════════════════════════════════╝

STORE MANAGEMENT:
  fga store list                           - List all stores
  fga store create --name "Store Name"     - Create new store
  fga store get --store-id <ID>            - Get store details

MODEL MANAGEMENT:
  fga model list --store-id <ID>           - List all models
  fga model write --store-id <ID> --file <FILE> - Deploy model
  fga model get --store-id <ID>            - Get latest model

TUPLE OPERATIONS:
  fga tuple write --store-id <ID> USER RELATION OBJECT
  fga tuple read --store-id <ID>
  fga tuple delete --store-id <ID> USER RELATION OBJECT

QUERIES:
  fga query check --store-id <ID> USER RELATION OBJECT
  fga query list-objects --store-id <ID> --user USER --relation REL --type TYPE
  fga query expand --store-id <ID> --relation REL --object OBJECT

CUSTOM ALIASES:
  fga-stores                               - List stores
  fga-models                               - List models
  fga-tuples                               - Read tuples
  fga-health                               - Server health check
  fga-check USER RELATION OBJECT           - Quick auth check
  fga-write-tuple USER RELATION OBJECT     - Quick tuple write
  fga-list-objects USER RELATION TYPE      - List accessible objects

EXAMPLES:
  fga store create --name "My Store"
  fga model write --store-id 01HV... --file models/model.fga
  fga tuple write --store-id 01HV... user:alice member organization:acme
  fga-check user:alice can_view document:doc1

ENVIRONMENT VARIABLES:
  export OPENFGA_STORE_ID=<your-store-id>
  export OPENFGA_MODEL_ID=<your-model-id>

AUTOMATION SCRIPTS:
  scripts/setup-store.sh          - Initial store setup
  scripts/load-sample-data.sh     - Load sample data
  scripts/manage.sh               - Interactive management console
EOF
```

#### Create `cli/scripts/setup-store.sh`:

```bash
#!/bin/bash
# Initial store setup script

echo "🚀 Setting up OpenFGA store..."

# Create store
echo "Creating store..."
STORE_RESPONSE=$(fga store create --name "Playground Store" --format json)
STORE_ID=$(echo $STORE_RESPONSE | jq -r '.id')

echo "✅ Store created: $STORE_ID"

# Export for current session
export OPENFGA_STORE_ID=$STORE_ID

# Save to file for persistence
echo "export OPENFGA_STORE_ID=$STORE_ID" > ~/.store_config
echo "💾 Store ID saved to ~/.store_config"

# Check if model file exists
if [ -f "/workspace/models/getting-started.fga" ]; then
    echo "📝 Deploying authorization model..."
    MODEL_RESPONSE=$(fga model write --store-id $STORE_ID --file /workspace/models/getting-started.fga --format json)
    MODEL_ID=$(echo $MODEL_RESPONSE | jq -r '.authorization_model_id')
    
    echo "✅ Model deployed: $MODEL_ID"
    export OPENFGA_MODEL_ID=$MODEL_ID
    echo "export OPENFGA_MODEL_ID=$MODEL_ID" >> ~/.store_config
    echo "💾 Model ID saved to ~/.store_config"
else
    echo "⚠️  No model file found at /workspace/models/getting-started.fga"
    echo "📝 Create a model file and run: fga model write --store-id $STORE_ID --file /workspace/models/your-model.fga"
fi

echo ""
echo "🎉 Setup complete!"
echo "To use these IDs in new shells, run: source ~/.store_config"
```

#### Create `cli/scripts/load-sample-data.sh`:

```bash
#!/bin/bash
# Load sample data

if [ -z "$OPENFGA_STORE_ID" ]; then
    echo "❌ OPENFGA_STORE_ID not set"
    echo "Run: source ~/.store_config"
    exit 1
fi

echo "📥 Loading sample data..."

# Users and Organizations
echo "Creating organization memberships..."
fga tuple write --store-id $OPENFGA_STORE_ID user:alice member organization:acme
fga tuple write --store-id $OPENFGA_STORE_ID user:bob admin organization:acme
fga tuple write --store-id $OPENFGA_STORE_ID user:charlie member organization:acme

# Documents
echo "Creating document relationships..."
fga tuple write --store-id $OPENFGA_STORE_ID organization:acme organization document:report-2025
fga tuple write --store-id $OPENFGA_STORE_ID user:alice owner document:report-2025
fga tuple write --store-id $OPENFGA_STORE_ID user:bob editor document:report-2025
fga tuple write --store-id $OPENFGA_STORE_ID user:charlie viewer document:report-2025

echo "✅ Sample data loaded successfully!"
echo ""
echo "Try these commands:"
echo "  fga-check user:alice can_view document:report-2025"
echo "  fga-check user:charlie can_delete document:report-2025"
echo "  fga-list-objects user:bob can_edit document"
```

#### Create `cli/scripts/manage.sh`:

```bash
#!/bin/bash
# Interactive management console

source ~/.store_config 2>/dev/null

function main_menu() {
    clear
    cat << "EOF"
╔════════════════════════════════════════════════════════════╗
║         OpenFGA Management Console                         ║
╚════════════════════════════════════════════════════════════╝
EOF
    echo ""
    echo "Current Configuration:"
    echo "  Store ID: ${OPENFGA_STORE_ID:-<not set>}"
    echo "  Model ID: ${OPENFGA_MODEL_ID:-<not set>}"
    echo ""
    echo "1.  Store Management"
    echo "2.  Model Management"
    echo "3.  Tuple Management"
    echo "4.  Query & Check"
    echo "5.  Load Sample Data"
    echo "6.  Run Tests"
    echo "7.  Server Health"
    echo "8.  Configuration"
    echo "9.  Help"
    echo "0.  Exit"
    echo ""
    read -p "Select option: " choice
    
    case $choice in
        1) store_menu ;;
        2) model_menu ;;
        3) tuple_menu ;;
        4) query_menu ;;
        5) ~/scripts/load-sample-data.sh; pause ;;
        6) /workspace/tests/auth-tests.sh 2>/dev/null || echo "No tests found"; pause ;;
        7) fga-health; pause ;;
        8) config_menu ;;
        9) ~/scripts/help.sh; pause ;;
        0) exit 0 ;;
        *) echo "Invalid option"; sleep 1 ;;
    esac
    
    main_menu
}

function store_menu() {
    clear
    echo "════════════════════════════════════════════════════════════"
    echo "         Store Management"
    echo "════════════════════════════════════════════════════════════"
    echo ""
    echo "1. List all stores"
    echo "2. Create new store"
    echo "3. Get current store details"
    echo "4. Back"
    echo ""
    read -p "Select option: " choice
    
    case $choice in
        1) fga store list; pause ;;
        2) create_store ;;
        3) get_store_details ;;
        4) return ;;
    esac
    
    store_menu
}

function create_store() {
    read -p "Enter store name: " name
    RESPONSE=$(fga store create --name "$name" --format json)
    STORE_ID=$(echo $RESPONSE | jq -r '.id')
    echo "✅ Store created: $STORE_ID"
    echo "export OPENFGA_STORE_ID=$STORE_ID" >> ~/.store_config
    export OPENFGA_STORE_ID=$STORE_ID
    pause
}

function get_store_details() {
    if [ -z "$OPENFGA_STORE_ID" ]; then
        echo "❌ Store ID not set"
    else
        fga store get --store-id $OPENFGA_STORE_ID
    fi
    pause
}

function model_menu() {
    clear
    echo "════════════════════════════════════════════════════════════"
    echo "         Model Management"
    echo "════════════════════════════════════════════════════════════"
    echo ""
    echo "1. List models"
    echo "2. Deploy model from file"
    echo "3. Get latest model"
    echo "4. Back"
    echo ""
    read -p "Select option: " choice
    
    case $choice in
        1) fga-models; pause ;;
        2) deploy_model ;;
        3) get_latest_model ;;
        4) return ;;
    esac
    
    model_menu
}

function deploy_model() {
    echo "Available models in /workspace/models/:"
    ls -1 /workspace/models/ 2>/dev/null || echo "No models found"
    echo ""
    read -p "Enter model filename: " filename
    
    if [ -f "/workspace/models/$filename" ]; then
        RESPONSE=$(fga model write --store-id $OPENFGA_STORE_ID --file /workspace/models/$filename --format json)
        MODEL_ID=$(echo $RESPONSE | jq -r '.authorization_model_id')
        echo "✅ Model deployed: $MODEL_ID"
        echo "export OPENFGA_MODEL_ID=$MODEL_ID" >> ~/.store_config
        export OPENFGA_MODEL_ID=$MODEL_ID
    else
        echo "❌ File not found: /workspace/models/$filename"
    fi
    pause
}

function get_latest_model() {
    fga model get --store-id $OPENFGA_STORE_ID
    pause
}

function tuple_menu() {
    clear
    echo "════════════════════════════════════════════════════════════"
    echo "         Tuple Management"
    echo "════════════════════════════════════════════════════════════"
    echo ""
    echo "1. Read all tuples"
    echo "2. Write tuple"
    echo "3. Delete tuple"
    echo "4. Back"
    echo ""
    read -p "Select option: " choice
    
    case $choice in
        1) fga-tuples | head -50; echo ""; echo "(Showing first 50 tuples)"; pause ;;
        2) write_tuple ;;
        3) delete_tuple ;;
        4) return ;;
    esac
    
    tuple_menu
}

function write_tuple() {
    read -p "Enter user (e.g., user:alice): " user
    read -p "Enter relation (e.g., member): " relation
    read -p "Enter object (e.g., organization:acme): " object
    fga-write-tuple $user $relation $object
    pause
}

function delete_tuple() {
    read -p "Enter user: " user
    read -p "Enter relation: " relation
    read -p "Enter object: " object
    fga tuple delete --store-id $OPENFGA_STORE_ID $user $relation $object
    echo "✅ Tuple deleted"
    pause
}

function query_menu() {
    clear
    echo "════════════════════════════════════════════════════════════"
    echo "         Query & Check"
    echo "════════════════════════════════════════════════════════════"
    echo ""
    echo "1. Check authorization"
    echo "2. List objects"
    echo "3. Expand relationship"
    echo "4. Back"
    echo ""
    read -p "Select option: " choice
    
    case $choice in
        1) check_auth ;;
        2) list_objects ;;
        3) expand_rel ;;
        4) return ;;
    esac
    
    query_menu
}

function check_auth() {
    read -p "Enter user (e.g., user:alice): " user
    read -p "Enter relation (e.g., can_view): " relation
    read -p "Enter object (e.g., document:doc1): " object
    fga-check $user $relation $object
    pause
}

function list_objects() {
    read -p "Enter user: " user
    read -p "Enter relation: " relation
    read -p "Enter type: " type
    fga-list-objects $user $relation $type
    pause
}

function expand_rel() {
    read -p "Enter relation: " relation
    read -p "Enter object: " object
    fga query expand --store-id $OPENFGA_STORE_ID --relation $relation --object $object
    pause
}

function config_menu() {
    clear
    echo "════════════════════════════════════════════════════════════"
    echo "         Configuration"
    echo "════════════════════════════════════════════════════════════"
    echo ""
    echo "Current Configuration:"
    echo "  API URL:  $FGA_API_URL"
    echo "  Store ID: ${OPENFGA_STORE_ID:-<not set>}"
    echo "  Model ID: ${OPENFGA_MODEL_ID:-<not set>}"
    echo ""
    echo "1. Set Store ID"
    echo "2. Set Model ID"
    echo "3. Reload configuration"
    echo "4. Show configuration file"
    echo "5. Back"
    echo ""
    read -p "Select option: " choice
    
    case $choice in
        1) set_store_id ;;
        2) set_model_id ;;
        3) source ~/.store_config; echo "✅ Configuration reloaded"; pause ;;
        4) cat ~/.store_config 2>/dev/null || echo "No configuration file found"; pause ;;
        5) return ;;
    esac
    
    config_menu
}

function set_store_id() {
    read -p "Enter Store ID: " id
    echo "export OPENFGA_STORE_ID=$id" >> ~/.store_config
    export OPENFGA_STORE_ID=$id
    echo "✅ Store ID set"
    pause
}

function set_model_id() {
    read -p "Enter Model ID: " id
    echo "export OPENFGA_MODEL_ID=$id" >> ~/.store_config
    export OPENFGA_MODEL_ID=$id
    echo "✅ Model ID set"
    pause
}

function pause() {
    echo ""
    read -p "Press Enter to continue..."
}

# Start the main menu
main_menu
```

**Make all scripts executable:**

```bash
chmod +x cli/scripts/*.sh
```

***

### Step 5: Create Sample Authorization Model

Create `models/getting-started.fga`:

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define admin: [user]
    define can_view: admin or member
    define can_manage: admin

type document
  relations
    define organization: [organization]
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    define can_edit: owner or editor
    define can_view: can_edit or viewer or (can_view from organization)
    define can_delete: owner
```

***

### Step 6: Create Test Suite

Create `tests/auth-tests.sh`:

```bash
#!/bin/bash
# Authorization test suite

if [ -z "$OPENFGA_STORE_ID" ]; then
    echo "❌ OPENFGA_STORE_ID not set. Run: source ~/.store_config"
    exit 1
fi

echo "════════════════════════════════════════════════════════════"
echo "         Running Authorization Tests"
echo "════════════════════════════════════════════════════════════"
echo ""

# Test suite
declare -a tests=(
    "user:alice can_view document:report-2025 true"
    "user:alice can_edit document:report-2025 true"
    "user:alice can_delete document:report-2025 true"
    "user:bob can_view document:report-2025 true"
    "user:bob can_edit document:report-2025 true"
    "user:bob can_delete document:report-2025 false"
    "user:charlie can_view document:report-2025 true"
    "user:charlie can_edit document:report-2025 false"
    "user:charlie can_delete document:report-2025 false"
    "user:bob can_manage organization:acme true"
    "user:alice can_view organization:acme true"
    "user:charlie can_manage organization:acme false"
)

passed=0
failed=0

for test in "${tests[@]}"; do
    IFS=' ' read -r user relation object expected <<< "$test"
    
    result=$(fga query check --store-id $OPENFGA_STORE_ID $user $relation $object --format json 2>/dev/null | jq -r '.allowed')
    
    if [ "$result" = "$expected" ]; then
        echo "✅ PASS: $user $relation $object = $result"
        ((passed++))
    else
        echo "❌ FAIL: $user $relation $object = $result (expected: $expected)"
        ((failed++))
    fi
done

echo ""
echo "════════════════════════════════════════════════════════════"
echo "Test Results: $passed passed, $failed failed"
echo "════════════════════════════════════════════════════════════"
```

Make it executable:

```bash
chmod +x tests/auth-tests.sh
```

***

### Step 7: Create Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
version: '3.8'

networks:
  openfga:
    driver: bridge

volumes:
  postgres_data:
    driver: local
  cli_home:
    driver: local

services:
  # PostgreSQL Database
  postgres:
    image: postgres:17
    container_name: openfga-postgres
    environment:
      - POSTGRES_USER=openfga
      - POSTGRES_PASSWORD=openfga_password
      - POSTGRES_DB=openfga
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - openfga
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U openfga"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Database Migration
  migrate:
    image: openfga/openfga:latest
    container_name: openfga-migrate
    depends_on:
      postgres:
        condition: service_healthy
    command: migrate
    environment:
      - OPENFGA_DATASTORE_ENGINE=postgres
      - OPENFGA_DATASTORE_URI=postgres://openfga:openfga_password@postgres:5432/openfga?sslmode=disable
    networks:
      - openfga

  # OpenFGA Server
  openfga:
    image: openfga/openfga:latest
    container_name: openfga-server
    depends_on:
      migrate:
        condition: service_completed_successfully
    environment:
      - OPENFGA_DATASTORE_ENGINE=postgres
      - OPENFGA_DATASTORE_URI=postgres://openfga:openfga_password@postgres:5432/openfga?sslmode=disable
      - OPENFGA_LOG_FORMAT=text
      - OPENFGA_LOG_LEVEL=info
    command: run
    networks:
      - openfga
    ports:
      - "8080:8080"    # HTTP API
      - "8081:8081"    # gRPC API
      - "3000:3000"    # Playground UI
    healthcheck:
      test: ["CMD", "/usr/local/bin/grpc_health_probe", "-addr=:8081"]
      interval: 5s
      timeout: 30s
      retries: 3

  # OpenFGA CLI Container
  cli:
    build:
      context: ./cli
      dockerfile: Dockerfile
    container_name: openfga-cli
    depends_on:
      openfga:
        condition: service_healthy
    environment:
      - FGA_API_URL=http://openfga:8080
      - OPENFGA_STORE_ID=${OPENFGA_STORE_ID:-}
      - OPENFGA_MODEL_ID=${OPENFGA_MODEL_ID:-}
    volumes:
      # Mount model and test files from host
      - ./models:/workspace/models
      - ./tests:/workspace/tests
      # Persist CLI container home directory
      - cli_home:/home/openfga-user
    networks:
      - openfga
    stdin_open: true
    tty: true
    command: /bin/bash
```

***

### Step 8: Start the Playground Environment

```bash
# Build and start all services
docker compose up -d --build

# Check service status
docker compose ps

# Expected output:
# NAME                 STATUS
# openfga-postgres     Up (healthy)
# openfga-server       Up (healthy)
# openfga-cli          Up
# openfga-migrate      Exited (0)

# View logs to verify everything started correctly
docker compose logs -f
```

***

## Working with the CLI Container

### Step 9: Enter the CLI Container

```bash
# Enter the CLI container
docker compose exec cli bash

# You should see the welcome banner with available commands
```

You'll be greeted with:

```
┌─────────────────────────────────────────────────────────────┐
│              OpenFGA CLI Container                          │
│             Ready for experimentation!                      │
└─────────────────────────────────────────────────────────────┘

Available commands:
  fga                  - OpenFGA CLI
  fga-stores          - List all stores
  fga-models          - List models in current store
  ...
```

***

### Step 10: Initial Setup (Inside CLI Container)

```bash
# Verify CLI installation
fga version

# Check OpenFGA server health
fga-health

# Run the automated setup script
scripts/setup-store.sh

# This script will:
# 1. Create a new store
# 2. Deploy the model from /workspace/models/getting-started.fga
# 3. Save Store ID and Model ID to ~/.store_config

# Load configuration in current shell
source ~/.store_config

# Verify configuration
echo "Store ID: $OPENFGA_STORE_ID"
echo "Model ID: $OPENFGA_MODEL_ID"
```

***

### Step 11: Load Sample Data (Inside CLI Container)

```bash
# Load sample tuples
scripts/load-sample-data.sh

# Verify data was loaded
fga-tuples

# You should see relationships for alice, bob, charlie, and documents
```

***

## Testing and Experimentation

### Step 12: Test Authorization Checks (Inside CLI Container)

```bash
# Quick checks using aliases
fga-check user:alice can_view document:report-2025
# Output: {"allowed":true}

fga-check user:charlie can_delete document:report-2025
# Output: {"allowed":false}

fga-check user:bob can_manage organization:acme
# Output: {"allowed":true}

# List what documents Alice can view
fga-list-objects user:alice can_view document

# List what documents Bob can edit
fga-list-objects user:bob can_edit document
```

***

### Step 13: Run the Test Suite (Inside CLI Container)

```bash
# Run automated tests
/workspace/tests/auth-tests.sh

# Expected output:
# ✅ PASS: user:alice can_view document:report-2025 = true
# ✅ PASS: user:alice can_edit document:report-2025 = true
# ✅ PASS: user:alice can_delete document:report-2025 = true
# ...
# Test Results: 12 passed, 0 failed
```

***

### Step 14: Interactive Management Console (Inside CLI Container)

```bash
# Launch the interactive console
scripts/manage.sh

# You'll see a menu-driven interface for:
# - Store management
# - Model deployment
# - Tuple operations
# - Authorization queries
# - Configuration
```

***

### Step 15: Experiment with Your Own Models

#### On Your Host Machine:

1. Create a new model file in `models/my-model.fga`
2. Edit it with your favorite IDE (VS Code, IntelliJ, etc.)

```fga
model
  schema 1.1

type user

type printer
  relations
    define organization: [organization]
    define can_access: [user]

type organization
  relations
    define member: [user]
```

#### In CLI Container:

```bash
# Deploy your new model
fga model write --store-id $OPENFGA_STORE_ID --file /workspace/models/my-model.fga

# Save the new model ID
NEW_MODEL_ID=$(fga model list --store-id $OPENFGA_STORE_ID --format json | jq -r '.authorization_models[0].id')
echo "export OPENFGA_MODEL_ID=$NEW_MODEL_ID" >> ~/.store_config
export OPENFGA_MODEL_ID=$NEW_MODEL_ID

# Write some tuples
fga-write-tuple user:alice member organization:acme
fga-write-tuple organization:acme organization printer:pr-001
fga-write-tuple user:bob can_access printer:pr-001

# Test
fga-check user:bob can_access printer:pr-001
```

***

## Advanced Features

### Step 16: Use the Playground UI (Optional)

OpenFGA includes a web-based playground for visual modeling:

1. Open your browser to: http://localhost:3000
2. Enter your Store ID (from `$OPENFGA_STORE_ID`)
3. Visualize your authorization model
4. Test relationships interactively

**Note:** The playground is for local development only - never expose it in production!

---

### Step 17: Multiple Shell Sessions

You can open multiple CLI container sessions:

```bash
# Terminal 1: Enter CLI container
docker compose exec cli bash

# Terminal 2: Open another session (from host)
docker compose exec cli bash

# Both sessions share the same store and configuration
```

***

### Step 18: Inspect Relationship Graphs

```bash
# Inside CLI container

# Expand how user:alice can view document:report-2025
fga query expand --store-id $OPENFGA_STORE_ID \
  --relation can_view \
  --object document:report-2025

# This shows all the paths through which access is granted
```

***

### Step 19: Working with Multiple Stores

```bash
# Create a second store for experimentation
STORE2_ID=$(fga store create --name "Experiment Store" --format json | jq -r '.id')

# Use it temporarily
fga model write --store-id $STORE2_ID --file /workspace/models/getting-started.fga

# Check queries against specific store
fga query check --store-id $STORE2_ID user:test can_view document:test
```

***

## Troubleshooting

### Services Won't Start

```bash
# From host machine

# Check Docker is running
docker info

# View all logs
docker compose logs

# View specific service logs
docker compose logs openfga
docker compose logs postgres

# Restart services
docker compose restart

# Complete restart
docker compose down
docker compose up -d
```

***

### Can't Connect to OpenFGA from CLI

```bash
# Inside CLI container

# Test connectivity
curl http://openfga:8080/healthz

# Check environment variables
echo $FGA_API_URL

# Verify OpenFGA is running
docker compose ps openfga
```

***

### Database Migration Failed

```bash
# From host machine

# Manually run migration
docker compose run --rm migrate

# Or recreate everything
docker compose down -v
docker compose up -d
```

***

### Lost Store ID

```bash
# Inside CLI container

# List all stores
fga-stores

# Find your store by name and copy the ID

# Set it in configuration
echo "export OPENFGA_STORE_ID=<your-store-id>" >> ~/.store_config
source ~/.store_config
```

***

### CLI Container Won't Start

```bash
# From host machine

# Rebuild CLI container
docker compose build cli

# Force recreate
docker compose up -d --force-recreate cli

# Check build logs
docker compose logs cli
```

***

## Quick Reference

### Container Management (From Host)

```bash
# Start environment
docker compose up -d

# Stop environment
docker compose down

# Rebuild CLI container after changes
docker compose build cli
docker compose up -d --force-recreate cli

# Enter CLI container
docker compose exec cli bash

# View logs
docker compose logs -f
docker compose logs -f cli
docker compose logs -f openfga

# Restart a service
docker compose restart cli
docker compose restart openfga

# Complete cleanup (deletes all data!)
docker compose down -v
```

***

### Inside CLI Container

```bash
# First-time setup
scripts/setup-store.sh
source ~/.store_config

# Load sample data
scripts/load-sample-data.sh

# Interactive management
scripts/manage.sh

# Quick reference
scripts/help.sh

# Common operations
fga-health                                    # Check server
fga-stores                                    # List stores
fga-models                                    # List models
fga-tuples                                    # Show tuples
fga-check user:alice can_view document:doc1  # Auth check
fga-write-tuple user:bob member org:acme     # Write tuple
fga-list-objects user:alice can_view document # List objects

# Run tests
/workspace/tests/auth-tests.sh

# Deploy a model
fga model write --store-id $OPENFGA_STORE_ID --file /workspace/models/my-model.fga

# Exit container
exit
```

***

### Project Structure Summary

```
openfga-playground/
├── docker-compose.yml              # Orchestration
├── cli/
│   ├── Dockerfile                  # CLI image
│   ├── .bashrc                     # Shell config
│   └── scripts/
│       ├── help.sh                 # Quick ref
│       ├── setup-store.sh          # Initial setup
│       ├── load-sample-data.sh     # Sample data
│       └── manage.sh               # Interactive console
├── models/
│   └── getting-started.fga         # Sample model
├── tests/
│   └── auth-tests.sh               # Test suite
└── data/                           # Auto-created volumes
```

***

## Summary

You now have a complete OpenFGA playground with:

✅ **Clean Host Machine** - No CLI installed locally, everything in containers
✅ **OpenFGA Server** - Running with PostgreSQL backend
✅ **Dedicated CLI Container** - Full OpenFGA CLI with helper scripts
✅ **Interactive Console** - Menu-driven management interface
✅ **Helper Aliases** - Quick commands for common operations
✅ **Sample Model & Data** - Ready-to-use authorization setup
✅ **Test Suite** - Automated testing framework
✅ **Hot Reload** - Edit models on host, use immediately in container
✅ **Persistent Configuration** - Store/Model IDs saved across restarts
✅ **Web Playground** - Visual interface at localhost:3000

### Next Steps

1. **Experiment with the sample model** - Try different queries and scenarios
2. **Create your own models** - Model your organization's access control
3. **Write custom tests** - Expand the test suite for your use cases
4. **Learn advanced features** - Conditions, contextual authorization
5. **Integrate with your code** - Use SDKs to connect your applications

Happy experimenting! 🎉

***

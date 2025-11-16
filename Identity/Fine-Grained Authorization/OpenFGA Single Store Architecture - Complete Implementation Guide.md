# OpenFGA Single Store Architecture: Complete Implementation Guide

## Table of Contents

1. [Overview](#overview)
2. [Architecture Philosophy](#architecture-philosophy)
3. [Initial Store Setup Flow](#initial-store-setup-flow)
4. [Store Distribution via ConfigMap](#store-distribution-via-configmap)
5. [Module Development Structure](#module-development-structure)
6. [Deployment Pipeline](#deployment-pipeline)
7. [Service Integration](#service-integration)
8. [Disaster Recovery and Store Retrieval](#disaster-recovery-and-store-retrieval)
9. [Operations and Maintenance](#operations-and-maintenance)
10. [Summary](#summary)

***

## Overview

This guide details the **Single Store Architecture** for OpenFGA authorization across multiple domain teams. In this approach, the Identity Team creates and manages a single OpenFGA store that serves as the centralized authorization database for all domains.

### Key Principles

- **One Store Per Environment**: Single store for dev, staging, and production
- **Centralized Configuration**: Store ID distributed via Kubernetes ConfigMap
- **Modular Models**: Each team owns their domain-specific module file
- **Combined Deployment**: All modules deployed together to single store
- **One-Time Setup**: Store creation is a one-time activity per environment

---

## Architecture Philosophy

### Mental Model: Central Library

```
┌─────────────────────────────────────────────────────────────────┐
│                    Single Store Concept                         │
│               "One Central Authorization Database"              │
└─────────────────────────────────────────────────────────────────┘

Think of it as a single library building with multiple floors:

┌────────────────────────────────────────────────────────────────┐
│                    CENTRAL LIBRARY                             │
│                                                                │
│  Foundation Floor (Core):                                      │
│  • User records                                                │
│  • Organization records                                        │
│  • Group memberships                                           │
│  • System roles                                                │
│  → Used by ALL departments                                     │
│                                                                │
│  Identity Department Floor:                                    │
│  • Federation profiles                                         │
│  • SCIM configurations                                         │
│  • SSO settings                                                │
│  → Identity team manages                                       │
│                                                                │
│  Print Department Floor:                                       │
│  • Printer registry                                            │
│  • Print job tracking                                          │
│  • Print queue management                                      │
│  → Print team manages                                          │
│                                                                │
│  Fleet Department Floor:                                       │
│  • Managed printer fleet                                       │
│  • IoT device registry                                         │
│  • Device groups                                               │
│  → Fleet team manages                                          │
│                                                                │
│  D365 Department Floor:                                        │
│  • CRM contracts                                               │
│  • Sales opportunities                                         │
│  → D365 team manages                                           │
│                                                                │
│  ALL FLOORS IN SAME BUILDING                                   │
│  One unified catalog system                                    │
│  Anyone can search across all floors                           │
└────────────────────────────────────────────────────────────────┘
```

### Why Single Store?

```
✅ Advantages:
   • No data duplication
   • Natural cross-domain authorization
   • Single source of truth
   • Simple deployment
   • Lower cost
   • Better testing
   • OpenFGA recommended pattern

❌ What We Avoid:
   • Data synchronization issues
   • Stale data across stores
   • Complex deployment coordination
   • Higher infrastructure costs
   • Cross-store query limitations
```

***

## Initial Store Setup Flow

### Phase 1: Identity Team Creates Store (One-Time Activity)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  INITIAL STORE CREATION FLOW                                │
│                  (One-Time Activity Per Environment)                        │
└─────────────────────────────────────────────────────────────────────────────┘

Step 1: Identity Team Lead
┌──────────────────────────────────────┐
│ Decision: Create OpenFGA Store       │
│ For: Production Environment          │
│ When: Before any domain can deploy   │
└────────────────┬─────────────────────┘
                 │
                 ▼
Step 2: Execute Store Creation
┌──────────────────────────────────────────────────────────────┐
│ $ export FGA_API_URL="http://openfga.platform.internal:8080"│
│                                                              │
│ $ fga store create --name "Company Authorization - Prod"    │
│                                                              │
│ Response:                                                    │
│ {                                                            │
│   "id": "01HVPRD1234567890ABCDEFGH",                        │
│   "name": "Company Authorization - Prod",                   │
│   "created_at": "2025-11-16T09:00:00Z",                     │
│   "updated_at": "2025-11-16T09:00:00Z"                      │
│ }                                                            │
│                                                              │
│ ✅ Store Created Successfully                               │
│ Store ID: 01HVPRD1234567890ABCDEFGH                         │
└──────────────────────────────────────────────────────────────┘
                 │
                 │ CRITICAL: Save this Store ID
                 ▼
Step 3: Document Store ID
┌──────────────────────────────────────────────────────────────┐
│ Create: docs/openfga-stores.md                               │
│                                                              │
│ # OpenFGA Store IDs                                          │
│                                                              │
│ ## Production                                                │
│ - Store ID: `01HVPRD1234567890ABCDEFGH`                     │
│ - Created: 2025-11-16                                        │
│ - Created By: Identity Team Lead                            │
│ - Environment: Production                                    │
│ - Purpose: Company-wide authorization                        │
│                                                              │
│ ## Staging                                                   │
│ - Store ID: `01HVSTG0987654321ZYXWVUTS`                     │
│ - Created: 2025-11-15                                        │
│                                                              │
│ ## Development                                               │
│ - Store ID: `01HVDEV1111111111AAAAAAAA`                     │
│ - Created: 2025-11-14                                        │
│                                                              │
│ ⚠️  IMPORTANT: These IDs never change                       │
│ ⚠️  Backup this file                                        │
└──────────────────────────────────────────────────────────────┘
                 │
                 │ Commit to repository
                 ▼
Step 4: Create Kubernetes ConfigMap
┌──────────────────────────────────────────────────────────────┐
│ Create: k8s/platform/openfga-config.yaml                     │
│                                                              │
│ apiVersion: v1                                               │
│ kind: ConfigMap                                              │
│ metadata:                                                    │
│   name: openfga-config                                       │
│   namespace: platform                                        │
│ data:                                                        │
│   OPENFGA_API_URL: "http://openfga.svc.local:8080"         │
│   OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"            │
│   OPENFGA_MODEL_ID: ""  # Updated by pipeline               │
│                                                              │
│ $ kubectl apply -f k8s/platform/openfga-config.yaml         │
│ configmap/openfga-config created                            │
└──────────────────────────────────────────────────────────────┘
                 │
                 │ ConfigMap deployed to cluster
                 ▼
Step 5: Replicate ConfigMap to All Namespaces
┌──────────────────────────────────────────────────────────────┐
│ # Identity namespace                                         │
│ $ kubectl create configmap openfga-config \                 │
│   --from-literal=OPENFGA_API_URL="http://..." \             │
│   --from-literal=OPENFGA_STORE_ID="01HVPRD..." \            │
│   -n identity                                                │
│                                                              │
│ # Print namespace                                            │
│ $ kubectl create configmap openfga-config \                 │
│   --from-literal=OPENFGA_API_URL="http://..." \             │
│   --from-literal=OPENFGA_STORE_ID="01HVPRD..." \            │
│   -n print                                                   │
│                                                              │
│ # Fleet namespace                                            │
│ $ kubectl create configmap openfga-config \                 │
│   --from-literal=OPENFGA_API_URL="http://..." \             │
│   --from-literal=OPENFGA_STORE_ID="01HVPRD..." \            │
│   -n fleet                                                   │
│                                                              │
│ # D365 namespace                                             │
│ $ kubectl create configmap openfga-config \                 │
│   --from-literal=OPENFGA_API_URL="http://..." \             │
│   --from-literal=OPENFGA_STORE_ID="01HVPRD..." \            │
│   -n d365                                                    │
│                                                              │
│ ✅ All namespaces now have access to Store ID               │
└──────────────────────────────────────────────────────────────┘
                 │
                 ▼
Step 6: Notify All Domain Teams
┌──────────────────────────────────────────────────────────────┐
│ Email/Slack to: @print-team, @fleet-team, @d365-team        │
│                                                              │
│ Subject: OpenFGA Production Store Ready                      │
│                                                              │
│ Team,                                                        │
│                                                              │
│ The OpenFGA production store has been created and is        │
│ ready for use.                                               │
│                                                              │
│ Store ID: 01HVPRD1234567890ABCDEFGH                         │
│                                                              │
│ Access:                                                      │
│ - ConfigMap: openfga-config (in your namespace)             │
│ - Variables: OPENFGA_API_URL, OPENFGA_STORE_ID              │
│                                                              │
│ Next Steps:                                                  │
│ 1. Update your services to use ConfigMap                    │
│ 2. Create your domain module (.fga file)                    │
│ 3. Submit PR to authorization-models repo                   │
│                                                              │
│ Documentation: docs/openfga-stores.md                        │
│                                                              │
│ Questions: #openfga-support                                  │
│                                                              │
│ - Identity Team                                              │
└──────────────────────────────────────────────────────────────┘
                 │
                 │ Teams notified
                 ▼
┌──────────────────────────────────────────────────────────────┐
│                ✅ STORE SETUP COMPLETE                       │
│                                                              │
│ This is a ONE-TIME activity per environment                  │
│ Store ID never changes                                       │
│ All teams now point to same store                           │
└──────────────────────────────────────────────────────────────┘
```

### Store Creation for All Environments

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  Environment-Specific Store Creation                        │
└─────────────────────────────────────────────────────────────────────────────┘

Development Environment
┌─────────────────────────────────────────┐
│ $ fga store create \                    │
│   --name "Company Auth - Dev"           │
│                                         │
│ Store ID: 01HVDEV1111111111AAAAAAAA    │
│                                         │
│ ConfigMap: dev-cluster/openfga-config   │
│ Purpose: Developer testing              │
└─────────────────────────────────────────┘

Staging Environment
┌─────────────────────────────────────────┐
│ $ fga store create \                    │
│   --name "Company Auth - Staging"       │
│                                         │
│ Store ID: 01HVSTG0987654321ZYXWVUTS    │
│                                         │
│ ConfigMap: staging-cluster/openfga-cfg  │
│ Purpose: Pre-production validation      │
└─────────────────────────────────────────┘

Production Environment
┌─────────────────────────────────────────┐
│ $ fga store create \                    │
│   --name "Company Auth - Production"    │
│                                         │
│ Store ID: 01HVPRD1234567890ABCDEFGH    │
│                                         │
│ ConfigMap: prod-cluster/openfga-config  │
│ Purpose: Production authorization       │
└─────────────────────────────────────────┘

⚠️  Each environment has its own store
⚠️  Store IDs are different per environment
⚠️  ConfigMap names/values differ per environment
```

***

## Store Distribution via ConfigMap

### ConfigMap Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Kubernetes ConfigMap Distribution                       │
└─────────────────────────────────────────────────────────────────────────────┘

                    ┌────────────────────────────┐
                    │  Platform Namespace        │
                    │  (Master ConfigMap)        │
                    ├────────────────────────────┤
                    │  openfga-config            │
                    │  - OPENFGA_API_URL         │
                    │  - OPENFGA_STORE_ID        │
                    │  - OPENFGA_MODEL_ID        │
                    └────────────┬───────────────┘
                                 │
                                 │ Replicated to
                                 │
        ┌────────────────────────┼────────────────────────┬──────────────┐
        │                        │                        │              │
        ▼                        ▼                        ▼              ▼
┌───────────────┐        ┌──────────────┐        ┌──────────┐    ┌──────────┐
│ identity      │        │  print       │        │  fleet   │    │  d365    │
│ namespace     │        │  namespace   │        │namespace │    │namespace │
├───────────────┤        ├──────────────┤        ├──────────┤    ├──────────┤
│ openfga-      │        │ openfga-     │        │openfga-  │    │openfga-  │
│ config        │        │ config       │        │config    │    │config    │
│               │        │              │        │          │    │          │
│ Same Store ID │        │Same Store ID │        │Same Store│    │Same Store│
└───────┬───────┘        └──────┬───────┘        └────┬─────┘    └────┬─────┘
        │                       │                     │               │
        │                       │                     │               │
        ▼                       ▼                     ▼               ▼
┌───────────────┐        ┌──────────────┐        ┌──────────┐    ┌──────────┐
│Identity       │        │Print         │        │Fleet     │    │D365      │
│Service        │        │Service       │        │Service   │    │Service   │
│               │        │              │        │          │    │          │
│References:    │        │References:   │        │Reference:│    │Reference:│
│ConfigMap      │        │ConfigMap     │        │ConfigMap │    │ConfigMap │
│               │        │              │        │          │    │          │
│Queries:       │        │Queries:      │        │Queries:  │    │Queries:  │
│SAME STORE     │        │SAME STORE    │        │SAME STORE│    │SAME STORE│
└───────────────┘        └──────────────┘        └──────────┘    └──────────┘
        │                       │                     │               │
        │                       │                     │               │
        └───────────────────────┴─────────────────────┴───────────────┘
                                 │
                                 │ All point to
                                 ▼
                    ┌────────────────────────────┐
                    │   OpenFGA Server           │
                    │                            │
                    │  Single Store:             │
                    │  01HVPRD1234567890ABCDEFGH │
                    │                            │
                    │  • Core model              │
                    │  • Identity module         │
                    │  • Print module            │
                    │  • Fleet module            │
                    │  • D365 module             │
                    │                            │
                    │  All domains together      │
                    └────────────────────────────┘
```

### ConfigMap Definition Examples

#### Platform Namespace (Master)

```yaml
# File: k8s/platform/openfga-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openfga-config
  namespace: platform
  labels:
    app: openfga
    managed-by: identity-team
  annotations:
    description: "OpenFGA store configuration for all services"
    created-by: "identity-team"
    created-date: "2025-11-16"
data:
  # OpenFGA API endpoint
  OPENFGA_API_URL: "http://openfga.platform.svc.cluster.local:8080"
  
  # Store ID - NEVER changes after creation
  OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"
  
  # Model ID - Updated by pipeline after each deployment
  OPENFGA_MODEL_ID: ""
  
  # Additional metadata
  ENVIRONMENT: "production"
  STORE_NAME: "Company Authorization - Production"
```

#### Domain Namespace (Replicated)

```yaml
# File: k8s/identity/openfga-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openfga-config
  namespace: identity
  labels:
    app: openfga
    domain: identity
data:
  OPENFGA_API_URL: "http://openfga.platform.svc.cluster.local:8080"
  OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"  # Same as platform
  OPENFGA_MODEL_ID: ""

---
# File: k8s/print/openfga-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openfga-config
  namespace: print
  labels:
    app: openfga
    domain: print
data:
  OPENFGA_API_URL: "http://openfga.platform.svc.cluster.local:8080"
  OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"  # Same as platform
  OPENFGA_MODEL_ID: ""

---
# File: k8s/fleet/openfga-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openfga-config
  namespace: fleet
  labels:
    app: openfga
    domain: fleet
data:
  OPENFGA_API_URL: "http://openfga.platform.svc.cluster.local:8080"
  OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"  # Same as platform
  OPENFGA_MODEL_ID: ""

---
# File: k8s/d365/openfga-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openfga-config
  namespace: d365
  labels:
    app: openfga
    domain: d365
data:
  OPENFGA_API_URL: "http://openfga.platform.svc.cluster.local:8080"
  OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"  # Same as platform
  OPENFGA_MODEL_ID: ""
```

***

## Module Development Structure

### Repository Organization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   Authorization Models Repository                           │
│                   github.com/your-org/authorization-models                  │
└─────────────────────────────────────────────────────────────────────────────┘

authorization-models/
├── README.md                          # Documentation
├── .github/
│   ├── workflows/
│   │   └── deploy-model.yml          # CI/CD pipeline
│   └── CODEOWNERS                     # Ownership enforcement
│
├── fga.mod                            # Manifest (combines all modules)
│
├── core/                              # Identity team - Foundation
│   ├── core.fga                       # Core types
│   └── README.md                      # Core module docs
│
├── identity/                          # Identity team - Domain specific
│   ├── identity.fga                   # Identity domain types
│   └── README.md                      # Identity module docs
│
├── print/                             # Print team
│   ├── print.fga                      # Print domain types
│   └── README.md                      # Print module docs
│
├── fleet/                             # Fleet team
│   ├── fleet.fga                      # Fleet domain types
│   └── README.md                      # Fleet module docs
│
├── d365/                              # D365 team
│   ├── d365.fga                       # D365 domain types
│   └── README.md                      # D365 module docs
│
├── tests/                             # All test files
│   ├── core-tests.yaml               # Core module tests
│   ├── identity-tests.yaml           # Identity module tests
│   ├── print-tests.yaml              # Print module tests
│   ├── fleet-tests.yaml              # Fleet module tests
│   ├── d365-tests.yaml               # D365 module tests
│   └── integration-tests.yaml        # Cross-domain tests
│
└── docs/                              # Documentation
    ├── openfga-stores.md             # Store IDs per environment
    ├── module-guidelines.md          # Module development guide
    └── deployment-guide.md           # Deployment procedures
```

### Module Dependency Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Module Dependency Structure                          │
└─────────────────────────────────────────────────────────────────────────────┘

                         ┌─────────────────┐
                         │   core.fga      │
                         │  (Foundation)   │
                         ├─────────────────┤
                         │ • type user     │
                         │ • type org      │
                         │ • type group    │
                         │ • type system   │
                         │   _role         │
                         └────────┬────────┘
                                  │
                                  │ All modules extend core
                                  │
         ┌────────────────────────┼────────────────────────┬──────────────┐
         │                        │                        │              │
         ▼                        ▼                        ▼              ▼
┌────────────────┐       ┌───────────────┐       ┌───────────┐   ┌───────────┐
│ identity.fga   │       │  print.fga    │       │fleet.fga  │   │ d365.fga  │
├────────────────┤       ├───────────────┤       ├───────────┤   ├───────────┤
│Federation      │       │Printer        │       │Managed    │   │Contract   │
│Profile         │       │               │       │Printer    │   │           │
│                │       │Print Job      │       │           │   │Opportunity│
│Federation      │       │               │       │IoT Device │   │           │
│Domain          │       │Print Queue    │       │           │   │           │
│                │       │               │       │Device Grp │   │           │
│SCIM Config     │       │Print          │       │           │   │           │
│                │       │Entitlement    │       │           │   │           │
└────────────────┘       └───────────────┘       └───────────┘   └───────────┘
         │                        │                      │              │
         │                        │                      │              │
         └────────────────────────┼──────────────────────┼──────────────┘
                                  │
                                  │ Combined by fga.mod
                                  ▼
                         ┌─────────────────┐
                         │  fga.mod        │
                         │  (Manifest)     │
                         ├─────────────────┤
                         │ modules:        │
                         │ - core.fga      │
                         │ - identity.fga  │
                         │ - print.fga     │
                         │ - fleet.fga     │
                         │ - d365.fga      │
                         └────────┬────────┘
                                  │
                                  │ Pipeline processes
                                  ▼
                         ┌─────────────────┐
                         │ Combined Model  │
                         │ (Single file)   │
                         ├─────────────────┤
                         │ All types from  │
                         │ all modules in  │
                         │ one model       │
                         └────────┬────────┘
                                  │
                                  │ Deployed to
                                  ▼
                         ┌─────────────────┐
                         │  Single Store   │
                         │  01HVPRD...     │
                         └─────────────────┘
```

### Core Module (Foundation)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Core Module Structure                                │
│                        File: core/core.fga                                  │
│                        Owner: Identity Team                                 │
└─────────────────────────────────────────────────────────────────────────────┘

Purpose:
────────
Defines foundational types used by ALL domains:
• Users
• Organizations
• Groups
• System roles

These types cannot be changed without Identity team approval.
All other modules depend on these types.

Structure:
──────────
┌─────────────────────────────────────┐
│ model                               │
│   schema 1.2                        │
│                                     │
│ type user                           │
│   # Represents people               │
│                                     │
│ type organization                   │
│   relations                         │
│     define member: [user, group#..] │
│     define org_admin: [user]        │
│     define user_admin: [user]       │
│     define parent: [organization]   │
│     define can_manage_users: ...    │
│     define can_view_org: ...        │
│                                     │
│ type group                          │
│   relations                         │
│     define member: [user, group#..] │
│     define manager: [user]          │
│     define can_manage: ...          │
│                                     │
│ type system_role                    │
│   relations                         │
│     define assignee: [user]         │
└─────────────────────────────────────┘

Key Characteristics:
────────────────────
✅ Stable - changes infrequently
✅ Well-tested - high test coverage required
✅ Documented - clear usage guidelines
✅ Versioned - semantic versioning for breaking changes
```

### Identity Module (Domain-Specific)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Identity Module Structure                              │
│                      File: identity/identity.fga                            │
│                      Owner: Identity Team                                   │
└─────────────────────────────────────────────────────────────────────────────┘

Purpose:
────────
Defines Identity team's domain-specific types:
• Federation profiles (SSO)
• Federation domains
• SCIM provisioning configurations
• OAuth/OIDC providers

These are used ONLY by Identity domain services.
Other domains don't directly interact with these types.

Structure:
──────────
┌─────────────────────────────────────┐
│ model                               │
│   schema 1.2                        │
│                                     │
│ type federation_profile             │
│   relations                         │
│     define organization: [org]      │
│     define admin: [user]            │
│     define can_configure:           │
│       admin or                      │
│       (org_admin from organization) │
│                                     │
│ type federation_domain              │
│   relations                         │
│     define federation_profile:      │
│       [federation_profile]          │
│     define can_manage:              │
│       can_configure from            │
│       federation_profile            │
│                                     │
│ type scim_configuration             │
│   relations                         │
│     define organization: [org]      │
│     define can_provision:           │
│       org_admin from organization   │
│                                     │
│ type oauth_provider                 │
│   relations                         │
│     define organization: [org]      │
│     define can_configure:           │
│       org_admin from organization   │
└─────────────────────────────────────┘

Key Characteristics:
────────────────────
✅ References core types (organization, user)
✅ Identity team has full control
✅ Changes don't require other team approvals
✅ Domain-specific authorization rules
```

### Print, Fleet, D365 Modules

```
Similar structure to Identity module:
• Each team owns their .fga file
• Each module references core types
• Each module is independent
• CODEOWNERS enforces boundaries
```

***

## Deployment Pipeline

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Continuous Deployment Pipeline                          │
└─────────────────────────────────────────────────────────────────────────────┘

Trigger: Git Push to Main Branch
─────────────────────────────────

Developer pushes changes to any module:
• core/core.fga
• identity/identity.fga
• print/print.fga
• fleet/fleet.fga
• d365/d365.fga

Pipeline automatically starts
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Stage 1: VALIDATION                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 1.1 Syntax Validation                                           │
│     $ fga model validate --file fga.mod                         │
│     → Checks DSL syntax                                         │
│     → Validates type references                                 │
│     → Ensures schema 1.2 compliance                             │
│                                                                 │
│ 1.2 Module Composition                                          │
│     → Combines core + identity + print + fleet + d365           │
│     → Verifies no type name conflicts                           │
│     → Validates cross-module references                         │
│                                                                 │
│ 1.3 Run All Tests                                               │
│     $ fga model test --file fga.mod --tests tests/              │
│     → Core module tests                                         │
│     → Identity module tests                                     │
│     → Print module tests                                        │
│     → Fleet module tests                                        │
│     → D365 module tests                                         │
│     → Integration tests (cross-domain scenarios)                │
│                                                                 │
│ If any test fails → Pipeline stops ❌                           │
│ If all pass → Continue to deployment ✅                         │
└─────────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Stage 2: DEPLOY TO DEVELOPMENT                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Environment: Development                                        │
│ Store ID: 01HVDEV1111111111AAAAAAAA                            │
│                                                                 │
│ 2.1 Deploy Combined Model                                       │
│     $ export FGA_STORE_ID="01HVDEV..."                         │
│     $ fga model write --store-id $FGA_STORE_ID --file fga.mod  │
│                                                                 │
│ 2.2 Capture Model ID                                            │
│     Response: {"authorization_model_id": "01HVMDL..."}          │
│     MODEL_ID=01HVMDL...                                         │
│                                                                 │
│ 2.3 Update ConfigMap                                            │
│     $ kubectl patch configmap openfga-config \                 │
│       -n platform \                                             │
│       --patch '{"data":{"OPENFGA_MODEL_ID":"01HVMDL..."}}'     │
│                                                                 │
│ 2.4 Verify Deployment                                           │
│     $ fga model get --store-id $FGA_STORE_ID                   │
│     → Confirms model deployed successfully                      │
│                                                                 │
│ Dev services automatically pick up new model                    │
└─────────────────────────────────────────────────────────────────┘
                │
                │ Automatic progression
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Stage 3: DEPLOY TO STAGING                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Requires: Manual Approval                                       │
│ Approver: Identity Team Lead or Platform Team                   │
│                                                                 │
│ Environment: Staging                                            │
│ Store ID: 01HVSTG0987654321ZYXWVUTS                            │
│                                                                 │
│ Same process as Development:                                    │
│ • Deploy model to staging store                                │
│ • Update staging ConfigMap                                      │
│ • Verify deployment                                             │
│                                                                 │
│ Staging validation:                                             │
│ • Run smoke tests                                               │
│ • Verify critical authorization scenarios                       │
│ • Performance testing                                           │
└─────────────────────────────────────────────────────────────────┘
                │
                │ After validation passes
                ▼
┌─────────────────────────────────────────────────────────────────┐
│ Stage 4: DEPLOY TO PRODUCTION                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Requires: Manual Approval                                       │
│ Approver: Identity Team Lead + Platform Team Lead               │
│                                                                 │
│ Environment: Production                                         │
│ Store ID: 01HVPRD1234567890ABCDEFGH                            │
│                                                                 │
│ 4.1 Deploy Combined Model                                       │
│     $ export FGA_STORE_ID="01HVPRD..."                         │
│     $ fga model write --store-id $FGA_STORE_ID --file fga.mod  │
│                                                                 │
│ 4.2 Capture Model ID                                            │
│     Response: {"authorization_model_id": "01HVMDL..."}          │
│     MODEL_ID=01HVMDL...                                         │
│                                                                 │
│ 4.3 Update ALL Namespace ConfigMaps                             │
│     $ kubectl patch configmap openfga-config \                 │
│       -n platform --patch '{"data":{"OPENFGA_MODEL_ID":"..."}}'│
│     $ kubectl patch configmap openfga-config \                 │
│       -n identity --patch '{"data":{"OPENFGA_MODEL_ID":"..."}}'│
│     $ kubectl patch configmap openfga-config \                 │
│       -n print --patch '{"data":{"OPENFGA_MODEL_ID":"..."}}'   │
│     $ kubectl patch configmap openfga-config \                 │
│       -n fleet --patch '{"data":{"OPENFGA_MODEL_ID":"..."}}'   │
│     $ kubectl patch configmap openfga-config \                 │
│       -n d365 --patch '{"data":{"OPENFGA_MODEL_ID":"..."}}'    │
│                                                                 │
│ 4.4 Services Rolling Update                                     │
│     → Services detect ConfigMap change                          │
│     → Gradual rollout (pod by pod)                             │
│     → Each pod picks up new model ID                           │
│     → Zero downtime deployment                                  │
│                                                                 │
│ 4.5 Create Release Tag                                          │
│     $ git tag -a model-v47 -m "Model deployment v47"          │
│     $ git push origin model-v47                                │
│                                                                 │
│ 4.6 Post-Deployment Verification                                │
│     → Monitor authorization success rate                        │
│     → Check error logs                                          │
│     → Verify cross-domain queries working                       │
└─────────────────────────────────────────────────────────────────┘
                │
                │ Deployment complete
                ▼
┌─────────────────────────────────────────────────────────────────┐
│                   ✅ DEPLOYMENT SUCCESS                         │
│                                                                 │
│ Model Version: 47                                               │
│ Model ID: 01HVMDL...                                            │
│ Deployed to Store: 01HVPRD1234567890ABCDEFGH                   │
│                                                                 │
│ All services now using new model:                              │
│ • Identity Service ✅                                           │
│ • Print Service ✅                                              │
│ • Fleet Service ✅                                              │
│ • D365 Service ✅                                               │
│                                                                 │
│ Total Time: ~15-20 minutes                                      │
│ Total Deployments: 1 (to single store)                         │
│ Total ConfigMaps Updated: 5 (all namespaces)                   │
└─────────────────────────────────────────────────────────────────┘
```

### What Triggers Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│              Changes That Trigger Pipeline                      │
└─────────────────────────────────────────────────────────────────┘

✅ Changes to any .fga file:
   • core/core.fga
   • identity/identity.fga
   • print/print.fga
   • fleet/fleet.fga
   • d365/d365.fga

✅ Changes to fga.mod (manifest)

✅ Changes to test files:
   • tests/*.yaml

❌ Changes that DON'T trigger:
   • README.md files
   • Documentation
   • Comments only
```

***

## Service Integration

### How Services Use the ConfigMap

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Service Integration Pattern                            │
└─────────────────────────────────────────────────────────────────────────────┘

Step 1: Kubernetes Deployment Configuration
────────────────────────────────────────────

apiVersion: apps/v1
kind: Deployment
metadata:
  name: print-service
  namespace: print
spec:
  template:
    spec:
      containers:
      - name: print-service
        image: your-registry/print-service:latest
        env:
          # OpenFGA configuration from ConfigMap
          - name: OPENFGA_API_URL
            valueFrom:
              configMapKeyRef:
                name: openfga-config
                key: OPENFGA_API_URL
          
          - name: OPENFGA_STORE_ID
            valueFrom:
              configMapKeyRef:
                name: openfga-config
                key: OPENFGA_STORE_ID
          
          - name: OPENFGA_MODEL_ID
            valueFrom:
              configMapKeyRef:
                name: openfga-config
                key: OPENFGA_MODEL_ID


Step 2: Application Configuration (C#/.NET Example)
────────────────────────────────────────────────────

// Configuration class
public class OpenFGAConfiguration
{
    public string ApiUrl { get; set; }
    public string StoreId { get; set; }
    public string AuthorizationModelId { get; set; }
}

// Startup configuration
public void ConfigureServices(IServiceCollection services)
{
    // Bind configuration from environment variables
    services.Configure<OpenFGAConfiguration>(options =>
    {
        options.ApiUrl = Environment.GetEnvironmentVariable("OPENFGA_API_URL");
        options.StoreId = Environment.GetEnvironmentVariable("OPENFGA_STORE_ID");
        options.AuthorizationModelId = Environment.GetEnvironmentVariable("OPENFGA_MODEL_ID");
    });
    
    // Register OpenFGA client
    services.AddSingleton<IOpenFgaClient>(sp =>
    {
        var config = sp.GetRequiredService<IOptions<OpenFGAConfiguration>>().Value;
        
        return new OpenFgaClient(new ClientConfiguration
        {
            ApiUrl = config.ApiUrl,
            StoreId = config.StoreId,
            AuthorizationModelId = config.AuthorizationModelId
        });
    });
}


Step 3: Service Usage
─────────────────────

public class PrinterAuthorizationService
{
    private readonly IOpenFgaClient _fgaClient;
    
    public PrinterAuthorizationService(IOpenFgaClient fgaClient)
    {
        _fgaClient = fgaClient;
    }
    
    public async Task<bool> CanUserAccessPrinter(string userId, string printerId)
    {
        var response = await _fgaClient.Check(new CheckRequest
        {
            User = $"user:{userId}",
            Relation = "can_access",
            Object = $"printer:{printerId}"
        });
        
        return response.Allowed;
    }
}


Step 4: ConfigMap Update Detection
───────────────────────────────────

When ConfigMap is updated (new MODEL_ID):

1. Kubernetes detects ConfigMap change
2. Environment variables updated in pods
3. Application detects change (if configured)
4. OR: Rolling restart of pods (automatic)
5. New pods use new MODEL_ID
6. Old pods gradually replaced
7. Zero downtime deployment


Result:
───────
✅ Services automatically use correct Store ID
✅ Services pick up new Model ID when deployed
✅ No hardcoded values in application code
✅ Same code runs in dev, staging, production
✅ Easy to update via ConfigMap
```

### Cross-Domain Authorization Example

```
┌─────────────────────────────────────────────────────────────────────────────┐
│             Real-World Authorization Flow                                   │
│             (Shows benefit of single store)                                 │
└─────────────────────────────────────────────────────────────────────────────┘

Scenario: User tries to access a printer
──────────────────────────────────────────

User: Bob (user:bob)
Printer: pr-001 (printer:pr-001)
Organization: Acme Corp (organization:acme)

Bob's role: org_admin of Acme


Flow:
─────

┌──────────────────┐
│ Print Service    │
│                  │
│ Request:         │
│ GET /printers/   │
│     pr-001       │
│                  │
│ User: bob        │
└────────┬─────────┘
         │
         │ 1. Check authorization
         ▼
┌───────────────────────────────────────────┐
│ Print Service Authorization Layer         │
│                                           │
│ Check: Can user:bob access printer:pr-001│
│                                           │
│ await _fgaClient.Check(                   │
│   user: "user:bob",                       │
│   relation: "can_access",                 │
│   object: "printer:pr-001"                │
│ )                                         │
└────────┬──────────────────────────────────┘
         │
         │ 2. Query OpenFGA (SAME STORE as identity data)
         ▼
┌─────────────────────────────────────────────────────────────────┐
│ OpenFGA Store: 01HVPRD1234567890ABCDEFGH                        │
│                                                                 │
│ Authorization Model contains:                                   │
│ • Core module (user, organization)                             │
│ • Print module (printer, can_access relation)                  │
│                                                                 │
│ Evaluation:                                                     │
│ ┌────────────────────────────────────────────────────────────┐ │
│ │ 1. Find printer:pr-001 definition                          │ │
│ │    → Type from print module                                │ │
│ │    → can_access = org_admin from organization              │ │
│ │                                                            │ │
│ │ 2. Find printer:pr-001's organization                      │ │
│ │    → Tuple: (organization:acme, organization, printer...)  │ │
│ │    → Printer belongs to organization:acme                  │ │
│ │                                                            │ │
│ │ 3. Check if bob is org_admin of organization:acme          │ │
│ │    → Look in SAME store for core tuples                    │ │
│ │    → Tuple: (user:bob, org_admin, organization:acme)       │ │
│ │    → YES, found in core domain data                        │ │
│ │                                                            │ │
│ │ 4. Result: ALLOWED ✅                                       │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ All data in SAME store = instant, accurate result               │
└─────────────────────────────────────────────────────────────────┘
         │
         │ 3. Response
         ▼
┌───────────────────────────────────────────┐
│ Print Service                             │
│                                           │
│ Result: {"allowed": true}                 │
│                                           │
│ → Return printer data to user             │
└───────────────────────────────────────────┘


Key Benefits:
─────────────
✅ Single query to OpenFGA
✅ No data synchronization needed
✅ Always accurate (bob's role is current)
✅ Natural cross-domain authorization
✅ Fast response time
```

***

## Disaster Recovery and Store Retrieval

### Scenario 1: Lost Store ID

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Recovery: Lost Store ID                                  │
└─────────────────────────────────────────────────────────────────────────────┘

Problem: Store ID lost, ConfigMap corrupted, or documentation missing
──────────────────────────────────────────────────────────────────────

Solution 1: List All Stores
────────────────────────────

$ fga store list

Response:
{
  "stores": [
    {
      "id": "01HVDEV1111111111AAAAAAAA",
      "name": "Company Authorization - Development",
      "created_at": "2025-11-14T10:00:00Z",
      "updated_at": "2025-11-16T08:00:00Z"
    },
    {
      "id": "01HVSTG0987654321ZYXWVUTS",
      "name": "Company Authorization - Staging",
      "created_at": "2025-11-15T10:00:00Z",
      "updated_at": "2025-11-16T08:00:00Z"
    },
    {
      "id": "01HVPRD1234567890ABCDEFGH",
      "name": "Company Authorization - Production",
      "created_at": "2025-11-16T09:00:00Z",
      "updated_at": "2025-11-16T08:30:00Z"
    }
  ]
}

Identify by name → Production store ID: 01HVPRD1234567890ABCDEFGH


Solution 2: Check Git Repository
─────────────────────────────────

$ cat docs/openfga-stores.md

# OpenFGA Store IDs

## Production
- Store ID: `01HVPRD1234567890ABCDEFGH`
- Created: 2025-11-16
...


Solution 3: Check Existing ConfigMap
─────────────────────────────────────

$ kubectl get configmap openfga-config -n platform -o yaml

apiVersion: v1
kind: ConfigMap
data:
  OPENFGA_STORE_ID: "01HVPRD1234567890ABCDEFGH"
...


Solution 4: Check Running Service
──────────────────────────────────

$ kubectl exec -it print-service-pod-xyz -n print -- env | grep OPENFGA

OPENFGA_STORE_ID=01HVPRD1234567890ABCDEFGH
...


Recovery Steps:
───────────────
1. List stores and identify by name
2. Update documentation (docs/openfga-stores.md)
3. Update ConfigMaps if needed
4. Verify services are using correct Store ID
5. Document incident for future reference
```

### Scenario 2: Corrupted Authorization Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Recovery: Corrupted Model                                │
└─────────────────────────────────────────────────────────────────────────────┘

Problem: Bad model deployed, authorization not working correctly
────────────────────────────────────────────────────────────────

Symptoms:
• Authorization checks returning incorrect results
• Services reporting OpenFGA errors
• Cross-domain queries failing

Diagnosis:
──────────

Step 1: Check Current Model
$ fga model get --store-id 01HVPRD1234567890ABCDEFGH

Step 2: Review Recent Deployments
$ git log --oneline -10

Step 3: Check Pipeline History
→ Review GitHub Actions/Azure DevOps runs
→ Identify when issue started


Recovery Option 1: Rollback to Previous Model
──────────────────────────────────────────────

List Model Versions:
$ fga model list --store-id 01HVPRD1234567890ABCDEFGH

Response:
{
  "authorization_models": [
    {
      "id": "01HVMDL48NEW",  ← Current (bad)
      "created_at": "2025-11-16T09:30:00Z"
    },
    {
      "id": "01HVMDL47PREV",  ← Previous (good)
      "created_at": "2025-11-16T08:00:00Z"
    },
    {
      "id": "01HVMDL46OLD",
      "created_at": "2025-11-15T15:00:00Z"
    }
  ]
}

Rollback Process:
─────────────────

1. Update ConfigMap to previous model ID:
   $ kubectl patch configmap openfga-config \
     -n platform \
     --patch '{"data":{"OPENFGA_MODEL_ID":"01HVMDL47PREV"}}'

2. Update all namespace ConfigMaps:
   $ kubectl patch configmap openfga-config -n identity --patch '...'
   $ kubectl patch configmap openfga-config -n print --patch '...'
   $ kubectl patch configmap openfga-config -n fleet --patch '...'
   $ kubectl patch configmap openfga-config -n d365 --patch '...'

3. Rolling restart services (if needed):
   $ kubectl rollout restart deployment print-service -n print
   $ kubectl rollout restart deployment fleet-service -n fleet
   (etc.)

4. Verify authorization working:
   → Test critical authorization scenarios
   → Monitor error logs
   → Check success rate

Result: Services now using previous (good) model
Time to recover: ~5-10 minutes


Recovery Option 2: Deploy Fixed Model
──────────────────────────────────────

If rollback not desired, fix and redeploy:

1. Identify issue in model file
2. Create fix in Git
3. Create PR with fix
4. CI/CD validates and tests
5. Deploy fixed model
6. Update ConfigMaps
7. Services pick up new model

Time to recover: ~15-20 minutes (includes testing)


Recovery Option 3: Emergency Fix
─────────────────────────────────

For critical issues requiring immediate fix:

1. Create hotfix branch
2. Apply minimal fix to model
3. Deploy directly to production (bypass normal approvals)
4. Update ConfigMaps
5. Verify fix
6. Backport to main branch
7. Post-incident review

Time to recover: ~10 minutes
```

### Scenario 3: Complete Store Corruption

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Recovery: Complete Store Loss                            │
└─────────────────────────────────────────────────────────────────────────────┘

Problem: Store deleted or database corruption, complete data loss
──────────────────────────────────────────────────────────────────

This is the worst-case scenario but recoverable.


Recovery Process:
─────────────────

Step 1: Create New Store
────────────────────────

$ fga store create --name "Company Authorization - Production (Restored)"

Response:
{
  "id": "01HVNEW9999999999NEWSTORE",
  "name": "Company Authorization - Production (Restored)",
  ...
}

NEW Store ID: 01HVNEW9999999999NEWSTORE


Step 2: Deploy Latest Model
────────────────────────────

$ git checkout main
$ fga model write --store-id 01HVNEW9999999999NEWSTORE --file fga.mod

Response:
{
  "authorization_model_id": "01HVMDLNEW..."
}

Model deployed to new store.


Step 3: Restore Relationship Tuples
────────────────────────────────────

Option A: From Database Backup
If you have tuple backup:

$ fga tuple write --store-id 01HVNEW9999999999NEWSTORE --file backup-tuples.json

Option B: Sync from Source Systems
If no backup, rebuild from source:

For each domain:
1. Query domain database for relationships
2. Generate tuples
3. Bulk import to OpenFGA

Example:
────────
# Identity domain - sync users and org memberships
$ ./scripts/sync-identity-tuples.sh --store-id 01HVNEW9999999999NEWSTORE

# Print domain - sync printer ownership
$ ./scripts/sync-print-tuples.sh --store-id 01HVNEW9999999999NEWSTORE

# Fleet domain - sync IoT devices
$ ./scripts/sync-fleet-tuples.sh --store-id 01HVNEW9999999999NEWSTORE

# D365 domain - sync contracts
$ ./scripts/sync-d365-tuples.sh --store-id 01HVNEW9999999999NEWSTORE


Step 4: Update All ConfigMaps
──────────────────────────────

$ kubectl patch configmap openfga-config \
  -n platform \
  --patch '{"data":{
    "OPENFGA_STORE_ID":"01HVNEW9999999999NEWSTORE",
    "OPENFGA_MODEL_ID":"01HVMDLNEW..."
  }}'

Repeat for all namespaces:
- identity
- print
- fleet
- d365


Step 5: Rolling Restart All Services
─────────────────────────────────────

$ kubectl rollout restart deployment -n identity
$ kubectl rollout restart deployment -n print
$ kubectl rollout restart deployment -n fleet
$ kubectl rollout restart deployment -n d365


Step 6: Verification
────────────────────

1. Test critical authorization scenarios
2. Verify cross-domain authorization
3. Check error logs
4. Monitor authorization success rate
5. Compare with expected behavior


Step 7: Update Documentation
─────────────────────────────

$ vi docs/openfga-stores.md

Update production Store ID to:
01HVNEW9999999999NEWSTORE

Commit to repository.


Post-Recovery:
──────────────
• Document incident
• Review backup procedures
• Implement tuple backup automation
• Add monitoring alerts
• Post-mortem meeting


Time to fully recover: 2-4 hours
(depending on tuple volume and sync method)
```

### Backup and Recovery Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Backup Strategy                                          │
└─────────────────────────────────────────────────────────────────────────────┘

1. Store ID Documentation
──────────────────────────
   ✅ Store in Git: docs/openfga-stores.md
   ✅ Store in Wiki: Confluence/SharePoint
   ✅ Store in Key Vault: Azure Key Vault
   ✅ Multiple copies, multiple locations


2. Authorization Model Backups
───────────────────────────────
   ✅ Git is source of truth
   ✅ All model versions in Git history
   ✅ Tagged releases for each deployment
   ✅ Can recreate from Git at any time


3. Relationship Tuple Backups
──────────────────────────────
   Option A: OpenFGA Native Backup
   $ fga store export --store-id <ID> > backup-$(date +%Y%m%d).yaml
   
   Option B: Database-Level Backup
   → Backup PostgreSQL/MySQL database behind OpenFGA
   → Daily automated backups
   → Point-in-time recovery
   
   Option C: Application-Level Sync
   → Each domain service maintains its own data
   → Can regenerate tuples from domain databases
   → Sync scripts available


4. Automated Backup Schedule
─────────────────────────────
   • Daily: Full tuple export
   • Weekly: Database backup
   • Monthly: Disaster recovery test
   • On-demand: Before major changes


5. Recovery Time Objectives
───────────────────────────
   • Lost Store ID: < 5 minutes
   • Corrupted Model: < 10 minutes
   • Complete Store Loss: < 4 hours


6. Testing Recovery Procedures
───────────────────────────────
   • Quarterly disaster recovery drills
   • Test in staging environment
   • Document lessons learned
   • Update procedures
```

***

## Operations and Maintenance

### Monitoring

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Monitoring Strategy                                  │
└─────────────────────────────────────────────────────────────────────────────┘

Key Metrics to Monitor:
───────────────────────

1. OpenFGA Server Health
   • API response time
   • Request rate
   • Error rate
   • Resource utilization (CPU, memory)

2. Store Metrics
   • Current model version
   • Tuple count (growth over time)
   • Query latency (p50, p95, p99)
   • Authorization success rate

3. Service Integration Health
   • Services using correct Store ID
   • Services using correct Model ID
   • Authorization check failures
   • Timeout rate

4. Pipeline Health
   • Model deployment success rate
   • Test pass rate
   • Deployment duration
   • ConfigMap update success


Alerting Thresholds:
────────────────────

⚠️  Warning:
   • Query latency > 100ms (p95)
   • Error rate > 1%
   • Model deployment > 30 minutes

🚨 Critical:
   • Query latency > 500ms (p95)
   • Error rate > 5%
   • Authorization completely failing
   • OpenFGA server down


Dashboards:
───────────

Primary Dashboard:
• Store health overview
• Active model version
• Authorization success rate
• Recent deployments

Per-Domain Dashboards:
• Identity domain queries
• Print domain queries
• Fleet domain queries
• D365 domain queries
```

### Regular Maintenance Tasks

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Maintenance Schedule                                 │
└─────────────────────────────────────────────────────────────────────────────┘

Daily:
──────
• Monitor authorization success rate
• Check error logs
• Verify ConfigMap consistency


Weekly:
───────
• Review query performance metrics
• Check tuple growth
• Review model deployment history
• Clean up old development stores


Monthly:
────────
• Review and archive old model versions
• Audit Store ID documentation
• Test disaster recovery procedures
• Review and update CODEOWNERS
• Security audit of authorization rules


Quarterly:
──────────
• Performance testing
• Capacity planning
• Team training on OpenFGA
• Review and update documentation
• Disaster recovery drill
```

***

## Summary

### Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Single Store Architecture Summary                        │
└─────────────────────────────────────────────────────────────────────────────┘

ONE-TIME SETUP (Identity Team):
────────────────────────────────
1. Create store once per environment
2. Publish Store ID via ConfigMap
3. Notify all domain teams
4. Document in Git repository

ONGOING DEVELOPMENT (All Teams):
─────────────────────────────────
1. Each team maintains their .fga module file
2. Teams develop independently
3. CODEOWNERS enforces boundaries
4. Tests run automatically

DEPLOYMENT (Automated):
───────────────────────
1. Pipeline combines all modules
2. Deploys to SINGLE store
3. Updates ConfigMap with new Model ID
4. Services pick up changes automatically

RECOVERY (If Needed):
─────────────────────
1. Store ID can be retrieved via list command
2. Models can be rolled back instantly
3. Complete recovery possible from backups
4. Well-documented procedures


BENEFITS ACHIEVED:
──────────────────
✅ No data duplication
✅ Natural cross-domain authorization
✅ Simple deployment (one store)
✅ Fast iterations (minutes not hours)
✅ Team autonomy maintained
✅ Lower costs (single infrastructure)
✅ Easy recovery (multiple backup methods)
✅ OpenFGA best practice compliance


CRITICAL POINTS:
────────────────
⚠️  Store ID NEVER changes after creation
⚠️  Document Store ID in multiple locations
⚠️  ConfigMap is single source of configuration
⚠️  Test recovery procedures regularly
⚠️  Monitor authorization success rate
```

### Decision Checklist

```
✅ Use Single Store Architecture When:
   □ Domains are interconnected
   □ Cross-domain authorization required
   □ Data consistency is critical
   □ Team autonomy via code (Git/CODEOWNERS)
   □ Want operational simplicity
   □ Cost efficiency important
   
✅ This Approach Gives You:
   □ One store per environment
   □ One deployment updates all
   □ Natural cross-domain queries
   □ Team-owned module files
   □ Simple ConfigMap distribution
   □ Fast disaster recovery
   □ OpenFGA recommended pattern
```

***

## Next Steps

1. **Review this architecture** with your team
2. **Create stores** for dev, staging, production
3. **Set up Git repository** with module structure
4. **Configure CI/CD pipeline** for automated deployment
5. **Deploy ConfigMaps** to all namespaces
6. **Update services** to use ConfigMap
7. **Test disaster recovery** procedures
8. **Monitor and iterate**

***

**Questions or concerns? Discuss with Identity Team Lead or Platform Team.**

**Documentation:** See `docs/` folder in authorization-models repository

**Support:** #openfga-support Slack channel

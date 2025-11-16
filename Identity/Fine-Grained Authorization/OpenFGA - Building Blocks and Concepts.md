# OpenFGA Complete Tutorial: Building Blocks and Concepts

## Introduction

This tutorial will teach you all the fundamental building blocks of OpenFGA using your organization's actual business context: identity management with organizations, users, system roles, and domain resources like printers and IoT devices. We'll build understanding step by step.

***

## Part 1: The Foundation - What is OpenFGA?

### OpenFGA in Simple Terms

**OpenFGA** is a specialized authorization engine that answers the question: **"Can user X perform action Y on resource Z?"**

### Traditional Authorization vs OpenFGA

```
Traditional Approach (Your Current System):
───────────────────────────────────────────

if (user.hasRole("ORG_ADMIN") && 
    user.organizationId == resource.organizationId) {
    return ALLOWED;
}

// You write this logic in EVERY service
// Hard to maintain, hard to change, duplicated everywhere
```

```
OpenFGA Approach:
─────────────────

You define authorization logic ONCE in OpenFGA
Services just ask: "Can Bob access printer pr-001?"
OpenFGA returns: {"allowed": true}

// Logic centralized, consistent, auditable
```

***

## Part 2: Building Block #1 - Store

### What is a Store?

A **store** is the top-level container that holds all your authorization data.

### Mental Model: Store as an Authorization Database

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenFGA Store                                │
│                    "Company Authorization"                      │
│                    ID: 01HVMMB1234567890ABCDEFGH                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Think of this as a complete authorization database for         │
│  your entire company                                            │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  Authorization Model (The Schema/Rules)                   │ │
│  │  - Defines what types exist (user, org, printer)          │ │
│  │  - Defines what relationships are possible                │ │
│  │  - Defines authorization rules                            │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  Relationship Tuples (The Data)                           │ │
│  │  - Bob is org_admin of Acme                               │ │
│  │  - Alice is member of Acme                                │ │
│  │  - Printer pr-001 belongs to Acme                         │ │
│  │  - ...millions of relationship records...                 │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Store Characteristics

| Characteristic | Description | Example |
|----------------|-------------|---------|
| **Unique ID** | Every store has a globally unique identifier | `01HVMMB1234567890ABCDEFGH` |
| **Name** | Human-readable name | "Company Authorization" |
| **Isolated** | Data in one store cannot access another store | N/A for single store setup |
| **Contains Models** | Multiple versions of authorization models | Version 1, 2, 3... 47 (active) |
| **Contains Tuples** | Millions of relationship records | All your org/user/resource relationships |

### Creating Your Store (One-Time Setup)

```bash
# Using FGA CLI
fga store create --name "Company Authorization"

# Response:
{
  "id": "01HVMMB1234567890ABCDEFGH",
  "name": "Company Authorization",
  "created_at": "2025-11-15T20:00:00Z",
  "updated_at": "2025-11-15T20:00:00Z"
}
```

**Important:** Save this Store ID - you'll use it for all subsequent operations.

### Store Analogy

```
Store = Your entire authorization system
      = Like a PostgreSQL database instance
      = Contains schema (model) + data (tuples)
```

***

## Part 3: Building Block #2 - Authorization Model

### What is an Authorization Model?

An **authorization model** defines the structure and rules for authorization - it's like a database schema.

### The Schema Analogy

```sql
Traditional Database              Authorization Model
───────────────────────           ───────────────────

CREATE TABLE users (              type user
  id VARCHAR(50),                 
  name VARCHAR(100)               
);                                

CREATE TABLE orgs (               type organization
  id VARCHAR(50),                   relations
  name VARCHAR(100)                   define member: [user]
);                                    define org_admin: [user]

CREATE TABLE memberships (        
  user_id VARCHAR(50),            
  org_id VARCHAR(50),             
  role VARCHAR(50)                
);                                
```

**Key Difference:** The authorization model defines **relationships and rules**, not just data structure.

### Authorization Model Structure

```
┌─────────────────────────────────────────────────────────────────┐
│              Authorization Model (Version 47)                   │
│              Schema Version: 1.1                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Contains:                                                      │
│    1. Type Definitions (what objects exist)                     │
│    2. Relation Definitions (what relationships are possible)    │
│    3. Authorization Rules (how relationships grant permissions) │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Understanding Schema Version vs Model Version

**This is an important distinction that often confuses people:**

#### Schema Version (e.g., `1.1`)

**What it is:** The version of the OpenFGA modeling language/syntax itself - NOT your specific model.

**Analogy:** Like saying "I'm using Python 3.10" or "SQL-92 standard"

**Who controls it:** OpenFGA project (not you)

**When it changes:** When OpenFGA introduces new features or syntax to the modeling language

**Schema version history:**
- `1.0` - Original OpenFGA DSL
- `1.1` - Added support for conditions, improved syntax, better type safety

**Your action:** You choose which schema version to use when writing your model (usually the latest: `1.1`)

#### Model Version (e.g., v1, v2, v47)

**What it is:** A unique instance of YOUR authorization model deployed to OpenFGA

**Analogy:** Like Git commits - each deployment creates a new immutable snapshot

**Who controls it:** OpenFGA automatically assigns a new version on each deployment

**When it changes:** ONLY when you deploy a new authorization model (change types/relations/rules)

**Does NOT change when:** You modify tuples (relationship data)

#### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCHEMA VERSION                               │
│                    (Language Version)                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Schema 1.0 (Released 2022)                                     │
│  ├── Basic type definitions                                     │
│  ├── Simple relations                                           │
│  └── Basic operators (or, and)                                  │
│                                                                 │
│  Schema 1.1 (Released 2023)                                     │
│  ├── Everything from 1.0                                        │
│  ├── Conditional tuples                                         │
│  ├── Enhanced operators (but not)                               │
│  └── Better type checking                                       │
│                                                                 │
│  You choose: schema 1.1 (recommended)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    MODEL VERSION                                │
│                    (Your Schema Snapshots)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Model v1 (Nov 1, 2025)                                         │
│  ID: 01HVM01ABC...                                              │
│  Schema: 1.1                                                    │
│  Changes: Initial model with user, organization                 │
│                                                                 │
│  Model v2 (Nov 8, 2025)                                         │
│  ID: 01HVM02DEF...                                              │
│  Schema: 1.1 (same language version)                            │
│  Changes: Added 'group' type                                    │
│                                                                 │
│  Model v3 (Nov 15, 2025)                                        │
│  ID: 01HVM03GHI...                                              │
│  Schema: 1.1 (still same language version)                      │
│  Changes: Added 'printer' type                                  │
│                                                                 │
│  OpenFGA assigns: New model ID on each deployment               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### When Model Version Changes

**✅ Model Version DOES Change When:**

```
1. You add a new type
   ──────────────────
   Before: type user, type organization
   After:  type user, type organization, type printer
   Result: NEW Model Version (v2)

2. You modify a type definition
   ─────────────────────────────
   Before: 
     type organization
       relations
         define member: [user]
   
   After:
     type organization
       relations
         define member: [user]
         define admin: [user]  ← Added new relation
   
   Result: NEW Model Version (v3)

3. You change authorization rules
   ───────────────────────────────
   Before:
     define can_view: admin
   
   After:
     define can_view: admin or member  ← Changed rule
   
   Result: NEW Model Version (v4)
```

**❌ Model Version DOES NOT Change When:**

```
1. You add/remove tuples
   ─────────────────────
   Action: Write tuple (user:bob, member, organization:acme)
   Result: NO new model version (just data change)

2. You modify relationship data
   ────────────────────────────
   Action: Delete tuple (user:alice, member, organization:acme)
   Result: NO new model version (just data change)

3. You perform check queries
   ─────────────────────────
   Action: Check if bob can access printer
   Result: NO new model version (read operation)
```

### Your First Model - With All Three Relation Types

```openfga
model
  schema 1.1    # ← This is the SCHEMA VERSION (language version)

# Type 1: User (represents people)
type user

# Type 2: Organization (represents customer orgs)
type organization
  relations
    # 1. DIRECT RELATIONSHIPS (stored as tuples)
    define member: [user]
    define org_admin: [user]
    define parent: [organization]
    
    # 2. COMPUTED RELATIONSHIPS (calculated using operators)
    define can_view_members: org_admin or member
    define can_manage_org: org_admin

# Type 3: Printer (represents printers)
type printer
  relations
    # 1. DIRECT RELATIONSHIP
    define organization: [organization]
    
    # 3. RELATIONSHIP TRAVERSAL (follows object relationships)
    define can_access: org_admin from organization
    define can_manage: org_admin from organization
    
    # 2. COMPUTED RELATIONSHIP (combines multiple checks)
    define can_view_status: can_access or can_manage
```

**What this model says:**
1. We have `user`, `organization`, and `printer` objects
2. Users can be `member` or `org_admin` of organizations (Direct)
3. Anyone who is `org_admin` OR `member` can view other members (Computed)
4. To access a printer, you must be `org_admin` of the printer's organization (Traversal)
5. Anyone who can access or manage can view status (Computed)

### Model Versioning - Immutability

**Critical Concept:** Authorization models are **immutable** (cannot be changed).

```
Timeline:
─────────

Nov 1:  Deploy Model v1 (user, organization)
        Model ID: 01HVM01...  ← MODEL VERSION (unique ID)
        Schema: 1.1            ← SCHEMA VERSION (language)
        Status: ACTIVE

Nov 8:  Deploy Model v2 (added group type)
        Model ID: 01HVM02...  ← NEW MODEL VERSION
        Schema: 1.1            ← Same SCHEMA VERSION
        Status: ACTIVE ← now active
        (v1 still exists for audit/rollback)

Nov 15: Deploy Model v47 (added printer type)
        Model ID: 01HVM47...  ← NEW MODEL VERSION
        Schema: 1.1            ← Still same SCHEMA VERSION
        Status: ACTIVE ← now active
        (v1, v2...v46 still exist)
```

**Why Immutability?**
- Audit trail of authorization logic changes
- Safe rollback if something breaks
- Multiple model versions for testing

***

## Part 4: Building Block #3 - Type

### What is a Type?

A **type** is a category of objects in your system - like a table in a database.

### Types in Your Organization

```
┌────────────────────────────────────────────────────────────────┐
│                  Your Organization's Types                     │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Identity Domain (owned by Identity team):                     │
│    ├── user             (Bob, Alice, Charlie)                  │
│    ├── organization     (Acme Corp, child orgs)                │
│    ├── group            (Engineering, Sales)                   │
│    └── system_role      (ORG_ADMIN, USER_ADMIN)               │
│                                                                │
│  Print Domain (owned by Print team):                           │
│    ├── printer          (pr-001, pr-042)                       │
│    ├── print_job        (job-12345, job-67890)                 │
│    └── print_entitlement (print_release feature)              │
│                                                                │
│  Fleet Domain (owned by Fleet team):                           │
│    ├── managed_printer  (fleet-pr-100, fleet-pr-200)          │
│    ├── iot_device       (sensor-42, scanner-17)               │
│    ├── device_group     (building-a-printers, floor-2-devices)│
│    └── maintenance_schedule (schedule-weekly, schedule-monthly)│
│                                                                │
│  D365 Domain (owned by D365 team):                             │
│    ├── contract         (contract-2025-001)                    │
│    └── opportunity      (opp-2025-042)                         │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Type Definition Syntax

```openfga
# Simple type (no relations)
type user

# Type with relations
type organization
  relations
    define member: [user]
    define org_admin: [user]
    define parent: [organization]

# Type that references other types
type managed_printer
  relations
    define organization: [organization]
    define can_access: [user]
    define can_manage_fleet: [user]
```

### Type Instances vs Type Definitions

```
Type Definition (in model):     Type Instances (in tuples):
───────────────────────────     ───────────────────────────

type user                       user:bob
                                user:alice
                                user:charlie

type organization               organization:acme
                                organization:child-org-1
                                organization:root-org

type managed_printer            managed_printer:fleet-pr-100
                                managed_printer:fleet-pr-200

type iot_device                 iot_device:sensor-42
                                iot_device:scanner-17
```

**Key Point:** The type is the **definition**. Actual instances (like `user:bob`) only exist in relationship tuples.

---

## Part 5: Building Block #4 - Relation

### What is a Relation?

A **relation** is a named connection between objects.

### The Three Types of Relations - Detailed Explanation

This is one of the most important concepts in OpenFGA. There are three distinct ways to define relationships:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Types of Relations                           │
└─────────────────────────────────────────────────────────────────┘

1. DIRECT RELATIONS (stored as tuples)
   ──────────────────────────────────
   Syntax: define relation: [type]
   
   type organization
     relations
       define member: [user]
   
   • Explicitly assigned via tuples
   • Stored in database
   • Example: Bob is member of Acme
   • You WRITE these as tuples

2. COMPUTED RELATIONS (calculated from rules)
   ───────────────────────────────────────────
   Syntax: define relation: relation1 or relation2
   
   type organization
     relations
       define member: [user]
       define admin: [user]
       define can_view: admin or member
   
   • NOT stored as tuples
   • Calculated based on other relations
   • Example: If Bob is admin → Bob can_view (computed)
   • You DON'T write these as tuples

3. RELATIONSHIP TRAVERSAL (follow connections)
   ────────────────────────────────────────────
   Syntax: define relation: relation_name from object_relation
   
   type printer
     relations
       define organization: [organization]
       define can_access: org_admin from organization
   
   • Follows relationship graph
   • Walks from one object to another
   • Example: printer → organization → check if user is org_admin there
   • You write the connecting tuples, OpenFGA traverses them
```

### Detailed Examples for Each Relation Type

#### Type 1: Direct Relation

**Definition:**
```openfga
type document
  relations
    define owner: [user]
    define editor: [user]
```

**Usage:**
```
Tuples you write:
─────────────────
(user:bob, owner, document:report-2025)
(user:alice, editor, document:report-2025)

Query: Is bob owner of document:report-2025?
Answer: Check if tuple exists → YES

How it works: Direct lookup in tuple database
```

#### Type 2: Computed Relation

**Definition:**
```openfga
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    # Computed: anyone who can edit can also view
    define can_view: owner or editor or viewer
    define can_edit: owner or editor
```

**Usage:**
```
Tuple you write:
────────────────
(user:bob, owner, document:report-2025)

Query: Can bob view document:report-2025?

OpenFGA Logic:
1. Check can_view relation
2. can_view = owner or editor or viewer
3. Is bob owner? YES (tuple exists)
4. Result: YES (computed, no tuple needed for can_view)
```

**Important:** You DON'T write a tuple for `can_view`. OpenFGA computes it automatically.

**Common Operators for Computed Relations:**
- `or` - User has ANY of the relations
- `and` - User has ALL of the relations
- `but not` - User has first relation but NOT the second

#### Type 3: Relationship Traversal

**Definition:**
```openfga
type organization
  relations
    define org_admin: [user]

type printer
  relations
    define organization: [organization]
    define can_access: org_admin from organization
    #                  └──────────────────────────┘
    #                  Traverse to organization,
    #                  then check if user is org_admin there
```

**Usage:**
```
Tuples you write:
─────────────────
(user:bob, org_admin, organization:acme)
(organization:acme, organization, printer:pr-001)

Query: Can bob access printer:pr-001?

OpenFGA walks the graph:
1. Find printer:pr-001's organization
   → Tuple: (organization:acme, organization, printer:pr-001)
   → Printer belongs to organization:acme

2. Check if bob is org_admin of organization:acme
   → Tuple: (user:bob, org_admin, organization:acme)
   → YES, tuple exists

3. Result: YES (bob can access printer:pr-001)
```

**Key Insight:** The `from` keyword tells OpenFGA to traverse from the printer to its organization, then check the relation there.

### Complete Example Showing All Three Types

```openfga
model
  schema 1.1

type user

type organization
  relations
    # DIRECT: Stored as tuples
    define member: [user]
    define org_admin: [user]
    define user_admin: [user]
    
    # COMPUTED: Calculated from rules
    define can_manage_users: org_admin or user_admin
    define can_view_org: org_admin or member

type printer
  relations
    # DIRECT: Stored as tuples
    define organization: [organization]
    define assigned_user: [user]
    
    # TRAVERSAL: Follow to org and check relation there
    define org_level_access: org_admin from organization
    
    # COMPUTED: Combine direct and traversal
    define can_access: assigned_user or org_level_access
```

**Real scenario:**
```
Tuples:
- (user:bob, org_admin, organization:acme)
- (organization:acme, organization, printer:pr-001)
- (user:alice, assigned_user, printer:pr-002)

Query 1: Can bob access printer:pr-001?
→ can_access = assigned_user or org_level_access
→ Is bob assigned_user? NO
→ Check org_level_access (traversal)
  → org_level_access = org_admin from organization
  → Find printer's org: acme
  → Is bob org_admin of acme? YES
→ Result: YES ✅

Query 2: Can alice access printer:pr-002?
→ can_access = assigned_user or org_level_access
→ Is alice assigned_user? YES (direct tuple)
→ Result: YES ✅ (short-circuits, doesn't need to check traversal)
```

### Quick Reference: How to Identify Each Type

```
Look at the syntax:

define member: [user]                    → DIRECT
define can_view: owner or editor         → COMPUTED  
define can_access: admin from org        → TRAVERSAL

define manager: [user, group#member]     → DIRECT (with user set)
define can_edit: owner and not archived  → COMPUTED (with AND)
define viewer: editor from document      → TRAVERSAL
```

***

## Part 6: Building Block #5 - Tuple

### What is a Relationship Tuple?

A **tuple** is a statement of fact: "Subject X has relation R with object Y".

### Tuple Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                     Relationship Tuple                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐      ┌──────────┐      ┌──────────────────────┐ │
│  │  user    │  →   │ relation │  →   │      object          │ │
│  └──────────┘      └──────────┘      └──────────────────────┘ │
│      ↓                  ↓                      ↓               │
│   "user:bob"       "org_admin"      "organization:acme"       │
│                                                                 │
│  Translation: "Bob is an org_admin of Acme"                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Tuple Format

```json
{
  "user": "user:bob",
  "relation": "org_admin",
  "object": "organization:acme"
}
```

**Components:**
- **user:** The subject (who/what has the relationship)
- **relation:** The type of relationship
- **object:** The target object (what they have relationship with)

### What Gets Stored as Tuples?

**Remember:** You only write tuples for DIRECT relationships. Computed and Traversal relations are NOT stored as tuples.

```
✅ Write as tuples (Direct Relations):
────────────────────────────────────
(user:bob, member, organization:acme)
(user:alice, org_admin, organization:acme)
(organization:acme, organization, printer:pr-001)

❌ DON'T write as tuples (Computed/Traversal):
──────────────────────────────────────────────
(user:bob, can_view, organization:acme)        ← Computed, don't write
(user:bob, can_access, printer:pr-001)         ← Traversal result, don't write
```

### Real Examples from Your System

```
Identity Domain Tuples:
───────────────────────

Bob is a member of Acme:
{
  "user": "user:bob",
  "relation": "member",
  "object": "organization:acme"
}

Alice is org_admin of Acme:
{
  "user": "user:alice",
  "relation": "org_admin",
  "object": "organization:acme"
}

Child-Org-1's parent is Acme:
{
  "user": "organization:acme",
  "relation": "parent",
  "object": "organization:child-org-1"
}


Print Domain Tuples:
────────────────────

Printer pr-001 belongs to Acme:
{
  "user": "organization:acme",
  "relation": "organization",
  "object": "printer:pr-001"
}


Fleet Domain Tuples (Printer Fleet & IoT):
───────────────────────────────────────────

Managed printer fleet-pr-100 belongs to Acme:
{
  "user": "organization:acme",
  "relation": "organization",
  "object": "managed_printer:fleet-pr-100"
}

IoT sensor belongs to Building A device group:
{
  "user": "device_group:building-a-devices",
  "relation": "member",
  "object": "iot_device:sensor-42"
}

Charlie is fleet_manager for Acme:
{
  "user": "user:charlie",
  "relation": "fleet_manager",
  "object": "organization:acme"
}
```

### Special Tuple Formats

#### 1. User Set (Group Membership)

```json
{
  "user": "group:engineering#member",
  "relation": "viewer",
  "object": "document:roadmap"
}
```

**Meaning:** All members of the engineering group are viewers of the roadmap.

#### 2. Wildcard (Everyone)

```json
{
  "user": "user:*",
  "relation": "viewer",
  "object": "document:public-announcement"
}
```

**Meaning:** All users can view the public announcement.

### Tuples vs Properties

```
❌ WRONG - Don't use tuples for properties:
───────────────────────────────────────────

{
  "user": "Bob Smith",
  "relation": "name",
  "object": "user:bob"
}

{
  "user": "bob@acme.com",
  "relation": "email",
  "object": "user:bob"
}


✅ RIGHT - Use tuples for relationships:
────────────────────────────────────────

{
  "user": "user:bob",
  "relation": "member",
  "object": "organization:acme"
}

{
  "user": "user:bob",
  "relation": "fleet_manager",
  "object": "organization:acme"
}
```

**Key Rule:** Tuples express RELATIONSHIPS, not properties.

***

## Part 7: Building Block #6 - Module

### What is a Module?

A **module** is a way to organize your authorization model into separate files, allowing different teams to maintain their parts independently.

### Why Modules Matter for Your Organization

```
Without Modules:                    With Modules:
───────────────                     ─────────────

One huge model file                 core.fga (Identity team)
- All types mixed together          print.fga (Print team)
- Identity team changes             fleet.fga (Fleet team - printer fleet)
  block Print team changes          d365.fga (D365 team)
- Merge conflicts
- Hard to maintain                  Each team works independently!
```

### Module Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    Module Architecture                          │
└─────────────────────────────────────────────────────────────────┘

              ┌──────────────────┐
              │   core.fga       │
              │ (Identity Team)  │
              │                  │
              │ - user           │
              │ - organization   │
              │ - group          │
              │ - system roles   │
              └────────┬─────────┘
                       │
                       │ extends/references
                       │
       ┌───────────────┼───────────────┬────────────────┐
       │               │               │                │
       ▼               ▼               ▼                ▼
┌──────────┐    ┌──────────┐   ┌──────────┐    ┌──────────┐
│print.fga │    │fleet.fga │   │d365.fga  │    │solutions │
│(Print)   │    │(Fleet)   │   │(D365)    │    │.fga      │
│          │    │          │   │          │    │(Solutions│
│- printer │    │-managed  │   │-contract │    │ Team)    │
│- print   │    │ _printer │   │          │    │          │
│  _job    │    │-iot      │   │          │    │          │
│          │    │ _device  │   │          │    │          │
│          │    │-device   │   │          │    │          │
│          │    │ _group   │   │          │    │          │
└──────────┘    └──────────┘   └──────────┘    └──────────┘
```

### Core Module Example (Identity Team Owns)

```openfga
# File: core.fga
# Owner: Identity Team
# Description: Core authorization model for identity management

module core

model
  schema 1.1

# ============================================
# Type: User
# ============================================
type user

# ============================================
# Type: Organization
# ============================================
type organization
  relations
    # Direct membership
    define member: [user, group#member]
    
    # System roles (defined by Identity team)
    define system_admin: [user]
    define org_admin: [user]
    define user_admin: [user]
    define partner_admin: [user]
    define help_desk: [user]
    
    # Organizational hierarchy
    define parent: [organization]
    
    # Authorization rules
    define can_manage_users: user_admin or org_admin or system_admin
    define can_view_org: help_desk or org_admin or system_admin

# ============================================
# Type: Group
# ============================================
type group
  relations
    define member: [user, group#member]
    define can_view: member
```

### Print Module Example (Print Team Owns)

```openfga
# File: print.fga
# Owner: Print Team
# Description: Authorization model for print domain

module print

# Import core types
extend core

model
  schema 1.1

# ============================================
# Type: Printer
# ============================================
type printer
  relations
    # Printer belongs to an organization
    define organization: [organization]
    
    # Authorization: Can access printer if user is org_admin of the org
    define can_access: org_admin from organization
    define can_manage: org_admin from organization

# ============================================
# Type: Print Job
# ============================================
type print_job
  relations
    define owner: [user]
    define printer: [printer]
    
    define can_view: owner or (can_access from printer)
    define can_cancel: owner
```

### Fleet Module Example (Fleet Team Owns - Printer Fleet & IoT)

```openfga
# File: fleet.fga
# Owner: Fleet Team
# Description: Authorization model for printer fleet management and IoT devices

module fleet

# Import core types
extend core

model
  schema 1.1

# ============================================
# Type: Managed Printer (Fleet-Managed)
# ============================================
type managed_printer
  relations
    # Printer belongs to an organization
    define organization: [organization]
    
    # Part of a device group
    define device_group: [device_group]
    
    # Fleet management roles
    define fleet_manager: [user]
    
    # Authorization rules
    define can_manage: fleet_manager or (org_admin from organization)
    define can_view_status: can_manage or (user_admin from organization)
    define can_configure: fleet_manager or (org_admin from organization)

# ============================================
# Type: IoT Device (Sensors, Scanners, etc.)
# ============================================
type iot_device
  relations
    # Device belongs to organization
    define organization: [organization]
    
    # Part of a device group
    define device_group: [device_group]
    
    # Device administrators
    define device_admin: [user]
    
    # Authorization rules
    define can_manage: device_admin or (org_admin from organization)
    define can_view_metrics: can_manage or (user_admin from organization)
    define can_send_commands: device_admin or (org_admin from organization)

# ============================================
# Type: Device Group (Logical grouping)
# ============================================
type device_group
  relations
    # Group belongs to organization
    define organization: [organization]
    
    # Group managers
    define group_manager: [user]
    
    # Parent group (for hierarchical grouping)
    define parent_group: [device_group]
    
    # Authorization rules
    define can_manage_group: group_manager or (org_admin from organization)
    define can_view_devices: can_manage_group or (user_admin from organization)

# ============================================
# Type: Maintenance Schedule
# ============================================
type maintenance_schedule
  relations
    # Schedule belongs to organization
    define organization: [organization]
    
    # Applies to device group
    define applies_to: [device_group]
    
    # Schedule managers
    define schedule_manager: [user]
    
    # Authorization rules
    define can_edit: schedule_manager or (org_admin from organization)
    define can_view: can_edit or (user_admin from organization)
```

**Key Point:** Fleet module `extends core`, meaning it can reference types defined in core module (like `user`, `organization`, `org_admin`) but focuses on printer fleet management and IoT devices rather than vehicle fleet.

### How Modules Compose

```
When deployed, modules are MERGED into a single authorization model:

core.fga + print.fga + fleet.fga + d365.fga
                  ↓
        Combined Authorization Model
                  ↓
            Deployed to Store
```

***

## Part 8: Building Block #7 - Check Query

### What is a Check Query?

A **check query** asks: "Can user X perform relation R on object Y?"

This is your primary authorization decision API.

### Check Query Structure

```
Question: Can user:bob manage managed_printer:fleet-pr-100?

API Call:
POST /stores/{store_id}/check
{
  "tuple_key": {
    "user": "user:bob",
    "relation": "can_manage",
    "object": "managed_printer:fleet-pr-100"
  }
}

Response:
{
  "allowed": true
}
```

### How Check Query Works (Under the Hood)

```
┌─────────────────────────────────────────────────────────────────┐
│  Query: Can user:bob manage managed_printer:fleet-pr-100?      │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 1: Load Authorization Model                              │
│  ────────────────────────────────────                          │
│  type managed_printer                                           │
│    relations                                                    │
│      define organization: [organization]                        │
│      define fleet_manager: [user]                               │
│      define can_manage: fleet_manager or                        │
│                        (org_admin from organization)            │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 2: Understand the Rule                                   │
│  ────────────────────────────                                  │
│  can_manage = fleet_manager or (org_admin from organization)    │
│                                                                 │
│  Translation: To manage a printer, you must be either:         │
│    - fleet_manager of the printer, OR                          │
│    - org_admin of the organization that owns the printer       │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 3: Check if Bob is fleet_manager                         │
│  ───────────────────────────────────────                       │
│  Look for tuple: (user:bob, fleet_manager, managed_printer:...)│
│  Not found: NO                                                  │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 4: Find Printer's Organization                           │
│  ────────────────────────────────────                          │
│  Look for tuple: (?, organization, managed_printer:fleet-pr-100│
│  Found: (organization:acme, organization, managed_printer:...)  │
│  → Printer belongs to organization:acme                         │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 5: Check if Bob is org_admin of Acme                     │
│  ───────────────────────────────────────────                   │
│  Look for tuple: (user:bob, org_admin, organization:acme)      │
│  Found: YES, tuple exists                                       │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 6: Return Result                                         │
│  ──────────────────────                                        │
│  {"allowed": true}                                              │
└─────────────────────────────────────────────────────────────────┘
```

### Real-World Examples

#### Example 1: Simple Direct Check

```openfga
Model:
type organization
  relations
    define member: [user]

Tuple:
(user:alice, member, organization:acme)

Check Query:
Is alice member of organization:acme?

Result: YES (direct tuple exists)
```

#### Example 2: Computed Permission Check

```openfga
Model:
type document
  relations
    define owner: [user]
    define editor: [user]
    define can_edit: owner or editor

Tuples:
(user:bob, owner, document:report)

Check Query:
Can bob edit document:report?

OpenFGA Logic:
- can_edit = owner or editor
- Is bob owner? YES
- Result: YES (computed from rule)
```

#### Example 3: Fleet Domain - IoT Device Access

```openfga
Model:
type organization
  relations
    define org_admin: [user]

type iot_device
  relations
    define organization: [organization]
    define device_admin: [user]
    define can_view_metrics: device_admin or (org_admin from organization)

Tuples:
(user:charlie, org_admin, organization:acme)
(organization:acme, organization, iot_device:sensor-42)

Check Query:
Can charlie view metrics for iot_device:sensor-42?

OpenFGA Logic:
- can_view_metrics = device_admin or (org_admin from organization)
- Is charlie device_admin? NO
- Find device's org: organization:acme
- Is charlie org_admin of acme? YES
- Result: YES (traversed relationship)
```

***

## Part 9: Building Block #8 - Write Operations

### Writing Relationship Tuples

When data changes in your system, you write tuples to OpenFGA.

### Write API Structure

```bash
POST /stores/{store_id}/write
{
  "writes": {
    "tuple_keys": [
      {
        "user": "user:bob",
        "relation": "member",
        "object": "organization:acme"
      }
    ]
  }
}
```

### When to Write Tuples

```
┌─────────────────────────────────────────────────────────────────┐
│             Business Event → Write Tuple                        │
└─────────────────────────────────────────────────────────────────┘

Event: User Bob joins organization Acme
────────────────────────────────────────
Write tuple: (user:bob, member, organization:acme)


Event: Bob is promoted to org_admin
────────────────────────────────────
Write tuple: (user:bob, org_admin, organization:acme)


Event: Organization creates child org
──────────────────────────────────────
Write tuple: (organization:acme, parent, organization:child-1)


Event: Managed printer registered to organization
──────────────────────────────────────────────────
Write tuple: (organization:acme, organization, managed_printer:fleet-pr-100)


Event: IoT device added to device group
────────────────────────────────────────
Write tuple: (device_group:building-a, member, iot_device:sensor-42)


Event: User assigned as fleet_manager
──────────────────────────────────────
Write tuple: (user:charlie, fleet_manager, organization:acme)
```

### Batch Write Example

```json
POST /stores/{store_id}/write
{
  "writes": {
    "tuple_keys": [
      {
        "user": "user:bob",
        "relation": "member",
        "object": "organization:acme"
      },
      {
        "user": "user:alice",
        "relation": "member",
        "object": "organization:acme"
      },
      {
        "user": "user:charlie",
        "relation": "fleet_manager",
        "object": "organization:acme"
      }
    ]
  }
}
```

**Limit:** Up to 100 tuples per write request.

### Delete Tuples

```json
POST /stores/{store_id}/write
{
  "deletes": {
    "tuple_keys": [
      {
        "user": "user:bob",
        "relation": "member",
        "object": "organization:acme"
      }
    ]
  }
}
```

**Use case:** When Bob leaves the organization, delete his member tuple.

***

## Part 10: Building Block #9 - List Operations

### List Objects Query

**Question:** "What can user X access?"

```bash
POST /stores/{store_id}/list-objects
{
  "user": "user:bob",
  "relation": "can_manage",
  "type": "managed_printer"
}

Response:
{
  "objects": [
    "managed_printer:fleet-pr-100",
    "managed_printer:fleet-pr-200",
    "managed_printer:fleet-pr-305"
  ]
}
```

**Use case:** Building a UI that shows all managed printers Bob can manage in the fleet.

### List Users Query

**Question:** "Who has access to object X?"

```bash
POST /stores/{store_id}/list-users
{
  "object": "iot_device:sensor-42",
  "relation": "can_view_metrics",
  "user_filters": [
    {"type": "user"}
  ]
}

Response:
{
  "users": [
    {"object": {"type": "user", "id": "bob"}},
    {"object": {"type": "user", "id": "alice"}},
    {"object": {"type": "user", "id": "charlie"}}
  ]
}
```

**Use case:** Auditing who has access to view metrics from a specific IoT sensor.

---

## Part 11: Key Concepts Summary

### The Complete Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenFGA Store                              │
│                 "Company Authorization"                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  Authorization Model (The Rules)                          │ │
│  │  Schema Version: 1.1  ← Language version                 │ │
│  │  Model Version: 47    ← Your deployment version          │ │
│  │                                                           │ │
│  │  Types:           Relations:        Authorization Logic: │ │
│  │  - user           - member          - Rules that combine │ │
│  │  - organization   - org_admin         relations          │ │
│  │  - managed_printer- can_manage      - Traversal logic    │ │
│  │  - iot_device     - fleet_manager   - Computed perms     │ │
│  │  - device_group   - device_admin                         │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  Relationship Tuples (The Facts)                          │ │
│  │                                                           │ │
│  │  (user:charlie, fleet_manager, organization:acme)         │ │
│  │  (organization:acme, organization, managed_printer:...)   │ │
│  │  (user:dave, device_admin, iot_device:sensor-42)          │ │
│  │  ... millions of relationship records ...                 │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
              │                                 │
              │                                 │
              ▼                                 ▼
      ┌───────────────┐              ┌───────────────────┐
      │ Check Query   │              │ Write Operation   │
      │ (Read)        │              │ (Write)           │
      └───────────────┘              └───────────────────┘
```

### Jargon Reference

| Term | Simple Definition | Example |
|------|-------------------|---------|
| **Store** | Container for all authorization data | Your company's authorization database |
| **Schema Version** | OpenFGA language version | `1.1` (like "Python 3.10") |
| **Model Version** | Your model deployment version | `01HVM47...` (like Git commit) |
| **Authorization Model** | Schema that defines types and rules | Like CREATE TABLE statements |
| **Type** | Category of objects | user, organization, managed_printer, iot_device |
| **Relation** | Named connection between objects | member, fleet_manager, device_admin, can_manage |
| **Tuple** | Statement of fact about a relationship | Charlie is fleet_manager of Acme |
| **Module** | Organizational unit for models | core.fga, print.fga, fleet.fga |
| **Direct Relation** | Explicitly stored relationship | Stored as tuple |
| **Computed Relation** | Calculated from rules | Derived from other relations |
| **Traversal** | Following relationship graph | managed_printer → org → check admin |
| **Check** | Authorization decision query | Can Charlie manage this printer? |
| **Write** | Adding/removing tuples | Charlie becomes fleet_manager |
| **List Objects** | Find accessible resources | What printers can Charlie manage? |
| **List Users** | Find users with access | Who can view this IoT device? |

### What You Write vs What OpenFGA Computes

```
YOU WRITE (Tuples):                    OPENFGA COMPUTES:
───────────────────                    ─────────────────

(user:charlie, fleet_manager,       →  Charlie can manage printers
 organization:acme)                     Charlie can view printer status
                                        Charlie can edit maintenance schedules
                                    
(org:acme, organization,            →  Combined with above:
 managed_printer:fleet-pr-100)          Charlie can manage fleet-pr-100
                                    
(user:dave, device_admin,           →  Dave can view metrics
 iot_device:sensor-42)                  Dave can send commands to sensor-42
```

**Key Rule:** Only write DIRECT relationships as tuples. OpenFGA computes the rest.

***

## Summary: Your Mental Checklist

```
✓ Store = Container for everything
✓ Schema Version = OpenFGA language version (1.1)
✓ Model Version = Your deployment version (auto-generated ID)
✓ Authorization Model = Schema defining types, relations, rules
✓ Type = Category of objects (user, organization, managed_printer, iot_device)
✓ Relation = Named connection (3 types: Direct, Computed, Traversal)
  ├─ Direct: Stored as tuples ([user])
  ├─ Computed: Calculated (admin or member)
  └─ Traversal: Follow graph (admin from organization)
✓ Tuple = Statement of fact (Charlie is fleet_manager of Acme)
✓ Module = Organizational unit (core.fga, fleet.fga)
✓ Check = Authorization decision API
✓ Write = Adding/removing relationship tuples
✓ Fleet Domain = Printer fleet management + IoT devices (NOT vehicles)
```

***

## Next Steps

Now that you understand all the building blocks with clear explanations of:
- Schema Version vs Model Version
- The three types of relations (Direct, Computed, Traversal)
- When to write tuples vs when OpenFGA computes

***

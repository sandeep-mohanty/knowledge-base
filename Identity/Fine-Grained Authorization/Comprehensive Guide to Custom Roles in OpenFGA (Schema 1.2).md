# Comprehensive Guide to Custom Roles in OpenFGA (Schema 1.2)

OpenFGA schema 1.2 introduces support for modular models and enhanced features while maintaining backward compatibility for core relations like custom roles. This updated tutorial uses `schema 1.2` throughout, with identical semantics for custom roles as in 1.1—update your models by changing the version and enabling experimental flags if needed for v1.5.x deployments. Examples remain production-ready for ReBAC in CIAM systems.[1][2][3]

## Core Concepts

Custom roles leverage `role` types with `assignee` relations, referenced as computed users (e.g., `role:media-manager#assignee`) for dynamic permissions. Schema 1.2 supports these via the same rewrite rules, plus modular extensions for complex enterprise models.[2][4]

- **Static Relations**: Fixed roles like `admin: [user]`.
- **Dynamic Roles**: User-created via separate `role` objects.
- **ComputedUsers**: `type#relation` for indirect access evaluation.

## Approach 1: Relations as Roles (Static)

Predefine roles in relations for simplicity.[5]

### Authorization Model
```dsl
model
schema 1.2

type user

type organization
  relations
    define admin: [user]
    define can_create_project: admin
    define can_edit_billing: admin
```

### Write Tuples
```
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:anne admin organization:acme
```
Anne inherits project creation rights.[5]

### Model Extension
```dsl
define project_admin: [user]
define can_create_project: admin or project_admin
define can_view_project: admin or project_admin
```
Existing tuples propagate automatically.[5]

## Approach 2: Simple User-Defined Roles

Dynamic roles scoped globally or per-tenant.[4]

### Authorization Model (Asset Management)
```dsl
model
schema 1.2

type user

type role
  relations
    define assignee: [user]

type asset-category
  relations
    define viewer: [user, role#assignee] or editor
    define editor: [role#assignee]
```

### Assign Users to Roles
```bash
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:anne assignee role:media-manager
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:beth assignee role:media-viewer
```

### Grant Role Permissions
```bash
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:media-manager editor asset-category:logos
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:media-viewer viewer asset-category:logos
```
Checks confirm Anne edits, Beth views.[4]

## Approach 3: Role Assignments (Instance-Specific)

Per-object assignments with `and` intersections.[5]

### Full Authorization Model
```dsl
model
schema 1.2

type user

type role
  relations
    define can_view_project: [user:*]
    define can_edit_project: [user:*]

type role_assignment
  relations
    define assignee: [user]
    define role: [role]
    define can_view_project: assignee and can_view_project from role
    define can_edit_project: assignee and can_edit_project from role

type organization
  relations
    define admin: [user]

type project
  relations
    define organization: [organization]
    define role_assignment: [role_assignment]
    define can_view_project: can_view_project from role_assignment or admin from organization
    define can_edit_project: can_edit_project from role_assignment or admin from organization
```

### Setup
1. Role permissions:
```bash
fga tuple write ... user:* can_view_project role:project-admin
fga tuple write ... user:* can_edit_project role:project-admin
```

2. Assignment:
```bash
fga tuple write ... user:anne assignee role_assignment:proj1-admin
fga tuple write ... role:project-admin role role_assignment:proj1-admin
fga tuple write ... role_assignment:proj1-admin role_assignment project:openfga
```

3. Links:
```bash
fga tuple write ... organization:acme organization project:openfga
fga tuple write ... user:anne admin organization:acme
```
Hybrid access works seamlessly.[5]

## Multi-Tenant Example

```dsl
type tenant
  relations
    define admin: [user]
    define role: [role]
    define can_manage_users: [role#assignee] or admin

type role
  relations
    define assignee: [user]
```
Tenant-isolated custom roles.[5]

## Best Practices (Schema 1.2)

- Update to `schema 1.2` for future modular models (enable `enable-modular-models` flag pre-v1.5.3).[6][3]
- Hybrid static/dynamic for performance.
- Test expansions; monitor tuple counts in Redis-backed stores.
- For Kubernetes: Use ConfigMaps for models, Azure Pipelines for tuple migrations.

## Pitfalls and Schema Migration

- Schema 1.2 requires server flag for early versions; fully stable post-v1.5.[3]
- Migrate: Write new model with `1.2`, test checks, batch migrate tuples.
- Debug with `expand`: Reveals computation paths unchanged from 1.1.[4]

This schema 1.2 version aligns with latest OpenFGA releases for advanced ReBAC in containerized CIAM setups.[7][2][4]

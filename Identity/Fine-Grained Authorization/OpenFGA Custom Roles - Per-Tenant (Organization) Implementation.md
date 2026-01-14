# Full Tutorial: Per-Organization Custom Roles in OpenFGA

Per-organization custom roles scope dynamic, user-defined roles to organizations, ensuring isolation in multi-tenant CIAM systems integrated with Azure AD B2C or Keycloak. This comprehensive tutorial uses schema 1.2 with correct DSL syntax (`from` for propagation), deep explanations, CLI/Python examples, and enterprise best practices. Ideal for ReBAC in Kubernetes/OpenFGA deployments modeling organizations as isolation boundaries.[1][2][3]

## Prerequisites and Setup

Deploy OpenFGA locally:
```bash
docker run -p 8080:8080 -p 3000:3000 -p 8081:8081 \
  openfga/openfga:latest run --store=default-test
```

Create store/model:
```bash
FGA_STORE_ID=$(fga store create --name per-org-tutorial)
MODEL_ID=$(fga write-model --store-id $FGA_STORE_ID --file model.fga)
```

## Complete Authorization Model

Full DSL using `organization` for tenant semantics.

### model.fga
```dsl
model
schema 1.2

type user

type role
  relations
    define assignee: [user]

type organization
  relations
    define admin: [user]              # Static role
    define role: [role]               # Dynamic custom roles
    define document: [document]       # Scoped resources
    define user_pool: [user_pool]     # CIAM extension

    # Permissions at organization level
    define can_view_document: [role#assignee] or admin
    define can_edit_document: [role#assignee] or admin
    define can_manage_users: [role#assignee] or admin

type document
  relations
    define organization: [organization]
    define can_view: can_view_document from organization
    define can_edit: can_edit_document from organization

type user_pool
  relations
    define organization: [organization]
    define member: [user]
    define can_manage: can_manage_users from organization
```
`can_view_document from organization` propagates org permissions to documents.[3][1]

## Implementation Steps

### Step 1: Bootstrap Organizations
```bash
# Acme Corp organization
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:anne admin organization:acme-corp

# Globant organization
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:charlie admin organization:globant

# Documents
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  document:acme-hr-policy organization:acme-corp
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  document:globant-secrets organization:globant
```

### Step 2: Create Per-Organization Custom Roles
```bash
# Acme HR Manager
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:acme-hr-manager role organization:acme-corp
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:bob assignee role:acme-hr-manager

# Acme Viewer
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:acme-viewer role organization:acme-corp
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:diana assignee role:acme-viewer

# Globant Engineer
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:globant-engineer role organization:globant
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:eric assignee role:globant-engineer
```

### Step 3: Assign Permissions to Roles on Organization
```bash
# Acme HR: edit
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:acme-hr-manager#assignee can_edit_document organization:acme-corp

# Acme Viewer: view
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:acme-viewer#assignee can_view_document organization:acme-corp

# Globant Engineer: edit
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  role:globant-engineer#assignee can_edit_document organization:globant
```

### Step 4: CIAM User Pools
```bash
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user_pool:acme-employees organization:acme-corp
fga tuple write --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:bob member user_pool:acme-employees
```

## Verification Examples

### CLI Checks
```bash
# Bob edits Acme policy
fga check --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:bob can_edit document:acme-hr-policy
# {"allowed": true}

# Diana views
fga check --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:diana can_view document:acme-hr-policy
# {"allowed": true}

# Cross-org isolation
fga check --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:bob can_edit document:globant-secrets
# {"allowed": false}

# User pool management
fga check --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  user:bob can_manage user_pool:acme-employees
# {"allowed": true}
```

### Python SDK
```python
from openfga_sdk import OpenFGAClient, CheckRequest, TupleKey

client = OpenFGAClient("http://localhost:8080")
req = CheckRequest(tuple_key=TupleKey(
    user="user:bob",
    relation="can_edit",
    object="document:acme-hr-policy"
))
resp = client.check(req, store_id=FGA_STORE_ID, authorization_model_id=MODEL_ID)
print(resp.allowed)  # True
```

### Expand Trace
```bash
fga expand --store-id=$FGA_STORE_ID --model-id=$MODEL_ID \
  document:acme-hr-policy can_edit
```
Paths: `organization:acme-corp` → `role:acme-hr-manager#assignee` → `user:bob`.[1]

## Advanced Patterns

### Per-Document Overrides
```dsl
type document
  relations
    define organization: [organization]
    define role_assignment: [role_assignment]
    define can_edit: can_edit_document from organization or can_edit from role_assignment
```

### Hierarchical Organizations
```dsl
type organization
  relations
    define parent: [organization]
    define can_view_document: [role#assignee] or admin or can_view_document from parent
```

## Production Best Practices

| Aspect | Recommendation | Rationale [2] |
|--------|----------------|--------------------|
| Naming | `role:{org}:{role}` | Clarity in lists |
| Scale | <100 roles/org | <100ms checks |
| CI/CD | Model/tuples in Azure Pipelines | Atomic updates |
| Storage | Redis Cluster | High TPS |
| Audit | `ReadChanges` by org | Compliance |
| Test | Playground + pytest-openfga | Full coverage [4] |

- **Docker/K8s**: Volume for datastore; HorizontalPodAutoscaler.
- **Cleanup**: Scripted `batch delete` on org termination.

## Errors and Fixes

- **Propagation Fail**: Verify `from organization` syntax.[3]
- **Leakage**: Ensure `role org:id` tuple exists.
- **Performance**: Check on org root first.
- **DSL Validation**: `fga validate --file model.fga`.[5]

Fully corrected and organization-focused for your multi-domain CIAM workflows.[2][1][3]

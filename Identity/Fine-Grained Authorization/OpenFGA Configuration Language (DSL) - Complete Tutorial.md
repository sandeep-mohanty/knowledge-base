# OpenFGA Configuration Language (DSL)

## Table of Contents

1. [Introduction](#introduction)
2. [Basic Structure](#basic-structure)
3. [Type Definitions](#type-definitions)
4. [Relations](#relations)
5. [Direct Relations](#direct-relations)
6. [Computed Relations](#computed-relations)
7. [Relation Operators](#relation-operators)
8. [Hierarchical Relationships](#hierarchical-relationships)
9. [Conditional Logic](#conditional-logic)
10. [Advanced Patterns](#advanced-patterns)
11. [Real-World Examples](#real-world-examples)
12. [Best Practices](#best-practices)

***

## Introduction

The OpenFGA DSL (Domain-Specific Language) is used to define authorization models. It describes:
- **Types**: Resources in your system (users, documents, organizations)
- **Relations**: Relationships between resources (owner, viewer, member)
- **Permissions**: What actions are allowed based on relations

### Key Concepts

```
Type: A category of object (e.g., document, folder, organization)
Relation: A connection between objects (e.g., owner, viewer, member)
Tuple: A stored relationship (e.g., "Alice is owner of Doc1")
Check: A query (e.g., "Can Alice view Doc1?")
```

***

## Basic Structure

### Minimal Model

```fga
model
  schema 1.1

type user
```

**Explanation:**
- `model`: Declares start of model
- `schema 1.1`: Specifies DSL version (use 1.1 for latest features)
- `type user`: Defines a type called "user"

### Model with Multiple Types

```fga
model
  schema 1.1

type user

type document
  relations
    define owner: [user]
    define viewer: [user]

type folder
  relations
    define owner: [user]
```

**Explanation:**
- Three types defined: `user`, `document`, `folder`
- `document` has relations: `owner` and `viewer`
- Relations specify allowed types in square brackets `[user]`

***

## Type Definitions

### Simple Type (No Relations)

```fga
type user
```

**Use case:** Represents users/principals that perform actions. Typically has no relations.

### Type with Relations

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
```

**Use case:** A resource with different access levels.

**Stored tuples example:**
```
(user:alice, owner, document:readme)
(user:bob, editor, document:readme)
(user:charlie, viewer, document:readme)
```

### Multiple Allowed Types

```fga
type document
  relations
    define owner: [user, team#member]
```

**Explanation:**
- `owner` can be either:
  - A `user` directly
  - A `team#member` (users who are members of a team)

**Stored tuples example:**
```
(user:alice, owner, document:readme)              # Direct user ownership
(team:engineering#member, owner, document:spec)   # Team member ownership
```

***

## Relations

### Anatomy of a Relation

```fga
define relation_name: [allowed_type1, allowed_type2#relation]
```

**Components:**
- `relation_name`: Name of the relationship
- `[...]`: List of allowed types that can have this relation
- `type#relation`: Reference to members of a relation from another type

### Examples

```fga
type organization
  relations
    # Direct user assignment
    define member: [user]
    
    # Can include groups
    define admin: [user, group#member]
    
    # Can reference other organizations
    define parent: [organization]
```

***

## Direct Relations

Direct relations are stored explicitly as tuples in the database.

### Example: Document Access

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
```

**How it works:**

**Tuple storage:**
```
(user:alice, owner, document:doc1)
(user:bob, editor, document:doc1)
(user:charlie, viewer, document:doc1)
```

**Queries:**
```
Check: user:alice owner document:doc1
Result: true (tuple exists)

Check: user:bob owner document:doc1
Result: false (no such tuple)

Check: user:bob editor document:doc1
Result: true (tuple exists)
```

***

## Computed Relations

Computed relations are derived from other relations using logic.

### Basic Computed Relation

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    # Computed: anyone who can edit can also view
    define can_view: viewer or editor or owner
    define can_edit: editor or owner
    define can_delete: owner
```

**How it works:**

**Stored tuples:**
```
(user:alice, owner, document:doc1)
(user:bob, editor, document:doc1)
```

**Query:**
```
Check: user:bob can_view document:doc1
Evaluation:
  can_view = viewer or editor or owner
  Is bob a viewer? No
  Is bob an editor? Yes ✓
Result: true
```

### Using `and` Operator

```fga
type document
  relations
    define owner: [user]
    define approved: [user]
    
    # Must be both owner AND approved
    define can_publish: owner and approved
```

**Stored tuples:**
```
(user:alice, owner, document:doc1)
(user:alice, approved, document:doc1)
(user:bob, owner, document:doc2)
```

**Queries:**
```
Check: user:alice can_publish document:doc1
Evaluation:
  can_publish = owner and approved
  Is alice owner? Yes ✓
  Is alice approved? Yes ✓
Result: true

Check: user:bob can_publish document:doc2
Evaluation:
  Is bob owner? Yes ✓
  Is bob approved? No ✗
Result: false
```

### Using `but not` Operator

```fga
type document
  relations
    define member: [user]
    define blocked: [user]
    
    # Members who are not blocked
    define can_access: member but not blocked
```

**Stored tuples:**
```
(user:alice, member, document:doc1)
(user:bob, member, document:doc1)
(user:bob, blocked, document:doc1)
```

**Queries:**
```
Check: user:alice can_access document:doc1
Evaluation:
  Is alice a member? Yes ✓
  Is alice blocked? No ✓
Result: true

Check: user:bob can_access document:doc1
Evaluation:
  Is bob a member? Yes ✓
  Is bob blocked? Yes ✗
Result: false
```

***

## Relation Operators

### OR Operator

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define can_edit: owner or editor
```

**Meaning:** Satisfies if ANY condition is true.

### AND Operator

```fga
type document
  relations
    define authorized: [user]
    define certified: [user]
    define can_publish: authorized and certified
```

**Meaning:** Satisfies only if ALL conditions are true.

### BUT NOT Operator

```fga
type document
  relations
    define member: [user]
    define suspended: [user]
    define active_member: member but not suspended
```

**Meaning:** First condition true AND second condition false.

### Combining Operators

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define guest: [user]
    define banned: [user]
    
    # Complex logic: (owner or editor) and not banned
    define can_edit: (owner or editor) but not banned
```

**Note:** Parentheses clarify precedence but aren't always required. Use them for readability.

***

## Hierarchical Relationships

### Parent-Child Relationships

```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    
    # Owner of this folder OR owner of parent folder
    define can_manage: owner or owner from parent

type document
  relations
    define folder: [folder]
    define owner: [user]
    
    # Owner of document OR owner of containing folder
    define can_edit: owner or owner from folder
```

**How it works:**

**Stored tuples:**
```
(user:alice, owner, folder:projects)
(folder:projects, parent, folder:team)
(user:bob, owner, folder:team)
(folder:docs, folder, document:readme)
(user:charlie, owner, document:readme)
```

**Query:**
```
Check: user:bob can_manage folder:projects
Evaluation:
  can_manage = owner or owner from parent
  Is bob owner of folder:projects? No
  Is bob owner from parent?
    parent = folder:team
    Is bob owner of folder:team? Yes ✓
Result: true
```

### Multi-Level Hierarchy

```fga
type organization
  relations
    define member: [user]
    define admin: [user]

type workspace
  relations
    define organization: [organization]
    define member: [user]
    
    # Members of workspace OR members of organization
    define can_access: member or member from organization

type document
  relations
    define workspace: [workspace]
    define owner: [user]
    
    # Document owner OR workspace members
    define can_view: owner or can_access from workspace
```

**Stored tuples:**
```
(user:alice, admin, organization:acme)
(user:bob, member, organization:acme)
(organization:acme, organization, workspace:engineering)
(user:charlie, member, workspace:engineering)
(workspace:engineering, workspace, document:spec)
(user:dave, owner, document:spec)
```

**Query:**
```
Check: user:bob can_view document:spec
Evaluation:
  can_view = owner or can_access from workspace
  Is bob owner? No
  can_access from workspace:
    workspace = workspace:engineering
    can_access = member or member from organization
    Is bob member of workspace? No
    Is bob member from organization?
      organization = organization:acme
      Is bob member of acme? Yes ✓
Result: true
```

***

## Conditional Logic

### Conditional Relations with `from`

The `from` keyword follows relationships from one object to another.

```fga
type organization
  relations
    define member: [user]
    define admin: [user]

type document
  relations
    define organization: [organization]
    define owner: [user]
    
    # Owner OR admin of the document's organization
    define can_delete: owner or admin from organization
```

**How `from` works:**

1. **Find related object:** Get document's organization
2. **Check relation:** Check if user has relation in that object
3. **Return result**

**Stored tuples:**
```
(organization:acme, organization, document:doc1)
(user:alice, admin, organization:acme)
(user:bob, owner, document:doc1)
```

**Query:**
```
Check: user:alice can_delete document:doc1
Steps:
  1. Find organization of document:doc1 → organization:acme
  2. Check if alice is admin of organization:acme → Yes ✓
Result: true
```

### Chained `from` Relationships

```fga
type organization
  relations
    define member: [user]

type team
  relations
    define organization: [organization]
    define member: [user]
    
    # Team members OR organization members
    define can_access: member or member from organization

type repository
  relations
    define team: [team]
    define owner: [user]
    
    # Owner OR anyone who can_access the team
    define can_view: owner or can_access from team
```

**Stored tuples:**
```
(user:alice, member, organization:acme)
(organization:acme, organization, team:backend)
(user:bob, member, team:backend)
(team:backend, team, repository:api)
(user:charlie, owner, repository:api)
```

**Query:**
```
Check: user:alice can_view repository:api
Steps:
  1. Is alice owner of repository:api? No
  2. Check can_access from team:
     - team = team:backend
     - can_access = member or member from organization
     - Is alice member of team:backend? No
     - Is alice member from organization?
       - organization = organization:acme
       - Is alice member of acme? Yes ✓
Result: true
```

***

## Advanced Patterns

### Self-Referential Relations

```fga
type user
  relations
    # Users can follow other users
    define follower: [user]
    define following: [user]

type document
  relations
    define owner: [user]
    # Document visible to owner's followers
    define can_view: owner or follower from owner
```

**Stored tuples:**
```
(user:bob, follower, user:alice)
(user:charlie, follower, user:alice)
(user:alice, owner, document:post1)
```

**Query:**
```
Check: user:bob can_view document:post1
Steps:
  1. Is bob owner? No
  2. Is bob follower from owner?
     - owner = user:alice
     - Is bob follower of alice? Yes ✓
Result: true
```

### Group Membership

```fga
type user

type group
  relations
    define member: [user, group#member]
    define manager: [user]
    
    # Managers can do everything members can
    define can_view: member or manager

type document
  relations
    define viewer_group: [group]
    
    # Anyone who can_view the viewer_group
    define can_view: can_view from viewer_group
```

**Stored tuples:**
```
(user:alice, member, group:engineers)
(user:bob, manager, group:engineers)
(group#member:engineers, member, group:all_staff)
(group:all_staff, viewer_group, document:handbook)
```

**Query:**
```
Check: user:alice can_view document:handbook
Steps:
  1. viewer_group = group:all_staff
  2. can_view from group:all_staff
     - Is alice member of all_staff? No
     - Is engineers#member member of all_staff? Yes
       - Is alice member of engineers? Yes ✓
Result: true
```

### Contextual Conditions

```fga
type document
  relations
    define owner: [user]
    define viewer: [user]
    define public: [user:*]
    
    define can_view: viewer or owner or public
```

**Special syntax `user:*`:** Represents "any user" (public access)

**Stored tuples:**
```
(user:alice, owner, document:private)
(user:*, public, document:terms)
```

**Queries:**
```
Check: user:alice can_view document:private
Result: true (alice is owner)

Check: user:anyone can_view document:terms
Result: true (user:* makes it public)
```

### Intersection of Sets

```fga
type document
  relations
    define allowed_viewer: [user]
    define department_member: [user]
    
    # Must be BOTH allowed AND in department
    define can_view: allowed_viewer and department_member
```

**Use case:** Document restricted to specific users within a department.

---

## Real-World Examples

### Example 1: Google Drive-Style Permissions

```fga
model
  schema 1.1

type user

type folder
  relations
    define parent: [folder]
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    # Computed permissions
    define can_view: viewer or editor or owner or can_view from parent
    define can_edit: editor or owner or can_edit from parent
    define can_delete: owner
    define can_share: owner

type document
  relations
    define parent: [folder]
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    # Document permissions can inherit from folder
    define can_view: viewer or editor or owner or can_view from parent
    define can_edit: editor or owner or can_edit from parent
    define can_delete: owner
    define can_share: owner
```

**Stored tuples:**
```
(user:alice, owner, folder:team)
(folder:team, parent, folder:projects)
(user:bob, viewer, folder:projects)
(folder:team, parent, document:spec)
```

**Query:**
```
Check: user:bob can_view document:spec
Path:
  document:spec → parent → folder:team → parent → folder:projects
  bob is viewer of folder:projects ✓
Result: true
```

### Example 2: GitHub-Style Repository Access

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define owner: [user]
    
    define can_create_repo: member or owner

type team
  relations
    define organization: [organization]
    define member: [user]
    define maintainer: [user]
    
    define can_manage: maintainer

type repository
  relations
    define organization: [organization]
    define owner: [user, team#member]
    define maintainer: [user, team#member]
    define writer: [user, team#member]
    define reader: [user, team#member]
    
    # Public repo flag
    define public: [user:*]
    
    # Computed permissions
    define can_read: reader or writer or maintainer or owner or public
    define can_write: writer or maintainer or owner
    define can_admin: owner or (owner from organization)
    define can_delete: can_admin

type issue
  relations
    define repository: [repository]
    define creator: [user]
    
    define can_view: creator or can_read from repository
    define can_edit: creator or can_write from repository
    define can_close: creator or can_admin from repository
```

**Stored tuples:**
```
(user:alice, owner, organization:acme)
(organization:acme, organization, team:backend)
(user:bob, member, team:backend)
(organization:acme, organization, repository:api)
(team#member:backend, writer, repository:api)
(repository:api, repository, issue:bug-123)
(user:charlie, creator, issue:bug-123)
```

**Query:**
```
Check: user:bob can_write repository:api
Steps:
  1. Is bob a writer? Check team#member:backend
  2. Is backend#member a writer? Yes ✓
  3. Is bob member of backend? Yes ✓
Result: true
```

### Example 3: Multi-Tenant SaaS Application

```fga
model
  schema 1.1

type user

type tenant
  relations
    define admin: [user]
    define member: [user]
    
    define can_manage: admin
    define can_access: member or admin

type workspace
  relations
    define tenant: [tenant]
    define owner: [user]
    define member: [user]
    
    # Tenant admins can access all workspaces
    define can_access: member or owner or can_manage from tenant
    define can_manage: owner or can_manage from tenant

type project
  relations
    define workspace: [workspace]
    define owner: [user]
    define contributor: [user]
    define viewer: [user]
    
    define can_view: viewer or contributor or owner or can_access from workspace
    define can_edit: contributor or owner or can_manage from workspace
    define can_delete: owner or can_manage from workspace

type resource
  relations
    define project: [project]
    define owner: [user]
    
    define can_access: owner or can_view from project
    define can_modify: owner or can_edit from project
```

**Stored tuples:**
```
(user:admin, admin, tenant:company1)
(tenant:company1, tenant, workspace:ws1)
(user:alice, owner, workspace:ws1)
(workspace:ws1, workspace, project:proj1)
(user:bob, contributor, project:proj1)
(project:proj1, project, resource:res1)
```

**Query:**
```
Check: user:admin can_modify resource:res1
Path:
  resource:res1 → project:proj1 → workspace:ws1 → tenant:company1
  admin is admin of company1 ✓
  can_manage from tenant → can_manage from workspace → can_edit from project
Result: true
```

### Example 4: Healthcare System with HIPAA Compliance

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define admin: [user]
    define compliance_officer: [user]

type patient
  relations
    define organization: [organization]
    
    # Primary care provider
    define primary_physician: [user]
    
    # Care team members
    define care_team: [user]
    
    # Authorized viewers (needs consent)
    define authorized_viewer: [user]
    
    # Emergency access flag
    define emergency_access: [user:*]
    
    # Computed access
    define can_view_records: (
      primary_physician or 
      care_team or 
      authorized_viewer or
      emergency_access or
      compliance_officer from organization
    )

type medical_record
  relations
    define patient: [patient]
    define created_by: [user]
    
    # Sensitivity levels
    define sensitive: [user:*]
    
    # Access: can view patient records AND created the record or not sensitive
    define can_view: (can_view_records from patient) and (created_by or not sensitive)
    define can_edit: created_by and can_view_records from patient

type prescription
  relations
    define patient: [patient]
    define prescribed_by: [user]
    
    define can_view: prescribed_by or can_view_records from patient
    define can_modify: prescribed_by
```

**Key features:**
- Role-based access (physician, care team)
- Consent-based access (authorized_viewer)
- Break-glass emergency access
- Sensitivity controls
- Audit compliance (compliance_officer)

***

## Best Practices

### 1. Name Relations Clearly

```fga
# ✅ Good - Clear intent
define can_view: viewer or editor or owner
define can_edit: editor or owner
define can_delete: owner

# ❌ Bad - Ambiguous
define access: viewer or editor or owner
define modify: editor or owner
define remove: owner
```

### 2. Use Consistent Naming Patterns

```fga
# Permission relations: use "can_" prefix
define can_view: [...]
define can_edit: [...]
define can_delete: [...]

# Role relations: use nouns
define owner: [user]
define editor: [user]
define viewer: [user]

# Relationship relations: use descriptive names
define parent: [folder]
define organization: [organization]
define workspace: [workspace]
```

### 3. Document Complex Logic

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define public: [user:*]
    define archived: [user:*]
    
    # Can view if: (owner or editor or public) AND not archived
    define can_view: (owner or editor or public) but not archived
```

### 4. Start Simple, Add Complexity

```fga
# Phase 1: Basic model
type document
  relations
    define owner: [user]
    define can_view: owner
    define can_edit: owner

# Phase 2: Add roles
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    define can_view: viewer or editor or owner
    define can_edit: editor or owner

# Phase 3: Add hierarchy
type document
  relations
    define folder: [folder]
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    define can_view: viewer or editor or owner or can_view from folder
    define can_edit: editor or owner or can_edit from folder
```

### 5. Avoid Circular Dependencies

```fga
# ❌ Bad - Circular
type group
  relations
    define member: [user, group#member]
    define can_access: member from member  # Circular!

# ✅ Good - Clear hierarchy
type group
  relations
    define parent_group: [group]
    define member: [user, group#member]
    define can_access: member or member from parent_group
```

### 6. Test Edge Cases

Always test:
- Empty relations (user has no roles)
- Multiple paths to permission (should they all work?)
- Negative cases (blocked users, archived resources)
- Deep hierarchies (10+ levels)

### 7. Use Type Namespacing for Large Models

```fga
# Identity domain
type identity_user
type identity_organization

# Print domain
type print_printer
type print_job

# Clear separation, prevents naming conflicts
```

***

## Common Patterns Quick Reference

### Pattern: Basic CRUD Permissions

```fga
type resource
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    define can_read: viewer or editor or owner
    define can_update: editor or owner
    define can_delete: owner
```

### Pattern: Hierarchical Inheritance

```fga
type parent
  relations
    define owner: [user]

type child
  relations
    define parent: [parent]
    define owner: [user]
    
    define can_access: owner or owner from parent
```

### Pattern: Group-Based Access

```fga
type group
  relations
    define member: [user, group#member]

type resource
  relations
    define viewer_group: [group]
    
    define can_view: member from viewer_group
```

### Pattern: Conditional Access

```fga
type resource
  relations
    define approved: [user]
    define member: [user]
    
    define can_access: member and approved
```

### Pattern: Exclusion

```fga
type resource
  relations
    define member: [user]
    define banned: [user]
    
    define can_access: member but not banned
```

***

## Summary

### Key Syntax Elements

```fga
model                              # Start of model
  schema 1.1                       # Schema version

type type_name                     # Define a type

  relations                        # Relations section
    define relation_name: [type]   # Direct relation
    define computed: logic         # Computed relation

Operators:
  or      # Any condition true
  and     # All conditions true
  but not # First true, second false
  from    # Follow relationship

Type references:
  [user]           # Direct user
  [type#relation]  # Users with relation in type
  [user:*]         # Wildcard (any user)
```

### Learning Path

1. **Start**: Simple types with direct relations
2. **Add**: Computed relations with `or`
3. **Introduce**: Hierarchies with `from`
4. **Advanced**: Complex logic with `and`, `but not`
5. **Master**: Multi-level hierarchies and group memberships

### Next Steps

- Practice with the examples above
- Model your own domain
- Test with OpenFGA CLI
- Iterate based on real requirements

***

**Remember:** Authorization models evolve. Start simple, test thoroughly, and add complexity only when needed.

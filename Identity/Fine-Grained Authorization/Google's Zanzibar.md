# Google's Zanzibar: A Comprehensive Tutorial

## Table of Contents

1. [Introduction to Zanzibar](#introduction-to-zanzibar)
2. [Core Concepts](#core-concepts)
3. [Relation Tuples](#relation-tuples)
4. [Namespace Configuration](#namespace-configuration)
5. [Modeling Authorization with Zanzibar](#modeling-authorization-with-zanzibar)
6. [Common Patterns](#common-patterns)
7. [Open-source Implementations](#open-source-implementations)
   - [OpenFGA](#openfga)
   - [SpiceDB](#spicedb)
8. [Practical Examples](#practical-examples)
   - [Document Management System](#document-management-system)
   - [Team Collaboration Platform](#team-collaboration-platform)
   - [Multi-tenant SaaS Application](#multi-tenant-saas-application)
9. [Advanced Concepts](#advanced-concepts)
10. [Performance Considerations](#performance-considerations)
11. [Migration Strategies](#migration-strategies)
12. [Best Practices](#best-practices)

## Introduction to Zanzibar

Google's Zanzibar is a global authorization system used to manage access control across Google's services, including Google Docs, Drive, Calendar, and YouTube. It was described in a paper published by Google in 2019 titled "Zanzibar: Google's Consistent, Global Authorization System."

Zanzibar provides:

1. **Consistency**: Ensures that access control decisions are consistent across globally distributed services
2. **Low latency**: Makes authorization decisions in milliseconds
3. **High availability**: Supports billions of users and trillions of objects
4. **Flexibility**: Models complex access control relationships
5. **Global scale**: Works across data centers worldwide

The paper has inspired open-source implementations such as OpenFGA and SpiceDB, which bring Zanzibar's concepts to developers outside Google.

## Core Concepts

To understand Zanzibar, we need to grasp its fundamental concepts:

### Objects and Subjects

- **Object**: An entity that is being accessed (e.g., document, folder, project)
- **Subject**: An entity that is accessing objects (e.g., user, group, service)

Note that subjects can also be objects, enabling complex hierarchical relationships.

### Relations

Relations define the type of access a subject has to an object. For example:

- `viewer`: Can view the object
- `editor`: Can edit the object
- `owner`: Has full control over the object

### User Sets

A User Set is a collection of subjects that have a specific relation to an object. For example, "all users who are viewers of document X."

### Check API

The Check API determines if a subject has a specific relation to an object. It answers questions like "Can user U view document D?"

### Expand API

The Expand API lists all subjects with a specific relation to an object. It answers questions like "Who can edit document D?"

### Read API

The Read API lists all objects that a subject has a specific relation to. It answers questions like "What documents can user U view?"

## Relation Tuples

The fundamental data structure in Zanzibar is the **relation tuple**, which represents a relationship between an object and a subject. The tuple format is:

```
object#relation@subject
```

Or more precisely:

```
namespace:object#relation@subject
```

Where:
- `namespace` is the type of the object (e.g., document, folder)
- `object` is the specific object identifier
- `relation` is the type of access relationship
- `subject` can be either a user identifier or another relation tuple (a user set)

Examples:

```
document:doc1#viewer@user:alice
folder:folder1#editor@user:bob
document:doc2#viewer@folder:folder1#editor
```

The last example represents "anyone who is an editor of folder1 is also a viewer of doc2" - this demonstrates the recursive nature of Zanzibar.

## Namespace Configuration

Namespace configuration defines the structure and rules for objects in your system. It specifies:

1. **Relations**: What relations can exist for a namespace
2. **Rewrites**: How relations can be derived from other relations
3. **Allowed child types**: What types of objects can be nested under this namespace

Here's an example namespace configuration in SpiceDB syntax:

```
definition document {
    relation viewer: user
    relation editor: user
    relation owner: user
    
    permission view = viewer + editor + owner
    permission edit = editor + owner
    permission admin = owner
}

definition folder {
    relation viewer: user
    relation editor: user
    relation owner: user
    
    permission view = viewer + editor + owner
    permission edit = editor + owner
    permission admin = owner
    
    // Folder viewers can view documents in the folder
    relation document_viewer: document
    permission view_documents = document_viewer->view
}
```

## Modeling Authorization with Zanzibar

Let's explore how to model common authorization scenarios with Zanzibar:

### Direct Access

The simplest form is direct access - a user has a specific relation to an object:

```
document:doc1#viewer@user:alice
```

This means Alice can view doc1.

### Group-Based Access

Groups are modeled as objects that users have relations with:

```
group:engineering#member@user:bob
document:doc1#viewer@group:engineering#member
```

This means Bob is a member of the engineering group, and all members of the engineering group can view doc1.

### Hierarchical Access

Objects can inherit permissions from their parent objects:

```
folder:folder1#viewer@user:charlie
document:doc1#parent@folder:folder1
```

With a rewrite rule: `document#viewer <- document#parent@folder#viewer`

This means Charlie can view folder1, and since doc1 is in folder1, Charlie can also view doc1.

### Role-Based Access Control (RBAC)

Roles can be modeled as relations:

```
team:product#admin@user:dave
project:project1#team@team:product
```

With rewrite rules defining what admins can do on projects.

### Attribute-Based Access Control (ABAC)

While Zanzibar doesn't directly support ABAC, you can model it by creating relation tuples based on attributes. For example, if documents have a "classification" attribute:

```
classification:public#reader@user:everyone
document:doc1#classification@classification:public
```

With a rewrite rule: `document#viewer <- document#classification@classification#reader`

## Common Patterns

### Owner-inherits-all Pattern

```
document:doc1#owner@user:alice
document:doc1#viewer@document:doc1#owner
document:doc1#editor@document:doc1#owner
```

This means the owner of a document automatically has viewer and editor permissions.

### Team Hierarchy Pattern

```
team:engineering#member@user:bob
team:frontend#parent@team:engineering
```

This means Bob is a member of engineering, and frontend is a subteam of engineering (and thus Bob is implicitly a member of frontend).

### Public Access Pattern

```
public:global#member@user:*
document:doc1#viewer@public:global#member
```

This means anyone can view doc1.

### Time-Limited Access Pattern

Some implementations like SpiceDB support time-based conditions:

```
document:doc1#viewer@user:charlie[until=2023-12-31]
```

This means Charlie can view doc1 until December 31, 2023.

## Open-source Implementations

### OpenFGA

[OpenFGA](https://openfga.dev/) is an open-source authorization solution inspired by Zanzibar, originally developed by Auth0 (now part of Okta).

Key features:
- Simple API for authorization checks
- Modeling language for authorization relationships
- Visualization tools
- SDKs for various languages
- Playground for testing models

Example OpenFGA model (in DSL format):

```
model
  schema 1.1

type user

type document
  relations
    define viewer: [user]
    define editor: [user]
    define owner: [user]
    define parent: [folder]
    define can_view: viewer or editor or owner or can_view from parent
    define can_edit: editor or owner
    define can_delete: owner

type folder
  relations
    define viewer: [user]
    define editor: [user]
    define owner: [user]
    define parent: [folder]
    define can_view: viewer or editor or owner or can_view from parent
    define can_edit: editor or owner
    define can_delete: owner
```

Example API calls:

```javascript
// Check if user can view a document
const checkResponse = await fgaClient.check({
  user: 'user:anne',
  relation: 'can_view',
  object: 'document:budget2023'
});

// Write a tuple
await fgaClient.write({
  writes: [{
    user: 'user:anne',
    relation: 'viewer',
    object: 'document:budget2023'
  }]
});
```

### SpiceDB

[SpiceDB](https://authzed.com/spicedb) is an open-source Zanzibar-inspired database for managing permissions, developed by Authzed.

Key features:
- Schema-based definition language
- Rich query capabilities
- Distributed architecture
- Strong consistency guarantees
- Playground for testing

Example SpiceDB schema:

```
definition user {}

definition document {
    relation viewer: user
    relation editor: user
    relation owner: user
    relation parent: folder

    permission view = viewer + editor + owner + parent->view
    permission edit = editor + owner
    permission delete = owner
}

definition folder {
    relation viewer: user
    relation editor: user
    relation owner: user
    relation parent: folder

    permission view = viewer + editor + owner + parent->view
    permission edit = editor + owner
    permission delete = owner
}
```

Example API calls:

```go
// Check if user can view a document
client.CheckPermission(ctx, &v1.CheckPermissionRequest{
    Resource: &v1.ObjectReference{
        ObjectType: "document",
        ObjectId:   "budget2023",
    },
    Permission: "view",
    Subject: &v1.SubjectReference{
        Object: &v1.ObjectReference{
            ObjectType: "user",
            ObjectId:   "anne",
        },
    },
})

// Write a relation
client.WriteRelationships(ctx, &v1.WriteRelationshipsRequest{
    Updates: []*v1.RelationshipUpdate{
        {
            Operation: v1.RelationshipUpdate_OPERATION_CREATE,
            Relationship: &v1.Relationship{
                Resource: &v1.ObjectReference{
                    ObjectType: "document",
                    ObjectId:   "budget2023",
                },
                Relation: "viewer",
                Subject: &v1.SubjectReference{
                    Object: &v1.ObjectReference{
                        ObjectType: "user",
                        ObjectId:   "anne",
                    },
                },
            },
        },
    },
})
```

## Practical Examples

Let's explore practical examples of using Zanzibar-style authorization in different scenarios:

### Document Management System

For a document management system like Google Docs, we need to model:
- Direct permissions on documents
- Folder structures with inherited permissions
- Sharing links with different access levels

**Schema (SpiceDB):**

```
definition user {}

definition folder {
    relation viewer: user
    relation editor: user
    relation owner: user
    relation parent: folder

    permission view = viewer + editor + owner + parent->view
    permission edit = editor + owner
    permission manage = owner
}

definition document {
    relation viewer: user
    relation commenter: user
    relation editor: user
    relation owner: user
    relation parent: folder

    permission view = viewer + commenter + editor + owner + parent->view
    permission comment = commenter + editor + owner
    permission edit = editor + owner
    permission manage = owner
}

definition link {
    relation document: document
    relation anyone: user
    relation anyone_with_link: user

    permission use = document->view
}
```

**Example relationships:**

```
// Direct permissions
document:report#owner@user:alice
document:report#editor@user:bob
document:report#viewer@user:charlie

// Folder structure
folder:team_docs#owner@user:alice
document:report#parent@folder:team_docs

// Sharing via link
link:report_sharing#document@document:report
link:report_sharing#anyone_with_link@user:*
```

**Example checks:**

```
// Can Bob edit the report?
check document:report#edit@user:bob  // Yes

// Can Dave view the report through the folder?
// If Dave has folder:team_docs#viewer, then Yes

// Can anyone with the link view the report?
check link:report_sharing#use@user:unknown  // Yes
```

### Team Collaboration Platform

For a team collaboration platform like Slack or Microsoft Teams, we need to model:
- Team membership
- Channels with different visibility
- Role-based permissions within teams

**Schema (OpenFGA):**

```
model
  schema 1.1

type user

type team
  relations
    define member: [user]
    define admin: [user, team#admin]

type channel
  relations
    define team: [team]
    define member: [user, team#member]
    define admin: [user, team#admin]
    define can_view: member or admin
    define can_post: member or admin
    define can_manage: admin
```

**Example relationships:**

```
// Team structure
team:engineering#member@user:alice
team:engineering#member@user:bob
team:engineering#admin@user:charlie

// Channels
channel:eng-general#team@team:engineering
channel:eng-private#member@user:alice
channel:eng-private#member@user:charlie
```

**Example checks:**

```
// Can Bob view eng-general?
check channel:eng-general#can_view@user:bob  // Yes (via team)

// Can Bob view eng-private?
check channel:eng-private#can_view@user:bob  // No (not a member)

// Can Charlie manage eng-private?
check channel:eng-private#can_manage@user:charlie  // Yes (admin)
```

### Multi-tenant SaaS Application

For a multi-tenant SaaS application, we need to model:
- Organization membership
- Role-based access within organizations
- Resource ownership across tenants

**Schema (SpiceDB):**

```
definition user {}

definition organization {
    relation member: user
    relation admin: user
    relation owner: user

    permission view = member + admin + owner
    permission manage = admin + owner
    permission manage_billing = owner
}

definition project {
    relation organization: organization
    relation editor: user
    relation admin: user

    permission view = editor + admin + organization->view
    permission edit = editor + admin
    permission manage = admin + organization->manage
}

definition resource {
    relation project: project
    relation editor: user
    
    permission view = editor + project->view
    permission edit = editor + project->edit
    permission manage = project->manage
}
```

**Example relationships:**

```
// Organization structure
organization:acme#owner@user:alice
organization:acme#admin@user:bob
organization:acme#member@user:charlie
organization:acme#member@user:dave

// Projects
project:acme-website#organization@organization:acme
project:acme-website#admin@user:bob
project:acme-website#editor@user:charlie

// Resources
resource:homepage#project@project:acme-website
resource:homepage#editor@user:dave
```

**Example checks:**

```
// Can Dave edit the homepage?
check resource:homepage#edit@user:dave  // Yes (direct permission)

// Can Charlie edit the homepage?
check resource:homepage#edit@user:charlie  // Yes (via project)

// Can Alice manage the acme-website project?
check project:acme-website#manage@user:alice  // Yes (via organization)
```

## Advanced Concepts

### Contextual (Just-In-Time) Permissions

Contextual permissions allow for dynamic authorization decisions based on request context:

**Example (conceptual):**

```
document:sensitive#viewer@user:alice[if=request.ip_in_corporate_network]
```

Some implementations like SpiceDB support this through "caveats" - conditions that must be met for the permission to be granted.

### Negative Permissions

While Zanzibar's model primarily focuses on positive assertions, negative permissions can be modeled:

```
document:report#blocked@user:mallory
```

With a rewrite rule: `document#viewer <- not(document#blocked)`

### Delegation

Permission delegation allows temporary transfer of permissions:

```
document:report#delegate@user:alice
document:report#viewer@document:report#delegate
```

With time constraints: `document:report#delegate@user:alice[until=2023-12-31]`

### Audit Trails

Tracking authorization decisions is crucial for compliance and security:

```
// Log when a check is performed
log(timestamp, user, action, object, decision)

// Track lineage of how a permission was granted
traceCheck(document:report#view@user:alice)
// -> via folder:team_docs#viewer
// -> granted directly at 2023-01-15
```

## Performance Considerations

### Caching

Authorization decisions should be cached to improve performance:

```
// Cache structure
cache[object][relation][subject] = decision
```

Invalidation strategies:
- Time-based expiry
- Relation tuple change invalidation
- Zookies (cryptographic capabilities like in Zanzibar)

### Bulk Operations

For efficiency, use bulk operations when possible:

```
// Bulk check
bulkCheck([
  { object: "document:1", relation: "view", subject: "user:alice" },
  { object: "document:2", relation: "view", subject: "user:alice" },
  { object: "document:3", relation: "view", subject: "user:alice" }
])

// Bulk write
bulkWrite([
  { object: "folder:team", relation: "viewer", subject: "user:new_hire" },
  { object: "project:onboarding", relation: "editor", subject: "user:new_hire" }
])
```

### Denormalization

For read-heavy workloads, denormalize permissions for quick lookup:

```
// Instead of checking recursively each time
user_permissions = {
  "alice": {
    "document:1": ["view", "edit"],
    "document:2": ["view"]
  }
}
```

## Migration Strategies

### Incremental Migration

1. **Dual-write phase**: Write permissions to both old system and Zanzibar
2. **Verification phase**: Check decisions in both systems, log discrepancies
3. **Shadow mode**: Use old system for decisions, verify with Zanzibar
4. **Reverse shadow**: Use Zanzibar for decisions, verify with old system
5. **Full migration**: Use only Zanzibar

### Schema Evolution

As your authorization needs evolve:

1. **Add new relations/permissions**: Backward compatible
2. **Deprecate old relations**: Mark as deprecated, stop writing new tuples
3. **Transform relations**: Bulk migrate from old to new structure
4. **Remove relations**: After ensuring no references remain

## Best Practices

### Modeling Tips

1. **Start with basic relations**: `viewer`, `editor`, `owner` are often sufficient
2. **Use computed permissions**: Define `can_view`, `can_edit` as combinations of relations
3. **Keep hierarchies shallow**: Deep hierarchies impact performance
4. **Prefer direct permissions**: When possible, assign permissions directly rather than through complex chains
5. **Limit wildcard usage**: `user:*` is powerful but can lead to performance issues

### Security Considerations

1. **Principle of least privilege**: Grant minimal permissions needed
2. **Regular audits**: Review permission structures regularly
3. **Avoid permission duplication**: Use hierarchies to avoid redundant tuples
4. **Monitor check latency**: Performance degradation can indicate issues
5. **Set up alerting**: Alert on unexpected permission changes for sensitive resources

### Testing and Validation

1. **Unit test core relationships**: Ensure basic permission structures work
2. **Scenario-based testing**: Test complex real-world permission scenarios
3. **Fuzz testing**: Generate random relationship structures to find edge cases
4. **Benchmark performance**: Regularly test performance for your access patterns
5. **Visual validation**: Use tools like OpenFGA's Visualizer to confirm relationships

## Conclusion

Google's Zanzibar and its open-source implementations like OpenFGA and SpiceDB provide a powerful, flexible framework for modeling complex authorization scenarios. By understanding the core concepts of relation tuples, namespaces, and permission computation, you can implement sophisticated access control for applications of any scale.

When choosing between implementations, consider your specific requirements:

- **OpenFGA**: Simpler API, easier to get started, great for web applications
- **SpiceDB**: More robust consistency guarantees, better for larger distributed systems

Both implementations offer solid foundations for building authorization systems based on Zanzibar principles. The right choice depends on your technical stack, scale requirements, and preferred developer experience.

Start with simple models, validate them against real-world scenarios, and incrementally expand as your authorization needs grow. With Zanzibar's approach, you can build authorization systems that are both powerful and maintainable, supporting your application's growth for years to come.
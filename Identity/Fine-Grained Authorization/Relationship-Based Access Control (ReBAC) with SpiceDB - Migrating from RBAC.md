# Relationship-Based Access Control (ReBAC) with SpiceDB: Migrating from RBAC and Using C# & Python Examples

***

## Introduction

SpiceDB is an open-source, Zanzibar-inspired authorization database that allows you to implement scalable **Relationship-Based Access Control (ReBAC)**. This tutorial guides you through:

- Migrating an existing RBAC system into SpiceDB's ReBAC model.
- Defining permissions via relationship tuples.
- Querying access using SpiceDB with **C#** and **Python** examples.

***

## Existing RBAC Mapping to Model

Here's the RBAC mapping we’ll translate into SpiceDB:

| Resource | Permission | Roles with Access |
|----------|------------|-------------------|
| Users    | UserRead   | SA, OA, UA, HELPDESK |
| Users    | UserReadExpanded | SA, OA, UA, HELPDESK, USER (self only) |
| Users    | UserWrite  | SA, OA, UA |
| Users    | UserCreate | SA, OA, UA |
| Users    | UserDelete | SA, OA, UA |
| Users    | UserUpdate | SA, OA, UA |
| Users    | UserBulkImport | SA, OA, UA |

***

## Step 1: Define Your Schema in SpiceDB

SpiceDB uses a **Schema Definition Language (SDL)** to declare types and relations.

```sdl
definition user {}

definition role {
  relation member: [user]
}

definition users_resource {
  relation userread: [role#member]        # allowed roles can read
  relation userreadexpanded: [role#member] or self  # roles and self user
  relation userwrite: [role#member]
  relation usercreate: [role#member]
  relation userdelete: [role#member]
  relation userupdate: [role#member]
  relation userbulkimport: [role#member]
  relation self: [user]                  # for self-access checking
}
```

**Key points:**

- `role` objects hold members (`user`s).
- Permissions (`userread`, `userwrite`, etc.) are relations on `users_resource` pointing to roles’ members.
- `userreadexpanded` allows either roles or the user themselves (`self`) to access.

***

## Step 2: Insert Relationship Tuples

You add tuples that link users to roles, and roles to resources.

Example tuples:

```json
[
  {"user": "user:alice", "relation": "member", "object": "role:SA"},
  {"user": "user:bob", "relation": "member", "object": "role:OA"},
  {"user": "user:charlie", "relation": "member", "object": "role:UA"},
  {"user": "user:diana", "relation": "member", "object": "role:HELPDESK"},
  {"user": "user:ed", "relation": "member", "object": "role:USER"}
]
```

Permissions assigned:

```json
[
  {"user": "role:SA", "relation": "userread", "object": "users_resource:all"},
  {"user": "role:OA", "relation": "userread", "object": "users_resource:all"},
  {"user": "role:UA", "relation": "userreadexpanded", "object": "users_resource:all"},
  {"user": "user:ed", "relation": "self", "object": "users_resource:user_ed"}
]
```

This says SA and OA roles have read access to all user resources, and `user:ed` has self access to their own record.

***

## Step 3: Querying Access via SpiceDB

You can query access via SpiceDB APIs or SDK clients.

***

### Python Example

```python
from authenticate import Authentication  # your auth helper
from pydantic import BaseModel
from grpc import insecure_channel
from spicedb.client.v1 import Client as SpiceDBClient
from spicedb.client.v1 import CheckRequest

# Connect to SpiceDB
channel = insecure_channel('localhost:50051')
client = SpiceDBClient(channel)

store_name = "my_store"

def check_permission(user_id, permission, resource):
    request = CheckRequest(
        store_name=store_name,
        tuple_key={
            "user": f"user:{user_id}",
            "relation": permission,
            "object": resource
        }
    )
    response = client.Check(request)
    return response.permissionship.exists

# Example Usage
allowed = check_permission("alice", "userread", "users_resource:all")
print(f"Alice userread allowed: {allowed}")

self_allowed = check_permission("ed", "userreadexpanded", "users_resource:user_ed")
print(f"Ed self read allowed: {self_allowed}")
```

***

### C# Example

```csharp
using System;
using System.Threading.Tasks;
using Grpc.Core;
using SpiceDb.Client.V1;
using static SpiceDb.Client.V1.PermissionService;

public class SpiceDbExample
{
    public static async Task Main()
    {
        var channel = new Channel("localhost:50051", ChannelCredentials.Insecure);
        var client = new PermissionServiceClient(channel);

        var storeName = "my_store";

        var checkRequest = new CheckRequest
        {
            StoreName = storeName,
            TupleKey = new TupleKey
            {
                User = "user:alice",
                Relation = "userread",
                Object = "users_resource:all"
            }
        };

        var response = await client.CheckAsync(checkRequest);
        Console.WriteLine($"Alice userread allowed: {response.Permissionship.Exists}");

        var selfCheckRequest = new CheckRequest
        {
            StoreName = storeName,
            TupleKey = new TupleKey
            {
                User = "user:ed",
                Relation = "userreadexpanded",
                Object = "users_resource:user_ed"
            }
        };

        var selfResponse = await client.CheckAsync(selfCheckRequest);
        Console.WriteLine($"Ed self read allowed: {selfResponse.Permissionship.Exists}");

        await channel.ShutdownAsync();
    }
}
```

***

## Step 4: Advantages of SpiceDB Model

| RBAC Element | SpiceDB Equivalent |
|--------------|--------------------|
| Role (e.g., SA, OA) | `role` object with `member` relation |
| User assigned to Role | Tuple connecting `user` to `role#member` |
| Resource Permissions | Relations on `users_resource` for each permission type |
| Self-access (per-user) | `self` relation on user-specific resource instance |

SpiceDB's expressive schema language and tuple-based relationships help you:

- Represent hierarchical and contextual permissions.
- Easily add new relations such as teams or groups.
- Model self-access succinctly for user-restricted permissions.
- Maintain consistent and scalable authorization processing.

***

## Step 5: Summary & Best Practices

- Model roles explicitly as objects for flexible grouping.
- Use tuples to represent both role membership and permission grants.
- Use `or` relations to combine role-based and self permissions.
- Version your schema and tuple data.
- Monitor and test permission checks regularly.

***

## Next Steps

- Explore deriving tuples dynamically from your business logic.
- Extend the schema to support hierarchical permissions (folders, teams).
- Use SpiceDB dashboards and visualizations for debugging.

***

## Useful Links

- [SpiceDB Getting Started](https://spicedb.dev/docs)
- [SpiceDB Schema Language Reference](https://spicedb.dev/docs/schema-language)
- [SpiceDB gRPC API](https://spicedb.dev/docs/api)
- [Zanzibar Research Paper](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/)

***

With SpiceDB, you transform static RBAC into a flexible, scalable ReBAC system capable of expression fine-grained, context-aware access for modern applications.

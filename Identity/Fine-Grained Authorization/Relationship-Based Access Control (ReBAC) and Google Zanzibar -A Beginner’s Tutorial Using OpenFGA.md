# Relationship-Based Access Control (ReBAC) and Google Zanzibar: A Beginner’s Tutorial Using OpenFGA (with RBAC Example Mapping and C# & Python Code)

***

## Introduction

This tutorial shows how you can **migrate an existing RBAC system** into a **ReBAC model** using OpenFGA with examples in **C#** and **Python**. We use the following resource-permission-role mapping as a baseline:

### Existing Resource → Permission → Role Mapping (RBAC)

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

## Step 1: Model Your Resource Types and Relations in OpenFGA

We represent roles as **groups** or **roles** in ReBAC and translate permissions into **relations** on objects.

Example Model:

```fga
type user

type role
  relations
    define member: [user]

type users_resource
  relations
    define userread: [role#member]
    define userreadexpanded: [role#member] or self
    define userwrite: [role#member]
    define usercreate: [role#member]
    define userdelete: [role#member]
    define userupdate: [role#member]
    define userbulkimport: [role#member]
```

**Explanation:**
- `role` objects represent roles like SA, OA, UA, HELPDESK, etc.
- Each role has members which are users: `role#member`.
- The `users_resource` object models the Users API resource with relations matching permissions.
- We add `or self` for `UserReadExpanded` to allow user-specific access on their own record.

***

## Step 2: Define Roles and Assign Members

Each existing RBAC role corresponds to a `role` object and its members.

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

***

## Step 3: Add Permission Tuples on the Resource

Map permissions to relations on the `users_resource` object for each role:

```json
[
  {"user": "role:SA", "relation": "userread", "object": "users_resource:all"},
  {"user": "role:OA", "relation": "userread", "object": "users_resource:all"},
  
  {"user": "role:SA", "relation": "userwrite", "object": "users_resource:all"},
  {"user": "role:OA", "relation": "userwrite", "object": "users_resource:all"},
  
  {"user": "role:UA", "relation": "userreadexpanded", "object": "users_resource:all"},
  
  {"user": "user:ed", "relation": "userreadexpanded", "object": "users_resource:user_ed"}  // self
]
```

The `"users_resource:user_ed"` object represents user Ed's personal record for the `self` relation.

***

## Step 4: Sample OpenFGA Permission Checks

### Python Example Check

```python
from openfga_sdk.client import OpenFgaClient, ClientConfiguration
from openfga_sdk.client.models import ClientCheckRequest

config = ClientConfiguration(api_url="http://localhost:8080", store_id="your_store_id")
client = OpenFgaClient(config)

check_request = ClientCheckRequest(
    user="user:alice",
    relation="userread",
    object="users_resource:all"
)
response = client.check(check_request)
print(f"Alice has userread access: {response.allowed}")

# Self access example
self_check = ClientCheckRequest(
    user="user:ed",
    relation="userreadexpanded",
    object="users_resource:user_ed"
)
print(f"Ed has self read access: {client.check(self_check).allowed}")
```

***

### C# Example Check

```csharp
using OpenFga.Sdk.Client;
using OpenFga.Sdk.Client.Model;
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        var config = new ClientConfiguration
        {
            ApiUrl = "http://localhost:8080",
            StoreId = "your_store_id"
        };

        var client = new OpenFgaClient(config);

        var checkRequest = new ClientCheckRequest
        {
            User = "user:alice",
            Relation = "userread",
            Object = "users_resource:all"
        };

        var response = await client.Check(checkRequest);
        Console.WriteLine($"Alice userread allowed: {response.Allowed}");

        var selfCheckRequest = new ClientCheckRequest
        {
            User = "user:ed",
            Relation = "userreadexpanded",
            Object = "users_resource:user_ed"
        };

        var selfResponse = await client.Check(selfCheckRequest);
        Console.WriteLine($"Ed self read allowed: {selfResponse.Allowed}");
    }
}
```

***

## Step 5: Summary and Benefits

| RBAC Concept | ReBAC Representation |
|--------------|----------------------|
| Role (e.g., SA, UA) | `role` object type with members relation |
| Resource (e.g., Users API) | `users_resource` object type |
| Permissions (UserRead, UserWrite) | Relations on resource type (`userread`, `userwrite`) |
| User assigned to Role | Tuple: `user:X` member of `role:Y` |
| Role granted Permission on Resource | Tuple: `role:Y` has relation on `resource:Z` |

**Benefits:**
- Flexible, hierarchical relationships allow easier modeling than hardcoded RBAC roles.
- `self` relations give user-scoped permissions naturally.
- Model easily extends into teams, folders, and inherited permissions.
- Simplifies permission checking: just check if a relationship exists.

***

## Additional Notes

- You define **self** access by adding special resource instances per user.
- Larger systems can use groups, folders, and nested relationships alongside roles.
- OpenFGA's expressive configuration language supports complex boolean logic over relations (`or`, `and`, `but not`).
- For enterprise apps, use audit logs and version your authorization model.

***

This approach offers a clean path for migrating an RBAC system based on roles and permissions into a scalable, relationship-centric model powered by OpenFGA.

For more help with OpenFGA model syntax and SDK usage, see [OpenFGA Documentation](https://openfga.dev/docs/getting-started).

***

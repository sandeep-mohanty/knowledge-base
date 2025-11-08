# OpenFGA Expert Training Guide for CIAM Applications
## From Novice to Expert: A Comprehensive Journey with Visual Models

Welcome to your comprehensive training program for mastering OpenFGA as your organization's authorization system. This guide will take you from foundational concepts to advanced implementation patterns specifically tailored for Customer Identity and Access Management (CIAM) applications.

***

## Phase 1: Understanding the Fundamentals

### What is OpenFGA?

OpenFGA is a high-performance, open-source authorization system inspired by Google Zanzibar that implements Fine-Grained Authorization (FGA) using Relationship-Based Access Control (ReBAC). Unlike traditional Role-Based Access Control (RBAC), OpenFGA determines access based on relationships between users and resources.[1][2][3]

**Key Characteristics:**
- **Performance**: Handles 1 million requests per second with sub-millisecond response times[3]
- **Flexibility**: Supports complex authorization scenarios through relationship modeling[4][1]
- **Scalability**: Designed for enterprise-scale applications with efficient caching and connection pooling[5][3]
- **ReBAC Model**: Authorization decisions based on "Does user U have relation R with object O?"[6][7]

### RBAC vs ReBAC: A Visual Comparison

![openFGA RBAC vs ReBA](../../images/openfga-ReBAC-RBAC.png)

In RBAC, permissions are tied to roles. In ReBAC, permissions are computed dynamically based on relationships between entities.[8][6]

### Core Concepts

#### 1. Authorization Models

Authorization models define the structure of your permission system using three key elements:[1][3]

![openFGA Authorization Models](../../images/openfga-authorization-models.png)

- **Types**: The kinds of objects in your system (users, documents, organizations)
- **Relations**: How objects relate to each other (owner, member, viewer)
- **Permissions**: What actions are allowed based on relations (read, write, delete)

#### 2. Relationship Tuples

Relationships are stored as tuples in the format: `object#relation@user`[9][10][3]

![openFGA Relationship Tuples](../../images/openfga-relationship-tuples.png)

Example tuples:
```
document:readme#owner@alice
organization:acme#member@bob
resource:report#viewer@organization:acme#member
```

#### 3. The Authorization Question

OpenFGA answers: "Can user U perform action A on object O?" by checking: "Does user U have relation R with object O?"[7][6]

![openFGA Authorization Question](../../images/openfga-authorization-question.png)

***

## Phase 2: Setting Up Your Development Environment

### Architecture Overview

![openFGA Architecture Overview](../../images/openfga-architecture-overview.png)

### Step 1: Install OpenFGA Server

**Using Docker (Recommended for Development):**

```bash
# Create a docker-compose.yml file
version: '3.8'
services:
  openfga:
    image: openfga/openfga:latest
    ports:
      - "8080:8080"
      - "8081:8081"
      - "3000:3000"
    command: run
    environment:
      - OPENFGA_DATASTORE_ENGINE=memory
      - OPENFGA_LOG_FORMAT=json
```

```bash
# Start the server
docker-compose up -d
```

The OpenFGA server will be available at `localhost:8080`.[11][5]

### Step 2: Install SDK Clients

**Python SDK:**

```bash
pip3 install openfga_sdk
```

**C# SDK:**

```bash
dotnet add package OpenFGA.Sdk
```

**FGA CLI (for testing and model management):**

```bash
# MacOS
brew install openfga/tap/fga

# Linux
# Download from releases page and install
sudo apt install ./fga_<version>_linux_<arch>.deb
```


### Step 3: Verify Installation

```bash
# Check if server is running
curl http://localhost:8080/healthz
```

***

## Phase 3: Creating Your First Authorization Model

### Understanding the Modeling Process

The modeling process follows six iterative steps:[7]

![openFGA Modelling Process](../../images/openfga-modelling-process.png)

1. Pick the most important feature
2. List the object types
3. List relations for those types
4. Define relations
5. Test the model
6. Iterate

### Example: CIAM Multi-Tenant SaaS Application

Let's model a typical CIAM scenario with organizations, users, and resources.

#### Step 1: Define Requirements in Plain Language

```
- A user can be a member of an organization
- An organization can have multiple members
- A user can create a resource if they are a member of an organization
- A user can view a resource if they are a member of the organization that owns it
- A user can edit a resource if they are the owner of the resource
- A user can delete a resource if they are an admin of the organization that owns it
- Members of an organization can be assigned as admin
```


#### Step 2: Visualize the Relationship Graph

![openFGA Relationship Graph](../../images/openfga-relationship-graph.png)


#### Step 3: Create the Authorization Model

**Using OpenFGA Configuration Language:**

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define admin: [user] and member

type resource
  relations
    define org: [organization]
    define owner: [user]
    define viewer: [user, organization#member] or owner
    define editor: owner
    define can_view: viewer or member from org
    define can_edit: editor or admin from org
    define can_delete: admin from org
```


#### Step 4: Understanding Permission Inheritance

![openFGA Understanding Permission Inheritance](../../images/openfga-permission-inheritance.png)


This graph shows how permissions flow through relationships. If alice is the `owner`, she automatically gets `editor`, `viewer`, `can_edit`, and `can_view` permissions.[12][8]

#### Step 5: Create a Store and Upload the Model

**Python Example:**

```python
import asyncio
import os
from openfga_sdk import OpenFgaClient, ClientConfiguration

async def setup_openfga():
    # Configure the client
    configuration = ClientConfiguration(
        api_url=os.environ.get('FGA_API_URL', 'http://localhost:8080'),
    )
    
    # Initialize client
    async with OpenFgaClient(configuration) as fga_client:
        # Create a store
        store_response = await fga_client.create_store({
            "name": "CIAM Authorization Store"
        })
        
        store_id = store_response.id
        print(f"Created store: {store_id}")
        
        # Set the store ID for subsequent calls
        fga_client.store_id = store_id
        
        # Define the authorization model
        body = {
            "schema_version": "1.1",
            "type_definitions": [
                {
                    "type": "user"
                },
                {
                    "type": "organization",
                    "relations": {
                        "member": {
                            "this": {}
                        },
                        "admin": {
                            "intersection": {
                                "child": [
                                    {"this": {}},
                                    {"computedUserset": {"relation": "member"}}
                                ]
                            }
                        }
                    },
                    "metadata": {
                        "relations": {
                            "member": {"directly_related_user_types": [{"type": "user"}]},
                            "admin": {"directly_related_user_types": [{"type": "user"}]}
                        }
                    }
                },
                {
                    "type": "resource",
                    "relations": {
                        "org": {
                            "this": {}
                        },
                        "owner": {
                            "this": {}
                        },
                        "viewer": {
                            "union": {
                                "child": [
                                    {"this": {}},
                                    {"computedUserset": {"relation": "owner"}}
                                ]
                            }
                        },
                        "can_view": {
                            "union": {
                                "child": [
                                    {"computedUserset": {"relation": "viewer"}},
                                    {"tupleToUserset": {
                                        "tupleset": {"relation": "org"},
                                        "computedUserset": {"relation": "member"}
                                    }}
                                ]
                            }
                        },
                        "can_edit": {
                            "union": {
                                "child": [
                                    {"computedUserset": {"relation": "owner"}},
                                    {"tupleToUserset": {
                                        "tupleset": {"relation": "org"},
                                        "computedUserset": {"relation": "admin"}
                                    }}
                                ]
                            }
                        },
                        "can_delete": {
                            "tupleToUserset": {
                                "tupleset": {"relation": "org"},
                                "computedUserset": {"relation": "admin"}
                            }
                        }
                    },
                    "metadata": {
                        "relations": {
                            "org": {"directly_related_user_types": [{"type": "organization"}]},
                            "owner": {"directly_related_user_types": [{"type": "user"}]},
                            "viewer": {"directly_related_user_types": [
                                {"type": "user"},
                                {"type": "organization", "relation": "member"}
                            ]}
                        }
                    }
                }
            ]
        }
        
        # Write the model
        model_response = await fga_client.write_authorization_model(body)
        model_id = model_response.authorization_model_id
        print(f"Created model: {model_id}")
        
        return store_id, model_id

# Run the setup
store_id, model_id = asyncio.run(setup_openfga())
```

**C# Example:**

```csharp
using OpenFga.Sdk.Client;
using OpenFga.Sdk.Client.Model;
using OpenFga.Sdk.Model;

public class OpenFgaSetup
{
    public static async Task<(string storeId, string modelId)> SetupOpenFga()
    {
        var configuration = new ClientConfiguration
        {
            ApiUrl = Environment.GetEnvironmentVariable("FGA_API_URL") ?? "http://localhost:8080",
        };

        var fgaClient = new OpenFgaClient(configuration);

        // Create a store
        var storeResponse = await fgaClient.CreateStore(new ClientCreateStoreRequest
        {
            Name = "CIAM Authorization Store"
        });

        var storeId = storeResponse.Id;
        Console.WriteLine($"Created store: {storeId}");

        // Set the store ID
        configuration.StoreId = storeId;

        // Define the authorization model
        var body = new ClientWriteAuthorizationModelRequest
        {
            SchemaVersion = "1.1",
            TypeDefinitions = new List<TypeDefinition>
            {
                new TypeDefinition { Type = "user" },
                new TypeDefinition
                {
                    Type = "organization",
                    Relations = new Dictionary<string, Userset>
                    {
                        { "member", new Userset { This = new object() } },
                        { "admin", new Userset 
                            { 
                                Intersection = new Usersets 
                                { 
                                    Child = new List<Userset>
                                    {
                                        new Userset { This = new object() },
                                        new Userset { ComputedUserset = new ObjectRelation { Relation = "member" } }
                                    }
                                } 
                            } 
                        }
                    },
                    Metadata = new Metadata
                    {
                        Relations = new Dictionary<string, RelationMetadata>
                        {
                            { "member", new RelationMetadata 
                                { 
                                    DirectlyRelatedUserTypes = new List<RelationReference>
                                    {
                                        new RelationReference { Type = "user" }
                                    }
                                } 
                            },
                            { "admin", new RelationMetadata 
                                { 
                                    DirectlyRelatedUserTypes = new List<RelationReference>
                                    {
                                        new RelationReference { Type = "user" }
                                    }
                                } 
                            }
                        }
                    }
                },
                // Add resource type definition similarly
            }
        };

        var modelResponse = await fgaClient.WriteAuthorizationModel(body);
        var modelId = modelResponse.AuthorizationModelId;
        Console.WriteLine($"Created model: {modelId}");

        return (storeId, modelId);
    }
}
```


***

## Phase 4: Managing Relationship Tuples

### Relationship Tuple Lifecycle

![openFGA Relationship Tuple Lifecycle](../../images/openfga-relationship-tuple-lifecycle.png)

### Writing Relationship Tuples

Relationship tuples represent the actual relationships in your system. They must be created when users perform actions.[13][1][7]

**Python Example:**

```python
async def write_relationships(fga_client):
    """
    Write relationship tuples to OpenFGA
    """
    
    # User alice joins organization acme
    await fga_client.write({
        "writes": [
            {
                "user": "user:alice",
                "relation": "member",
                "object": "organization:acme"
            },
            {
                "user": "user:bob",
                "relation": "member",
                "object": "organization:acme"
            },
            {
                "user": "user:alice",
                "relation": "admin",
                "object": "organization:acme"
            }
        ]
    })
    
    # Create a resource owned by acme organization
    await fga_client.write({
        "writes": [
            {
                "user": "organization:acme",
                "relation": "org",
                "object": "resource:report-2024"
            },
            {
                "user": "user:alice",
                "relation": "owner",
                "object": "resource:report-2024"
            }
        ]
    })
    
    print("Relationships written successfully")
```

**C# Example:**

```csharp
public async Task WriteRelationships(OpenFgaClient fgaClient)
{
    // Write relationship tuples
    var body = new ClientWriteRequest
    {
        Writes = new List<ClientTupleKey>
        {
            new ClientTupleKey
            {
                User = "user:alice",
                Relation = "member",
                Object = "organization:acme"
            },
            new ClientTupleKey
            {
                User = "user:bob",
                Relation = "member",
                Object = "organization:acme"
            },
            new ClientTupleKey
            {
                User = "user:alice",
                Relation = "admin",
                Object = "organization:acme"
            },
            new ClientTupleKey
            {
                User = "organization:acme",
                Relation = "org",
                Object = "resource:report-2024"
            },
            new ClientTupleKey
            {
                User = "user:alice",
                Relation = "owner",
                Object = "resource:report-2024"
            }
        }
    };

    await fgaClient.Write(body);
    Console.WriteLine("Relationships written successfully");
}
```


### Deleting Relationship Tuples

When relationships change (e.g., user leaves organization), you must delete the tuples:[1]

**Python Example:**

```python
async def remove_user_from_org(fga_client, user_id, org_id):
    """
    Remove a user from an organization
    """
    await fga_client.write({
        "deletes": [
            {
                "user": f"user:{user_id}",
                "relation": "member",
                "object": f"organization:{org_id}"
            }
        ]
    })
```

**C# Example:**

```csharp
public async Task RemoveUserFromOrg(OpenFgaClient fgaClient, string userId, string orgId)
{
    var body = new ClientWriteRequest
    {
        Deletes = new List<ClientTupleKey>
        {
            new ClientTupleKey
            {
                User = $"user:{userId}",
                Relation = "member",
                Object = $"organization:{orgId}"
            }
        }
    };

    await fgaClient.Write(body);
}
```

***

## Phase 5: Performing Authorization Checks

### Authorization Check Flow

![openFGA Authorization Check Flow](../../images/openfga-authorization-check-flow.png)

### Basic Check Operations

The Check API answers "Can user U perform action A on object O?"[6][7][1]

**Python Example:**

```python
async def check_authorization(fga_client, user_id, relation, object_id):
    """
    Check if a user has a specific relation to an object
    """
    response = await fga_client.check({
        "user": f"user:{user_id}",
        "relation": relation,
        "object": object_id
    })
    
    return response.allowed

# Usage examples
can_view = await check_authorization(fga_client, "alice", "can_view", "resource:report-2024")
can_edit = await check_authorization(fga_client, "alice", "can_edit", "resource:report-2024")
can_delete = await check_authorization(fga_client, "bob", "can_delete", "resource:report-2024")

print(f"Alice can view: {can_view}")  # True
print(f"Alice can edit: {can_edit}")  # True (she's owner)
print(f"Bob can delete: {can_delete}") # False (not admin)
```

**C# Example:**

```csharp
public async Task<bool> CheckAuthorization(
    OpenFgaClient fgaClient, 
    string userId, 
    string relation, 
    string objectId)
{
    var response = await fgaClient.Check(new ClientCheckRequest
    {
        User = $"user:{userId}",
        Relation = relation,
        Object = objectId
    });

    return response.Allowed;
}

// Usage
var canView = await CheckAuthorization(fgaClient, "alice", "can_view", "resource:report-2024");
var canEdit = await CheckAuthorization(fgaClient, "alice", "can_edit", "resource:report-2024");
var canDelete = await CheckAuthorization(fgaClient, "bob", "can_delete", "resource:report-2024");

Console.WriteLine($"Alice can view: {canView}");
Console.WriteLine($"Alice can edit: {canEdit}");
Console.WriteLine($"Bob can delete: {canDelete}");
```


### List Objects API

The List Objects API returns all objects a user has a specific relation with:[1]

![openFGA List Objects API](../../images/openfga-list-objects.png)

**Python Example:**

```python
async def list_user_resources(fga_client, user_id, relation, object_type):
    """
    List all objects of a specific type that a user has access to
    """
    response = await fga_client.list_objects({
        "user": f"user:{user_id}",
        "relation": relation,
        "type": object_type
    })
    
    return response.objects

# Get all resources alice can view
viewable_resources = await list_user_resources(
    fga_client, 
    "alice", 
    "can_view", 
    "resource"
)
print(f"Alice can view: {viewable_resources}")
```

**C# Example:**

```csharp
public async Task<List<string>> ListUserResources(
    OpenFgaClient fgaClient,
    string userId,
    string relation,
    string objectType)
{
    var response = await fgaClient.ListObjects(new ClientListObjectsRequest
    {
        User = $"user:{userId}",
        Relation = relation,
        Type = objectType
    });

    return response.Objects;
}

// Get all resources alice can view
var viewableResources = await ListUserResources(
    fgaClient,
    "alice",
    "can_view",
    "resource"
);
Console.WriteLine($"Alice can view: {string.Join(", ", viewableResources)}");
```


***

## Phase 6: Advanced Modeling Patterns for CIAM

### Pattern 1: Hierarchical Organizations

Many CIAM applications need nested organization structures:[4][7]

![openFGA Hierarchical Organizations](../../images/openfga-hierarchical-organizations.png)

```fga
model
  schema 1.1

type user

type organization
  relations
    define parent: [organization]
    define member: [user, organization#member]
    define admin: [user] and member
    define member_or_from_parent: member or member from parent
```

**Use case**: Enterprise customers with departments, teams, and sub-teams.

### Pattern 2: Contextual Authorization

For scenarios requiring dynamic context (IP address, time, location):[14][4]

![openFGA Contextual Authorization](../../images/openfga-contextual-authorization.png)

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define user_in_context: [user]
    define active_member: member and user_in_context

type resource
  relations
    define org: [organization]
    define can_access: active_member from org
```

**Python Example with Contextual Tuples:**

```python
async def check_with_context(fga_client, user_id, resource_id, org_id):
    """
    Check authorization with contextual information
    """
    response = await fga_client.check({
        "user": f"user:{user_id}",
        "relation": "can_access",
        "object": f"resource:{resource_id}",
        "contextual_tuples": [
            {
                "user": f"user:{user_id}",
                "relation": "user_in_context",
                "object": f"organization:{org_id}"
            }
        ]
    })
    
    return response.allowed
```


### Pattern 3: Custom Roles

Allow users to create custom roles within their organization:[4]

![openFGA Custom Roles](../../images/openfga-custom-roles.png)

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define admin: [user] and member

type role
  relations
    define org: [organization]
    define assignee: [user]
    define permission: [user]

type resource
  relations
    define org: [organization]
    define can_access: [role#assignee]
```

### Pattern 4: Conditional Access

Implement conditions like IP restrictions or MFA requirements:[4]

```fga
model
  schema 1.1

type user

type resource
  relations
    define viewer: [user with mfa_verified]
    define editor: [user with ip_allowed]
    define can_view: viewer
    define can_edit: editor

condition mfa_verified(mfa_enabled: bool) {
  mfa_enabled == true
}

condition ip_allowed(ip_address: string) {
  ip_address == "192.168.1.0/24"
}
```

**Python Example with Conditions:**

```python
async def check_with_conditions(fga_client, user_id, resource_id):
    """
    Check with conditional attributes
    """
    response = await fga_client.check({
        "user": f"user:{user_id}",
        "relation": "can_view",
        "object": f"resource:{resource_id}",
        "context": {
            "mfa_enabled": True,
            "ip_address": "192.168.1.100"
        }
    })
    
    return response.allowed
```


***

## Phase 7: Integration Patterns for CIAM

### Authentication + Authorization Flow

![openFGA Authentication Authorization](../../images/openfga-authn-authz.png)

### Pattern 1: Integration with Authentication (OIDC/OAuth2)

OpenFGA handles authorization, while your IdP handles authentication:[15][16]

**Python Flask Example:**

```python
from flask import Flask, request, jsonify
from functools import wraps
import jwt
from openfga_sdk import OpenFgaClient

app = Flask(__name__)
fga_client = OpenFgaClient(configuration)

def require_auth(f):
    @wraps(f)
    async def decorated_function(*args, **kwargs):
        # Extract JWT token from Authorization header
        auth_header = request.headers.get('Authorization')
        if not auth_header:
            return jsonify({"error": "No authorization header"}), 401
        
        try:
            token = auth_header.split()[1]
            # Verify token with your IdP (Keycloak, Auth0, etc.)
            payload = jwt.decode(token, verify=False)  # Use proper verification
            user_id = payload['sub']
            
            # Store user_id for authorization checks
            request.user_id = user_id
            return await f(*args, **kwargs)
        except Exception as e:
            return jsonify({"error": "Invalid token"}), 401
    
    return decorated_function

def require_permission(relation, object_type):
    def decorator(f):
        @wraps(f)
        async def decorated_function(*args, **kwargs):
            user_id = request.user_id
            object_id = kwargs.get('resource_id')
            
            # Perform authorization check
            allowed = await fga_client.check({
                "user": f"user:{user_id}",
                "relation": relation,
                "object": f"{object_type}:{object_id}"
            })
            
            if not allowed.allowed:
                return jsonify({"error": "Forbidden"}), 403
            
            return await f(*args, **kwargs)
        
        return decorated_function
    return decorator

@app.route('/api/resources/<resource_id>', methods=['GET'])
@require_auth
@require_permission('can_view', 'resource')
async def get_resource(resource_id):
    # User is authenticated and authorized
    return jsonify({"resource_id": resource_id, "data": "..."})

@app.route('/api/resources/<resource_id>', methods=['PUT'])
@require_auth
@require_permission('can_edit', 'resource')
async def update_resource(resource_id):
    # User is authenticated and authorized
    return jsonify({"message": "Resource updated"})
```

**C# ASP.NET Core Example:**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;
using OpenFga.Sdk.Client;

[ApiController]
[Route("api/[controller]")]
public class ResourcesController : ControllerBase
{
    private readonly OpenFgaClient _fgaClient;
    
    public ResourcesController(OpenFgaClient fgaClient)
    {
        _fgaClient = fgaClient;
    }

    [HttpGet("{resourceId}")]
    [Authorize] // JWT authentication middleware
    public async Task<IActionResult> GetResource(string resourceId)
    {
        var userId = User.FindFirst("sub")?.Value;
        
        // Check authorization
        var allowed = await _fgaClient.Check(new ClientCheckRequest
        {
            User = $"user:{userId}",
            Relation = "can_view",
            Object = $"resource:{resourceId}"
        });

        if (!allowed.Allowed)
        {
            return Forbid();
        }

        // Return resource
        return Ok(new { resourceId, data = "..." });
    }

    [HttpPut("{resourceId}")]
    [Authorize]
    public async Task<IActionResult> UpdateResource(string resourceId, [FromBody] object data)
    {
        var userId = User.FindFirst("sub")?.Value;
        
        var allowed = await _fgaClient.Check(new ClientCheckRequest
        {
            User = $"user:{userId}",
            Relation = "can_edit",
            Object = $"resource:{resourceId}"
        });

        if (!allowed.Allowed)
        {
            return Forbid();
        }

        // Update resource
        return Ok(new { message = "Resource updated" });
    }
}
```


### Pattern 2: Authorization Middleware

Create reusable middleware for consistent authorization checks:[17]

**Python Example:**

```python
class OpenFGAMiddleware:
    def __init__(self, fga_client):
        self.fga_client = fga_client
    
    async def check_permission(self, user_id, relation, object_id):
        """
        Centralized permission checking
        """
        response = await self.fga_client.check({
            "user": f"user:{user_id}",
            "relation": relation,
            "object": object_id
        })
        return response.allowed
    
    async def batch_check(self, checks):
        """
        Perform multiple checks efficiently
        """
        results = []
        for check in checks:
            result = await self.check_permission(
                check['user_id'],
                check['relation'],
                check['object_id']
            )
            results.append(result)
        return results
    
    async def get_accessible_resources(self, user_id, relation, object_type):
        """
        Get all resources user can access
        """
        response = await self.fga_client.list_objects({
            "user": f"user:{user_id}",
            "relation": relation,
            "type": object_type
        })
        return response.objects
```

**C# Example:**

```csharp
public class OpenFGAMiddleware
{
    private readonly OpenFgaClient _fgaClient;

    public OpenFGAMiddleware(OpenFgaClient fgaClient)
    {
        _fgaClient = fgaClient;
    }

    public async Task<bool> CheckPermission(string userId, string relation, string objectId)
    {
        var response = await _fgaClient.Check(new ClientCheckRequest
        {
            User = $"user:{userId}",
            Relation = relation,
            Object = objectId
        });

        return response.Allowed;
    }

    public async Task<List<bool>> BatchCheck(List<(string userId, string relation, string objectId)> checks)
    {
        var results = new List<bool>();
        
        foreach (var (userId, relation, objectId) in checks)
        {
            var result = await CheckPermission(userId, relation, objectId);
            results.Add(result);
        }

        return results;
    }

    public async Task<List<string>> GetAccessibleResources(string userId, string relation, string objectType)
    {
        var response = await _fgaClient.ListObjects(new ClientListObjectsRequest
        {
            User = $"user:{userId}",
            Relation = relation,
            Type = objectType
        });

        return response.Objects;
    }
}
```

### Pattern 3: Event-Driven Relationship Management

Automatically sync relationships when domain events occur:

![openFGA Event-driven Relationship](../../images/openfga-event-driven-relationship.png)

**Python Example:**

```python
from typing import Dict, Any
import asyncio

class RelationshipManager:
    def __init__(self, fga_client):
        self.fga_client = fga_client
    
    async def handle_user_joined_org(self, event: Dict[str, Any]):
        """
        Handle UserJoinedOrganization event
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"user:{event['user_id']}",
                    "relation": "member",
                    "object": f"organization:{event['org_id']}"
                }
            ]
        })
    
    async def handle_user_left_org(self, event: Dict[str, Any]):
        """
        Handle UserLeftOrganization event
        """
        await self.fga_client.write({
            "deletes": [
                {
                    "user": f"user:{event['user_id']}",
                    "relation": "member",
                    "object": f"organization:{event['org_id']}"
                },
                {
                    "user": f"user:{event['user_id']}",
                    "relation": "admin",
                    "object": f"organization:{event['org_id']}"
                }
            ]
        })
    
    async def handle_resource_created(self, event: Dict[str, Any]):
        """
        Handle ResourceCreated event
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"user:{event['owner_id']}",
                    "relation": "owner",
                    "object": f"resource:{event['resource_id']}"
                },
                {
                    "user": f"organization:{event['org_id']}",
                    "relation": "org",
                    "object": f"resource:{event['resource_id']}"
                }
            ]
        })
    
    async def handle_resource_shared(self, event: Dict[str, Any]):
        """
        Handle ResourceShared event
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"user:{event['shared_with_user_id']}",
                    "relation": event['permission'],  # 'viewer' or 'editor'
                    "object": f"resource:{event['resource_id']}"
                }
            ]
        })
```

***

## Phase 8: Production Deployment Best Practices

### Production Architecture

![openFGA Production Architecture](../../images/openfga-production-architecture.png)

### Database Configuration

OpenFGA requires a persistent database for production:[3][5]

**PostgreSQL Setup:**

```yaml
# docker-compose.yml for production
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: openfga
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: openfga
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  openfga:
    image: openfga/openfga:latest
    depends_on:
      - postgres
    ports:
      - "8080:8080"
      - "8081:8081"
    environment:
      - OPENFGA_DATASTORE_ENGINE=postgres
      - OPENFGA_DATASTORE_URI=postgres://openfga:${DB_PASSWORD}@postgres:5432/openfga?sslmode=disable
      - OPENFGA_DATASTORE_MAX_OPEN_CONNS=100
      - OPENFGA_DATASTORE_MAX_IDLE_CONNS=100
      - OPENFGA_DATASTORE_CONN_MAX_IDLE_TIME=3600s
      - OPENFGA_LOG_FORMAT=json
      - OPENFGA_LOG_LEVEL=info
      - OPENFGA_PLAYGROUND_ENABLED=false
      - OPENFGA_METRICS_ENABLED=true
      - OPENFGA_DATASTORE_METRICS_ENABLED=true
      - OPENFGA_TRACE_ENABLED=true
      - OPENFGA_TRACE_SAMPLE_RATIO=0.3
      - OPENFGA_CHECK_QUERY_CACHE_ENABLED=true
      - OPENFGA_CHECK_QUERY_CACHE_LIMIT=10000
      - OPENFGA_CHECK_QUERY_CACHE_TTL=60s
    command: run --http-tls-enabled --grpc-tls-enabled

volumes:
  postgres_data:
```


### Performance Optimization

**Key Configuration Settings**:[3][5]

![openFGA Performance Optimization](../../images/openfga-performance-optimization.png)

1. **Enable Caching:**
   - Reduces latency but increases staleness
   - Recommended for read-heavy workloads

2. **Connection Pooling:**
   - Set `OPENFGA_DATASTORE_MAX_OPEN_CONNS` equal to DB max connections divided by number of instances
   - Set `OPENFGA_DATASTORE_MAX_IDLE_CONNS` to avoid reconnection overhead

3. **Cluster Recommendations:**
   - Use fewer servers with higher capacity for better cache hit ratios
   - Co-locate database in same datacenter to minimize latency

4. **Monitoring:**
   - Enable metrics collection
   - Enable tracing with low sample ratio (0.3)

**Python Client Optimization:**

```python
from openfga_sdk import OpenFgaClient, ClientConfiguration

# Configure client with optimal settings
configuration = ClientConfiguration(
    api_url=os.environ.get('FGA_API_URL'),
    store_id=os.environ.get('FGA_STORE_ID'),
    authorization_model_id=os.environ.get('FGA_MODEL_ID'),
    credentials={
        "method": "api_token",
        "config": {
            "token": os.environ.get('FGA_API_TOKEN')
        }
    }
)

# Use connection pooling
async with OpenFgaClient(configuration) as fga_client:
    # Batch operations when possible
    await fga_client.write_tuples([
        {"user": "user:1", "relation": "member", "object": "org:1"},
        {"user": "user:2", "relation": "member", "object": "org:1"},
        # ... more tuples
    ])
```


### Security Best Practices

1. **Enable Authentication**:[5]

```bash
# Generate API tokens
OPENFGA_AUTHN_METHOD=preshared
OPENFGA_AUTHN_PRESHARED_KEYS=key1,key2
```

2. **Enable TLS/HTTPS**:[5]

```bash
OPENFGA_HTTP_TLS_ENABLED=true
OPENFGA_HTTP_TLS_CERT=/path/to/cert.pem
OPENFGA_HTTP_TLS_KEY=/path/to/key.pem
```

3. **Disable Playground in Production**:[5]

```bash
OPENFGA_PLAYGROUND_ENABLED=false
```

### Migration Strategies

**Gradual Rollout Pattern**:[18][19]

![openFGA Migration Strategies](../../images/openfga-migration-strategies.png)

```python
class AuthorizationService:
    def __init__(self, fga_client, legacy_auth_service):
        self.fga_client = fga_client
        self.legacy_auth = legacy_auth_service
        self.migration_percentage = 0.0  # Start at 0%
    
    async def check_permission(self, user_id, action, resource_id):
        """
        Gradually migrate from legacy to OpenFGA
        """
        import random
        
        # Use OpenFGA for percentage of traffic
        if random.random() < self.migration_percentage:
            try:
                result = await self.fga_client.check({
                    "user": f"user:{user_id}",
                    "relation": action,
                    "object": f"resource:{resource_id}"
                })
                return result.allowed
            except Exception as e:
                # Fallback to legacy on error
                logger.error(f"OpenFGA check failed: {e}")
                return await self.legacy_auth.check(user_id, action, resource_id)
        else:
            # Use legacy system
            return await self.legacy_auth.check(user_id, action, resource_id)
    
    def increase_migration(self, percentage):
        """
        Gradually increase OpenFGA usage
        """
        self.migration_percentage = min(percentage, 1.0)
```


***

## Phase 9: Testing and Validation

### Model Testing with Assertions

Use assertions to prevent regressions:[7]

```yaml
# test-assertions.yaml
model: |
  model
    schema 1.1
  
  type user
  
  type organization
    relations
      define member: [user]
      define admin: [user] and member
  
  type resource
    relations
      define org: [organization]
      define owner: [user]
      define can_view: owner or member from org
      define can_edit: owner or admin from org
      define can_delete: admin from org

tuples:
  - user: user:alice
    relation: member
    object: organization:acme
  - user: user:alice
    relation: admin
    object: organization:acme
  - user: user:bob
    relation: member
    object: organization:acme
  - user: organization:acme
    relation: org
    object: resource:doc1
  - user: user:alice
    relation: owner
    object: resource:doc1

tests:
  - name: Owner can view their resource
    check:
      - user: user:alice
        object: resource:doc1
        assertions:
          can_view: true
          can_edit: true
          can_delete: true
  
  - name: Member can view org resources
    check:
      - user: user:bob
        object: resource:doc1
        assertions:
          can_view: true
          can_edit: false
          can_delete: false
  
  - name: Admin can delete org resources
    check:
      - user: user:alice
        object: resource:doc1
        assertions:
          can_delete: true
```

**Run tests with FGA CLI:**

```bash
fga model test --file test-assertions.yaml
```


### Testing Workflow

![openFGA Testing Workflow](../../images/openfga-testing-workflow.png)

### Integration Testing

**Python Integration Test Example:**

```python
import pytest
import asyncio
from openfga_sdk import OpenFgaClient, ClientConfiguration

@pytest.fixture
async def fga_client():
    """
    Setup test OpenFGA client
    """
    configuration = ClientConfiguration(
        api_url="http://localhost:8080"
    )
    
    async with OpenFgaClient(configuration) as client:
        # Create test store
        store = await client.create_store({"name": "test-store"})
        client.store_id = store.id
        
        # Write test model
        # ... (model definition)
        
        yield client

@pytest.mark.asyncio
async def test_user_can_view_org_resource(fga_client):
    """
    Test that organization members can view org resources
    """
    # Setup: Create relationships
    await fga_client.write({
        "writes": [
            {"user": "user:alice", "relation": "member", "object": "organization:acme"},
            {"user": "organization:acme", "relation": "org", "object": "resource:doc1"}
        ]
    })
    
    # Act: Check permission
    result = await fga_client.check({
        "user": "user:alice",
        "relation": "can_view",
        "object": "resource:doc1"
    })
    
    # Assert
    assert result.allowed == True

@pytest.mark.asyncio
async def test_non_member_cannot_view_org_resource(fga_client):
    """
    Test that non-members cannot view org resources
    """
    # Setup: Create resource without user membership
    await fga_client.write({
        "writes": [
            {"user": "organization:acme", "relation": "org", "object": "resource:doc1"}
        ]
    })
    
    # Act: Check permission
    result = await fga_client.check({
        "user": "user:bob",
        "relation": "can_view",
        "object": "resource:doc1"
    })
    
    # Assert
    assert result.allowed == False
```

***

## Phase 10: Advanced Topics

### Monitoring and Observability

**Key Metrics to Monitor**:[3][5]

![openFGA Monitoring](../../images/openfga-monitoring.png)

1. **Latency Metrics:**
   - P50, P95, P99 response times
   - Check API latency
   - List Objects API latency

2. **Throughput Metrics:**
   - Requests per second
   - Cache hit ratio
   - Database connection usage

3. **Error Metrics:**
   - Authorization check failures
   - Database connection errors
   - Timeout errors

**Python Monitoring Example:**

```python
import time
import logging
from prometheus_client import Counter, Histogram

# Define metrics
check_latency = Histogram(
    'openfga_check_latency_seconds',
    'Latency of OpenFGA check operations'
)

check_total = Counter(
    'openfga_check_total',
    'Total number of OpenFGA checks',
    ['result']
)

async def monitored_check(fga_client, user_id, relation, object_id):
    """
    Perform authorization check with monitoring
    """
    start_time = time.time()
    
    try:
        response = await fga_client.check({
            "user": f"user:{user_id}",
            "relation": relation,
            "object": object_id
        })
        
        check_total.labels(result='allowed' if response.allowed else 'denied').inc()
        return response.allowed
        
    except Exception as e:
        check_total.labels(result='error').inc()
        logging.error(f"Authorization check failed: {e}")
        raise
    
    finally:
        duration = time.time() - start_time
        check_latency.observe(duration)
```

### Model Versioning

OpenFGA models are immutable - each change creates a new version:[1]
![openFGA Model Versioning](../../images/openfga-model-versioning.png)

**Python Model Migration Example:**

```python
async def migrate_authorization_model(fga_client, new_model):
    """
    Deploy a new authorization model version
    """
    # Write new model
    response = await fga_client.write_authorization_model(new_model)
    new_model_id = response.authorization_model_id
    
    print(f"Created new model version: {new_model_id}")
    
    # Update application configuration to use new model
    # Both old and new models can coexist during migration
    
    return new_model_id

async def dual_check_during_migration(fga_client, old_model_id, new_model_id, check_request):
    """
    Check against both models during migration to validate
    """
    # Check with old model
    fga_client.authorization_model_id = old_model_id
    old_result = await fga_client.check(check_request)
    
    # Check with new model
    fga_client.authorization_model_id = new_model_id
    new_result = await fga_client.check(check_request)
    
    # Log discrepancies
    if old_result.allowed != new_result.allowed:
        logging.warning(f"Model migration discrepancy detected: {check_request}")
    
    # Use new model result
    return new_result.allowed
```


### Scaling Considerations

**Horizontal Scaling Strategy**:[3][5]

```yaml
# Kubernetes deployment example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openfga
spec:
  replicas: 3  # Start with 3 replicas
  selector:
    matchLabels:
      app: openfga
  template:
    metadata:
      labels:
        app: openfga
    spec:
      containers:
      - name: openfga
        image: openfga/openfga:latest
        ports:
        - containerPort: 8080
        - containerPort: 8081
        env:
        - name: OPENFGA_DATASTORE_ENGINE
          value: "postgres"
        - name: OPENFGA_DATASTORE_URI
          valueFrom:
            secretKeyRef:
              name: openfga-db
              key: uri
        - name: OPENFGA_DATASTORE_MAX_OPEN_CONNS
          value: "30"  # 100 DB connections / 3 replicas
        - name: OPENFGA_CHECK_QUERY_CACHE_ENABLED
          value: "true"
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: openfga-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: openfga
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

***

## Phase 11: Real-World CIAM Use Cases

### Use Case 1: Multi-Tenant SaaS Platform

**Requirements:**
- Organizations can have multiple workspaces
- Users can belong to multiple organizations
- Permissions cascade from workspace to resources
- Support for custom roles per organization

![openFGA SaaS](../../images/openfga-saas.png)

**Model:**

```fga
model
  schema 1.1

type user

type organization
  relations
    define member: [user]
    define admin: [user] and member
    define owner: [user] and admin

type workspace
  relations
    define org: [organization]
    define member: [user, organization#member]
    define admin: [user, organization#admin]
    define owner: owner from org
    define can_manage: admin or owner

type resource
  relations
    define workspace: [workspace]
    define owner: [user]
    define editor: [user]
    define viewer: [user, organization#member]
    define can_view: viewer or editor or owner or member from workspace
    define can_edit: editor or owner or admin from workspace
    define can_delete: owner or admin from workspace
    define can_share: can_edit
```

**Implementation:**

```python
class TenantManager:
    def __init__(self, fga_client):
        self.fga_client = fga_client
    
    async def create_organization(self, org_id, owner_id):
        """
        Create a new organization with owner
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"user:{owner_id}",
                    "relation": "owner",
                    "object": f"organization:{org_id}"
                }
            ]
        })
    
    async def create_workspace(self, workspace_id, org_id, creator_id):
        """
        Create a workspace within an organization
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"organization:{org_id}",
                    "relation": "org",
                    "object": f"workspace:{workspace_id}"
                },
                {
                    "user": f"user:{creator_id}",
                    "relation": "admin",
                    "object": f"workspace:{workspace_id}"
                }
            ]
        })
    
    async def invite_user_to_org(self, user_id, org_id, role="member"):
        """
        Invite user to organization
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"user:{user_id}",
                    "relation": role,
                    "object": f"organization:{org_id}"
                }
            ]
        })
    
    async def get_user_organizations(self, user_id):
        """
        Get all organizations user belongs to
        """
        response = await self.fga_client.list_objects({
            "user": f"user:{user_id}",
            "relation": "member",
            "type": "organization"
        })
        return response.objects
```

### Use Case 2: Healthcare Platform with HIPAA Compliance

**Requirements:**
- Patient data access requires explicit consent
- Doctors can access patient records they're treating
- Audit trail for all access
- Emergency access for on-call doctors

![openFGA HIPPA](../../images/openfga-hipaa.png)

**Model:**

```fga
model
  schema 1.1

type user

type patient
  relations
    define treating_physician: [user]
    define consented_viewer: [user]
    define emergency_contact: [user]

type medical_record
  relations
    define patient: [patient]
    define can_view: treating_physician from patient or consented_viewer from patient
    define can_edit: treating_physician from patient
    define emergency_access: [user with on_call]

condition on_call(is_on_call: bool, current_time: timestamp) {
  is_on_call == true
}
```

**Implementation with Audit:**

```python
import logging
from datetime import datetime

class AuditedAuthorizationService:
    def __init__(self, fga_client, audit_logger):
        self.fga_client = fga_client
        self.audit_logger = audit_logger
    
    async def check_medical_record_access(
        self, 
        user_id, 
        record_id, 
        action,
        reason=None
    ):
        """
        Check access with audit logging
        """
        start_time = datetime.utcnow()
        
        # Perform authorization check
        result = await self.fga_client.check({
            "user": f"user:{user_id}",
            "relation": action,
            "object": f"medical_record:{record_id}"
        })
        
        # Log access attempt
        self.audit_logger.log({
            "timestamp": start_time.isoformat(),
            "user_id": user_id,
            "resource_type": "medical_record",
            "resource_id": record_id,
            "action": action,
            "allowed": result.allowed,
            "reason": reason,
            "ip_address": "...",  # Get from request context
        })
        
        return result.allowed
    
    async def grant_treatment_access(self, physician_id, patient_id):
        """
        Grant physician access to patient records
        """
        await self.fga_client.write({
            "writes": [
                {
                    "user": f"user:{physician_id}",
                    "relation": "treating_physician",
                    "object": f"patient:{patient_id}"
                }
            ]
        })
        
        # Log permission grant
        self.audit_logger.log({
            "timestamp": datetime.utcnow().isoformat(),
            "action": "GRANT_ACCESS",
            "physician_id": physician_id,
            "patient_id": patient_id,
            "access_type": "treating_physician"
        })
```

### Use Case 3: Enterprise B2B Platform with Partner Access

**Requirements:**
- Partners can access shared resources
- Different partner tiers with different permissions
- Time-limited access grants
- Partner admins can manage their users

![openFGA Partner Access](../../images/openfga-pa.png)

**Model:**

```fga
model
  schema 1.1

type user

type partner_org
  relations
    define member: [user]
    define admin: [user] and member
    define tier: [tier]

type tier
  relations
    define subscriber: [partner_org]

type shared_resource
  relations
    define owner_org: [organization]
    define shared_with: [partner_org]
    define can_view: member from shared_with or tier_gold_access
    define can_edit: admin from shared_with and tier_premium_access
    define tier_gold_access: subscriber from tier:gold
    define tier_premium_access: subscriber from tier:premium
```

***

## Phase 12: Troubleshooting and Common Patterns

### Common Issues and Solutions
![openFGA Troubleshooting](../../images/openfga-troubleshooting.png)


**Issue 1: Slow Authorization Checks**

**Solutions:**
1. Enable caching:[5]
   ```bash
   OPENFGA_CHECK_QUERY_CACHE_ENABLED=true
   OPENFGA_CHECK_QUERY_CACHE_LIMIT=10000
   OPENFGA_CHECK_QUERY_CACHE_TTL=60s
   ```

2. Optimize model depth:
   ```fga
   # Bad: Deep nesting
   define can_edit: admin from team from department from organization
   
   # Good: Flatter structure
   define can_edit: org_admin from organization or owner
   ```

3. Use direct relationships when possible

**Issue 2: Relationship Tuple Explosion**

**Solution:** Use group relationships instead of individual grants:

```python
# Bad: Individual grants
for user in users:
    await fga_client.write({
        "writes": [{"user": f"user:{user}", "relation": "viewer", "object": "resource:1"}]
    })

# Good: Group grant
await fga_client.write({
    "writes": [{"user": "team:engineering#member", "relation": "viewer", "object": "resource:1"}]
})
```

**Issue 3: Inconsistent Authorization State**

**Solution:** Implement transactional write patterns:

```python
async def atomic_resource_creation(fga_client, resource_id, owner_id, org_id):
    """
    Ensure all relationships are created atomically
    """
    try:
        # Write all relationships in single call
        await fga_client.write({
            "writes": [
                {"user": f"user:{owner_id}", "relation": "owner", "object": f"resource:{resource_id}"},
                {"user": f"organization:{org_id}", "relation": "org", "object": f"resource:{resource_id}"}
            ]
        })
    except Exception as e:
        # Rollback application state if OpenFGA write fails
        logging.error(f"Failed to create resource relationships: {e}")
        raise
```

***

## Phase 13: Next Steps and Continued Learning

### Learning Path
![openFGA Learning Path](../../images/openfga-learning-path.png)

### Learning Resources

1. **Official Documentation**:[1]
   - https://openfga.dev/docs
   - Modeling guides and examples
   - API reference

2. **Sample Applications**:[4]
   - Sample repository with real-world examples
   - Community tutorials and patterns

3. **Community Support**:
   - GitHub Discussions
   - OpenFGA Slack community
   - Stack Overflow

### Practice Exercises

**Exercise 1:** Model your current application's authorization
- Write requirements in plain language
- Create the OpenFGA model
- Test with assertions

**Exercise 2:** Implement gradual migration
- Set up parallel authorization checks
- Compare results between systems
- Gradually increase OpenFGA traffic

**Exercise 3:** Performance testing
- Load test with realistic data volumes
- Measure and optimize latency
- Configure caching appropriately

### Advanced Topics to Explore

1. **Multi-region deployment**[5]
2. **Advanced caching strategies**[3][5]
3. **Custom authorization DSLs**[7]
4. **Integration with service mesh**
5. **GraphQL authorization patterns**

---

## Conclusion

You now have a comprehensive understanding of OpenFGA from basic concepts to advanced implementation patterns. The key to mastery is:
![openFGA Mind Map](../../images/openfga-mindmap.png)

1. **Start simple** - Begin with core use cases
2. **Iterate** - Continuously refine your model as requirements evolve[7]
3. **Test thoroughly** - Use assertions and integration tests[20][7]
4. **Monitor** - Track performance and errors in production[3][5]
5. **Optimize gradually** - Don't over-engineer initially

OpenFGA provides the flexibility and performance needed for modern CIAM applications while maintaining the simplicity of relationship-based authorization. With practice and the patterns outlined in this guide, you'll be able to implement sophisticated authorization systems that scale with your organization's needs.hat scale with your organization's needs.[12][13][11]

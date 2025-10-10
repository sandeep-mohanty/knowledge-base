# SpiceDB: A Comprehensive Tutorial

## Table of Contents

1. [Introduction to SpiceDB](#introduction-to-spicedb)
2. [Core Concepts](#core-concepts)
3. [Schema Definition Language](#schema-definition-language)
4. [Designing Authorization Models](#designing-authorization-models)
5. [API and Client Libraries](#api-and-client-libraries)
6. [Practical Examples](#practical-examples)
   - [Document Management System](#document-management-system)
   - [Team Collaboration Platform](#team-collaboration-platform)
   - [Healthcare Application](#healthcare-application)
   - [Multi-tenant SaaS Platform](#multi-tenant-saas-platform)
7. [Advanced Concepts](#advanced-concepts)
8. [Deployment Options](#deployment-options)
9. [Migration Strategies](#migration-strategies)
10. [Performance Optimization](#performance-optimization)
11. [Security Considerations](#security-considerations)
12. [Best Practices](#best-practices)

## Introduction to SpiceDB

SpiceDB is an open-source, Zanzibar-inspired database for managing application permissions. Developed by Authzed, SpiceDB implements the relationship-based access control model described in Google's Zanzibar paper while adding features to make it more accessible and practical for adoption by development teams.

Key features of SpiceDB include:

1. **Schema-driven**: Uses a clear schema language to define permission models
2. **Flexible relationship modeling**: Supports complex authorization scenarios
3. **Transactional consistency**: Provides strong consistency guarantees
4. **High availability**: Designed for mission-critical applications
5. **Caching**: Built-in caching for improved performance
6. **Open source**: Available under the Apache 2.0 license
7. **Developer tooling**: Includes playground, visualization tools, and validation

SpiceDB is particularly well-suited for applications that require:
- Complex permission structures
- Hierarchical resource management
- Role-based and relationship-based access control
- Cross-organizational permissions
- Audit trail capabilities

## Core Concepts

### Object Definitions

Object definitions in SpiceDB describe the different types of resources in your system, such as:
- Users (users, groups, services)
- Resources (documents, folders, projects)
- Organizational entities (teams, departments)

### Relations

Relations define the ways that subjects can connect to objects. For example:
- `viewer`: Can view a document
- `member`: Is a member of a group
- `owner`: Owns a resource

### Permissions

Permissions in SpiceDB define the access rights derived from relations. They combine multiple relations to form authorization rules.

### Relationships

Relationships (or tuples) are the core data structure in SpiceDB, connecting subjects and objects through relations. A relationship has:
- **Resource**: The object being accessed
- **Relation**: The type of relationship
- **Subject**: The entity accessing the resource

Example relationship: `document:budget#viewer@user:anne`

### Computed Relationships

SpiceDB allows for complex relationship derivation using its permission system. For example:
- "Anyone who can edit can also view"
- "Members of a team that owns a document can edit it"

### Consistency

SpiceDB provides strong consistency guarantees, ensuring that permission changes are atomic and visible globally.

## Schema Definition Language

SpiceDB uses a declarative schema definition language to model authorization rules. Let's explore its syntax and capabilities:

### Basic Schema Structure

```
definition user {}

definition document {
    relation viewer: user
    relation editor: user
    relation owner: user
}
```

This defines two object types: `user` and `document`. The `document` type has three relations: `viewer`, `editor`, and `owner`.

### Permissions

Permissions combine relations to define access rights:

```
definition document {
    relation viewer: user
    relation editor: user
    relation owner: user
    
    permission view = viewer + editor + owner
    permission edit = editor + owner
    permission delete = owner
}
```

The `+` operator means "or", so `view` permission is granted to users with either `viewer`, `editor`, or `owner` relation.

### Relation References

Relations can refer to other objects' relations:

```
definition folder {
    relation viewer: user
    relation editor: user
    relation owner: user
    
    permission view = viewer + editor + owner
    permission edit = editor + owner
    permission delete = owner
}

definition document {
    relation parent: folder
    relation viewer: user
    relation editor: user
    relation owner: user
    
    permission view = viewer + editor + owner + parent->view
    permission edit = editor + owner + parent->edit
    permission delete = owner + parent->delete
}
```

The arrow operator (`->`) lets you reference permissions from another relationship. In this example, users who can view a folder can also view documents within it.

### Union and Intersection

SpiceDB's schema language supports both union (`+`) and intersection (`&`) operators:

```
definition resource {
    relation reader: user
    relation writer: user
    relation organization: organization
    
    // Union: reader OR writer
    permission view = reader + writer
    
    // Intersection: writer AND organization admin
    permission delete = writer & organization->admin
}

definition organization {
    relation admin: user
}
```

### Exclusion

The exclusion operator (`-`) lets you exclude certain relations:

```
definition document {
    relation viewer: user
    relation blocked: user
    
    // All viewers except blocked users
    permission view = viewer - blocked
}
```

### Wildcards

SpiceDB supports wildcards using the ellipsis (`...`) operator:

```
definition document {
    relation viewer: user | group#member
    relation editor: user
    relation owner: user
}

definition group {
    relation member: user | group#...
}
```

The `...` operator allows for recursive references, enabling group nesting in this example.

## Designing Authorization Models

Let's walk through the process of designing an authorization model in SpiceDB:

### 1. Identify Object Types

First, identify the different types of objects in your system:
- Users and groups
- Resources (documents, projects, etc.)
- Organizational units (teams, departments)

### 2. Define Relations

For each object type, define the possible relations:

```
definition user {}

definition group {
    relation member: user | group#member
    relation admin: user
}

definition document {
    relation viewer: user | group#member
    relation editor: user | group#member
    relation owner: user
}
```

### 3. Define Permissions

Add permissions to represent derived access rights:

```
definition document {
    relation viewer: user | group#member
    relation editor: user | group#member
    relation owner: user
    
    permission view = viewer + editor + owner
    permission edit = editor + owner
    permission manage = owner
}
```

### 4. Add Hierarchical Relationships

For complex systems, add parent-child relationships:

```
definition folder {
    relation viewer: user | group#member
    relation editor: user | group#member
    relation owner: user
    
    permission view = viewer + editor + owner
    permission edit = editor + owner
    permission manage = owner
}

definition document {
    relation parent: folder
    relation viewer: user | group#member
    relation editor: user | group#member
    relation owner: user
    
    permission view = viewer + editor + owner + parent->view
    permission edit = editor + owner + parent->edit
    permission manage = owner + parent->manage
}
```

### 5. Test and Refine

Use SpiceDB's playground and validation tools to test and refine your model with sample relationships and authorization checks.

## API and Client Libraries

SpiceDB provides a gRPC API and multiple client libraries for integration:

### Core API Functions

1. **Schema Management**
   - Read and write schema definitions

2. **Relationship Management**
   - Write, read, delete relationships

3. **Permission Checks**
   - Check if a subject has a permission for a resource
   
4. **Relationship Queries**
   - LookupResources: Find resources a subject has access to
   - LookupSubjects: Find subjects that have access to a resource

### C# Client Library Example

First, install the SpiceDB C# client package:

```bash
dotnet add package Authzed.Api
```

Basic usage:

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Authzed.Api.V1;
using Authzed.Api.V1.Client;
using Grpc.Core;
using Grpc.Net.Client;

public class SpiceDbAuthorizationService
{
    private readonly PermissionsService.PermissionsServiceClient _client;

    public SpiceDbAuthorizationService(string endpoint, string token)
    {
        // Configure channel with authentication
        var channel = GrpcChannel.ForAddress(endpoint, new GrpcChannelOptions
        {
            Credentials = ChannelCredentials.Create(
                ChannelCredentials.SecureSsl,
                CallCredentials.FromInterceptor((context, metadata) =>
                {
                    metadata.Add("Authorization", $"Bearer {token}");
                    return Task.CompletedTask;
                })
            )
        });
        
        _client = new PermissionsService.PermissionsServiceClient(channel);
    }
    
    // Check if a user has permission on a resource
    public async Task<bool> CheckPermissionAsync(
        string resourceType, 
        string resourceId, 
        string permission, 
        string subjectType, 
        string subjectId)
    {
        try
        {
            var request = new CheckPermissionRequest
            {
                Resource = new ObjectReference
                {
                    ObjectType = resourceType,
                    ObjectId = resourceId
                },
                Permission = permission,
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = subjectType,
                        ObjectId = subjectId
                    }
                }
            };
            
            var response = await _client.CheckPermissionAsync(request);
            return response.Permissionship == CheckPermissionResponse.Types.Permissionship.HasPermission;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error checking permission: {ex.Message}");
            return false;
        }
    }
    
    // Write a relationship
    public async Task WriteRelationshipAsync(
        string resourceType, 
        string resourceId, 
        string relation, 
        string subjectType, 
        string subjectId)
    {
        try
        {
            var request = new WriteRelationshipsRequest
            {
                Updates = 
                {
                    new RelationshipUpdate
                    {
                        Operation = RelationshipUpdate.Types.Operation.Create,
                        Relationship = new Relationship
                        {
                            Resource = new ObjectReference
                            {
                                ObjectType = resourceType,
                                ObjectId = resourceId
                            },
                            Relation = relation,
                            Subject = new SubjectReference
                            {
                                Object = new ObjectReference
                                {
                                    ObjectType = subjectType,
                                    ObjectId = subjectId
                                }
                            }
                        }
                    }
                }
            };
            
            await _client.WriteRelationshipsAsync(request);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error writing relationship: {ex.Message}");
            throw;
        }
    }
    
    // Delete a relationship
    public async Task DeleteRelationshipAsync(
        string resourceType, 
        string resourceId, 
        string relation, 
        string subjectType, 
        string subjectId)
    {
        try
        {
            var request = new WriteRelationshipsRequest
            {
                Updates = 
                {
                    new RelationshipUpdate
                    {
                        Operation = RelationshipUpdate.Types.Operation.Delete,
                        Relationship = new Relationship
                        {
                            Resource = new ObjectReference
                            {
                                ObjectType = resourceType,
                                ObjectId = resourceId
                            },
                            Relation = relation,
                            Subject = new SubjectReference
                            {
                                Object = new ObjectReference
                                {
                                    ObjectType = subjectType,
                                    ObjectId = subjectId
                                }
                            }
                        }
                    }
                }
            };
            
            await _client.WriteRelationshipsAsync(request);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error deleting relationship: {ex.Message}");
            throw;
        }
    }
    
    // Find resources a user has access to
    public async Task<List<string>> FindResourcesUserCanAccessAsync(
        string resourceType, 
        string permission, 
        string userType, 
        string userId)
    {
        try
        {
            var request = new LookupResourcesRequest
            {
                ResourceObjectType = resourceType,
                Permission = permission,
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = userType,
                        ObjectId = userId
                    }
                }
            };
            
            var results = new List<string>();
            using var call = _client.LookupResources(request);
            
            await foreach (var response in call.ResponseStream.ReadAllAsync())
            {
                results.Add(response.ResourceObjectId);
            }
            
            return results;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error finding resources: {ex.Message}");
            return new List<string>();
        }
    }
}
```

### Using the Authorization Service

```csharp
// Example usage
public class Program
{
    public static async Task Main()
    {
        var authService = new SpiceDbAuthorizationService(
            "spicedb.example.com:443", 
            "your-api-token"
        );
        
        // Check if a user can view a document
        bool canView = await authService.CheckPermissionAsync(
            "document", "budget2023", "view", "user", "anne"
        );
        
        Console.WriteLine($"Can Anne view the budget document? {canView}");
        
        // Grant editor access to a user
        await authService.WriteRelationshipAsync(
            "document", "budget2023", "editor", "user", "bob"
        );
        
        // Find all documents a user can view
        var viewableDocuments = await authService.FindResourcesUserCanAccessAsync(
            "document", "view", "user", "anne"
        );
        
        Console.WriteLine($"Anne can view {viewableDocuments.Count} documents:");
        foreach (var docId in viewableDocuments)
        {
            Console.WriteLine($"- {docId}");
        }
    }
}
```

## Practical Examples

Let's explore practical examples of SpiceDB models for different application domains:

### Document Management System

This example shows how to model a document management system similar to Google Docs:

#### Schema

```
definition user {}

definition group {
    relation member: user | group#member
    relation manager: user
    
    permission manage = manager
}

definition folder {
    relation parent: folder
    relation owner: user | group#member
    relation editor: user | group#member
    relation viewer: user | group#member
    
    permission view = viewer + editor + owner + parent->view
    permission edit = editor + owner + parent->edit
    permission share = owner + parent->share
    permission delete = owner + parent->delete
}

definition document {
    relation parent: folder
    relation owner: user | group#member
    relation editor: user | group#member
    relation viewer: user | group#member
    relation commenter: user | group#member
    
    permission view = viewer + commenter + editor + owner + parent->view
    permission comment = commenter + editor + owner
    permission edit = editor + owner + parent->edit
    permission share = owner + parent->share
    permission delete = owner + parent->delete
}

definition share_link {
    relation document: document
    relation access_level: share_level
    
    permission use = document->view
}

definition share_level {
    relation public: share_link
    relation view_only: share_link
    relation comment: share_link
    relation edit: share_link
}
```

#### Sample Relationships

```csharp
// Direct user assignments
await authService.WriteRelationshipAsync("folder", "team_documents", "owner", "user", "anne");
await authService.WriteRelationshipAsync("folder", "team_documents", "editor", "user", "bob");
await authService.WriteRelationshipAsync("folder", "team_documents", "viewer", "user", "charlie");

// Document in folder
await authService.WriteRelationshipAsync("document", "budget_2023", "parent", "folder", "team_documents");
await authService.WriteRelationshipAsync("document", "budget_2023", "editor", "user", "anne");

// Group membership
await authService.WriteRelationshipAsync("group", "finance", "member", "user", "dave");
await authService.WriteRelationshipAsync("document", "budget_2023", "viewer", "group", "finance");

// Share link
await authService.WriteRelationshipAsync("share_link", "budget_view", "document", "document", "budget_2023");
await authService.WriteRelationshipAsync("share_level", "view_only", "public", "share_link", "budget_view");
```

#### Check Examples

```csharp
// Can Bob edit the budget document?
bool canBobEdit = await authService.CheckPermissionAsync(
    "document", "budget_2023", "edit", "user", "bob"
);
// Result: true (because Bob is an editor of the parent folder)

// Can Dave view the budget document?
bool canDaveView = await authService.CheckPermissionAsync(
    "document", "budget_2023", "view", "user", "dave"
);
// Result: true (because Dave is a member of the finance group)

// List all documents Anne can view
var annesDocuments = await authService.FindResourcesUserCanAccessAsync(
    "document", "view", "user", "anne"
);
// Will include budget_2023 and any other accessible documents
```

### Team Collaboration Platform

This example demonstrates modeling a team collaboration platform similar to Slack:

#### Schema

```
definition user {}

definition organization {
    relation owner: user
    relation admin: user
    relation member: user
    
    permission admin = owner + admin
    permission view = owner + admin + member
}

definition team {
    relation organization: organization
    relation lead: user
    relation member: user
    
    permission view = member + lead + organization->admin
    permission manage = lead + organization->admin
}

definition channel {
    relation team: team
    relation creator: user
    relation member: user | team#member
    relation private: user
    
    permission view = creator + member + team->manage - private
    permission post = creator + member + team->manage
    permission manage = creator + team->manage
}

definition message {
    relation channel: channel
    relation author: user
    
    permission view = author + channel->view
    permission edit = author + channel->manage
    permission delete = author + channel->manage
}
```

#### Sample Relationships

```csharp
// Organization structure
await authService.WriteRelationshipAsync("organization", "acme", "owner", "user", "founder");
await authService.WriteRelationshipAsync("organization", "acme", "admin", "user", "admin1");
await authService.WriteRelationshipAsync("organization", "acme", "member", "user", "employee1");
await authService.WriteRelationshipAsync("organization", "acme", "member", "user", "employee2");

// Team structure
await authService.WriteRelationshipAsync("team", "engineering", "organization", "organization", "acme");
await authService.WriteRelationshipAsync("team", "engineering", "lead", "user", "lead_dev");
await authService.WriteRelationshipAsync("team", "engineering", "member", "user", "employee1");
await authService.WriteRelationshipAsync("team", "engineering", "member", "user", "employee2");

// Channels
await authService.WriteRelationshipAsync("channel", "general", "team", "team", "engineering");
await authService.WriteRelationshipAsync("channel", "general", "creator", "user", "lead_dev");

// Private channel
await authService.WriteRelationshipAsync("channel", "project-x", "team", "team", "engineering");
await authService.WriteRelationshipAsync("channel", "project-x", "creator", "user", "lead_dev");
await authService.WriteRelationshipAsync("channel", "project-x", "member", "user", "employee1");
await authService.WriteRelationshipAsync("channel", "project-x", "private", "user", "employee2");

// Messages
await authService.WriteRelationshipAsync("message", "msg1", "channel", "channel", "general");
await authService.WriteRelationshipAsync("message", "msg1", "author", "user", "employee1");
```

#### Check Examples

```csharp
// Can employee1 view the general channel?
bool canViewGeneral = await authService.CheckPermissionAsync(
    "channel", "general", "view", "user", "employee1"
);
// Result: true (team member)

// Can employee2 view the project-x channel?
bool canViewProjectX = await authService.CheckPermissionAsync(
    "channel", "project-x", "view", "user", "employee2"
);
// Result: false (excluded by private relation)

// Can employee1 edit their own message?
bool canEditMessage = await authService.CheckPermissionAsync(
    "message", "msg1", "edit", "user", "employee1"
);
// Result: true (author of the message)
```

### Healthcare Application

This example shows a model for a healthcare application with strict privacy controls:

#### Schema

```
definition user {}

definition hospital {
    relation administrator: user
    relation staff: user
    
    permission access = administrator + staff
}

definition department {
    relation hospital: hospital
    relation head: user
    relation staff: user
    
    permission access = head + staff + hospital->administrator
    permission manage = head + hospital->administrator
}

definition patient {
    relation self: user
    relation primary_doctor: user
    relation department: department
    relation care_team: user
    
    permission view_demographics = self + primary_doctor + care_team + department->staff
    permission view_medical = self + primary_doctor + care_team
    permission update_medical = primary_doctor + care_team
    permission manage = primary_doctor + department->head
}

definition medical_record {
    relation patient: patient
    relation author: user
    
    permission view = author + patient->view_medical
    permission edit = author & patient->update_medical
}

definition appointment {
    relation patient: patient
    relation provider: user
    
    permission view = provider + patient->view_demographics
    permission schedule = provider + patient->primary_doctor
    permission cancel = provider + patient->self
}
```

#### Sample Relationships

```csharp
// Hospital structure
await authService.WriteRelationshipAsync("hospital", "general", "administrator", "user", "admin");
await authService.WriteRelationshipAsync("hospital", "general", "staff", "user", "receptionist");

// Department structure
await authService.WriteRelationshipAsync("department", "cardiology", "hospital", "hospital", "general");
await authService.WriteRelationshipAsync("department", "cardiology", "head", "user", "dr_heart");
await authService.WriteRelationshipAsync("department", "cardiology", "staff", "user", "nurse1");
await authService.WriteRelationshipAsync("department", "cardiology", "staff", "user", "nurse2");

// Patient relationships
await authService.WriteRelationshipAsync("patient", "john_doe", "self", "user", "john");
await authService.WriteRelationshipAsync("patient", "john_doe", "primary_doctor", "user", "dr_heart");
await authService.WriteRelationshipAsync("patient", "john_doe", "department", "department", "cardiology");
await authService.WriteRelationshipAsync("patient", "john_doe", "care_team", "user", "nurse1");

// Medical records
await authService.WriteRelationshipAsync("medical_record", "heart_exam", "patient", "patient", "john_doe");
await authService.WriteRelationshipAsync("medical_record", "heart_exam", "author", "user", "dr_heart");

// Appointments
await authService.WriteRelationshipAsync("appointment", "checkup_may15", "patient", "patient", "john_doe");
await authService.WriteRelationshipAsync("appointment", "checkup_may15", "provider", "user", "dr_heart");
```

#### Check Examples

```csharp
// Can Dr. Heart view the patient's medical record?
bool canViewRecord = await authService.CheckPermissionAsync(
    "medical_record", "heart_exam", "view", "user", "dr_heart"
);
// Result: true (as author and primary doctor)

// Can Nurse1 view the patient's medical record?
bool canNurseViewRecord = await authService.CheckPermissionAsync(
    "medical_record", "heart_exam", "view", "user", "nurse1"
);
// Result: true (as care team member)

// Can Nurse2 view the patient's medical record?
bool canNurse2ViewRecord = await authService.CheckPermissionAsync(
    "medical_record", "heart_exam", "view", "user", "nurse2"
);
// Result: false (on staff but not in care team)

// Can the receptionist schedule an appointment?
bool canSchedule = await authService.CheckPermissionAsync(
    "appointment", "checkup_may15", "schedule", "user", "receptionist"
);
// Result: false (not a provider or primary doctor)
```

### Multi-tenant SaaS Platform

This example demonstrates a multi-tenant SaaS application:

#### Schema

```
definition user {}

definition organization {
    relation owner: user
    relation admin: user
    relation member: user
    
    permission admin = owner + admin
    permission access = owner + admin + member
    permission manage_billing = owner
}

definition subscription {
    relation organization: organization
    relation plan: plan_type
    
    permission view = organization->admin
    permission change = organization->owner
}

definition plan_type {
    relation basic: subscription
    relation professional: subscription
    relation enterprise: subscription
}

definition project {
    relation organization: organization
    relation owner: user
    relation editor: user
    relation viewer: user
    
    permission view = viewer + editor + owner + organization->admin
    permission edit = editor + owner + organization->admin
    permission manage = owner + organization->admin
    permission delete = owner + organization->admin
}

definition resource {
    relation project: project
    relation creator: user
    
    permission view = creator + project->view
    permission edit = creator + project->edit
    permission delete = creator + project->manage
}
```

#### Sample Relationships

```csharp
// Organization structure
await authService.WriteRelationshipAsync("organization", "company1", "owner", "user", "alice");
await authService.WriteRelationshipAsync("organization", "company1", "admin", "user", "bob");
await authService.WriteRelationshipAsync("organization", "company1", "member", "user", "charlie");

// Subscription
await authService.WriteRelationshipAsync("subscription", "company1_sub", "organization", "organization", "company1");
await authService.WriteRelationshipAsync("subscription", "company1_sub", "plan", "plan_type", "professional");
await authService.WriteRelationshipAsync("plan_type", "professional", "professional", "subscription", "company1_sub");

// Projects
await authService.WriteRelationshipAsync("project", "website_redesign", "organization", "organization", "company1");
await authService.WriteRelationshipAsync("project", "website_redesign", "owner", "user", "bob");
await authService.WriteRelationshipAsync("project", "website_redesign", "editor", "user", "charlie");

// Resources
await authService.WriteRelationshipAsync("resource", "homepage_mockup", "project", "project", "website_redesign");
await authService.WriteRelationshipAsync("resource", "homepage_mockup", "creator", "user", "charlie");
```

#### Check Examples

```csharp
// Can Charlie edit the homepage mockup?
bool canEdit = await authService.CheckPermissionAsync(
    "resource", "homepage_mockup", "edit", "user", "charlie"
);
// Result: true (as creator and project editor)

// Can Alice change the subscription plan?
bool canChangePlan = await authService.CheckPermissionAsync(
    "subscription", "company1_sub", "change", "user", "alice"
);
// Result: true (as organization owner)

// Can Bob delete the resource?
bool canDeleteResource = await authService.CheckPermissionAsync(
    "resource", "homepage_mockup", "delete", "user", "bob"
);
// Result: true (as project owner with manage permission)
```

## Advanced Concepts

### Caveats

SpiceDB supports caveats, which are conditions that must be satisfied for a permission to be granted:

```
definition document {
    relation viewer: user with caveat viewer_caveat
    relation owner: user
    
    permission view = viewer + owner
}

caveat viewer_caveat(time_of_day: string) {
    time_of_day == "business_hours"
}
```

To check permission with a caveat:

```csharp
var request = new CheckPermissionRequest
{
    Resource = new ObjectReference
    {
        ObjectType = "document",
        ObjectId = "budget"
    },
    Permission = "view",
    Subject = new SubjectReference
    {
        Object = new ObjectReference
        {
            ObjectType = "user",
            ObjectId = "anne"
        }
    },
    Context = new Dictionary<string, string>
    {
        { "time_of_day", "business_hours" }
    }
};

var response = await _client.CheckPermissionAsync(request);
```

### Preconditions

SpiceDB supports preconditions, which allow you to check if a relationship exists before writing:

```csharp
var request = new WriteRelationshipsRequest
{
    Updates = 
    {
        new RelationshipUpdate
        {
            Operation = RelationshipUpdate.Types.Operation.Create,
            Relationship = new Relationship
            {
                Resource = new ObjectReference
                {
                    ObjectType = "document",
                    ObjectId = "budget"
                },
                Relation = "editor",
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = "user",
                        ObjectId = "bob"
                    }
                }
            }
        }
    },
    OptionalPreconditions =
    {
        new Precondition
        {
            Operation = Precondition.Types.Operation.MustNotMatch,
            Filter = new RelationshipFilter
            {
                ResourceType = "document",
                OptionalResourceId = "budget",
                OptionalRelation = "blocked",
                OptionalSubjectFilter = new SubjectFilter
                {
                    SubjectType = "user",
                    OptionalSubjectId = "bob"
                }
            }
        }
    }
};
```

This ensures that Bob isn't in the "blocked" relation before adding him as an editor.

### Transactional Consistency

SpiceDB provides transactional guarantees for relationship writes:

```csharp
// Atomic batch of relationship updates
var request = new WriteRelationshipsRequest
{
    Updates = 
    {
        new RelationshipUpdate
        {
            Operation = RelationshipUpdate.Types.Operation.Create,
            Relationship = new Relationship
            {
                Resource = new ObjectReference
                {
                    ObjectType = "document",
                    ObjectId = "budget"
                },
                Relation = "viewer",
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = "user",
                        ObjectId = "charlie"
                    }
                }
            }
        },
        new RelationshipUpdate
        {
            Operation = RelationshipUpdate.Types.Operation.Delete,
            Relationship = new Relationship
            {
                Resource = new ObjectReference
                {
                    ObjectType = "document",
                    ObjectId = "budget"
                },
                Relation = "editor",
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = "user",
                        ObjectId = "dave"
                    }
                }
            }
        }
    }
};

await _client.WriteRelationshipsAsync(request);
```

### Recursive Relationships

SpiceDB supports recursive relationships, such as nested group memberships:

```
definition group {
    relation member: user | group#member
    relation admin: user
    
    permission view = member + admin
    permission manage = admin
}
```

This allows for:

```
group:engineering#member@user:alice
group:frontend#member@group:engineering#member
```

This means Alice is a member of the engineering group, and any member of the engineering group (including Alice) is also a member of the frontend group.

## Deployment Options

### Self-Hosted

SpiceDB can be self-hosted using Docker:

```bash
docker run -p 50051:50051 authzed/spicedb serve --grpc-preshared-key "my-key"
```

For production, use a more comprehensive configuration:

```yaml
# docker-compose.yml
version: '3.8'

services:
  spicedb:
    image: authzed/spicedb:latest
    command: ["serve", 
              "--grpc-preshared-key", "my-key",
              "--datastore-engine", "postgres",
              "--datastore-conn-uri", "postgres://postgres:password@postgres:5432/spicedb?sslmode=disable"]
    ports:
      - "50051:50051"
    depends_on:
      - postgres
    restart: always

  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: spicedb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  postgres_data:
```

### Cloud-Hosted Options

Authzed offers a managed SpiceDB service:

1. **Authzed Cloud**: Fully managed SpiceDB deployment
2. **Private Cloud**: Dedicated SpiceDB clusters for enterprise customers

For self-hosted deployments on cloud providers:

1. **Kubernetes**: Deploy using Helm chart
2. **AWS/GCP/Azure**: Deploy using container services

### High Availability Setup

For production deployments, consider:

1. **Multiple replicas**: Deploy multiple SpiceDB instances
2. **Load balancing**: Use gRPC load balancing
3. **Database clustering**: Use a clustered database for reliability
4. **Monitoring**: Set up Prometheus and Grafana for observability

Example Kubernetes deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spicedb
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spicedb
  template:
    metadata:
      labels:
        app: spicedb
    spec:
      containers:
      - name: spicedb
        image: authzed/spicedb:latest
        args:
        - serve
        - --grpc-preshared-key
        - $(PRESHARED_KEY)
        - --datastore-engine
        - postgres
        - --datastore-conn-uri
        - $(DATABASE_URL)
        env:
        - name: PRESHARED_KEY
          valueFrom:
            secretKeyRef:
              name: spicedb-secrets
              key: preshared-key
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: spicedb-secrets
              key: database-url
        ports:
        - containerPort: 50051
        readinessProbe:
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:50051"]
          initialDelaySeconds: 5
        resources:
          limits:
            cpu: "1"
            memory: "1Gi"
          requests:
            cpu: "500m"
            memory: "512Mi"
```

## Migration Strategies

### From RBAC to SpiceDB

If migrating from a role-based system:

1. **Map roles to relations**: Convert roles like "admin" to relations
2. **Define resources**: Map protected resources to SpiceDB object definitions
3. **Create permissions**: Define permissions combining relations
4. **Migrate data**: Convert role assignments to relationships

Example RBAC to SpiceDB mapping:

**RBAC:**
```
User 123 has role ADMIN for Project 456
User 789 has role VIEWER for Project 456
```

**SpiceDB:**
```
project:456#admin@user:123
project:456#viewer@user:789
```

### From Custom Authorization Solutions

When migrating from a custom authorization solution:

1. **Audit existing permissions**: Document all permission types
2. **Design SpiceDB schema**: Create a schema that covers all scenarios
3. **Dual-write phase**: Write to both systems during transition
4. **Validation phase**: Compare authorization decisions between systems
5. **Cutover**: Switch to SpiceDB as the source of truth

### Data Migration Tools

SpiceDB provides tools for bulk imports:

```csharp
// Batch relationship writes
var request = new WriteRelationshipsRequest
{
    Updates = new List<RelationshipUpdate>()
};

// Build batch of updates (example with 1000 relationships)
for (int i = 0; i < 1000; i++)
{
    request.Updates.Add(new RelationshipUpdate
    {
        Operation = RelationshipUpdate.Types.Operation.Create,
        Relationship = new Relationship
        {
            Resource = new ObjectReference
            {
                ObjectType = "document",
                ObjectId = $"doc{i}"
            },
            Relation = "viewer",
            Subject = new SubjectReference
            {
                Object = new ObjectReference
                {
                    ObjectType = "user",
                    ObjectId = $"user{i % 100}" // Distribute across 100 users
                }
            }
        }
    });
}

await _client.WriteRelationshipsAsync(request);
```

For very large datasets, process in batches:

```csharp
public async Task MigrateRelationshipsInBatches(List<RelationshipData> relationships, int batchSize = 100)
{
    for (int i = 0; i < relationships.Count; i += batchSize)
    {
        var batch = relationships
            .Skip(i)
            .Take(batchSize)
            .Select(r => new RelationshipUpdate
            {
                Operation = RelationshipUpdate.Types.Operation.Create,
                Relationship = new Relationship
                {
                    Resource = new ObjectReference
                    {
                        ObjectType = r.ResourceType,
                        ObjectId = r.ResourceId
                    },
                    Relation = r.Relation,
                    Subject = new SubjectReference
                    {
                        Object = new ObjectReference
                        {
                            ObjectType = r.SubjectType,
                            ObjectId = r.SubjectId
                        }
                    }
                }
            })
            .ToList();
            
        var request = new WriteRelationshipsRequest
        {
            Updates = batch
        };
        
        await _client.WriteRelationshipsAsync(request);
        Console.WriteLine($"Migrated batch {i/batchSize + 1} of {Math.Ceiling(relationships.Count / (double)batchSize)}");
    }
}

public class RelationshipData
{
    public string ResourceType { get; set; }
    public string ResourceId { get; set; }
    public string Relation { get; set; }
    public string SubjectType { get; set; }
    public string SubjectId { get; set; }
}
```

## Performance Optimization

### Query Optimization

1. **Limit schema complexity**: Simpler schemas perform better
2. **Reduce nesting depth**: Deep relation chains impact performance
3. **Use caching**: SpiceDB has built-in caching mechanisms
4. **Optimize permission checks**: Check permissions at the right level

### Caching Strategies

SpiceDB has built-in caching, but you can add application-level caching:

```csharp
public class AuthorizationCache
{
    private readonly MemoryCache _cache;
    private readonly TimeSpan _defaultTtl;
    
    public AuthorizationCache(TimeSpan? defaultTtl = null)
    {
        _cache = new MemoryCache(new MemoryCacheOptions());
        _defaultTtl = defaultTtl ?? TimeSpan.FromMinutes(5);
    }
    
    public bool TryGetPermission(string resourceType, string resourceId, string permission, string subjectType, string subjectId, out bool? result)
    {
        var key = $"{resourceType}:{resourceId}#{permission}@{subjectType}:{subjectId}";
        if (_cache.TryGetValue(key, out bool cachedResult))
        {
            result = cachedResult;
            return true;
        }
        
        result = null;
        return false;
    }
    
    public void SetPermission(string resourceType, string resourceId, string permission, string subjectType, string subjectId, bool result)
    {
        var key = $"{resourceType}:{resourceId}#{permission}@{subjectType}:{subjectId}";
        _cache.Set(key, result, _defaultTtl);
    }
    
    public void InvalidateResource(string resourceType, string resourceId)
    {
        // This is a simplified approach - in production you'd need a more sophisticated 
        // way to track and invalidate related cache entries
        var pattern = $"{resourceType}:{resourceId}";
        
        // This is inefficient but works for demonstration
        var keysToRemove = _cache.GetKeys<string>()
            .Where(k => k.StartsWith(pattern))
            .ToList();
            
        foreach (var key in keysToRemove)
        {
            _cache.Remove(key);
        }
    }
}

// Extension method to get all cache keys
public static class MemoryCacheExtensions
{
    public static IEnumerable<T> GetKeys<T>(this IMemoryCache cache)
    {
        var field = typeof(MemoryCache).GetProperty("EntriesCollection", BindingFlags.NonPublic | BindingFlags.Instance);
        var collection = field.GetValue(cache) as ICollection;
        
        foreach (var item in collection)
        {
            var methodInfo = item.GetType().GetProperty("Key");
            var key = methodInfo.GetValue(item);
            if (key is T typedKey)
            {
                yield return typedKey;
            }
        }
    }
}
```

### Bulk Operations

Use bulk operations to reduce API calls:

```csharp
// Batch permission checks
public async Task<Dictionary<string, bool>> BatchCheckPermissionsAsync(
    string subjectType, 
    string subjectId,
    string permission,
    string resourceType,
    List<string> resourceIds)
{
    var tasks = resourceIds.Select(resourceId => 
        _client.CheckPermissionAsync(new CheckPermissionRequest
        {
            Resource = new ObjectReference
            {
                ObjectType = resourceType,
                ObjectId = resourceId
            },
            Permission = permission,
            Subject = new SubjectReference
            {
                Object = new ObjectReference
                {
                    ObjectType = subjectType,
                    ObjectId = subjectId
                }
            }
        })
    );
    
    var results = await Task.WhenAll(tasks);
    
    return resourceIds.Zip(results, (id, result) => 
        new KeyValuePair<string, bool>(id, result.Permissionship == CheckPermissionResponse.Types.Permissionship.HasPermission)
    ).ToDictionary(kv => kv.Key, kv => kv.Value);
}
```

## Security Considerations

### API Security

1. **Authentication**: Use strong authentication for API access
2. **Authorization**: Limit who can modify schemas and relationships
3. **TLS**: Always use TLS for API communication
4. **Token security**: Protect the preshared key or API token

### Audit Logging

Implement comprehensive logging:

```csharp
public class AuditedAuthorizationService
{
    private readonly PermissionsService.PermissionsServiceClient _client;
    private readonly ILogger<AuditedAuthorizationService> _logger;
    
    public AuditedAuthorizationService(
        PermissionsService.PermissionsServiceClient client,
        ILogger<AuditedAuthorizationService> logger)
    {
        _client = client;
        _logger = logger;
    }
    
    public async Task<bool> CheckPermissionWithAuditAsync(
        string resourceType, 
        string resourceId, 
        string permission, 
        string subjectType, 
        string subjectId,
        string userId,
        string actionContext)
    {
        var stopwatch = Stopwatch.StartNew();
        bool result = false;
        
        try
        {
            var request = new CheckPermissionRequest
            {
                Resource = new ObjectReference
                {
                    ObjectType = resourceType,
                    ObjectId = resourceId
                },
                Permission = permission,
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = subjectType,
                        ObjectId = subjectId
                    }
                }
            };
            
            var response = await _client.CheckPermissionAsync(request);
            result = response.Permissionship == CheckPermissionResponse.Types.Permissionship.HasPermission;
            
            return result;
        }
        finally
        {
            stopwatch.Stop();
            
            _logger.LogInformation(
                "Permission Check: User={UserId} Action={Action} Resource={ResourceType}/{ResourceId} " +
                "Subject={SubjectType}/{SubjectId} Permission={Permission} Result={Result} Duration={DurationMs}ms",
                userId,
                actionContext,
                resourceType,
                resourceId,
                subjectType,
                subjectId,
                permission,
                result,
                stopwatch.ElapsedMilliseconds
            );
        }
    }
    
    public async Task WriteRelationshipWithAuditAsync(
        string resourceType, 
        string resourceId, 
        string relation, 
        string subjectType, 
        string subjectId,
        string userId,
        string actionContext)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var request = new WriteRelationshipsRequest
            {
                Updates = 
                {
                    new RelationshipUpdate
                    {
                        Operation = RelationshipUpdate.Types.Operation.Create,
                        Relationship = new Relationship
                        {
                            Resource = new ObjectReference
                            {
                                ObjectType = resourceType,
                                ObjectId = resourceId
                            },
                            Relation = relation,
                            Subject = new SubjectReference
                            {
                                Object = new ObjectReference
                                {
                                    ObjectType = subjectType,
                                    ObjectId = subjectId
                                }
                            }
                        }
                    }
                }
            };
            
            await _client.WriteRelationshipsAsync(request);
        }
        finally
        {
            stopwatch.Stop();
            
            _logger.LogInformation(
                "Relationship Written: User={UserId} Action={Action} Resource={ResourceType}/{ResourceId} " +
                "Relation={Relation} Subject={SubjectType}/{SubjectId} Duration={DurationMs}ms",
                userId,
                actionContext,
                resourceType,
                resourceId,
                relation,
                subjectType,
                subjectId,
                stopwatch.ElapsedMilliseconds
            );
        }
    }
}
```

### Schema Validation

Validate schema changes before applying them:

```csharp
public async Task<bool> ValidateSchema(string schemaDefinition)
{
    try
    {
        // Validate schema without writing it
        var request = new ValidateSchemaRequest
        {
            Schema = new SchemaDefinition
            {
                SchemaText = schemaDefinition
            }
        };
        
        var response = await _client.ValidateSchemaAsync(request);
        
        if (response.ValidationErrors.Count > 0)
        {
            foreach (var error in response.ValidationErrors)
            {
                Console.WriteLine($"Schema validation error: {error}");
            }
            return false;
        }
        
        return true;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Schema validation failed: {ex.Message}");
        return false;
    }
}
```

## Best Practices

### Schema Design Best Practices

1. **Start simple**: Begin with basic relations and expand as needed
2. **Use permissions**: Define permissions for related access rights
3. **Limit nesting depth**: Keep object hierarchies shallow for performance
4. **Be consistent**: Use consistent naming conventions
5. **Document your schema**: Comment complex relationships
6. **Use standard patterns**: Follow established patterns for common scenarios

### Implementation Best Practices

1. **Separate concerns**: Decouple authorization logic from business logic
2. **Abstract the client**: Create an authorization service layer
3. **Handle errors gracefully**: Implement proper error handling
4. **Use preconditions**: Validate assumptions before writing relationships
5. **Implement caching**: Cache authorization decisions when appropriate
6. **Monitoring**: Set up alerts for authorization failures

### Complete Authorization Service Example

```csharp
public class SpiceDbAuthorizationService
{
    private readonly PermissionsService.PermissionsServiceClient _client;
    private readonly ILogger<SpiceDbAuthorizationService> _logger;
    private readonly AuthorizationCache _cache;
    
    public SpiceDbAuthorizationService(
        string endpoint, 
        string token,
        ILogger<SpiceDbAuthorizationService> logger)
    {
        // Configure channel with authentication
        var channel = GrpcChannel.ForAddress(endpoint, new GrpcChannelOptions
        {
            Credentials = ChannelCredentials.Create(
                ChannelCredentials.SecureSsl,
                CallCredentials.FromInterceptor((context, metadata) =>
                {
                    metadata.Add("Authorization", $"Bearer {token}");
                    return Task.CompletedTask;
                })
            )
        });
        
        _client = new PermissionsService.PermissionsServiceClient(channel);
        _logger = logger;
        _cache = new AuthorizationCache();
    }
    
    public async Task<bool> HasPermissionAsync(
        string resourceType, 
        string resourceId, 
        string permission, 
        string subjectType, 
        string subjectId)
    {
        // Try to get from cache first
        if (_cache.TryGetPermission(resourceType, resourceId, permission, subjectType, subjectId, out var cachedResult) && 
            cachedResult.HasValue)
        {
            return cachedResult.Value;
        }
        
        var stopwatch = Stopwatch.StartNew();
        try
        {
            var request = new CheckPermissionRequest
            {
                Resource = new ObjectReference
                {
                    ObjectType = resourceType,
                    ObjectId = resourceId
                },
                Permission = permission,
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = subjectType,
                        ObjectId = subjectId
                    }
                }
            };
            
            var response = await _client.CheckPermissionAsync(request);
            var result = response.Permissionship == CheckPermissionResponse.Types.Permissionship.HasPermission;
            
            // Cache the result
            _cache.SetPermission(resourceType, resourceId, permission, subjectType, subjectId, result);
            
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking permission: {Message}", ex.Message);
            return false;
        }
        finally
        {
            stopwatch.Stop();
            _logger.LogDebug(
                "Permission check: {ResourceType}/{ResourceId}#{Permission}@{SubjectType}:{SubjectId} " +
                "took {ElapsedMs}ms",
                resourceType, resourceId, permission, subjectType, subjectId, stopwatch.ElapsedMilliseconds);
        }
    }
    
    public async Task GrantPermissionAsync(
        string resourceType, 
        string resourceId, 
        string relation, 
        string subjectType, 
        string subjectId)
    {
        try
        {
            var request = new WriteRelationshipsRequest
            {
                Updates = 
                {
                    new RelationshipUpdate
                    {
                        Operation = RelationshipUpdate.Types.Operation.Create,
                        Relationship = new Relationship
                        {
                            Resource = new ObjectReference
                            {
                                ObjectType = resourceType,
                                ObjectId = resourceId
                            },
                            Relation = relation,
                            Subject = new SubjectReference
                            {
                                Object = new ObjectReference
                                {
                                    ObjectType = subjectType,
                                    ObjectId = subjectId
                                }
                            }
                        }
                    }
                }
            };
            
            await _client.WriteRelationshipsAsync(request);
            
            // Invalidate cache for this resource
            _cache.InvalidateResource(resourceType, resourceId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error granting permission: {Message}", ex.Message);
            throw;
        }
    }
    
    public async Task RevokePermissionAsync(
        string resourceType, 
        string resourceId, 
        string relation, 
        string subjectType, 
        string subjectId)
    {
        try
        {
            var request = new WriteRelationshipsRequest
            {
                Updates = 
                {
                    new RelationshipUpdate
                    {
                        Operation = RelationshipUpdate.Types.Operation.Delete,
                        Relationship = new Relationship
                        {
                            Resource = new ObjectReference
                            {
                                ObjectType = resourceType,
                                ObjectId = resourceId
                            },
                            Relation = relation,
                            Subject = new SubjectReference
                            {
                                Object = new ObjectReference
                                {
                                    ObjectType = subjectType,
                                    ObjectId = subjectId
                                }
                            }
                        }
                    }
                }
            };
            
            await _client.WriteRelationshipsAsync(request);
            
            // Invalidate cache for this resource
            _cache.InvalidateResource(resourceType, resourceId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error revoking permission: {Message}", ex.Message);
            throw;
        }
    }
    
    public async Task<List<string>> FindResourcesWithPermissionAsync(
        string resourceType, 
        string permission, 
        string subjectType, 
        string subjectId)
    {
        try
        {
            var request = new LookupResourcesRequest
            {
                ResourceObjectType = resourceType,
                Permission = permission,
                Subject = new SubjectReference
                {
                    Object = new ObjectReference
                    {
                        ObjectType = subjectType,
                        ObjectId = subjectId
                    }
                }
            };
            
            var results = new List<string>();
            using var call = _client.LookupResources(request);
            
            await foreach (var response in call.ResponseStream.ReadAllAsync())
            {
                results.Add(response.ResourceObjectId);
            }
            
            return results;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error finding resources: {Message}", ex.Message);
            return new List<string>();
        }
    }
}
```

### Schema Versioning

As your authorization needs evolve:

1. **Version your schema**: Keep track of schema changes
2. **Backwards compatibility**: Ensure new schemas don't break existing checks
3. **Migration scripts**: Create scripts to update relationships for schema changes
4. **Testing**: Test new schemas against existing permission scenarios

## Conclusion

SpiceDB provides a robust, flexible framework for implementing fine-grained authorization in modern applications. By modeling permissions as relationships between resources and subjects, you can express complex authorization rules in a way that's both intuitive and performant.

Key takeaways:

1. **Schema-driven**: Define your authorization model clearly in SpiceDB's schema language
2. **Relationship-based**: Think in terms of relationships between subjects and resources
3. **Permission computation**: Use SpiceDB's permission system to derive access rights
4. **Performance-oriented**: Designed for high-throughput, low-latency permission checks
5. **Consistency guarantees**: Strong consistency for mission-critical applications

Whether you're building a document sharing application, team collaboration platform, healthcare system, or multi-tenant SaaS platform, SpiceDB provides the tools you need to implement robust, scalable authorization. By starting with a solid schema and following best practices, you can ensure your application's permissions system can grow and evolve with your business needs.

SpiceDB's relationship to Google's Zanzibar paper means it builds on proven concepts deployed at massive scale, while its open-source nature makes it accessible to developers and organizations of all sizes. The availability of both self-hosted and managed options provides flexibility in deployment, allowing you to choose the approach that best fits your infrastructure and security requirements.

By understanding and implementing the concepts covered in this tutorial, you'll be well-equipped to build sophisticated authorization systems that protect your application's resources while providing the flexibility your users need.
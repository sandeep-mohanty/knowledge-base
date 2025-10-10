# OpenFGA: A Comprehensive Tutorial

## Table of Contents

1. [Introduction to OpenFGA](#introduction-to-openfga)
2. [Core Concepts](#core-concepts)
3. [OpenFGA Modeling Language](#openfga-modeling-language)
4. [Authorization Models](#authorization-models)
5. [API and SDKs](#api-and-sdks)
6. [Practical Examples](#practical-examples)
   - [Document Management System](#document-management-system)
   - [E-commerce Platform](#e-commerce-platform)
   - [Healthcare Application](#healthcare-application)
   - [Educational Platform](#educational-platform)
7. [Advanced Concepts](#advanced-concepts)
8. [Deployment Options](#deployment-options)
9. [Migration Strategies](#migration-strategies)
10. [Performance Optimization](#performance-optimization)
11. [Security Considerations](#security-considerations)
12. [Best Practices](#best-practices)

## Introduction to OpenFGA

OpenFGA (Fine-Grained Authorization) is an open-source authorization solution inspired by Google's Zanzibar paper. It was originally developed by Auth0 (now part of Okta) and released as open-source in 2022. OpenFGA provides a flexible, scalable approach to implementing fine-grained authorization in modern applications.

Key features of OpenFGA include:

1. **Relationship-based**: Models permissions as relationships between users and objects
2. **Flexible modeling**: Supports complex authorization scenarios through its modeling language
3. **High performance**: Designed for low-latency authorization decisions
4. **Language agnostic**: Provides SDKs for multiple programming languages
5. **Developer-friendly**: Includes tools for testing and visualizing authorization models
6. **Open-source**: Available under the Apache 2.0 license

OpenFGA is particularly well-suited for applications that require:
- Complex permission structures
- Team and organization-based access control
- Hierarchical resource management
- Fine-grained sharing capabilities

## Core Concepts

### Authorization Model

The authorization model in OpenFGA defines the types of objects in your system, their possible relationships, and how those relationships translate into permissions. Models are written in OpenFGA's Domain-Specific Language (DSL).

### Types

Types represent the different kinds of objects in your system, such as:
- Users (users, groups, services)
- Resources (documents, folders, projects)
- Organizational entities (teams, departments, organizations)

### Relations

Relations define the ways that subjects can relate to objects. For example:
- `viewer`: Can view a document
- `member`: Is a member of a group
- `owner`: Owns a resource

### User-Defined Relations

In OpenFGA, you can define what each relation means and how it can be computed from other relations using the DSL.

### Direct Relations

Direct relations are explicitly assigned:
```
document:budget#viewer@user:anne
```
This means Anne is directly assigned as a viewer of the budget document.

### Computed Relations

Computed relations (or "userset rewrite rules") are derived from other relations:
```
define can_view: viewer or editor or owner
```

### Tuples

Tuples are the core data structure in OpenFGA, representing a single relationship between an object and a subject. A tuple has three components:
- **User**: The subject of the relationship (who)
- **Relation**: The type of relationship (how)
- **Object**: The target of the relationship (what)

Example tuple: `{user: "user:anne", relation: "viewer", object: "document:budget"}`

## OpenFGA Modeling Language

OpenFGA uses a Domain-Specific Language (DSL) to define authorization models. Let's explore the syntax and capabilities:

### Basic Model Structure

```
model
  schema 1.1

type user

type document
  relations
    define viewer: [user]
    define editor: [user]
    define owner: [user]
```

This defines a simple model with two types: `user` and `document`. The `document` type has three relations: `viewer`, `editor`, and `owner`.

### Computed Relations (Userset Rewrites)

```
type document
  relations
    define viewer: [user]
    define editor: [user]
    define owner: [user]
    define can_view: viewer or editor or owner
    define can_edit: editor or owner
    define can_delete: owner
```

Here, we've added computed relations `can_view`, `can_edit`, and `can_delete` that are derived from the direct relations.

### Relation References

Relations can refer to other objects and their relations:

```
type folder
  relations
    define viewer: [user]
    define owner: [user]

type document
  relations
    define parent: [folder]
    define viewer: [user]
    define owner: [user]
    define can_view: viewer or owner or parent->viewer or parent->owner
```

This means a user can view a document if they are a direct viewer, an owner, or have viewer or owner access to the document's parent folder.

### Tuples with Conditions

OpenFGA 1.1+ supports conditions on tuples (contextual tuples):

```
type document
  relations
    define viewer: [user] with time_constraint
    define owner: [user]
```

With a contextual tuple like:
```
{
  user: "user:anne",
  relation: "viewer",
  object: "document:budget",
  condition: {
    name: "time_constraint",
    context: {
      starts_at: "2023-01-01T00:00:00Z",
      ends_at: "2023-12-31T23:59:59Z"
    }
  }
}
```

This means Anne can view the budget document only within the specified time period.

### Public Access

For public resources, OpenFGA provides a special syntax:

```
type document
  relations
    define viewer: [user, public]
    define can_view: viewer

type public
```

With a tuple like:
```
{user: "public", relation: "viewer", object: "document:public_announcement"}
```

Anyone can view the public_announcement document.

## Authorization Models

Let's walk through the process of designing an authorization model in OpenFGA:

### 1. Identify Types

First, identify the different types of objects in your system:
- Users and groups
- Resources (documents, projects, etc.)
- Organizational units (teams, departments)

### 2. Define Relations

For each type, define the possible relations:

```
type user

type group
  relations
    define member: [user, group#member]
    define admin: [user]

type document
  relations
    define viewer: [user, group#member]
    define editor: [user, group#member]
    define owner: [user]
```

### 3. Define Computed Relations

Add computed relations to represent permissions:

```
type document
  relations
    define viewer: [user, group#member]
    define editor: [user, group#member]
    define owner: [user]
    define can_view: viewer or editor or owner
    define can_edit: editor or owner
    define can_manage: owner
```

### 4. Add Hierarchical Relationships

For complex systems, add parent-child relationships:

```
type folder
  relations
    define viewer: [user, group#member]
    define editor: [user, group#member]
    define owner: [user]
    define can_view: viewer or editor or owner
    define can_edit: editor or owner
    define can_manage: owner

type document
  relations
    define parent: [folder]
    define viewer: [user, group#member]
    define editor: [user, group#member]
    define owner: [user]
    define can_view: viewer or editor or owner or parent->can_view
    define can_edit: editor or owner or parent->can_edit
    define can_manage: owner or parent->can_manage
```

### 5. Test and Iterate

Use OpenFGA's playground to test and refine your model with sample tuples and authorization checks.

## API and SDKs

OpenFGA provides a RESTful API and multiple language SDKs for integration:

### Core API Endpoints

1. **Authorization Models**
   - Create, read, update, delete models

2. **Relationship Tuples**
   - Write, read, delete tuples

3. **Authorization Checks**
   - Check if a user has a relation to an object
   
4. **Relationship Queries**
   - List objects a user has access to
   - List users that have access to an object

### JavaScript/TypeScript SDK Example

```javascript
import { OpenFgaClient } from '@openfga/sdk';

// Initialize client
const fgaClient = new OpenFgaClient({
  apiUrl: 'https://api.fga.example',
  storeId: 'store-id',
  apiToken: 'api-token'
});

// Check if a user can view a document
async function canUserViewDocument(userId, documentId) {
  const { allowed } = await fgaClient.check({
    user: `user:${userId}`,
    relation: 'can_view',
    object: `document:${documentId}`
  });
  
  return allowed;
}

// Add a relationship
async function addDocumentEditor(userId, documentId) {
  await fgaClient.write({
    writes: [{
      user: `user:${userId}`,
      relation: 'editor',
      object: `document:${documentId}`
    }]
  });
}

// List documents a user can view
async function listViewableDocuments(userId) {
  const response = await fgaClient.listObjects({
    user: `user:${userId}`,
    relation: 'can_view',
    type: 'document'
  });
  
  return response.objects.map(obj => obj.split(':')[1]);
}
```

### C# SDK Example

```csharp
using OpenFga.Sdk;
using OpenFga.Sdk.Model;

public class AuthorizationService
{
    private readonly OpenFgaClient _fgaClient;

    public AuthorizationService(string apiUrl, string storeId, string apiToken)
    {
        // Initialize client
        var clientConfiguration = new ClientConfiguration
        {
            ApiUrl = apiUrl,
            StoreId = storeId,
            Credentials = new Credentials
            {
                Method = CredentialsMethod.ApiToken,
                Config = new ApiTokenCredentialConfiguration
                {
                    Token = apiToken
                }
            }
        };
        
        _fgaClient = new OpenFgaClient(clientConfiguration);
    }
    
    // Check if a user can view a document
    public async Task<bool> CanUserViewDocumentAsync(string userId, string documentId)
    {
        var request = new CheckRequest
        {
            User = $"user:{userId}",
            Relation = "can_view",
            Object = $"document:{documentId}"
        };
        
        var response = await _fgaClient.Check(request);
        return response.Allowed;
    }
    
    // Add a relationship
    public async Task AddDocumentEditorAsync(string userId, string documentId)
    {
        var request = new WriteRequest
        {
            Writes = new List<TupleKey>
            {
                new TupleKey
                {
                    User = $"user:{userId}",
                    Relation = "editor",
                    Object = $"document:{documentId}"
                }
            }
        };
        
        await _fgaClient.Write(request);
    }
    
    // List documents a user can view
    public async Task<List<string>> ListViewableDocumentsAsync(string userId)
    {
        var request = new ListObjectsRequest
        {
            User = $"user:{userId}",
            Relation = "can_view",
            Type = "document"
        };
        
        var response = await _fgaClient.ListObjects(request);
        return response.Objects
            .Select(obj => obj.Split(':')[1])
            .ToList();
    }
}
```

## Practical Examples

Let's explore practical examples of OpenFGA models for different application domains:

### Document Management System

This example shows how to model a document management system similar to Google Docs or Dropbox:

#### Model

```
model
  schema 1.1

type user

type group
  relations
    define member: [user, group#member]
    define manager: [user, group#manager]

type folder
  relations
    define parent: [folder]
    define owner: [user, group#member]
    define editor: [user, group#member]
    define viewer: [user, group#member]
    define can_view: viewer or editor or owner or parent->can_view
    define can_edit: editor or owner or parent->can_edit
    define can_share: owner or parent->can_share
    define can_delete: owner or parent->can_delete

type document
  relations
    define parent: [folder]
    define owner: [user, group#member]
    define editor: [user, group#member]
    define viewer: [user, group#member]
    define can_view: viewer or editor or owner or parent->can_view
    define can_edit: editor or owner or parent->can_edit
    define can_share: owner or parent->can_share
    define can_delete: owner or parent->can_delete

type share_link
  relations
    define for: [document, folder]
    define creator: [user]
    define can_use: creator or public
    define can_view_target: for->can_view and can_use

type public
```

#### Sample Tuples

```javascript
// Direct user assignments
{ user: "user:anne", relation: "owner", object: "folder:team_documents" }
{ user: "user:bob", relation: "editor", object: "folder:team_documents" }
{ user: "user:charlie", relation: "viewer", object: "folder:team_documents" }

// Document in folder
{ user: "folder:team_documents", relation: "parent", object: "document:budget_2023" }
{ user: "user:anne", relation: "editor", object: "document:budget_2023" }

// Group membership
{ user: "user:dave", relation: "member", object: "group:finance" }
{ user: "group:finance", relation: "viewer", object: "document:budget_2023" }

// Share link
{ user: "document:budget_2023", relation: "for", object: "share_link:budget_view" }
{ user: "user:anne", relation: "creator", object: "share_link:budget_view" }
{ user: "public", relation: "can_use", object: "share_link:budget_view" }
```

#### Check Examples in C#

```csharp
// Can Bob edit the budget document?
var canEdit = await _fgaClient.Check(new CheckRequest
{
    User = "user:bob",
    Relation = "can_edit",
    Object = "document:budget_2023"
});
// Result: true (because Bob is an editor of the parent folder)

// Can Dave view the budget document?
var canView = await _fgaClient.Check(new CheckRequest
{
    User = "user:dave",
    Relation = "can_view",
    Object = "document:budget_2023"
});
// Result: true (because Dave is a member of the finance group)

// Can a public user access the document via the share link?
var canAccessViaLink = await _fgaClient.Check(new CheckRequest
{
    User = "public",
    Relation = "can_view_target",
    Object = "share_link:budget_view"
});
// Result: true (public share link access)
```

### E-commerce Platform

This example demonstrates modeling permissions for an e-commerce platform:

#### Model

```
model
  schema 1.1

type user

type store
  relations
    define owner: [user]
    define admin: [user]
    define staff: [user]
    define can_view: owner or admin or staff
    define can_edit: owner or admin
    define can_manage_staff: owner or admin
    define can_manage_billing: owner

type product
  relations
    define store: [store]
    define editor: [user]
    define can_view: editor or store->can_view
    define can_edit: editor or store->can_edit
    define can_delete: store->owner or store->admin

type order
  relations
    define store: [store]
    define customer: [user]
    define can_view: customer or store->can_view
    define can_process: store->admin or store->staff
    define can_refund: store->admin or store->owner

type discount
  relations
    define store: [store]
    define can_view: store->can_view
    define can_apply: store->owner or store->admin
    define can_edit: store->owner or store->admin
```

#### Sample Tuples

```javascript
// Store relationships
{ user: "user:alice", relation: "owner", object: "store:alices_boutique" }
{ user: "user:bob", relation: "admin", object: "store:alices_boutique" }
{ user: "user:charlie", relation: "staff", object: "store:alices_boutique" }

// Products
{ user: "store:alices_boutique", relation: "store", object: "product:vintage_dress" }
{ user: "user:designer", relation: "editor", object: "product:vintage_dress" }

// Orders
{ user: "store:alices_boutique", relation: "store", object: "order:12345" }
{ user: "user:customer", relation: "customer", object: "order:12345" }

// Discounts
{ user: "store:alices_boutique", relation: "store", object: "discount:summer_sale" }
```

#### Check Examples in C#

```csharp
// Can Charlie (staff) process orders?
var canProcess = await _fgaClient.Check(new CheckRequest
{
    User = "user:charlie",
    Relation = "can_process",
    Object = "order:12345"
});
// Result: true

// Can Charlie apply discounts?
var canApplyDiscount = await _fgaClient.Check(new CheckRequest
{
    User = "user:charlie",
    Relation = "can_apply",
    Object = "discount:summer_sale"
});
// Result: false (only admins and owners can)

// Can a designer edit the product?
var canEditProduct = await _fgaClient.Check(new CheckRequest
{
    User = "user:designer",
    Relation = "can_edit",
    Object = "product:vintage_dress"
});
// Result: true (direct editor)
```

### Healthcare Application

This example shows a model for a healthcare application with strict privacy controls:

#### Model

```
model
  schema 1.1

type user

type hospital
  relations
    define administrator: [user]
    define medical_staff: [user]
    define can_access: administrator or medical_staff

type department
  relations
    define hospital: [hospital]
    define head: [user]
    define staff: [user]
    define can_access: head or staff or hospital->administrator
    define can_manage: head or hospital->administrator

type patient
  relations
    define self: [user]
    define primary_doctor: [user]
    define assigned_department: [department]
    define caregiver: [user] with care_period
    define can_view_basic: self or primary_doctor or caregiver or assigned_department->staff
    define can_view_full: self or primary_doctor or assigned_department->head
    define can_edit: primary_doctor or assigned_department->head

type medical_record
  relations
    define patient: [patient]
    define author: [user]
    define can_view: author or patient->can_view_full
    define can_edit: author and patient->can_edit
```

#### Sample Tuples

```javascript
// Hospital structure
{ user: "user:dr_admin", relation: "administrator", object: "hospital:general" }

// Department structure
{ user: "hospital:general", relation: "hospital", object: "department:cardiology" }
{ user: "user:dr_heart", relation: "head", object: "department:cardiology" }
{ user: "user:dr_assist", relation: "staff", object: "department:cardiology" }

// Patient relationships
{ user: "user:patient1", relation: "self", object: "patient:john_doe" }
{ user: "user:dr_heart", relation: "primary_doctor", object: "patient:john_doe" }
{ user: "department:cardiology", relation: "assigned_department", object: "patient:john_doe" }
{ 
  user: "user:family_member", 
  relation: "caregiver", 
  object: "patient:john_doe",
  condition: {
    name: "care_period",
    context: {
      starts_at: "2023-01-01T00:00:00Z",
      ends_at: "2023-12-31T23:59:59Z"
    }
  }
}

// Medical records
{ user: "patient:john_doe", relation: "patient", object: "medical_record:heart_exam" }
{ user: "user:dr_heart", relation: "author", object: "medical_record:heart_exam" }
```

#### Check Examples in C#

```csharp
// Can Dr. Heart view the patient's medical record?
var canViewRecord = await _fgaClient.Check(new CheckRequest
{
    User = "user:dr_heart",
    Relation = "can_view",
    Object = "medical_record:heart_exam"
});
// Result: true (as author and primary doctor)

// Can a family member view the patient's basic info?
var contextualTupleKey = new ContextualTupleKey
{
    User = "user:family_member",
    Relation = "can_view_basic",
    Object = "patient:john_doe",
    Context = new Dictionary<string, string>
    {
        { "current_time", "2023-06-15T12:00:00Z" }
    }
};

var canViewBasicInfo = await _fgaClient.Check(new CheckRequest
{
    User = "user:family_member",
    Relation = "can_view_basic",
    Object = "patient:john_doe",
    Context = new Dictionary<string, string>
    {
        { "current_time", "2023-06-15T12:00:00Z" }
    }
});
// Result: true (within care period)

// Can Dr. Assist edit the medical record?
var canEditRecord = await _fgaClient.Check(new CheckRequest
{
    User = "user:dr_assist",
    Relation = "can_edit",
    Object = "medical_record:heart_exam"
});
// Result: false (not the author or head of department)
```

### Educational Platform

This example demonstrates modeling an educational platform like Google Classroom:

#### Model

```
model
  schema 1.1

type user

type school
  relations
    define administrator: [user]
    define teacher: [user]
    define student: [user]
    define can_view: administrator or teacher or student
    define can_manage_courses: administrator or teacher
    define can_manage_users: administrator

type course
  relations
    define school: [school]
    define teacher: [user, school#teacher]
    define teaching_assistant: [user]
    define student: [user, school#student]
    define can_view: teacher or teaching_assistant or student or school->administrator
    define can_edit: teacher or school->administrator
    define can_grade: teacher or teaching_assistant
    define can_manage_enrollment: teacher or school->administrator

type assignment
  relations
    define course: [course]
    define author: [user]
    define assigned_to: [user] with deadline
    define can_view: author or course->teacher or course->teaching_assistant or assigned_to
    define can_edit: author or course->teacher
    define can_submit: assigned_to
    define can_grade: course->can_grade

type submission
  relations
    define assignment: [assignment]
    define student: [user]
    define can_view: student or assignment->can_grade
    define can_edit: student and assignment->can_submit
    define can_grade: assignment->can_grade
```

#### Sample Tuples

```javascript
// School structure
{ user: "user:principal", relation: "administrator", object: "school:high_school" }
{ user: "user:ms_math", relation: "teacher", object: "school:high_school" }
{ user: "user:student1", relation: "student", object: "school:high_school" }
{ user: "user:student2", relation: "student", object: "school:high_school" }

// Course
{ user: "school:high_school", relation: "school", object: "course:algebra_101" }
{ user: "user:ms_math", relation: "teacher", object: "course:algebra_101" }
{ user: "user:ta_math", relation: "teaching_assistant", object: "course:algebra_101" }
{ user: "user:student1", relation: "student", object: "course:algebra_101" }
{ user: "user:student2", relation: "student", object: "course:algebra_101" }

// Assignment
{ user: "course:algebra_101", relation: "course", object: "assignment:quadratic_equations" }
{ user: "user:ms_math", relation: "author", object: "assignment:quadratic_equations" }
{ 
  user: "user:student1", 
  relation: "assigned_to", 
  object: "assignment:quadratic_equations",
  condition: {
    name: "deadline",
    context: {
      due_date: "2023-11-15T23:59:59Z"
    }
  }
}
{ 
  user: "user:student2", 
  relation: "assigned_to", 
  object: "assignment:quadratic_equations",
  condition: {
    name: "deadline",
    context: {
      due_date: "2023-11-15T23:59:59Z"
    }
  }
}

// Submission
{ user: "assignment:quadratic_equations", relation: "assignment", object: "submission:student1_homework" }
{ user: "user:student1", relation: "student", object: "submission:student1_homework" }
```

#### Check Examples in C#

```csharp
// Can the TA grade a student's submission?
var canGradeSubmission = await _fgaClient.Check(new CheckRequest
{
    User = "user:ta_math",
    Relation = "can_grade",
    Object = "submission:student1_homework"
});
// Result: true (TAs can grade submissions)

// Can Student1 submit the assignment before the deadline?
var canSubmit = await _fgaClient.Check(new CheckRequest
{
    User = "user:student1",
    Relation = "can_submit",
    Object = "assignment:quadratic_equations",
    Context = new Dictionary<string, string>
    {
        { "current_time", "2023-11-10T14:30:00Z" }
    }
});
// Result: true (before deadline)

// Can Student1 edit their submission?
var canEditSubmission = await _fgaClient.Check(new CheckRequest
{
    User = "user:student1",
    Relation = "can_edit",
    Object = "submission:student1_homework",
    Context = new Dictionary<string, string>
    {
        { "current_time", "2023-11-10T14:30:00Z" }
    }
});
// Result: true (students can edit their own submissions before deadline)
```

## Advanced Concepts

### Contextual Tuples

OpenFGA 1.1+ supports contextual tuples, allowing for dynamic authorization decisions based on conditions:

```
type document
  relations
    define viewer: [user] with time_constraint
    define owner: [user]
```

Defining a condition type:

```json
{
  "name": "time_constraint",
  "parameters": {
    "starts_at": {
      "type": "timestamp"
    },
    "ends_at": {
      "type": "timestamp"
    }
  }
}
```

Using the condition in a check with C#:

```csharp
var check = await _fgaClient.Check(new CheckRequest
{
    User = "user:anne",
    Relation = "viewer",
    Object = "document:budget",
    Context = new Dictionary<string, string>
    {
        { "current_time", "2023-06-15T12:00:00Z" }
    }
});
```

### Type Aliases

Type aliases allow for more flexible modeling:

```
model
  schema 1.1

type user

type group
  relations
    define member: [user]

// Type alias for multiple object types
type resource
  relations
    define owner: [user, group#member]
    define can_access: owner

// Concrete types using alias
type document extends resource

type folder extends resource
  relations
    define parent: [folder]
    define can_access: owner or parent->can_access
```

### Custom User Types

OpenFGA allows for custom user types beyond the default:

```
model
  schema 1.1

type user

type service_account
  relations
    define admin: [user]

// Use service_account as a subject
type document
  relations
    define viewer: [user, service_account]
    define editor: [user, service_account]
```

### Direct and Indirect Checks

OpenFGA supports both direct relation checks and transitive permission checks:

```csharp
// Direct relation check
var isEditor = await _fgaClient.Check(new CheckRequest
{
    User = "user:anne",
    Relation = "editor",
    Object = "document:budget"
});

// Permission check (transitive through relationships)
var canEdit = await _fgaClient.Check(new CheckRequest
{
    User = "user:anne",
    Relation = "can_edit",
    Object = "document:budget"
});
```

## Deployment Options

### Self-Hosted

OpenFGA can be self-hosted using Docker:

```bash
docker run -p 8080:8080 openfga/openfga run
```

For production, use a more comprehensive configuration:

```yaml
# docker-compose.yml
version: '3.8'

services:
  openfga:
    image: openfga/openfga:latest
    ports:
      - "8080:8080"
      - "8081:8081"
      - "3000:3000"
    environment:
      - OPENFGA_DATASTORE_ENGINE=postgres
      - OPENFGA_DATASTORE_URI=postgres://postgres:password@postgres:5432/postgres
    command: run
    depends_on:
      - postgres
    restart: always

  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  postgres_data:
```

### Cloud-Hosted Options

Several cloud providers offer hosted OpenFGA:

1. **Auth0/Okta**: Integrated with their authorization services
2. **FGA.dev**: Dedicated hosted service for OpenFGA
3. **AWS/GCP/Azure**: Deploy using container services like ECS, GKE, or AKS

### High Availability Setup

For production deployments, consider:

1. **Multiple replicas**: Deploy multiple OpenFGA instances
2. **Load balancing**: Use a load balancer to distribute requests
3. **Database clustering**: Use a clustered database for reliability
4. **Monitoring**: Set up Prometheus and Grafana for observability

## Migration Strategies

### From RBAC to OpenFGA

If migrating from a role-based system:

1. **Map roles to relations**: Convert roles like "admin" to relations
2. **Identify resources**: Map protected resources to OpenFGA types
3. **Define permissions**: Create computed relations for permissions
4. **Migrate data**: Convert role assignments to relation tuples

Example RBAC to OpenFGA mapping:

**RBAC:**
```
User 123 has role ADMIN for Project 456
User 789 has role VIEWER for Project 456
```

**OpenFGA:**
```
{ user: "user:123", relation: "admin", object: "project:456" }
{ user: "user:789", relation: "viewer", object: "project:456" }
```

### From Custom Solutions

When migrating from a custom authorization solution:

1. **Audit existing permissions**: Document all permission types
2. **Design OpenFGA model**: Create a model that covers all scenarios
3. **Dual-write phase**: Write to both systems during transition
4. **Validation phase**: Compare authorization decisions between systems
5. **Cutover**: Switch to OpenFGA as the source of truth

### Data Migration Tools

OpenFGA provides tools for bulk imports in C#:

```csharp
// Bulk write example
await _fgaClient.Write(new WriteRequest
{
    Writes = new List<TupleKey>
    {
        new TupleKey { User = "user:1", Relation = "viewer", Object = "document:a" },
        new TupleKey { User = "user:2", Relation = "editor", Object = "document:a" },
        new TupleKey { User = "user:3", Relation = "owner", Object = "document:a" },
        // ... thousands more tuples
    }
});
```

For very large datasets, consider batching writes in chunks of 1000 tuples.

## Performance Optimization

### Query Optimization

1. **Use direct relations when possible**: Direct relation checks are faster than computed ones
2. **Limit relation depth**: Deep relation chains impact performance
3. **Use targeted queries**: Be specific in your relationship checks

### Caching Strategies

Implement caching at different levels:

1. **Client-side caching**: Cache common authorization decisions
2. **API gateway caching**: Cache at the API level
3. **OpenFGA's internal cache**: Configure appropriate cache settings

Example client-side cache in C#:

```csharp
public class AuthzCache
{
    private readonly Dictionary<string, CacheItem> _cache = new Dictionary<string, CacheItem>();
    private readonly TimeSpan _ttl;
    
    private class CacheItem
    {
        public bool Value { get; set; }
        public DateTime Expiry { get; set; }
    }
    
    public AuthzCache(TimeSpan? ttl = null)
    {
        _ttl = ttl ?? TimeSpan.FromMinutes(5);
    }
    
    private string GetCacheKey(string user, string relation, string obj)
    {
        return $"{user}:{relation}:{obj}";
    }
    
    public bool? Get(string user, string relation, string obj)
    {
        var key = GetCacheKey(user, relation, obj);
        
        if (!_cache.TryGetValue(key, out var item))
            return null;
            
        if (DateTime.UtcNow > item.Expiry)
        {
            _cache.Remove(key);
            return null;
        }
        
        return item.Value;
    }
    
    public void Set(string user, string relation, string obj, bool value)
    {
        var key = GetCacheKey(user, relation, obj);
        var expiry = DateTime.UtcNow.Add(_ttl);
        
        _cache[key] = new CacheItem { Value = value, Expiry = expiry };
    }
    
    public void Invalidate(string pattern)
    {
        var keysToRemove = _cache.Keys.Where(k => k.Contains(pattern)).ToList();
        foreach (var key in keysToRemove)
        {
            _cache.Remove(key);
        }
    }
}
```

### Bulk Operations

Use bulk operations to reduce API calls:

```csharp
// Bulk check
var batchCheckRequest = new BatchCheckRequest
{
    Checks = new List<CheckRequest>
    {
        new CheckRequest { User = "user:anne", Relation = "can_view", Object = "document:1" },
        new CheckRequest { User = "user:anne", Relation = "can_view", Object = "document:2" },
        new CheckRequest { User = "user:anne", Relation = "can_view", Object = "document:3" }
    }
};

var batchResponse = await _fgaClient.BatchCheck(batchCheckRequest);
// batchResponse.Responses contains an array of CheckResponse objects
```

## Security Considerations

### API Security

1. **Authentication**: Secure your OpenFGA API with strong authentication
2. **Authorization**: Limit who can write relationship tuples
3. **TLS**: Always use HTTPS for API communication
4. **Rate limiting**: Protect against DoS attacks

### Audit Logging

Implement comprehensive logging in C#:

```csharp
// Log all authorization checks
public async Task<bool> CheckWithAuditAsync(string user, string relation, string obj)
{
    var stopwatch = System.Diagnostics.Stopwatch.StartNew();
    bool result = false;
    
    try
    {
        var response = await _fgaClient.Check(new CheckRequest
        {
            User = user,
            Relation = relation,
            Object = obj
        });
        
        result = response.Allowed;
        return result;
    }
    finally
    {
        stopwatch.Stop();
        _logger.LogInformation(
            "AuthZ Check: User={User}, Relation={Relation}, Object={Object}, Result={Result}, Duration={Duration}ms", 
            user, relation, obj, result, stopwatch.ElapsedMilliseconds
        );
    }
}
```

### Regular Validation

Periodically validate your authorization model:

1. **Test critical paths**: Ensure key workflows have correct permissions
2. **Property-based testing**: Generate random authorization scenarios
3. **Regression tests**: Prevent regressions in permission logic

## Best Practices

### Modeling Best Practices

1. **Start simple**: Begin with basic relations and expand as needed
2. **Use computed relations**: Prefer computed relations for permissions checks
3. **Limit nesting depth**: Keep object hierarchies shallow for performance
4. **Be consistent**: Use consistent naming conventions
5. **Test extensively**: Validate with real-world scenarios
6. **Document your model**: Comment complex relationships for future reference

### Implementation Best Practices

1. **Separate concerns**: Decouple authorization logic from business logic
2. **Abstract the client**: Create an authorization service layer
3. **Handle errors gracefully**: Implement proper error handling
4. **Implement fallbacks**: Have fallback strategies for service unavailability
5. **Monitoring**: Set up alerts for authorization failures

### API Usage Patterns

```csharp
// Authorization service abstraction
public class AuthorizationService
{
    private readonly OpenFgaClient _fgaClient;
    private readonly AuthzCache _cache;
    private readonly ILogger<AuthorizationService> _logger;
    
    public AuthorizationService(
        OpenFgaClient fgaClient,
        ILogger<AuthorizationService> logger)
    {
        _fgaClient = fgaClient;
        _cache = new AuthzCache();
        _logger = logger;
    }
    
    public async Task<bool> IsAllowedAsync(string userId, string action, string resourceType, string resourceId)
    {
        var user = $"user:{userId}";
        var relation = MapActionToRelation(action, resourceType);
        var obj = $"{resourceType}:{resourceId}";
        
        // Check cache first
        var cachedResult = _cache.Get(user, relation, obj);
        if (cachedResult.HasValue)
        {
            return cachedResult.Value;
        }
        
        try
        {
            var response = await _fgaClient.Check(new CheckRequest
            {
                User = user,
                Relation = relation,
                Object = obj
            });
            
            // Cache result
            _cache.Set(user, relation, obj, response.Allowed);
            
            return response.Allowed;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Authorization check failed: {Message}", ex.Message);
            // Depending on your security posture, default to deny or allow
            return false;
        }
    }
    
    // Map actions like 'view', 'edit' to relations like 'can_view', 'can_edit'
    private string MapActionToRelation(string action, string resourceType)
    {
        var actionMap = new Dictionary<string, string>
        {
            { "view", "can_view" },
            { "edit", "can_edit" },
            { "delete", "can_delete" },
            { "share", "can_share" }
        };
        
        return actionMap.TryGetValue(action, out var relation) ? relation : action;
    }
    
    // Invalidate cache entries for a resource
    public void InvalidateResource(string resourceType, string resourceId)
    {
        _cache.Invalidate($"{resourceType}:{resourceId}");
    }
}
```

### Model Versioning

As your authorization needs evolve:

1. **Version your models**: Keep track of model changes
2. **Backwards compatibility**: Ensure new models support existing tuples
3. **Migration scripts**: Create scripts to update tuples for model changes
4. **Testing**: Test new models against existing authorization scenarios

## Conclusion

OpenFGA provides a powerful, flexible framework for implementing fine-grained authorization in modern applications. By modeling permissions as relationships between users and objects, you can express complex authorization rules in a way that's both intuitive and performant.

Key takeaways:

1. **Relationship-based**: Think in terms of relationships between users and resources
2. **Flexible modeling**: Use OpenFGA's DSL to model complex scenarios
3. **Performance-oriented**: Designed for low-latency authorization checks
4. **Developer-friendly**: Rich tooling and documentation
5. **Community-supported**: Active open-source community

Whether you're building a simple document sharing application or a complex multi-tenant SaaS platform, OpenFGA provides the tools you need to implement robust, scalable authorization. By starting with a solid model and following best practices, you can ensure your application's permissions system can grow and evolve with your business needs.

OpenFGA's relationship to Google's Zanzibar paper means it builds on proven concepts deployed at massive scale, while its open-source nature makes it accessible to developers and organizations of all sizes. The availability of both self-hosted and managed options provides flexibility in deployment, allowing you to choose the approach that best fits your infrastructure and security requirements.

By understanding and implementing the concepts covered in this tutorial, you'll be well-equipped to build sophisticated authorization systems that protect your application's resources while providing the flexibility your users need.
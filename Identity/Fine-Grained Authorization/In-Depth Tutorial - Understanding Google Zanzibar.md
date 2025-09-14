# 📘 In-Depth Tutorial: Understanding Google Zanzibar  
## A Beginner-Friendly Guide to Google's Global Authorization System

> "Zanzibar is Google’s global system for managing access control policies across all of its services." – [Google Research Paper](https://research.google/pubs/pub48190/ )

---

## 🧠 What is Google Zanzibar?

**Zanzibar** is a centralized authorization system developed by Google to manage **fine-grained access control** across all of Google's services at **global scale**.

It powers permissions in products like:
- Google Drive
- Google Calendar
- Google Docs
- YouTube (sharing features)
- And many others...

### 🔑 Key Features:
- Centralized policy management
- High scalability and availability
- Strong consistency
- Relationship-based access control (RBAC + ABAC hybrid)
- Supports complex hierarchies and nested groups
- Designed for multi-region, distributed environments

---

## 📚 Core Concepts of Zanzibar

Let’s break down the key concepts you need to understand before diving deeper.

---

### 1. **Namespaces**

A **namespace** represents a type of resource or subject in the system.

Examples:
```plaintext
/user
/document
/group
/folder
/workspace
```

Each namespace has:
- Relations (roles or connections between objects)
- Permissions (rules that combine relations)
- Optional caveats (conditions)

---

### 2. **Relations**

A **relation** defines how one object relates to another.

Example:
```plaintext
user:alice → viewer → document:doc1
group:admins#member → owner → folder:shared_folder
```

In Zanzibar terminology:

- **Subject**: The entity performing an action (`user:alice`)
- **Object**: The resource being accessed (`document:doc1`)
- **Relation**: How the subject is related to the object (`viewer`)

---

### 3. **Permissions**

A **permission** is a logical expression over relations.

Example:
```plaintext
permission view = viewer + owner
```
This means a user can `view` if they are either a `viewer` or an `owner`.

You can define multiple permissions per namespace.

---

### 4. **Relationships (Tuples)**

A relationship (or tuple) is an instance of a relation between two entities.

Structure:
```json
{
  "object": "document:doc1",
  "relation": "viewer",
  "subject": "user:alice"
}
```

This says: "Alice is a viewer of document doc1."

---

### 5. **Caveats (Conditions)**

**Caveats** allow conditional access control based on context or time.

Example:
```cel
caveat time_based_access(current_time: timestamp) {
  currentTime < current_time.addDuration("PT1H")
}
```

Apply this caveat to a relationship so that Alice only has access for one hour.

---

## 🏗️ Architecture Overview

Zanzibar is built to be **globally consistent**, **highly available**, and **horizontally scalable**.

Here's a simplified breakdown of its architecture:

### 🌐 1. **Global Consistency**
Zanzibar uses **TrueTime** (a time API from Google) and **Spanner** (Google's globally-distributed database) to ensure strong consistency across data centers worldwide.

### 💾 2. **Data Model**
All relationships are stored as **tuples** in a globally-replicated database.

### ⚙️ 3. **Query Processing**
When a service wants to check if a user can perform an action, it sends a query to Zanzibar, which resolves the relationship using recursive logic.

### 🧮 4. **Resolution Engine**
Zanzibar recursively evaluates relationships to determine whether a permission is granted. For example, checking if Alice can edit a document might involve checking group memberships, nested groups, and inherited roles.

---

## 🛠️ Step-by-Step: Modeling Access Control in Zanzibar

We’ll walk through building a simple access model for a document-sharing app similar to Google Drive.

---

### 🎯 Step 1: Define Namespaces

Start by defining the types of resources and users in your system.

#### Namespace: User
Represents individual users.
```plaintext
definition user {}
```

#### Namespace: Group
Represents a collection of users or nested groups.
```plaintext
definition group {
    relation member: user | group#member
}
```

#### Namespace: Document
Represents a document that can be shared.
```plaintext
definition document {
    relation owner: user | group#member
    relation viewer: user | group#member
    relation editor: user | group#member

    permission view = viewer + owner
    permission edit = editor + owner
}
```

Explanation:
- `owner`, `viewer`, and `editor` are **relations**
- `view` and `edit` are **permissions**
- The `+` operator combines relations

---

### 📥 Step 2: Write Schema to Zanzibar

You would typically use Zanzibar’s gRPC API to write the schema.

Example request (conceptual):
```protobuf
WriteSchemaRequest {
  schema = `
    definition user {}

    definition group {
        relation member: user | group#member
    }

    definition document {
        relation owner: user | group#member
        relation viewer: user | group#member
        relation editor: user | group#member

        permission view = viewer + owner
        permission edit = editor + owner
    }
  `
}
```

---

### 💾 Step 3: Create Relationships

Now we create actual relationships between users, groups, and documents.

#### Example 1: Assign ownership
Alice owns document `doc1`.

```protobuf
WriteRelationshipsRequest {
  updates = [
    {
      operation = CREATE
      relationship = {
        object = "document:doc1"
        relation = "owner"
        subject = { object = "user:alice" }
      }
    }
  ]
}
```

#### Example 2: Share document with a group
Group `writers` has edit access to `doc1`.

```protobuf
WriteRelationshipsRequest {
  updates = [
    {
      operation = CREATE
      relationship = {
        object = "document:doc1"
        relation = "editor"
        subject = { object = "group:writers", optional_relation = "member" }
      }
    }
  ]
}
```

---

### 🔍 Step 4: Check Permissions

To check if Alice can edit `doc1`:

```protobuf
CheckPermissionRequest {
  resource = "document:doc1"
  permission = "edit"
  subject = "user:alice"
}
```

If the response returns `"has_permission": true`, then access is allowed.

---

### 🔁 Step 5: Use Caveats (Optional)

Let’s say we want to give temporary access to Alice for 1 hour.

First, define the caveat:
```cel
caveat time_based_access(current_time: timestamp) {
  currentTime < current_time.addDuration("PT1H")
}
```

Then attach it to a relationship:

```protobuf
WriteRelationshipsRequest {
  updates = [
    {
      operation = CREATE
      relationship = {
        object = "document:doc1"
        relation = "viewer"
        subject = { object = "user:alice" }
        caveat = {
          name = "time_based_access"
          context = { "current_time": "2025-04-05T12:00:00Z" }
        }
      }
    }
  ]
}
```

Now Alice can only view the document until 1 PM UTC.

---

## 📌 Summary of Key Components

| Component     | Description |
|---------------|-------------|
| **Namespace** | Type of object/resource (e.g., user, document) |
| **Relation**  | Defines how one object relates to another (e.g., viewer, owner) |
| **Permission**| Logical rule combining relations (e.g., `view = viewer + owner`) |
| **Tuple**     | A concrete relationship between two objects |
| **Caveat**    | Conditional access based on context (e.g., time restrictions) |

---

## 🧪 Real-World Use Cases

### 1. **Document Sharing (Like Google Drive)**
- Users can share documents with individuals or groups.
- Nested groups allow for team-wide sharing.
- Caveats can expire links after a certain time.

### 2. **Multi-Tenant SaaS Applications**
- Each tenant is a namespace.
- Users are scoped to their tenant.
- Roles like admin, editor, viewer are defined via relations.

### 3. **Collaborative Platforms (e.g., Slack, Notion)**
- Channels/workspaces can have owners, members, guests.
- Fine-grained access to messages/files.
- Temporary invites using caveats.

---

## 📚 Additional Resources

- [Original Zanzibar Research Paper](https://research.google/pubs/pub48190/ )
- [SpiceDB - Open Source Implementation of Zanzibar](https://github.com/authzed/spicedb )
- [Authzed Playground](https://play.authzed.com/ )
- [OpenFGA - Another Zanzibar-inspired Open Source Project](https://openfga.dev/ )
- [Zanzibar Explained by Authzed Team](https://www.authzed.com/blog/zanzibar-explained/ )

---

## ✅ Final Notes

Zanzibar is not just an authorization system — it's a **paradigm shift** in how large-scale applications manage permissions. By modeling everything as relationships, Zanzibar provides a flexible, powerful, and maintainable way to reason about access control.

While Zanzibar itself is internal to Google, open-source projects like **SpiceDB** and **OpenFGA** offer faithful implementations of its core design, making it accessible to everyone.

---

## 📝 Next Steps

- Try modeling access control for your own application using SpiceDB or OpenFGA.
- Explore advanced topics like hierarchical namespaces, recursive relations, and performance optimization.
- Join the community on Discord or GitHub to ask questions and contribute!

---
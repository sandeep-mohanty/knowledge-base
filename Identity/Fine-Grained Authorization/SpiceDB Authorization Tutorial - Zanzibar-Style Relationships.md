# 📘 SpiceDB Authorization Tutorial: Zanzibar-Style Relationships

SpiceDB is an open-source database system inspired by Google's Zanzibar, purpose-built for managing complex authorization at scale. This tutorial will guide you through the basics of modeling permissions using SpiceDB’s schema language and Zanzibar-style relationships.

---

## 🧠 What is Zanzibar?

Zanzibar is Google's global-scale authorization system used to manage permissions across services like Drive, YouTube, and Calendar. It introduces a relationship-based access control (ReBAC) model where access is granted based on explicit relationships between subjects (users) and resources (objects).

---

## 🚀 Getting Started with SpiceDB

### ✅ Prerequisites

Before proceeding, make sure you have:

- Docker installed (for running SpiceDB locally)
- `zed` CLI installed (for managing schemas and relationships)

You can install `zed` using:

```bash
curl -L https://github.com/authzed/zed/releases/latest/download/zed-$(uname -s)-$(uname -m) -o zed
chmod +x zed
sudo mv zed /usr/local/bin
```

---

## 🏗️ Defining a Basic Authorization Model

Let's say you are building a document-sharing app. You want to model:

- Users
- Documents
- Access levels: viewer, editor, owner

### ✍️ Step 1: Write the Schema

Create a file called `schema.zed` with the following content:

```zed
definition user {}

definition document {
  relation viewer: user
  relation editor: user
  relation owner: user

  permission read = viewer + editor + owner
  permission write = editor + owner
  permission manage = owner
}
```

This schema defines:

- `user` as a subject
- `document` as an object that can have relations to users
- Permissions (`read`, `write`, `manage`) derived from relationships

---

## 📥 Step 2: Upload the Schema

```bash
zed schema write --file schema.zed
```

This command sends your schema to the SpiceDB instance.

---

## 🔗 Step 3: Create Relationships

Relationships are the core of Zanzibar-style authorization. They're stored as triples: **resource, relation, subject**.

For example, to say user `alice` is an `editor` of document `doc1`:

```bash
zed relationship create document:doc1#editor@user:alice
```

Add more relationships:

```bash
zed relationship create document:doc1#viewer@user:bob
zed relationship create document:doc1#owner@user:carol
```

---

## 🔍 Step 4: Check Permissions

To check if `bob` can `read` `doc1`:

```bash
zed permission check document:doc1 read user:bob
```

To check if `alice` can `write` `doc1`:

```bash
zed permission check document:doc1 write user:alice
```

To see all relationships:

```bash
zed relationship list
```

---

## 🧠 Advanced Concept: Indirect Relationships

You can define more complex models. For example, users can belong to groups, and groups have access to documents:

```zed
definition group {
  relation member: user
}

definition document {
  relation viewer: user | group#member
  relation editor: user | group#member
  relation owner: user | group#member

  permission read = viewer + editor + owner
  permission write = editor + owner
  permission manage = owner
}
```

Now, group members can be granted access via the group:

```bash
# Add alice to group devs
zed relationship create group:devs#member@user:alice

# Grant the group access to doc1
zed relationship create document:doc1#editor@group:devs
```

---

## 🧪 Testing Permissions with Expansion

To see why a user has a permission, use the `expand` command:

```bash
zed permission expand document:doc1 read
```

This shows the full tree of relationships contributing to the `read` permission.

---

## 🧰 Useful Commands

- **Write Schema**: `zed schema write --file schema.zed`
- **Read Schema**: `zed schema read`
- **Create Relationship**: `zed relationship create <resource>#<relation>@<subject>`
- **Delete Relationship**: `zed relationship delete <resource>#<relation>@<subject>`
- **Check Permission**: `zed permission check <resource> <permission> <subject>`
- **Expand Permission**: `zed permission expand <resource> <permission>`
- **List Relationships**: `zed relationship list`

---

## 🏁 Conclusion

This tutorial gave you a hands-on introduction to using SpiceDB with a Zanzibar-style authorization model. You learned how to:

- Define resources, relationships, and permissions
- Create and manage relationships
- Check and debug permissions

SpiceDB provides a scalable, flexible, and auditable way to implement fine-grained access control in your systems.

---

## 🔗 Resources

- [SpiceDB Documentation](https://authzed.com/docs/spicedb)
- [Zed CLI GitHub](https://github.com/authzed/zed)
- [Zanzibar Paper (by Google)](https://research.google/pubs/pub48190/)

---

Happy modeling! 🛠️
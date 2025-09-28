# 📘 SpiceDB Authorization Tutorial: Zanzibar-Style Relationships (Extended Edition)

SpiceDB is an **open-source fine-grained authorization database**, inspired by Google’s Zanzibar. It’s designed for apps that need **large-scale, flexible access control** across many users, groups, and resources.

This tutorial will help you understand SpiceDB with **diagrams, step-by-step examples, and mental models**. By the end, you’ll not only know the commands but also truly *picture* how SpiceDB models access.

---

## 🧠 What is Zanzibar?

Zanzibar is Google's global distributed authorization system used internally in services like Google Drive, YouTube, and Calendar. It introduced **Relationship-based Access Control (ReBAC)**, where **relationships** define access.

Instead of saying *“Bob has role=viewer on doc1”*, Zanzibar defines relationships:

```
document:doc1#viewer@user:bob
```

This states: **user:bob is a viewer of document:doc1**.

Access isn’t about static role checks. Instead, SpiceDB/ Zanzibar builds an **access graph**, and access is derived by walking connections.

---

## 📦 SpiceDB Mental Picture

Here’s how to visualize SpiceDB:

```plaintext
   user:alice
       │
       │ member
       ▼
   group:devs
       │
       │ editor
       ▼
 document:doc1
```

- Alice is **member** of `group:devs`
- `group:devs` is an **editor** of `document:doc1`
- SpiceDB resolves this → Alice is transitively an **editor** of `doc1`.

That’s the core magic: **relationship graph traversal**.

---

## 🚀 Getting Started with SpiceDB

### ✅ Prerequisites

- Docker: Run SpiceDB locally.
- Zed CLI: Manage schemas, permissions, and relationships.

Install Zed CLI:

```bash
curl -L https://github.com/authzed/zed/releases/latest/download/zed-$(uname -s)-$(uname -m) -o zed
chmod +x zed
sudo mv zed /usr/local/bin
```

---

## 🏗️ Step 1: Write a Basic Schema

Schemas define **resources (objects)**, **subjects (users/groups)**, and **how relations imply permissions**.

➡ Example: document-sharing app.

`schema.zed`

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

### Explanation
- **Relations**: link users to resources (`viewer: user`, `editor: user`).
- **Permissions**: derive from relations.
  - `read = viewer OR editor OR owner`
  - `write = editor OR owner`

---

## 📥 Step 2: Upload Schema

```bash
zed schema write --file schema.zed
```

This installs the schema into SpiceDB.

Use this to confirm:
```bash
zed schema read
```

---

## 🔗 Step 3: Create Relationships (Tuples)

Relationships are stored as **triples: resource#relation@subject**.

Examples:

```bash
# alice is an editor of doc1
zed relationship create document:doc1#editor@user:alice

# bob is a viewer
zed relationship create document:doc1#viewer@user:bob

# carol is the owner
zed relationship create document:doc1#owner@user:carol
```

---

## 🔍 Step 4: Check Permissions

Ask SpiceDB: *Can this user do this thing?*

```bash
zed permission check document:doc1 read user:bob
# ✅ allowed, because bob is a viewer
```

```bash
zed permission check document:doc1 write user:alice
# ✅ allowed, because alice is an editor
```

```bash
zed permission check document:doc1 manage user:bob
# ❌ denied, only carol (owner) can manage
```

---

## 🧠 Step 5: Indirect Relationships (Groups!)

Let’s extend model: users can belong to groups, and groups can hold permissions.

`schema.zed`

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

### Tuples

```bash
# alice is a member of group devs
zed relationship create group:devs#member@user:alice

# group devs is an editor of doc1
zed relationship create document:doc1#editor@group:devs
```

Now:
```bash
zed permission check document:doc1 write user:alice
# ✅ allowed via group devs
```

---

## 🧪 Step 6: Explain Permissions with Expansion

Want to see *why* Alice can write?

```bash
zed permission expand document:doc1 write
```

This shows a **tree of relationships**:

```plaintext
write
 ├─ editor
 │   └─ group:devs#member
 │       └─ user:alice
```

So permissions aren’t a mystery — SpiceDB gives a clear explanation tree.

---

## 🧰 Useful Zed CLI Commands

- **Schemas**
  - Write Schema → `zed schema write --file schema.zed`
  - Read Schema → `zed schema read`

- **Relationships**
  - Create → `zed relationship create <resource>#<relation>@<subject>`
  - Delete → `zed relationship delete <resource>#<relation>@<subject>`
  - List → `zed relationship list`

- **Permissions**
  - Check → `zed permission check <resource> <permission> <subject>`
  - Expand → `zed permission expand <resource> <permission>`

---

## 🌍 Example: Nested Hierarchies (Orgs > Projects > Docs)

SpiceDB handles hierarchies like orgs/projects.

```zed
definition organization {
  relation admin: user
}

definition project {
  relation parent: organization
  relation member: user
}

definition document {
  relation parent: project
  relation editor: user | project#member

  permission write = editor + parent->member
}
```

- If `organization` admin → inherits control over projects & docs.
- If `project` member → automatically can edit docs in project.

---

## 📊 Diagram of Hierarchy

```plaintext
organization:acme#admin@user:carol
       │
       └── parent
           │
     project:billing#member@user:bob
           │
           └── parent
               │
          document:invoice123
```

Carol (org admin) gets top-level access;
Bob (project member) gets doc edit rights via project-parent relation.

---

## 🏁 Conclusion

In this tutorial, you’ve learned:

1. Define a **schema** with relations → permissions.
2. Create **relationships** (subject-resource links).
3. **Check** user permissions quickly.
4. **Expand** permissions to **explain why** they’re granted.
5. Scale up to **groups and nested hierarchies**.

SpiceDB lets you **model flexible, fine-grained authorization at scale** the Zanzibar way.

---

## 🔗 Resources

- [SpiceDB Documentation](https://authzed.com/docs/spicedb)
- [Zed CLI GitHub](https://github.com/authzed/zed)
- [Zanzibar Paper](https://research.google/pubs/pub48190/)
- [SpiceDB Playground](https://play.authzed.com)

---

🚀 With SpiceDB, you’re not sprinkling `if(role == admin)` all over your code. Instead, you’re building a **centralized, graph-powered access model** that scales with your app.
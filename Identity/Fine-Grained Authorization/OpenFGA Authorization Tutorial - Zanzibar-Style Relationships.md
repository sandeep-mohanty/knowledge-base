# 🛡️ OpenFGA Authorization Tutorial: Zanzibar-Style Relationships

[OpenFGA](https://openfga.dev) is an open-source fine-grained authorization system inspired by Google's Zanzibar. It is designed for high performance and scalability and is ideal for modern applications. This tutorial walks you through creating a Zanzibar-style authorization model using OpenFGA.

---

## 📦 What is OpenFGA?

OpenFGA (Fine-Grained Authorization) manages permissions by modeling relationships between users and resources. It supports logic such as:

- A user can `read` a document if they are a viewer, editor, or owner.
- A user can `write` a document if they are an editor or owner.

These relationships are defined in a **type system** and evaluated at runtime.

---

## ✅ Prerequisites

To follow along, make sure you have:

- Docker installed
- OpenFGA running locally (or use Auth0 FGA cloud)
- [FGA CLI](https://github.com/openfga/cli) installed

Install FGA CLI:

```bash
brew install openfga/tap/fga
```

Or download from GitHub Releases: https://github.com/openfga/cli/releases

---

## 🚀 Running OpenFGA Locally (Optional)

You can run OpenFGA using Docker:

```bash
docker run --rm -p 8080:8080 openfga/openfga
```

The service will be available at `http://localhost:8080`.

---

## 🏗️ Step 1: Define the Authorization Model

We will model a document-sharing system with users, groups, and documents.

### 🧩 Model Overview

- **Users** can be viewers, editors, or owners of documents.
- **Groups** can contain users.
- **Documents** have relationships to both users and groups.

---

### 📝 Authorization Model Definition

Save this as `model.json`:

```json
{
  "type_definitions": [
    {
      "type": "user"
    },
    {
      "type": "group",
      "relations": {
        "member": {
          "this": {}
        }
      }
    },
    {
      "type": "document",
      "relations": {
        "viewer": {
          "union": {
            "child": [
              { "this": {} },
              { "computedUserset": { "relation": "member", "type": "group" } }
            ]
          }
        },
        "editor": {
          "union": {
            "child": [
              { "this": {} },
              { "computedUserset": { "relation": "member", "type": "group" } }
            ]
          }
        },
        "owner": {
          "union": {
            "child": [
              { "this": {} },
              { "computedUserset": { "relation": "member", "type": "group" } }
            ]
          }
        }
      },
      "metadata": {
        "relations": {
          "viewer": { "directly_related_user_types": [ { "type": "user" }, { "type": "group", "relation": "member" } ] },
          "editor": { "directly_related_user_types": [ { "type": "user" }, { "type": "group", "relation": "member" } ] },
          "owner": { "directly_related_user_types": [ { "type": "user" }, { "type": "group", "relation": "member" } ] }
        }
      }
    }
  ]
}
```

### 🔑 Define Permissions (in your app logic):

- `read = viewer OR editor OR owner`
- `write = editor OR owner`
- `manage = owner`

OpenFGA does not define permissions explicitly — you compute them via relationships.

---

## 📥 Step 2: Upload the Model

Use the FGA CLI to upload the model:

```bash
fga model create --file model.json
```

Note the returned model ID — you'll need it for subsequent API calls.

---

## 🔗 Step 3: Create Relationship Tuples

Tuples define who has what access.

```bash
# Alice is an editor of document:doc1
fga tuple write --user "user:alice" --relation "editor" --object "document:doc1"

# Bob is a viewer
fga tuple write --user "user:bob" --relation "viewer" --object "document:doc1"

# Carol is the owner
fga tuple write --user "user:carol" --relation "owner" --object "document:doc1"

# Add alice to group:devs
fga tuple write --user "user:alice" --relation "member" --object "group:devs"

# Grant group:devs editor access to doc1
fga tuple write --user "group:devs" --relation "editor" --object "document:doc1"
```

---

## 🔍 Step 4: Check Access

To check if `alice` can `write` to `doc1`:

```bash
fga check --user "user:alice" --relation "editor" --object "document:doc1"
```

To check if `bob` can `read` (you must check viewer, editor, AND owner):

```bash
fga check --user "user:bob" --relation "viewer" --object "document:doc1"
```

Repeat for `editor` and `owner` to fully simulate `read`.

---

## 🧪 Step 5: Explain Access

Use the **explain** feature to understand *why* access is granted:

```bash
fga explain --user "user:alice" --relation "editor" --object "document:doc1"
```

This gives you a tree of relationships showing how access is derived.

---

## 🧰 Useful FGA CLI Commands

- `fga model create --file model.json`
- `fga tuple write --user <user> --relation <rel> --object <object>`
- `fga check --user <user> --relation <rel> --object <object>`
- `fga explain --user <user> --relation <rel> --object <object>`
- `fga tuple list`
- `fga store list`
- `fga model list`

---

## 🏁 Conclusion

You’ve just built a Zanzibar-style authorization model using OpenFGA!

You learned how to:

- Define a type system using JSON
- Model relationships between users, groups, and documents
- Check and explain access using tuple relationships
- Build flexible access control logic in your app

---

## 🔗 Resources

- [OpenFGA Docs](https://openfga.dev/docs)
- [FGA CLI GitHub](https://github.com/openfga/cli)
- [Zanzibar Paper (by Google)](https://research.google/pubs/pub48190/)
- [OpenFGA Playground](https://playground.openfga.dev/)

---

🛠️ Happy modeling with OpenFGA!
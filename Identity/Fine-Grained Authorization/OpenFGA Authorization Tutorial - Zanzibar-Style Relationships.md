# 🛡️ OpenFGA Authorization Tutorial: Zanzibar-Style Relationships (Extended Edition)

[OpenFGA](https://openfga.dev) is an **open-source** fine-grained authorization system inspired by Google’s Zanzibar paper. It lets you model **who can access what** using a **graph of relationships (tuples)** between *users, resources, and permissions*.  

Think of it as a **social graph for authorization**: just like Facebook models *friends, groups, and posts*, you model *users, groups, and resources*.

---

## 🖼️ The Mental Model (Diagram)

Here’s the big picture:

```plaintext
                 ┌───────────────┐
                 │    user:alice │
                 └───────┬───────┘
                         │
         "member"        │
                         │
                 ┌───────▼───────┐
                 │  group:devs   │
                 └───────┬───────┘
                         │ "editor"
                         │
                 ┌───────▼─────────┐
                 │ document:doc1   │
                 │ [editor,owner]  │
                 └─────────────────┘

```

- Alice is a **member** of group:devs
- devs group has **editor** rights on doc1
- Therefore, Alice is transitively an **editor** of doc1

That’s the Zanzibar “graph style” authorization.

---

## 📦 What is OpenFGA?

**OpenFGA** = “Fine-Grained Authorization”.

It lets you define:

- **Types:** user, document, folder, group, project, etc.
- **Relations:** viewer, editor, member, owner, etc.
- **Tuples:** `user bob is a viewer of document:doc1`
- **Checks:** Is Alice allowed to `write` doc1?

Instead of encoding ACL logic into your app, you offload relationship storage and **query evaluation** to OpenFGA.

---

## ✅ Prerequisites

- Install **Docker**
- Run **OpenFGA server** locally:
  ```bash
  docker run --rm -p 8080:8080 openfga/openfga
  ```
  available at `http://localhost:8080`.

- Install **FGA CLI** (command-line tool):
  ```bash
  brew install openfga/tap/fga
  ```

---

## 🏗️ Step 1: Define an Authorization Model

Create a file `model.json`:

```json
{
  "type_definitions": [
    {
      "type": "user"
    },
    {
      "type": "group",
      "relations": {
        "member": { "this": {} }
      }
    },
    {
      "type": "document",
      "relations": {
        "viewer": {
          "union": {
            "child": [
              { "this": {} },
              { "computedUserset": { "type": "group", "relation": "member" } }
            ]
          }
        },
        "editor": {
          "union": {
            "child": [
              { "this": {} },
              { "computedUserset": { "type": "group", "relation": "member" } }
            ]
          }
        },
        "owner": { "this": {} }
      },
      "metadata": {
        "relations": {
          "viewer": {
            "directly_related_user_types": [
              { "type": "user" }, { "type": "group", "relation": "member" }
            ]
          },
          "editor": {
            "directly_related_user_types": [
              { "type": "user" }, { "type": "group", "relation": "member" }
            ]
          },
          "owner": {
            "directly_related_user_types": [
              { "type": "user" }
            ]
          }
        }
      }
    }
  ]
}
```

---

### 📝 Permissions in Application Logic

OpenFGA doesn’t define permissions directly, you define **relations** and **derive permissions** in app logic:

- `read = viewer OR editor OR owner`
- `write = editor OR owner`
- `manage = owner`

This keeps OpenFGA focused on graphs, while your API decides the semantics.

---

## 📥 Step 2: Upload the Model

```bash
fga model create --file model.json
```

This returns a `model_id`. Remember it.

---

## 🔗 Step 3: Create Tuples (Relationships)

Tuples = facts. Who can do what.

```bash
# Alice is editor of doc1
fga tuple write --user "user:alice" --relation "editor" --object "document:doc1"

# Bob is viewer
fga tuple write --user "user:bob" --relation "viewer" --object "document:doc1"

# Carol is owner
fga tuple write --user "user:carol" --relation "owner" --object "document:doc1"

# Alice is a member of devs
fga tuple write --user "user:alice" --relation "member" --object "group:devs"

# Group devs is editor of doc1
fga tuple write --user "group:devs" --relation "editor" --object "document:doc1"
```

---

## 🔍 Step 4: Check Access

Ask: *Does Alice have editor rights on doc1?*

```bash
fga check --user "user:alice" --relation "editor" --object "document:doc1"
```

Ask: *Does Bob have read rights?*
(because read = viewer OR editor OR owner):

```bash
fga check --user "user:bob" --relation "viewer" --object "document:doc1"
```

You can repeat for `editor` or `owner`.

---

## 🧪 Step 5: Explain Access

Why does Alice have editor rights? Let’s see:

```bash
fga explain --user "user:alice" --relation "editor" --object "document:doc1"
```

Output will show:

- Alice → member of devs
- devs → editor of document
- therefore Alice → editor of document

This lets you debug authorization graphs.

---

## 🔄 Extra Example: Nested Resources

Beyond documents, model **folders/projects**:

```json
{
  "type": "folder",
  "relations": {
    "viewer": { "this": {} },
    "editor": { "this": {} },
    "owner": { "this": {} },
    "parent": { "this": {} }
  }
}
```

- `folder:finance` is parent of `document:budget-report`
- If you’re `viewer` of a folder, you inherit `viewer` of contained docs.

This lets you scale Zanzibar logic across hierarchical systems.

---

## 🧰 Useful CLI Commands

- `fga model create --file model.json` – upload model
- `fga model list` – list models
- `fga tuple write` – add relationships
- `fga tuple list` – view stored tuples
- `fga check` – check access
- `fga explain` – why access is granted

---

## 🌟 Best Practices

- **Keep permissions simple**: model as relations (`viewer`, `editor`, `owner`). Compute permissions in app code.
- **Use groups for scaling**: instead of giving 100 users rights individually, put them in a group.
- **Leverage explain**: always debug access graphs with `fga explain`.
- **Hierarchies**: use “parent” relations to cascade permissions across folders/projects.
- **Consistency**: keep `read`, `write`, `manage` logic consistent in your application layer.

---

## 🏁 Conclusion

We just built a Zanzibar-style auth model with OpenFGA:

- Defined **types & relations**
- Created **tuples** to model access graphs
- Queried rights with `fga check`
- Understood graph traversal with `fga explain`
- Extended the model to nested resources

---

## 🔗 Resources

- [OpenFGA Docs](https://openfga.dev/docs)
- [Zanzibar Research Paper](https://research.google/pubs/pub48190/)
- [FGA Playground (try before coding!)](https://playground.openfga.dev/)

---

🛠️ With OpenFGA, you’re not writing endless role-check if-statements. You’re building an **authorization graph** — scalable, debuggable, and elegant.
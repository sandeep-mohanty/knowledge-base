# OpenFGA DSL Question Bank

## Table of Contents

1. [Basic Concepts (Questions 1-10)](#basic-concepts)
2. [Type Definitions (Questions 11-20)](#type-definitions)
3. [Relations (Questions 21-30)](#relations)
4. [Operators (Questions 31-45)](#operators)
5. [Hierarchical Relations (Questions 46-60)](#hierarchical-relations)
6. [Advanced Patterns (Questions 61-75)](#advanced-patterns)
7. [Real-World Scenarios (Questions 76-90)](#real-world-scenarios)
8. [Debugging and Troubleshooting (Questions 91-100)](#debugging-and-troubleshooting)

***

## Basic Concepts

### Question 1
**What are the three main components of an OpenFGA authorization model?**

**Answer:**
The three main components are:
1. **Types**: Categories of objects/resources (e.g., document, folder, user)
2. **Relations**: Connections between objects (e.g., owner, viewer, member)
3. **Permissions**: What actions are allowed based on relations (computed from relations)

**Explanation:**
Types represent the "nouns" in your system, relations represent the "verbs" or connections, and permissions are the "rules" that determine access.

---

### Question 2
**What is the difference between a tuple and a relation?**

**Answer:**
- **Relation**: A definition in the model specifying a possible relationship type (e.g., `define owner: [user]`)
- **Tuple**: An actual stored instance of that relationship (e.g., `(user:alice, owner, document:doc1)`)

**Explanation:**
Think of a relation as a "template" or "schema" and a tuple as the "data" that conforms to that schema. Relations are defined in `.fga` files, tuples are stored in the OpenFGA database.

***

### Question 3
**What does this model define?**

```fga
model
  schema 1.1

type user

type document
  relations
    define owner: [user]
```

**Answer:**
This model defines:
- Two types: `user` and `document`
- One relation: `owner` on `document` type
- The `owner` relation can only contain references to `user` type objects

**Explanation:**
This is a minimal model for document ownership. To store data, you would create tuples like `(user:alice, owner, document:doc1)`.

***

### Question 4
**What is the purpose of the `schema 1.1` declaration?**

**Answer:**
The `schema 1.1` declaration specifies which version of the OpenFGA DSL is being used.

**Explanation:**
Schema versions determine which language features are available. Version 1.1 is the current version and includes features like:
- `but not` operator
- Improved type syntax
- Better error messages

Always use `schema 1.1` for new models.

***

### Question 5
**Can a type have no relations?**

**Answer:**
Yes, a type can have no relations.

**Example:**
```fga
type user
```

**Explanation:**
Types without relations are typically used as "principals" (actors) in the authorization model. They represent entities that perform actions but don't have complex relationships defined. The `user` type commonly has no relations.

***

### Question 6
**What is the difference between a direct relation and a computed relation?**

**Answer:**
- **Direct relation**: Stored as tuples in the database (e.g., `define owner: [user]`)
- **Computed relation**: Calculated from other relations using logic (e.g., `define can_view: owner or editor`)

**Example:**
```fga
type document
  relations
    define owner: [user]           # Direct - stored as tuple
    define editor: [user]          # Direct - stored as tuple
    define can_view: owner or editor  # Computed - not stored
```

**Explanation:**
Direct relations require explicit tuple storage. Computed relations are evaluated at query time based on the logic defined.

***

### Question 7
**Can you store a tuple for a computed relation?**

**Answer:**
No, you cannot store tuples for computed relations.

**Explanation:**
Computed relations are derived from other relations. OpenFGA will reject attempts to write tuples for computed relations. Only direct relations can have tuples stored.

**Example:**
```fga
define can_view: owner or editor  # Computed - no tuples stored
```

If you try:
```
fga tuple write (user:alice, can_view, document:doc1)  # ERROR!
```

***

### Question 8
**What does `[user]` mean in a relation definition?**

**Answer:**
`[user]` specifies the allowed type(s) that can be assigned to this relation.

**Example:**
```fga
define owner: [user]
```

This means only objects of type `user` can be owners. Valid tuples:
```
(user:alice, owner, document:doc1)  ✓
(document:doc2, owner, document:doc1)  ✗ Wrong type
```

**Explanation:**
The square brackets `[...]` define type constraints. This ensures type safety in your authorization model.

***

### Question 9
**Can a relation accept multiple types?**

**Answer:**
Yes, you can specify multiple allowed types separated by commas.

**Example:**
```fga
define owner: [user, team#member]
```

This allows:
```
(user:alice, owner, document:doc1)           ✓ Direct user
(team:engineering#member, owner, document:doc2)  ✓ Team members
```

**Explanation:**
Multiple types provide flexibility. In this case, both individual users and entire team memberships can be owners.

***

### Question 10
**What does `user:*` represent?**

**Answer:**
`user:*` is a wildcard that represents "any user" or "all users".

**Example:**
```fga
type document
  relations
    define public: [user:*]
    define can_view: public or owner
```

**Tuple:**
```
(user:*, public, document:terms)
```

**Explanation:**
This makes the document publicly accessible to all users. Any check for `can_view` will return true for any user on this document.

***

## Type Definitions

### Question 11
**What's wrong with this model?**

```fga
model
  schema 1.1

type document
  relations
    define owner: [person]
```

**Answer:**
The type `person` is not defined in the model. All types referenced in relations must be defined.

**Fix:**
```fga
model
  schema 1.1

type person

type document
  relations
    define owner: [person]
```

**Explanation:**
OpenFGA requires all types to be explicitly defined before they can be referenced. The model must be self-contained.

***

### Question 12
**Can you reference a type that's defined later in the file?**

**Answer:**
Yes, forward references are allowed in OpenFGA.

**Example:**
```fga
type document
  relations
    define folder: [folder]  # folder defined below

type folder
  relations
    define parent: [folder]
```

**Explanation:**
OpenFGA parses the entire model before validation, so type definition order doesn't matter.

---

### Question 13
**What does this relation definition mean?**

```fga
define viewer: [user, group#member]
```

**Answer:**
The `viewer` relation can contain:
1. Direct user references (e.g., `user:alice`)
2. Group member references (e.g., `group:engineering#member`)

**Stored tuples:**
```
(user:alice, viewer, document:doc1)           # Direct user
(group:engineering#member, viewer, document:doc2)  # All members of group
```

**Explanation:**
The `group#member` syntax means "users who have the `member` relation to a `group`". This enables group-based permissions.

***

### Question 14
**How would you model a resource that can be owned by users OR other resources of the same type?**

**Answer:**
```fga
type folder
  relations
    define owner: [user, folder#owner]
```

**Example tuples:**
```
(user:alice, owner, folder:root)           # User owns root
(folder:root#owner, owner, folder:child)   # Root's owners own child
```

**Explanation:**
This creates cascading ownership. If Alice owns root and root owns child, then Alice effectively owns child through the ownership chain.

***

### Question 15
**What happens if you define the same relation twice?**

```fga
type document
  relations
    define owner: [user]
    define owner: [user, team#member]
```

**Answer:**
This will cause a **validation error**. Each relation can only be defined once per type.

**Fix:**
```fga
type document
  relations
    define owner: [user, team#member]  # Define once with all types
```

**Explanation:**
Duplicate relation definitions are not allowed. If you need to add more types, include them all in a single definition.

***

### Question 16
**Can two different types have a relation with the same name?**

**Answer:**
Yes, different types can have relations with the same name.

**Example:**
```fga
type document
  relations
    define owner: [user]

type folder
  relations
    define owner: [user]
```

**Explanation:**
Relations are scoped to their type. `document#owner` and `folder#owner` are distinct relations even though they share the name "owner".

***

### Question 17
**What's the difference between these two models?**

**Model A:**
```fga
type document
  relations
    define owner: [user]
    define viewer: [user]
```

**Model B:**
```fga
type document
  relations
    define owner: [user]
    define can_view: owner
```

**Answer:**
- **Model A**: Has two direct relations (`owner` and `viewer`). Both require explicit tuples.
- **Model B**: Has one direct relation (`owner`) and one computed relation (`can_view`). Only `owner` tuples are stored; `can_view` is evaluated from `owner`.

**Tuples for Model A:**
```
(user:alice, owner, document:doc1)
(user:bob, viewer, document:doc1)
```

**Tuples for Model B:**
```
(user:alice, owner, document:doc1)
# No viewer tuple needed - bob can't view unless he's owner
```

**Explanation:**
Model A provides explicit viewer control. Model B derives viewing permission from ownership.

***

### Question 18
**Can a type reference itself?**

**Answer:**
Yes, self-referential types are allowed.

**Example:**
```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    
    define can_view: owner or owner from parent
```

**Explanation:**
This models hierarchical structures like folder trees. A folder can have another folder as its parent, creating nested hierarchies.

***

### Question 19
**What does this model allow?**

```fga
type user
  relations
    define follower: [user]
```

**Answer:**
This allows users to follow other users (like social media).

**Example tuples:**
```
(user:bob, follower, user:alice)      # Bob follows Alice
(user:charlie, follower, user:alice)  # Charlie follows Alice
```

**Query:**
```
Check: user:bob follower user:alice
Result: true
```

**Explanation:**
Users can have relationships with other users of the same type. This is common in social networks.

***

### Question 20
**Why might you create a type with no relations?**

**Answer:**
Types with no relations serve as:
1. **Principals/Actors**: Entities that perform actions (e.g., `user`)
2. **Simple identifiers**: Objects that are referenced but don't have complex relationships
3. **Future extensibility**: Placeholder for future relations

**Example:**
```fga
type user  # No relations - just an actor

type api_key  # No relations - just an identifier

type document
  relations
    define owner: [user]
    define api_key: [api_key]
```

***

## Relations

### Question 21
**Given this model, which tuples are valid?**

```fga
type user

type document
  relations
    define owner: [user]
    define editor: [user]
```

**Options:**
```
A) (user:alice, owner, document:doc1)
B) (user:bob, viewer, document:doc1)
C) (document:doc2, owner, document:doc1)
D) (user:charlie, editor, document:doc1)
```

**Answer:**
Valid: **A** and **D**

**Explanation:**
- A: ✓ Valid - user can be owner
- B: ✗ Invalid - `viewer` relation not defined
- C: ✗ Invalid - `document` not allowed as owner (only `[user]`)
- D: ✓ Valid - user can be editor

---

### Question 22
**What does this relation definition enable?**

```fga
type group
  relations
    define member: [user, group#member]
```

**Answer:**
This enables **nested group membership**. Groups can contain:
1. Direct users
2. Members of other groups (indirect users)

**Example:**
```
(user:alice, member, group:engineers)
(user:bob, member, group:seniors)
(group:seniors#member, member, group:engineers)
```

Now:
- Alice is directly a member of engineers
- Bob is directly a member of seniors
- Bob is indirectly a member of engineers (through seniors)

**Explanation:**
The `group#member` syntax creates transitive membership, allowing hierarchical group structures.

***

### Question 23
**What's the difference between these two relation definitions?**

```fga
# Definition A
define viewer: [user]

# Definition B
define can_view: viewer
```

**Answer:**
- **Definition A**: Direct relation requiring tuple storage
- **Definition B**: Computed relation derived from another relation

**Storage:**
- A: Requires tuple: `(user:alice, viewer, document:doc1)`
- B: No tuple stored, evaluated from `viewer` relation

**Explanation:**
Definition A is a "base" relation. Definition B is a "permission" derived from the base relation.

***

### Question 24
**Can you write a tuple for this relation?**

```fga
define can_view: owner or editor or viewer
```

**Answer:**
No, you cannot write tuples for computed relations.

**Explanation:**
`can_view` is computed from other relations. OpenFGA evaluates it at query time by checking if the user has any of: `owner`, `editor`, or `viewer` relations.

**Error if attempted:**
```
fga tuple write user:alice can_view document:doc1
Error: cannot write to computed relation
```

***

### Question 25
**What does this query check?**

```
Check: user:alice owner document:doc1
```

**Answer:**
This checks if there exists a tuple `(user:alice, owner, document:doc1)` in the database.

**Possible results:**
- `true`: Tuple exists
- `false`: Tuple doesn't exist

**Explanation:**
For direct relations, OpenFGA simply checks for tuple existence. For computed relations, it evaluates the logic.

***

### Question 26
**Given these tuples:**

```
(user:alice, owner, document:doc1)
(user:bob, editor, document:doc1)
```

**And this model:**

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define can_view: owner or editor
```

**What does this check return?**

```
Check: user:bob can_view document:doc1
```

**Answer:**
`true`

**Explanation:**
Evaluation:
1. `can_view = owner or editor`
2. Is bob owner? No
3. Is bob editor? Yes ✓
4. Result: true (because `or` requires only one condition to be true)

***

### Question 27
**What's wrong with this definition?**

```fga
type document
  relations
    define can_view: owner or editor
```

**Answer:**
The relations `owner` and `editor` are not defined in the type.

**Fix:**
```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define can_view: owner or editor
```

**Explanation:**
Computed relations can only reference other relations that exist in the same type. You must define base relations before using them in computed relations.

***

### Question 28
**Can a computed relation reference another computed relation?**

**Answer:**
Yes, computed relations can reference other computed relations.

**Example:**
```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    define can_edit: owner or editor
    define can_view: viewer or can_edit  # References can_edit
```

**Explanation:**
This creates a hierarchy of permissions. Anyone who `can_edit` also `can_view`. OpenFGA resolves these recursively.

***

### Question 29
**What type of relation is `parent` in this model?**

```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
```

**Answer:**
`parent` is a **direct relation** that creates a link to another folder.

**Example tuple:**
```
(folder:child, parent, folder:parent)
```

**Explanation:**
This establishes a parent-child relationship between folders. It's a direct relation because it requires explicit tuple storage.

---

### Question 30
**Can relations have multiple levels of indirection?**

**Answer:**
Yes, you can chain multiple `from` clauses.

**Example:**
```fga
type organization
  relations
    define member: [user]

type team
  relations
    define organization: [organization]
    define member: [user]

type project
  relations
    define team: [team]
    define owner: [user]
    
    define can_access: owner or member from team or member from organization from team
```

**Explanation:**
This allows:
1. Project owner to access
2. Team members to access
3. Organization members to access (through team's organization)

***

## Operators

### Question 31
**What does the `or` operator do?**

**Answer:**
The `or` operator returns `true` if **any** of the conditions are true.

**Example:**
```fga
define can_view: owner or editor or viewer
```

**Truth table:**
```
owner=true,  editor=false, viewer=false → true
owner=false, editor=true,  viewer=false → true
owner=false, editor=false, viewer=true  → true
owner=false, editor=false, viewer=false → false
```

**Explanation:**
It's a logical OR - at least one condition must be satisfied.

***

### Question 32
**What does the `and` operator do?**

**Answer:**
The `and` operator returns `true` only if **all** conditions are true.

**Example:**
```fga
define can_publish: owner and approved
```

**Truth table:**
```
owner=true,  approved=true  → true
owner=true,  approved=false → false
owner=false, approved=true  → false
owner=false, approved=false → false
```

**Explanation:**
Both conditions must be satisfied for the check to pass.

***

### Question 33
**Given this model and tuples:**

**Model:**
```fga
type document
  relations
    define authorized: [user]
    define certified: [user]
    define can_publish: authorized and certified
```

**Tuples:**
```
(user:alice, authorized, document:doc1)
(user:alice, certified, document:doc1)
(user:bob, authorized, document:doc1)
```

**What is the result of this check?**
```
Check: user:bob can_publish document:doc1
```

**Answer:**
`false`

**Explanation:**
Evaluation:
1. `can_publish = authorized and certified`
2. Is bob authorized? Yes ✓
3. Is bob certified? No ✗
4. Result: false (both conditions must be true for `and`)

***

### Question 34
**What does the `but not` operator do?**

**Answer:**
The `but not` operator returns `true` if the first condition is true AND the second condition is false.

**Example:**
```fga
define can_access: member but not blocked
```

**Truth table:**
```
member=true,  blocked=false → true
member=true,  blocked=true  → false
member=false, blocked=false → false
member=false, blocked=true  → false
```

**Explanation:**
This is an exclusion operator - it removes a subset from a set.

***

### Question 35
**Given this model:**

```fga
type document
  relations
    define member: [user]
    define banned: [user]
    define can_access: member but not banned
```

**Tuples:**
```
(user:alice, member, document:doc1)
(user:bob, member, document:doc1)
(user:bob, banned, document:doc1)
```

**What are the results of these checks?**

```
A) Check: user:alice can_access document:doc1
B) Check: user:bob can_access document:doc1
```

**Answer:**
- A: `true` (alice is member and NOT banned)
- B: `false` (bob is member BUT is banned)

**Explanation:**
The `but not` operator excludes banned members from accessing, even though they're members.

***

### Question 36
**Can you combine multiple operators?**

**Answer:**
Yes, you can combine `or`, `and`, and `but not`.

**Example:**
```fga
define can_edit: (owner or editor) but not suspended
```

**Explanation:**
Evaluation order:
1. First evaluate: `owner or editor`
2. Then apply: `but not suspended`
3. Result: (owner OR editor) AND NOT suspended

Use parentheses for clarity, though they're not always required.

***

### Question 37
**What's the difference between these two definitions?**

```fga
# Definition A
define can_access: (owner or editor) but not banned

# Definition B
define can_access: owner or (editor but not banned)
```

**Answer:**
- **Definition A**: Banned users can't access, even if they're owners or editors
- **Definition B**: Owners can always access; only editors are affected by the ban

**Truth table:**

| owner | editor | banned | Definition A | Definition B |
|-------|--------|--------|--------------|--------------|
| true  | false  | false  | true         | true         |
| true  | false  | true   | false        | true         |
| false | true   | false  | true         | true         |
| false | true   | true   | false        | false        |

**Explanation:**
Parentheses determine precedence and create different logical meanings.

***

### Question 38
**Is this valid?**

```fga
define can_view: owner or editor and viewer
```

**Answer:**
Yes, it's valid but ambiguous.

**What it likely means:**
```fga
define can_view: owner or (editor and viewer)
```

**Explanation:**
Without parentheses, `and` has higher precedence than `or`, so this evaluates as:
- owner, OR
- (editor AND viewer)

**Best practice:** Use parentheses for clarity:
```fga
define can_view: (owner or editor) and viewer
```

***

### Question 39
**Can you use `not` without `but`?**

**Answer:**
No, `not` must be used as part of `but not`.

**Invalid:**
```fga
define can_access: not banned  # ERROR
```

**Valid:**
```fga
define can_access: member but not banned  # Correct
```

**Explanation:**
OpenFGA doesn't support standalone `not`. You must have a positive condition followed by `but not` for exclusions.

***

### Question 40
**What does this definition mean?**

```fga
define active_editor: editor but not (archived or deleted)
```

**Answer:**
An editor who is NOT archived AND NOT deleted.

**Truth table:**

| editor | archived | deleted | Result |
|--------|----------|---------|--------|
| true   | false    | false   | true   |
| true   | true     | false   | false  |
| true   | false    | true    | false  |
| true   | true     | true    | false  |

**Explanation:**
The `(archived or deleted)` evaluates first, then `but not` excludes those users.

***

### Question 41
**Given:**

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define approved: [user]
    define can_publish: (owner or editor) and approved
```

**Tuples:**
```
(user:alice, owner, document:doc1)
(user:bob, editor, document:doc1)
(user:bob, approved, document:doc1)
```

**Results:**

```
A) Check: user:alice can_publish document:doc1
B) Check: user:bob can_publish document:doc1
```

**Answer:**
- A: `false` (alice is owner but NOT approved)
- B: `true` (bob is editor AND approved)

**Explanation:**
The `and` operator requires both conditions:
1. Must be owner OR editor
2. Must be approved

***

### Question 42
**Can you chain multiple `but not` clauses?**

**Answer:**
Yes, but it's clearer to combine them in one clause.

**Option 1 (chained):**
```fga
define can_access: member but not banned but not suspended
```

**Option 2 (combined - preferred):**
```fga
define can_access: member but not (banned or suspended)
```

**Explanation:**
Both work the same way, but Option 2 is more readable and efficient.

***

### Question 43
**What's the result of this?**

```fga
type document
  relations
    define owner: [user]
    define public: [user:*]
    define can_view: owner or public
```

**Tuple:**
```
(user:*, public, document:terms)
```

**Check:**
```
Check: user:anyone can_view document:terms
```

**Answer:**
`true`

**Explanation:**
The wildcard `user:*` makes the document public. Any user check against `can_view` will match the `public` relation.

***

### Question 44
**Can you use `and` with `from`?**

**Answer:**
Yes, you can combine `and` with `from`.

**Example:**
```fga
type document
  relations
    define organization: [organization]
    define approved: [user]
    define can_publish: approved and admin from organization
```

**Explanation:**
This requires the user to be:
1. Approved for the document, AND
2. An admin of the document's organization

***

### Question 45
**What happens if all conditions in an `or` are false?**

**Answer:**
The result is `false`.

**Example:**
```fga
define can_view: owner or editor or viewer
```

**If user has none of these relations:**
```
Check: user:unknown can_view document:doc1
Result: false
```

**Explanation:**
`or` requires at least one condition to be true. If all are false, the overall result is false.

***

## Hierarchical Relations

### Question 46
**What does the `from` keyword do?**

**Answer:**
The `from` keyword follows a relationship from one object to another and checks a relation there.

**Example:**
```fga
type document
  relations
    define folder: [folder]
    define owner: [user]
    
    define can_view: owner or owner from folder
```

**Explanation:**
1. Find the document's folder
2. Check if the user is owner of that folder
3. If yes, they can view the document

***

### Question 47
**Given this model:**

```fga
type folder
  relations
    define owner: [user]

type document
  relations
    define folder: [folder]
    define can_view: owner from folder
```

**Tuples:**
```
(user:alice, owner, folder:project)
(folder:project, folder, document:spec)
```

**What is the result?**

```
Check: user:alice can_view document:spec
```

**Answer:**
`true`

**Explanation:**
1. spec's folder is project
2. alice is owner of project
3. Therefore alice can view spec

***

### Question 48
**Can you chain multiple `from` clauses?**

**Answer:**
Yes, you can follow multiple relationship levels.

**Example:**
```fga
type organization
  relations
    define member: [user]

type team
  relations
    define organization: [organization]

type project
  relations
    define team: [team]
    define can_access: member from organization from team
```

**Explanation:**
This follows: project → team → organization → check member

**Path:**
```
project:proj1 
  → team:backend 
    → organization:acme 
      → check if user is member of acme
```

***

### Question 49
**What's wrong with this model?**

```fga
type document
  relations
    define folder: [folder]
    define can_view: owner from folder
```

**Answer:**
The `folder` type is not defined, and `owner` relation is not defined on `folder`.

**Fix:**
```fga
type folder
  relations
    define owner: [user]

type document
  relations
    define folder: [folder]
    define can_view: owner from folder
```

**Explanation:**
When using `from`, you must ensure:
1. The referenced type exists
2. The relation you're checking exists on that type

***

### Question 50
**Given this hierarchy:**

```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    
    define can_view: owner or owner from parent
```

**Tuples:**
```
(user:alice, owner, folder:root)
(folder:root, parent, folder:sub)
(folder:sub, parent, folder:leaf)
```

**How many levels deep does permission inheritance go?**

**Answer:**
Only **one level** in this model.

**Explanation:**
The definition `owner from parent` only checks the immediate parent. To check grandparent, you'd need to explicitly define it or use recursion (which OpenFGA doesn't directly support).

**Checks:**
```
user:alice can_view folder:root  → true (direct owner)
user:alice can_view folder:sub   → true (owner from parent)
user:alice can_view folder:leaf  → false (grandparent not checked)
```

***

### Question 51
**How would you model unlimited hierarchy depth?**

**Answer:**
Define the relation to reference itself:

```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    
    # This creates transitive closure
    define can_view: owner or can_view from parent
```

**Explanation:**
By referencing `can_view from parent` instead of `owner from parent`, you create recursion. OpenFGA will traverse up the entire parent chain.

**Now:**
```
user:alice can_view folder:root → true
user:alice can_view folder:sub  → true
user:alice can_view folder:leaf → true (traverses all levels)
```

***

### Question 52
**What does this model do?**

```fga
type organization
  relations
    define parent: [organization]
    define admin: [user]
    
    define can_manage: admin or can_manage from parent
```

**Answer:**
This creates a hierarchical organization structure where admins of parent organizations can manage child organizations.

**Example:**
```
(user:ceo, admin, organization:company)
(organization:company, parent, organization:division)
(organization:division, parent, organization:team)
```

**Result:**
CEO can manage company, division, AND team (unlimited hierarchy).

**Explanation:**
The recursive `can_manage from parent` traverses the entire organizational hierarchy.

***

### Question 53
**Can you use `from` with computed relations?**

**Answer:**
Yes, `from` can reference computed relations.

**Example:**
```fga
type organization
  relations
    define member: [user]
    define admin: [user]
    define can_manage: admin or member

type resource
  relations
    define organization: [organization]
    define can_access: can_manage from organization
```

**Explanation:**
`can_access` checks if the user has the `can_manage` permission (computed) in the resource's organization.

***

### Question 54
**What happens if the `from` reference is null?**

**Answer:**
If the relationship doesn't exist, the check returns `false`.

**Example:**
```fga
type document
  relations
    define folder: [folder]
    define can_view: owner from folder
```

**If no folder tuple exists:**
```
Check: user:alice can_view document:orphan
Result: false (no folder to check)
```

**Explanation:**
OpenFGA requires the relationship path to exist. If any link is missing, the check fails.

---

### Question 55
**Can you combine `from` with operators?**

**Answer:**
Yes, you can combine `from` with `or`, `and`, `but not`.

**Example:**
```fga
type document
  relations
    define folder: [folder]
    define owner: [user]
    define blocked: [user]
    
    define can_view: (owner or owner from folder) but not blocked
```

**Explanation:**
This allows:
- Document owners
- Folder owners
- But excludes blocked users

***

### Question 56
**What does this model enable?**

```fga
type user
  relations
    define manager: [user]

type resource
  relations
    define owner: [user]
    define can_access: owner or manager from owner
```

**Answer:**
This allows a user's manager to access resources owned by that user.

**Example:**
```
(user:bob, manager, user:alice)
(user:alice, owner, resource:doc1)
```

**Check:**
```
user:bob can_access resource:doc1 → true
```

**Explanation:**
Bob is Alice's manager, and Alice owns doc1, so Bob can access doc1.

***

### Question 57
**Can you use `from` multiple times in one definition?**

**Answer:**
Yes, you can reference multiple relationships.

**Example:**
```fga
type document
  relations
    define folder: [folder]
    define workspace: [workspace]
    define owner: [user]
    
    define can_view: owner or owner from folder or owner from workspace
```

**Explanation:**
This checks three different paths for permission:
1. Direct document owner
2. Folder owner
3. Workspace owner

---

### Question 58
**What's the difference between these?**

```fga
# Model A
define can_view: owner from folder

# Model B
define can_view: owner or owner from folder
```

**Answer:**
- **Model A**: ONLY folder owners can view (not document owners)
- **Model B**: BOTH document owners AND folder owners can view

**Example tuples:**
```
(user:alice, owner, document:doc1)
(user:bob, owner, folder:project)
(folder:project, folder, document:doc1)
```

**Model A:**
```
user:alice can_view doc1 → false
user:bob can_view doc1 → true
```

**Model B:**
```
user:alice can_view doc1 → true
user:bob can_view doc1 → true
```

***

### Question 59
**Can you use `and` with multiple `from` clauses?**

**Answer:**
Yes, but it requires the user to have relationships through all paths.

**Example:**
```fga
type document
  relations
    define team: [team]
    define department: [department]
    
    define can_access: member from team and member from department
```

**Explanation:**
The user must be:
- A member of the document's team, AND
- A member of the document's department

This is rare but valid for strict access control.

***

### Question 60
**Given this model:**

```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    define can_view: owner or can_view from parent

type document
  relations
    define folder: [folder]
    define owner: [user]
    define can_view: owner or can_view from folder
```

**Tuples:**
```
(user:alice, owner, folder:root)
(folder:root, parent, folder:sub1)
(folder:sub1, parent, folder:sub2)
(folder:sub2, folder, document:deep)
```

**Result:**

```
Check: user:alice can_view document:deep
```

**Answer:**
`true`

**Explanation:**
Path traversal:
1. document:deep → folder:sub2
2. folder:sub2 → parent:sub1
3. folder:sub1 → parent:root
4. user:alice is owner of root
5. Permission inherited through all levels

This demonstrates multi-level hierarchy traversal.

***

## Advanced Patterns

### Question 61
**What does this model implement?**

```fga
type group
  relations
    define member: [user, group#member]
```

**Answer:**
This implements **nested group membership** (groups within groups).

**Example:**
```
(user:alice, member, group:developers)
(user:bob, member, group:seniors)
(group:seniors#member, member, group:developers)
```

**Result:**
- Alice is directly in developers
- Bob is directly in seniors
- Bob is indirectly in developers (through seniors)

**Explanation:**
The `group#member` syntax allows groups to contain other groups' members, creating group hierarchies.

***

### Question 62
**Given:**

```fga
type group
  relations
    define member: [user, group#member]

type document
  relations
    define viewer_group: [group]
    define can_view: member from viewer_group
```

**Tuples:**
```
(user:alice, member, group:engineers)
(group:engineers#member, member, group:all_staff)
(group:all_staff, viewer_group, document:handbook)
```

**Check:**
```
user:alice can_view document:handbook
```

**Answer:**
`true`

**Explanation:**
1. handbook's viewer_group is all_staff
2. all_staff includes engineers#member
3. engineers includes alice
4. Therefore alice can view handbook

***

### Question 63
**What's the purpose of this pattern?**

```fga
type resource
  relations
    define approved: [user]
    define member: [user]
    define can_access: member and approved
```

**Answer:**
This implements **dual-approval** or **two-factor authorization**.

**Use case:** A user must be both a member AND explicitly approved to access.

**Example:**
- Healthcare: Must be staff member AND have patient consent
- Finance: Must be employee AND have compliance clearance

---

### Question 64
**How would you model "editors can do everything viewers can do"?**

**Answer:**
```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define viewer: [user]
    
    define can_view: viewer or editor or owner
    define can_edit: editor or owner
    define can_delete: owner
```

**Explanation:**
By including `editor` in `can_view`, editors automatically inherit viewing permission. This creates a permission hierarchy:
- Viewer: can_view only
- Editor: can_view + can_edit
- Owner: can_view + can_edit + can_delete

***

### Question 65
**What does this model do?**

```fga
type document
  relations
    define owner: [user]
    define public: [user:*]
    define archived: [user:*]
    
    define can_view: (owner or public) but not archived
```

**Answer:**
This implements **conditional public access with archival**.

**Behavior:**
- Owners can always view (even archived)... wait, no!
- Public documents can be viewed by anyone
- But archived documents can't be viewed by ANYONE (including owners)

**Correction:** If you want owners to view archived documents:
```fga
define can_view: owner or (public but not archived)
```

***

### Question 66
**How would you model time-based access?**

**Answer:**
OpenFGA doesn't support time-based conditions directly in the DSL. You must handle this at the application level.

**Workaround:**
```fga
type document
  relations
    define active_viewer: [user]
    define can_view: active_viewer
```

**Application logic:**
```
if (currentTime > expirationTime) {
  fga.tuple.delete(user, active_viewer, document)
}
```

**Explanation:**
OpenFGA models relationships, not temporal conditions. Time-based logic must be implemented by adding/removing tuples based on time.

***

### Question 67
**What's the difference between these two models?**

**Model A:**
```fga
type document
  relations
    define group: [group]
    define can_view: member from group
```

**Model B:**
```fga
type document
  relations
    define viewer: [user, group#member]
    define can_view: viewer
```

**Answer:**
- **Model A**: Document has ONE group; all members can view
- **Model B**: Document can have MULTIPLE viewers (individual users or group members)

**Model A tuples:**
```
(group:engineers, group, document:spec)
```

**Model B tuples:**
```
(user:alice, viewer, document:spec)
(group:engineers#member, viewer, document:spec)
```

**Explanation:**
Model A ties the document to a single group. Model B allows flexible viewer assignment.

***

### Question 68
**How would you model "organization admins can access everything in the organization"?**

**Answer:**
```fga
type organization
  relations
    define admin: [user]
    define member: [user]

type document
  relations
    define organization: [organization]
    define owner: [user]
    
    define can_view: owner or member from organization
    define can_edit: owner or admin from organization
    define can_delete: admin from organization
```

**Explanation:**
- Members can view all org documents
- Admins can edit all org documents
- Admins can delete all org documents
- Individual owners maintain their specific permissions

***

### Question 69
**What does this enable?**

```fga
type user
  relations
    define blocked_user: [user]

type document
  relations
    define owner: [user]
    define viewer: [user]
    
    define can_view: (owner or viewer) but not blocked_user from owner
```

**Answer:**
This allows document owners to block specific users.

**Example:**
```
(user:alice, owner, document:doc1)
(user:bob, viewer, document:doc1)
(user:bob, blocked_user, user:alice)
```

**Check:**
```
user:bob can_view document:doc1 → false
```

**Explanation:**
Even though Bob is a viewer, Alice (the owner) has blocked him, so he can't view.

***

### Question 70
**Can you model "at least 2 approvers"?**

**Answer:**
No, OpenFGA doesn't support counting or cardinality constraints in the DSL.

**Workaround:**
```fga
type document
  relations
    define approver_1: [user]
    define approver_2: [user]
    define approver_3: [user]
    
    define fully_approved: approver_1 and approver_2
    define triple_approved: approver_1 and approver_2 and approver_3
```

**Explanation:**
You must model specific approval slots. There's no way to express "count >= 2" in the DSL.

***

### Question 71
**What does this model implement?**

```fga
type document
  relations
    define owner: [user]
    define editor: [user]
    define pending_editor: [user]
    define invite_accepted: [user]
    
    define active_editor: pending_editor and invite_accepted
    define can_edit: owner or editor or active_editor
```

**Answer:**
This implements **invitation-based editing** with acceptance requirement.

**Workflow:**
1. Owner adds user as pending_editor
2. User accepts invite (invite_accepted tuple created)
3. User becomes active_editor and can edit

**Explanation:**
Requires two-step authorization: invitation + acceptance.

***

### Question 72
**How would you model "recently active members"?**

**Answer:**
OpenFGA doesn't support time-based queries directly.

**Workaround with application logic:**
```fga
type organization
  relations
    define active_member: [user]
    define inactive_member: [user]
    
    define can_access: active_member but not inactive_member
```

**Application:**
```
// Periodically run:
if (lastActivity > 30 days ago) {
  fga.tuple.delete(user, active_member, org)
  fga.tuple.write(user, inactive_member, org)
}
```

***

### Question 73
**What's wrong with this model?**

```fga
type document
  relations
    define owner: [user]
    define can_view: can_edit
    define can_edit: owner
```

**Answer:**
Nothing is wrong - this is valid! Computed relations can reference each other as long as there's no circular dependency.

**Explanation:**
`can_view` is defined before `can_edit` in the file, but OpenFGA resolves this correctly because it parses the entire model first.

***

### Question 74
**Can you have circular dependencies?**

**Answer:**
Yes, but only in specific patterns that OpenFGA can resolve.

**Valid (recursive traversal):**
```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    define can_view: owner or can_view from parent
```

**Invalid (direct circular):**
```fga
type document
  relations
    define can_view: can_edit
    define can_edit: can_view  # ERROR: circular
```

**Explanation:**
OpenFGA supports recursion through relationships (`from parent`) but not direct circular references between permissions.

***

### Question 75
**What does this pattern accomplish?**

```fga
type tenant
  relations
    define admin: [user]

type workspace
  relations
    define tenant: [tenant]
    define member: [user]

type project
  relations
    define workspace: [workspace]
    define owner: [user]
    
    define can_access: (
      owner or 
      member from workspace or 
      admin from tenant from workspace
    )
```

**Answer:**
This implements **multi-level tenant hierarchy** with cascading permissions:
- Project owners can access
- Workspace members can access
- Tenant admins can access everything

**Explanation:**
Three levels of access control:
1. Project level (owner)
2. Workspace level (member)
3. Tenant level (admin)

***

## Real-World Scenarios

### Question 76
**Model this requirement: "Users can view documents they created OR documents in folders they can view."**

**Answer:**
```fga
type user

type folder
  relations
    define member: [user]

type document
  relations
    define creator: [user]
    define folder: [folder]
    
    define can_view: creator or member from folder
```

**Explanation:**
Combines direct ownership (creator) with inherited access (folder membership).

---

### Question 77
**Model GitHub repository permissions:**
- Owners can admin, write, read
- Writers can write, read
- Readers can read

**Answer:**
```fga
type repository
  relations
    define owner: [user]
    define writer: [user]
    define reader: [user]
    
    define can_read: reader or writer or owner
    define can_write: writer or owner
    define can_admin: owner
```

**Explanation:**
Permission hierarchy where higher roles inherit lower permissions.

***

### Question 78
**Model Google Drive sharing:**
- Files inherit permissions from folders
- Can share directly with users or groups
- Can make public

**Answer:**
```fga
type user

type group
  relations
    define member: [user, group#member]

type folder
  relations
    define parent: [folder]
    define owner: [user]
    define editor: [user, group#member]
    define viewer: [user, group#member]
    define public: [user:*]
    
    define can_view: viewer or editor or owner or public or can_view from parent
    define can_edit: editor or owner or can_edit from parent

type file
  relations
    define parent: [folder]
    define owner: [user]
    define editor: [user, group#member]
    define viewer: [user, group#member]
    define public: [user:*]
    
    define can_view: viewer or editor or owner or public or can_view from parent
    define can_edit: editor or owner or can_edit from parent
```

***

### Question 79
**Model Slack workspace permissions:**
- Workspace owners can do everything
- Channel creators can manage their channels
- Members can post in public channels
- Only invited users can view private channels

**Answer:**
```fga
type user

type workspace
  relations
    define owner: [user]
    define member: [user]

type channel
  relations
    define workspace: [workspace]
    define creator: [user]
    define member: [user]
    define public: [user:*]
    
    define can_view: (
      member or 
      (public and member from workspace) or
      owner from workspace
    )
    
    define can_post: can_view
    
    define can_manage: creator or owner from workspace
```

***

### Question 80
**Model HIPAA-compliant medical records:**
- Primary physician always has access
- Care team members have access
- Patient can grant access to specific doctors
- Compliance officer can audit all records

**Answer:**
```fga
type user

type organization
  relations
    define compliance_officer: [user]

type patient
  relations
    define organization: [organization]
    define primary_physician: [user]
    define care_team: [user]
    define authorized_viewer: [user]
    
    define can_view_records: (
      primary_physician or 
      care_team or 
      authorized_viewer or
      compliance_officer from organization
    )

type medical_record
  relations
    define patient: [patient]
    define created_by: [user]
    
    define can_view: (
      created_by or 
      can_view_records from patient
    )
```

***

### Question 81
**Model a multi-tenant SaaS with workspaces:**
- Tenant admins can access all workspaces
- Workspace owners can manage their workspace
- Project members can only access their projects

**Answer:**
```fga
type user

type tenant
  relations
    define admin: [user]
    define member: [user]

type workspace
  relations
    define tenant: [tenant]
    define owner: [user]
    define member: [user]
    
    define can_access: member or owner or admin from tenant
    define can_manage: owner or admin from tenant

type project
  relations
    define workspace: [workspace]
    define owner: [user]
    define member: [user]
    
    define can_view: member or owner or can_manage from workspace
    define can_edit: owner or can_manage from workspace
```

***

### Question 82
**Model an approval workflow:**
- Submitter can view their own submission
- Reviewers can view and comment
- Approvers must review AND approve to publish

**Answer:**
```fga
type submission
  relations
    define submitter: [user]
    define reviewer: [user]
    define approver: [user]
    define has_reviewed: [user]
    define has_approved: [user]
    
    define can_view: submitter or reviewer or approver
    define can_comment: reviewer or approver
    define can_approve: approver and has_reviewed
    define can_publish: has_approved
```

***

### Question 83
**Model emergency access (break-glass):**
- Normal access through roles
- Admins can grant temporary emergency access
- Emergency access logs required

**Answer:**
```fga
type resource
  relations
    define owner: [user]
    define viewer: [user]
    define emergency_access: [user]
    
    define can_view: viewer or owner or emergency_access
```

**Application handles:**
- Logging when emergency_access tuple created
- TTL on emergency_access tuples
- Alerting when emergency access used

***

### Question 84
**Model document lifecycle:**
- Draft: only author can edit
- Review: reviewers can view, author can edit
- Published: everyone can view, no one can edit
- Archived: no one can edit or view except archivists

**Answer:**
```fga
type document
  relations
    define author: [user]
    define reviewer: [user]
    define archivist: [user]
    define status_draft: [user:*]
    define status_review: [user:*]
    define status_published: [user:*]
    define status_archived: [user:*]
    
    define can_view: (
      (author and (status_draft or status_review)) or
      (reviewer and status_review) or
      (user:* and status_published) or
      (archivist and status_archived)
    )
    
    define can_edit: (
      author and (status_draft or status_review)
    )
```

***

### Question 85
**Model team-based project access with external contractors:**
- Team members have full access
- Contractors need explicit project assignment
- Managers can see all projects

**Answer:**
```fga
type user

type team
  relations
    define member: [user]
    define manager: [user]

type project
  relations
    define team: [team]
    define contractor: [user]
    
    define can_access: (
      member from team or 
      contractor or
      manager from team
    )
    
    define can_manage: manager from team
```

***

### Question 86
**Model organization with departments and sub-departments:**

**Answer:**
```fga
type user

type organization
  relations
    define admin: [user]

type department
  relations
    define organization: [organization]
    define parent_department: [department]
    define manager: [user]
    define member: [user]
    
    define can_view: (
      member or 
      manager or
      can_view from parent_department or
      admin from organization
    )
```

***

### Question 87
**Model feature flags with user targeting:**

**Answer:**
```fga
type user

type feature_flag
  relations
    define enabled_for_user: [user]
    define enabled_for_organization: [organization]
    define globally_enabled: [user:*]
    
    define can_access: (
      enabled_for_user or
      member from enabled_for_organization or
      globally_enabled
    )
```

***

### Question 88
**Model a learning management system:**
- Teachers create and grade assignments
- Students can view assignments and submit
- TAs can grade but not modify

**Answer:**
```fga
type user

type course
  relations
    define teacher: [user]
    define ta: [user]
    define student: [user]

type assignment
  relations
    define course: [course]
    define creator: [user]
    
    define can_view: (
      creator or
      teacher from course or
      ta from course or
      student from course
    )
    
    define can_edit: creator or teacher from course
    
    define can_grade: (
      teacher from course or 
      ta from course
    )
    
    define can_submit: student from course
```

***

### Question 89
**Model content moderation:**
- Authors can edit their content
- Moderators can hide/delete any content
- Users can report content
- Admins can ban users

**Answer:**
```fga
type user
  relations
    define banned: [user:*]

type content
  relations
    define author: [user]
    define moderator: [user]
    define admin: [user]
    define hidden: [user:*]
    
    define can_view: (author or moderator or admin) and not hidden
    define can_edit: author and not banned from author
    define can_moderate: moderator or admin
    define can_delete: can_moderate
```

***

### Question 90
**Model API key permissions:**
- API keys belong to users
- API keys have scoped permissions
- API keys can be revoked

**Answer:**
```fga
type user

type api_key
  relations
    define owner: [user]
    define revoked: [user:*]
    define read_scope: [user:*]
    define write_scope: [user:*]
    
    define is_active: owner and not revoked
    define can_read: is_active and read_scope
    define can_write: is_active and write_scope

type resource
  relations
    define api_key: [api_key]
    
    define can_read: can_read from api_key
    define can_write: can_write from api_key
```

***

## Debugging and Troubleshooting

### Question 91
**You created this model but checks always return false. Why?**

```fga
type document
  relations
    define owner: [user]
    define can_view: viewer
```

**Answer:**
The relation `viewer` is not defined. It's referenced in `can_view` but doesn't exist.

**Fix:**
```fga
type document
  relations
    define owner: [user]
    define viewer: [user]
    define can_view: viewer or owner
```

***

### Question 92
**This check returns false but you expect true. Why?**

**Model:**
```fga
type document
  relations
    define editor: [user]
    define can_edit: owner
```

**Tuple:**
```
(user:alice, editor, document:doc1)
```

**Check:**
```
Check: user:alice can_edit document:doc1
Result: false
```

**Answer:**
`can_edit` is defined as `owner`, not `editor`. The tuple is for `editor` but the check requires `owner`.

**Fix:**
```fga
define can_edit: owner or editor
```

***

### Question 93
**Your model passes validation but checks are slow. What could be wrong?**

**Model:**
```fga
type folder
  relations
    define parent: [folder]
    define owner: [user]
    define can_view: owner or can_view from parent
```

**Answer:**
If you have very deep folder hierarchies (100+ levels), the recursive `can_view from parent` can cause performance issues.

**Solutions:**
1. Limit hierarchy depth
2. Denormalize permissions at certain levels
3. Cache intermediate results at application level

***

### Question 94
**This check returns true but you expect false. Diagnose:**

**Model:**
```fga
type document
  relations
    define member: [user]
    define blocked: [user]
    define can_access: member or blocked
```

**Tuples:**
```
(user:bob, member, document:doc1)
(user:bob, blocked, document:doc1)
```

**Check:**
```
Check: user:bob can_access document:doc1
Result: true (expected false)
```

**Answer:**
The definition uses `or` instead of `but not`. Bob is blocked but still has access.

**Fix:**
```fga
define can_access: member but not blocked
```

***

### Question 95
**How do you test if a tuple exists?**

**Answer:**
Query the direct relation, not a computed relation.

**Check tuple existence:**
```
Check: user:alice owner document:doc1
```

**This checks:**
- If `owner` is a direct relation: checks tuple existence
- If `owner` is computed: evaluates the logic

**Best practice:** Use direct relations for existence checks.

---

### Question 96
**Your model works in testing but fails in production. Why?**

**Answer:**
Common causes:
1. **Different schema version**: Dev uses `schema 1.1`, prod uses `1.0`
2. **Missing tuples**: Tuples not migrated to production
3. **Wrong store**: Querying dev store in prod environment
4. **Model not deployed**: Old model still active in production

**Debug:**
```bash
# Check active model
fga model get --store-id <prod-store-id>

# List tuples
fga tuple read --store-id <prod-store-id>

# Verify schema version in model file
```

***

### Question 97
**How do you debug which part of a complex definition is failing?**

**Model:**
```fga
define can_access: (member or owner) and approved and not banned
```

**Answer:**
Break down the check:
```
1. Check: user:bob member document:doc1
2. Check: user:bob owner document:doc1  
3. Check: user:bob approved document:doc1
4. Check: user:bob banned document:doc1
```

Identify which condition is false.

**Alternative:** Simplify the model temporarily:
```fga
define can_access: member or owner
```

Then add conditions back one by one.

***

### Question 98
**You get "cycle detected" error. What does this mean?**

**Answer:**
You have a circular dependency in your computed relations.

**Example:**
```fga
type document
  relations
    define can_view: can_edit
    define can_edit: can_view  # Circular!
```

**Fix:**
Define relations in terms of direct relations, not each other:
```fga
define can_view: viewer or editor or owner
define can_edit: editor or owner
```

***

### Question 99
**Your group membership isn't working. Diagnose:**

**Model:**
```fga
type group
  relations
    define member: [user, group#member]

type document
  relations
    define viewer_group: [group]
    define can_view: member from viewer_group
```

**Tuples:**
```
(user:alice, member, group:engineers)
(group:engineers, viewer_group, document:doc1)
```

**Check:**
```
Check: user:alice can_view document:doc1
Result: false (expected true)
```

**Answer:**
The tuple should be `(group:engineers#member, ...)` not just `(group:engineers, ...)`.

**Fix tuple:**
```
Actually, the tuple is correct. Let me re-examine...
```

Wait - the issue is that we're using `member from viewer_group` which looks for members OF the group itself.

**Correct model:**
```fga
define can_view: viewer_group and member from viewer_group
```

Actually, this doesn't work either. The correct approach:

**Fix:**
```fga
type document
  relations
    define viewer: [user, group#member]
    define can_view: viewer
```

**Tuple:**
```
(group:engineers#member, viewer, document:doc1)
```

This allows all members of engineers group to view.

***

### Question 100
**How do you validate your model before deploying?**

**Answer:**
Use the OpenFGA CLI:

```bash
# Validate syntax
fga model validate --file model.fga

# Run tests
fga model test --file model.fga --tests tests.yaml

# Test in staging
fga model write --store-id <staging-store> --file model.fga

# Run test queries
fga query check --store-id <staging-store> \
  --user user:alice \
  --relation can_view \
  --object document:doc1
```

**Best practices:**
1. Write tests for all critical paths
2. Test both positive and negative cases
3. Validate in staging before production
4. Use version control for models
5. Document expected behavior

***

## Summary

This question bank covers:
- **100 questions** spanning basic to advanced OpenFGA concepts
- **Detailed answers** with explanations
- **Code examples** for each scenario
- **Real-world patterns** for common use cases
- **Debugging techniques** for troubleshooting

**Study approach:**
1. Start with Basic Concepts (Q1-10)
2. Master Type Definitions (Q11-20)
3. Understand Relations (Q21-30)
4. Learn Operators (Q31-45)
5. Practice Hierarchies (Q46-60)
6. Explore Advanced Patterns (Q61-75)
7. Apply to Real Scenarios (Q76-90)
8. Debug Effectively (Q91-100)

**Next steps:**
- Practice writing your own models
- Test with OpenFGA CLI
- Implement real authorization scenarios
- Review and refine iteratively
- Join OpenFGA community for support
- Study open-source examples on GitHub
- Build a sample application
- Document your authorization model thoroughly

***

## Quick Reference Card

### Essential Syntax

```fga
model
  schema 1.1

type type_name
  relations
    define relation: [type]                # Direct relation
    define permission: relation            # Computed from relation
    define complex: rel1 or rel2          # OR operator
    define strict: rel1 and rel2          # AND operator
    define exclude: rel1 but not rel2     # Exclusion
    define inherit: relation from parent  # Hierarchical
```

### Common Patterns

**Basic Permission:**
```fga
define can_action: role1 or role2 or role3
```

**Hierarchical:**
```fga
define can_access: direct_access or can_access from parent
```

**Group-Based:**
```fga
define member: [user, group#member]
define can_view: member from viewer_group
```

**Exclusion:**
```fga
define can_access: member but not banned
```

**Conditional:**
```fga
define can_publish: (owner or editor) and approved
```

***

## Practice Exercises

### Exercise 1: Blog Platform
Model a blog platform where:
- Authors can create, edit, and delete their posts
- Editors can edit any post
- Readers can view published posts
- Admins can do everything

### Exercise 2: Project Management
Model a project management tool where:
- Project owners can manage projects
- Team members can view and comment
- Guests can only view
- Organization admins can see all projects

### Exercise 3: File Storage
Model a file storage system where:
- Files belong to folders
- Folders can have parent folders
- Permissions inherit from parent folders
- Users and groups can be granted access

### Exercise 4: E-commerce
Model an e-commerce system where:
- Customers can view products
- Customers can manage their own orders
- Store managers can manage all orders
- Product managers can edit product catalog

### Exercise 5: Healthcare
Model a healthcare system where:
- Doctors can view their patients' records
- Nurses can view but not edit
- Patients can grant access to specialists
- Admin can audit all access

---

## Answer Key for Exercises

### Exercise 1 Solution: Blog Platform

```fga
model
  schema 1.1

type user

type blog
  relations
    define admin: [user]
    define editor: [user]

type post
  relations
    define blog: [blog]
    define author: [user]
    define published: [user:*]
    
    define can_view: (user:* and published) or author or editor from blog or admin from blog
    define can_edit: author or editor from blog or admin from blog
    define can_delete: author or admin from blog
    define can_publish: author or editor from blog or admin from blog
```

### Exercise 2 Solution: Project Management

```fga
model
  schema 1.1

type user

type organization
  relations
    define admin: [user]
    define member: [user]

type project
  relations
    define organization: [organization]
    define owner: [user]
    define team_member: [user]
    define guest: [user]
    
    define can_view: guest or team_member or owner or admin from organization
    define can_comment: team_member or owner or admin from organization
    define can_manage: owner or admin from organization
```

### Exercise 3 Solution: File Storage

```fga
model
  schema 1.1

type user

type group
  relations
    define member: [user, group#member]

type folder
  relations
    define parent: [folder]
    define owner: [user]
    define editor: [user, group#member]
    define viewer: [user, group#member]
    
    define can_view: viewer or editor or owner or can_view from parent
    define can_edit: editor or owner or can_edit from parent
    define can_delete: owner

type file
  relations
    define folder: [folder]
    define owner: [user]
    define editor: [user, group#member]
    define viewer: [user, group#member]
    
    define can_view: viewer or editor or owner or can_view from folder
    define can_edit: editor or owner or can_edit from folder
    define can_delete: owner
```

### Exercise 4 Solution: E-commerce

```fga
model
  schema 1.1

type user

type store
  relations
    define owner: [user]
    define manager: [user]
    define product_manager: [user]

type product
  relations
    define store: [store]
    
    define can_view: user:*
    define can_edit: product_manager from store or manager from store or owner from store

type order
  relations
    define store: [store]
    define customer: [user]
    
    define can_view: customer or manager from store or owner from store
    define can_manage: manager from store or owner from store
```

### Exercise 5 Solution: Healthcare

```fga
model
  schema 1.1

type user

type hospital
  relations
    define admin: [user]

type patient
  relations
    define hospital: [hospital]
    define primary_doctor: [user]
    define nurse: [user]
    define authorized_specialist: [user]
    
    define can_view_records: (
      primary_doctor or 
      nurse or 
      authorized_specialist or
      admin from hospital
    )

type medical_record
  relations
    define patient: [patient]
    
    define can_view: can_view_records from patient or admin from hospital from patient
    define can_edit: primary_doctor from patient
    define can_audit: admin from hospital from patient
```

***

## Advanced Challenge Problems

### Challenge 1: Multi-Tenant SaaS
Design a complete authorization model for a multi-tenant SaaS application with:
- Multiple tenants with isolation
- Workspaces within tenants
- Projects within workspaces
- Resources within projects
- User roles at each level
- Cascading permissions

### Challenge 2: Social Media Platform
Model a social media platform with:
- Users can follow other users
- Posts can be public, friends-only, or private
- Groups with public and private posts
- Moderation system
- Blocking and muting

### Challenge 3: Enterprise Resource Planning
Model an ERP system with:
- Multiple organizations in a hierarchy
- Departments and sub-departments
- Resources with different sensitivity levels
- Approval workflows
- Delegation of authority

### Challenge 4: Content Management System
Model a CMS with:
- Multiple sites under one account
- Content types (pages, posts, media)
- Workflow states (draft, review, published, archived)
- Role-based access at site and content level
- Scheduled publishing

### Challenge 5: Banking System
Model a banking system with:
- Account ownership and co-ownership
- Transaction approval limits
- Temporary access delegation
- Compliance officer oversight
- Emergency access procedures

***

## Self-Assessment Checklist

After completing this question bank, you should be able to:

**Basic Understanding:**
- [ ] Explain the difference between types, relations, and tuples
- [ ] Write basic type definitions with direct relations
- [ ] Create simple computed relations using `or`
- [ ] Understand how tuple storage works

**Intermediate Skills:**
- [ ] Use all three operators (`or`, `and`, `but not`)
- [ ] Implement hierarchical permissions with `from`
- [ ] Model group-based access control
- [ ] Design permission inheritance patterns

**Advanced Capabilities:**
- [ ] Create complex multi-level hierarchies
- [ ] Model real-world authorization scenarios
- [ ] Debug authorization model issues
- [ ] Optimize models for performance
- [ ] Design for future scalability

**Professional Proficiency:**
- [ ] Architect authorization for enterprise systems
- [ ] Handle edge cases and security concerns
- [ ] Document authorization models effectively
- [ ] Collaborate with security teams
- [ ] Implement compliance requirements

***

## Additional Resources

### Official Documentation
- OpenFGA Docs: https://openfga.dev/docs
- DSL Reference: https://openfga.dev/docs/modeling
- API Reference: https://openfga.dev/api

### Community
- GitHub Discussions: https://github.com/openfga/openfga/discussions
- Discord Server: https://discord.gg/8naAwJfWN6
- Community Examples: https://github.com/openfga/sample-stores

### Tools
- OpenFGA Playground: https://play.fga.dev
- VS Code Extension: Search "OpenFGA" in VS Code marketplace
- CLI Documentation: https://github.com/openfga/cli

### Learning Path
1. Complete this question bank (100 questions)
2. Work through the 5 practice exercises
3. Attempt the 5 advanced challenges
4. Build a real project with OpenFGA
5. Contribute to open-source examples

***

## Feedback and Next Steps

**Mastered the content?**
- Build a complete authorization model for your project
- Share your model with the OpenFGA community
- Contribute examples or documentation
- Help others learn OpenFGA

**Need more practice?**
- Revisit questions you struggled with
- Re-read the tutorial sections
- Try the exercises with variations
- Join community discussions for help

**Ready for production?**
- Review security best practices
- Plan your tuple migration strategy
- Set up monitoring and alerting
- Document your authorization model
- Train your team on the model

***

## Final Notes

Authorization modeling is both an art and a science. The patterns in this question bank represent common scenarios, but every application has unique requirements. 

**Key principles to remember:**
1. Start simple and iterate
2. Test thoroughly
3. Document your decisions
4. Review with security team
5. Monitor and audit access
6. Plan for changes

**Success indicators:**
- Authorization logic is clear and understandable
- Model handles all business requirements
- Performance is acceptable
- Team can maintain and extend the model
- Security requirements are met

**When in doubt:**
- Consult OpenFGA documentation
- Ask in community forums
- Review open-source examples
- Test in a staging environment
- Get peer review before production

---

## Congratulations!

You've completed the OpenFGA DSL Question Bank. You now have:
- Deep understanding of OpenFGA concepts
- Practical experience with 100 scenarios
- Real-world modeling patterns
- Debugging and troubleshooting skills
- Foundation for enterprise authorization

**Keep learning, keep building, and welcome to the OpenFGA community!**

***

**Question Bank Version:** 1.0  
**Last Updated:** November 2025  
**OpenFGA Schema Version:** 1.1  
**Questions:** 100  
**Exercises:** 5  
**Challenges:** 5

**Companion Resources:**
- OpenFGA DSL Tutorial (prerequisite)
- OpenFGA CLI Guide
- OpenFGA API Reference
- Community Examples Repository

**For updates and corrections:**
Visit the OpenFGA documentation site or GitHub discussions.

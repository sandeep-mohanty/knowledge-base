# 📘 Essential YAML Data Structures

YAML (YAML Ain’t Markup Language) is a human-friendly data serialization format commonly used in configuration files and infrastructure-as-code tools like Kubernetes, Ansible, Docker Compose, and GitHub Actions.

This guide covers the **essential YAML data structures** every DevOps engineer should know, with examples and best practices.

---

## 📦 1. Scalars (Basic Values)

Scalars are single values: strings, integers, booleans, floats.

```yaml
name: John Doe
age: 30
is_admin: true
pi: 3.14
```

### 📝 Notes:
- Strings can be unquoted, single-quoted (`'`) or double-quoted (`"`)
- Use quotes if the value contains special characters or starts with `@`, `&`, `*`, etc.

```yaml
password: 'P@ssw0rd!'
```

---

## 📋 2. Sequences (Lists)

A sequence is a list of items, denoted with dashes (`-`).

```yaml
servers:
  - web01
  - web02
  - db01
```

### Inline List

You can also write sequences inline:

```yaml
zones: [us-east-1, us-west-1, eu-central-1]
```

---

## 🗂️ 3. Mappings (Dictionaries / Key-Value Pairs)

Mappings associate keys with values.

```yaml
database:
  host: localhost
  port: 5432
  user: admin
  password: secret
```

This is equivalent to a JSON object:

```json
{
  "database": {
    "host": "localhost",
    "port": 5432,
    "user": "admin",
    "password": "secret"
  }
}
```

---

## 🔁 4. Nested Structures

You can nest mappings and sequences to represent complex configurations.

```yaml
environments:
  dev:
    url: dev.example.com
    replicas: 2
  prod:
    url: prod.example.com
    replicas: 5
```

```yaml
users:
  - name: Alice
    role: admin
  - name: Bob
    role: user
```

---

## 💡 5. Multi-line Strings

Use `|` to preserve line breaks, and `>` to fold into a single line.

### Literal Block (`|`)

```yaml
message: |
  Hello,
  This is a multi-line message.
  Goodbye!
```

### Folded Block (`>`)

```yaml
message: >
  This will be
  folded into a single
  line with spaces.
```

---

## 🧬 6. Anchors and Aliases

YAML lets you reuse blocks using anchors (`&`) and aliases (`*`).

```yaml
defaults: &default_settings
  retries: 3
  timeout: 30

service_a:
  <<: *default_settings
  endpoint: /api/v1

service_b:
  <<: *default_settings
  endpoint: /api/v2
```

### 💡 Use-case:
Avoid duplication and ensure consistent configuration across services.

---

## 💬 7. Comments

Use `#` to leave comments in YAML files.

```yaml
# This is a comment
port: 8080  # Inline comment
```

---

## 🛑 8. Common Mistakes to Avoid

- **Tabs are not allowed**! Always use spaces (usually 2 or 4).
- **Inconsistent indentation** will break your file.
- **Avoid trailing spaces** — some parsers are very strict.
- **Duplicate keys** in the same mapping are not allowed.

---

## 🔧 YAML Use Cases in DevOps

| Tool             | Use Case                         |
|------------------|----------------------------------|
| Kubernetes       | Resource manifests (`Deployment`, `Service`) |
| Ansible          | Playbooks and roles              |
| Docker Compose   | Multi-container Docker apps      |
| GitHub Actions   | CI/CD workflows (`.github/workflows/*.yml`) |
| CircleCI, Travis | CI pipelines                     |

---

## ✅ Best Practices

- Use consistent indentation (2 spaces is common)
- Quote strings that contain special characters
- Use anchors and aliases to reduce duplication
- Validate your YAML using tools like:
  - [YAML Lint](https://www.yamllint.com/)
  - `yamllint` CLI
  - `pyyaml` or other parsers

---

## 📚 Bonus: YAML vs JSON

| Feature            | YAML                        | JSON                      |
|--------------------|-----------------------------|---------------------------|
| Readability        | More human-friendly         | More machine-friendly     |
| Syntax             | Indentation-based           | Curly/bracket-based       |
| Comments           | ✅ Yes (`#`)                | ❌ No                      |
| File extension     | `.yaml` or `.yml`           | `.json`                   |

---

## 📌 Summary

- YAML is widely used for configuration in DevOps.
- Mastering key structures like **scalars**, **lists**, **mappings**, and **anchors** is essential.
- Always follow best practices to ensure your YAML files are clean, reusable, and error-free.

---

## ✅ Recommended Tools

- [YAML Lint](https://www.yamllint.com/)
- VS Code + YAML extension
- `yamllint` (CLI): `sudo apt install yamllint`
- `python -c 'import yaml'` (to validate with PyYAML)

---

## 🙌 Final Thoughts

YAML is simple but powerful. Whether you're writing Kubernetes manifests, CI/CD workflows, or Ansible playbooks, understanding YAML structures will help you write better, cleaner, and more maintainable configs.

Happy YAMLing! 🚀
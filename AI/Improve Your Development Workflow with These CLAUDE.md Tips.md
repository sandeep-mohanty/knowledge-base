# Improve Your Development Workflow with These CLAUDE.md Tips
**We will gain an in-depth understanding of how to use CLAUDE.md for Software Development and improve productivity.**

If you have used Claude Code for a while, you already know the basics. You open a terminal, type a prompt, and Claude writes code for you.

Claude starts a fresh session every time. **It knows nothing about our project.** It does not remember the patterns you prefer, the tools we use, or the mistakes it made last week.

### What is CLAUDE.md?
**CLAUDE.md** is a plain Markdown file you place at the root of our project. Claude Code reads it automatically at the start of every session.

This file can be in a few places:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NyFoORKPK-HiA0SA4--W9g@2x.png)


We will discuss the project-level Claude.md file.

---

### Tip 1: Use the WHY–WHAT–HOW…
Before writing a single line, think about what Claude actually needs to know. A good **CLAUDE.md** should define your project’s **WHY, WHAT, and HOW**.

Here is what that looks like in practice:

```markdown
# Project: PayTrack API

## Why
A REST API that handles salary disbursement and payroll tracking
for mid-sized companies. Built for correctness and auditability.

## What
- Java 21, Spring Boot 3.x
- PostgreSQL for persistence
- Maven for builds
- Flyway for DB migrations

## How
- Run tests: `mvn test`
- Run the app: `mvn spring-boot:run`
- Lint: `mvn checkstyle:check`
- Never use field injection. Always use constructor injection.
- All DB changes go through Flyway migration files.
```

This structure tells Claude what it is working on, what tools to use, and how to behave on the project.

---

### Tip 2: Document What Claude Gets Wrong, Not What You Hope It Does Right
This is the most underrated tip.

Boris Cherny, the creator of Claude Code, follows a golden rule: **Anytime we see Claude do something incorrectly, we add it to CLAUDE.md so it doesn’t repeat next time.** The file is checked into git and updated multiple times a week. It is a living document, not a one-time setup.

Every time Claude makes a mistake in your project, ask: *Can I add a rule to prevent this?*

For example, if Claude keeps adding unnecessary `System.out.println` statements in production code:

```markdown
## Rules
- Never use System.out.println. Use SLF4J logger instead.
- Always add @Slf4j annotation when adding log statements.
```

If it keeps writing `catch (Exception e) { e.printStackTrace(); }`:

```markdown
- Never swallow exceptions silently.
- Always log exceptions with logger.error() and rethrow or handle explicitly.
```

This turns the CLAUDE.md into a project memory that gets smarter over time.

---

### Tip 3: Use Progressive Disclosure
We do not have to write every detail into CLAUDE.md directly.

We can use progressive disclosure, i.e., do not tell Claude all the information you could want him to know. Instead, tell it how to locate important information so it can retrieve and use it only when needed. This avoids bloating your context window.

Here is an example:

```markdown
## Architecture
See `docs/architecture.md` for service boundaries and data flow.

## API Contracts
OpenAPI spec lives at `src/main/resources/openapi.yaml`.
Always check this before adding or modifying endpoints.

## Database Schema
The current schema is located at `src/main/resources/db/migration/`.
Run `mvn flyway:info` to check migration status.
```

Now Claude knows where to look for context when needed. We do not have to paste entire files into the prompt every session.

---

### Tip 4: Context-Specific Rules
Not every rule applies to every file. Claude Code supports lazy-loaded rule files. These only activate when Claude touches matching files.

Rules in `.claude/rules/*.md` with YAML frontmatter are lazy-loaded only when Claude touches matching files. Without frontmatter, they load into every session, just like CLAUDE.md.

Suppose we want strict rules to apply only when Claude edits your service layer. Create `.claude/rules/service-layer.md`:

```markdown
---
paths:
  - src/main/java/com/yourapp/service/**
---

## Service Layer Rules
- All service classes must be annotated with @Service and @Transactional.
- No direct repository calls from Controllers. Always go through Services.
- Throw domain-specific exceptions, not generic RuntimeException.
```

This keeps the main CLAUDE.md clean while still enforcing rules where they matter.

---

### Tip 5: Add a Verification Section
Claude should not mark a task done without proving it works.

```markdown
## Definition of Done
Before marking any task complete:
1. Run `mvn test` — all tests must pass.
2. Run `mvn checkstyle:check` — no style violations.
3. Check that no debug logs or TODOs are left in changed files.
4. Confirm that no new dependencies were added without updating pom.xml comments.
```

---

### Tip 6: Put Principles at the Bottom
At the bottom of a well-crafted CLAUDE.md, add three principles: Make every change as simple as possible, Find root causes instead of applying band-aids, and only touch what is necessary with no unintended side effects.

```markdown
## Core Principles
- Simple over clever. Write the boring solution.
- Fix root causes. No temporary workarounds.
- Minimal footprint. Do not touch files you do not need to.
```

---

### Example CLAUDE.md file
```markdown
# Project: [Your Project Name]

## Why
[One paragraph: what this project does and who uses it]

## Stack
- Java 21, Spring Boot 3.x
- PostgreSQL + Flyway
- Maven

## How to Run
- Build: `mvn clean install`
- Test: `mvn test`
- Run: `mvn spring-boot:run`

## Architecture
See `docs/architecture.md`.

## Rules
- Constructor injection only. Never field injection.
- Use SLF4J for logging. Never System.out.println.
- All exceptions must be logged or explicitly handled.
- Database changes must use Flyway migration files.
- Service layer only. No repository calls from Controllers.

## Definition of Done
- All tests pass (`mvn test`)
- No Checkstyle violations
- No debug code or TODOs left behind

## Principles
- Simple over clever.
- Fix root causes, not symptoms.
- Minimal changes — only touch what you need to.
```

Every session we have with Claude is only as good as the context you give it. A well-maintained CLAUDE.md means we stop repeating ourselves, Claude stops making the same mistakes, and we get consistent output from day one.
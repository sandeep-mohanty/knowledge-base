# PostgreSQL Row Level Security: Baking Tenant Isolation Into the Database

> *Stop relying on developers to remember WHERE clauses. Let the database enforce isolation automatically — and make data leaks structurally harder to cause.*

---

## The Real Problem Is Architectural, Not Human

You're building a multi-tenant SaaS platform. Schools, hospitals, and companies all share the same database. The only thing separating Tenant A's sensitive data from Tenant B's is a `WHERE tenant_id = ?` clause.

One developer forgets it. One raw query written at midnight during a hotfix. One new junior engineer who didn't know the convention. That's a data breach.

This isn't hypothetical. It's one of the most common real-world causes of multi-tenant data leakage — and it's entirely a systemic design failure, not a people problem. When forgetting a filter is *easy*, someone will eventually forget it.

The conventional fix is discipline: code reviews, linting rules, base repository classes that inject the filter automatically. These help, but they're all application-layer guardrails that can be bypassed. Every new query path, raw JDBC call, admin script, or reporting job has to opt into the convention.

**PostgreSQL Row Level Security (RLS) flips this.** Instead of requiring developers to opt *in* to filtering, RLS makes unfiltered access opt-out — enforced at the database engine level, invisible to application code, impossible to forget.

---

## What Row Level Security Actually Does

RLS lets you attach policies to a table. When a policy is active, PostgreSQL silently rewrites every query against that table to include the policy's condition — before the query planner even sees it.

A developer writes:

```sql
SELECT * FROM patient_records;
```

PostgreSQL, with an RLS policy active, executes something closer to:

```sql
SELECT * FROM patient_records
WHERE tenant_id = current_setting('app.tenant_id')::uuid;
```

The application never wrote the filter. The database added it. The developer cannot forget what they never had to write.

This works for `SELECT`, `INSERT`, `UPDATE`, and `DELETE` — all DML operations respect active policies. There's no backdoor through a bulk update or a reporting query.

---

## Setting Up RLS in PostgreSQL

### 1. Enable RLS on the Table

```sql
ALTER TABLE patient_records ENABLE ROW LEVEL SECURITY;
```

By default, once RLS is enabled, **table owners bypass all policies**. Regular application users do not. This distinction matters — your app should connect as a non-owner role.

### 2. Force RLS for Table Owners Too (Important)

```sql
ALTER TABLE patient_records FORCE ROW LEVEL SECURITY;
```

Without `FORCE ROW LEVEL SECURITY`, the database owner (often the migration user) bypasses all policies silently. In a shared environment, this can be a surprise. Add this line to be safe.

### 3. Create the Isolation Policy

```sql
CREATE POLICY tenant_isolation
ON patient_records
USING (
   tenant_id = current_setting('app.tenant_id')::uuid
)
WITH CHECK (
   tenant_id = current_setting('app.tenant_id')::uuid
);
```

Two clauses here, both important:

- `USING` — controls which rows are **visible** (applies to `SELECT`, `UPDATE`, `DELETE`)
- `WITH CHECK` — controls which rows can be **written** (applies to `INSERT`, `UPDATE`)

Without `WITH CHECK`, a tenant could potentially insert rows belonging to another tenant. Always define both.

### 4. Index Your Tenant Column

```sql
CREATE INDEX idx_patient_records_tenant_id ON patient_records(tenant_id);
```

RLS policies are evaluated per row. Without an index, PostgreSQL does a full table scan and filters afterward — that's catastrophic at scale. The index lets the database go directly to the right rows. This is non-negotiable in production.

---

## Wiring RLS Into Spring Boot

The flow is straightforward:

```
HTTP Request → JWT Filter → Extract tenant_id
→ Before query: SET LOCAL app.tenant_id = ?
→ All queries automatically scoped by RLS
→ Transaction commits → session variable cleared
```

### The Tenant-Aware Authentication Token

```java
public class TenantAuthentication extends UsernamePasswordAuthenticationToken {

   private final UUID tenantId;
   private final boolean superAdmin;

   public TenantAuthentication(Object principal, UUID tenantId, boolean superAdmin) {
       super(principal, null, extractAuthorities(superAdmin));
       this.tenantId = tenantId;
       this.superAdmin = superAdmin;
   }

   public UUID getTenantId() { return tenantId; }
   public boolean isSuperAdmin() { return superAdmin; }

   private static List<GrantedAuthority> extractAuthorities(boolean superAdmin) {
       return superAdmin
           ? List.of(new SimpleGrantedAuthority("ROLE_SUPER_ADMIN"))
           : List.of(new SimpleGrantedAuthority("ROLE_USER"));
   }
}
```

### The Tenant Context Filter

This filter runs before every request and sets the PostgreSQL session variable. The critical detail: use `SET LOCAL`, not `SET`. `SET LOCAL` scopes the variable to the current transaction — when the transaction ends, the variable is gone. The connection returns to the pool clean.

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class TenantContextFilter extends OncePerRequestFilter {

   private final DataSource dataSource;

   @Override
   protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain chain)
           throws ServletException, IOException {

       Authentication rawAuth = SecurityContextHolder.getContext().getAuthentication();

       if (rawAuth instanceof TenantAuthentication auth) {
           try (Connection conn = dataSource.getConnection()) {
               setTenantContext(conn, auth);
           } catch (SQLException ex) {
               log.error("Failed to set tenant context", ex);
               response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
               return;
           }
       }

       chain.doFilter(request, response);
   }

   private void setTenantContext(Connection conn, TenantAuthentication auth)
           throws SQLException {

       if (auth.isSuperAdmin()) {
           try (PreparedStatement stmt = conn.prepareStatement(
                   "SET LOCAL app.role = 'super_admin'")) {
               stmt.execute();
           }
       } else {
           try (PreparedStatement stmt = conn.prepareStatement(
                   "SET LOCAL app.tenant_id = ?")) {
               stmt.setObject(1, auth.getTenantId().toString());
               stmt.execute();
           }
       }
   }
}
```

### The Service and Repository Layer

Your repositories become completely clean. No tenant filtering anywhere:

```java
@Repository
public interface PatientRecordRepository extends JpaRepository<PatientRecord, UUID> {
   List<PatientRecord> findAll();                        // RLS scopes this automatically
   Optional<PatientRecord> findById(UUID id);            // RLS prevents cross-tenant access
   List<PatientRecord> findByStatus(RecordStatus status); // still scoped to current tenant
}
```

```java
@Service
@Transactional   // Critical — ensures SET LOCAL lives and dies with this transaction
@RequiredArgsConstructor
public class PatientRecordService {

   private final PatientRecordRepository repository;

   public List<PatientRecord> getAll() {
       return repository.findAll();  // Returns only current tenant's records
   }

   public PatientRecord getById(UUID id) {
       return repository.findById(id)
           .orElseThrow(() -> new ResourceNotFoundException(
               "Record not found or not accessible"
           ));
       // If id belongs to a different tenant, RLS makes it invisible
       // → orElseThrow fires → attacker learns nothing useful
   }
}
```

Notice the error handling in `getById`: since RLS makes another tenant's row invisible (not an error), a cross-tenant ID lookup simply returns empty — indistinguishable from a genuinely missing record. This is good security behavior by default.

---

## Super Admin Access

### Option A: BYPASSRLS Role (Blunt, Simple)

```sql
CREATE ROLE admin_service_user;
ALTER ROLE admin_service_user BYPASSRLS;
```

The admin service connects with these credentials and bypasses all policies entirely. Simple, but you lose the safety net globally for every query that admin service makes.

### Option B: Explicit Super Admin Policy (Recommended)

Keep one database user. Add a second RLS policy that opens access when the session declares itself as super admin:

```sql
-- Policy 1: normal tenant isolation (already exists)
CREATE POLICY tenant_isolation
ON patient_records
USING (tenant_id = current_setting('app.tenant_id')::uuid)
WITH CHECK (tenant_id = current_setting('app.tenant_id')::uuid);

-- Policy 2: super admin read access across all tenants
CREATE POLICY super_admin_read
ON patient_records
FOR SELECT
USING (
   current_setting('app.role', true) = 'super_admin'
);

-- Policy 3: super admins can't write without a valid tenant context
CREATE POLICY super_admin_write_guard
ON patient_records
FOR INSERT
WITH CHECK (
   tenant_id = current_setting('app.tenant_id')::uuid
);
```

PostgreSQL evaluates multiple policies with **OR logic** — a row is visible if it passes *any* active policy. So regular users see their tenant's rows (policy 1), super admins see everything for reads (policy 2), but nobody can insert a row without a valid tenant context (policy 3).

The `true` second argument in `current_setting('app.role', true)` tells PostgreSQL to return `NULL` instead of throwing an error when the variable isn't set. Without it, any session that doesn't set `app.role` would get a runtime error.

---

## The Production Problems Nobody Warns You About

### Connection Pooling: The Silent Killer

Connection pools (HikariCP, PgBouncer) reuse connections across requests. Session variables survive connection reuse. If a connection goes back to the pool with `app.tenant_id` still set from the previous request, the next request that picks up that connection inherits the wrong tenant context — silently.

`SET LOCAL` is the fix. It scopes the variable to the current transaction:

```sql
BEGIN;
SET LOCAL app.tenant_id = 'tenant-abc';
-- queries here
COMMIT;  -- app.tenant_id is gone. Connection returns to pool clean.
```

Spring's `@Transactional` enforces this automatically. **Make all service methods that touch RLS-protected tables transactional.** No exceptions.

If you use PgBouncer in transaction-pooling mode (the most common production setup), `SET LOCAL` is required — `SET` (session-scoped) won't work at all because connections aren't dedicated per session.

### Debugging Invisible Filters

RLS invisibility is a feature, but it's disorienting during debugging. A developer writes a query, gets zero rows back, and spends an hour questioning joins and data migrations — not realizing RLS is filtering everything.

Add this to your team's runbook:

> **If a query returns unexpectedly empty results on a tenant-isolated table, check whether `app.tenant_id` is set in the current session:**
> ```sql
> SELECT current_setting('app.tenant_id', true);
> ```
> If this returns NULL or empty, RLS is filtering all rows.

You can also temporarily disable RLS for debugging purposes (as a superuser):

```sql
SET row_security = off;  -- Only works for superusers; harmless for regular users
SELECT * FROM patient_records;
SET row_security = on;
```

### Policy Complexity and Performance

Simple equality policies (`tenant_id = ...`) are fast. Policies involving subqueries or function calls are evaluated per-row and can be expensive.

**Avoid this:**

```sql
-- Slow: subquery evaluated for every row
CREATE POLICY tenant_isolation ON orders
USING (
   tenant_id IN (SELECT id FROM allowed_tenants WHERE user_id = current_user)
);
```

**Prefer this:**

```sql
-- Fast: simple equality against a session variable
CREATE POLICY tenant_isolation ON orders
USING (
   tenant_id = current_setting('app.tenant_id')::uuid
);
```

If you genuinely need complex access rules, compute the answer in the application layer, set it as a session variable, and use a simple equality check in the policy.

### Schema Migrations

RLS policies must be updated when you add new tables. Create a checklist for your migration process:

```sql
-- Template for every new tenant-scoped table
ALTER TABLE new_table ENABLE ROW LEVEL SECURITY;
ALTER TABLE new_table FORCE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON new_table
USING (tenant_id = current_setting('app.tenant_id')::uuid)
WITH CHECK (tenant_id = current_setting('app.tenant_id')::uuid);

CREATE INDEX idx_new_table_tenant_id ON new_table(tenant_id);
```

Consider a Flyway/Liquibase migration template that includes these four statements automatically for any table with a `tenant_id` column.

---

## Testing Your RLS Policies

RLS is useless if it's broken and nobody notices. Write integration tests that verify isolation:

```java
@SpringBootTest
@Transactional
class RlsIsolationTest {

   @Autowired PatientRecordRepository repository;
   @Autowired EntityManager em;

   @Test
   void tenantA_cannotSee_tenantBRecords() {
       UUID tenantA = UUID.randomUUID();
       UUID tenantB = UUID.randomUUID();

       // Seed data as superuser (bypassing RLS)
       em.createNativeQuery(
           "INSERT INTO patient_records (id, tenant_id, name) VALUES (gen_random_uuid(), ?, 'Alice')"
       ).setParameter(1, tenantA).executeUpdate();

       em.createNativeQuery(
           "INSERT INTO patient_records (id, tenant_id, name) VALUES (gen_random_uuid(), ?, 'Bob')"
       ).setParameter(1, tenantB).executeUpdate();

       // Set session context to Tenant A
       em.createNativeQuery("SET LOCAL app.tenant_id = ?")
           .setParameter(1, tenantA.toString())
           .executeUpdate();

       List<?> results = repository.findAll();

       assertThat(results).hasSize(1);
       assertThat(results.get(0)).extracting("name").isEqualTo("Alice");
       // Bob is invisible — RLS works
   }

   @Test
   void crossTenantInsert_isRejected() {
       UUID tenantA = UUID.randomUUID();
       UUID tenantB = UUID.randomUUID();

       em.createNativeQuery("SET LOCAL app.tenant_id = ?")
           .setParameter(1, tenantA.toString())
           .executeUpdate();

       // Attempt to insert a row for Tenant B while authenticated as Tenant A
       assertThatThrownBy(() ->
           em.createNativeQuery(
               "INSERT INTO patient_records (id, tenant_id, name) VALUES (gen_random_uuid(), ?, 'Malicious')"
           ).setParameter(1, tenantB).executeUpdate()
       ).isInstanceOf(Exception.class); // WITH CHECK violation
   }
}
```

Run these tests in your CI pipeline. A failing RLS test is a P0 incident waiting to happen in production.

---

## When to Use RLS — and When Not To

**RLS is the right tool when:**
- You have multiple developers writing queries against shared tables
- Strict tenant isolation is a regulatory or contractual requirement (HIPAA, SOC 2, GDPR)
- You use reporting tools, admin panels, or analytics pipelines that bypass your ORM
- The cost of a data leak — reputationally or legally — is high

**RLS adds unnecessary complexity when:**
- You're building a single-tenant application
- Your team is small and the codebase is new enough to review every query
- You're in very early product discovery where iteration speed dominates
- You use a database that doesn't support RLS (then consider application-level Hibernate filters as an alternative)

---

## Alternatives Worth Knowing

If you can't use RLS (wrong database, existing architecture constraints), here are the next-best options:

**Hibernate Filters** — Define a filter at the ORM level that gets applied to all queries through that session:
```java
@FilterDef(name = "tenantFilter", parameters = @ParamDef(name = "tenantId", type = UUID.class))
@Filter(name = "tenantFilter", condition = "tenant_id = :tenantId")
@Entity
public class PatientRecord { ... }
```
Less safe than RLS (bypassed by native queries) but better than manual `WHERE` clauses everywhere.

**Schema-per-tenant** — Each tenant gets its own schema. Full isolation, but operationally expensive at scale (thousands of tenants = thousands of schemas).

**Database-per-tenant** — Maximum isolation, maximum cost. Reserved for high-compliance scenarios where data residency or regulatory requirements demand it.

---

## Summary

Row Level Security moves tenant isolation from a convention developers must remember to a constraint the database engine enforces:

| Approach | Where Enforced | Bypassable By | Failure Mode |
|---|---|---|---|
| Manual `WHERE` clauses | Application code | Any raw query | Silent data leak |
| Hibernate Filters | ORM layer | Native queries | Silent data leak |
| PostgreSQL RLS | Database engine | Only superuser | Query returns empty |

The fundamental shift is this: with manual filtering, the secure path requires action — a developer must remember to add the clause. With RLS, the secure path is the default — a developer has to actively work to bypass it.

The strongest security systems aren't the ones that rely on everyone remembering every rule. They're the ones designed so that the dangerous mistakes are structurally harder to make than the safe ones.

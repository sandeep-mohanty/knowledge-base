# 14 Essential Liquibase Best Practices for Spring Boot Developers

In the fast-paced world of Spring Boot development, your database schema is often the bottleneck. While Java code is easy to refactor, database changes are stubborn, risky, and hard to undo. If you are using **Liquibase**, you have the right tool — but are you using it correctly?

A messy changelog leads to deployment failures, “schema drift,” and performance nightmares. Whether you are a solo developer or leading a large team, these **Liquibase best practices** will transform your database evolution from a liability into a competitive advantage.

---

### 1. Document Naming Standards in a Team-Wide Style Guide
Chaos starts with inconsistent file names. Before you write your next changeset, create a shared document that dictates how files and IDs are named.
* **Files:** `2025-11-27-add-users.xml` vs. `changelog-v1.xml`.
* **IDs:** Sequential (1, 2, 3) vs. Timestamp (2025112701).
* **Authors:** Real usernames vs. generic tags.

Consistency here prevents merge conflicts and makes the project easier to navigate for new hires.

---

### 2. Follow Consistent Naming Conventions (Especially for Constraints!)
This is the number one cause of headaches during refactoring. If you don’t name your constraints, the database generates random names like `FK_293847`.

**The Fix:** Always explicitly name constraints so you can easily drop or reference them later. Use a pattern like `PK_{Table}`, `FK_{Table}_{Column}`, or `IX_{Table}_{Column}`.

```xml
<addForeignKeyConstraint baseColumnNames="user_id"
                         baseTableName="orders"
                         constraintName="FK_ORDERS_USER_ID"
                         referencedColumnNames="id"
                         referencedTableName="users"/>
```

---

### 3. Always Write Comments
Code tells you what; comments tell you why. A changeset adds a column, but why was it added? Was it for a specific Jira ticket? A compliance requirement?

Always add a `<comment>` tag to your changesets. Two years from now, when you are debugging a legacy issue, you will thank yourself.

```xml
<changeSet id="20251127-01" author="m.scott">
    <comment>Adding 'is_priority' flag for the new VIP 
customer feature (Ticket JIRA-420)</comment>
    <addColumn tableName="customers">
        <column name="is_priority" type="boolean"/>
    </addColumn>
</changeSet>
```

---

### 4. Use Meaningful Commit Messages
Your version control history is part of your documentation. When modifying changelog files, avoid lazy commit messages like “update db.” Instead, describe the business value: “feat: Add inventory_count to products table for real-time stock tracking.”

---

### 5. Avoid Combining Too Many Changes in a Single Changeset
Keep your changesets atomic. Do not create five tables, three indexes, and insert reference data all in one changeset ID.

If one part of a massive changeset fails, the whole transaction rolls back (on supported DBs), but debugging exactly which line failed becomes difficult. Follow the **Single Responsibility Principle**: One logical change = One changeset.

**Bad:** One changeset that creates the `users`, `orders`, and `products` tables.  
**Good:**
* **Changeset 1:** `create-table-users`
* **Changeset 2:** `create-table-products`
* **Changeset 3:** `create-table-orders`

---

### 6. Separate Schema Changes from Data Changes
Don’t mix DDL (Create Table) and DML (Insert Data) in the same file or changeset. Schema changes are usually fast; data changes (like backfilling a new column) can be slow and lock tables. Separating them allows you to manage transaction logs better and isolate failures.

**changelog-master.xml**
```xml
    <include file="changes/01-create-lookup-tables.xml"/> (Schema)
    <include file="data/01-load-country-codes.xml"/> (Data)
```



---

### 7. Write Preconditions
Preconditions are your safety guard rails. They check the state of the database before attempting a change.

For example, before adding a column, check if it already exists. This prevents your migration from crashing with “Column already exists” errors if a script is accidentally run twice or against a drift environment.

```xml
<preConditions onFail="MARK_RAN">
    <not>
        <columnExists tableName="users" columnName="email"/>
    </not>
</preConditions>
```

---

### 8. Provide Rollback Scenarios
In production, the ability to “undo” is just as important as the ability to deploy. While Liquibase can auto-generate rollbacks for simple commands (like `createTable`), it has no idea how to reverse complex logic or raw SQL.

Always define a `<rollback>` block. If a deployment fails, you need a defined path back to a stable state without manually hacking the database.

```xml
<changeSet id="20251127-03" author="mammad">
    <sql>
        CREATE VIEW active_users AS SELECT * FROM users WHERE active = 1;
    </sql>
    <rollback>
        <sql>DROP VIEW active_users;</sql>
    </rollback>
</changeSet>
```

---

### 9. Consider Adding Constraints and Indexes Immediately
Don’t deploy “naked” tables.

* **Constraints:** Enforce data integrity immediately (Not Null, Unique). Cleaning up bad data later is infinitely harder than preventing it now.
* **Indexes:** If you add a Foreign Key or a column that will be queried heavily, add the index in the same release. Forgetting this is a common cause of performance degradation immediately after a release.

```xml
<changeSet id="20251127-04" author="mammad">
    <addColumn tableName="orders">
        <column name="customer_id" type="bigint">
            <constraints nullable="false" foreignKeyName="FK_ORDERS_CUSTOMER"/>
        </column>
    </addColumn>
    <createIndex tableName="orders" indexName="IX_ORDERS_CUSTOMER_ID">
        <column name="customer_id"/>
    </createIndex>
</changeSet>
```

---

### 10. Consider Giving Default Values
When adding a new Non-Null column to a table that already contains data, the migration will crash because existing rows won’t have a value. You should provide a `defaultValue` in your changeset.

```xml
<changeSet id="20251127-05" author="mammad">
    <addColumn tableName="users">
        <column name="newsletter_opt_in" type="boolean" defaultValue="false">
            <constraints nullable="false"/>
        </column>
    </addColumn>
</changeSet>
```

---

### 11. Use Formatted SQL Only When Necessary
Liquibase’s XML or YAML formats are powerful because they are database-agnostic. They allow you to deploy the same changelog to H2 (test) and PostgreSQL (prod). 

Only use SQL-formatted files when:
* You are executing massive ETL scripts (performance).
* You need vendor-specific features (e.g., Oracle PL/SQL, Postgres Triggers).
* You are importing legacy SQL dumps.

---

### 12. Avoiding Manual Changes
**The Golden Rule:** Once you adopt Liquibase, you must revoke write access for developers in higher environments. Never manually run `ALTER` or `INSERT` statements to "fix" things quickly. This causes **Schema Drift**, where your Liquibase changelog no longer matches the actual database, leading to checksum errors and failed future deployments.

---

### 13. Monitor DATABASECHANGELOGLOCK Table
If your Spring Boot application hangs during startup, it is often due to a “zombie lock.”

Liquibase uses this table to ensure only one instance runs migrations at a time. If a pod crashes mid-migration, the lock might remain set (`locked=1`). Monitor this table for entries that have been locked for more than a few minutes.

---

### 14. Monitoring and Troubleshooting: Track DB Performance
Schema changes have consequences. Adding an index speeds up reads but slows down writes. Changing a column type can trigger a table rewrite.

**Action:** Immediately after a deployment involving DB changes, monitor your APM tools (Datadog, New Relic, etc.). Look for spikes in Database CPU or increased Query Latency to catch performance regressions early.

---

### Conclusion
Liquibase is a powerful tool, but it requires discipline. By enforcing naming conventions, writing robust rollbacks, and treating your XML/YAML files with the same care as your Java classes, you ensure that your Spring Boot application rests on a stable, scalable data foundation.

Stop fearing database releases. Start automating them.
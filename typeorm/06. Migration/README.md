## **Migrations**

<details>
<summary>81. What are migrations in TypeORM?</summary>

Migrations in TypeORM are version-controlled scripts that modify your database schema over time. They allow you to evolve your database structure safely and consistently, especially in production environments or teams. Migrations track changes such as creating tables, altering columns, adding indices, and more.

**Key Points:**
- Migrations are TypeScript/JavaScript files containing up() and down() methods.
- up(): Defines the schema changes to apply (e.g., create table, add column).
- down(): Defines how to revert those changes (e.g., drop table, remove column).
- Migrations are tracked in a special migrations table in your database.
- They ensure all environments (dev, staging, prod) have the same schema.

**Production Use-Case:**
- Never use `synchronize: true` in production; always use migrations for schema changes.
- Migrations are essential for CI/CD, rollbacks, and team collaboration.

**Interview Tip:**
- Explain migrations as "version control for your database schema".
- Mention that migrations are repeatable, reversible, and auditable.
- Highlight their role in safe deployments and team workflows.

</details>

<details>
<summary>82. How do you generate a migration file?</summary>

You generate a migration file using the TypeORM CLI or programmatically. The CLI compares your current entities with the database and creates a migration script for the differences.

**CLI Command:**
```bash
typeorm migration:generate -n AddUserTable
```
- `-n AddUserTable`: Names the migration file.
- The CLI creates a file in the migrations directory (e.g., `src/migrations/`).

**Manual Creation:**
```bash
typeorm migration:create -n CustomMigration
```
- Creates an empty migration file for custom logic.

**Migration File Structure:**
```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddUserTable1680000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`CREATE TABLE "user" ("id" SERIAL PRIMARY KEY, "name" VARCHAR(255))`);
  }
  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "user"`);
  }
}
```

**Interview Tip:**
- Mention the difference between `migration:generate` (auto-generates from entity changes) and `migration:create` (manual, for custom logic).
- Always review generated migrations before running them in production.

</details>

<details>
<summary>83. How do you run migrations?</summary>

You run migrations using the TypeORM CLI or programmatically. This applies all pending migration scripts to your database.

**CLI Command:**
```bash
typeorm migration:run
```
- Applies all migrations that haven't been run yet.

**Programmatic Execution:**
```typescript
await dataSource.runMigrations();
```
- Useful for custom scripts or CI/CD pipelines.

**Production Use-Case:**
- Always run migrations before deploying new code.
- Use migrations in CI/CD to keep schema in sync across environments.

**Interview Tip:**
- Emphasize that running migrations is a safe, repeatable way to update schema.
- Mention that migrations are tracked in a migrations table to prevent double execution.

</details>

<details>
<summary>84. How do you revert migrations?</summary>

You revert migrations using the TypeORM CLI or programmatically. This rolls back the last applied migration, undoing its schema changes.

**CLI Command:**
```bash
typeorm migration:revert
```
- Reverts the most recent migration (calls its `down()` method).

**Programmatic Execution:**
```typescript
await dataSource.undoLastMigration();
```
- Useful for automated rollback scripts.

**Production Use-Case:**
- Use revert for emergency rollbacks or to undo accidental schema changes.
- Always test reverts in staging before running in production.

**Interview Tip:**
- Explain that revert is the "undo" for migrations, using the down() method.
- Mention that revert is tracked in the migrations table for auditability.

</details>

<details>
<summary>85. What is the difference between synchronize and migrations?</summary>

**synchronize** is a TypeORM option that automatically updates the database schema to match your entities every time the app starts. **migrations** are explicit, version-controlled scripts for schema changes.

| Feature         | synchronize           | migrations                |
|-----------------|----------------------|---------------------------|
| Safety          | Risky in production  | Safe, controlled          |
| Rollback        | No                   | Yes (down method)         |
| Audit Trail     | No                   | Yes (migrations table)    |
| Team Workflow   | Not team-friendly    | Team-friendly             |
| Production Use  | Not recommended      | Recommended               |

**Best Practice:**
- Use `synchronize: true` only for local development or prototyping.
- Always use migrations for production and team environments.

**Interview Tip:**
- Explain that synchronize is "auto-sync" (dangerous for production), migrations are "versioned, auditable, and reversible".
- Mention that migrations support CI/CD, rollbacks, and team collaboration.

</details>

<details>
<summary>86. How do you write custom migration logic?</summary>

Custom migration logic is written by manually editing the migration file created with `typeorm migration:create`. You implement the `up()` and `down()` methods using the `QueryRunner` API to execute any SQL or schema changes you need.

**Example:**
```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class CustomMigration1680000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    // Custom SQL: Add a column
    await queryRunner.query(`ALTER TABLE "user" ADD COLUMN "isActive" BOOLEAN DEFAULT true`);
    // Custom data: Update all users to active
    await queryRunner.query(`UPDATE "user" SET "isActive" = true`);
  }
  async down(queryRunner: QueryRunner): Promise<void> {
    // Revert column addition
    await queryRunner.query(`ALTER TABLE "user" DROP COLUMN "isActive"`);
  }
}
```

**Tips:**
- You can use any SQL statement: DDL, DML, etc.
- Use QueryRunner methods for more complex logic (e.g., transactions, table creation).
- Always provide a safe rollback in `down()`.

**Interview Tip:**
- Explain that custom migrations allow for data fixes, complex schema changes, and business logic not covered by auto-generation.
- Mention the importance of testing custom logic in staging.

</details>

<details>
<summary>87. What methods are available in the migration class?</summary>

A migration class implements the `MigrationInterface` and must provide two methods:
- `up(queryRunner: QueryRunner): Promise<void>` — applies the migration.
- `down(queryRunner: QueryRunner): Promise<void>` — reverts the migration.

**QueryRunner API:**
- `query(sql: string, params?: any[])`: Run raw SQL.
- `createTable`, `dropTable`, `addColumn`, `dropColumn`, `createIndex`, `dropIndex`, etc.
- Transaction methods: `startTransaction`, `commitTransaction`, `rollbackTransaction`.

**Example:**
```typescript
export class AddProfileTable1680000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(/* ... */);
  }
  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable(/* ... */);
  }
}
```

**Interview Tip:**
- Mention that up/down are required, and QueryRunner provides full control over schema and data.

</details>

<details>
<summary>88. How do you handle data migrations?</summary>

Data migrations involve changing the actual data, not just the schema. You use the migration's `up()` and `down()` methods to run SQL UPDATE, INSERT, DELETE, or custom scripts.

**Example:**
```typescript
export class MigrateUserStatus1680000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    // Set all users to active
    await queryRunner.query(`UPDATE "user" SET "isActive" = true`);
    // Migrate old status column to new enum
    await queryRunner.query(`UPDATE "user" SET "status" = 'active' WHERE "isActive" = true`);
  }
  async down(queryRunner: QueryRunner): Promise<void> {
    // Revert status migration
    await queryRunner.query(`UPDATE "user" SET "status" = NULL WHERE "status" = 'active'`);
  }
}
```

**Best Practices:**
- Always test data migrations on a copy of production data.
- Make migrations idempotent and reversible.
- Use transactions for safety.

**Interview Tip:**
- Explain that data migrations are for business logic changes, not just schema.
- Mention the importance of backups and testing.

</details>

<details>
<summary>89. How do you create indices in migrations?</summary>

You create indices using the QueryRunner API or raw SQL in the migration's `up()` method, and drop them in `down()`.

**Example:**
```typescript
export class AddUserEmailIndex1680000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`CREATE INDEX "IDX_USER_EMAIL" ON "user" ("email")`);
  }
  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX "IDX_USER_EMAIL"`);
  }
}
```

**Best Practices:**
- Name indices clearly (e.g., IDX_TABLE_COLUMN).
- Add indices for frequently queried columns.
- Remove unused indices to improve performance.

**Interview Tip:**
- Explain that indices improve query speed and are managed via migrations for consistency.

</details>

<details>
<summary>90. How do you rename columns or tables in migrations?</summary>

You rename columns or tables using raw SQL or QueryRunner methods in the migration file.

**Example:**
```typescript
export class RenameUserNameColumn1680000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "user" RENAME COLUMN "name" TO "fullName"`);
    await queryRunner.query(`ALTER TABLE "user" RENAME TO "account"`);
  }
  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`ALTER TABLE "account" RENAME COLUMN "fullName" TO "name"`);
    await queryRunner.query(`ALTER TABLE "account" RENAME TO "user"`);
  }
}
```

**Best Practices:**
- Always update related indices and constraints after renaming.
- Test renames in staging to catch issues with ORM mappings.

**Interview Tip:**
- Mention that renaming is a schema change and must be tracked for code compatibility.

</details>

<details>
<summary>91. How do you handle migration conflicts in a team environment?</summary>

Migration conflicts occur when multiple team members generate migrations that affect the same schema objects. To resolve:
- Communicate and coordinate migration creation.
- Rebase and merge migrations in order.
- Use descriptive migration names and timestamps.
- Test migrations in a shared environment before merging.
- If conflicts occur, manually edit migration files to resolve order and logic.

**Best Practices:**
- Use a single source of truth for migrations (e.g., main branch).
- Run all migrations in CI before merging.
- Document migration changes in PRs.

**Interview Tip:**
- Explain that migration conflicts are a team challenge, solved by communication, version control, and testing.

</details>

<details>
<summary>92. What is the migrations table and what does it track?</summary>

The migrations table (usually `migrations` or `typeorm_migrations`) is a special table in your database that tracks which migrations have been applied. It stores:
- Migration name (class/file name)
- Timestamp of execution
- Order of migrations

**Purpose:**
- Prevents running the same migration twice.
- Enables safe rollbacks and audit trails.
- Ensures all environments are in sync.

**Interview Tip:**
- Mention that the migrations table is the "history log" for schema changes.
- It's essential for reliable deployments and debugging.

</details>

<details>
<summary>93. How do you test migrations?</summary>

Testing migrations ensures they work as expected and don't break your database. Steps:
- Run migrations on a test database (local or CI).
- Verify schema and data changes.
- Revert migrations and check rollback.
- Use backups for production-like data.
- Automate migration tests in CI/CD pipelines.

**Best Practices:**
- Never test on production data directly.
- Use migration:run and migration:revert in scripts.
- Validate with integration tests.

**Interview Tip:**
- Explain that migration testing is critical for safe deployments and disaster recovery.

</details>

<details>
<summary>94. How do you create production-safe migrations?</summary>

Production-safe migrations are:
- Reviewed and tested before deployment.
- Idempotent (can be run multiple times safely).
- Reversible (down() method works).
- Avoid long-running locks or blocking operations.
- Use transactions for atomicity.
- Avoid destructive changes without backups.
- Documented for the team.

**Best Practices:**
- Always test in staging.
- Use backups and monitoring.
- Communicate changes to the team.

**Interview Tip:**
- Mention that production-safe migrations are the backbone of reliable deployments.
- Emphasize testing, backups, and rollback plans.

</details>

<details>
<summary>95. What is the migrationsRun option?</summary>

`migrationsRun` is a TypeORM DataSource option. If set to `true`, it automatically runs all pending migrations when the app starts.

**Usage:**
```typescript
const dataSource = new DataSource({
  // ...other options
  migrationsRun: true,
});
```

**Best Practices:**
- Use in development or CI environments for convenience.
- Avoid in production unless you have strict migration review and testing.

**Interview Tip:**
- Explain that migrationsRun automates schema updates, but manual control is safer for production.

</details>
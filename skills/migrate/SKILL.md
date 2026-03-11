---
name: migrate
description: Database migration workflow. Use when user says /migrate, needs to create or review schema migrations, add/modify database tables or columns, or manage migration state. Auto-detects the project's ORM and migration tool.
---

# Database Migrations

Safe schema change workflow. Auto-detect the migration tool before proceeding.

## Usage

```
/migrate                    # Check migration state and pending changes
/migrate create <name>      # Generate a new migration
/migrate review             # Review pending migrations for safety
```

## Tool Detection

Identify the migration tool from project files:

| File/Config | Tool |
|-------------|------|
| `alembic.ini` or `alembic/` | Alembic (SQLAlchemy) |
| `prisma/schema.prisma` | Prisma Migrate |
| `knexfile.*` or `migrations/` in JS project | Knex.js |
| `db/migrate/` in Rails project | ActiveRecord Migrations |
| `django.contrib.migrations` | Django Migrations |
| `diesel.toml` or `migrations/` in Rust project | Diesel |
| `flyway.*` or `sql/migrations/` | Flyway |
| `liquibase.*` | Liquibase |
| `ent/migrate/` | Ent (Go) |
| `drizzle.config.*` | Drizzle ORM |
| `migrations/` + `goose` or `golang-migrate` | Go migrations |
| `TypeORM` in package.json | TypeORM |
| `sequelize` in package.json | Sequelize |

Read the project's existing migrations to understand conventions before generating new ones.

## Migration Workflow

```
1. DETECT   → Identify migration tool and current state
2. PLAN     → Design the schema change
3. GENERATE → Create migration file(s)
4. REVIEW   → Check for safety issues
5. TEST     → Run migration against test DB
6. APPLY    → Run in target environment
```

## Phase 1: Detect State

- Identify the migration tool (see table above)
- Check current migration status (pending, applied)
- Read recent migrations to understand conventions (naming, style)
- Check if the project uses auto-generation or hand-written migrations

## Phase 2: Plan the Change

Before writing any migration:

- **What's changing?** (new table, new column, modify column, drop, index, constraint)
- **Is it backward compatible?** Can the old code still work during deployment?
- **Does it need data migration?** Not just schema change but data transformation?
- **What's the rollback plan?** Can this be reversed without data loss?

### Schema Design: Prefer Normalization

**Default to normalized schemas** (3NF) — eliminate data duplication, enforce referential integrity, keep a single source of truth.

**Before denormalizing, consider views first:**
- A **database view** gives you a flattened query shape while keeping the underlying tables normalized — no data duplication, no sync issues
- A **materialized view** adds caching on top — precomputed results that refresh on a schedule or trigger, ideal for dashboards and reporting
- Views are the right default when the problem is "this join is annoying to write repeatedly" rather than "this join is too slow to execute"

**Denormalize only when views aren't sufficient:**
- Materialized view refresh lag is unacceptable for the use case
- Write patterns require denormalized data inline (e.g., embedding a snapshot of a value at the time of an event)
- The database doesn't support materialized views well
- Event sourcing or append-only logs where the shape is inherently flat

When denormalizing, **document why** in a comment on the migration or in the model. Unjustified denormalization becomes tech debt fast — duplicated data drifts out of sync.

## Phase 3: Generate

Follow the project's existing conventions for:
- Migration file naming (timestamps, sequential numbers)
- Up/down or forward/rollback structure
- SQL style (raw SQL vs ORM-generated)

## Phase 4: Review (Safety Checklist)

### Destructive Operations (HIGH RISK)
- [ ] **DROP TABLE / DROP COLUMN**: Is the data backed up? Is the column truly unused?
- [ ] **ALTER COLUMN type**: Will existing data convert cleanly? Could it truncate?
- [ ] **NOT NULL without default**: Will this fail on existing rows?
- [ ] **Rename column/table**: Is all application code updated to use the new name?

### Performance Considerations
- [ ] **Large table ALTER**: On tables with millions of rows, will this lock the table?
- [ ] **New index on large table**: Consider `CREATE INDEX CONCURRENTLY` (Postgres) or equivalent
- [ ] **Backfill data**: Use batched updates, not a single UPDATE on millions of rows

### Backward Compatibility
- [ ] Can the current application code work with both the old AND new schema?
- [ ] If deploying code and migration separately, which order is safe?
- [ ] Does this migration need to be split into phases? (add column → backfill → add constraint)

### General
- [ ] Migration has a rollback/down method
- [ ] Foreign keys reference correct tables and have appropriate ON DELETE behavior
- [ ] Indexes added for columns used in WHERE, JOIN, ORDER BY
- [ ] Default values make sense for existing rows
- [ ] Column types are appropriate for the data

## Phase 5: Test

- Run migration against a test database (or local dev)
- Verify the schema matches expectations
- Run application tests to confirm nothing breaks
- Test the rollback/down migration

## Common Patterns

### Add Nullable Column (Safe)
```
Add column with NULL allowed → Deploy code that uses it → Backfill data → Add NOT NULL constraint
```

### Rename Column (Multi-phase)
```
Phase 1: Add new column, copy data, update code to write to both
Phase 2: Deploy code that reads from new column
Phase 3: Drop old column
```

### Add Index Without Downtime
```
Use CONCURRENTLY (Postgres) or ALGORITHM=INPLACE (MySQL) to avoid table locks
```

## Anti-Patterns

- **Never** commit a migration that drops data without a backup plan
- **Never** run migrations in production without testing in staging first
- **Never** modify an already-applied migration — create a new one
- **Never** put application logic in migrations (imports from app code that may change)
- **Avoid** long-running migrations that lock tables during peak traffic

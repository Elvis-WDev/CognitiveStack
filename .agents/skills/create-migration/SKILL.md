---
name: create-migration
description: Use when changing the SQLite schema, migration scripts, repositories that depend on the schema, generated schema docs, or data migration behavior.
---

# Create Migration

## Context To Load

Read:

- `docs/architecture/database.md`
- `docs/architecture/stack.md`
- The OpenSpec change's design.
- `docs/security/principles.md`
- `docs/security/access-control.md` when the database holds several companies or workspaces.
- The existing scripts in `Infrastructure/Migrations/` and the `schema_migrations` table.

## Workflow

1. Understand the data model change and the affected workflows.
2. Decide whether the migration is additive, destructive, or data-transforming.
3. Add the next numbered script, such as `Migrations/0007_add_stock_movements.sql`, embedded
   as a resource in `Infrastructure`.
4. Declare new tables `STRICT`, with the storage types from `docs/architecture/database.md`:
   money and fractional quantities never `REAL`, enumerations and booleans under `CHECK`.
5. Express critical invariants as `UNIQUE`, `CHECK`, and foreign key constraints. When the
   database holds several companies, owned tables carry the owner column, indexes lead with
   it, and children use composite foreign keys that include it.
6. Index every foreign key and business lookup key, and confirm critical queries with
   `EXPLAIN QUERY PLAN`.
7. Update repositories, Dapper type handlers, and application DTOs.
8. Test the migration against an empty database and against a copy of a database at the
   previous version, including a failure that must roll back.
9. Regenerate the schema reference and migration list in `docs/generated/`.
10. Verify with `dotnet build {{ProjectName}}.slnx -warnaserror` and
    `dotnet test {{ProjectName}}.slnx`.

## Guardrails

- Never edit a script that has been applied anywhere; its checksum is recorded. Correct it
  with a new script.
- The runner takes a verified backup before applying pending scripts; never bypass it.
- Each script runs in its own transaction, followed by `PRAGMA foreign_key_check` and
  `PRAGMA integrity_check`.
- Destructive changes require an explicit data plan: copy, verify, then remove, across
  separate releases when the data matters.
- An owned table never ships without its owner column when the database holds several
  companies.
- Generated schema references are regenerated, not hand edited.

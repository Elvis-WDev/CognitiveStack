# Database Architecture

## Primary Database

- Engine: PostgreSQL.
- ORM/query layer: Prisma.
- Migration strategy: versioned Prisma migrations.

## Data Ownership

| Table/Collection | Owner | Tenant scoped | Notes |
| --- | --- | --- | --- |
| `{{TABLE}}` | `{{MODULE}}` | Yes/No | Description. |

## Migration Rules

- Schema changes require a migration.
- Generated database docs must be refreshed after migrations.
- Destructive migrations require an explicit data plan.
- Production migration credentials should be limited after initial provisioning.
- Use PostgreSQL in production. Do not silently fall back to SQLite.
- Avoid data loss on redeploy. Application containers must not own production database storage.

## Integrity Constraints

Critical invariants live in the database, not only in application code. Code-level checks
race, and every new code path can forget them.

```sql
-- One outcome per idempotency key, scoped by tenant, user, and operation.
UNIQUE (tenant_id, user_id, operation, idempotency_key)

-- One local record per provider payment, per tenant.
UNIQUE (tenant_id, external_payment_id)

-- Each provider event is processed once.
UNIQUE (provider, provider_event_id)

-- A balance never goes negative, whatever the code does.
CHECK (balance >= 0)
```

- Use foreign keys for every real relationship.
- For tenant-owned children, use composite foreign keys that include the tenant, so a row cannot reference another tenant's parent.
- Prisma does not express `CHECK` constraints or Row-Level Security policies in the schema. Add them to the migration SQL and list them in this document.
- When Row-Level Security is used, set the tenant per transaction and never per pooled connection.

```prisma
model Project {
  id       String @id @default(cuid())
  tenantId String
  tasks    Task[]

  @@unique([tenantId, id])
}

model Task {
  id        String  @id @default(cuid())
  tenantId  String
  projectId String
  project   Project @relation(fields: [tenantId, projectId], references: [tenantId, id])

  @@index([tenantId, projectId])
}
```

See `idempotency.md` and `../security/access-control.md` for the rules these constraints back.

## Indexing

- List critical queries.
- Map each critical query to an index.
- Lead composite indexes with the tenant column for tenant-scoped queries, such as `(tenant_id, created_at)`.

## Auth Tables

Better Auth owns its auth-related tables. Application code should not manually mutate password/session internals except through documented Better Auth APIs.

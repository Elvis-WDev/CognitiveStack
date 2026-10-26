# Database Architecture

## Primary Database

- Engine: SQLite, one database file per installation, company, or workspace as the product
  defines.
- Access: Microsoft.Data.Sqlite with Dapper, inside `Infrastructure` only.
- Migrations: versioned SQL scripts embedded in `Infrastructure` and applied at startup.
- Backups: `SqliteConnection.BackupDatabase`, verified after every copy.

## File Location

- Store databases, backups, logs, and generated documents in application data folders, never
  in the installation folder:
  - per user: `%LocalAppData%\{{Company}}\{{ProjectName}}\`;
  - per machine, shared by every Windows user: `%ProgramData%\{{Company}}\{{ProjectName}}\`,
    with folder permissions set by the installer.
- Never keep the only copy of a database in `Program Files` or in any folder the updater
  replaces.
- Resolve paths through one application-paths service. No other code builds data paths.

## Connections

Open a short-lived connection per unit of work, and configure every connection the same way:

```sql
PRAGMA foreign_keys = ON;      -- off by default in SQLite, per connection
PRAGMA busy_timeout = 5000;    -- wait for a lock instead of failing immediately
```

Set once per database file, during creation or migration:

```sql
PRAGMA journal_mode = WAL;     -- readers do not block the writer
PRAGMA synchronous = FULL;     -- durability over speed for money and stock
```

- Use `synchronous = NORMAL` only for data the product can afford to lose on power failure,
  and record that decision.
- Start write transactions immediately, not deferred, so the write lock is taken at the start
  and two writers cannot both read stale data before writing.
- One writer at a time is a property of SQLite. Keep write transactions short.

## Multiple Terminals

SQLite is a local, single-machine database. Do not share one database file between several
machines through a network folder: file locking over SMB is unreliable and corrupts data.
When several terminals must share data, design a local server process or a client-server
database, and record the decision in an ADR.

## Dapper Rules

- Every value reaches SQL as a parameter. Never concatenate user input, file names, or sort
  fields into SQL. Allowlist column names for dynamic sorting.
- SQL lives in repositories in `Infrastructure`, close to the mapping it feeds.
- Map rows to explicit records or DTOs; never return `dynamic` beyond the repository.
- Register Dapper type handlers once at startup for types SQLite does not store natively:
  `decimal`, `DateTimeOffset`, `DateOnly`, `Guid`, and value objects.
- Pass the active transaction to every command that belongs to it.
- Read lists with an explicit limit and a deterministic order. Prefer keyset paging on large
  tables. See `data-tables.md`.

## Types And Storage

| Value                | Storage                                                                 |
| -------------------- | ----------------------------------------------------------------------- |
| Money and amounts    | `INTEGER` in minor units with a documented scale, or `TEXT` with invariant culture; never `REAL` |
| Quantities with fractions | `INTEGER` scaled by a documented factor, or `TEXT`; never `REAL`   |
| Instants             | `TEXT` in ISO 8601 UTC, or `INTEGER` Unix milliseconds; one choice per project |
| Calendar dates       | `TEXT` as `yyyy-MM-dd`                                                  |
| Booleans             | `INTEGER` with `CHECK (column IN (0, 1))`                               |
| Enumerations         | `TEXT` with a `CHECK` listing the allowed values                        |
| Internal keys        | `INTEGER PRIMARY KEY`                                                   |
| Public references    | A separate `TEXT` column with a `UNIQUE` constraint                     |

Declare tables `STRICT` so SQLite rejects values of the wrong type instead of silently
storing them.

## Integrity Constraints

Critical invariants live in the database, not only in C#. Code-level checks race, and every
new code path can forget them.

```sql
-- One document number per series, per company.
UNIQUE (company_id, series, sequential_number)

-- One outcome per idempotency key and operation.
UNIQUE (operation, idempotency_key)

-- Stock and balances never go negative, whatever the code does.
CHECK (quantity_on_hand >= 0)
```

- Use foreign keys for every real relationship, and keep `PRAGMA foreign_keys = ON`.
- When one database holds several companies or workspaces, every owned table carries the
  owner column, and children reference parents through composite foreign keys that include
  it. See `../security/access-control.md`.
- Wrap every multi-step change that must succeed or fail together in one transaction: a sale,
  its lines, its stock movements, its cash movement, and its sequential number.
- Assign sequential numbers inside the same transaction that uses them, from a counter row
  updated with a guarded statement. Never compute the next number with `MAX() + 1` outside
  the transaction.

## Migrations

- Scripts are numbered and named: `Migrations/0001_create_products.sql`. Embed them as
  resources in `Infrastructure`.
- A `schema_migrations` table records version, name, checksum, and applied time.
- At startup, the runner compares checksums, stops with a clear message when an applied
  script changed, and applies pending scripts in order.
- Before applying any pending script, the runner takes a verified backup.
- Each script runs in its own transaction. Run `PRAGMA foreign_key_check` and
  `PRAGMA integrity_check` afterwards, and roll back on failure.
- Never edit an applied migration. Correct it with a new one.
- Destructive changes, such as dropping or rewriting a column, need an explicit data plan:
  copy, verify, then remove, across separate releases when the data matters.
- A new application version refuses to open a database whose schema is newer than it knows.
- Test every migration against an empty database and against a copy of a database at the
  previous version. See `../quality/testing.md`.

## Backups And Restore

- Copy with `SqliteConnection.BackupDatabase`, never by copying the file while it is open.
- Verify every backup by opening it and running `PRAGMA integrity_check`.
- Keep a rotation of local backups, and support an external destination, such as a removable
  drive or a folder the customer synchronizes, encrypted when it holds personal or financial
  data. A backup on the same disk does not survive a disk failure.
- Show the date and status of the last good backup where the operator can see it.
- Practice restore: a backup that was never restored is a hope, not a backup.
- When the data mirrors an external authority, such as issued documents or remote
  transactions, reconcile with that authority after a restore and before creating new
  records, so sequential numbers are not reused and later external results are not lost.

## Data Ownership

| Table | Owner module | Owned by company | Notes |
| --- | --- | --- | --- |
| `{{TABLE}}` | `{{MODULE}}` | Yes/No | Description. |

## Indexing

- List critical queries.
- Map each critical query to an index, and confirm it with `EXPLAIN QUERY PLAN`.
- Index every foreign key column and every business key used for lookup.
- Lead composite indexes with the owner column when queries are scoped by company.

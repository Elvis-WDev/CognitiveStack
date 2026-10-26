# Idempotency And Concurrency

Operators double-click, press Enter twice, and retry after an error. Commands are re-entered
from a shortcut while they still run. The application closes or the power fails in the
middle of an operation. Two windows, two Windows users on a shared database, or a background
job touch the same record at once. Remote services time out after they already succeeded.
An operation that is not idempotent will eventually run twice.

An idempotent operation produces its effect once, however many times the same intent
arrives.

## Which Operations Need It

Any use case whose repetition would duplicate an effect:

- registering sales, payments, refunds, or cash movements;
- issuing invoices or any document with a sequential number;
- moving stock, granting credits, or consuming balance;
- sending emails, printing fiscal documents, or calling remote APIs;
- running pending operations and background jobs;
- creating any record from an action the user might repeat.

The design of each OpenSpec change declares, for each mutation, whether it is idempotent and
by which mechanism.

## Prefer Natural Idempotency

Before adding a key, check whether the operation can be idempotent by construction.

| Mechanism                | Example                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------- |
| Set state, not deltas    | Save the full desired value instead of incrementing it                                                |
| Guarded state transition | `UPDATE sales SET status = 'voided' WHERE id = @Id AND company_id = @CompanyId AND status = 'completed'` |
| Unique natural key       | `UNIQUE (company_id, series, sequential_number)` turns a duplicate into a conflict, not a second row  |
| Conditional arithmetic   | `UPDATE stock SET quantity = quantity - @Qty WHERE product_id = @Id AND company_id = @CompanyId AND quantity >= @Qty` |

Check the affected row count of every guarded update. Zero means the precondition no longer
held, and the operation must not report success.

```csharp
int affected = await connection.ExecuteAsync(VoidSaleSql, args, transaction);
if (affected == 0)
    return Result.Fail(Errors.Sales.NotVoidable); // already voided, missing, or other company
```

Natural mechanisms live in the database, so they survive restarts, concurrency, and every
code path.

## Idempotency Keys

Use a key when the operation has no natural key, such as registering a new sale.

- The ViewModel generates the key with `Guid.NewGuid()` when the user intent starts: when a
  new sale or form begins, not when the command fires.
- Repeated clicks, automatic retries, and retrying the same failed attempt reuse that key.
- A new key is generated only when the user starts a new intent or changes the payload.
- The key travels in the use-case input DTO and is stored with the operation it protects.

### Scope

Store and look up the key by operation, plus company when the database holds several:

```text
company_id + operation + idempotency_key
```

Never use a key to read a record outside the caller's company.

### Where To Store It

Prefer a column on the record the operation creates:

```sql
CREATE TABLE sales (
    id              INTEGER PRIMARY KEY,
    company_id      INTEGER NOT NULL REFERENCES companies (id),
    idempotency_key TEXT    NOT NULL,
    -- ...
    UNIQUE (company_id, idempotency_key)
) STRICT;
```

Use a shared table when the operation creates no single row, or its result must be replayed:

```sql
CREATE TABLE idempotency_records (
    id              INTEGER PRIMARY KEY,
    company_id      INTEGER NOT NULL REFERENCES companies (id),
    operation       TEXT    NOT NULL,
    idempotency_key TEXT    NOT NULL,
    request_hash    TEXT    NOT NULL,
    result_json     TEXT,
    created_at      TEXT    NOT NULL,
    UNIQUE (company_id, operation, idempotency_key)
) STRICT;
```

### Claim Inside An Immediate Transaction

Check-then-act races when it runs outside a write lock. Claim the key in the same immediate
transaction as the effect:

```sql
INSERT INTO idempotency_records
    (company_id, operation, idempotency_key, request_hash, created_at)
VALUES (@CompanyId, @Operation, @Key, @RequestHash, @Now)
ON CONFLICT (company_id, operation, idempotency_key) DO NOTHING
RETURNING id;
```

```csharp
using var tx = connection.BeginTransaction(deferred: false); // BEGIN IMMEDIATE
long? claimed = await connection.QuerySingleOrDefaultAsync<long?>(ClaimSql, args, tx);
if (claimed is null)
    return await ReplayAsync(connection, tx, args); // stored result, or payload mismatch

// ... the effect, with the same transaction ...
await connection.ExecuteAsync(StoreResultSql, new { Id = claimed, ResultJson = json }, tx);
tx.Commit();
```

The immediate transaction serializes writers; the `UNIQUE` constraint keeps the claim correct
even when a code path forgets it. Because the claim, the effect, and the result commit
together, a crash leaves either all of them or none, and a retry after restart runs cleanly.

### Replay Rules

| When the key already exists          | Result                                                        |
| ------------------------------------ | ------------------------------------------------------------- |
| Same payload                         | The stored result, such as the sale and its number; nothing runs again |
| Different payload                    | Error `idempotency.payload_mismatch`; nothing runs. It is a bug |
| Same key, other company              | Invisible: the scoped lookup treats it as a new key            |

Detect a different payload by hashing the validated, canonical input at claim time. Purge
`idempotency_records` after a declared window longer than any retry window, such as 30 days.

### The Interface Helps, The Database Guarantees

`AsyncRelayCommand` disables itself while running and rejects concurrent execution by
default. That prevents most double clicks. Only the database guarantees a single effect.
See `feedback-and-states.md`.

## External Side Effects

A remote call or an email never runs inside the local transaction. Record it as a pending
operation in the same transaction as the business change, and let a hosted service execute
it after commit.

```sql
CREATE TABLE pending_operations (
    id              INTEGER PRIMARY KEY,
    company_id      INTEGER NOT NULL REFERENCES companies (id),
    kind            TEXT    NOT NULL,          -- 'send_document_email', 'submit_invoice'
    dedup_key       TEXT    NOT NULL,          -- sent to the remote service as its key
    payload_json    TEXT    NOT NULL,
    status          TEXT    NOT NULL CHECK (status IN
                        ('pending', 'in_progress', 'succeeded', 'failed', 'needs_review')),
    attempts        INTEGER NOT NULL DEFAULT 0,
    next_attempt_at TEXT    NOT NULL,
    last_error_code TEXT,
    updated_at      TEXT    NOT NULL,
    UNIQUE (company_id, kind, dedup_key)
) STRICT;
```

- A worker claims a row with a guarded update, `SET status = 'in_progress' WHERE id = @Id AND
  status = 'pending'`, and proceeds only when one row changed.
- Send `dedup_key` as the idempotency key to remote services that accept one.
- At startup, a row left `in_progress` is ambiguous: the remote call may have succeeded.
- Retry with bounded exponential backoff and jitter, up to a declared attempt limit. Then set
  `needs_review` and show it to the operator. See `feedback-and-states.md`.
- Mark `failed` immediately on a definitive rejection; retries cannot fix it.

### Ambiguous Timeouts

A timeout or a lost connection does not mean the remote operation failed.

1. Query the remote status of the same operation by its key or document reference.
2. When it exists, record its result locally and finish.
3. When the remote service confirms it does not exist, retry with the same key.
4. When the status cannot be determined, back off and ask again later.

Never create a new remote document, with a new number or key, to compensate for an unknown
outcome. See `integrations.md`.

## SQLite Concurrency

- SQLite allows one writer at a time. Start every write transaction with `BEGIN IMMEDIATE`,
  `BeginTransaction(deferred: false)` in Microsoft.Data.Sqlite, so the lock is taken before
  the first read.
- SQLite has no `SELECT ... FOR UPDATE`. Inside an immediate transaction no other writer can
  change what was read; still prefer guarded updates, which are correct in any transaction.
- `busy_timeout` makes a second writer wait. If it still fails, return a retryable error.
- Keep write transactions short. Never await a network call, a device, or a dialog inside one.
- For edits that span user think time, use optimistic concurrency with a version column:

```sql
UPDATE products
SET name = @Name, price_minor = @PriceMinor, version = version + 1
WHERE id = @Id AND company_id = @CompanyId AND version = @Version;
```

Zero affected rows means someone else saved first: return `concurrency.conflict`, reload,
and let the user decide. Never overwrite silently.

## Verification

- [ ] Every mutation with a meaningful side effect declares how it is idempotent.
- [ ] Natural idempotency was preferred wherever the operation allows it.
- [ ] Guarded updates check the affected row count.
- [ ] Keys are generated when the intent starts, reused on retries, scoped by company and
      operation, and claimed in the same immediate transaction as the effect.
- [ ] Replays return the stored result; payload mismatches are rejected.
- [ ] External side effects run from `pending_operations` after commit, with bounded backoff.
- [ ] Ambiguous outcomes query the remote status before retrying.
- [ ] Edits across user think time use a version column.
- [ ] The idempotency tests in `../quality/testing.md` exist and pass, including a restart
      mid-operation and two genuinely concurrent writers on a real SQLite file.

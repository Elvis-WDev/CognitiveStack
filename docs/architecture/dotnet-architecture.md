# .NET Architecture

How the domain, application, and infrastructure layers are built. `ARCHITECTURE.md` gives
the project map; `wpf-mvvm.md` covers the interface layers.

## Runtime

- Language: C# 14.
- Runtime: .NET 10 LTS.
- Composition: .NET Generic Host with dependency injection, options, configuration, and logging.
- Database: SQLite through Microsoft.Data.Sqlite and Dapper.
- External services: `HttpClient` through `IHttpClientFactory`, MailKit for SMTP.

## Layers

### Domain

Responsibilities:

- Entities with identity and lifecycle.
- Value objects for anything with rules: money, quantity, identification numbers, codes,
  sequential numbers, percentages, and date ranges.
- Business invariants, enforced in constructors and methods, so an invalid object cannot exist.
- Domain errors with stable codes.

Rules:

- No references to WPF, SQLite, Dapper, HTTP, SMTP, logging, or configuration types.
- Money uses `decimal`, with explicit rounding mode and scale at every calculation that rounds.
  Never `double` or `float` for amounts.
- Time comes from an injected `TimeProvider`, never from `DateTime.Now` inside domain logic.
- Regulated rules, such as taxes, record their source, effective date, and version, and each
  one has tests built from documented examples.

### Application

Responsibilities:

- Use cases, one class per user goal, named after the goal: `RegisterSale`, `IssueInvoice`,
  `RetryDelivery`. Never after a control or a screen.
- Ports: repository interfaces, unit of work, clock, file storage, external services,
  notification senders, `ISecretStore`, and `ICurrentSession` for the signed-in user, role,
  permissions, and selected company.
- Input and output DTOs.
- Authorization of every sensitive use case. See `../security/access-control.md`.
- Transaction boundaries.

Rules:

- References only `Domain`.
- No SQL, no Dapper, no `HttpClient`, no file paths, and no WPF types.
- A use case receives its input, validates it, authorizes the caller, runs the domain logic,
  persists inside one transaction, and returns a result.

### Infrastructure

Responsibilities:

- Repositories and the unit of work over SQLite and Dapper. See `database.md`.
- Migration runner and backups.
- Typed HTTP clients, SMTP delivery, file storage, printers, and other devices.
- Secret storage, logging configuration, and update checks.
- Hosted services that resume persisted pending work.

Rules:

- Implements ports from `Application`. Never referenced by `Domain`, `Application`, or
  `Presentation`.
- Maps rows and external payloads into application or domain objects at the boundary.
- Normalizes provider failures into application errors before they leave the layer.

## Results And Errors

- Expected failures, such as validation, business rule violations, conflicts, and missing
  records, return a result type with a stable error code. They are not exceptions.
- Exceptions mean a bug or an unavailable resource. They are logged with context and
  translated into a sanitized failure at the use-case boundary.
- Never catch and ignore an exception. Never show an exception message or stack trace to the
  user. See `feedback-and-states.md`.
- One result shape across the solution, for example `Result<T>` with `Value`, `Error.Code`,
  and `Error.Message`, so ViewModels handle every use case the same way.
- Error codes are stable, lowercase, and dotted by area. Shared codes:

| Code                           | Meaning                                                          |
| ------------------------------ | ---------------------------------------------------------------- |
| `validation.<field>`           | The input is invalid; the ViewModel maps it to the field         |
| `not_found`                    | The record does not exist or lies outside the caller's scope     |
| `forbidden`                    | The caller lacks the permission                                  |
| `concurrency.conflict`         | Someone else changed the record first; reload and retry          |
| `idempotency.payload_mismatch` | A key was reused with different input; a bug, never retried      |
| `integration.<reason>`         | An external service failed; see `integrations.md` for reasons    |

Add module codes, such as `stock.insufficient`, in the module that owns the rule.

## Composition And Configuration

- The Generic Host is built once in the WPF `App` startup, and the main window is resolved
  from the container.
- Register services by lifetime deliberately: stateless services as singletons, use cases and
  repositories as transient or scoped per operation, ViewModels as transient unless they hold
  shell state.
- Bind settings to typed options and validate them at startup with `ValidateOnStart`. The
  application refuses to start with invalid required configuration and tells the user what
  to fix.
- Configuration files never contain secrets in plain text. See `../security/hardening.md`.

## Async And Cancellation

- Every I/O-bound port is asynchronous and accepts a `CancellationToken`.
- Library code in `Application` and `Infrastructure` uses `ConfigureAwait(false)`.
- Never block on async code with `.Result`, `.Wait()`, or `GetAwaiter().GetResult()`.
- Never use `async void` outside event handlers.
- CPU-bound work that would freeze the interface runs on the thread pool, not on the UI thread.

## Background Work

- Job ownership:
- Persistence:
- Retry policy:
- Cancellation:
- Progress reporting:

Work that must survive closing the window or restarting the application, such as delivery
retries, synchronization, or exports, is stored as a pending operation in SQLite before it
starts. A hosted service resumes it at startup. Each pending operation has a deduplication
key and an idempotent handler; see `idempotency.md`.

## Sensitive Operations

A sensitive operation reads or changes private data, money, inventory, stock, sequential
numbers, permissions, files, backups, credentials, or anything another user or company must
not see. Before writing one, answer these in the design of its OpenSpec change:

1. Who may execute this action, and which permission does it require?
2. Which company or workspace does the caller act for, when the database holds several?
3. Which company or workspace does the resource belong to?
4. How is ownership validated in the query itself?
5. What happens if the command runs twice, from a double click, a retry, or a restart?
6. What happens if two windows, two users, or a background job run it at once?
7. Does the operation need idempotency, and by which mechanism?
8. Which database constraints protect its invariants?
9. What happens if the application closes between two of its steps?
10. Does it need an audit record?
11. Which tests must be added?
12. Is there any path for data, secrets, or personal information to leak into logs, files,
    or another user's view?

An unanswered question blocks implementation, as the screen brief does for interfaces.

### Pipeline

Process a sensitive operation in this order. Each step depends on the ones before it.

```text
session resolution
  -> authorization
  -> ownership validation
  -> input validation
  -> idempotency claim, when required
  -> transaction
  -> persistence
  -> audit record
  -> commit
  -> external side effects, through a persisted pending operation
  -> sanitized result
```

The session service resolves the user and role. Authorization, ownership, idempotency, and
the transaction live in the use case and its repositories. External side effects, such as
sending an email or calling a remote API, happen after the local commit and are recorded as
pending operations, so a crash never leaves a side effect without its local record.

### Reporting Format

When implementing a sensitive operation, report in this order:

1. The relevant risks.
2. The authorization boundaries.
3. Whether it needs idempotency, and by which mechanism.
4. The implementation.
5. The validations added.
6. The database constraints added.
7. The tests added.
8. Security risks found in existing code, even outside the task.

Never weaken an authorization control to make a test pass, and never trade data integrity
for simpler code.

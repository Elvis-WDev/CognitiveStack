# Access Control

The PC and its local users are outside our control. Views and ViewModels can be bypassed by a
defect, a new code path, or a background job, so every sensitive rule lives in the use case.
`CanExecute`, hidden buttons, and disabled fields are presentation only.

Knowing an identifier is never authorization. An ID, a document number, or a file name in a
use-case input proves only that the caller has seen it. Every access to private data is
validated against the session and its company.

When a convenient option competes with a slightly more complex secure one, choose the secure
one. Never trade data isolation for simpler code.

## What Application Authorization Protects

Anyone with access to the data folder can open SQLite with any tool, read every table, and
edit any row. Anyone with administrator rights can patch the binaries. State the limits
honestly in each product:

| Threat | Application authorization | Also needs |
| --- | --- | --- |
| Operator exceeds their role through the interface | Stops it | Audit |
| Operator acts for another company in the same database | Stops it | Tests with two companies |
| Mistake or defect in a new screen or job | Stops it, when checks live in use cases | Tests |
| Windows user reads the database file | Nothing | OS folder permissions, encryption at rest |
| Someone edits rows with a SQLite tool | Nothing | Tamper evidence, integrity checks, reconciliation |
| Disk or PC stolen | Nothing | BitLocker or database encryption, encrypted backups |
| Administrator patches the binaries | Nothing | Signed binaries, server-side checks where an authority exists |

- Per-machine data in `%ProgramData%` gets folder permissions from the installer, limited to
  the users who run the application. Per-user data in `%LocalAppData%` is protected by the
  Windows account.
- Encryption at rest, such as BitLocker or an encrypted SQLite build, is an ADR decision. A
  key stored on the same machine protects against copying the file elsewhere, not against
  the signed-in Windows user.

## Four Different Questions

| Question          | Meaning                                       | Answered by                                  |
| ----------------- | --------------------------------------------- | -------------------------------------------- |
| Authentication    | Who is the operator?                          | Local sign-in, through the session service. See `../architecture/authentication.md` |
| Authorization     | What may the operator do?                     | The permission policy, checked in the use case |
| Ownership         | Which records may the operator act on?        | A query scoped by the session context        |
| Company scoping   | Whose data may the operator see at all?       | The company filter on every private query    |

Passing one check never implies another. An administrator of company A has no ownership of
any record in company B.

## Identity Comes From The Session

- Use cases read user, role, permissions, and selected company from `ICurrentSession`.
- Input DTOs never carry `UserId`, `Role`, `CompanyId`, or permissions that decide access.
- Selecting a company is a request for a context. The session service verifies the user's
  membership before switching, and clears data cached for the previous company.

```csharp
// Wrong: the caller decides whose data this is.
public sealed record VoidSaleInput(long SaleId, long CompanyId, string Reason);

// Right: the session decides.
public sealed record VoidSaleInput(long SaleId, string Reason);
// inside the use case
var companyId = session.CompanyId;
```

## Roles And Permissions

When roles exist, such as `operator`, `supervisor`, and `admin`, map them to explicit
permissions in one policy in `Application`, and check the permission in every sensitive use
case.

```csharp
if (!session.Has(Permissions.Sales.Void))
    return Result.Fail(Errors.Forbidden);
```

- Deny by default. A use case without an explicit rule is forbidden.
- Prevent vertical escalation: nobody grants a role above their own, nobody changes their own
  role, and the last administrator cannot be removed or demoted.
- Prevent horizontal escalation: a permission applies inside the selected company only.
- Actions that need a second person use a supervisor override. See
  `../architecture/authentication.md`.
- ViewModels bind `CanExecute` to the same permission so the interface stays honest, but the
  use case is the control.

## Company Scoping

When one database holds several companies, branches, or workspaces, isolation is an
architectural requirement.

- Every owned table carries a `company_id` column; use one name everywhere.
- Every private query filters by it, with the value from the session.
- Repository methods for owned data take the company as a parameter. No `GetById(id)` exists
  for them.

```sql
-- Wrong
SELECT id, number, total_minor FROM sales WHERE id = @Id;

-- Right
SELECT id, number, total_minor FROM sales WHERE id = @Id AND company_id = @CompanyId;
```

- Updates and deletes filter by company too, so a record outside the scope fails as not found.
- Verify every referenced ID in an input. A sale line pointing to another company's product
  is cross-company access through a foreign key.
- Enforce it in the schema with composite foreign keys:

```sql
CREATE TABLE products (
    id         INTEGER PRIMARY KEY,
    company_id INTEGER NOT NULL REFERENCES companies (id),
    UNIQUE (company_id, id)
) STRICT;

CREATE TABLE sale_lines (
    id         INTEGER PRIMARY KEY,
    company_id INTEGER NOT NULL,
    sale_id    INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    FOREIGN KEY (company_id, sale_id)    REFERENCES sales (company_id, id),
    FOREIGN KEY (company_id, product_id) REFERENCES products (company_id, id)
) STRICT;
```

See `../architecture/database.md` for the constraints themselves.

## Explicit Input DTOs

Never map a whole ViewModel or entity into an `UPDATE`.

```csharp
// Wrong: every property is written, including role, company, balance, or status.
await connection.UpdateAsync(user);              // generic helper over the entity

// Right: the use case accepts only the editable fields, and SQL names them.
public sealed record UpdateProfileInput(string FirstName, string LastName);

await connection.ExecuteAsync(
    "UPDATE users SET first_name = @FirstName, last_name = @LastName " +
    "WHERE id = @UserId AND company_id = @CompanyId",
    new { input.FirstName, input.LastName, session.UserId, session.CompanyId }, tx);
```

Role, permissions, `company_id`, owner, balances, lifecycle status, sequential numbers, and
audit timestamps change only through the workflow that owns them.

## Do Not Leak Existence

- A record in another company returns `not_found`, exactly as if it did not exist.
- Inside the company, `forbidden` is acceptable when the operator may know the record exists.
- Keep messages uniform where they could reveal users or records, such as sign-in failures.
- Errors never carry internal keys, SQL, paths, stack traces, or another company's data.

## Exports And Generated Files

- Exports, reports, printed documents, and generated files are authorized and scoped like
  reads, and audited.
- A file saved outside the application data folders leaves our control. Include only what the
  user asked for, and warn when it holds personal data.
- Temporary files are created in the application data folder and deleted when done.

## Tamper Evidence

Prevention ends at the file system, so detect what cannot be prevented:

- An `audit_log` table, written in the same transaction as the action, append-only for
  application code. See `../quality/observability.md`.
- Integrity checks at startup and on demand: `PRAGMA integrity_check`, totals that must
  reconcile, and sequential numbers without gaps.
- Optionally, a hash chain over critical records, such as issued documents and cash
  movements: each row stores `hash = SHA-256(previous_hash || canonical_row)`. A
  verification job reports the first broken link. The chain is stronger when its latest hash
  is also sent to a vendor service or printed on a closing report.
- Reconciliation with an external authority, when one exists, after any restore or
  suspected tampering.

## Review: Patterns To Hunt

- `WHERE id = @Id` on owned tables without `company_id`;
- input DTOs with `UserId`, `CompanyId`, or `Role` used to decide access;
- permission checks only in `CanExecute`, visibility bindings, or menus;
- generic update helpers or SQL built from every property of an entity;
- exports, reports, or file generation without an authorization check;
- background jobs that run use cases without a session or explicit system context.

## Reject The Change When

- a private record is looked up by an identifier alone;
- identity, company, or role is read from input instead of the session;
- a whole ViewModel or entity is written to the database;
- a permission is enforced only in the interface;
- a new private use case has no cross-company or foreign-ID test;
- an authorization check was removed or loosened to make a test pass.

## Verification

- [ ] User, role, and company come only from `ICurrentSession`.
- [ ] Every private query is scoped by company and ownership, including updates and deletes.
- [ ] Every referenced ID in an input is verified against the selected company.
- [ ] Roles map to explicit permissions, denied by default, checked in each use case.
- [ ] Inputs are explicit DTOs with editable fields only.
- [ ] Records in other companies are indistinguishable from missing ones.
- [ ] Exports and generated files are authorized, scoped, and audited.
- [ ] The product states which threats need OS permissions, encryption, or tamper evidence.
- [ ] The access tests in `../quality/testing.md` exist and pass.

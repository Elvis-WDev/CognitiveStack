# Testing Strategy

## Test Layers

| Layer              | Runs on        | Covers                                                                       |
| ------------------ | -------------- | ---------------------------------------------------------------------------- |
| Unit               | Linux, Windows | Values, money and tax rounding, invariants, state machines, time changes     |
| SQLite integration | Linux, Windows | Schema, migrations, transactions, concurrency, backup, restore, indexes      |
| External contracts | Linux, Windows | HTTP fixtures and fakes, XML against its XSD if used, timeouts               |
| Email              | Linux, Windows | MIME structure, recipients, encoding, and attachments, read from Mailpit     |
| ViewModel          | Linux, Windows | Commands, validation, loading, error, empty, and cancellation states         |
| Wine smoke         | Linux          | Startup, main bindings, and navigation to each module, via `scripts/wine.sh` |
| Windows acceptance | Windows 10, 11 | WPF rendering, DPI, printers, installer, updater, and rollback               |

- Unit tests drive time with `FakeTimeProvider`.
- Integration tests use a real database in a temporary file, and run migrations both from an
  empty database and from a copy at the previous version. Concurrency tests open two
  connections to the same file.
- ViewModel tests run without WPF, against fakes of the use cases and of `IDialogService`,
  `INavigationService`, and `IFeedbackService`.

`dotnet test {{ProjectName}}.slnx` runs every layer except WPF UI, which never runs on
Linux. Wine smoke runs catch startup and binding failures; they do not certify
compatibility.

## Writing Tests

- Keep tests deterministic: inject `TimeProvider`, use fakes behind ports, and load fixtures
  from files. No sleeps, no wall-clock time, no machine-specific paths or culture.
- Name tests `Method_Scenario_Result`, for example
  `Register_DuplicateKey_ReturnsStoredResult`.
- Structure each test as Arrange, Act, Assert.
- Cover risks, not a coverage number. A high percentage over trivial code proves nothing.
- Never mock SQL. A repository test runs its real SQL against a real SQLite file.

## What Must Be Tested

- Validation boundaries, including the rejection paths and not only the accepted input.
- Authorization, including access to a record the caller does not own.
- Every result code a use case can return.
- Persistence of critical writes, and rollback of every multi-step write that fails midway.
- Pending operations: resume after restart, retry, and cancellation.
- External service failure modes: timeout, refusal, malformed response, and partial success.
- Grid search, filters, sorting, paging, and row actions in the grid ViewModel.
- Dialog results, cancellation, and repeated command execution in the ViewModel.
- Feedback for successful and failed commands through `IFeedbackService`.
- Permission-driven navigation and command availability.

## Cases That Are Never Skipped

| Case                                                          | Expected                                             |
| ------------------------------------------------------------- | ---------------------------------------------------- |
| The same operation runs twice                                 | One effect; the second run returns the stored result |
| Two simultaneous attempts, from two connections               | Exactly one effect; the other gets a conflict        |
| Crash between the local commit and the external call          | Pending operation resumes; no duplicate              |
| External side effect fails after the main operation succeeded | Local record kept, retried, user informed            |
| Empty database, and the first import into it                  | Usable screens, correct import                       |
| A migration fails midway                                      | Rolled back, backup intact, clear message            |
| Disk full while writing or backing up                         | Clear error; no partial backup reported as good      |
| Invalid or corrupted backup                                   | Rejected before restore; current data untouched      |
| Restore when external records are newer than the backup       | Reconciled first; numbers never reused               |
| Expired certificate or credential                             | Actionable error; no silent retry loop               |
| Several users on the same database                            | Each acts only within their role                     |
| A permission changes during a session                         | The next sensitive command honors it                 |

## Access Control Tests

Every use case that touches private data carries these cases:

| Case                                                             | Expected                                       |
| ---------------------------------------------------------------- | ---------------------------------------------- |
| User A acts on a resource of company A                           | Allowed                                        |
| User of company B acts on a resource of company A                | `not_found`, as if the resource did not exist  |
| A command carries the ID of a record from another company        | `not_found`; nothing is returned or changed    |
| Input references another company's record as a foreign key       | Rejected                                       |
| A user without the required role runs an admin use case          | `forbidden`, and nothing changes               |
| Input carries a role, company, or balance the caller may not set | Ignored by the input DTO, and nothing changes  |

When one database holds several companies or workspaces, fixtures always include at least
two, so a missing scope fails a test instead of passing silently. Single-company products
keep the role and foreign-ID cases.

## Idempotency Tests

| Case                                          | Expected                                                |
| --------------------------------------------- | ------------------------------------------------------- |
| First execution with a key                    | Processed normally                                      |
| Same operation repeated with the same key     | Same stored result, effect not repeated                 |
| Two simultaneous executions with the same key | Exactly one effect                                      |
| Same key in different companies               | Completely independent outcomes                         |
| Same key with a different input               | `idempotency.payload_mismatch`, and nothing executes    |
| Same key while the first is still running     | Waits for the writer, then returns the stored result    |
| Same pending operation resumed twice          | One effect                                              |

Test concurrency with two real connections to the same SQLite file and genuinely parallel
tasks, not sequential calls against a fake. A race the test cannot produce is a race the
test cannot catch.

Never loosen an authorization check or a constraint to make one of these tests pass.

## Test Data And External Environments

- Use deterministic fixtures with synthetic data. Never use real customer data or secrets.
- Keep unstable external services out of the default test run.
- Fake external services behind their ports; record contract fixtures from documentation or
  sanitized sandbox responses.
- Tests that touch a real external test environment are explicit, segregated in their own
  category or project, opt-in through configuration, and never run with real customer data.

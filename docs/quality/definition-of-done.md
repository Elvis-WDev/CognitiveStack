# Definition Of Done

A change is done only when:

- The implementation matches the specs of its OpenSpec change, including the failure and
  exception scenarios.
- Every scenario in those specs is satisfied, or the specs were explicitly updated.
- Restore, build, tests, and format verification pass, and new warnings were reviewed.
- Architecture, data, security, or workflow docs are updated in their canonical source when
  behavior changes.
- New dependencies are justified, versions stay centralized and locked, and upgrades were
  reviewed one package at a time with the lock file diff.
- No secrets, certificates, private keys, or real customer data are committed.
- Performance-sensitive changes carry before and after numbers from representative Windows
  hardware; neutral results were reverted.
- Generated files are produced by scripts or documented sources, not manually edited.
- Work is split into coherent commits, each one verified and reviewable on its own.
- Commit subjects describe what changed, and bodies stay short or absent.
- Stack decisions remain aligned with `docs/architecture/stack.md` or an ADR explains the
  deviation.
- What was not verified, such as pending Windows acceptance, is stated explicitly. Nothing
  untested is announced as available.

For domain, application, and infrastructure work:

- Project references respect `ARCHITECTURE.md`; no dependency points the wrong way.
- `Domain` stays free of framework, database, and I/O types.
- Use cases do not reference infrastructure, and ViewModels do not reference Dapper, SQLite,
  `HttpClient`, or the file system.
- Money uses `decimal` with explicit rounding, and time comes from an injected `TimeProvider`.
- Sensitive operations answer the brief in `docs/architecture/dotnet-architecture.md`.
- Multi-step writes run in one transaction, and critical invariants are database constraints.
- Operations whose repetition would duplicate an effect are idempotent, and ambiguous external
  failures are resolved by querying status, never by creating a new external record.
- Permissions are enforced in use cases, denied by default.
- When one database holds several companies, every private query is scoped by the owner from
  the session, with fixtures that contain at least two companies.
- Schema changes have a migration, tested from an empty database and from the previous
  version, with a verified backup taken before it runs.
- Rules with high risk are covered by tests: money and rounding, sequential numbers,
  transactions, external state transitions, backup, and restore.
- Logs carry no secrets, tokens, full documents, or unnecessary personal data.

For interface work:

- The screen brief is recorded, and information is tiered P0-P3.
- Exactly one action carries primary weight, within the documented action budget.
- Nothing is displayed only because a use case returns it.
- Visual values come from the XAML tokens; no hard-coded spacing, color, font size, radius,
  or duration.
- Interactive controls cover their states, and every animation explains a change.
- Controls are chosen from the user's intent rather than the data type, and add no
  complexity the task does not need.
- Every state looks as deliberate as the happy path: loading, empty, partial, error,
  permission, and success.
- No disk, database, network, or heavy work runs on the UI thread; long commands show
  progress and can be cancelled when cancellation is safe.
- Forms validate on input and the use case validates again.
- Large lists are paged in the query and virtualized in the grid.
- Text and controls fit at the minimum window size and at 100 to 200 percent display scaling.
- Keyboard-only operation works, with visible focus, logical tab order, and shortcuts for
  frequent actions; touch targets meet the documented size when touch is supported.
- No control ships without use-case support, and no screen ships on mocked data.
- Shared patterns are used for grids, paging, filters, forms, dialogs, notifications, date
  inputs, and navigation.
- Every command that changes data reports pending, success, and error through the shared
  feedback service.
- Destructive actions follow the confirmation ladder in
  `docs/architecture/forms-and-workflows.md`.
- No database keys, file paths, provider internals, exception messages, or secret references
  are exposed without a documented user need.
- Binding errors are absent from the debug output.
- Light, dark, and high-contrast themes, display scales, and window sizes pass
  `docs/quality/ui-checklist.md`.
- A Wine smoke run passed during development, and the change is on the Windows acceptance list
  before release.

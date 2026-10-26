# Core Instructions

Apply to C# files in `Domain`, `Application`, and `Infrastructure`.

- Keep `Domain` free of WPF, SQLite, Dapper, HTTP, SMTP, logging, and configuration types.
- Define application ports before adding infrastructure adapters.
- Validate all external input at the use-case boundary, including imported files and responses.
- Keep SQL, Dapper, and `HttpClient` inside `Infrastructure`, with parameterized SQL only.
- Use `decimal` for money and inject `TimeProvider` instead of reading `DateTime.Now`.
- Accept a `CancellationToken` in every I/O method; never block with `.Result` or `.Wait()`.
- Enforce authorization in the use case per record, not only by hiding commands.
- Resolve user, role, and company from the session service, never from command input.
- Scope every query on private data by company when the database holds several.
- Map input through explicit DTOs; never update every column from a caller-supplied object.
- Wrap multi-step writes in one transaction, make critical writes idempotent, and enforce
  invariants with database constraints.
- Add schema changes as new numbered migration scripts; never edit an applied one.
- Bound every list query and never load a relation inside a loop.
- Keep secrets out of code, `appsettings.json`, and logs.
- Measure before optimizing, and revert an optimization that does not beat its baseline.

# Observability

## Required Signals

- Application start, version, and shutdown, including an unclean previous shutdown.
- Migration and backup results.
- Use-case failures, with their result code.
- Pending operation status: queued, retried, failed, and completed.
- External service failures and timeouts.
- Sign-in failures, when the product has local users.
- Import and export progress when applicable.

## Logging

- Log through `ILogger<T>`, with Serilog writing a rolling file in the logs folder that the
  application-paths service resolves, such as
  `%LocalAppData%\{{Company}}\{{ProjectName}}\logs\`.
- Write structured messages with named properties, not interpolated strings.
- Enrich every entry with the application version, machine name, and a correlation or
  operation ID that follows one user action across use cases, pending operations, and
  external calls.
- Cap file size and retention, so logs never fill the disk.
- Never log secrets, passwords, tokens, connection strings, personal data, complete
  documents, or full external payloads. Log identifiers and safe summaries instead.

## Error Records

Every unexpected failure logs one entry that an operator or agent can act on:

| Field          | Content                                                                      |
| -------------- | ---------------------------------------------------------------------------- |
| Type           | Exception type and stable error code                                         |
| Layer          | Domain, Application, Infrastructure, Presentation, or Desktop                |
| Correlation    | Operation ID of the user action or pending operation                         |
| Document state | Safe state of the affected record, such as its status and business reference |
| Detail         | Message and stack trace, in the log only, never on screen                    |

The user sees a sanitized message and, when useful, the operation ID to quote to support.
See `../architecture/feedback-and-states.md`.

## Support Bundle

Provide an export, behind a permission when the product has roles, that packages recent
logs, the application and schema version, the operating system version, and the last
backup status into one file. Redact it before writing, and let the user see its contents.
Never include the database itself unless the user explicitly chooses to.

## Audit Trail

Record an audit entry for critical actions: permission and role changes, money and stock
movements, deletions, exports, backups and restores, credential and integration changes,
and access to sensitive records.

```text
user_id  company_id  operation  resource_type  resource_id
occurred_at  operation_id  machine  result
```

- Write the entry in the same SQLite transaction as the action, so neither exists without
  the other.
- Record denied attempts as well as successful ones.
- Audit entries are append-only for application code: no update or delete path exists.
- Omit `company_id` when the database holds a single company.
- Never record passwords, tokens, secret keys, complete documents, or sensitive values the
  investigation does not need.
- Expose the trail only through permission-gated audit views, as
  `../architecture/wpf-mvvm.md` requires for diagnostic tools.

## Diagnosing A Defect

Report every defect with:

1. A reproducible hypothesis of the cause.
2. The environment: application version, Windows version or Wine version, display scale, and
   database schema version.
3. The steps to reproduce.
4. The expected and the observed behavior.
5. The evidence: log entries with their operation ID, screenshots, or a support bundle.

Separate real WPF defects from Wine-specific incompatibilities. A failure seen only in Wine
is confirmed on real Windows before it is fixed in the product; a Wine-only issue is noted,
not worked around in application code.

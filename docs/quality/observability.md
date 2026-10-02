# Observability

## Required Signals

- Application health.
- Request latency and errors.
- Background job status.
- External service failures.
- Authentication failures.
- Import/export progress when applicable.

## Audit Trail

Record an audit entry for critical actions: permission and role changes, financial
movements, deletions, exports, shares, API key and integration changes, and access to
sensitive records.

```text
user_id  tenant_id  operation  resource_type  resource_id  timestamp  request_id  ip  result
```

- Write the entry in the same transaction as the action when the action is a database write, so neither exists without the other.
- Record denied attempts as well as successful ones; a burst of `404` and `403` across many IDs is an enumeration signal.
- Audit entries are append-only for application code.
- Never record passwords, full tokens, secret keys, signed URLs, complete financial data, or sensitive values the investigation does not need.
- Expose the trail only through permission-gated audit views, as `../architecture/frontend.md` requires.

## Logging Rules

- Do not log secrets.
- Do not log full sensitive payloads.
- Include stable identifiers for debugging.
- Make error messages actionable for operators and agents.


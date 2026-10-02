# Threat Model

## Assets

- User data.
- Auth sessions.
- Database credentials.
- External provider tokens.
- Generated files and exports.
- The isolation between tenants.
- Balances, credits, inventory, and other counted values.

## Trust Boundaries

- Browser to API.
- API to database.
- API to external provider.
- External provider webhooks to API.
- Tenant to tenant, inside the same database.
- Background job runner to database.
- CI/deployment to production environment.

## Threats

| Threat | Impact | Mitigation |
| --- | --- | --- |
| Token leakage | Provider abuse | Keep tokens server-side and out of logs. |
| Unauthorized data access | Privacy breach | Enforce auth middleware and permission checks. |
| IDOR / BOLA: editing an ID in the request | Another account's record exposed or changed | Scope every private query by tenant and ownership. See `access-control.md`. |
| Tenant hopping: sending another tenant's ID | Cross-account breach | Resolve the tenant from the session only; composite foreign keys; cross-tenant tests. |
| Role manipulation | Vertical privilege escalation | Server-side role policy; roles never client-writable; no self-promotion. |
| Mass assignment | Tampered role, tenant, or balance | Strict allowlisted schemas for every write. |
| Resource enumeration | Discovery of other accounts' records | `404` outside the tenant; non-sequential public identifiers. |
| Leaked signed URL | File exposure | Short expiry, single-object scope, ownership checked before signing, never logged. |
| Duplicate request: retry, double submit, replay | Double charge or duplicate record | Natural idempotency or scoped idempotency keys. See `../architecture/idempotency.md`. |
| Concurrent writes | Negative balance, oversold stock | Database constraints, guarded updates, transactions, and locks. |
| Webhook replay or forgery | Repeated or forged effects | Signature on the raw body, timestamp tolerance, event ID deduplication. |
| Unvalidated bulk input | Data corruption | Validate, chunk, and report progress. |
| Hand-rolled authentication | Account compromise | Use Better Auth and documented auth middleware. |
| Overprivileged database user | Production data loss | Reduce privileges after migrations. |
| External outage | Workflow failure | Normalize errors and expose retry-safe states. |

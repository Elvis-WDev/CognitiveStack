# Integrations

## External Services

| Service | Purpose | Auth | Owner |
| --- | --- | --- | --- |
| `{{SERVICE}}` | Purpose. | Method. | Module. |

## Integration Rules

- Tokens stay server-side.
- Use typed payloads at boundaries.
- Normalize provider errors before exposing them internally.
- Avoid provider names in user-facing UI unless necessary.
- Document rate limits and usage gates.
- Use Axios for server-side external API calls.
- Do not call external provider APIs directly from the browser when tokens are required.
- Add runtime timeout configuration.
- Add backoff, queueing, or persisted jobs for bulk provider calls.
- Send an idempotency key with outbound mutations to providers that accept one, derived from the local operation record. A timeout is ambiguous, and the retry must not duplicate the effect. See `idempotency.md`.

## Provider Template

### `{{PROVIDER_NAME}}`

- Base URL:
- Docs:
- Auth:
- Timeout:
- Rate limit:
- Usage endpoint:
- Internal route prefix:
- Error behavior:
- Idempotency support:
- Webhook signature scheme and event ID field:

## Inbound Webhooks

Treat every webhook as untrusted and potentially repeated.

1. Mount the webhook route with `express.raw()` before the JSON middleware, because the signature covers the exact bytes received.
2. Verify the provider signature with a constant-time comparison, and reject on mismatch.
3. Reject events whose timestamp falls outside the provider's tolerance, typically a few minutes, so a captured request cannot be replayed later.
4. Record the provider event ID under a unique constraint. When it already exists, answer success without repeating any effect.
5. Acknowledge quickly with a `2xx`, and do the work in a persisted job.
6. Process idempotently and do not assume ordering. Check the current state before applying the event.

```text
UNIQUE (provider, provider_event_id)
```

## Usage / Quota Gates

When a provider exposes usage or quota endpoints:

- Query usage before starting bulk jobs.
- Block jobs when quota is exhausted.
- Respect provider requests-per-minute limits.
- Surface user-friendly availability state in the UI.
- Keep raw provider tokens and provider internals out of user-facing text unless needed.

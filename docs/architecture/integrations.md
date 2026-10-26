# Integrations

Every external service, device, and file format sits behind a port defined in `Application`
and implemented in `Infrastructure`. Use cases never see `HttpClient`, MailKit, serial ports,
or file formats.

## External Services

| Service | Purpose | Auth | Owner |
| --- | --- | --- | --- |
| `{{SERVICE}}` | Purpose. | Method. | Module. |

## Integration Rules

- Credentials live in the machine secret store, never in shipped configuration or code. See
  `../security/hardening.md`.
- Map external payloads to typed contracts at the boundary, and validate them like any other
  untrusted input.
- Normalize provider failures into application errors before they leave `Infrastructure`.
- Keep provider names and raw errors out of the interface unless the operator needs them.
- Document rate limits, quotas, and timeouts for each provider.
- Calls never run on the UI thread, and every call accepts a `CancellationToken`.
- Outbound mutations run from a pending operation after the local commit, carrying its key.
  See `idempotency.md`.

## HTTP Clients

Register one typed client per provider through `IHttpClientFactory`, with an explicit
resilience pipeline:

```csharp
services.AddHttpClient<IBillingGateway, BillingGatewayClient>(client =>
    {
        client.BaseAddress = options.BaseAddress;
        client.Timeout = Timeout.InfiniteTimeSpan; // the pipeline owns timeouts
    })
    .AddStandardResilienceHandler(o =>
    {
        o.AttemptTimeout.Timeout = TimeSpan.FromSeconds(10);
        o.TotalRequestTimeout.Timeout = TimeSpan.FromSeconds(30);
        o.Retry.MaxRetryAttempts = 2;
        o.Retry.DisableForUnsafeHttpMethods();
    });
```

- Automatic retries apply to safe reads only. A mutation is retried by its pending operation,
  with the same idempotency key, after checking the remote status.
- Use System.Text.Json with an explicit naming policy, and source-generated contexts for
  contracts on hot paths. Amounts travel as decimal numbers or strings, never as `double`.
- Tolerate unknown fields in responses; reject missing required ones.
- Log the provider, operation, duration, status, and correlation identifier, never tokens or
  full payloads with personal data.

### Normalized Errors

| Remote condition                         | Application error           | Retry           |
| ---------------------------------------- | --------------------------- | --------------- |
| Timeout or connection lost after sending | `integration.ambiguous`     | After a status query |
| Connection refused, DNS failure, `5xx`   | `integration.unavailable`   | With backoff    |
| `429` or quota exhausted                 | `integration.rate_limited`  | After the hinted delay |
| `401` or `403`                           | `integration.unauthorized`  | No; credentials need attention |
| `400`, `409`, `422`, business rejection  | `integration.rejected`      | No; store the reason |
| Unparseable or unexpected response       | `integration.contract_mismatch` | No; a defect or a version gap |

## Contracts With Our Own Backend

When the application talks to services the vendor operates, such as licensing, updates,
synchronization, or document relays, old clients stay in the field for months.

- Version the API contract explicitly, in the path or a header, and keep the contract types
  in one place shared by both sides or generated from one definition.
- The server supports every client version the update policy still allows. Removing or
  renaming a field is a breaking change and needs a new version.
- Add compatibility tests on both sides: the client against recorded responses of each
  supported server version, and the server against requests of each supported client.
- The client sends its version, so the server can answer with a clear "update required".

## Email

- MailKit sends through SMTP. Use `SecureSocketOptions.StartTls` or `SslOnConnect` in
  production; plain connections are for the development sink only.
- In development and tests, Mailpit runs in Docker as the sink, on ports 1025 for SMTP and
  8025 for its web interface, unless the project documents others.
- Sending is a separate, retryable pending operation, created in the same transaction as the
  business operation it belongs to. The sale or invoice succeeds whether or not the email
  does, and the interface shows the delivery state with a retry action.
- SMTP has no idempotency. Set a stable `Message-Id` derived from the operation key so a
  duplicate after an ambiguous timeout can be recognized. Accept that risk only for email.

## Devices

Printers, barcode scanners, cash drawers, scales, and card terminals are ports, such as
`IReceiptPrinter` or `ICashDrawer`, with an implementation per device family.

- Device settings are per machine, chosen in a settings screen, and validated with a test
  action.
- Every device call has a timeout and returns a normalized result: ready, busy, offline, out
  of paper, or failed.
- A device failure never rolls back a committed business operation. Offer reprint or retry.
- Treat scanner and keyboard-wedge input as untrusted text: validate format and length.
- Provide fake implementations for tests, Wine smoke runs, and machines without the device.

## File Import And Export

- Import in stages: parse, validate everything, preview, then apply in one transaction or in
  resumable chunks with progress. See `forms-and-workflows.md`.
- Enforce size and row limits, detect encoding, and report errors by row and column.
- Write exports to a temporary file and move it into place when complete.
- Escape spreadsheet cells that start with `=`, `+`, `-`, or `@` to prevent formula injection.
- Authorize exports like reads, scoped by company. See `../security/access-control.md`.

## Usage And Quota Gates

When a provider exposes usage or quota:

- Query usage before starting bulk work, and block it when the quota is exhausted.
- Respect the provider's request-rate limits in the pending-operation worker.
- Show availability in business terms in the interface.

A desktop application does not receive inbound calls. When a provider pushes events, they
reach a vendor backend, and the application polls it or the provider for status.

## Provider Template

### `{{PROVIDER_NAME}}`

- Base URL and environments (sandbox, production):
- Docs:
- Auth and where the credential is stored:
- Contract version:
- Attempt and total timeouts:
- Retry policy for reads:
- Rate limit and quota endpoint:
- Idempotency support and key field:
- Status query for ambiguous outcomes:
- Error mapping:

# Testing Strategy

## Test Pyramid

- Unit tests for pure domain logic.
- Use-case tests for application behavior.
- Integration tests for database, HTTP, auth, and external boundaries.
- End-to-end tests for critical user workflows.

## What Must Be Tested

- Validation boundaries, including the rejection paths and not only the accepted input.
- Auth and permissions, including access to a record the caller does not own.
- Error responses.
- Data persistence.
- Background jobs and cancellation.
- External service failure modes.
- Better Auth protected-route behavior.
- Prisma repository behavior for critical writes.
- PostgreSQL migration assumptions.
- React Hook Form validation paths.
- Table search, filters, sorting, pagination, column visibility, and row actions.
- Modal focus management, dismissal, scroll containment, and submission states.
- Toast semantics and retry behavior for successful and failed actions.
- Responsive layout, long text, empty collections, large collections, and dark mode.
- Permission-driven navigation and action visibility.

## Access Control Tests

Every endpoint that touches private data carries these cases:

| Case                                                          | Expected                                            |
| ------------------------------------------------------------- | --------------------------------------------------- |
| User A acts on resource A                                     | Allowed                                             |
| User B acts on resource A                                     | `404`, as if the resource did not exist             |
| User A edits the ID in the path to reach resource B           | `404`, and nothing is returned or changed           |
| User A sends another tenant's ID in the body or query         | Ignored or rejected; the session tenant still holds |
| User A references another tenant's record as a foreign key    | Rejected                                            |
| A user without the required role calls an admin operation     | `403`, and nothing changes                          |
| The payload carries `role`, `tenantId`, `isAdmin`, or `balance` | Rejected by the strict schema, and nothing changes |

Cover IDOR, BOLA, tenant hopping, role manipulation, mass assignment, and parameter
tampering explicitly. Fixtures always include at least two tenants, so a missing filter
fails a test instead of passing silently.

## Idempotency Tests

| Case                                       | Expected                                  |
| ------------------------------------------ | ----------------------------------------- |
| First request with a key                   | Processed normally                        |
| Same request repeated with the same key    | Same outcome, effect not repeated         |
| Two simultaneous requests with the same key | Exactly one effect                       |
| Same key in different tenants              | Completely independent outcomes           |
| Same key with a different payload          | `422`, and nothing executes               |
| Same key while the first is still running  | `409`, and nothing runs concurrently      |
| Same webhook event delivered twice         | One effect, both deliveries acknowledged  |
| Same job executed twice                    | One effect                                |

Test concurrency with genuinely parallel requests against PostgreSQL, not sequential calls
against a mock. A race the test cannot produce is a race the test cannot catch.

Never loosen an authorization check or a constraint to make one of these tests pass.

## Test Data

- Use deterministic fixtures.
- Avoid real secrets.
- Avoid relying on unstable external services in CI.
- Mock external APIs behind typed client ports.

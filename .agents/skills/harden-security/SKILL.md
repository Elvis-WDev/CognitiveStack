---
name: harden-security
description: Use when adding or changing a boundary that handles untrusted input, local users and roles, authorization, company data, secrets, imported files, backups, external services, or side effects that must not repeat. Also use when reviewing a change for security defects.
---

# Harden Security

## Context To Load

Read:

- `docs/security/hardening.md`
- `docs/security/access-control.md`
- `docs/architecture/idempotency.md`
- The sensitive operation section of `docs/architecture/dotnet-architecture.md`.
- `docs/security/principles.md`
- `docs/security/threat-model.md`
- `docs/architecture/authentication.md` for auth work.
- The OpenSpec change being implemented, for the permissions the product requires.

## Workflow

1. Answer the sensitive operation brief in `docs/architecture/dotnet-architecture.md` before writing code.
2. List the boundaries the change touches: interface input, configuration, company, database, session, files, backups, external services, pending operations.
3. For each boundary, name what is untrusted and where it is validated.
4. Confirm user, role, and company come from the session service, and every private query, including updates, deletes, and referenced IDs, is scoped by company and ownership when the database holds several.
5. Confirm writes accept only the fields of an explicit input DTO.
6. Decide how each side effect is idempotent, and confirm critical invariants are database constraints.
7. Confirm secrets are stored behind the secret-store port, protected with DPAPI or Windows Credential Manager, and stay out of code, `appsettings.json`, logs, support bundles, and the screen.
8. Confirm errors reaching the user are sanitized, records outside the caller's scope return `not_found`, and diagnostics stay in logs.
9. Add the access control and idempotency tests from `docs/quality/testing.md`, covering rejection paths and not only accepted ones.
10. Report in the format of `docs/architecture/dotnet-architecture.md`, including residual risk and risks found in existing code.

## Guardrails

- Do not trust a disabled button or a hidden command as authorization; the use case checks.
- Do not look up a private record by an identifier alone when the database holds several companies.
- Do not read user, role, or company from command input.
- Do not map input straight onto an entity or an `UPDATE` of every column.
- Do not ship a critical write that can run twice, or an idempotency check that reads before it claims.
- Do not loosen an authorization check or a constraint to make a test pass.
- Do not build SQL, paths, or process arguments by concatenating input.
- Do not trust imported files, backups, or external responses before validating them.
- Do not show provider payloads, stack traces, file paths, or secret references.
- Do not add a NuGet package without verifying its name, maintenance, vulnerabilities, and license.
- Do not implement custom cryptography or password hashing when the documented approach covers the need.
- Do not silence a security finding by narrowing its test.

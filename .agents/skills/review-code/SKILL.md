---
name: review-code
description: Use when reviewing changes for bugs, security risks, architecture drift, missing tests, and documentation gaps.
---

# Review Code

## Context To Load

Read:

- `AGENTS.md`
- `docs/quality/code-review.md`
- `docs/security/hardening.md`
- `docs/security/access-control.md`
- `docs/architecture/idempotency.md`
- `docs/quality/performance.md`
- `docs/quality/ui-checklist.md` for interface changes.
- Relevant architecture docs.
- The OpenSpec change under review: its proposal, specs, design, and tasks.
- `docs/architecture/stack.md`

## Review Stance

Lead with findings. Prioritize:

1. Bugs and data loss.
2. Security and auth issues.
3. Behavioral regressions.
4. Missing tests or verification.
5. Architecture drift.

Stack-specific checks:

- The dependency direction in `ARCHITECTURE.md` holds: no WPF types in ViewModels or `Application`, no Dapper, SQL, or `HttpClient` outside `Infrastructure`.
- User, role, and company come from the session service, and private queries are scoped by company and ownership when the database holds several.
- Critical writes are idempotent, pending operations are deduplicated, and money, stock, and sequential-number changes run in one transaction.
- SQL is parameterized Dapper; schema changes are new numbered migration scripts, never edits to applied ones.
- Money uses `decimal`, and time comes from an injected `TimeProvider`.
- Async code accepts a `CancellationToken`, never blocks with `.Result` or `.Wait()`, and uses `async void` only in event handlers.
- External HTTP calls use typed clients from `IHttpClientFactory` with resilience and explicit timeouts.
- Packages are versioned in `Directory.Packages.props`, with `packages.lock.json` updated and NuGet Audit clean.
- Commands report through `IFeedbackService`, and dialogs and navigation go through `IDialogService` and `INavigationService`.

## Output Format

- Findings first, ordered by severity.
- Label each finding Critical, Required, Consider, Nit, or FYI, so the author knows what blocks merge.
- Name the structural remedy when the finding is structural; do not report complexity without a move.
- Include file and line references.
- Add open questions after findings.
- Keep summaries secondary.

If no issues are found, say so clearly and mention residual risk.

# Project Agent Guide

## Project

`{{PROJECT_NAME}}` is a Windows desktop application built with:

- C# 14 on .NET 10 LTS.
- WPF and XAML for the interface.
- MVVM with CommunityToolkit.Mvvm.
- The .NET Generic Host for dependency injection, options, configuration, and logging.
- SQLite through Microsoft.Data.Sqlite, with Dapper for data access.
- Versioned SQL migrations applied by a startup runner.
- `HttpClient` through `IHttpClientFactory` for external HTTP services, and MailKit for SMTP.
- xUnit for tests, and Mailpit as the development SMTP sink.
- NuGet with Central Package Management and lock files.

## Read Before Working

- Product overview: `docs/product/overview.md`
- Architecture map: `ARCHITECTURE.md`
- Architecture details: `docs/architecture/index.md`
- Stack baseline: `docs/architecture/stack.md`
- Layers and sensitive operations: `docs/architecture/dotnet-architecture.md`
- WPF and MVVM: `docs/architecture/wpf-mvvm.md`
- Database: `docs/architecture/database.md`
- Authentication: `docs/architecture/authentication.md`
- Cognitive architecture: `docs/architecture/cognitive-architecture.md`
- Design foundations: `docs/architecture/design-foundations.md`
- Component selection: `docs/architecture/component-selection.md`
- Interface design: `docs/architecture/interface-design.md`
- Current behavior: `openspec/specs/`
- Changes in progress: `openspec/changes/`
- Change lifecycle: `docs/quality/change-lifecycle.md`
- Definition of done: `docs/quality/definition-of-done.md`
- Testing: `docs/quality/testing.md`
- Version control: `docs/quality/version-control.md`
- UI review checklist: `docs/quality/ui-checklist.md`
- Security principles: `docs/security/principles.md`
- Access control: `docs/security/access-control.md`
- Idempotency and concurrency: `docs/architecture/idempotency.md`
- Security hardening: `docs/security/hardening.md`
- Performance: `docs/quality/performance.md`
- Distribution and updates: `docs/architecture/distribution.md`

## Workflow

1. Read this file and the nearest `AGENTS.md` for the files you will touch.
2. Open only the relevant docs for the current task.
3. Search the existing implementation and tests before adding anything. Do not invent
   interfaces or duplicate modules.
4. For non-trivial behavior changes, work through an OpenSpec change: explore, propose,
   review, apply, verify, and archive. See `docs/quality/change-lifecycle.md`. For a purely
   technical change, state why it needs no OpenSpec change; record architectural decisions
   as ADRs.
5. Before changing migrations, money, sequential numbers, credentials, licenses, updates, or
   backups, read the related security and database documents and plan the rollback.
6. Implement the smallest coherent change, with tests that match its risk.
7. Update documentation when behavior, architecture, data, or decisions change.
8. Run the project's verification commands before calling work complete.
9. Commit each coherent unit separately, with a short descriptive message, when asked to commit.

## Required Verification

Replace the placeholders when the project exposes different names:

- Restore: `dotnet restore {{ProjectName}}.slnx --locked-mode`
- Build: `dotnet build {{ProjectName}}.slnx -warnaserror`
- Test: `dotnet test {{ProjectName}}.slnx`
- Format: `dotnet format {{ProjectName}}.slnx --verify-no-changes`
- WPF smoke run under Wine, during development on Linux: `scripts/wine.sh`
- Before a release: the Windows acceptance checklist in `docs/quality/testing.md`

`dotnet test` on Linux does not exercise the WPF interface, and running under Wine does not
certify Windows compatibility. Report what was not verified instead of implying it was.

## Constraints

- Do not put product or architecture knowledge in `.codex/`.
- Keep `Domain` free of WPF, SQLite, Dapper, HTTP, SMTP, logging, and configuration types.
- Keep SQL, Dapper, and SQLite connections inside `Infrastructure`.
- Keep WPF types out of ViewModels, `Application`, and `Domain`. Code-behind holds purely
  visual behavior only.
- Never block the UI thread with disk, database, network, or heavy work, and never block on
  async code with `.Result` or `.Wait()`.
- Use `decimal` for money and amounts, with explicit rounding. Never `double` or `float`.
- Inject `TimeProvider` instead of reading the system clock in domain and application code.
- Pass every value to SQL as a parameter; never concatenate input into SQL, paths, or commands.
- Use versioned migrations for every schema change, take a verified backup before applying
  them, and never edit an applied migration.
- Store databases, backups, and logs in application data folders, never in the installation
  folder.
- Do not share one SQLite file between machines through a network folder.
- Wrap every multi-step write that must succeed or fail together in one transaction, and
  enforce critical invariants with database constraints.
- Make operations whose repetition would duplicate an effect idempotent. After an ambiguous
  external failure, query the status of the same operation before retrying; never create a
  new external record to compensate.
- Enforce permissions in use cases; a disabled button or hidden menu is presentation only.
- When one database holds several companies or workspaces, scope every private query by the
  owner resolved from the session; knowing an identifier is not authorization.
- Do not hand-roll password hashing or cryptography; use the documented platform primitives.
- Never store secrets, certificates, private keys, or real customer data in the repository,
  in configuration files in plain text, or in logs. Never ship a private signing key inside
  the application.
- Verify the signature of every update and license before trusting it.
- Do not introduce production dependencies without documenting why, and keep NuGet versions
  centralized and locked.
- Do not modify generated files manually, including the OpenSpec skills, commands, and
  prompts; regenerate them with `openspec update`.
- Describe current behavior only in `openspec/specs/`, and change it only through an OpenSpec
  change.
- Measure before optimizing, on representative Windows hardware, and revert any optimization
  that does not beat its baseline.
- Do not mark work complete while verification is failing.
- Commit one coherent unit at a time; never batch a whole working session into one commit.
- Write commit subjects as `type(scope): imperative subject` under 72 characters, and keep
  bodies to three short lines at most.
- Commit and push only when asked, and never rewrite published history without an explicit
  request.
- Answer the screen brief before implementing an interface: user, goal, primary decision,
  minimum information, and single primary action.
- Do not design a screen from the database schema, and do not display a value only because a
  use case returns it.
- Keep one primary action per screen, and take spacing, type, color, radius, and motion values
  from the documented XAML tokens.
- Choose each UI control from the user's intent, not from the data type, and never add a
  control that does not improve the experience.
- Do not expose internal IDs, database keys, file paths, provider names, exception messages,
  or other technical references in product UI when a business-facing label is possible.
- Reuse the documented grid, form, dialog, feedback, navigation, and window patterns instead
  of inventing a different interaction model per module.
- Optimize operational screens for scanning, keyboard use, and repeated work. Remove redundant
  summaries, duplicated headings, explanatory boxes, and nested cards.
- Separate a module's areas as navigation submodules, not as tabs inside one screen.
- Route every command that changes data through the shared feedback service; no action ends
  without pending, success, or error feedback.
- Confirm irreversible actions by typing the record name, and keep reversible ones to a single
  step.

## Agent Report

End every task with: a summary of the change, the files affected, the tests run and their
result, the context documents updated, open risks and assumptions, and the manual steps still
pending on Windows or with external services. Never present untested work as complete.

## Definition Of Done

Implementation, verification, documentation, and the OpenSpec specs must agree.

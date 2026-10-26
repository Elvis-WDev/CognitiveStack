# Architecture Map

## System Style

Default architecture:

- A modular monolith that runs locally on one Windows machine. No microservices inside the PC.
- A WPF host using MVVM, composed through the .NET Generic Host and dependency injection.
- SQLite as the local database, accessed with Dapper from the infrastructure layer only.
- External services, such as HTTP APIs, SMTP, printers, and update servers, isolated behind
  interfaces defined by the application layer.
- Long-running or retryable work persisted, so it survives a restart of the application.

## Projects

```text
{{ProjectName}}.Desktop          WPF host: views, XAML resources, composition root
        |              \
        v               v
{{ProjectName}}.Presentation    {{ProjectName}}.Infrastructure
(ViewModels, no WPF types)      (SQLite and Dapper, HTTP, SMTP, files,
        |                        devices, logging, updates)
        v                               |
{{ProjectName}}.Application  <----------+
(use cases, ports, DTOs)
        |
        v
{{ProjectName}}.Domain
(entities, value objects, invariants, domain errors)

{{ProjectName}}.Tests.*         xUnit projects for each layer
```

| Project          | Target            | Responsibility                                                  |
| ---------------- | ----------------- | --------------------------------------------------------------- |
| `Domain`         | `net10.0`         | Entities, value objects, invariants, domain errors              |
| `Application`    | `net10.0`         | Use cases, ports, DTOs, transaction boundaries, authorization   |
| `Infrastructure` | `net10.0`         | Implementations of ports: SQLite, HTTP, SMTP, files, devices    |
| `Presentation`   | `net10.0`         | ViewModels, navigation and dialog abstractions, no WPF types    |
| `Desktop`        | `net10.0-windows` | Views, XAML resources, platform services, composition root      |
| `Tests.*`        | `net10.0`         | Tests that run on Linux and Windows                             |
| `Tests.Desktop`  | `net10.0-windows` | Optional tests that need WPF, run on Windows only               |

`Presentation` is optional. Add it when ViewModels must be tested on Linux; until then,
ViewModels may live in `Desktop`. Do not create any other project until it removes a real
friction.

## Dependency Direction

```text
Desktop -> Presentation -> Application -> Domain
Desktop -> Infrastructure -> Application -> Domain
```

Rules:

- `Domain` references no other product project and no framework: no WPF, SQLite, Dapper,
  HTTP, SMTP, or logging types.
- `Application` references only `Domain`, and defines the ports it needs: repositories, unit
  of work, clock, external services, file storage, and notifications.
- `Infrastructure` implements those ports. It never references `Presentation` or `Desktop`.
- `Presentation` references `Application`, never `Infrastructure`, and never WPF assemblies.
- `Desktop` composes everything with dependency injection and is the only project that
  knows every implementation.
- No circular references, and no infrastructure type in a domain or application signature.

## Boundaries

- **Interface boundary:** views bind to ViewModels; ViewModels call use cases. A view or
  ViewModel never runs SQL, opens files, or calls an external service directly.
- **Identity boundary:** the signed-in user and role, when the product has them, are resolved
  by the application session service and passed to use cases as plain data.
- **Company boundary:** when one database holds several companies, branches, or workspaces,
  every private query is scoped by that owner. See `docs/security/access-control.md`.
- **Database boundary:** SQLite connections, SQL, and Dapper exist only in `Infrastructure`.
- **External-service boundary:** HTTP, SMTP, printers, and update servers sit behind ports
  with normalized results and errors.
- **File-system boundary:** data files live in per-user or per-machine application data
  folders, never in the installation folder.
- **Background-work boundary:** retryable operations are persisted and resumed after restart.
- **Thread boundary:** the UI thread only renders and binds; disk, database, network, and
  heavy computation run asynchronously off it.

## Detailed Documents

- Architecture index: `docs/architecture/index.md`
- Stack baseline: `docs/architecture/stack.md`
- Bootstrap guide: `docs/architecture/bootstrap.md`
- Layers, use cases, and sensitive operations: `docs/architecture/dotnet-architecture.md`
- WPF and MVVM: `docs/architecture/wpf-mvvm.md`
- Cognitive architecture: `docs/architecture/cognitive-architecture.md`
- Design foundations: `docs/architecture/design-foundations.md`
- Component selection: `docs/architecture/component-selection.md`
- Shared component contracts: `docs/architecture/component-system.md`
- Interface design: `docs/architecture/interface-design.md`
- Data grids: `docs/architecture/data-tables.md`
- Forms and workflows: `docs/architecture/forms-and-workflows.md`
- Feedback and states: `docs/architecture/feedback-and-states.md`
- Navigation, windows, and input: `docs/architecture/navigation-and-windows.md`
- Authentication: `docs/architecture/authentication.md`
- Database: `docs/architecture/database.md`
- Idempotency and concurrency: `docs/architecture/idempotency.md`
- Integrations: `docs/architecture/integrations.md`
- Distribution and updates: `docs/architecture/distribution.md`
- ADRs: `docs/architecture/adr/`

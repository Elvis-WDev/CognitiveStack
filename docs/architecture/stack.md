# Stack Baseline

This is the default technology stack for Windows desktop applications using this context.
A project that deviates records the reason and its consequences in an ADR in
`docs/architecture/adr/`.

## Language And Runtime

- C# 14.
- .NET 10 LTS. Libraries target `net10.0`; only the WPF host targets `net10.0-windows`.
- Nullable reference types enabled, implicit usings enabled, analyzers at the latest level.
- `decimal` for money, quantities with fractions, rates, and taxes. Never `double` or `float` for amounts.

## Interface

- WPF with XAML.
- MVVM with CommunityToolkit.Mvvm: `ObservableObject`, `[ObservableProperty]`,
  `[RelayCommand]`, `AsyncRelayCommand`, and `ObservableValidator`.
- One icon family, stored as vector geometry resources, never as a font that may be missing
  on the target machine.
- Visual tokens in XAML resource dictionaries; see `design-foundations.md`.

## Composition

- Generic Host from Microsoft.Extensions.Hosting, started by the WPF application.
- Microsoft.Extensions.DependencyInjection, Options, Configuration, and Logging.
- Serilog with a rolling file sink behind `ILogger<T>`.

## Persistence

- SQLite through Microsoft.Data.Sqlite.
- Dapper for queries and commands, with parameterized SQL only.
- Versioned SQL migration scripts embedded in the infrastructure project and applied by a
  small runner at startup. See `database.md`.
- Backups through `SqliteConnection.BackupDatabase`.

## External Services

- `HttpClient` through `IHttpClientFactory`, with typed clients.
- Microsoft.Extensions.Http.Resilience for timeouts, retries, and circuit breaking.
- System.Text.Json, with source generation for contracts on hot paths.
- MailKit for SMTP.

## Testing

- xUnit for unit, integration, and ViewModel tests.
- Real SQLite databases in temporary files for integration tests, never mocks of SQL.
- Mailpit, run in Docker, as the SMTP sink during development and tests.

## Packages

- NuGet with Central Package Management in `Directory.Packages.props`.
- Lock files enabled (`packages.lock.json`) and restored in locked mode in CI.
- NuGet Audit enabled for direct and transitive packages.
- Every production dependency is justified in its commit or in an ADR.

## Development Environment

- Development may happen on Linux, with VS Code or Zed and the .NET SDK.
- The WPF host builds on Linux with `EnableWindowsTargeting` set to `true`.
- Wine runs quick smoke tests of the WPF host during development. It does not certify
  compatibility.
- Acceptance happens on real Windows 10 and 11 machines before any release.
- Docker is used only for auxiliary tools, such as Mailpit or a read-only SQLite browser.

## Distribution

- `dotnet publish` for `win-x64`, self-contained unless an ADR chooses framework-dependent.
- Signed executables, a signed installer, and signed update packages.
- The installer and updater technology is recorded in an ADR. See `distribution.md`.

## Platform Scope

The baseline supports Windows 10 and 11 on the .NET 10 runtime. It does not promise support
for Windows versions without vendor support. A legacy edition, if ever justified, gets its
own decisions, code, dependencies, and tests; the main project is never degraded to sustain
it.

## Tooling Outside The Application

OpenSpec runs on Node.js as a developer tool only. It is not part of the application, and
no Node.js dependency enters the solution. See `../quality/change-lifecycle.md`.

# New Application Bootstrap

Use this guide when starting a new Windows desktop application from this context.

## 1. Create The Repository

Copy the context kit into the new project:

```bash
git clone --depth 1 --branch Cognitive-dotnet https://github.com/Elvis-WDev/CognitiveStack.git context-template
rsync -av --exclude='.git' context-template/ path/to/your-project/
rm -rf context-template
```

The kit is:

- `AGENTS.md`
- `ARCHITECTURE.md`
- `CLAUDE.md`
- `.agents/skills/`
- `.claude/`
- `openspec/`
- `docs/`
- `.github/`
- `.codex/README.md`

Then adapt it before asking an agent for changes:

- Replace `{{PROJECT_NAME}}`, `{{ProjectName}}`, and `{{Company}}` everywhere.
- Replace the verification commands in `AGENTS.md` when the project exposes different names.
- Adapt every document to the architecture the project actually has, not the one it may get
  later. Remove the optional `Presentation` project from `ARCHITECTURE.md` if it is not used.

When an existing repository already has documentation, merge by difference: compare each
file, keep project-specific knowledge, and never overwrite folders or generated skills blindly.

## 2. Development Machine

Install the .NET 10 SDK and confirm it:

```bash
dotnet --version
```

On Linux, also install Wine for smoke runs of the WPF host, and Docker for auxiliary tools.
Real Windows 10 and 11 machines are still required for acceptance before any release.

## 3. Solution Layout

```bash
dotnet new sln --format slnx --name {{ProjectName}}
dotnet new classlib -o src/{{ProjectName}}.Domain
dotnet new classlib -o src/{{ProjectName}}.Application
dotnet new classlib -o src/{{ProjectName}}.Infrastructure
dotnet new classlib -o src/{{ProjectName}}.Presentation
dotnet new wpf      -o src/{{ProjectName}}.Desktop
dotnet new xunit    -o tests/{{ProjectName}}.Domain.Tests
dotnet new xunit    -o tests/{{ProjectName}}.Application.Tests
dotnet new xunit    -o tests/{{ProjectName}}.Infrastructure.Tests
dotnet new xunit    -o tests/{{ProjectName}}.Presentation.Tests
```

Add every project to the solution, and add references only in the directions allowed by
`ARCHITECTURE.md`.

## 4. Shared Build Settings

At the repository root:

- `global.json` pins the SDK feature band.
- `Directory.Build.props` sets, for every project: `LangVersion` 14, `Nullable` enable,
  `ImplicitUsings` enable, `TreatWarningsAsErrors` true, `AnalysisLevel` latest,
  `EnforceCodeStyleInBuild` true, and `RestorePackagesWithLockFile` true.
- `Directory.Packages.props` enables Central Package Management, NuGet Audit for direct and
  transitive packages, and lists every package version once.
- `.editorconfig` holds naming, formatting, and analyzer severities, so
  `dotnet format --verify-no-changes` has one source of truth.
- `.gitignore` covers `bin/`, `obj/`, `.vs/`, `*.user`, `TestResults/`, local databases, and logs.

The WPF host targets `net10.0-windows`, sets `UseWPF` true, and sets
`EnableWindowsTargeting` true so it builds on Linux. Every other project targets `net10.0`.

## 5. Interface Baseline

Create the WPF host with:

- The Generic Host built in `App` startup, and the shell window resolved from the container.
- An application manifest with Per-Monitor V2 DPI awareness.
- Resource dictionaries for tokens, light and dark themes, and shared styles. See
  `design-foundations.md`.
- The shared services from `wpf-mvvm.md`: navigation, dialogs, file pickers, and feedback.
- A global handler for unhandled exceptions that logs, shows a sanitized message, and keeps
  data safe.

## 6. Database Baseline

Create the infrastructure for SQLite and Dapper. See `database.md`:

- An application-paths service resolving the data, backup, and log folders.
- A connection factory that applies the connection pragmas.
- Dapper type handlers for `decimal`, dates, `Guid`, and value objects.
- The migration runner, the `schema_migrations` table, and `0001_initial.sql`.
- The backup service with integrity verification.

## 7. Authentication Baseline

When the product has local users, follow `authentication.md`:

- Create the first administrator through the setup flow; never ship default credentials.
- Hash passwords with the documented platform primitive.
- Resolve the current user and role through the session port, and lock after inactivity.

## 8. Tooling Baseline

- `scripts/wine.sh` builds the WPF host and runs it under Wine for smoke tests.
- A `docker-compose.yml` for auxiliary tools only: Mailpit as the SMTP sink, and a read-only
  SQLite browser when useful. Document their ports in the project README.
- A CI workflow that runs restore in locked mode, build, test, and format verification.

## 9. OpenSpec

The kit ships OpenSpec already initialized for Codex, Claude Code, and GitHub Copilot, with
`openspec/config.yaml` bridging it to these documents. OpenSpec is a Node.js developer tool;
it never becomes a dependency of the solution. Each developer prepares their machine once:

```bash
npm install --global @fission-ai/openspec
openspec config set profile custom
openspec config set workflows '["propose","explore","apply","update","sync","archive","verify"]'
openspec update
```

`openspec update` regenerates the generated skills for the installed CLI version. To support
another agent, add it with `openspec init --tools <id>`; existing files stay untouched. See
`docs/quality/change-lifecycle.md` for the lifecycle.

## 10. Distribution Baseline

Before the first release, decide and record in ADRs:

- the installer and updater technology;
- self-contained or framework-dependent publishing;
- the code-signing certificate and where it is kept;
- the update signing keys and the release channels.

See `distribution.md`.

## 11. Documentation Baseline

Before implementing features:

- Complete `docs/product/overview.md`, including scope and what the application does not do.
- Capture behavior in `openspec/specs/` through the first OpenSpec changes.
- Update `ARCHITECTURE.md` for project-specific boundaries.
- Create ADRs for deviations from this stack.

Start with a local workflow that exercises the whole stack, such as one transaction that
writes several tables, before integrating any external service.

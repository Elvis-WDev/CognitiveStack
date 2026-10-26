# Distribution And Updates

How the application is built, signed, installed, configured, and updated on customer
machines. The installer and updater technology is a project decision recorded in an ADR.

## Publish

```bash
dotnet publish src/{{ProjectName}}.Desktop -c Release -r win-x64 --self-contained true \
  -p:PublishReadyToRun=true -o artifacts/publish/win-x64
```

- Publish self-contained for `win-x64`, so the machine needs no separately installed runtime.
  Framework-dependent publishing is an ADR decision.
- ReadyToRun is optional: it improves startup and grows the package. Measure both.
- Trimming is not supported for WPF. Do not enable `PublishTrimmed`.
- Single-file publishing has caveats for WPF: native libraries, such as the SQLite and WPF
  graphics binaries, stay beside the executable unless they are extracted to a temporary
  folder at startup, which is slower and often blocked by antivirus or policy, and
  `Assembly.Location` is empty. Prefer a folder publish unless an ADR accepts those costs.
- Builds may run on Linux. Acceptance runs on real Windows 10 and 11 machines, never on Wine.

## Signing

- Sign every executable and library the project produces, the installer, and every update
  package with Authenticode, using SHA-256 and an RFC 3161 timestamp so signatures stay valid
  after the certificate expires.
- The signing key lives in a hardware token, an HSM, or a cloud signing service. Never in Git,
  in a plain CI variable, or on a developer laptop as a loose `.pfx`.
- Restrict who and which pipeline may sign, log every signing, and keep a revocation and
  re-signing plan. See `../security/threat-model.md`.
- Unsigned or newly signed binaries trigger SmartScreen warnings until they earn reputation.
  Plan the first releases accordingly.

## Installer And Updater

Choose one in an ADR, such as MSIX, an updater framework, or a classic installer with a
custom updater. The ADR records:

- per-user or per-machine installation, and whether installation needs elevation;
- how updates are discovered, downloaded, verified, applied, and rolled back;
- how the installer creates data folders and their permissions;
- what uninstalling does with data.

Whatever the technology:

- The application runs as a standard user. Administrator rights, if ever needed, are for
  installation only.
- The installation folder is read-only at runtime and replaced by updates. Databases,
  backups, logs, generated documents, and user settings live in application data folders.
  See `database.md`.
- Uninstalling keeps data by default and says where it is. Deleting it is an explicit choice.

## Update Manifests

The application learns about updates from a signed manifest:

```json
{
  "version": "1.8.0",
  "channel": "stable",
  "minimumUpgradeFrom": "1.5.0",
  "schemaVersion": 42,
  "minimumReadableSchema": 40,
  "package": { "url": "https://updates.example.com/1.8.0/app.zip", "size": 98213344,
               "sha256": "..." },
  "publishedAt": "2026-10-01T12:00:00Z",
  "rollout": { "percentage": 25 }
}
```

Before installing, verify all of these, and refuse on any failure:

1. The manifest signature, with a public key embedded in the application. The private key
   never ships. See `../security/hardening.md`.
2. The package hash against the signed manifest, and the package's Authenticode signature.
3. The version is newer than the installed one. A downgrade needs an explicitly signed
   rollback manifest.
4. The channel matches the one configured for the machine.
5. The current version is at least `minimumUpgradeFrom`, and the local database schema is
   compatible with the package.

HTTPS protects the download; the signature decides trust. Never execute scripts, commands, or
URLs supplied by the server outside the signed package, and download only from allowlisted
hosts.

## Applying An Update

1. Download and verify in the background, without interrupting work.
2. Install only with the application closed. Ask the operator to close it, never in the
   middle of an operation, and wait until pending operations reach a safe point.
3. Take a verified backup of every database before applying. See `database.md`.
4. Install. On the next start, the migration runner applies pending scripts.
5. Keep the previous version until the new one starts and passes its health check.

## Rollback

- When the new version fails to start or migrate, restore the previous version and the
  pre-update backup, and report it.
- An older version refuses a newer schema, so rolling back after the new version wrote data
  means restoring the backup and losing that data. Prefer a forward fix once operators have
  worked on the new version, and reconcile with external authorities after any restore.
- Make migrations additive across releases where possible, so a forward fix is cheap.

## Channels And Rollout

- Use channels such as `dev`, `pilot`, and `stable`. A machine's channel is configuration,
  not a code branch.
- Roll out in stages: internal machines, then pilot customers, then a growing percentage of
  `stable`. Be able to pause a rollout.
- Every release has release notes, a version, the minimum schema it needs, and a tested
  upgrade path from each supported earlier version.

## Configuration

- `appsettings.json` ships in the installation folder with defaults only, never secrets.
- Machine settings, such as devices and channel, live in `%ProgramData%`; user preferences,
  such as theme and window placement, live in `%LocalAppData%`. Updates preserve both.
- Bind configuration to typed options and validate it at startup. See
  `dotnet-architecture.md`.
- Secrets live in the machine secret store. See `../security/hardening.md`.

## Verification

- [ ] The publish command and its options are documented, and single-file or trimming
      choices are justified.
- [ ] Executables, installer, and update packages are signed and timestamped.
- [ ] The signing key is in hardware or a signing service.
- [ ] Manifest signature, hash, version, channel, and schema compatibility are verified
      before install.
- [ ] Updates install only with the application closed, after a verified backup.
- [ ] A failed update rolls back to a working version, tested on real Windows.
- [ ] No data lives in the installation folder, and no secret ships in configuration.

## Reject The Change When

- a release ships unsigned binaries, installer, or update package;
- the updater installs anything before verifying signature and hash;
- the updater executes server-supplied scripts or commands;
- a private signing, license, or update key appears in the repository or the client;
- an update can run while the application is open or without a backup.

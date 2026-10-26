# Security Hardening

`principles.md` states why. `threat-model.md` states what is defended against. This document
states which controls to implement at each boundary. `access-control.md` specifies
authorization and company scoping in depth.

## Untrusted Input

Everything crossing a boundary is untrusted until validated: operator input, scanner and
keyboard-wedge input, imported files, configuration files, command-line arguments, external
API responses, device replies, and data restored from a backup.

- Validate shape, type, range, and length at the use-case boundary, before business logic.
- Validate configuration at startup and refuse to start when a required value is invalid.
- Reject unknown fields when a file or contract format is closed.
- Re-validate values stored earlier by an untrusted source, such as an old import.
- Show a business message on failure and keep diagnostics in logs.

## Injection And Paths

- Every value reaches SQL as a Dapper parameter. Never concatenate input into SQL; allowlist
  sort and filter columns. See `../architecture/database.md`.
- Never pass input to a shell, `Process.Start` arguments, or anything evaluated at runtime.
- Canonicalize every user-influenced path and confirm it stays under the expected root:

```csharp
var root = Path.GetFullPath(paths.ExportsFolder) + Path.DirectorySeparatorChar;
var full = Path.GetFullPath(Path.Combine(root, fileName));
if (!full.StartsWith(root, StringComparison.OrdinalIgnoreCase))
    return Result.Fail(Errors.InvalidPath);
```

- Generate stored file names; never trust a name from an import or the user.
- Never use `BinaryFormatter` or type-name handling when deserializing untrusted data.

## XML

When the product reads XML, such as electronic documents or provider responses, disable
DTDs and external resolution:

```csharp
var settings = new XmlReaderSettings
{
    DtdProcessing = DtdProcessing.Prohibit,
    XmlResolver = null,
    MaxCharactersInDocument = 10_000_000,
};
using var reader = XmlReader.Create(stream, settings);
var document = XDocument.Load(reader);
```

When verifying XML signatures, confirm that the signed reference is the element the
application actually uses.

## Secrets On The Machine

- Store API tokens, SMTP passwords, and certificate passwords with DPAPI
  (`ProtectedData.Protect`) or Windows Credential Manager, behind an `ISecretStore` port
  implemented as a Windows platform service. Tests on Linux use a fake.
- Use `DataProtectionScope.CurrentUser` for per-user secrets. `LocalMachine` lets any process
  on the machine decrypt; use it only for secrets shared by every user, and record why.
- Never place secrets in `appsettings.json`, code, logs, error messages, or support bundles.
- A leaked secret is rotated, not merely deleted.

## Certificates

- Client certificates (`.pfx`, `.p12`), such as those required for electronic invoicing, go
  into the Windows certificate store with a non-exportable key, or stay in the application
  data folder with their password in the secret store.
- Load them with `X509CertificateLoader`, never commit them to Git, and never log them or
  their passwords.
- Warn administrators well before a certificate expires.

## Licenses And Binaries

- Binaries are decompilable. Obfuscation is not security, and any value in the client is
  public.
- No private key for licenses, updates, or code signing ships in the client. The client holds
  public keys only.
- Licenses are signed files verified with the embedded public key on every start.
- Offline grace periods are explicit. Record the last trusted time and detect a clock moved
  backwards.
- Decide in an ADR what an expired license blocks. Never lock customers out of reading or
  exporting their own data.

## Updates

- Verify update manifests and packages before installing: signature, hash, version, channel,
  and schema compatibility. See `../architecture/distribution.md`.
- Never execute scripts or commands supplied by a server outside the signed package.

## Dependencies And Supply Chain

- Prefer the base class library. Every dependency is a liability.
- Use Central Package Management, lock files restored in locked mode in CI, and NuGet Audit
  for direct and transitive packages. See `../architecture/stack.md`.
- Use package source mapping, so each package comes from its expected feed.
- Verify the exact package name and owner before adding it; typosquatting relies on a glance.
- Confirm the license is compatible, and that the package is maintained.
- Review the lock file diff; a single bump can pull in many transitive changes.

## Logging And Personal Data

- Never log passwords, PINs, tokens, license keys, certificate passwords, card data, or full
  payloads with personal data. Redact with Serilog destructuring policies where values are
  structured.
- Include a correlation identifier so a reported error can be traced.
- Log authorization failures and suspicious input, and keep them reviewable.
- Collect only the personal data the product needs, with a retention rule for each kind.
- Support bundles are created on demand, list their contents, are redacted, and include the
  database only with explicit consent.

## Backups

- Encrypt backups that leave the machine when they hold personal or financial data.
- Derive the key from a passphrase or keep it in a documented escrow. A key protected only by
  DPAPI on the original machine makes the backup unrecoverable after that machine is lost.
- Verify every backup. See `../architecture/database.md`.

## Least Privilege

- The application runs as a standard user and never writes to the installation folder.
- Installer elevation, when needed, creates folders and permissions and then ends.
- Do not install services that run as `SYSTEM` unless an ADR justifies them.
- Grant per-machine data folders only the permissions the application's users need.

## Verification

- [ ] Every new boundary validates its input before use.
- [ ] No SQL, path, or command is built by concatenating input; paths are canonicalized.
- [ ] XML parsing prohibits DTDs and external resolution.
- [ ] Secrets and certificate passwords live in DPAPI or Credential Manager, never in files
      or logs.
- [ ] No private key ships in the client; licenses and updates are verified with public keys.
- [ ] New dependencies were verified, audited, and justified, with lock files committed.
- [ ] Logs and support bundles are redacted, and external backups are encrypted.
- [ ] The application works without administrator rights.

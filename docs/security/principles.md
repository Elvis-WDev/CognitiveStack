# Security Principles

- Treat the PC, its Windows users, and its files as outside our control. Enforce rules in use
  cases; the interface only mirrors them.
- Be honest about limits: application authorization does not stop someone who can read or
  edit the database file. Use OS permissions, encryption, and tamper evidence for that.
- Knowing an identifier is never authorization. Scope every private query by company and
  ownership.
- Derive user, role, and company from the session service, never from input DTOs.
- Deny by default. Every sensitive use case checks an explicit permission.
- Make every critical side effect idempotent, and enforce critical invariants in the database.
- Validate input at every boundary: files, imports, devices, configuration, and remote
  responses.
- Never write custom cryptography. Use the platform's password hashing, DPAPI, and signature
  verification.
- Secrets are never committed and never shipped. Machine secrets live in DPAPI or Windows
  Credential Manager.
- Binaries are public. No private key for licenses, updates, or signing ships in the client.
- Sign what we ship, and verify what we install.
- The application runs as a standard user; elevation is for installation only.
- Collect only the personal data the product needs, and keep it out of logs and support
  bundles.
- Do not expose provider names, paths, or internal details unless users need them.

## Configuration

Document required settings in the application repository. Shipped configuration holds
defaults and references, never secret values:

```json
{
  "Billing": {
    "BaseAddress": "https://api.example.com/",
    "CredentialName": "{{ProjectName}}/Billing"
  }
}
```

The credential itself is stored on the machine. See `hardening.md`.

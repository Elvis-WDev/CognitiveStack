# Threat Model

## Assets

- Business records: sales, invoices, payments, stock, cash, and sequential numbers.
- Personal data of customers, employees, and operators.
- Local users, roles, and password hashes.
- Secrets on the machine: API tokens, SMTP credentials, and client certificates.
- Vendor keys: code-signing certificate and the private keys for licenses and updates.
- The update channel and the shipped binaries.
- Backups, the audit trail, logs, and support bundles.
- The isolation between companies inside one database.

## Trust Boundaries

- Operator to application, through the interface.
- Windows user or process to the application's files, outside the application.
- Application to SQLite, which anyone with file access can also open.
- Application to external services: HTTP APIs and SMTP.
- Application to devices, imported files, and configuration.
- Vendor update and license services to the application.
- Vendor build and signing pipeline to the shipped binaries.
- Company to company, inside the same database.
- The PC to external backup media.
- Support channel to the PC: remote sessions and support bundles.

## Threats

| Threat | Impact | Mitigation |
| --- | --- | --- |
| Unlocked PC used by someone else | Actions under another operator's identity | Auto-lock, re-authentication for privileged actions, audit. See `../architecture/authentication.md`. |
| PC or disk stolen, database copied | Data breach | BitLocker or database encryption by ADR, data minimization, encrypted backups. See `hardening.md`. |
| Malware on the PC | Stolen data and credentials | Standard-user runtime, DPAPI per user, no plaintext secrets, signed binaries. Not fully preventable; say so. |
| Binary decompiled for embedded secrets | Abused vendor services, forged licenses | No private keys or shared secrets in the client; public keys only. See `hardening.md`. |
| SQLite edited with an external tool | Fraud, inconsistent books | Database constraints, audit log, integrity checks, optional hash chain, reconciliation. See `access-control.md`. |
| Signing certificate stolen | Malware signed as the vendor | Key in hardware or a signing service, restricted signers, revocation plan. See `../architecture/distribution.md`. |
| Forged or tampered update | Code execution on every customer PC | Signed manifest and package, verified before install, no server-supplied scripts. |
| Old update replayed | Known vulnerabilities reintroduced | Monotonic version check; downgrades only by signed rollback manifest. |
| Power loss or crash mid-operation | Corrupted or half-written data | Transactions, `synchronous = FULL`, WAL, pending operations, verified backups. See `../architecture/database.md`. |
| Double click, retry, or restart repeats a command | Duplicate sale, payment, or document | Natural idempotency or scoped keys. See `../architecture/idempotency.md`. |
| Two windows, users, or a job write at once | Oversold stock, negative balance | Immediate transactions, guarded updates, version columns, constraints. |
| Operator fraud: voids, discounts, cash | Financial loss | Permissions, supervisor override, limits, audit with both users, per-operator void and discount reports, cash reconciliation. |
| Role manipulation or interface bypass | Privilege escalation | Authorization in use cases, deny by default, no self-promotion. See `access-control.md`. |
| Operator reaches another company's data | Cross-company breach | Company from the session, scoped queries, composite foreign keys, two-company tests. |
| Local password brute force or default account | Account takeover | Approved password hashing, lockout with backoff, first administrator created in setup. |
| Personal data in logs or support bundles | Privacy breach | Redaction, minimization, reviewed and consented bundles. See `hardening.md`. |
| Unauthorized remote control of the PC | Attacker operates the application | Auto-lock, re-authentication and override for privileged actions, audit, no built-in remote access. |
| Malicious imported file or XML | Corruption, file disclosure | Staged validated imports, safe XML settings, canonical paths. See `hardening.md`. |
| Compromised or typosquatted NuGet package | Malicious code in the build | Central Package Management, lock files, NuGet Audit, source mapping, review. |
| License file or clock tampering | Unlicensed use | Signed licenses verified with a public key, last-trusted-time checks. |
| External service outage or ambiguous timeout | Lost results or duplicate remote documents | Pending operations, status query before retry, never a new document to compensate. See `../architecture/integrations.md`. |
| Restore from an old backup | Reused sequential numbers, lost external results | Reconcile with the external authority before creating records. See `../architecture/database.md`. |
| Leaked SMTP or API credential | Abuse under the customer's account | Secret store, per-installation credentials, rotation. |

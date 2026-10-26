# Authentication Architecture

Local sign-in is optional. A product that runs as one trusted operator per machine may skip
it and work with a fixed local session, recorded in an ADR. A product with operators,
supervisors, and administrators, or with actions that must be attributable to a person,
needs local users.

## Default

- Users, roles, and password hashes live in the application database, in the company scope
  the product defines. See `../security/access-control.md`.
- The application session service resolves the current user, role, permissions, and
  selected company, and exposes them to use cases through an `ICurrentSession` port.
- Windows account integration, signing the user in from `WindowsIdentity`, is an option
  chosen in an ADR. It moves identity to the operating system; roles stay in the
  application.

## Password Storage

Never write custom cryptography. Choose one of these and record it in `stack.md` or an ADR:

| Option | Settings |
| --- | --- |
| `Rfc2898DeriveBytes.Pbkdf2` | SHA-256, at least 600,000 iterations, 16-byte random salt per user, 32-byte output |
| `PasswordHasher<TUser>` from Microsoft.Extensions.Identity.Core | Current format version; rehash when it returns `SuccessRehashNeeded` |

```csharp
byte[] salt = RandomNumberGenerator.GetBytes(16);
byte[] hash = Rfc2898DeriveBytes.Pbkdf2(password, salt, iterations, HashAlgorithmName.SHA256, 32);

// Verification: recompute with the stored salt and iterations, compare in constant time.
bool valid = CryptographicOperations.FixedTimeEquals(candidate, storedHash);
```

- Store the algorithm, iteration count, salt, and hash together, so stronger settings can be
  adopted by rehashing at the next successful sign-in.
- A short PIN for fast user switching, when the product needs one, is hashed the same way and
  locks out sooner than a password.
- Never store, log, or display a password or PIN, not even masked or truncated.

## Sign-In And Lockout

- Show one message for every failure: "Invalid user or password". Do not reveal which part
  was wrong or whether the user exists.
- Count consecutive failures per user in the database, so a restart does not reset them.
- Apply increasing delays after a few failures, and lock the account for a period after a
  documented threshold. An administrator or supervisor may unlock it; the unlock is audited.
- Record failed and successful sign-ins in the audit trail without the attempted secret. See
  `../quality/observability.md`.

## Session

- The session lives in memory only. Closing the application ends it; no token is persisted
  to restore it.
- Auto-lock after a configurable period of inactivity. The lock screen hides business data,
  keeps open work intact, and asks the same user, or an authorized one, to sign in again.
- Signing out or switching user clears cached data that belonged to the previous session.
- Role or permission changes take effect at the next sign-in at the latest; revoking access
  of a signed-in user ends their session.

## Supervisor Override

Some actions need a second person, such as authorizing a void, a discount above a limit, or
opening the cash drawer without a sale.

- The use case requests an override; the interface shows a dialog through `IDialogService`
  where the supervisor enters their own credentials.
- The approval is single-use, bound to one operation and one resource, and expires in
  seconds. It never switches the session to the supervisor.
- The use case verifies the supervisor's permission itself; the dialog only collects
  credentials.
- The audit record holds both the operator and the approving supervisor.
- Failed override attempts count toward the supervisor's lockout.

## First Administrator

- Never ship default credentials or a built-in account.
- On first start with no users, a setup wizard creates the first administrator, with
  password rules shown before submission.
- The wizard creates the user only while no user exists, checked in the same write
  transaction, so it cannot be replayed later to add an administrator.
- Password recovery is offline by design: an administrator resets other users' passwords.
  For a lost last administrator, document a recovery procedure, such as a one-time recovery
  code shown at setup and stored hashed.

## Verification

- [ ] Password hashing uses one of the approved options, recorded in stack or ADR.
- [ ] Hash comparison is constant-time; failures share one message.
- [ ] Lockout survives a restart, and unlocks are audited.
- [ ] Auto-lock hides data and keeps open work.
- [ ] Overrides are single-use, scoped, verified by the use case, and audited with both users.
- [ ] No default credentials exist, and the setup wizard cannot run once a user exists.
- [ ] No password, PIN, or hash appears in logs, errors, or support bundles.

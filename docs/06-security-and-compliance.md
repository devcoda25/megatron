# 06. Security and Compliance

Megatron holds access to real trading accounts. Treat every item below as a requirement, not a suggestion.

## 1. Authentication and sessions

| Area | Requirement |
|---|---|
| Password hashing | Argon2id with parameters tuned to the host. Never log or return hashes |
| Password rules | Minimum length 12, check against a breached-password list, strength meter in the UI |
| Access token | JWT, short life (15 minutes for users, 5 for admins), signed with a rotating key, contains user ID, role, session ID |
| Refresh token | Opaque random value, stored **hashed**, in an httpOnly, Secure, SameSite cookie. **Rotates on every use**. Reuse of an old token revokes the whole token family (theft detection) |
| Browser storage | Never put tokens in `localStorage` or `sessionStorage` |
| 2FA | TOTP (RFC 6238). Optional for users, **mandatory for admins**. Secrets encrypted at rest. Recovery codes shown once, stored hashed, single use |
| PIN | Short numeric app lock, stored hashed with rate limiting. Used for step-up and for re-entering a locked session. It is a convenience, not a replacement for the password |
| Passkeys | Planned (WebAuthn) as the "biometric login" in a web app |
| Lockout | Progressive delay and temporary lock after repeated failures, with email notice. Generic error messages that do not reveal which field was wrong |
| New device | Email code and a "new login" notification |
| Session list | Users and admins can see and revoke sessions |
| Idle and absolute timeouts | User: idle 30 days with refresh, absolute 90 days (assumption). Admin: idle 15 minutes, absolute 8 hours (assumption) |

### Step-up authentication

A recent password, TOTP or PIN check (valid for 5 minutes, assumption) is required for:

- switching to **Live**,
- starting the bot in Live,
- changing risk limits upward (higher risk per trade, higher daily loss),
- linking or removing a broker account,
- changing password, 2FA or PIN,
- deleting the account,
- every sensitive admin action.

**Stopping is never gated.** The kill switch and "stop bot" need a confirmation in the UI but no extra authentication.

## 2. Roles and authorisation

- Two roles: `user` and `super_admin`.
- Every route declares its required role. A dependency checks it on each request.
- Every query filters by the authenticated `user_id` (no trusting IDs from the client).
- Admin routes live under `/admin`, have their own login, and return `403` without detail to non-admins.
- The frontend hides admin screens from normal users, but that is cosmetic. **The server is the only enforcement.**
- Admins can manage users and see status data. They **cannot** read broker credentials, place trades for users, or read PIN or 2FA secrets.

## 3. Broker credentials and secrets

| Area | Requirement |
|---|---|
| Encryption at rest | **Envelope encryption**: a random data key per credential encrypts it with AES-256-GCM, and the data key is wrapped by a master key. The master key comes from a secret manager or environment, **never from the database or the repo** |
| Key rotation | Store `key_version`. Support re-wrapping without downtime |
| Access | Only the broker adapter layer decrypts, in memory, at the moment of use. API routes can never select the ciphertext columns |
| Never | Log, return in an API response, put in an error message, or include in an audit row |
| Permissions | Accept **trade-only** access. Reject or warn when the broker reveals withdrawal or transfer rights. Users are told to use investor/trade-only credentials where available |
| Honest wording | The UI must not claim a "hardware enclave" or similar unless one is actually used |
| Local development | Use demo accounts only. `.env` is git-ignored. Provide `.env.example` with fake values |

## 4. Application security

- Validate every input with Pydantic. Reject unknown fields on write endpoints.
- Parameterised queries only (SQLAlchemy). No string-built SQL.
- Rate limiting and basic abuse protection on auth, password reset and broker linking.
- CORS: not needed when the frontend uses the same-origin proxy. If enabled, allow only the exact frontend origin.
- Security headers on the frontend: CSP, `X-Content-Type-Options`, frame protection, strict referrer policy, HSTS in production.
- CSRF: cookies are `SameSite`, state-changing requests also require a custom header or a CSRF token.
- Dependency hygiene: pinned versions, automated vulnerability scanning, and secret scanning in CI.
- Backups: encrypted, tested restores, for Postgres. Never test restores on production.
- Logging: structured logs with secret redaction. No credentials, tokens, TOTP secrets, PINs or full account numbers.

## 5. Trading safety controls (security of money)

These are covered in [03](03-system-architecture.md) and [04](04-backend-design.md) and repeated here because they matter most:

- Broker-side SL and TP on every order.
- A single order path through the risk manager, with an `ApprovedOrder` type.
- Idempotent order submission and an `unknown` state that blocks new entries.
- Reconciliation with the broker, fail closed on mismatch.
- Kill switch with two modes (halt new entries, or halt and flatten).
- Server-enforced go-live checklist.
- Daily loss limit, max open trades, spread cap, loss-streak cooldown.
- Watchdog on the engine heartbeat.

## 6. Threat model (summary)

| Threat | Mitigation |
|---|---|
| Stolen password | 2FA, new-device alerts, step-up for sensitive actions, lockout |
| Stolen refresh token | Rotation and reuse detection, httpOnly cookie, session list and revoke |
| Database leak | Argon2id hashes, encrypted credentials with the key held elsewhere, no plaintext secrets |
| Malicious or compromised admin | Mandatory 2FA, short sessions, step-up, audit log, no access to broker secrets or trading |
| Account takeover then trading | Step-up for Live, trade-only broker permissions (no withdrawals), loss limits, notifications |
| Duplicate or runaway orders | Idempotency keys, per-day trade cap, risk veto, kill switch |
| Bad data feed | Candle validation, staleness checks, fail closed |
| Insider reading logs | Redaction, no secrets in logs |
| Phishing look-alike of the app | Out of scope for code, but tell users the real domain and never ask for broker passwords by email |

## 7. Regulatory, legal and platform notes

This is not legal advice. Get professional advice before widening the audience.

- **Own account vs other people's accounts.** Trading only the project owner's account for themselves is far simpler. Letting **other people** connect accounts and have a service trade for them can be a regulated activity (for example investment advice, portfolio management or a licensed trading service) depending on the country. That includes Uganda, where the capital markets and other regulators may apply, and the home countries of the users. Decide this before opening sign-ups.
- **Broker terms.** Each broker's client agreement may limit automated trading, scalping, high order rates or sharing a trading password with a third-party service. Verify in the official agreement. Do not rely on third-party summaries.
- **Risk disclosure.** Show a clear risk notice on sign-up, on the Bot activation sheet and on the Broker screen. Make the user accept it, and record the acceptance with the version and time.
- **No promises.** No "guaranteed", "risk-free" or win-probability claims. Show measured results with sample sizes.
- **Privacy.** Collect the minimum personal data. Provide data export and account deletion. Store only what is needed.
- **App stores.** If a wrapper app is ever published, finance and trading apps face extra review rules (Apple is strict). The v1 plan is a mobile web app, so this is a future concern.
- **Marketing copy.** Do not describe the engine as AI price prediction. It is rule-based automation.

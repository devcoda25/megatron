# 05. API Specification

The backend publishes an **OpenAPI 3.1** document, which is the contract. The frontend generates its typed client from it. This page defines conventions and the endpoint inventory; the OpenAPI file is authoritative for exact schemas.

## 1. Conventions

| Topic | Rule |
|---|---|
| Base path | `/v1` for users, `/v1/admin` for admin |
| Format | JSON (`application/json`). Errors use **RFC 9457** `application/problem+json` with a stable machine `code` |
| Money and prices | **Decimal strings**, for example `"2654.20"`. Never JSON numbers |
| Times | ISO 8601 UTC, for example `2026-10-28T09:30:00Z` |
| IDs | UUIDv7 strings |
| Pagination | Cursor-based: `?limit=50&cursor=...`, response `{ items, next_cursor }` |
| Filtering | Query parameters, documented per endpoint |
| Idempotency | Mutating endpoints that start real-world effects (`POST /bot/start`, `/positions/{id}/close`, `/bot/kill`) accept an `Idempotency-Key` header and return the original result on replay |
| Auth | Short-lived access token plus rotating refresh token in an httpOnly, Secure, SameSite cookie. See [06](06-security-and-compliance.md) |
| Step-up | Sensitive endpoints require a recent step-up (password, TOTP or PIN). Missing step-up returns `403` with code `step_up_required` |
| Rate limits | Per IP and per user, stricter on auth endpoints. Returns `429` with `Retry-After` |
| Versioning | Breaking changes go to `/v2`. Additive changes stay in `/v1` |
| Roles | `user` and `super_admin`. Enforced **server-side** on every route. A user calling an admin route gets `403` with no detail |

### Error shape

```json
{
  "type": "https://megatron.dev/problems/risk-veto",
  "title": "Order rejected by risk manager",
  "status": 409,
  "code": "risk_daily_loss_limit",
  "detail": "Daily loss limit reached. Trading resumes at 00:00 UTC.",
  "instance": "/v1/bot/start",
  "request_id": "01JB..."
}
```

## 2. Authentication (users)

| Method and path | Purpose | Notes |
|---|---|---|
| `POST /auth/signup` | Create account | Sends verification email. Requires accepted terms and risk disclosure |
| `POST /auth/verify-email` | Confirm email code or link | |
| `POST /auth/resend-verification` | Resend | Rate limited |
| `POST /auth/login` | Email and password | Returns `needs_2fa` or session. Generic error on failure |
| `POST /auth/login/2fa` | TOTP or recovery code | |
| `POST /auth/refresh` | Rotate refresh token | Reuse of an old token revokes the whole family |
| `POST /auth/logout`, `POST /auth/logout-all` | End session(s) | |
| `POST /auth/forgot-password`, `POST /auth/reset-password` | Reset flow | Always responds the same whether or not the email exists |
| `POST /auth/reauth` | Step-up with password, TOTP or PIN | Returns a short-lived step-up grant on the session |
| `POST /auth/google` | Google sign-in | Later phase |
| `GET/PATCH /me` | Profile | |
| `POST /me/2fa/setup`, `POST /me/2fa/enable`, `POST /me/2fa/disable` | Manage TOTP | Enable returns recovery codes once |
| `PUT /me/pin` | Set or change app PIN | |
| `GET /me/sessions`, `DELETE /me/sessions/{id}` | Active sessions | |
| `DELETE /me` | Delete account | Step-up required. Refused while positions are open |

## 3. Admin authentication (separate)

| Method and path | Purpose |
|---|---|
| `POST /admin/auth/bootstrap` | One-time first super admin creation, guarded by a setup token set at deploy time. Disabled once an admin exists |
| `POST /admin/auth/login` | Email and password. No social login |
| `POST /admin/auth/login/2fa` | TOTP (mandatory) |
| `POST /admin/auth/login/recovery` | One-time recovery code |
| `POST /admin/auth/refresh`, `POST /admin/auth/logout` | Session |
| `POST /admin/auth/password/reset` and verify | Needs email code **and** TOTP |
| `POST /admin/auth/password/change` | Needs current password |
| `GET /admin/sessions`, `DELETE /admin/sessions/{id}` | Active admin sessions |

Admin sessions have a shorter lifetime and idle timeout than user sessions. Every admin route re-checks the role.

## 4. Brokers

| Method and path | Purpose |
|---|---|
| `GET /brokers` | Supported brokers and their protocols (registry, not hardcoded brands) |
| `GET /broker-accounts` | The user's linked accounts (masked) |
| `POST /broker-accounts` | Link an account. Credentials are encrypted immediately and never returned |
| `GET /broker-accounts/{id}` | Status, masked login, balance, leverage, latency, last sync |
| `POST /broker-accounts/{id}/test` | Test connection and measure latency |
| `POST /broker-accounts/{id}/reconnect` | |
| `GET /broker-accounts/{id}/symbols` | Availability, spread and lot limits for XAUUSD, BTCUSD, GBPUSD |
| `DELETE /broker-accounts/{id}` | Disconnect. Refused while positions are open, unless flattened first |

## 5. Bot control

| Method and path | Purpose | Auth |
|---|---|---|
| `GET /bot` | Status, mode, uptime, per-symbol state, last signal, active risk limits | |
| `GET /bot/readiness` | Go-live checklist with pass or fail per item | |
| `POST /bot/start`, `/pause`, `/resume`, `/stop` | Lifecycle | Start in Live needs step-up |
| `POST /bot/mode` | Body `{ "mode": "paper" or "live" }` | **Step-up required** for `live`, and readiness must pass |
| `PUT /bot/symbols/{symbol}` | Enable or disable a symbol | |
| `POST /bot/kill` | Body `{ "mode": "halt" or "flatten" }` | Confirmation in the UI. **No step-up** by design |

`GET /bot/readiness` items: broker connected with trade-only permissions verified, risk limits set, feed synchronised, paper trading minimum met (days and trades), no unresolved reconciliation mismatch.

## 6. Settings

| Method and path | Purpose |
|---|---|
| `GET/PUT /settings/risk` | Risk limits. `min_rr` is read-only. Changes are validated and audited |
| `POST /settings/risk/reset` | Reset to defaults |
| `GET/PUT /settings/strategy` | Confirmation types, advanced parameters, per-symbol overrides, `require_approval` |
| `POST /settings/strategy/reset` | |
| `GET/PUT /settings/notifications` | Per kind and channel, quiet hours |
| `POST /push/subscribe`, `DELETE /push/subscribe` | Web push |

## 7. Markets and signals

| Method and path | Purpose |
|---|---|
| `GET /markets/symbols` | Snapshot: last price, change, 24h high and low, spread, bot state per symbol |
| `GET /markets/{symbol}/candles?tf=5m&limit=500&before=...` | Closed candles, plus the forming one flagged `is_closed=false` |
| `GET /markets/{symbol}/zones?state=...` | Zones with state, levels, the four checks with measured values, R:R |
| `GET /zones/{id}` | One zone with its event history |
| `GET /signals?status=...` | Signals |
| `GET /signals/{id}` | Detail: chart data, entry, stop, target, R:R, confirmation type, timeline |
| `POST /signals/{id}/approve`, `POST /signals/{id}/reject` | Semi-auto mode |

## 8. Positions and trades

| Method and path | Purpose |
|---|---|
| `GET /positions` | Open positions with live P&L, R, progress to TP |
| `PATCH /positions/{id}` | Modify SL or TP. **Rule:** SL may move toward profit or break-even but never further from entry than the original. Each change is audited and counted as a manual intervention |
| `POST /positions/{id}/close` | Close one position |
| `POST /positions/close-all` | Close all, with confirmation |
| `GET /trades?symbol=&result=&from=&to=` | History |
| `GET /trades/{id}` | Detail with strategy tags |
| `PUT /trades/{id}/note` | Personal journal note |
| `GET /exports/trades.csv` | Export |

## 9. Performance and backtest

| Method and path | Purpose |
|---|---|
| `GET /performance?period=today\|week\|month\|all` | KPIs: net profit, win rate, profit factor, average R:R, max drawdown, total trades, per instrument and session, hourly win-rate heatmap |
| `GET /performance/equity-curve?period=` | Series |
| `POST /backtests` | Start a run (symbol, range, parameters). Returns `202` with run ID |
| `GET /backtests`, `GET /backtests/{id}` | Status and results |
| `GET /paper/status` | Days run, paper P&L, readiness progress |

## 10. Notifications and system

| Method and path | Purpose |
|---|---|
| `GET /notifications?kind=&cursor=` | List |
| `POST /notifications/read` | Mark read |
| `GET /system/status` | Engine heartbeat age, feed latency (shown in the top bar), maintenance flag |
| `GET /health` | Liveness |
| `GET /ready` | Readiness (database and Redis reachable) |

## 11. Admin

| Method and path | Purpose |
|---|---|
| `GET /admin/users?status=&q=` | List and search |
| `GET /admin/users/{id}` | Detail without secrets |
| `POST /admin/users/{id}/suspend`, `/unsuspend`, `/force-logout`, `/reset-2fa` | Each needs step-up and a reason, and is audited |
| `GET /admin/bots` | All running bots (status only) |
| `POST /admin/platform/halt` | Global halt of new entries, with optional flatten. Needs step-up |
| `GET /admin/audit-log?actor=&action=&from=&to=` | Append-only log |
| `GET /admin/system/health` | Engine, feeds, queue depth, database, Redis, per-broker status |
| `PUT /admin/maintenance` | Toggle maintenance mode |

## 12. Realtime: `GET /stream` (Server-Sent Events)

One authenticated stream per client. Each message has an `event` name, a monotonic `id` (for `Last-Event-ID` resume) and JSON `data`.

| Event | Payload summary |
|---|---|
| `bot.status` | Status, mode, per-symbol state |
| `price.tick` | Symbol, bid, ask, time (throttled, for example at most 2 per second per symbol) |
| `candle.update` | Symbol, forming candle, and `candle.closed` when it closes |
| `zone.update` | Zone ID, state change, checks |
| `signal.update` | Signal ID, status |
| `position.update` | Position ID, price, P&L, R, SL, TP |
| `trade.closed` | Trade summary |
| `pnl.today` | Realised, unrealised, win count, loss count |
| `notification.new` | Notification |
| `system.health` | Feed latency, engine heartbeat, broker connection |
| `session.expiring` | Warn before the session ends |

On reconnect the client resumes with `Last-Event-ID`. If the gap is too large the server sends `resync`, and the client refetches state over REST. The server sends a comment heartbeat every 15 seconds.

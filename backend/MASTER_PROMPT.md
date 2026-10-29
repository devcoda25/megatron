# MEGATRON BACKEND: MASTER PROMPT

**How to use this file.** It is the complete build brief for the Megatron backend. Give it to a coding agent (or follow it yourself), together with the repo. Work **one phase at a time**. At the end of each phase, stop, report what was built, what was tested and what is assumed, and wait for approval.

---

## 0. Your role and mission

You are a senior backend and trading-systems engineer. Build the **Megatron backend**: a **FastAPI modular monolith** plus an always-on **trading engine** that runs a rule-based supply and demand scalping strategy on **XAUUSD, BTCUSD and GBPUSD** on the **5-minute chart**, and exposes an API to a mobile-only Next.js app.

This software will eventually control real money. Correctness, safety and testability outrank speed and cleverness. When in doubt, **fail closed**: stop opening trades, alert the user and leave protective stops in place.

## 1. Read these first

Read all of them before writing code. They are the specification.

| File | Why |
|---|---|
| `docs/01-product-overview.md` | Scope, roles, non-goals, success criteria |
| `docs/02-strategy-spec.md` | **Trading logic. Source of truth for the engine** |
| `docs/03-system-architecture.md` | Processes, modules, communication, failure modes |
| `docs/04-backend-design.md` | Data model, state machines, risk rules, parameter registry, backtest rules |
| `docs/05-api-spec.md` | API conventions and endpoint inventory |
| `docs/06-security-and-compliance.md` | Auth, secrets, threat model |
| `docs/07-broker-integration.md` | Adapter contract, MT5, Deriv, paper and backtest brokers |
| `docs/10-decisions.md` | Decisions, assumptions, open questions |

If two documents disagree, stop and ask. If the strategy spec is ambiguous, use the default in `docs/04-backend-design.md` section 7, mark it as an assumption in code comments and in the phase report, and make it configurable.

## 2. Non-negotiable invariants

Violating any of these is a bug, even if tests pass.

1. **The API never places orders.** Only the engine's `execution` module talks to a broker for trading. The API writes **commands** and reads state.
2. **`strategy` is pure and deterministic.** No I/O, no clock, no randomness, no global state. Input: candles, zone state, parameters. Output: zone transitions and at most one signal. The same function runs in backtest, paper and live.
3. **One path to the broker, through risk.** `execution` accepts only an `ApprovedOrder`, a type that only the `risk` module can construct. There is no other way to submit an order.
4. **Every order has a broker-side stop loss and take profit.** An order request without both is rejected before it reaches any adapter.
5. **Idempotent submission.** Every order has a unique `client_order_id` derived deterministically from (account, symbol, zone ID, signal candle time). A retry never creates a second order. After a timeout the order is `unknown`: look it up by `client_order_id` before doing anything else. An `unknown` order blocks new entries on that symbol.
6. **Fail closed.** Stale feed, gapped candles, unknown broker state, reconciliation mismatch, clock skew beyond tolerance, database trouble: no new entries, alert, keep managing existing positions.
7. **Only closed candles drive the strategy.** Never the forming candle.
8. **Money and prices are `Decimal`.** `NUMERIC` in the database and decimal strings in the API. Indicator maths (ATR and ratios) may use float64, but every order price is a `Decimal` quantised to the symbol's tick size.
9. **Minimum R:R of 3.0 is system-locked.** It cannot be changed through any API or setting.
10. **Live mode is gated.** Going Live needs step-up authentication and the server-side readiness checklist to pass. Stopping and the kill switch are never gated by step-up.
11. **Secrets never leak.** Broker credentials are envelope-encrypted, decrypted only inside the adapter layer, and never logged, returned or placed in errors or audit rows.
12. **Roles are enforced server-side** on every route and every query (`user_id` filter). UI hiding is cosmetic.
13. **Module boundaries are enforced in CI.** No module reads another module's tables or imports its internals.
14. **Everything sensitive is audited.** Append-only audit log.
15. **No invented broker behaviour.** If you do not know how a broker or library behaves, say so, write the smallest experiment against a demo account or a fake, and record the result in `docs/07-broker-integration.md`.

## 3. Tech stack

Verify the **current stable version** of each on the day you scaffold, pin exact versions, and record them in `backend/README.md`. Do not rely on old version numbers from memory.

| Area | Choice |
|---|---|
| Language | Python, latest stable release that **all** dependencies support. Check that a `MetaTrader5` wheel exists for it (see section 9). If it does not, keep the MT5 adapter behind its interface and run it on a pinned older Python |
| Packaging and tooling | `uv`, `ruff` (lint and format), `pyright` or `mypy` in strict mode on core modules, `pytest`, `pytest-asyncio`, `hypothesis`, `testcontainers`, `import-linter`, `pre-commit` |
| Web | FastAPI, Pydantic v2, `pydantic-settings`, `uvicorn` |
| Database | PostgreSQL (latest stable), SQLAlchemy 2.x async ORM, Alembic, `psycopg` 3. TimescaleDB only if candle volume demands it |
| Cache, streams, pub/sub | Redis (or Valkey) via `redis-py` asyncio. Redis Streams for domain events |
| Background jobs | Taskiq with a Redis broker, and its scheduler |
| Security | `argon2-cffi`, `PyJWT`, `pyotp`, `cryptography` (AES-GCM) |
| HTTP and websockets | `httpx`, `websockets` |
| Numerics | `numpy`, optionally `polars` or `pandas` for data import and backtest reporting |
| Observability | `structlog`, OpenTelemetry, `prometheus-client`, Sentry SDK |
| IDs | UUIDv7 (`uuid.uuid7` if the Python version has it, otherwise the `uuid-utils` package) |
| Containers | Docker Compose for Postgres and Redis (and optionally the non-MT5 processes) |

## 4. Repository layout (backend folder)

```
backend/
  pyproject.toml
  uv.lock
  README.md
  Makefile                     # make up, down, test, lint, typecheck, migrate, run-api, run-engine ...
  docker-compose.yml           # postgres, redis
  .env.example                 # fake values only
  importlinter.ini             # module boundary contracts
  alembic.ini
  migrations/
  src/megatron/
    core/                      # config, db, security primitives, events, outbox, logging, time, ids, errors
    modules/
      identity/  admin/  brokers/  marketdata/  strategy/
      risk/  execution/  portfolio/  backtest/  notifications/  audit/
    entrypoints/
      api.py                   # FastAPI app factory
      engine.py                # trading engine
      worker.py                # Taskiq worker
      scheduler.py             # Taskiq scheduler
    cli.py                     # admin bootstrap, data import, backtest, etc.
  tests/
    unit/  integration/  contract/  failure_injection/  golden/
  data/                        # git-ignored: historical candle files
```

Each module contains: `router.py` (if it exposes HTTP), `service.py` (public interface), `repository.py`, `models.py`, `schemas.py`, `events.py`, `errors.py`, and an `__init__.py` that **exports only the public interface**. Other modules import only from that `__init__`.

## 5. Module specifications

Interfaces are indicative. Keep them small and typed.

### 5.1 `core`
Settings (env-driven, validated at start-up, fail fast on missing secrets), async DB session and unit-of-work, **transactional outbox** and a publisher loop to Redis Streams, an idempotent consumer helper, structured logging with redaction, UUIDv7, a `Clock` abstraction (so tests control time), a `Problem` error type that renders RFC 9457 responses.

### 5.2 `identity`
Implements `docs/06` and the auth endpoints in `docs/05` sections 2 and 3: signup, email verification, login, refresh rotation with reuse detection, TOTP, recovery codes, PIN, step-up grants, lockout, sessions, roles. Admin login is a **separate flow** with mandatory TOTP and shorter lifetimes. Provide a CLI command to bootstrap the first super admin using a setup token, disabled once an admin exists. Emails go through an HTTP email provider interface (stubbed locally to print to the log).

### 5.3 `audit`
`record(actor, action, target, metadata)`. Append-only table, revoke `UPDATE` and `DELETE` from the application role. Redact secrets.

### 5.4 `brokers`
`BrokerAdapter` protocol (see `docs/07`), a registry of supported brokers, account linking, credential vault (envelope encryption), `symbol_map`, permission verification, connection test and latency, and the adapters: `PaperBroker`, `BacktestBroker`, `MT5Adapter`. `DerivWsAdapter` is a later phase.

### 5.5 `marketdata`
Candle model and storage, ingestion from the adapter, validation (no gaps, no duplicates, UTC, closed only), backfill, feed health and lag, a `subscribe_closed_candles()` stream, and historical import from CSV for backtests.

### 5.6 `strategy` (the heart)
Implements `docs/02-strategy-spec.md` exactly. See section 6 below for details.

### 5.7 `risk`
Implements `docs/04` section 5: the ordered checks, position sizing, locked min R:R, correlation cap, daily limits, cooldowns, kill-switch state. Produces `ApprovedOrder` or `Rejection(reasons)`.

### 5.8 `execution`
Commands table consumer, order lifecycle state machine, idempotency, submit and monitor orders, position management, SL/TP modification rules, close and close-all, **kill switch** (halt, flatten), and **reconciliation** with the broker. Owns the engine runner loop.

### 5.9 `portfolio`
Positions, trades, P&L, R-multiples, equity snapshots, performance queries, CSV export.

### 5.10 `backtest`
Runs the same `strategy` and `risk` code against a `BacktestBroker`. Reproducible, with a stored parameter snapshot and code version. Runs as a Taskiq job.

### 5.11 `notifications`
Kinds and channels from `docs/04` section 9. In-app first. Preferences and quiet hours. Web push and email as later phases.

### 5.12 `admin`
Admin use cases: list and search users, suspend, unsuspend, force logout, reset 2FA (each with step-up and a reason), platform halt, maintenance mode, system health. No access to broker secrets.

## 6. The strategy engine in detail

### 6.1 Inputs and outputs

```python
def evaluate(
    candles: Sequence[Candle],          # closed candles up to and including "now"
    state: StrategyState,               # zones and their states, last processed candle
    params: StrategyParams,             # validated, per symbol
    market: MarketContext,              # spread, symbol spec, session flags (passed in, not fetched)
) -> EvaluationResult:                  # new state, zone events, optional Signal
```

No hidden inputs. Same inputs always give the same output.

### 6.2 What to implement

Follow `docs/02-strategy-spec.md`:

1. **Indicators:** ATR(period), swing highs and lows (fractal), candle metrics (body, wicks, range ratios).
2. **Zone detection:** base (a cluster of small candles) then departure (large, same-direction, small-wick candles). Classify as DBR, RBD, RBR or DBD, which gives demand or supply. Compute proximal and distal lines.
3. **Four-point qualification:** `big_candles`, `fresh`, `structure_break`, `rr_ok`. Store each check's **measured value** (for example "departure was 2.4x ATR") so the UI can display it.
4. **Zone state machine** per `docs/04` section 3. Every transition produces a stored zone event.
5. **Sweep validation:** penetration beyond the distal line, reclaim within the allowed number of candles, depth limit, breakdown detection. `valid_sweep = qualified AND fresh AND penetration AND reclaim AND depth_ok`.
6. **Confirmation candle:** pin bar, tailed bar, doji, engulfing, with configurable thresholds. Must close inside or beyond the zone in the trade direction, on or after the reclaim candle.
7. **Signal construction:** direction, planned entry (candle close), stop beyond the confirmation candle extreme plus buffer, target at the nearest opposing zone or swing level, planned R:R. No signal if R:R is below 3.0.
8. **Overlapping zones:** keep the better R:R, invalidate the other (record the reason).
9. **One open trade per symbol** is enforced by `risk`, but `strategy` must not emit a signal for a symbol flagged as already in a trade in `market`.

### 6.3 Parameters

All tunables come from the registry in `docs/04` section 7, validated by Pydantic, versioned, with per-symbol overrides. **Nothing is hardcoded.** Anything marked (assumption) is a starting value to be tuned by backtest.

### 6.4 Testing the strategy

- **Golden fixtures:** hand-built candle sequences for each of the four patterns, each of the four checks passing and failing, valid and invalid sweeps (including a breakdown that must not be traded), each confirmation type, R:R rejection, overlapping zones, and a zone that is touched without a sweep (must be invalidated).
- **Property tests (Hypothesis):** output never changes when future candles are appended and earlier candles stay the same (**no look-ahead**); evaluating twice gives the same result; a signal always has SL on the correct side of entry, TP on the correct side, and R:R at least 3.0; zones never move backwards through the state machine.
- **Real-data replay:** a recorded week of XAUUSD 5-minute candles with the expected signals stored as JSON. Changes to the expected output must be reviewed on purpose.
- Coverage target for `strategy`, `risk` and `execution` is very high. Treat uncovered branches in these modules as defects.

## 7. Execution, safety and reconciliation

- **Engine runner:** one asyncio task per (broker account, symbol): wait for a closed candle, validate, evaluate, persist transitions, handle signal (semi-auto creates `pending_approval`, auto goes to risk), submit approved orders, heartbeat. See `docs/04` section 6.
- **Order lifecycle:** `docs/04` section 4.
- **Position management:** monitor positions, mirror broker state, support `PATCH /positions/{id}` under the rule "SL may move toward profit but never further from entry than the original" and flag each change as a manual intervention.
- **Kill switch:** two modes, `halt` (no new entries, existing positions keep their SL/TP) and `flatten` (halt, then close all). The engine acts on a Redis notification **and** polls `commands` every second, so a lost message cannot delay a kill. Target: acknowledged within 2 seconds. The kill path must work even if the strategy or risk modules throw.
- **Reconciliation:** on start-up (before any entry), then periodically (default every 30 seconds): compare broker positions and deals with the database. Repair known differences (a position closed by the broker, a fill that arrived late). On an unexplained mismatch: halt new entries on the account, raise `ReconciliationMismatch`, notify, require acknowledgement.
- **Watchdog and leases:** an engine holds a lease row per account so two engines never trade the same account. A missing heartbeat marks the bot `error` and notifies.
- **Time:** the adapter reports broker time offset. At start-up compare broker time with system time and refuse to trade beyond a tolerance (assumption: 5 seconds).

## 8. API requirements

- Implement `docs/05-api-spec.md`. The **OpenAPI 3.1 document is the contract**. Publish it, keep it accurate, and export it to `backend/openapi.json` in CI so the frontend can generate its client.
- Conventions: `/v1`, RFC 9457 errors with stable `code`s, decimal strings, cursor pagination, `Idempotency-Key` on side-effecting endpoints, rate limits, `403` with no detail for role violations, step-up required code.
- **SSE** at `GET /v1/stream` with named events, `Last-Event-ID` resume, `resync` event, 15-second heartbeat, per-user scoping, and throttled price ticks.
- Health: `/health` (liveness) and `/ready` (database and Redis).
- Every endpoint has request and response schemas, an example, and tests.

## 9. Brokers and the MT5 caveat

- Build `PaperBroker` and `BacktestBroker` first. They must exercise **all error paths** (timeouts, rejects, requotes, partial fills) at configurable rates.
- `MT5Adapter` uses the `MetaTrader5` package, which is **Windows-only** and needs a locally installed, logged-in terminal. Verify wheel availability for the chosen Python version. Verify broker time offset handling, netting versus hedging behaviour, filling modes and symbol suffixes **on a demo account**, and record findings in `docs/07-broker-integration.md`.
- Do not assume anything about Exness, FBS or Deriv beyond what `docs/07` says. Run the verification checklist in `docs/07` section 5 on a demo account.
- Credentials: envelope encryption (AES-256-GCM, per-record data key wrapped by a master key from the environment or a secret manager). Store `key_version`. Provide a re-wrap command.

## 10. Security requirements (summary of `docs/06`)

Argon2id, short-lived JWT access tokens, rotating hashed refresh tokens in httpOnly cookies with reuse detection, mandatory admin 2FA, step-up for sensitive actions, lockout, generic auth errors, rate limiting, CSRF protection for cookie-authenticated writes, strict input validation, parameterised queries, secret redaction in logs, dependency and secret scanning in CI, and an append-only audit log.

## 11. Observability

Structured JSON logs with a correlation ID per candle and per order; metrics for feed latency, candle lag, order latency, rejects, reconciliation mismatches, heartbeat age, risk vetoes by reason; traces from API request to command to engine action; error tracking and alerts for every failure mode in `docs/03` section 9.

## 12. Testing and quality gates

CI must run, and block merges on: `ruff`, type checking, `import-linter` contracts, unit and integration tests (with real Postgres and Redis via containers), contract tests against the OpenAPI document, golden strategy tests, and a dependency vulnerability scan.

**Failure-injection suite** (must pass before Phase 4 ends). Each scenario asserts: zero duplicate orders, no unprotected position, correct final state.

- Broker times out after the order was actually accepted.
- Engine process killed between order submit and database write.
- Duplicate fill events.
- Broker returns partial fill, then reject.
- Feed gap and stale feed.
- Database unavailable during a trade, and during a kill command.
- Redis unavailable.
- Clock skew beyond tolerance.
- Reconciliation finds a position the database does not know about, and the reverse.
- Daily loss limit reached with positions open.

## 13. Development environment

- `make up` starts Postgres and Redis. `make migrate` applies migrations. `make run-api`, `make run-engine`, `make run-worker`, `make test`, `make lint`.
- `.env.example` with **fake** values and comments. Never commit real secrets. Never use a live account in development.
- The engine runs natively when the broker is MT5. Document the Windows set-up in `backend/README.md`.
- Seed command that creates a demo user and paper account, and a command that imports historical candles from CSV.

## 14. Delivery phases

Stop at the end of each phase and report. Do not start the next phase without approval.

### Phase B0. Scaffold
Repo tooling, Docker Compose, config, logging, database, Alembic, outbox and publisher, `Clock`, error type, health endpoints, import-linter contracts, CI.
**Accept when:** CI is green on an empty app, `make up && make run-api` serves `/health` and `/ready`, and an import-boundary violation fails CI (prove it with a deliberate test).

### Phase B1. Identity
Everything in 5.2 and 5.3, plus the admin bootstrap command.
**Accept when:** all auth flows have tests, including refresh-token reuse revoking the family, lockout, step-up expiry, admin 2FA being mandatory, role enforcement on every route, and audit rows written for each sensitive action.

### Phase B2. Strategy core and backtest
Candle model, import of historical CSV for the three symbols, the full `strategy` module, `BacktestBroker`, the backtest runner and report.
**Accept when:** golden and property tests pass, the no-look-ahead test passes, a backtest on real exported data is **bit-for-bit reproducible**, and a report with costs is produced per symbol. Report honestly, including results that look bad.

### Phase B3. Risk, execution, paper
`risk`, `execution`, `PaperBroker`, engine runner, commands, kill switch, heartbeats.
**Accept when:** an end-to-end run on replayed candles opens and closes paper trades, the kill switch works in both modes within the target time, and the failure-injection suite passes.

### Phase B4. Real broker (demo only)
Credential vault, `MT5Adapter`, symbol specs, time normalisation, reconciliation, readiness checklist, broker verification checklist results written to the docs.
**Accept when:** a full trading day on a **demo** account ends with clean reconciliation, SL/TP verified at the broker for every order, and all failure-injection scenarios pass against the adapter where applicable.

### Phase B5. App API
All user endpoints, SSE, notifications, performance and export, published OpenAPI.
**Accept when:** the frontend can run against it, contract tests pass, and the SSE stream resumes correctly after a dropped connection.

### Phase B6. Admin and hardening
Admin API, audit views, platform halt, rate limits, observability, security review against `docs/06`, load and chaos tests.
**Accept when:** the checklist in `docs/06` is fully ticked with evidence.

### Phase B7. Extensions (only if requested)
Deriv adapter, engine leases for several accounts, Google sign-in, passkeys, web push, email.

## 15. How to work

- Ask before guessing when a decision is expensive to reverse (data model changes, public API shapes, anything touching money). Otherwise pick the sensible default, state it, and continue.
- Mark every assumption with `# ASSUMPTION:` in code and list them in the phase report.
- Keep documents current: update `docs/` in the same change as the code.
- Small, reviewable commits with clear messages. No secrets in git history.
- Prefer boring, explicit code over clever code in `strategy`, `risk` and `execution`.
- Do not add features beyond the scope in `docs/01`. Do not add instruments, timeframes or modes.
- Do not claim something works unless a test or a demo-account run shows it.

## 16. Definition of done (whole backend)

- All phases B0 to B6 accepted.
- OpenAPI matches the implementation and drives the frontend client.
- Golden, property, integration and failure-injection suites are green in CI.
- A reproducible backtest report exists for each symbol, with costs and honest conclusions.
- A paper run and a demo-account run have completed with clean reconciliation.
- The kill switch has been exercised in both modes.
- `docs/06` security checklist is ticked.
- The go-live gate in `docs/09` is documented, and nothing in the code enables real-money trading without it.

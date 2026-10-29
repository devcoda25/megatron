# 09. Roadmap

Two tracks, backend (B) and frontend (F), that meet at the API contract. Each phase has an exit criterion. Do not start the next phase until the current one meets it.

The detailed build instructions per phase are in [`backend/MASTER_PROMPT.md`](../backend/MASTER_PROMPT.md) and [`frontend/MASTER_PROMPT.md`](../frontend/MASTER_PROMPT.md).

## Backend

| Phase | Scope | Exit criterion |
|---|---|---|
| **B0 Scaffold** | Repo tooling, Docker Compose (Postgres, Redis), config, logging, database and migrations, event outbox, health endpoints, CI with lint, types, tests and import-boundary check | CI green on an empty app, `docker compose up` gives a healthy stack |
| **B1 Identity** | Signup, verification, login, refresh rotation, TOTP, PIN, step-up, roles, sessions, audit log, admin bootstrap | All auth flows tested, including token reuse and lockout |
| **B2 Strategy core and backtest** | Candle model, zone detection, four-point qualification, sweep validation, confirmation, order construction, backtest runner and simulated broker, historical data import for the three symbols | Deterministic backtest on real exported data. Golden tests for every pattern and rule. No-look-ahead test passes. A first report with costs |
| **B3 Risk, execution, paper** | Risk manager, position sizing, execution module with idempotency and order states, paper broker, engine runner, kill switch, heartbeats | End-to-end run on replayed candles. Failure-injection tests pass with zero duplicate orders |
| **B4 Real broker (demo)** | Credential vault, MT5 adapter, symbol specs, time normalisation, reconciliation, readiness checklist | A full day on a **demo** account with reconciliation clean and broker-side SL/TP verified |
| **B5 App API** | All user endpoints, SSE stream, notifications, performance, exports, OpenAPI published | Frontend can run against it. Contract tests pass |
| **B6 Admin and hardening** | Admin API, audit views, platform halt, observability, rate limits, security review, load and chaos tests | Security checklist ([06](06-security-and-compliance.md)) all ticked |
| **B7 Extensions** | Deriv adapter, multi-account engine leases, Google sign-in, passkeys, web push, email | Chosen by the owner after the go-live gate |

## Frontend

| Phase | Scope | Exit criterion |
|---|---|---|
| **F0 Scaffold** | Next.js app, TypeScript strict, Tailwind with tokens, fonts, icons, lint, tests, CI | Empty shell renders at mobile width with the design tokens |
| **F1 Design system** | Shell (top bar, dock), core components, Storybook | All components in Storybook, reviewed against `DESIGN.md` |
| **F2 Data layer** | Generated API client, query cache, SSE client, same-origin proxy, mock server | Screens can run fully on mocks |
| **F3 Auth** | User auth screens and flows, route protection, session handling, step-up sheet | Sign-in to the dock works against the backend (B1) |
| **F4 Core screens** | Home, Markets, Bot, Trades, Broker, Risk, matched to the references | Visual match to references, driven by real API data (B5) or mocks |
| **F5 Remaining screens** | Signal detail, trade detail, performance, strategy settings, backtest, notifications, profile and security, system states | Every route in [08](08-frontend-design.md) exists with all states |
| **F6 Admin** | Admin auth and panel | Admin flows work with mandatory 2FA, and a normal user cannot reach them |
| **F7 Hardening** | Accessibility audit, end-to-end tests on mobile viewports, performance budget, optional PWA and push | Axe checks clean, e2e green, budgets met |

## Go-live gate (project owner's own account)

Do not enable real-money trading until **all** of these hold:

1. Backtests on at least one year of data per symbol with realistic costs, reviewed by the owner.
2. Paper trading for the agreed period with no unexplained differences (defaults: 14 days, 30 trades, see assumption A6).
3. Demo-account run with a clean reconciliation history.
4. Failure-injection suite green.
5. Kill switch tested live on a demo account (both modes).
6. Broker agreement reviewed for automated scalping.
7. Starting size set to the minimum, and a daily loss limit the owner is comfortable losing in full.

## Immediately next

1. Owner answers the open questions in [10-decisions.md](10-decisions.md), at least Q1, Q2 and Q3.
2. Run B0 and F0 in parallel.
3. Resolve the strategy ambiguities (Q6) before B2 begins.

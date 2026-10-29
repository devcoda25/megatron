# 10. Decisions, Assumptions and Open Questions

## Decisions made

| # | Decision | Why |
|---|---|---|
| D1 | **Modular monolith** with microservice-ready boundaries, run as four processes (api, engine, worker, scheduler) | Fast to build and test locally, safe to split later |
| D2 | **Rule-based engine, no ML.** An LLM may later explain setups, but never places trades | Deterministic, testable, explainable |
| D3 | **Scalp only**, 5-minute chart | Owner decision |
| D4 | **Instruments:** XAUUSD, BTCUSD, GBPUSD only | Owner decision |
| D5 | **Minimum R:R 3:1 is locked** by the system and not user-editable | Strategy rule |
| D6 | **Liquidity sweep is mandatory** for entry | Owner decision |
| D7 | **Python and FastAPI** backend, **Next.js** frontend, mobile-only | Owner decision |
| D8 | **PostgreSQL + Redis** for state and events. SQLAlchemy 2 async, Alembic | Durable state, transactional outbox |
| D9 | **Server-Sent Events** for realtime to the app, REST for commands | Simpler than WebSockets through proxies, enough for one-way streams |
| D10 | **Same-origin API proxy (BFF)** in Next.js and httpOnly cookies for the refresh token | Keeps tokens out of JavaScript storage |
| D11 | **Strategy is a pure function** shared by backtest, paper and live | Test what you trade |
| D12 | **Every order carries a broker-side SL and TP** | Survives crashes |
| D13 | **Kill switch needs confirmation but not re-authentication.** Going **Live** does need step-up authentication | Stopping must never be hard |
| D14 | **Live is gated** by a server-enforced readiness checklist | Prevents premature real-money trading |
| D15 | **Bottom navigation:** Home, Markets, Bot (centre), Trades, Profile (from the six reference screens). `DESIGN.md` lists a different set of labels, and the screens win | Screens are what was approved visually |
| D16 | **Lightweight Charts** for candlesticks with zone overlays | Fits the TradingView-style need |
| D17 | **Taskiq** (Redis broker) for background jobs | Async-native, fits an asyncio codebase |
| D18 | **Broker order of work:** MT5 adapter first (Exness, FBS), Deriv adapter later | MT5 covers spot-style XAUUSD, BTCUSD, GBPUSD positions |

## Assumptions to confirm

These were chosen to keep moving. Change them if they are wrong.

| # | Assumption | Impact if wrong |
|---|---|---|
| A1 | **Postgres in Docker from day one** (not SQLite) | Avoids a later migration. Needs Docker installed. |
| A2 | **Multi-user-ready data model, single-account engine for v1.** Every table carries `user_id`, but acceptance testing runs one account | Opening to other users adds credential isolation and a possible regulatory burden |
| A3 | The project owner's computer or VPS is **Windows**, so the MT5 Python package works | On Mac or Linux the MT5 route needs a Windows host or MetaApi |
| A4 | Strategy numbers in `02-strategy-spec.md` marked (assumption) are starting points and will be tuned by backtest | Results change with them |
| A5 | Per-symbol spread cap is expressed relative to the planned stop distance (default: spread at most 10% of risk distance) | Needs calibration against each broker's real spreads |
| A6 | Go-live gate defaults: at least 14 days of paper trading, at least 30 paper trades, no unresolved reconciliation mismatch | Pure placeholders, to be set by the owner |
| A7 | Notification channels: in-app first, then web push, then email | Order of work only |

## Open questions

| # | Question | Needed for |
|---|---|---|
| Q1 | Which broker is the first one: Exness, FBS or Deriv? | Backend Phase 4 |
| Q2 | Does the owner's chosen broker's client agreement allow this kind of automated trading, and does it allow a third-party service to hold the MT5 login? (One review says Exness warns against sharing the trading password with managed-bot services. Verify in the official agreement.) | Anything beyond own-account trading |
| Q3 | Will other people connect their own accounts, or is this for the owner only? If others, what is the regulatory position in Uganda (and for users elsewhere)? Get legal advice before opening it. | Product scope |
| Q4 | Deriv: use its options API (multipliers), or its MT5 servers for normal positions? They behave differently and need separate backtests | Deriv adapter |
| Q5 | Is there a plan or pricing model? The Stitch profile screen shows a "plan badge" | Profile screen, billing |
| Q6 | Which strategy ambiguities from `02-strategy-spec.md` section 9 should be fixed first (target selection, swing definition, zone drawn on wicks or bodies)? | Backend Phase 2 |
| Q7 | Passkeys (WebAuthn) for "biometric login", and Google sign-in: needed in v1 or later? | Auth scope |
| Q8 | Historical data source for backtests: export from the broker's MT5, or another provider? | Backend Phase 2 |

## Differences found in the design references

These came from comparing the Stitch output with the product decisions. The frontend brief handles each one.

| Finding | Resolution |
|---|---|
| Broker screen lists IC Markets, Pepperstone, Binance and Bybit | Replace with the real broker registry (Exness, FBS, Deriv first). Do not hardcode brands. |
| Copy such as "Zero Slippage", "Hardware Enclave", "institutional FIX", "AES-256 enclave" | Remove or make true. Credentials are encrypted at rest with AES-256-GCM using envelope encryption, but there is no hardware enclave. Slippage cannot be promised. |
| "76% WIN PROB" on zones and "5M WIN RATE" per symbol | Sample data. Show only measured results from real or backtested data with the sample size, or omit. |
| Home mode chip reads "LIVE NET" in cyan, while the design system says LIVE is red with a pulsing dot | Use one Mode pill component everywhere: PAPER cyan, LIVE red with pulsing dot. |
| Reference HTML sets `user-scalable=no` and `maximum-scale=1` | Do not copy. It blocks pinch zoom and fails accessibility guidance. |
| `DESIGN.md` names tabs Dashboard, Charts, Bot Hub, Positions, Logs | Use Home, Markets, Bot, Trades, Profile (see D15). |
| PNG screenshots in the zip are error pages, not images | Only the HTML and `DESIGN.md` were usable. Regenerate screenshots from the HTML if needed. |
| Sample data shows Gold at 2,654 and "Welcome back, Alex" | Placeholder data only. All values come from the API. |

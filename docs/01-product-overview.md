# 01. Product Overview

## What Megatron is

Megatron is a **remote control and monitoring app** for an automated trading engine. The engine scalps three instruments on the 5-minute chart using a supply and demand zone strategy confirmed by a liquidity sweep. The mobile web app lets a trader:

- link a broker account (trade-only permissions),
- turn the bot on or off, and switch between **Paper** and **Live** mode,
- set risk limits that the engine cannot override,
- watch zones, signals, positions and P&L in real time,
- stop everything instantly with a **kill switch**,
- review performance and run backtests.

A **super admin** manages users, can suspend accounts, can halt trading platform-wide, and sees system health and audit logs.

The engine is **rule-based, not machine learning**. It detects zones and applies the checks in [02-strategy-spec.md](02-strategy-spec.md) exactly. This makes it explainable, testable and reproducible. The "AI" in the product name refers to automation, so the product must never imply it predicts prices.

## Users and roles

| Role | Who | Can do | Cannot do |
|---|---|---|---|
| **User** | A trader | Link own broker account, run the bot on own account, set own risk limits, view own data | See other users, see admin screens, change the locked minimum R:R |
| **Super admin** | Platform owner | Manage users (suspend, force logout, reset 2FA), view audit logs and system health, halt all trading | Read broker secrets, place trades for users |

Admins sign in through a **separate entry point** (`/admin`), with mandatory two-factor authentication, short sessions and no self-service sign-up.

## Instruments and timeframe

| Symbol | Asset | Hours |
|---|---|---|
| XAUUSD | Gold | About 24h, Monday to Friday |
| BTCUSD | Bitcoin | 24/7 |
| GBPUSD | Forex | 24h, Monday to Friday |

Timeframe is **5 minutes only**. There are no intraday or swing modes. Any other symbol is rejected.

## Capabilities (v1)

1. **Authentication**: email and password, email verification, optional TOTP 2FA, PIN, password reset, session management. Google sign-in and passkeys are later phases.
2. **Broker linking**: choose broker, enter credentials, verify trade-only permissions, check that the three symbols exist with acceptable spreads.
3. **Bot control**: start, pause, stop, Paper or Live, per-symbol on/off, session filters, kill switch (halt new entries, or halt and close everything).
4. **Strategy engine**: zone detection, four-point qualification, liquidity sweep validation, confirmation candle, order construction with broker-side stop and target.
5. **Risk management**: risk per trade, daily loss limit, max open trades, max trades per day, spread cap, consecutive-loss cooldown, daily profit lock, minimum R:R locked at 3:1.
6. **Markets view**: live candlestick chart with supply and demand zones, sweep markers and a zone scanner with the four-point checklist.
7. **Trades**: open positions with live P&L, modify SL/TP within rules, close, close all, history with R-multiple and close reason, personal notes.
8. **Performance and backtest**: equity curve, win rate, profit factor, drawdown, per instrument and per session. Backtests run on the same strategy code as live trading.
9. **Notifications**: in-app first, then push and email.
10. **Admin**: user management, platform kill switch, audit log, health.

## Non-goals

- No machine-learning price prediction.
- No timeframes other than 5 minutes. No intraday or swing profiles.
- No instruments other than XAUUSD, BTCUSD and GBPUSD.
- No custody of funds. Megatron never needs withdrawal permission and rejects accounts that have it where the broker allows checking.
- No copy trading, signal selling or managing other people's money in v1.
- No native mobile apps and no desktop layout. The frontend is a **mobile-only responsive web app**.

## Product principles

1. **Safety before features.** When uncertain, the engine stops opening trades, alerts the user and keeps protective stops in place.
2. **Explainable trades.** Every trade shows why it happened: the zone, the four checks, the sweep and the confirmation candle.
3. **One code path.** Backtest, paper and live use the same strategy and risk code. Only the broker implementation differs.
4. **The user stays in control.** The kill switch is always one tap away and never needs extra authentication. Going Live does.
5. **Honest numbers.** The app shows measured results with sample sizes. It never shows a win probability it cannot back with data.

## Success criteria for v1

These are engineering targets, to be tuned once measured:

- A backtest on the same data and settings gives the same result every run.
- Failure-injection tests (broker timeout after order submit, engine crash mid-order, duplicate fill events, database outage) produce **zero duplicate orders** and **no unprotected position**.
- Reconciliation detects and reports any difference between the database and the broker within 30 seconds (assumption).
- A kill-switch command is acknowledged by the engine within 2 seconds of being issued (assumption).
- Paper trading runs continuously for at least 14 days with no unexplained trade or state drift (assumption) before Live is allowed for the project owner's own account.

## Glossary

| Term | Meaning |
|---|---|
| **Supply zone** | Price area where selling stopped a rally and price dropped away |
| **Demand zone** | Price area where buying stopped a decline and price rose away |
| **Base** | The small-candle consolidation that forms a zone |
| **Departure** | The strong move away from the base |
| **Proximal / distal line** | The zone edge nearest current price / the far edge |
| **Fresh zone** | A zone price has not returned to since it formed |
| **Liquidity sweep** | Price pokes beyond the zone's far edge to trigger stops, then closes back on the zone side |
| **Confirmation candle** | Pin bar, tailed bar, doji or engulfing bar after the sweep |
| **R:R / R-multiple** | Reward divided by risk. A trade that makes twice its risk is +2R |
| **ATR** | Average True Range, used to size candles and thresholds relative to each instrument |
| **Paper / Live** | Simulated orders against a demo or simulated broker / real orders on a real account |
| **Kill switch** | Emergency stop: halt new entries, optionally close all positions |
| **Reconciliation** | Comparing the database with what the broker reports, and repairing drift |

## Disclaimer

Educational software, not financial advice. Trading involves risk of loss. Past or simulated performance does not guarantee future results.

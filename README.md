# Megatron

Megatron is a mobile-first web app that controls an automated **scalping bot**. The bot trades **XAUUSD, BTCUSD and GBPUSD** on the **5-minute chart** using a **supply and demand zone + liquidity sweep** strategy. The trader links a broker account, switches the bot on or off, sets risk limits, watches live trades and reviews performance. A super admin manages users and the platform.

> **Risk notice.** Trading involves risk and you can lose money. This project is educational software, not financial advice, and no strategy in it is guaranteed to be profitable. Always validate with backtests and paper trading before using real money.

## Repository layout

| Folder | Contents |
|---|---|
| [`docs/`](docs/README.md) | Product, strategy, architecture, backend design, API, security, broker integration, frontend design, roadmap and decisions |
| [`backend/`](backend/MASTER_PROMPT.md) | FastAPI backend (modular monolith) and trading engine. Start with `MASTER_PROMPT.md` |
| [`frontend/`](frontend/MASTER_PROMPT.md) | Next.js mobile-only frontend. Start with `MASTER_PROMPT.md`. Stitch design references are in `design-reference/` |

## Start here

1. Read [`docs/01-product-overview.md`](docs/01-product-overview.md) for what we are building.
2. Read [`docs/02-strategy-spec.md`](docs/02-strategy-spec.md). It is the source of truth for the trading logic.
3. Read [`docs/10-decisions.md`](docs/10-decisions.md) for decisions made, assumptions to confirm and open questions.
4. Build with [`backend/MASTER_PROMPT.md`](backend/MASTER_PROMPT.md) and [`frontend/MASTER_PROMPT.md`](frontend/MASTER_PROMPT.md). Each is a complete brief for a coding agent or developer, split into phases with acceptance criteria.

## Status

Planning and design. No application code yet. The first build step is backend Phase 0 (scaffold) and frontend Phase F0 (scaffold).

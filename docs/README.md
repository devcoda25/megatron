# Megatron Documentation

| # | Document | What it answers |
|---|---|---|
| 01 | [Product overview](01-product-overview.md) | What is Megatron, who uses it, what is in and out of scope |
| 02 | [Strategy spec](02-strategy-spec.md) | The exact trading logic. **Source of truth for the engine** |
| 03 | [System architecture](03-system-architecture.md) | Processes, modules, communication, data flow, deployment, failure modes |
| 04 | [Backend design](04-backend-design.md) | Modules, data model, state machines, events, risk rules, parameter registry |
| 05 | [API specification](05-api-spec.md) | Conventions, endpoints, realtime stream |
| 06 | [Security and compliance](06-security-and-compliance.md) | Auth, secrets, threat model, regulatory and platform notes |
| 07 | [Broker integration](07-broker-integration.md) | Adapter contract, MT5, Deriv, paper and backtest brokers, symbol mapping |
| 08 | [Frontend design](08-frontend-design.md) | Routes, screens, design system, realtime, states, copy rules |
| 09 | [Roadmap](09-roadmap.md) | Phases and exit criteria for backend and frontend |
| 10 | [Decisions and open questions](10-decisions.md) | Decisions made, assumptions to confirm, open questions |

Also:

- [`design/stitch-prompt.md`](design/stitch-prompt.md): the Stitch AI prompt set used to generate the screens.
- [`../frontend/design-reference/`](../frontend/design-reference/): the generated design system (`DESIGN.md`) and six reference screens (HTML).
- [`../backend/MASTER_PROMPT.md`](../backend/MASTER_PROMPT.md) and [`../frontend/MASTER_PROMPT.md`](../frontend/MASTER_PROMPT.md): the build briefs.

## Conventions used in these docs

- **(assumption)** marks a value or choice that is not given by the strategy source or by the project owner. It must be confirmed or tuned, usually through backtesting.
- **TBD** marks something that is not decided yet. The list is in [10-decisions.md](10-decisions.md).
- All times are UTC unless stated. All prices and money amounts are decimals, never binary floats.

## Rule for changing these docs

If code and docs disagree, fix whichever is wrong in the same pull request. Strategy changes start in `02-strategy-spec.md`, then flow into code and tests.

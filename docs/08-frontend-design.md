# 08. Frontend Design

The frontend is a **Next.js, mobile-only** web app. It is a remote control for the engine: it shows state and sends commands, and it never makes trading decisions.

Design sources:

- [`frontend/design-reference/DESIGN.md`](../frontend/design-reference/DESIGN.md): the design system ("Cybernetic Trading Command") with tokens, typography, shapes, components.
- [`frontend/design-reference/screens/`](../frontend/design-reference/screens/): six generated reference screens as HTML.
- [`design/stitch-prompt.md`](design/stitch-prompt.md): the prompts for the screens that are not designed yet.

## 1. Layout rules

- Design width **390 x 844**, supporting **360 to 430** wide. On larger screens, show a centred column no wider than 430 px. No desktop layouts.
- 16 px side margins, 12 px gutters between cards, 4 and 8 px micro spacing.
- Top command bar 52 px, bottom dock 64 px with a raised 56 px centre button. Scroll areas keep an **84 px** bottom buffer.
- Respect safe areas: `env(safe-area-inset-top)` and `env(safe-area-inset-bottom)`.
- Minimum touch target 44 x 44 px. High-frequency actions sit in the lower 45% of the screen.
- Dark theme only in v1. The Profile screen's Dark / Light / System switch is a later feature.

## 2. Route map

```
(auth)                         public, no dock
  /login  /signup  /verify-email  /forgot-password  /reset-password
  /security-setup/2fa  /security-setup/pin
  /welcome (onboarding, 3 slides)

(app)                          signed-in user, with top bar and dock
  /                  Home dashboard                 [designed]
  /markets           Markets and zone scanner       [designed]
  /markets/zones/[id]   Zone and signal detail      [to design]
  /bot               Bot control                    [designed]
  /trades            Open positions and history     [designed]
  /trades/[id]       Trade detail                   [to design]
  /performance       Analytics                      [to design]
  /profile           Profile and menu               [to design]
  /settings/broker   Broker connection              [designed]
  /settings/risk     Risk management                [designed]
  /settings/strategy Scalp strategy settings        [to design]
  /settings/backtest Backtest and paper results     [to design]
  /settings/notifications  Preferences              [to design]
  /settings/security Password, 2FA, PIN, sessions   [to design]
  /notifications     Notification list              [to design]

admin                          super admin only, separate layout
  /admin/login  /admin/2fa  /admin/recovery  /admin/setup
  /admin            Dashboard (users, bots, health) [to design]
  /admin/users      List and detail                 [to design]
  /admin/audit      Audit log                       [to design]
  /admin/system     System health and halt          [to design]

system states (components, not routes): offline, server unreachable,
broker disconnected, bot halted by loss limit, maintenance, empty lists,
session expired, 403, 404
```

Bottom dock: **Home, Markets, Bot (centre, raised), Trades, Profile**. (`DESIGN.md` names them differently; the screens and this list win. See [10](10-decisions.md) D15.)

## 3. Design tokens (from `DESIGN.md`)

| Group | Tokens |
|---|---|
| Surfaces | canvas `#070A12`, card `#0F1524`, elevated `#161E33`, inset `#0B0F1A`, outline `#222B45` |
| Accents | electric `#2DE2FF`, violet `#7B61FF`, gradient `linear-gradient(135deg, #2DE2FF, #7B61FF)` |
| Semantic | profit and buy `#22E58B`, loss and sell `#FF4D6A`, warning and paused `#FFB84D` |
| Text | high `#EAF0FF`, secondary `#8C97B8`, muted `#4A5472` |
| Radius | cards 16 px, inputs 12 px, controls and chips full pill, dock 28 px |
| Type | **Plus Jakarta Sans** for UI, **JetBrains Mono** for every number, with tabular figures. Caption caps 11 px, +0.06em tracking |
| Effects | glass card (80% alpha, blur 16 px, 1 px border), floating layer (92% alpha, blur 24 px), "Megatron glow" on key CTAs, dot-grid canvas background at 15% opacity |

Colour rules: green and red mean profit/loss and buy/sell **only**. Amber means paused or warning. Cyan and violet mark interactive focus and brand.

The reference HTML uses a Tailwind token set (`surface`, `surface-container-low`, `primary-container`, `tertiary-fixed-dim`, and so on). Map those into the project's Tailwind theme so the reference markup can be ported with minimal change.

## 4. Component inventory

Build once, reuse everywhere:

- **Shell:** TopCommandBar (title, ModePill, connection ping, bell), BottomDock (with centre Bot button), Screen scroll container, SafeArea.
- **Status:** ModePill (PAPER cyan, LIVE red with pulsing dot), StatusBadge (Running, Paused, Stopped, Error, Setup required), ConnectionDot with latency, RiskDisclaimer.
- **Surfaces:** GlassCard, BottomSheet, Modal, Toast, Skeleton, EmptyState, ErrorState.
- **Controls:** PrimaryCTA (gradient), GhostButton, DangerButton, KillSwitchButton, Toggle, SegmentedControl, SymbolPills, Chip, Stepper, Slider (with risk zones), PasswordInput with strength meter, OtpInput, PinPad, **SlideToConfirm**.
- **Data:** MetricTile, PnlPill, RMultiple, PriceText (mono, tabular), PositionCard, TradeRow, ZoneCard, ChecklistRow (pass or fail with measured value), TimelineItem, ProgressRing, Sparkline, ProgressToTarget bar.
- **Charts:** CandlestickChart with zone boxes, sweep markers, entry/SL/TP lines; EquityCurve; DailyPnlBars; HourHeatmap.

## 5. Reference screens: data needed

Each designed screen, the data it needs, and where it comes from. All values are from the API. The numbers in the reference HTML are placeholders.

### Home (`/`)

| UI element | Source |
|---|---|
| Greeting (name) | `GET /me` |
| Equity, today's P&L and %, sparkline | `GET /performance?period=today`, SSE `pnl.today` |
| Bot status chip and quick pause/resume | `GET /bot`, SSE `bot.status`; `POST /bot/pause`, `/resume` |
| Win rate, active trades, today's trade count (W/L) | `GET /performance`, `GET /positions` |
| Target Instruments cards (price, change %, ON/PAUSED, current setup label) | `GET /markets/symbols`, SSE `price.tick`, `zone.update` |
| Open Positions preview (max 2) | `GET /positions`, SSE `position.update` |
| Telemetry feed (recent signals) | `GET /signals?limit=5`, SSE `signal.update` |
| Emergency Kill Switch row | opens the kill sheet (see Bot) |

### Markets (`/markets`)

| UI element | Source |
|---|---|
| Symbol selector | the three symbols |
| Quote header (price, change, 24h high/low, spread) | `GET /markets/symbols`, SSE `price.tick` |
| Timeframe pills | 1m, 5m, 15m, 1h, 4h. Default 5m. The engine trades 5m only, the others are for viewing |
| Candlestick chart with supply and demand boxes, FRESH or TESTED tags, sweep marker, entry/SL/TP lines | `GET /markets/{symbol}/candles`, `GET /markets/{symbol}/zones`, SSE `candle.update`, `zone.update` |
| Active Zone Matrix cards and the four-point validation (candle imbalance, freshness, structure shift, min R:R) with measured values | `zones[].checks` |
| "Deploy Bot on this zone", "Configure alert", "Export order flow" | See open items below |

Open items for Markets: "Deploy bot on zone" conflicts with the rule that the engine decides entries itself. Treat it as **"Enable bot on this symbol"**. "Configure alert" creates a notification preference for that zone. "Export order flow" has no backend data source, so omit it in v1.

### Bot (`/bot`)

| UI element | Source |
|---|---|
| Power control with ring, RUNNING/PAUSED/STOPPED, uptime, execution latency | `GET /bot`, SSE `bot.status`, `system.health` |
| PAPER / LIVE switch with warning banner | `POST /bot/mode` (step-up and readiness for Live) |
| Session chips (London, New York, Asia, Crypto 24/7) | `GET /settings/strategy` (per-symbol session filter) |
| Per-symbol cards with status, last signal, state text | `GET /bot`, SSE |
| Active Risk Limits summary and "Edit" | `GET /settings/risk` |
| Pre-Activation Safety Checklist | `GET /bot/readiness` |
| **Kill switch** with confirmation sheet | `POST /bot/kill` with `halt` or `flatten`, slide to confirm |

Show measured per-symbol win rate only if the API provides it with a sample size. Otherwise hide it.

### Trades (`/trades`)

| UI element | Source |
|---|---|
| Tabs Open / History with counts | `GET /positions`, `GET /trades` |
| Floating P&L summary, average R, win rate, margin used | `GET /performance`, `GET /bot` |
| Filter chips (All, Wins, Losses, per symbol) | query params on `/trades` |
| Position cards with progress to TP | `GET /positions`, SSE `position.update` |
| Modify SL/TP | `PATCH /positions/{id}` (server enforces the rules) |
| Close position, Close all | `POST /positions/{id}/close`, `POST /positions/close-all` |
| Execution history rows (P&L, close reason, R, duration, time) | `GET /trades` |

### Broker connection (`/settings/broker`)

| UI element | Source |
|---|---|
| Connected broker card (account masked, balance, leverage, latency) | `GET /broker-accounts/{id}` |
| Test ping, Reconnect, Disconnect | `POST .../test`, `/reconnect`, `DELETE` |
| Trade-only permission notice | static, accurate copy (see copy rules) |
| Symbol availability and spread check | `GET /broker-accounts/{id}/symbols` |
| Broker picker and credentials form, Validate | `GET /brokers`, `POST /broker-accounts` |

The reference shows IC Markets, Pepperstone, Binance and Bybit. Use the **broker registry** from the API instead (first: Exness, FBS, Deriv).

### Risk (`/settings/risk`)

| UI element | Source |
|---|---|
| Baseline balance, risk preview ("max risk per trade", "target at 3.2:1") | computed from `GET /broker-accounts/{id}` and settings, shown as preview only |
| Risk per trade slider and stepper, daily drawdown, max open orders, max daily fills, max spread | `GET/PUT /settings/risk` |
| Min R:R (locked) | read-only |
| Circuit breakers (consecutive losses, cooldown), daily profit auto-lock | `GET/PUT /settings/risk` |
| Save and deploy, Reset to defaults | `PUT`, `POST /settings/risk/reset` |

Raising risk needs step-up authentication. The server validates ranges, and the UI mirrors them.

## 6. Screens still to design

Generate these in Stitch with [`design/stitch-prompt.md`](design/stitch-prompt.md), then port them: splash and onboarding, sign up and log in, verification and security setup, password reset, account states, **admin authentication and admin panel**, signal detail, trade detail, performance, strategy settings, backtest and paper results, notifications, profile and settings, system states.

For the **admin panel** (not yet in the prompt set), design: dashboard (active users, running bots, engine and feed health, alerts), users list with search and detail with suspend/force logout/reset 2FA, audit log with filters, system health with platform halt, and maintenance toggle. Same design system, with the ADMIN badge and a more serious tone.

## 7. Realtime and data handling

- One SSE connection (`/api/v1/stream`) opened after sign-in. Reconnect with backoff and `Last-Event-ID`.
- Server state lives in a query cache. SSE events **update the cache**, so screens stay simple.
- Show a visible **stale data** indicator when the stream is down, and fall back to polling.
- Throttle UI updates for price ticks (for example 4 per second) to keep the main thread free.
- **Critical actions** (start, Live, kill, close) are never optimistic. Show a pending state until the server confirms, then show the result.
- Money and prices are decimal strings. Do arithmetic with a decimal library. Format for display only.

## 8. States every screen needs

Loading skeleton, empty, error with retry, offline, **stale stream**, session expired, forbidden, maintenance. The system-wide mode pill (PAPER or LIVE) is always visible in the top bar.

## 9. Copy rules

- Never promise results: no "guaranteed", "risk-free", "AI predicts", or win probabilities without data and sample size.
- Be accurate about security: credentials are encrypted at rest, and Megatron asks for trade-only permission. Do not claim hardware enclaves, "zero slippage", or "institutional" infrastructure.
- Show the risk notice ("Trading involves risk. Not financial advice.") on Home, Bot activation and Broker.
- Live mode copy must be unmistakable. Going Live uses the slide-to-confirm sheet and the "I understand the risk" checkbox.
- Use plain, calm wording for errors. Say what happened and what the user can do.

## 10. Accessibility

- Contrast: check muted text on dark surfaces against WCAG AA, and adjust tokens if needed.
- Visible focus states, labels for every control, and `aria-live` regions for status changes (bot state, kill switch result).
- **Do not disable pinch zoom.** Do not copy `user-scalable=no` or `maximum-scale=1` from the reference HTML.
- Respect `prefers-reduced-motion`: turn off pulsing and glow animations.
- Do not rely on colour alone: pair green and red with arrows, signs or labels.

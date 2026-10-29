# MEGATRON FRONTEND: MASTER PROMPT

**How to use this file.** It is the complete build brief for the Megatron frontend. Give it to a coding agent (or follow it yourself), together with the repo. Work **one phase at a time**. At the end of each phase, stop, show what was built (screenshots at 390 px width), list assumptions and deviations, and wait for approval.

---

## 0. Your role and mission

You are a senior frontend engineer with strong visual and accessibility skills. Build the **Megatron frontend**: a **Next.js, mobile-only** web app that is the remote control and monitoring dashboard for an automated scalping bot. It must look and feel like the supplied design ("Cybernetic Trading Command"): a futuristic, precise command center in the palm of the hand.

The frontend **shows state and sends commands**. It never makes trading decisions and never calculates anything that the server is the authority on.

## 1. Read these first

| File | Why |
|---|---|
| `docs/01-product-overview.md` | What the product is, roles, scope |
| `docs/08-frontend-design.md` | **Routes, tokens, components, per-screen data, states, copy and accessibility rules** |
| `frontend/design-reference/DESIGN.md` | The design system: colours, type, spacing, shapes, components |
| `frontend/design-reference/screens/*.html` | Six generated reference screens: Home, Markets, Bot, Trades, Broker, Risk |
| `docs/05-api-spec.md` | The API you consume, and the SSE stream |
| `docs/06-security-and-compliance.md` | Auth, cookies, step-up |
| `docs/design/stitch-prompt.md` | Prompts describing the screens that are not designed yet |
| `docs/10-decisions.md` | Known differences between the design references and the product rules |

If these disagree, `docs/08` and `docs/10` win over the reference HTML. If something is still unclear, ask.

## 2. Non-negotiable rules

1. **Mobile only.** Design width 390 x 844, supporting 360 to 430. On wider screens show a **centred column no wider than 430 px**. Build no desktop layout.
2. **Match the design system exactly.** Use the tokens from `DESIGN.md`. **No hardcoded colours or font sizes in components**: everything comes from theme tokens.
3. **The server is the source of truth.** The UI never computes win rate, P&L, R-multiple, zone validity, readiness or limits for decisions. It may display a preview (for example "max risk per trade") labelled as a preview.
4. **Critical actions are never optimistic.** Start, go Live, kill switch, close position, close all, link broker: show a pending state and update only after the server confirms.
5. **LIVE mode is always unmistakable.** One `ModePill` component used everywhere: PAPER in cyan, LIVE in red with a pulsing dot.
6. **Kill switch is always reachable** (Home and Bot) and requires a **slide-to-confirm** sheet with two options: stop new trades only, or stop and close all positions. It never asks for a password.
7. **Going Live** requires the step-up sheet (password, TOTP or PIN), the readiness checklist passing, the "I understand the risk" checkbox, and slide-to-confirm.
8. **Money and prices are decimal strings.** Do arithmetic with a decimal library, never `parseFloat`. Format for display only, with tabular monospace numbers.
9. **Tokens stay out of JavaScript storage.** No `localStorage` or `sessionStorage` for tokens. Use the same-origin proxy and httpOnly cookies (section 5).
10. **Role gating in the UI is cosmetic.** The server enforces roles. A normal user never sees admin links, and the admin area handles `403` gracefully.
11. **No fake data in production builds.** Mocks exist only for development, tests and Storybook.
12. **Accessibility is a requirement.** WCAG AA contrast, 44 px touch targets, visible focus, labels, `aria-live` for status changes, `prefers-reduced-motion` respected, and **pinch zoom must stay enabled** (do not copy `user-scalable=no` or `maximum-scale=1` from the reference HTML).
13. **Honest copy** (section 9).

## 3. Tech stack

Verify the **current stable version** of each when you scaffold, pin versions, and record them in `frontend/README.md`.

| Area | Choice |
|---|---|
| Framework | Next.js (App Router), React, TypeScript in **strict** mode |
| Styling | Tailwind CSS with a theme generated from `DESIGN.md` tokens (CSS variables for colour so a palette change is one file) |
| Components | Radix UI primitives (or equivalent headless library) for sheets, dialogs, sliders, switches, tabs, tooltips, with our own styling. Do not adopt a full visual kit that fights the design system |
| Server state | TanStack Query |
| UI state | Zustand (small, local to features) |
| Forms | React Hook Form with Zod schemas |
| API client | **Generated from the backend's OpenAPI** (`backend/openapi.json`) using `openapi-typescript` plus a thin fetch wrapper, or Orval. Never hand-write request types |
| Realtime | A small SSE client over `EventSource`-style streaming with reconnect, backoff and `Last-Event-ID`. Because auth is cookie-based, `EventSource` works with the same-origin proxy |
| Charts | **TradingView Lightweight Charts** for candlesticks and equity curves, with a custom series primitive (plugin) for zone boxes, sweep markers and level lines. Check the installed version's docs for the primitives API. Small custom SVG for sparklines, progress rings and heatmap. Dynamic-import chart code so it does not weigh down the first load |
| Decimals | `decimal.js` (or `big.js`) |
| Icons | Material Symbols Outlined (as in the reference). **Self-host a subset**, do not load Google Fonts at runtime |
| Fonts | Plus Jakarta Sans and JetBrains Mono through `next/font`, tabular numerals enabled |
| Tests | Vitest and Testing Library, Playwright (mobile viewports), `axe-core` checks, MSW for mocks, Storybook with interaction tests |
| Quality | ESLint, Prettier, `tsc --noEmit`, `knip` or similar for dead code, bundle-size budget in CI |
| Optional later | PWA manifest and service worker, Web Push |

## 4. Project structure

```
frontend/
  package.json
  README.md
  next.config.ts            # rewrites /api/* to the FastAPI origin, security headers
  tailwind / theme files    # generated from design tokens
  .env.example
  design-reference/         # DESIGN.md and reference HTML (do not edit, copy from them)
  public/
  src/
    app/
      (auth)/               # login, signup, verify-email, forgot/reset, security-setup, welcome
      (app)/                # layout with TopCommandBar and BottomDock
        page.tsx            # Home
        markets/ bot/ trades/ performance/ profile/ notifications/
        settings/ broker/ risk/ strategy/ backtest/ notifications/ security/
      admin/                # separate layout, admin auth and panel
      not-found.tsx  forbidden.tsx  error.tsx  maintenance/
    components/
      ui/                   # buttons, badges, cards, sheets, inputs, SlideToConfirm, PinPad ...
      trading/              # PositionCard, ZoneCard, ChecklistRow, PnlPill, ModePill ...
      charts/               # CandlestickChart (+ zone primitive), EquityCurve, Sparkline, Heatmap
      shell/                # TopCommandBar, BottomDock, SafeArea
    features/               # one folder per feature: queries, hooks, local state, feature components
    lib/
      api/                  # generated client, fetch wrapper, problem+json handling
      realtime/             # SSE client and cache bridge
      format/               # money, price, percent, time (display only)
      decimal/              # decimal helpers
      auth/                 # session helpers, step-up
    styles/                 # tokens.css, globals.css
    mocks/                  # MSW handlers and fixtures (dev and tests only)
  tests/ e2e/
  .storybook/
```

## 5. Architecture rules

- **Same-origin proxy (BFF).** The browser calls `/api/*`, and Next.js rewrites it to the FastAPI service. This keeps cookies same-site, avoids CORS and keeps tokens in httpOnly cookies that JavaScript cannot read.
- **Auth model.** Short-lived access token and rotating refresh token, both handled by cookies set by the backend. The client never reads tokens. On a `401`, call the refresh endpoint once, then retry. If refresh fails, go to sign-in. Handle `step_up_required` (`403`) by opening the step-up sheet, then retrying the original request.
- **Route protection.** Use middleware or layout checks to redirect signed-out users, and a separate guard for `/admin`. This is a convenience. The server decides.
- **Data flow.** Pages read from TanStack Query. The **SSE client updates the query cache** (for example, `position.update` patches the positions query), so screens stay declarative. Show a visible "stale" indicator when the stream is down and fall back to polling.
- **Throttling.** Price ticks update the UI at a limited rate (for example 4 per second). Keep the main thread free.
- **Errors.** Parse RFC 9457 problem responses and map `code` to friendly messages. Never show raw server text for unknown codes.
- **Server vs client components.** Keep screens as thin as practical. Anything that touches the stream, charts or gestures is a client component. Dynamic-import heavy pieces.
- **Environment.** `NEXT_PUBLIC_*` only for non-secret values. No secrets in the frontend.
- **Security headers** in `next.config`: CSP, frame protection, `X-Content-Type-Options`, strict referrer policy.

## 6. Design system implementation

Build the design system **before** the screens. Source of truth: `design-reference/DESIGN.md`.

- Generate the Tailwind theme and CSS variables from the tokens: surfaces, accents, semantic colours, text levels, radii, spacing, typography scale (including the monospace telemetry styles), shadows and the "Megatron glow".
- The reference HTML uses a Tailwind token set (`surface`, `surface-container-low`, `primary-container`, `tertiary-fixed-dim`, `font-display-hero-mono`, and so on). Define the same names so markup from the reference ports across with little change.
- Implement the **shell**: TopCommandBar (52 px, title, ModePill, connection ping with latency, notification bell) and the BottomDock (64 px, frosted glass, five stations Home, Markets, **Bot** raised centre button, Trades, Profile). Scroll containers keep an **84 px** bottom buffer. Respect safe-area insets.
- Implement the component inventory in `docs/08` section 4, each with all states (default, pressed, disabled, loading, error) and a Storybook story. Include `SlideToConfirm`, `PinPad`, `OtpInput`, `PasswordInput` with strength meter, `Slider` with risk zones, and `ChecklistRow`.
- Effects: glass cards, dot-grid background, glow, pulsing status dots. Wrap animation in `prefers-reduced-motion` checks.
- Numbers always use the mono font with tabular figures. Caption labels use the uppercase tracked style from `DESIGN.md`.

## 7. Screens

### 7.1 Designed screens (port from the reference HTML and wire to real data)

Per-screen data sources are in `docs/08` section 5. Follow them. Summary:

| Route | Reference | Key points |
|---|---|---|
| `/` Home | `home_dashboard.html` | Equity and today's P&L hero, status chip with quick pause or resume, win rate, active and today's counts, three instrument cards, up to two open positions, telemetry feed, kill-switch row, risk disclaimer |
| `/markets` | `markets_zone_scanner.html` | Symbol pills, quote header, timeframe pills, candlestick chart with supply and demand boxes, FRESH or TESTED tags, sweep marker, entry, stop and target lines, zone matrix cards with the four-point validation and measured values |
| `/bot` | `bot_control.html` | Power control with ring, PAPER or LIVE switch with warning, session chips, per-symbol cards, risk limit summary, readiness checklist, kill switch |
| `/trades` | `trades_positions.html` | Open and History tabs, P&L summary, filter chips, position cards with progress to TP, modify SL/TP, close, close all, execution history |
| `/settings/broker` | `broker_connection.html` | Connected card, test, reconnect, disconnect, trade-only notice, symbol availability and spread check, broker picker and credentials form |
| `/settings/risk` | `risk_management.html` | Risk preview, risk-per-trade slider and stepper, daily drawdown, max orders, max fills, locked min R:R, spread cap, loss-streak cooldown, daily profit lock, save, reset |

Deviations from the reference that are **required** (see `docs/10`):

- Broker list comes from `GET /brokers` (first Exness, FBS, Deriv). Remove IC Markets, Pepperstone, Binance and Bybit and any brand-specific text.
- Remove or correct claims: "Zero Slippage", "Hardware Enclave", "institutional FIX", "AES-256 enclave". Use the accurate copy in section 9.
- Win-probability and per-symbol win-rate figures appear only if the API returns them **with a sample size**. Otherwise hide them.
- Use one `ModePill`. The Home "LIVE NET" chip becomes the standard pill.
- On Markets, "Deploy Bot on this zone" becomes **"Enable bot on this symbol"**. "Configure alert" creates a notification preference. Omit "Export order flow".
- Names, balances, prices and trades in the reference are placeholders. All values come from the API.
- Do not copy `user-scalable=no` or `maximum-scale=1`.

### 7.2 Screens to build from the design system and the prompts

No reference exists yet. Generate them in Stitch with `docs/design/stitch-prompt.md` if available, otherwise build them directly in the same design system, following the prompt text as the spec:

splash and onboarding, sign up and log in, verification and security setup (OTP, 2FA with QR, PIN, biometric later), password reset, account states (verification pending, locked, suspended, session expired, new device), signal detail, trade detail with notes, performance, strategy settings (scalp profile, confirmation toggles, advanced section, approval mode), backtest and paper results, notifications and notification settings, profile and security (sessions list, change password, 2FA, PIN, export data, legal, log out, delete account), and the **system states** (offline, server unreachable, broker disconnected, halted by loss limit, maintenance, empty lists, 403, 404).

### 7.3 Admin area

A separate layout with a red-amber **SUPER ADMIN** badge and a darker, more serious look in the same design system.

- **Admin auth:** `/admin/login` (no social login, no sign-up), `/admin/2fa` (mandatory TOTP with countdown ring), `/admin/recovery`, `/admin/setup` (first-time: account, authenticator QR, save recovery codes, done), password reset and change, session-timeout warning sheet, re-authentication sheet for sensitive actions, new-login alert, locked state, active sessions.
- **Admin panel:** dashboard (active users, running bots, engine and feed health, alerts), users list with search and detail (suspend, unsuspend, force logout, reset 2FA, each with a reason and re-authentication), audit log with filters, system health with **platform halt**, maintenance toggle.
- A normal user hitting `/admin` gets the Access Denied screen with no admin details.

## 8. States every screen must handle

Loading (skeletons), empty, error with retry, offline, **stale stream**, session expired (PIN or biometric re-entry), forbidden, maintenance, and broker disconnected. The ModePill is visible in the top bar at all times.

## 9. Copy rules

- Never promise results. No "guaranteed", "risk-free", "AI predicts", and no win probability without data and sample size.
- Security wording must be true: "Credentials are encrypted at rest. Megatron needs trade-only permission and does not need withdrawal access." No claims of hardware enclaves, zero slippage or institutional infrastructure.
- Show "Trading involves risk. Not financial advice." on Home, Bot activation and Broker, and on the sign-up consent.
- Live-mode confirmations are explicit: what will happen, which account, which symbols, which limits.
- Error messages say what happened and what to do next, in calm plain language.

## 10. Testing and quality gates

CI blocks merges on: lint, `tsc --noEmit`, unit tests, Storybook interaction tests, Playwright e2e at **iPhone and Android viewports**, `axe` accessibility checks on every route, and a bundle-size budget.

**Playwright flows to cover** (against mocks, then against the backend when available):

- Sign up, verify, sign in, 2FA, refresh after expiry, sign out.
- Start bot in Paper, pause, resume, stop.
- Switch to Live: blocked when readiness fails, step-up required, slide-to-confirm, success.
- Kill switch in both modes, with the slide gesture.
- Close a position and close all, with pending and confirmed states.
- Edit risk limits (and step-up when raising risk), reset to defaults.
- Link a broker, test connection, disconnect.
- SSE drop: stale indicator appears, polling fallback works, state resyncs.
- Offline and server-unreachable states.
- Admin: normal user is denied, admin login with 2FA, suspend a user with re-authentication.

Unit-test the formatters and decimal helpers heavily (rounding, negative values, very large and small numbers, no float drift).

## 11. Performance and robustness

- First-load JavaScript per route stays within a budget you set and record (target: Home well under typical mobile budgets). Dynamic-import charts.
- Virtualise long lists (trade history, notifications).
- Avoid layout shift: reserve chart and card heights, use skeletons.
- Use the platform: native `dialog` or Radix for sheets, CSS for animations where possible.
- Test on a throttled mid-range Android profile.

## 12. Delivery phases

Stop at the end of each phase and report with screenshots at 390 px.

### Phase F0. Scaffold
Next.js, TypeScript strict, Tailwind with tokens from `DESIGN.md`, fonts, self-hosted icons, lint and test tooling, CI, a placeholder page in the centred 430 px column.
**Accept when:** the shell renders with correct tokens and fonts at 390 px, CI is green.

### Phase F1. Design system
Shell (TopCommandBar, BottomDock), every component in `docs/08` section 4, Storybook with stories for all states.
**Accept when:** components are reviewed against `DESIGN.md` side by side with the reference screens, axe checks pass in Storybook, and no component contains a hardcoded colour.

### Phase F2. Data layer
Generated API client, fetch wrapper with problem+json and refresh and step-up handling, SSE client bridged to the query cache, same-origin proxy, MSW mocks.
**Accept when:** the six designed screens' data shapes can run on mocks, SSE reconnect and resync are covered by tests.

### Phase F3. Auth
User auth screens and flows, route protection, session handling, step-up sheet, PIN and OTP components in use.
**Accept when:** the full sign-in flow works against mocks and, once available, against backend Phase B1.

### Phase F4. Core screens
Home, Markets, Bot, Trades, Broker, Risk, matched to the reference screens, with the required deviations in section 7.1.
**Accept when:** side-by-side visual review against the references passes, every critical action follows the non-optimistic rule, kill switch and Live flows work end to end on mocks, and the e2e flows for them pass.

### Phase F5. Remaining user screens
Everything in 7.2.
**Accept when:** every route in `docs/08` exists with all states, and axe is clean on each.

### Phase F6. Admin
Everything in 7.3.
**Accept when:** admin flows work with mandatory 2FA, a normal user cannot reach them, and sensitive actions require re-authentication.

### Phase F7. Hardening
Accessibility audit with a screen reader pass, performance budget check on a throttled device, e2e suite complete, optional PWA and push, security header review.
**Accept when:** budgets are met, e2e is green on both viewports, and the checklist in `docs/08` section 10 is complete.

## 13. How to work

- Ask before guessing when the answer changes visible behaviour on money-related screens. Otherwise choose a sensible default, state it, and continue.
- Port from the reference HTML, then refactor into components. Do not paste the reference markup wholesale.
- Keep `docs/08` current in the same change as the code (routes, components, data sources).
- Mark assumptions with `// ASSUMPTION:` and list them in the phase report.
- Do not add screens, features or instruments beyond `docs/01`. Do not add a desktop layout.
- Do not claim a screen is done until it has all states, passes axe and has an e2e or interaction test.

## 14. Definition of done (whole frontend)

- Phases F0 to F6 accepted (F7 before any real-money use).
- All routes in `docs/08` exist and match the design system.
- No hardcoded colours, no tokens in browser storage, no optimistic critical actions.
- Kill switch, Live gating, step-up and admin separation verified by e2e tests.
- Accessibility checks clean, performance budget met on a throttled mobile profile.
- Copy follows section 9 everywhere.

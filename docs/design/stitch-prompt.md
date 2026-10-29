# MEGATRON: Master Prompt for Stitch AI (Mobile UI)

How to use: paste **Part 1 (Master Prompt)** first, and attach the Behance reference image so Stitch can extract the color palette. Then paste each screen from **Part 2** one at a time (or in small groups), so every screen stays consistent with the system.

---

## PART 1: MASTER PROMPT (paste this first)

```
Design a complete mobile app UI called "MEGATRON": an automated trading bot controller for retail traders. The app is a remote control and monitoring dashboard for an AI-assisted trading engine that scalps XAUUSD (Gold), BTCUSD (Bitcoin) and GBPUSD on the 5-minute chart using a supply and demand zone + liquidity sweep strategy. The user links their real broker account, switches the bot on or off, sets risk limits, watches live trades, and reviews performance. It must feel powerful, precise, premium and trustworthy: like a futuristic command center in the palm of your hand.

PLATFORM AND FRAME
- Mobile only. Design at 390 x 844 (iPhone-class), portrait. Must also stay clean at 360 x 800 (small Android) and 430 x 932.
- Built later in Next.js as a mobile-responsive web app, so use standard web-friendly patterns (no desktop layouts, no hover-only interactions).
- Respect safe areas (status bar top, home indicator bottom). Side margins 16px. 4/8px spacing grid.
- Minimum touch target 44 x 44px. One-handed use: primary actions reachable in the bottom half of the screen.

COLOR AND STYLE
- Use the attached reference image as the primary source for the color palette, mood and visual language. Extract its background, surface, accent and gradient colors and apply them consistently.
- If a color cannot be taken from the reference, use this fallback dark theme:
  - Background: #070A12 (near black blue)
  - Surface / cards: #0F1524, elevated surface #161E33
  - Border / divider: #222B45
  - Primary accent (Megatron glow): electric cyan-blue #2DE2FF, with a gradient to violet #7B61FF for hero elements and the main call-to-action
  - Profit / buy / demand: #22E58B
  - Loss / sell / supply: #FF4D6A
  - Warning: #FFB84D
  - Text primary #EAF0FF, secondary #8C97B8, disabled #4A5472
- Dark theme is the default. Style: sleek, futuristic, subtle neon glow on key elements, soft glassmorphism on cards (blur + thin 1px border), rounded corners (16px cards, 12px buttons, full-pill chips), smooth gradients. Do not overdo effects: data must always be readable.
- Green and red are reserved for profit/loss and buy/sell. Never use them for decoration.

TYPOGRAPHY
- Clean geometric sans-serif for UI (like Inter or Plus Jakarta Sans), and a tabular/monospace-style numeral font for prices, P&L and lot sizes so digits align.
- Scale: Display 32/700, Title 22/700, Heading 18/600, Body 15/400, Caption 12/500. Big bold numbers for balance and P&L.

NAVIGATION
- Bottom tab bar with 5 tabs: Home, Markets, Bot (center, emphasized with a glowing circular button), Trades, Profile. Active tab uses the accent color with a glow; others use secondary text.
- Top app bar per screen: title on the left, a connection/status indicator and notification bell on the right.
- A persistent, always-reachable KILL SWITCH ("Stop all trading") on the Bot screen and as a quick action on Home. It must be red, clear and need a confirmation sheet.

SYSTEM-WIDE UI RULES
- A global MODE BADGE in the top bar: "PAPER" (blue) or "LIVE" (red, pulsing dot). Live mode must always be visibly obvious.
- A bot status chip everywhere it matters: Running (green pulse), Paused (amber), Stopped (grey), Error (red), Waiting for setup (blue).
- Include loading skeletons, empty states, error states and offline/disconnected states for data screens.
- Use bottom sheets for confirmations and quick edits instead of full pages where possible.
- Include a small "Trading involves risk. Not financial advice." disclaimer on Home, Bot activation and Broker screens.
- Charts: candlestick charts with dark grid, green/red candles, and supply (red translucent box) and demand (green translucent box) zone overlays, entry/stop/target lines with labels.

USER ROLES AND AUTH
- Two roles: NORMAL USER (trader who links a broker and runs the bot) and SUPER ADMIN (platform owner who manages users and the system).
- Normal user auth is friendly and fast: email + password, Google sign-in, optional 2FA, biometric and PIN.
- Super admin auth is visibly different and stricter: separate entry point (/admin/login), a red-amber "ADMIN" badge, a darker more serious look with the same brand glow, mandatory two-factor authentication, no social sign-in, no self-service sign-up, and short session timeouts. A normal user must never see admin screens or admin links.
- All auth screens share the same components, spacing and typography as the rest of the app.

COMPONENT LIBRARY (design these once, reuse everywhere)
Buttons (primary gradient, secondary outline, danger, ghost, icon), chips, status badges, cards, list rows, toggles, sliders, segmented controls, steppers, inputs with validation, OTP input, bottom sheet, modal, toast, tab bar, top bar, symbol selector pills (XAUUSD / BTCUSD / GBPUSD), P&L pill, sparkline, candlestick chart, equity curve chart, bar chart, progress ring, timeline item, empty-state illustration.

DELIVERABLE
Produce all screens listed next as one cohesive, consistent mobile design system, with the same components, spacing, color and tone across every screen.
```

---

## PART 2: SCREEN-BY-SCREEN PROMPTS (paste one at a time)

### A. Onboarding and Auth

**1. Splash**
```
Megatron splash screen. Dark background with a subtle animated-looking energy grid. Centered glowing Megatron logomark (abstract geometric "M" with candlestick motif) and wordmark. Tagline: "Trade with precision. Automated." Small loading indicator at the bottom.
```

**2. Onboarding (3 swipeable slides)**
```
Three onboarding slides with illustrations, page dots, Skip and Next. Slide 1: "Your strategy, automated" (supply and demand zones on a chart). Slide 2: "Secure by design" (broker linked with trade-only permissions, no withdrawals). Slide 3: "You stay in control" (kill switch, risk limits). Final slide has a gradient "Get started" button and a "I already have an account" link.
```

**3. Sign up / Log in**
```
Sign up and Log in screen with a segmented switch. Fields: email, password (show/hide), confirm password for sign up. Buttons: Continue (gradient), Continue with Google. Terms and risk disclaimer checkbox on sign up. "Forgot password?" link. Clean, centered, thumb-friendly.
```

**4. Verify and secure**
```
Screens: (a) 6-digit OTP verification with resend timer; (b) Set up two-factor authentication (authenticator app, QR code and manual key); (c) Enable biometric login (Face ID / fingerprint) prompt; (d) Set an app PIN (numeric keypad with dots). Consistent progress indicator showing security setup steps.
```

**5. Forgot / reset password**
```
Forgot password flow: enter email screen, check-your-email confirmation screen, set new password screen with password strength meter.
```

### A2. Normal user: extra auth states

**5b. Account states (normal user)**
```
Set of normal-user auth state screens: (a) Email verification pending with Resend and Change email; (b) Account locked after too many failed attempts, with an unlock timer and "Reset password" button; (c) Account suspended by admin, showing a short reason and "Contact support" button; (d) Session expired, asking for PIN or biometric to continue; (e) Log in on a new device, asking for an email code; (f) Logged-out success screen. Same style as the user login.
```

### A3. Super admin auth (stricter, separate entry)

**5c. Super admin login**
```
Super Admin login screen, reached from a separate admin entry point. Darker, more serious look than the user login, same Megatron brand. A red-amber "SUPER ADMIN" badge and a shield icon at the top. Fields: admin email, password (show/hide). No Google sign-in, no sign-up link. Primary button "Sign in securely". Small text: "Authorized personnel only. All access is logged." Footer link "Back to Megatron" for normal users. Error state for invalid credentials without revealing which field was wrong.
```

**5d. Admin two-factor verification (mandatory)**
```
Admin two-factor screen after the password step. 6-digit authenticator code input with auto-advance boxes, a countdown ring showing the code window, "Use recovery code instead" link, and a "Trust this device for 8 hours" toggle that is off by default. Error and attempts-remaining state (for example "2 attempts left"). Shows the signed-in email (masked) at the top.
```

**5e. Admin recovery code**
```
Recovery code entry screen for admins who lost their authenticator: single input for a recovery code (format XXXX-XXXX-XXXX), clear warning that each code works once, Verify button, and a "Back to authenticator code" link. Include a success state telling the admin to set up a new authenticator.
```

**5f. First-time super admin setup**
```
First-time super admin setup flow with a 4-step progress bar: (1) Create admin account: name, email, strong password with live strength meter and rules checklist; (2) Set up authenticator: QR code plus manual key and a code confirmation field; (3) Save recovery codes: grid of 8 one-time codes with Copy and Download buttons and a required checkbox "I have saved these codes"; (4) Done: success screen with "Go to admin dashboard".
```

**5g. Admin password reset and change**
```
Admin password flow: (a) Reset request: email field, with a note that a verification code is also required; (b) Verify identity: email code plus authenticator code; (c) Set new password with strength meter and a notice that all other sessions will be signed out; (d) Change password from inside admin settings (current password, new password, confirm). Strict, calm tone.
```

**5h. Admin session and security prompts**
```
Admin security state screens: (a) Session timeout warning bottom sheet "You will be signed out in 60 seconds" with Stay signed in and Sign out; (b) Re-authentication sheet for sensitive actions (asks for password and authenticator code before actions like suspending a user or stopping all bots); (c) New login alert screen showing device, location, IP and time with "This was me" and "Secure my account"; (d) Admin account locked after failed attempts with a longer timer and recovery option; (e) Active sessions list with device, last active time and a Sign out button per session.
```

**5i. Access control screens**
```
Access screens: (a) "Access denied" (403) shown when a normal user tries to open an admin page, with a Back to home button and no admin details exposed; (b) "Page not found" (404); (c) Role badge component showing USER or SUPER ADMIN in the profile header. Same dark style.
```

### B. Core

**6. Home (Dashboard)**
```
Home dashboard. Top: greeting, MODE badge (PAPER/LIVE), notification bell. Hero card: account equity (large number), today's P&L in $ and %, tiny sparkline, and bot status chip with a quick Start/Pause toggle. Row of 3 stat cards: Win rate, Open trades, Today's trades. "Instruments" horizontal row with 3 cards for XAUUSD, BTCUSD, GBPUSD: price, 24h change, bot on/off per symbol. "Open positions" preview (max 2 rows) with live P&L and a "View all" link. "Recent signals" list. A red "Stop all trading" quick action. Risk disclaimer at the bottom. Bottom tab bar.
```

**7. Markets**
```
Markets screen. Segmented symbol selector: XAUUSD, BTCUSD, GBPUSD. Large live price header with change %. Candlestick chart (5m default) with timeframe pills (1m, 5m, 15m, 1h), supply zones in translucent red and demand zones in translucent green, a "fresh zone" tag on untested zones, and sweep markers (small arrow icons) where a liquidity sweep occurred. Below the chart: Zone scanner card list (zone type, price range, status chips: Fresh / Qualified / Waiting for sweep / Invalid, R:R value). Tapping a zone opens a bottom sheet with its 4-point checklist (Candle size, Freshness, Structure break, R:R at least 3:1) as check/cross rows.
```

**8. Zone / Signal detail (bottom sheet or screen)**
```
Signal detail screen for a setup on XAUUSD: mini chart with the zone box, sweep marker, confirmation candle (pin bar), entry, stop loss and take profit lines. Details list: direction, entry, stop, target, risk:reward (3.2:1), risk amount, lot size, confirmation type (Pin bar / Tailed bar / Doji / Engulfing), status timeline (Zone qualified, Price retested, Liquidity sweep, Confirmation candle, Order placed). Buttons: Close detail; if pending approval mode is on, Approve and Reject.
```

**9. Bot control (center tab)**
```
Bot control screen, the command center. Huge glowing activation control (circular power button with progress ring) showing state: Running, Paused or Stopped. Mode switch PAPER / LIVE with a warning when switching to LIVE. Per-instrument toggles for XAUUSD, BTCUSD, GBPUSD, each with status chip and last signal time. Session filter chips (London, New York, Asia, 24/7 for BTC). Summary of active risk limits (risk per trade, max daily loss, max open trades). Big red "KILL SWITCH: Stop all trading and close positions" button with a confirmation bottom sheet that offers: Stop new trades only, or Stop and close all positions. Pre-activation checklist card (Broker connected, Risk limits set, Paper test done) with green ticks or warnings. Disclaimer text.
```

**10. Activate bot confirmation (bottom sheet)**
```
Confirmation bottom sheet before going LIVE: shows account, balance, symbols enabled, risk per trade, max daily loss, a checkbox "I understand that trading involves risk and I can lose money", and a slide-to-confirm control "Slide to go LIVE".
```

**11. Trades (tabs: Open / History)**
```
Trades screen with a segmented control: Open and History. Open tab: cards per trade with symbol, buy/sell badge, lots, entry, current price, live P&L (colored), stop and take-profit progress bar, and a "Close trade" button (with confirm). History tab: filter chips (symbol, win/loss, date range), list of closed trades with P&L, R-multiple, duration, close reason (TP hit / SL hit / Manual / Kill switch). Summary strip at top: total P&L, win rate, average R.
```

**12. Trade detail**
```
Trade detail screen: header with symbol, direction, result badge and P&L. Chart snapshot with entry, stop, target and the zone box. Info grid: open and close time, entry, exit, SL, TP, lot size, risk $, R-multiple, spread, slippage. Strategy tags that triggered the trade (Fresh zone, Structure break, Sweep valid, Pin bar). Notes field for a personal journal entry.
```

**13. Performance / Analytics**
```
Performance screen. Period pills (Today, Week, Month, All). Equity curve chart. KPI grid: Net profit, Win rate, Profit factor, Average R:R, Max drawdown, Total trades. Bar chart of P&L per day. Breakdown cards per instrument (XAUUSD, BTCUSD, GBPUSD) and per session. Simple heatmap of win rate by hour of day.
```

### C. Setup and Settings

**14. Broker connection**
```
Broker connection screen. Step flow: (1) Choose broker (searchable list with logos), (2) Enter account login / server or API key and secret with secure input fields, (3) Test connection. Prominent security note card: "Use trade-only permissions. Withdrawals must be disabled. Keys are encrypted." Connected state shows account number (masked), balance, leverage, a green "Connected" status, latency, last sync time, and buttons Reconnect and Disconnect. Error state shows a clear message and Retry. Include a "Symbols available on this broker" check for XAUUSD, BTCUSD, GBPUSD with ticks.
```

**15. Risk management**
```
Risk settings screen with sliders and steppers: Risk per trade (%), Max daily loss (% or $), Max open trades total, Max trades per day, Max allowed spread per symbol, Minimum risk:reward (locked default 3:1), Daily profit target (optional auto-stop). Toggle: "Pause bot after consecutive losses" with a stepper. A live preview card: "With a $10,000 balance you risk $100 per trade". Save button sticky at the bottom.
```

**16. Strategy settings (Scalp only)**
```
Strategy screen showing the fixed profile "Scalp, 5-minute". Read-friendly cards explaining the entry rules as a visual checklist: Zone qualification (Candle size, Freshness, Structure break, Min R:R 3:1), Liquidity sweep required, Confirmation candle types (toggles for Pin bar, Tailed bar, Doji, Engulfing). Advanced section (collapsed): sweep reclaim candles, max sweep depth, stop buffer, per-symbol overrides for XAUUSD, BTCUSD, GBPUSD. Toggle "Require my approval before each trade" (semi-auto mode). A "Reset to defaults" link.
```

**17. Backtest and Paper results**
```
Backtest screen: choose symbol, date range, and Run button. Results card with equity curve, win rate, profit factor, max drawdown, number of trades, and a list of sample trades. A banner: "Past performance does not guarantee future results." Also a Paper trading status card showing days run, paper P&L, and a "Ready for LIVE?" checklist.
```

**18. Notifications and Alerts**
```
Notifications screen with grouped list (Today, Earlier): trade opened, trade closed (with P&L), zone ready, sweep detected, daily loss limit hit, broker disconnected, bot error. Filter chips. Notification settings screen with toggles per type and channel (push, email), plus quiet hours.
```

**19. Profile and Settings**
```
Profile screen: avatar, name, email, plan badge. Menu sections: Account, Broker, Security (password, 2FA, biometric, PIN, active sessions), Notifications, Appearance (Dark / Light / System), Data (export trade history CSV), Legal (Terms, Privacy, Risk disclosure), Help and Support, Log out (red), Delete account (danger).
```

**20. System states**
```
Set of reusable state screens: no internet, server unreachable, broker disconnected (with Reconnect), bot stopped by daily loss limit (explain why, show reset time), maintenance mode, empty trades list, empty notifications, session expired (re-enter PIN or biometric).
```

---

## PART 3: EXTRA INSTRUCTIONS TO ADD AT THE END (optional)

```
Keep every screen visually consistent. Reuse the same top bar, tab bar, cards and buttons. Show realistic sample data: XAUUSD around 2,650.00, BTCUSD around 68,000, GBPUSD around 1.2950, with believable P&L numbers, a mix of wins and losses, and a $10,000 demo balance. Use real-looking trade rows, not lorem ipsum. Avoid clutter: one primary action per screen. Make the Bot control screen and Home the most visually striking.
```

---

## Notes for building in Next.js later

- Auth and roles: enforce roles on the server (FastAPI), not just by hiding screens. Admin routes live under `/admin`, require the `super_admin` role, mandatory 2FA, short-lived sessions, rate limiting and an audit log of admin actions. Never rely on the UI alone for access control.

- Mobile-first layout, max width about 430px, centered on larger screens.
- Use CSS variables for the color tokens above so the palette can be adjusted after extracting the reference colors.
- The UI is only a remote control. The live data and trade decisions come from the FastAPI engine.

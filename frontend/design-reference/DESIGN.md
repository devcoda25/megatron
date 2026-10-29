---
name: Cybernetic Trading Command
colors:
  surface: '#0d1322'
  surface-dim: '#0d1322'
  surface-bright: '#333949'
  surface-container-lowest: '#080e1c'
  surface-container-low: '#151b2a'
  surface-container: '#191f2f'
  surface-container-high: '#242a3a'
  surface-container-highest: '#2f3445'
  on-surface: '#dde2f7'
  on-surface-variant: '#bac9cd'
  inverse-surface: '#dde2f7'
  inverse-on-surface: '#2a3040'
  outline: '#859397'
  outline-variant: '#3b494c'
  surface-tint: '#14d9f6'
  primary: '#c2f3ff'
  on-primary: '#00363e'
  primary-container: '#2de2ff'
  on-primary-container: '#00616f'
  inverse-primary: '#006877'
  secondary: '#c9bfff'
  on-secondary: '#2e009c'
  secondary-container: '#4720ca'
  on-secondary-container: '#baaeff'
  tertiary: '#a0ffc0'
  on-tertiary: '#00391e'
  tertiary-container: '#2eeb90'
  on-tertiary-container: '#006539'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#a3eeff'
  primary-fixed-dim: '#14d9f6'
  on-primary-fixed: '#001f25'
  on-primary-fixed-variant: '#004e5a'
  secondary-fixed: '#e5deff'
  secondary-fixed-dim: '#c9bfff'
  on-secondary-fixed: '#1a0063'
  on-secondary-fixed-variant: '#441cc8'
  tertiary-fixed: '#58ffa5'
  tertiary-fixed-dim: '#1ce289'
  on-tertiary-fixed: '#00210f'
  on-tertiary-fixed-variant: '#00522e'
  background: '#0d1322'
  on-background: '#dde2f7'
  surface-variant: '#2f3445'
typography:
  display-hero:
    fontFamily: Plus Jakarta Sans
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
  display-hero-mono:
    fontFamily: JetBrains Mono
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
  title-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 22px
    fontWeight: '700'
    lineHeight: 28px
  title-mono:
    fontFamily: JetBrains Mono
    fontSize: 22px
    fontWeight: '700'
    lineHeight: 28px
  headline-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '600'
    lineHeight: 24px
  body-default:
    fontFamily: Plus Jakarta Sans
    fontSize: 15px
    fontWeight: '400'
    lineHeight: 22px
  body-medium:
    fontFamily: Plus Jakarta Sans
    fontSize: 15px
    fontWeight: '500'
    lineHeight: 22px
  body-mono:
    fontFamily: JetBrains Mono
    fontSize: 15px
    fontWeight: '500'
    lineHeight: 22px
  label-regular:
    fontFamily: Plus Jakarta Sans
    fontSize: 13px
    fontWeight: '500'
    lineHeight: 18px
  caption-caps:
    fontFamily: JetBrains Mono
    fontSize: 11px
    fontWeight: '600'
    lineHeight: 14px
  caption-ui:
    fontFamily: Plus Jakarta Sans
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
  telemetry-sm:
    fontFamily: JetBrains Mono
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  gutter: 0.75rem
  margin: 1rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 0.75rem
  space-lg: 1rem
  space-xl: 1.5rem
---

## Brand & Style

This design system expresses a high-velocity, precision-engineered cybernetic aesthetic built specifically for autonomous algorithmic trading. The visual tone is tactical, mission-critical, and authoritative—bridging the gap between aerospace telemetry and institutional quantitative execution. 

Key attributes:
- **Atmospheric Depth:** Deep-space navy and obsidian backdrops (#070A12) establish focused low-light contrast, eliminating cognitive fatigue during volatile market monitoring.
- **Controlled Luminescence:** Neon accents (electric cyan #2DE2FF and hyper-violet #7B61FF) are applied strictly to interactive focal points, execution paths, and agent status rings, preventing visual clutter while conveying machine intelligence.
- **Zero-Latency Telemetry:** Monospace typographic grids ensure numbers align to fixed optical widths, allowing rapid vertical scanning of fluctuating P&L, lot sizes, and order book states.
- **Mission Criticality:** Critical triggers—such as emergency liquidation protocols and master kill switches—utilize distinct high-consequence patterns requiring verified multi-step intent.

## Colors

The palette operates under strict dark-mode constraints, reserving pure saturation exclusively for telemetry status and execution signals.

### Surface System
- **Canvas Base:** `#070A12` — Deep obsidian-blue void that recedes into the hardware bezel.
- **Card Surface (Base Tier):** `#0F1524` — Primary module containers and card surfaces.
- **Elevated Surface (Top Tier):** `#161E33` — Bottom sheets, floating action modals, and active tap states.
- **Glass Inset / Recessed Surface:** `#0B0F1A` — Inset metrics tracks, empty chart gutters, and slider tracks.
- **Structural Outlines:** `#222B45` — Hairline 1px borders defining card perimeters, segment dividers, and table splits.

### Core Signals & Semantic Tokens
- **Electric Accent:** `#2DE2FF` — Primary action items, connection states, selected states, and brand signatures.
- **Hyper-Violet Accent:** `#7B61FF` — Secondary accent applied as the end-stop in diagonal execution gradients (`linear-gradient(135deg, #2DE2FF 0%, #7B61FF 100%)`).
- **Demand / Profit / Bullish:** `#22E58B` — Strictly allocated to positive P&L values, buy executions, long orders, and active running engine loops.
- **Supply / Loss / Bearish:** `#FF4D6A` — Strictly allocated to negative drawdowns, sell executions, short triggers, and critical errors.
- **Operational Amber:** `#FFB84D` — Standby warnings, API rate-limit throttles, and bot paused states.

### Typography & Content Levels
- **Text High-Contrast:** `#EAF0FF` — Hero figures, currency tickers, screen titles, primary value labels.
- **Text Standard / Secondary:** `#8C97B8` — Table keys, field placeholders, subheadings, unit abbreviations.
- **Text Muted / Low-Priority:** `#4A5472` — Timestamps, inactive tabs, disabled control states, gridlines.

## Typography

The type system is divided cleanly across two specialized typefaces:
- **Structural Text (Plus Jakarta Sans):** Handles general interface navigation, card titles, status narratives, and operational prompts.
- **Monospace Telemetry (JetBrains Mono):** Mandated for all numerical data feeds, currency amounts, order book levels, leverage ratios, trade sizes, P&L percentages, and UTC execution timestamps. Tabular metrics prevent layout jumping across active WebSockets.

### Typography Rules
- All numerical figures must activate tabular numbers (`tnum`) to preserve fixed vertical column alignment.
- Hero metrics above 22px must feature an optical character spacing adjustment of -0.02em to preserve density.
- Tactical metadata labels (such as `ENTRY`, `MARK`, `24H VOL`, `SL/TP`) must render in `caption-caps` using uppercase tracking (+0.06em) for rapid skimming.

## Layout & Spacing

Designed around a native iPhone-class target (390×844px), the layout adheres to strict ergonomic thumb-zone zoning. High-frequency interactions (bot activation, safety cancelations, tab changes) occupy the bottom 45% of the viewport. Observational telemetry (live balance, master chart preview, connection status) occupies the upper vertical zone.

### Mobile Grid & Layout Principles
- **Screen Margins:** Fixed 16px (`1rem`) horizontal margin anchors cards flush with safe areas.
- **Component Gutters:** 12px (`0.75rem`) gutters between vertical card stacks and two-column statistical modules.
- **Micro Spacing Scale:**
  - `space-xs` (4px): Metric label-to-value stack gap.
  - `space-sm` (8px): Icon-to-text inline label spacing; badge horizontal padding.
  - `space-md` (12px): Internal card cell padding; row spacing in order blocks.
  - `space-lg` (16px): Standard internal card padding; bottom sheet header gap.
  - `space-xl` (24px): Distinct macro section gaps; clearance above bottom action bars.
- **Scroll Padding:** Bottom scroll containers enforce an 84px content buffer to prevent occlusion by the elevated navigation dock.

## Elevation & Depth

Visual hierarchy is maintained via a cybernetic glassmorphism framework that preserves dark readability without clouding data legibility.

### Surface Tiers & Blurs
- **Level 0 (Base Grid):** `#070A12` with a subtle 24px CSS background dot grid rendered in `#222B45` at 15% opacity to evoke tactical plotting interfaces.
- **Level 1 (Card Glass):** Background `#0F1524` with 80% alpha, `backdrop-filter: blur(16px)`, bound by a crisp 1px perimeter border of `#222B45`.
- **Level 2 (Active/Floating Overlays):** Background `#161E33` with 92% alpha, `backdrop-filter: blur(24px)`, 1px border of `#313D60`, cast with an ambient drop shadow: `0 12px 32px -4px rgba(0, 0, 0, 0.6)`.

### Dynamic Luminescence & Light Blooms
- **The Megatron Glow:** Critical CTA nodes and running bot cores utilize a specialized multi-layer drop glow: `0 0 24px rgba(45, 226, 255, 0.35), 0 0 8px rgba(123, 97, 255, 0.45)`.
- **Pulsing Engine Indicators:** Active status dots cast an animated radial glow ring extending 6px outwards in `#22E58B` with CSS opacity animation cycling between 0.3 and 0.8.

## Shapes

The geometric architecture pairs softened rectangular containers with high-precision pill elements, balancing technical efficiency with tactile feel.

### Shape Scale
- **Cards & Data Modules:** 16px corner radius (`rounded-lg`). Creates a defined boundary for nested telemetry modules.
- **Input Fields & Tactical Tiles:** 12px corner radius (`rounded-md`). Internal modules nested inside cards maintain consistent concentric spacing (16px outer - 4px inset = 12px inner).
- **Controls & Status Chips:** 9999px fully rounded pill containers (`rounded-full`). All status indicators, segmented tabs, and floating triggers use full capsule curvature.
- **Floating Action Dock:** 28px continuous curvature along bottom bar perimeters, with a dedicated central negative-space cutout accommodating the circular 56px Bot Command trigger.

## Components

### Top Command Bar
- **Dimensions:** 52px fixed height, positioning below the device status bar.
- **Components:** Left-aligned screen title (`title-lg`), accompanied by a persistent Global Mode pill:
  - **PAPER:** `#2DE2FF` text over `rgba(45, 226, 255, 0.12)` pill border `#2DE2FF`.
  - **LIVE:** `#FF4D6A` text with a 6px pulsing dot over `rgba(255, 77, 106, 0.15)` pill border `#FF4D6A`.
- **Controls:** Right side hosts a low-latency connection status indicator (green telemetry dot showing ms ping) alongside a notification bell triggering alert overlays.

### Cybernetic Navigation Dock (Bottom Bar)
- **Geometry:** 64px tall frosted glass bar (`#0F1524` at 85% opacity, `backdrop-filter: blur(20px)`), docked above the home indicator.
- **Tab Layout:** 5 distinct stations: Dashboard, Charts, [Center Bot Hub], Positions, Logs.
- **Center Hero Station:** A 56px elevated circular button floating 18px above the nav horizon. Filled with an angled gradient (`#2DE2FF` to `#7B61FF`), bound by a 2px outer cyan rim and active megatron glow. Tapping opens bot orchestration routines.

### Cards & Modules
- Glass surfaces with `#0F1524` base and 1px border `#222B45`.
- **Header:** Accommodates ticker pair, bot name, and top-right status pill.
- **Body:** Split into two vertical halves: left for key telemetry metrics (P&L, Entry, Size), right for sparkline trends or micro execution rings.

### Pill-Shaped Status Badges
- Height: 24px; padding: 0 10px; font: `caption-caps`.
- **Running:** Text `#22E58B`, background `rgba(34, 229, 139, 0.12)`, 1px border `rgba(34, 229, 139, 0.3)`. Left side includes a 4px green pulsing light indicator.
- **Paused:** Text `#FFB84D`, background `rgba(255, 184, 77, 0.12)`, border `rgba(255, 184, 77, 0.3)`.
- **Stopped:** Text `#8C97B8`, background `rgba(140, 151, 184, 0.1)`, border `#222B45`.
- **Error:** Text `#FF4D6A`, background `rgba(255, 77, 106, 0.15)`, border `rgba(255, 77, 106, 0.4)`.

### Buttons & Tactical Controls
- **Hero Execution CTA:** Full-width 50px pill button. Background: `linear-gradient(135deg, #2DE2FF 0%, #7B61FF 100%)`. Text: `#070A12`, font weight 700. Pressed state dims brightness to 90% and increases glow spread.
- **Secondary Ghost Controls:** 44px pill button. Background: transparent; 1px border `#222B45`; text `#EAF0FF`. Active state shifts border to `#2DE2FF`.
- **Kill Switch Button:** High-prominence, high-contrast red action button. Background: `linear-gradient(180deg, #FF4D6A 0%, #C42642 100%)`. Label: `EMERGENCY KILL SWITCH` in `caption-caps` bold text. Tapping opens a specialized slider confirmation sheet ("Slide to Terminate All Open Positions") to avoid unintended triggers.

### Sparklines & Candlestick Zones
- In-card sparklines are rendered as 2px stroke vectors with an underlying gradient fill terminating to 0% opacity at the card floor.
- Profit sparklines use `#22E58B`; loss sparklines use `#FF4D6A`.
- Zone overlays on order sheets represent Demand zones with `#22E58B` fills at 8% opacity with dashed borders; Supply zones utilize `#FF4D6A` at 8% opacity.

### Progress Rings & Sliders
- Concentric circular SVG meters displaying bot CPU usage, margin usage, and risk tolerance thresholds.
- Unfilled track: `#161E33` with 1px border `#222B45`.
- Filled arc: Gradient stroke from `#2DE2FF` to `#7B61FF` with a rounded head cap.
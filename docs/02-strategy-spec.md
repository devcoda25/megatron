# Supply & Demand Liquidity-Sweep Scalping Strategy

Source: converted from a video transcript. **The bot is scalp-only.** There are no intraday or swing modes, so no mode switch is needed.

This document is educational material and not financial advice. The source itself states this is a framework to practice and test, not a guaranteed system.

---

## 1. Overview

| Item | Value |
|---|---|
| Timeframe | 5-minute chart only, no top-down analysis |
| Markets (the video) | Forex, indices, crypto, futures |
| Instruments (this bot) | **XAUUSD, BTCUSD, GBPUSD only** |
| Style | Scalping, quick trades |
| Core idea | Trade fresh supply/demand zones only after a liquidity sweep and a rejection candle |
| Trade management | Set and forget: the trade ends at take-profit or stop-loss |
| Min risk:reward | 3:1 |

Pipeline:

1. Detect supply and demand zones.
2. Qualify each zone with the four-point checklist.
3. Wait for price to return to a qualified zone.
4. Wait for a **valid** liquidity sweep of the zone (section 4.2). No valid sweep, no trade.
5. Wait for a confirmation candle after the sweep.
6. Enter, set stop and target, then leave the trade alone.

---

## 2. Zone Patterns

Every zone comes from one of four base patterns: a move, a small consolidation (the base), then another move.

| Pattern | Sequence | Type | Meaning |
|---|---|---|---|
| Drop-Base-Rally (DBR) | down, base, up | Demand | Bullish reversal |
| Rally-Base-Drop (RBD) | up, base, down | Supply | Bearish reversal |
| Rally-Base-Rally (RBR) | up, base, up | Demand | Bullish continuation |
| Drop-Base-Drop (DBD) | down, base, down | Supply | Bearish continuation |

- **Demand zone:** area where buying pressure stopped a decline. Forms from DBR or RBR.
- **Supply zone:** area where selling pressure stopped a rally. Forms from RBD or DBD.
- The zone is drawn around the **base**.
- The trade is taken when price **returns (retests)** the zone, never by chasing the move away from it.

---

## 3. Zone Qualification Checklist (all four required)

If any one criterion fails, the zone is ignored.

1. **Candle size of the move.** The move leaving the base is made of large candles, mostly the same color, with small wicks. Small, mixed-color, wicky candles mean the zone is skipped.
2. **Freshness.** The zone has never been touched since it formed. The first retest is the entry. A zone already tested once is discarded.
3. **Breakout of structure.** The move out of the zone breaks a key structure level, such as a previous high or low (or a previous opposing zone).
4. **Risk:reward of at least 3:1.** Measured from the planned entry and stop to the next opposing level. Below 3:1, skip the trade even if the zone looks strong.

---

## 4. Liquidity Sweep (second confirmation)

Do not enter when price first touches the zone. Wait for the sweep.

**Demand zone (buy setup):**
1. Price returns to the zone.
2. Price dips below the zone, taking out the stops sitting under it.
3. Price closes back above, or back inside, the zone.

**Supply zone (sell setup):**
1. Price returns to the zone.
2. Price spikes above the zone, taking out the stops sitting above it.
3. Price closes back below, or back inside, the zone.

Rationale in the source: stop-losses and pending orders cluster near zones, and large players sweep them to get fills before reversing.

### 4.1 Liquidity sweep is a required entry condition

A qualified zone alone is never an entry. **No valid sweep, no trade.** The sweep plus the confirmation candle (section 5) together form the entry signal.

### 4.2 What makes a sweep valid

A sweep is valid only if all of these hold:

1. **Qualified zone:** it happens at a zone that passed all four checks in section 3, on its first retest (fresh).
2. **Penetration:** price trades beyond the distal line (below for demand, above for supply). This is the stop run.
3. **Reclaim:** a candle closes back inside or beyond the zone in the trade direction, so price is back on the zone side of the distal line.
4. **Rejection:** the reclaiming candle (or the one right after it) is a confirmation candle from section 5. In the video's examples the pin bar or tailed bar is often the sweep candle itself.
5. **Order:** the sweep comes first, then the confirmation. A confirmation candle without a preceding sweep is not a signal.

### 4.3 What is NOT a valid sweep (zone breakdown)

- A candle **closes beyond the distal line and stays there**: this is a breakout, not a sweep. The zone is invalidated and must not be traded.
- The zone is not fresh, or fails any of the four checks.
- The sweep happens but no confirmation candle follows, so skip it.
- Risk:reward drops below 3:1 once the stop is placed beyond the sweep wick, so skip it.

### 4.4 Validity thresholds (assumptions, tune in backtesting)

These numbers are not in the video:

- Reclaim must happen within 1 to 3 candles of the penetration.
- Maximum penetration depth beyond the distal line: about 1x ATR(14). A deeper move is treated as a breakdown.
- Minimum penetration: at least a small buffer beyond the line (more than spread), so a simple touch does not count.

---

## 5. Confirmation Candle

After the sweep, one of these candles must form (rejection of the swept side):

| Candle | Description |
|---|---|
| Pin bar | Small body, long wick on the swept side. Bullish at demand, bearish at supply. |
| Tailed bar | Small body, long tail on the swept side. The body closes back above/inside the zone (demand) or below/inside the zone (supply). |
| Doji | Treated as a type of tailed bar. |
| Engulfing bar | Mentioned in the step summary as an accepted confirmation. |

Note: the source describes pin bar and tailed bar in detail and lists engulfing only in the summary, so engulfing criteria are not specified.

---

## 6. Trade Execution

| | Buy (demand) | Sell (supply) |
|---|---|---|
| Entry | At the close of the confirmation candle | At the close of the confirmation candle |
| Stop loss | Just below the confirmation candle's wick low | Just above the confirmation candle's wick high |
| Take profit | Next resistance level | Next support level (or opposing demand zone) |

- Management is set and forget. The trade ends at take-profit or stop-loss with no manual intervention.
- Skip the trade if the distance to target gives less than 3:1 against the stop.

---

## 7. Worked Examples from the Source

1. **Demand + pin bar:** qualified zone, sweep below, close back inside, pin bar, enter at pin bar close, stop below wick, target next resistance. Hit target.
2. **Supply + bearish pin bar:** strong bearish move from the zone broke a previous demand zone, price swept above then closed back below, enter at close, stop above wick, target next support. Hit target.
3. **Demand + tailed bar:** qualified zone, sweep below, tailed bar with long lower tail, enter at close, stop below the tail low, target next resistance. Hit target.
4. **Rejected demand zone:** candles, freshness and structure break all passed, but R:R was under 3:1, so the zone was ignored.

---

## 8. Algorithmic Rules (for the bot)

The source is discretionary. The rules below turn it into testable logic. **Items marked (assumption) are not in the source and must be tuned or backtested.**

### 8.1 Zone detection
- Base: a cluster of N consecutive small-range candles (assumption: N = 1 to 6, each range below 0.5x ATR(14)).
- Departure: the candle(s) after the base with a combined body larger than a threshold (assumption: at least 1.5x ATR(14)), with most candles in the same direction and small wicks (assumption: wick under 30% of the candle range).
- Direction of the departure and the move into the base decide the pattern (DBR/RBD/RBR/DBD).
- Zone boundaries: **proximal** line (nearest to current price) and **distal** line (far edge), taken from the base candles' high/low or body (assumption: use highs/lows).

### 8.2 Qualification
- `big_candles`: departure meets the size and wick tests above.
- `fresh`: no candle after zone formation has traded into the zone.
- `structure_break`: departure closes beyond the most recent swing high (demand) or swing low (supply). Swing definition (assumption): fractal of 5 candles.
- `rr_ok`: (distance from entry to next opposing level) / (distance from entry to stop) >= 3.
- Zone is tradable only if all four are true.

### 8.3 Sweep detection (implements section 4)
- `penetration`: demand: candle low < distal line - min_buffer; supply: candle high > distal line + min_buffer.
- `reclaim`: demand: a candle closes above the distal line within `max_reclaim_candles` (assumption: 3); supply: mirror image.
- `depth_ok`: penetration depth <= `max_sweep_depth` (assumption: 1x ATR(14)); deeper means breakdown.
- `breakdown`: any close beyond the distal line that is not reclaimed within the window invalidates the zone permanently.
- `valid_sweep = zone_qualified AND zone_fresh AND penetration AND reclaim AND depth_ok`.
- Entry is allowed only when `valid_sweep` is true AND a confirmation candle (8.4) closes after or on the reclaim candle.

### 8.4 Confirmation detection (assumption thresholds)
- Pin bar / tailed bar: the wick on the swept side is at least 60% of the candle's total range, and the body is at most 30% of the range.
- Doji: body at most 10% of range.
- Engulfing: body fully engulfs the previous candle's body in the trade direction.
- Confirmation candle must close inside or beyond the zone in the trade direction.

### 8.5 Order construction
- Entry: market order at the confirmation candle close (or at next candle open).
- Stop: beyond the confirmation candle's extreme plus a buffer for spread (assumption: spread + small ATR fraction).
- Target: nearest opposing zone or swing level. Skip if R:R < 3.
- Position size: fixed % of account risked per trade (assumption: 0.5% to 1%).

### 8.6 Bot-level safeguards (not in source)
- One open trade per symbol.
- Max trades per day and max daily loss, then stop trading until the next day.
- Avoid trading during high-impact news and wide-spread periods.
- Mark a zone as used once a trade is taken or the zone is touched.
- Invalidate a zone if price closes beyond the distal line without a sweep-and-reclaim.

---

## 9. Ambiguities to Resolve Before Coding

- Exact definition of "large" candles and "small" wicks.
- How a base is identified (candle count, size limits).
- Whether zones are drawn on wicks or bodies.
- What counts as a "key structure level" for the breakout test.
- How to pick the target when several opposing levels exist.
- Whether a sweep must occur within a set number of candles of the retest.
- Whether the engulfing bar needs extra rules.
- Handling of overlapping zones and of a new signal while already in a trade.
- The video says "set and forget", but a live bot should still hold a broker-side stop-loss and respect risk limits.

---

## 10. Scope: Scalp Only

The bot trades one profile only. Settings are fixed in config, not switchable per user.

| Parameter | Value |
|---|---|
| Zone/entry timeframe | 5m |
| Min R:R | 3:1 |
| Hold time | Minutes |

### Instruments

The bot trades only these three symbols. Any other symbol is rejected.

| Symbol | Asset | Trading hours | Notes |
|---|---|---|---|
| XAUUSD | Gold | About 24h, Mon to Fri | Volatile, spreads widen around news and rollover |
| BTCUSD | Bitcoin | 24/7 | Trades on weekends, when spreads and liquidity can differ |
| GBPUSD | Forex | 24h, Mon to Fri | Most active during London and New York sessions |

Each symbol needs its own config, because price scale, volatility and costs differ:

- ATR multiples for base, departure and sweep depth (section 8).
- Maximum allowed spread: skip a setup if the spread is above the limit.
- Stop buffer, tick size, contract size and lot step.
- Session or time filter (for example, avoid rollover, high-impact news and the thin hours).
- Risk per trade and daily loss limit, with position size computed per symbol from the stop distance.

Run each symbol as an independent instance of the same logic (own zones, own state), with a global cap on open trades and total daily loss across all three. Treat the three as correlated, since XAU and GBP both move with the USD, and BTC can move with risk sentiment.

Intraday and swing trading are out of scope. Because scalping is sensitive to spreads, slippage and execution latency, backtests must include realistic costs and the broker must offer low spreads and fast order execution.

---

## 11. Next Steps

1. Resolve the ambiguities in section 9 and fix the parameter values.
2. Implement zone detection and the four-point filter (`zones.py`).
3. Backtest on historical 5-minute data for one market, with realistic spread and slippage.
4. Paper trade on a broker demo account.
5. Add the risk manager, then go live with small size.

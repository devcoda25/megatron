# 07. Broker Integration

## 1. Goal

The strategy, risk and execution code must not know which broker is in use. Everything broker-specific hides behind a `BrokerAdapter` interface, with these implementations:

| Adapter | Used for | Notes |
|---|---|---|
| `PaperBroker` | Paper mode and local development | Simulates fills against a real or replayed price feed, with spread and slippage |
| `BacktestBroker` | Backtests | Same behaviour as `PaperBroker`, fed with historical candles |
| `MT5Adapter` | Live and demo accounts at MT5 brokers (Exness, FBS, Deriv's MT5 servers) | First real adapter |
| `DerivWsAdapter` | Deriv's own API | Later, and only after deciding which Deriv product (see below) |

## 2. Adapter contract

Async, typed, and free of strategy logic. Method names are indicative:

```python
class BrokerAdapter(Protocol):
    async def connect(self) -> None: ...
    async def disconnect(self) -> None: ...
    async def account_info(self) -> AccountInfo: ...            # balance, equity, margin, leverage, currency
    async def verify_permissions(self) -> PermissionReport: ...  # trade-only? withdrawals possible? (where detectable)
    async def symbol_spec(self, symbol: str) -> SymbolSpec: ...  # tick size/value, lot limits, digits, sessions
    async def candles(self, symbol: str, tf: str, from_: datetime, to: datetime) -> list[Candle]: ...
    async def subscribe_ticks(self, symbols: list[str]) -> AsyncIterator[Tick]: ...
    async def quote(self, symbol: str) -> Quote: ...             # bid, ask, spread
    async def place_order(self, req: OrderRequest) -> OrderAck: ...   # includes SL, TP, client_order_id
    async def find_order(self, client_order_id: str) -> OrderStatus | None: ...
    async def positions(self) -> list[BrokerPosition]: ...
    async def modify_position(self, position_id: str, sl: Decimal | None, tp: Decimal | None) -> None: ...
    async def close_position(self, position_id: str, volume: Decimal | None = None) -> CloseResult: ...
    async def deals(self, since: datetime) -> list[Deal]: ...    # fills, for reconciliation
    def server_time_offset(self) -> timedelta: ...               # to normalise to UTC
```

Rules every adapter must follow:

1. **Normalise to UTC.** MT5 timestamps are broker server time, not UTC. The adapter detects the offset and converts. A startup self-check compares broker time to system time.
2. **Closed candles only** are emitted as candles. The forming candle is flagged.
3. **Idempotency.** Put the `client_order_id` into the broker's comment or magic-number field and support `find_order()` so a retry after a timeout can check first.
4. **SL and TP are part of the order request.** An order without them is rejected before it reaches the broker.
5. **Errors map to a small set** (`Rejected`, `Timeout`, `NotConnected`, `InvalidVolume`, `MarketClosed`, `Requote`, `Unknown`). The execution module reacts to these, not to raw broker codes.
6. **Never log credentials.** The adapter receives decrypted credentials in memory only.
7. **Rate limit** calls to stay inside the broker's limits.

## 3. MetaTrader 5 adapter

- Uses the official `MetaTrader5` Python package, which talks to a **locally installed MT5 terminal**. The package is **Windows-only**.
- Check that a `MetaTrader5` wheel exists for the Python version you chose. If it does not, run the engine (or just the MT5 adapter as a small bridge process) on a pinned older Python behind the same interface.
- The terminal must be logged into the account and allow algorithmic trading. For several accounts, run one terminal instance per account (portable installs), each with its own adapter.
- Candle source: copy rates from the terminal, then store them in `marketdata.candles`.
- Position and order matching: set a **magic number** per bot and put the `client_order_id` in the order comment.
- Account types: handle both **netting** and **hedging** accounts. The engine assumes **one open trade per symbol**, which is simple on both, but the position-matching code differs.
- Filling mode and symbol suffixes vary by broker (for example `XAUUSDm`). The `symbol_map` table and the adapter's `symbol_spec()` handle this. Verify the filling mode the broker supports.
- Hosting: needs a Windows machine or VPS. It cannot run on Render.

### Alternative: a cloud MT5 API

A hosted service such as MetaApi exposes MT4 and MT5 through REST and WebSocket, so the engine can run on Linux and any host. Trade-offs: it needs the **MT5 login and password** to be held by a third party, it adds latency and cost, and some brokers' terms discourage sharing the trading password with a managed service. Decide only after reading the broker's agreement (see [10](10-decisions.md) Q2).

## 4. Deriv

Deriv offers two different things:

| Route | What it is | Fit |
|---|---|---|
| **Deriv API (WebSocket)** | Options-style products: binary options, multipliers and accumulators, with contract-based trading | Different product and cost structure from spot positions. Multipliers allow a stop loss and take profit, but results will not match a spot backtest |
| **Deriv MT5** | Standard MT5 positions on Deriv's MT5 servers | Closest to the Megatron strategy. Whether a given Deriv MT5 account offers all three symbols must be checked |

Plan: treat Deriv MT5 like any other MT5 broker. Build a `DerivWsAdapter` only if the project owner wants the contract-based product, and backtest it separately.

## 5. Per-broker verification checklist

Run this on a **demo** account before any live use:

1. Do XAUUSD, BTCUSD and GBPUSD exist, and under which names (suffixes)?
2. Tick size, tick value, contract size, minimum lot, lot step, maximum lot, digits.
3. Typical and worst-case spread for each symbol, by session.
4. Trading hours, including BTCUSD weekends and daily rollover.
5. Account type (netting or hedging), leverage, margin rules.
6. Supported order filling modes. Stop level and freeze level limits (minimum distance for SL and TP).
7. Does the broker's agreement allow automated scalping and the order rates the bot will use?
8. Can a third party (a cloud service) hold the login, or must the terminal run on your own machine?
9. Timestamp offset between broker server time and UTC, and how it changes around daylight saving.
10. Behaviour on requotes, partial fills and rejected orders.

Record the answers in `symbol_map` and in a short per-broker note under `docs/`.

## 6. Symbol mapping

Canonical symbols are `XAUUSD`, `BTCUSD`, `GBPUSD`. Each broker account has its own mapping to the broker's names, and the engine only uses canonical names internally. Candles from different brokers differ slightly, so **detect zones from the same account's candles that you execute on**.

## 7. Paper and backtest broker behaviour

To keep results honest:

- Fill at the next price after the signal, with the current spread added, plus a configurable slippage model.
- Apply minimum stop distance, lot step and margin rules from the real `SymbolSpec`.
- Model rejections and requotes at a configurable small rate in paper mode so error paths run.
- If SL and TP both fall inside one candle, assume the stop was hit first unless finer data says otherwise.
- Use the same commission and swap model as the real account where known.

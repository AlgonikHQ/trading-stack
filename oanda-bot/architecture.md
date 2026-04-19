# OANDA Bot — Architecture & Risk Framework

> Deep-dive into how the system is structured, how risk is managed, and how the execution pipeline works. Strategy logic (indicators, thresholds, parameters) is not included.

---

## Signal Flow

```
Candle fetch (M15 + D1)
        │
        ▼
Daily trend filter
  → Requires alignment across majority of pairs on D1
  → Longs need bullish bias, shorts need bearish bias
  → Misaligned pairs skipped entirely
        │
        ▼
Entry scoring engine
  → Each pair scored against [REDACTED] indicators
  → Score must exceed threshold to proceed
  → Direction (long/short) determined by score polarity
        │
        ▼
Pre-trade filter stack (all must pass)
  ├── Spread filter          → reject if spread too wide
  ├── Session filter         → reject outside valid windows
  ├── News blackout          → reject near high-impact events
  ├── Correlation guard      → reject if correlated pair already open
  ├── Daily loss limit       → reject if -3% daily drawdown hit
  └── Max open trades        → reject if 4 positions already open
        │
        ▼
Position sizing
  → Risk = fixed % of starting capital (not running balance)
  → SL calculated per instrument (ATR-informed, floor enforced)
  → Units calculated from risk / pip value / SL distance
        │
        ▼
Order execution (OANDA v20 REST)
  → Market order with attached SL
  → TPs set as separate limit orders (TP1, TP2, TP3)
  → Trailing stop armed after TP3
        │
        ▼
Position monitoring loop
  → Checks open positions every 15 min
  → Manages partial closes at staircase levels
  → Updates trailing stop post-TP3
  → Enforces Friday close at 20:00 UTC
```

---

## Risk Framework

### Position Sizing

```
Risk per trade = LIVE_STARTING_CAPITAL × RISK_PCT
SL distance   = max(instrument_SL_floor, calculated_SL_pips)
Units         = Risk / (SL_pips × pip_value_per_unit)
```

Capital used for sizing is always `min(current_balance, LIVE_STARTING_CAPITAL)` — this prevents risk from ballooning after profitable runs before a planned capital review.

### SL Floors (minimum pip distances)

| Pair | SL Floor |
|---|---|
| GBP/JPY | 15 pips |
| USD/JPY | 10 pips |
| GBP/USD | 10 pips |
| All others | 8 pips |

These floors exist to prevent absurdly tight stops being placed during low-volatility periods.

### Daily Loss Limit

Once the account drops **3% from the day's opening balance**, the bot enters a cooldown state:
- No new entries for the remainder of the trading day
- Open positions continue to be managed normally
- State resets at midnight UTC

### Max Concurrent Trades

Hard cap of **4 open positions** at any time. New signals are discarded (not queued) once the limit is hit.

### Correlation Guard

Pairs are grouped by shared currency exposure. If one pair from a correlation group is already open, no new entry is taken in the same group. This prevents unintentional doubling of USD exposure across e.g. EUR/USD + GBP/USD simultaneously.

---

## Session Logic

| Window | Rule |
|---|---|
| London open (08:00–08:15 UTC) | No new entries — avoids spike fills |
| Friday 18:00 UTC | No new entries for the remainder of the day |
| Friday 20:00 UTC | All open positions force-closed |
| USD/JPY | Restricted to specific session hours only |

The Friday rules exist to eliminate weekend gap risk entirely. The bot treats the weekend as a hard stop.

---

## Exit System (Staircase)

The bot uses R-multiples (multiples of initial risk) to define exit levels. The exact R values are not published, but the structure is:

```
Entry
  │
  ├── TP1 (partial close — 25% of position)
  │     → SL moved toward breakeven
  │
  ├── TP2 (partial close — 25% of position)
  │     → SL moved to breakeven or beyond
  │
  ├── TP3 (partial close — 25% of position)
  │     → Trailing stop armed on remaining 25%
  │
  └── Trail (final 25%)
        → Trails price, closes when trail hit
```

This structure locks in profit progressively while keeping a runner on strong moves. The goal is a right-skewed distribution: many small wins and breakevens, occasional large wins.

---

## API Integration

**Endpoint:** OANDA v20 REST API  
**Environment:** Live (`api-fxtrade.oanda.com`) / Demo (`api-fxpractice.oanda.com`)  
**Auth:** Bearer token via `Authorization` header  
**Data fetched:**
- M15 OHLC candles (entry scoring)
- D1 candles (trend filter)
- Account summary (balance, margin)
- Open positions and pending orders
- Instrument details (pip size, minimum units, spread)

**Reliability:**
- 502 retry wrapper on all candle fetches (3 retries, exponential backoff)
- Units re-fetched live before each order to prevent UNITS_INVALID rejection
- All API errors logged and alerted via Telegram

---

## Logging & Monitoring

**Systemd journal:** Primary log, captured by `journalctl -u oanda-bot.service`  
**Telegram (public channel):**
- Trade opens with pair, direction, SL, risk amount
- Partial close alerts (TP1/TP2/TP3 hit)
- Full close with P&L in R and GBP
- Daily summary at 21:00 UTC (trades, win rate, net P&L)
- 2-hour heartbeat card (confirms service alive)
- Patch notes on every restart (bot version shown)

---

## Versioning

| Version | Change |
|---|---|
| v1.0 | Initial build — long-only, single TP |
| v2.0 | Full rewrite — bidirectional, staircase exits, all 9 pairs |
| v2.1 | SL floors, correlation guard, demo audit fixes |
| v2.2 | Go-live patch — live API endpoint, London blackout, USD/JPY session lock |

---

## What's Not Documented Here

- Indicator types, periods, and weighting
- Score thresholds for entry and daily alignment
- ATR-based SL calculation methodology
- Backtest results and forward test performance data
- Strategy rationale and edge hypothesis

This is intentional. The above represents the infrastructure and risk management layer — the parts that are reusable, educational, and safe to share. The strategy layer is the proprietary edge.

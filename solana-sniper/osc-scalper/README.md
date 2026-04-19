# OSC Scalper

> A companion staircase position manager for established memecoins. Not a scanner — this bot manages existing holdings in a fixed watchlist, selling in stages as each coin hits predefined profit levels.

**Status:** 🟢 Live  
**Version:** v2.1  
**Coins managed:** BONK · WIF · POPCAT · PENGU · MEW

---

## What It Does

While the Sniper Bot hunts new launches, the OSC Scalper manages longer-term memecoin positions — coins that are already established but still have upside potential.

It runs a continuous loop checking each coin's current price against its entry price. When a coin hits a TP level, it sells a portion automatically. It never sells everything at once — it scales out progressively, locking in profit while keeping exposure to further upside.

---

## Staircase Structure

```
Entry
  │
  ├── TP1  (+20%)   → partial sell
  ├── TP2  (+40%)   → partial sell
  ├── TP3  (+75%)   → partial sell
  ├── TP4  (+120%)  → partial sell
  ├── TP5  (+200%)  → partial sell
  └── TP6  (+400%)  → final sell
```

Each TP level has a sold flag — once triggered it won't fire again, preventing double-sells on the same level even if the bot restarts.

A **30% peak stop-loss** activates after TP1. If the price drops 30% from its highest point after the first TP is hit, the remaining position is closed automatically.

---

## State Management

The bot persists its state to disk (`osc_state.json`) — it knows which TP levels have been hit for each coin and survives restarts cleanly. No position is lost or double-sold due to a service restart.

---

## Infrastructure

- **Runtime:** Python 3, separate systemd service (`scalp-sniper.service`)
- **Exchange:** Kraken (via API)
- **State file:** `osc_state.json`
- **Logs:** `/var/log/scalp_sniper.log`

---

## Why a Separate Service?

The Sniper and OSC Scalper run as independent systemd services. This means:
- Restarting the sniper for a patch doesn't affect open OSC positions
- Each bot can be updated, debugged, and monitored independently
- A crash in one doesn't take down the other

---

## What's Not Here

- Entry prices and position sizes for current holdings
- TP sell percentages per level
- Full source code

---

*This is not financial advice. UK CGT applies on every sell event.*

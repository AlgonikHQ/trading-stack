# Solana Sniper Bot

> 24/7 automated memecoin scanner built on Solana. Detects new token launches in real-time, runs them through a multi-layer filter stack, and executes buys in milliseconds. Manages open positions through a staircase take-profit system.

**Status:** 🟢 Live  
**Alerts:** [t.me/SolanaSniper_Alerts](https://t.me/SolanaSniper_Alerts)  
**Version:** v3.1

---

## What It Does

Most memecoins are rugs. This bot's job is to find the ones that aren't — fast enough to matter.

Every new token that launches on Solana gets scanned automatically. It passes through a series of filters designed to kill rugs, honeypots, and low-quality launches before any capital is deployed. If it passes everything, the bot buys, then manages the position automatically through a staircase of take-profit levels.

No manual intervention. No watching charts. It runs 24/7 as a systemd service on a Hetzner VPS.

---

## The Filter Stack

Eight layers between a new token and a buy order. Every layer must pass.

```
New token detected
        │
        ▼
┌───────────────────┐
│  1. Honeypot check │  Can the token actually be sold?
└────────┬──────────┘
         │
┌────────▼──────────┐
│  2. Market checks  │  Mcap, liquidity, age, volume thresholds
└────────┬──────────┘
         │
┌────────▼──────────┐
│  3. Holder check   │  Top holders can't be too concentrated
└────────┬──────────┘
         │
┌────────▼──────────┐
│  4. Bundle detect  │  Flags coordinated launch wallet clusters
└────────┬──────────┘
         │
┌────────▼──────────┐
│  5. Deployer check │  Deployer history checked — serial ruggers blacklisted
└────────┬──────────┘
         │
┌────────▼──────────┐
│  6. LP watchdog    │  Liquidity must be stable before entry
└────────┬──────────┘
         │
┌────────▼──────────┐
│  7. Rug detection  │  Price action and LP drain monitored post-entry
└────────┬──────────┘
         │
┌────────▼──────────┐
│  8. Dupe guard     │  Cross-checks against open positions — no double buys
└────────────────────┘
         │
         ▼
      BUY ✅
```

The exact thresholds for each layer are not published — that's the edge.

---

## Execution

Buys execute via **Jupiter Ultra** on Solana. The bot calculates stake size using **Kelly Criterion** — position size scales with confidence, not a flat amount. Wallet balance is read live from the chain via **Helius RPC** before every buy.

---

## Position Management — Staircase TPs

Once in a position, the bot manages the exit automatically across 6 levels:

```
Entry
  │
  ├── TP1  (+20%)   → partial sell
  ├── TP2  (+40%)   → partial sell
  ├── TP3  (+75%)   → partial sell
  ├── TP4  (+120%)  → partial sell
  ├── TP5  (+200%)  → trail mode activated
  └── TP6  (+400%)  → final target
```

After TP5 the remaining position switches to a trailing stop — if the token keeps running, the bot rides it. If it reverses, it exits automatically.

A **30% peak stop-loss** activates after TP1 — if the price drops 30% from its highest point, the bot exits regardless of TP levels.

---

## Post-Entry Protection

The bot doesn't just watch price. After buying, it continues monitoring:

- **LP watchdog** — if liquidity drops significantly after entry, the bot exits. Classic slow rug pattern.
- **Rug detection** — monitors for sudden price collapse. Hard exit triggered immediately.
- **Deployer blacklist** — auto-populated. If a deployer rugs one token, all future tokens from that wallet are blocked.

---

## Infrastructure

- **Runtime:** Python 3, systemd service, Hetzner Ubuntu 24.04 VPS
- **RPC:** Helius (Solana)
- **Execution:** Jupiter Ultra
- **Alerts:** Telegram (public channel + 2hr heartbeat)
- **Logs:** `/var/log/solana_sniper.log`

---

## Telegram Alerts

Every buy, sell, and TP hit is posted publicly. The channel also gets:
- 2-hour heartbeat cards (confirms the bot is alive)
- Patch notes on every restart
- P&L on every close

📡 [t.me/SolanaSniper_Alerts](https://t.me/SolanaSniper_Alerts)

---

## What's Not Here

- Filter thresholds and scoring parameters
- Kelly sizing formula and inputs
- Rug detection sensitivity settings
- Full scanner and execution source code

See [OSC Scalper](./osc-scalper/) for the companion position manager running on established coins.

---

*Solana trading involves significant risk. This is not financial advice. UK CGT applies on every sell event.*

# Solana Sniper Bot

> 24/7 automated memecoin scanner built on Solana. Detects new token launches in real-time, runs them through a multi-layer filter stack, and executes buys via Jupiter Ultra. Manages open positions through a staircase take-profit system with trailing stop.

**Status:** 🟢 Live  
**Version:** v3.2  
**Alerts:** [t.me/SolanaSniper_Alerts](https://t.me/SolanaSniper_Alerts)

---

## What It Does

Most memecoins are rugs. This bot's job is to find the ones that aren't — fast enough to matter.

Every new token that launches on Solana gets scanned automatically. It passes through a series of filters designed to kill rugs, honeypots, and low-quality launches before any capital is deployed. If it passes everything, the bot buys and manages the position automatically through a staircase of take-profit levels with a trailing stop on the remainder.

No manual intervention. No watching charts. Runs 24/7 as a systemd service on a Hetzner VPS.

---

## The Filter Stack

Nine layers between a new token and a buy order. Every layer must pass.

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
│  5. Deployer check │  Serial ruggers auto-blacklisted
└────────┬──────────┘
         │
┌────────▼──────────┐
│  6. Socials check  │  Token must have verified social presence
└────────┬──────────┘
         │
┌────────▼──────────┐
│  7. LP watchdog    │  Liquidity must be stable before entry
└────────┬──────────┘
         │
┌────────▼──────────┐
│  8. Rug detection  │  Price action and LP drain monitored post-entry
└────────┬──────────┘
         │
┌────────▼──────────┐
│  9. Dupe guard     │  No double buys across open positions
└────────────────────┘
         │
         ▼
      BUY ✅
```

The exact thresholds for each layer are not published — that's the edge.

---

## Execution

Buys execute via **Jupiter Ultra** on Solana. Stake size is set at **£5 default** with Kelly Criterion adjusting within a £3–£6 range based on available SOL balance. Wallet balance is read live from the chain via **Helius RPC** before every buy — no stale ledger reads.

---

## Position Management — Staircase TPs

Once in a position, the bot manages the exit automatically across 5 TP levels plus trail:

```
Entry
  │
  ├── TP1  (+20%)   → 15% of holdings sold
  ├── TP2  (+40%)   → 15% of holdings sold
  ├── TP3  (+60%)   → 10% of holdings sold
  ├── TP4  (+80%)   → 10% of holdings sold
  ├── TP5  (+100%)  → 10% of holdings sold → trail mode activated
  └── Trail         → 25% below peak → full exit
```

After TP5 the remaining position switches to a trailing stop — if the token keeps running, the bot rides it. If it reverses 25% from peak, it exits automatically.

A **tiered floor system** protects banked profits — once TPs are hit, the floor rises so the position can never give back more than a defined amount of gains.

---

## Post-Entry Protection

The bot doesn't just watch price. After buying, it continues monitoring:

- **LP watchdog** — if liquidity drops >20% after entry, the bot exits immediately
- **Rug detection v3** — monitors for sudden price collapse and liquidity drain. Hard exit triggered immediately with 3 retries
- **Ride-on hard floor** — if price drops >15% during an extended hold, exits immediately regardless of trail
- **Ride-on LP guard** — if liquidity drops >12% during an extended hold, exits immediately
- **Deployer blacklist** — auto-populated on confirmed rugs. All future tokens from that wallet are blocked
- **Ghost position protection** — all sell paths retry 3x before closing the position. If sell fails, position stays open and a Telegram alert fires

---

## Ride-On (Extended Hold)

If a position reaches the time stop still profitable with positive 5-minute momentum, the bot activates ride-on mode — extending the hold beyond the normal time limit to capture bigger moves.

Ride-on has its own protection layer:
- Hard floor at -15% (exits immediately if price drops this far during ride-on)
- LP guard at -12% liquidity drop
- Stagnation exit if momentum fades over 20 minutes
- Tighter trail stop (20% below peak)

---

## Infrastructure

- **Runtime:** Python 3, systemd service, Hetzner Ubuntu 24.04 VPS
- **RPC:** Helius (primary) + Solana mainnet (fallback)
- **Execution:** Jupiter Ultra
- **Alerts:** Telegram (private + public channel)
- **Logs:** `/var/log/solana_sniper.log`

---

## Telegram Alerts

Every buy, sell, and TP hit fires a Telegram card. The private channel also gets:
- Hourly heartbeat showing live SOL balance, today P&L, all-time P&L
- Daily P&L summary
- Weekly P&L summary (Sunday)
- Patch notes on every restart

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

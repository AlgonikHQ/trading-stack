# Solana Sniper 🎯

Live Solana memecoin sniper. Monitors Raydium for new token launches, applies a 10-layer filter stack, stakes via Kelly criterion, and manages exits via a 6-stage staircase TP system with rug detection.

**Status:** Live — balance reset 14 Apr 2026 (0.5933 SOL / 37.40 GBP)
**Wallet:** B679niFFxdhs14zUxe65ZKHhSxEXJJmMaMWZMgUqPw1p
**Built by:** [@AlgonikHQ](https://github.com/AlgonikHQ)

---

## Filter Stack

| Filter | Purpose |
|---|---|
| Honeypot detection | Rejects unswappable tokens |
| Mcap / Liquidity / Age / Volume | Baseline quality gates |
| Momentum | Requires buy pressure before entry |
| Holder concentration | Rejects whale-dominated supply |
| Bundle wallet detection | Rejects coordinated launch manipulation |
| Cross-bot dupe prevention | No double positions |
| Pre-entry liquidity drain watchdog | Rejects liq pulls before buy |
| Deployer blacklist | Auto-populates on confirmed rugs |
| Kelly stake sizing | Position size scaled to edge |

---

## Exit System

| Stage | Trigger | Action |
|---|---|---|
| TP1-TP6 | +20/40/75/120/200/400% | Staircase partial sells |
| Trail stop | Post-TP3 | Trails peak price |
| Peak chase v2 | New ATH detected | Adjusts trail upward |
| Rug detection v3 | LP drain > 20% | Emergency exit + Telegram alert |

---

## Stack

| Component | Technology |
|---|---|
| Chain | Solana |
| DEX | Raydium via Jupiter |
| RPC | Helius |
| Notifications | Telegram card-format alerts |
| Runtime | systemd (solana-sniper.service), Ubuntu 24.04 |

---

## Companion Bot

**OSC Bot (scalp-sniper.service)** — pure staircase sell manager for 5 held coins (BONK/WIF/POPCAT/PENGU/MEW). No buying. Staircase: +20/40/75/120/200/400%.

---

## Roadmap

- [ ] Name/ticker blacklist regex in filters.py
- [ ] Smart money wallet tracker (alert-only first)
- [ ] Raydium Phase 2 direct integration
- [ ] Fix [FILTER ERROR] run_all_filters() missing positional argument

---

*Part of the AlgonikHQ FIRE@45 automated income stack.*

# trading-stack

> Transparent documentation of a live algo trading infrastructure — built in public by [@AlgonikHQ](https://x.com/AlgonikHQ).

---

## What is this?

This repo documents the engineering behind a fully automated, multi-strategy trading stack running 24/5 on a Hetzner VPS. The goal is simple: **build toward financial independence at 45 through automated systems**, and do it in public.

Every system here is live and trading real capital. This repo shows the architecture, infrastructure, and risk framework — not the strategy logic (that stays closed source).

---

## The Stack

| System | Market | Status |
|---|---|---|
| [OANDA Forex Bot](./oanda-bot/) | FX — 9 pairs, M15 | 🟢 Live |
| Solana Sniper | Solana memecoins | 🟢 Live |
| Kraken DCA Bot | BTC / ETH / SOL | 🟢 Live |
| Trading 212 ISA | ETF / Dividend pies | 🟢 Live |

---

## The Goal

**FIRE at 45 (2032).** Target: ~£855,000.

Four parallel automated income streams compound into a tax-efficient ISA flywheel:
- Spread betting profits (tax-free under UK law) → ISA top-ups
- ISA dividends eventually fund bot capital → salary removed from equation
- Crypto staking + sniper gains → BTC accumulation

This repo is the engineering layer of that plan. Follow along on X at [@AlgonikHQ](https://x.com/AlgonikHQ).

---

## What's Open vs Closed

| Open (this repo) | Closed (private) |
|---|---|
| System architecture | Entry/exit signal logic |
| Risk framework & position sizing | Indicator parameters & thresholds |
| Session & filter rules | Backtests & forward test data |
| Infrastructure & deployment | Live P&L beyond public Telegram |
| Telegram alert structure | Strategy source code |

---

## Infrastructure

- **VPS:** Hetzner Ubuntu 24.04 (dedicated)
- **Process management:** systemd services
- **Languages:** Python 3, Bash
- **Alerts:** Telegram bot (public channels per system)
- **Remote access:** Termius (mobile), SCP/PowerShell (Windows)

---

## Follow the Build

- X: [@AlgonikHQ](https://x.com/AlgonikHQ)
- Telegram alerts: linked per system in each folder

---

*This is not financial advice. All systems trade real capital at real risk. Document shared for educational and transparency purposes only.*

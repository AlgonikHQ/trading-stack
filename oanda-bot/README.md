# OANDA Forex Bot

> Automated spread betting bot trading 9 FX pairs on M15 candles via the OANDA v20 REST API. Going live on a UK spread betting account end of April 2026 — profits 100% tax-free under UK law.

---

## Overview

This bot runs continuously as a systemd service on a Hetzner Ubuntu VPS. It scans 9 major FX pairs every 15 minutes, scores potential entries using a multi-indicator framework, and manages positions through a staircase exit system with partial closes.

**Current version:** v2.6  
**Status:** 📄 Paper trading — go-live end of April 2026  
**Demo period:** 6 weeks prior to live  

---

## Pairs Traded
EUR/USD  GBP/USD  USD/JPY  USD/CHF
AUD/USD  USD/CAD  EUR/GBP  GBP/JPY  NZD/USD
Timeframe: **M15** (15-minute candles)  
Direction: **Bidirectional** (long & short)

---

## Architecture
┌─────────────────────────────────────────────────────┐
│                   oanda-bot.service                  │
│                  (systemd, Python 3)                 │
└──────────────────────┬──────────────────────────────┘
│ every 15 min
┌───────────▼───────────┐
│    Candle Fetcher     │  OANDA v20 REST API
│  (M15 + D1 prices)   │  502 retry wrapper
└───────────┬───────────┘
│
┌───────────▼───────────┐
│    Entry Scoring      │  Multi-indicator
│    Engine             │  signal framework
└───────────┬───────────┘
│
┌───────────▼───────────┐
│    Filter Stack       │  Session / spread /
│                       │  correlation / VIX /
│                       │  daily limit / pair guards
└───────────┬───────────┘
│
┌───────────▼───────────┐
│   Position Manager    │  Staircase exits
│                       │  Partial closes (3x)
└───────────┬───────────┘
│
┌───────────▼───────────┐
│   Telegram Alerts     │  Public + private channels
│   + Daily Summary     │  22:00 UTC daily
└───────────────────────┘
---

## Key Features

**Entry logic**
- Multi-indicator scoring system (daily score threshold required)
- Daily timeframe EMA alignment filter (trend confirmation)
- Bidirectional signal detection (long and short)
- Correlation guard — prevents over-exposure to correlated USD pairs
- Per-pair entry guards (GBP/USD requires 4/4 score + ADX > 30)
- M5 confirmation required before entry fires

**Risk framework**
- Fixed risk per trade (% of starting capital, not running balance)
- SL floor per instrument (minimum pip distance enforced)
- Maximum concurrent open trades: **3**
- Daily loss limit: **3%** of capital
- Per-pair loss cooldown: 60 minutes after any loss on that instrument
- All positions closed Friday 20:00 UTC (no weekend gap exposure)
- No new entries from 18:00 UTC Friday

**Session filters**
- London open blackout: 08:00–08:15 UTC
- US data window blackout: 13:00–13:30 UTC
- US market open blackout: 14:30–15:00 UTC
- Fed decisions blackout: 18:45–19:00 UTC
- BoE decisions blackout: 11:45–12:00 UTC
- GBP/USD blocked during Asian/Sydney session: 00:00–07:00 UTC
- JPY pairs restricted to London/NY sessions only
- BOJ decision blackout on USD/JPY

**Volatility filter (v2.6)**
- VIX fetched every 15 mins during NY session (14:30–21:00 UTC)
- VIX > 25 — new entries blocked
- VIX > 35 — all open positions closed immediately

**State persistence (v2.6)**
- Daily P&L, win count, trade count survive bot restarts
- Weekly pair P&L survives bot restarts
- Trade state reconciled against OANDA on every startup

**Exit system**
- Staircase with 3 partial closes (25% each at TP1/TP2/TP3)
- Final 25% trails to capture extended moves
- R-based TP levels (multiples of initial risk)

---

## Telegram Alerts

The bot posts to a public Telegram channel (@OandaForex_Sigs):
- Trade open / partial close / full close alerts
- Daily P&L summary at 22:00 UTC
- Weekly P&L report every Friday 20:00 UTC
- Patch notes on each restart (versioned)

---

## Configuration

See [config.example.env](./config.example.env) for the full configuration shape. All sensitive values are injected via environment variables — never hardcoded.

---

## Deployment

```bash
sudo systemctl restart oanda-bot.service
journalctl -u oanda-bot.service -f
sudo systemctl status oanda-bot.service
```

---

## Changelog

See [changelog.json](../changelog.json) for full version history.

---

## What's Not Here

Entry/exit signal logic, indicator parameters, thresholds, and backtesting data are not included. This folder documents architecture and operational framework only.

---

*Spread betting profits are tax-free for UK residents. This is not financial advice.*

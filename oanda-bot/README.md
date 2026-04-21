# OANDA Forex Bot

> Automated spread betting bot trading 9 FX pairs on M15 candles via the OANDA v20 REST API. Live on a UK spread betting account — profits 100% tax-free under UK law.

---

## Overview

This bot runs continuously as a systemd service on a Hetzner Ubuntu VPS. It scans 9 major FX pairs every 15 minutes, scores potential entries using a multi-indicator framework, and manages positions through a staircase exit system with partial closes.

**Current version:** v2.5  
**Go-live:** April 2026  
**Demo period:** ~6 weeks prior to live  

---

## Pairs Traded

```
EUR/USD  GBP/USD  USD/JPY  USD/CHF
AUD/USD  USD/CAD  EUR/GBP  GBP/JPY  NZD/USD
```

Timeframe: **M15** (15-minute candles)  
Direction: **Bidirectional** (long & short)

---

## Architecture

See [architecture.md](./architecture.md) for the full signal flow and risk framework.

```
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
           │                       │  correlation / daily limit
           └───────────┬───────────┘
                       │
           ┌───────────▼───────────┐
           │   Position Manager    │  Staircase exits
           │                       │  Partial closes (3x)
           └───────────┬───────────┘
                       │
           ┌───────────▼───────────┐
           │   Telegram Alerts     │  Public channel
           │   + Daily Summary     │  21:00 UTC daily
           └───────────────────────┘
```

---

## Key Features

**Entry logic**
- Multi-indicator scoring system (score threshold required to open)
- Daily timeframe alignment filter (trend confirmation)
- Bidirectional signal detection (long and short)
- Correlation guard — prevents over-exposure to correlated USD pairs

**Risk framework**
- Fixed risk per trade (% of starting capital, not running balance)
- SL floor per instrument (minimum pip distance enforced)
- Maximum concurrent open trades: **4**
- Daily loss limit: **3%** of capital
- **Per-pair loss cooldown: 60 minutes after any losing trade on that instrument**
- All positions closed Friday 20:00 UTC (no weekend gap exposure)
- No new entries from 18:00 UTC Friday

**Session filters**
- London open blackout: 08:00–08:15 UTC (avoids volatile open spike)
- US data window blackout: 13:00–13:30 UTC (NFP, CPI, GDP, Retail Sales)
- US market open blackout: 14:30–15:00 UTC (cash equities spike window)
- Fed decisions blackout: 18:45–19:00 UTC
- BoE decisions blackout: 11:45–12:00 UTC
- USD/JPY restricted to specific session windows
- News event blackout support

**Exit system**
- Staircase with 3 partial closes (25% each)
- Final 25% trails to capture extended moves
- R-based TP levels (multiples of initial risk)

---

## Telegram Alerts

The bot posts to a public Telegram channel:
- Trade open / partial close / full close alerts
- Daily P&L summary at 21:00 UTC
- 2-hour heartbeat confirming the service is alive
- Patch notes on each restart (versioned)

---

## Configuration

See [config.example.env](./config.example.env) for the full configuration shape. All sensitive values (API keys, account IDs) are injected via environment variables — never hardcoded.

---

## Deployment

The bot runs as a `systemd` service. See [systemd/oanda-bot.service](./systemd/oanda-bot.service) for the sanitised unit file.

```bash
# Deploy / restart
sudo systemctl restart oanda-bot.service

# View live logs
journalctl -u oanda-bot.service -f

# Check status
sudo systemctl status oanda-bot.service
```

---

## What's Not Here

The entry scoring logic, indicator parameters, thresholds, and backtesting data are not included in this repo. This folder documents the system's architecture and operational framework only.

---

*Spread betting profits are tax-free for UK residents. This is not financial advice.*

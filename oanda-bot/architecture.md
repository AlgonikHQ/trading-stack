# OANDA Bot — Architecture & Changelog

## Overview
Systematic intraday forex trading bot running on a Hetzner VPS (Ubuntu 24.04).
Trades M15 candles across multiple FX pairs during London and NY sessions only.

## Strategy Summary
- Multi-factor entry scoring: Daily EMA alignment, ADX trend strength, RSI, MACD
- ATR-based stop loss with staircase trailing exit
- Partial close system at TP levels
- Session filter, news blackout windows, spread check, correlation guard
- Per-pair loss cooldown after losing trades
- Max concurrent open trades cap

## Risk Model
- Fixed risk per trade, Kelly-fraction sizing
- Daily loss limit hard stop
- Minimum balance floor
- Friday close — all positions flat before weekend

## Version History

### v2.5.1 — 22 Apr 2026
**Entry quality hardening — loss mitigation patch**

- TP1 exit floor raised to minimum 1R. Previous value created negative expected value
  even at above-average win rates, as single losses exceeded multiple partial wins.
- RSI overbought threshold tightened. Squeeze bypass entries now blocked when RSI
  indicates momentum exhaustion, preventing late entries into extended moves.
- Riding condition RSI cap tightened. Trend-continuation entries require momentum
  to still be building, not already stretched.
- Ghost riding state guard added. Riding signals now require a verified open position
  to exist for that pair. Prevents phantom entries from stale in-memory state
  persisting across bot restarts — root cause of an erroneous short entry on 22 Apr.
- Scan skip logging added. Pairs silently dropped due to trade cap now log explicitly,
  improving post-session audit visibility.

### v2.5 — 21 Apr 2026
- Bot goes live on VPS
- Sniper Mode active: full filter stack required before entry
- Per-pair loss cooldown (60 min block after losing trade)
- News blackout windows tightened
- Friday hard close at 20:00 UTC
- Max 4 concurrent trades, 3% daily loss limit

### v2.0
- Shorts enabled on low-score pairs with ADX guard
- Session filter added (London + NY only)
- MACD continuation filter
- Spread check added
- Staircase trailing SL with partial close at TP1

## Infrastructure
- Runs as systemd service with auto-restart
- Environment variables via systemd unit (no credentials in code)
- Logs via journalctl
- Trade state persisted to JSON on disk
- Telegram notifications for all entries, exits, and daily summary

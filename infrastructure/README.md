# Infrastructure

> The full deployment stack behind AlgonikHQ — a single Hetzner Ubuntu VPS running four automated trading bots as systemd services, with Telegram as the monitoring layer.

---

## Overview

All bots run on a single dedicated VPS. Each bot is an independent systemd service — they can be started, stopped, updated, and monitored individually without affecting each other.

```
┌─────────────────────────────────────────────────────────┐
│              Hetzner VPS — Ubuntu 24.04                  │
│                                                          │
│  ┌─────────────────┐      ┌─────────────────┐           │
│  │ oanda-bot        │      │ solana-sniper    │           │
│  │ .service         │      │ .service         │           │
│  └─────────────────┘      └─────────────────┘           │
│                                                          │
│  ┌─────────────────┐      ┌─────────────────┐           │
│  │ scalp-sniper     │      │ statiqfc         │           │
│  │ .service         │      │ .service         │           │
│  └─────────────────┘      └─────────────────┘           │
│                                                          │
│  Telegram bot → public alert channels per service        │
└─────────────────────────────────────────────────────────┘
```

---

## The Four Services

| Service | Bot | Language |
|---------|-----|----------|
| oanda-bot.service | OANDA Forex Bot | Python 3 |
| solana-sniper.service | Solana Sniper | Python 3 |
| scalp-sniper.service | OSC Scalper | Python 3 |
| statiqfc.service | StatiqFC Football Stats | Python 3 |

Each service is configured to restart automatically on failure — if a bot crashes, it comes back on its own.

---

## VPS Spec

| | |
|---|---|
| Provider | Hetzner |
| OS | Ubuntu 24.04 LTS |
| Architecture | x64 |
| Process manager | systemd |
| Remote access | SSH (Termius on mobile, PowerShell/SCP on Windows) |

---

## Common Commands

```bash
# Check status of any service
sudo systemctl status oanda-bot.service

# Start / stop / restart
sudo systemctl start oanda-bot.service
sudo systemctl stop oanda-bot.service
sudo systemctl restart oanda-bot.service

# View live logs
journalctl -u oanda-bot.service -f

# View last 100 lines
journalctl -u oanda-bot.service -n 100

# Enable service to start on boot
sudo systemctl enable oanda-bot.service
```

Replace `oanda-bot` with any service name from the table above.

---

## Deployment Workflow

Updates are deployed via SCP from a Windows machine, then the relevant service is restarted:

```powershell
# Windows — copy updated file to VPS
scp C:\Users\username\Downloads\updated_file.py root@YOUR_VPS_IP:/root/bot_folder/
```

```bash
# VPS — restart the service
sudo systemctl restart oanda-bot.service

# Confirm it's running
sudo systemctl status oanda-bot.service
```

**Critical rule:** Always stop the relevant service before deploying changes, back up the existing file, then restart. Never hot-swap a file while the service is running.

---

## Logging

Each service writes to the systemd journal. Some bots also write to dedicated log files:

| Service | Log location |
|---|---|
| All services | `journalctl -u SERVICE_NAME` |
| solana-sniper | `/var/log/solana_sniper.log` |
| scalp-sniper | `/var/log/scalp_sniper.log` |

Logs are the first place to check when something looks wrong. The Telegram heartbeat (every 2 hours) is the early warning system — if a heartbeat stops, the service has likely crashed.

---

## Telegram Monitoring

Every bot posts to its own public Telegram channel. The monitoring pattern is the same across all bots:

- **2-hour heartbeat** — confirms the service is alive
- **Trade alerts** — every open, partial close, and full close
- **Daily summary** — P&L recap at end of day
- **Patch notes** — posted automatically on every restart, includes bot version

If the 2-hour heartbeat stops arriving, the service has crashed. Check `journalctl` for the error.

---

## Security

- All API keys and secrets are stored as environment variables in `.env` files on the VPS
- `.env` files are never committed to version control
- Each service loads its own `.env` via `EnvironmentFile=` in the systemd unit file
- SSH access via key authentication only

---

## File Structure on VPS

```
/root/
├── oanda_trading_bot.py
├── oanda_bot.env
├── solana_sniper/
│   ├── scanner.py
│   ├── scalp_scanner.py
│   ├── filters.py
│   ├── jupiter_swap.py
│   ├── config.py
│   └── .env
├── statiq/
│   └── bot/
│       ├── statiq_bot.py
│       ├── scanner.py
│       ├── fetcher.py
│       ├── database.py
│       └── config.py (gitignored)
/etc/systemd/system/
├── oanda-bot.service
├── solana-sniper.service
├── scalp-sniper.service
└── statiqfc.service
/var/log/
├── solana_sniper.log
└── scalp_sniper.log
```

---

See [systemd/](./systemd/) for sanitised service unit files.

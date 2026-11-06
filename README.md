# HyperTrade — Railway + Telegram Token Login

This repository uses the supplied `HyperTrade.py` as the main dashboard/API application. The old standalone candle collector logic is replaced by HyperTrade's live tick/candle engine. Quotex authentication is token-based (SSID + optional CSRF); email/password Quotex login is not used.

## Root files

- `app.py` — HyperTrade dashboard, API, live candle engine, persistent SQLite storage, Telegram controller
- `pyquotex-1.0.3.tar.gz` — supplied pyquotex library archive
- `requirements.txt` — Python dependencies
- `Dockerfile` — Railway Docker build and cloudflared install
- `railway.json` — Railway deployment configuration
- `Procfile` — process start command
- `.gitignore`, `.dockerignore`

All repository files belong in the repository root; there is no enclosing project folder in the ZIP.

## Railway variables

Required:

```env
BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN
APP_PASSWORD=SET_A_LONG_RANDOM_DASHBOARD_PASSWORD
```

Optional:

```env
USE_DEMO=true
DATA_DIR=/data
MAX_CANDLES=5000
GMAIL_EMAIL=
GMAIL_APP_PASSWORD=
ADMIN_PASSWORD=
```

Never commit actual Telegram bot tokens, Quotex SSID/CSRF values, Gmail app passwords, or dashboard passwords.

Create a Railway Volume and mount it at `/data`. SQLite candle history and signal history are stored there. Without a persistent volume, the container's writable filesystem may be lost during redeploys.

## Telegram commands

- `/start` — asks for Quotex SSID token, then CSRF token
- `/skipcsrf` — continue without CSRF if the provider session accepts it
- `/logout` — disconnects the Quotex session but preserves saved candles
- `/server` — starts a Cloudflare Quick Tunnel and returns a temporary `trycloudflare.com` URL
- `/status` — shows feed/database status
- `/cancel` — cancels token entry

The first Telegram chat to use `/start` claims ownership. Keep the bot private. SSID/CSRF are processed in memory and are not written to disk. Delete credential messages from the Telegram chat after login.

## Time handling

Candle timestamps are converted from Unix UTC timestamps to `Asia/Dhaka` (UTC+06:00). API candle `time` values and live candle labels use this timezone. Signal entry time is calculated as server Unix time + 120 seconds, then displayed in UTC+06:00. `/signal-time` exposes the server-side calculation.

## API endpoints

- `GET /` — dashboard
- `GET /health` — Railway health check
- `GET /?Pair=USDPHP_otc&Candle=500&Time=1m` — candle history and running candle
- `GET /markets` — detected markets
- `GET /latest?Pair=USDPHP_otc&Time=1m` — latest candle
- `GET /payout-history?Pair=USDPHP_otc&Time=1m` — payout history
- `GET /signal-time?Pair=USDPHP_otc&Direction=CALL` — entry time +120 seconds in UTC+06:00

## Important notes

- This uses an unofficial third-party Quotex client. Provider-side changes may break login or candle collection.
- Historical backfill depends on the provider returning the requested candle window; missing history cannot be guaranteed if the provider limits or omits it.
- Cloudflare Quick Tunnel URLs are temporary and can change after restart. Use Railway's public domain for a stable endpoint.
- Keep `USE_DEMO=true` unless you fully understand the risks of real-account trading. The app includes trading-related endpoints; test carefully before enabling any real-money action.

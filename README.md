# ExitAnt Connors RSI2 — LBank

Deploy this automated trading bot to **your own Render account** as a background worker. The bot runs from a prebuilt Docker image;
this repository contains **only the Render Blueprint (`render.yaml`)** — no source code.

The bot trades **Larry Connors RSI2** or **Turtle + Connors RSI2 merge** on LBank USDT perpetual futures with exactly the rules of the
TanQuant backtest engine, on the same Yahoo daily candles (stocks/ETFs: New York regular session · crypto: UTC day · gold/silver: COMEX trading day).
Copy a TanQuant result with **“Copy bot setup”** and paste it into the bot to set everything up at once.

## Deploy to Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/ExitAnt-dev/connors-rsi2-lbank)

```
https://render.com/deploy?repo=https://github.com/ExitAnt-dev/connors-rsi2-lbank
```

1. Click the button (or open the link) and sign in to Render.
2. Render reads `render.yaml` and proposes **one background worker** with a **1 GB persistent disk** mounted at `/data`
   (settings, books and positions survive restarts and redeploys).
3. Enter the environment variables listed below. They are marked `sync: false`, so Render asks you for them on the deploy screen
   and stores them **only in your own Render account**.
4. Click **Apply** and wait for the first deploy to finish.
5. Open Telegram and send **`/start`** to your bot to begin setup.

## Before you deploy

- **LBank account registered with an ExitAnt invite code.** At every start the bot reads your own account's UID with your key and checks the invite code with LBank's affiliate API.
- **API key** with futures trading permission. The key must be allowed to read the account and trade USDT perpetual futures.
  *For the Turtle merge's cash vault* the key needs transfer permission (futures ↔ spot).
- **Hedge (two-way) position mode** in LBank USDT-M futures (LBank → Futures → settings). The bot checks it at start and tells you if it is off.
- LBank API keys need an **IP whitelist**. At every start the bot shows the server's current outbound IP and waits for **✅ Applied** after you add it in LBank → API Management → Edit → IP Address.
- USDT in your LBank **USDT futures** account.

## Environment variables

| Variable | What to enter |
|---|---|
| `API_KEY` | LBank API key |
| `SECRET_KEY` | LBank API secret |
| `TELEGRAM_BOT_TOKEN` | Bot token issued by [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Your own Telegram chat ID — the bot ignores every other sender |

Nothing else to configure: the exchange, trading mode and state-file location are built into the image.

## What the bot does

- Decides once per daily candle, right after Yahoo closes it, and sends market orders at once (the backtest's “fill at the close”).
- Watches the stops itself while the asset's market is open (stocks: New York regular session) and closes at market when a stop is reached —
  the stops are **not** resting orders on the exchange, so **keep the bot running**. Stopping the bot stops the stop-watching.
- Keeps one account per symbol (trading pocket + vault), like TanQuant's multi-asset backtest. The Turtle merge moves part of the profit to the
  vault (spot wallet) on a trend exit and reinjects it after a deep drawdown, by the USA Turtle rules.
- Never stops trading because of an exchange answer — a refused or failed request skips that symbol for that cycle and is retried.

⚠️ Futures trading carries a very high risk of loss. Backtest results do not guarantee future returns.

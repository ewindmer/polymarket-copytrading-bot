<div align="center">

[![Node](https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![MongoDB](https://img.shields.io/badge/MongoDB-Required-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://mongodb.com)
[![Network](https://img.shields.io/badge/Polygon-USDC-8247E5?style=flat-square&logo=polygon&logoColor=white)](https://polygon.technology)
[![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://typescriptlang.org)
[![License](https://img.shields.io/badge/License-ISC-blue?style=flat-square)](LICENSE)

</div>

---

**3:47am.**

Car just moved on a market nobody was watching.

Position sized heavy. Conviction trade. The kind that looks obvious in hindsight and invisible in the moment.

You were asleep.

Your bot wasn't.

By 3:47:03 — two seconds after Car's fill hit the chain — your order was already in the queue. Same market. Proportional size. Same direction.

You woke up to a green position and a MongoDB log with 1 new entry.

**That's what this bot does.**

---

## Why Car

Car isn't a casual bettor. This is one of Polymarket's most studied wallets — consistently ranked in the top tier across political, sports, and macro event markets.

The edge is public, on-chain, and traceable. This bot converts it into executable signals before you've opened your laptop.

| | |
|---|---|
| **Profile** | [Car on Polymarket](https://polymarket.com/@Car?tab=activity) |
| **Wallet** | `0x7C3Db723F1D4d8cB9C550095203b686cB11E5C6B` |
| **Network** | Polygon · USDC |
| **Known for** | High-conviction entries, top leaderboard presence |

---

## What $5,000 Looks Like Running This Bot

> ⚠️ Illustrative only. Based on proportional sizing mechanics — not a performance guarantee. Fill in real numbers from your own run.

Assume Car holds $50,000 USDC and you deploy $5,000.

Your sizing ratio: **10%**. Every trade Car makes, you mirror at 10% of their size.

| Car's Trade | Your Mirrored Position | At 65% win rate | At 55% win rate |
|---|---|---|---|
| $10,000 | $1,000 | +$650 avg | +$550 avg |
| $5,000 | $500 | +$325 avg | +$275 avg |
| $2,000 | $200 | +$130 avg | +$110 avg |

*Win rate is Car's — your actual rate will be lower due to slippage and latency. This table exists to show the sizing mechanic, not to promise returns.*

**The core math:** more capital deployed = larger absolute positions = larger absolute returns (and larger absolute losses). The bot handles sizing automatically. You control the starting balance.

---

## The Bot, Live

While you're working, sleeping, or not watching charts:

```
[03:47:01] Polling Car's wallet...           no new activity
[03:47:02] Polling Car's wallet...           no new activity
[03:47:03] Polling Car's wallet...           ── TRADE DETECTED ──
           Market   Will X happen before Y?
           Side     BUY
           Size     $4,200
           Price    0.61

[03:47:03] Staleness check...                0s old — OK
[03:47:03] Price drift check...              current 0.62 — within 5% threshold
[03:47:03] Sizing your position...           $5,000 ÷ $50,000 × $4,200 = $420
[03:47:04] Submitting order...               FILLED at 0.62
[03:47:04] Writing to MongoDB...             done ✓

           Entry    0.62
           Size     $420
           P&L      open

[03:47:04] Waiting for next signal...
```

That loop runs every second. Every hour. Every night you don't feel like watching markets.

---

## Capital Deployment Guide

Not all capital is equal. Here's how to think about sizing before you start:

| Deployment Size | Behavior | Recommended for |
|---|---|---|
| **< $500** | Very small positions — fills may fail on low-liquidity markets | Testing and validation only |
| **$500 – $2,000** | Functional mirroring — real fills, real data, manageable risk | First month of running |
| **$2,000 – $5,000** | Meaningful exposure — positions large enough to matter | Confident after 30+ days of logs |
| **$5,000 – $20,000** | Full proportional parity with Car on most trades | Experienced, with your own PnL data |
| **> $20,000** | Liquidity risk increases — large orders can move thin markets | Advanced use only |

**Rule of thumb:** only deploy what you'd be comfortable seeing go to zero in a bad month.

---

## Execution Guardrails

The bot doesn't blindly chase every signal. Before any order is placed:

| Guard | Threshold | What it prevents |
|---|---|---|
| **Staleness filter** | 24 hours | Acting on old, irrelevant data |
| **Price drift gate** | 5% max deviation | Chasing already-moved markets |
| **Proportional sizing** | Auto-calculated | Overexposure relative to your balance |
| **Retry cap** | 3 attempts | Infinite loops on failed fills |
| **Deduplication** | Transaction hash | Double-executing the same trade |
| **Full audit log** | MongoDB | Every event recorded — nothing hidden |

---

## What Others Are Watching

Car's wallet is one of the most observed on Polymarket. These are the kinds of markets where activity has been concentrated — publicly visible on their profile:

- **US political events** — elections, appointments, policy outcomes
- **Macro & economic** — Fed decisions, inflation prints, GDP calls
- **Sports & entertainment** — high-liquidity, fast-resolving markets
- **World events** — geopolitical outcomes with clear resolution criteria

The bot doesn't filter by market type. It mirrors whatever Car trades, whenever they trade it.

---

## Five-Minute Setup

**Prerequisites:** Node.js 18+, MongoDB (local or [Atlas free tier](https://mongodb.com/atlas)), funded Polygon wallet with USDC.

```bash
# 1. Clone and install
git clone https://github.com/LemnLabs/polymarket-trading-bot.git
cd polymarket-trading-bot
npm install

# 2. Configure
cp env.example .env
```

Open `.env` — fill in only three required values to start:

```env
# ── Already set to Car — do not change ─────────────────────────
USER_ADDRESS=0x7C3Db723F1D4d8cB9C550095203b686cB11E5C6B

# ── Your wallet ─────────────────────────────────────────────────
PROXY_WALLET=0xYourWalletAddress
PRIVATE_KEY=your_private_key_here        # never commit this

# ── Leave defaults unless you know what you're changing ─────────
CLOB_HTTP_URL=https://clob.polymarket.com
CLOB_WS_URL=wss://clob-ws.polymarket.com
RPC_URL=https://polygon-rpc.com
USDC_CONTRACT_ADDRESS=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174
MONGO_URI=mongodb://localhost:27017/polymarket_car
FETCH_INTERVAL=1
TOO_OLD_TIMESTAMP=24
RETRY_LIMIT=3
```

```bash
# 3. Run
npm run build && npm start
```

---

## Before You Scale Past $1,000

Do these before increasing capital:

- [ ] Watch the first 10–20 trades execute manually in your terminal
- [ ] Verify MongoDB is logging every detection and fill correctly
- [ ] Check your actual fill prices vs Car's — measure your real slippage
- [ ] Set a monthly loss limit you will actually respect
- [ ] Use a dedicated wallet — not your main holdings
- [ ] Make sure your RPC endpoint is reliable (consider a paid node)

This checklist exists because most people skip it. Don't skip it.

---

## The Honest Section

Copy trading is not passive income. It is active risk with automated execution.

| What people expect | What actually happens |
|---|---|
| "I'll mirror Car's exact returns" | You enter later, at worse prices — returns will be lower |
| "The bot runs itself" | It needs monitoring, a live RPC, and a running MongoDB |
| "Profitable traders stay profitable" | Car will have losing months. You will mirror them. |
| "More capital = more profit" | More capital = more exposure in both directions |

**None of this is financial advice. This software is provided for educational purposes. You are fully responsible for your own capital.**

---

## Under the Hood

```
src/
├── index.ts                   Entry point — env validation + orchestration
├── services/
│   ├── tradeMonitor.ts        Polls wallet, writes new TRADE events to DB
│   └── tradeExecutor.ts       Reads pending trades, places orders
├── utils/
│   ├── postOrder.ts           Buy / sell / merge order logic
│   ├── createClobClient.ts    CLOB API key derivation + client init
│   └── getMyBalance.ts        USDC balance via Polygon RPC
├── models/
│   └── userHistory.ts         MongoDB schema — per-wallet activity
└── config/
    └── db.ts                  MongoDB connection handler
```

**Stack:** TypeScript (strict) · Node.js · `@polymarket/clob-client` · `ethers` · `mongoose` · `axios` · `ora` · `chalk`

---

## Commands

```bash
npm run dev          # Development — hot reload via ts-node
npm run build        # Compile TypeScript → dist/
npm start            # Run compiled build
npm run lint         # ESLint check
npm run lint:fix     # ESLint auto-fix
npm run format       # Prettier format
```

---

## Contributing

Run it. Log your results. Open a PR if you've improved execution speed, sizing logic, slippage measurement, or PnL reporting.

The bot gets more useful the more people stress-test it with real capital.

---

<div align="center">

*Car is trading right now.*
*The only question is whether your bot is running.*

**[⭐ Star this repo](https://github.com/LemnLabs/polymarket-trading-bot)** · **[View Car's wallet](https://polymarket.com/@Car?tab=activity)**

ISC © [LemnLabs](https://github.com/LemnLabs)

</div>

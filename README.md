<div align="center">

[![Node](https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![MongoDB](https://img.shields.io/badge/MongoDB-Required-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://mongodb.com)
[![Network](https://img.shields.io/badge/Polygon-USDC-8247E5?style=flat-square&logo=polygon&logoColor=white)](https://polygon.technology)
[![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://typescriptlang.org)
[![License](https://img.shields.io/badge/License-ISC-blue?style=flat-square)](LICENSE)

</div>

---

**You already know who to follow.**

You've been watching their wallet for weeks.
Every market they enter - they're right more than they're wrong.
You've seen the trades. You've done the math.

The only problem?

By the time you notice they moved, open the app, check the price, and place the order -
the odds have already shifted. The edge is gone. You got the trade wrong.

**This bot closes that gap.**

Paste their wallet address. Press run.
The next time they trade - your order goes in within seconds, automatically, at the right size.

---

## How It Works

The bot watches any Polymarket wallet you point it at. The moment that wallet executes a trade:

```
[02:13:44] Polling target wallet...          no new activity
[02:13:45] Polling target wallet...          no new activity
[02:13:46] Polling target wallet...          ── TRADE DETECTED ──
           Wallet   0xYourTargetWallet...
           Market   Will X happen before Y?
           Side     BUY
           Size     $8,500
           Price    0.44

[02:13:46] Staleness check...                0s old — OK
[02:13:46] Price drift check...              current 0.45 — within threshold
[02:13:46] Sizing your position...           $5,000 ÷ $85,000 × $8,500 = $500
[02:13:47] Submitting order...               FILLED at 0.45
[02:13:47] Writing to MongoDB...             done ✓

[02:13:47] Waiting for next signal...
```

One wallet address in your `.env`. That's all it takes.

---

## Who Are You Following?

This bot works with **any public Polymarket wallet.**

You bring the trader. The bot brings the execution.

Some of the most-watched wallets on Polymarket right now:

| Trader | Profile | Known For |
|---|---|---|
| **Car** | [polymarket.com/@Car](https://polymarket.com/@Car?tab=activity) | Top leaderboard, high-conviction sizing |
| **0x8dxd** | [polymarket.com/@0x8dxd](https://polymarket.com/@0x8dxd?tab=activity) | Highly active, watched by the community |
| **Your whale** | whoever you've been watching | — |

> Find wallets worth following at [polymarket.com/leaderboard](https://polymarket.com/leaderboard) - sort by profit, filter by market type, pick your signal.

You can follow **multiple wallets at once** by comma-separating addresses in `USER_ADDRESS`.

---

## What $5,000 Looks Like

> Illustrative only — based on the proportional sizing model, not a return guarantee. Your actual fills will differ due to slippage and latency.

Assume your target holds $50,000 USDC. You deploy $5,000. Your sizing ratio: **10%.**

| Target's Trade | Your Position | If they're right | If they're wrong |
|---|---|---|---|
| $10,000 | $1,000 | you win proportionally | you lose proportionally |
| $5,000 | $500 | you win proportionally | you lose proportionally |
| $2,000 | $200 | you win proportionally | you lose proportionally |

The bot scales every position to your balance automatically. You never overexpose. You never undersize. You just mirror.

---

## Capital Deployment Guide

| Your Balance | What to Expect | Recommended Stage |
|---|---|---|
| **< $500** | Very small positions - some may fail on thin markets | Testing only |
| **$500 – $2,000** | Functional mirroring, real fills, real data | First 30 days |
| **$2,000 – $5,000** | Meaningful exposure, positions that matter | After reviewing your logs |
| **$5,000 – $20,000** | Full proportional parity on most trades | With your own PnL data |
| **> $20,000** | Large orders can move thin markets - use carefully | Advanced only |

---

## Execution Guardrails

Before any order fires, the bot runs four checks:

| Guard | Threshold | Purpose |
|---|---|---|
| **Staleness filter** | 24h max event age | No acting on old data |
| **Price drift gate** | ≤ 5% from target's fill | No chasing moved markets |
| **Proportional sizing** | Auto-calculated | No overexposure |
| **Retry cap** | 3 attempts max | No infinite loops on failures |
| **Deduplication** | Transaction hash | No double-executing the same trade |
| **Full audit log** | MongoDB | Every event recorded - nothing hidden |

---

## Five-Minute Setup

**Prerequisites:** Node.js 18+, MongoDB, funded Polygon wallet with USDC.

```bash
# 1. Clone and install
git clone https://github.com/ewindmer/polymarket-copy-trading-bot.git
cd polymarket-copy-trading-bot
npm install

# 2. Configure
cp env.example .env
```

Open `.env`. Three values to fill in:

```env
#  Who you're following 
# Single wallet:
USER_ADDRESS=0xTargetWalletAddress

# Multiple wallets (comma-separated):
# USER_ADDRESS=0xWallet1,0xWallet2,0xWallet3

# Your wallet 
PROXY_WALLET=0xYourWalletAddress
PRIVATE_KEY=your_private_key_here        # never commit this

# Leave these as defaults to start
CLOB_HTTP_URL=https://clob.polymarket.com
CLOB_WS_URL=wss://clob-ws.polymarket.com
RPC_URL=https://polygon-rpc.com
USDC_CONTRACT_ADDRESS=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174
MONGO_URI=mongodb://localhost:27017/polymarket_bot
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

- [ ] Watch the first 10–20 trades execute manually in your terminal
- [ ] Verify MongoDB is logging every detection and fill
- [ ] Measure your actual fill prices vs the target's - know your real slippage
- [ ] Set a monthly loss limit and respect it
- [ ] Use a dedicated wallet - not your main holdings
- [ ] Use a reliable RPC endpoint (consider a paid node for production)

---

## The Honest Part

| What people expect | What actually happens |
|---|---|
| "I'll mirror their exact returns" | You enter later, at worse prices - expect lower returns |
| "Set it and forget it" | It needs a live RPC, running MongoDB, and occasional monitoring |
| "Good traders stay good" | Every trader has losing months. You will mirror them. |
| "More capital = more profit" | More capital = larger exposure in both directions |
| "I can follow 10 wallets at once" | Yes, but position sizing gets more complex across wallets |

This software is provided for educational purposes. Not financial advice. Your capital, your responsibility.

---

## Under the Hood

```
src/
├── index.ts                   Entry point - env validation + orchestration
├── services/
│   ├── tradeMonitor.ts        Polls target wallets, writes TRADE events to DB
│   └── tradeExecutor.ts       Reads pending trades, places orders
├── utils/
│   ├── postOrder.ts           Buy / sell / merge order logic
│   ├── createClobClient.ts    CLOB API key derivation + client init
│   └── getMyBalance.ts        USDC balance via Polygon RPC
├── models/
│   └── userHistory.ts         MongoDB schema - per-wallet activity
└── config/
    └── db.ts                  MongoDB connection handler
```

**Stack:** TypeScript (strict) · Node.js · `@polymarket/clob-client` · `ethers` · `mongoose` · `axios`

---

## Contributing

Found a wallet worth following? Improved execution or sizing logic? Added PnL reporting?

Open a PR. The more wallets people run this against, the sharper it gets.

---

<div align="center">

*Someone you're watching just moved.*
*The only question is whether your bot caught it.*

**[⭐ Star this repo](https://github.com/ewindmer/polymarket-copy-trading-bot)** · **[Find wallets on the leaderboard](https://polymarket.com/leaderboard)**

ISC © [ewindmer](https://github.com/ewindmer)

</div>

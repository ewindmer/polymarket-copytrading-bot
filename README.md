<div align="center">
           
# Polymarket Copy Trading Bot
           
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Node](https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Required-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://mongodb.com/)
[![Network](https://img.shields.io/badge/Polygon-USDC-8247E5?style=flat-square&logo=polygon&logoColor=white)](https://polygon.technology)
[![License: ISC](https://img.shields.io/badge/License-ISC-blue?style=flat-square)](https://opensource.org/licenses/ISC)

</div>

---

# You're Losing On Polymarket. You Know It. Let's Talk About Why.

It's not bad luck.

You researched. You reasoned. You placed the trade with conviction.  
And then some wallet you've never heard of took the opposite side, sized in heavy, and resolved green.

Again.

You've seen this wallet before. And the one before that. The same 30–40 addresses sitting at the top of every resolved market leaderboard, week after week, while you're calculating what percentage of your stack you've given back this month.

Here's the uncomfortable truth:

**They're not smarter than you. They're faster. And they never sleep.**

---

## What's Actually Happening To Your Money

Every minute you spend noticing a whale's move, opening the app, checking the current odds, deciding your size, and placing the order -

- the market has already adjusted.

You're not getting their trade. You're getting the trade *after* their trade. The one where the edge is already priced in. The one retail always gets.

The gap between their fill and yours isn't skill. It's time.

| What the whale does | What you do | The gap |
|---|---|---|
| Moves at 2:13:46am | Sees it at 2:17am | **3 min 14 sec** |
| Fills at 0.44 | Checks at 0.51 | **16% worse entry** |
| Sizes $8,500 | Hesitates, sizes $400 | **Fraction of the upside** |

That gap is not closeable manually. Not by you. Not by anyone.

The only fix is a bot that doesn't sleep, doesn't hesitate, and doesn't check Twitter before placing the order.

---

## The Fix

```
[02:13:46] Polling target wallets...         ── TRADE DETECTED ──

           Wallet   0xYourTargetWallet...
           Market   Will X happen before Y?
           Side     BUY
           Size     $8,500
           Price    0.44

[02:13:46] Staleness check...                0s old — OK
[02:13:46] Price drift check...              current 0.45 — within threshold
[02:13:46] Sizing your position...           $5,000 ÷ $85,000 × $8,500 × 2 = $1,000
[02:13:47] Submitting order...               FILLED at 0.45
[02:13:47] Writing to MongoDB...             done ✓
```

**0 minutes of hesitation. 0 emotional second-guessing. 1 second from detection to fill.**

While you were asleep, the whale moved. The bot caught it. Your order went in.  
By the time your alarm goes off, the position is already open and logged.

That's the only edge that actually closes the gap.

---

## Stop Guessing Who To Follow. The Leaderboard Exists.

Polymarket's on-chain data is fully public. Every wallet. Every trade. Every timestamp.  
The top performers aren't hiding. You can see exactly who they are right now at [polymarket.com/leaderboard](https://polymarket.com/leaderboard).

This bot works with **any** of them.

Some of the most-studied wallets right now:

| Trader | Profile | Why They're Watched |
|---|---|---|
| **Car** | [polymarket.com/@Car](https://polymarket.com/@Car?tab=activity) | Consistent top-tier leaderboard, high-conviction sizing |
| **madril** | [polymarket.com/@madril](https://polymarket.com/@madril?tab=activity) | Highly active, widely tracked by the community |
| **Your whale** | The one you've been watching manually | Stop watching. Start mirroring. |

Paste their wallet into `USER_ADDRESS`. Done. The bot handles the rest.  
Follow one. Follow ten. Comma-separate them. The engine treats each as its own independent stream.

---

## "But What If I Don't Have Whale-Sized Capital?"

You don't need it.

The bot sizes your position proportionally to your balance vs theirs - automatically, on every trade.

**Formula:** `your_balance ÷ target_balance × target_position_size × 2`

Assuming the target holds **$50,000** and trades **$10,000**:

| Your Balance | Sizing Ratio | Your Mirrored Position |
|---|---|---|
| $10,000 | 40% | **$4,000** |
| $5,000 | 20% | **$2,000** |
| $3,000 | 12% | **$1,200** |
| $1,000 | 4% | **$400** |

You're not betting their size. You're betting *your* proportional equivalent.  
The math is automatic. The discipline is enforced by code, not willpower.

---

## "What Stops It From Blowing Up My Account?"

Six checks run before any order touches your wallet:

| Guard | What It Does |
|---|---|
| **Staleness filter** | Ignores trades older than 24h - no acting on dead signals |
| **Price drift gate** | Skips if the market moved more than 5% from the target's fill |
| **Proportional sizing** | Keeps your exposure locked to your balance ratio |
| **Retry cap** | Stops at 3 failed attempts - no infinite loops |
| **Deduplication** | Transaction-hash check - no double-executing the same trade |
| **Full audit log** | Every detection and fill written to MongoDB - nothing opaque |

It will not blindly chase a market that's already gone. It will not oversize. It will not retry forever.  
It knows when to say no.

---

## Run It. Right Now.

You've been reading long enough.  
In the time it took you to get here, somewhere on Polymarket, a sharp wallet just moved.

```bash
git clone https://github.com/epsil1on/polymarket-copy-trading-bot.git
cd polymarket-copy-trading-bot
npm install
cp env.example .env
```

Three values in `.env`. That's all that stands between you and a running bot:

```env
# The wallet you're following
USER_ADDRESS=0xTargetWalletAddress

# Your wallet
PROXY_WALLET=0xYourWalletAddress
PRIVATE_KEY=your_private_key_here        # never commit this

# Leave everything else as default
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
npm run build && npm start
```

**Prerequisites:** Node.js 18+, MongoDB, Polygon wallet with USDC.

---

## Before You Scale Past $10,000

Most people skip this. Most people also wonder why their results are inconsistent.

- [ ] Watch the first 10-20 trades execute live in your terminal
- [ ] Verify MongoDB is logging every detection and fill correctly
- [ ] Compare your actual fill prices vs the target's - measure real slippage
- [ ] Set a hard monthly loss limit before you're emotional about it
- [ ] Run from a dedicated wallet - never your main holdings
- [ ] Use a paid RPC node - free endpoints go down at the worst times

Do this list. Then scale.

---

## The Part Everyone Else Buries In Fine Print

| What you want to believe | What is actually true |
|---|---|
| "I'll mirror their exact returns" | You enter after them, at worse prices. Your returns will be lower. |
| "Set it and forget it" | It needs a live RPC, a running MongoDB, and occasional eyes on it |
| "Great traders stay great" | Every wallet has a bad month. You will mirror it. |
| "More capital, more upside" | More capital, more exposure - in both directions |
| "I'll follow 10 wallets at once" | You can. Position sizing across that many wallets needs monitoring. |

No guarantees. No promises. Your capital. Your responsibility.  
This software is provided for educational purposes only. Nothing here is financial advice.

---

## How It's Built

```
src/
├── index.ts                   Entry point - env validation + orchestration
├── services/
│   ├── tradeMonitor.ts        Polls target wallets, writes TRADE events to DB
│   └── tradeExecutor.ts       Reads pending trades, places mirrored orders
├── utils/
│   ├── postOrder.ts           Buy / sell / merge logic + price guards + retries
│   ├── createClobClient.ts    CLOB API key derivation + client init
│   └── getMyBalance.ts        USDC balance via Polygon RPC
├── models/
│   └── userHistory.ts         MongoDB schema — per-wallet activity collections
└── config/
    └── db.ts                  MongoDB connection handler
```

**Stack:** TypeScript (strict) · Node.js · `@polymarket/clob-client` · `ethers` · `mongoose` · `axios`

Your `PRIVATE_KEY` signs orders. Nothing else. No telemetry. No external calls.  
The signing logic is in `src/utils/createClobClient.ts`. Read it before you fund.

---

## Contributing

Ran it with real capital? Improved execution speed, slippage measurement, or PnL tracking?  
Open a PR. The people who run this seriously make it sharper for everyone.

---

<div align="center">

**The whale just moved.**

**Your only question is whether your bot was running.**

[⭐ Star this repo](https://github.com/epsil1on/polymarket-copy-trading-bot) · [Find your whale](https://polymarket.com/leaderboard)

ISC © [epsil1on](https://github.com/epsil1on)

</div>

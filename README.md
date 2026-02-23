# Polymarket CopyTrading Bot | Polymarket Copy Trading Bot

- A Node.js bot that **copies trades** from a target Polymarket user to your own wallet. It monitors the target’s activity on [Polymarket](https://polymarket.com) (including weather and other prediction markets), stores new trades in MongoDB, and mirrors them on your proxy wallet with size scaled by your balance ratio.
It enables on **Weather**, **Sports**, **Politics**, **Crypto** markets

---

## What it does

- **Trade monitor** – Polls Polymarket’s data API for the target user’s trades and saves new ones to MongoDB.
- **Trade executor** – Reads unexecuted/copyable trades from the DB and places matching orders on the CLOB (Polygon) with your proxy wallet.
- **Strategies** – Copies **BUY**, **SELL**, and **MERGE** (e.g. redeeming/merging outcome tokens). Your order size is scaled by the ratio of your balance to the target’s (so you don’t over-size).
- **Retries** – Failed orders are retried up to `RETRY_LIMIT` times before the trade is marked as done.

The bot uses the **Polymarket CLOB client** (Polygon, chain ID 137), **USDC** for sizing, and **MongoDB** for trade history and bot config.

---

## Requirements

- **Node.js** (v16+)
- **MongoDB** (local or hosted)
- A **Polygon wallet** with USDC for the proxy (your copy-trading account)
- **Target user’s Ethereum address** (the Polymarket account you want to copy)

---

## Setup

### 1. Clone and install

```bash
git clone <repo-url>
cd polymarket-copy-trading-weather-market
npm install
```

### 2. Environment variables

Create a `.env` in the project root with:

| Variable | Required | Description |
|----------|----------|-------------|
| `USER_ADDRESS` | Yes | Polymarket address of the **user you copy** (target). |
| `PROXY_WALLET` | Yes | **Your** wallet address (the one that will place copy trades). |
| `PRIVATE_KEY` | Yes | Private key of the proxy wallet (used to sign CLOB orders). |
| `CLOB_HTTP_URL` | Yes | Polymarket CLOB API base URL (e.g. `https://clob.polymarket.com`). |
| `CLOB_WS_URL` | Yes | Polymarket CLOB WebSocket URL (e.g. `wss://ws-subscriptions-clob.polymarket.com/ws/market`). |
| `MONGO_URI` | Yes | MongoDB connection string (e.g. `mongodb://localhost:27017/polymarket_copytrading`). |
| `RPC_URL` | Yes | Polygon RPC URL (e.g. from Alchemy/Infura). |
| `USDC_CONTRACT_ADDRESS` | Yes | USDC contract on Polygon. |
| `FETCH_INTERVAL` | No | Seconds between polling for new trades (default: `1`). |
| `TOO_OLD_TIMESTAMP` | No | Ignore trades older than this many hours (default: `24`). |
| `RETRY_LIMIT` | No | Max retries per order (default: `3`). |

Example `.env`:

```env
USER_ADDRESS=0x...
PROXY_WALLET=0x...
PRIVATE_KEY=0x...
CLOB_HTTP_URL=https://clob.polymarket.com
CLOB_WS_URL=wss://ws-subscriptions-clob.polymarket.com/ws/market
MONGO_URI=mongodb://localhost:27017/polymarket_copytrading
RPC_URL=https://polygon-mainnet.g.alchemy.com/v2/YOUR_KEY
USDC_CONTRACT_ADDRESS=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174
```

**Security:** Never commit `.env` or share your `PRIVATE_KEY`. The proxy wallet holds real USDC on Polygon.

### 3. MongoDB

The app expects MongoDB to be reachable at the URI you set. **Note:** The code in `src/config/db.ts` may use a hardcoded URI; if your connection fails, ensure that file uses `ENV.MONGO_URI` (or `process.env.MONGO_URI`) so your `.env` value is used.

### 4. Build and run

```bash
npm run build
npm start
```

For development (run TypeScript without building):

```bash
npm run dev
```

---

## How it works

1. **Startup**  
   Connects to MongoDB, ensures bot config exists for `PROXY_WALLET`, creates a Polymarket CLOB client (and API key if needed), then starts two loops:
   - **Trade monitor** – Every `FETCH_INTERVAL` seconds, fetches activities for `USER_ADDRESS`, filters to `TRADE` type, and inserts new trades (by `transactionHash`) that are within `TOO_OLD_TIMESTAMP` hours.
   - **Trade executor** – Every second, loads trades that are still “copyable” (not yet executed by the bot or under retry limit), then for each trade:
     - Fetches your and the target’s positions and USDC balances.
     - Decides action: **buy**, **sell**, or **merge**.
     - Calls `postOrder` to place CLOB orders (FOK market orders) with size scaled by balance ratio and with basic price checks (e.g. don’t buy if ask is 0.05 above the target’s price).
   - Marks trades as executed (or max retries) in MongoDB so they aren’t processed again.

2. **Markets**  
   The bot is market-agnostic: it copies whatever markets the target trades (e.g. weather, politics, crypto). No special “weather market” logic—it works for any Polymarket market the CLOB supports.

---

## Project structure (main pieces)

- `src/index.ts` – Entry: DB connect, config sync, start monitor + executor.
- `src/services/tradeMonitor.ts` – Fetches and stores target user’s new trades.
- `src/services/tradeExecutor.ts` – Reads stored trades and executes copy orders.
- `src/utils/postOrder.ts` – BUY/SELL/MERGE order logic and CLOB calls.
- `src/utils/createClobClient.ts` – Builds Polymarket CLOB client (Polygon, proxy wallet).
- `src/config/env.ts` – Loads and validates `.env`.
- `src/config/db.ts` – MongoDB connection.
- `src/models/` – Mongoose models for bot config and per-user trade history.

---

## Scripts

| Command | Description |
|--------|-------------|
| `npm run dev` | Run with `ts-node` (no build). |
| `npm run build` | Compile TypeScript to `dist/`. |
| `npm start` | Run compiled `dist/index.js`. |
| `npm run lint` | Run ESLint. |
| `npm run lint:fix` | ESLint with auto-fix. |
| `npm run format` | Prettier format. |

---

## License

ISC (see `package.json`). Author: ewindmer.

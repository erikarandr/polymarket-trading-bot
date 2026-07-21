<div align="center">

# polymarket trading bot

**A high-performance, type-safe trading engine for Polymarket - built in Node.js + TypeScript.**

Detects and executes on YES/NO mispricings, crypto Up/Down markets (BTC, ETH, SOL), and cross-venue inefficiencies in real time.

[![Node.js](https://img.shields.io/badge/Node.js-20+-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)
[![Status](https://img.shields.io/badge/status-active--development-blue.svg)](#roadmap)

[![Polymarket Profile](https://img.shields.io/badge/Polymarket-@ratue-1652F0)](https://polymarket.com/@ratue)
[![Twitter Follow](https://img.shields.io/badge/Twitter-@dontoverfit-000000?logo=x&logoColor=white)](https://x.com/dontoverfit)

https://github.com/user-attachments/assets/a97d1a7c-76db-49d8-a597-94f5f5c76012

</div>

---

<table>
  <tr>
    <td align="center" width="50%">
      <b>Live Dashboard UI</b><br><br>
      <img src="https://github.com/user-attachments/assets/48095fe0-e960-456b-b756-364907f05b3e" alt="Live trading dashboard" width="100%">
    </td>
    <td align="center" width="50%">
      <b>Performance & Opportunities</b><br><br>
      <img src="https://github.com/user-attachments/assets/554f3ddc-31e6-4ef2-bd6c-63f26f4b514d" alt="Performance and opportunity tracking" width="100%">
    </td>
  </tr>
</table>

---

## Table of Contents

- [Why This Bot](#-why-this-bot)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Quick Start](#-quick-start)
- [Configuration](#-configuration)
- [Strategies](#-strategies)
- [Risk Management](#-risk-management)
- [Performance & Transparency](#-performance--transparency)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [Disclaimer](#-disclaimer)
- [License](#-license)

---

## Why This Bot

Prediction markets like Polymarket move fast, and pricing inefficiencies between YES/NO legs - or between Polymarket and other venues - tend to close within seconds. This project exists to make that detection-and-execution loop **fast, observable, and safe to operate**, instead of relying on manual monitoring or fragile scripts.

It's built around three principles:

| Principle | What it means in practice |
|---|---|
| **Speed** | Async WebSocket pipelines, no polling where a stream is available |
| **Safety** | Explicit exposure limits, slippage guards, and kill-switches before execution |
| **Transparency** | Every trade, signal, and skipped opportunity is logged and inspectable |

---

## Features

- **Real-Time trading Scanner** - continuously monitors Polymarket 5-minute Up/Down contracts and other markets for pricing mismatches.
- **Low-Latency Execution Engine** - async/await pipelines, persistent WebSocket connections, and optimized RPC batching.
- **Modular Strategy System**:
  - Intra-market trading (YES/NO parity breaks)
  - Cross-venue price discrepancies
  - Short-term momentum on crypto Up/Down markets
  - Merge/redeem liquidity optimizer
- **Built-In Risk Management** - position sizing, stop logic, slippage protection, and configurable per-market exposure caps.
- **Multi-Channel Alerts** - Telegram, Discord, or custom webhooks for fills, errors, and opportunity alerts.
- **Live Dashboard** - optional Express + WebSocket UI for real-time P&L, open positions, and opportunity feed.
- **Type-Safe End to End** - Zod-validated inputs/outputs across the entire pipeline, so bad data fails loud, not silent.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js 20+ |
| Language | TypeScript |
| Blockchain | ethers.js / viem |
| Market data | Polymarket SDK, REST + WebSocket |
| Queuing | BullMQ |
| Logging | Winston |
| Validation | Zod |
| Dashboard | Express + WebSocket |

---

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌───────────────────┐
│  Market Data    │────▶│  Signal Engine  │────▶│  Risk Manager     │
│  (WS + REST)    │     │  (strategies)    │     │  (sizing/limits)  │
└─────────────────┘     └──────────────────┘     └─────────┬─────────┘
                                                           │
                          ┌──────────────────┐             ▼
                          │  Notifications   │◀────┌───────────────────┐
                          │ (TG/Discord/Hook)│      │  Execution Engine │
                          └──────────────────┘      │  (ethers/viem)    │
                                                    └───────────────────┘
```

---

## Quick Start

```bash
git clone https://github.com/erikarandr/polymarket-trading-bot.git
cd polymarket-trading-bot
npm install
cp .env.example .env
npm run build
npm run start
```

---

## Configuration

Set these in your `.env` file:

```env
# Wallet & network
PRIVATE_KEY=your_wallet_private_key
RPC_URL=https://polygon-rpc.com

# Notifications
TELEGRAM_BOT_TOKEN=...
TELEGRAM_CHAT_ID=...

# Risk controls
MAX_POSITION_USD=500
MAX_DAILY_LOSS_USD=100

```

---

## Strategies

| Strategy | Description |
|---|---|
| **5-Min Up/Down Arb** | Core focus on BTC/ETH/SOL short-duration contracts |
| **YES/NO Price Discrepancy** | Classic trading when `YES + NO < $1` |
| **Cross-Platform** | Compares Polymarket pricing against other venues |
| **Statistical / Momentum** | Pluggable custom signal layer |
| **Liquidity Optimization** | Automated merge/redeem to free up capital |

Each strategy is a self-contained module - enable, disable, or tune thresholds independently in `config/strategies.ts`.

---

## Risk Management

- Configurable **max position size** and **max daily loss** limits
- **Slippage protection** on every execution path
- **Stop logic** to halt trading on abnormal market conditions or repeated failed fills
- **Dry-run mode** to validate strategy behavior with zero capital at risk

---

## Performance & Transparency

- Historical run logs are stored in `/results` for independent review.
- On-chain wallet activity can be tracked directly on Polygonscan for full transparency - no black-box claims.
- Results vary by market conditions, latency, and configuration; past logs are not a guarantee of future performance.

### Author's Track Record

Rather than quoting a P&L screenshot that goes stale the moment markets move, the maintainer's live Polymarket activity is public and verifiable in real time:

- **Polymarket profile:** [polymarket.com/@ratue](https://polymarket.com/@ratue) - live positions, P&L, and trade history
- **Twitter/X:** [@dontoverfit](https://x.com/dontoverfit) - build updates, strategy notes, and market commentary

> These links show real, current on-chain activity rather than a fixed number in this README - check them directly for up-to-date performance.

**Snapshot (as of July 8, 2026):**

| Metric | Value |
|---|---|
| Predictions made | 33,753 |
| Positions value | $876.57 |
| Biggest win | $2,036.37 |
| P&L (past day) | +$935.53 |
| Profile views | 22.7K |

> This table is a point-in-time snapshot and will not update automatically. For current figures, always check the [live profile](https://polymarket.com/@ratue).

---

## Roadmap

- [ ] Advanced ML signals (TensorFlow.js or external model integration)
- [ ] Rust core for ultra-low-latency execution
- [ ] Expanded market support (politics, sports, perps)
- [ ] Docker + Kubernetes deployment
- [ ] Public API for community-contributed strategies

---

## Contributing

Contributions are welcome. Open an issue or PR for new strategies, bug fixes, or performance improvements. Please include tests and a clear description of the change.

Questions, feedback, or strategy ideas - reach out on [Telegram](https://t.me/dontoverfit) or check out live trading activity on [Polymarket](https://polymarket.com/@ratue).

---

## Disclaimer

Automated trading on prediction markets carries substantial risk of loss, including total loss of deployed capital. Smart contract, liquidity, and execution risks all apply. This project is provided **as-is**, with no guarantee of profitability. **This is not financial advice.** Start with small sizes, use dry-run mode first, and do your own research.

---

## License

MIT License - free to use and modify with attribution. Thanks you

---

<div align="center">

**Keywords:** polymarket trading bot · polymarket arb bot · polymarket trading bot typescript · node.js polymarket bot · prediction market trading · crypto up/down bot · polymarket automation · defi trading bot

</div>

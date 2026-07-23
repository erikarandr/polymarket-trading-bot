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

https://github.com/user-attachments/assets/84301f19-c3b2-4e6a-b678-397b5297a342

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

Every Polymarket market resolves to exactly one outcome, and its complementary shares (YES/NO, or Up/Down) are designed so that a complete pair - one share of each - redeems for exactly $1. Every strategy below is a different way of exploiting a *temporary* gap between what the market is charging for that pair right now and the $1 it's worth at resolution. None of them predict direction; they all depend on finding and filling the gap before it closes.

### 1. YES/NO Price Discrepancy

**The mechanic:** in a healthy market, `price(YES) + price(NO)` should sit at or very close to $1.00. Thin liquidity, a burst of one-sided order flow, or a stale quote can briefly push that sum below $1.00.

**The trade:** buy both YES and NO whenever their combined cost is under $1.00. Whichever side resolves true, the pair redeems for $1.00 - so the entry discount (`$1.00 - combined cost`) is the theoretical profit, before fees and slippage.

**Example:** YES quoted at $0.46, NO quoted at $0.51 → combined cost $0.97. Buying one share of each for $0.97 and redeeming the winning side for $1.00 nets $0.03/share before costs.

**What actually determines the outcome:** the gap closes fast once it's visible, so this is a race against other bots and market makers for the same fill. Fees, gas, and slippage on the *second* leg (the one that hasn't filled yet when you send the order) eat directly into that $0.03. In practice a chunk of "discrepancies" the scanner sees are already gone, or only fillable at worse prices, by the time an order lands - see [Risk Management](#-risk-management) for the guards that exist because of this.

### 2. 5-Minute Up/Down Arb (BTC/ETH/SOL)

**The mechanic:** same underlying idea as #1, applied specifically to the short-duration crypto Up/Down markets, which resolve every 5 minutes and see the highest volume and fastest-moving order books on the platform. Because these markets reset constantly, mispricings between the Up and Down legs open and close far more often than in longer-dated markets - more opportunities, but also more noise and more competition for each one.

**The trade:** the same dual-leg entry as #1 (buy both legs when combined cost < $1.00), but tuned specifically for this market's speed: tighter timing windows, faster fill logic, and price bands set for how quickly these particular order books move.

**What actually determines the outcome:** fill timing dominates here more than in slower markets. A combined cost that looks attractive at scan time can be gone in the time it takes to route two orders, and a sharp last-second BTC/ETH/SOL move can push the "cheap" leg to fill at a materially worse price than it showed on the scanner.

### 3. Cross-Platform Discrepancy

**The mechanic:** the same event can sometimes be listed - or have a close proxy - on more than one venue, with each venue's order book pricing it independently. Those prices can drift apart when one venue's liquidity or order flow moves faster than the other's.

**The trade:** compare Polymarket's price for an outcome against the equivalent price elsewhere, and take the position on whichever venue is mispriced relative to the other.

**What actually determines the outcome:** this is the least "clean" strategy of the set - it depends on the two markets actually being equivalent (not just similar), on both venues having enough liquidity to fill at the quoted price, and on execution being fast enough on both legs that the gap hasn't already closed by the time both orders land. Venue-specific fees, withdrawal/settlement timing, and contract-wording differences (a market can resolve on subtly different terms across venues) are real risks specific to this strategy that don't apply to the single-venue ones above.

### 4. Statistical / Momentum

**The mechanic:** a pluggable signal layer, separate from the parity-based strategies above. Instead of trading a pricing gap, it looks for short-term directional signals (e.g., order-flow imbalance, recent price momentum) and takes a one-sided position based on them.

**The trade:** this is the one strategy in the set that *is* a directional bet rather than a hedged pair - it's intentionally left as a pluggable module (`config/strategies.ts`) rather than a tuned default, since a good momentum signal is highly market- and timeframe-specific.

**What actually determines the outcome:** this carries meaningfully different risk than the other three - there's no "$1.00 pair" backstop if the direction is wrong. Treat any custom signal you plug in here as unproven until you've validated it in dry-run mode against real order flow.

### 5. Liquidity Optimization (Merge/Redeem)

**The mechanic:** this isn't a source of trading profit - it's plumbing. When the bot ends up holding both YES and NO shares in the same market (a "complete set"), those shares can be merged and redeemed for $1.00 combined, rather than sold individually back into the order book (which usually means eating the spread twice).

**The trade:** automatically detect complete sets across open positions and merge/redeem them, freeing the $1.00 in capital to be redeployed instead of sitting idle in matched positions.

**What actually determines the outcome:** the benefit here is capital efficiency and avoided spread cost, not new profit - it's what makes the other four strategies compound faster rather than leaving capital tied up.

---

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
- **Wallet:** `0x912a58103662ebe2e30328a305bc33131eca0f92` - viewable on [Polygonscan](https://polygonscan.com/address/0x912a58103662ebe2e30328a305bc33131eca0f92)
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

# Prediction Markets

## What Are Prediction Markets?

Prediction markets are exchange-traded markets where participants buy and sell shares in the outcome of future events. The price of a share reflects the crowd's probability estimate of that outcome occurring.

---

## How They Work

### Binary Markets

AnonMarkets uses binary markets with two outcomes: **YES** and **NO**.

- **YES shares** pay out if the event happens
- **NO shares** pay out if the event doesn't happen

### Share Pricing

Share prices range from 0 to 1 SOL:
- A YES share priced at **0.70 SOL** implies a **70% probability** the event will occur
- A NO share priced at **0.30 SOL** implies a **30% probability** the event won't occur

**Key property:** YES price + NO price = 1.00 SOL (always)

### Payouts

When a market resolves:
- Winning shares pay out **1 SOL each**
- Losing shares pay out **0 SOL**

**Example:**
- You buy 10 YES shares at 0.40 SOL each (cost: 4 SOL)
- The event happens (YES wins)
- You receive 10 SOL
- Profit: 6 SOL (150% return)

---

## Why Prediction Markets?

### Information Aggregation

Prediction markets aggregate diverse information from many participants. Each trader brings their own knowledge, research, and insights. The resulting price is often more accurate than polls, expert opinions, or individual forecasts.

### Incentive Alignment

Unlike polls or surveys, prediction markets put real money at stake. Participants are incentivized to:
- Research thoroughly before betting
- Update their beliefs when new information emerges
- Bet proportionally to their confidence

### Real-Time Updates

Prices update instantly as new information becomes available. When news breaks, traders immediately incorporate it into their bets, moving the market price.

---

## Market Lifecycle

```
1. CREATION
   ↓
   Market is created with a question and resolution criteria
   Initial odds are set by early traders

2. TRADING
   ↓
   Users buy/sell YES and NO shares
   Prices move based on supply and demand
   Anyone can enter or exit positions

3. RESOLUTION
   ↓
   Event outcome becomes known
   Market is resolved to YES or NO

4. SETTLEMENT
   ↓
   Winning shares pay out 1 SOL
   Losing shares worth 0
   Users claim their winnings
```

---

## Example Markets

| Market | Current YES Price | Implied Probability |
|--------|-------------------|---------------------|
| "Bitcoin above $100k by Dec 2024" | 0.65 SOL | 65% |
| "Rain in NYC tomorrow" | 0.30 SOL | 30% |
| "Team A wins championship" | 0.45 SOL | 45% |

---

## Trading Strategies

### Going Long
Buy shares if you think the market underestimates the probability.

**Example:** Market shows 30% chance of rain, but you checked multiple weather models showing 50%. Buy YES shares at 0.30, potentially sell at 0.50 or hold until resolution.

### Going Short
Sell (or buy the opposite side) if you think the market overestimates.

**Example:** Market shows 80% chance of a product launch, but you have insider knowledge of delays. Buy NO shares at 0.20.

### Arbitrage
If YES + NO doesn't equal 1.00 (rare in AMM markets), buy both sides and lock in risk-free profit.

---

## Prediction Markets vs. Gambling

| Aspect | Prediction Markets | Traditional Gambling |
|--------|-------------------|---------------------|
| House edge | None (peer-to-peer) | 5-15% |
| Price discovery | Dynamic, market-driven | Fixed by bookmaker |
| Information value | Aggregates knowledge | Entertainment only |
| Exit options | Sell anytime | Usually locked in |

---

## Next Steps

- [How the Bonding Curve Works](bonding-curve.md)
- [Understanding Privacy](privacy-model.md)

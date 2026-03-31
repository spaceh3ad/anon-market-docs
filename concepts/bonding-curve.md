# Bonding Curve AMM

## Overview

AnonMarkets uses a **bonding curve AMM** (Automated Market Maker) to price shares. Unlike order book exchanges, there's always liquidity - you can buy or sell at any time.

---

## The Pricing Formula

The price of YES shares is determined by the ratio of shares:

```
price_yes = yes_shares / (yes_shares + no_shares)
price_no  = no_shares / (yes_shares + no_shares)
```

**Key properties:**
- `price_yes + price_no = 1.0` (always)
- Prices range from 0 to 1
- More demand for one side = higher price for that side

---

## How It Works

### Initial State

When a market is created, it starts with equal shares on both sides:

```
yes_shares: 1000
no_shares:  1000

price_yes = 1000 / 2000 = 0.50 (50%)
price_no  = 1000 / 2000 = 0.50 (50%)
```

### After Buying YES

When someone buys YES shares:
1. They add SOL to the pool
2. The pool mints new YES shares for them
3. The ratio shifts, increasing YES price

```
yes_shares: 1500  (+500 bought)
no_shares:  1000

price_yes = 1500 / 2500 = 0.60 (60%)
price_no  = 1000 / 2500 = 0.40 (40%)
```

### After Buying NO

Same mechanics apply when buying NO:

```
yes_shares: 1500
no_shares:  1300  (+300 bought)

price_yes = 1500 / 2800 = 0.536 (53.6%)
price_no  = 1300 / 2800 = 0.464 (46.4%)
```

---

## Cost Calculation

### Buying Shares

The cost to buy shares uses the **constant product formula**:

```
cost = pool_before - pool_after

Where:
pool = yes_shares * no_shares (constant product invariant)
```

**Example: Buying 100 YES shares**

```
Before:
  yes_shares = 1000
  no_shares = 1000
  pool = 1,000,000

After:
  yes_shares = 1100 (+100)
  no_shares = 1,000,000 / 1100 = 909.09

Cost = 1000 - 909.09 = 90.91 SOL
```

### Selling Shares

Selling works in reverse - you return shares and receive SOL based on the same formula.

---

## Price Impact

Larger trades move the price more. This is called **slippage** or **price impact**.

| Trade Size | Approximate Price Impact |
|------------|-------------------------|
| 1% of pool | ~1% |
| 5% of pool | ~5% |
| 10% of pool | ~10% |
| 50% of pool | ~33% |

**Tip:** For large trades, consider breaking them into smaller chunks.

---

## Effective Price vs. Spot Price

- **Spot price**: The current market price (for infinitely small trades)
- **Effective price**: What you actually pay per share (includes slippage)

```
spot_price = 0.50
You buy 100 shares for 55 SOL

effective_price = 55 / 100 = 0.55 per share
slippage = 0.55 - 0.50 = 0.05 (10%)
```

---

## Visual Example

```
Price Movement When Buying YES
==============================

Start: 50/50
         YES ████████████████████ 50%
         NO  ████████████████████ 50%

Buy YES (small):
         YES ██████████████████████ 55%
         NO  ██████████████████ 45%

Buy YES (more):
         YES ████████████████████████ 60%
         NO  ████████████████ 40%

Buy YES (large):
         YES ██████████████████████████████ 75%
         NO  ██████████ 25%
```

---

## Benefits of Bonding Curve AMM

### Always Liquid
Unlike order books, you can always trade. No waiting for counterparties.

### No Order Management
Just specify how much you want to buy/sell. No limit orders, no partial fills.

### Transparent Pricing
The formula is public. Anyone can calculate the exact cost before trading.

### Self-Balancing
Prices automatically adjust to reflect supply and demand.

---

## Comparison with Other AMMs

| Feature | AnonMarkets (CPMM) | Uniswap (CPMM) | Order Book |
|---------|-------------------|----------------|------------|
| Always liquid | Yes | Yes | No |
| Price discovery | Automatic | Automatic | Manual |
| Slippage | Yes | Yes | Varies |
| MEV resistant | Partially | No | Partially |
| Gas efficient | Yes | Yes | No |

---

## Advanced: The Math

For those interested in the exact formulas:

```
// Constant Product Market Maker
invariant = yes_shares * no_shares

// Buy YES shares
new_yes = yes_shares + shares_bought
new_no = invariant / new_yes
cost = no_shares - new_no

// Sell YES shares
new_yes = yes_shares - shares_sold
new_no = invariant / new_yes
proceeds = new_no - no_shares
```

---

## Next Steps

- [Privacy Model](privacy-model.md)
- [Market Resolution](resolution.md)

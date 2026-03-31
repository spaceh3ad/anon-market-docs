# Market Resolution

## Overview

Market resolution is the process of determining the outcome of a prediction market. AnonMarkets uses a multi-layered approach combining AI verification with human oversight.

---

## Resolution Flow

```
Market Expires
      ↓
┌─────────────────────────────────────┐
│  AI Agent Proposes Resolution       │
│  (Confidence-based)                 │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  24-Hour Challenge Period           │
│  Anyone can dispute with stake      │
└─────────────────────────────────────┘
      ↓
   Challenged?
   /        \
  No        Yes
  ↓          ↓
Finalize   Creator Reviews
           (or community vote)
```

---

## AI-Assisted Resolution

When a market expires, an AI agent analyzes available information:

### What the AI Does

1. **Fetches market question** and resolution criteria
2. **Searches web sources** for relevant information
3. **Analyzes multiple sources** for consensus
4. **Proposes outcome** (YES or NO) with confidence score
5. **Provides evidence** (sources, quotes, timestamps)

### Confidence Thresholds

| Confidence | Action |
|------------|--------|
| > 85% | Auto-propose resolution |
| 70-85% | Propose with "needs review" flag |
| < 70% | Escalate to manual review |

### Example

```
Market: "Bitcoin above $100k by March 31, 2024"

AI Analysis:
- Checked: CoinGecko, CoinMarketCap, Binance, Kraken
- BTC price on March 31: $97,432
- Confidence: 98%
- Proposed resolution: NO

Evidence:
- CoinGecko historical: $97,432.15
- CoinMarketCap historical: $97,428.00
- Consensus across 4 major sources
```

---

## Challenge Period

After a resolution is proposed, there's a **24-hour window** for disputes.

### How to Challenge

1. Review the proposed resolution
2. If you believe it's incorrect, stake SOL to challenge
3. Provide counter-evidence (links, screenshots, etc.)
4. Challenge is recorded on-chain

### Challenge Requirements

- Minimum stake: 0.1 SOL (subject to change)
- Must provide evidence
- Stake is returned if challenge succeeds
- Stake is forfeited if challenge fails

### Why Stake?

The staking requirement prevents spam challenges. If you're confident the resolution is wrong, you'll stake. If you're just trolling, you'll lose money.

---

## Dispute Resolution

If a market is challenged:

### Phase 1: Creator Review

The market creator reviews the dispute and can:
- **Uphold** the AI resolution (challenger loses stake)
- **Override** to the challenger's outcome (challenger wins stake back)
- **Abstain** and escalate to community vote

### Phase 2: Community Vote (Future)

If the creator abstains or is unavailable:
- Token holders vote on the outcome
- Voting is stake-weighted
- Majority decision wins
- Rewards distributed to correct voters

---

## Resolution States

```
Market States:
─────────────
Open → Trading allowed
  ↓
Expired → Trading stopped, awaiting resolution
  ↓
Proposed → AI proposed outcome, in challenge period
  ↓
Challenged → Someone disputed, under review
  ↓
Resolved → Final outcome confirmed
```

---

## Edge Cases

### No Clear Outcome

If the market question is ambiguous or the outcome is unclear:
- AI flags for manual review
- Creator determines interpretation
- If disputed, community vote decides

### Event Cancelled

If the underlying event is cancelled (e.g., game postponed indefinitely):
- Market may be voided
- All positions refunded at original cost
- No winners or losers

### Information Unavailable

If resolution information isn't available:
- Extended waiting period
- Creator may extend market deadline
- Worst case: market voided

---

## Gaming Prevention

### Manipulation Attempts

- **Fake news**: AI checks multiple sources
- **Insider trading**: On-chain records create accountability
- **Creator collusion**: Challenge mechanism allows disputes

### Sybil Attacks on Voting

Future community voting will include:
- Token-based stake weighting
- Time-locked tokens for voting power
- Reputation scores from accurate predictions

---

## Timeline

| Event | Duration |
|-------|----------|
| Market expires | T+0 |
| AI proposes | T+0 to T+1h |
| Challenge period | 24 hours |
| Creator review (if challenged) | 48 hours |
| Community vote (if escalated) | 72 hours |
| Final resolution | T+1 to T+6 days |

---

## Best Practices for Market Creators

1. **Write clear resolution criteria**
   - Bad: "Will BTC go up?"
   - Good: "Will BTC/USD on CoinGecko be above $100,000 at 00:00 UTC on Dec 31, 2024?"

2. **Specify data sources**
   - Name specific oracles or websites
   - Define tie-breaker sources

3. **Handle edge cases upfront**
   - What if the event is cancelled?
   - What if data is unavailable?

4. **Set appropriate end dates**
   - Allow buffer time after the event
   - Don't expire during weekends/holidays

---

## Next Steps

- [System Architecture](../architecture/README.md)
- [Developer Setup](../developers/local-setup.md)

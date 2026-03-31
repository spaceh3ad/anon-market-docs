# Privacy Model

## Overview

AnonMarkets provides **selective privacy**: market data is public for transparency, while individual positions remain private.

---

## What's Public vs. Private

| Data | Visibility | Rationale |
|------|------------|-----------|
| Market prices (odds) | **Public** | Price discovery requires transparency |
| Total shares per side | **Public** | Shows market liquidity |
| Total pool value | **Public** | Shows market size |
| Deposit transactions | **Public** | On-chain SOL transfers are visible |
| Withdrawal transactions | **Public** | On-chain SOL transfers are visible |
| Your balance | **Private** | Hidden in merkle tree |
| Your positions | **Private** | Hidden in merkle tree |
| Your betting history | **Private** | Events use unlinkable UUIDs |

---

## How Privacy Works

### 1. Compressed Merkle Tree

User balances and positions are stored in a **SPL Compressed Merkle Tree** on Solana:

```
                    Root Hash
                   /         \
             Hash A           Hash B
            /     \          /     \
         Hash    Hash     Hash    Hash
          |       |        |       |
        User1   User2    User3   User4
        Leaf    Leaf     Leaf    Leaf
```

- Only the **root hash** is stored on-chain
- Individual leaves (user data) are **not readable** from the chain
- To prove your balance, you need the **merkle proof** + your leaf data

### 2. MPC State Management

[Lit Protocol](https://litprotocol.com) manages state updates using Multi-Party Computation:

```
User Request → Lit MPC Network → Signed Transaction → Solana
                    ↑
            Verifies merkle proofs
            Computes new state
            Signs with distributed key
```

- No single party sees your full state
- Computation is distributed across multiple nodes
- Only valid state transitions are signed

### 3. UUID-Based Events

Betting events on-chain use **unlinkable UUIDs** instead of wallet addresses:

```typescript
// Your secret salt (stored locally + Arweave backup)
uuid_salt = random_bytes(32)

// Event UUID (unlinkable to your wallet)
user_uuid = keccak256(wallet_pubkey + uuid_salt)
```

- Without your `uuid_salt`, events can't be linked to your wallet
- You can always reconstruct your history using your salt

---

## Privacy Guarantees

### What an observer can see:

1. **Deposit event**: "Address X deposited 10 SOL"
2. **Market activity**: "Someone bought YES, price moved from 0.50 to 0.55"
3. **Withdrawal event**: "Address X withdrew 15 SOL"

### What an observer CANNOT determine:

1. How much of your deposit went to which markets
2. What positions you hold
3. Whether your withdrawal includes profits or losses
4. Which bet events belong to which wallet

---

## Privacy Limitations

### Known Associations

If you deposit 10 SOL and later withdraw 15 SOL, observers know you profited 5 SOL overall. They just don't know which bets produced that profit.

### Timing Correlation

If you deposit, immediately bet on one market, and withdraw - timing analysis could reveal your activity. Mitigation: Vary your deposit/withdraw timing.

### Amount Correlation

Unusual deposit amounts might be linkable to specific bet sizes. Mitigation: Use round numbers.

---

## Data Storage

### On-Chain (Solana)
- Merkle tree root (32 bytes)
- Market pools (aggregated totals)
- Events (with UUIDs, not addresses)

### Off-Chain (Your Browser)
- Your full leaf data (balance, positions)
- Merkle proofs
- UUID salt

### Backup (Arweave via Irys)
- Encrypted copy of your leaf data
- Encrypted UUID salt
- Tagged with your wallet for recovery

---

## Recovery Without Compromising Privacy

If you lose your browser data:

1. Query Arweave for your encrypted backup (tagged with your pubkey)
2. Decrypt using a wallet signature
3. Restore your positions and salt
4. Continue trading with full history

Your backup is encrypted - only you can read it.

---

## Comparison with Other Protocols

| Protocol | Position Privacy | Balance Privacy | Deposit/Withdraw |
|----------|-----------------|-----------------|------------------|
| **AnonMarkets** | Private | Private | Public |
| Polymarket | Public | Public | Public |
| Augur | Public | Public | Public |
| Tornado Cash | N/A | Private | Private |

AnonMarkets provides meaningful privacy for betting activity while maintaining a simpler, non-custodial design.

---

## Threat Model

### What we protect against:
- Other users seeing your positions
- Chain analysts linking bets to wallets
- Front-running your specific trades

### What we don't protect against:
- Lit Protocol operators (they see requests, but are distributed)
- Government subpoenas (deposits/withdrawals are on-chain)
- Your own device being compromised

---

## Next Steps

- [Market Resolution](resolution.md)
- [Architecture Overview](../architecture/README.md)

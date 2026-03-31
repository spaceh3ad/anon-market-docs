# Merkle Trees

## Overview

AnonMarkets uses **two complementary merkle tree structures**:

1. **User State Tree** - Tracks all user balances and positions (~1M users)
2. **Positions Tree** - Tracks individual positions within each user's leaf

This dual-tree architecture enables:
- **Privacy**: Individual balances/bets hidden; only merkle roots visible
- **Scalability**: ~1M users with minimal on-chain storage (~0.5 SOL)
- **Instant settlements**: No per-user on-chain accounts

---

## Architecture

```
                         ON-CHAIN (Solana)
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  GlobalState                                                     │
│  ├── state_nonce: u64                                            │
│  ├── mpc_authority: Pubkey                                       │
│  ├── total_deposited: u64                                        │
│  └── merkle_tree: Pubkey ─────────────┐                          │
│                                       │                          │
│  SPL Compressed Merkle Tree ◄─────────┘                          │
│  ├── depth: 20 (1M leaves)                                       │
│  ├── buffer: 64 concurrent updates                               │
│  └── Queryable via getAssetProof RPC                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Why SPL Account Compression?

We considered storing just a 32-byte merkle root on-chain. However:

| Problem | SPL Account Compression Solution |
|---------|----------------------------------|
| Concurrent updates | Two users updating would invalidate proofs. SPL's changelog buffer handles this. |
| New user registration | Need to track which leaf indices are used. SPL tracks tree structure. |
| Proof availability | If user loses proof, they need to get it somewhere. SPL provides `getAssetProof` RPC. |
| Trustlessness | Alternative is centralized indexer. SPL is on-chain, trustless. |

**Cost:** ~0.5 SOL one-time. Worth it for the benefits.

---

## User State Tree

### Structure

A sparse merkle tree with depth 20 (~1M capacity) where each leaf represents one user's complete state.

### Leaf Structure (88 bytes)

```rust
struct UserLeafData {
    pubkey: [u8; 32],          // User's wallet pubkey
    available_balance: u64,     // SOL available for bets/withdrawal
    locked_balance: u64,        // SOL locked in active positions
    nonce: u64,                 // Replay protection
    positions_root: [u8; 32],   // Merkle root of user's positions
}
```

**Leaf identifier:** `leaf_id = keccak256(pubkey)`

### Storage Locations

| Component | Location | Notes |
|-----------|----------|-------|
| Merkle tree structure | On-chain (SPL) | ~0.5 SOL |
| Merkle proofs | Queryable via RPC | `getAssetProof(leaf_id)` |
| User's leaf data | localStorage | Updated after each action |
| Backup | Arweave (Irys) | Tagged with pubkey |

---

## Positions Tree

### Structure

A per-user merkle tree of betting positions. The root is stored in the user's leaf (`positions_root`).

### Position Structure (57 bytes)

```rust
struct Position {
    market: [u8; 32],    // Market pubkey
    side: u8,            // 0 = YES, 1 = NO
    shares: u64,         // Number of shares owned
    cost_basis: u64,     // Total SOL paid
    bet_nonce: u64,      // Unique identifier
}
```

### Why Two Trees?

Storing full positions on-chain = ~570 bytes per user (10 positions x 57 bytes).
For 1M users = 570MB.

**Solution:** Store only `positions_root` (32 bytes) on-chain. Full position data lives off-chain.

---

## Flow Examples

### Deposit Flow

```
User                          Solana                      Lit
  │                              │                          │
  │ deposit(10 SOL) ────────────►│                          │
  │                              │ Vault += 10 SOL          │
  │◄──────────── tx_sig ─────────│                          │
  │                              │                          │
  │ "Process deposit" ──────────────────────────────────────►
  │ + tx_sig + current_leaf      │                          │
  │                              │◄── getAssetProof ────────│
  │                              │──── proof ──────────────►│
  │                              │   Verify deposit tx      │
  │                              │   Compute new leaf       │
  │                              │   Sign replaceLeaf tx    │
  │◄─────────────────────────────────── signed_tx ──────────│
  │ Submit tx ──────────────────►│                          │
  │◄──────────── confirmed ──────│                          │
```

### Place Bet Flow

```
User                          Lit                         Solana
  │                              │                            │
  │ "Bet 1 SOL YES" ─────────────►                            │
  │ + wallet signature           │── getAssetProof ──────────►│
  │ + current_leaf               │◄── proof ──────────────────│
  │                              │── getMarketState ─────────►│
  │                              │◄── market data ────────────│
  │                              │                            │
  │                              │ 1. Verify wallet signature │
  │                              │ 2. Verify leaf in tree     │
  │                              │ 3. Verify available >= 1   │
  │                              │ 4. Calculate shares (AMM)  │
  │                              │ 5. Update leaf + pools     │
  │                              │ 6. Sign transaction        │
  │◄───────── signed_tx ─────────│                            │
  │ Submit tx ───────────────────────────────────────────────►│
  │◄──────────────────────────────────────── confirmed ───────│
```

---

## Recovery

### Primary: localStorage
```typescript
const state = getStoredUserState(wallet);
```

### Secondary: Irys Tagged Query
```typescript
const query = `{
  transactions(
    tags: [
      { name: "app", values: ["anon-market"] }
      { name: "wallet", values: ["${wallet}"] }
    ]
    sort: HEIGHT_DESC
    first: 1
  ) { edges { node { id } } }
}`;
```

### Last Resort: Emergency Withdrawal
```typescript
// After 24h of Lit being unresponsive
await program.methods.emergencyWithdraw(leafData, proof).rpc();
```

---

## Cost Analysis

### Traditional Approach
| Item | Cost |
|------|------|
| Account rent per user | ~0.002 SOL |
| 1M users | 2,000 SOL (~$400,000) |

### Merkle Tree Approach
| Item | Cost |
|------|------|
| GlobalState account | ~0.003 SOL |
| SPL compressed tree | ~0.5 SOL |
| Per-action (leaf update) | ~0.000005 SOL |
| Irys backup | FREE (<100KB) |
| **Total for 1M users** | **~0.5 SOL** |

**Savings: 99.97%**

---

## Constants

```rust
pub const MERKLE_TREE_DEPTH: usize = 20;         // ~1M users
pub const COMPRESSED_TREE_BUFFER_SIZE: u32 = 64; // Concurrent updates
pub const MAX_POSITIONS: usize = 10;             // Per user
pub const DEFAULT_EMERGENCY_TIMEOUT: i64 = 86400; // 24 hours
```

---

## Next Steps

- [Lit Protocol Integration](lit-protocol.md)
- [Developer Setup](../developers/local-setup.md)

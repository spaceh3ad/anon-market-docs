# State Tree Architecture

## Overview

The MPC State Tree is the core privacy mechanism. User balances and positions are stored in a compressed merkle tree, making individual data invisible while remaining verifiable.

---

## Data Structures

### UserLeafData (On-Chain, 88 bytes)

```rust
pub struct UserLeafData {
    pub user_pubkey: [u8; 32],      // Wallet identity
    pub available_balance: u64,     // Can withdraw
    pub locked_balance: u64,        // In active bets
    pub nonce: u64,                 // Replay protection
    pub positions_root: [u8; 32],   // Merkle root of positions
}
```

### Position (localStorage only)

```typescript
interface Position {
  market: string;       // Market pubkey (base58)
  side: 0 | 1;          // 0=YES, 1=NO
  shares: bigint;       // Number of shares owned
  cost_basis: bigint;   // Total SOL paid (lamports)
  bet_nonce: number;    // Unique per position
}
```

### UUID Derivation

```typescript
// User's secret salt (generated once, stored in backup)
const uuid_salt = crypto.randomBytes(32);

// Derive UUID for events (unlinkable to pubkey without salt)
const user_uuid = keccak256(
  Buffer.concat([wallet.publicKey.toBytes(), uuid_salt])
);
```

---

## Privacy Model

| Data | Visibility | Who Can Link |
|------|------------|--------------|
| Deposits/Withdrawals | Public (wallet visible) | Everyone |
| User balances | Hidden (in merkle tree) | Only user |
| Bet events | Public (UUID only) | Only user (has salt) |
| Market pools | Public | Everyone |
| Individual positions | Hidden | Only user |

---

## State Transitions

### User Lifecycle

```
┌──────────────┐
│  NEW USER    │
│  (no leaf)   │
└──────┬───────┘
       │ First deposit
       ▼
┌──────────────┐
│   ACTIVE     │◄────────────────┐
│  (has leaf)  │                 │
└──────┬───────┘                 │
       │                         │
       ├── Deposit ──────────────┤
       │   available += amount   │
       │                         │
       ├── Place Bet ────────────┤
       │   available -= cost     │
       │   locked += cost        │
       │   positions_root updated│
       │                         │
       ├── Exit Bet ─────────────┤
       │   locked -= cost_basis  │
       │   available += proceeds │
       │   positions_root updated│
       │                         │
       ├── Claim Win ────────────┤
       │   locked -= cost_basis  │
       │   available += payout   │
       │   positions_root updated│
       │                         │
       ├── Withdraw ─────────────┤
       │   available -= amount   │
       │   SOL sent to wallet    │
       │                         │
       └── Emergency Withdraw ───┘
           (if MPC offline 24h+)
```

---

## Sync Strategy

### When to Sync

| Action | Sync Before? | Sync After? | Why |
|--------|--------------|-------------|-----|
| Page Load | Yes (Irys) | No | Recover state |
| Before Bet | Yes (RPC) | Yes (Irys) | Verify current leaf |
| After Bet | No | Yes (Irys) | Backup new position |
| Before Exit | Yes (RPC) | Yes (Irys) | Verify position exists |
| Before Withdraw | Yes (RPC) | Yes (Irys) | Verify balance |

---

## Security Considerations

### 1. Position Integrity (positions_root)

- On-chain `positions_root` is authoritative
- User must provide positions that hash to on-chain root
- Irys backup is convenience, NOT security source
- User cannot inject fake positions (hash mismatch = Lit rejects)

### 2. Nonce Levels (Replay Protection)

```
state_nonce (global)     - Incremented on every MPC state update
    ↓
user_nonce (per-user)    - Incremented on every user leaf change
    ↓
bet_nonce (per-position) - Unique identifier for each position
```

### 3. Root Update Chain

```
Position changes
    ↓
positions_root = merkle_hash(all_positions)
    ↓
User leaf changes (new positions_root)
    ↓
leaf_hash = keccak256(user_leaf_data)
    ↓
replace_user_leaf() in SPL Compressed Tree
    ↓
Tree root automatically updates
    ↓
update_state() updates global state_nonce
```

### 4. Emergency Escape

- 24h timeout allows withdrawal if MPC offline
- User provides merkle proof of their leaf
- No MPC signature required after timeout

---

## Implementation Files

| File | Purpose |
|------|---------|
| `state_tree/state.rs` | GlobalState, UserLeafData, Position structs |
| `state_tree/compressed_tree.rs` | SPL compression integration |
| `state_tree/withdraw.rs` | Withdrawal with merkle proof |
| `state_tree/initialize.rs` | System setup |

---

## Next Steps

- [Merkle Trees Deep Dive](merkle-trees.md)
- [Lit Protocol Integration](lit-protocol.md)

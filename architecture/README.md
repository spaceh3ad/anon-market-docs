# System Overview

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        SOLANA (L1)                              │
├─────────────────────────────────────────────────────────────────┤
│  GlobalState PDA:                                               │
│  ├── state_nonce: u64             // Replay protection          │
│  ├── mpc_authority: Pubkey        // Lit Wrapped Key            │
│  ├── emergency_timeout: i64       // 24h default                │
│  ├── merkle_tree: Pubkey          // SPL Compression tree       │
│  └── total_deposited/withdrawn    // Vault accounting           │
│                                                                 │
│  Compressed Merkle Tree (SPL Account Compression):              │
│  ├── depth: 20 (supports ~1M users)                             │
│  ├── Each leaf = UserLeafData (88 bytes)                        │
│  └── Managed by Lit MPC authority                               │
│                                                                 │
│  DepositVault PDA: Holds all user funds                         │
│                                                                 │
│  MarketState (per market):                                      │
│  ├── total_shares_yes / total_shares_no                         │
│  ├── total_cost (pool)                                          │
│  └── status, winner                                             │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │ Lit signs & submits txs
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    LIT PROTOCOL (MPC)                           │
├─────────────────────────────────────────────────────────────────┤
│  Wrapped Key: Ed25519 Solana keypair                            │
│  ├── Signs full Solana transactions                             │
│  ├── Funded with SOL for gas                                    │
│  └── Pubkey stored as mpc_authority in GlobalState              │
│                                                                 │
│  Stateless Actions:                                             │
│  ├── process-deposit: Verify deposit tx, update user leaf       │
│  ├── buy-shares: Validate, update leaf + market pools           │
│  ├── sell-shares: Calculate proceeds, update leaf + pools       │
│  ├── claim-winnings: Verify winner, calculate payout            │
│  └── request-withdrawal: Sign withdrawal approval tx            │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      FRONTEND + STORAGE                         │
├─────────────────────────────────────────────────────────────────┤
│  localStorage (primary):                                        │
│  ├── positions: Position[]        // Active bets                │
│  ├── uuid_salt: string            // Secret for UUID derivation │
│  ├── leaf_index: number           // Position in merkle tree    │
│  └── last_sync: timestamp         // Last Irys sync             │
│                                                                 │
│  Irys/Arweave (backup):                                         │
│  ├── Encrypted with wallet signature                            │
│  ├── ~1-2KB per user                                            │
│  └── Recovery if localStorage lost                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Components

### 1. Solana Program

The on-chain smart contract handles:
- User deposits and withdrawals
- Market creation and resolution
- AMM swap execution
- Merkle tree management

### 2. Lit Protocol MPC

The MPC network provides:
- Distributed key management
- Stateless action execution
- Transaction signing
- State verification

### 3. Frontend Application

The Next.js frontend manages:
- Wallet connections
- Local state management
- Lit Action invocation
- UI rendering

### 4. Arweave Backup

Irys/Arweave provides:
- Encrypted state backups
- Recovery capability
- Permanent storage

---

## Data Flow

### Deposit Flow

```
1. User sends SOL to DepositVault
2. Frontend calls Lit Action "process-deposit"
3. Lit verifies deposit transaction on-chain
4. Lit computes new user leaf (balance increased)
5. Lit signs merkle tree update transaction
6. Frontend submits transaction to Solana
7. User's balance updated in compressed tree
```

### Betting Flow

```
1. User requests bet via frontend
2. Frontend calls Lit Action "buy-shares"
3. Lit fetches current merkle proof
4. Lit verifies user has sufficient balance
5. Lit computes AMM swap (shares received)
6. Lit updates user leaf + market pools
7. Lit signs combined transaction
8. Frontend submits to Solana
9. Market pools and user leaf updated atomically
```

### Withdrawal Flow

```
1. User requests withdrawal
2. Frontend calls Lit Action "request-withdrawal"
3. Lit verifies user's available balance
4. Lit signs withdrawal transaction
5. Frontend submits to Solana
6. SOL transferred from vault to user wallet
7. User leaf updated (balance decreased)
```

---

## Security Properties

### 1. Non-Custodial

Users always control their funds:
- Deposits go to on-chain vault
- Withdrawals are signed by MPC authority
- Emergency withdrawal after 24h timeout

### 2. Privacy-Preserving

Individual data is hidden:
- Balances in compressed merkle tree
- Positions in per-user merkle tree
- Events use unlinkable UUIDs

### 3. Verifiable

All state transitions are verifiable:
- Merkle proofs for leaf inclusion
- AMM invariants checked on-chain
- Nonce-based replay protection

---

## Next Steps

- [State Tree Architecture](state-tree.md)
- [Merkle Tree Deep Dive](merkle-trees.md)
- [Lit Protocol Integration](lit-protocol.md)

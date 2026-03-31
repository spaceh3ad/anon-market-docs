# Smart Contracts

## Overview

The AnonMarkets Solana program is built with Anchor framework. It handles:

- User deposits and withdrawals
- Market creation and resolution
- AMM swap execution
- Compressed merkle tree management

---

## Program Structure

```
programs/anon_market/src/
├── lib.rs                     # Program entrypoint, instruction routing
├── error.rs                   # Custom error definitions
├── fuzz_tests.rs              # Property-based tests
└── instructions/
    ├── market/
    │   ├── mod.rs
    │   ├── state.rs           # MarketState account
    │   ├── create_market.rs   # Create new market
    │   ├── resolve_market.rs  # Resolve market outcome
    │   └── dispute_market.rs  # Dispute resolution
    └── state_tree/
        ├── mod.rs
        ├── state.rs           # GlobalState, UserLeafData
        ├── initialize.rs      # System initialization
        ├── deposit.rs         # User deposits
        ├── withdraw.rs        # User withdrawals
        ├── update_state.rs    # MPC state updates
        ├── amm_swap.rs        # AMM buy/sell
        ├── claim_winnings.rs  # Claim payouts
        └── compressed_tree.rs # SPL compression helpers
```

---

## Key Accounts

### GlobalState

System-wide configuration and state:

```rust
#[account]
pub struct GlobalState {
    pub bump: u8,
    pub state_nonce: u64,           // Replay protection
    pub mpc_authority: Pubkey,      // Lit Wrapped Key
    pub merkle_tree: Pubkey,        // SPL compressed tree
    pub deposit_vault: Pubkey,      // SOL vault PDA
    pub total_deposited: u64,       // Cumulative deposits
    pub total_withdrawn: u64,       // Cumulative withdrawals
    pub emergency_timeout: i64,     // 24h default
    pub deposit_fee_bps: u16,       // Deposit fee (basis points)
    pub protocol_fee_bps: u16,      // Protocol fee on winnings
}
```

### MarketState

Per-market state:

```rust
#[account]
pub struct MarketState {
    pub bump: u8,
    pub creator: Pubkey,
    pub question: String,           // Market question
    pub resolution_criteria: String,
    pub end_timestamp: i64,
    pub total_shares_yes: u64,      // YES shares outstanding
    pub total_shares_no: u64,       // NO shares outstanding
    pub total_cost: u64,            // Pool value (lamports)
    pub status: MarketStatus,
    pub winner: Option<u8>,         // 0=YES, 1=NO
}

pub enum MarketStatus {
    Open,
    Locked,      // Future: for batching
    Resolved,
    Disputed,
}
```

### UserLeafData

User state stored in compressed tree:

```rust
pub struct UserLeafData {
    pub user_pubkey: [u8; 32],
    pub available_balance: u64,
    pub locked_balance: u64,
    pub nonce: u64,
    pub positions_root: [u8; 32],
}
```

---

## Key Instructions

### initialize

Sets up GlobalState and creates compressed merkle tree.

```rust
pub fn initialize(
    ctx: Context<Initialize>,
    mpc_authority: Pubkey,
    emergency_timeout: i64,
) -> Result<()>
```

### deposit

User deposits SOL to vault.

```rust
pub fn deposit(
    ctx: Context<Deposit>,
    amount: u64,
) -> Result<()>
```

### amm_swap

Execute AMM buy or sell (called by MPC).

```rust
pub fn amm_swap(
    ctx: Context<AmmSwap>,
    side: u8,           // 0=YES, 1=NO
    is_buy: bool,
    amount: u64,        // SOL amount (buy) or shares (sell)
    user_leaf_data: UserLeafData,
    proof: Vec<[u8; 32]>,
) -> Result<()>
```

### withdraw

User withdraws SOL from vault (signed by MPC).

```rust
pub fn withdraw(
    ctx: Context<Withdraw>,
    amount: u64,
    user_leaf_data: UserLeafData,
    proof: Vec<[u8; 32]>,
) -> Result<()>
```

### create_market

Create a new prediction market.

```rust
pub fn create_market(
    ctx: Context<CreateMarket>,
    question: String,
    resolution_criteria: String,
    end_timestamp: i64,
    initial_liquidity: u64,
) -> Result<()>
```

### resolve_market

Resolve market with winning outcome.

```rust
pub fn resolve_market(
    ctx: Context<ResolveMarket>,
    winner: u8,  // 0=YES, 1=NO
) -> Result<()>
```

---

## AMM Implementation

### Pricing Formula

```rust
// Constant Product Market Maker
pub fn calculate_price(yes_shares: u64, no_shares: u64) -> (f64, f64) {
    let total = yes_shares + no_shares;
    let price_yes = yes_shares as f64 / total as f64;
    let price_no = no_shares as f64 / total as f64;
    (price_yes, price_no)
}
```

### Buy Shares

```rust
pub fn calculate_buy(
    yes_shares: u64,
    no_shares: u64,
    side: u8,
    sol_amount: u64,
) -> (u64, u64, u64) {
    // Returns: (shares_received, new_yes, new_no)
    let invariant = yes_shares * no_shares;

    if side == 0 {
        // Buying YES
        let new_no = no_shares + sol_amount;
        let new_yes = invariant / new_no;
        let shares = yes_shares - new_yes;
        (shares, new_yes, new_no)
    } else {
        // Buying NO
        let new_yes = yes_shares + sol_amount;
        let new_no = invariant / new_yes;
        let shares = no_shares - new_no;
        (shares, new_yes, new_no)
    }
}
```

---

## Error Codes

```rust
#[error_code]
pub enum AnonMarketError {
    #[msg("Insufficient balance for this operation")]
    InsufficientBalance,

    #[msg("Invalid merkle proof")]
    InvalidMerkleProof,

    #[msg("Market is not open for trading")]
    MarketNotOpen,

    #[msg("Market has not ended yet")]
    MarketNotEnded,

    #[msg("Invalid nonce - possible replay attack")]
    InvalidNonce,

    #[msg("Unauthorized - not MPC authority")]
    Unauthorized,

    #[msg("Emergency timeout not reached")]
    EmergencyTimeoutNotReached,

    #[msg("Arithmetic overflow")]
    ArithmeticOverflow,
}
```

---

## PDAs

| PDA | Seeds | Purpose |
|-----|-------|---------|
| GlobalState | `["global_state"]` | System configuration |
| DepositVault | `["deposit_vault"]` | Holds all user SOL |
| MarketState | `["market", creator, question_hash]` | Per-market state |

---

## Events

```rust
#[event]
pub struct DepositEvent {
    pub user: Pubkey,
    pub amount: u64,
    pub timestamp: i64,
}

#[event]
pub struct WithdrawEvent {
    pub user: Pubkey,
    pub amount: u64,
    pub timestamp: i64,
}

#[event]
pub struct BetPlacedEvent {
    pub user_uuid: [u8; 32],  // Unlinkable UUID
    pub market: Pubkey,
    pub side: u8,
    pub shares: u64,
    pub cost: u64,
}

#[event]
pub struct MarketResolvedEvent {
    pub market: Pubkey,
    pub winner: u8,
    pub timestamp: i64,
}
```

---

## Security Considerations

### Access Control

- Most state-changing instructions require MPC authority signature
- Emergency withdrawal requires timeout period
- Market resolution requires creator or MPC authority

### Arithmetic Safety

All arithmetic uses checked operations:

```rust
let new_balance = balance
    .checked_add(amount)
    .ok_or(AnonMarketError::ArithmeticOverflow)?;
```

### Replay Protection

- Global `state_nonce` incremented on every state update
- Per-user `nonce` in leaf data
- Merkle proofs include current tree root

---

## Building

```bash
# Build program
anchor build

# Run tests
anchor test

# Deploy to localnet
anchor deploy

# Deploy to devnet
anchor deploy --provider.cluster devnet
```

---

## Next Steps

- [Testing Guide](testing.md)
- [Deployment](deployment.md)

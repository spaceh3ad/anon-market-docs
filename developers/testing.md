# Testing Guide

## Overview

AnonMarkets uses multiple testing strategies:

- **Integration tests** - End-to-end Anchor tests
- **Fuzz tests** - Property-based testing with proptest
- **Unit tests** - Frontend component tests
- **Rust checks** - Formatting and linting

---

## Integration Tests

### Run All Tests

```bash
# Starts local validator automatically
yarn test

# Or with existing validator
yarn test:local
```

### Run Specific Test

```bash
yarn ts-mocha -p ./tsconfig.json -t 1000000 tests/integration/amm.ts
```

### Test Structure

```typescript
describe("AMM", () => {
  it("should buy YES shares", async () => {
    // Setup
    const market = await createTestMarket();
    const user = await createTestUser();

    // Execute
    const tx = await program.methods
      .ammSwap(0, true, new BN(1_000_000_000))
      .accounts({ ... })
      .rpc();

    // Verify
    const marketState = await program.account.marketState.fetch(market);
    expect(marketState.totalSharesYes.toNumber()).to.be.greaterThan(1000);
  });
});
```

---

## Fuzz Tests

Fuzz tests use [proptest](https://github.com/proptest-rs/proptest) to find edge cases through randomized inputs.

### Run Fuzz Tests

```bash
# All fuzz tests
cargo test --features anchor-debug fuzz_ -- --nocapture

# Specific test
cargo test --features anchor-debug fuzz_price_sum_equals_one -- --nocapture

# More iterations (thorough)
PROPTEST_CASES=10000 cargo test --features anchor-debug fuzz_ -- --nocapture
```

### Available Fuzz Tests

| Test | Property Verified |
|------|-------------------|
| `fuzz_price_sum_equals_one` | YES + NO prices always = 1.0 |
| `fuzz_price_in_valid_range` | Prices always between 0 and 1 |
| `fuzz_buy_returns_positive_shares` | Buying always returns shares > 0 |
| `fuzz_buy_effective_price_bounded` | Effective price is reasonable |
| `fuzz_more_shares_more_return` | More shares = more sell return |
| `fuzz_cannot_overdraw` | Can't sell more shares than exist |
| `fuzz_vault_solvency` | Vault always has enough for withdrawals |
| `fuzz_leaf_hash_deterministic` | Same input = same hash |
| `fuzz_position_hash_deterministic` | Position hashing is deterministic |
| `fuzz_checked_arithmetic_no_panic` | No arithmetic overflow panics |

### Example Fuzz Test

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn fuzz_price_sum_equals_one(
        yes_shares in 1u64..1_000_000_000,
        no_shares in 1u64..1_000_000_000,
    ) {
        let (price_yes, price_no) = calculate_price(yes_shares, no_shares);

        // Property: prices must sum to 1.0
        let sum = price_yes + price_no;
        prop_assert!((sum - 1.0).abs() < 0.0001, "Sum was {}", sum);
    }
}
```

---

## Frontend Tests

### Run Tests

```bash
cd frontend

# All tests
yarn test

# Watch mode
yarn test:watch

# Specific file
yarn test src/test/mock-amm.test.ts
```

### Test Structure

```typescript
import { describe, it, expect } from "vitest";
import { calculateBuy, calculatePrice } from "@/lib/amm";

describe("AMM calculations", () => {
  it("should calculate correct price", () => {
    const { priceYes, priceNo } = calculatePrice(1000n, 1000n);
    expect(priceYes).toBe(0.5);
    expect(priceNo).toBe(0.5);
  });

  it("should return positive shares on buy", () => {
    const shares = calculateBuy(1000n, 1000n, 0, 100n);
    expect(shares).toBeGreaterThan(0n);
  });
});
```

---

## Rust Checks

### Format Check

```bash
cargo fmt --all -- --check
```

### Clippy Lints

```bash
cargo clippy --all-targets --all-features -- -D warnings
```

### Fix Formatting

```bash
cargo fmt --all
```

---

## Full CI Suite

Run everything locally before pushing:

```bash
# Lint
yarn lint

# Rust checks
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings

# Build
anchor build

# Tests
anchor test
cd frontend && yarn test
```

---

## Test Utilities

### Create Test User

```typescript
async function createTestUser(): Promise<TestUser> {
  const keypair = Keypair.generate();
  await airdrop(connection, keypair.publicKey, 10);
  return { keypair, publicKey: keypair.publicKey };
}
```

### Create Test Market

```typescript
async function createTestMarket(): Promise<PublicKey> {
  const [marketPda] = PublicKey.findProgramAddressSync(
    [Buffer.from("market"), creator.toBytes(), questionHash],
    program.programId
  );

  await program.methods
    .createMarket("Test question?", "Resolution criteria", endTimestamp, liquidity)
    .accounts({ market: marketPda, creator: wallet.publicKey })
    .rpc();

  return marketPda;
}
```

### Wait for Confirmation

```typescript
async function confirmTx(signature: string): Promise<void> {
  const latestBlockhash = await connection.getLatestBlockhash();
  await connection.confirmTransaction({
    signature,
    ...latestBlockhash,
  });
}
```

---

## Debugging Failed Tests

### View Transaction Logs

```bash
# Enable verbose logging
RUST_LOG=solana_runtime::system_instruction_processor=trace anchor test
```

### Inspect Account State

```typescript
const state = await program.account.marketState.fetch(marketPda);
console.log("Market state:", JSON.stringify(state, null, 2));
```

### Check Balances

```typescript
const balance = await connection.getBalance(publicKey);
console.log("Balance:", balance / LAMPORTS_PER_SOL, "SOL");
```

---

## Coverage

### Rust Coverage (requires nightly)

```bash
cargo +nightly tarpaulin --out Html
open tarpaulin-report.html
```

### Frontend Coverage

```bash
cd frontend
yarn test --coverage
```

---

## Next Steps

- [Deployment](deployment.md)
- [Smart Contracts](smart-contracts.md)

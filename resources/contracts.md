# Contract Addresses

## Devnet

| Component | Address |
|-----------|---------|
| **Program ID** | `45Jwc6unadj52ZEMd7HRcmkqQ4rpHaSxy628dDzKo4xR` |
| GlobalState PDA | Derived from `["global_state"]` |
| DepositVault PDA | Derived from `["deposit_vault"]` |
| Merkle Tree | Created during initialization |

### PDA Derivation

```typescript
import { PublicKey } from "@solana/web3.js";

const PROGRAM_ID = new PublicKey("45Jwc6unadj52ZEMd7HRcmkqQ4rpHaSxy628dDzKo4xR");

// GlobalState
const [globalState] = PublicKey.findProgramAddressSync(
  [Buffer.from("global_state")],
  PROGRAM_ID
);

// DepositVault
const [depositVault] = PublicKey.findProgramAddressSync(
  [Buffer.from("deposit_vault")],
  PROGRAM_ID
);

// Market PDA
const [marketPda] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("market"),
    creator.toBytes(),
    questionHash,
  ],
  PROGRAM_ID
);
```

---

## Mainnet

> **Note:** Mainnet deployment pending audit completion.

| Component | Address |
|-----------|---------|
| Program ID | TBD |
| GlobalState PDA | TBD |
| DepositVault PDA | TBD |

---

## External Dependencies

### Solana

| Component | Mainnet | Devnet |
|-----------|---------|--------|
| SPL Token Program | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` | Same |
| Account Compression | `cmtDvXumGCrqC1Age74AVPhSRVXJMd8PJS91L8KbNCK` | Same |
| Noop Program | `noopb9bkMVfRPU8AsbpTUg8AQkHtKwMYZiFUjNRtMmV` | Same |

### Lit Protocol

| Component | Value |
|-----------|-------|
| Network | `datil-dev` (devnet) / `datil` (mainnet) |
| Wrapped Key Type | Ed25519 |
| Chronicle Yellowstone | `https://yellowstone.litprotocol.com` |

---

## Verifying Contracts

### Check Program Deployment

```bash
# Devnet
solana program show 45Jwc6unadj52ZEMd7HRcmkqQ4rpHaSxy628dDzKo4xR --url devnet

# Mainnet
solana program show <PROGRAM_ID> --url mainnet-beta
```

### View Account Data

```bash
# GlobalState
solana account <GLOBAL_STATE_PDA> --url devnet --output json

# Market
solana account <MARKET_PDA> --url devnet --output json
```

### IDL

The program IDL is available at:
- `target/idl/anon_market.json` (local build)
- [GitHub](https://github.com/anon-markets/anon-market/blob/main/target/idl/anon_market.json)

---

## RPC Endpoints

### Devnet

| Provider | URL |
|----------|-----|
| Solana | `https://api.devnet.solana.com` |
| Helius | `https://devnet.helius-rpc.com/?api-key=<KEY>` |
| Alchemy | `https://solana-devnet.g.alchemy.com/v2/<KEY>` |

### Mainnet

| Provider | URL |
|----------|-----|
| Solana | `https://api.mainnet-beta.solana.com` |
| Helius | `https://mainnet.helius-rpc.com/?api-key=<KEY>` |
| Alchemy | `https://solana-mainnet.g.alchemy.com/v2/<KEY>` |
| Triton | `https://anonmarkets-mainnet.rpcpool.com` |

---

## Block Explorers

### View Program

- [Solscan (Devnet)](https://solscan.io/account/45Jwc6unadj52ZEMd7HRcmkqQ4rpHaSxy628dDzKo4xR?cluster=devnet)
- [Solana Explorer (Devnet)](https://explorer.solana.com/address/45Jwc6unadj52ZEMd7HRcmkqQ4rpHaSxy628dDzKo4xR?cluster=devnet)

### View Transactions

- Solscan: `https://solscan.io/tx/<SIGNATURE>?cluster=devnet`
- Explorer: `https://explorer.solana.com/tx/<SIGNATURE>?cluster=devnet`

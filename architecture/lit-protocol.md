# Lit Protocol MPC

## Overview

[Lit Protocol](https://litprotocol.com) provides the MPC (Multi-Party Computation) layer that manages private state transitions. It acts as a trusted intermediary that can verify user requests and sign Solana transactions.

---

## Why Lit Protocol?

### The Problem

Traditional on-chain state is fully public. Anyone can read:
- User balances
- Bet positions
- Transaction history

### The Solution

Lit Protocol enables:
- **Private computation**: Verify balances without revealing them
- **Distributed signing**: No single party controls the signing key
- **Stateless actions**: Code runs on-demand, stores nothing

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIT PROTOCOL NETWORK                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Node 1     │  │   Node 2     │  │   Node 3     │   ...    │
│  │  (Key Share) │  │  (Key Share) │  │  (Key Share) │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         │                 │                 │                   │
│         └─────────────────┼─────────────────┘                   │
│                           │                                     │
│                    ┌──────▼──────┐                              │
│                    │   Wrapped   │                              │
│                    │     Key     │                              │
│                    │  (Ed25519)  │                              │
│                    └─────────────┘                              │
│                           │                                     │
│                    Signs Solana transactions                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Wrapped Keys

A **Wrapped Key** is a cryptographic key managed by Lit's MPC network:

- **Type**: Ed25519 (Solana-compatible)
- **Storage**: Distributed across Lit nodes (no single party has full key)
- **Signing**: Requires threshold of nodes to produce signature
- **Identity**: Public key stored in GlobalState as `mpc_authority`

### Properties

| Property | Value |
|----------|-------|
| Key type | Ed25519 |
| Threshold | 2/3 of network |
| Signing speed | ~1-2 seconds |
| Key rotation | Supported |

---

## Lit Actions

Lit Actions are JavaScript functions executed by Lit nodes:

```javascript
const go = async () => {
  // 1. Fetch current state from Solana RPC
  const userLeaf = await fetchUserLeaf(params.userPubkey);

  // 2. Verify user's request
  const isValid = await verifySignature(params.signature, params.message);
  if (!isValid) throw new Error("Invalid signature");

  // 3. Verify merkle proof
  const proofValid = verifyMerkleProof(userLeaf, params.proof);
  if (!proofValid) throw new Error("Invalid merkle proof");

  // 4. Compute new state
  const newLeaf = computeNewLeaf(userLeaf, params.action);

  // 5. Build Solana transaction
  const tx = buildTransaction(newLeaf, params.market);

  // 6. Sign with Wrapped Key
  const signature = await Lit.Actions.signAndCombineEcdsa({
    toSign: tx.serializeMessage(),
    publicKey: wrappedKeyPubkey,
    sigName: "solana_tx",
  });

  // 7. Return signed transaction
  return { signedTx: attachSignature(tx, signature), newLeaf };
};

go();
```

---

## Available Actions

| Action | Purpose | Inputs | Outputs |
|--------|---------|--------|---------|
| `process-deposit` | Confirm deposit, update leaf | tx_sig, pubkey | signed_update_tx |
| `buy-shares` | Execute buy order | market, side, amount | signed_swap_tx |
| `sell-shares` | Execute sell order | market, side, shares | signed_swap_tx |
| `claim-winnings` | Claim resolved market | market, position | signed_claim_tx |
| `request-withdrawal` | Approve withdrawal | amount | signed_withdraw_tx |

---

## Security Model

### What Lit Nodes See

- User's public key
- Requested action and parameters
- Merkle proofs

### What Lit Nodes DON'T See

- User's private key
- Other users' data
- Full transaction history

### Trust Assumptions

1. **Threshold security**: 2/3 of Lit nodes must be honest
2. **Availability**: Lit network must be online (emergency exit if not)
3. **Correctness**: Action code is open source and auditable

---

## Action Flow

```
┌─────────┐     ┌─────────────┐     ┌──────────────┐     ┌─────────┐
│ Frontend│────►│ Lit Gateway │────►│ Lit Network  │────►│ Solana  │
│         │     │             │     │ (Execute)    │     │         │
└─────────┘     └─────────────┘     └──────────────┘     └─────────┘
     │                                     │                   │
     │  1. Build request                   │                   │
     │  2. Sign with wallet                │                   │
     │  3. Send to Lit ───────────────────►│                   │
     │                                     │                   │
     │                          4. Verify signature            │
     │                          5. Fetch RPC data ────────────►│
     │                          6. Verify proofs  ◄────────────│
     │                          7. Compute new state           │
     │                          8. Sign transaction            │
     │                                     │                   │
     │  9. Receive signed tx ◄─────────────│                   │
     │ 10. Submit to Solana ──────────────────────────────────►│
     │ 11. Confirmation ◄──────────────────────────────────────│
```

---

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `INVALID_SIGNATURE` | Wallet signature doesn't match | Re-sign request |
| `INSUFFICIENT_BALANCE` | Not enough SOL for action | Deposit more |
| `INVALID_MERKLE_PROOF` | Stale proof | Refresh state |
| `NONCE_MISMATCH` | Replay attack or stale state | Refresh state |
| `NETWORK_UNAVAILABLE` | Lit network down | Wait or emergency exit |

---

## Registering Actions

Actions are registered with Lit and assigned IPFS CIDs:

```bash
# Register all actions
cd frontend
npm run lit:register

# Output:
# LIT_ACTION_BUY_SHARES_CID=QmXxx...
# LIT_ACTION_SELL_SHARES_CID=QmYyy...
# ...
```

---

## Configuration

Required environment variables:

```bash
# Lit Protocol
LIT_API_KEY=<your-api-key>
LIT_ACCOUNT_API_KEY=<your-account-api-key>
LIT_GROUP_ID=<your-group-id>

# Wrapped Key
NEXT_PUBLIC_LIT_WRAPPED_KEY_PUBKEY=<solana-pubkey>
LIT_WRAPPED_KEY_ID=<wrapped-key-id>
LIT_PKP_ADDRESS=<pkp-eth-address>

# Action CIDs
LIT_ACTION_BUY_SHARES_CID=Qm...
LIT_ACTION_SELL_SHARES_CID=Qm...
# ...
```

---

## Mock Mode

For local development without Lit:

```bash
USE_MOCK_LIT=true
```

Mock mode:
- Uses local keypair instead of MPC
- Same API interface
- Faster execution
- No network dependencies

---

## Next Steps

- [Developer Setup](../developers/local-setup.md)
- [Smart Contracts](../developers/smart-contracts.md)

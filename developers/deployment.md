# Deployment

## Overview

AnonMarkets can be deployed to:
- **Localnet** - Local development
- **Devnet** - Testing with devnet SOL
- **Mainnet** - Production (requires real SOL)

---

## Localnet Deployment

### Full Setup

```bash
# Starts validator, deploys program, creates test market
yarn local

# With auto-validator flag
yarn local -a
```

### Step by Step

```bash
# 1. Start validator
solana-test-validator

# 2. Deploy program
anchor deploy

# 3. Initialize state
npx tsx migrations/admin/setup.ts --network local

# 4. Create test market
node scripts/create-test-market.cjs
```

---

## Devnet Deployment

### Prerequisites

```bash
# Configure for devnet
solana config set --url devnet

# Get devnet SOL
solana airdrop 2
```

### Deploy

```bash
# Full deployment
yarn devnet

# Or step by step:
anchor build
anchor deploy --provider.cluster devnet

# Copy IDL to frontend
yarn idl:copy

# Initialize state
npx tsx migrations/admin/setup.ts --network devnet
```

### Verify Deployment

```bash
# Check program exists
solana program show <PROGRAM_ID>

# Check GlobalState
solana account <GLOBAL_STATE_PDA>
```

---

## Mainnet Deployment

### Prerequisites

- [ ] Audited smart contracts
- [ ] Sufficient SOL for deployment (~3-5 SOL)
- [ ] Lit Protocol mainnet keys
- [ ] Production environment variables

### Security Checklist

Before mainnet deployment:

1. **Code Review**
   - [ ] All code reviewed by team
   - [ ] No console.log or debug statements
   - [ ] Error handling complete

2. **Testing**
   - [ ] All tests passing
   - [ ] Fuzz tests with high iteration count
   - [ ] Manual testing on devnet

3. **Security**
   - [ ] Access controls verified
   - [ ] No admin backdoors
   - [ ] Emergency withdrawal tested

4. **Infrastructure**
   - [ ] RPC endpoints configured
   - [ ] Monitoring in place
   - [ ] Backup procedures documented

### Deploy

```bash
# Configure for mainnet
solana config set --url mainnet-beta

# Deploy (requires SOL)
anchor deploy --provider.cluster mainnet

# Initialize with production MPC authority
npx tsx migrations/admin/setup.ts --network mainnet
```

---

## Configuration

### Environment Variables

**Local/Devnet:**
```bash
NEXT_PUBLIC_RPC_URL=http://127.0.0.1:8899
NEXT_PUBLIC_PROGRAM_ID=<program-id>
USE_MOCK_LIT=true
```

**Production:**
```bash
NEXT_PUBLIC_RPC_URL=https://api.mainnet-beta.solana.com
NEXT_PUBLIC_PROGRAM_ID=<program-id>
USE_MOCK_LIT=false
LIT_API_KEY=<api-key>
LIT_WRAPPED_KEY_ID=<wrapped-key-id>
# ... other Lit vars
```

### Update Frontend Config

After deployment, update `frontend/.env.local`:

```bash
NEXT_PUBLIC_PROGRAM_ID=<new-program-id>
```

---

## Program Upgrades

### Prepare Upgrade

```bash
# Build new version
anchor build

# Write buffer
solana program write-buffer target/deploy/anon_market.so
# Note the buffer address
```

### Deploy Upgrade

```bash
# Deploy from buffer
solana program deploy --buffer <BUFFER_ADDRESS> --program-id <PROGRAM_ID>
```

### Upgrade Authority

The upgrade authority can:
- Deploy new program versions
- Close the program
- Transfer authority

```bash
# Check current authority
solana program show <PROGRAM_ID>

# Transfer authority (for multisig)
solana program set-upgrade-authority <PROGRAM_ID> --new-upgrade-authority <NEW_AUTHORITY>
```

---

## Lit Protocol Setup

### 1. Get API Keys

1. Visit [Lit Protocol Developer Portal](https://developer.litprotocol.com)
2. Create account and get API keys
3. Note `LIT_API_KEY` and `LIT_ACCOUNT_API_KEY`

### 2. Generate Wrapped Key

```bash
# Use Lit SDK to generate Ed25519 wrapped key
# Save the wrapped key ID and Solana pubkey
```

### 3. Register Lit Actions

```bash
cd frontend
npm run lit:register

# Note the CIDs output
```

### 4. Configure Environment

```bash
LIT_ACTION_BUY_SHARES_CID=Qm...
LIT_ACTION_SELL_SHARES_CID=Qm...
LIT_ACTION_PROCESS_DEPOSIT_CID=Qm...
# ...
```

---

## Monitoring

### Program Logs

```bash
solana logs <PROGRAM_ID>
```

### Account Monitoring

```bash
# Watch GlobalState
watch -n 5 'solana account <GLOBAL_STATE_PDA> --output json'
```

### Frontend Monitoring

- Set up error tracking (Sentry, etc.)
- Monitor RPC rate limits
- Track transaction success rates

---

## Rollback

If issues arise after deployment:

### Revert Program

```bash
# Deploy previous version from buffer
solana program deploy --buffer <OLD_BUFFER_ADDRESS> --program-id <PROGRAM_ID>
```

### Emergency Pause

The program doesn't have a built-in pause. For emergencies:
1. Users can emergency withdraw after 24h
2. Deploy patched version ASAP

---

## Addresses

### Devnet

| Component | Address |
|-----------|---------|
| Program ID | `45Jwc6unadj52ZEMd7HRcmkqQ4rpHaSxy628dDzKo4xR` |
| GlobalState | (derived PDA) |
| DepositVault | (derived PDA) |

### Mainnet

| Component | Address |
|-----------|---------|
| Program ID | TBD |
| GlobalState | TBD |
| DepositVault | TBD |

---

## Next Steps

- [Contract Addresses](../resources/contracts.md)
- [Smart Contracts](smart-contracts.md)

# Local Setup

## Prerequisites

### Required Software

| Software | Version | Installation |
|----------|---------|--------------|
| Node.js | 20+ | [nodejs.org](https://nodejs.org) |
| Rust | stable | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Solana CLI | 2.0.21 | See below |
| Anchor CLI | 0.32.1 | See below |
| Yarn | 4.9.1 | Via corepack |

### Install Solana CLI

```bash
# Download and install
SOLANA_VERSION="v2.0.21"
sh -c "$(curl -sSfL https://release.solana.com/${SOLANA_VERSION}/install)"

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"

# Verify
solana --version

# Generate keypair (if you don't have one)
solana-keygen new
```

### Install Anchor CLI

```bash
# Install AVM (Anchor Version Manager)
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force

# Install and use Anchor 0.32.1
avm install 0.32.1
avm use 0.32.1

# Verify
anchor --version
```

### Enable Yarn 4

```bash
corepack enable
```

---

## Quick Start

```bash
# Clone repository
git clone https://github.com/anon-markets/anon-market
cd anon-market

# Install dependencies
yarn install

# Build the Anchor program
anchor build

# Start local development
yarn dev
```

---

## Environment Setup

### Local Development

```bash
# Copy local env template
cp frontend/.env.local.example frontend/.env.local
```

### Devnet

```bash
cp frontend/.env.devnet frontend/.env.local
```

### Required Variables

```bash
# Solana Configuration
NEXT_PUBLIC_RPC_URL=http://127.0.0.1:8899
NEXT_PUBLIC_PROGRAM_ID=<program-id>

# Mock Mode (no Lit Protocol)
USE_MOCK_LIT=true
```

For production with Lit Protocol, see [Lit Protocol Setup](../architecture/lit-protocol.md).

---

## Running Locally

### Option 1: Full Setup

```bash
# Starts validator, deploys program, creates test market
yarn local
```

### Option 2: Step by Step

```bash
# Terminal 1: Start validator
solana-test-validator

# Terminal 2: Deploy program
anchor deploy

# Terminal 3: Start frontend
cd frontend && yarn dev
```

---

## Project Structure

```
anon-market/
├── programs/
│   └── anon_market/
│       └── src/
│           ├── lib.rs                 # Program entrypoint
│           ├── error.rs               # Custom errors
│           └── instructions/
│               ├── market/            # Market operations
│               └── state_tree/        # MPC state operations
├── lit-actions/
│   └── src/                           # Lit Action JavaScript
├── frontend/
│   └── src/
│       ├── app/                       # Next.js app router
│       ├── components/                # React components
│       ├── hooks/                     # Custom hooks
│       └── lib/                       # Utilities
├── tests/
│   └── integration/                   # Integration tests
├── scripts/                           # Dev utilities
├── migrations/                        # Admin scripts
└── docs/                              # Technical docs
```

---

## Development Workflow

### Making Smart Contract Changes

```bash
# Edit programs/anon_market/src/*.rs
anchor build
anchor test
```

### Making Frontend Changes

```bash
cd frontend
yarn dev    # Hot reload
yarn test   # Run tests
```

### Making Lit Action Changes

```bash
# Edit lit-actions/src/*.js
# Re-register to get new CIDs
cd frontend && npm run lit:register
```

---

## Debugging

### View Solana Logs

```bash
solana logs
```

### View Program Logs During Test

```bash
RUST_LOG=solana_runtime::system_instruction_processor=trace anchor test
```

### Frontend Console

Check browser DevTools for React and API errors.

---

## Common Issues

### "Insufficient funds"

```bash
solana airdrop 2  # Localnet
solana airdrop 2 --url devnet  # Devnet
```

### "Account not found"

```bash
# Ensure program is deployed
solana program show <PROGRAM_ID>

# Re-deploy if needed
anchor deploy
```

### Yarn version mismatch

```bash
corepack enable
yarn --version  # Should be 4.9.1
```

---

## Next Steps

- [Smart Contracts](smart-contracts.md)
- [Testing Guide](testing.md)
- [Deployment](deployment.md)

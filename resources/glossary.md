# Glossary

## A

### AMM (Automated Market Maker)
A smart contract that provides liquidity for trading without requiring traditional order books. AnonMarkets uses a bonding curve AMM.

### Anchor
A framework for Solana smart contract development that provides safety and developer experience improvements over raw Solana programs.

### Arweave
A permanent, decentralized storage network. AnonMarkets uses Arweave (via Irys) for encrypted user state backups.

## B

### Bonding Curve
A mathematical formula that determines the price of an asset based on its supply. In AnonMarkets, the bonding curve prices YES and NO shares.

### BPS (Basis Points)
A unit equal to 1/100th of a percent. 100 bps = 1%. Used for expressing fees.

## C

### Compressed Merkle Tree
A Solana data structure (SPL Account Compression) that stores large amounts of data off-chain while keeping a verifiable root on-chain. Enables ~1M users for ~0.5 SOL.

### CPMM (Constant Product Market Maker)
An AMM type where `x * y = k` (the product of two token reserves stays constant). Used in Uniswap and AnonMarkets.

## D

### Devnet
Solana's test network with free SOL for development.

## E

### Ed25519
An elliptic curve digital signature algorithm. Solana uses Ed25519 for keypairs. Lit Protocol's Wrapped Keys for Solana are Ed25519.

### Emergency Withdrawal
A safety mechanism allowing users to withdraw funds directly if the MPC network is unresponsive for 24+ hours.

## G

### GlobalState
The main on-chain account storing system-wide configuration: MPC authority, merkle tree address, vault address, and totals.

## I

### Irys
A Layer 2 for Arweave that provides fast, cheap uploads with tags for querying. AnonMarkets uses Irys for user state backups.

### Invariant
A property that must always remain true. In CPMM, the invariant is `yes_shares * no_shares = constant`.

## L

### Lamports
The smallest unit of SOL. 1 SOL = 1,000,000,000 lamports (10^9).

### Leaf
A single entry in a merkle tree. In AnonMarkets, each leaf represents one user's complete state.

### Lit Action
JavaScript code executed by Lit Protocol nodes. AnonMarkets uses Lit Actions for state verification and transaction signing.

### Lit Protocol
A decentralized key management and compute network. Provides MPC-based signing for AnonMarkets.

## M

### Mainnet
Solana's production network with real SOL.

### MarketState
The on-chain account storing per-market data: question, resolution criteria, share totals, and status.

### Merkle Proof
A cryptographic proof that a specific leaf exists in a merkle tree. Users provide proofs to verify their state.

### Merkle Root
The top-level hash of a merkle tree. Changes if any leaf changes.

### MPC (Multi-Party Computation)
A cryptographic technique where multiple parties jointly compute a function without revealing their individual inputs. Lit Protocol uses MPC for distributed key management.

## N

### Nonce
A number used once. AnonMarkets uses nonces at multiple levels (global, user, position) for replay protection.

## P

### PDA (Program Derived Address)
A Solana address derived from seeds and a program ID. PDAs can sign transactions on behalf of programs.

### PKP (Programmable Key Pair)
A Lit Protocol key pair that can be programmed with custom signing logic.

### Position
A user's bet in a specific market. Contains: market address, side (YES/NO), shares, cost basis, and nonce.

### Positions Root
A merkle root of all user positions, stored in the user's leaf.

### Prediction Market
A market where participants trade shares based on their beliefs about future events.

## R

### Replay Attack
An attack where a valid transaction is maliciously repeated. Prevented by incrementing nonces.

### Resolution
The process of determining a market's outcome (YES or NO wins).

## S

### Share
A unit of ownership in a market outcome. Winning shares pay 1 SOL each.

### Slippage
The difference between expected and actual price when trading. Larger trades have more slippage.

### SOL
Solana's native cryptocurrency.

### SPL (Solana Program Library)
Official Solana programs for common functionality (tokens, compression, etc.).

### Spot Price
The current market price for an infinitely small trade. The actual price paid for larger trades differs due to slippage.

## U

### UserLeafData
The structure stored in each user's merkle leaf: pubkey, available balance, locked balance, nonce, and positions root.

### UUID (Universally Unique Identifier)
In AnonMarkets, a hash of `pubkey + secret_salt` used to identify events without revealing the wallet address.

## V

### Vault
The on-chain account holding all deposited SOL. A PDA controlled by the program.

## W

### Wrapped Key
A cryptographic key managed by Lit Protocol's MPC network. No single party has the full key.

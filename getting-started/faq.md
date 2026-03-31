# Frequently Asked Questions

## General

### What is AnonMarkets?
AnonMarkets is a privacy-preserving prediction market built on Solana. You can bet on real-world outcomes while keeping your positions private.

### How is it different from other prediction markets?
Unlike traditional prediction markets where all bets are visible on-chain, AnonMarkets hides individual positions using merkle trees and MPC (Multi-Party Computation). Market odds are public, but who bet what remains private.

### Is it decentralized?
Yes. The smart contracts run on Solana, and the MPC network (Lit Protocol) is decentralized. No single entity can see your positions or censor your trades.

---

## Fees

### Are there any fees?
**Zero platform fees.** You pay only Solana network transaction fees (fractions of a cent).

### How does the protocol make money?
The protocol stakes vault deposits in liquid staking tokens (like jitoSOL), earning yield without charging users.

---

## Privacy

### What information is public?
- Market prices (odds)
- Total volume per market
- Your deposit and withdrawal transactions (amounts visible)

### What information is private?
- Your individual bet positions
- Your current balance
- Which markets you've bet on

### Can anyone see my bets?
No. Your positions are stored in an encrypted merkle tree. Only you (with your wallet) can decrypt and view your positions.

### What about deposits and withdrawals?
Deposit and withdrawal transactions are visible on-chain, including amounts. However, there's no way to link these to your betting activity.

---

## Trading

### How do prices work?
Prices are determined by a bonding curve AMM (Automated Market Maker). When more people bet YES, the YES price goes up. When more bet NO, the NO price goes up.

### Can I exit a bet early?
Yes! You can sell your position at any time before the market resolves. You'll receive the current market price for your shares.

### What happens if I lose?
Your bet amount stays in the pool and is distributed to winners. Losses are automatically handled - no action needed from you.

### What's the maximum I can win?
Your maximum payout depends on your position size and how many others bet on the same outcome. Fewer people on the winning side = bigger payout per share.

---

## Markets

### How are markets created?
Currently, markets are created by the protocol team. Permissionless market creation is coming soon.

### How are markets resolved?
Markets are resolved using AI-assisted verification with a challenge period. If the AI's decision is disputed, the market creator can override, or (in the future) a stake-weighted community vote decides.

### What if a market is resolved incorrectly?
There's a 24-hour challenge period after resolution is proposed. Anyone can stake SOL to challenge the outcome with counter-evidence.

---

## Technical

### Which wallets are supported?
Any Solana-compatible wallet including:
- Phantom
- Solflare
- Backpack
- Ledger (via Phantom/Solflare)

### What happens if I lose access to my browser?
Your positions are backed up to Arweave (via Irys) encrypted with your wallet signature. As long as you have your wallet, you can recover your state on any device.

### Is there an emergency withdrawal option?
Yes. If the MPC network is unresponsive for 24+ hours, you can withdraw directly using a merkle proof. This ensures you're never locked out of your funds.

### Where can I see the code?
AnonMarkets is fully open source: [GitHub Repository](https://github.com/anon-markets/anon-market)

---

## Troubleshooting

### My transaction failed
- Check you have enough SOL for gas fees
- Ensure your wallet is connected to the correct network
- Try refreshing the page and reconnecting your wallet

### I can't see my positions
- Positions are stored in your browser's localStorage
- If you cleared browser data, click "Recover" to restore from Arweave backup
- Make sure you're using the same wallet

### Market shows wrong odds
- Odds update in real-time based on trades
- Try refreshing the page
- Large trades can move prices significantly

---

## Still Have Questions?

- Join our [Discord](https://discord.gg/anonmarkets)
- Check the [technical documentation](../architecture/README.md)
- Read the [smart contract code](https://github.com/anon-markets/anon-market/tree/main/programs)

# Deposit Flow Documentation

## 📥 Player Deposit Process

### Step 1: Wallet Connection
```typescript
// Player connects Solana wallet
const wallet = await connectWallet('phantom'); // or 'solflare', 'backpack'
const publicKey = wallet.publicKey.toString();
```

### Step 2: Stake Selection
- **Minimum Stake:** 0.1 SOL
- **Maximum Stake:** 10 SOL
- **Supported Amounts:** 0.1, 0.5, 1.0, 2.5, 5.0, 10.0 SOL

### Step 3: Fund Transfer
```typescript
// SOL transfer to main escrow wallet
const transaction = new Transaction().add(
  SystemProgram.transfer({
    fromPubkey: playerPublicKey,
    toPubkey: ESCROW_WALLET,
    lamports: stakeAmount * LAMPORTS_PER_SOL,
  })
);

const signature = await wallet.signTransaction(transaction);
await connection.sendRawTransaction(signature.serialize());
```

### Step 4: Backend Confirmation
```typescript
// Backend verifies on-chain deposit
const deposit = await verifyDeposit(signature, stakeAmount);
if (deposit.confirmed) {
  await createEscrowLock(playerId, stakeAmount);
}
```

## 🔐 Security Measures

### Transaction Validation
- **Signature Verification:** Cryptographic proof of ownership
- **Amount Verification:** Exact stake amount confirmation
- **Double-Spend Prevention:** Transaction uniqueness check
- **Network Confirmation:** Required blockchain confirmations

### Escrow Lock Creation
```typescript
interface EscrowLock {
  playerId: string;
  amount: number;
  depositTx: string;
  lockTime: number;
  status: 'locked' | 'released' | 'refunded';
  matchId?: string;
}
```

## 📊 Deposit Statistics

### Real-time Metrics
- **Total Deposits:** Sum of all player deposits
- **Active Escrow:** Currently locked funds
- **Daily Volume:** 24-hour deposit amount
- **Success Rate:** Percentage of successful deposits

### Monitoring Alerts
- **Failed Deposits:** Automatic retry mechanism
- **Large Deposits:** Manual review for amounts > 5 SOL
- **Unusual Patterns:** Bot detection and prevention

## 🔄 Refund Policy

### Automatic Refunds
- **Match Timeout:** If no opponent found within 60s
- **Server Error:** Backend failure during match
- **Player Disconnect:** Network issues during gameplay
- **Game Glitch:** Technical problems affecting fairness

### Refund Process
```typescript
const refund = await processRefund(playerId, depositTx);
// SOL returned to original wallet
// Transaction hash provided for verification
```

## 📈 Deposit Flow Diagram

```
Player Wallet → Solana Network → Escrow Wallet → Backend Lock → Match Creation
     ↓              ↓                ↓              ↓              ↓
  SOL Transfer   On-Chain Tx    Fund Receipt   Escrow Lock    Game Start
```

## 🎯 Best Practices

### For Players
- **Network Fees:** Account for Solana gas fees
- **Confirmation Time:** Wait for blockchain confirmation
- **Backup Wallets:** Keep recovery phrases secure
- **Transaction History:** Save important transaction hashes

### For System
- **Rate Limiting:** Prevent spam deposits
- **Amount Limits:** Cap maximum stake amounts
- **Geographic Compliance:** Follow local regulations
- **AML/KYC:** Implement as required by jurisdiction

---

**⚠️ Important:** All deposits are final once matched. Refunds only available for technical failures.
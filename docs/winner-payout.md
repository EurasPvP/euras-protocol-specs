# Winner Payout Documentation

## 🏆 Payout System Overview

Euras ensures **instant, transparent winner payouts** through automated Solana blockchain transfers with verifiable transaction history.

## 💰 Payout Calculation

### Formula Application
```typescript
function calculateWinnerPayout(totalPot: number, rakeRate: number = 0.035): number {
  const rake = totalPot * rakeRate;
  const winnerPayout = totalPot - rake;
  return winnerPayout;
}
```

### Payout Examples
| Player A Stake | Player B Stake | Total Pot | Rake (3.5%) | Winner Receives |
|---------------|---------------|-----------|------------|-----------------|
| 0.5 SOL       | 0.5 SOL       | 1.0 SOL   | 0.035 SOL  | 0.965 SOL       |
| 1.0 SOL       | 1.0 SOL       | 2.0 SOL   | 0.07 SOL   | 1.93 SOL        |
| 2.5 SOL       | 2.5 SOL       | 5.0 SOL   | 0.175 SOL  | 4.825 SOL       |
| 5.0 SOL       | 5.0 SOL       | 10.0 SOL  | 0.35 SOL   | 9.65 SOL        |

## 🔄 Payout Process Flow

### Step 1: Match Completion Validation
```typescript
async function validateMatchCompletion(matchId: string, winnerId: string): Promise<boolean> {
  const match = await getMatch(matchId);
  
  // Verify match state
  if (match.status !== 'completed') {
    throw new Error('Match not properly completed');
  }
  
  // Verify winner legitimacy
  if (!isValidWinner(match, winnerId)) {
    throw new Error('Invalid winner determination');
  }
  
  // Verify no cheating detected
  if (await hasCheatingAttempt(matchId)) {
    throw new Error('Cheating detected - payout blocked');
  }
  
  return true;
}
```

### Step 2: Escrow Release
```typescript
async function releaseEscrowFunds(matchId: string, winnerId: string): Promise<PayoutResult> {
  // Get all escrow locks for this match
  const locks = await getMatchEscrowLocks(matchId);
  const totalPot = locks.reduce((sum, lock) => sum + lock.amount, 0);
  
  // Calculate payout
  const winnerPayout = calculateWinnerPayout(totalPot);
  
  // Update lock statuses
  for (const lock of locks) {
    lock.status = lock.playerId === winnerId ? 'released' : 'released';
    lock.releaseTime = Date.now();
    await updateEscrowLock(lock);
  }
  
  return {
    winnerId,
    amount: winnerPayout,
    totalPot,
    rake: totalPot - winnerPayout,
    matchId
  };
}
```

### Step 3: Blockchain Transaction
```typescript
async function processPayoutTransaction(payoutResult: PayoutResult): Promise<string> {
  const winnerWallet = await getPlayerWallet(payoutResult.winnerId);
  
  // Create Solana transaction
  const transaction = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: ESCROW_WALLET_PUBKEY,
      toPubkey: new PublicKey(winnerWallet),
      lamports: payoutResult.amount * LAMPORTS_PER_SOL,
    })
  );
  
  // Sign and send transaction
  const signature = await sendTransaction(transaction);
  
  // Wait for confirmation
  const confirmation = await connection.confirmTransaction(signature);
  
  if (confirmation.value.err) {
    throw new Error(`Payout transaction failed: ${confirmation.value.err}`);
  }
  
  return signature;
}
```

## 📊 Payout States & Tracking

### Transaction Status
```typescript
enum PayoutStatus {
  PENDING = 'pending',        // Calculated, awaiting blockchain
  PROCESSING = 'processing',  // Transaction submitted
  CONFIRMED = 'confirmed',    // Blockchain confirmed
  FAILED = 'failed',         // Transaction failed
  RETRYING = 'retrying'       // Automatic retry in progress
}
```

### Payout Record
```typescript
interface PayoutRecord {
  id: string;
  matchId: string;
  winnerId: string;
  amount: number;
  rake: number;
  status: PayoutStatus;
  transactionSignature?: string;
  createdAt: number;
  confirmedAt?: number;
  retryCount: number;
  errorMessage?: string;
}
```

## ⚡ Instant Payout Features

### Real-Time Processing
- **Sub-Second Execution:** Payout initiated immediately after match
- **Blockchain Confirmation:** ~400ms average confirmation time
- **Zero Gas Fees:** Players pay no transaction fees
- **Automatic Retry:** Failed transactions retried automatically

### Notification System
```typescript
async function notifyPayoutSuccess(payoutRecord: PayoutRecord): Promise<void> {
  // In-game notification
  await sendGameNotification(payoutRecord.winnerId, {
    type: 'payout_success',
    amount: payoutRecord.amount,
    transactionHash: payoutRecord.transactionSignature
  });
  
  // Email confirmation
  await sendPayoutEmail(payoutRecord.winnerId, payoutRecord);
  
  // Push notification
  await sendPushNotification(payoutRecord.winnerId, 'Payout received!');
}
```

## 🔍 Transaction Verification

### On-Chain Transparency
```typescript
async function verifyPayoutTransaction(signature: string): Promise<VerificationResult> {
  const transaction = await connection.getTransaction(signature, {
    commitment: 'confirmed',
    maxSupportedTransactionVersion: 0
  });
  
  if (!transaction) {
    return { verified: false, reason: 'Transaction not found' };
  }
  
  // Verify transaction details
  const transfer = transaction.transaction.message.instructions.find(
    (instruction) => instruction.programId.equals(SystemProgram.programId)
  );
  
  if (!transfer) {
    return { verified: false, reason: 'Not a transfer transaction' };
  }
  
  return {
    verified: true,
    amount: transfer.lamports / LAMPORTS_PER_SOL,
    from: transfer.keys[0].pubkey.toString(),
    to: transfer.keys[1].pubkey.toString(),
    timestamp: transaction.blockTime * 1000
  };
}
```

### Explorer Integration
- **Solscan Integration:** Direct links to transaction details
- **Solana Explorer:** Alternative blockchain viewer
- **QR Code Support:** Mobile-friendly transaction sharing
- **Export Features:** PDF receipts for large payouts

## 🛡️ Security & Reliability

### Fail-Safe Mechanisms
```typescript
async function handlePayoutFailure(payoutId: string, error: string): Promise<void> {
  const payout = await getPayout(payoutId);
  
  // Log error for monitoring
  await logPayoutError(payoutId, error);
  
  // Check retry eligibility
  if (payout.retryCount < 3) {
    payout.status = PayoutStatus.RETRYING;
    payout.retryCount += 1;
    await updatePayout(payout);
    
    // Schedule retry with exponential backoff
    const retryDelay = Math.pow(2, payout.retryCount) * 1000;
    setTimeout(() => processPayoutTransaction(payout), retryDelay);
  } else {
    // Manual intervention required
    payout.status = PayoutStatus.FAILED;
    payout.errorMessage = error;
    await updatePayout(payout);
    
    // Alert admin team
    await alertAdminTeam('PAYOUT_FAILURE', { payoutId, error });
  }
}
```

### Balance Monitoring
```typescript
async function monitorEscrowBalance(): Promise<void> {
  const walletBalance = await getWalletBalance(ESCROW_WALLET);
  const totalLocked = await getTotalLockedAmount();
  const availableBalance = walletBalance - totalLocked;
  
  // Alert if balance is low
  if (availableBalance < 10) { // Less than 10 SOL available
    await alertAdminTeam('LOW_ESCROW_BALANCE', {
      walletBalance,
      totalLocked,
      availableBalance
    });
  }
}
```

## 📈 Payout Analytics

### Performance Metrics
```typescript
interface PayoutMetrics {
  totalPayouts: number;           // Total SOL paid out
  averagePayoutTime: number;      // Average time from match to payout
  successRate: number;            // Percentage of successful payouts
  failureRate: number;            // Percentage of failed payouts
  retryRate: number;              // Percentage requiring retry
  dailyVolume: number;            // Daily payout volume
  monthlyVolume: number;          // Monthly payout volume
}
```

### Real-Time Dashboard
- **Live Payouts:** Currently processing transactions
- **Success Rate:** Real-time success percentage
- **Average Time:** Payout processing speed
- **Failure Analysis:** Common failure reasons
- **Balance Tracking:** Escrow wallet status

## 🎯 User Experience

### Payout Confirmation
```typescript
// In-game payout display
const PayoutConfirmation = ({ payoutRecord }: { payoutRecord: PayoutRecord }) => (
  <div className="payout-success">
    <h2>🏆 Victory!</h2>
    <p>You won: {payoutRecord.amount} SOL</p>
    <div className="transaction-details">
      <span>Transaction: </span>
      <a href={`https://solscan.io/tx/${payoutRecord.transactionSignature}`}>
        {payoutRecord.transactionSignature?.slice(0, 8)}...
      </a>
    </div>
    <div className="balance-update">
      New balance: {updatedBalance} SOL
    </div>
  </div>
);
```

### Mobile Optimization
- **Push Notifications:** Instant payout alerts
- **Deep Links:** Direct to Solscan transaction
- **Share Features:** Social media victory sharing
- **Receipt Storage:** In-app payout history

---

**⚡ Euras Payouts: Instant, Transparent, Verifiable**  
*Winner receives funds in seconds with full blockchain transparency*
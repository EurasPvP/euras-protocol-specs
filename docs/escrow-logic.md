# Escrow Logic Documentation

## 🔒 Escrow System Overview

Euras uses a **centralized escrow model** to ensure instant gameplay with zero gas fees during matches, while maintaining 100% security for player funds.

## 🏗️ Escrow Architecture

### Core Components
```typescript
interface EscrowSystem {
  mainWallet: string;           // Ct63e3zhh3NebJKQF2C19UHQXoamXvHyiffuLbSQiNLf
  activeLocks: EscrowLock[];   // Currently locked funds
  totalLocked: number;         // Sum of all active locks
  availableBalance: number;     // Wallet balance minus locked funds
}
```

### Lock State Management
```typescript
enum LockStatus {
  PENDING = 'pending',      // Deposit confirmed, waiting for match
  LOCKED = 'locked',        // Funds locked during active match
  RELEASED = 'released',    // Funds released to winner
  REFUNDED = 'refunded',    // Funds returned due to cancellation
  EXPIRED = 'expired'       // Lock timeout (rare edge case)
}
```

## 🔄 Lock Lifecycle

### 1. Lock Creation
```typescript
async function createEscrowLock(playerId: string, amount: number, depositTx: string): Promise<EscrowLock> {
  const lock: EscrowLock = {
    id: generateLockId(),
    playerId,
    amount,
    depositTx,
    lockTime: Date.now(),
    status: LockStatus.PENDING,
    expiryTime: Date.now() + (10 * 60 * 1000), // 10 minutes
  };
  
  await saveLock(lock);
  updateTotalLocked(amount);
  return lock;
}
```

### 2. Match Activation
```typescript
async function activateLock(lockId: string, matchId: string): Promise<void> {
  const lock = await getLock(lockId);
  lock.status = LockStatus.LOCKED;
  lock.matchId = matchId;
  lock.activeTime = Date.now();
  
  await saveLock(lock);
  emitEvent('escrow_locked', { lockId, matchId, amount: lock.amount });
}
```

### 3. Winner Release
```typescript
async function releaseToWinner(matchId: string, winnerId: string): Promise<void> {
  const locks = await getMatchLocks(matchId);
  const totalPot = locks.reduce((sum, lock) => sum + lock.amount, 0);
  const rake = totalPot * 0.035; // 3.5% rake
  const winnerPayout = totalPot - rake;
  
  // Update lock statuses
  for (const lock of locks) {
    lock.status = lock.playerId === winnerId ? LockStatus.RELEASED : LockStatus.RELEASED;
    lock.releaseTime = Date.now();
    await saveLock(lock);
  }
  
  // Process payout
  await processPayout(winnerId, winnerPayout, matchId);
  updateTotalLocked(-totalPot);
}
```

## ⏱️ Timing & Expiry

### Lock Durations
- **Pending State:** 5 minutes (waiting for opponent)
- **Active Match:** Variable (game-specific duration)
- **Buffer Period:** 30 seconds (post-match processing)
- **Total Maximum:** 10 minutes (absolute timeout)

### Expiry Handling
```typescript
async function handleExpiredLocks(): Promise<void> {
  const expiredLocks = await getExpiredLocks();
  
  for (const lock of expiredLocks) {
    if (lock.status === LockStatus.PENDING) {
      // Refund if no match found
      await refundLock(lock.id);
    } else if (lock.status === LockStatus.LOCKED) {
      // Handle active match timeout
      await handleMatchTimeout(lock.matchId);
    }
  }
}
```

## 🔐 Security Features

### Multi-Layer Protection
1. **Blockchain Verification:** All deposits confirmed on-chain
2. **Database Locks:** Backend prevents double-spending
3. **Time-Based Expiry:** Automatic refund on timeout
4. **Audit Trail:** Complete transaction history
5. **Balance Monitoring:** Real-time wallet state tracking

### Attack Prevention
```typescript
// Prevent double-locking
if (await hasActiveLock(playerId)) {
  throw new Error('Player already has active escrow lock');
}

// Prevent amount manipulation
if (lock.amount !== expectedAmount) {
  throw new Error('Lock amount mismatch');
}

// Prevent unauthorized release
if (!isValidMatchCompletion(matchId, winnerId)) {
  throw new Error('Invalid match result');
}
```

## 📊 Real-Time Monitoring

### Key Metrics
```typescript
interface EscrowMetrics {
  totalLocked: number;           // Total SOL currently locked
  activeLocks: number;           // Number of active matches
  pendingLocks: number;           // Locks waiting for matches
  averageLockTime: number;       // Average lock duration
  successRate: number;           // Percentage of successful releases
  refundRate: number;            // Percentage of refunds
}
```

### Alert System
- **High Lock Volume:** Alert if > 1000 SOL locked
- **Long Pending Locks:** Alert if lock pending > 3 minutes
- **Failed Releases:** Alert for payout failures
- **Balance Anomalies:** Alert for wallet balance issues

## 🔄 Refund Mechanisms

### Automatic Refunds
```typescript
async function processAutomaticRefund(lockId: string, reason: string): Promise<void> {
  const lock = await getLock(lockId);
  
  // Create refund transaction
  const refundTx = await createRefundTransaction(lock.playerId, lock.amount);
  
  // Update lock status
  lock.status = LockStatus.REFUNDED;
  lock.refundTx = refundTx.signature;
  lock.refundReason = reason;
  lock.refundTime = Date.now();
  
  await saveLock(lock);
  updateTotalLocked(-lock.amount);
  
  emitEvent('refund_processed', { lockId, reason, amount: lock.amount });
}
```

### Refund Triggers
- **Match Timeout:** No opponent found within time limit
- **Server Error:** Backend failure during match
- **Player Disconnect:** Network issues during gameplay
- **Game Glitch:** Technical problems affecting fairness
- **Manual Review:** Admin-initiated refunds for disputes

## 🎯 Performance Optimization

### Database Indexing
```typescript
// Optimized queries for escrow management
db.escrow.createIndex({ playerId: 1, status: 1 });
db.escrow.createIndex({ status: 1, lockTime: 1 });
db.escrow.createIndex({ matchId: 1 });
db.escrow.createIndex({ expiryTime: 1 });
```

### Caching Strategy
- **Active Locks:** Redis cache for fast access
- **Player Status:** In-memory lock checking
- **Balance Updates:** Batched wallet balance queries
- **Metrics Dashboard:** Real-time aggregated data

## 📈 Scalability Considerations

### Load Handling
- **Concurrent Matches:** Support for 1000+ simultaneous matches
- **Throughput:** 100+ deposits/second processing capability
- **Database Sharding:** Horizontal scaling for large user base
- **Rate Limiting:** Prevent abuse and ensure stability

### Future Enhancements
- **Smart Contract Integration:** Optional on-chain escrow
- **Multi-Asset Support:** USDC, USDT, and other tokens
- **Cross-Chain Compatibility:** Ethereum, BSC integration
- **DeFi Integration:** Yield generation on idle funds

---

**🔒 Euras Escrow: Maximum Speed, Maximum Security**  
*Centralized escrow for instant gameplay with blockchain-level security*
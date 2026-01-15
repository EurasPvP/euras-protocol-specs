# Rake Calculation Documentation

## 💰 Rake Model Overview

Euras implements a **transparent 3.5% house edge** on all matches, ensuring sustainable platform operations while maintaining competitive player returns.

## 📊 Rake Formula

### Mathematical Representation
$$W = P \cdot (1 - R)$$

Where:
- **W** = Winner Payout (SOL)
- **P** = Total Pot (SOL)  
- **R** = Rake Percentage (3.5% or 0.035)

### Implementation
```typescript
function calculateRake(totalPot: number, rakeRate: number = 0.035): RakeCalculation {
  const rakeAmount = totalPot * rakeRate;
  const winnerPayout = totalPot - rakeAmount;
  
  return {
    totalPot,
    rakeRate,
    rakeAmount,
    winnerPayout,
    platformRevenue: rakeAmount
  };
}
```

## 🎯 Rake Calculation Examples

### Standard Match Scenarios
| Stake per Player | Total Pot | Rake (3.5%) | Winner Receives | Platform Revenue |
|------------------|-----------|------------|-----------------|------------------|
| 0.1 SOL          | 0.2 SOL   | 0.007 SOL  | 0.193 SOL       | 0.007 SOL        |
| 0.5 SOL          | 1.0 SOL   | 0.035 SOL  | 0.965 SOL       | 0.035 SOL        |
| 1.0 SOL          | 2.0 SOL   | 0.07 SOL   | 1.93 SOL        | 0.07 SOL         |
| 2.5 SOL          | 5.0 SOL   | 0.175 SOL  | 4.825 SOL       | 0.175 SOL        |
| 5.0 SOL          | 10.0 SOL  | 0.35 SOL   | 9.65 SOL        | 0.35 SOL         |
| 10.0 SOL         | 20.0 SOL  | 0.7 SOL    | 19.3 SOL        | 0.7 SOL         |

### High-Volume Example
```typescript
// Daily platform revenue calculation
const dailyMatches = 1000;
const averageStake = 1.0; // SOL per player
const dailyVolume = dailyMatches * averageStake * 2; // 2000 SOL
const dailyRake = dailyVolume * 0.035; // 70 SOL platform revenue
```

## 💸 Revenue Distribution

### Platform Allocation
```typescript
interface RevenueAllocation {
  totalRake: number;
  operations: number;      // 40% - Servers, infrastructure
  development: number;     // 25% - New features, maintenance
  marketing: number;       // 20% - User acquisition, promotions
  support: number;         // 10% - Customer service, disputes
  reserve: number;         // 5%  - Emergency fund, stability
}
```

### Breakdown Calculation
```typescript
function allocateRake(rakeAmount: number): RevenueAllocation {
  return {
    totalRake: rakeAmount,
    operations: rakeAmount * 0.40,
    development: rakeAmount * 0.25,
    marketing: rakeAmount * 0.20,
    support: rakeAmount * 0.10,
    reserve: rakeAmount * 0.05
  };
}
```

## 📈 Rake Analytics & Reporting

### Real-Time Metrics
```typescript
interface RakeMetrics {
  dailyRake: number;           // Today's rake revenue
  weeklyRake: number;          // This week's rake revenue
  monthlyRake: number;         // This month's rake revenue
  totalRake: number;           // All-time rake revenue
  averageRakePerMatch: number; // Average rake per match
  rakeGrowthRate: number;      // Month-over-month growth
  profitMargin: number;        // Net profit percentage
}
```

### Dashboard Tracking
```typescript
async function getRakeDashboard(): Promise<RakeDashboard> {
  const today = new Date();
  const metrics = await calculateRakeMetrics(today);
  
  return {
    revenue: {
      daily: metrics.dailyRake,
      weekly: metrics.weeklyRake,
      monthly: metrics.monthlyRake,
      total: metrics.totalRake
    },
    performance: {
      averagePerMatch: metrics.averageRakePerMatch,
      growthRate: metrics.rakeGrowthRate,
      profitMargin: metrics.profitMargin
    },
    predictions: {
      tomorrow: predictRakeRevenue(1),
      nextWeek: predictRakeRevenue(7),
      nextMonth: predictRakeRevenue(30)
    }
  };
}
```

## 🔍 Transparency & Verification

### On-Chain Rake Tracking
```typescript
// Every rake transaction is recorded on-chain
interface RakeTransaction {
  id: string;
  matchId: string;
  totalPot: number;
  rakeAmount: number;
  winnerPayout: number;
  timestamp: number;
  transactionHash: string;
  blockNumber: number;
}
```

### Public Audit Trail
```typescript
async function generateRakeReport(startDate: Date, endDate: Date): Promise<RakeReport> {
  const transactions = await getRakeTransactions(startDate, endDate);
  
  return {
    period: { startDate, endDate },
    totalMatches: transactions.length,
    totalVolume: transactions.reduce((sum, tx) => sum + tx.totalPot, 0),
    totalRake: transactions.reduce((sum, tx) => sum + tx.rakeAmount, 0),
    averageRakeRate: 0.035, // Fixed at 3.5%
    transactions: transactions.map(tx => ({
      matchId: tx.matchId,
      amount: tx.rakeAmount,
      hash: tx.transactionHash,
      verified: await verifyTransaction(tx.transactionHash)
    }))
  };
}
```

## 🎮 Competitive Analysis

### Industry Comparison
| Platform | Rake Rate | Model | Speed |
|----------|-----------|-------|-------|
| Euras    | 3.5%      | Fixed | Instant |
| Stake    | 5-10%     | Variable | Instant |
| Gamdom   | 5-15%     | Variable | Instant |
| Traditional Casinos | 10-20% | Variable | Slow |

### Value Proposition
```typescript
interface CompetitiveAdvantage {
  lowerRake: boolean;          // 3.5% vs industry 5-15%
  instantPayouts: boolean;     // No withdrawal delays
  transparent: boolean;        // On-chain verification
  skillBased: boolean;         // Not gambling
  zeroGasFees: boolean;        // No transaction costs
}
```

## 📊 Financial Planning

### Revenue Projections
```typescript
async function projectRevenue(months: number): Promise<RevenueProjection> {
  const currentMetrics = await getCurrentMetrics();
  const growthRate = 0.15; // 15% monthly growth assumption
  
  const projections = [];
  let monthlyVolume = currentMetrics.monthlyVolume;
  
  for (let i = 1; i <= months; i++) {
    monthlyVolume *= (1 + growthRate);
    const monthlyRake = monthlyVolume * 0.035;
    
    projections.push({
      month: i,
      volume: monthlyVolume,
      rake: monthlyRake,
      netProfit: monthlyRake * 0.85 // After costs
    });
  }
  
  return {
    projections,
    totalProjectedRake: projections.reduce((sum, p) => sum + p.rake, 0),
    totalProjectedProfit: projections.reduce((sum, p) => sum + p.netProfit, 0)
  };
}
```

### Break-Even Analysis
```typescript
interface BreakEvenAnalysis {
  monthlyCosts: number;        // Fixed + variable costs
  requiredVolume: number;      // Volume needed to break even
  requiredMatches: number;     // Matches needed per day
  timeToBreakEven: number;     // Months until profitability
}
```

## 🎯 User Communication

### Transparent Disclosure
```typescript
// In-game rake display
const RakeDisplay = ({ totalPot }: { totalPot: number }) => {
  const rake = totalPot * 0.035;
  const winnerPayout = totalPot - rake;
  
  return (
    <div className="rake-breakdown">
      <div className="pot-info">
        <span>Total Pot: {totalPot} SOL</span>
        <span>Rake (3.5%): {rake} SOL</span>
        <span>Winner Receives: {winnerPayout} SOL</span>
      </div>
      <div className="transparency-note">
        <a href="/rake-transparency">View Rake Transparency Report</a>
      </div>
    </div>
  );
};
```

### Educational Content
- **Help Center:** Detailed rake explanation
- **FAQ:** Common questions about house edge
- **Blog Posts:** Industry comparison articles
- **Video Tutorials:** How rake works on Euras

## 🔮 Future Rake Considerations

### Potential Adjustments
```typescript
interface RakeStrategy {
  current: {
    rate: 0.035;
    model: 'fixed';
  },
  future: {
    volumeBased: boolean;      // Lower rates for high volume
    vipProgram: boolean;       // Reduced rake for loyal players
    promotionalRates: boolean; // Temporary rate reductions
    tournamentRake: boolean;    // Special event pricing
  }
}
```

### Loyalty Program Integration
```typescript
// VIP players could receive reduced rake
function calculateVIPRake(baseRake: number, vipLevel: number): number {
  const discountRate = Math.min(vipLevel * 0.005, 0.02); // Max 2% discount
  return baseRake * (1 - discountRate);
}
```

---

**💰 Euras Rake: Transparent, Competitive, Sustainable**  
*3.5% house edge with full on-chain verification and player-friendly returns*
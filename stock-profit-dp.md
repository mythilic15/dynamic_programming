# Stock Profit DP — Initial Ownership, Multiple Transactions, No Consecutive Buys or Sells

## Problem Definition

- You **already own** the stock on day 0
- You may **buy and sell multiple times**
- You **cannot perform two consecutive BUYs** (must sell before buying again)
- You **cannot perform two consecutive SELLs** (must buy before selling again)
- Given `prices[]` where `prices[i]` is the stock price on day `i`
- Find the **maximum profit**

---

## Step 1: Define the States

At each day `i`, you're in one of two states:

- `hold[i]` = max profit on day `i` while **holding** the stock
- `sold[i]` = max profit on day `i` while **not holding** the stock

---

## Step 2: Base Cases (Day 0)

- `hold[0] = 0` — you already own it on day 0, paid nothing
- `sold[0] = prices[0]` — if you sell immediately on day 0, you gain `prices[0]`

---

## Step 3: Transitions

The core question for each transition is: **how could I have ended up in this state today?**

There are always exactly two ways to arrive at each state — either you took an action, or you did nothing. The DP picks whichever gives more profit.

---

### `hold[i]` — you are holding the stock at the end of day i

To be holding on day `i`, one of two things happened:

**Option A: You were already holding yesterday and did nothing today**
```
hold[i-1]  →  (do nothing)  →  hold[i]
```
Your profit doesn't change — you just kept the stock. So the profit is whatever `hold[i-1]` was.

**Option B: You were NOT holding yesterday, and you bought today**
```
sold[i-1]  →  (buy at prices[i])  →  hold[i]
```
You had `sold[i-1]` profit going into today. Buying costs `prices[i]`, so you subtract it.
Profit = `sold[i-1] - prices[i]`

You want the best of these two options:
```
hold[i] = max(hold[i-1],  sold[i-1] - prices[i])
               ↑                ↑
          did nothing       bought today
```

**Concrete example** — `prices = [3, 1, 4, ...]`, at day 2 (price = 4):
- Option A: held since day 1, `hold[1] = 2` → profit stays 2
- Option B: was in sold state yesterday `sold[1] = 3`, buy at 4 → `3 - 4 = -1`
- `hold[2] = max(2, -1) = 2` → better to keep holding, don't buy at 4

---

### `sold[i]` — you are NOT holding the stock at the end of day i

To not be holding on day `i`, one of two things happened:

**Option A: You were not holding yesterday and did nothing today**
```
sold[i-1]  →  (do nothing)  →  sold[i]
```
You still don't own the stock. Profit stays the same: `sold[i-1]`.

**Option B: You were holding yesterday and sold today**
```
hold[i-1]  →  (sell at prices[i])  →  sold[i]
```
You had `hold[i-1]` profit while holding. Selling earns `prices[i]`, so you add it.
Profit = `hold[i-1] + prices[i]`

You want the best of these two options:
```
sold[i] = max(sold[i-1],  hold[i-1] + prices[i])
               ↑                ↑
          did nothing       sold today
```

**Concrete example** — `prices = [3, 1, 4, ...]`, at day 2 (price = 4):
- Option A: wasn't holding since day 1, `sold[1] = 3` → profit stays 3
- Option B: was holding yesterday `hold[1] = 2`, sell at 4 → `2 + 4 = 6`
- `sold[2] = max(3, 6) = 6` → better to sell at 4

---

### Why `max`?

The DP doesn't know in advance which choice leads to the best overall profit. So it keeps both options alive and picks the better one greedily at each step. By the time you reach the last day, the optimal decisions have propagated through.

---

### The full picture

```
         do nothing          sell
hold ─────────────► hold    ──────────────► sold
  ▲                                            │
  │  buy                        do nothing     │
  └────────────────────────────────────────────┘
sold ─────────────► sold
```

- `hold → hold`: do nothing (Option A of hold transition)
- `sold → hold`: buy (Option B of hold transition)
- `hold → sold`: sell (Option B of sold transition)
- `sold → sold`: do nothing (Option A of sold transition)

---

## Step 4: Trace Through an Example

`prices = [3, 1, 4, 8, 2, 9]`

| day | price | hold[i] | sold[i] | reasoning |
|-----|-------|---------|---------|-----------|
| 0 | 3 | 0 | 3 | base: own it for free / sell at 3 |
| 1 | 1 | max(0, 3-1) = **2** | max(3, 0+1) = **3** | buy at 1 → hold=2 |
| 2 | 4 | max(2, 3-4) = **2** | max(3, 2+4) = **6** | sell at 4 → sold=6 |
| 3 | 8 | max(2, 6-8) = **2** | max(6, 2+8) = **10** | sell at 8 → sold=10 |
| 4 | 2 | max(2, 10-2) = **8** | max(10, 2+2) = **10** | buy at 2 → hold=8 |
| 5 | 9 | max(8, 10-9) = **8** | max(10, 8+9) = **17** | sell at 9 → sold=17 |

Answer: `sold[5]` = **17**

Optimal path: own@3 → sell@3 (+3) → buy@1 (+2) → sell@8 (+10) → buy@2 (+8) → sell@9 (+17)

---

## Step 5: Implementation

```java
public class StockProfitDP {

    public static int maxProfit(int[] prices) {
        int n = prices.length;
        int[] hold = new int[n];
        int[] sold = new int[n];

        // base case: already own stock on day 0
        hold[0] = 0;
        sold[0] = prices[0];

        for (int i = 1; i < n; i++) {
            hold[i] = Math.max(hold[i - 1], sold[i - 1] - prices[i]);
            sold[i] = Math.max(sold[i - 1], hold[i - 1] + prices[i]);
        }

        // always best to not be holding on the last day
        return sold[n - 1];
    }

    public static void main(String[] args) {
        System.out.println(maxProfit(new int[]{3, 1, 4, 8, 2, 9})); // 17
        System.out.println(maxProfit(new int[]{5, 1, 2, 3, 4}));    // 7
        System.out.println(maxProfit(new int[]{5, 4, 3, 2, 1}));    // 5
        System.out.println(maxProfit(new int[]{1, 2, 3, 4, 5}));    // 8
    }
}
```

---

## Step 6: Why `sold[n-1]` and not `hold[n-1]`?

Holding stock on the last day with no future days to sell is worthless.
The best outcome is always to have sold — so the answer is always in `sold[n-1]`.

---

## Step 7: Space Optimization

Since each day only depends on the previous day's values, drop the arrays:

```java
public static int maxProfitOptimized(int[] prices) {
    int hold = 0;
    int sold = prices[0];

    for (int i = 1; i < prices.length; i++) {
        int prevHold = hold;
        int prevSold = sold;
        hold = Math.max(prevHold, prevSold - prices[i]);
        sold = Math.max(prevSold, prevHold + prices[i]);
    }

    return sold;
}
```

> Note: `prevHold` and `prevSold` are necessary — both transitions read the old values, so you must snapshot them before overwriting.

Time: O(n) | Space: O(1)

---

## Step 8: Verify Edge Cases

| prices | expected | reasoning |
|--------|----------|-----------|
| `[5, 4, 3, 2, 1]` | 5 | prices only fall — sell immediately on day 0 |
| `[1, 2, 3, 4, 5]` | 8 | sell@5, started free so buy@1 → profit=4, plus initial ownership sell@? |
| `[3]` | 3 | only one day — sell immediately |

---

## Key Insight: Two-State DP

This is a **two-state DP** — at every step you track two possible situations and transition between them.

| From \ To | hold | sold |
|-----------|------|------|
| hold | do nothing | sell |
| sold | buy | do nothing |

The no-consecutive-buys and no-consecutive-sells constraints are naturally enforced:
- To buy, you must be in `sold` state (can't buy twice in a row)
- To sell, you must be in `hold` state (can't sell twice in a row)

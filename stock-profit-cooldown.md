# Stock Profit DP — Buy/Sell with Cooldown (No Same-Day Buy After Sell)

## Problem Definition

- You start with **no stock** on day 0
- You may **buy and sell multiple times**
- After selling, you must **wait 1 day (cooldown)** before buying again
- You **cannot buy on the same day you sell**
- Given `prices[]` where `prices[i]` is the stock price on day `i`
- Find the **maximum profit**

---

## Step 1: Define the States

At each day `i`, you're in one of three states:

- `hold[i]` = max profit on day `i` while **holding** the stock
- `sold[i]` = max profit on day `i` having **just sold** today (now in cooldown)
- `rest[i]` = max profit on day `i` while **not holding and not in cooldown** (free to buy)

The cooldown state is the key addition here. After selling, you land in `sold` (cooldown). The next day you move to `rest` — only then can you buy again.

---

## Step 2: Base Cases (Day 0)

- `hold[0] = -prices[0]` — buy on day 0, paying `prices[0]`
- `sold[0] = 0` — can't sell on day 0 without owning, so 0 (unreachable in practice)
- `rest[0] = 0` — start with no stock, no action taken

---

## Step 3: Transitions

The question for each transition: **how could I have ended up in this state today?**

---

### `hold[i]` — holding the stock at end of day i

**Option A: You were already holding yesterday and did nothing today**
```
hold[i-1]  →  (do nothing)  →  hold[i]
```
Profit = `hold[i-1]`

**Option B: You were in rest state yesterday and bought today**
```
rest[i-1]  →  (buy at prices[i])  →  hold[i]
```
Profit = `rest[i-1] - prices[i]`

> You can only buy from `rest`, NOT from `sold` — that's the cooldown rule.

```
hold[i] = max(hold[i-1],  rest[i-1] - prices[i])
               ↑                ↑
          did nothing       bought today
```

---

### `sold[i]` — just sold today (entering cooldown)

There is only one way to arrive here: you were holding yesterday and sold today.

```
hold[i-1]  →  (sell at prices[i])  →  sold[i]
```
Profit = `hold[i-1] + prices[i]`

```
sold[i] = hold[i-1] + prices[i]
```

---

### `rest[i]` — not holding, not in cooldown (free to act)

**Option A: You were already in rest yesterday and did nothing**
```
rest[i-1]  →  (do nothing)  →  rest[i]
```
Profit = `rest[i-1]`

**Option B: You were in cooldown (sold) yesterday — cooldown expires today**
```
sold[i-1]  →  (cooldown ends)  →  rest[i]
```
Profit = `sold[i-1]`

```
rest[i] = max(rest[i-1],  sold[i-1])
               ↑               ↑
          did nothing     cooldown expired
```

---

### The full picture

```
         do nothing
hold ─────────────► hold
  ▲                    │
  │  buy               │ sell
  │                    ▼
rest ◄────────────  sold
         cooldown
         expires
  ▲
  │ do nothing
  └──────────────  rest
```

- `hold → hold`: do nothing
- `rest → hold`: buy
- `hold → sold`: sell (enters cooldown)
- `sold → rest`: cooldown expires (next day, free to buy)
- `rest → rest`: do nothing

---

## Step 4: Trace Through an Example

`prices = [1, 2, 3, 0, 2]`

| day | price | hold[i] | sold[i] | rest[i] | reasoning |
|-----|-------|---------|---------|---------|-----------|
| 0 | 1 | -1 | 0 | 0 | buy at 1 → hold=-1 |
| 1 | 2 | max(-1, 0-2) = **-1** | -1+2 = **1** | max(0, 0) = **0** | sell at 2 → sold=1 |
| 2 | 3 | max(-1, 0-3) = **-1** | -1+3 = **2** | max(0, 1) = **1** | cooldown expires → rest=1 |
| 3 | 0 | max(-1, 1-0) = **1** | -1+0 = **-1** | max(1, 2) = **2** | buy at 0 → hold=1 |
| 4 | 2 | max(1, 2-2) = **1** | 1+2 = **3** | max(2, -1) = **2** | sell at 2 → sold=3 |

Answer: `max(sold[4], rest[4])` = `max(3, 2)` = **3**

Optimal path: buy@1 → sell@2 (+1) → cooldown day 2 → buy@0 → sell@2 (+2) = **3**

---

## Step 5: Implementation

Instead of a separate `rest` array, we observe that `rest[i-1]` is simply `sold[i-2]` — the cooldown has expired, so the best "free cash" available is whatever profit was locked in two days ago. We replace the array with an if condition.

```java
public class StockCooldownDP {

    // O(n) time | O(n) space — no rest array, cooldown handled via if condition
    public static int maxProfit(int[] prices) {
        int n = prices.length;
        if (n == 0) return 0;

        int[] hold = new int[n];
        int[] sold = new int[n];

        hold[0] = -prices[0];
        sold[0] = 0;

        for (int i = 1; i < n; i++) {
            // cooldown: can only buy using profit from sold[i-2]
            // if i == 1, no i-2 exists yet, so free cash is 0
            int freeCash = (i >= 2) ? sold[i - 2] : 0;

            hold[i] = Math.max(hold[i - 1], freeCash - prices[i]);
            sold[i] = hold[i - 1] + prices[i];
        }

        return Math.max(sold[n - 1], 0);
    }

    public static void main(String[] args) {
        System.out.println(maxProfit(new int[]{1, 2, 3, 0, 2}));     // 3
        System.out.println(maxProfit(new int[]{1}));                  // 0
        System.out.println(maxProfit(new int[]{2, 1, 4}));            // 3
        System.out.println(maxProfit(new int[]{6, 1, 3, 2, 4, 7}));  // 6
    }
}
```

---

## Step 6: Why `max(sold[n-1], 0)`?

Unlike the previous problem, you don't start owning the stock — so `hold[0] = -prices[0]` (you paid to buy). On the last day:

- `sold[n-1]`: you just sold today — profit is locked in
- `hold[n-1]`: still holding with no future day to sell — worthless

So the answer is `max(sold[n-1], 0)` (0 in case no trade was ever profitable).

---

## Step 7: Space Optimization

We only need `hold`, `sold`, and `sold[i-2]` (two days back). Track just three variables:

```java
public static int maxProfitOptimized(int[] prices) {
    if (prices.length == 0) return 0;

    int hold = -prices[0];
    int sold = 0;
    int prevSold = 0; // acts as sold[i-2]

    for (int i = 1; i < prices.length; i++) {
        int prevHold = hold;
        int currSold = hold + prices[i];

        // buy using prevSold (sold two days ago = cooldown expired)
        hold = Math.max(hold, prevSold - prices[i]);
        prevSold = sold;  // shift: today's sold becomes tomorrow's prevSold
        sold = currSold;
    }

    return Math.max(sold, 0);
}
```

> `prevSold` slides forward each iteration — it always holds `sold[i-2]` relative to the current `i`.

Time: O(n) | Space: O(1)

---

## Step 8: Verify Edge Cases

| prices | expected | reasoning |
|--------|----------|-----------|
| `[1]` | 0 | only one day — can't buy and sell |
| `[2, 1]` | 0 | prices fall — never profitable to trade |
| `[1, 2]` | 1 | buy@1, sell@2 |
| `[1, 2, 3]` | 2 | buy@1, sell@3 (skip day 2 sell due to cooldown math) |
| `[6, 1, 3, 2, 4, 7]` | 6 | buy@1, sell@3, cooldown, buy@2, sell@7 → 2+5=6 (wait, actually buy@1 sell@7 skipping = 6) |

---

## Key Insight: Two Arrays, One if Condition

By recognizing that `rest[i-1] = sold[i-2]`, we eliminate the `rest` array entirely. The cooldown is enforced by a single if condition — when buying, look back two days instead of one:

```
hold[i] = max(hold[i-1],  sold[i-2] - prices[i])   // if i >= 2, else 0 - prices[i]
sold[i] = hold[i-1] + prices[i]
```

The state machine still has three logical states, but we only need two arrays to represent them. The "rest" state is implicit — it's just the value of `sold` from two days ago.

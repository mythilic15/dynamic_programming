# Day 2: Bottom-Up DP — Tabulation

## Learning Objectives
By the end of Day 2, you should be able to:
- Explain the difference between top-down and bottom-up DP
- Build a DP table iteratively from base cases
- Define state, base cases, and transition formulas
- Solve linear DP problems using tabulation

---

## 1. What Is Tabulation?

Tabulation is a bottom-up dynamic programming technique where you solve a problem by filling in a table (usually an array) starting from the smallest subproblems and working your way up to the final answer — no recursion involved.

The name comes from the idea of literally building a table of results. Each cell in the table represents the answer to a subproblem, and you fill it in using previously computed cells.

### How it works

1. Allocate an array `dp[]` where each index represents a subproblem.
2. Fill in the base cases directly (the smallest known answers).
3. Iterate forward, computing each `dp[i]` from earlier entries using a transition formula.
4. Return `dp[n]` (or whatever index holds the final answer).

### Why it matters

- No call stack — eliminates stack overflow risk for large inputs.
- Cache-friendly — iterating over an array sequentially is fast in practice.
- Forces you to think clearly about state, which makes the solution easier to reason about and optimize.

### The mental model

Think of it like filling out a spreadsheet row by row. You never need a value you haven't computed yet — every cell only looks backward.

```
Fibonacci example:
Index:  0  1  2  3  4  5  6  7  8  9  10
dp[]:   0  1  1  2  3  5  8  13 21 34 55
                ↑
         dp[2] = dp[1] + dp[0]
```

Each value depends only on what came before it. That's the essence of tabulation.

---

## 2. Top-Down vs Bottom-Up

| | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| Direction | Start from the answer, recurse down | Start from base cases, build up |
| Implementation | Recursion + cache | Iterative + array |
| Stack overflow risk | Yes (deep recursion) | No |
| Ease of understanding | More intuitive | Requires careful state design |
| Performance | Slightly slower (function call overhead) | Slightly faster |

Both have the same time and space complexity in most cases. Choose based on preference and problem constraints.

---

## 3. The 3-Step Tabulation Framework

### Step 1: Define the State
What does `dp[i]` represent?

### Step 2: Set Base Cases
What are the smallest known values?

### Step 3: Write the Transition
How does `dp[i]` relate to previous states?

---

## 4. Example: Fibonacci — Bottom-Up

```java
public class Fibonacci {

    public static int fib(int n) {
        if (n <= 1) return n;

        int[] dp = new int[n + 1];

        // Base cases
        dp[0] = 0;
        dp[1] = 1;

        // Transition: dp[i] = dp[i-1] + dp[i-2]
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }

    public static void main(String[] args) {
        System.out.println(fib(10)); // 55
    }
}
```

Space optimization — we only need the last two values:

```java
public static int fibOptimized(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

---

## 5. Problem: Climbing Stairs (LeetCode 70)

You can climb 1 or 2 steps at a time. How many distinct ways to reach step `n`?

### Step 1: Define the State
`dp[i]` = the number of distinct ways to reach step `i`

### Step 2: Identify the Base Cases
- `dp[0] = 1` — there's exactly 1 way to be at the ground (do nothing)
- `dp[1] = 1` — only one way to reach step 1 (take a single 1-step)

### Step 3: Write the Transition
To reach step `i`, you either:
- came from step `i-1` (took 1 step), or
- came from step `i-2` (took 2 steps)

So: `dp[i] = dp[i-1] + dp[i-2]`

This is identical in structure to Fibonacci — the number of ways grows the same way.

### Step 4: Trace Through an Example

For `n = 4`:

| i | dp[i] | Reasoning |
|---|-------|-----------|
| 0 | 1 | base case |
| 1 | 1 | base case |
| 2 | 2 | dp[1] + dp[0] = 1 + 1 |
| 3 | 3 | dp[2] + dp[1] = 2 + 1 |
| 4 | 5 | dp[3] + dp[2] = 3 + 2 |

Answer: **5 ways** to climb 4 stairs.

The 5 paths are: `1+1+1+1`, `1+1+2`, `1+2+1`, `2+1+1`, `2+2`

### Step 5: Implement It

```java
public class ClimbingStairs {

    public static int climbStairs(int n) {
        if (n <= 1) return 1;

        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;

        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }

    public static void main(String[] args) {
        System.out.println(climbStairs(4)); // 5
        System.out.println(climbStairs(5)); // 8
    }
}
```

### Step 6: Space Optimization (Optional)

Since each step only needs the previous two values, you can drop the array entirely:

```java
public static int climbStairs(int n) {
    if (n <= 1) return 1;
    int prev2 = 1, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

Time: O(n) | Space: O(1)

---

## 6. Problem: Min Cost Climbing Stairs (LeetCode 746)

Each step has a cost. You can start from index 0 or 1. Find the minimum cost to reach the top.

### State
`dp[i]` = minimum cost to reach step `i`

### Base Cases
- `dp[0] = cost[0]`
- `dp[1] = cost[1]`

### Transition
`dp[i] = cost[i] + min(dp[i-1], dp[i-2])`

```java
public class MinCostClimbingStairs {

    public static int minCostClimbingStairs(int[] cost) {
        int n = cost.length;
        int[] dp = new int[n];

        dp[0] = cost[0];
        dp[1] = cost[1];

        for (int i = 2; i < n; i++) {
            dp[i] = cost[i] + Math.min(dp[i - 1], dp[i - 2]);
        }

        return Math.min(dp[n - 1], dp[n - 2]);
    }

    public static void main(String[] args) {
        System.out.println(minCostClimbingStairs(new int[]{10, 15, 20}));    // 15
        System.out.println(minCostClimbingStairs(new int[]{1, 100, 1, 1, 1, 100, 1, 1, 100, 1})); // 6
    }
}
```

---

## 7. Problem: House Robber (LeetCode 198)

You cannot rob two adjacent houses. Maximize the total amount robbed.

### State
`dp[i]` = maximum money robbed from houses 0 to i

### Base Cases
- `dp[0] = nums[0]`
- `dp[1] = max(nums[0], nums[1])`

### Transition
`dp[i] = max(dp[i-1], dp[i-2] + nums[i])`
(either skip this house, or rob it and add to best from two houses back)

```java
public class HouseRobber {

    public static int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) return nums[0];

        int[] dp = new int[n];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);

        for (int i = 2; i < n; i++) {
            dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
        }

        return dp[n - 1];
    }

    public static void main(String[] args) {
        System.out.println(rob(new int[]{1, 2, 3, 1})); // 4
        System.out.println(rob(new int[]{2, 7, 9, 3, 1})); // 12
    }
}
```

---

## 8. Array Exercises

### Exercise 1: Best Time to Buy and Sell Stock
Given an array `prices` where `prices[i]` is the stock price on day `i`, find the maximum profit from one buy and one sell (buy before sell).

```
Input:  [7, 1, 5, 3, 6, 4]
Output: 5  (buy at 1, sell at 6)

Input:  [7, 6, 4, 3, 1]
Output: 0  (prices only fall, no profit possible)
```

State hint: `dp[i]` = max profit considering prices up to day `i`.

<details>
<summary>Hint</summary>

Track the minimum price seen so far as you iterate. At each day:
`dp[i] = max(dp[i-1], prices[i] - minPriceSoFar)`
</details>

---

### Exercise 2: Maximum Subarray Sum (Kadane's Algorithm)
Given an integer array, find the contiguous subarray with the largest sum.

```
Input:  [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6  (subarray [4, -1, 2, 1])

Input:  [1]
Output: 1

Input:  [-1, -2, -3]
Output: -1
```

State hint: `dp[i]` = maximum subarray sum ending at index `i`.
Transition: `dp[i] = max(nums[i], dp[i-1] + nums[i])`

---

### Exercise 3: Array Jump — Count Paths with Exactly K Steps
Given an array of size `n` and a number `k`, count the number of ways to go from index 0 to index `n-1` using exactly `k` jumps, where each jump moves exactly 1 or 2 positions forward.

```
Input:  n = 5, k = 3
Output: 3
Paths: 0→1→2→4, 0→1→3→4, 0→2→3→4
```

State hint: `dp[i][steps]` = number of ways to reach index `i` in exactly `steps` jumps.

---

### Exercise 4: Trapping Rain Water
Given an elevation map as an array, compute how much water it can trap after raining.

```
Input:  [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
Output: 6

Input:  [4, 2, 0, 3, 2, 5]
Output: 9
```

State hint: Precompute `leftMax[i]` and `rightMax[i]` arrays.
Water at index `i` = `min(leftMax[i], rightMax[i]) - height[i]`

---

### Solutions Skeleton

```java
public class Day2ArrayExercises {

    // Exercise 1: Best Time to Buy and Sell Stock
    public static int maxProfit(int[] prices) {
        int[] dp = new int[prices.length];
        int minPrice = prices[0];
        dp[0] = 0;

        for (int i = 1; i < prices.length; i++) {
            minPrice = Math.min(minPrice, prices[i]);
            dp[i] = Math.max(dp[i - 1], prices[i] - minPrice);
        }

        return dp[prices.length - 1];
    }

    // Exercise 2: Maximum Subarray (Kadane's)
    public static int maxSubArray(int[] nums) {
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        int maxSum = dp[0];

        for (int i = 1; i < nums.length; i++) {
            dp[i] = Math.max(nums[i], dp[i - 1] + nums[i]);
            maxSum = Math.max(maxSum, dp[i]);
        }

        return maxSum;
    }

    // Exercise 4: Trapping Rain Water
    public static int trap(int[] height) {
        int n = height.length;
        int[] leftMax = new int[n];
        int[] rightMax = new int[n];

        leftMax[0] = height[0];
        for (int i = 1; i < n; i++)
            leftMax[i] = Math.max(leftMax[i - 1], height[i]);

        rightMax[n - 1] = height[n - 1];
        for (int i = n - 2; i >= 0; i--)
            rightMax[i] = Math.max(rightMax[i + 1], height[i]);

        int water = 0;
        for (int i = 0; i < n; i++)
            water += Math.min(leftMax[i], rightMax[i]) - height[i];

        return water;
    }

    public static void main(String[] args) {
        System.out.println(maxProfit(new int[]{7, 1, 5, 3, 6, 4})); // 5
        System.out.println(maxProfit(new int[]{7, 6, 4, 3, 1}));    // 0

        System.out.println(maxSubArray(new int[]{-2, 1, -3, 4, -1, 2, 1, -5, 4})); // 6
        System.out.println(maxSubArray(new int[]{-1, -2, -3}));                     // -1

        System.out.println(trap(new int[]{0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1})); // 6
        System.out.println(trap(new int[]{4, 2, 0, 3, 2, 5}));                    // 9
    }
}
```

---

## 9. More Exercises

### Exercise 5: Coin Change (LeetCode 322)
Given an array of coin denominations and a target amount, find the minimum number of coins needed to make that amount. Return `-1` if it's not possible.

```
Input:  coins = [1, 5, 6, 9], amount = 11
Output: 2  (5 + 6)

Input:  coins = [2], amount = 3
Output: -1

Input:  coins = [1, 2, 5], amount = 11
Output: 3  (5 + 5 + 1)
```

#### Step 1: Define the State
`dp[i]` = minimum number of coins needed to make amount `i`

#### Step 2: Identify the Base Cases
- `dp[0] = 0` — zero coins needed to make amount 0
- All other `dp[i]` initialized to `amount + 1` (a sentinel meaning "not reachable yet")

#### Step 3: Write the Transition
For each amount `i`, try every coin. If the coin fits (`coin <= i`), check if using it gives a better result:

`dp[i] = min(dp[i], dp[i - coin] + 1)`

The idea: if you can make `i - coin` with some coins, adding this one coin makes amount `i`.

#### Step 4: Trace Through an Example

`coins = [1, 5, 6, 9]`, `amount = 11`

| i | coins tried | dp[i] |
|---|-------------|-------|
| 0 | — | 0 |
| 1 | coin=1: dp[0]+1=1 | 1 |
| 2 | coin=1: dp[1]+1=2 | 2 |
| 3 | coin=1: dp[2]+1=3 | 3 |
| 4 | coin=1: dp[3]+1=4 | 4 |
| 5 | coin=1: 5, coin=5: dp[0]+1=1 | 1 |
| 6 | coin=1: 2, coin=5: dp[1]+1=2, coin=6: dp[0]+1=1 | 1 |
| 7 | coin=1: 2, coin=6: dp[1]+1=2 | 2 |
| 8 | coin=1: 3, coin=6: dp[2]+1=3 | 3 |
| 9 | coin=1: 4, coin=5: dp[4]+1=5, coin=6: dp[3]+1=4, coin=9: dp[0]+1=1 | 1 |
| 10 | coin=1: 2, coin=5: dp[5]+1=2, coin=9: dp[1]+1=2 | 2 |
| 11 | coin=1: 3, coin=5: dp[6]+1=2, coin=6: dp[5]+1=2 | **2** |

Answer: **2 coins** (5 + 6)

#### Step 5: Implement It

```java
public static int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);  // sentinel: "not reachable"
    dp[0] = 0;

    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }

    // if still sentinel, amount was unreachable
    return dp[amount] > amount ? -1 : dp[amount];
}
```

#### Step 6: Why `amount + 1` as the sentinel?
The maximum coins you'd ever need is `amount` itself (using all 1-cent coins). So `amount + 1` is safely "infinity" — any real answer will be smaller, and if `dp[amount]` is still `amount + 1` after filling, it means the amount is unreachable → return `-1`.

Complexity: Time O(amount × coins.length) | Space O(amount)

---

### Exercise 6: Longest Increasing Subsequence (LeetCode 300)
Find the length of the longest strictly increasing subsequence in an array. Elements don't have to be contiguous.

```
Input:  [10, 9, 2, 5, 3, 7, 101, 18]
Output: 4  (2, 3, 7, 101)

Input:  [0, 1, 0, 3, 2, 3]
Output: 4

Input:  [7, 7, 7, 7]
Output: 1
```

State: `dp[i]` = length of the longest increasing subsequence ending at index `i`
Base case: `dp[i] = 1` for all `i` (every element is a subsequence of length 1)
Transition: for each `j < i`, if `nums[j] < nums[i]`, then `dp[i] = max(dp[i], dp[j] + 1)`

<details>
<summary>Hint</summary>

You need a nested loop: outer loop over `i`, inner loop over all `j < i`. The answer is the max value in the entire `dp[]` array, not just `dp[n-1]`.
</details>

---

### Exercise 7: Unique Paths (LeetCode 62)
A robot starts at the top-left corner of an `m x n` grid and can only move **right** or **down**. Count the number of unique paths to reach the bottom-right corner.

```
Input:  m = 3, n = 7
Output: 28

Input:  m = 3, n = 2
Output: 3
Paths: R→D→D, D→R→D, D→D→R
```

#### Step 1: Define the State
`dp[i][j]` = number of unique paths to reach cell `(i, j)` from `(0, 0)`

#### Step 2: Identify the Base Cases
- **First row** (`i = 0`): the robot can only travel right, so there's exactly **1 way** to reach any cell → `dp[0][j] = 1`
- **First column** (`j = 0`): the robot can only travel down, so there's exactly **1 way** to reach any cell → `dp[i][0] = 1`

#### Step 3: Write the Transition
To reach cell `(i, j)`, the robot must have come from either:
- the cell **above** `(i-1, j)` — moved down, or
- the cell **to the left** `(i, j-1)` — moved right

So: `dp[i][j] = dp[i-1][j] + dp[i][j-1]`

#### Step 4: Trace Through an Example

For `m = 3, n = 3` (3 rows, 3 columns):

Fill base cases first (all 1s on edges), then fill interior:

```
     col0  col1  col2
row0 [  1,    1,    1  ]
row1 [  1,    2,    3  ]
row2 [  1,    3,    6  ]
```

- `dp[1][1]` = dp[0][1] + dp[1][0] = 1 + 1 = **2**
- `dp[1][2]` = dp[0][2] + dp[1][1] = 1 + 2 = **3**
- `dp[2][1]` = dp[1][1] + dp[2][0] = 2 + 1 = **3**
- `dp[2][2]` = dp[1][2] + dp[2][1] = 3 + 3 = **6**

Answer: **6 unique paths** for a 3×3 grid.

Visually, each cell's value is the sum of the cell above it and the cell to its left — just like Pascal's triangle:

```
  S . . .
  . . . .
  . . . E
```

Every path from S to E is exactly `(m-1) + (n-1)` moves long — some combination of downs and rights.

#### Step 5: Implement It

```java
public static int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];

    // Base cases: first row and first column are all 1s
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;

    // Fill the rest of the grid
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }

    return dp[m - 1][n - 1];
}
```

#### Step 6: Space Optimization

You only ever look at the current row and the row above it. So you can compress the 2D table into a single 1D array:

```java
public static int uniquePathsOptimized(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1);          // base case: first row all 1s

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[j] = dp[j] + dp[j - 1];
            // dp[j]     = value from row above (not yet overwritten)
            // dp[j - 1] = value from left in current row (already updated)
        }
    }

    return dp[n - 1];
}
```

Time: O(m × n) | Space: O(n)

---

### Exercise 8: Word Break (LeetCode 139)
Given a string `s` and a dictionary of words, determine if `s` can be segmented into a space-separated sequence of dictionary words.

```
Input:  s = "leetcode", wordDict = ["leet", "code"]
Output: true

Input:  s = "applepenapple", wordDict = ["apple", "pen"]
Output: true  ("apple pen apple")

Input:  s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
Output: false
```

State: `dp[i]` = true if the substring `s[0..i-1]` can be segmented using the dictionary
Base case: `dp[0] = true` (empty string is always valid)
Transition: `dp[i] = true` if there exists some `j < i` where `dp[j] == true` and `s[j..i-1]` is in the dictionary

<details>
<summary>Hint</summary>

Use a `HashSet` for O(1) dictionary lookups. The outer loop goes from 1 to `n`, the inner loop tries all split points `j` from 0 to `i-1`.
</details>

---

### Exercise 9: Triangle Minimum Path Sum (LeetCode 120)
Given a triangle array, find the minimum path sum from top to bottom. At each step you may move to an adjacent number in the row below (index `i` or `i+1`).

```
Input:
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
Output: 11  (2 → 3 → 5 → 1)
```

State: `dp[i][j]` = minimum path sum to reach cell `(i, j)`
Base case: `dp[0][0] = triangle[0][0]`
Transition: `dp[i][j] = triangle[i][j] + min(dp[i-1][j-1], dp[i-1][j])`
(handle edge columns carefully — leftmost and rightmost only have one parent)

<details>
<summary>Hint</summary>

You can do this in-place by modifying the triangle itself, or use a 1D dp array and iterate bottom-up from the last row to the first. The bottom-up 1D approach is the cleanest space optimization.
</details>

---

### Solutions Skeleton

```java
import java.util.*;

public class Day2MoreExercises {

    // Exercise 5: Coin Change
    public static int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);
        dp[0] = 0;

        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }

        return dp[amount] > amount ? -1 : dp[amount];
    }

    // Exercise 6: Longest Increasing Subsequence
    public static int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int max = 1;

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            max = Math.max(max, dp[i]);
        }

        return max;
    }

    // Exercise 7: Unique Paths
    public static int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];

        for (int i = 0; i < m; i++) dp[i][0] = 1;
        for (int j = 0; j < n; j++) dp[0][j] = 1;

        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];

        return dp[m - 1][n - 1];
    }

    // Exercise 8: Word Break
    public static boolean wordBreak(String s, List<String> wordDict) {
        Set<String> dict = new HashSet<>(wordDict);
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true;

        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && dict.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }

        return dp[n];
    }

    // Exercise 9: Triangle Minimum Path Sum (bottom-up 1D)
    public static int minimumTotal(List<List<Integer>> triangle) {
        int n = triangle.size();
        int[] dp = new int[n];

        // Start from the bottom row
        for (int j = 0; j < n; j++)
            dp[j] = triangle.get(n - 1).get(j);

        // Work upward
        for (int i = n - 2; i >= 0; i--)
            for (int j = 0; j <= i; j++)
                dp[j] = triangle.get(i).get(j) + Math.min(dp[j], dp[j + 1]);

        return dp[0];
    }

    public static void main(String[] args) {
        System.out.println(coinChange(new int[]{1, 5, 6, 9}, 11)); // 2
        System.out.println(coinChange(new int[]{2}, 3));            // -1

        System.out.println(lengthOfLIS(new int[]{10, 9, 2, 5, 3, 7, 101, 18})); // 4
        System.out.println(lengthOfLIS(new int[]{7, 7, 7, 7}));                  // 1

        System.out.println(uniquePaths(3, 7)); // 28
        System.out.println(uniquePaths(3, 2)); // 3

        System.out.println(wordBreak("leetcode", Arrays.asList("leet", "code")));              // true
        System.out.println(wordBreak("catsandog", Arrays.asList("cats", "dog", "sand", "and", "cat"))); // false

        List<List<Integer>> triangle = Arrays.asList(
            Arrays.asList(2),
            Arrays.asList(3, 4),
            Arrays.asList(6, 5, 7),
            Arrays.asList(4, 1, 8, 3)
        );
        System.out.println(minimumTotal(triangle)); // 11
    }
}
```

---

## 10. Permutation & Combination Exercises

These problems follow the same tabulation pattern as Coin Change — the key difference is **order matters** in permutations and **order doesn't matter** in combinations.

---

### Exercise A: Coin Change II — Count Combinations (LeetCode 518)
Given coin denominations and a target amount, count the number of **distinct combinations** (order doesn't matter) that make up the amount.

```
Input:  coins = [1, 2, 5], amount = 5
Output: 4
Combinations: [1,1,1,1,1], [1,1,1,2], [1,2,2], [5]

Input:  coins = [2], amount = 3
Output: 0
```

State: `dp[i]` = number of combinations to make amount `i`
Base case: `dp[0] = 1` (one way to make 0 — use no coins)
Transition: `dp[i] += dp[i - coin]` for each coin ≤ i

> Key: loop **coins in outer, amounts in inner** — this ensures each coin is only counted once per combination.

<details>
<summary>Hint</summary>

```java
for (int coin : coins)
    for (int i = coin; i <= amount; i++)
        dp[i] += dp[i - coin];
```
</details>

---

### Exercise B: Combination Sum IV — Count Permutations (LeetCode 377)
Given an array of distinct integers and a target, count the number of **permutations** (order matters) that sum to the target.

```
Input:  nums = [1, 2, 3], target = 4
Output: 7
Permutations: [1,1,1,1], [1,1,2], [1,2,1], [2,1,1], [1,3], [3,1], [2,2]

Input:  nums = [9], target = 3
Output: 0
```

State: `dp[i]` = number of ordered sequences that sum to `i`
Base case: `dp[0] = 1`
Transition: `dp[i] += dp[i - num]` for each num ≤ i

> Key: loop **amounts in outer, nums in inner** — this counts every ordering separately.

<details>
<summary>Hint</summary>

```java
for (int i = 1; i <= target; i++)
    for (int num : nums)
        if (num <= i)
            dp[i] += dp[i - num];
```
</details>

---

### Exercise C: Perfect Squares (LeetCode 279)
Given an integer `n`, find the minimum number of perfect squares (1, 4, 9, 16...) that sum to `n`.

```
Input:  n = 12
Output: 3  (4 + 4 + 4)

Input:  n = 13
Output: 2  (4 + 9)
```

State: `dp[i]` = minimum perfect squares that sum to `i`
Base case: `dp[0] = 0`, all others = `n + 1` (sentinel)
Transition: `dp[i] = min(dp[i], dp[i - j*j] + 1)` for each j where `j*j <= i`

<details>
<summary>Hint</summary>

Generate squares on the fly inside the loop: `for (int j = 1; j * j <= i; j++)`. Same sentinel trick as Coin Change.
</details>

---

### Solutions Skeleton

```java
import java.util.*;

public class Day2PermCombExercises {

    // Exercise A: Coin Change II — Combinations (order doesn't matter)
    public static int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
        dp[0] = 1;

        for (int coin : coins)           // outer: coins
            for (int i = coin; i <= amount; i++)  // inner: amounts
                dp[i] += dp[i - coin];

        return dp[amount];
    }

    // Exercise B: Combination Sum IV — Permutations (order matters)
    public static int combinationSum4(int[] nums, int target) {
        int[] dp = new int[target + 1];
        dp[0] = 1;

        for (int i = 1; i <= target; i++)   // outer: amounts
            for (int num : nums)             // inner: nums
                if (num <= i)
                    dp[i] += dp[i - num];

        return dp[target];
    }

    // Exercise C: Perfect Squares
    public static int numSquares(int n) {
        int[] dp = new int[n + 1];
        Arrays.fill(dp, n + 1);
        dp[0] = 0;

        for (int i = 1; i <= n; i++)
            for (int j = 1; j * j <= i; j++)
                dp[i] = Math.min(dp[i], dp[i - j * j] + 1);

        return dp[n];
    }

    public static void main(String[] args) {
        System.out.println(change(5, new int[]{1, 2, 5}));       // 4
        System.out.println(change(3, new int[]{2}));              // 0

        System.out.println(combinationSum4(new int[]{1, 2, 3}, 4)); // 7
        System.out.println(combinationSum4(new int[]{9}, 3));        // 0

        System.out.println(numSquares(12)); // 3
        System.out.println(numSquares(13)); // 2
    }
}
```

---

### The Critical Difference: Loop Order

| | Outer loop | Inner loop | Counts |
|---|---|---|---|
| Combinations | coins | amounts | `[1,2]` and `[2,1]` as **same** |
| Permutations | amounts | coins | `[1,2]` and `[2,1]` as **different** |

This single swap in loop order is what separates the two. Everything else — state, base case, transition — stays the same.

---

## 11. Key Takeaways

- Tabulation builds the answer from the ground up — no recursion needed
- Always ask: "What does `dp[i]` mean?" before writing any code
- Base cases are the foundation — get them wrong and everything breaks
- The transition formula is the heart of the solution

---

## 11. Day 2 Checklist
- [ ] Can convert a memoized solution to tabulation
- [ ] Solved Climbing Stairs with correct state definition
- [ ] Solved House Robber and explained the transition
- [ ] Understand when to use space-optimized DP (O(1) space)

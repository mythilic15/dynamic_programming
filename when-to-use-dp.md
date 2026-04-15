# When to Use Dynamic Programming

## What Is DP, Really?

Dynamic Programming is not an algorithm — it's a **problem-solving technique**. It applies whenever a problem can be broken into smaller subproblems that overlap, and the solution to the whole can be built from solutions to the parts.

The two-line definition:
> 1. The problem has **overlapping subproblems** — the same smaller problem is solved more than once.
> 2. The problem has **optimal substructure** — the best solution to the whole is built from best solutions to its parts.

If both are true, DP will work. If either is missing, DP won't help.

---

## The Two Properties in Plain English

### Overlapping Subproblems

When you recurse naively, you end up solving the same subproblem repeatedly. DP stores those results so you only solve each one once.

```
Fibonacci without DP:

fib(5)
├── fib(4)
│   ├── fib(3)          ← computed here
│   │   ├── fib(2)
│   │   └── fib(1)
│   └── fib(2)
└── fib(3)              ← computed AGAIN (wasted work)
    ├── fib(2)
    └── fib(1)

fib(3) is computed twice, fib(2) three times.
With n=50, this explodes to billions of redundant calls.
```

DP stores `fib(3)` the first time it's computed. Every subsequent call just looks it up.

### Optimal Substructure

The optimal answer to the big problem is composed of optimal answers to smaller problems.

```
Shortest path from A to C through B:
  shortest(A → C) = shortest(A → B) + shortest(B → C)

If the path A→B→C is optimal, then A→B must be the shortest way to reach B,
and B→C must be the shortest way from B to C.
You can't have a suboptimal segment inside an optimal whole path.
```

Not all problems have this. For example, the **longest path** in a graph does NOT have optimal substructure — the longest path from A to C through B is not necessarily the longest A→B plus longest B→C (they might share nodes).

---

## How to Recognize a DP Problem

Ask these questions when you see a problem:

```
1. Does it ask for a COUNT, MIN, MAX, or YES/NO over all possibilities?
2. Can I make a sequence of decisions where each decision affects future ones?
3. If I try brute force recursion, do I compute the same subproblem twice?
4. Can I define the answer to the big problem in terms of answers to smaller versions?
```

If yes to most of these → DP is likely the right tool.

### Signal Words in Problem Statements

| Words | Likely DP type |
|---|---|
| "number of ways to..." | Counting DP |
| "minimum cost / steps / jumps" | Optimization DP |
| "maximum profit / sum / length" | Optimization DP |
| "can you reach / is it possible" | Reachability DP |
| "longest subsequence / substring" | Sequence DP |
| "how many distinct paths" | Grid / path counting DP |
| "partition into subsets" | Knapsack DP |

---

## When NOT to Use DP

DP is often the wrong tool when:

- **No overlapping subproblems** — each subproblem is unique. Example: merge sort splits the array into non-overlapping halves, so there's nothing to cache.
- **Greedy works** — if a locally optimal choice always leads to a globally optimal solution, greedy is simpler and faster. Example: activity selection, fractional knapsack.
- **The state space is too large** — if the state requires tracking too many variables, the DP table becomes impractical.
- **The problem asks for the actual path, not just the value** — DP gives you the optimal value easily; reconstructing the actual solution requires extra bookkeeping.

```
Greedy vs DP:

Coin change with coins [1, 5, 6, 9], amount = 11:
  Greedy: 9 + 1 + 1 = 3 coins  ← WRONG
  DP:     5 + 6    = 2 coins  ← CORRECT

Greedy fails here because a locally big coin (9) blocks a better combination.
DP explores all options and picks the true minimum.

But for coins [1, 5, 10, 25] (standard denominations):
  Greedy works perfectly — always take the largest coin that fits.
```

---

## The Two Types of DP

Both approaches solve the same problem and give the same answer. They differ in **direction** and **implementation style**.

```
                    ┌─────────────────────────────────┐
                    │         The Problem              │
                    └─────────────────────────────────┘
                           /                \
              Top-Down                    Bottom-Up
           (Memoization)               (Tabulation)
         Start at the answer          Start at base cases
         Recurse downward             Build upward iteratively
         Cache results                Fill a table
```

---

## Type 1: Memoization (Top-Down)

### What It Is

Start with recursion. Solve the problem naturally from the top. The first time you compute a subproblem, store the result. Every subsequent call returns the cached value instead of recomputing.

### The Template

```java
Map<State, Result> memo = new HashMap<>();

Result solve(State state) {
    // 1. Base case — smallest known answer
    if (isBaseCase(state)) return baseAnswer;

    // 2. Return cached result if available
    if (memo.containsKey(state)) return memo.get(state);

    // 3. Compute by combining subproblem results
    Result result = combine(solve(smallerState1), solve(smallerState2));

    // 4. Cache before returning
    memo.put(state, result);
    return result;
}
```

### Example: Fibonacci

```java
Map<Integer, Long> memo = new HashMap<>();

long fib(int n) {
    if (n <= 1) return n;                          // base case
    if (memo.containsKey(n)) return memo.get(n);   // cache hit

    long result = fib(n - 1) + fib(n - 2);        // recurse
    memo.put(n, result);                           // cache
    return result;
}
```

```
Call tree with memoization for fib(5):

fib(5)
├── fib(4)
│   ├── fib(3)
│   │   ├── fib(2)
│   │   │   ├── fib(1) → 1
│   │   │   └── fib(0) → 0
│   │   │   returns 1, cached memo[2]=1
│   │   └── fib(1) → 1
│   │   returns 2, cached memo[3]=2
│   └── fib(2) → memo hit! returns 1 instantly
│   returns 3, cached memo[4]=3
└── fib(3) → memo hit! returns 2 instantly
returns 5

Total unique calls: 6 (not 15 like naive recursion)
```

### When Memoization Shines

- The problem has a natural recursive structure
- Not all subproblems need to be solved (sparse subproblem space)
- You want to write the solution top-down and let the cache handle efficiency
- The state is complex (e.g., multiple parameters, strings as keys)

### Pitfalls

- **Stack overflow** for very deep recursion (n = 100,000 will blow the call stack)
- **HashMap overhead** — slightly slower than array-based tabulation
- **State key design** — if your state has multiple dimensions, you need a composite key

---

## Type 2: Tabulation (Bottom-Up)

### What It Is

Build the solution iteratively from the smallest subproblems up. Fill a table (array) in order so that when you need a value, it's already computed. No recursion, no call stack.

### The Template

```java
// 1. Define what dp[i] means
// 2. Set base cases
// 3. Fill the table using a transition formula
// 4. Return dp[target]

int[] dp = new int[n + 1];

// Base cases
dp[0] = baseValue;

// Fill
for (int i = 1; i <= n; i++) {
    dp[i] = transition(dp[i-1], dp[i-2], ...);
}

return dp[n];
```

### Example: Fibonacci

```java
long fib(int n) {
    if (n <= 1) return n;

    long[] dp = new long[n + 1];
    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];   // transition
    }

    return dp[n];
}
```

```
Table fill for fib(7):

Index:  0   1   2   3   4   5   6   7
dp[]:   0   1   1   2   3   5   8   13
            ↑   ↑
        base cases

Each cell = sum of the two before it.
No recursion. No stack. Just a loop.
```

### When Tabulation Shines

- You need to solve ALL subproblems anyway (dense subproblem space)
- The recursion depth would be too large (stack overflow risk)
- You want maximum performance (array access is faster than HashMap)
- You need space optimization (can often reduce to O(1) by keeping only last few values)

### Space Optimization

When `dp[i]` only depends on the last 1 or 2 values, you can drop the array entirely:

```java
// Fibonacci: only needs previous two values
long fib(int n) {
    if (n <= 1) return n;
    long prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        long curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// Space: O(n) → O(1)
```

---

## Side-by-Side Comparison

| | Memoization (Top-Down) | Tabulation (Bottom-Up) |
|---|---|---|
| Direction | Problem → subproblems | Base cases → problem |
| Implementation | Recursion + cache | Loops + array |
| Stack overflow risk | Yes (deep recursion) | No |
| Solves only needed subproblems | Yes | No (fills entire table) |
| Space optimization | Harder | Easy (rolling array) |
| Code complexity | Usually simpler to write | Requires careful state design |
| Performance | Slightly slower (HashMap/call overhead) | Slightly faster (array access) |
| Best for | Sparse subproblems, complex state | Dense subproblems, large n |

---

## DP Problem Categories with Examples

### Category 1: Linear DP (1D state)

State depends on a single index. The most common pattern.

```
dp[i] = answer for the first i elements / reaching index i

Examples:
- Fibonacci:          dp[i] = dp[i-1] + dp[i-2]
- Climbing Stairs:    dp[i] = dp[i-1] + dp[i-2]
- House Robber:       dp[i] = max(dp[i-1], dp[i-2] + nums[i])
- Coin Change:        dp[i] = min(dp[i-coin] + 1) for each coin
```

```java
// House Robber — classic linear DP
// dp[i] = max money robbing houses 0..i
int rob(int[] nums) {
    int n = nums.length;
    if (n == 1) return nums[0];
    int[] dp = new int[n];
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0], nums[1]);
    for (int i = 2; i < n; i++)
        dp[i] = Math.max(dp[i-1], dp[i-2] + nums[i]);
    return dp[n-1];
}
```

---

### Category 2: Grid DP (2D state)

State depends on a position in a 2D grid. Common in path-counting and min-cost problems.

```
dp[r][c] = answer to reach cell (r, c)

Examples:
- Unique Paths:       dp[r][c] = dp[r-1][c] + dp[r][c-1]
- Min Path Sum:       dp[r][c] = grid[r][c] + min(dp[r-1][c], dp[r][c-1])
- Dungeon Game:       dp[r][c] = min health needed to reach exit from (r,c)
```

```java
// Unique Paths — grid DP
// dp[r][c] = number of ways to reach (r,c) from (0,0)
int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    for (int r = 0; r < m; r++) dp[r][0] = 1;  // left column
    for (int c = 0; c < n; c++) dp[0][c] = 1;  // top row
    for (int r = 1; r < m; r++)
        for (int c = 1; c < n; c++)
            dp[r][c] = dp[r-1][c] + dp[r][c-1];
    return dp[m-1][n-1];
}
```

---

### Category 3: Sequence DP (two sequences)

State depends on indices into two strings or arrays. Used for comparison and alignment problems.

```
dp[i][j] = answer for first i chars of s1 and first j chars of s2

Examples:
- Longest Common Subsequence: dp[i][j] = 1 + dp[i-1][j-1]  if s1[i]==s2[j]
                                        = max(dp[i-1][j], dp[i][j-1])  otherwise
- Edit Distance:              dp[i][j] = dp[i-1][j-1]  if s1[i]==s2[j]
                                        = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
```

```java
// Edit Distance — sequence DP
// dp[i][j] = min operations to convert s1[0..i-1] to s2[0..j-1]
int editDistance(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m+1][n+1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;  // delete all of s1
    for (int j = 0; j <= n; j++) dp[0][j] = j;  // insert all of s2
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1))
                dp[i][j] = dp[i-1][j-1];         // no operation needed
            else
                dp[i][j] = 1 + Math.min(dp[i-1][j-1],   // replace
                               Math.min(dp[i-1][j],       // delete
                                        dp[i][j-1]));     // insert
        }
    }
    return dp[m][n];
}
```

---

### Category 4: Knapsack DP (subset selection)

Choose items from a set, each usable a limited number of times, to maximize/minimize a value subject to a constraint.

```
dp[i][w] = best value using first i items with capacity w

0/1 Knapsack:    each item used at most once → traverse weight RIGHT TO LEFT
Unbounded:       each item used any times    → traverse weight LEFT TO RIGHT

Examples:
- 0/1 Knapsack:           can I fill a bag of weight W with max value?
- Partition Equal Subset:  can I split array into two equal-sum halves?
- Coin Change:             min coins to make amount (unbounded — coins reusable)
- Coin Change II:          count combinations (unbounded)
```

```java
// 0/1 Knapsack
// dp[w] = max value achievable with capacity w
int knapsack(int[] weights, int[] values, int capacity) {
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < weights.length; i++) {
        // RIGHT TO LEFT — each item used at most once
        for (int w = capacity; w >= weights[i]; w--) {
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}

// Unbounded Knapsack (e.g. Coin Change count)
// dp[w] = number of ways to make amount w
int countWays(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;
    for (int coin : coins) {
        // LEFT TO RIGHT — each coin reusable
        for (int w = coin; w <= amount; w++) {
            dp[w] += dp[w - coin];
        }
    }
    return dp[amount];
}
```

---

### Category 5: Interval DP

State is a range `[i, j]`. You split the range at some point `k` and combine results. Common in problems about splitting, merging, or evaluating expressions.

```
dp[i][j] = answer for the subproblem on range [i..j]

Examples:
- Matrix Chain Multiplication: min cost to multiply matrices i through j
- Burst Balloons:              max coins from bursting balloons i through j
- Palindrome Partitioning:     min cuts to partition s[i..j] into palindromes
```

```java
// Palindrome Partitioning II — interval DP
// dp[i] = min cuts for s[0..i]
int minCut(String s) {
    int n = s.length();
    // isPalin[i][j] = true if s[i..j] is a palindrome
    boolean[][] isPalin = new boolean[n][n];
    for (int len = 1; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            isPalin[i][j] = s.charAt(i) == s.charAt(j)
                         && (len <= 2 || isPalin[i+1][j-1]);
        }
    }
    int[] dp = new int[n];
    for (int i = 0; i < n; i++) {
        if (isPalin[0][i]) { dp[i] = 0; continue; } // whole prefix is palindrome
        dp[i] = i; // worst case: cut every character
        for (int j = 1; j <= i; j++) {
            if (isPalin[j][i])
                dp[i] = Math.min(dp[i], dp[j-1] + 1);
        }
    }
    return dp[n-1];
}
```

---

### Category 6: State Machine DP

The problem has distinct modes or states you transition between. Common in stock trading problems.

```
State: which "mode" are you currently in?
Transition: what happens when you take an action from each mode?

Examples:
- Stock Buy/Sell:         states = {holding, not_holding}
- Stock with Cooldown:    states = {holding, sold, cooldown}
- Stock with K transactions: states = {transactions_used}
```

```java
// Best Time to Buy and Sell Stock (one transaction)
// Two states: holding a stock, or not holding
int maxProfit(int[] prices) {
    int hold = Integer.MIN_VALUE; // max profit while holding
    int free = 0;                 // max profit while not holding

    for (int price : prices) {
        int prevFree = free;
        free = Math.max(free, hold + price);   // sell today or do nothing
        hold = Math.max(hold, prevFree - price); // buy today or do nothing
    }
    return free;
}
```

---

## Decision Framework: Which DP Type to Use?

```
Is the state a single index?
  └─ Yes → Linear DP (1D array)

Is the state a position in a 2D grid?
  └─ Yes → Grid DP (2D array)

Does the state involve two strings/sequences?
  └─ Yes → Sequence DP (2D array, compare char by char)

Are you selecting items from a set with a capacity constraint?
  └─ Yes → Knapsack DP
       └─ Each item used once?  → 0/1 Knapsack (right-to-left inner loop)
       └─ Items reusable?       → Unbounded Knapsack (left-to-right inner loop)

Is the state a range [i, j] that you split at some point k?
  └─ Yes → Interval DP (2D array, fill by increasing length)

Does the problem have distinct modes you switch between?
  └─ Yes → State Machine DP (one variable per state)
```

---

## Memoization vs Tabulation: Which to Choose?

```
Start with memoization when:
  ✓ The recursive structure is obvious
  ✓ Not all subproblems need to be solved
  ✓ The state is complex (multiple params, string keys)
  ✓ You want to write it quickly and correctly first

Switch to tabulation when:
  ✓ n is very large (stack overflow risk)
  ✓ You need maximum performance
  ✓ You want to optimize space (rolling array)
  ✓ All subproblems will be needed anyway
```

A common workflow: **write memoization first to verify correctness, then convert to tabulation for performance**.

Converting memoization → tabulation:
1. Identify the state variables → they become array indices
2. Find the base cases → fill those array cells first
3. Find the recursion order → determine loop direction
4. Replace recursive calls with array lookups

---

## Quick Reference

| Category | State | Transition shape | Classic problem |
|---|---|---|---|
| Linear | `dp[i]` | `dp[i-1]`, `dp[i-2]` | House Robber, Coin Change |
| Grid | `dp[r][c]` | `dp[r-1][c]`, `dp[r][c-1]` | Unique Paths, Min Path Sum |
| Sequence | `dp[i][j]` | `dp[i-1][j-1]`, `dp[i-1][j]`, `dp[i][j-1]` | LCS, Edit Distance |
| Knapsack | `dp[w]` | `dp[w - weight]` | 0/1 Knapsack, Partition Sum |
| Interval | `dp[i][j]` | `dp[i][k] + dp[k+1][j]` | Burst Balloons, Matrix Chain |
| State Machine | `hold`, `free`, etc. | transitions between states | Stock problems |

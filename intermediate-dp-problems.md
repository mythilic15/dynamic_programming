# Intermediate DP Problems

Three problems that each introduce a new DP pattern not covered in the existing curriculum. Each is solved with both **memoization (top-down)** and **tabulation (bottom-up)**.

---

## Problem 1: Partition Equal Subset Sum

### Problem Statement

Given an array of positive integers, determine whether it can be partitioned into **two subsets with equal sum**.

```
Input:  [1, 5, 11, 5]
Output: true   → [1, 5, 5] and [11]

Input:  [1, 2, 3, 5]
Output: false  → no equal partition exists
```

---

### How to Think About It

If the total sum is odd, it's immediately impossible — you can't split an odd number into two equal halves.

If the total sum is even, the question becomes: **can we find a subset that sums to `total / 2`?** This is the classic **0/1 Knapsack** pattern — for each element you decide to either include it or exclude it.

This is different from Coin Change because each element can only be used **once**.

---

### Identifying the DP State

At each index `i` with a remaining target `t`:
- **Skip** element `i` → solve `(i+1, t)`
- **Take** element `i` → solve `(i+1, t - nums[i])`

State: `dp(i, t)` = can we reach sum `t` using elements from index `i` onwards?

---

### Step-by-Step Solving

Use input `[1, 5, 11, 5]`.

**Step 1 — Compute total sum**
```
1 + 5 + 11 + 5 = 22
```
Total is even, so continue.

**Step 2 — Reduce the problem**
```
target = 22 / 2 = 11
```
Now the question is: can any subset of `[1, 5, 11, 5]` sum to `11`?

**Step 3 — Define the recurrence**

At each index `i` with remaining `target`:
- Take `nums[i]` → recurse with `(i+1, target - nums[i])`
- Skip `nums[i]` → recurse with `(i+1, target)`

Base cases:
- `target == 0` → found it, return `true`
- `i >= length` or `target < 0` → dead end, return `false`

**Step 4 — Trace the key decisions**

```
canReach(0, 11)  → nums[0]=1
  ├── TAKE 1 → canReach(1, 10)  → nums[1]=5
  │     ├── TAKE 5 → canReach(2, 5)  → nums[2]=11
  │     │     ├── TAKE 11 → canReach(3, -6)  → target<0 ✗
  │     │     └── SKIP 11 → canReach(3, 5)   → nums[3]=5
  │     │           ├── TAKE 5 → canReach(4, 0)  → target==0 ✓  ← FOUND!
  │     │           └── (short-circuits, already true)
  │     └── (short-circuits, already true)
  └── (short-circuits, already true)
```

The winning subset is: `1 (index 0) + 5 (index 1) + 5 (index 3) = 11` ✓

**Step 5 — Memo saves repeated work**

Any `(i, target)` pair computed once is cached. If both TAKE and SKIP branches eventually reach the same `(i, target)`, the second call hits the memo instantly instead of re-exploring.

---

### Call Trace (Full Tree for `[1, 5, 11, 5]`, target=11)

Each node is `canReach(i, target)`. `✓` = true, `✗` = false, short-circuit means OR already resolved.

```
canReach(0, 11)
├── TAKE nums[0]=1 → canReach(1, 10)
│     ├── TAKE nums[1]=5 → canReach(2, 5)
│     │     ├── TAKE nums[2]=11 → canReach(3, -6)  → target<0  ✗
│     │     └── SKIP nums[2]=11 → canReach(3, 5)
│     │           ├── TAKE nums[3]=5 → canReach(4, 0)  → target==0  ✓
│     │           └── short-circuit (OR already true)
│     │         canReach(3, 5) = ✓  → stored in memo["3,5"]
│     │     canReach(2, 5) = ✓  → stored in memo["2,5"]
│     └── short-circuit (OR already true)
│   canReach(1, 10) = ✓  → stored in memo["1,10"]
└── short-circuit (OR already true)
canReach(0, 11) = ✓
```

**What memo looks like after the trace:**
```
"3,5"  → true
"2,5"  → true
"1,10" → true
"0,11" → true
```

Note: `canReach(4, 0)` hits the base case `target == 0` and returns immediately — it never gets stored in memo because we return before the memo check.

**Why the SKIP branch at index 0 never runs:**

The `||` operator short-circuits. Once `canReach(1, 10)` returns `true`, Java doesn't evaluate the SKIP branch `canReach(1, 11)` at all. This is a key efficiency win on top of memoization.

---

### Memoization (Top-Down)

```java
import java.util.HashMap;
import java.util.Map;

public class PartitionEqualSubset {

    // Key = "index,target"
    private Map<String, Boolean> memo = new HashMap<>();

    public boolean canPartition(int[] nums) {
        int total = 0;
        for (int n : nums) total += n;

        // Odd total → impossible
        if (total % 2 != 0) return false;

        return canReach(nums, 0, total / 2);
    }

    private boolean canReach(int[] nums, int i, int target) {
        // Found a valid subset
        if (target == 0) return true;
        // Ran out of elements or overshot
        if (i >= nums.length || target < 0) return false;

        String key = i + "," + target;
        if (memo.containsKey(key)) return memo.get(key);

        // Take nums[i] OR skip nums[i]
        boolean result = canReach(nums, i + 1, target - nums[i])
                      || canReach(nums, i + 1, target);

        memo.put(key, result);
        return result;
    }

    public static void main(String[] args) {
        PartitionEqualSubset sol = new PartitionEqualSubset();
        System.out.println(sol.canPartition(new int[]{1, 5, 11, 5})); // true
        System.out.println(sol.canPartition(new int[]{1, 2, 3, 5}));  // false
        System.out.println(sol.canPartition(new int[]{2, 2, 1, 1}));  // true
    }
}
```

**Complexity:** Time O(N × sum), Space O(N × sum)

---

### Tabulation (Bottom-Up)

`dp[j]` = can we form sum `j` using elements seen so far?

We iterate over each number and update the table **backwards** (right to left) to ensure each element is only used once.

```java
import java.util.Arrays;

public class PartitionEqualSubsetTab {

    public boolean canPartition(int[] nums) {
        int total = 0;
        for (int n : nums) total += n;
        if (total % 2 != 0) return false;

        int target = total / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true; // sum of 0 is always achievable (take nothing)

        for (int num : nums) {
            // Traverse RIGHT TO LEFT to avoid using the same element twice
            for (int j = target; j >= num; j--) {
                // dp[j] stays true if it was already reachable,
                // OR becomes true if (j - num) was reachable before adding this element
                dp[j] = dp[j] || dp[j - num];
            }
        }

        return dp[target];
    }

    public static void main(String[] args) {
        PartitionEqualSubsetTab sol = new PartitionEqualSubsetTab();
        System.out.println(sol.canPartition(new int[]{1, 5, 11, 5})); // true
        System.out.println(sol.canPartition(new int[]{1, 2, 3, 5}));  // false
    }
}
```

**Complexity:** Time O(N × sum), Space O(sum)

---

### Why Traverse Right to Left?

If you go left to right, you might use the same element twice:

```
nums = [3], target = 6

Left to right (WRONG):
  dp[0]=true → dp[3] = dp[3] || dp[0] = true
  dp[3]=true → dp[6] = dp[6] || dp[3] = true  ← used 3 twice!

Right to left (CORRECT):
  dp[6] = dp[6] || dp[3] = false  (dp[3] still false at this point)
  dp[3] = dp[3] || dp[0] = true
  → dp[6] stays false ✓
```

---

### Trace for `[1, 5, 11, 5]`, target = 11

```
Initial:  dp = [T, F, F, F, F, F, F, F, F, F, F, F]
                0  1  2  3  4  5  6  7  8  9 10 11

After 1:  dp = [T, T, F, F, F, F, F, F, F, F, F, F]
After 5:  dp = [T, T, F, F, F, T, T, F, F, F, F, F]
After 11: dp = [T, T, F, F, F, T, T, F, F, F, F, T]  ← dp[11] = true ✓
After 5:  dp = [T, T, F, F, F, T, T, F, F, F, T, T]

dp[11] = true → answer: true
```

---

## Problem 2: Longest Common Subsequence (LCS)

### Problem Statement

Given two strings, find the length of their **longest common subsequence** — a subsequence that appears in both strings in the same relative order (not necessarily contiguous).

```
Input:  s1 = "abcde", s2 = "ace"
Output: 3   → "ace"

Input:  s1 = "abc", s2 = "abc"
Output: 3   → "abc"

Input:  s1 = "abc", s2 = "def"
Output: 0   → no common subsequence
```

---

### How to Think About It

Compare the strings character by character from the end (or start). At each position `(i, j)`:

- If `s1[i] == s2[j]` → this character is part of the LCS. Take it and move both pointers: `1 + LCS(i-1, j-1)`
- If they differ → skip one character from either string and take the best: `max(LCS(i-1, j), LCS(i, j-1))`

This is a **2D DP** problem — the state depends on two indices.

---

### Memoization (Top-Down)

```java
import java.util.HashMap;
import java.util.Map;

public class LongestCommonSubsequence {

    private Map<String, Integer> memo = new HashMap<>();

    public int lcs(String s1, String s2) {
        return solve(s1, s2, s1.length() - 1, s2.length() - 1);
    }

    private int solve(String s1, String s2, int i, int j) {
        // Base case: one string exhausted
        if (i < 0 || j < 0) return 0;

        String key = i + "," + j;
        if (memo.containsKey(key)) return memo.get(key);

        int result;
        if (s1.charAt(i) == s2.charAt(j)) {
            // Characters match — include in LCS
            result = 1 + solve(s1, s2, i - 1, j - 1);
        } else {
            // Characters differ — try skipping from either string
            result = Math.max(
                solve(s1, s2, i - 1, j),
                solve(s1, s2, i, j - 1)
            );
        }

        memo.put(key, result);
        return result;
    }

    public static void main(String[] args) {
        LongestCommonSubsequence sol = new LongestCommonSubsequence();
        System.out.println(sol.lcs("abcde", "ace"));  // 3
        System.out.println(sol.lcs("abc", "abc"));    // 3
        System.out.println(sol.lcs("abc", "def"));    // 0
        System.out.println(sol.lcs("abcba", "abcbcba")); // 5
    }
}
```

**Complexity:** Time O(M × N), Space O(M × N)

---

### Tabulation (Bottom-Up)

Build a 2D table where `dp[i][j]` = LCS length of `s1[0..i-1]` and `s2[0..j-1]`.

```java
public class LongestCommonSubsequenceTab {

    public int lcs(String s1, String s2) {
        int m = s1.length(), n = s2.length();

        // dp[i][j] = LCS of first i chars of s1 and first j chars of s2
        int[][] dp = new int[m + 1][n + 1];

        // Base cases: dp[0][j] = 0 and dp[i][0] = 0 (empty string has LCS 0)
        // Already 0 by default in Java

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    // Characters match — extend the LCS
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    // Characters differ — best of skipping one from either string
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[m][n];
    }

    public static void main(String[] args) {
        LongestCommonSubsequenceTab sol = new LongestCommonSubsequenceTab();
        System.out.println(sol.lcs("abcde", "ace"));  // 3
        System.out.println(sol.lcs("abc", "def"));    // 0
    }
}
```

**Complexity:** Time O(M × N), Space O(M × N) — reducible to O(N) with rolling array

---

### DP Table Trace for `"abcde"` and `"ace"`

```
     ""  a  c  e
""  [ 0, 0, 0, 0 ]
a   [ 0, 1, 1, 1 ]
b   [ 0, 1, 1, 1 ]
c   [ 0, 1, 2, 2 ]
d   [ 0, 1, 2, 2 ]
e   [ 0, 1, 2, 3 ]  ← answer: dp[5][3] = 3
```

Reading the table:
- `dp[1][1]`: 'a'=='a' → 1 + dp[0][0] = 1
- `dp[3][2]`: 'c'=='c' → 1 + dp[2][1] = 2
- `dp[5][3]`: 'e'=='e' → 1 + dp[4][2] = 3

---

## Problem 3: Jump Game II — Minimum Jumps

### Problem Statement

Given an array where `nums[i]` is the **maximum** jump length from index `i`, find the **minimum number of jumps** to reach the last index. You are guaranteed you can always reach the end.

```
Input:  [2, 3, 1, 1, 4]
Output: 2   → jump from 0→1 (length 2), then 1→4 (length 3)

Input:  [2, 3, 0, 1, 4]
Output: 2   → jump from 0→1, then 1→4

Input:  [1, 1, 1, 1]
Output: 3   → must take every step
```

> Note: Day 1 covered "minimum jumps" with memoization where `nums[i]` was the **exact** jump length and the array could be unreachable. This problem uses **maximum** jump length and guarantees reachability — a different recurrence and a cleaner DP structure.

---

### How to Think About It

At each index `i`, you can jump to any index from `i+1` to `i + nums[i]`. You want the minimum total jumps to reach `n-1`.

State: `dp[i]` = minimum jumps to reach index `i`

For each index `i`, every position `j` that can reach `i` (i.e., `j + nums[j] >= i`) can contribute: `dp[i] = min(dp[j] + 1)` for all valid `j`.

---

### Memoization (Top-Down)

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

public class JumpGameII {

    private Map<Integer, Integer> memo = new HashMap<>();

    public int jump(int[] nums) {
        return minJumps(nums, 0);
    }

    // Returns min jumps from index i to last index
    private int minJumps(int[] nums, int i) {
        int n = nums.length;
        if (i >= n - 1) return 0; // already at or past the end

        if (memo.containsKey(i)) return memo.get(i);

        int maxReach = Math.min(i + nums[i], n - 1);
        int best = Integer.MAX_VALUE;

        // Try every possible jump length from this position
        for (int next = i + 1; next <= maxReach; next++) {
            int sub = minJumps(nums, next);
            if (sub != Integer.MAX_VALUE) {
                best = Math.min(best, 1 + sub);
            }
        }

        memo.put(i, best);
        return best;
    }

    public static void main(String[] args) {
        JumpGameII sol = new JumpGameII();
        System.out.println(sol.jump(new int[]{2, 3, 1, 1, 4})); // 2
        System.out.println(sol.jump(new int[]{2, 3, 0, 1, 4})); // 2
        System.out.println(sol.jump(new int[]{1, 1, 1, 1}));    // 3
    }
}
```

**Complexity:** Time O(N²), Space O(N)

---

### Tabulation (Bottom-Up)

Build `dp[i]` = minimum jumps to reach index `i`, filling left to right.

```java
import java.util.Arrays;

public class JumpGameIITab {

    public int jump(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0; // 0 jumps to reach the start

        for (int i = 0; i < n - 1; i++) {
            if (dp[i] == Integer.MAX_VALUE) continue; // can't reach i, skip

            // From index i, update all reachable positions
            int maxReach = Math.min(i + nums[i], n - 1);
            for (int j = i + 1; j <= maxReach; j++) {
                dp[j] = Math.min(dp[j], dp[i] + 1);
            }
        }

        return dp[n - 1];
    }

    public static void main(String[] args) {
        JumpGameIITab sol = new JumpGameIITab();
        System.out.println(sol.jump(new int[]{2, 3, 1, 1, 4})); // 2
        System.out.println(sol.jump(new int[]{2, 3, 0, 1, 4})); // 2
        System.out.println(sol.jump(new int[]{1, 1, 1, 1}));    // 3
        System.out.println(sol.jump(new int[]{5, 1, 1, 1, 1})); // 1
    }
}
```

**Complexity:** Time O(N²), Space O(N)

---

### DP Table Trace for `[2, 3, 1, 1, 4]`

```
Index:   0   1   2   3   4
nums:   [2,  3,  1,  1,  4]
dp:     [0, INF, INF, INF, INF]

Process i=0 (dp[0]=0, can jump up to index 2):
  dp[1] = min(INF, 0+1) = 1
  dp[2] = min(INF, 0+1) = 1
  dp: [0, 1, 1, INF, INF]

Process i=1 (dp[1]=1, can jump up to index 4):
  dp[2] = min(1, 1+1) = 1  (no improvement)
  dp[3] = min(INF, 1+1) = 2
  dp[4] = min(INF, 1+1) = 2
  dp: [0, 1, 1, 2, 2]

Process i=2 (dp[2]=1, can jump up to index 3):
  dp[3] = min(2, 1+1) = 2  (no improvement)

Process i=3 (dp[3]=2, can jump up to index 4):
  dp[4] = min(2, 2+1) = 2  (no improvement)

Answer: dp[4] = 2
```

---

## Summary

| Problem | Pattern | State dimensions | Key decision |
|---|---|---|---|
| Partition Equal Subset Sum | 0/1 Knapsack | 1D (target sum) | Take or skip each element |
| Longest Common Subsequence | 2D string DP | 2D (two indices) | Match or skip from either string |
| Jump Game II | Forward reachability DP | 1D (index) | Which position to jump to next |

### When to use each pattern

- **0/1 Knapsack** — you have items, each usable once, and a capacity/target to hit exactly
- **2D string DP** — comparing or aligning two sequences character by character
- **Reachability DP** — you're moving through positions and want min/max cost to reach the end

# Day 1: Foundations — Recursion & Memoization

## Learning Objectives
By the end of Day 1, you should be able to:
- Explain what Dynamic Programming is and when to use it
- Identify overlapping subproblems and optimal substructure
- Implement top-down DP using memoization
- Trace through a recursion tree and understand redundant calls

---

## 1. What is Dynamic Programming?

Dynamic Programming (DP) is an optimization technique for solving problems by breaking them into smaller subproblems and storing their results to avoid redundant computation.

There are two main approaches to implement DP:

### Memoization (Top-Down)
Start with recursion and cache results as you go. You solve the problem naturally from the top, but store each subproblem's answer in a map so you never compute it twice. Feels like recursion, just with a memory.

### Tabulation (Bottom-Up)
Build the solution iteratively from the smallest subproblems up. You fill a table (usually an array) in order, so by the time you need a result it's already computed. No recursion, no call stack — just loops.

Both approaches give the same time complexity. Memoization is usually easier to write; tabulation is often more space-efficient.

---

Two key properties must hold for DP to apply:

### Overlapping Subproblems
The same subproblems are solved multiple times during recursion.

Example: Computing `fib(5)` requires `fib(3)` twice — once via `fib(4)` and once directly.

### Optimal Substructure
The optimal solution to a problem can be built from optimal solutions of its subproblems.

Example: The shortest path from A to C through B = shortest(A→B) + shortest(B→C).

---

## 2. Recursion — The Starting Point

Recursion is a technique where a function calls itself to solve a smaller version of the same problem. Every recursive solution needs two things:

1. **Base case** — the simplest input where the answer is known directly (stops the recursion)
2. **Recursive case** — break the problem into a smaller subproblem and call itself

### How the Call Stack Works

When a function calls itself, each call is pushed onto the call stack. When a base case is hit, calls start returning and popping off the stack.

```
factorial(4)
  └── 4 * factorial(3)
            └── 3 * factorial(2)
                      └── 2 * factorial(1)
                                └── returns 1  ← base case
                      returns 2 * 1 = 2
            returns 3 * 2 = 6
  returns 4 * 6 = 24
```

### Anatomy of a Recursive Function

```java
public static int factorial(int n) {
    // 1. Base case — stop recursion here
    if (n <= 1) return 1;

    // 2. Recursive case — solve smaller subproblem
    return n * factorial(n - 1);
}
```

### Common Recursion Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| Linear | One recursive call per step | Factorial, sum of array |
| Binary | Two recursive calls per step | Fibonacci, binary search |
| Tree | Multiple recursive calls | Tree traversal |
| Backtracking | Recurse and undo | Permutations, maze solving |

### Example: Sum of Array

```java
public static int arraySum(int[] nums, int i) {
    if (i == nums.length) return 0;          // base case: past the end
    return nums[i] + arraySum(nums, i + 1);  // add current + rest
}
```

### Example: Power Function

```java
public static long power(int base, int exp) {
    if (exp == 0) return 1;                        // base case: anything^0 = 1
    return base * power(base, exp - 1);            // base * base^(exp-1)
}
```

### Example: Fibonacci (Naive Recursion)

```java
public static int fib(int n) {
    if (n <= 1) return n;           // base cases: fib(0)=0, fib(1)=1
    return fib(n - 1) + fib(n - 2); // binary recursion
}
```

### Recursion Tree for fib(5)
```
                fib(5)
              /        \
          fib(4)       fib(3)
          /    \       /    \
       fib(3) fib(2) fib(2) fib(1)
       /   \
    fib(2) fib(1)
```

Notice `fib(3)` and `fib(2)` are computed multiple times — this is the problem DP solves.

Time Complexity: O(2^n)
Space Complexity: O(n) — call stack depth

### Thinking Recursively — 3 Questions to Ask
1. **What is the base case?** (smallest input with a known answer)
2. **What does one recursive call return?** (trust it returns the right answer)
3. **How do I combine the result?** (build the full answer from subproblem answers)

---

## 3. Memoization — Top-Down DP

Memoization caches the result of each subproblem the first time it's solved, and returns the cached result on subsequent calls.

### Example: Fibonacci with Memoization

```java
import java.util.HashMap;
import java.util.Map;

public class Fibonacci {

    private static Map<Integer, Integer> memo = new HashMap<>();

    public static int fib(int n) {
        if (n <= 1) return n;

        // Return cached result if available
        if (memo.containsKey(n)) return memo.get(n);

        // Compute and cache
        int result = fib(n - 1) + fib(n - 2);
        memo.put(n, result);

        return result;
    }

    public static void main(String[] args) {
        System.out.println(fib(10)); // 55
        System.out.println(fib(50)); // 12586269025
    }
}
```

Time Complexity: O(n) — each subproblem solved once
Space Complexity: O(n) — memo table + call stack

---

## 4. Example: Factorial with Memoization

```java
import java.util.HashMap;
import java.util.Map;

public class Factorial {

    private static Map<Integer, Long> memo = new HashMap<>();

    public static long factorial(int n) {
        if (n <= 1) return 1;
        if (memo.containsKey(n)) return memo.get(n);

        long result = n * factorial(n - 1);
        memo.put(n, result);
        return result;
    }

    public static void main(String[] args) {
        System.out.println(factorial(10)); // 3628800
        System.out.println(factorial(15)); // 1307674368000
    }
}
```

---

## 5. Memoization Template

Use this template for any top-down DP problem:

```java
Map<StateType, ResultType> memo = new HashMap<>();

ResultType solve(StateType state) {
    // 1. Base case
    if (isBaseCase(state)) return baseResult;

    // 2. Check cache
    if (memo.containsKey(state)) return memo.get(state);

    // 3. Recurse and compute
    ResultType result = /* combine subproblem results */;

    // 4. Cache and return
    memo.put(state, result);
    return result;
}
```

---

## 6. Practice Problems

### Problem 1: Fibonacci Number (LeetCode 509)
Return the nth Fibonacci number.
- Input: n = 6
- Output: 8

```java
import java.util.HashMap;
import java.util.Map;

public class FibonacciPractice {

    private static Map<Integer, Integer> memo = new HashMap<>();

    public static int fib(int n) {
        if (n <= 1) return n;
        if (memo.containsKey(n)) return memo.get(n);
        int result = fib(n - 1) + fib(n - 2);
        memo.put(n, result);
        return result;
    }

    public static void main(String[] args) {
        System.out.println(fib(6));  // 8
        System.out.println(fib(10)); // 55
    }
}
```

---

### Problem 2: Climbing Stairs (LeetCode 70)
You can climb 1 or 2 steps. How many ways to reach step n?
- Input: n = 4
- Output: 5

**How to think about it:**

At any step `i`, you could have arrived from either `i-1` (took 1 step) or `i-2` (took 2 steps). So the number of ways to reach step `i` is just the sum of ways to reach the two steps before it — exactly like Fibonacci.

Define `f(n)` = number of ways to reach step `n`. Then:
- `f(0) = 1` — one way to be at the start (do nothing)
- `f(1) = 1` — only one way to reach step 1 (take 1 step)
- `f(n) = f(n-1) + f(n-2)` — came from step n-1 or step n-2

**Step-by-step example for `n = 4` (4 stairs):**

Imagine you're standing at the bottom. You want to count every distinct sequence of 1-step and 2-step moves that gets you to stair 4.

```
Ways to reach stair 1:  [1]                          → 1 way
Ways to reach stair 2:  [1,1]  [2]                   → 2 ways
Ways to reach stair 3:  [1,1,1]  [1,2]  [2,1]        → 3 ways
Ways to reach stair 4:  [1,1,1,1]  [1,1,2]  [1,2,1]
                        [2,1,1]    [2,2]              → 5 ways
```

Notice the pattern — ways(4) = ways(3) + ways(2) = 3 + 2 = 5.
That's because every path to stair 4 either:
- came from stair 3 (took 1 step), or
- came from stair 2 (took 2 steps)

**Recursive call trace:**
```
climbStairs(4)
  ├── climbStairs(3)
  │     ├── climbStairs(2)
  │     │     ├── climbStairs(1) → 1  (base case)
  │     │     └── climbStairs(0) → 1  (base case)
  │     │     returns 1 + 1 = 2  ← cached as memo[2]
  │     └── climbStairs(1) → 1  (base case)
  │     returns 2 + 1 = 3  ← cached as memo[3]
  └── climbStairs(2) → 2  (returned from cache, no recompute!)
  returns 3 + 2 = 5  ✓
```

Without memoization this has O(2^n) time due to repeated calls. With memoization each `f(i)` is computed once → O(n).

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ClimbingStairs {

    private static Map<Integer, Integer> memo = new HashMap<>();

    // Count the number of ways
    public static int climbStairs(int n) {
        if (n <= 1) return 1;
        if (memo.containsKey(n)) return memo.get(n);
        int result = climbStairs(n - 1) + climbStairs(n - 2);
        memo.put(n, result);
        return result;
    }

    // Collect all combinations
    public static void printCombinations(int n, List<Integer> current) {
        if (n == 0) {
            System.out.println(current);
            return;
        }
        // try taking 1 step
        if (n >= 1) {
            current.add(1);
            printCombinations(n - 1, current);
            current.remove(current.size() - 1);
        }
        // try taking 2 steps
        if (n >= 2) {
            current.add(2);
            printCombinations(n - 2, current);
            current.remove(current.size() - 1);
        }
    }

    public static void main(String[] args) {
        int n = 4;
        System.out.println("Total ways to climb " + n + " stairs: " + climbStairs(n));
        System.out.println("All combinations:");
        printCombinations(n, new ArrayList<>());
    }
}
```

Output for `n = 4`:
```
Total ways to climb 4 stairs: 5
All combinations:
[1, 1, 1, 1]
[1, 1, 2]
[1, 2, 1]
[2, 1, 1]
[2, 2]
```

`printCombinations` uses backtracking — at each step it tries taking 1 or 2 stairs, adds the choice to the current path, recurses on the remaining stairs, then removes the choice (backtracks) before trying the next option. No memoization here since we need every unique path, not just the count.

---

## 7. Array Exercises

### Exercise 1: Count Array Sum Paths
Given an array of positive integers, count the number of ways to reach the last index starting from index 0, where at each step you can jump 1 or 2 positions forward.

```
Input:  nums = [1, 2, 3, 4, 5]  (5 elements, indices 0..4 → 4 steps)
Output: 8
```

**Solution:**

```java
import java.util.HashMap;
import java.util.Map;

public class CountArrayPaths {

    private static Map<Integer, Integer> memo = new HashMap<>();

    // f(i) = number of ways to reach index i
    public static int countPaths(int[] nums, int i) {
        int last = nums.length - 1;
        if (i == last) return 1;   // reached the end — one valid path
        if (i > last) return 0;    // overshot — not valid
        if (memo.containsKey(i)) return memo.get(i);

        int result = countPaths(nums, i + 1) + countPaths(nums, i + 2);
        memo.put(i, result);
        return result;
    }

    public static void main(String[] args) {
        memo.clear();
        System.out.println(countPaths(new int[]{1, 2, 3, 4, 5}, 0)); // 8
        memo.clear();
        System.out.println(countPaths(new int[]{1, 2, 3}, 0));        // 3
    }
}
```

---

### Exercise 2: Maximum Sum of Non-Adjacent Elements
Given an array of integers, find the maximum sum such that no two chosen elements are adjacent.

```
Input:  [3, 2, 5, 10, 7]
Output: 15  (3 + 5 + 7 — indices 0, 2, 4, none adjacent)

Input:  [5, 1, 1, 5]
Output: 10  (5 + 5 — indices 0 and 3)
```

**Solution:**

```java
import java.util.HashMap;
import java.util.Map;

public class MaxNonAdjacentSum {

    private static Map<Integer, Integer> memo = new HashMap<>();

    // f(i) = max sum we can get from index i to end
    public static int maxSum(int[] nums, int i) {
        if (i >= nums.length) return 0;
        if (memo.containsKey(i)) return memo.get(i);

        int skip = maxSum(nums, i + 1);              // don't take nums[i]
        int take = nums[i] + maxSum(nums, i + 2);    // take nums[i], skip adjacent

        int result = Math.max(skip, take);
        memo.put(i, result);
        return result;
    }

    public static void main(String[] args) {
        memo.clear();
        System.out.println(maxSum(new int[]{3, 2, 5, 10, 7}, 0)); // 15
        memo.clear();
        System.out.println(maxSum(new int[]{5, 1, 1, 5}, 0));     // 10
        memo.clear();
        System.out.println(maxSum(new int[]{2, 1, 1, 2}, 0));     // 4
    }
}
```

---

### Exercise 3: Minimum Steps to Reach End
Given an array where `nums[i]` is the maximum jump length from index `i`, find the minimum number of jumps to reach the last index. Return -1 if not reachable.

```
Input:  [2, 3, 1, 1, 4]
Output: 2  (0 → 1 → 4)

Input:  [3, 2, 1, 0, 4]
Output: -1  (stuck at index 3)
```

**Solution:**

```java
import java.util.HashMap;
import java.util.Map;

public class MinJumps {

    private static Map<Integer, Integer> memo = new HashMap<>();

    // f(i) = min jumps needed from index i to reach the last index
    public static int minJumps(int[] nums, int i) {
        int n = nums.length;
        if (i >= n - 1) return 0;          // already at or past the end
        if (nums[i] == 0) return Integer.MAX_VALUE; // stuck — can't move

        if (memo.containsKey(i)) return memo.get(i);

        int minSteps = Integer.MAX_VALUE;
        for (int jump = 1; jump <= nums[i]; jump++) {
            int sub = minJumps(nums, i + jump);
            if (sub != Integer.MAX_VALUE) {
                minSteps = Math.min(minSteps, 1 + sub);
            }
        }

        memo.put(i, minSteps);
        return minSteps;
    }

    public static void main(String[] args) {
        memo.clear();
        int r = minJumps(new int[]{2, 3, 1, 1, 4}, 0);
        System.out.println(r == Integer.MAX_VALUE ? -1 : r); // 2

        memo.clear();
        r = minJumps(new int[]{3, 2, 1, 0, 4}, 0);
        System.out.println(r == Integer.MAX_VALUE ? -1 : r); // -1
    }
}
```

---

### Exercise 4: Count Subarrays with Target Sum
Given an array of non-negative integers and a target `T`, count how many contiguous subarrays sum to exactly `T`.

```
Input:  nums = [1, 2, 3, 1, 2], T = 3
Output: 3  ([1,2], [3], [1,2])
```

**Solution:**

```java
public class CountSubarraysWithSum {

    // No memoization needed — straightforward prefix sum approach
    public static int countSubarrays(int[] nums, int target) {
        int count = 0;

        for (int start = 0; start < nums.length; start++) {
            int sum = 0;
            for (int end = start; end < nums.length; end++) {
                sum += nums[end];
                if (sum == target) count++;
                if (sum > target) break; // no point continuing (non-negative array)
            }
        }

        return count;
    }

    public static void main(String[] args) {
        System.out.println(countSubarrays(new int[]{1, 2, 3, 1, 2}, 3)); // 3
        System.out.println(countSubarrays(new int[]{1, 1, 1}, 2));        // 2
        System.out.println(countSubarrays(new int[]{5, 2, 1}, 5));        // 1
    }
}
```

---

## 7. Key Takeaways

- DP = identify repeated subproblems + store results
- Memoization is the easiest way to convert recursion into DP
- Always define your **state** clearly — what does `f(n)` represent?
- Draw the recursion tree to spot overlapping subproblems

---

## Day 1 Checklist
- [ ] Can explain overlapping subproblems with an example
- [ ] Can implement Fibonacci with memoization
- [ ] Understand time complexity improvement: O(2^n) → O(n)
- [ ] Comfortable using a HashMap as a memo cache

---

## 8. Dry Run: Minimum Steps to Reach End

**Input:** `nums = [2, 3, 1, 1, 4]`

```
index:  0  1  2  3  4
value: [2, 3, 1, 1, 4]
```

We call `minJumps(nums, 0)` and want the minimum jumps to reach index 4.

---

### Call Tree

```
minJumps(0)  — can jump 1 or 2 steps
├── jump 1 → minJumps(1)  — can jump 1, 2, or 3 steps
│   ├── jump 1 → minJumps(2)  — can jump 1 step
│   │   └── jump 1 → minJumps(3)  — can jump 1 step
│   │       └── jump 1 → minJumps(4) → 0  (at end ✓)
│   │       minJumps(3) = 1 + 0 = 1  ← cached
│   │   minJumps(2) = 1 + 1 = 2  ← cached
│   ├── jump 2 → minJumps(3) → 1  (from cache, no recompute)
│   └── jump 3 → minJumps(4) → 0  (at end ✓)
│   minJumps(1) = min(1+2, 1+1, 1+0) = 1  ← cached
└── jump 2 → minJumps(2) → 2  (from cache, no recompute)
minJumps(0) = min(1+1, 1+2) = 2  ✓
```

---

### Step-by-Step Table

| Call | Jumps tried | Results | Cached value |
|------|-------------|---------|--------------|
| `minJumps(4)` | — (at end) | 0 | `memo[4] = 0` |
| `minJumps(3)` | jump 1 → index 4 | 1 + 0 = 1 | `memo[3] = 1` |
| `minJumps(2)` | jump 1 → index 3 | 1 + 1 = 2 | `memo[2] = 2` |
| `minJumps(1)` | jump 1 → index 2 (cost 2) | 1 + 2 = 3 | |
|               | jump 2 → index 3 (cost 1) | 1 + 1 = 2 | |
|               | jump 3 → index 4 (cost 0) | 1 + 0 = **1** | `memo[1] = 1` |
| `minJumps(0)` | jump 1 → index 1 (cost 1) | 1 + 1 = **2** | |
|               | jump 2 → index 2 (cost 2) | 1 + 2 = 3 | `memo[0] = 2` |

**Answer: 2 jumps** — path is `0 → 1 → 4`

---

### Why Memoization Matters Here

Without memoization, `minJumps(2)` and `minJumps(3)` would be recomputed every time they're reached from a different path. With the cache, each index is solved exactly once.

```
Without memo: exponential calls (branches multiply at each level)
With memo:    each of the n indices computed once → O(n²)
```

---

### Stuck Case: `[3, 2, 1, 0, 4]`

```
index:  0  1  2  3  4
value: [3, 2, 1, 0, 4]
```

| Call | Jumps tried | Result |
|------|-------------|--------|
| `minJumps(3)` | nums[3] = 0, no jumps possible | `Integer.MAX_VALUE` |
| `minJumps(2)` | jump 1 → index 3 → MAX_VALUE, skip | `Integer.MAX_VALUE` |
| `minJumps(1)` | jump 1 → index 2 → MAX_VALUE | |
|               | jump 2 → index 3 → MAX_VALUE | `Integer.MAX_VALUE` |
| `minJumps(0)` | all paths lead to MAX_VALUE | returns `-1` |

Every path funnels through index 3 which is a dead end — so the answer is **-1**.

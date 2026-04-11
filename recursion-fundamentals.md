# Recursion Fundamentals

A standalone reference for understanding recursion from the ground up — how it works, how to think about it, and how to apply it across different problem types.

---

## 1. What Is Recursion?

Recursion is when a function solves a problem by calling itself on a smaller version of the same problem. It keeps breaking the problem down until it hits a case small enough to answer directly — the **base case** — then builds the answer back up as the calls return.

```
Problem(n)
  └── Problem(n-1)
        └── Problem(n-2)
              └── ... base case → answer known
              ← result bubbles back up
        ← combined with current step
  ← final answer
```

Every recursive function needs exactly two things:

| Part | Purpose |
|------|---------|
| Base case | Stops the recursion — the smallest known answer |
| Recursive case | Breaks the problem down and calls itself |

Without a base case, the function recurses forever and crashes with a stack overflow.

---

## 2. The Call Stack

Each function call gets its own frame on the call stack. When a recursive function calls itself, a new frame is pushed. When it returns, the frame is popped and the result flows back to the caller.

```
factorial(4)
  └── 4 * factorial(3)
            └── 3 * factorial(2)
                      └── 2 * factorial(1)
                                └── returns 1   ← base case hit
                      returns 2 * 1 = 2
            returns 3 * 2 = 6
  returns 4 * 6 = 24
```

```java
public static int factorial(int n) {
    if (n <= 1) return 1;           // base case
    return n * factorial(n - 1);   // recursive case
}
```

The call stack depth equals the number of recursive calls before hitting the base case. For `factorial(n)` that's O(n) frames. Deep recursion on large inputs can overflow the stack — something to watch for.

---

## 3. Three Questions to Ask Before Writing Any Recursive Function

1. **What is the base case?** — the smallest input where the answer is known directly
2. **What does one recursive call return?** — trust it returns the correct answer for a smaller input
3. **How do I combine the result?** — use the subproblem's answer to build the full answer

This "trust the recursion" mindset is key. You don't need to trace every call — just define what the function does, trust it works for smaller inputs, and write the combination step.

---

## 4. Common Recursion Patterns

### Linear Recursion — one call per step

Each call makes exactly one recursive call. The problem shrinks by one step each time.

```java
// Sum of array elements
public static int sum(int[] nums, int i) {
    if (i == nums.length) return 0;          // base: nothing left
    return nums[i] + sum(nums, i + 1);       // current + rest
}

// Reverse a string
public static String reverse(String s) {
    if (s.length() <= 1) return s;           // base: single char
    return reverse(s.substring(1)) + s.charAt(0); // reverse tail + first char
}
```

Time: O(n) | Space: O(n) call stack

---

### Binary Recursion — two calls per step

Each call spawns two recursive calls. Common in divide-and-conquer and tree problems.

```java
// Fibonacci (naive)
public static int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}

// Count nodes in a binary tree
public static int countNodes(TreeNode root) {
    if (root == null) return 0;
    return 1 + countNodes(root.left) + countNodes(root.right);
}
```

Time: O(2^n) for Fibonacci without memoization | Space: O(n) call stack depth

---

### Divide and Conquer — split in half each time

Split the problem into two halves, solve each, then merge.

```java
// Merge sort
public static void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;               // base: single element
    int mid = (left + right) / 2;
    mergeSort(arr, left, mid);               // sort left half
    mergeSort(arr, mid + 1, right);          // sort right half
    merge(arr, left, mid, right);            // combine
}

// Binary search
public static int binarySearch(int[] arr, int target, int lo, int hi) {
    if (lo > hi) return -1;                  // base: not found
    int mid = (lo + hi) / 2;
    if (arr[mid] == target) return mid;
    if (arr[mid] < target) return binarySearch(arr, target, mid + 1, hi);
    return binarySearch(arr, target, lo, mid - 1);
}
```

Time: O(n log n) for merge sort | Space: O(log n) call stack

---

### Backtracking — explore, then undo

Try a choice, recurse, then undo the choice before trying the next option. Used when you need to explore all possibilities.

```java
// Generate all subsets of an array
public static void subsets(int[] nums, int i, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));    // record current subset
    for (int j = i; j < nums.length; j++) {
        current.add(nums[j]);                // choose
        subsets(nums, j + 1, current, result); // explore
        current.remove(current.size() - 1); // undo
    }
}
```

---

## 5. Worked Examples

### Example 1: Power Function

Compute `base^exp` recursively.

```java
public static long power(long base, int exp) {
    if (exp == 0) return 1;                  // anything^0 = 1
    if (exp % 2 == 0) {
        long half = power(base, exp / 2);    // even: square the half result
        return half * half;
    }
    return base * power(base, exp - 1);      // odd: multiply by base once
}
```

Naive version is O(n). This version halves the exponent each time → O(log n).

---

### Example 2: Flatten a Nested List

```java
public static List<Integer> flatten(Object[] nested) {
    List<Integer> result = new ArrayList<>();
    for (Object item : nested) {
        if (item instanceof Integer) {
            result.add((Integer) item);          // base: it's a number
        } else {
            result.addAll(flatten((Object[]) item)); // recurse into sub-list
        }
    }
    return result;
}
```

The structure of the data mirrors the recursive structure of the solution.

---

### Example 3: Tower of Hanoi

Move `n` disks from peg A to peg C using peg B as a helper. Rules: only move one disk at a time, never place a larger disk on a smaller one.

```java
public static void hanoi(int n, char from, char to, char aux) {
    if (n == 1) {
        System.out.println("Move disk 1 from " + from + " to " + to);
        return;
    }
    hanoi(n - 1, from, aux, to);   // move top n-1 disks out of the way
    System.out.println("Move disk " + n + " from " + from + " to " + to);
    hanoi(n - 1, aux, to, from);   // move n-1 disks onto the destination
}
```

Output for `hanoi(3, 'A', 'C', 'B')`:
```
Move disk 1 from A to C
Move disk 2 from A to B
Move disk 1 from C to B
Move disk 3 from A to C
Move disk 1 from B to A
Move disk 2 from B to C
Move disk 1 from A to C
```

This is a classic example where the recursive solution is elegant and the iterative one is not. Time: O(2^n) — unavoidable, since 2^n - 1 moves are required.

---

### Example 4: Permutations

Generate all permutations of a string.

```java
public static void permutations(String str, String current) {
    if (str.isEmpty()) {
        System.out.println(current);         // base: nothing left to add
        return;
    }
    for (int i = 0; i < str.length(); i++) {
        char chosen = str.charAt(i);
        String remaining = str.substring(0, i) + str.substring(i + 1);
        permutations(remaining, current + chosen); // choose one char, recurse on rest
    }
}

// Usage: permutations("abc", "")
// Output: abc, acb, bac, bca, cab, cba
```

---

### Example 5: Count Paths in a Grid

Count the number of unique paths from the top-left to the bottom-right of an `m x n` grid, moving only right or down.

```java
public static int countPaths(int m, int n) {
    if (m == 1 || n == 1) return 1;          // base: only one direction possible
    return countPaths(m - 1, n) + countPaths(m, n - 1); // came from above or from left
}
```

Recursion tree for `countPaths(3, 3)`:
```
countPaths(3,3)
  ├── countPaths(2,3)
  │     ├── countPaths(1,3) → 1
  │     └── countPaths(2,2)
  │           ├── countPaths(1,2) → 1
  │           └── countPaths(2,1) → 1
  │           returns 2
  │     returns 1 + 2 = 3
  └── countPaths(3,2)
        ├── countPaths(2,2) → 2  (same subproblem!)
        └── countPaths(3,1) → 1
        returns 2 + 1 = 3
  returns 3 + 3 = 6
```

Notice `countPaths(2,2)` is computed twice — this is where memoization or tabulation would help.

---

## 6. Common Mistakes

### Missing or wrong base case
```java
// BUG: fib(-1) will recurse forever
public static int fib(int n) {
    if (n == 0) return 0;
    return fib(n - 1) + fib(n - 2);  // fib(1) calls fib(-1) → infinite loop
}

// FIX: guard both base cases
public static int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

### Not making progress toward the base case
```java
// BUG: n never changes
public static int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n);   // should be sum(n - 1)
}
```

### Returning without combining correctly
```java
// BUG: discards the recursive result
public static int arraySum(int[] nums, int i) {
    if (i == nums.length) return 0;
    arraySum(nums, i + 1);           // result thrown away
    return nums[i];                  // only returns current element
}

// FIX
public static int arraySum(int[] nums, int i) {
    if (i == nums.length) return 0;
    return nums[i] + arraySum(nums, i + 1);
}
```

---

## 7. Recursion vs Iteration

| | Recursion | Iteration |
|---|---|---|
| Readability | Often cleaner for tree/graph problems | Better for simple loops |
| Stack usage | O(depth) call stack | O(1) typically |
| Overflow risk | Yes, for deep inputs | No |
| Debugging | Harder to trace | Easier to step through |
| When to use | Tree traversal, divide & conquer, backtracking | Linear scans, simple accumulation |

A recursive solution can always be converted to an iterative one using an explicit stack — but it's often not worth it unless you're hitting stack limits.

---

## 8. Exercises

### Exercise 1: Sum of Digits
Given a non-negative integer, return the sum of its digits recursively.
```
Input:  1234
Output: 10  (1 + 2 + 3 + 4)
```
Hint: `sumDigits(n) = n % 10 + sumDigits(n / 10)`

---

### Exercise 2: Check Palindrome
Given a string, return true if it's a palindrome using recursion (no loops).
```
Input:  "racecar"  → true
Input:  "hello"    → false
```
Hint: compare first and last characters, then recurse on the middle.

---

### Exercise 3: Binary Search (Recursive)
Implement binary search recursively. Return the index of the target, or -1 if not found.
```
Input:  arr = [1, 3, 5, 7, 9, 11], target = 7
Output: 3
```

---

### Exercise 4: Flatten Nested Array
Given a nested integer array (arrays within arrays), return a flat list of all integers.
```
Input:  [1, [2, [3, 4], 5], 6]
Output: [1, 2, 3, 4, 5, 6]
```

---

### Solutions

```java
import java.util.ArrayList;
import java.util.List;

public class RecursionExercises {

    // Exercise 1: Sum of digits
    public static int sumDigits(int n) {
        if (n < 10) return n;
        return n % 10 + sumDigits(n / 10);
    }

    // Exercise 2: Check palindrome
    public static boolean isPalindrome(String s) {
        if (s.length() <= 1) return true;
        if (s.charAt(0) != s.charAt(s.length() - 1)) return false;
        return isPalindrome(s.substring(1, s.length() - 1));
    }

    // Exercise 3: Binary search
    public static int binarySearch(int[] arr, int target, int lo, int hi) {
        if (lo > hi) return -1;
        int mid = (lo + hi) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) return binarySearch(arr, target, mid + 1, hi);
        return binarySearch(arr, target, lo, mid - 1);
    }

    // Exercise 4: Flatten nested array (using Object[] to simulate nesting)
    public static List<Integer> flatten(Object[] nested) {
        List<Integer> result = new ArrayList<>();
        for (Object item : nested) {
            if (item instanceof Integer) {
                result.add((Integer) item);
            } else {
                result.addAll(flatten((Object[]) item));
            }
        }
        return result;
    }

    public static void main(String[] args) {
        System.out.println(sumDigits(1234));   // 10
        System.out.println(sumDigits(9));      // 9

        System.out.println(isPalindrome("racecar")); // true
        System.out.println(isPalindrome("hello"));   // false

        int[] arr = {1, 3, 5, 7, 9, 11};
        System.out.println(binarySearch(arr, 7, 0, arr.length - 1));  // 3
        System.out.println(binarySearch(arr, 4, 0, arr.length - 1));  // -1

        Object[] nested = {1, new Object[]{2, new Object[]{3, 4}, 5}, 6};
        System.out.println(flatten(nested)); // [1, 2, 3, 4, 5, 6]
    }
}
```

---

## Key Takeaways

- Every recursive function needs a base case and a recursive case
- Trust the recursion — define what the function does, not how it does it
- Draw the call stack or recursion tree when you're stuck
- Binary recursion (two calls per step) leads to exponential time without memoization
- Backtracking = recursion + undo — use it when you need all possible solutions

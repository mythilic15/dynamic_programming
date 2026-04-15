# Giant Slalom — At Most 2 Direction Changes

## Problem Summary

You ski downhill through N gates. Gates are ordered by distance from the start (index = row), and positioned by distance from the right barrier (`A[i]` = column).

- You **must start going left** (increasing `A` values).
- You may change direction **at most twice**.
- Goal: pass the **maximum number of gates**.

The allowed movement patterns are:

| Pattern | Direction changes |
|---|---|
| L | 0 |
| L → R | 1 |
| L → R → L | 2 |

In terms of `A` values (subsequence must respect gate index order):

- **L**: strictly increasing subsequence
- **L → R**: increasing then decreasing
- **L → R → L**: increasing → decreasing → increasing

---

## Why This Approach?

### Naive idea — and why it's too slow

A brute-force approach would try every possible pair of "turn points" `(i, j)` where `i` is where you switch from L→R and `j` is where you switch from R→L. For each pair you'd compute the best increasing run before `i`, the best decreasing run between `i` and `j`, and the best increasing run after `j`. That's O(N²) pairs × O(N) per pair = **O(N³)** — way too slow for N = 100,000.

You could precompute prefix/suffix LIS arrays to bring it to O(N²), but that's still too slow.

### The right framing — chained LIS

The key observation is: **you don't need to know where the turns happen upfront**. Instead, think of each gate `i` as potentially being in one of three "phases":

1. Still going left (increasing run)
2. Now going right (decreasing run, after at least one left run)
3. Going left again (increasing run, after a left-then-right run)

If you define DP values for each gate in each phase, the transitions are clean and you can compute everything in a single left-to-right pass. This is essentially **three interleaved LIS problems**, each feeding into the next.

### Why patience sorting (binary search)?

The classic O(N²) LIS uses nested loops: for each `i`, scan all `j < i`. Patience sorting replaces that inner scan with a **binary search on a "tails" array**, giving O(N log N).

The tails array maintains a compact summary of all subsequences seen so far:
- For increasing sequences: `tails[k]` = the **smallest** possible last element of any increasing subsequence of length `k+1`. Smallest is best because it's easiest to extend.
- For decreasing sequences: `tails[k]` = the **largest** possible last element of any decreasing subsequence of length `k+1`. Largest is best because it's easiest to extend downward.

The length of the tails array at any point equals the length of the longest subsequence found so far.

### Why seed the next phase's tails array?

When we finish computing `up[i]` (the best increasing run ending at gate `i`), gate `i` could be the **turn point** — the moment you switch from going left to going right. So we immediately "offer" this to the decreasing-phase tails array: there now exists a decreasing subsequence of length `up[i]` whose last element is `A[i]`. This is the seeding step.

The same logic applies when transitioning from phase 2 → phase 3.

This seeding is what connects the three phases without needing to track turn points explicitly.

---

## Step-by-Step DP Explanation

### The three DP arrays

| Array | Meaning |
|---|---|
| `up[i]` | Longest increasing subseq ending at gate `i` |
| `down[i]` | Longest inc→dec subseq ending at gate `i` |
| `up2[i]` | Longest inc→dec→inc subseq ending at gate `i` |

### Recurrences

```
up[i]   = 1 + max{ up[j]   | j < i, A[j] < A[i] }
down[i] = 1 + max{ up[j]   | j < i, A[j] > A[i] }   ← turn happened at j
        = 1 + max{ down[j] | j < i, A[j] > A[i] }   ← continuing the right run
up2[i]  = 1 + max{ down[j] | j < i, A[j] < A[i] }   ← turn happened at j
```

The answer is `max over all i of { up[i], down[i], up2[i] }`.

Note that `down[i]` covers both "j was the first turn" and "j was already in the right run" — both are captured by the same recurrence because `down[j]` already encodes the best inc→dec chain ending at `j`.

### Why the answer covers all cases

- If you never change direction: `up[i]` gives the answer.
- If you change once (L→R): `down[i]` gives the answer.
- If you change twice (L→R→L): `up2[i]` gives the answer.

Taking the max across all three and all gates covers every valid ski run.

---

## Algorithm — O(N log N)

Use **patience sorting** (binary search on tail arrays) for each phase, seeding the next phase's tail array as we go.

### Tail array invariants

- `tailsInc1[k]` (ascending): smallest last value of any increasing subseq of length `k+1`
- `tailsDec[k]` (descending): largest last value of any dec subseq of length `k+1`
- `tailsInc2[k]` (ascending): smallest last value of any inc subseq (phase 3) of length `k+1`

### Per gate `i` with value `v = A[i]`:

**Phase 1 — compute `up[i]`:**
- Binary search `tailsInc1` for the first slot `>= v` → call it `p1`.
- `up[i] = p1 + 1` (we can extend a run of length `p1` with `v`).
- Set `tailsInc1[p1] = v` (replace with smaller tail for future extensibility).

**Phase 2 — compute `down[i]`:**
- Seed: gate `i` could be the turn point. A decreasing run of length `up[i]` now ends at `v`. Update `tailsDec[up[i]-1] = max(tailsDec[up[i]-1], v)`.
- Binary search `tailsDec` for count of elements `> v` → call it `p2`.
- `down[i] = p2 + 1`.
- Update `tailsDec[p2] = max(tailsDec[p2], v)`.

**Phase 3 — compute `up2[i]`:**
- Seed: gate `i` could be the second turn point. An increasing run of length `down[i]` now ends at `v`. Update `tailsInc2[down[i]-1] = min(tailsInc2[down[i]-1], v)`.
- Binary search `tailsInc2` for first slot `>= v` → call it `p3`.
- `up2[i] = p3 + 1`.
- Update `tailsInc2[p3] = min(tailsInc2[p3], v)`.

---

## Worked Example — Trace

```
A = [15, 13, 5, 7, 4, 10, 12, 8, 2, 11, 6, 9, 3]
idx:  0   1   2  3  4   5   6  7  8   9 10  11 12
```

Let's trace the DP values for each gate:

| i | A[i] | up[i] | down[i] | up2[i] |
|---|------|-------|---------|--------|
| 0 | 15   | 1     | 1       | 1      |
| 1 | 13   | 1     | 2       | 2      |
| 2 | 5    | 1     | 3       | 3      |
| 3 | 7    | 2     | 3       | 4      |
| 4 | 4    | 1     | 4       | 4      |
| 5 | 10   | 3     | 4       | 5      |
| 6 | 12   | 4     | 4       | 5      |
| 7 | 8    | 3     | 5       | 6      |
| 8 | 2    | 1     | 6       | 6      |
| 9 | 11   | 4     | 6       | 7      |
| 10| 6    | 2     | 7       | 7      |
| 11| 9    | 3     | 7       | 8      |
| 12| 3    | 1     | 8       | 8      |

Maximum `up2[i]` = **8** at indices 11 and 12.

The actual path that achieves 8:
- Indices `2, 3, 5, 6` → values `5, 7, 10, 12` (going left, increasing)
- Indices `7, 8` → values `8, 2` (going right, decreasing)
- Indices `10, 11` → values `6, 9` (going left again, increasing)

```
Answer: 8
```

---

## Edge Cases

| Input | Expected | Reason |
|---|---|---|
| `[1]` | 1 | Single gate, always passable |
| `[1, 5]` | 2 | Pure L, both gates passed |
| `[5, 4, 3, 2, 1]` | 1 | Must start L; no two gates form an increasing pair |
| `[1, 2, 3, 4, 5]` | 5 | Pure L, all gates passed |
| `[3, 1, 2]` | 3 | L→R→L: 3 (L), 1 (R), 2 (L) |
| `[1, 3, 2]` | 3 | L→R: 1→3 then 2, all three passed |

---

## Java Solution

```java
public class Solution {

    public int solution(int[] A) {
        int N = A.length;
        if (N == 0) return 0;

        int[] tailsInc1 = new int[N];
        int[] tailsDec  = new int[N];
        int[] tailsInc2 = new int[N];
        int lenInc1 = 0, lenDec = 0, lenInc2 = 0;

        int ans = 0;

        for (int i = 0; i < N; i++) {
            int v = A[i];

            // ── Phase 1: longest increasing subseq ending at i ──────────────
            int p1 = lowerBound(tailsInc1, lenInc1, v);
            int upI = p1 + 1;
            tailsInc1[p1] = v;
            if (p1 == lenInc1) lenInc1++;

            // ── Phase 2: longest inc→dec subseq ending at i ─────────────────
            // Seed: gate i is the turn point for a dec run of length upI
            int seed2 = upI - 1;
            if (seed2 >= lenDec) {
                for (int k = lenDec; k < seed2; k++) tailsDec[k] = -1;
                tailsDec[seed2] = v;
                lenDec = seed2 + 1;
            } else if (tailsDec[seed2] < v) {
                tailsDec[seed2] = v;
            }
            int p2 = upperBoundDesc(tailsDec, lenDec, v);
            int downI = p2 + 1;
            int best2 = downI - 1;
            if (best2 >= lenDec) {
                for (int k = lenDec; k < best2; k++) tailsDec[k] = -1;
                tailsDec[best2] = v;
                lenDec = best2 + 1;
            } else if (tailsDec[best2] < v) {
                tailsDec[best2] = v;
            }

            // ── Phase 3: longest inc→dec→inc subseq ending at i ────────────
            // Seed: gate i is the second turn point for an inc run of length downI
            int seed3 = downI - 1;
            if (seed3 >= lenInc2) {
                for (int k = lenInc2; k < seed3; k++) tailsInc2[k] = Integer.MAX_VALUE;
                tailsInc2[seed3] = v;
                lenInc2 = seed3 + 1;
            } else if (tailsInc2[seed3] > v) {
                tailsInc2[seed3] = v;
            }
            int p3 = lowerBound(tailsInc2, lenInc2, v);
            int up2I = p3 + 1;
            int best3 = up2I - 1;
            if (best3 >= lenInc2) {
                for (int k = lenInc2; k < best3; k++) tailsInc2[k] = Integer.MAX_VALUE;
                tailsInc2[best3] = v;
                lenInc2 = best3 + 1;
            } else if (tailsInc2[best3] > v) {
                tailsInc2[best3] = v;
            }

            ans = Math.max(ans, Math.max(upI, Math.max(downI, up2I)));
        }

        return ans;
    }

    /**
     * Returns the index of the first element >= v in arr[0..len).
     * arr is sorted ascending. Used for increasing-phase binary search.
     */
    private int lowerBound(int[] arr, int len, int v) {
        int lo = 0, hi = len;
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;
            if (arr[mid] < v) lo = mid + 1;
            else hi = mid;
        }
        return lo;
    }

    /**
     * Returns the count of elements strictly > v in arr[0..len).
     * arr is sorted descending. Used for decreasing-phase binary search.
     * This count equals the length of the longest decreasing subseq we can extend with v.
     */
    private int upperBoundDesc(int[] arr, int len, int v) {
        int lo = 0, hi = len;
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;
            if (arr[mid] > v) lo = mid + 1;
            else hi = mid;
        }
        return lo;
    }
}
```

---

## Complexity

| | Value | Reason |
|---|---|---|
| Time | O(N log N) | One pass over N gates, each doing O(log N) binary searches |
| Space | O(N) | Three tail arrays of size N |

### Why not O(N²)?

The naive DP recurrence `up[i] = 1 + max{ up[j] | j < i, A[j] < A[i] }` requires scanning all previous gates — O(N) per gate, O(N²) total. Patience sorting replaces that scan with a binary search on a compact tails array, cutting it to O(log N) per gate. The tails array works because we only ever need to know "what's the best (smallest/largest) tail for each possible length" — we don't need to remember every individual subsequence.

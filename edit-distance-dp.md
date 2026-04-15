# LeetCode 72 — Edit Distance

Given two strings `word1` and `word2`, return the **minimum number of operations** to convert `word1` into `word2`.

Allowed operations (each costs 1):
- **Insert** a character
- **Delete** a character
- **Replace** a character

```
Input:  word1 = "horse", word2 = "ros"
Output: 3

Input:  word1 = "intention", word2 = "execution"
Output: 5
```

---

## How to Think About It

Compare the two strings character by character. At each position `(i, j)`:

- If `word1[i] == word2[j]` → characters already match, **no operation needed**, move both pointers
- If they differ → you have three choices, each costing 1:
  - **Replace** `word1[i]` with `word2[j]` → then solve `(i-1, j-1)`
  - **Delete** `word1[i]` → then solve `(i-1, j)`
  - **Insert** `word2[j]` into `word1` → then solve `(i, j-1)`

Take the minimum of the three.

---

## DP State

```
dp[i][j] = min operations to convert word1[0..i-1] to word2[0..j-1]
```

### Recurrence

```
if word1[i-1] == word2[j-1]:
    dp[i][j] = dp[i-1][j-1]           // free — characters match

else:
    dp[i][j] = 1 + min(
        dp[i-1][j-1],                  // replace
        dp[i-1][j],                    // delete  (advance i, j stays)
        dp[i][j-1]                     // insert  (i stays, advance j)
    )
```

### Base Cases

```
dp[i][0] = i    // delete all i chars from word1 to reach empty string
dp[0][j] = j    // insert all j chars to build word2 from empty string
```

---

## Step-by-Step Solving: `"horse"` → `"ros"`

**Step 1 — Set up the table**

Rows = characters of `word1` ("horse"), columns = characters of `word2` ("ros").
Fill the first row and column with 0, 1, 2, 3... (base cases).

```
       ""  r  o  s
  ""  [ 0, 1, 2, 3 ]
   h  [ 1, ?, ?, ? ]
   o  [ 2, ?, ?, ? ]
   r  [ 3, ?, ?, ? ]
   s  [ 4, ?, ?, ? ]
   e  [ 5, ?, ?, ? ]
```

**Step 2 — Fill row by row**

`dp[1][1]`: h vs r → differ → 1 + min(dp[0][0], dp[0][1], dp[1][0]) = 1 + min(0,1,1) = **1**
`dp[1][2]`: h vs o → differ → 1 + min(dp[0][1], dp[0][2], dp[1][1]) = 1 + min(1,2,1) = **2**
`dp[1][3]`: h vs s → differ → 1 + min(dp[0][2], dp[0][3], dp[1][2]) = 1 + min(2,3,2) = **3**

`dp[2][1]`: o vs r → differ → 1 + min(dp[1][0], dp[1][1], dp[2][0]) = 1 + min(1,1,2) = **2**
`dp[2][2]`: o vs o → **match** → dp[1][1] = **1**
`dp[2][3]`: o vs s → differ → 1 + min(dp[1][2], dp[1][3], dp[2][2]) = 1 + min(2,3,1) = **2**

`dp[3][1]`: r vs r → **match** → dp[2][0] = **2**
`dp[3][2]`: r vs o → differ → 1 + min(dp[2][1], dp[2][2], dp[3][1]) = 1 + min(2,1,2) = **2**
`dp[3][3]`: r vs s → differ → 1 + min(dp[2][2], dp[2][3], dp[3][2]) = 1 + min(1,2,2) = **2**

`dp[4][1]`: s vs r → differ → 1 + min(dp[3][0], dp[3][1], dp[4][0]) = 1 + min(3,2,4) = **3**
`dp[4][2]`: s vs o → differ → 1 + min(dp[3][1], dp[3][2], dp[4][1]) = 1 + min(2,2,3) = **3**
`dp[4][3]`: s vs s → **match** → dp[3][2] = **2**

`dp[5][1]`: e vs r → differ → 1 + min(dp[4][0], dp[4][1], dp[5][0]) = 1 + min(4,3,5) = **4**
`dp[5][2]`: e vs o → differ → 1 + min(dp[4][1], dp[4][2], dp[5][1]) = 1 + min(3,3,4) = **4**
`dp[5][3]`: e vs s → differ → 1 + min(dp[4][2], dp[4][3], dp[5][2]) = 1 + min(3,2,4) = **3**

**Step 3 — Completed table**

```
       ""  r  o  s
  ""  [ 0, 1, 2, 3 ]
   h  [ 1, 1, 2, 3 ]
   o  [ 2, 2, 1, 2 ]
   r  [ 3, 2, 2, 2 ]
   s  [ 4, 3, 3, 2 ]
   e  [ 5, 4, 4, 3 ]
```

Answer: `dp[5][3] = 3` ✓

**Step 4 — What are the 3 operations?**

Trace back from `dp[5][3]`:
1. `dp[5][3]=3` came from `dp[4][2]=3` (delete 'e') → **delete 'e'**  → "hors"
2. `dp[4][2]=3` came from `dp[3][2]=2` (delete 's') → **delete 's'**  → "hor"
3. `dp[3][2]=2` came from `dp[2][1]=2` (replace 'r'→'o'... wait, r vs o differ)
   Actually `dp[3][1]=2` is a match (r==r), so `dp[3][2]` came from replace.
   → **replace 'h' with 'r'** → "ros"

---

## Full DP Table: `"intention"` → `"execution"` (answer = 5)

```
        ""  e  x  e  c  u  t  i  o  n
   ""  [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    i  [ 1, 1, 2, 3, 4, 5, 6, 6, 7, 8]
    n  [ 2, 2, 2, 3, 4, 5, 6, 7, 7, 7]
    t  [ 3, 3, 3, 3, 4, 5, 5, 6, 7, 8]
    e  [ 4, 3, 4, 3, 4, 5, 6, 6, 7, 8]
    n  [ 5, 4, 4, 4, 4, 5, 6, 7, 7, 7]
    t  [ 6, 5, 5, 5, 5, 5, 5, 6, 7, 8]
    i  [ 7, 6, 6, 6, 6, 6, 6, 5, 6, 7]
    o  [ 8, 7, 7, 7, 7, 7, 7, 6, 5, 6]
    n  [ 9, 8, 8, 8, 8, 8, 8, 7, 6, 5]
```

Answer: `dp[9][9] = 5` ✓

---

## Java Implementation

### Tabulation — O(m×n) time, O(m×n) space

```java
public int minDistance(String word1, String word2) {
    int m = word1.length();
    int n = word2.length();

    int[][] dp = new int[m + 1][n + 1];

    // Base cases
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];          // match — free
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i - 1][j - 1],                  // replace
                    Math.min(
                        dp[i - 1][j],                  // delete
                        dp[i][j - 1]                   // insert
                    )
                );
            }
        }
    }

    return dp[m][n];
}
```

### Space-Optimised — O(m×n) time, O(n) space

Since each row only depends on the previous row, you only need two 1D arrays.

```java
public int minDistance(String word1, String word2) {
    int m = word1.length();
    int n = word2.length();

    int[] prev = new int[n + 1];
    int[] curr = new int[n + 1];

    for (int j = 0; j <= n; j++) prev[j] = j; // base case: empty word1

    for (int i = 1; i <= m; i++) {
        curr[0] = i; // base case: delete all i chars
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                curr[j] = prev[j - 1];
            } else {
                curr[j] = 1 + Math.min(prev[j - 1],   // replace
                              Math.min(prev[j],         // delete
                                       curr[j - 1]));   // insert
            }
        }
        int[] tmp = prev; prev = curr; curr = tmp; // swap rows
    }

    return prev[n];
}
```

---

## What Each Operation Means in the Table

| Operation | Comes from    | Meaning |
|-----------|---------------|---------|
| Replace   | `dp[i-1][j-1]`| swap `word1[i]` for `word2[j]` |
| Delete    | `dp[i-1][j]`  | remove `word1[i]`, try matching `word2[j]` again |
| Insert    | `dp[i][j-1]`  | insert `word2[j]` into `word1`, advance `j` |
| Match     | `dp[i-1][j-1]`| characters equal, no cost |

---

## Complexity

| Version | Time | Space |
|---------|------|-------|
| Full table | O(m × n) | O(m × n) |
| Rolling rows | O(m × n) | O(n) |

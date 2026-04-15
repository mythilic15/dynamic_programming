# Maze Problem — Finding Your Way Through a Grid

## Problem Statement

You are given a 2D grid representing a maze. Each cell is either:
- `0` — open, you can walk through it
- `1` — a wall, you cannot enter it

You start at a given cell `(startRow, startCol)` and want to reach a target cell `(endRow, endCol)`. You can move in four directions: **up, down, left, right** (no diagonals).

**Questions the maze problem asks:**
1. **Can you reach the exit at all?**
2. **What is the shortest path (minimum steps) to the exit?**
3. **What does the actual path look like?**

```
Maze example (0 = open, 1 = wall):

  col→  0  1  2  3
row 0 [ S, 0, 1, 0 ]    S = start (0,0)
row 1 [ 0, 0, 1, 0 ]    E = exit  (3,3)
row 2 [ 1, 0, 0, 0 ]
row 3 [ 1, 1, 0, E ]

Answer: shortest path = 6 steps
Path:   (0,0)→(1,0)→(1,1)→(2,1)→(2,2)→(2,3)→... wait
        (0,0)→(1,0)→(1,1)→(2,1)→(2,2)→(3,2)→(3,3)
```

---

## Why This Is a Graph Problem

At first glance this looks like a 2D grid problem. But if you squint, it's just a graph:

- Every open cell `(r, c)` is a **node**
- Two cells are **connected by an edge** if they are adjacent (up/down/left/right) and both open
- Finding a path = graph traversal
- Finding the shortest path = BFS

You never need to build an explicit adjacency list. You generate neighbors on the fly using direction arrays — a standard trick for grid graphs.

```
Direction arrays (the 4 moves):

dr = [-1,  1,  0,  0]   ← row delta
dc = [ 0,  0, -1,  1]   ← col delta

For cell (r, c), the 4 neighbors are:
  (r-1, c)  ← up
  (r+1, c)  ← down
  (r, c-1)  ← left
  (r, c+1)  ← right
```

---

## Approach

### Question 1: Can you reach the exit? → Use DFS

DFS is the right tool when you just need to know **if** a path exists. It dives deep along one branch, backtracks when stuck, and returns true the moment it finds the exit. It doesn't care about path length — just reachability.

**Mental model:** imagine dropping a thread as you walk. If you reach the exit, yes. If you run out of moves, no.

### Question 2: What is the shortest path? → Use BFS

BFS explores cells in order of their distance from the start — all cells 1 step away, then all cells 2 steps away, and so on. The first time BFS reaches the exit, it's guaranteed to be via the minimum number of steps.

**Mental model:** imagine ripples spreading outward from a stone dropped in water. The ripple that first touches the exit tells you the shortest distance.

### Why DFS fails for shortest path

DFS might find a path, but it could be a long winding one. It has no concept of "distance so far" — it just follows one branch until it hits a dead end or the exit.

```
BFS (explores by distance rings):    DFS (dives one branch):

  S → → ↓                              S ↓
  . . . ↓                              ↓ . . .
  . . . ↓                              ↓ . . .
  . . . E   ← finds shortest           ↓ → → E  ← might find a longer path
```

---

## Step-by-Step BFS Trace

```
Maze:
[ S, 0, 1, 0 ]    S=(0,0)  E=(3,3)
[ 0, 0, 1, 0 ]
[ 1, 0, 0, 0 ]
[ 1, 1, 0, 0 ]
```

```
Initial: queue=[(0,0,dist=0)], visited={(0,0)}

Step 1: pop (0,0) dist=0
        neighbors: (1,0)✓ open  →  enqueue (1,0, dist=1)
                   (0,1)✓ open  →  enqueue (0,1, dist=1)
                   (-1,0) out of bounds
                   (0,-1) out of bounds

Step 2: pop (1,0) dist=1
        neighbors: (0,0) already visited
                   (2,0) = wall, skip
                   (1,1)✓ open  →  enqueue (1,1, dist=2)

Step 3: pop (0,1) dist=1
        neighbors: (0,0) already visited
                   (0,2) = wall, skip
                   (1,1) already queued

Step 4: pop (1,1) dist=2
        neighbors: (1,0) visited
                   (1,2) = wall, skip
                   (0,1) visited
                   (2,1)✓ open  →  enqueue (2,1, dist=3)

Step 5: pop (2,1) dist=3
        neighbors: (1,1) visited
                   (3,1) = wall, skip
                   (2,0) = wall, skip
                   (2,2)✓ open  →  enqueue (2,2, dist=4)

Step 6: pop (2,2) dist=4
        neighbors: (1,2) = wall, skip
                   (3,2)✓ open  →  enqueue (3,2, dist=5)
                   (2,1) visited
                   (2,3)✓ open  →  enqueue (2,3, dist=5)

Step 7: pop (3,2) dist=5
        neighbors: (2,2) visited
                   (3,1) = wall, skip
                   (3,3)✓ open  →  enqueue (3,3, dist=6)

Step 8: pop (2,3) dist=5
        neighbors: (1,3)✓ open  →  enqueue (1,3, dist=6)
                   (3,3) already queued
                   (2,2) visited

Step 9: pop (3,3) dist=6
        → THIS IS THE EXIT! Return 6.
```

**Shortest path = 6 steps.**

---

## Step-by-Step DFS Trace (Reachability)

```
dfs(0,0, visited={})
  mark (0,0) visited
  try up    → out of bounds
  try down  → (1,0) open, recurse
    dfs(1,0)
      mark (1,0) visited
      try up    → (0,0) visited, skip
      try down  → (2,0) = wall, skip
      try left  → out of bounds
      try right → (1,1) open, recurse
        dfs(1,1)
          ...continues deeper...
          eventually reaches (3,3) → return true ✓
        ← true
      ← true
    ← true
  ← true
→ EXIT IS REACHABLE
```

DFS returns `true` as soon as any branch finds the exit. The `true` bubbles back up the call stack immediately.

---

## Java Solution

```java
import java.util.*;

public class MazeSolver {

    // The 4 directions: up, down, left, right
    static int[] dr = {-1, 1, 0, 0};
    static int[] dc = {0, 0, -1, 1};

    /**
     * BFS — returns the shortest number of steps from start to end.
     * Returns -1 if the exit is unreachable.
     *
     * Why BFS? It explores cells level by level (by distance).
     * The first time it reaches the exit, that distance is guaranteed minimal.
     */
    public static int shortestPath(int[][] maze, int[] start, int[] end) {
        int rows = maze.length;
        int cols = maze[0].length;
        boolean[][] visited = new boolean[rows][cols];

        // Queue stores {row, col, distance}
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{start[0], start[1], 0});
        visited[start[0]][start[1]] = true;

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int r = curr[0], c = curr[1], dist = curr[2];

            // Reached the exit
            if (r == end[0] && c == end[1]) return dist;

            // Explore all 4 neighbors
            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d];
                int nc = c + dc[d];

                boolean inBounds = nr >= 0 && nr < rows && nc >= 0 && nc < cols;
                if (inBounds && maze[nr][nc] == 0 && !visited[nr][nc]) {
                    visited[nr][nc] = true;
                    queue.offer(new int[]{nr, nc, dist + 1});
                }
            }
        }

        return -1; // no path exists
    }

    /**
     * DFS — returns true if the exit is reachable from (r, c).
     *
     * Why DFS? Simple recursion, no need to track distance.
     * Returns as soon as any path to the exit is found.
     */
    public static boolean canReach(int[][] maze, int r, int c,
                                   int[] end, boolean[][] visited) {
        int rows = maze.length, cols = maze[0].length;

        // Out of bounds or wall or already visited
        if (r < 0 || r >= rows || c < 0 || c >= cols) return false;
        if (maze[r][c] == 1 || visited[r][c]) return false;

        // Reached the exit
        if (r == end[0] && c == end[1]) return true;

        visited[r][c] = true;

        // Try all 4 directions
        for (int d = 0; d < 4; d++) {
            if (canReach(maze, r + dr[d], c + dc[d], end, visited))
                return true;
        }

        return false;
    }

    /**
     * BFS with path reconstruction — returns the actual path as a list of cells.
     * Returns empty list if unreachable.
     *
     * Key idea: store the parent of each cell so we can trace back from exit to start.
     */
    public static List<int[]> findPath(int[][] maze, int[] start, int[] end) {
        int rows = maze.length, cols = maze[0].length;
        int[][][] parent = new int[rows][cols][2]; // parent[r][c] = {parentRow, parentCol}
        boolean[][] visited = new boolean[rows][cols];

        // Sentinel: no parent
        for (int[][] row : parent)
            for (int[] cell : row)
                Arrays.fill(cell, -1);

        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{start[0], start[1]});
        visited[start[0]][start[1]] = true;

        boolean found = false;

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int r = curr[0], c = curr[1];

            if (r == end[0] && c == end[1]) {
                found = true;
                break;
            }

            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d];
                int nc = c + dc[d];

                boolean inBounds = nr >= 0 && nr < rows && nc >= 0 && nc < cols;
                if (inBounds && maze[nr][nc] == 0 && !visited[nr][nc]) {
                    visited[nr][nc] = true;
                    parent[nr][nc] = new int[]{r, c};
                    queue.offer(new int[]{nr, nc});
                }
            }
        }

        if (!found) return Collections.emptyList();

        // Trace back from end to start using parent pointers
        List<int[]> path = new ArrayList<>();
        int[] curr = end;
        while (curr[0] != -1) {
            path.add(curr);
            int[] p = parent[curr[0]][curr[1]];
            curr = p;
        }

        Collections.reverse(path);
        return path;
    }

    public static void main(String[] args) {
        int[][] maze = {
            {0, 0, 1, 0},
            {0, 0, 1, 0},
            {1, 0, 0, 0},
            {1, 1, 0, 0}
        };

        int[] start = {0, 0};
        int[] end   = {3, 3};

        // Shortest path length
        System.out.println("Shortest path: " + shortestPath(maze, start, end));
        // → 6

        // Reachability
        System.out.println("Reachable: " + canReach(maze, 0, 0, end, new boolean[4][4]));
        // → true

        // Actual path
        List<int[]> path = findPath(maze, start, end);
        System.out.print("Path: ");
        for (int[] cell : path)
            System.out.print("(" + cell[0] + "," + cell[1] + ") ");
        // → (0,0) (1,0) (1,1) (2,1) (2,2) (3,2) (3,3)
    }
}
```

---

## Path Reconstruction — How It Works

When BFS visits a cell, it records which cell it came from (the parent). After reaching the exit, you trace backwards from exit → parent → parent → ... → start, then reverse the list.

```
parent map after BFS:

(1,0) came from (0,0)
(0,1) came from (0,0)
(1,1) came from (1,0)
(2,1) came from (1,1)
(2,2) came from (2,1)
(3,2) came from (2,2)
(3,3) came from (3,2)

Trace back from (3,3):
(3,3) → (3,2) → (2,2) → (2,1) → (1,1) → (1,0) → (0,0)

Reverse:
(0,0) → (1,0) → (1,1) → (2,1) → (2,2) → (3,2) → (3,3)  ✓
```

---

## Common Variations

| Variation | What changes | Algorithm |
|---|---|---|
| Shortest path (basic) | Count steps | BFS |
| Just reachability | Yes/no answer | DFS |
| Weighted moves (terrain cost) | Each step has a cost | Dijkstra |
| Find all paths | Don't mark visited permanently | DFS (backtracking) |
| Count islands | Multiple disconnected regions | BFS/DFS + outer loop |
| Maze with keys and doors | State = `(r, c, keys_held)` | BFS on expanded state |
| Minimum steps to collect all items | Bitmask over items | BFS with bitmask state |

---

## Key Takeaways

- A maze is a graph — cells are nodes, valid moves are edges
- Use **BFS** for shortest path (explores by distance, guarantees minimum steps)
- Use **DFS** for reachability (simpler, just needs to find any path)
- Direction arrays `dr[]` / `dc[]` are the standard way to generate neighbors in a grid
- Always mark cells visited **before** enqueuing (not after popping) to avoid duplicates in the queue
- Path reconstruction uses a `parent` array filled during BFS, then traced backwards from exit to start

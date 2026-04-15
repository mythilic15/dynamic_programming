# Topological Order

## What Is It?

A **topological ordering** of a directed graph is a linear arrangement of all nodes such that for every directed edge `A → B`, node `A` appears before node `B` in the ordering.

Think of it as answering: **"in what order must I do these things, given their dependencies?"**

```
Example: Course prerequisites

  Math101 → Math201 → Math301
               ↑
  CS101  → CS201

One valid topological order: Math101, CS101, Math201, CS201, Math301
Another valid order:         CS101, Math101, CS201, Math201, Math301

Both are correct — topological order is not always unique.
```

---

## When Does It Apply?

Topological order only works on a **Directed Acyclic Graph (DAG)**:
- **Directed** — edges have a direction (A → B, not A — B)
- **Acyclic** — no cycles (if there's a cycle, there's no valid ordering)

```
Valid DAG (can topologically sort):    Has a cycle (cannot sort):

  A → B → D                             A → B
  ↓       ↑                             ↑   ↓
  C ──────┘                             D ← C
```

If `A` must come before `B`, and `B` must come before `A`, there's no valid ordering — it's a contradiction.

---

## Real-World Examples

| Scenario | Nodes | Edges mean |
|---|---|---|
| Course prerequisites | Courses | "must take before" |
| Build systems (Make, Maven) | Files/modules | "must compile before" |
| Task scheduling | Tasks | "must finish before" |
| Package dependencies (npm, pip) | Packages | "must install before" |
| Spreadsheet cell evaluation | Cells | "must compute before" |

---

## Two Algorithms

There are two standard ways to compute topological order:

| | Kahn's Algorithm (BFS) | DFS-based |
|---|---|---|
| Approach | Process nodes with no incoming edges first | Post-order DFS, reverse the result |
| Data structure | Queue + in-degree array | Stack / recursion |
| Cycle detection | Yes — if result has fewer nodes than graph | Yes — detect back edges |
| Intuition | "Do tasks with no prerequisites first" | "Finish all dependents before adding a node" |

---

## Algorithm 1: Kahn's Algorithm (BFS)

### Intuition

A node with **in-degree 0** has no prerequisites — it can go first. Once you "complete" it, remove its outgoing edges, which may reduce the in-degree of its neighbors to 0. Repeat.

```
In-degree = number of incoming edges to a node
```

### Steps

```
1. Compute in-degree for every node
2. Enqueue all nodes with in-degree = 0
3. While queue is not empty:
   a. Dequeue a node, add it to result
   b. For each neighbor of that node:
      - Decrement neighbor's in-degree by 1
      - If neighbor's in-degree becomes 0, enqueue it
4. If result contains all nodes → valid topological order
   If result is shorter than total nodes → cycle exists
```

### Trace Example

```
Graph:
  5 → 0
  5 → 2
  4 → 0
  4 → 1
  2 → 3
  3 → 1

In-degrees:
  0: 2  (from 5 and 4)
  1: 2  (from 4 and 3)
  2: 1  (from 5)
  3: 1  (from 2)
  4: 0  ← no prerequisites
  5: 0  ← no prerequisites

Step 1: queue = [4, 5]  (in-degree 0)

Step 2: pop 4, result=[4]
        neighbors of 4: 0 (in-degree 2→1), 1 (in-degree 2→1)
        neither hits 0, queue=[5]

Step 3: pop 5, result=[4,5]
        neighbors of 5: 0 (in-degree 1→0 ✓), 2 (in-degree 1→0 ✓)
        queue=[0, 2]

Step 4: pop 0, result=[4,5,0]
        neighbors of 0: none
        queue=[2]

Step 5: pop 2, result=[4,5,0,2]
        neighbors of 2: 3 (in-degree 1→0 ✓)
        queue=[3]

Step 6: pop 3, result=[4,5,0,2,3]
        neighbors of 3: 1 (in-degree 1→0 ✓)
        queue=[1]

Step 7: pop 1, result=[4,5,0,2,3,1]
        queue=[]

Result: [4, 5, 0, 2, 3, 1]  ✓  (all 6 nodes — no cycle)
```

---

## Algorithm 2: DFS-Based

### Intuition

Run DFS. When you finish exploring all descendants of a node (post-order), push it onto a stack. After all nodes are processed, pop the stack — that's the topological order.

Why does this work? A node is added to the stack only after all nodes it points to are already in the stack. Reversing gives the correct order.

### Steps

```
1. For each unvisited node, run DFS
2. After fully exploring a node (all its neighbors done), push it to a stack
3. Pop the stack at the end → topological order
```

### Trace Example

```
Same graph: 5→0, 5→2, 4→0, 4→1, 2→3, 3→1

DFS from 5:
  visit 5
    visit 0 (no unvisited neighbors) → push 0    stack=[0]
    visit 2
      visit 3
        visit 1 (no unvisited neighbors) → push 1  stack=[0,1]
        push 3                                      stack=[0,1,3]
      push 2                                        stack=[0,1,3,2]
    push 5                                          stack=[0,1,3,2,5]

DFS from 4 (0 and 1 already visited):
  visit 4 → push 4                                  stack=[0,1,3,2,5,4]

Pop stack: 4, 5, 2, 3, 1, 0

Result: [4, 5, 2, 3, 1, 0]  ✓
```

Both algorithms give valid (possibly different) topological orderings.

---

## Java Solution

```java
import java.util.*;

public class TopologicalSort {

    // ─────────────────────────────────────────────
    // Kahn's Algorithm (BFS)
    // Returns topological order, or empty list if cycle detected
    // ─────────────────────────────────────────────
    public static List<Integer> kahns(int numNodes, List<List<Integer>> adj) {
        int[] inDegree = new int[numNodes];

        // Count incoming edges for each node
        for (int u = 0; u < numNodes; u++)
            for (int v : adj.get(u))
                inDegree[v]++;

        // Start with all nodes that have no prerequisites
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numNodes; i++)
            if (inDegree[i] == 0) queue.offer(i);

        List<Integer> result = new ArrayList<>();

        while (!queue.isEmpty()) {
            int node = queue.poll();
            result.add(node);

            for (int neighbor : adj.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0)
                    queue.offer(neighbor);
            }
        }

        // If result doesn't contain all nodes, there's a cycle
        if (result.size() != numNodes) return Collections.emptyList();
        return result;
    }

    // ─────────────────────────────────────────────
    // DFS-based Topological Sort
    // Returns topological order, or empty list if cycle detected
    // ─────────────────────────────────────────────
    public static List<Integer> dfsBased(int numNodes, List<List<Integer>> adj) {
        // 0 = unvisited, 1 = in current DFS path, 2 = fully processed
        int[] state = new int[numNodes];
        Deque<Integer> stack = new ArrayDeque<>();
        boolean[] hasCycle = {false};

        for (int i = 0; i < numNodes; i++)
            if (state[i] == 0)
                dfs(i, adj, state, stack, hasCycle);

        if (hasCycle[0]) return Collections.emptyList();

        List<Integer> result = new ArrayList<>(stack);
        return result;
    }

    private static void dfs(int node, List<List<Integer>> adj,
                             int[] state, Deque<Integer> stack, boolean[] hasCycle) {
        if (hasCycle[0]) return;

        state[node] = 1; // mark as "currently visiting"

        for (int neighbor : adj.get(node)) {
            if (state[neighbor] == 1) {
                // Back edge — we're visiting a node already on the current path
                hasCycle[0] = true;
                return;
            }
            if (state[neighbor] == 0)
                dfs(neighbor, adj, state, stack, hasCycle);
        }

        state[node] = 2;   // fully processed
        stack.push(node);  // push AFTER all descendants are done
    }

    // ─────────────────────────────────────────────
    // Helper: build adjacency list from edge list
    // ─────────────────────────────────────────────
    static List<List<Integer>> buildGraph(int n, int[][] edges) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] e : edges) adj.get(e[0]).add(e[1]);
        return adj;
    }

    public static void main(String[] args) {
        // Graph: 5→0, 5→2, 4→0, 4→1, 2→3, 3→1
        int[][] edges = {{5,0},{5,2},{4,0},{4,1},{2,3},{3,1}};
        List<List<Integer>> adj = buildGraph(6, edges);

        System.out.println("Kahn's:  " + kahns(6, adj));
        // → [4, 5, 0, 2, 3, 1]

        System.out.println("DFS:     " + dfsBased(6, adj));
        // → [5, 4, 2, 3, 1, 0]  (valid, different ordering)

        // Cycle detection
        int[][] cycleEdges = {{0,1},{1,2},{2,0}};
        List<List<Integer>> cycleAdj = buildGraph(3, cycleEdges);
        System.out.println("Cycle:   " + kahns(3, cycleAdj));
        // → []  (empty = cycle detected)
    }
}
```

---

## Cycle Detection — How Each Algorithm Handles It

### Kahn's Algorithm
After processing, if `result.size() < numNodes`, some nodes were never enqueued — their in-degree never reached 0, meaning they're stuck in a cycle.

```
Cycle: A → B → C → A

In-degrees: A=1, B=1, C=1
No node starts with in-degree 0 → queue is empty from the start
Result = [] → cycle detected
```

### DFS-based
Track three states per node:
- `0` = unvisited
- `1` = currently on the DFS call stack (being explored)
- `2` = fully done

If you reach a neighbor with state `1`, you've found a **back edge** — a cycle.

```
Visiting A → B → C → A
When we try to visit A again, state[A] = 1 → back edge → cycle!
```

Using just visited/unvisited (boolean) is not enough for directed graphs — you need the three-state approach to distinguish "already fully processed" from "currently on the path."

---

## Classic Problem: Course Schedule

> Given `n` courses and a list of prerequisite pairs `[a, b]` meaning "must take b before a", determine if it's possible to finish all courses.

This is just: **does a valid topological order exist?** = **is the graph a DAG?**

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    for (int[] p : prerequisites) adj.get(p[1]).add(p[0]);

    List<Integer> order = TopologicalSort.kahns(numCourses, adj);
    return order.size() == numCourses; // empty = cycle = impossible
}
```

---

## Key Takeaways

- Topological order = linear ordering of a DAG respecting all edge directions
- Only possible on **Directed Acyclic Graphs** — cycles make it impossible
- **Kahn's (BFS)**: process zero-in-degree nodes first, reduce neighbors' in-degrees
- **DFS**: push nodes post-order onto a stack, pop for the result
- Both detect cycles: Kahn's via result size, DFS via back edges (three-state visited)
- Multiple valid orderings can exist — any is acceptable unless the problem specifies otherwise

# Day 4: Graph Basics

## Learning Objectives
By the end of Day 4, you should be able to:
- Understand what a graph is and its core terminology
- Represent graphs using adjacency lists and matrices
- Implement BFS and DFS traversals
- Detect cycles in undirected graphs

---

## 1. What is a Graph?

Graph is a non-linear data structure like tree data structure. A Graph is composed of a set of vertices(V) and a set of edges(E). The vertices are connected with each other through edges.

A graph is a collection of **nodes (vertices)** connected by **edges**.

The limitation of tree is, it can only represent hierarchical data. For situations where nodes or vertices are randomly connected with each other other, we use Graph.
Example situations where we use graph data structure are, a social network, a computer network, a network of locations used in GPS and many more examples where different nodes or vertices are connected without any hierarchic or constraint on structure.

```
    1 --- 2
    |     |
    3 --- 4
         |
         5
```

### Key Terminology

| Term | Meaning |
|------|---------|
| Vertex (Node) | A point in the graph |
| Edge | A connection between two vertices |
| Directed | Edges have direction (A → B) |
| Undirected | Edges go both ways (A — B) |
| Weighted | Edges have a cost/weight |
| Degree | Number of edges connected to a node |
| Path | Sequence of vertices connected by edges |
| Cycle | A path that starts and ends at the same vertex |
| Connected | Every node is reachable from every other node |

---

## 2. Types of Edges

### Undirected Edge
Connection goes both ways — if A connects to B, then B connects to A.
```
A ——— B        A can reach B, B can reach A
```

### Directed Edge
Connection goes one way only — an arrow shows the direction.
```
A ——→ B        A can reach B, but B cannot reach A
```

A graph made entirely of directed edges is called a **directed graph (digraph)**.

### Weighted Edge
Edge carries a cost, distance, or capacity.
```
A —5— B        traveling A→B costs 5
```
Can be combined with direction:
```
A —5→ B        directed AND weighted
```

### Self-Loop
An edge that connects a vertex to itself.
```
A ——→ A        A points to itself
```

### Multi-Edge (Parallel Edge)
Two or more edges connecting the same pair of vertices.
```
A ═══ B        two separate connections between A and B
```

### Summary

| Type | Symbol | Meaning |
|------|--------|---------|
| Undirected | A — B | Bidirectional connection |
| Directed | A → B | One-way connection |
| Weighted | A —5— B | Connection with a cost |
| Self-loop | A → A | Node points to itself |
| Multi-edge | A ═ B | Multiple connections between same nodes |

---

## 3. Sparse vs Dense Graphs

### Sparse Graph
A sparse graph has far fewer edges than the maximum possible. Max possible edges for `V` vertices = `V * (V-1) / 2`.

```
Sparse (few edges):

1 --- 2
|
3     4 --- 5

V=5, E=3  →  E << V²
```

Edges are close to `V` (linear). Most nodes connect to only a few others.

Real world examples:
- Road maps — a city connects to 3-4 roads, not every other city
- Social networks — you have hundreds of friends, not billions
- The internet — each webpage links to a few others, not all pages

**Use adjacency list** — space is O(V + E), so sparse graphs stay lean.

---

### Dense Graph
A dense graph has edges close to the maximum possible — most nodes are connected to most other nodes.

```
Dense (many edges):

1 --- 2
|\ /\ |
| X  \|
3 --- 4
 \   /
   5

V=5, E=9  →  E ≈ V²
```

Edges are close to `V²` (quadratic). Almost every pair of nodes has a connection.

Real world examples:
- Flight networks between major hubs — most airports connect to most others
- Complete communication networks — every device talks to every other device
- Correlation graphs in data science — every feature compared against every other

**Use adjacency matrix** — edge lookup is O(1) and the memory cost of O(V²) is justified since most cells are filled anyway.

---

### Why it matters

For a graph with 1000 nodes but only 1200 edges (sparse):
- Adjacency list → ~2200 slots used
- Adjacency matrix → 1,000,000 slots, most of them empty zeros

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| Space | O(V + E) | O(V²) |
| Edge lookup | O(degree) | O(1) |
| Sparse graph | ✓ efficient | ✗ wastes memory |
| Dense graph | okay | ✓ fast lookups |

Most real world graphs are sparse — adjacency list is the default choice.

---

## 4. Graph Representations

There are two main ways to store a graph in memory. The choice affects both space usage and how fast you can answer questions like "are A and B connected?" or "what are A's neighbors?".

We'll use this graph for both examples:
```
    1 --- 2
    |     |
    3 --- 4
          |
          5
```
Edges: 1-2, 1-3, 2-4, 3-4, 4-5

---

### Adjacency List (preferred for sparse graphs)

Each vertex maps to a list of its direct neighbors. You only store edges that actually exist.

```
1 → [2, 3]
2 → [1, 4]
3 → [1, 4]
4 → [2, 3, 5]
5 → [4]
```

Think of it as: "for each node, here's who it's connected to."

```java
import java.util.*;

// Undirected graph with 5 nodes
Map<Integer, List<Integer>> graph = new HashMap<>();
graph.put(1, Arrays.asList(2, 3));
graph.put(2, Arrays.asList(1, 4));
graph.put(3, Arrays.asList(1, 4));
graph.put(4, Arrays.asList(2, 3, 5));
graph.put(5, Arrays.asList(4));
```

To find neighbors of node 4: `graph.get(4)` → `[2, 3, 5]` — instant.
To check if edge 1-5 exists: scan node 1's list → `[2, 3]` — not there.

---

### Adjacency Matrix (preferred for dense graphs)

A 2D grid where `matrix[i][j] = 1` means there's an edge between node `i` and node `j`, and `0` means no edge.

```
     1  2  3  4  5
1  [ 0, 1, 1, 0, 0 ]
2  [ 1, 0, 0, 1, 0 ]
3  [ 1, 0, 0, 1, 0 ]
4  [ 0, 1, 1, 0, 1 ]
5  [ 0, 0, 0, 1, 0 ]
```

Think of it as: "every possible pair of nodes has a cell — 1 if connected, 0 if not."

```java
int n = 5;
int[][] matrix = new int[n + 1][n + 1];

// Add edge between 1 and 2 (undirected)
matrix[1][2] = 1;
matrix[2][1] = 1;
```

To check if edge 1-5 exists: `matrix[1][5]` → `0` — instant O(1).
To find all neighbors of node 4: scan entire row 4 → O(V).

For weighted graphs, store the weight instead of 1: `matrix[i][j] = weight`.

---

### Comparison

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| Space | O(V + E) | O(V²) |
| Edge lookup | O(degree) | O(1) |
| Neighbor iteration | O(degree) | O(V) |
| Best for | Sparse graphs | Dense graphs |

---

## 3. Building a Graph from Edge List

```java
import java.util.*;

public class Graph {

    private int vertices;
    private Map<Integer, List<Integer>> adjList;

    public Graph(int vertices) {
        this.vertices = vertices;
        adjList = new HashMap<>();
        for (int i = 0; i < vertices; i++) {
            adjList.put(i, new ArrayList<>());
        }
    }

    public void addEdge(int u, int v) {
        adjList.get(u).add(v);
        adjList.get(v).add(u); // remove this line for directed graph
    }

    public List<Integer> neighbors(int node) {
        return adjList.getOrDefault(node, new ArrayList<>());
    }
}
```

---

## 4. Breadth-First Search (BFS)

BFS explores all neighbors at the current depth before going deeper. Uses a **queue**.

### Use cases
- Shortest path in unweighted graphs
- Level-order traversal
- Finding all nodes at distance k

```java
import java.util.*;

public class BFS {

    public static void bfs(Map<Integer, List<Integer>> graph, int start) {
        Set<Integer> visited = new HashSet<>();
        Queue<Integer> queue = new LinkedList<>();

        visited.add(start);
        queue.offer(start);

        while (!queue.isEmpty()) {
            int node = queue.poll();
            System.out.print(node + " ");

            for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    queue.offer(neighbor);
                }
            }
        }
    }

    public static void main(String[] args) {
        Map<Integer, List<Integer>> graph = new HashMap<>();
        graph.put(0, Arrays.asList(1, 2));
        graph.put(1, Arrays.asList(0, 3, 4));
        graph.put(2, Arrays.asList(0));
        graph.put(3, Arrays.asList(1));
        graph.put(4, Arrays.asList(1));

        bfs(graph, 0); // 0 1 2 3 4
    }
}
```

**Graph structure:**
```
        0
       / \
      1   2
     / \
    3   4
```

`0` is directly connected to `1` and `2`. `3` and `4` are connected to `1`, not `0` — they are 2 hops away from `0`.

BFS visits them in order by distance from the start node:
```
visit 0  → enqueue neighbors: [1, 2]
visit 1  → enqueue neighbors: [3, 4]     (0 already visited, skip)
visit 2  → no new neighbors
visit 3  → no new neighbors
visit 4  → no new neighbors

Output: 0 1 2 3 4
```

`3` and `4` appear in the output not because `0` connects to them directly, but because BFS reached `1` first, and from `1` it discovered `3` and `4`.

---

**Example 2 — BFS on a wider graph:**

```java
Map<Integer, List<Integer>> graph2 = new HashMap<>();
graph2.put(0, Arrays.asList(1, 2, 3));
graph2.put(1, Arrays.asList(0, 4));
graph2.put(2, Arrays.asList(0, 4, 5));
graph2.put(3, Arrays.asList(0));
graph2.put(4, Arrays.asList(1, 2));
graph2.put(5, Arrays.asList(2));

bfs(graph2, 0); // 0 1 2 3 4 5
```

**Graph structure:**
```
        0
      / | \
     1  2  3
     \ / \
      4   5
```

BFS trace starting from `0`:
```
visit 0  → enqueue: [1, 2, 3]
visit 1  → enqueue: [4]          (0 already visited, skip)
visit 2  → enqueue: [5]          (0 and 4 — 4 already queued, skip)
visit 3  → no new neighbors      (0 already visited)
visit 4  → no new neighbors      (1 and 2 already visited)
visit 5  → no new neighbors      (2 already visited)

Output: 0 1 2 3 4 5
```

Notice `3` is visited before `4` and `5` even though `4` connects to two nodes — BFS always finishes the current level before going deeper.
```

---

## 5. Depth-First Search (DFS)

DFS explores as far as possible along a branch before backtracking. Uses a **stack** (or recursion).

### Use cases
- Cycle detection
- Topological sort
- Connected components
- Maze solving

```java
import java.util.*;

public class DFS {

    // Recursive DFS
    public static void dfs(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
        visited.add(node);
        System.out.print(node + " ");

        for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
            if (!visited.contains(neighbor)) {
                dfs(graph, neighbor, visited);
            }
        }
    }

    // Iterative DFS
    public static void dfsIterative(Map<Integer, List<Integer>> graph, int start) {
        Set<Integer> visited = new HashSet<>();
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(start);

        while (!stack.isEmpty()) {
            int node = stack.pop();
            if (visited.contains(node)) continue;
            visited.add(node);
            System.out.print(node + " ");

            for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
                if (!visited.contains(neighbor)) {
                    stack.push(neighbor);
                }
            }
        }
    }

    public static void main(String[] args) {
        Map<Integer, List<Integer>> graph = new HashMap<>();
        graph.put(0, Arrays.asList(1, 2));
        graph.put(1, Arrays.asList(0, 3, 4));
        graph.put(2, Arrays.asList(0));
        graph.put(3, Arrays.asList(1));
        graph.put(4, Arrays.asList(1));

        Set<Integer> visited = new HashSet<>();
        dfs(graph, 0, visited); // 0 1 3 4 2
    }
}
```

---

## 6. BFS vs DFS

| | BFS | DFS |
|---|---|---|
| Data structure | Queue | Stack / Recursion |
| Traversal order | Level by level | Branch by branch |
| Shortest path | Yes (unweighted) | No |
| Memory | More (stores all neighbors) | Less (stores one path) |
| Best for | Shortest path, level order | Cycle detection, topological sort |

---

## 7. Cycle Detection in Undirected Graph

### What is a cycle?

A cycle exists when you can start at a node, follow edges, and return to the same node without retracing any edge.

```
Cycle:              No cycle (tree):

0 --- 1             0 --- 1
|     |             |
2 --- 3             2 --- 3
                          |
                          4
```

In the left graph, you can go `0 → 1 → 3 → 2 → 0` — that's a cycle.
In the right graph, every path is a dead end — no way back to the start.

---

### Core idea

Use DFS and track two things for every node:
1. **visited** — have we seen this node before?
2. **parent** — which node did we come from?

If during DFS we reach a neighbor that is already visited AND it's not our parent, we've found a back edge — proof of a cycle.

```
Rule:
  neighbor is visited  AND  neighbor != parent  →  CYCLE
  neighbor is visited  AND  neighbor == parent  →  just the edge we came from, skip it
```

---

### Why the parent check?

In an undirected graph, every edge goes both ways. So when DFS is at node `1` and looks at its neighbors, it will see node `0` (where it just came from). That's not a cycle — it's just the edge we used to get here.

```
0 --- 1 --- 2
```

DFS from `0`:
- visit `0`, go to neighbor `1`
- visit `1`, neighbors are `[0, 2]`
  - `0` is visited — but `0` is the **parent**, so skip it
  - go to `2`
- visit `2`, neighbors are `[1]`
  - `1` is visited — but `1` is the **parent**, so skip it
- done — no cycle

The rule: if a neighbor is already visited **and** it's not the parent, we found a back edge — that's a cycle.

---

### Step-by-step trace (with cycle)

```
Graph:
0 --- 1
|     |
2 --- 3
```

Edges: 0-1, 0-2, 1-3, 2-3

DFS from `0` (parent = -1):

```
Step 1: visit 0  (parent=-1)  visited={0}
        neighbors of 0: [1, 2]

Step 2:   visit 1  (parent=0)  visited={0,1}
          neighbors of 1: [0, 3]
          → 0 is visited, but 0 == parent → skip

Step 3:     visit 3  (parent=1)  visited={0,1,3}
            neighbors of 3: [1, 2]
            → 1 is visited, but 1 == parent → skip
            → 2 is visited, and 2 != parent (1) → CYCLE FOUND ✓
```

Node `2` was already visited and it's not `3`'s parent — that back edge `3→2` closes the loop `0 → 2 → 3 → 1 → 0`.

---

### Step-by-step trace (no cycle)

```
Graph:
0 --- 1 --- 3
|
2
```

Edges: 0-1, 0-2, 1-3

DFS from `0` (parent = -1):

```
Step 1: visit 0  (parent=-1)  visited={0}
        neighbors of 0: [1, 2]

Step 2:   visit 1  (parent=0)  visited={0,1}
          neighbors of 1: [0, 3]
          → 0 is visited, but 0 == parent → skip

Step 3:     visit 3  (parent=1)  visited={0,1,3}
            neighbors of 3: [1]
            → 1 is visited, but 1 == parent → skip
            → no more neighbors, backtrack

Step 4:   back to 0, visit 2  (parent=0)  visited={0,1,3,2}
          neighbors of 2: [0]
          → 0 is visited, but 0 == parent → skip
          → no more neighbors, backtrack

Result: no cycle found ✓
```

Every visited neighbor was the parent — no back edges, no cycle.

---

### Call stack visualization (cycle case)

```
dfs(0, parent=-1)
  dfs(1, parent=0)
    dfs(3, parent=1)
      neighbor 2 is visited AND 2 != parent(1)
      → return true  ← cycle detected here
    ← true bubbles up
  ← true bubbles up
← true bubbles up → hasCycle returns true
```

The `true` return value propagates all the way back up the call stack as soon as the cycle is found — no need to keep searching.

---

### Using DFS — if we visit a node that's already visited and it's not the parent, there's a cycle.

```java
import java.util.*;

public class CycleDetection {

    public static boolean hasCycle(Map<Integer, List<Integer>> graph, int nodes) {
        Set<Integer> visited = new HashSet<>();

        for (int i = 0; i < nodes; i++) {
            if (!visited.contains(i)) {
                if (dfs(graph, i, -1, visited)) return true;
            }
        }
        return false;
    }

    private static boolean dfs(Map<Integer, List<Integer>> graph, int node, int parent, Set<Integer> visited) {
        visited.add(node);

        for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
            if (!visited.contains(neighbor)) {
                if (dfs(graph, neighbor, node, visited)) return true;
            } else if (neighbor != parent) {
                // neighbor is already visited — but WHY do we check != parent?
                //
                // In an undirected graph, every edge A-B is stored in BOTH directions:
                //   A's list contains B
                //   B's list contains A
                //
                // So when we arrive at node B from A, and we loop over B's neighbors,
                // we will always see A in that list. A is already visited — but that
                // doesn't mean there's a cycle. It's just the edge we came from.
                //
                // Example (no cycle):
                //   0 --- 1 --- 2
                //   DFS: visit 0 → visit 1 (parent=0) → see neighbor 0
                //   0 is visited, but 0 == parent → NOT a cycle, skip it
                //
                // Example (cycle):
                //   0 --- 1
                //   |     |
                //   2 --- 3
                //   DFS: visit 3 (parent=1) → see neighbor 2
                //   2 is visited, and 2 != parent (1) → real back edge → CYCLE
                //
                // So the check is: "is this visited neighbor someone OTHER than
                // the node I just came from?" If yes → back edge → cycle.
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Map<Integer, List<Integer>> g1 = new HashMap<>();
        g1.put(0, Arrays.asList(1, 2));
        g1.put(1, Arrays.asList(0, 2));
        g1.put(2, Arrays.asList(0, 1));
        System.out.println(hasCycle(g1, 3)); // true

        Map<Integer, List<Integer>> g2 = new HashMap<>();
        g2.put(0, Arrays.asList(1));
        g2.put(1, Arrays.asList(0, 2));
        g2.put(2, Arrays.asList(1));
        System.out.println(hasCycle(g2, 3)); // false
    }
}
```

---

## 8. Exercises

### Exercise 1: Number of Connected Components
Given `n` nodes and a list of edges, return the number of connected components.
```
Input: n=5, edges=[[0,1],[1,2],[3,4]]
Output: 2
```

### Exercise 2: Find All Paths from Source to Target
Given a directed acyclic graph, find all paths from node 0 to node n-1.
```
Input: graph = [[1,2],[3],[3],[]]
Output: [[0,1,3],[0,2,3]]
```

### Exercise 3: BFS Shortest Path
Given an unweighted undirected graph, find the shortest path length between two nodes. Return -1 if no path exists.
```
Input: n=6, edges=[[0,1],[0,2],[1,3],[2,4],[3,5],[4,5]], src=0, dst=5
Output: 3
```

---

## Key Takeaways

- Graphs model relationships — nodes are entities, edges are connections
- Adjacency list is the go-to representation for most problems
- BFS = queue, shortest path in unweighted graphs
- DFS = recursion/stack, cycle detection, path finding
- Always track visited nodes to avoid infinite loops

---

## Day 4 Checklist
- [ ] Can represent a graph as an adjacency list
- [ ] Implemented BFS and traced the traversal order
- [ ] Implemented DFS both recursively and iteratively
- [ ] Can detect a cycle in an undirected graph
- [ ] Understand when to use BFS vs DFS

DP programs
Graph algorithms
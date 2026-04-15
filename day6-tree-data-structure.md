# Day 6: Tree Data Structure

## Learning Objectives
By the end of Day 6, you should be able to:
- Understand what a tree is and its core terminology
- Implement a binary tree and binary search tree (BST)
- Traverse a tree using inorder, preorder, and postorder
- Implement BFS (level-order) traversal on a tree
- Solve common tree problems using recursion

---

## 1. What is a Tree?

A tree is a hierarchical data structure made of **nodes**, where each node has a value and zero or more **children**. Unlike graphs, trees have no cycles and have a single root.

```
          1          ← root
        /   \
       2     3
      / \     \
     4   5     6
```

### Key Terminology

| Term | Meaning |
|------|---------|
| Root | The topmost node (no parent) |
| Leaf | A node with no children |
| Parent | A node that has children |
| Child | A node connected below a parent |
| Sibling | Nodes sharing the same parent |
| Height | Longest path from root to a leaf |
| Depth | Distance from root to a given node |
| Subtree | A node and all its descendants |
| Level | Depth + 1 (root is level 1) |

---

## 2. Binary Tree

A **binary tree** is a tree where each node has at most **two children**: left and right.

### Node Definition

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int val) {
        this.val = val;
    }
}
```

### Building a Tree Manually

```java
public class BinaryTree {
    public static void main(String[] args) {
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.left.left = new TreeNode(4);
        root.left.right = new TreeNode(5);
        root.right.right = new TreeNode(6);

        /*
                  1
                /   \
               2     3
              / \     \
             4   5     6
        */
    }
}
```

---

## 3. Tree Traversals

Traversal means visiting every node exactly once. There are four main ways to traverse a binary tree.

We'll use this tree for all four examples:

```
        1
       / \
      2   3
     / \
    4   5
```

---

### Inorder (Left → Root → Right)

**How it works:** Recursively go as far left as possible, print the node, then explore the right. Because of this order, inorder traversal on a BST always produces values in ascending sorted order — that's its most important property.

Think of it as: "finish everything on my left, then process me, then handle my right."

**Step-by-step on the tree above:**
```
Visit 1 → go left to 2 → go left to 4
  4 has no left → print 4
  4 has no right → done with 4
back at 2 → print 2
  go right to 5 → print 5
back at 1 → print 1
  go right to 3 → print 3

Output: 4 → 2 → 5 → 1 → 3
```

```java
public static void inorder(TreeNode node) {
    if (node == null) return;   // base case: nothing to visit
    inorder(node.left);         // 1. recurse left
    System.out.print(node.val + " "); // 2. visit current node
    inorder(node.right);        // 3. recurse right
}
```

**When to use:** Reading a BST in sorted order, validating a BST.

---

### Preorder (Root → Left → Right)

**How it works:** Print the current node first, then recurse into the left subtree, then the right. The root is always the first thing printed — this mirrors how you'd naturally describe a tree top-down.

Think of it as: "process me first, then go explore."

**Step-by-step on the tree above:**
```
Visit 1 → print 1
  go left to 2 → print 2
    go left to 4 → print 4
      no children → done with 4
    go right to 5 → print 5
      no children → done with 5
  go right to 3 → print 3
    no children → done with 3

Output: 1 → 2 → 4 → 5 → 3
```

```java
public static void preorder(TreeNode node) {
    if (node == null) return;        // base case
    System.out.print(node.val + " "); // 1. visit current node
    preorder(node.left);             // 2. recurse left
    preorder(node.right);            // 3. recurse right
}
```

**When to use:** Copying a tree, serializing a tree to a file, printing directory structures.

---

### Postorder (Left → Right → Root)

**How it works:** Recurse into both children first, then process the current node last. A node is only visited after all of its descendants have been visited.

Think of it as: "handle all my children before you deal with me."

**Step-by-step on the tree above:**
```
go left to 2
  go left to 4
    no children → print 4
  go right to 5
    no children → print 5
  both children done → print 2
go right to 3
  no children → print 3
both children done → print 1

Output: 4 → 5 → 2 → 3 → 1
```

```java
public static void postorder(TreeNode node) {
    if (node == null) return;        // base case
    postorder(node.left);            // 1. recurse left
    postorder(node.right);           // 2. recurse right
    System.out.print(node.val + " "); // 3. visit current node
}
```

**When to use:** Deleting a tree (delete children before parent), evaluating expression trees (evaluate operands before operator).

---

### Level-Order / BFS (Level by Level)

**How it works:** Unlike the three DFS traversals above, level-order doesn't use recursion — it uses a **queue**. Start by enqueuing the root. Then repeatedly dequeue a node, print it, and enqueue its children. Because a queue is FIFO, all nodes at depth d are processed before any node at depth d+1.

Think of it as: "visit everyone on my floor before going to the next floor."

**Step-by-step on the tree above:**
```
Queue: [1]
  dequeue 1 → print 1, enqueue 2 and 3
Queue: [2, 3]
  dequeue 2 → print 2, enqueue 4 and 5
Queue: [3, 4, 5]
  dequeue 3 → print 3, no children
Queue: [4, 5]
  dequeue 4 → print 4, no children
Queue: [5]
  dequeue 5 → print 5, no children
Queue: []  → done

Output: 1 → 2 → 3 → 4 → 5
```

```java
import java.util.*;

public static void levelOrder(TreeNode root) {
    if (root == null) return;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);                          // start with the root

    while (!queue.isEmpty()) {
        TreeNode node = queue.poll();           // dequeue front node
        System.out.print(node.val + " ");       // visit it

        if (node.left != null) queue.offer(node.left);   // enqueue left child
        if (node.right != null) queue.offer(node.right); // enqueue right child
    }
}
```

**When to use:** Finding the shortest path, printing a tree level by level, finding nodes at a specific depth.

---

### Traversal Summary

| Traversal | Order | Use Case |
|-----------|-------|----------|
| Inorder | Left → Root → Right | Sorted output from BST |
| Preorder | Root → Left → Right | Copy / serialize tree |
| Postorder | Left → Right → Root | Delete tree, evaluate expressions |
| Level-order | Level by level | Shortest path, level grouping |

---

## 4. Binary Search Tree (BST)

A BST is a binary tree with one extra rule:
- All values in the **left** subtree are **less than** the root
- All values in the **right** subtree are **greater than** the root
- This holds recursively for every subtree

```
        5
       / \
      3   7
     / \ / \
    2  4 6  8
```

### Insert into BST

```java
public static TreeNode insert(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);  // found the spot

    if (val < root.val) {
        root.left = insert(root.left, val);       // go left
    } else if (val > root.val) {
        root.right = insert(root.right, val);     // go right
    }
    // if val == root.val, ignore duplicates
    return root;
}
```

### Search in BST

```java
public static boolean search(TreeNode root, int target) {
    if (root == null) return false;               // not found
    if (root.val == target) return true;          // found

    if (target < root.val) return search(root.left, target);
    return search(root.right, target);
}
```

Time Complexity: O(h) where h = height of tree
- Balanced BST: O(log n)
- Skewed BST: O(n)

---

## 5. Common Tree Problems

### Height of a Tree
The height is the number of edges on the longest path from root to a leaf.

```java
public static int height(TreeNode root) {
    if (root == null) return -1;                  // empty tree has height -1
    int leftH = height(root.left);
    int rightH = height(root.right);
    return 1 + Math.max(leftH, rightH);
}
```

---

### Count Nodes

```java
public static int countNodes(TreeNode root) {
    if (root == null) return 0;
    return 1 + countNodes(root.left) + countNodes(root.right);
}
```

---

### Sum of All Nodes

```java
public static int sumNodes(TreeNode root) {
    if (root == null) return 0;
    return root.val + sumNodes(root.left) + sumNodes(root.right);
}
```

---

### Check if Two Trees are Identical

```java
public static boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) return true;
    if (p == null || q == null) return false;
    return p.val == q.val
        && isSameTree(p.left, q.left)
        && isSameTree(p.right, q.right);
}
```

---

### Check if Tree is Symmetric (Mirror)

```java
public static boolean isSymmetric(TreeNode root) {
    return isMirror(root, root);
}

private static boolean isMirror(TreeNode left, TreeNode right) {
    if (left == null && right == null) return true;
    if (left == null || right == null) return false;
    return left.val == right.val
        && isMirror(left.left, right.right)
        && isMirror(left.right, right.left);
}
```

---

## 6. Dry Run: Inorder Traversal

**Tree:**
```
        4
       / \
      2   6
     / \ / \
    1  3 5  7
```

**Call trace for `inorder(4)`:**
```
inorder(4)
  └── inorder(2)
        └── inorder(1)
              └── inorder(null) → return
              print 1
              └── inorder(null) → return
        print 2
        └── inorder(3)
              └── inorder(null) → return
              print 3
              └── inorder(null) → return
  print 4
  └── inorder(6)
        └── inorder(5)
              └── inorder(null) → return
              print 5
              └── inorder(null) → return
        print 6
        └── inorder(7)
              └── inorder(null) → return
              print 7
              └── inorder(null) → return

Output: 1 2 3 4 5 6 7  ← sorted! (BST property)
```

---

## 7. Practice Problems

### Problem 1: Maximum Depth of Binary Tree (LeetCode 104)
Return the maximum depth (number of nodes along the longest path from root to leaf).

- Input: `[3,9,20,null,null,15,7]`
- Output: `3`

```java
public class MaxDepth {
    public static int maxDepth(TreeNode root) {
        if (root == null) return 0;
        return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(3);
        root.left = new TreeNode(9);
        root.right = new TreeNode(20);
        root.right.left = new TreeNode(15);
        root.right.right = new TreeNode(7);

        System.out.println(maxDepth(root)); // 3
    }
}
```

---

### Problem 2: Invert Binary Tree (LeetCode 226)
Flip the tree so left and right children are swapped at every node.

- Input: `[4,2,7,1,3,6,9]`
- Output: `[4,7,2,9,6,3,1]`

```java
public class InvertTree {
    public static TreeNode invertTree(TreeNode root) {
        if (root == null) return null;

        // swap children
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;

        // recurse on both subtrees
        invertTree(root.left);
        invertTree(root.right);

        return root;
    }
}
```

---

### Problem 3: Path Sum (LeetCode 112)
Return true if there exists a root-to-leaf path whose node values sum to `targetSum`.

- Input: root = `[5,4,8,11,null,13,4,7,2]`, targetSum = 22
- Output: `true` (path: 5 → 4 → 11 → 2)

```java
public class PathSum {
    public static boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) return false;

        // leaf node — check if remaining sum equals this node's value
        if (root.left == null && root.right == null) {
            return root.val == targetSum;
        }

        int remaining = targetSum - root.val;
        return hasPathSum(root.left, remaining) || hasPathSum(root.right, remaining);
    }
}
```

---

### Problem 4: Level Order Traversal (LeetCode 102)
Return node values grouped by level as a list of lists.

- Input: `[3,9,20,null,null,15,7]`
- Output: `[[3],[9,20],[15,7]]`

```java
import java.util.*;

public class LevelOrderGrouped {
    public static List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int levelSize = queue.size();          // snapshot of current level
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }
        return result;
    }
}
```

---

## 8. Exercises

### Exercise 1: Diameter of Binary Tree
The diameter is the length of the longest path between any two nodes (may not pass through root).

```
Input:
        1
       / \
      2   3
     / \
    4   5

Output: 3  (path: 4 → 2 → 1 → 3  or  5 → 2 → 1 → 3)
```

**Hint:** At each node, diameter through it = height(left) + height(right) + 2. Track the global max.

```java
public class DiameterOfTree {

    private static int maxDiameter = 0;

    public static int diameterOfBinaryTree(TreeNode root) {
        maxDiameter = 0;
        height(root);
        return maxDiameter;
    }

    private static int height(TreeNode node) {
        if (node == null) return -1;
        int leftH = height(node.left);
        int rightH = height(node.right);
        maxDiameter = Math.max(maxDiameter, leftH + rightH + 2);
        return 1 + Math.max(leftH, rightH);
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.left.left = new TreeNode(4);
        root.left.right = new TreeNode(5);

        System.out.println(diameterOfBinaryTree(root)); // 3
    }
}
```

---

### Exercise 2: Lowest Common Ancestor (LCA)
Given a BST and two nodes p and q, find their lowest common ancestor.

```
BST:        6
           / \
          2   8
         / \ / \
        0  4 7  9
          / \
         3   5

LCA(2, 8) = 6
LCA(2, 4) = 2
```

```java
public class LCAinBST {
    public static TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null) return null;

        // both values are smaller → LCA is in left subtree
        if (p.val < root.val && q.val < root.val) {
            return lowestCommonAncestor(root.left, p, q);
        }
        // both values are larger → LCA is in right subtree
        if (p.val > root.val && q.val > root.val) {
            return lowestCommonAncestor(root.right, p, q);
        }
        // they split here (or one equals root) → root is the LCA
        return root;
    }
}
```

---

### Exercise 3: Validate Binary Search Tree (LeetCode 98)
Return true if the tree is a valid BST.

```
Valid:      5          Invalid:    5
           / \                    / \
          3   7                  3   7
         / \                    / \
        2   4                  2   6   ← 6 > 5, violates BST
```

```java
public class ValidateBST {
    public static boolean isValidBST(TreeNode root) {
        return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    private static boolean validate(TreeNode node, long min, long max) {
        if (node == null) return true;
        if (node.val <= min || node.val >= max) return false;

        return validate(node.left, min, node.val)
            && validate(node.right, node.val, max);
    }
}
```

---

## 9. DFS vs BFS on Trees

### DFS (Depth-First Search)

DFS explores as deep as possible down one branch before backtracking. On trees, inorder/preorder/postorder are all forms of DFS. You can implement DFS **recursively** (using the call stack) or **iteratively** (using an explicit stack).

**Iterative DFS (Preorder) using a Stack:**

```java
import java.util.*;

public static void dfsIterative(TreeNode root) {
    if (root == null) return;

    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);

    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();           // visit the top node
        System.out.print(node.val + " ");

        // push right first so left is processed first (LIFO)
        if (node.right != null) stack.push(node.right);
        if (node.left != null) stack.push(node.left);
    }
}
```

**Step-by-step on:**
```
        1
       / \
      2   3
     / \
    4   5
```
```
stack: [1]
  pop 1 → print 1, push 3, push 2
stack: [3, 2]
  pop 2 → print 2, push 5, push 4
stack: [3, 5, 4]
  pop 4 → print 4, no children
stack: [3, 5]
  pop 5 → print 5, no children
stack: [3]
  pop 3 → print 3, no children

Output: 1 → 2 → 4 → 5 → 3  (same as recursive preorder)
```

**When to use DFS:**
- Finding a path between two nodes
- Detecting cycles
- Topological sort
- Solving maze/puzzle problems
- When you need to explore all possibilities (backtracking)

---

### BFS (Breadth-First Search)

BFS explores all nodes at the current depth before going deeper. It always uses a **queue** (FIFO). On trees, this is the same as level-order traversal.

**BFS using a Queue:**

```java
import java.util.*;

public static void bfs(TreeNode root) {
    if (root == null) return;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        TreeNode node = queue.poll();          // dequeue front node
        System.out.print(node.val + " ");

        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}
```

**Step-by-step on the same tree:**
```
queue: [1]
  poll 1 → print 1, enqueue 2, 3
queue: [2, 3]
  poll 2 → print 2, enqueue 4, 5
queue: [3, 4, 5]
  poll 3 → print 3, no children
queue: [4, 5]
  poll 4 → print 4, no children
queue: [5]
  poll 5 → print 5, no children

Output: 1 → 2 → 3 → 4 → 5  (level by level)
```

**When to use BFS:**
- Finding the **shortest path** (BFS guarantees shortest in unweighted graphs/trees)
- Printing nodes level by level
- Finding nodes at a specific depth
- Checking if a tree is complete

---

### DFS vs BFS — Quick Comparison

| | DFS | BFS |
|---|---|---|
| Data structure | Stack (or recursion) | Queue |
| Order | Deep first | Level by level |
| Memory | O(h) — tree height | O(w) — max width of tree |
| Shortest path | ✗ Not guaranteed | ✓ Guaranteed (unweighted) |
| Best for | Path finding, backtracking | Shortest path, level queries |

> Rule of thumb: if the answer is likely **near the root**, use BFS. If it's likely **deep in the tree**, use DFS.

---

## 10. Key Takeaways

- Trees are hierarchical — one root, no cycles, each node has at most one parent
- Binary trees have at most two children per node
- BST property: left < root < right (enables O(log n) search on balanced trees)
- Inorder traversal of a BST gives sorted output
- Most tree problems are solved with recursion — trust that the recursive call handles the subtree correctly
- BFS (level-order) uses a queue; DFS traversals use recursion or a stack

---

## Day 6 Checklist
- [ ] Can define a TreeNode class and build a tree manually
- [ ] Implemented all four traversals (inorder, preorder, postorder, level-order)
- [ ] Can insert and search in a BST
- [ ] Solved height, count, and path sum problems recursively
- [ ] Understand the difference between height and depth
- [ ] Can validate a BST using min/max bounds
- [ ] Implemented iterative DFS using a stack
- [ ] Implemented BFS using a queue
- [ ] Know when to choose DFS vs BFS

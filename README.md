# lecture disjoint set and suffix array:
Let’s dive into **disjoint sets** (also known as **union-find data structures**) in extreme detail, breaking everything down to the basics. I’ll explain what they are, why we need them, how they work, their implementations, optimizations, and applications, referencing the lecture slides "Lec10 Disjoint Set & Suffix Array.pdf" for clarity. Since you mentioned not understanding anything, I’ll start from the ground up and ensure every concept is clear, using examples and step-by-step explanations.

---

### **What Are Disjoint Sets?**

A **disjoint set** is a data structure that keeps track of a collection of elements partitioned into non-overlapping (disjoint) subsets. Each subset represents a group of elements that are considered "equivalent" or "connected" based on some relation. The key idea is that:
- Subsets are **disjoint**: No element belongs to more than one subset.
- The data structure supports operations to:
  - **Merge** two subsets into one (Union).
  - **Check** if two elements belong to the same subset (Find).

In the lecture slides:
- **Page 68** defines the **Disjoint Set ADT** (Abstract Data Type) with two main operations:
  - **Find(a)**: Returns the identifier (name) of the subset (equivalence class) that element `a` belongs to.
  - **Union(a, b)**: Merges the subsets containing elements `a` and `b` into a single subset.

#### **Why Do We Need Disjoint Sets?**
Disjoint sets are used to solve problems involving **grouping** or **connectivity**. They help answer questions like:
- Are two elements connected or related?
- How can we group related elements together efficiently?

**Real-World Analogy**:
Imagine you’re organizing a party and want to group friends into cliques where each clique consists of friends who know each other. Initially, each person is in their own clique (a singleton set). As you learn that two people are friends, you merge their cliques. Later, you might want to check if two people are in the same clique (i.e., connected through a chain of friendships). Disjoint sets help manage these groups efficiently.

**Applications** (referenced in the slides):
- **Maze Generation** (Page 72-73): Disjoint sets are used to generate a maze by breaking walls between cells, ensuring all cells are eventually connected (one path from entrance to exit).
- **Equivalence Relations** (Page 64-66): Determine if elements are equivalent under a relation (e.g., "same country" on a world map).
- **Graph Problems**: Find connected components in a graph (e.g., in social networks, are two people connected through mutual friends?).
- **Kruskal’s Algorithm** for Minimum Spanning Tree: Avoid cycles by checking if adding an edge connects two vertices already in the same component.

---

### **Understanding Equivalence Relations (The Motivation)**

Disjoint sets are often used to manage **equivalence relations**, which are the foundation for grouping elements. Let’s break this down (refer to **Page 64-66** in the slides).

#### **What Is an Equivalence Relation?**
An **equivalence relation** `~` on a set `S` is a relation between elements that satisfies three properties:
1. **Reflexive**: For every element `a` in `S`, `a ~ a` (every element is related to itself).
2. **Symmetric**: If `a ~ b`, then `b ~ a` (the relation works both ways).
3. **Transitive**: If `a ~ b` and `b ~ c`, then `a ~ c` (if `a` is related to `b`, and `b` to `c`, then `a` is related to `c`).

**Example** (Page 64):
- Consider the relation "same country" on a world map. If person A and person B are in the same country, and person B and person C are in the same country, then A and C are in the same country. This relation is:
  - Reflexive: A person is in the same country as themselves.
  - Symmetric: If A is in the same country as B, B is in the same country as A.
  - Transitive: If A and B are in the same country, and B and C are, then A and C are.

#### **Equivalence Classes**
An **equivalence class** is a subset of elements that are all equivalent to each other under the relation (Page 66). Because of the properties above:
- Equivalence classes are **disjoint**: No element can be in two different equivalence classes.
- The union of all equivalence classes covers the entire set `S`.

**Example** (Page 66 Exercise):
- Elements: `a, b, c, d, e, f, g, h, i, j, k`
- Relations: `a ~ b, b ~ c, b ~ d, e ~ f, g ~ h, i ~ e, j ~ k, k ~ c`
- Let’s compute the equivalence classes:
  - Start with `a ~ b`: `a` and `b` are in the same class.
  - `b ~ c`: Add `c` to the class with `a` and `b` (by transitivity, `a ~ c`).
  - `b ~ d`: Add `d` to the class (`a, b, c, d`).
  - `k ~ c`: Add `k` to the class (`a, b, c, d, k`).
  - `j ~ k`: Add `j` to the class (`a, b, c, d, k, j`).
  - `e ~ f`: New class (`e, f`).
  - `i ~ e`: Add `i` to the class (`e, f, i`).
  - `g ~ h`: New class (`g, h`).
- Final equivalence classes:
  1. `{a, b, c, d, j, k}`
  2. `{e, f, i}`
  3. `{g, h}`
- Number of classes: 3.

Disjoint sets help manage these classes by allowing us to merge classes (Union) and check if two elements are in the same class (Find).

---

### **Disjoint Set Operations**

The disjoint set data structure supports two primary operations (Page 68):
1. **Find(a)**: Returns the identifier (or "name") of the subset containing element `a`. If `Find(a) == Find(b)`, then `a` and `b` are in the same subset.
2. **Union(a, b)**: Merges the subsets containing `a` and `b` into a single subset.

**Precondition and Postcondition for Union** (Page 68):
- Suppose `Find(a) == Find(c)` and `Find(b) == Find(d)`.
- After `Union(a, b)`, `a, b, c, d` should all be in the same subset (i.e., `Find(a) == Find(b) == Find(c) == Find(d)`).

---

### **How Disjoint Sets Work: Implementations**

There are two main ways to implement disjoint sets: using an **array** or a **tree**. Let’s explore both in detail, starting with the simpler array implementation, then moving to the more efficient tree implementation, as shown in the slides.

#### **1. Array Implementation (Page 69-70)**

**Idea**:
- Use an array `s[]` where `s[i]` stores the **equivalence class name** (or identifier) of element `i`.
- Initially, each element is in its own subset, so `s[i] = i`.

**Example Setup** (Page 69):
- Elements: `0, 1, 2, ..., 9`
- Initially: `s = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`
  - `s[0] = 0`: Element 0 is in subset named "0".
  - `s[1] = 1`: Element 1 is in subset named "1".
  - And so on.

**Operations**:
- **Find(i)**: Simply return `s[i]`. Time complexity: **O(1)**.
  - Example: `Find(1)` returns `s[1]`.
- **Union(i, j)**: Merge the subsets containing `i` and `j` by updating all elements in `j`’s subset to have the same name as `i`’s subset.
  - Steps:
    1. Check if `Find(i) != Find(j)` (if they’re already in the same subset, do nothing).
    2. Get the name of `j`’s subset: `temp = s[j]`.
    3. Scan the array and update all elements whose name is `temp` to `s[i]`.
  - Time complexity: **O(N)**, where `N` is the number of elements (we may need to scan the entire array).

**Example** (Page 69):
- Initial array: `s = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`
- **Union(1, 2)**:
  - `Find(1) = s[1] = 1`, `Find(2) = s[2] = 2`. They’re different, so proceed.
  - `temp = s[2] = 2`.
  - Update all elements where `s[k] == 2` to `s[1] = 1`.
  - New array: `s = [0, 1, 1, 3, 4, 5, 6, 7, 8, 9]`.
- **Union(3, 1)**:
  - `Find(3) = s[3] = 3`, `Find(1) = s[1] = 1`. They’re different.
  - `temp = s[1] = 1`.
  - Update all elements where `s[k] == 1` to `s[3] = 3`.
  - New array: `s = [0, 3, 3, 3, 4, 5, 6, 7, 8, 9]`.
- **Find(1)**: Returns `s[1] = 3`.

**Code** (Page 70):
```cpp
class DisjointSet {
public:
    int Find(int i) { return name[i]; }
    void Union(int i, int j) {
        if (Find(i) != Find(j)) {
            int temp = name[j];
            for (int k = 0; k < MAX_SIZE; k++) {
                if (name[k] == temp)
                    name[k] = name[i];
            }
        }
    }
private:
    int name[MAX_SIZE];
};
```

**Pros**:
- `Find` is very fast: O(1).
- Simple to implement.

**Cons**:
- `Union` is slow: O(N) because we need to scan the entire array to update names.
- Inefficient for large datasets or many `Union` operations.

#### **2. Tree Implementation (Page 74-78)**

The array implementation is inefficient for `Union`, so we use a **tree-based implementation**, where each subset is represented as a tree, and elements point to their "parent" in the tree. The root of each tree is the representative (or "name") of the subset.

**Idea**:
- Each element has a pointer to its parent in the tree.
- The root of the tree has no parent (or points to itself).
- The root’s value is the identifier of the subset.

**Representation** (Page 77):
- Use an array `parent[]` where `parent[i]` is the parent of element `i`.
- If `i` is a root, set `parent[i] = -1` (or `i` could point to itself, depending on the convention).

**Example Setup** (Page 74):
- Elements: `0, 1, 2, ..., 7`
- Initially, each element is its own root: `parent = [-1, -1, -1, -1, -1, -1, -1, -1]`.
  - This represents 8 disjoint subsets: `{0}, {1}, {2}, ..., {7}`.

**Operations**:
- **Find(i)** (Page 78):
  - Follow the parent pointers from `i` until you reach a root (where `parent[root] = -1`).
  - The root is the representative of the subset.
  - Time complexity: O(h), where `h` is the height of the tree.
- **Union(i, j)** (Page 78):
  - Find the roots of the trees containing `i` and `j`: `r_i = Find(i)`, `r_j = Find(j)`.
  - If `r_i != r_j`, make one root the parent of the other (e.g., `parent[r_i] = r_j`).
  - Time complexity: O(h) due to the `Find` operations.

**Example** (Page 74-76):
- Initial: `parent = [-1, -1, -1, -1, -1, -1, -1, -1]`
- **Union(1, 2)**:
  - `Find(1) = 1`, `Find(2) = 2`.
  - Make `2` the parent of `1`: `parent[1] = 2`.
  - `parent = [-1, 2, -1, -1, -1, -1, -1, -1]`.
  - Tree: `2 -> 1`.
- **Union(3, 4)**:
  - `Find(3) = 3`, `Find(4) = 4`.
  - `parent[3] = 4`.
  - `parent = [-1, 2, -1, 4, -1, -1, -1, -1]`.
  - Tree: `4 -> 3`.
- **Union(1, 5)**:
  - `Find(1) = 2` (follow `1 -> 2`), `Find(5) = 5`.
  - `parent[5] = 2`.
  - `parent = [-1, 2, -1, 4, -1, 2, -1, -1]`.
  - Tree: `2 -> {1, 5}`.
- **Union(1, 3)**:
  - `Find(1) = 2`, `Find(3) = 4` (follow `3 -> 4`).
  - `parent[2] = 4`.
  - `parent = [-1, 2, 4, 4, -1, 2, -1, -1]`.
  - Tree: `4 -> {3, 2}, 2 -> {1, 5}`.
- **Union(4, 6)**:
  - `Find(4) = 4`, `Find(6) = 6`.
  - `parent[6] = 4`.
  - `parent = [-1, 2, 4, 4, -1, 2, 4, -1]`.
  - Tree: `4 -> {3, 2, 6}, 2 -> {1, 5}`.

**Find(7)**:
- `parent[7] = -1`, so `Find(7) = 7`.

**Problem**:
- Trees can become tall (e.g., a chain), making `Find` slow (O(N) in the worst case).

---

### **Optimizations: Making Disjoint Sets Efficient**

To improve performance, we use two key optimizations: **Union by Rank** (or Union by Size) and **Path Compression**.

#### **1. Union by Rank (Smart Union, Page 79)**

**Idea**:
- When performing `Union(i, j)`, always attach the shorter tree to the root of the taller tree to keep the tree height small.
- Each root stores a "rank" (an upper bound on the tree’s height).
- If `rank[r_i] < rank[r_j]`, make `r_j` the parent of `r_i`.
- If ranks are equal, choose one as the parent and increment its rank by 1.

**Example** (Page 79):
- Consider two trees after `Union(3, 4)`:
  - Tree 1 (root 4, rank 1): `4 -> 3`.
  - Tree 2 (root 2, rank 2): `2 -> {1, 5}`.
- If we attach `4` to `2`, the height is 2 (better).
- If we attach `2` to `4`, the height becomes 3 (worse).
- Union by Rank chooses the first option.

**Benefit**:
- Keeps tree height logarithmic: O(log N) for `Find`.

#### **2. Path Compression**

**Idea**:
- During `Find(i)`, when you traverse from `i` to the root, make all nodes on the path point directly to the root.
- This flattens the tree, making future `Find` operations faster.

**Example**:
- Tree: `4 -> 2 -> 1`.
- `Find(1)`:
  - Path: `1 -> 2 -> 4`.
  - Root is `4`.
  - Update: `parent[1] = 4`, `parent[2] = 4`.
- New tree: `4 -> {1, 2}`.

**Benefit**:
- After path compression, subsequent `Find` operations are nearly O(1).
- Amortized time per operation becomes almost constant (technically, O(α(N)), where `α` is the inverse Ackermann function, which grows extremely slowly).

**Combined Effect**:
- With Union by Rank and Path Compression, the amortized time per operation is **O(α(N))**, which is effectively constant for all practical purposes.

---

### **Application: Maze Generation (Page 72-73)**

Disjoint sets are perfect for generating a maze, as shown in the slides. Let’s break down the process.

**Problem**:
- Generate a maze on a grid (e.g., 4x4 grid, cells numbered 1 to 16).
- Start with walls between all adjacent cells.
- Break walls to create a path from the entrance (cell 1) to the exit (cell 16), ensuring all cells are eventually connected.

**Algorithm** (Page 72):
```
While the entrance cell is not connected to the exit cell:
    Randomly pick a wall (between two adjacent cells i and j);
    If Find(i) != Find(j):
        Union(i, j); // Break the wall
```

**Example** (Page 73):
- Grid: 4x4, cells `1` to `16`.
- Initially, each cell is its own subset (using tree implementation).
- **Union(1, 2)**: Cells 1 and 2 are adjacent (right). `Find(1) != Find(2)`, so merge them and break the wall.
- **Union(4, 8)**: Cells 4 and 8 are adjacent (below). Merge and break the wall.
- **Union(6, 7)**: Merge and break.
- **Union(14, 15)**: Merge and break.
- **Union(5, 9)**: Merge and break.
- **Union(3, 7)**: Merge and break.
- **Union(2, 6)**: Merge and break.
- **Union(8, 12)**: Merge and break.
- **Union(3, 4)**: Merge and break.
- **Union(11, 12)**: Merge and break.
- **Union(11, 15)**: Merge and break.
- **Union(15, 16)**: Merge and break.
- Now, `Find(1) == Find(16)`, so stop.

**Result**:
- The broken walls form a maze with a path from cell 1 to cell 16.

**Why Disjoint Sets?**
- Disjoint sets efficiently track which cells are connected (via `Find`).
- `Union` ensures we only break walls between disconnected cells, avoiding cycles and ensuring a single path.

---

### **Summary of Disjoint Sets**

- **Purpose**: Manage a collection of elements partitioned into disjoint subsets, supporting grouping (Union) and connectivity queries (Find).
- **Why We Need It**:
  - Solve problems involving equivalence relations, connectivity, or grouping (e.g., maze generation, graph components).
- **How It Works**:
  - **Array Implementation**: Fast `Find` (O(1)), slow `Union` (O(N)).
  - **Tree Implementation**: Represents subsets as trees.
    - `Find`: Follow parent pointers to the root.
    - `Union`: Link roots of two trees.
  - **Optimizations**:
    - Union by Rank: Keep trees shallow (O(log N)).
    - Path Compression: Flatten trees during `Find` (amortized O(α(N))).
- **Applications**:
  - Maze generation (Page 72-73).
  - Equivalence relations (Page 64-66).
  - Graph algorithms (e.g., connected components, Kruskal’s algorithm).

**For Your Notes**:
- Disjoint sets are all about **grouping** and **checking membership**.
- Start with the array implementation to understand the concept, then move to the tree implementation for efficiency.
- Always use Union by Rank and Path Compression in practice for near-constant time operations.

If you’d like to dive deeper into any part (e.g., more examples, code implementation, or another application), let me know!

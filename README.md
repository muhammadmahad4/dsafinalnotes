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
- 

 # lecture Suffix and Array
 Let’s dive into the topics covered in the lecture slides **"Lec11 Suffix Array & Graphs.pdf"**, breaking everything down into manageable sections. I’ll explain each concept step by step, highlight relationships between topics, and ensure you understand how they connect to the previous lecture on disjoint sets (from "Lec10 Disjoint Set & Suffix Array.pdf"). Since you’ve asked for a similar teaching style, I’ll start from the basics, use examples, and reference the slides for clarity. We’ll cover **Disjoint Sets** (review), **Suffix Arrays**, and **Graphs**, exploring their implementations, applications, and interconnections.

---

## **1. Review of Disjoint Sets**

The lecture starts with a review of disjoint sets (Pages 2–31), which we covered extensively in the previous lecture. Let’s recap the key points and connect them to the new topics.

### **1.1 Equivalence Relations and Classes (Pages 3–5)**

**What Are Equivalence Relations?**
- A relation `R` on a set `S` is an **equivalence relation** if it satisfies:
  - **Reflexive**: `a R a` for all `a` in `S`.
  - **Symmetric**: If `a R b`, then `b R a`.
  - **Transitive**: If `a R b` and `b R c`, then `a R c`.
- Example (Page 3): The relation "same country" on a world map divides the map into disjoint countries (e.g., people in the same country are equivalent).

**Equivalence Classes** (Page 5):
- An **equivalence class** is a subset of elements equivalent to each other under the relation.
- These classes are **disjoint** (no overlap) and collectively cover the entire set `S`.
- **Exercise** (Page 5):
  - Elements: `a, b, c, d, e, f, g, h, i, j, k`.
  - Relations: `a ~ b, b ~ c, b ~ d, e ~ f, g ~ h, i ~ e, j ~ k, k ~ c`.
  - Using transitivity:
    - `a ~ b, b ~ c, b ~ d` → `{a, b, c, d}`.
    - `k ~ c, j ~ k` → `{a, b, c, d, j, k}`.
    - `e ~ f, i ~ e` → `{e, f, i}`.
    - `g ~ h` → `{g, h}`.
  - **Result**: 3 equivalence classes: `{a, b, c, d, j, k}`, `{e, f, i}`, `{g, h}`.

**Connection to Disjoint Sets**:
- Disjoint sets are a data structure to manage these equivalence classes efficiently. Each class is a subset, and the disjoint set operations help us merge classes (Union) and check if two elements are in the same class (Find).

### **1.2 Disjoint Set ADT (Page 6)**

**Operations**:
- **Find(a)**: Returns the name (identifier) of the equivalence class containing `a`. If `Find(a) == Find(b)`, then `a` and `b` are in the same class.
- **Union(a, b)**: Merges the classes containing `a` and `b`.
  - **Precondition**: Suppose `Find(a) == Find(c)` and `Find(b) == Find(d)`.
  - **Postcondition**: After `Union(a, b)`, `a, b, c, d` are in the same class.

### **1.3 Array Implementation (Pages 7–8)**

**How It Works**:
- Use an array `s[]` where `s[i]` is the name of the equivalence class for element `i`.
- Initially: `s[i] = i` (each element in its own class).
- **Find(i)**: Return `s[i]`. Time: **O(1)**.
- **Union(i, j)**: Update all elements with the name `s[j]` to `s[i]`. Time: **O(N)**.

**Example** (Page 7):
- Initial: `s = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`.
- **Union(1, 2)**: `s[2] = s[1] = 1` → `s = [0, 1, 1, 3, 4, 5, 6, 7, 8, 9]`.
- **Union(3, 1)**: `s[1] = s[2] = s[3] = 3` → `s = [0, 3, 3, 3, 4, 5, 6, 7, 8, 9]`.
- **Find(1)**: Returns `s[1] = 3`.

**Code** (Page 8):
```cpp
class DisjointSet {
public:
    int Find(int i) { return name[i]; }
    void Union(int i, int j) {
        if (Find(i) != Find(j)) {
            int temp = name[j];
            for (int k = 0; k < MAX_SIZE; k++) {
                if (name[k] == temp) name[k] = name[i];
            }
        }
    }
private:
    int name[MAX_SIZE];
};
```

**Drawback**: `Union` is slow (O(N)).

### **1.4 Tree Implementation (Pages 11–15)**

**Idea**:
- Represent each subset as a tree, where each node points to its parent.
- Use an array `parent[]`, where `parent[i]` is the parent of `i`. If `i` is a root, `parent[i] = -1`.
- Initially: `parent = [-1, -1, ..., -1]` (each element is its own root).

**Operations** (Page 15):
- **Find(i)**: Follow parent pointers to the root. Time: O(h), where `h` is the tree height.
- **Union(i, j)**:
  - Find roots `r_i = Find(i)`, `r_j = Find(j)`.
  - Link one root to the other (e.g., `parent[r_i] = r_j`).

**Example** (Pages 11–14):
- Initial: `parent = [-1, -1, -1, -1, -1, -1, -1, -1]` (elements 0–7).
- **Union(1, 2)**: `parent[1] = 2` → `[-1, 2, -1, -1, -1, -1, -1, -1]`.
- **Union(3, 4)**: `parent[3] = 4` → `[-1, 2, -1, 4, -1, -1, -1, -1]`.
- **Union(1, 5)**: `Find(1) = 2`, `Find(5) = 5`, `parent[5] = 2` → `[-1, 2, -1, 4, -1, 2, -1, -1]`.
- **Union(1, 3)**: `Find(1) = 2`, `Find(3) = 4`, `parent[2] = 4` → `[-1, 2, 4, 4, -1, 2, -1, -1]`.

### **1.5 Optimizations: Union by Height and Path Compression (Pages 16–29)**

**Union by Height (Pages 16–23)**:
- Attach the shorter tree to the root of the taller tree to minimize height.
- Each root stores its height (or rank).
- Time for `Find`: O(log N) (Pages 17, 19–23).
- **Proof** (Lemma 1, Pages 19–22): For a root `x`, `size(x) ≥ 2^height(x)`.
  - Base case: Height = 0, size = 1 → `1 ≥ 2^0`.
  - Inductive step: After `Union(x, y)`, the new height increases only if `height(x) = height(y)`, and the new size is at least `2^(height+1)`.
  - Theorem 1 (Page 23): Height `h ≤ log N`.

**Union by Size (Page 24)**:
- Attach the smaller tree (by size) to the larger tree. Also guarantees height O(log N).

**Path Compression (Page 26)**:
- During `Find(i)`, make all nodes on the path point directly to the root.
- Code:
```cpp
int Find(int element) {
    if (A[element] < 0) return element;
    return A[element] = Find(A[element]); // Path compression
}
```

**Combined Effect (Page 28)**:
- Union by Height + Path Compression → Amortized time per operation: **O(α(N))**, where `α(N)` is the inverse Ackermann function (Page 29).
- `α(N)` is very small (e.g., `α(2^65535) = 5`), making operations nearly O(1).

### **1.6 Application: Maze Generation (Pages 9–10)**

**Algorithm** (Page 9):
- Start with a grid (e.g., 4x4, cells 1–16) with walls between adjacent cells.
- While entrance (cell 1) and exit (cell 16) are not connected:
  - Pick a random wall between cells `i` and `j`.
  - If `Find(i) != Find(j)`, do `Union(i, j)` and break the wall.

**Example** (Page 10):
- **Union(1, 2)**, **Union(4, 8)**, ..., **Union(15, 16)**.
- Eventually, `Find(1) == Find(16)`, forming a maze with a path from 1 to 16.

**Connection**:
- Disjoint sets track connectivity, a concept we’ll revisit in graphs (connected components).

### **1.7 Exercises (Pages 18, 27, 31)**

**Exercise (Page 31)**:
- Sequence: `Union(0, 1), Union(2, 3), Union(4, 5), Union(0, 2), Union(5, 6), Union(5, 7), Union(5, 8), Union(2, 4)`.
- Use Union by Height (when heights are equal, attach the first tree to the second).
- Final array: `[3, 3, 3, -2, 3, 3, 3, 3, 3, -1]`.
  - Element 9 stays isolated (`parent[9] = -1`).
  - Elements 0–8 form one set with root 3 (`parent[3] = -2` indicates height 2).

**Relationship to New Topics**:
- Disjoint sets are used in graph algorithms (e.g., finding connected components, Kruskal’s algorithm for Minimum Spanning Trees), which we’ll see later.

---

## **2. Suffix Arrays**

Now we transition to **suffix arrays** (Pages 32–58), a data structure for string processing, particularly pattern matching.

### **2.1 What Are Suffixes? (Page 33)**

**Definition**:
- For a string `T = t_1 t_2 ... t_n`, the suffix starting at position `i` is `Suffix(T, i) = t_i t_{i+1} ... t_n`.
- Example: `T = innovation`.
  - `Suffix(T, 1) = innovation`
  - `Suffix(T, 2) = nnovation`
  - ...
  - `Suffix(T, 10) = n`

**Why Suffixes? (Page 34)**:
- Any substring of `T` is a **prefix** of some suffix of `T`.
- Example: `T = mississippi`, pattern `P = ssip`.
  - `P` is the first 4 characters of `Suffix(T, 6) = ssippi`.

**Connection**:
- This property makes suffixes useful for pattern matching, as we can search for a pattern by examining suffixes.

### **2.2 Exact Pattern Matching (Page 35)**

**Problem**:
- Find all occurrences of pattern `P` in text `T`.
- Naïve approach: For each `i`, check if `P` is a prefix of `Suffix(T, i)`.
  - Time: **O(|P| * |T|)** (too slow).

**Knuth-Morris-Pratt (KMP) Algorithm**:
- Preprocess `P` in O(|P|) time.
- Search in O(|P| + |T|) time by skipping impossible suffixes.
- Covered in CS 4335 (not detailed here).

### **2.3 Matching Concurrently (Pages 36–38)**

**Idea**:
- Instead of checking suffixes sequentially (like KMP), why not test all suffixes concurrently?
- Example: `T = mississippi`, `P = ssip`.
  - List all suffixes: `mississippi`, `ississippi`, ..., `i`.
  - Check which suffixes start with `ssip` (e.g., `ssissippi`, `ssippi` match).

**Issue**:
- Checking suffixes sequentially is inefficient. We need a better structure: the **suffix array**.

### **2.4 Suffix Array (Page 51)**

**Definition**:
- A **suffix array (SA)** is an array of indices of all suffixes of a string `T`, sorted in **lexicographical order**.
- Example: `T = mississippi` (we’ll compute this later).

**Usage**:
- Find occurrences of `P` using binary search on the suffix array.
  - Time: **O(|P| * log |T|)** (naïve binary search).
  - Can be optimized to **O(|P|)** (advanced topic).

**Key Question**:
- How do we build the suffix array? (i.e., how to sort suffixes efficiently?)

### **2.5 Sorting Suffixes (Page 52)**

**Methods**:
1. **Comparison-Based Sorting (e.g., QuickSort)**:
   - Time: **O(n^2 log n)** (comparing two suffixes takes O(n)).
   - Practical for real-world problems but not optimal.
2. **Radix Sort**:
   - Time: **O(n^2)** (sort by each character position).
3. **Doubling Algorithm (Manber and Myers, 1990)**:
   - Time: **O(n log n)**.
4. **Skew Algorithm (Kärkkäinen et al., 2006)**:
   - Time: **O(n)** (for integer alphabets).

### **2.6 Doubling Algorithm (Pages 54–57)**

**Idea**:
- Sort suffixes by their prefixes of increasing lengths: 1-order, 2-order, 4-order, ..., until exceeding `n`.
- **L-order**: Compare suffixes using their first `L` characters (Page 53).
- To extend from L-order to 2L-order (Page 56):
  - If `S[i] <L S[j]`, then `S[i] <2L S[j]`.
  - If `S[i] =L S[j]`, compare `S[i+L]` and `S[j+L]` using L-order.

**Example** (Page 55):
- `T = mississippi`.
- Suffixes: `S[1] = mississippi`, ..., `S[11] = i`.
- After sorting by 2-order, extend to 4-order:
  - Compare `S[3] = ssissippi` and `S[6] = ssippi`.
  - First 2 chars (`ss`) are equal, so compare `S[5] = issippi` and `S[8] = ippi`.

**Complexity** (Page 57):
- **Phases**: `L = 1, 2, 4, ..., n` → O(log n) phases.
- **Each Phase**: O(n) comparisons.
- **Total Time**: **O(n log n)**.
- **Space**: **O(n)**.

**Connection**:
- Suffix arrays are a space-efficient alternative to suffix trees, both used for string matching. They don’t directly relate to disjoint sets but are another example of a data structure for a specific problem (string processing vs. grouping).

---

## **3. Graphs**

The lecture concludes with an introduction to **graphs** (Pages 59–76), which are fundamental in computer science for modeling relationships.

### **3.1 Terms and Definitions (Pages 60–63)**

**What Is a Graph? (Page 60)**:
- A graph `G = (V, E)` consists of:
  - `V`: Set of vertices (nodes).
  - `E`: Set of edges (connections between vertices).
- Vertices represent entities (e.g., cities), and edges represent relationships (e.g., roads).

**Types of Graphs**:
- **Directed vs. Undirected**:
  - Directed: Edges have direction (e.g., `A → B`).
  - Undirected: Edges are bidirectional (e.g., `A — B`).
- **Weighted vs. Unweighted (Page 62)**:
  - Weighted: Edges have values (e.g., distance between cities).
  - Unweighted: Edges have no values (e.g., just connectivity).
- **Simple Graph (Page 63)**:
  - Undirected, no loops, no multiple edges between vertices.
- **Complete Graph (Page 63)**:
  - Every pair of vertices is connected.
  - If `|V| = n`, number of edges = `n(n-1)/2`.

**Degree (Page 61)**:
- **Degree of a vertex**: Number of edges connected to it.
- In a directed graph:
  - **In-degree**: Number of incoming edges.
  - **Out-degree**: Number of outgoing edges.
- Example: Vertex `W` with in-degree 2 and out-degree 1.

**Connection to Disjoint Sets**:
- Graphs often involve connectivity questions (e.g., are two vertices in the same component?), which disjoint sets can efficiently answer.

### **3.2 Representations of Graphs (Pages 64–69)**

**1. Adjacency Matrix (Pages 65–66)**:
- Use a `N x N` matrix where `matrix[i][j]` is the weight of the edge from vertex `i` to `j` (or 1 if unweighted, 0 if no edge).
- Example (Page 65):
  - Vertices: `A, B, C, D, E, F, G`.
  - Matrix: `C` connects to `A, B, D, E`.
- **Pros**:
  - Fast edge queries: O(1).
- **Cons**:
  - Space: **O(N^2)**.
  - Inefficient for sparse graphs (many zeros).

**2. Adjacency List (Page 67)**:
- For each vertex, store a list of its neighbors.
- **Pros**:
  - Space-efficient for sparse graphs: O(|V| + |E|).
  - Fast neighbor enumeration.
- **Cons**:
  - Slower edge queries: O(degree of vertex).

**3. Compressed Sparse Row (CSR) (Page 68)**:
- Store edges in an `edge-array` (sorted by source vertex) and a `vertex-array` (offsets into `edge-array`).
- Example:
  - `edge-array = [0, 1, 1, 2, 0, 2, 3, 1, 3]`.
  - `vertex-array = [0, 2, 4, 7, 9]`.
- **Pros**:
  - Very space-efficient for sparse graphs.
  - Fast neighbor enumeration.

**Connection**:
- Graph representations are crucial for algorithms like DFS and BFS, which we’ll see next. They also relate to disjoint sets in algorithms like Kruskal’s (for Minimum Spanning Trees), where we need to track connected components.

### **3.3 Graph Searching (Page 70)**

**Goals**:
- Check if two vertices are connected.
- List all vertices in a connected component.
- Find the shortest path (in unweighted graphs).

**Algorithms**:
- **DFS (Depth-First Search)**: Explore as deep as possible.
- **BFS (Breadth-First Search)**: Explore level by level.

### **3.4 Depth-First Search (DFS) (Pages 71–73)**

**How It Works** (Page 71):
- Use a **stack** to track nodes.
- Start at vertex `v`, mark it as visited.
- While the stack is not empty:
  - Check the top node.
  - If it has unvisited neighbors, push one onto the stack.
  - If not, pop the node.

**Example** (Page 71):
- Possible DFS orders starting from vertex 1: `1, 2, 4, 6, 8, 5, 3, 7` or `1, 5, 7, 3, 2, 8, 4, 6`.

**Code** (Page 73):
```cpp
void DFS(int v) {
    visited[v] = true;
    for (each vertex w adjacent to v) {
        if (!visited[w]) DFS(w);
    }
}
```
- Example: `DFS(A)` → `A, C, B, D, F, E, G`.

### **3.5 Breadth-First Search (BFS) (Pages 74–76)**

**How It Works** (Page 74):
- Use a **queue** to track nodes.
- Start at vertex `v`, mark it as visited, enqueue it.
- While the queue is not empty:
  - Dequeue a node `x`.
  - Enqueue all unvisited neighbors of `x`.

**Example** (Page 74):
- Possible BFS orders starting from vertex 1: `1, 2, 5, 4, 8, 3, 7, 6` or `1, 5, 2, 7, 3, 8, 4, 6`.

**Code** (Page 76):
```cpp
void BFS(int v) {
    visited[v] = true;
    Enqueue(v);
    while (queue not empty) {
        x = Dequeue();
        for (each vertex w adjacent to x) {
            if (!visited[w]) {
                Enqueue(w);
                visited[w] = true;
            }
        }
    }
}
```
- Example: `BFS(A)` → `A, C, B, D, E, G, F`.

**Comparison**:
- **DFS**: Goes deep, good for finding cycles or connected components.
- **BFS**: Goes broad, good for finding the shortest path in unweighted graphs.

**Connection to Disjoint Sets**:
- Both DFS and BFS can identify connected components in a graph. Disjoint sets offer a more efficient way to do this for dynamic graphs (e.g., in Kruskal’s algorithm).

### **3.6 Applications (Page 59)**

**Mentioned Applications**:
- **Minimum Spanning Trees (MST)**:
  - Find a tree that connects all vertices with minimum total edge weight.
  - Kruskal’s algorithm uses disjoint sets to avoid cycles.
- **Shortest Path**:
  - BFS finds the shortest path in unweighted graphs.
  - For weighted graphs, algorithms like Dijkstra’s are used (not covered here).

**Connection**:
- Disjoint sets, suffix arrays, and graphs all solve connectivity or relationship problems:
  - Disjoint sets: Group elements and check equivalence.
  - Suffix arrays: Find patterns in strings (a form of relationship).
  - Graphs: Model and analyze relationships explicitly.

---

## **Relationships Between Topics**

1. **Disjoint Sets and Graphs**:
   - Disjoint sets are used in graph algorithms like Kruskal’s MST algorithm to track connected components efficiently.
   - Both deal with connectivity: disjoint sets via equivalence classes, graphs via edges.

2. **Disjoint Sets and Suffix Arrays**:
   - Less direct connection, but both are data structures for specific problems:
     - Disjoint sets for grouping and connectivity.
     - Suffix arrays for string pattern matching.
   - Both involve efficient implementations (e.g., Union by Height + Path Compression for disjoint sets, Doubling Algorithm for suffix arrays).

3. **Suffix Arrays and Graphs**:
   - Suffix arrays can be used in graph-related problems, e.g., finding repeated patterns in a graph’s string representation (like a genome graph).
   - Both involve algorithmic techniques (e.g., sorting in suffix arrays, traversal in graphs).

4. **Overall Theme**:
   - All three topics are about managing and querying relationships:
     - Disjoint sets: Equivalence relationships.
     - Suffix arrays: Substring relationships.
     - Graphs: Explicit vertex-edge relationships.
   - Efficiency is key: Union-Find optimizations, suffix array construction, and graph representations all aim to reduce time and space complexity.

---

## **Learning Objectives (Pages 30, 58)**

- **Disjoint Sets**:
  1. Understand the concept (equivalence relations, operations).
  2. Analyze time complexities (O(α(N)) with optimizations).
  3. Know the best implementation (tree with Union by Height + Path Compression).
  4. Apply to problems (e.g., maze generation, graph algorithms).

- **Suffix Arrays**:
  1. Understand the concept (suffixes, sorting).
  2. Analyze complexities (O(n log n) for Doubling Algorithm).
  3. Know the implementation (Doubling Algorithm).
  4. Apply to pattern matching.

- **Graphs**:
  1. Understand graph terminology and representations.
  2. Know DFS and BFS for searching.
  3. Apply to problems like shortest paths and connected components.

---

## **Summary**

- **Disjoint Sets**: Efficiently manage equivalence classes with Union and Find, used in connectivity problems (e.g., maze generation, graph algorithms).
- **Suffix Arrays**: Sort suffixes for fast pattern matching in strings, built efficiently using the Doubling Algorithm.
- **Graphs**: Model relationships with vertices and edges, searched using DFS (deep) and BFS (broad), with applications like MST and shortest paths.

If you’d like to dive deeper into any section (e.g., more examples, code walkthroughs, or applications), let me know!

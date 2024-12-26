---
layout: post
title: CS 61B (Sp21) Notes 2, Data Structures
categories: course
tags: [data structures]
description: >
  Asymptotics, Set, Map, Disjoint Sets, Priority Queues, Tree, Graph
toc:
  sidebar: right
media_subpath: /assets/img/cs61b/
thumbnail: /assets/img/cs61b/cs61b-logo.png
---

## 13. Asymptotics I
Our goal is to somehow **<u>characterize the runtimes</u>** of two functions. Characterization should be (1) **simple** and **mathematically rigorous**; and (2) **demonstrate superiority** of one over the other.

### 13.1 Asymptotic Analysis
#### 13.1.1 Scaling Matters
In most cases, we care only about **<u>asymptotic behavior</u>**, i.e. what happens for very large $N$.

Algorithms which scale well (e.g. look like lines) have better asymptotic runtime behavior than algorithms that scale relatively poorly (e.g. look like parabolas). We’ll informally refer to the “shape” of a runtime function as its **order of growth** (will formalize soon).
- Often determines whether a problem can be solved at all.

#### 13.1.2 Computing Worst Case Order of Growth (Tedious Approach)
- Construct a table of exact counts of all possible operations.

![count](13/count.png){: w="550"}

- Convert table into worst case order of growth using 4 simplifications:
1. Only consider the worst case.
2. Pick a representative operation (a.k.a. the cost model).
3. Ignore lower order terms.
4. Ignore multiplicative constants.

![simlipy](13/simplify.png){: w="500"}

#### 13.1.3 Computing Worst Case Order of Growth (Simplified Approach)
- Choose a representative operation to count (a.k.a. cost model).
- Figure out the order of growth for the count of the representative operation by either:
  - Making an exact count, then discarding the unnecessary pieces.
  - Using intuition and inspection to determine order of growth (only possible with lots of practice).


### 13.2 Asymptotic Notation

#### 13.2.1 Big-Theta (a.k.a. Order of Growth)
For some function $R(N)$ with **order of growth** $f(N)$, we write that $R(N) \in \Theta(f(N))$. There exists some positive constants $k_1$, $k_2$  such that $k_1 \cdot f(N) \leq R(N) \leq k_2 \cdot f(N)$, for all values $N$ greater than some $N_0$ (a very large $N$). 

![Big-Theta](13/Big-Theta.png){: w="450"}
*[Visualization](https://sp19.datastructur.es/materials/demos/asymptotics.html?rN=4*N%5E2+40*sin(N)&fN=N%5E2&k1=3&k2=5&maxN=15&maxY=1000)*

#### 13.2.2 Big-O
$R(N) \in O(f(N))$ means that there exists positive constant $k_2$ such that $R(N) \leq k_2 \cdot f(N)$, for all values of $N$ greater than some $N_0$ (a very large $N$).

![Big-O](13/Big-O.png){: w="450"}
*[Visualization](https://sp19.datastructur.es/materials/demos/asymptotics.html?rN=4*N%5E2+40*sin(N)&fN=N%5E4&k1=0&k2=5&maxN=15&maxY=1000)*

Whereas $\Theta$ can informally be thought of as something like “equals”, $O$ can be thought of as “less than or equal”.

### 13.3 Summary
Given a piece of code, we can express its runtime as a function $R(N)$, where $N$ is a property of the input of the function often representing the **size** of the input. Rather than finding the exact value of $R(N)$ , we only worry about finding the **order of growth** of $R(N)$. One approach (not universal):
- Choose a representative operation
- Let $C(N)$ be the count of how many times that operation occurs as a function of $N$
- Determine order of growth $f(N)$ for $C(N)$, i.e. $C(N) \in \Theta(f(N))$
- Often (but not always) we consider the worst case count
- If operation takes constant time, then $R(N) \in \Theta(f(N))$
- Can use $O$ as an alternative for $\Theta$. $O$ is used for upper bounds.

## 14. Disjoint Sets
### 14.1 Dynamic Connectivity Problem
Deriving the "Disjoint Sets" data structure for solving the "Dynamic Connectivity" problem.
The Disjoint Sets data structure has two operations: 
- `connect(x, y)`: Connects `x` and `y`. 
- `isConnected(x, y)`: Returns `true` if `x` and `y` are connected. Connections can be transitive, i.e. they don't need to be direct.

Goal: Design an efficient `DisjointSets` implementation.
![disjoint_set_interface](14/disjoint_set_interface.png){:w="500"}

For each item, its **connected component** is the set of all items that are connected to that item. Model connectedness in terms of <u>sets</u> to keep track of which connected component each item belongs to.

![connected_components](14/connected_components.png){:w="500"}

### 14.2 Quick Find
Use an array of integers, where `i`th entry gives set number (a.k.a. "id") of item `i`. 
- `connect(x, y)`: Change entries that equal `id[x]` to `id[y]`.
- `isConnected(x, y)`: Check if `id[x]` equals `id[y]`.

![quick_find](14/quick_find.png){:w="500"}

`QuickFind` is too slow for practical use: Connecting two items takes $N$ time.

### 14.3 Quick Union
How could we change our set representation so that combining two sets into their union requires changing **one** value? Assign each item a `parent` (instead of an `id`). Results in a tree-like shape. Note: <u>for root items, we have item’s parent as itself.</u>
- `connect(x, y)`: make `root(x)` into a child of `root(y)`.
- `isConnected(x, y)`: Check if `root(x)` equals `root(y)`.

![quick_union](14/quick_union.png){:w="500"}

![quick_union_connect](14/quick_union_connect.png){:w="500"}
_connect(5, 2): Make root(5) into a child of root(2)_

Compared to `QuickFind`, we have to climb up a tree. Tree can get too tall and `root(x)` becomes expensive. **Things would be fine if we just keep our tree balanced.**

### 14.4 Weighted Quick Union
Modify quick-union to avoid tall trees. Track tree **size** (number of elements), and always <u>link root of smaller tree to larger tree</u>.
- `connect(x, y)`: create a separate `size` array to keep track of sizes. make `root(x)` into a child of `root(y)`, if `size[root(x)]` is smaller than `size[root(y)]`, or vice versa.
- `isConnected(x, y)`: Check if `root(x)` equals `root(y)`.

![choice_of_root](14/choice_of_root.png){:w="500"}

### 14.5 Path Compression
When we do `isConnected(x, y)`, tie all nodes seen to the root.

![path_compression_1](14/path_compression_1.png){: w="450"}

![path_compression_2](14/path_compression_2.png){: w="450"}

![path_compression_3](14/path_compression_3.png){: w="450"}

## 16. ADTs, Sets, Maps, BSTs
### 16.1 Abstract Data Types
An **Abstract Data Type (ADT)** is defined only by its operations, not by its implementation. The built-in [**java.util**](https://docs.oracle.com/javase/8/docs/api/java/util/package-summary.html) package provides a number of useful:
- **Interfaces**: ADTs (List, Set, Map, etc.) and other stuff.
- **Implementations**: Concrete classes you can use.

```java
List<Integer> L = new ArrayList<>();
```

![java](16/Java_utils.png){: w="550"}
_Common interfaces in Java and their implementations_

This lecture is about the basic ideas behind the `TreeSet` and `TreeMap`.

### 16.2 Binary Search Trees

#### 16.2.1 Derivation
For the **ordered linked list** set implementation below, `contains` and `add` take worst case linear time, i.e. $\Theta(N)$. Fundamental Problem: Slow search, even though it’s in order.

![ordered_linked_list](16/ordered_linked_list.png){: w="550"}

In **binary search**, we know the list is sorted, so we can use this information to narrow our search. <u>Applying binary search to a linked list</u> might seem challenging at first. We need to traverse all the way to the middle to check the element there, which would take linear time.

However, we can optimize this process. One way is to **keep a reference to the middle node**. This allows us to reach the middle in constant time. Additionally, if we **reverse the nodes' pointers**, we can traverse both the left and right halves of the list, effectively halving our runtime. We can further optimize by **adding pointers to the middle of each recursive half like so**.

![halved](16/halved.png){: w="550"}
_A linked list with a middle pointer_

![better](16/better.png){: w="550"}
_A linked list with recursive middle pointers is a binary tree_

Now, if you stretch this structure vertically, you will see a tree. This specific tree is called a **binary tree** because each juncture splits in 2.

#### 16.2.2 BST Definition
A **binary search tree** is a rooted binary tree with the BST property, i.e., For every node `X` in the tree: every key in the left subtree is less than `X`’s key, and every key in the right subtree is greater than `X`’s key.

```java
private class BST<Key> {
  private Key key;
  private BST left;
  private BST right;

  public BST(Key key, BST left, BST Right) {
    this.key = key;
    this.left = left;
    this.right = right;
  }

  public BST(Key key) {
    this.key = key;
  }
}
```

#### 16.2.3 Contains
To find a searchKey in a BST, we employ binary search, which is made easy due to the BST property.
```java
static BST find(BST T, Key sk) {
  if (T == null)
    return null;
  if (sk.equals(T.key))
    return T;
  else if (sk < T.key)
    return find(T.left, sk);
  else
    return find(T.right, sk);
}
```

The runtime to complete a search on a “bushy” BST in the worst case is $\Theta(\log N)$, where $N$ is the number of nodes.

#### 16.2.4 Insert
We **always** insert at a leaf node.

```java
static BST insert(BST T, Key ik) {
  if (T == null)
    return new BST(ik);
  if (ik < T.key)
    T.left = insert(T.left, ik);
  else if (ik > T.key)
    T.right = insert(T.right, ik);
  return T;
}
```

#### 16.2.5 Deletion
Deleting from a binary tree is a little bit more complicated because whenever we delete, we need to make sure we **reconstruct the tree and still maintain its BST property**. Let's break this problem down into three categories:
- the node we are trying to delete has no children
- has 1 child
- has 2 children

##### Case 1: Key with no Children
We can just <u>delete its parent pointer</u> and the node will eventually be swept away by the garbage collector.

##### Case 2: Key with one Child
The child maintains the BST property with the parent of the node because the property is recursive to the right and left subtrees. Therefore, we can just <u>reassign the parent's child pointer to the node's child</u> and the node will eventually be garbage collected.

Example: **delete("flat")**

![one child](16/del_1_child.png){: w="300"}

##### Case 3: Key with two Children (Hibbard)
If the node has two children, the process becomes a little more complicated because we can't just assign one of the children to be the new root. This might break the BST property.
Instead, we <u>choose a new node to replace the deleted one</u>.
We know that the new node must:
- be > than everything in left subtree.
- be < than everything right subtree.

Example: **delete("dog")**

![two children](16/del_2_children.png){: w="300"}

Choose either <u>predecessor</u> (`cat`, the right-most node in the left subtree) or <u>successor</u> (`elf`, the left-most node in the right subtree). Delete `cat` or `elf`, and stick new copy in the root position. This deletion guaranteed to be either case 1 or 2. This strategy is known as **Hibbard deletion**.

### 16.3 Sets and Maps
#### 16.3.1 Set
Can think of the BST below as representing a Set:
- {mo, no, sumomo, uchi, momo}

![set](16/set.png){: w="450"}

#### 16.3.2 Map

To represent maps, just have each BST node store key/value pairs.

![map](16/map.png){: w="450"}

Note: No efficient way to look up by value.
- Example: Cannot find all the keys with value = 1 without iterating over **ALL** nodes. This is fine.

### 16.4 Summary
- Abstract data types (ADTs) are defined in terms of operations, not implementation. 
  - Several useful ADTs: Disjoint Sets, Map, Set, List.
- Java provides `Map`, `Set`, `List` interfaces, along with several implementations.
- We’ve seen two ways to implement a Set (or Map):
  - ArraySet: $\Theta(N)$ operations in the worst case.
  - BST: $\Theta(\log N)$ operations if tree is balanced.
- BST Implementations:
  - Search and insert are straightforward (but insert is a little tricky).
  - Deletion is more challenging. Typical approach is "Hibbard deletion".


## 17. B-Trees (2-3, 2-3-4 Trees)
### 17.1 Binary Search Trees
#### 17.1.1 BST Height
BST height is all four of these:
- $O(N)$.
- $\Theta(\log N)$ in the best case (“bushy”).
- $\Theta(N)$ in the worst case (“spindly”).
- $O(N^2)$.

![BST](17/BST.png){: w="550"}
_Trees range from best-case “bushy” to worst-case “spindly”_

Difference is dramatic!

<!-- ### 1.2 Big O vs. Worst Case Big Theta

> Big O is **NOT** mathematically the same thing as “worst case”.
- e.g. BST heights are $O(N^2)$, but are not quadratic in the worst case.
- … but Big O often used as shorthand for “worst case”.
{: .prompt-warning}

> Big O is still a useful idea:
- Allows us to make simple blanket statements, e.g. can just say “binary search is $O(\log N)$” instead of “binary search is $\Theta(\log N)$ in the worst case”.
- Sometimes don’t know the exact runtime, so use $O$ to give an upper bound.
- Easier to write proofs for Big O than Big Theta.
{: .prompt-tip} -->

#### 17.1.2 Worst Case Performance
**Height and Depth**
- The **depth** of a node is how far it is from the root, e.g. `depth(g)` = 2.
- The **height** of a tree is the depth of its deepest leaf, e.g. `height(T)` = 4.

![height_and_depth](17/height_and_depth.png){: w="550"}

- The **average depth** of a tree is the average depth of a tree’s nodes.
    - (0x1 + 1x2 + 2x4 + 3x6 + 4x1)/(1+2+4+6+1) = 2.35

**Runtime**
- The height of a tree determines <u>the worst case runtime to find a node</u>.
  - Example: Worst case is `contains(s)`, requires 5 comparisons (height + 1).
- The “average depth” determines <u>the average case runtime to find a node</u>.
  - Example: Average case is 3.35 comparisons (average depth + 1).

**Nice Property** 

Random trees have $\Theta(\log N)$ average depth and height.
- Good news: BSTs have great performance if we insert items randomly. Performance is $\Theta(\log N)$ per operation.
- Bad News: We can’t always insert our items in a random order.


### 17.2 B-Trees
#### 17.2.1 Splitting Juicy Nodes

##### 17.2.1.1 Avoiding Imbalance through Overstuffing
> If we could simply avoid adding new leaves in our BST, the height would never increase.
{: .prompt-tip}

Instead of adding a new node upon insertion, we simply stack the new value into an existing leaf node at the appropriate location. Suppose we add `17`, `18`:

![overstuffing](17/overstuffing.png){: w="600"}
_Avoid new leaves by "overstuffing" the leaf nodes_

##### 17.2.1.2 Moving Items Up
> Height is balanced, but we have a new problem: Leaf nodes can get too juicy.
{: .prompt-danger}

We can set a limit $L$ on the number of items, say $L=3$. If any node has more than $L$ items, give an item to parent. Which one? Let’s say (arbitrarily) the left-middle.

![move_up](17/move_up.png){: w="550"}
_Moving 17 from a leaf node to its parent_

However, this runs into the issue that our binary search property is no longer preserved: `16` is to the right of `17`. As such, we need a second fix: split the overstuffed node into ranges: $(-\infty,15)$, $(15, 17)$ and $(17, +\infty)$. Parent node now has three children.

![split](17/split.png){: w="250"}
_Splitting the children of an overstuffed node_

##### 17.2.1.3 Chain Reaction Splitting

Suppose we add `25`, `26`:

![chain_1](17/chain_1.png){: w="600"}

In the case when our root is above the limit, we are forced to increase the tree height.

![root](17/root.png){: w="550"}

#### 17.2.2 B-Tree Terminology
> The origin of "B-tree" has never been explained by the authors. As we shall see, "balanced," "broad," or "bushy" might apply. Others suggest that the "B" stands for Boeing. Because of his contributions, however, it seems appropriate to think of B-trees as "Bayer"-trees. -- Douglas Corner (The Ubiquitous B-Tree)

Observe that our new splitting-tree data structure has <u>perfect balance</u>. If we split the root, every node is pushed down by one level. If we split a leaf or internal node, the height does not change. There is never a change that results in imbalance.

The real name for this data structure is a **B-Tree**. B-Trees with a limit of 3 items per node are also called 2-3-4 trees or 2-4 trees (a node can have 2, 3, or 4 children). Setting a limit of 2 items per node results in a 2-3 tree.

B-Trees are used mostly in two specific contexts:
- Small $L$ ($L=2$ or $L=3$):
  - Used as a conceptually simple balanced search tree (as today).
- $L$ is very large (say thousands).
  - Used in practice for databases and filesystems (i.e. systems with very large records).

#### 17.2.3 B-Tree Invariants
Because of the way B-Trees are constructed, we get two nice invariants:
- All leaves must be the same distance from the root.
- A non-leaf node with $k$ items must have exactly $k+1$ children.

#### 17.2.4 Worst Case Performance
Let $L$ be the maximum items per node. Based on our invariants, the maximum height must be somewhere between $\log_{L+1}(N)$ and $\log_2(N)$.

- Largest possible height is all non-leaf nodes have 1 item.

![worst_case](17/worst_case.png){: w="250"}

- Smallest possible height is all nodes have $L$ items.

![best_case](17/best_case.png){: w="500"}

Overall height is therefore $\Theta(\log N)$.

**Runtime for `contains`**

In the worst case, we have to examine up to $L$ items per node. We know that height is logarithmic, so the runtime of `contains` is bounded by $O(L \log N)$. Since $L$ is a constant, we can drop the multiplicative factor, resulting in a runtime of $O(\log N)$.

**Runtime for `add`**

A similar analysis can be done for `add`, except we have to consider the case in which we must split a leaf node. Since the height of the tree is $O(\log N)$, at worst, we do $\log N$ split operations (cascading from the leaf to the root). This simply adds an additive factor of $\log N$ to our runtime, which still results in an overall runtime of $O(\log N)$.

### 17.3 Summary
BSTs have best case height $\Theta(\log N)$, and worst case height $\Theta(N)$. B-Trees are a modification of the binary search tree that avoids $\Theta(N)$ worst case.
- Nodes may contain between 1 and $L$ items.
- `contains` works almost exactly like a normal BST.
- `add` works by adding items to existing leaf nodes.
  - If nodes are too full, they split.
- Resulting tree has perfect balance. Runtime for operations is $O(\log N)$.
- B-trees are more complex, but they can efficiently handle any **insertion order**.

## 18. Red Black Trees
### 18.1 Tree Rotation
#### 18.1.1 BSTs
Suppose we have a BST with the numbers `1`, `2`, `3`. Five possible BSTs.
- The specific BST you get is based on the insertion order.
- More generally, for $N$ items, there are [Catalan(N)](https://en.wikipedia.org/wiki/Catalan_number) different BSTs.

![BSTs](18/BSTs.png){:w="550"}

Given any BST, it is possible to move to a different configuration using “rotation”.
- In general, can move from any configuration to any other in 2n - 6 rotations (see [Rotation Distance, Triangulations, and Hyperbolic Geometry](https://www.cs.cmu.edu/~sleator/papers/rotation-distance.pdf) or [Amy Liu](https://medium.com/@liuamyj/its-triangles-all-the-way-down-part-1-17f932f4c438)).

#### 18.1.2 Definition
##### rotateLeft
`rotateLeft(G)`: Suppose `x` is the right child of `G`, make `G` the new left child of `x`.

![rotate_left_1](18/rotate_left_1.png){:w="550"}

- Can think of as temporarily merging `G` and `P`, then sending `G` down and left.

![rotate_left](18/rotate_left.png){:w="550"}

##### rotateRight
`rotateRight(P)`: Suppose `x` is the left child of `P`, make `P` the new right child of `x`.
- Can think of as temporarily merging `G` and `P`, then sending `P` down and right.

![rotate_right](18/rotate_right.png){:w="550"}

#### 18.1.3 Tree Balancing
Rotation:
- Can shorten (or lengthen) a tree.
- Preserves search tree property.

![rotation](18/rotation.png){:w="600"}

Rotation allows balancing of a BST in $O(N)$ moves.

### 18.2 Left Leaning Red-Black Trees (LLRBs)
#### 18.2.1 The 2-3 Tree Isometry
2-3 trees always remain balanced, but they are very hard to implement. On the other hand, BSTs can be unbalanced, but are simple and intuitive. Is there a way to combine the best of two worlds? Why not create <u>a tree that is implemented using a BST, but is structurally identical to a 2-3 tree and thus stays balanced</u>?

**Representing a 2-3 Tree as a BST**

A 2-3 tree with only 2-nodes is trivial. BST is exactly the same.

![2-nodes](18/2-nodes.png){:w="550"}

**Dealing with 3-Nodes**

- Possibility 1: Create dummy “glue” nodes.

![dummy_glue_node](18/dummy_glue_node.png){:w="550"}
_Result is inelegant. Wasted link. Code will be ugly._

- Possibility 2: Create “glue” links with the smaller item **<u>off to the left</u>**.
    - For convenience, we’ll mark glue links as “**<font color=red>red</font>**”.

![glue_link](18/glue_link.png){:w="550"}
_Idea is commonly used in practice (e.g. java.util.TreeSet)._

A BST with left glue links that represents a 2-3 tree is often called a “Left Leaning Red Black Binary Search Tree” or LLRB.

#### 18.2.2 LLRB Properties
- Suppose we have a 2-3 tree of height H. The maximum height of the corresponding LLRB is
  - H (black) + <font color=red>H + 1 (red)</font> = 2H + 1.

![LLRB](18/LLRB.png){:w="550"}

- No node has two red links [otherwise it’d be analogous to a 4 node, which are disallowed in 2-3 trees].
- Every path from root to null has same number of **<u>black links</u>** [because 2-3 trees have the same number of links to every leaf]. LLRBs are therefore balanced.

#### 18.2.3 Maintaining Isometry with Rotations
When inserting into an LLRB tree, we <u>always insert the new node with a red link to its parent node</u>. This is because in a 2-3 tree, we are always inserting by adding to a leaf node, the color of the link we add should always be red.

But sometimes, inserting red links at certain places might lead to cases where we break one of the invariants of LLRBs. Below are three such cases where we need to perform certain tasks address in order to maintain the LLRB tree's proper structure.

**Case 1: Insertion on the Right**

- If the left child is also a red link, go to case 3.
- Otherwise, Rotate `E` left.

![case1](18/case1.png){:w="550"}

**Case 2: Double Insertion on the Left**

- Rotate `Z` right. Then go to case 3.

![case2](18/case2.png){:w="550"}

**Case 3: Node has Two Red Children**

- Flip the colors of all edges touching `B`.

![case3](18/case3.png){:w="350"}

**Cascading operations**

It is possible that a rotation or flip operation will cause an additional violation that needs fixing.

#### 18.2.4 Runtime and Implementation
The runtime analysis for LLRBs is simple if you trust the 2-3 tree runtime.
- LLRB tree has height $O(\log N)$.
- `contains` is trivially $O(\log N)$.
- `insert` is $O(\log N)$.
  - $O(\log N)$ to add the new node.
  - $O(\log N)$ rotation and color flip operations per insert.
  

## 19. Hashing
### 19.1 Motivation
We’ve now seen several implementations of the Set (or Map) ADT.

![set_map_implementations](19/set_map_implementations.png){:w="650"}

Limits of **Search Tree** Based Sets:
- require items to be **comparable**
  - Could we somehow avoid the need for objects to be comparable?
- have excellent **performance**, but could maybe be better
  - Could we somehow do better than $\Theta(\log N)$?

### 19.2 The Hash Table
- Data is converted by a hash function into an integer representation called a **hash code**. 
- The hash code is then **reduced** to a valid index, usually using the modulus operator, e.g. `2348762878 % 10 = 8`.

![hash_table](19/hash_table.png){:w="550"}

### 19.3 Hash Table Runtime
Suppose we have: an increasing number of buckets $M$, and an increasing number of items $N$. Even if items are spread out evenly, lists are of length $Q = \frac{N}{M}$.
The `contains(x)` and `add(x)` have the worst case runtimes $\Theta(Q)$.

As long as $M = \Theta(N)$, then $O(\frac{N}{M}) = O(1)$. **Resize** when load factor $\frac{N}{M}$ exceeds some constant. If items are spread out nicely, you get $\Theta(1)$ average runtime. One example strategy: When $\frac{N}{M}$ is ≥ 1.5, then double $M$. 

## 20. Heaps and PQs
###  20.1 Priority Queue
(Min) Priority Queue: Allowing **tracking** and **removal** of the smallest item in a priority queue. Useful if you want to keep track of the "smallest", "largest", "best" etc. seen so far.
```java
public interface MinPQ<Item> {
    /** Adds the item to the priority queue. */
    public void add(Item x);
    /** Returns the smallest item in the priority queue. */
    public Item getSmallest();
    /** Removes the smallest item from the priority queue. */
    public Item removeSmallest();
    /** Returns the size of the priority queue. */
    public int size();
}
```

### 20.2 Heaps
#### 20.2.1 Heap Structure
BSTs would work, but need to be kept bushy and duplicates are awkward.

**Binary min-heap**: Binary tree that is **complete** and obeys **min-heap** property.
- **Min-heap**: Every node is less than or equal to both of its children.
- **Complete**: Missing items (if any) only at the bottom level, all nodes are as far left as possible.

![heap](20/heap.png)

#### 20.2.2 Add
> Algorithm for `add(x)`: to maintain the **completeness**, the natural thought is to place `x` in the leftmost empty spot on the lowest level. However, this doesn't guarantee the **min-heap** property, so further adjustments are needed.
{: .prompt-tip}

**swim**: Continually compare `x` with its parent. If `x` is smaller, swap it with the parent. Repeat this process until `x` is in the right position.

<img src="20/insert.png" alt="swim" style="zoom:100%;" />

#### 20.2.3 Delete
> Algorithm for `removeSmallest()`: to maintain the **completeness** of the structure, the intuitive approach is to swap the root node with the node at the end, then remove it. However, this doesn't guarantee the **min-heap** property, so further adjustments are needed.
{: .prompt-tip}

**sink**: Continually compare `x` with its left and right nodes. Swap x with the smaller of the two nodes. Repeat this process until `x` is in the right position.
{: .prompt-info}

<img src="20/remove.png" alt="sink" style="zoom:80%;" />

#### 20.2.4 Heap Operations Summary
- `getSmallest()` - return the item in the root node.
- `add(x)` - place the new employee in the last position, and promote as high as possible.
- `removeSmallest()` - assassinate the president (of the company), promote the rightmost person in the company to president. Then demote repeatedly, always taking the 'better' successor.

### 20.3 Tree Representation
How do we Represent a Tree in Java?

<img src="20/tree.png" alt="tree" style="zoom:50%;" />

#### 20.3.1 Approach 1a, 1b, and 1c
Create mapping from node to children.

![ap1digrams](20/ap1digrams.png){: w="550"}

![ap1codes](20/ap1codes.png){: w="550"}

#### 20.3.2 Approach 2
Store keys in an array. Store parentIDs in an array.
- Similar to what we did with disjointSets.

<img src="20/ap2-1.png" alt="ap2-1" style="zoom: 40%;" />

A more complex example：

![ap2-2](20/ap2-2.png){: w="550"}

#### 20.3.3 Approach 3
Store keys in an array. Don’t store structure anywhere.

In Approach2, we observe that when numbering a binary tree in the order of level traversal, if the tree has the property of being complete, there is a pattern between the parent and child node numbers: **the parent number** corresponding to **the node number** $k$ is $\frac{k-1}{2}$.

Further, if we leave the 0th position empty, the following patterns emerge:
- `leftChild(k)` $= 2k$
- `rightChild(k)` $= 2k + 1$
- `parent(k)` = $\frac{k}{2}$

![ap3](20/ap3.png)


## 21. Tree and Graph Traversals
### 21.1 Trees
#### 21.1.1 Tree Definition
A tree consists of a set of **nodes** and a set of **edges** that connect those nodes, where there is **exactly one** path between any two nodes.

![graphs](21/graphs.png){:w="500"}
_Green structures below are trees. Pink ones are not._

#### 21.1.2 Tree Traversals
Sometimes you want to iterate over a tree. What one might call "tree iteration" is actually called "**tree traversal**". Unlike lists, there are many orders in which we might visit the nodes.

![tree](21/tree.png){:w="350"}

##### 21.1.2.1 Level Order
- DBFACEG

##### 21.1.2.2 Depth First Traversals
![tree_traversal_algo](21/tree_traversal_algo.png)
_DBACFEG | ABCDEFG | ACBEGFD_

<!-- ##### A Visual Trick
 We trace a path around the graph, from the top going counter-clockwise.
- Preorder traversal: "Visit" every time we pass the LEFT of a node. **1 2 4 5 7 8 3 6 9**
- Inorder traversal: "Visit" when you cross the BOTTOM of a node. **4 2 7 5 8 1 3 6 9**
- Postorder traversal: "Visit" when you cross the RIGHT a node. **4 7 8 5 2 9 6 3 1**

![tree_traversal_trick](21/tree_traversal_trick.png){:w="350"} -->

#### 21.1.3 Usefulness of Tree Traversals
##### Preorder Traversal for printing directory listing
![directory_listing](21/directory_listing.png){:w="450"}

##### Postorder Traversal for gathering file sizes
![file_sizes](21/file_sizes.png){:w="650"}

### 21.2 Graphs
#### 21.2.1 Graph Definition
Trees are fantastic for representing strict hierarchical relationships. But not every relationship is hierarchical. A **graph** consists of a set of **nodes** and a set of zero or more **edges**, each of which connects two nodes. Note, all trees are graphs.

A **simple graph** is a graph with:
- No edges that connect a vertex to itself, i.e. no "length 1 loops".
- No two edges that connect the same vertices, i.e. no "parallel edges".

![simpl_graph](21/simple_graph.png){:w="350"}
_Green graph below is simple, pink graphs are not._

In 61B, unless otherwise explicitly stated, all graphs will be simple.

#### 21.2.2 Graph Types
![graph_terms](21/graph_terms.png){:w="500"}

#### 21.2.3 Graph Problems
Some well known graph problems and their common names:
- **s-t Path**. Is there a path between vertices s and t?
- **Connectivity**. Is the graph connected, i.e. is there a path between all vertices?
- **Biconnectivity**. Is there a vertex whose removal disconnects the graph?
- **Shortest s-t Path**. What is the shortest path between vertices s and t?
- **Cycle Detection**. Does the graph contain any cycles?
- **Euler Tour**. Is there a cycle that uses every edge exactly once?
- **Hamilton Tour**. Is there a cycle that uses every vertex exactly once?
- **Planarity**. Can you draw the graph on paper with no crossing edges?
- **Isomorphism**. Are two graphs isomorphic (the same graph in disguise)?


### 21.3 Graph Traversals
#### 21.3.1 Motivation: s-t Connectivity
Let’s solve a classic graph problem called the s-t connectivity problem.
Given source vertex `s` and a target vertex `t`, is there a path between `s` and `t`?

> A **path** is a sequence of vertices connected by edges.
{: .prompt-info}

![s-t_connectivity](21/s-t_connectivity.png){:w="400"}

One possible recursive algorithm for `connected(s, t)`.

```py
def connected(s, t):
    if s == t:
        return True
    for v in neighbors_of(s):
        if connected(v, t):
            return True
    return False
```

What is wrong with it? Can get caught in an infinite loop. Example:
```
connected(0, 7):
    Does 0 == 7? No, so...
    If (connected(1, 7)) return true;
connected(1, 7):
    Does 1 == 7? No, so…
    If (connected(0, 7)) … ← Infinite loop.
```
How do we fix it?

#### 21.3.2 Depth First Search
Basic idea is same as before, but visit each vertex at most once. [[Demo](https://docs.google.com/presentation/d/1OHRI7Q_f8hlwjRJc8NPBUc1cMu5KhINH1xGXWDfs_dA/edit?usp=sharing)]

```py
def connected(s, t):
    marked[s] = True   # added
    if s == t:
        return True
    for v in neighbors_of(s):
        if not marked[v]  # added
            if connected(v, t):
                return True
    return False
```

#### 21.3.3 Tree vs. Graph Traversals
##### Another example: DepthFirstPaths
Find a path from `s` to every other reachable vertex. [[Demo](https://docs.google.com/presentation/d/1lTo8LZUGi3XQ1VlOmBUF9KkJTW_JWsw_DOPq8VBiI3A/edit#slide=id.g76e0dad85_2_380)]

```py
def dfs(v):
    marked[v] = True
    for w in neighbors_of[v]:
        if not marked[w]:
            edgeTo[w] = v
            dfs(w)
```

This is called "**DFS Preorder**". i.e., **Action** (setting `edgeTo`) is before DFS calls to neighbors. One valid DFS preorder for this graph: `012543678`, equivalent to the order of **dfs calls**.

![s-t_connectivity](21/s-t_connectivity.png){:w="400"}

We could also do actions in **DFS Postorder**. i.e., Action is after DFS calls to neighbors. Example:

```py
def dfs(s):
  marked[s] = True
  for w in neighbors_of[v]:
    if not marked[w]:
      dfs(w)
  print(s)
```
Results for `dfs(0)` would be: `347685210`, equivalent to the order of **dfs returns**.

Just as there are many tree traversals, so too are there many graph traversals:
- **DFS Preorder**: `012543678` (dfs calls).
- **DFS Postorder**: `347685210` (dfs returns).
- **BFS order**: Act in order of distance from `s`.
    - BFS stands for "breadth first search".
    - Analogous to "level order". Search is wide, not deep.
    - `0 1 24 53 68 7`

### 21.4 Summary
Graphs are a more general idea than a tree. A tree is a graph where there are no cycles and every vertex is connected.
Graph problems vary widely in difficulty. Common tool for solving almost all graph problems is traversal. A **traversal** is an order in which you visit / act upon vertices.
- Tree traversals: Preorder, inorder, postorder, level order.
- Graph traversals: DFS preorder, DFS postorder, BFS.

By performing actions / setting instance variables during a graph (or tree) traversal, you can solve problems like s-t connectivity or path finding.

## 22. Graph Traversals and Implementations
### 22.1 Breadth First Search
#### 22.1.1 Shortest Paths Challenge
Given the graph above, find the shortest path from `s` to all other vertices.

BFS Answer [[Demo](https://docs.google.com/presentation/d/1JoYCelH4YE6IkSMq_LfTJMzJ00WxDj7rEa49gYmAtc4/edit?usp=sharing)]
```py
def bfs():
    fringe = deque([s])
    while fringe:
        v = fringe.popleft()
        for w in neighbors_of[v]:
            if not marked[w]:
                marked[w] = True
                edgeTo[w] = v
                # distTo[w] = distTo[v] + 1
            fringe.append(w)
```

### 22.2 Graph Representations
#### 22.2.1 Adjacency Matrix
![adjacency_matrix](22/adjacency_matrix.png){:w="350"}

DFS, BFS Runtime: $O(V^2)$

#### 22.2.2 Edge Sets
![edge_sets](22/edge_sets.png){:w="350"}

#### 22.2.3 Adjacency Lists
Common approach: Maintain array of lists indexed by vertex number. Most popular approach for representing graphs. Efficient when graphs are "sparse" (not too many edges).

![adjacency_lists](22/adjacency_lists.png){:w="400"}

DFS, BFS Runtime: $O(V+E)$
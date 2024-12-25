---
layout: post
title: CS 61B (Sp21) Notes 3, Algorithms
categories: course
tags: [data structures, algorithms]
toc:
  sidebar: right
media_subpath: /assets/img/cs61b/
---

## 23. Shortest Paths
### 23.1 Introduction
#### 23.1.1 Why BFS Doesn't Work
BFS would **not** be a good choice for a google maps style navigation application. BFS returns path with shortest **number of edges**, not necessarily the shortest path.

![bfs_example](23/bfs_example.png){:w="550"}
_BFS yields a route of length ~330 m instead of ~150 m._

We need an algorithm that takes into account edge distances, also known as **edge weights**.

#### 23.1.2 The Shortest Paths Tree
##### Single Source Single Target Shortest Path
![single_source_single_target](23/single_source_single_target.png){:w="450"}
_Find the shortest paths from source vertex s to some target vertex t._

Observation: Solution will always be a path with no cycles (assuming non-negative weights).

##### Single Source Shortest Paths
![single_source](23/single_source.png){:w="450"}
_Find the shortest paths from source vertex s to every other vertex._

Observation: Solution will always be a **tree**. (Can think of as the union of the shortest paths to all vertices.) This is so-called **Shortest Paths Tree** (SPT).

### 23.2 Dijkstra's Algorithm
#### 23.2.1 Algorithm
We can find the SPT using **Dijkstra's algorithm** [[Demo](https://docs.google.com/presentation/d/1_bw2z1ggUkquPdhl7gwdVBoTaoJmaZdpkV6MoAgxlJc/pub?start=false&loop=false&delayms=3000&slide=id.g771336078_0_180)]. Perform a **best first search**: Visit vertices in **order of best-known distance** from source. On visit, ***relax*** every edge from the visited vertex. 

> **Relax**: If we find a better path, update our preferred edge and distance.
{:.prompt-info}

```py
def dijkstra():
    pq = heapq.heapify([])   # priority queue; i.e., min heap
    heapq.heappush(pq, (0, s))
    while pq:
        dis, v = heapq.heappop(pq)
        for (weight, x) in neighbors_of[v]:
            # relax ⬇
            if dis + weight < distTo[x]:
                distTo[x] = dis + weight
                edgeTo[x] = v
                heapq.heappush(pq, (distTo[x], x))
            # relax ⬆
```

#### 23.2.2 Correctness
Dijkstra's is guaranteed to return a correct result if all edges are non-negative.

### 23.3 A* Algorithm
If we have only a single target in mind, A* algorithm can do better:
>**A* Algorithm** Visit vertices in order of `d(source, v) + h(v, goal)`, where `h(v, goal)` is an estimate of the distance from v to our goal. [[Demo](https://docs.google.com/presentation/d/177bRUTdCa60fjExdr9eO04NHm0MRfPtCzvEup1iMccM/edit#slide=id.g771336078_0_180)]
{:.prompt-tip}


## 24. Minimum Spanning Trees
### 24.1 Concepts
Given an **undirected** graph `G`, a **spanning tree** `T` is a subgraph of `G`, where `T`:
- Is connected and acyclic. (make it a tree)
- Includes all of the vertices. (makes it spanning)

![mst](24/mst.png){:w="300"}

A ***minimum spanning tree*** (MST) is a spanning tree of minimum total weight.

### 24.2 The Cut Property
#### 24.2.1 Concepts
- A **cut** is an assignment of a graph's nodes to two non-empty sets.
- A **crossing edge** is an edge which connects a node from one set to a node from the other set.
- **Cut property**: Given any cut, minimum weight crossing edge is in the MST. (assume edge weights are unique)

![cut_property](24/cut_property.png){:w="350"}

#### 24.2.2 Proof
Suppose that the minimum crossing edge `e` were not in the MST.
- Adding `e` to the MST creates a cycle.
- Some other edge `f` must also be a crossing edge.
- Removing `f` and adding `e` is a lower weight spanning tree.
Contradiction!

![cut_property_proof](24/cut_property_proof.png){:w="450"}

### 24.3 Algorithms
#### 24.3.1 Prim's Algorithm
**Prim's Algorithm** [[Demo](https://docs.google.com/presentation/d/1NFLbVeCuhhaZAM1z3s9zIYGGnhT4M4PWwAc-TLmCJjc/edit#slide=id.g9a60b2f52_0_0)]: Start from some arbitrary start node. Add shortest edge that has **one** node inside the MST under construction. Repeat until $V-1$ edges. 

![prim](24/prim.png){:w="600"}

**Implementation** [[Demo](https://docs.google.com/presentation/d/1GPizbySYMsUhnXSXKvbqV4UhPCvrt750MiqPPgU-eCY/edit#slide=id.g9a60b2f52_0_0)]: Insert all vertices into fringe `PQ`, storing vertices in order of distance from tree.
Repeat: Remove (closest) vertex `v` from `PQ`, and relax all edges pointing from `v`. 

#### 24.3.2 Kruskal's Algorithm
**Kruskal's Algorithm** [[Demo](https://docs.google.com/presentation/d/1RhRSYs9Jbc335P24p7vR-6PLXZUl-1EmeDtqieL9ad8/edit#slide=id.g9a60b2f52_0_0)]: Consider edges in order of increasing weight. Add to MST unless a cycle is created. Repeat until $V-1$ edges.

![kruskals](24/kruskals.png){:w="600"}
_Blue and green colorings for vertices show cut being implicitly utilized by Kruskal’s algorithm._

**Implementation** [[Demo](https://docs.google.com/presentation/d/1KpNiR7aLIEG9sm7HgX29nvf3yLD8_vdQEPa0ktQfuYc/edit?usp=sharing)]: Insert all edges into `PQ`.
Repeat: Remove smallest weight edge. Add to MST if no cycle created. Use Weighted Quick Union UF (WQU) to detech cycle: For each edge, check if the two vertices are connected. If not, union them. If so, there is a cycle.

## 26. Prefix Operations and Tries
### 26.1 Introduction
HashMaps are already incredibly fast. But if we have some additional insight on the data we are storing, we could get even faster.

#### Special Case 1: **Character Keyed Map**

Suppose we know that our keys are always ASCII characters. We can just use an array. Simple and fast.

```java
public class DataIndexedCharMap<V> {
    private V[] items;
    // R is the number of possible characters, e.g. 128 for ASCII.
    public DataIndexedCharMap(int R) {
        items = (V[]) new Object[R];
    }
    public void put(char c, V val) {
        items[c] = val;
    }
    public V get(char c) {
        return items[c];
    }
}
```

#### Special Case 2: String Keyed Map
Suppose we know that our keys are always strings. We can use a special data structure known as a **Trie**. The basic idea: store **each letter** of the string as a node in a tree.

**Trie**: Short for Re**trie**val Tree. Inventor _Edward Fredkin_ suggested it should be pronounced "tree", but almost everyone pronounces it like "try".

![trie](26/trie_map.png){:w="500"}
_Use Trie to implement Map_

![set](26/set.png){:w="500"}
_different implementations of set_

### 26.2 Trie Implementation
#### 26.2.1 Basic Trie Implementation
The first approach might look something like the code below. Each node stores a letter (`ch`), a map from `ch` to all child nodes (`next`), and a color (`isKey`).

```java
public class TrieSet {
    private static final int R = 128; // ASCII
    private Node root;	// root of trie
    
    private static class Node {
        // private char ch; removed
        private boolean isKey;
        // Since we know our keys are characters, can use a DataIndexedCharMap.   
        private DataIndexedCharMap<Node> next;
        private Node( /*char c, removed*/ boolean b, int R) {
            // ch = c; removed
            isKey = b;
            next = new DataIndexedCharMap<>(R);
        }
    }
}
```
Since we use a `DataIndexedCharMap` to track children, every node has `R` links, mostly `null`. Observation: The letter stored inside each node is actually redundant. Can remove from the representation and things will work fine.

![basic_impl](26/basic_impl.png){:w="550"}

Assuming the length of the key is $L$, the runtime of `add` is $\Theta(L)$ and that of `contain` is $O(L)$.

#### 26.2.2 Alternate Child Tracking Strategies
Using a `DataIndexedCharMap` is very memory hungry. Every node has to store `R` links, most of which are `null`. Using BST or Hash Table will be slightly slower, but more memory efficient.

![implementation](26/implementation.png){:w="750"}
_The Hash-Table Based Trie (mid) and The BST-Based Trie (right)_

#### 26.2.3 Performance
Using a BST or a Hash Table to store links to children will usually use less memory.
- DataIndexedCharMap: 128 links per node.
- BST: $C$ links per node, where $C$ is the number of children.
- Hash Table: $C$ links per node.
- Note: Cost per link is higher in BST and Hash Table.

Using a BST or a Hash Table will take slightly more time.
- DataIndexedCharMap is $\Theta(1)$.
- BST is $O(\log R)$, where $R$ is size of alphabet.
- Hash Table is $O(R)$, where $R$ is size of alphabet.
- Since $R$ is fixed (e.g. 128), can think of all 3 as $\Theta(1)$.


### 26.3 Trie String Operations
Theoretical asymptotic speed improvement is nice. But the main appeal of tries is their ability to efficiently support string specific operations like **prefix matching**.
- Finding all keys that match a given prefix: `keysWithPrefix("sa")`
- Finding longest prefix of: `longestPrefixOf("sample")`

#### 26.3.1 Collecting Trie Keys
Give an algorithm for collecting all the keys in a Trie.
```java
public class TrieSet {
    private static final int R = 128;
    private Node root;

    private static class Node {
        private boolean isKey;
        private Map<Character, Node> next;

        private Node(boolean b, int R) {
            isKey = b;
            next = new HashMap<>(R);
        }
    }

    public List<String> collect() {
        List<String> x = new ArrayList<>();
        for (Character c: root.next.keySet()) {
            colHelp("c", x, root.next.get(c));
        }
        return x;
    }

    private void colHelp(String s, List<String> x, Node n) {
        if (n.isKey) {
            x.add(s);
        }
        for (Character c: n.next.keySet()) {
            colHelp(s + c, x, n.next.get(c));
        }
    }
}
```

#### 26.3.2 keysWithPrefix
Give an algorithm for `keysWithPrefix`.
```java
public class TrieSet {
    //...
    public List<String> keysWithPrefix(String prefix) {
        Node n = root;
        for (int i = 0; i < prefix.length(); i++) {
            for (Character c: n.next.keySet()) {
                if (Character.valueOf(c) == prefix.charAt(i)) {
                    n = n.next.get(c);
                }
            }
        }
        List<String> x = new ArrayList<>();
        for (Character c: n.next.keySet()) {
            colHelp(prefix + c, x, n.next.get(c));
        }
        return x;
    }

    private void colHelp(String s, List<String> x, Node n) {
        if (n.isKey) {
            x.add(s);
        }
        for (Character c: n.next.keySet()) {
            colHelp(s + c, x, n.next.get(c));
        }
    }
}
```

### 26.4. Autocomplete
#### 26.4.1 The Autocomplete Problem
Example, when I type "how are" into Google, I get 10 results, shown to the below.

One way to do this is to create a **Trie based map** from strings to values
- Value represents how important Google thinks that string is.
- Can store billions of strings efficiently since they share nodes.
- When a user types in a string "hello", we:
    - Call `keysWithPrefix("hello")`.
    - Return the 10 strings with the highest value.

![google_search](26/google_search.png){:w="300"}

The approach above has one major flaw. If we enter a short string, the number of keys with the appropriate prefix will be too big.
- We are collecting billions of results only to keep 10.
- This is extremely inefficient.

![google_search_2](26/google_search_2.png){:w="300"}

#### 26.4.2 A More Efficient Autocomplete
![efficient_autocomplete](26/efficient_autocomplete.png){:w="300"}

One way to address this issue:
- Each node stores its own value, as well as the value of its best substring.

Search will consider nodes in order of "best".
- Consider 'sp' before 'sm'.
- Can stop when top 3 matches are all better than best remaining.

#### 26.4.3 Even More Efficient Autocomplete
![more_efficient_autocomplete](26/more_efficient_autocomplete.png){:w="300"}

Can also merge nodes that are redundant.
- This version of trie is known as a "**radix tree**" or "radix trie".
- Won’t discuss.

### 26.5 Summary
When your key is a string, you can use a Trie:
- Theoretically better performance than hash table or search tree.
- Have to decide on a mapping from letter to node. Three natural choices:
    - DataIndexedCharMap, i.e. an array of all possible child links.
    - Bushy BST.
    - Hash Table.
- All three choices are fine, though hash table is probably the most natural.
- Supports special string operations like `longestPrefixOf` and `keysWithPrefix`.
    - `keysWithPrefix` is the heart of important technology like autocomplete.
    - Optimal implementation of Autocomplete involves use of a priority queue.

---
layout: post
title: CS 61B (Sp21) Notes 3, Algorithms
categories: course
tags: [data structures, algorithms]
toc:
  sidebar: right
description: >
  Shortest Path, Minumum Spanning Trees, Tries
media_subpath: /assets/img/cs61b/
thumbnail: /assets/img/cs61b/cs61b-logo.png
---

## 23. Shortest Paths
### Introduction
BFS would NOT be a good choice for a google maps style navigation application. BFS returns path with shortest <u class="under_mark">number of edges</u>, not necessarily the shortest path.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/23/bfs_example.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
BFS yields a route of length ~330 m instead of ~150 m.
</div>

We need an algorithm that takes into account edge distances, also known as edge weights.

#### The Shortest Paths Tree
**Single Source Single Target Shortest Path**: Find the shortest paths from source vertex `s` to some target vertex `t`.
<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/23/single_source_single_target.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
Observation: Solution will always be a path with no cycles (assuming non-negative weights).
</div>

**Single Source Shortest Path**: Find the shortest paths from source vertex `s` to every other vertex.
<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/23/single_source.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
Observation: Solution will always be a tree. <br>
(Can think of as the union of the shortest paths to all vertices.) 
</div>

This is so-called **Shortest Paths Tree** (SPT).

### Dijkstra's Algorithm

{% include audio.liquid path="assets/audio/cs61b/Dijkstra.mp3" controls=true %}

#### Algorithm
We can find the SPT using **Dijkstra's algorithm** [[Demo](https://docs.google.com/presentation/d/1_bw2z1ggUkquPdhl7gwdVBoTaoJmaZdpkV6MoAgxlJc/pub?start=false&loop=false&delayms=3000&slide=id.g771336078_0_180)]. Perform a <u class="under_mark">best first search</u>: Visit vertices in <u class="under_mark">order of best-known distance</u> from source. On visit, ***relax*** every edge from the visited vertex. 

> **Relax**: If we find a better path, update our preferred edge and distance.
{:.prompt-info}

```java
// Dijkstra’s shortest-paths algorithm
public class DijkstraSP {
  private DirectedEdge[] edgeTo;
  private double[] distTo;
  private IndexMinPQ<Double> pq;
  
  public DijkstraSP(EdgeWeightedDigraph G, int s) {
    edgeTo = new DirectedEdge[G.V()];
    distTo = new double[G.V()];
    pq = new IndexMinPQ<Double>(G.V());
    
    for (int v = 0; v < G.V(); v++)
      distTo[v] = Double.POSITIVE_INFINITY;
    distTo[s] = 0.0;

    pq.insert(s, 0.0);
    while (!pq.isEmpty()) {
      relax(G, pq.delMin())
    }
  }

  private void relax(EdgeWeightedDigraph G, int v) {
    for(DirectedEdge e : G.adj(v)) {
      int w = e.to();
      if (distTo[w] > distTo[v] + e.weight()) {
        distTo[w] = distTo[v] + e.weight();
        edgeTo[w] = e;
        if (pq.contains(w)) pq.change(w, distTo[w]);
        else                pq.insert(w, distTo[w]);
      }
    }
  }
}
```

#### Correctness
Dijkstra's is guaranteed to return a correct result if <u class="under_mark">all edges are non-negative</u>. Proof relies on the property that relaxation always fails on edges to visited (white) vertices.

Proof sketch: Assume all edges have non-negative weights.
At start, `distTo[source]` = `0`, which is optimal.
After relaxing all edges from `source`, let vertex `v1` be the vertex with minimum weight, i.e. that is closest to the `source`. Claim: `distTo[v1]` is optimal, and thus future relaxations will fail. Why? 
- `distTo[p]`       ≥ `distTo[v1]` for all `p`, therefore
- `distTo[p]` + `w` ≥ `distTo[v1]`

Can use induction to prove that this holds for all vertices after dequeuing.

#### Runtime
Priority Queue operation count, assuming binary heap based PQ:
- `add`: $V$, each costing $O(\log V)$ time.
- `removeSmallest`: $V$, each costing $O(\log V)$ time.
- `changePriority`: $E$, each costing $O(\log V)$ time.

Overall runtime: $O(V\log V +V\log V +E\log V)$. Assuming $E$ > $V$, this is just $O(E\log V)$ for a connected graph.

### A* Algorithm
If we have only <u class="under_mark">a single target</u> in mind, A* algorithm can do better:
>**A* Algorithm** Visit vertices in order of `d(source, v) + h(v, goal)`, where `h(v, goal)` is an estimate of the distance from v to our goal. [[Demo](https://docs.google.com/presentation/d/177bRUTdCa60fjExdr9eO04NHm0MRfPtCzvEup1iMccM/edit#slide=id.g771336078_0_180)]
{:.prompt-tip}

<hr class="dotted">

## 24. Minimum Spanning Trees
### Introduction
Given an **undirected** graph `G`, a **spanning tree** `T` is a subgraph of `G`, where `T`:
- Is connected and acyclic. (make it a tree)
- Includes all of the vertices. (makes it spanning)

<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/24/mst.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

A ***minimum spanning tree*** (MST) is a spanning tree of <u class="under_mark">minimum total weight</u>.

### The Cut Property
#### Terms
- A **cut** is an assignment of a graph's nodes to <u class="under_mark">two non-empty sets</u>.
- A **crossing edge** is an edge which connects a node from one set to a node from the other set.
- **Cut property**: Given any cut, minimum weight crossing edge is in the MST. (assume edge weights are unique)

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/24/cut_property.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

#### Proof
Suppose that the minimum crossing edge `e` were not in the MST.
- Adding `e` to the MST creates a cycle.
- Some other edge `f` must also be a crossing edge.
- Removing `f` and adding `e` is a lower weight spanning tree.
Contradiction!

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/24/cut_property_proof.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

### Algorithms
#### Prim's Algorithm
**Prim's Algorithm** [[Demo](https://docs.google.com/presentation/d/1NFLbVeCuhhaZAM1z3s9zIYGGnhT4M4PWwAc-TLmCJjc/edit#slide=id.g9a60b2f52_0_0)]: Start from some arbitrary start node. Add shortest edge that has **one** node inside the MST under construction. Repeat until $V-1$ edges. 

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/24/prim.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

**Implementation** [[Demo](https://docs.google.com/presentation/d/1GPizbySYMsUhnXSXKvbqV4UhPCvrt750MiqPPgU-eCY/edit#slide=id.g9a60b2f52_0_0)]: Insert all vertices into fringe `PQ`, storing vertices in order of distance from tree.
Repeat: Remove (closest) vertex `v` from `PQ`, and relax all edges pointing from `v`. 

#### Kruskal's Algorithm
**Kruskal's Algorithm** [[Demo](https://docs.google.com/presentation/d/1RhRSYs9Jbc335P24p7vR-6PLXZUl-1EmeDtqieL9ad8/edit#slide=id.g9a60b2f52_0_0)]: Consider edges in order of increasing weight. Add to MST unless a cycle is created. Repeat until $V-1$ edges.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/24/kruskals.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
Blue and green colorings for vertices show cut being implicitly utilized by Kruskal’s algorithm.
</div>

**Implementation** [[Demo](https://docs.google.com/presentation/d/1KpNiR7aLIEG9sm7HgX29nvf3yLD8_vdQEPa0ktQfuYc/edit?usp=sharing)]: Insert all edges into `PQ`.
Repeat: Remove smallest weight edge. Add to MST if no cycle created.
- Use Weighted Quick Union(WQU) UF to detech cycle: For each edge, check if the two vertices are connected. If not, union them. If so, there is a cycle.

<hr class="dotted">

## 25. Prefix Operations and Tries
### Introduction
HashMaps are already incredibly fast. But if we have some additional insight on the data we are storing, we could get even faster.

#### Character Keyed Map

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

#### String Keyed Map
Suppose we know that our keys are always strings. We can use a special data structure known as a **Trie**. The basic idea: store <u class="under_mark">each letter</u> of the string as a node in a tree.

> **Trie**: Short for Re**trie**val Tree. Inventor _Edward Fredkin_ suggested it should be pronounced "tree", but almost everyone pronounces it like "try".

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/26/trie_map.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
Use Trie to implement Map
</div>

### Implementation
#### Basic Trie Implementation
The first approach might look something like the code below. Each node stores a letter (`ch`), a map from `ch` to all child nodes (`next`), and a color (`isKey`).

```java
public class TrieSet {
    private static final int R = 128; // ASCII
    private Node root;	// root of trie
    
    private static class Node {
        private char ch;
        private boolean isKey;
        // Since we know our keys are characters,
        // can use a DataIndexedCharMap.   
        private DataIndexedCharMap<Node> next;
        private Node(char c, boolean b, int R) {
            ch = c;
            isKey = b;
            next = new DataIndexedCharMap<>(R);
        }
    }
}
```
Observation: The letter stored inside each node is actually redundant. Can remove from the representation and things will work fine.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/26/basic_impl.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

```java
public class TrieSet {
    private static final int R = 128;
    private Node root;
    
    private static class Node {
        private boolean isKey;
        private DataIndexedCharMap<Node> next;
        private Node(boolean b, int R) {
            isKey = b;
            next = new DataIndexedCharMap<>(R);
        }
    }
}
```

Assuming the length of the key is $L$, the runtime of `add` is $\Theta(L)$ and that of `contain` is $O(L)$.

#### Alternate Child Tracking Strategies
Using a `DataIndexedCharMap` is very memory hungry. Every node has to store `R` links, most of which are `null`. Using BST or Hash Table will be slightly slower, but more memory efficient.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/26/implementation.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
The Hash-Table Based Trie (mid) and The BST-Based Trie (right)
</div>

#### Performance
Using a BST or a Hash Table to store links to children will usually use less memory.
- DataIndexedCharMap: `128` links per node.
- BST: $C$ links per node, where $C$ is the number of children.
- Hash Table: $C$ links per node.
- Note: Cost per link is higher in BST and Hash Table.

Using a BST or a Hash Table will take slightly more time.
- DataIndexedCharMap is $\Theta(1)$.
- BST is $O(\log R)$, where $R$ is size of alphabet.
- Hash Table is $O(R)$, where $R$ is size of alphabet.
- Since $R$ is fixed (e.g. 128), can think of all 3 as $\Theta(1)$.


### Trie String Operations
Theoretical asymptotic speed improvement is nice. But the main appeal of tries is their ability to efficiently support string specific operations like **prefix matching**.
- Finding all keys that match a given prefix: `keysWithPrefix("sa")`
- Finding longest prefix of: `longestPrefixOf("sample")`

#### Collecting Trie Keys
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
            colHelp(new StringBuilder(c), x, root.next.get(c));
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

#### keysWithPrefix
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

### Autocomplete
#### The Autocomplete Problem
Example, when I type "how are" into Google, I get 10 results, shown to the below.

One way to do this is to create a **Trie based map** from strings to values
- Value represents how important Google thinks that string is.
- Can store billions of strings efficiently since they share nodes.
- When a user types in a string "hello", we:
    - Call `keysWithPrefix("hello")`.
    - Return the 10 strings with the highest value.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/26/google_search.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

The approach above has one major flaw. If we enter a short string, the number of keys with the appropriate prefix will be too big. We are collecting billions of results only to keep 10. This is extremely inefficient.

#### A More Efficient Autocomplete
One way to address this issue: Each node stores its own value, as well as the value of its best substring. Search will consider nodes in order of "best".
- Consider `'sp'` before `'sm'`.
- Can stop when top 3 matches are all better than best remaining.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/26/efficient_autocomplete.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

#### Even More Efficient Autocomplete
Can also merge nodes that are redundant. This version of trie is known as a **radix tree** or **radix trie**.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="/assets/img/cs61b/26/more_efficient_autocomplete.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

### Summary
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


# Java Data Structures Deep Dive (Part 4)
# Graphs, DFS, BFS, Dijkstra, Topological Sort, Union-Find, MST

Level: SDE-2 / Senior Backend Engineer

---

# Table of Contents

1. What is a Graph?
2. Graph Terminology
3. Types of Graphs
4. Graph Representations
5. Adjacency Matrix
6. Adjacency List
7. Graph Traversal
8. DFS (Depth First Search)
9. BFS (Breadth First Search)
10. Cycle Detection
11. Connected Components
12. Topological Sort
13. Dijkstra Algorithm
14. Bellman-Ford
15. Floyd-Warshall
16. Union Find (Disjoint Set)
17. Path Compression
18. Union By Rank
19. Minimum Spanning Tree
20. Kruskal Algorithm
21. Prim Algorithm
22. Real World Applications
23. Interview Questions
24. Cheat Sheet

---

# 1. What is a Graph?

A Graph is a data structure used to model relationships.

---

Examples

```text
Google Maps
Social Networks
Airline Routes
Dependency Graphs
Microservices
Computer Networks
```

---

Mathematically

```text
Graph = Vertices + Edges
```

---

Example

```text
A ----- B
|       |
|       |
C ----- D
```

Vertices

```text
A B C D
```

Edges

```text
A-B
A-C
B-D
C-D
```

---

# Why Graphs?

Arrays

```text
Store Data
```

Trees

```text
Store Hierarchical Data
```

Graphs

```text
Store Relationships
```

---

# 2. Graph Terminology

---

## Vertex (Node)

```text
A
B
C
```

---

## Edge

Connection between nodes.

```text
A ---- B
```

---

## Degree

Number of connected edges.

Example

```text
A connected to B,C,D
```

Degree:

```text
3
```

---

## Path

Sequence of nodes.

```text
A → B → D
```

---

## Cycle

Path returning to same node.

```text
A → B → C → A
```

---

# 3. Types of Graphs

---

## Undirected Graph

```text
A ----- B
```

Connection both ways.

---

Example

```text
Facebook Friends
```

---

## Directed Graph

```text
A → B
```

One-way relationship.

---

Example

```text
Twitter Follow
```

---

## Weighted Graph

Edges contain weights.

```text
A --10--> B
```

---

Example

```text
Road Distance
```

---

## Unweighted Graph

```text
A → B
```

No cost information.

---

# 4. Graph Representations

Two major representations.

---

## Adjacency Matrix

---

## Adjacency List

---

Interview favorite.

---

# 5. Adjacency Matrix

Example

```text
A -- B
|    |
C -- D
```

---

Matrix

```text
      A B C D

A     0 1 1 0
B     1 0 0 1
C     1 0 0 1
D     0 1 1 0
```

---

Meaning

```text
1 = Edge Exists

0 = No Edge
```

---

Space Complexity

```text
O(V²)
```

---

Advantages

```text
Fast Edge Lookup
```

---

Check edge:

```text
A → B
```

Complexity

```text
O(1)
```

---

Disadvantages

```text
Huge Memory Usage
```

for sparse graphs.

---

# 6. Adjacency List

Most used representation.

---

Example

```text
A -- B
|    |
C -- D
```

---

Representation

```text
A → B,C

B → A,D

C → A,D

D → B,C
```

---

Java

```java
Map<Integer, List<Integer>>
```

---

Example

```java
Map<Integer,List<Integer>> graph
```

---

Space Complexity

```text
O(V + E)
```

---

Preferred for:

```text
Large Sparse Graphs
```

---

# Adjacency Matrix vs List

| Feature | Matrix | List |
|----------|----------|----------|
| Space | O(V²) | O(V+E) |
| Edge Lookup | O(1) | O(degree) |
| Traversal | O(V²) | O(V+E) |

---

# 7. Graph Traversal

Need to visit all nodes.

---

Two methods

```text
DFS
BFS
```

---

# 8. DFS (Depth First Search)

Go deep first.

---

Example

```text
A
|
B
|
C
|
D
```

Traversal

```text
A
B
C
D
```

---

Uses

```text
Stack
```

internally.

---

Recursive DFS

```java
void dfs(node){

    visited[node] = true;

    for(neighbor)
        dfs(neighbor);
}
```

---

Example

```text
      A
     / \
    B   C
   /
  D
```

DFS

```text
A B D C
```

---

Complexity

```text
O(V + E)
```

---

Applications

```text
Cycle Detection
Connected Components
Maze Problems
Topological Sort
```

---

# DFS Internal Working

Call Stack

```text
dfs(A)
  dfs(B)
    dfs(D)
```

Stack

```text
D
B
A
```

---

# 9. BFS (Breadth First Search)

Visits level by level.

---

Uses

```text
Queue
```

---

Example

```text
      A
     / \
    B   C
   /
  D
```

BFS

```text
A B C D
```

---

Process

```text
Queue

A

Remove A

Add B,C

Remove B

Add D
```

---

Complexity

```text
O(V + E)
```

---

Applications

```text
Shortest Path
Level Order Traversal
Social Networks
Recommendation Engines
```

---

# BFS Finds Shortest Path

Important interview point.

---

Unweighted graph.

```text
A → B → D
```

and

```text
A → C → D
```

---

BFS guarantees:

```text
Minimum number of edges.
```

---

DFS does not.

---

# 10. Cycle Detection

Very common interview question.

---

## Undirected Graph

Use:

```text
DFS + Parent Tracking
```

---

Example

```text
A
| \
|  \
B---C
```

Cycle exists.

---

Condition

```text
Visited neighbor
and
neighbor != parent
```

---

Cycle found.

---

# Directed Graph Cycle Detection

Use:

```text
DFS
+
Recursion Stack
```

---

States

```text
Unvisited
Visiting
Visited
```

---

Encounter:

```text
Visiting Node Again
```

means cycle.

---

# 11. Connected Components

Question:

```text
How many disconnected groups?
```

---

Example

```text
A-B

C-D

E
```

Components:

```text
3
```

---

Algorithm

```text
Run DFS/BFS
from every unvisited node.
```

---

Complexity

```text
O(V+E)
```

---

# 12. Topological Sort

Applicable only to:

```text
DAG
```

(Directed Acyclic Graph)

---

Example

```text
Login

↓

Cart

↓

Payment
```

---

Need dependency order.

---

Output

```text
Login
Cart
Payment
```

---

Applications

```text
Build Systems
Dependency Resolution
Task Scheduling
Course Scheduling
```

---

# DFS Based Topological Sort

Process

```text
DFS

Add node after exploring
all children.
```

---

Store in stack.

---

Reverse stack.

---

Complexity

```text
O(V+E)
```

---

# Kahn's Algorithm

Uses:

```text
In-Degree
+
Queue
```

---

Steps

1.

```text
Find nodes with indegree 0
```

2.

```text
Push into queue
```

3.

```text
Remove node
```

4.

```text
Reduce indegree
```

---

# 13. Dijkstra Algorithm

Most asked graph algorithm.

---

Purpose

```text
Shortest Path
```

in weighted graph.

---

Condition

```text
No Negative Weights
```

---

Example

```text
A --4--> B

A --1--> C

C --2--> B
```

Shortest

```text
A → C → B
```

Cost

```text
3
```

---

# Core Idea

Always expand:

```text
Current Minimum Distance Node
```

---

Uses

```text
PriorityQueue (Min Heap)
```

---

Data Structures

```java
distance[]
visited[]

PriorityQueue
```

---

Complexity

```text
O((V+E) log V)
```

---

Applications

```text
Google Maps
Network Routing
Uber
GPS
```

---

# Dijkstra Internal Working

Initialize

```text
Source = 0

Others = ∞
```

---

Process

```text
Pick smallest node

Relax neighbors

Update distances
```

---

Repeat.

---

# Why PriorityQueue?

Need:

```text
Fast access to
minimum distance node.
```

---

Min Heap provides

```text
O(log n)
```

updates.

---

# 14. Bellman-Ford

Handles:

```text
Negative Weights
```

---

Unlike Dijkstra.

---

Process

```text
Relax all edges
V-1 times
```

---

Complexity

```text
O(VE)
```

---

Can detect:

```text
Negative Cycles
```

---

# 15. Floyd-Warshall

Finds:

```text
All Pair Shortest Paths
```

---

Example

```text
A→B

A→C

B→D

C→D
```

---

Output

```text
Shortest distance between
every pair.
```

---

Complexity

```text
O(V³)
```

---

Used when:

```text
Graph Small
Need all pair paths
```

---

# 16. Union Find (Disjoint Set)

Most important advanced DS.

---

Purpose

```text
Determine whether
two nodes belong to same group.
```

---

Operations

```text
Find
Union
```

---

Example

```text
1 2 3 4
```

Initially separate.

---

Union

```text
1,2

2,3
```

Now

```text
1,2,3
```

same set.

---

# Internal Structure

Array

```text
Parent

1→1

2→2

3→3
```

---

After union

```text
1→1

2→1

3→1
```

---

# 17. Path Compression

Most asked Union-Find topic.

---

Before

```text
4→3→2→1
```

---

Find(4)

Traverses:

```text
4
3
2
1
```

---

Compress

```text
4→1

3→1

2→1
```

---

Future lookup

```text
O(1)
```

almost.

---

# 18. Union By Rank

Avoid tall trees.

---

Always attach:

```text
Smaller Tree
```

under

```text
Larger Tree
```

---

Result

```text
Very shallow structure.
```

---

Combined with:

```text
Path Compression
```

Complexity:

```text
Almost O(1)
```

---

# 19. Minimum Spanning Tree (MST)

Interview favorite.

---

Definition

```text
Connect all nodes
with minimum edge cost.
```

---

Conditions

```text
No Cycles
All Nodes Connected
```

---

Example

```text
A-B = 1

B-C = 2

A-C = 5
```

---

Choose

```text
A-B
B-C
```

Cost:

```text
3
```

---

# 20. Kruskal Algorithm

Uses:

```text
Sorting
+
Union Find
```

---

Steps

1.

```text
Sort edges
```

2.

```text
Pick smallest edge
```

3.

```text
If cycle not formed
take edge
```

---

Complexity

```text
O(E log E)
```

---

# Why Union Find?

Need quick answer:

```text
Will adding edge
create cycle?
```

---

# 21. Prim Algorithm

Another MST algorithm.

---

Uses

```text
PriorityQueue
```

---

Idea

```text
Grow MST
one node at a time.
```

---

Complexity

```text
O(E log V)
```

---

# Kruskal vs Prim

| Feature | Kruskal | Prim |
|----------|----------|----------|
| Uses | Union Find | PriorityQueue |
| Edge Based | Yes | No |
| Dense Graph | Slower | Better |
| Sparse Graph | Excellent | Good |

---

# 22. Real World Applications

## Social Networks

Graph

```text
User → Friends
```

---

## Google Maps

Graph

```text
Cities → Roads
```

---

Uses

```text
Dijkstra
```

---

## Build Systems

```text
Maven
Gradle
```

Use:

```text
Topological Sort
```

---

## Network Routing

Uses

```text
Shortest Path Algorithms
```

---

## Recommendation Systems

Graph

```text
Users
Products
Relationships
```

---

## Kubernetes

Dependency graph.

---

# 23. Common Interview Questions

---

## DFS vs BFS?

DFS

```text
Stack
Go Deep
```

BFS

```text
Queue
Level Order
```

---

## Which Finds Shortest Path?

```text
BFS
```

(Unweighted graph)

---

## Why Dijkstra Uses PriorityQueue?

```text
Need minimum distance node quickly.
```

---

## Why Union Find Fast?

```text
Path Compression
+
Union By Rank
```

---

## When Topological Sort Possible?

```text
Only DAG
```

---

## Difference Between MST and Shortest Path?

MST

```text
Connect all nodes
minimum total cost
```

Shortest Path

```text
Source → Destination
minimum cost
```

---

# 24. Final Cheat Sheet

## DFS

```text
Stack
O(V+E)
```

---

## BFS

```text
Queue
O(V+E)
```

Shortest path in unweighted graph.

---

## Dijkstra

```text
PriorityQueue
O((V+E)logV)
```

No negative weights.

---

## Bellman Ford

```text
Negative Weights Supported
O(VE)
```

---

## Topological Sort

```text
DAG Only
O(V+E)
```

---

## Union Find

```text
Find
Union
Path Compression
Union By Rank
```

Near O(1).

---

## Kruskal

```text
Sort Edges
Union Find
```

---

## Prim

```text
PriorityQueue
Grow Tree
```

---

# Golden Interview Answer

> Graphs model relationships between entities and are commonly represented using adjacency lists for space efficiency. DFS uses a stack (or recursion) to explore deeply, while BFS uses a queue to explore level-by-level and guarantees shortest paths in unweighted graphs. Dijkstra's algorithm finds shortest paths in weighted graphs using a priority queue. Union-Find, enhanced with path compression and union-by-rank, provides near O(1) connectivity checks and powers Kruskal's MST algorithm. Topological sorting is used for dependency resolution in Directed Acyclic Graphs (DAGs), making it fundamental to build systems, workflow engines, and task schedulers.

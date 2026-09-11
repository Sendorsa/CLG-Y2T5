# Graph Algorithms Notes (Based on Class Lecture)

## Topics Covered
1. Matrix as a Graph
2. Rotting Oranges Problem (Multi-Source BFS)
3. Connected Components
4. Weighted Graphs
5. Dijkstra's Algorithm
6. Minimum Spanning Tree (Prim's Algorithm)
7. Bellman-Ford Algorithm
8. Negative Cycle Detection

---

# 1. Matrix as a Graph

A matrix can be treated as a graph where:

- Each cell = Node
- Neighbours: Up, Down, Left, Right
- Sometimes diagonal neighbours may also be considered

Source: ClassScaler Academy — "Rotting Oranges" interview problem (LeetCode #994)

---

# 2. Rotting Oranges Problem (Multi-Source BFS)

## Problem Statement

You are given a 2D matrix representing a grid of oranges with three possible values:

| Value | Meaning        | Description              |
|-------|----------------|--------------------------|
| `0`   | Empty Cell     | No orange present        |
| `1`   | Fresh Orange   | Can rot if adjacent to rotten |
| `2`   | Rotten Orange  | Spreads rot to neighbors |

## Rules

- Every day, fresh oranges sharing a boundary (up, down, left, right) with a rotten orange become rotten.
- Diagonal adjacency does **not** count — only direct neighbors.
- All infections happen simultaneously.
- Return the number of days until all oranges rot, or `-1` if impossible.

## Example Grid

```
[2, 1, 1]
[1, 1, 0]
[0, 1, 1]
```

## Key Approach: Multi-Source BFS

Instead of starting BFS from a single source:

```text
BFS(source1)
BFS(source2)
BFS(source3)
```

Use a queue with all sources simultaneously:

```text
Queue = [source1, source2, source3]
```

Each BFS level corresponds to one day.
Number of levels = number of days.

## Companies Where This Question Has Appeared

- Amazon
- Microsoft
- Adobe

## Time Complexity

```text
O(rows × cols)
```

---

# 3. Connected Components

A graph may contain multiple disconnected groups of nodes.

## Example

```
A -- B

C -- D -- E

F
```

Connected Components:

```
{A,B}
{C,D,E}
{F}
```

## Finding Components

```python
for each node:
    if not visited:
        BFS/DFS(node)
        count += 1
```

### Complexity

```text
Time: O(V + E)
```

---

# 4. Weighted Graphs

Edges carry weights representing:

- Distance
- Time
- Fuel
- Cost

## Shortest Path Problem

Find the minimum cost path from a source node to all other nodes.

---

# 5. Dijkstra's Algorithm

## Condition

Works only when:

```text
All edge weights >= 0
```

## Core Idea

Always process the node with the smallest currently known distance.

Maintain `dist[]`:

```python
dist[source] = 0
dist[others] = infinity
```

## Relaxation

For an edge `u --w--> v`:

```python
if dist[u] + w < dist[v]:
    dist[v] = dist[u] + w
```

This process is called **Relaxation**.

## Data Structure Used

**Priority Queue (Min Heap)** — stores `(node, distance)`, always removes the minimum distance node.

## Pseudocode

```cpp
dist[source] = 0
PQ.push(source)

while(!PQ.empty())
{
    u = PQ.top()

    for(each neighbour v)
    {
        if(dist[u] + weight(u,v) < dist[v])
        {
            dist[v] = dist[u] + weight(u,v)
            PQ.push(v)
        }
    }
}
```

## Complexity

| Operation | Cost |
|-----------|------|
| Insert PQ | O(log V) |
| Delete Min | O(log V) |

**Overall Time:** `O(E log V)`
**Space:** `O(V)`

---

# 6. Minimum Spanning Tree (MST) — Prim's Algorithm

## Goal

Connect all vertices:

- Without cycles
- Using minimum total edge weight

### Properties

For a graph with V vertices:

```text
Edges in MST = V - 1
```

Must satisfy: connected, acyclic, minimum total weight.

## Prim's Algorithm — Idea

Start from any node. Repeatedly choose the **minimum edge connecting visited → unvisited**. Continue until all vertices become part of the MST.

### Complexity (using Priority Queue)

```text
O(E log V)
```

---

# 7. Bellman-Ford Algorithm

## Use Case

Used when graph contains **negative edge weights** — which Dijkstra cannot handle.

## Initialization

```python
dist[source] = 0
dist[others] = infinity
```

## Main Idea

Relax every edge repeatedly for `V - 1` times:

```cpp
dist[source] = 0

for(i = 1; i <= V-1; i++)
{
    for(all edges)
    {
        relax(edge)
    }
}
```

## Why V−1 Times?

A shortest path can contain at most `V - 1` edges. After `V - 1` passes, all shortest paths are guaranteed to be found.

### Complexity

| Metric | Value |
|--------|-------|
| Per iteration | E edges |
| Total iterations | V − 1 |
| **Overall Time** | **O(VE)** |
| Space | O(V) |

---

# 8. Negative Cycle Detection

## Definition

A cycle whose total weight is negative.

### Example

```
A -> B = 1
B -> C = -10
C -> A = -2

Total: 1 + (-10) + (-2) = -11 (negative cycle)
```

## Detection Rule

After running Bellman-Ford for `V - 1`:

Perform **one additional pass** over all edges. If any edge can still relax:

```python
if dist[u] + wt < dist[v]:
```

Then a **Negative Cycle Exists**.

---

# Why Dijkstra Fails for Negative Weights

### Example

```
A -> B = 5        (dist[B] = 5)
A -> C = 6        (dist[C] = 6)
C -> B = -4       (shortest A->B = 2, not 5!)
```

Shortest path: `A -> C -> B = 6 + (-4) = 2`

Dijkstra may finalize B too early (with distance 5) and never reconsider the cheaper path through C.

---

# Quick Comparison Table

| Algorithm | Use Case | Negative Weights? | Time Complexity |
|-----------|----------|-------------------|-----------------|
| BFS | Unweighted shortest path | Yes | O(V+E) |
| DFS | Traversal | Yes | O(V+E) |
| Multi-Source BFS | Rotting Oranges | Yes | O(V+E) |
| Dijkstra | Shortest Path (non-negative) | No | O(E log V) |
| Prim's | MST | Yes | O(E log V) |
| Bellman-Ford | Shortest Path (negative allowed) | Yes | O(VE) |
| Bellman-Ford + Extra Pass | Negative Cycle Detection | Yes | O(VE) |

---

# Important Viva Questions

1. Why does Dijkstra fail for negative edges?
2. What is relaxation?
3. Why does Bellman-Ford run V−1 times?
4. What is a negative cycle?
5. How is a negative cycle detected?
6. Difference between Prim and Dijkstra?
7. What is Multi-Source BFS?
8. Why use a Priority Queue in Dijkstra?
9. Time complexity of Dijkstra?
10. Difference between MST and Shortest Path Tree?

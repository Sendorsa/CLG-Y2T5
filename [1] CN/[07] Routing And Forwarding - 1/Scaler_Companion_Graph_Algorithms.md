# Scaler Companion — Graph Algorithms & Applications

_Revision Notes (Cleaned & Consolidated)_

---

## Overview

In this session from ClassScaler Academy, we explored graph traversal techniques and shortest path algorithms:

- Rotting Oranges problem using **Multi-Source BFS** to simulate spread within a matrix
- **Dijkstra's algorithm** for shortest paths with non-negative weights
- **Bellman-Ford algorithm** for graphs allowing negative edge weights and detecting negative cycles

Emphasis was placed on understanding when and why to use each algorithm, their time complexities, and the nature of graph edges.

---

## 1. Rotting Oranges Problem (Multi-Source BFS)

### Problem Statement

You are given a 2D grid/matrix with the following cell types:

| Value | Meaning | Description |
|-------|---------|-------------|
| `0`   | Empty cell | No orange present |
| `1`   | Fresh orange | Can rot if adjacent to rotten |
| `2`   | Rotten orange | Spreads rot to neighbors each day |

**Goal:** Determine the minimum number of days for all fresh oranges to become rotten. If any fresh orange cannot be reached, return `-1`.

### Rules

- Every day, fresh oranges sharing a **direct boundary** (up, down, left, right) with a rotten orange become rotten.
- Diagonal adjacency does **not** count.
- All infections happen **simultaneously**.

### Solution Approach: Multi-Source BFS

Instead of starting BFS from a single source, initialize the queue with all rotten oranges simultaneously — each BFS level corresponds to one day.

### Example

The source from ClassScaler Academy used this grid (empty cell at bottom-left):

```
Initial:    Day 1:     Day 2:     Day 3:
2 1 0        2 2 0      2 2 0      2 2 0
1 1 0   →    2 2 0  →   2 2 0  →   2 2 0
0 1 2        0 2 2      0 2 2      0 2 2

Total Days: 3
```

> **Note on unreachable oranges:** If a fresh orange has no path to any rotten orange (blocked entirely by empty cells), return `-1`. Example:
> ```
> 2 0 0
> 0 1 0
> 0 0 0
> → -1 (the fresh orange at (1,1) is isolated by empty cells)
> ```

### Key Implementation Details

- **Queue initialization:** Add all rotten oranges (`2`) to the BFS queue at time `t = 0`.
- **Track visited cells:** Use a `visited[]` matrix to avoid reprocessing.
- **Count days:** Each BFS level (all nodes popped before processing the next level) = 1 day.

```python
days = 0
while queue:
    size = len(queue)
    rotted_any = False
    for _ in range(size):
        r, c = queue.popleft()
        for dr, dc in directions:
            nr, nc = r + dr, c + dc
            if valid(nr, nc) and grid[nr][nc] == 1:
                grid[nr][nc] = 2      # rot it
                queue.append((nr, nc))
                rotted_any = True
    if rotted_any:
        days += 1
```

### Time Complexity

| Metric | Value |
|--------|-------|
| Time | O(rows × cols) = O(V + E) |
| Space | O(rows × cols) for queue |

### Companies Where This Question Has Appeared

- Amazon
- Microsoft
- Adobe

---

## 2. Dijkstra's Algorithm

### When to Use

Graphs with **non-negative edge weights** only. Works from a single source to find shortest paths to all other nodes.

### Core Idea

Always process the node with the smallest currently known distance using a **Priority Queue (Min Heap)**.

### Key Concepts

**Distance array `dist[]`:**
```python
dist[source] = 0
dist[others] = ∞
```

**Relaxation:** For edge `u --w--> v`:
```python
if dist[u] + w < dist[v]:
    dist[v] = dist[u] + w
    PQ.push(v)
```

### Pseudocode (C++)

```cpp
dist[source] = 0;
PQ.push({source, 0});           // {node, distance}

while (!PQ.empty())
{
    auto [u, d] = PQ.top(); PQ.pop();
    
    for each neighbor v with weight w:
        if (d + w < dist[v])
        {
            dist[v] = d + w;
            PQ.push({v, dist[v]});
        }
}
```

### Complexity

| Operation | Cost |
|-----------|------|
| Insert into PQ | O(log V) |
| Delete min from PQ | O(log V) |
| **Overall Time** | **O(E log V)** |
| **Space** | **O(V)** |

### Why Dijkstra Fails for Negative Weights

Dijkstra may finalize a node too early and never reconsider cheaper paths through negative edges.

Example:
```
A → B = 5       (Dijkstra finalizes dist[B] = 5)
A → C = 6        (dist[C] = 6)
C → B = -4       (shortest A→B is actually 2!)
```

Shortest path: `A → C → B = 6 + (-4) = 2` — but Dijkstra misses it.

---

## 3. Minimum Spanning Tree (Prim's Algorithm)

### Goal

Connect all vertices with minimum total edge weight, without cycles.

**Properties:** For a graph with V vertices:
```
Edges in MST = V − 1
Connected, acyclic, minimum total weight
```

### Prim's Algorithm — Strategy

Start from any node. Repeatedly choose the **minimum-weight edge connecting visited → unvisited** vertices. Continue until all vertices are in the MST.

### Complexity

| Metric | Value |
|--------|-------|
| Time (with Priority Queue) | O(E log V) |
| Space | O(V) |

---

## 4. Bellman-Ford Algorithm

### When to Use

Graphs that contain **negative edge weights** — which Dijkstra cannot handle. Also detects negative cycles.

### Initialization

```python
dist[source] = 0
dist[others] = ∞
```

### Main Idea

Relax every edge repeatedly for `V − 1` passes:

```cpp
for (i = 1; i <= V - 1; i++)
    for each edge (u, v, w):
        relax(u, v, w)
```

### Why V − 1 Times?

A shortest path can contain at most `V − 1` edges. After `V − 1` passes, all shortest paths are guaranteed to be found.

### Complexity

| Metric | Value |
|--------|-------|
| Per iteration | E edge relaxations |
| Total iterations | V − 1 |
| **Overall Time** | **O(VE)** |
| Space | O(V) |

---

## 5. Negative Cycle Detection

### Definition

A cycle whose total weight is negative — meaning the path cost decreases indefinitely by traversing the cycle repeatedly.

### Example:
```
A → B = 1
B → C = -10
C → A = -2
Total: 1 + (-10) + (-2) = -11 (negative cycle!)
```

### Detection Rule

After running Bellman-Ford for `V − 1` iterations, perform **one additional pass** over all edges. If any edge can still relax:

```python
if dist[u] + w < dist[v]:
    → Negative Cycle Exists!
```

---

## Quick Comparison Table

| Algorithm | Use Case | Handles Neg. Weights? | Time Complexity |
|-----------|----------|----------------------|-----------------|
| BFS | Unweighted shortest path | Yes | O(V + E) |
| DFS | Graph traversal | Yes | O(V + E) |
| Multi-Source BFS | Rotting Oranges | Yes | O(V + E) |
| Dijkstra | Shortest path, non-negative edges | No | O(E log V) |
| Prim's | Minimum Spanning Tree | N/A (no negatives needed) | O(E log V) |
| Bellman-Ford | Shortest path, allows negatives | Yes | O(VE) |
| Bellman-Ford (+1 pass) | Negative cycle detection | Yes | O(VE) |

---

## Important Viva Questions

1. Why does Dijkstra fail for negative edges?
2. What is relaxation?
3. Why does Bellman-Ford run V−1 times?
4. What is a negative cycle?
5. How is a negative cycle detected?
6. Difference between Prim's and Dijkstra's?
7. What is Multi-Source BFS?
8. Why use a Priority Queue in Dijkstra?
9. Time complexity of Dijkstra?
10. Difference between MST and Shortest Path Tree?

---

## Conclusions

In this session we explored:

1. **Multi-Source BFS** for local propagation problems (rotting oranges)
2. **Dijkstra's algorithm** for shortest paths with non-negative weights
3. **Prim's algorithm** for minimum spanning trees
4. **Bellman-Ford algorithm** for shortest paths allowing negative weights
5. **Negative cycle detection** using an extra Bellman-Ford pass

Understanding the conditions where each algorithm shines is crucial:

- **Dijkstra** — optimal for non-negative weighted graphs (O(E log V))
- **Bellman-Ford** — necessary when negative weights exist (O(VE)), also detects negative cycles
- **Prim's** — connects all vertices at minimum cost (O(E log V))
- **Multi-Source BFS** — simulates spread in grids/matrices (O(V + E))

These algorithms form the bedrock of practical applications in network routing, pathfinding, infrastructure design, and distributed systems.

---

## Glossary

| Term | Definition |
|------|------------|
| Matrix | A grid-like data structure consisting of rows and columns. |
| Zero in Matrix | Represents an empty cell in the given matrix. |
| One in Matrix | Represents a fresh orange in the problem matrix. |
| Two in Matrix | Represents a rotten orange in the problem matrix. |
| Boundary Sharing | Condition where two cells in a matrix share a side. |
| BFS (Breadth-First Search) | An algorithm for traversing or searching tree or graph data structures. |
| Multi-source BFS | BFS variation starting from multiple nodes simultaneously. |
| Shortest Distance | The minimal distance between two nodes in a graph. |
| Dijkstra's Algorithm | A graph search algorithm that finds the shortest path between nodes with non-negative edge weights. |
| Priority Queue | A data structure where each element has a priority and the highest priority element is served before others. |
| Bellman-Ford Algorithm | An algorithm that computes shortest paths from a single source vertex to all of the other vertices in a weighted graph even with negative weights. |
| Negative Weight Cycle | A cycle in a graph where the sum of the edge weights is negative, indicating no feasible shortest path exists. |


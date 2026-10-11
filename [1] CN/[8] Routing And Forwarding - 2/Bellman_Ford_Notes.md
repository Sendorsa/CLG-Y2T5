# Bellman-Ford Algorithm — Class Notes

## Overview

- **Objective:** From a given source node, find the **minimum distance** (shortest path) to all other vertices in a graph.
- **Returns:** An array containing the minimum distance from the source node to every other vertex.
- **Core Idea:** List all edges, check current distances, and **relax** (update) them if a shorter path is found.

---

## Algorithm Steps

### 1. Initialize Distances

| Vertex | Distance from Source (0) |
|--------|--------------------------|
| 0      | 0                        |
| 1      | ∞ (infinity)             |
| 2      | ∞ (infinity)             |
| 3      | ∞ (infinity)             |
| 4      | ∞ (infinity)             |

- Source node distance = **0**
- All other nodes initially = **∞**

### 2. List All Edges

| Edge | Weight |
|------|--------|
| 0 → 1 | 5    |
| 1 → 2 | 1    |
| 1 → 3 | 2    |
| 2 → 4 | 1    |
| 4 → 3 | -1   |

### 3. Relax Each Edge

For each edge **(u, v)** with weight **w**:

```
if distance[u] + w < distance[v]:
    distance[v] = distance[u] + w
```

#### Iteration Example (First Edge: 0 → 1)

- `distance[0] = 0` (source node)
- Edge weight = 5
- New distance to node 1: `0 + 5 = 5`
- Since `5 < ∞`, **update** `distance[1] = 5`

#### Subsequent Edges (per-edge relaxation within one pass)

| Edge | Calculation | Resulting Distance Array |
|------|-------------|--------------------------|
| 0 → 1 | `0 + 5 = 5` | [0, 5, ∞, ∞, ∞] |
| 1 → 2 | `5 + 1 = 6` | [0, 5, 6, ∞, ∞] |
| 1 → 3 | `5 + 2 = 7` | [0, 5, 6, 7, ∞] |
| 2 → 4 | `6 + 1 = 7` | [0, 5, 6, 7, 7] |
| 4 → 3 | `7 + (-1) = 6` | [0, 5, 6, 6, 7] ✓ improved! |

---

## Key Concepts

### Relaxation

- The fundamental operation: **checking if a shorter path exists** and updating distances accordingly.
- Repeated for **|V| - 1 iterations** (where |V| = number of vertices).

### Negative Weight Edges

- Bellman-Ford handles graphs with **negative weight edges** (e.g., 4 → 3 with weight -1).
- Dijkstra's algorithm cannot handle negative weights; Bellman-Ford can.

### Cycle Detection

- After |V| - 1 iterations, perform one more pass.
- If any distance **changes**, a **negative weight cycle** exists → shortest path is undefined.

---

## Complexity

| Metric | Value |
|--------|-------|
| **Time** | O(|V| × |E|) — vertices × edges |
| **Space** | O(|V|) — distance array |

---

## When to Use Bellman-Ford

- Graphs with **negative weight edges**
- Need to **detect negative cycles**
- Distributed / decentralized environments (each node updates independently)

## When Not to Use

- Graphs with only **positive weights** → use **Dijkstra's algorithm** (faster: O((V+E) log V))
- Large sparse graphs where performance matters

---

## Sample Walkthrough Summary

```
Graph: 5 vertices (0–4), 5 edges

Source Node: 0

Initial distances: [0, ∞, ∞, ∞, ∞]

After relaxing all edges once: [0, 5, 6, 6, 7]
                                    (improved via negative edge 4→3)

Final minimum distances from node 0:
  Node 0: 0
  Node 1: 5
  Node 2: 6
  Node 3: 6   ← improved by negative weight edge
  Node 4: 7
```

---

## Network Routing Concepts

### Distance Vector Routing (RIP)

**Core Idea:**
- Nodes (e.g., A, B, C, D) maintain **routing tables** that track the destination node and the shortest known distance to it.
- The **Routing Information Protocol (RIP)** is an implementation of this distance vector approach.
- Routers inform neighbors of distance vectors (paths measured in 'hops' or units of distance to each destination).
- Each router calculates paths independently but leverages information about directly connected routers to build paths to distant routers.

**Graph Connectivity in Highly Connected Networks:**
- In a highly connected graph, the number of edges relates to the vertices as **E ≈ O(V²)**.
- The maximum possible number of edges **(undirected graphs)** is derived from: **(N × (N - 1)) / 2 ≈ N²**.

**Limitations:**
- Bellman-Ford is not used in large networks due to inefficiencies and potential for infinite loops.
- Negative cycles can cause issues.

### MST Properties & Edge Selection

**MST Requirements:**
- A valid MST must remain entirely **non-cyclic**.
- An MST must be **fully connected**, reaching all nodes.
- A valid MST will always contain exactly **(V - 1) edges**.
- The resulting structure of an MST is a **tree**.

**Edge Selection Process (Kruskal's):**
- Edges are added sequentially by prioritizing the lowest available weights (e.g., selecting edges with weights 1, 2, and 3).
- Edges that would complete a closed loop or cycle (e.g., a crossing edge with a weight of 5) are explicitly rejected to maintain the tree structure.

---

## Minimum Spanning Trees (MST)

Finding the minimum cost way to connect all nodes in a network.

### Prim's Algorithm

**Goal:** Connect all vertices with minimum total edge weight without forming cycles.

**Process:**
1. Start from any vertex and mark it as visited
2. Continuously add the lowest-weight edge connecting a visited vertex to an unvisited vertex
3. Use a priority queue to efficiently select the minimum weight edge
4. Repeat until all vertices are included

### Kruskal's Algorithm

**Objective:** Construct an MST by sorting all edges and sequentially adding the smallest edge, provided it does not form a cycle.

**Requirements:** Use of Disjoint Set Union (DSU) to manage components and detect cycles effectively.

### Disjoint Set Union (DSU)

**Purpose:** DSU groups nodes into distinct connected components and efficiently manages sets for cycle detection in Kruskal's Algorithm.

**Key Properties:**
- Sets are represented as **hierarchical trees** with a designated root node.
- Each set contains all nodes that have been unified together.

**Example:**
- Nodes 1, 2, and 3 form one set with node 1 acting as the parent (root).
- Nodes 4, 5, and 6 form another distinct set with node 4 as the parent (root).

### Example Walkthrough (Prim's Algorithm)
1. Begin at node 0
2. Select edges based on smallest weights
3. Add edges incrementally while ensuring no cycles are formed
4. Continue until all nodes are connected

---

## Real-World Analogies & Applications

**Network Routers:**
- Use shortest-path algorithms to calculate optimal routes for data packets
- Reduces latency across the network
- Balances computational cost with reliable data transmission

**Network Topology Management:**
- Spanning tree algorithms manage network topology efficiently
- Ensures every router can communicate while avoiding circular paths

---

## Summary & Key Takeaways

This class covered:
1. **Bellman-Ford Algorithm** — finds shortest paths from a single source, handles negative weights, detects negative cycles
2. **Network Routing Applications** — distance vector protocols, hop limits for convergence
3. **Minimum Spanning Trees** — Prim's and Kruskal's algorithms for network optimization
4. **Practical Considerations** — understanding algorithm limitations, balancing computational cost with reliability in real-world networks

Recognizing these limitations helps in:
- Optimizing real-world network operations
- Ensuring reliable data transmission
- Balancing performance with accuracy in routing decisions

---

## Glossary

| Term | Definition |
|------|-----------|
| **Bellman-Ford Algorithm** | Algorithm to find shortest paths from a single source node to all other nodes in a graph with negative weights. |
| **Relaxation** | Process of updating the shortest distance to a vertex in Bellman-Ford if a shorter path is found. |
| **V-1 Iterations** | Bellman-Ford runs V-1 times to ensure shortest path detection, where V is the number of vertices. |
| **Negative Weight Cycle** | A cycle in a graph where the sum of edge weights is negative, making it impossible to find a shortest path. |
| **Distance Vector Routing** | Routing method based on computing minimum distance using algorithms like Bellman-Ford. |
| **Routing Table** | Data table stored in a router used to determine the paths for data packets. |
| **Minimum Spanning Tree (MST)** | A subset of edges connecting all vertices in a graph with minimum total edge weight and no cycles. |
| **Prim's Algorithm** | An algorithm to find the MST by expanding vertices and choosing the lowest weight edge. |
| **Priority Queue** | Data structure used in Prim's algorithm to determine the next edge with minimum weight. |
| **Kruskal's Algorithm** | An algorithm to find the MST by sorting all edges and adding them one by one, avoiding cycles. |
| **Disjoint Set Union (DSU)** | Data structure used in Kruskal's algorithm to detect cycles efficiently by union-find operations. |
| **Cycle Detection** | The process of identifying cycles using DSU to ensure a tree structure in Kruskal's algorithm. |



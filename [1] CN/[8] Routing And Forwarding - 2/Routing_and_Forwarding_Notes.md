# Routing & Forwarding - Class Notes

## 1. Bellman-Ford Algorithm

### Problem Statement
Find the minimum distance (shortest path) from a source node to all other vertices in a weighted graph. Works even with **negative edge weights**.

### How It Works — Step by Step

1. **Initialize distances**: Set `dist[source] = 0`, all others = ∞
2. **List all edges** as (source, destination, weight) tuples
3. **Relax each edge** repeatedly: For every edge `u → v` with weight `w`:
   ```
   if dist[u] + w < dist[v]:
       dist[v] = dist[u] + w
   ```
4. **Repeat `V - 1` times** (where V = number of vertices)

### Example Walkthrough

Graph edges: `0→1(5), 1→2(1), 1→3(2), 2→4(1), 4→3(-1)`
Source = node 0

| Step | dist[0] | dist[1] | dist[2] | dist[3] | dist[4] |
|------|---------|---------|---------|---------|---------|
| Init | 0       | ∞       | ∞       | ∞       | ∞       |
| Edge 0→1 (w=5) | 0 | **5** | ∞ | ∞ | ∞ |
| Edge 1→2 (w=1) | 0 | 5 | **6** | ∞ | ∞ |
| Edge 1→3 (w=2) | 0 | 5 | 6 | **7** | ∞ |
| Edge 2→4 (w=1) | 0 | 5 | 6 | 7 | **7** |
| Edge 4→3 (w=-1) | 0 | 5 | 6 | **6** | 7 |

After Round 1: `dist = [0, 5, 6, 6, 7]`

**Why V-1 iterations?** In the worst case, the shortest path can have at most `V - 1` edges. Each iteration propagates distance information one hop further.

**Short-circuit optimization:** If a full pass produces no updates, we can stop early.

### Pseudocode

```
function bellmanFord(graph, source):
    V = number of vertices
    dist = array of size V initialized to ∞
    dist[source] = 0
    
    for i from 1 to V - 1:
        updated = false
        for each edge (u, v, w) in graph.edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                updated = true
        if not updated:
            break  // No more improvements possible
    return dist
```

### Time Complexity
- Outer loop runs `V` times, inner loop runs `E` times → **O(V × E)**
- Maximum edges in a graph with N nodes: `N(N-1)/2` (quadratic) **(for undirected graphs)**
- So worst-case complexity for dense graphs is **O(V × E) = O(N³)** — cubic time

---

## 2. Application in Computer Networks

### Why Shortest Path Matters
- Routers need to forward packets efficiently
- Shorter paths → lower latency → better user experience and application performance
- Cost of a link depends on its **bandwidth** (higher bandwidth = lower cost)

### Distance Vector Routing Protocol

Each router:
1. Maintains a **routing table** for directly connected neighbors
2. Shares its routing table with adjacent routers
3. Runs Bellman-Ford using received neighbor tables to update distances
4. Periodically recalculates as link costs change

### Routing Table Structure

| Destination | Distance (Cost) | Number of Hops | Next Hop Router |
|-------------|----------------|---------------|----------------|
| A           | 3              | 2             | R1              |

### How Information Propagates

1. Each router knows only its **directly connected** neighbors initially
2. Routers exchange routing tables with direct neighbors
3. Neighbors forward this info → information spreads across the network
4. After `V - 1` exchanges, all routers know shortest paths to all nodes

### Dynamic Cost Updates
- Link costs change based on bandwidth (not static)
- Routers periodically measure and update costs
- When a cost changes → routing tables are updated → new info propagated to neighbors

---

## 3. Limitations of Bellman-Ford in Networks

### Why Not Used in Large Networks?
- **O(N³) complexity** is too expensive for thousand-router ISP networks
- **High data transfer overhead** — constantly exchanging full routing tables
- Used only in **small / edge networks** (LANs, access layers)
- For large networks, **Dijkstra's algorithm** is preferred

---

## 4. Count-to-Infinity Problem

### What Happens?
When a direct link breaks between two routers, they may incorrectly believe there's an alternate path through each other, causing distances to keep increasing.

### Example (A ↔ B ↔ C, link A-B breaks)

1. Link A→B breaks: Both know distance becomes ∞
2. B receives A's table → sees A can reach C at cost 3
3. B calculates: `dist(B,C) = dist(B,A) + dist(A,C) = 1 + 3 = 4` (was ∞ — updates!)
4. A receives B's updated table → recalculates: `dist(A,C) = 1 + 4 = 5` (was 3 — updates!)
5. Loop continues: 6, 7, 8... eventually → **∞**

This is called the **count-to-infinity problem**.

### Solutions

1. **Hop Count Limit** (e.g., RIP protocol)
   - Set max hops (e.g., 15)
   - Beyond that threshold → router deemed unreachable → triggers recalculation

2. **Use Dijkstra's Algorithm Instead**
   - Dijkstra doesn't continuously update values iteratively like Bellman-Ford
   - Takes entire topology, computes once — no cascading updates
   - Only reruns when weights genuinely change

### Analogy
Like choosing sorting algorithms: QuickSort/MergeSort (O(n log n)) are efficient for large data, but Bubble/Insertion sort work fine for small datasets. Similarly, Bellman-Ford works well in small networks despite its limitations.

---

## 5. Routing Loops in LANs

### The Problem with Dumb Switches
- In a looped topology (A → B → C → A), broadcast messages circulate infinitely
- Same packet seen from different ports → re-broadcast → creates a cycle
- Can bring the entire network down (demonstrated by connecting two ports of a hub)

### Why Redundant Connections?
- **Reliability** and **durability** — networks should survive link failures
- Multiple paths provide fault tolerance

### Solution: Logically Break the Loop
- Use spanning tree protocols to **logically block** certain ports
- Keep backup links dormant until needed
- Activates only when primary path fails

---

## 6. Minimum Spanning Tree (MST)

### Problem Definition
Connect **all vertices** with **minimum total edge weight**. 
- Every vertex must be reachable from every other vertex
- Not shortest path from a single source — minimize the **sum of all selected edges**

### Key Properties
- No cycles possible (a cycle means at least one redundant edge)
- Connected graph (all nodes reachable)
- Exactly **V - 1** edges for V vertices
- Forms a **tree** structure

---

## 7. Prim's Algorithm (for MST)

### Greedy Approach
At each step, pick the **smallest-weight edge** that connects a visited vertex to an unvisited one.

### Algorithm Steps
1. Pick any starting vertex → mark as visited
2. Add all edges from visited vertices to a priority queue (ordered by weight)
3. Pop smallest weighted edge from the queue
4. If destination is **not yet visited** → add edge to MST, mark node as visited, add its edges to queue
5. If destination **already visited** → discard (avoids cycle)
6. Repeat until all V vertices are visited

### Data Structure Required
- **Priority Queue** ordered by edge weight for efficient minimum extraction

---

## 8. Kruskal's Algorithm (for MST)

### Greedy Approach
Sort **all edges by weight**. Pick the smallest edges one at a time, as long as they don't form a cycle.

### Algorithm Steps
1. Sort all edges in ascending order of weight
2. Iterate through sorted edges
3. For each edge, add it to MST **only if it doesn't create a cycle**
4. Stop when V - 1 edges are selected

### Cycle Detection Challenge
Simply checking "visited" sets isn't enough — two visited nodes might belong to the same connected component (forming a cycle).

### Solution: Disjoint Set Union (DSU)
- Initially, each node is its own set/group
- When adding edge `(u, v)`:
  - Check if `find(u) == find(v)` → if same parent, they're already connected → skip (would form cycle)
  - If different parents → add edge → `union(u, v)` to merge their sets
- **Time complexity:** Nearly O(1) per operation with path compression and union by rank

### Data Structures Required
- **Priority Queue** (to process edges in order of weight)
- **Disjoint Set Union** (for cycle detection)

---

## 9. Prim's vs Kruskal's Comparison

| Aspect | Prim's | Kruskal's |
|--------|--------|-----------|
| Approach | Grow from a starting vertex | Process all edges globally |
| Decision basis | Edges from visited set only | All edges sorted by weight |
| Key data structures | Priority Queue | Priority Queue + Disjoint Set Union |
| Implementation complexity | Moderate | Moderate (DSU adds complexity) |
| Works well when | Dense graphs | Sparse graphs |

**Note:** Both are greedy algorithms — make locally optimal choices at each step, expecting global optimality.

---

## 10. Summary of Key Algorithms Discussed

| Algorithm | Purpose | Time Complexity | Use Case |
|-----------|---------|----------------|----------|
| Bellman-Ford | Single-source shortest path (handles negative weights) | O(V × E) | Small networks, distance vector routing |
| Dijkstra | Single-source shortest path (no negative weights) | O(E + V log V) | Large networks |
| Prim's | Minimum Spanning Tree | O(E log V) | Network design to minimize total cost |
| Kruskal's | Minimum Spanning Tree | O(E log E) | Network design with cycle detection via DSU |

---

## 11. Edge Class Data Structure

### Purpose
A custom data class for storing graph edges during MST construction (particularly Prim's Algorithm).

### Structure
Each edge object stores three attributes:
- **source**: The starting vertex of the edge
- **destination**: The ending vertex of the edge  
- **weight**: The cost/distance of the edge

### Constructor Attributes
```
class Edge:
  source     // starting node
  destination // ending node
  weight     // cost to traverse
```

This structure supports seamless graph operations: traversal, sorting by weight for MST, and cycle detection.

---

## 12. Subnetting Basics

### What is an IP Address?
An IP address consists of two parts: **network portion** + **host portion**, determined by the subnet mask.

The portion set to `255` in the subnet mask = network bits.
The portion set to `0` in the subnet mask = host bits.

### What is Subnetting?
Dividing a larger network into smaller, manageable sub-networks (subnets) to:
- Reduce network congestion and improve performance
- Organize devices logically by department/location
- Improve security through network segmentation

### Key Terminology
- **Subnet mask**: Defines which bits identify the network vs. host portion
- **Wildcard mask**: The inverse of a subnet mask (255.255.255.255 - subnet mask)
- **Network address**: First address in a subnet (all host bits = 0), cannot be assigned to devices
- **Broadcast address**: Last address in a subnet (all host bits = 1), used to send to all hosts

### Subnet Masks by Host Requirement

**Subnet masks possible (from most to least specific):**

| Subnet Mask (/) | Binary Host Bits | Calculation | Usable Hosts |
|-----------------|-----------------|-------------|--------------|
| /24 | 8 bits | 2^8 - 2 | 254 |
| /25 | 7 bits | 2^7 - 2 | 126 |
| /26 | 6 bits | 2^6 - 2 | 62 |

### CIDR Notation (Classless Inter-Domain Routing)
- A shorthand notation (e.g., `/24`, `/25`, `/26`) representing the **number of bits in the network portion** of the IP address.
- Also indicates which part of the IP is the mask, not just the network address.

### /24 Network Example: 192.168.1.0/24
- Original network: `192.168.1.0` with `255.255.255.0` mask
- Hosts available: 8 bits for hosts → **2^8 - 2 = 254** usable IPs
- Network address: `192.168.1.0`, Broadcast: `192.168.1.255`

### Creating Equal Subnets from /24
To divide into **4 equal subnets**, take 2 bits from the host portion:

| Subnet | Network Address | CIDR | First Host | Last Host | Broadcast |
|--------|-----------------|------|------------|-----------|-----------|
| 0 | 192.168.1.0 | /26 | .1 | .62 | .63 |
| 1 | 192.168.1.64 | /26 | .65 | .126 | .127 |
| 2 | 192.168.1.128 | /26 | .129 | .190 | .191 |
| 3 | 192.168.1.192 | /26 | .193 | .254 | .255 |

Each subnet now has **62 usable hosts** (2^6 - 2).

### Variable-Length Subnetting
When departments need different numbers of hosts, use variable-length subnetting to minimize IP wastage by sorting requirements from largest to smallest.

### Worked Example: Department Subnet Requirements

| Department | Hosts Needed | Bits for Network (from host) | Required Mask (CIDR) | Available Hosts per Subnet |
|------------|-------------|------------------------------|----------------------|----------------------------|
| Engineering | 64 | 1 bit (borrowed) | /25 | 126 |
| Sales | 32 | 2 bits | /26 | 62 |
| HR | 16 | 3 bits | /27 | 30 |
| Marketing | <8 | 4 bits | /28 | 14 |

### Analogy: Subnetting and Hierarchy Like Postal Codes
Subnets act like postal codes within a city — they help data packets navigate more efficiently by establishing regions. Each region (subnet) can handle local traffic internally, while only requiring routing between regions for cross-subnet communication. This hierarchy reduces congestion across the entire network.

---

## 13. Applications of Network Algorithms in Real Life

### MST in Real-World Network Design
Connecting routers/devices with minimal total cabling cost, establishing backbone networks, and minimizing infrastructure expenses while ensuring every node remains reachable.

### Prim's Algorithm for Router Updates
Starting from routers connected to the fewest other routers first allows updates to propagate most effectively across the network when scheduling firmware or configuration updates.

---

## 14. Network Router Update Problem (K-Connections Threshold)

### Problem Statement
In a network topology, identify and apply software updates **only to routers that maintain at least K active connections** (e.g., K = 2).

### Algorithm — Step by Step

1. **Calculate initial degree** of each node (number of direct connections). Example:
   - Node A → 1 connection
   - Node B → 2 connections ✓
   - Node C → 3 connections ✓

2. **Prune low-degree nodes**: Any node with fewer than K connections is removed from the network.

3. **Cascade effect**: Pruning a node reduces the degree of its neighboring nodes, which may then also fall below K and become eligible for pruning.

4. **Iterate** until no more nodes have fewer than K connections. The remaining routers receive their updates.

### Key Insight
This is a **degree-threshold filtering** problem — low-degree nodes are removed iteratively, and the cascade continues until all remaining nodes satisfy the connectivity requirement.

---

## 15. VLSM — Worked Example with Exact IP Ranges

From a base network of **192.168.10.0/24**, allocate blocks by department:

### Engineering — 60 hosts needed
- Nearest upper power of 2: **64 addresses** (2⁶)
- Block: **192.168.10.0 → 192.168.10.63**
- Subnet mask: /26

### Sales — 30 hosts needed
- Nearest upper power of 2: **32 addresses** (2⁵)
- Starts right after Engineering ends: **192.168.10.64 → 192.168.10.95**
- Subnet mask: /27

### HR — 16 hosts needed
- Nearest upper power of 2: **16 addresses** (2⁴)
- Starts right after Sales ends: **192.168.10.96 → 192.168.10.111**
- Subnet mask: /28

### Marketing — 8 hosts needed
- Nearest upper power of 2: **8 addresses** (2³)
- Starts right after HR ends: **192.168.10.112 → 192.168.10.119**
- Subnet mask: /29

### How VLSM Works
VLSM allocates IP blocks based on each department's exact host-volume requirement, matching to the nearest upper power of 2. Departments are sorted from largest to smallest allocation to avoid address fragmentation.

---

## 16. Caching and Data Invalidation

### The Problem — Data Staleness
A local cache may retain an older value (e.g., **A = 20**) while the primary database receives a new write (**W: A = 10**). Until the cache is cleared, read requests (**R: A?**) return the stale cached value instead of the updated database value.

### Cache Invalidation Strategies
- **Explicitly clear** the cache after any write operation to the primary database.
- Use **Time-To-Live (TTL)** parameters — cached entries expire automatically and are refreshed from the source when accessed again.

### Trade-offs
| Strategy | Pros | Cons |
|----------|------|------|
| Cache-aside (lazy load) | Simple, cache only on demand | Stale reads until first access after write |
| Write-through | Always consistent with DB | Latency on every write |
| TTL-based expiration | Automatic refresh, no explicit invalidation needed | May serve stale data until expiry |

---

*Note: Disjoint Set Union data structure will be covered in the advanced DSA course.*

---

## 17. Important Data Structures & Concepts

### Priority Queue
A specialized data structure that always provides access to the element with the highest (or lowest) priority — commonly used in graph algorithms like Prim's to efficiently extract the minimum-weight edge.

### Custom Comparator
A method or function that defines custom sorting rules for objects in a data structure like a priority queue, allowing users to specify how elements should be ordered (e.g., by edge weight, distance, or any custom criteria).

### Visited Set
A collection (typically a set or boolean array) used to track which nodes or vertices have already been processed during graph traversal algorithms such as MST, BFS, DFS, or Dijkstra's — preventing revisiting and infinite loops.

### Adjacency Matrix
A graph representation using an N×N matrix where the cell at row `i` and column `j` indicates whether an edge exists between node `i` and node `j`. Useful for dense graphs but requires O(N²) space.

### Adjacency List
A graph representation where each node maintains a list (or array of lists) of its connected neighbors, along with edge weights. More space-efficient than adjacency matrix for sparse graphs, requiring O(V + E) space.

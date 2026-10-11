# Routing Protocols - Class Notes

## Topic: Graph Algorithms, Router Updates & Subnetting

---

## 1. Creating a Graph Data Structure

### Graph Adjacency List Representation

Graph is stored using an **adjacency list** where each node maps to its weighted connections:

```
adj[1] = [(3,5), (7,2)]   // Node 1 → Node 3 (weight 5), Node 7 (weight 2)
adj[2] = [(1,7), (6,2)]   // Node 2 → Node 1 (weight 7), Node 6 (weight 2)
adj[3] = [(1,3)]          // Node 3 → Node 1 (weight 3)
```

### Edge Class Structure

Every **Edge** object has 3 attributes:
- **Source node**: The starting node of the edge
- **Destination node**: The ending node of the edge  
- **Weight**: The cost/distance between the two nodes

A constructor is used to create Edge objects with these three values.

### Weight Lookup by Graph Storage Format

- **Adjacency matrix**: O(1) lookup via `matrix[u][v]`
- **Adjacency list** must be preprocessed into a hashmap/matrix for O(1) weight access in O(V+E) time

---

## 2. Prim's Algorithm for MST (Dijkstra-style Framework)

### Data Structures Used

- **Edge class:** Stores source node, destination node, and edge weight as a custom object
- **PriorityQueue<Edge>:** Continuously selects the edge with minimum weight; requires a **custom comparator** to sort by weight (else PQ won't know which attribute to use)
- **HashSet<Node> visited:** Tracks nodes included in the spanning tree to prevent cyclic loops
- **ArrayList<Edge> mst:** Collects the final edges constituting the Minimum Spanning Tree

### Initialization Steps

1. **Add source node** to the `visited` set (to prevent repetition)
2. **Insert all outward-connected edges from source** into the Priority Queue as initial candidates
3. **Critical:** Must implement a **custom comparator** that sorts PQ by edge weight — otherwise the PQ won't know which attribute (source, destination, or weight) to use for sorting

### Algorithm Logic

```
visited = {source}
pq = PriorityQueue<Edge>()          // with weight-based custom comparator

// Step 1: Add all edges from source
for each neighbor v of source:
    pq.add(new Edge(source, v, weight))

// Step 2: Main loop
while pq is not empty:
    edge = pq.removeMin()           // pop minimum weight edge
    
    // Check if dest already visited (if already listed, skip — redundant)
    if edge.dest in visited:
        continue
    
    // Add to answer/MST
    add edge to MST
    
    // Mark destination as visited and expand its edges
    visited.add(edge.dest)
    
    // Only add edges from neighbors that are unvisited
    for each neighbor w of edge.dest:
        if w not in visited:
            pq.add(new Edge(edge.dest, w, weight))
```

### Key Points

- Selection is based on **minimum weight**, not node label/order
- Once a node is marked as visited, its result is final — it will never be updated
- Loop runs while `pq.size > 0`

---

## 3. Kruskal's Algorithm (Minimum Spanning Tree)

### Purpose

Finds the **Minimum Spanning Tree (MST)** for a graph using greedy approach on edges directly.

### Initialization Steps

1. Put **all edges** of the graph into a **Min-Heap** (priority queue sorted by weight)
2. Create:
   - A `Set` to track visited nodes
   - Empty MST edge list

### Algorithm Logic

For each edge from minimum to maximum weight in the Min-Heap:

1. **Check if two endpoints are in different sets:**
   - If yes (different sets / unconnected path) → add edge to MST, merge the two sets
   
2. **If endpoints are in the same set:**
   - Skip this edge (it would create a cycle)

3. Continue until all nodes are connected

### Cycle Detection Implementation Note

- For now: use a brute-force DFS-based `hasCycle` function
- Optimal approach uses **DSU (Disjoint Set Union)** — to be covered in next module/term

### Prim's vs Kruskal's Comparison

| **Prim's** | **Kruskal's** |
|------------|---------------|
| Grows one connected tree from a starting node | Considers all edges across entire graph |
| Uses a `visited` set to track added nodes | Uses DSU (Disjoint Set Union) to track edge sets |
| Good for dense graphs with many edges | Good for sparse graphs |

---

## 4. Router Software Update Problem (BFS Application)

### Problem Statement

- A network of routers needs a software update (e.g., security patches, algorithm improvements)
- Update should only install on routers with **minimum K connections**
- When an update is installed on a router, it **propagates to all connected routers**
- Constraint: **every** router receiving the update must also satisfy the minimum K connections condition
- If any connected router does NOT meet the threshold, a link can be "broken" (ignored) — but only if the target router still has ≥ K connections after removal

### Key Insight: Negative Cases Are Certain

- Cannot determine with certainty which routers WILL get updates (positive cases uncertain)
- **Can determine with 100% certainty** which routers will NOT get updates — those with degree < K
- Start from the certain negative cases and propagate elimination

### Solution: Multi-Source BFS (Level Order Traversal)

```
1. Compute degree of each router node
2. Queue all routers with degree < K  (these are guaranteed to be eliminated)
3. While queue not empty:
   - Pop a router
   - For each neighbor still in the graph:
     - Reduce neighbor's degree by 1
     - If neighbor's degree drops below K → add to queue
4. Remaining routers with degree ≥ K get the update
```

### BFS vs DFS Decision Framework

| **BFS** | **DFS** |
|---------|---------|
| Deterministic: answer found in one pass | Answer may need future value review |
| Propagate level-by-level | Explore full path before backtracking |
| No re-evaluation needed | Can revisit and update values |

---

## 5. Subnetting

### Why Subnetting?

- **Reduces congestion**: Communication within a subnet stays local, doesn't traverse external links
- Groups with high internal traffic benefit from separate subnets
- Creates network hierarchy

### IP Address + Subnet Mask Fundamentals

- **Subnet mask** identifies which bits are for network vs. host
- **Network IP** = bitwise `AND` of IP address and subnet mask
- Router uses subnet mask to determine: should this packet go to local LAN or be forwarded externally?

### Equal-Sized Subnets

Example: `192.168.1.0/24` → create 4 equal subnets

```
Original: 192.168.1.0     → subnet mask = /24 = 255.255.255.0
Host bits available = 32 - 24 = 8 bits
Total hosts possible = 2^8 - 2 = 254 (minus network address + broadcast address)

To create 4 subnets: need log₂(4) = 2 bits from host portion
New prefix = /26  (= 24 + 2)
Host bits remaining = 6 bits
Hosts per subnet = 2^6 - 2 = 62

Result:
  Subnet A: 192.168.1.0/26   (hosts .1–.62,   broadcast .63)
  Subnet B: 192.168.1.64/26  (hosts .65–.126,  broadcast .127)
  Subnet C: 192.168.1.128/26 (hosts .129–.190, broadcast .191)
  Subnet D: 192.168.1.192/26 (hosts .193–.254, broadcast .255)
```

**Important:** Total hosts across all subnets < original. Each subnet reserves its own network + broadcast addresses.

- Subnets are identified by the first **(number of borrowed bits)** of the last octet (e.g., for 4 subnets: binary 00, 01, 10, 11)

### Key Rule: Contiguous Bits Only

- Network bits must be a **contiguous sequence of 1s from the left**, followed by contiguous 0s (host bits)
- Non-contiguous bit patterns are NOT valid for subnetting
- This constraint enables fast bitwise AND calculation for network IP determination

---

## 6. Variable-Length Subnet Masking (VLSM)

### Problem

Different departments need different numbers of hosts:
- Engineering: 60 hosts
- Sales: 30 hosts
- HR: 10 hosts
- Marketing: 5 hosts
- Total IPs needed ≈ 108+, but must use powers of two

### Solution: Allocate by Largest First (Sort Descending)

**Rule:** Always start from the **largest group** and allocate consecutive power-of-two blocks. This avoids IP address fragmentation.

```
Original: 192.168.10.0/24 → 8 host bits = 256 IPs total

Step 1: Engineering (needs 60, nearest power of 2 = 64)
  → /26 subnet starting at 192.168.10.0  (hosts .0–.63 = 64 IPs)

Step 2: Sales (needs 30, nearest power of 2 = 32)
  → /27 subnet starting at 192.168.10.64 (hosts .64–.95   = 32 IPs)

Step 3: HR (needs 10, nearest power of 2 = 16)
  → /28 subnet starting at 192.168.10.96 (hosts .96–.111  = 16 IPs)

Step 4: Marketing (needs 5, nearest power of 2 = 8)
  → /29 subnet starting at 192.168.10.112(hosts .112–.119 = 8 IPs)

Remaining: 192.168.10.120/29 (8 unused IPs — available for future)
```

### Important Considerations

- **Round up** host requirement to next power of two (e.g., 58 hosts → reserve 64)
- Always include network address and broadcast address in capacity calculation
  - Want ≥31 usable hosts? Need 31+2=33 → next power of two = 64
  - Want 31 total IPs (including network+broadcast)? Need ceil(31)=31 → next power of two = 32

### Why Sorting by Size Matters

- If allocated randomly/fragmented, unused gaps between subnets may be too small for any future department
- Sorting descending ensures all remaining space is **contiguous** at the end

---

## 7. Caching and Data Invalidation

### What is Cache?

- **Replicated/duplicate data** stored closer to the processor/user for fast access
- Can reside in: registers → L1/L2/L3 cache → RAM → Redis/Memcached → disk
- Trade-off: **Consistency vs. Performance**

### The Consistency Problem

```
Database: value of "a" = 20 (updated via write)
Cache:    value of "a" = 15 (stale, not updated)

Read query arrives → returns stale "15" from cache → WRONG VALUE
```

### Solution: TTL (Time To Live)

- Every cached entry has an expiration time
- When TTL expires → treat as invalid/stale → fetch from database and refresh cache
- Most common invalidation mechanism used in practice (Redis, Memcached, DNS)

---

## 8. Router Functions (Conceptual Review)

A router must:
1. **Run a routing algorithm** — either Bellman-Ford or Dijkstra's
   - Bellman-Ford: receives routing tables from all connected neighbors
   - Dijkstra's: receives full topology info, builds entire network graph
2. **Perform longest prefix match** against its routing table to determine forwarding path
3. **Maintain and update its routing table** based on information received from connected devices
4. **Handle security patches** via software updates (the scenario in the BFS problem above)

---



## Glossary of Key Terms

| Term | Definition |
|------|-----------|
| **Edge Class** | A class to store edge information with source, destination, and weight attributes. Used in graph algorithms like Prim's and Kruskal's. |
| **Priority Queue (PQ)** | A data structure to select the edge with minimum weight in graph algorithms. Enables efficient retrieval of the smallest element. |
| **Custom Comparator** | A method to define the sorting rule for objects in a priority queue. Required for PQ to know which attribute (e.g., weight) to sort by. |
| **Visited Set** | A set to track nodes that have been visited in an algorithm like MST or BFS. Prevents cyclic loops and redundant processing. |
| **Adjacency Matrix** | A graph representation where matrix elements indicate if edges exist between node pairs. Enables O(1) weight lookup. |
| **Adjacency List** | A graph representation where each node has a list of connected nodes. Efficient for sparse graphs. |
| **Kruskal's Algorithm** | A graph algorithm used to find the minimum spanning tree by considering all edges across the entire graph, sorted by weight. |
| **Routing Table** | A data table stored in a router to determine the best path for forwarding packets. Uses longest prefix match against incoming IP addresses. |
| **Subnetting** | The process of dividing a network into smaller networks for efficiency and management. Reduces congestion and creates network hierarchy. |
| **CIDR Notation** | A compact representation of an IP address and its subnet mask, written as IP/prefix (e.g., `192.168.1.0/24`). The prefix indicates the number of network bits. |
| **Cache** | A storage layer used to temporarily store copies of data for quick access. Exists at multiple levels: registers → L1/L3 cache → RAM → Redis/Memcached → disk. |
| **Time to Live (TTL)** | The lifespan of data in cache before it is considered stale and needs refreshing from the database. Most common invalidation mechanism used in practice. |


# Graph Algorithms for Networking - Lecture Notes

## 1. Context: Where Routing Fits in Network Layers

- **OSI Model** has 7 layers; routing happens at the **Network Layer** (Layer 3)
- **Layers 1-3** combined = Application layer
  - Layer 4: Presentation layer (data formatting, encryption)
  - Layer 5: Session layer
  - Layer 6: Transport layer (TCP/UDP)
  - Layer 7: Network layer (**routing happens here**)
- Application-layer protocols determine **how** data is formatted for server recognition
- Actual data transfer across the internet requires routing through a physical network of routers

## 2. The Internet as a Graph

The internet is **not linear** -- it's a massive web of interconnected routers and networks:

- **ISP (Internet Service Provider)** manages its own network of sub-networks
  - Ranges from home Wi-Fi, hostel networks to company networks
- Signals travel via copper wires, electromagnetic radiation (Wi-Fi), optical fiber, etc.
- Once packets are ready with IP address, port, and MAC address of the **next router**, routing begins
- Router forwards packets through this network as electrical signals toward the destination

### Key Insight: Finding the Shortest Path

Two major factors in routing:
1. **Number of hops** -- fewer router connections between source and destination is generally faster
2. **Traffic/Load** -- a shorter hop path may be congested; a longer path could be faster if less busy

This is exactly why graph algorithms are essential for routing.

---

## 3. What is a Graph?

**Definition:** A graph `G = (V, E)` consists of:
- **V (Vertices/Nodes):** entities or objects
- **E (Edges):** connections between nodes

### Real-World Examples of Graphs

| Context | Nodes | Edges |
|---------|-------|-------|
| Relational Database (RDBMS) | Tables (Student-Course, Instructor-Course) | Relationships between tables (cardinality: n-to-n) |
| Worldwide Web (WWW) | Websites / URLs | Hyperlinks between pages |
| Google Maps / Roadways | Landmarks, cities, intersections | Roads connecting them |
| Railway Networks | Stations | Train routes |
| Airline Networks | Cities / Airports | Flight routes |
| Electric / Utility Networks | Power stations, substations | Transmission lines (electric networks) |
| Water & Gas Pipelines | Junctions | Pipeline connections |
| Social Media | User profiles | Follow/friend connections + engagement weights |
| Document Object Model (DOM) | HTML elements | Parent-child element relationships |
| Neural Networks (NN) | Neurons / nodes | Synaptic connections |

---

## 4. Types of Graphs

### Undirected Graph
- Edges have **no direction** -- reciprocal / bidirectional relationship
- Example: LinkedIn connections (goes both ways), like a two-way road
- Matrix representation is symmetric

### Directed Graph
- Edges point in **one specific direction** -- one-way relationship
- Examples: Instagram followers, Twitter/X (like a one-way street)
  - A → B doesn't mean B → A
- Matrix may NOT be symmetric (A to C = true, C to A = false)

### Weighted Graph
- Each edge has a **numerical weight/cost/value** attached
- Map context: physical distances (e.g., 5km, 30km, 150km) between points
- Social media context: weights represent likelihood of **user engagement** (Instagram shows posts from profiles most likely to engage)

### Unweighted Graph
- All edges have **equal cost** (binary: connected / not connected)
- Example: simple connectivity check between routers

---

## 5. Graph Representation in Memory

### Node Structure
- **BinaryTree node:** strictly `left` + `right` pointers
- **GraphNode class:** contains `data` (integer) + `ArrayList<GraphNode>` for multiple adjacent neighbors

### A. Adjacency List
- Uses a **HashMap**: key = Node, value = list of neighboring nodes
- Example: Node B → neighbor list {C, D}
- Better for sparse graphs
- Next class topic: **Adjacency Lists + DFS implementation**

### B. Adjacency Matrix
- **Square grid:** size = |V| x |V| (rows and columns both represent all nodes)
- For **unweighted graphs:** Boolean cells (`true`/`false`) indicate edge existence
- For **directed graphs:** Reflects one-way paths (A → C true, C → A false)
- For **weighted graphs:** Stores numerical weight (e.g., 20, 30, 100) instead of boolean
- Edge lookup: **O(1)** time complexity
- **Space Complexity:** O(V²) -- wasteful for sparse graphs

**Properties:**
- **Undirected graph:** Symmetric matrix (if A→B connects, B→A also connects)
- **Directed graph:** May NOT be symmetric

---

## 6. Graph Traversal & Pathfinding

### Pathfinding Problem
**Given:** A graph with a source node (S) and destination node (D).
**Question:** Does a valid path exist from S to D?

### Why This Is Harder Than Trees

In a **binary tree:**
- One node (root) is sufficient to reach all other nodes
- Structure guarantees no cycles -- only one unique path to any node
- Note: binary tree IS a directed graph -- parent to child direction matters (e.g., from D you cannot go up to C without the root as source)

In a **generic graph:**
- May have **multiple connected components**
  - Need at least one node from each component to traverse everything
  - Worst case: need all |V| nodes as input for full traversal
- May have **cycles** (infinite loops possible) -- requires tracking visited nodes

### Traversal Methods

| Method | Tree Analogy | Exploration Order |
|--------|-------------|-------------------|
| **DFS** | Pre-order / In-order / Post-order | Goes deep along each branch, backtracks at dead end |
| **BFS** | Level-order | Explores level by level |

### DFS Implementation Logic (Pathfinding)
Because graphs can contain cycles, DFS must maintain a `visited[]` array to avoid infinite loops.

```
DFS(src, dest):
1. Mark visited[src] = true     → mark source as visited immediately
2. If src == dest               → return true (found destination)
3. For each unvisited neighbor v of src:
     - Only recurse if visited[v] == false
4. Return false (no path found)
```

### Key Differences: Graph Traversal vs Tree Traversal
1. **Loop through all neighbors** instead of just 2 children
2. **Track visited nodes using `visited[]` array** -- prevent revisiting in cycles
3. **Check before visiting:** Only process if `visited[v] == false`
4. Multiple starting nodes may be needed (one per connected component)

### How DFS Works on Generic Graphs
1. Start from source node
2. Mark current node as visited
3. Process the node (check if it's destination or perform action)
4. Loop through all unvisited neighbors:
   - For each unvisited neighbor → recurse DFS
5. Backtrack when dead end reached
6. Continue with next unexplored connected component (if needed)

### BFS Preview
- Explores graph **level by level** (BFS = level-order traversal analog)
- Next class: implement BFS for path-finding using adjacency lists

---

## 7. Homework / Quiz Info

1. **Apply level-order traversal (BFS)** to solve the path-finding problem
2. Practice graph problems requiring DFS and BFS
3. Revise **subnetting** (important for upcoming quiz)

*Quiz on the 11th. Last class on the 8th -- subnetting revision + additional graph problems.*

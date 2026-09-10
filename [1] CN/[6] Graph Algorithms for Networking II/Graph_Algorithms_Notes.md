# Graph Algorithms for Networking II — Notes

## Topics Covered

---

### 1. Modelling Grid Problems as Graphs

Certain interview problems (Microsoft, Amazon) present a **binary grid** but hide a graph structure:

- **0s = water**, **1s = land**
- Cells connected if they share a boundary (4 directions: up, down, left, right)
- Task: count **islands** — contiguous groups of 1s surrounded by 0s or grid edges

> **Key Insight:** The grid itself is the graph. Each cell `(i, j)` is a node; its 4 neighbors are its adjacency list.

Example:
```
0 1 0    3 islands: one isolated 1, another isolated 1, and a group of two connected 1s below
1 1 1
0 1 0
```

---

### 2. Counting Islands — DFS on Grid

**Approach:** Run DFS from every unvisited `1` cell; each DFS call marks a full island. Increment counter per call.

**Modified DFS function:**

```python
def dfs(m, visited, i, j):
    # Base cases → return early
    if i < 0 or j < 0 or i >= len(m) or j >= len(m[0]):   # out of bounds
        return
    if m[i][j] == 0 or visited[i][j]:          # water or already visited
        return

    # Mark visited
    visited[i][j] = True

    # Recurse on all 4 neighbors
    dfs(m, visited, i - 1, j)   # up
    dfs(m, visited, i + 1, j)   # down
    dfs(m, visited, i, j - 1)   # left
    dfs(m, visited, i, j + 1)   # right
```

**Count islands:**

```python
def countIslands(m):
    rows, cols = len(m), len(m[0]) if m else 0
    visited = [[False]*cols for _ in range(rows)]
    ans = 0
    for i in range(n):
        for j in range(n):
            if m[i][j] == 1 and not visited[i][j]:
                dfs(m, visited, i, j)
                ans += 1
    return ans
```

> **Time Complexity:** O(n²) — every cell visited once.
> **Space Complexity:** O(n²) for the `visited` array.

---

### 3. Cycle Detection in Undirected Graphs (DFS)

**Problem:** Given a connected undirected graph, detect if it contains a cycle.

**Approach:** Run DFS tracking the **parent** of each node. A visit to an already-visited node that is **not** the parent signals a cycle.

```python
def hasCycle(adj, visited, src, parent=None):
    visited[src] = True
    for neighbor in adj[src]:
        if not visited[neighbor]:
            if hasCycle(adj, visited, neighbor, src):   # pass current node as parent
                return True
        elif neighbor != parent:
            return True       # back edge found → cycle
    return False
```

---

### 4. Shortest Path — Why BFS (Not DFS)?

**Problem:** Find the shortest distance (minimum edges) between two nodes in an unweighted graph.

**Why DFS fails:** DFS commits to one path fully before backtracking. The first destination found is **not guaranteed** to be via the shortest path.

**BFS works because** it explores level by level — all nodes at distance `d` are visited before any node at distance `d+1`.

---

### 5. BFS for Shortest Distance

**Concept:** Level-order traversal using a **queue**. Each queued entry stores `(node, distance)`.

```
Queue operations:
1. Enqueue source with distance 0
2. Dequeue → iterate neighbors
3. For each unvisited neighbor: mark visited, enqueue with parent's distance + 1
4. Repeat until queue is empty
```

**Pseudo-code:**

```python
def shortestDistance(adj, src, dst):
    queue = [(src, 0)]       # (node, distance)
    visited = {src}
    
    while queue:
        node, dist = queue.pop(0)
        
        if node == dst:
            return dist
        
        for neighbor in adj[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
    
    return -1   # unreachable
```

> **Why a queue is mandatory:** BFS requires strict level-by-level ordering. A stack (DFS ordering) produces wrong distances. Within the same level, node order doesn't matter.

---

### 6. Complexity Analysis of BFS

| Metric | Value | Reasoning |
|---|---|---|
| **Time** | O(V + E) | Each node enqueued at most once → outer loop V iterations; adjacency lists traversed globally → total inner work = 2E (undirected) |
| **Space** | O(V) | Queue holds at most all nodes in the widest level; visited set stores V entries |

---

### 7. Homework

1. **Determine time complexity of DFS** — explain your reasoning.
2. **Modify BFS for weighted graphs** with positive edge weights (hint: this leads to **Dijkstra's algorithm**). Negative weights are excluded.

---

### Key Takeaways

- Grid problems often hide graph structures; recognize the adjacency pattern.
- **DFS** → traversal / reachability / cycle detection / connected components.
- **BFS** → shortest path in unweighted graphs (level-by-level guarantee).
- BFS requires a **queue** for correct level ordering; DFS naturally uses recursion (implicit stack).
- Time complexity of both: **O(V + E)**.

---

### Supplemental Notes

#### 1. Number of Islands Problem (Summary)

- Count the total number of islands in an M × N grid where 0 = water, 1 = land.
- Iterate through every cell; when an unvisited `1` is found, increment the island counter and trigger DFS to mark all connected land cells visited.
- DFS base cases: indices out of bounds (i < 0 or j < 0), cell is water (`m[i][j] == 0`), or already visited.

#### 2. Cycle Detection (Summary)

- Core graph problem: determine if a cycle exists in the given network.
- Specifically targets graphs that are both **undirected** and **connected**.
- Uses DFS with parent tracking — visiting an already-visited neighbor that is not the parent indicates a cycle.

#### 3. Shortest Path Calculation (Summary)

- Find shortest distance between source node (S) and destination node (D) using BFS.
- BFS maps adjacency lists (e.g., node 1 → nodes 2, 4) to navigate the graph iteratively.
- Relies on a **Queue** data structure to evaluate nodes level by level.
- Starting node is marked visited and added to the queue; as the queue empties, unvisited neighbors are marked and enqueued.

#### 4. Complexity and Weighted Graphs (Summary)

- **BFS Time Complexity:** O(V + E), where V = vertices, E = edges.
- Graphs may have numerical edge weights (w > 0), e.g., distance of 10 between node 1 and node 2 — requires different handling than unweighted pathfinding (leads to Dijkstra's algorithm).

#### Missing Topics Worth Adding

- **Dijkstra's Algorithm** — shortest path for weighted graphs with positive edges.
- **Flood Fill Algorithm** — variation of DFS/BFS for connected component labeling.
- **Connected Components** — counting independent subgraphs in disconnected graphs (requires iterating over all unvisited nodes, not just the source).
- **Bidirectional BFS** — optimized shortest path using forward and reverse searches meeting in the middle.
- **Cycle detection in directed graphs** — uses DFS with recursion stack tracking (three states: unvisited, visiting, visited).

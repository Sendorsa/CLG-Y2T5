# Networking & Internet Infrastructure — Detailed Notes

*Based on: DNS_and_Internet_Applications_-_Class_Scaler_Academy.txt*

---

## Table of Contents

1. [Introduction to Routing](#1-introduction-to-routing)
2. [Distance Vector Routing (RIP)](#2-distance-vector-routing-rip)
3. [Link State Routing (OSPF)](#3-link-state-routing-ospf)
4. [Scaling Link State — Autonomous Systems & Areas](#4-scaling-link-state--autonomous-systems--areas)
5. [Border Gateway Protocol (BGP)](#5-border-gateway-protocol-bgp)
6. [Domain Name System (DNS)](#6-domain-name-system-dns)

---

## 1. Introduction to Routing

### Routing Algorithms vs Routing Protocols

- **Routing algorithms** determine the shortest path in a graph (e.g., Dijkstra's, Bellman-Ford).
- **Routing protocols** are how routers actually **use** these algorithms in practice across the network.
- TCP and UDP are *transport* layer protocols — routing deals exclusively with the **network layer** (routers, not end hosts).

### Two Families of Routing Protocols

| Family | Example | Core Idea |
|--------|---------|-----------|
| Distance Vector | RIP | Each router shares its distance vector (routing table) with neighbors |
| Link State | OSPF | Each router floods link-state advertisements to all routers in the network |

---

## 2. Distance Vector Routing (RIP)

### How It Works

- Every router maintains a **distance vector** — essentially its own routing table containing:
  - All reachable destinations
  - Next-hop for each destination
  - Number of hops to reach each destination
- Routers periodically broadcast their distance vectors to **directly connected neighbors only**.
- Each router updates its own table using the **Bellman-Ford algorithm** based on received information.

### Bellman-Ford Algorithm

- Converges in **V − 1 iterations** (where V = number of routers/nodes).
- Each iteration refines the shortest path estimates.
- Used to compute shortest paths from a source to all destinations.

### Key Characteristics

- Works well for **small networks (< 15 hops)**.
- RIP uses **hop count** as its metric (each router adds 1 hop).
- Routes exceeding **15 hops** are treated as unreachable (RIP uses **16 = infinity**) — any count above 15 is considered "unreachable" in the protocol.
- Simple, but has limitations at scale.

### Problems with Distance Vector

1. **Routing loops** — routers may create circular dependencies.
2. **Count-to-infinity** — incorrect metrics propagate and gradually increase to infinity.
3. **Slow convergence** — updates propagate one-hop-at-a-time.
4. **"Information of neighbors only"** — each router only knows what its neighbors tell it, creating indirect knowledge chains (A → B → C → D).

---

## 3. Link State Routing (OSPF)

### How It Works

1. **Hello Protocol**: Each router sends hello packets to discover and validate directly connected neighbors.
   - Checks if neighbors use the *same* protocol.
   - If yes, adjacency is established; if no, an error is raised.
   - Bandwidth is measured during this handshake → used to assign a **weight** to each link.

2. **Link State Advertisement (LSA)**: Each router floods the network with its own direct connections and their weights.
    - Contains info about directly connected neighbors and edge weights.
    - Flooded to **all routers within the same area** on link state changes (e.g., bandwidth/congestion updates) or periodically every 30 minutes.
    - Example: Node A broadcasts its links to nodes C and B.

3. **Link State Database (LSD)**: Every router collects all LSAs to build a complete topology map — essentially the **full adjacency list of the entire network graph**.
    - This LSD is maintained identically among **all routers within an area**.
    - If a router falls out of sync, it is brought back in sync during the next flooding.

4. **Dijkstra's Algorithm (Shortest Path First)**: Each router independently runs Dijkstra on its LSD to build its shortest-path tree.
   - Every router now has the complete graph topology and can compute the optimal path to any destination.

### Key Characteristics

- No hop-count limit — suitable for larger networks.
- **Faster convergence** — changes propagate immediately via flooding, not one-hop-at-a-time.
- New routers connecting quickly receive the full database from neighbors (not incremental).
- Weights are based on **bandwidth**, not hop count. OSPF LSA refreshes periodically every 30 minutes by default (vendor implementation), but link weights can be updated dynamically when conditions change.

### Advantages over Distance Vector

- Eliminates indirection: every router knows the *entire network topology*, not just neighbor information.
- Faster, more consistent convergence.
- No concept of "max hops" — scales better within an area.

### Still Has Problems at Internet Scale

- Imagine all routers in an ISP (10,000–20,000+ routers) maintaining the entire internet topology.
- The overhead of storing and updating this database would be enormous and impractical.
- New router startup time would be very long with massive flooding.
- **Solution: Divide the network into areas.**

---

## 4. Scaling Link State — Autonomous Systems & Areas

### Area Division

- An ISP's network is divided into smaller **areas** (typically 50–64 routers per area max).
- Within each area, all routers know all edges and run OSPF/Dijkstra independently.
- Topology information **stays within the area** — it is not broadcast across areas.

### Inter-Area Routing

- A **border router** connects different areas (similar to a gateway/switch in subnetting, but all nodes are routers).
- The border router acts as a representative of its area for external routing.
- Packets exiting an area go through the border router.

---

## 5. Border Gateway Protocol (BGP)

### Why BGP Is Needed

Routers within one ISP may use RIP; across ISPs, different networks use OSPF with different areas and policies. But **how does data cross from one ISP to another?**

- Each ISP is an **Autonomous System (AS)**: a network under a single administrative domain with its own routing policies.
- Different ASes don't know about each other's internal topology or protocols.
- We need a **common protocol** to connect these disparate autonomous systems — just as HTTP connects web servers and browsers.

While individual, localized networks internally utilize protocols like RIP or OSPF, BGP serves as the **overarching protocol** that connects these separate network clusters (ASNs) together.

### What Is BGP?

- **Border Gateway Protocol** — the protocol used to route traffic *between* autonomous systems (inter-ISP routing).
- It is **aware only of borders**, not internal topology.
- Traffic flow:
  1. **Internal network** → gateway router (via RIP or OSPF)
  2. **Border router** of ISP A → border router of ISP B (via BGP)
  3. **Across multiple ISPs** → BGP continues hop-by-hop

### How BGP Chooses Routes

BGP uses a **path vector algorithm** with policy-based route selection, because:

1. **Policy restrictions**: An AS may refuse to route certain traffic through another, even if it's the shortest path.
    - Example: `ASN 1 → ASN 2 → ASN 3` routing between distinct autonomous systems.
2. **Geopolitical, legal, and commercial rules** influence routing decisions — not just distance/bandwidth.
3. If both paths satisfy policy, the shortest distance is chosen.

### Backbone Routers

- Very high-capacity routers that interconnect major ISPs.
- Far fewer than internal network routers but handle massive traffic volumes.

---

## 6. Domain Name System (DNS)

### The Problem: Names vs IP Addresses

Routers understand only **IP addresses**, not domain names like `amazon.com` or `google.com`.
When you type a URL, your system must first resolve it to an IP address.

---

### DNS = Domain Name System (or Server)

- Maintains the mapping: **domain name ↔ IP address**.
- Managed by **ICANN** (Internet Corporation for Assigned Names and Numbers) — a nonprofit organization.
- **Problem**: If ICANN were a single server, it would be a massive single point of failure.

---

### DNS Resolution Process

#### Step 0: Browser Cache

- Your browser maintains a cache of recent DNS lookups.
- You can view it in Chrome at `chrome://net-internals/#dns`.
- **TTL (Time To Live)** governs cache validity:
  - If TTL has expired → record is invalid → discard and query again.
  - If server behind the IP has changed/dead → you get a 404 or unreachable.

#### Step 1: DNS Resolver Cache

- If not in browser cache, query the **local DNS resolver**.
- The resolver also maintains its own cache (with TTL).
- ~90–99% of lookups are served from caches at this level.

#### Step 2: Root Server

- If not in resolver cache → query a **root server**.
- The root server does *not* know individual URLs but knows which top-level domain (TLD) servers to ask.
- Example: For `amazon.com`, the root server points to the `.com` TLD server.

#### Step 3: Top-Level Domain (TLD) Server

- Handles specific **TLD extensions**: `.com, .org, .edu, .net`, etc.
- Each TLD is managed by a different registry/organization.
- Tells you which **authoritative name server** holds the record for `amazon.com`.

#### Step 4: Authoritative Name Server

- The final source of truth for a specific domain (e.g., all Amazon.com DNS records).
- Returns the actual IP address(es) for `amazon.com`.

---

### Hierarchical DNS Summary

```
Your device
    → Browser Cache (check TTL)
        → DNS Resolver Cache (check TTL)
            → Root Server  (knows: ".com servers exist at X.X.X.X")
                → TLD Server (knows: "amazon.com -> authoritative server at A.A.A.A")
                    → Authoritative Name Server (returns: IP address of amazon.com)
```

---

### Key Takeaways

| Stage | What Happens | Where Cache Helps |
|-------|-------------|-------------------|
| Browser | Checks local cache | ~most common case |
| Resolver | Checks resolver cache | ~second most common |
| Root Server | Finds correct TLD server | May cache TLD info |
| TLD Server | Finds authoritative server | May cache domain->nameserver mapping |
| Authoritative Server | Returns IP address | Stores all records for the domain |

---

### Why DNS Is Distributed (Not a Single Point of Failure)

- DNS data is **sharded across multiple servers** at each level.
- If one authoritative nameserver goes down, only the domains it holds are affected — not the entire internet.
- Large sites like Amazon have **multiple IP addresses** for load distribution and redundancy.
- TTL values ensure cache entries eventually expire, allowing updates to propagate naturally.

---

## Quick Comparison

| Protocol | Scope | Algorithm | Metric | Scale |
|----------|-------|-----------|--------|-------|
| **RIP** (Distance Vector) | Small networks (< 15 hops) | Bellman-Ford | Hop count | Small/departmental |
| **OSPF** (Link State) | Within an area/AS | Dijkstra | Bandwidth-weighted | Medium (areas of ~50–64 routers) |
| **BGP** | Between ASes (inter-ISP) | Modified Bellman-Ford + policies | Policy + distance | Global internet |

| DNS Level | Role |
|-----------|------|
| Browser Cache | Fastest lookup; TTL-based validity |
| DNS Resolver | ISP/cache server; distributes load from users |
| Root Server (.) | Directs to correct TLD servers |
| TLD Server (e.g., .com) | Directs to authoritative nameservers |
| Authoritative Nameserver | Returns the final IP address for a domain |

---

## Summary Diagram: Internet End-to-End

```
User (amazon.com typed in browser)
    1. Check browser DNS cache → if TTL valid, use it
    2. Query DNS resolver → check resolver cache
    3. Resolver queries Root Server → gets .com TLD IP
    4. Resolver queries .com TLD Server → gets Amazon's nameserver IP
    5. Resolver queries Authoritative Nameserver → gets IP address
    6. Browser now has IP, creates HTTP request

    Now routing begins:
    7. Inside LAN: Router finds gateway (RIP or OSPF)
    8. ISP network: OSPF routes within areas of the ISP
    9. Between ISPs: BGP routes from ISP A → ISP B → ... → Amazon's ISP
    10. Packet reaches Amazon server via its internal routing

    DNS + IP addressing + multi-protocol routing = Internet works
```

---

## Glossary — Key Terms

| Term | Definition |
|------|-----------|
| **Routing Protocols** | Protocols used to determine the route of data packets from source to destination in a network. |
| **Distance Vector Routing** | A routing protocol where each router broadcasts its distance to destination to its neighbors. → RIP |
| **Link State Routing** | A routing protocol where routers have a complete map of the network to independently compute shortest paths using Dijkstra's algorithm. → OSPF |
| **Routing Table** | A data table stored in a router or networked computer that lists the routes to particular network destinations. |
| **Hops** | Intermediate steps or nodes in a network path. |
| **Bellman-Ford Algorithm** | An algorithm used to find the shortest paths from a single source to all other nodes in a weighted graph. Used by distance-vector protocols. |
| **Dijkstra's Algorithm** | An algorithm used to compute shortest paths in a graph → used by OSPF on link-state databases. |
| **RIP** (Routing Information Protocol) | A distance-vector routing protocol used for smaller networks (< 15 hops). |
| **OSPF** (Open Shortest Path First) | A link-state routing protocol for larger networks within an area/AS. |
| **BGP** (Border Gateway Protocol) | Used to exchange routing information between autonomous systems on the internet (inter-ISP routing). |
| **Autonomous System (AS)** | A collection of IP routing prefixes under the control of one or more network operators, implementing its own routing policies and internal protocols. |
| **DNS** (Domain Name System) | Translates human-friendly domain names to IP addresses. Hierarchical: browser cache → resolver → root → TLD → authoritative nameserver. |
| **TTL** (Time To Live) | Determines the lifespan or expiration of a DNS cache record; invalidates stale entries so updates propagate naturally. |

---

## Terminology Map

```
Routing Protocols
├── Distance Vector Routing  →  RIP  →  Bellman-Ford algorithm
└── Link State Routing       →  OSPF  →  Dijkstra's algorithm

Scaling Link State
├── Area Division (within an AS)
└── Autonomous System  →  BGP connects ASes together

Naming
├── DNS (Domain Name System)  →  resolves domain names → IP addresses
│   ├── Browser Cache
│   ├── DNS Resolver Cache
│   ├── Root Server (.)
│   ├── TLD Server (.com, .org, .edu…)
│   └── Authoritative Nameserver
└── TTL (Time To Live)  →  governs cache validity across all DNS levels

Routing Infrastructure
├── Routing Table   (per router — stores destinations, next-hops, metrics)
├── Hops            (intermediate steps between source and destination)
└── Backbone Routers (high-capacity routers interconnecting major ISPs)
```

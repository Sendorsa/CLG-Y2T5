# DNS - Domain Name System | Study Notes

## 1. Why DNS is Needed

- **URLs vs IP Addresses:** Users type URLs (e.g., `amazon.com`) in browsers, but the internet only understands IP addresses
- Every server on the internet is recognized **only by its IP address**
- Your system needs to know the target server's IP before connecting
- **DNS maps human-readable domain names ↔ machine-readable IP addresses**

**Resolution Flow (Simplified):**
1. User types `seller.amazon.com` in browser
2. Browser queries a DNS Resolver for the IP address
3. Resolver queries Root Server (one of 13 globally managed) → returns .com TLD server IP (e.g., `50.43.10.3`)
4. Resolver queries TLD server → returns Authoritative Name Server IP (e.g., `203.10.100.6`)
5. Resolver queries ANS → returns final IP for seller.amazon.com (e.g., `51.100.31.7`)
6. Browser connects to that IP address

---

## 2. URL Anatomy

```
https://www.scanner.snapdeal.com/GPS.kittens.apich
────┘ ──┘ ────┘     ────────┘  ──────────────────
 │      │       │           │             │
Scheme   Subdomain   Domain        TLD          Path
(protocol)                                          (resource location)
```

**URL Components:**
| Component | Description | Example |
|-----------|------------|---------|
| **Scheme** | Application layer protocol | `https://scaler.com/file?id=1035#resume` |
| **Subdomain** | Optional prefix | `www`, `seller` |
| **Domain Name (Second-Level Domain)** | The chosen brand/website name | `scaler.com` |
| **TLD (Top-Level Domain)** | Managed by registries | `.com`, `.in`, `.ai`, country-code TLDs (.gov.in, .edu.in) |
| **Path** | Resource location on the server | `/file` |
| **Query Parameters** | Key-value data passed to server | `?id=1035` |
| **Fragment Identifier** | Page positioning anchor | `#resume` |

> **Structure:** `[scheme]://[subdomain].[domain].[tld]/[path]?query_params#fragment`

**Example URL Breakdown:**
```
https://www.scaler.com/file?id=1035#resume
 ──┘   ─┘  ─────┘ ─┘  ───── ───────
scheme  subdomain    domain   path  query     fragment
```

**Key Notes:**
- `www` is just a convention dating from early internet (web resource identifier)
- URLs without subdomains work fine (e.g., `mysite.com`) — must be registered in DNS
- Multiple nested subdomains are possible: `seller.scanner.snapdeal.com`
- Every part of the URL needs to be registered in DNS

---

## 3. DNS Hierarchy — The 4 Key Systems

DNS is **decentralized** (not a single server) to avoid:
- Single point of failure → whole internet goes down
- Operational complexity (verifying availability, managing durations, updating IPs)
- Massive scale challenges

### Layer 1: DNS Resolver
- Your system's **first point of contact** for DNS queries
- Does **not** maintain source of truth for domain→IP mappings
- Main job: **delegate the query** through the hierarchy
- Maintains a **cache** (domain names → IP addresses)
- Can also cache root server IPs (only ~1500 TLD records to store)

### Layer 2: Root Servers
- Returns the IP address(es) of the **TLD servers** (e.g., `.com`)
- Stores ~1500–2000 records mapping TLDs → their server IPs
- Only **TLD-level** info, not full domain records

**Root Server Facts:**
- **13 globally managed** named organizations maintain root server data (**ICANN**-managed through A to M)
- Each organization has **thousands of replicas** globally
- Examples: ICANN (non-profit), US Army, NASA, universities
- **10 of the 13 are based in the USA** — gives strong US grip over internet
- All replicas are equivalent (source of truth); updates may propagate at different rates

**Why Multiple Replicas?**
| Problem | Solution |
|---------|----------|
| High latency for far-away users | Geographically distributed servers |
| Massive query load | Distributed across thousands of instances |
| Single point of failure | No single server handles all traffic |

### Layer 3: TLD (Top-Level Domain) Servers
- Returns the IP address(es) of **authoritative name servers** for a domain
- Each TLD has its own set of servers

**TLD Management Examples:**

| TLD | Managed By | Type |
|-----|-----------|------|
| `.com` | Verisign | For-profit company |
| `.in` | NIXI (National Internet Exchange of India) | Non-profit, government-regulated |
| `.gov.in` / `.edu.in` | Government | Restricted access |
| `.us` | US Government | Government |
| `.ai` | Anguilla (caribbean territory ~100M/year revenue) | Small territory |

**Restrictions:** Some TLDs restrict certain names:
- Cannot buy `.gov.in`, `.edu.in` freely
- Two-char subdomains often restricted (e.g., `ab.in`)

### Layer 4: Authoritative Name Servers
- The **final source of truth** for a specific domain
- Stores actual domain → IP address mappings
- Handles ALL subdomains of a registered domain (e.g., `customers.mysite.com`, `mail.mysite.com`)

**Providers:** GoDaddy, Amazon Route 53, Cloudflare, etc.

---

## 4. Domain Registration Process — Step by Step

### Key Terminology

| Role | Description | Example |
|------|------------|---------|
| **Registrar** | Broker that sells domains to users | GoDaddy, Amazon Route 53 |
| **Registry** | Manages a TLD; source of truth for it | Verisign (.com), NIXI (.in) |

### Registration Flow

```
1. User chooses domain (e.g., myshop.com) via Registrar (GoDaddy)
2. Registrar queries Registry (Verisign): "Is available?"
3. Registry responds: Yes / No
4. If yes: User pays → Registrar reserves the domain
5. Domain is now yours for a set duration (NOT permanently)

6. Deploy your application on a server with IP X.X.X.X
7. Register DNS records via authoritative name server (e.g., AWS Route 53: ns1.aws.net, ns2.aws.net, ns3.aws.net):
   - myshop.com → X.X.X.X  (A record / AAAA record)
   + optional mail, CNAME, TXT, etc.

8. Registrar informs the Registry of the domain's Authoritative Name Server so TLD servers 
   know where to point future queries
```

### Money Flow
```
User pays Registrar (e.g., ₹500/yr for .in)
    ↓
Registrar keeps portion, sends remainder to Registry
Example: For .com → Verisign gets a portion
        For .in → NIXI receives portion from Indian govt
```

### Propagation Note
After buying & registering a domain, updates must **propagate** through all DNS servers globally (assumed ~2 days).

---

## 5. Full DNS Resolution Flow — End-to-End

Assuming user registers `myshop.com` + A record → `210.200.300.X` via Route 53, with name server info given to GoDaddy → registered in Verisign.

```
User types: myshop.com
              │
              ▼
Browser Cache (check first; if cached → done)
              │
              ▼
Resolver Cache (ISP/8.8.8.8/1.1.1.1 — if cached → done)
              │
              ▼
┌────────── Resolver queries Root Server ────────┐
│ Root returns TLD server IP (.com = Verisign)   │
└─────────────────────────────────────────────────┘
              │
              ▼
┌────────── Resolver queries TLD Server ─────────┐
│ Verisign returns Authoritative Name Server       │
│ (e.g., ns3.aws.net IP: 50.50.10.XX)            │
└─────────────────────────────────────────────────┘
              │
              ▼
┌────────── Resolver queries Authoritative NS ───┐
│ Returns final A record: myshop.com → X.X.X.X    │
└─────────────────────────────────────────────────┘
              │
              ▼
Resolver returns IP to client → Client connects!
```

**Query Counts (iterative resolution):**
- **3 DNS queries** made (Root → TLD → Authoritative NS)
- **6 network round-trips** total (each query gets a request + response)
- Without caching, this adds significant latency before any website content transfers

**Caching Levels:**
| Cache Location | What it caches | Impact |
|---------------|----------------|--------|
| Browser | Fully resolved URL→IP mappings | Eliminates first network hop entirely |
| Router | DNS responses | Prevents querying ISP for common sites |
| Resolver (ISP) | Common domain records + TLD server IPs | Reduces full resolution to partial lookups |

Cache TTL determines how long entries remain valid. After expiry, cache refreshes.

---

## 6. Anycast Protocol

**Problem:** Multiple servers share the **same public IP address** — should work since public IPs are unique.

**Solution: Anycast protocol via BGP (Border Gateway Protocol)**
- Servers in different locations/networks register the same IP via BGP
- Routing directs requests to the **geographically closest** server
- Each request goes to the nearest replica

```
Multiple servers with same public IP: 8.8.8.8
    ┌── Server (India)
    ├── Server (US)  
    ├── Server (Mumbai)
    └── Server (Bangalore)

User in Mumbai → routed to Mumbai server (same IP, different physical host)
User in US     → routed to US server
```

**Key Points:**
- Uses BGP for ISP-to-ISP routing
- Not in IIT curriculum but essential concept
- DNS providers offer Anycast (often as paid extra)
- Root servers & public resolvers (8.8.8.8, 1.1.1.1) use this

---

## 7. DNS Record Types

| Record | Full Name | Purpose | Example |
|--------|-----------|---------|---------|
| **A** | Address | Domain → IPv4 address | `mysite.com. IN A 201.168.10.7`, `seller.amazon.com → 51.100.31.7` |
| **AAAA** | Quad-A | Domain → IPv6 address | `mysite.com. IN AAAA 2001:db8::1`, `myshop.com → 2961:c925::619c` |
| **CNAME** | Canonical Name | Alias → Target domain | `www.mysite.com → mysite.com` (both resolve to same IP) |
| **MX** | Mail Exchange | Mail server for domain | `mysite.com. IN MX mail.sst.mysite.com` |
| **TXT** | Text | Policies/verification records | SPF, DKIM email verification |
| **SOA** | Start of Authority | Zone metadata (multi-server DNS maintenance) | Serial number, refresh intervals, etc. |

**CNAME Deep Dive:**
- Browser cache knows `mysite.com` and `www.mysite.com` both point to same IP via CNAME record
- Browser automatically rewrites URL in address bar from `www.` → bare domain (or vice versa)
- This is **NOT a redirect** — happens purely at DNS resolution level

---

## 8. Resolver Servers & Recursive Resolution

### What a Resolver Does
Resolver caches records to speed up lookups. If cache has what's needed, no further queries are made (recursive resolution).

**Popular Resolvers:**

| Resolver | IP Address | Provider |
|----------|-----------|----------|
| Google Public DNS | `8.8.8.8` | Google |
| Cloudflare DNS | `1.1.1.1` | Cloudflare |
| ISP-assigned | Random private IP from DHCP | Your ISP |
| **Default Gateway** | `192.168.1.1` (common) | Local network router |

### What Resolver You're Using?
Check Wi-Fi settings → network settings → advanced → look at **DNS Server IP**:
- If `8.8.8.8` or `1.1.1.1` → you chose a public DNS
- If something like `192.168.X.X` → your **router** is the resolver (default gateway)
- Default routers act as resolvers themselves, maintaining their own cache

### Why Pick a Public Resolver?
- **Speed:** Google/Cloudflare can cache near-complete global DNS data → faster lookups
- **Uncensorship:** ISPs remove DNS records by government order
  - Example: Some websites unreachable on ISP but work on `8.8.8.8` or `1.1.1.1`
- You can change it manually to avoid geo-blocking/blocks

---

## 9. Geographic/Government Blocking

- ISPs are **regulated** by telecommunication ministries (e.g., India's Telecom Ministry)
- Governments order ISPs to make specific websites inaccessible
- ISPs comply by **removing DNS records** from their resolvers' caches
- Result: Websites blocked via ISP DNS but accessible via public DNS (`8.8.8.8`, `1.1.1.1`)

---

## 10. `dig` Command — Diagnosis Tool

### Basic Usage
```bash
dig scanner.com
```
Shows complete DNS resolution process including all servers contacted and response details.

### Understanding `dig` Output

```
scanner.com.    60      IN    A    <IP addresses>
```
| Component | Meaning |
|-----------|---------|
| `scanner.com.` | Domain queried |
| `60` | **TTL** (Time To Live) — entry expires after 60 seconds |
| `IN` | Class (Internet) |
| `A` | Record type (IPv4 address record) |
| `<IP addresses>` | Resolved IP(s) |

The **Answer section** shows resolved records. No answer section = domain doesn't exist.

### `dig` Features to Explore
- Query specific servers: `dig @server-ip scanner.com`
- Check `.com` server response times across different TLD servers
- View full resolution sequence for any domain
- Try non-existent domains — no A record/answer section returned

---

## 11. Load Balancing via DNS

For high-traffic apps (Amazon, ChatGPT), many backend servers have **different IPs** but act as one service.

**Problem:** How to register all these IPs when servers are constantly added/removed?

**Solution:** Use DNS as a load balancer (e.g., Amazon Route 53)
- Register IP addresses of all backend servers in Route 53
- Route 53 returns the **geographically nearest** server for each query
- Automatically handles adding/removing IPs as autoscale events happen

**Why not just one central load balancer?**
- Becomes a single point of failure
- Must be replicated globally → defeats the purpose of having many servers

**Architecture:**
```
User Query → DNS (Route 53) → Returns nearest Load Balancer IP
    ↓
Load Balancer (lightweight — just redirects to internal servers)
    ↓
Internal/Application Servers (handle actual API/database logic)
```

---

## 12. Key Takeaways Summary

- **DNS = Decentralized, hierarchical, recursive, cached** domain-to-IP system
- **4 layers:** Resolver → Root (~1500 TLD records) → TLD Servers (e.g., .com by Verisign) → Authoritative Name Servers (actual DNS records)
- **Domain buying:** Registrar (GoDaddy) queries Registry (Verisign) → payment → domain registration
- **Caching at every level:** Browser, Router, Resolver — critical for speed
- **Anycast:** Multiple servers, same IP, routing directs to nearest replica
- **Record types:** A/AAAA (IPs), CNAME (alias), MX (mail), TXT (policies), SOA (zone metadata)
- **Resolvers:** ISP-assigned or public (`8.8.8.8`, `1.1.1.1`) — affects speed & accessibility
- **Blocking:** ISPs remove DNS records per government orders
- **`dig`:** Essential diagnostic tool; view TTL, record types, resolution sequence

---

## 13. Glossary

| Term | Definition |
|------|-----------|
| **IP Address** | Unique identifiers for servers on the internet, essential for server communication. |
| **DNS (Domain Name System)** | Translates human-friendly URLs into IP addresses. |
| **Caching** | Technique to store data for faster retrieval, reducing latency in DNS queries. |

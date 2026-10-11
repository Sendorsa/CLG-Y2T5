# IP Addresses and Subnetting — Detailed Notes

## 1. Internet/OSI Layer Model

### The 7 Layers (Top to Bottom)
| Layer | Name | Key Protocols / Concepts |
|-------|------|------------------------|
| 7th | Application + Presentation + Session | HTTP, HTTPS, FTP/SFTP, DNS — adds application metadata & headers |
| 6th | Transport | **TCP** (reliable), **UDP** (unreliable) — adds **source & destination port numbers**, segments data |
| 5th | Network | **IP** (IPv4/IPv6) — adds **source & destination IP addresses** |
| 4th | Data Link | Ethernet, Wi-Fi — adds **MAC address** of next hop; enforces **MTU** (largest packet size that can be sent over a link) |
| 3rd | Physical | Cables, Radio signals |

### Packet Encapsulation Flow
1. **Application Layer**: Raw data (JSON, image, video) + application header (e.g., HTTP GET/POST, size)
2. **Transport Layer**: Adds source port (randomly assigned), destination port (e.g., 80 for HTTP), segments data with indexing
3. **Network Layer**: Adds source IP + destination IP
4. **Data Link Layer**: Adds MAC address of next device; enforces MTU limits → packets fragment if too large

### Decapsulation
- Reverse process at the receiving end — removes each layer's header to retrieve original data.

---

## 2. Port Numbers

- **Destination port**: Identifies the service on the remote server (e.g., 80 = HTTP, 443 = HTTPS)
- **Source port**: Randomly assigned by OS for each connection; maps the response back to the correct app/process on the local machine
- Multiple browser tabs → different source ports even for the same website
- No fixed mapping between application and port — assigned dynamically per session

---

## 3. Packet Forwarding Through Routers

### Why Subnet Mask Matters
- Helps device determine whether a destination is on the same network or requires the gateway to reach it.

### How a router handles a packet:
1. Checks if destination IP belongs to its own subnet (uses **subnet mask**)
2. If yes → forwards to the correct device via MAC address
3. If no → forwards to next-hop router using routing table
4. Updates MAC addresses at each hop: removes old source/destination MAC, adds its own
- **Example**: Router routes traffic from Source IP → Destination IP (e.g., `143.42.59.2`)

### Packet Fragmentation & MTU
- **MTU (Maximum Transmission Unit)**: Maximum packet size that can be sent over a network layer / link; if data exceeds MTU, it is fragmented across multiple packets
- If packet > MTU → router fragments it into smaller packets
- Every link may have different MTU; routers at each hop check and re-fragment if needed
- Higher MTU → faster transfer; lower MTU → more fragmentation → slower
- **Example**: a 4500B payload is broken down into smaller 1500B chunks for transmission over the network

---

## 4. Switches vs Routers

| Switch | Router |
|--------|--------|
| Connects multiple hosts within a network | Connects different networks together |
| Forwards based on MAC address | Routes based on IP address |
| No switch needed to reach internet (router suffices) | Required to connect to the internet |

### Local Network Topology
- **Private IPs** (e.g., `192.168.1.6`, `192.168.1.7`) are used within a local network
- Devices in the same private network connect through a **switch**, not directly via router
- Switch forwards frames based on MAC addresses to deliver traffic within the LAN

---

## 5. ARP — Address Resolution Protocol

- **Problem**: IP is logical; MAC is physical (burned into NIC). Ethernet/WiFi needs MAC to deliver data.
- **How it works**:
  1. Device knows destination IP but not MAC
  2. Broadcasts ARP request: *"Who has this IP?"* to all devices on the subnet
  3. Owner replies with its MAC address
  4. Both sides cache the mapping for future use
- **ARP Cache**: Stores recent IP→MAC mappings; entries expire/evict over time (FIFO, LRU etc.)
- **CLI Command**: `arp -a` displays the current ARP cache

---

## 6. NAT — Network Address Translation

### The Problem
- Private IPs (e.g., 192.168.x.x) are meaningless outside the local network
- All devices in a home/office share **one public IP** assigned to the router

### How NAT works
1. Internal host sends packet with private source IP
2. Router rewrites **source IP** → its own public IP
3. When server responds, it sees the router's public IP as source

### Problem: Multiple devices accessing the same server
- If two laptops both request `amazon.com`, both get NAT'd to the same public IP
- When response arrives, router can't tell which device it's for

### Solution — PAT / NAT Overload (Port Address Translation / NAPT)
- Router also rewrites **source port** → assigns a unique random port
- Hash table mapping: `(public IP + assigned port) ↔ (private IP + original port)`
- When response arrives on that port, router forwards to correct internal device
- **Example**: Private `192.168.1.7:2222` is translated to public `43.25.10.17:2121`

### Port Limits
- Ports are 4-digit numbers (0–65535)
- Some ports reserved; router has limited memory → dynamic assignment
- When limit reached → rate limiting may apply

---

## 7. Port Forwarding

- **Use case**: Make an internal server accessible from the internet (e.g., gaming server, web server)
- Configuration: Map a specific public port on the router to a private IP + private port
- Conflict resolution: Different devices can't share the same forwarded port — must use different ports
- Available in router's admin panel

---

## 8. Subnetting

### IPv4 Address Structure
- 32-bit number divided into **4 octets** (e.g., `192.168.1.0`)
- Each octet = 8 bits → 0 to 255 per octet

### Subnet Mask
- Determined which bits belong to the **network portion** and which to the **host portion**
- Ones (1) on the left = network bits; Zeros (0) on the right = host bits
- Valid masks always have consecutive 1s followed by consecutive 0s (e.g., `255.255.255.0`)

### Network IP Address
- Perform **bitwise AND** between IP address and subnet mask
- Always yields the same value for all devices in the same subnet
- Identifies the network itself

### Broadcast IP Address
- Keep network bits unchanged; set **all host bits to 1**
- A packet sent to this address is delivered to **every device** in the subnet

### Number of Usable Hosts
- `hosts = 2^(number of host bits) - 2`
- Subtract 2 because: all-zeros = network ID; all-ones = broadcast address (both reserved)

---

## 9. CIDR Notation (Slash Notation)

- Instead of writing full subnet mask, use `/N` where N = number of consecutive 1-bits
- Examples:
  - `255.255.255.0` → `/24`
  - `255.255.255.224` → `/27`

### Quick Calculations with CIDR
| Step | Description |
|------|-------------|
| 1 | First octets fully covered by N remain unchanged (8 bits per octet) |
| 2 | For the boundary octet: check which bits are set in the subnet mask (power of 2: 128, 64, 32...) |
| 3 | Network IP = bitwise AND of IP and mask |
| 4 | Broadcast IP = keep network part, set all host bits to 1 |
| 5 | Hosts = `2^(32-N) - 2` |

### Worked Examples from Lecture

**Example 1**: IP=`...`, Subnet=`/27` (+ `29`)
- Network: `202.0.0.0` (first 29 bits fixed, rest zeroed)
- Broadcast: Last 3 host bits → all ones
- Hosts: `2³ - 2 = 6`

**Example 2**: IP=`...15`, Subnet=`/18` → `32.190.x.y` with `/18`
- First 18 bits (16 + 2) fixed → `32.190` unchanged
- Network IP: `32.190.0.0`
- Broadcast IP: `32.190.63.255` (remaining 6 bits of 3rd octet all 1s = 63; 4th octet all 1s = 255)
- Hosts: `2¹⁴ - 2 = 16,382`

**Example 3**: IP=`...`, Subnet=`/25` → `1.136.2.139`
- First 25 bits → `1.136.2` unchanged; last octet: first bit = network (128)
- Network IP: `1.136.2.128` (bitwise AND: `139 & 128 = 128`)
- Broadcast IP: `1.136.2.255`
- Hosts: `2⁷ - 2 = 126`

**Example 4**: IP=`...75`, Subnet=`/19` → `151.223.x.y`
- First 19 bits (16 + 3) → `151.223` unchanged
- In 3rd octet: subnet masks first bit (64). Since `75` has bit for 64 set → network part = 64
- Network IP: `151.223.64.0`
- Broadcast IP: `151.223.95.255` (5 host bits in 3rd octet → `32+16+8+4+2+1=95`, but with 3 fixed at net + 0 = first available, so `64 + 31 = 95`; 14 total host bits)
- Actual: `/19` means 13 host bits → hosts = `2¹³ - 2 = 8,190`

### Additional Specific Subnetting Examples (with full IPs)

**Example A**: IP=`192.188.99.211/29`
- Subnet Mask: `255.255.255.248`
- Network Address: `192.188.99.208`
- Broadcast Address: `192.188.99.215`
- Number of Hosts: 6

**Example B**: IP=`212.136.201.139/25`
- Network Address: `212.136.201.128`
- Broadcast Address: `212.136.201.255`
- Number of Hosts: 2⁷ - 2 = 126

---

## 10. Key Protocols & Terms Summary

| Term | Full Form / Meaning | Purpose |
|------|-------------------|---------|
| **MTU** | Maximum Transmission Unit | Max packet size per link; triggers fragmentation |
| **ARP** | Address Resolution Protocol | Maps IP → MAC within same subnet (broadcast) |
| **NAT** | Network Address Translation | Maps private IP → router's public IP |
| **PAT / NAPT** | Port Address Translation | Adds port mapping for multi-device NAT |
| **CIDR** | Classless Inter-Domain Routing | Compact subnet notation (`/N`) |
| **Network ID** | — | Subnet identifier (all host bits = 0) |
| **Broadcast IP** | — | Delivers to all devices in subnet (all host bits = 1) |

---

## 11. Additional Terms

| Term | Definition |
|------|-----------|
| **MAC Address** | A media access control address, a unique identifier assigned to a network interface controller (NIC) |
| **IP Address** | A unique identifier for a device on a TCP/IP network |
| **Dynamic Port** | A port number assigned by the operating system for use temporarily, typically when a request is made |

## 12. Upcoming Topics (Next Class)

- Routing fundamentals using **graphs** and graph algorithms
- **Subnetting**: Dividing a large network into smaller subnets
- Assigning IP ranges to departments/segments
- **Classful addressing**: Class A, B, C classifications based on subnet mask
- Expected quiz questions: Network ID, broadcast IP, number of hosts

---

*Notes compiled from Class — Scalar Academy lecture on IP Addresses and Subnetting.*

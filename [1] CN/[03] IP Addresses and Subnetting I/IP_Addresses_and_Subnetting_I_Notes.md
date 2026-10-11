# IP Addresses & Subnetting I — Structured Notes

## 1. Networking Models

### OSI Model (7-Layer)
- A standardization model introduced for the first time to describe network communication.
- Layers (bottom → top): Physical, Data Link, Network, Transport, Session, Presentation, Application.
- Layer 7 = Application layer in OSI terminology.

### TCP/IP Model (5-Layer)
- The model on which the internet was built.
- **TCP** was the first **reliable** data transfer protocol created.
- **IP addressing** is used across all networking models.
- Layers: Physical, Data Link, Network, Transport, Application.
- In the TCP/IP model, layers 7–5 of OSI are merged into a single **Application layer**.

### Comparison & Usage
| Aspect | OSI Model | TCP/IP Model |
|---|---|---|
| Origin | Standardization body | Internet / DoD |
| Layers | 7 | 5 (practical) |
| Real-world use | Educational / reference | Actual internet operations |

**Key takeaway:** In practice, **TCP/IP** is what powers the internet. However, we still use **OSI layer numbers** when discussing networking concepts (e.g., Application = Layer 7).

---

## 2. Network Architecture: Edge vs. Core

### Two Broad Layers
| Category | Examples | Role |
|---|---|---|
| **Edge devices (hosts)** | Laptops, phones, servers, smartwatches | Clients that request / Servers that serve |
| **Core network** | Routers, ISPs | Interconnects networks, carries data between them |

- Edge devices connect to the internet through a local device → ISP network → larger/deeper internet.
- ISPs are interconnected with each other via a dense network of routers carrying data between them.

---

## 3. Network Devices (Hardware)

### Hub
- **Dumb device**: broadcasts incoming packets to **all** connected devices.
- Pros: useful for broadcasting use cases.
- Cons: **security risk** (data goes to all devices), **performance loss** (unused bandwidth consumed).

### Switch
- **Intelligent device**: keeps track of connected devices' **MAC addresses**.
- Only forwards packets to the specific destination device.
- Still limited to a **single local network** — does not understand IP addresses, cannot route to the internet.

### Router
- Connects a local network to the **external internet**.
- **Understands IP addresses** and uses their structure for routing.
- Often includes built-in switch functionality in home routers.
- Typical setup: end devices → switch(es) → single router → ISP → internet.

### Repeater
- Regenerates degraded electrical signals (e.g., over long-distance cables under the sea).
- Prevents signal decay between transmitting and receiving nodes.

### Modem
- **Modulator-Demodulator**: converts digital data from devices to analog signals for transmission over telephone/DSL lines and vice versa.
- Provides access to the internet by modulating and demodulating carrier signals.
- Typically sits between the ISP connection and the router.

---

## 4. IP Addresses Basics

### What is an IP Address?
- A unique identifier assigned to a device for network communication.
- **IPv4**: 32-bit number represented as 4 groups (octets) separated by dots, e.g., `192.168.3.70`.
- Range: `0–255` per octet (8 bits → 2⁸ = 256 values).

### DHCP (Dynamic Host Configuration Protocol)
- Assigns local IP addresses, subnet masks, default gateway, and DNS server info to devices automatically.
- Runs on the router; even with no internet, the router still hands out local IPs to connected devices.

### Why Do You Always See `192.168.x.x`?
- Ranges like `192.168.x.x`, `10.x.x.x`, `172.16–31.x.x` are **RFC 1918 private ranges** reserved for internal networks.
- They are not routable on the public internet, so every local network independently assigns from these same ranges via DHCP.

---

## 5. IPv4 Address Space Limitations

### The Scale Problem
- ~8–10 billion devices connected to the internet.
- Total IPv4 addresses possible: **2³² ≈ 4.3 billion**.
- By the pigeonhole principle, some devices must share the same IP address.

### Solution: IPv6
- Newer protocol with much larger address space (addresses shown with colons).
- Not yet fully deployed — not all hardware/applications support it.
- Current OSes use both IPv4 and IPv6 simultaneously.

---

## 6. Private vs. Global IP Addresses

### Local (Private) IP
- Visible via `ifconfig` / `ipconfig` or Wi-Fi settings.
- Only relevant **inside the local network**.
- Can be duplicated across different networks without conflict.
- Assigned by your router (not by ISP).

### Global (Public) IP
- Visible by Googling "my IP address" — what the outside world sees.
- Your router's WAN IP, shared by all devices on your local network.
- Changes dynamically over time (**dynamic IP**).

### NAT (Network Address Translation)
When you request Amazon.com:
1. HTTP request includes source (your local IP + local port) and destination (Amazon server IP + port 443).
2. Router **translates the source IP and port** to its own global IP + a unique port, keeping a translation table mapping each device's connection.
3. Amazon responds to the router's global IP → router uses the port-based mapping to forward traffic back to the correct internal device.

**Implication:** All devices behind one router share the same global IP → can trigger rate limiting on servers (e.g., LeetCode blocking all students from a campus).

### Static vs. Dynamic IPs
- Default: **dynamic** — router reassigns IPs based on availability; ISPs refresh pools for security.
- **Static**: fixed IP that never changes, available by paying extra to the ISP. Required for servers/hosted services.

---

## 7. Hierarchical IP Addressing & Routing

### Why Hierarchy Matters
- No global uniqueness → two devices in different networks can have the same IP (e.g., `192.168.3.70` at home and hostel).
- Only **global IPs** need to be unique; **local IPs can repeat**.

### Hierarchical Allocation (Top-Down)
```
IANA → Registries → ISPs → ISPs assign smaller sub-ranges → Routers → Devices
```
- IANA = Internet Assigned Numbers Authority: single source of truth for all IP addresses.
 - ISPs get ranges, then subdivided by geography/region/datacenters.

### Prefix Matching in Routing
- Routers don't need to match complete IP addresses; they do **longest prefix match**.
  - Example: destination `14.15.60.30` → router checks if `14.x.x.x` matches a known route → forwards accordingly → refines at each hop.

---

## 8. Subnetting Principles & Masks

### Principles
- To calculate subnets, **decimal IP addresses are converted into binary formats** using descending powers of 2 (128, 64, 32, 16, 8, 4, 2, 1).
- The **Subnet Mask (SM)** applies a **bitwise AND operation** against the IP address to isolate the network identifier from the host identifier.

### What Is It?
- Determines how many bits of an IP address belong to the **network** vs. the **host**.
- Like an IP address but also visible alongside your local IP in Wi-Fi settings/`ifconfig`.

### Reading a Subnet Mask (`255.255.255.0`)
- `255` = 8 ones (binary `11111111`) → those bits belong to **network**.
- `0` = 8 zeros → those bits belong to **host**.
- So `255.255.255.0` has first 24 bits as network and last 8 bits as host.

### Key Addresses in a Subnet
| Address Type | How to Determine | Purpose |
|---|---|---|
| **Network IP** (`/n` notation `10.255.64.0/19`) | Bitwise AND of IP & subnet mask; marks the network itself (not assignable) | Identifies the subnet; reserved address |
| **Broadcast IP** (`79.255.86.255`, i.e., `n` bits = 1s then `h` bits = 1s) | Set all host bits to `1`; sends a message to **all** devices | Broadcast traffic within the subnet |
| **Usable IPs** (`2^13 - 2 = 8,190`) | Total addresses in that subnet minus network and broadcast (i.e., 2^(host bits) − 2); these are allocated to actual devices | Assigned to endpoints |

### Calculation Formula
- Number of assignable hosts: **2^(# host bits) − 2** (minus network + broadcast addresses).

> Each router has a single public IP from the ISP and private IPs for its interface inside the local network, so it can route between them. It also assigns DNS configuration to clients connected to it.

---

## 9. Worked Calculation Examples

### Example 1: /27 subnet
- **Given**: IP = `79.255.86.75`, Subnet Mask = `255.255.255.224`
- Third octet of mask: 224 = `11100000` → /27 (3 additional network bits beyond the first 24)
- Host bits = 32 − 27 = **5** → total IPs = 2⁵ = **32**, usable hosts = 32 − 2 = **30**
- Network IP: Fourth octet ANDed — 75 = `01001011`, mask 224 = `11100000` → result: `01000000` = **64** → **`79.255.86.64`**
- Broadcast IP: Set all 5 host bits to 1 → octet 4th position: 64 + 31 = 95 → **`79.255.86.95`**

### Example 2: /30 subnet (Point-to-point)
- **Given**: IP = `223.210.30.148`, Subnet Mask = `255.255.255.252`
- Fourth octet of mask: 252 = `11111100` → /30 (remaining 2 host bits)
- Host bits = 32 − 30 = **2** → total IPs = 2² = **4**, usable hosts = 4 − 2 = **2**
- Network IP: 4th octet ANDed — 148 = `10010100`, 252 = `11111100` → result: `10010100` = **148**
- Broadcast IP: Set last 2 host bits to 1 → 148 + 3 = 151 → **`223.210.30.151`**

---

## 10. DNS Resolution (How Your Device Finds Server IPs)

When you type a URL like `amazon.com` — the browser needs the **server's IP**, not the domain name:

1. **Device-level DNS resolution** (your OS handles this):
    - Your device queries a DNS server configured by your router (via DHCP).
    - The DNS server checks its cache for a recent entry → if found, returns cached result immediately.
    - Otherwise, the DNS server recursively queries root servers → TLD servers → authoritative nameservers until the IP is resolved.
2. **Result**: Resolved IP is returned to your browser → browser initiates a TCP connection with that server IP.

> You can override this by manually setting custom DNS in Wi-Fi/preferences (e.g., `8.8.8.8`, `1.1.1.1`) for privacy or speed.

---

## 11. IP vs. MAC Addresses

| | IP Address | MAC Address |
|---|---|---|
| **Nature** | Logical (changes with location/network) | Physical (burned into the NIC, permanent) |
| **Uniqueness** | Global only; local IPs can repeat | Globally unique per device |

---

## 12. Key Takeaways

1. Multiple networking models exist; **OSI** and **TCP/IP** are the two practical ones. OSI layer numbers remain standard in terminology even when TCP/IP is used.
2. Network architecture divides into **edge hosts** (clients/servers) and **core network** (ISPs + routers).
3. Three main devices: **hub** (dumb / broadcast), **switch** (intelligent / local MAC-based), **router** (understands IP / connects to internet).
4. IPv4 has ~4.3B addresses; IPv4 and IPv6 coexist today due to IPv6 not yet being universally deployed.
5. IPs are **hierarchical**: IANA → registries → ISPs → routers → devices, enabling prefix-based routing decisions.
6. Every device has two IPs: **local** (DHCP-assigned, private) vs. **global** (NAT-exposed by the router).
7. A **subnet mask** divides an IP into network bits + host bits; used to find the network address, broadcast address, and total usable hosts in a subnet.
8. MAC addresses are permanent physical identifiers; IP addresses are logical and change with context.

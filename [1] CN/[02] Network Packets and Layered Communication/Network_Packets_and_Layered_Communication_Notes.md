# Network Packets & Layered Communication - Notes

## Internet Overview

- **Internet**: A network of networks connecting individual smaller networks (homes, offices, schools) globally.
- Two-level structure:
  1. Devices connected to each other via network devices (routers, modems).
  2. Servers located on different geographic networks, connected through the deeper internet backbone.

### Key Terms
| Term | Meaning |
|------|---------|
| **Host** | Any device connected to the internet (client or server) |
| **Client** | Sends requests for data/resources |
| **Server** | Receives requests and provides data/resources |
| **ISP** | Internet Service Provider (e.g., Act, Airtel) |
| **Router/Modem** | Network devices that route traffic between networks |

---

## Evolution of the Internet

### 1. Early Data Transfer — Physical Media (Pre-networking Era)
- Before networking, data was moved by physically transporting hard disks between locations.
- Problem: Labor intensive, slow.

### 2. Direct Connections Between Computers
- Connected computers using cables directly.
- Problem: Data centers ran at near-freezing temperatures (for cooling). Programmers had to work in extreme conditions to write/debug code on remote machines.
- Also: Full-duplex line usage blocked the entire channel for one data transfer → inefficient.

### 3. Packet Switching Era (1960s)
- **Problem**: Transferring large files was slow, unreliable, and blocked lines entirely.
- **Solution**: Break data into smaller chunks (packets), send them independently, reassemble at the receiver.
- Packets from different sources could share the network simultaneously → efficient utilization.

### 4. ARPANET & Cold War Motivation
- Early networks were isolated (university networks in USA, France, UK).
- Cold War tension: Soviet satellite launch created fear of communication vulnerability.
- Need: A **distributed, survivable** network where no single point of failure = total collapse.
- Solution: Redundant paths — more connections between routers ensure at least one path survives.

### 5. Early Networking Concepts (1960s–1970s)
- **Client-Server model** introduced as networking concept.
- **NCP** (Network Control Program) was the early networking protocol before TCP/IP.

### 6. TCP/IP Protocol (Early 1970s)
- Different hardware/software/OS couldn't communicate → needed a **common protocol**.
- US military created early network protocol but it was **unreliable** (no acknowledgment).
- **Vint Cerf** (father of the internet) created **TCP/IP**:
  - **TCP** — ensures reliable delivery and correct data.
  - **IP** — determines the route/path data takes to reach destination.
- This IP is still used today.

### 7. DNS & 32-bit Architecture (1983–1984)
- Problem: Memorizing IP addresses wasn't scalable as more machines joined.
- **DNS** introduced hierarchical naming (`something.com`, `something.edu`) → abstraction that hides complexity.
- **32-bit IP addresses** → address space: 2^{32} \approx 10^9 addresses.
- Mathematical context: 2^{10} \approx 10^3 (1024), used to estimate address space sizes.

### 8. The World Wide Web (Late 1980s–1990s)
- Tim Berners-Lee created:
  - **HTML** — format to present data as web pages/images/videos.
  - **URI** — resource locator to access specific resources.
  - **HTTP** — application-layer protocol for requests between clients and servers.
  - **Web Browser** — tool to view web pages and communicate via HTTP.
- Created the foundation of browsing research papers and content online.

### 9. Text Giants & Beyond (2000s–present)
- Google, Amazon generated massive data → led to NoSQL databases, messaging queues, cloud computing, etc.

---

## The Need for Standardization (OSI Model)

Without standardization, every manufacturer would build different hardware/protocols → no global internet.

**Analogy**: Before USB-C, every phone had a different charger. Government mandated standardization. Similarly, all network components follow the same conventions.

### Why Layering?
- Each layer handles one specific concern.
- Layers are **abstracted** — higher layers don't need to know how lower layers work.
- Lower layers don't care what type of data is being sent (image, video, text).

---

## OSI Model — 7 Layers

| Layer # | Name | Responsibility | Protocol Examples | PDU (Protocol Data Unit) |
|---------|------|----------------|----------------------------|----------|
| **7** | Application | What app sends/receives data; defines protocol for communication | HTTP, HTTPS, FTP, FTPS, SMTP, Telnet | Data / DPU |
| **6** | Presentation | Data formatting, encoding/decoding, encryption/decryption | JPEG, PNG, GIF, MOV, MP4, PDF, JSON serialization | Data |
| **5** | Session | Manages sessions (start/end time), retry mechanisms, timeouts | — | Data |
| **4** | Transport | Segments data into smaller chunks; adds **port number** & **sequence number**; ensures reliable or best-effort delivery | TCP (reliable), UDP (low latency) | Segment |
| **3** | Network | Adds **source & destination IP addresses**; routes packets across networks | IPv4 (32-bit), IPv6 (128-bit) | Packet |
| **2** | Data Link | Adds **MAC address of next device** (not end destination); handles hop-to-hop delivery | Ethernet, Wi-Fi | Frame |
| **1** | Physical | Converts bits to physical signals; transmits over cables/waves | Copper wire, optical fiber, radio waves | Bits |

> Note: OSI model is numbered 1→7 from bottom (physical) to top (application). We describe it 7→1 (top-to-bottom), matching the sending journey.

---

## Encapsulation — Sending Data (Client Side)

When you type `google.com` and press enter, data flows **down** through layers:

1. **Application Layer**: App creates the request (e.g., HTTP GET). Adds headers (request type, content-type, size).
2. **Presentation Layer**: Encrypts/encodes data as needed. Serializes objects (e.g., Java object → JSON string).
3. **Session Layer**: Manages session parameters (timeout, retry logic).
4. **Data becomes a PDU** at this point.

5. **Transport Layer**: 
   - Segments large data into smaller chunks.
   - Adds **port number** (identifies the destination service).
   - Adds **sequence number** (for correct reassembly).
   - Either TCP or UDP header added → becomes a **segment**.

6. **Network Layer**:
   - Adds **source IP** + **destination IP** addresses.
   - Each packet gets its own IPs because packets may take different routes via packet switching.
   - Becomes a **packet**.

7. **Data Link Layer**:
   - Adds **MAC address of the next device** (next hop, not the final destination).
   - MAC address is burned into the network card by the manufacturer (mostly immutable).
   - Becomes a **frame**, ready for transmission.

8. **Physical Layer**:
   - Converts frames into **electrical/light/electromagnetic signals**.
   - Signals travel through physical medium to the router.

---

## Decapsulation — Receiving Data (Server Side)

Data arrives at the destination in reverse order:

1. **Physical Layer** → converts signals back to bits/frames.
2. **Data Link Layer** → strips MAC header (next hop address is no longer needed).
3. **Network Layer** → checks IP headers; source and destination swap roles.
4. **Transport Layer** → reorders packets by sequence number; TCP verifies reliability and removes segments.
5. **Presentation Layer** → decrypts/decodes data to original format.
6. **Session Layer** → manages session cleanup.
7. **Application Layer** → delivers original data to the server app (e.g., HTTP server processes the request).

Server response follows the same process in reverse direction back to the client.

---

## IP Address vs MAC Address

| | IP Address | MAC Address |
|--|------------|-------------|
| Meaning | Internet Protocol address — logical, changes with network | Media Access Control address — physical, burned into hardware |
| Analogy | Mailing address (can change) | Aadhaar/PAN number (permanent ID) |
| Scope | Identifies which **network** the device is on; helps determine path | Identifies **next hop** device on the local network |
| Mutability | ✓ Changeable (assigned by ISP/local network) | ✗ Mostly immutable |
| Example | `192.168.1.X` | `00:1A:2B:3C:4D:5E` |

---

## Key Protocol Properties

### TCP (Transmission Control Protocol) — Layer 4 Transport
- **Reliable** — guarantees delivery and correctness.
- Adds latency because of acknowledgments, retransmissions.
- Use case: Everything that needs accuracy (web browsing, email, file transfer).

### UDP (User Datagram Protocol) — Layer 4 Transport
- **Best-effort** — no guarantee of delivery or ordering.
- Lower latency.
- Use case: Real-time apps where speed > reliability (online gaming, video calls).

### Routing Through Routers
1. Packet reaches your local router with Destination IP + MAC address → next hop device.
2. Router reads Destination IP, determines optimal path, and replaces MAC address with the **next router's MAC**.
3. Repeat at each hop until destination is reached.
4. Each router only needs the MAC of its immediate neighbor — not the full path.

---

## Layering Abstraction Principle

Each layer has a "contract" with adjacent layers:
- Sends data **downward** with specific headers/structure.
- Receives data **upward** and interprets those headers.
- Higher layers are unaware of lower-layer implementation details (technology, medium).
- Lower layers don't know what data is being sent — to them it's just bits.

### OSI vs TCP/IP Model Comparison

The 7-layer OSI model condenses into the 4-layer TCP/IP model:

| OSI (7 layers)       | TCP/IP (4 layers)              |
|---------------------|-------------------------------|
| Application Layer   |                               |
| Presentation Layer  | **Application Layer**         |
| Session Layer       | *(layers 5–7 combined)*       |
| Transport Layer     | **Transport Layer**           |
| Network Layer       | **Internet Layer**            |
| Data Link Layer     |                               |
| Physical Layer      | **Link Layer** *(layers 1–2 combined)* |

> Note: OSI layers 5–7 (Session, Presentation, Application) collapse into TCP/IP "Application".
>    OSI layers 1–2 (Physical, Data Link) collapse into TCP/IP "Link".

---

## Summary Timeline

| Era | Milestone |
|-----|-----------|
| Pre-1960s | Physical data transport (hard disks) |
| 1960s | Packet switching invented |
| Late 1960s | ARPANET — early isolated computer networks |
| Cold War | Need for fault-tolerant military network → redundant paths |
| Early 1970s | Vint Cerf creates TCP/IP protocol for reliable cross-network communication |
| Mid 1980s | DNS introduced for easier name-based addressing |
| Late 1980s–1990s | Tim Berners-Lee: HTML, URI, HTTP → World Wide Web |
| 2000s+ | Text giants (Google, Amazon) → cloud, NoSQL, messaging queues |

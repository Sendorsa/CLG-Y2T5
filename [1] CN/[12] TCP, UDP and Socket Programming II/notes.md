# TCP, UDP & Socket Programming II — Detailed Notes

## 1. DNS Resolution Recap (Previous Class)

### DNS Resolution Process
1. **Resolver Server** — First server to receive the request. Maintained by ISP or organizations (e.g., Google DNS, Cloudflare DNS). Your OS registers the resolver IP when you connect to the internet.
2. **Root Server** — Returns the IP of the appropriate Top-Level Domain (TLD) server. Does *not* return the final URL's IP.
   - Example: For abc.com → returns TLD server `50.1.3.17` for .com.
3. **TLD Server** — Handles 1500+ top-level domains (.com, .org, .in, country codes, etc.). Returns the IP of the authoritative name server.
4. **Authoritative Name Server (ANS)** — The server responsible for the specific domain. Returns the exact IP address of the website's server.
   - Example: ns1.godaddy.net contains the A record mapping `abc.com` → `201.36.17.3`.

### URL Components Breakdown
| Component | Example | Role |
|-----------|---------|------|
| Scheme | `https` | Protocol for communication |
| Subdomain | `www` | Convention; not mandatory (can be skipped, or other names used) |
| Domain name | `amazon` | The host / brand name |
| TLD | `.com` | Top-level domain (.com, .org, .in, etc.) |
| Path | `/s/laptop` | Resource location — where exactly on the server the resource lives |

### Full URL Breakdown — Worked Example
```
https://www.scaler.com/admin/candiolo?id=1035#resume
```
| Component | Value | Description |
|-----------|-------|-------------|
| Protocol | `https://` | Communication protocol |
| Subdomain | `www` | Convention; not mandatory |
| Hostname | `scaler.com` | Domain name + TLD |
| Path | `/admin/candiolo` | Resource location on the server |
| Query Parameter | `?id=1035` | Parameters sent to server |
| Fragment | `#resume` | Client-side navigation only; not sent to server |

- **URL = Resource Locator** — origin (machine) + path (exact file location).
- Protocol tells the server how to decode the request; both client and server need compatible protocol software.

---

### Cookies & Statelessness

- **REST APIs are stateless** — every API call is independent. Server has *no memory* of previous calls from that client.
- **State of an object** = current values of its attributes at a given time (e.g., student name, email, CGPA). Two API calls can't be dependent on each other.

#### How browsers work around statelessness:
| Storage Type | Persistence | Use Case | Size Limit |
|--------------|-------------|----------|------------|
| **Session storage** | Survives tab refresh; cleared when tab closes | Forms, partial progress | Larger |
| **IndexedDB** | Permanent in browser | Heavy local data (1-4 GB) | Very large |
| **Cookies** | Server-set; expires per config | State tokens (session ID) | Very small |

#### Why cookies?
- Cookies carry a **session ID** (server-generated token).
- Browser sends cookie with every request → server checks:
  1. Is the session ID valid?
  2. Has the session expired?
- Different sessions have different expiration times (YouTube / social media = long-lived; banking app = ~3 minutes).
- **Size is small** because cookies are part of every HTTP request header → only store tokens/keys, not large data.

---

## 2. TCP (Transmission Control Protocol)

### Why TCP Was Created
- **Origin**: US military needed reliable data transfer where every byte is guaranteed to reach correctly and in order.
- **Promise**: Reliable, ordered, byte-stream delivery.
- **Trade-off**: Latency — reliability comes at the cost of speed.

### 3-Way Handshake (Connection Establishment)

| Step | Direction              | Packet Contains                             | Purpose                                      |
|------|------------------------|---------------------------------------------|----------------------------------------------|
| 1    | Client → Server        | `SYN` + random sequence number (e.g., 1000) | Client initiates connection                  |
| 2    | Server → Client        | `SYN-ACK` + ACK = client seq + 1, own seq (e.g., 7000) | Server acknowledges and sends its own sequence |
| 3    | Client → Server        | `ACK` + ACK = server seq + 1 (e.g., 7001), own seq + 1 (e.g., 1001) | Client confirms; connection established      |

### Data Transfer — Sequence Numbers & Acknowledgements
- **First packet**: Uses the sequence number agreed during handshake.
- **Subsequent sequences**: = previous starting sequence + size of data in bytes.
  - Example: seq=1000, data=500 bytes → next packet starts at seq=1501.
- **Server ACK**: Always sends `next expected sequence number` (not the one received).
  - Example: Server receives packets with seq=1000 → ACK = 2001 (needs 2000+).
- **Both sides maintain buffers** for sent/received packets.

### Handling Packet Loss — Retransmission
- If a packet is lost, the server keeps sending the **same ACK** (asking for the missing sequence repeatedly).
- This is called **duplicate ACKs**.
- **Client logic**: After receiving 2–3 duplicate ACKs for the same missing sequence → assumes packet was lost → retransmits.
- If a packet was merely *delayed* (not lost), future packets may arrive out of order but are buffered until the missing one arrives.

### Key TCP Properties
| Property          | Description                                                  |
|-------------------|--------------------------------------------------------------|
| Reliability       | Guaranteed delivery via ACKs and retransmission              |
| Ordering          | Packets reassembled using sequence numbers                   |
| Connection-oriented | 3-way handshake required before data transfer               |
| Flow/Congestion Control | Prevents overwhelming sender/receiver (not covered here) |
| Full-duplex       | Both sides can send simultaneously                          |

---

## 3. UDP (User Datagram Protocol)

### Why UDP Exists
- TCP guarantees ~90% of use cases, but 10% need **lower latency** over reliability.
- In those cases, retransmitting lost data is useless because the moment has passed.

### Use Cases
| Application     | Reason UDP is preferred                              |
|-----------------|------------------------------------------------------|
| Video calls     | A dropped video frame can't be inserted later        |
| Online gaming   | Old position data is irrelevant once the game moves on |
| Audio streaming | Stitching past audio gaps sounds unnatural           |

### How UDP Works
- **No handshake** → no connection establishment overhead.
- **No sequence numbers** → no ordering guarantee.
- **No acknowledgements** → sender doesn't know if data arrived.
- Send a datagram and move on — that's it.

### Can You Build Reliable Systems Over UDP?
- **Yes** — by implementing TCP-like logic at the application layer.
- Example: **HTTP/3** uses UDP but adds its own reliability mechanisms above it.

---

## 4. Evolution of HTTP & Protocol Stack

### HTTP/1.1
- Multip **le TCP connections** needed to avoid head-of-line blocking.
- Responses returned in the **exact order** requests were sent.
- Headers are **human-readable strings** → serialization overhead.
- Opening many TCP connections is expensive (each requires a 3-way handshake).

### HTTP/2
- Uses a **single TCP connection** with **multiplexing**.
- Rendering a modern webpage requires pulling HTML from a CDN alongside hundreds of separate API calls for individual assets (images, CSS, JS, fonts, etc.).
- Network multiplexing optimizes this by allowing multiple independent data streams (e.g., Stream 1, Stream 2) to transfer concurrently over a single underlying connection.
- Data is split into **streams** (each API call/resource = separate stream).
- Multiple requests/responses travel simultaneously on the same connection.
- **Headers are binary** (not string) → faster serialization, smaller overhead.
- Inspired Google's **gRPC** protocol.

#### HTTP/2 Limitation — Head-of-Line Blocking
- TCP sits below HTTP and **does not understand streams**.
- TCP sequences packets globally (across all streams).
- If **one packet** in any stream is lost → **all streams stall**.
- Even if a full image download completed, it waits for a missing JS packet.

### HTTP/3
- **Eliminates TCP entirely** → uses UDP as transport layer.
- Uses the **QUIC protocol** built on top of UDP.
- QUIC adds its own sequence numbers + stream IDs above UDP datagrams.
- The server treats these like UDP packets, routes them, but QUIC handles:
  - Per-stream sequencing (not global)
  - Retransmission of only lost packets in the affected stream
  - Stream-level reordering (no cross-stream blocking)
- No head-of-line blocking because each stream is handled independently.

---

## 5. Sockets — The Bridge Between App & OS Network Stack

### What Is a Socket?
- Abstraction that allows applications to communicate over the network.
- An **interface provided by the operating system** to use TCP/UDP protocols.
- You cannot use raw TCP directly — you interact through sockets.

### Socket Object Stores
| Detail                  | Description                                         |
|-------------------------|-----------------------------------------------------|
| Destination IP          | Server/client IP address                            |
| Destination Port        | Target port number                                  |
| Protocol type           | TCP socket vs. UDP socket (different classes/types) |
| Connection state        | Current state of the connection                     |
| Read buffer             | Incoming packets before application reads them      |
| Write buffer            | Outgoing packets before network sends them          |

### How a Connection Is Made in Practice
1. Application (e.g., browser) creates a **socket object** (TCP or UDP).
2. Specifies destination IP and port.
3. Browser tells the OS to establish the TCP connection → OS performs 3-way handshake.
4. Both sides register each other's IP, port, and protocol in their OS memory.
5. Data flows through buffers → decapsulated at receiver → delivered to application.

### What "Establishing a Connection" Really Means
- **NOT** reserving a dedicated physical route between client and server.
- Routing is handled independently by routers based on IP addresses.
- Different packets may take different routes.
- **Connection = mutual awareness**: Both sides know each other's (IP, port, protocol) pair.

### Who Manages TCP/UDP?
- The **operating system** manages it — not the application developer directly.
- OS abstracts networking complexity (just like it abstracts hardware for keyboards/mice).
- Developers use high-level APIs; OS handles lower-layer details.

---

## 6. Key Terms Summary

| Term                     | Definition                                                                                       |
|--------------------------|--------------------------------------------------------------------------------------------------|
| Sequence Number          | 32-bit identifier for each TCP packet, enabling ordering and loss detection                      |
| Acknowledgement (ACK)    | Response confirming receipt; contains next expected sequence number                              |
| Duplicate ACK            | Same ACK sent multiple times → signals missing packet                                            |
| Multiplexing             | Sending multiple streams over one TCP connection (HTTP/2)                                        |
| Head-of-Line Blocking    | One lost packet stalls all data streams in HTTP/2                                                |
| Data Stream              | HTTP/2 concept — each request/response pair gets its own stream ID                               |
| QUIC                     | Protocol used in HTTP/3 over UDP, providing reliable, ordered delivery per-stream                |
| Socket                   | OS-level interface for applications to create TCP/UDP connections                                |

---

## 7. Comparison: TCP vs. UDP

| Feature             | TCP                                     | UDP                          |
|---------------------|-----------------------------------------|------------------------------|
| Connection          | Connection-oriented (3-way handshake)   | Connectionless               |
| Reliability         | Guaranteed delivery + retransmission    | No guarantees                |
| Ordering            | Packets arrive in order                 | No ordering                  |
| Acknowledgements    | Yes                                     | No                           |
| Sequence Numbers    | Yes                                     | No                           |
| Latency             | Higher (overhead of reliability)        | Lower                        |
| Header overhead     | Larger                                  | Smaller                      |
| Use cases           | Web browsing, email, file transfer      | Video calls, gaming, streaming |
| Used by HTTP version| HTTP/1.1, HTTP/2                        | HTTP/3 (via QUIC)            |


---

## 8. Homework / Self-Study
- On your own system, figure out how to list all currently established TCP connections using terminal commands (e.g., `netstat`, `lsof`, or `ss` on Linux/macOS).
- Explore whether you can terminate those connections from the command line.

---

## 9. URL Components — Query & Fragment (Additional)

| Component | Example | Role |
|-----------|---------|------|
| Query | `?id=123` | Parameters sent to server |
| Fragment | `#section` | Client-side navigation only; not sent to the server |

---

## 10. Domain Registration & DNS Records

### Domain Registration
- Domains are purchased through **registrars** (e.g., GoDaddy).
- Registrars interact with **TLD organizations** (e.g., VeriSign for .com domains).
- Once registered, the domain becomes unavailable for others to buy.
- The registrar assigns an **Authoritative Name Server (ANS)** (e.g., ns1.godaddy.net) and creates DNS records that link the domain to this ANS.

### DNS Records
| Record Type | Purpose |
|-------------|---------|
| **A Record** | Maps a domain name to an IPv4 address |
| **AAAA Record** | Maps a domain name to an IPv6 address |
| **NS Record** | Specifies the authoritative name server for the domain |
| **MX Record** | Directs mail to mail exchange servers |
| **TXT Record** | Stores arbitrary text data (e.g., SPF records for email validation) |

---

## 11. HTTP Requests — Components

| Component | Description |
|-----------|-------------|
| **Method** | GET, POST, PUT — action to perform |
| **Path** | Resource being requested |
| **Protocol Version** | HTTP version used (e.g., HTTP/1.1, HTTP/2, HTTP/3) |
| **Headers** | Metadata: Host, User-Agent, Cookies, Content-Type, etc. |
| **Body** | Request payload data (used in POST/PUT requests) |

## 13. Vocabulary Glossary

| Term | Definition |
|------|------------|
| DNS Resolution | Process of translating a domain name to an IP address. |
| Resolver Server | Initial server that receives DNS queries, usually maintained by ISPs. |
| Root Server | Provides IP address of top-level domain servers, first in the DNS hierarchy. |
| Top-Level Domain Server | Server with information on authoritative name servers for domains. |
| Authoritative Name Server | Provides IP address for a specific URL within a domain. |
| A Record | DNS record mapping a domain name to an IPv4 address. |
| AAAA Record | DNS record mapping a domain name to an IPv6 address. |
| NS Record | Indicates which authoritative name server holds the DNS records for a domain. |
| HTTP Request | A message sent by a client to a server to request a resource, uses methods like GET or POST. |
| Cookies | Data stored by a browser to remember user sessions and preferences. |
| Statelessness | REST API principle where each request from client to server must contain all necessary information. |
| TCP Handshake | Three-step process to establish a reliable connection: SYN, SYN-ACK, ACK. |
| UDP | User Datagram Protocol, focuses on low latency rather than reliability, no ordering guarantee. |
| HTTP/1.1 | Features persistent connections but requires request-response ordering. |
| HTTP/2 | Introduces multiplexing, allowing multiple requests over a single connection. |
| HTTP/3 | Uses UDP and employs QUIC for reduced latency and improved performance. |
| Multiplexing | Sending multiple streams of data over a single connection simultaneously. |
| QUIC | Protocol that provides reliable, low-latency connections over UDP for HTTP/3. |
| Socket | An endpoint for sending and receiving data across a network, associated with an IP address and port. |

---

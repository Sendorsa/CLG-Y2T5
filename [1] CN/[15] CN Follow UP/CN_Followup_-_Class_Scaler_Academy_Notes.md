# Computer Networking — Application Layer Protocols (Class Notes)

## 1. HTTP (Versions 1.1, 2, 3)

- **All three are request-response protocols**: Client → server connection → request → response
- **HTTP/1 & HTTP/2** use **TCP** as the transport layer protocol
- **HTTP/3** uses **UDP + QUIC** (implements its own reliability, sequence numbers, framing to avoid TCP head-of-line blocking)
- **Key insight**: HTTP/3 enables multiplexing via multiple independent streams within a single QUIC connection — data from different requests can share the same connection without blocking each other (no head-of-line blocking)

---

## 2. FTP (File Transfer Protocol)

### How FTP differs from HTTP:

| Aspect | HTTP | FTP |
|--------|------|-----|
| **Connections** | Single TCP (usually) | **Two separate TCP connections** — control + data |
| **State** | HTTP/1 supports statelessness; HTTP/2 adds multiplexing | Stateful (control connection persists across transfers) |

### FTP's Two Connections:

1. **Control Connection**: Handles all commands (explore filesystem, read, write, create/delete folders). Stays alive across multiple operations.
2. **Data Connection**: Dedicated for one file transfer at a time. Opens for a download/upload → closes after completion. Can reopen for the next file (control connection stays open).

### FTP Connection Modes:
- **Active** and **Passive** — these are types of data connections, not the two separate connections

> Both connections use TCP because reliability is needed for file transfers.

---

## 3. WebSockets

- **Persistent, full-duplex** client-server connection (not peer-to-peer!)
- Data flows both ways: client → server AND server → client **independently of requests**
- Server can push data to clients without any prior request from the client

### How it works:
1. Starts as a regular TCP connection (three-way handshake)
2. Followed by an HTTP upgrade request on the same connection
3. Server acknowledges upgrade → connection becomes bidirectional full-duplex WebSocket
4. Once upgraded, protocol is no longer request-response

### Real-time messaging flow:
1. Client sends message to server via WebSocket
2. Server processes it (saves to DB, runs analytics, etc.)
3. Server identifies recipient
4. Server pushes message to recipient's **separate** WebSocket connection
5. **Important**: The sender does NOT deliver directly to the receiver — the server always mediates

### Each client has its own independent WebSocket connection with the server. Client 1 and Client 2 are NOT directly connected.

---

## 4. WebRTC (Web Real-Time Communication)

- **NOT 100% peer-to-peer** — it uses P2P to optimize latency, but falls back to a relay server if P2P fails
- Uses **UDP** for media streaming (not TCP!) because:
  - Dropped frames are acceptable in live video/audio (out-of-order or delayed packets hurt more)
  - TCP's retransmission causes unacceptable latency for real-time media

### WebRTC Connection Setup:

1. **ICE framework collects candidates** — tries multiple ways to connect peers
2. **STUN server**: Both peers contact a STUN server to discover their public IP + port (behind NAT)
3. **Hole punching via UDP**: Peers exchange public IPs, then punch through NAT using UDP:
   - A sends UDP to B's public IP → B accepts because B initiated connection to A first
   - B responds → A accepts because A just sent to B
   - Works because TCP requires full handshake; UDP is stateless
4. **Codecs negotiation**: Peers agree on video format/quality for mutual compatibility

### Fallback — TURN (Relay) Server:
- If P2P fails (firewalls, NAT issues), media routes through a TURN relay server
- Relay servers are geographically distributed (e.g., Bangalore relay for users in Bangalore)

### Large group calls — Selective Forwarding Unit (SFU):
- For 50–100 person calls, full mesh P2P is unscalable
- Everyone streams their video to an **SFU server** instead
- SFU decides who gets what (resolution, audio/video based on each recipient's config)
- Converts mesh topology → star/tree topology

---

## 5. BitTorrent (Pure Peer-to-Peer)

- Torrent servers do **NOT store any data** — they only maintain metadata about who has what
- Files are broken into smaller segments, each with a hash computed
- **Info hash**: hash of all segment hashes stored in `.torrent` file
- Process:
  1. User downloads `.torrent` file → contains info hash + tracker server address
  2. Torrent client contacts tracker → gets list of **seeders** (peers who have the file)
  3. **Hole punching via UDP** to discover peers' public IPs/ports
  4. TCP connections established between peers for actual data transfer
  5. Download multiple segments from different peers simultaneously
  6. **Priority seeding**: Segments with fewer seeders are downloaded first (redundancy)
  7. As soon as you finish downloading a segment, your machine becomes a seeder too

---

## 6. Email Protocols

### SMTP (Simple Mail Transfer Protocol) — SENDING only
- Responsible for **sending** emails (NOT receiving/reading)
- Email address resolution uses **DNS MX records** (Mail Exchange records) to find the recipient's mail server IP
- Client sends email to **its own provider's SMTP server** (e.g., Gmail server), NOT directly to recipient

### Email Sending Flow:
1. User writes email → press send
2. Client sends email to its SMTP server (e.g., Google's) — client only waits for this acknowledgment
3. Google's SMTP server handles everything else:
   - DNS MX lookup for `outlook.com`/Yahoo/etc.
   - Caches DNS records (much more extensive than browser cache)
4. Messages go into **queue** (rate limiting, congestion, etc.)
5. Server pushes to recipient's mail server

### Email Receiving Protocols:

| Protocol | Behavior | Sync across devices? |
|----------|----------|----------------------|
| **IMAP** | Centralized storage; status synced everywhere; email stays on server | ✅ Yes |
| **POP3** (Post Office Protocol v3) | Download + delete from server ("post office" model: get letter, take it home, server deletes copy) | ❌ No — emails deleted after delivery |

### Email States/Flow:
- **Draft**: Client auto-saves to browser (IndexedDB/localStorage) → periodically syncs to server. Even unsent drafts are recovered on next login
- **Sent box**: Server acknowledged receipt. Email may NOT yet be delivered to recipient
- **Delivery failure**: SMTP generates bounce-back message if delivery fails

### Scaling email systems:
- Load balancers route users to appropriate servers
- Data sharding/partitioning (millions/billions of users)
- Attachments stored separately from email text content

---

## 7. DHCP (Dynamic Host Configuration Protocol)

DHCP assigns **IP addresses** to devices on a network. Four-step DORA process:

### **D** — Discover:
- New device has no IP → broadcasts `0.0.0.0` as source, `255.255.255.255` as destination
- UDP (source port 68, dest port 67) — **TCP can't work** because TCP needs a connection first

### **O** — Offer:
- DHCP server responds with available IP address + subnet mask + lease time (e.g., 24 hours)
- Multiple servers may offer → client picks first one

### **R** — Request:
- Client broadcasts acceptance of chosen IP from specific server
- Other servers know that IP is taken

### **A** — Acknowledge:
- Server confirms assignment with lease details

### Lease Renewal:
- At **50%** of lease time → client sends renewal request to current server (UDP unicast)
- At **87.5%** of lease → rebind: broadcasts renewal to any server
- If both fail: IP is freed at lease expiry; new IP assigned next time device connects

### Key points:
- Lease reserves the IP for that device — even if offline, no other device gets it during lease
- DHCP failure fallback: client generates self-assigned IP (169.254.x.x link-local range) with fixed prefix → possible IP conflicts

---

## 8. Scaling Issues (HLD Concepts)

### Problems when scaling WebRTC/WebSocket servers to millions of users:
- **One server is NOT enough** — need multiple servers
- **Durability**: If a server dies, connected users get disconnected
- Need **replication** (backups doing the same work) + **sharding/partitioning** (division of labor)
- These are HLD (High-Level Design) concepts

---

## 9. Wireshark — Network Analysis Tool

- Used by network administrators for packet sniffing/troubleshooting
- Shows: source/dest IPs, protocols, packet length, TCP sequence/ack numbers
- Can trace TCP handshakes, DNS resolution, HTTP content (unencrypted only)
- **HTTPS** packets are encrypted so you can't read the content
- Identifies suspicious patterns by MAC-to-IP mapping
- Pattern-based detection (replaced much of it with ML now): rapid IP changes, connection flooding, etc.

---

## 10. Important Exam & Interview Tips

### Debugging rule: **Rebooting fixes ~99% of network issues** — clears stuck lease states, DHCP problems, memory/congestion issues

### Voice agent architecture — interview question answer:
- Recording (voice → packets): **UDP**
- Text after speech-to-text (REST API): **TCP**
- Receiving voice responses: **UDP**
- Need to specify transport protocol **per API endpoint**, not one blanket answer

### Shortest path algorithms for exam: DFS, BFS, Dijkstra, Bellman-Ford, Prim's (maximum 2 from these)

### Cellular networks are different from internet — voice data converted to analog radio waves at different frequency bands (out of scope for this course but noted as related)

---

## Appendix A: Architecture Models Reference

*   **Client-Server:** Request-response model (e.g., HTTP cycle). Server is centralized.
*   **Peer-to-Peer (P2P):** No central server; all nodes communicate directly (e.g., BitTorrent, WebRTC optimization).
*   **Reference Models:** 7-layer OSI & TCP-IP networking models were referenced in class discussions.

> *These models apply across all protocols covered above.*

---

## Summary of Protocol Transport Layers Used

| Protocol | Transport Layer | Notes |
|----------|----------------|-------|
| HTTP/1, HTTP/2 | TCP | — |
| HTTP/3 | UDP + QUIC | Multiplexing, no head-of-line blocking |
| FTP | TCP (two connections) | Control + data |
| WebSockets | TCP | Full-duplex, persistent |
| WebRTC media | UDP | Low latency preferred over reliability |
| BitTorrent peers | UDP (discovery) + TCP (transfer) | — |
| SMTP | TCP | Sending only |
| IMAP / POP3 | TCP | Receiving only |
| DHCP | UDP | DORA process |

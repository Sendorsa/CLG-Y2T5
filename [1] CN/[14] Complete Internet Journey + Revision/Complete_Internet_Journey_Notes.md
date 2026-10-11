# Complete Internet Journey — Structured Revision Notes

---

## Class Overview

This revision covers networking protocols and their real-world applications, focusing on the **transport** and **application** layers. Topics include TCP/UDP fundamentals, HTTP evolution (1.x → 2 → 3), FTP architecture, WebSockets for persistent communication, WebRTC for peer-to-peer media, and BitTorrent's decentralized file distribution. These concepts underpin scalable, efficient networking solutions used in modern digital communications.

---

## 1. Transport Layer: TCP vs UDP

### TCP (Transmission Control Protocol)

**Key characteristics:** Connection-oriented, reliable, byte-stream oriented.

- **Three-way handshake:** `SYN → SYN-ACK → ACK` (establishes connection)
- **Sequence numbers (`seq`):** Byte-level tracking; each segment's `seq = last_byte_received + 1`. Enables reassembly of bytes in correct order.
- **Acknowledgements (`ack`):** `ack = next_expected_byte_seq`. Unacked segments are retransmitted.
- **Flow control:** Receiver advertises a **window size** (how much it can buffer). Sender never exceeds the advertised window.
- **Congestion control (slow start → congestion avoidance → fast retransmit/fast recovery):** Increases `congestion_window` exponentially (doubles per RTT) until loss/drop, then halves.
- **Termination:** Each side sends a `FIN`, receiving an `ACK` — two independent half-closes before connection fully closes.

### UDP (User Datagram Protocol)

**Key characteristics:** Connectionless, unreliable, datagram-oriented.

- No handshake, no retransmission, no ordering.
- **Datagram boundary is preserved** (unlike TCP byte-stream).
- Minimal overhead: 8-byte header (sport, dport, length, checksum).
- Used where speed > reliability: DNS, DHCP, streaming, VoIP, WebRTC signaling/media, QUIC (UDP-based)

### Comparison Table

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery, ordering | Best-effort, no guarantee |
| Ordering | Sequence numbers reorder | No sequencing |
| Flow control | Window-based | None |
| Congestion control | Yes (slow start etc.) | None |
| Overhead | Higher (handshake, ACKs, headers) | Lower (8-byte header) |
| Data model | Byte stream | Datagram boundaries preserved |
| Use cases | HTTP, FTP, SSH, SMTP | DNS, DHCP, VoIP, WebRTC, QUIC |

---

## 2. HTTP Versions & Client-Server Model

### Client-Server Model (Fundamental Architecture)

```
Browser/Client                          Server
┌─────────────┐              ┌──────────────────┐
│ Send request │─────────────› │ Parse request    │
│ Wait for     │              │ Process / Query    │
│ response     │←───────────── │ Generate response  │
└─────────────┘              │ Send response      │
                             └──────────────────┘
```

- **Request cycle:** `connect → send request → receive response → close/read`
- HTTP is **request/response** (pull-based): client initiates, server responds.
- Stateless protocol: each request is independent (state maintained via cookies/sessions).

### HTTP Evolution

#### HTTP 1.x (HTTP/1.0 & HTTP/1.1)

- Each resource requires a **separate TCP connection** (or pipelining in 1.1, rarely used).
- **Head-of-line blocking:** Entire response blocked waiting for headers/body.
- Headers sent **uncompressed** — redundant (e.g., Cache-Control repeated across every request).
- HTTP/1.1 improvements: persistent connections (`Connection: keep-alive`), chunked transfer encoding, pipelining (theoretical).

#### HTTP/2

- **Multiplexing:** Multiple request/response streams over a single TCP connection — no head-of-line blocking at the application level.
- **Header compression (HPACK):** Compresses headers using a dynamic table + Huffman coding; eliminates redundancy.
- **Binary framing:** Frame-based protocol instead of text; supports prioritisation & server push (`PUSH_PROMISE`).
- One connection per host — reduces TCP slow-start penalty vs HTTP/1.x fan-out.

#### HTTP/3 (QUIC over UDP)

- Replaces TCP with **QUIC** (Quick UDP Internet Connections) built on UDP.
- **Eliminates TCP head-of-line blocking:** packet loss for one stream does NOT block others.
- Uses **stream numbers** for efficient multiplexing across QUIC's UDP transport.
- Built-in TLS 1.3 — encrypted by default.
- **0-RTT resumption:** Cached connection parameters allow sending data immediately on reconnect.
- Connection migration via connection IDs (platform support varies; not universally available).

### Comparison Table: HTTP Versions

| Feature | HTTP/1.x | HTTP/2 | HTTP/3 |
|---|---|---|---|
| Transport | TCP | TCP | **UDP (QUIC)** |
| Multiplexing | ❌ (per connection) | ✅ Single connection, multiple streams | ✅ Multiple streams over QUIC |
| Header compression | ❌ | ✅ HPACK | ✅ QPACK |
| Connection setup | Full TCP + TLS handshake | Full TCP + TLS handshake | **0-RTT** (resumed), ~1-RTT (new) |
| Head-of-line blocking | Yes (header delay) | Reduced (stream-level only) | **Eliminated** (per-packet) |
| Server push | ❌ Not specified in HTTP/1.x spec | ✅ PUSH_PROMISE | ❌ Removed from final RFC (never implemented) |
| Encryption | Optional (TLS) | Typically TLS | **Always encrypted by default** |

---

## 3. FTP (File Transfer Protocol)

### Architecture

FTP uses **two separate TCP connections**:

1. **Control connection** (port 21): Sends commands (`RETR`, `STOR`, `LIST`) — persistent for the session
2. **Data connection** (ephemeral port): Transfers file data, opens/closes per command

### Connection Modes

#### Active Mode

```
Client                              Server (control:21)
  │                                   │
  ├── PASSIVE / PORT cmd ────────────►│  Client tells server its IP & ephemeral port
  │          (PORT:port#)             │
  │◄──── Connection from server ──────┤  Server DIALS client on specified port
  │        source port = 20           │
  │                                   │◄── data connection (server → client)
```

- **NAT traversal problem:** Server tries to dial the client's *private* IP — always fails behind NAT. Requires static ports on NAT router + FTP ALG (Application Layer Gateway).

#### Passive Mode (PASV)

```
Client                              Server (control:21)
  │                                   │
  ├── PASV ───────────────────────────►│  Ask server for a port
  │◄──── 227 Entering Passive Mode (IP, p1*256+p2, port#)─┤
  │              = data port           │
  ├── DIAL to server on data port ────►│  Client DIALS server on the port
  │          source: ephemeral         │  it just learned
  │                                   │────> data connection (client → server)
```

- **NAT-friendly:** Both connections initiated by client. Most common in practice.
- Still needs NAT traversal for the *server* (server must advertise correct IP).

### FTP over TLS/SSL — VSFTP

- Adds an SSL layer between FTP commands and TCP; both control + data encrypted.
- Often called **FTPS** or **VSFTP**. Configurable via `vsftpd.conf`.

---

## 4. WebSockets

### Overview

Bidirectional, persistent connection over a single TCP link — enables server-to-client push without polling overhead.

### Upgrade Mechanism

```
Client                              Server
  │      GET /chat HTTP/1.1           │
  ├── Host: example.com───────────────►│
  ├── Upgrade: websocket              │
  ├── Connection: Upgrade             │
  ├── Sec-WebSocket-Key: xxxxx        │
  ├── Sec-WebSocket-Version: 13       │
  │                                   │
  │      HTTP/1.1 101 Switching Protocols
  │◄─── Upgrade: websocket────────────┤
  │◄─── Connection: Upgrade───────────┤
  │◄─── Sec-WebSocket-Accept: yyyyy   │
  
  ═══════ WebSocket Frame Exchange (bidirectional) ══════
```

Key steps:
1. Client sends HTTP GET with `Upgrade: websocket` header — triggers **handshake**
2. Server responds `101 Switching Protocols` if it accepts
3. Connection is upgraded to a **full-duplex** persistent socket
4. Frames exchanged (text/binaries) over the same TCP connection

### Keepalive & Liveness

- **WebSocket Ping/Pong frames:** Built-in mechanism (like HTTP heartbeats)
  ```
  Client ── Frame[OPCODE=PING, body="heartbeat"] ──► Server
  Client ◄── Frame[OPCODE=PONG,   body="heartbeat"] ──┤
  ```
- If no pong within timeout → connection dead → client reconnects (exponential backoff).

### WebSocket vs SSE Comparison

| Feature | WebSockets | Server-Sent Events (SSE) |
|---|---|---|
| Direction | **Bidirectional** | Server → Client only (pull for requests) |
| Transport | WSS (persistent socket) | HTTP long-lived response |
| Reconnect | Manual or application logic | **Automatic on disconnect** |
| Built-in keepalive | ✅ Ping/Pong frames | ✅ Periodic `:` comment / timeout |
| Complexity | More complex frame protocol | Simple text stream |
| Use case | Chat, games, collaborative editing | News feeds, stock ticker, notifications |

---

## 5. HTTP Polling & Server-Sent Events

### Short Polling (Classic Push Simulation)

```
Client                              Server
  │      GET /updates                   │
  ├── "Any new data?" ────────────────►│
  │      HTTP 200, body: []             │
  │◄── empty response ─────────────────┤
  ... wait N seconds ...
  │      GET /updates                   │
  ├── "Any new data?" ────────────────►│
  │      HTTP 200, body: [new items]    │
  │◄── data ───────────────────────────┤
```

- **Pros:** Simple (plain HTTP)
- **Cons:** Latency = polling interval; wasted bandwidth on empty polls; server overload with many clients

### Long Polling (Comet)

```
Client                              Server
  │      GET /updates?timeout=30        │
  ├── "Any new data?" ────────────────►│
  │              ... waits up to 30s ...
  │      HTTP 200, body: [new items]    │
  │◄── response with data ─────────────┤
  │      (immediately reconnects)        │
      REPEAT...
```

- Server holds the connection open until new data arrives or timeout expires.
- **Pros:** Near real-time, low latency, no empty polls.
- **Cons:** Persistent connection on every poll; server resource usage.

### Server-Sent Events (SSE)

Native HTTP event stream — built-in reconnect, keepalive, event typing.

```
Client                              Server
  │      GET /events                    │
  ├── "Open stream" ──────────────────►│
  │                                    │
  │         Event [data: {temp=22}]    │
  │◄── data: {"type":"reading","val":22}┤
  │         (newline separated)          │
  │                                    │
  │         Event [data: {temp=23}]    │
  │◄── data: {"type":"reading","val":23}┤
```

- Standard event format: `event-name\ndata\nid:sent-event-id\n\n`
- **Retry** hints: `retry:3000` (client auto-reconnects after 3s on disconnect)
- Server can send special comment lines `:` for keepalive
- One-way: server → client. Client sends data via a separate POST request.

---

## 6. Video Communication Protocols (WebRTC)

### WebRTC Architecture

```
Browser A                         Browser B
     │                               │
     │         Internet              │
PeerConnection       ── P2P media stream (RTP/RTCP over UDP)──▶
```

**Three essential components:**

1. **PeerConnection (WebRTC):** Full-duplex P2P RTP media stream — audio/video codecs, NAT traversal via ICE/STUN/TURN
2. **Signaling API (application-level, e.g., WebSocket):** Exchange SDP offers/answers + STUN candidates between peers (WebRTC does NOT define signaling)
3. **MediaStream API:** Captures from `navigator.mediaDevices.getUserMedia()` (camera/mic)

### NAT Traversal — ICE / STUN / TURN

```
Peer A ── STUN server (Internet) ── discovers public IP/port
Peer B ── STUN server (Internet) ── discovers public IP/port
Peer A ───────── P2P hole ──────── Peer B   ← NAT punchhole: direct connection

NAT blocks it?
     │
     ├── TURN relay server ── forwards media via tunnel ──▶ TURN
```

**ICE (Interactive Connectivity Establishment):** Process that tries all possible candidate pairs until a working one is found:

| Candidate type | How discovered | Description |
|---|---|---|
| **Host candidate** | Local ICE agent | Private IP + port (e.g., 192.168.1.5:4000) |
| **STUN candidate** | STUN server query | Public IP + port as seen by STUN server |
| **Relay candidate** | TURN server allocates Fingerprinted allocation | Relay address on TURN server (always works) |

**Order of preference:**
1. **Host candidate pair** → direct P2P over local network ✅ best
2. **Server reflexive (STUN)** → public IP punchhole ✅ works in most cases
3. **Relay (TURN)** → always works, but highest latency/bandwidth cost ❌ backup

### SDP (Session Description Protocol)

- Exchanged during signaling to negotiate: media type, codecs (`opus`, `vp8`), encryption, candidates
- Not part of this revision — details omitted per scope.

---

## 7. Torrent Protocol (BitTorrent P2P)

### Core Architecture

```
Peer A                          Peer B                        Peer C
     │                               │                               │
     └─────────── Internet ─────────┴───────────────────────────────┘
         Seed / Tracker                 Leecher                    Seeder
    (has full file)               (downloading)                (completes then seeds)
```

- **No central server files:** Peers download chunks from each other.
- **Tracker** announces peers (lists IP/port of swarm members). Modern: DHT / PEX replace trackers.

### Chunk-Based Downloading Process

1. User has `.torrent` file → contains **info-hash** + tracker metadata
2. Client contacts **tracker** → gets list of peers in the swarm
3. Connect to peers, exchange chunk requests using **bitfield** (which chunks each peer has)
4. Download chunks *simultaneously* from multiple peers → reassemble into complete file
5. Each chunk is a fixed-size block: default **1 MiB** (original BitTorrent spec)

### In-Protocol Flow

```
Peer A (has chunks 1,3)          Peer B (has chunks 2,4)
     │                                  │
     ├── GET piece 2 ──────────────────►│    Download from B
     │◄── CHUNK 2 ─────────────────────┤
     │                                  │
     │      exchange bitfields /       │
     │             requests              │
     ├── PIECE REQUEST (piece=3) ──────▶│    Ask A for piece 3
     │◄── CHUNK 3 ─────────────────────┤
     
     (repeats with all peers in swarm)
```

### Key Concepts

- **Info-hash:** SHA1 hash of the info-dictionary in the torrent file → uniquely identifies a file/swarm. Used for DHT lookups.
- **Swarm:** Group of peers sharing one torrent.
- **Seeder:** Has complete file; only uploads.
- **Leecher:** Downloading; also uploads chunks it has.
- **Rarest-first / Tit-for-tat (anti-choking):** Peers prefer to download rarest pieces and reciprocate upload bandwidth — optimises swarm health.
- No central storage — fully distributed P2P → no single point of failure.

---

## Analogies & Key Comparisons

### HTTP vs WebSocket — Postal Service vs Phone Call

- **HTTP is like postal mail**: Each request needs a stamp, routing, and processing. Messages go one-way (request → response), then you start again.
- **WebSocket is like a phone call**: One connection stays open; both parties exchange information freely without re-establishing contact each time — true bidirectional communication.

### HTTP Evolution at-a-glance

| Version | What changed? | Key benefit |
|---|---|---|
| **HTTP/1.x** | One TCP per resource, no compression | Simple, works everywhere |
| **HTTP/2** | Multiplexed streams + HPACK headers | Parallel over one connection; less bloat |
| **HTTP/3 / QUIC** | UDP underlay; 0-RTT, per-stream flow control | No TCP HOL blocking; faster connects |

### Real-time Delivery Spectrum

```
Lowest latency ←——————————————————————————→ Highest reliability

UDP (WebRTC, streaming)   WebSockets     TCP-based HTTP/2/3
(fast, no guarantees)    (bidirectional)  (reliable, ordered)
```

---

## Quick Summary: Communication Paradigms Comparison

| Protocol | Direction | Connection Type | Keepalive | Best Use Case |
|---|---|---|---|---|
| **HTTP 1.x** | Pull (client→server) | Per request TCP | Keep-alive header | Static pages, simple APIs |
| **HTTP/2** | Pull with push | Multiplexed over TCP | No explicit | Modern web apps |
| **HTTP/3** | Pull with push | Streamed over QUIC | Embedded in QUIC | Low-latency web (QUIC) |
| **WebSockets** | Bidirectional | Persistent socket | Ping/Pong frames | Chat, games, editing |
| **SSE** | Server→Client stream | Long-lived HTTP response | `:` comment / retry | Feeds, telemetry |
| **Long Polling** | Pull (simulated push) | Held-open TCP | N/A | Legacy push for simple clients |

## 8. Protocol Ports & Application-Specific Details

### HTTP/2 & RPC/gRPC over Multiplexed Streams

- gRPC and modern RPC frameworks rely on HTTP/2 **multiplexing** to transmit multiple independent RPC calls concurrently over a single TCP connection — each call runs as its own stream.
- Without multiplexing, parallel RPCs would need separate TCP connections (increasing latency via slow-start) or suffer head-of-line blocking.

### HTTPS over QUIC (UDP Port 443)

- HTTPS on port **443** traditionally used TCP; with HTTP/3 it shifts entirely to **QUIC over UDP**.
- No two-step connection dance: TLS is embedded in the QUIC handshake — no separate TCP `SYN` → `ACK`.
- Connection migration via **connection IDs** survives IP changes (WiFi ↔ LTE).

### BitTorrent Chunk Integrity

- Each chunk in a torrent is independently hashed with **SHA1** upon receipt; only verified chunks are uploaded to peers.
- **Merkle tree** (BitTorrent v2; original spec: flat hash list); the root hash equals the **info-hash** in the `.torrent` metadata — uniquely identifying the shared file/swarm.

### WebRTC Group Communication Scaling

- Peer-to-peer mesh gets complex at scale (N² connections for N peers).
- For group calls, traffic routes through a central **Selectively Forwarding Unit (SFU)** instead of broadcasting directly to every participant — SFU selects and forwards only the streams each peer wants/receives.

### WebRTC Port Range & ICE Candidates

- Media uses UDP ports; STUN defaults to **3478**, TURN to **3478/5349** for candidacy collection on local machines.
- Each host has multiple candidate types: **Host** (local private IP/port) and **Server-reflexive** (public IP/port via STUN query).

## 9. Glossary — Quick Reference

| Term | Definition |
|---|---|
| TCP | A reliable transport layer protocol ensuring packet delivery and order. |
| UDP | A fast and unreliable transport layer protocol with no delivery guarantees. |
| Transport Layer | The layer responsible for packet delivery between systems in a network. |
| Application Layer | The layer where protocols determine the type of data being sent. |
| HTTP | A client-server protocol for requesting resources over the web. |
| HTTPS | An extension of HTTP with secure data encryption. |
| TCP Handshake | A three-step process to establish a TCP connection between a client and server. |
| HTTP/1.1 | Supports single TCP connection for multiple sequential data streams. |
| HTTP/2 | Introduces multiplexing to allow simultaneous packet transfers in one connection. |
| Multiplexing | Sending multiple signals or data streams on a single connection. |
| HTTP/3 | Uses UDP with QUIC for faster, reliable connections using multiple streams. |
| QUIC Protocol | Adds reliability to UDP with sequence and stream numbers for HTTP/3. |

(End of file - total 412 lines)

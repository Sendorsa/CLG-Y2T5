# Computer Networks - Detailed Notes

**Source: Class Scaler Academy Introduction to Computer Networks – Lecture 1**

---

## 1. What is a Computer Network?

### Definition
- **Computer System:** Any device running an operating system that exchanges data through a network (laptop, desktop, smartwatch, washing machine, AC, etc.)
- **Computer Network:** A collection of interconnected devices/systems exchanging data via links (wired or wireless)
- **Internet:** A collection of multiple computer networks connected together

### Key Insight
A network is the **basic building block** of the internet. The internet = interconnected computer networks.

---

## 2. Why Study Computer Networks?

### For Scalable System Design
All communication between backend services happens over the **network layer**. If this layer has faults, user experience suffers:
- High latency → poor UX
- Data loss or corruption
- Communication failures between microservices

### Industry Example (Google Overhead Problem)
- Google estimated wasting ~200–300 years of computation daily using generic protocols internally
- This problem motivated the creation of specialized industry protocols (see **Section 5b: Industry-Built Protocols**)

---

## 3. Core Problems in Computer Networks

### Problem 1: IP Addresses & DNS

#### IP Address
- Every device on a network has a unique numerical identifier (e.g., `128.115.1...`)

#### Why Not Use IPs Directly?
| Approach | Pros | Cons |
|----------|------|------|
| **IP Address** | Machine-readable | Hard for humans to remember (e.g., 10-digit number) |
| **URL/Domain Name** | Easy to remember (e.g., `amazon.com`) | Need translation mechanism → **DNS** |

### How DNS Works
1. User types URL (`amazon.com`) in browser
2. Browser contacts **DNS Server** first
3. DNS returns the IP address of `amazon.com`
4. Browser creates connection with that IP

#### DNS Reliability
- DNS could be a single point of failure → entire internet down if DNS fails
- **Reality:** Internet has never gone completely down despite pandemics, wars, natural disasters
- **Solution:** DNS is designed with redundancy and distributed architecture (studied in detail later)

#### Key DNS Questions
1. How does the browser know the DNS server's IP address? (No URL for DNS!)
2. Who maintains DNS servers? (Answered through delegation hierarchy studied in course)
3. How is IP lookup so fast? (Data structures and algorithms power DNS)

---

### Problem 2: Data Transmission & Efficiency

#### Scenario: Streaming a Movie on Netflix
- Movie size: **10–12 GB** (HD quality)
- If downloaded all at once: Long waiting time = `Data / Network Speed`
- Real-world: Network is **unreliable** → downloads fail and restart

##### 2a. Data Corruption & Security

- **Malicious Users (Eavesdropping):** Unencrypted data can be intercepted and read; sensitive info like payment details, bank details, personal data → easily decrypted by attackers

- **How Data is Fundamentally Stored**
  | Data Type | What it Is at Bit Level |
  |-----------|------------------------|
  | Video | Matrix of pixels → RGB values → integers → bits |
  | Image | Matrix of pixels → color/transparency values → bits |
  | Audio | Analog → converted to bits |
  | Software/Files | Everything = bits |

- **Corruptions During Transmission**
  - Electrical voltage fluctuations (copper wires)
  - Light signal disruptions (optical fiber)
  - **Question:** How do we verify received data is correct?
  - Answers are mechanisms that themselves could be corrupted → **recursive problem** to solve

---

##### 2b. Bandwidth & Packetization

- **Analogy: Transporting Luggage**

  | Approach | When It Works | When It Fails |
  |----------|--------------|---------------|
  | One large truck | Wide, reliable roads | Narrow roads → traffic/congestion |
  | Many small vehicles | Wide roads (less efficient use) | Narrow roads (better handling) |

- **Solution: Packets (Data Packetization)**
  - Break large data into small chunks → called **packets**
  - **Benefits:**
    - Partial transfer failure → only re-send corrupted packets (not whole data)
    - Smaller packets → less congestion, faster routing through network
    - Compromised packet ≠ entire data exposed

---

### Problem 3: Packet Ordering

#### Why Ordering Matters
- If Netflix receives video packets out of order → seconds appear scrambled (2nd over before 1st)
- Live streaming requires **correct sequence** for playback

#### Can't We Just Use a Queue?

##### Simple Answer: YES (in theory)
- Push packet → FIFO queue → pop in same order

##### Real Problem: Internet is NOT a simple queue
- Packets may take **different routes** through the network
- Some packets arrive earlier due to shorter/faster paths, not because they were sent first
- Router loads and congestion → unpredictable timing

---

## 4. Internet as a Data Structure: Graphs

### Network Topology
```
User's Devices → Router → ISP (Airtel/Jio/etc.) → ISP Routers → 
    → Millions of devices, networks, ISPs → Amazon's servers
```

### Internet = Graph Data Structure
- **Nodes:** Routers, switches, server endpoints, ISPs
- **Edges:** Connections between nodes (physical links)
- Packets follow **different paths** through the graph based on:
  - Shortest path? Not always fastest!
  - Heavily loaded routers → longer paths may be faster
  - Dynamic congestion & traffic

### Routing Algorithms
- Need algorithms to determine best path for each packet
- Builds on DSA graph concepts (BFS, DFS, Dijkstra's shortest path, etc.)

---

## 5. Network Protocols

### Why Do We Need Protocols?

| Communication Scenario | Needs? |
|------------------------|--------|
| External client ↔ Server (internet) | Auth, tokens, cookies, security ✅ |
| Internal server ↔ Server (within company) | Same overhead? ❌ Wasteful! |

### What Are Network Protocols?

A set of rules that enable communication between devices regardless of:
- Operating system
- Memory/computation power
- Native language/format

> **Analogy:** Two people from different countries can communicate by agreeing on a common language & grammar → same principle for computers.

### Why Study Existing Protocols (TCP, UDP, HTTP, etc.)?

#### Reason #1: They Are NOT Perfect for Every Use Case
- Designed for client-server communication with external entities
- Heavy overhead for internal microservices
- Google's analysis: Wasting ~200-300 years of computation daily using generic protocols internally

#### Reason #2: Innovation Requires Deep Understanding

**📋 See consolidated details in Section 5b: Industry-Built Protocols.**
All were created by engineers who understood existing protocols → their limitations → constraints → and how to build better alternatives.

---

## 5b. Industry-Built Protocols

| Protocol | Creator | Use Case | Key Innovation |
|----------|---------|----------|----------------|
| **gRPC** | Google | Internal service-to-service communication | Efficient serialization; eliminated unnecessary overhead (auth, tokens, cookies not needed internally); saved ~200–300 years of computation daily across millions of servers |
| **WebRTC** | Google | Real-time video/audio/chat | Direct P2P connections for low-latency real-time communication |
| **Kafka** | LinkedIn | Protocol-based messaging system with proprietary data transfer protocol | Built its own transport protocol optimized for messaging workloads |

> **Key Insight:** Each innovation followed the pattern: Real-world problem → Understanding existing protocol limitations → Identifying constraints → Building a better alternative.

---

## 6. Postal System Analogy (Layers Explained)

### How Physical Mail Works → Parallels with Network Layers

| Stage | Action | Packet Layer Equivalent |
|-------|--------|----------------------|
| 1. Letter from sender | Recipient name/address on envelope | **Application Data** (raw content + recipient address) |
| 2. Postbox collection | Postman collects letters from hundreds of postboxes | **Network Aggregation** |
| 3. Sorting at local post office | Group by destination → packets for Bihar, Tamil Nadu, etc. | **Encapsulation / Routing Prep** |
| 4. Add truck ID & station info | New metadata added over the packet | **Transport Header** (sequence, order info) |
| 5. Truck → Train station | Multiple post office packets loaded into same train; railways add tracking/routing info | **Network Header** (IP addresses, routing data) |
| 6. At destination station | Remove railway info → offload at destination post office | **Decapsulation (decreasing layers)** |
| 7. Local sorting in Bihar | Sort by district, local post offices | Final routing to recipient's neighborhood |
| 8. Postman delivery | Strip all extra packaging; only original letter remains | Recipient gets raw data intact |

### Key Takeaways
- **Every layer adds its own metadata** (headers) for routing/reliability
- **Every layer strips away its info** when passing upward at the destination
- Analogy explains **7-layer OSI Model**: Logical layers of abstraction, not physical barriers

---

## 7. The 7-Layer OSI Model

*(Overview from lecture; detailed study ahead in course)*

| Layer | Responsible For | Examples |
|-------|----------------|---------|
| **Application** (Layer 7) | User interface / Application logic | HTTP, HTTPS, FTP, SMTP |
| **Presentation** (Layer 6) | Encryption, compression, translation/encoding | Data encryption, SSL/TLS wrapping |
| **Session** (Layer 5) | Establish & manage connections throughout interaction | Session management, token handling |
| **Transport** (Layer 4) | **TCP**: reliable data delivery, error checking, packet ordering, retransmission <br> **UDP**: unreliable, faster delivery | TCP, UDP |
| **Network** (Layer 3) | Routing, addressing (IP), path selection | IP, routers, routing algorithms |
| **Data Link** (Layer 2) | Node-to-node delivery, MAC addressing, error detection | Ethernet, switches, frames |
| **Physical** (Layer 1) | Raw bit transmission over physical medium | Cables, radio waves, voltage/light signals |

### Where Does the Browser Act?
- Browsers implement **Application**, **Transport**, and partly **Session** layer functions
- Lower layers are handled by OS & networking hardware

---

## 8. Course Syllabus Overview

### What Will Be Covered
1. **Introduction to Internet & Packets** – fundamental concepts, terminology
2. **7-Layer Model (OSI)** – detailed study of each layer and its functions
3. **Transport Layer Deep Dive** – TCP vs UDP, reliability, flow control, error handling
4. **Industry-Specific Protocols** – gRPC, WebRTC, Kafka — built by engineers with deep protocol knowledge (see Section 5b)
5. **Network Devices** – Router, Switch – their roles & differences
6. **Packet Analysis & Troubleshooting** - wireshark/tcpdump to inspect packet contents

### Prerequisites for System Design
- Computer Networks → prerequisite for upcoming **High-Level System Design** course
- All server-to-server interactions (databases, caches, load balancers, microservices) rely on networking fundamentals

---

## 9. Grading & Assessment

| Component | Details |
|-----------|---------|
| **Assignments** | MCQs (single/multi-select), numerical problems, coding (e.g., socket programming for chat app) |
| **Quiz** | Mid-course assessment → exempted if approved internship secured |
| **End-Term Exam** | Two sections: (a) questions from assignments, (b) new end-term questions <br> Final score = problem-solving percentage × exam score |
| **Assignments Source** | Dashboard + GitHub problems with auto-evaluation on cloud |

### Important Note for Quiz Exemption via Internship
- If internship is secured → quiz is exempted
- Replacement: Viva conducted during end term

---

## 10. Key Concepts & Principles

### 🧠 Core Takeaway #1: Trade-offs Over "Correct" Solutions
> In DSA (data structures & algorithms): well-defined problems → optimal solutions exist → trade-off = time vs space
> 
> **In Computer Networks (and all later subjects):** Every solution introduces new problems. No good/bad solutions, only **suitable solutions for the use case**.

### 🧠 Core Takeaway #2: Protocol Design Philosophy
```
Problem → Existing Solution → Limitations Identified → New Protocol Created
     ↓                                    ↓
   Real-world need                Deep understanding required
   (e.g., latency in video chat)   (know constraints & architecture)
```

### 🧠 Core Takeaway #3: The Network is Unreliable by Default
When designing scalable systems always assume:
- Data WILL be corrupted
- Packets WILL arrive out of order
- Connections WILL drop
- Bandwidth WILL be limited
- **Goal:** Minimize (never eliminate) failure probability

---

## 11. Socket Programming Preview

### Application Example: Messaging App
Using socket programming to build a WhatsApp-like application:

### Problem with HTTP for Messaging
```
Request → Connect → Send data → Disconnect → Repeat
```
- Each HTTP request creates/disconnects connection
- High latency ❌ Unacceptable for messaging

### Solution: Persistent Connection
- Long-lived TCP connections (WebSockets)
- Low-latency, bidirectional communication
- Study of protocols that support this → **Socket Programming**

---

## 12. Glossary of Key Terms

| Term | Definition |
|------|-----------|
| **IP Address** | A unique address assigned to each device on the internet used for identification and communication. |
| **URL / Domain Name** | Human-readable address (e.g., amazon.com) |
| **DNS** | Domain Name System translates human-friendly domain names to IP addresses. |
| **Packet** | A small chunk of data sent over a network, used to transmit information efficiently and securely. |
| **Latency** | Time delay for data to travel from source to destination. |
| **Protocol** | Set of rules governing network communication (TCP, UDP, HTTP, etc.) — see Section 5b for industry-built protocols (gRPC, WebRTC, Kafka) |
| **Routing** | Determining the path packets take through the network graph |
| **Queue** | Data structure that ensures packets are processed and transported first-in, first-out. |
| **Network Protocol** | A set of rules that governs data communication between different devices. |
| **TCP** | Transmission Control Protocol ensures reliable transmission of data over a network. |
| **gRPC** | A Google-developed protocol for efficient remote procedure calls. |
| **Serialization/Deserialization** | Conversion of data from human-readable form to bits and vice versa |
| **WebRTC** | A protocol developed by Google for real-time audio & video communication. |
| **Kafka** | A data streaming platform developed by LinkedIn with its own transfer protocol. |
| **Router** | A device that routes data between devices on the same network and beyond. |
| **Graph Data Structure** | The underlying data structure that models network connections. |
| **Encapsulation** | Adding headers/metadata at each layer during transmission downwards |
| **Decapsulation** | Removing layers/headers during reception going upwards |

---

## 13. Recommended Reading

- Textbook link to be shared via syllabus sheet → read after lectures to reinforce concepts

---

*Notes prepared from: Introduction_to_Computer_Networks_-_Class_Scaler_Academy.txt*

# Comprehensive Revision Notes: TCP Connections and Socket Programming

## Overview
This class focused on understanding TCP connections and socket programming. It delved into the details of establishing a TCP connection through the three-way handshake process, the roles of input and output streams in socket communication, and implementing an Echo Server and Client application using TCP in Java. Furthermore, the class introduced the concept of UDP and explored potential differences between TCP and UDP connections.

## Topics Covered
TCP connection establishment (3-way handshake), DNS resolution, client-server architecture, Java Socket programming (ServerSocket and Socket), input/output streams, newline delimiters, Echo Server/Client implementation, HashMap-based data lookup over network, UDP vs TCP differences.

---

## 1. TCP vs UDP

### TCP (Transmission Control Protocol)
- **Reliable**, connection-oriented protocol
- Guarantees **no bytes are dropped or lost** during transmission
- Guarantees **byte ordering** — data arrives in the exact order it was sent
- Transmits a **stream of bytes** (not discrete messages); TCP does not know message boundaries
- Requires a **connection to be established first** via 3-way handshake
- Uses the operating system's TCP stack automatically

### UDP (User Datagram Protocol)
- **Unreliable**, connectionless protocol
- Does **not guarantee delivery** — packets may be dropped or never arrive
- Does **not guarantee ordering** — packets may arrive out of order
- Sends data as **separate, individual datagrams**
- No 3-way handshake; simpler and faster
- Use Java's `DatagramSocket` for UDP networking

### Key Difference: Message Boundaries
```
// Sending on TCP side (2 separate println calls):
outputStream.println("hello");
outputStream.println("world");

// On the receiving side, these bytes arrive as ONE continuous byte stream:
// You might receive: "h", "el", "lo\nwor", "ld\n" in any byte chunks
// The newline delimiter (\n) is what tells the receiver where a message ends
```

---

## 2. DNS Resolution Flow

Before any TCP connection is established, when a client opens a URL or hits an API:

1. The client extracts the host/domain from the URL
2. **DNS resolution** translates the domain name to an IP address
3. The client now knows:
   - Destination IP address
   - Destination port number (e.g., 80 for HTTP, 5089 for custom servers)
4. Only then does the client initiate a TCP connection

---

## 3. TCP 3-Way Handshake

Establishes a TCP connection between client and server:

### Step 1: SYN (Synchronize)
- Client sends a segment to the server with a **random sequence number** (e.g., `seq = 7000`)

### Step 2: SYN-ACK (Synchronize-Acknowledge)
- Server responds with TWO things:
  - Its own **random sequence number** (e.g., `seq = 3003`)
  - An **acknowledgement** = client's sequence number + 1 (i.e., `ack = 7001`), meaning "I have received everything up to 7000, I now need 7001"

### Step 3: ACK (Acknowledge)
- Client sends back an acknowledgement = server's sequence number + 1 (e.g., `ack = 3004`)
- At this point, the connection is **established**

### What "Connection Established" Means
- On the server side, a **socket object** is created by the OS
- The socket records:
  - Client's IP address
  - Client's port number
  - Other TCP metadata
- This socket object provides methods to **read** (input stream) and **write** (output stream) data

---

## 4. Socket Programming Concepts

### Architecture Overview

```
CLIENT SIDE                         SERVER SIDE
┌───────────────┐              ┌──────────────────┐
│   Socket      │◄────────────►│   ServerSocket   │
│               │  TCP Pipe    │   (LISTENING)    │
│ ├─ InputStrm  │              │ ├─ InputStrm     │
│ └─ OutputStrm │              │ └─ OutputStrm    │
└───────────────┘              └──────────────────┘
```

### Key Concepts

- **ServerSocket** (server side): Opens a socket and listens on a specific port for incoming connections
- **Socket** (client side): Connects to a server's IP address and port
- **Socket** (server side, for each client): Created when `accept()` returns; holds one client's connection info
- **Input stream**: Receives data from the remote side (populated by TCP)
- **Output stream**: Sends data to the remote side (TCP picks up what you write)

### Critical Understanding: Socket Objects Are NOT Transferred Over Network
- Socket objects are created locally on each machine by the OS and application code
- Only **data** (messages, strings, bytes) is transmitted across the network
- The socket object on the server side holding client info is created BY the server's OS, not sent from the client

### Stream Analogy
- **Output stream = Postbox**: Push a letter in, TCP delivers it to the remote input stream
- **Input stream = Mailbox**: TCP drops incoming packets here; a Scanner worker reads them for your program
- TCP is managed entirely by the operating system — your code only writes to output streams and reads from input streams

---

## 5. Data Transfer in TCP

### Input and Output Streams
- **Input Stream:** Responsible for receiving data from the connected party's output stream. Populated by TCP, read by your code using `Scanner`.
- **Output Stream:** Sends data to the connected party's input stream. Your code writes to it; TCP picks up and delivers the data.

### Byte Stream — No Message Boundaries
- TCP sends a **continuous stream of bytes**, not discrete messages
- TCP does not understand or preserve message boundaries between `println()` calls

### Ordering vs. Delimiters
- **Ordering guarantee:** TCP ensures data arrives in the exact order it was sent, but at the byte level, not at the message level
- **Delimiters (`\n`):** Essential for partitioning messages and ensuring correct interpretation — they're the ONLY way to define where one message ends and the next begins

### Echo Server Concept (Analogy)
> An **Echo Server** repeats whatever the client sends back, similar to sound reflecting off a surface. This demonstrates basic socket communication: client sends → server receives → server echoes → client receives reply.

---

## 6. Java Echo Server Implementation

### Server-Side Code (`EchoServer.java`)

```java
import java.net.*;
import java.io.*;

public class EchoServer {
    public static void main(String[] args) throws IOException {
        // Step 1: Create ServerSocket and listen on a port
        ServerSocket serverSocket = new ServerSocket(5089);
        
        // Step 2: Wait for client to connect (blocking call)
        // OS performs the 3-way handshake behind the scenes
        Socket client = serverSocket.accept();  
        // Now client socket object holds info about this specific client
        
        // Step 3: Create input stream (to read what client sends)
        Scanner scanner = new Scanner(client.getInputStream());
        
        // Step 4: Create output stream (to send response back to client)
        PrintStream output = new PrintStream(client.getOutputStream());
        
        // Step 5: Read message from client
        String message = scanner.nextLine();
        System.out.println("Server received: " + message);
        
        // Step 6: Echo the same message back
        output.println(message);  
        // println() adds \n (newline delimiter) — CRITICAL for message boundaries
        
        // Step 7: Close connection
        client.close();
        serverSocket.close();
    }
}
```

### Why Newline Delimiter (`\n`) Is Critical
- TCP sends a continuous byte stream with no message boundaries
- Without `\n`, the receiver cannot determine where one message ends and the next begins
- `PrintStream.println()` automatically appends `\n` to each message
- `Scanner.nextLine()` reads until it encounters `\n`, then strips it

### Client-Side Code (`EchoClient.java`)

```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class EchoClient {
    public static void main(String[] args) throws IOException {
        // Step 1: Create socket to connect to server
        // localhost = same machine; use server's IP for remote machines
        Socket server = new Socket("localhost", 5089);
        
        // Step 2: Create output stream (to send messages to server)
        PrintStream output = new PrintStream(server.getOutputStream());
        
        // Step 3: Create input stream (to read server's response)
        Scanner scanner = new Scanner(server.getInputStream());
        
        // Step 4: Read custom message from user keyboard
        Scanner keyboard = new Scanner(System.in);
        System.out.print("Enter message: ");
        String message = keyboard.nextLine();
        
        // Step 5: Send message to server
        output.println(message);
        
        // Step 6: Read server's reply (blocks until data arrives)
        String reply = scanner.nextLine();
        System.out.println("Server replied: " + reply);
        
        // Step 7: Close connection
        server.close();
    }
}
```

### Port Selection / Troubleshooting
- If you get `"Address already in use"` error, the port is occupied by another application
- Solution: Change to an available odd-numbered port (e.g., `5089`)
- Both client and server must use the **same port number**

---

## 7. Advanced Echo Server with HashMap Data Lookup

### Enhanced Server (`PlayerServer.java`) — Server-Side

```java
import java.net.*;
import java.io.*;
import java.util.*;

public class PlayerServer {
    
    // Build the lookup map (separate helper to keep main clean)
    public static HashMap<String, Integer> buildMap() {
        HashMap<String, Integer> playerData = new HashMap<>();
        playerData.put("Sachin", 49);
        playerData.put("Rahul", 18);
        playerData.put("Dravid", 11);
        // ... add more entries
        return playerData;
    }

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(5089);
        HashMap<String, Integer> players = buildMap();
        
        while (true) {
            Socket client = serverSocket.accept();  // Accept each connection
            
            Scanner scanner = new Scanner(client.getInputStream());
            PrintStream output = new PrintStream(client.getOutputStream());
            
            // Process multiple queries from one connection
            while (scanner.hasNextLine()) {
                String playerName = scanner.nextLine();
                System.out.println("Server received: " + playerName);
                
                if (players.containsKey(playerName)) {
                    int centuries = players.get(playerName);
                    output.println(playerName + " is introduced by coach and " + centuries + " centuries");
                } else {
                    output.println("player does not exist");
                }
                
                // Client sends "end" to close the connection
                if (playerName.equals("end")) break;
            }
            
            client.close();
        }
    }
}
```

### Enhanced Client (`PlayerClient.java`) — Client-Side

```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class PlayerClient {
    public static void main(String[] args) throws IOException {
        Socket server = new Socket("localhost", 5089);
        
        PrintStream output = new PrintStream(server.getOutputStream());
        Scanner scanner = new Scanner(server.getInputStream());
        Scanner keyboard = new Scanner(System.in);
        
        while (true) {
            System.out.print("Enter player name (or 'end' to quit): ");
            String playerName = keyboard.nextLine();
            
            // Send query to server (blocks until TCP delivers it)
            output.println(playerName);
            
            if (playerName.equals("end")) break;
            
            // Read server's response (blocks until data arrives from server)
            String reply = scanner.nextLine();
            System.out.println(reply);
        }
        
        server.close();
    }
}
```

---

## 8. Full Request Flow (Client → Server Example)

### What Happens When Client Sends a Request:

```
1. Client code: output.println("Sachin")
   └─ Writes "Sachin\n" to client's output stream

2. TCP stack (OS): 
   └─ Converts string to bytes
   └─ Segments into TCP packets
   └─ Transmits through network layers

3. Server TCP stack (OS): 
   └─ Receives packets in correct order
   └─ Assembles bytes into server's input stream buffer

4. Server code: scanner.nextLine()
   └─ Reads from input stream until '\n'
   └─ Returns "Sachin" to the application

Server processes → HashMap lookup → output.println("Sachin... 49 centuries")
Response travels back through identical mechanism (reversed direction)
```

---

## 9. Running Client-Server on Different Machines

### Localhost vs IP Address
- **`localhost`**: Works when client and server are on the same machine
- **Private IP address**: Use for machines on the same subnet/network
- **Public IP + port forwarding**: Required for machines on different networks (configure router settings)

### Same-Network Communication Example
```java
// Server runs on Machine A:
ServerSocket server = new ServerSocket(5089);  // Listens on all interfaces

// Client on Machine B connects:
Socket server = new Socket("192.168.1.100", 5089);  // Machine A's private IP
```

- Both machines must be on the **same subnet/network**
- The server accepts only one client at a time (blocking `accept()`)
- For multiple simultaneous clients, the server needs threads or asynchronous I/O

---

## 10. UDP vs TCP Implementation Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Handshake | 3-way handshake (automatic, OS handles) | No handshake |
| Reliability | Guaranteed delivery and ordering | No guarantees |
| Java Class | `ServerSocket`, `Socket` | `DatagramSocket`, `DatagramPacket` |
| Connection | Connected (point-to-point) | Unconnected (datagrams sent independently) |
| Use case | Reliable data transfer, HTTP, file transfer | Real-time streaming, gaming |
| Server accepts clients | `serverSocket.accept()` returns a Socket | No accept — just receive datagrams |

### UDP Implementation Note
- UDP uses different socket classes (`DatagramSocket`)
- No 3-way handshake occurs
- Each datagram is independent — no persistent connection state

---

## 11. Key Takeaways

1. **TCP = reliable byte stream** with ordering guarantee; message boundaries rely on delimiters (e.g., `\n`)
2. **DNS resolution happens first** before any TCP connection to a domain name
3. **3-way handshake**: SYN → SYN-ACK → ACK (handled by OS, not application code)
4. **Socket objects are created locally** — only data travels over the network
5. **Input stream receives, output stream sends** — on both client and server sides
6. **Scanner + newline (`\n`)** is the standard pattern for reading/writing delimited messages in Java networking
7. **Hash Maps give O(1) lookup** — useful for server-side data queries over network
8. **Port conflicts** are common during development; use `netstat` or pick higher odd-numbered ports
9. **Client and server must match on port number** to communicate
10. **Same subnet = private IP works**; different networks require public IP + router port forwarding

## 12. Glossary of Key Terms

| Term | Definition |
|------|------------|
| TCP | A protocol ensuring secure and ordered data transmission over networks |
| UDP | A protocol allowing fast but unreliable data transmission without order |
| DNS resolution | The process of translating a domain name into an IP address |
| Three-way handshake | A TCP mechanism to establish a connection using SYN, SYN-ACK, and ACK |
| Socket | An endpoint for sending or receiving data across a network |
| Input stream | A means to read data from a source in a program |
| Output stream | A means to write data to a destination in a program |
| Stream of bytes | TCP sends continuous sequences of data as streams, not discrete messages |
| Delimiter | A character used to define boundaries between separate data items in TCP |
| IOException | An error that occurs due to failed input-output operations |
| Server socket | An object that listens for requests and establishes socket-based connections |
| Scanner | A Java class for parsing primitive types and strings using regular expressions |

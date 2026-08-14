# Networking — Phase 2: OSI & TCP/IP Models

> **Goal:** Understand how networking is divided into layers, what each layer is responsible for, how data moves through the layers, and how the OSI model maps to the practical TCP/IP model.

---

## 1. Why Do We Need Networking Layers?

Imagine an application wants to send:

```text
Hello
```

The application should not need to know:

- How Ethernet works
- How Wi-Fi transmits signals
- How routers forward packets
- How TCP handles retransmission
- How a network card sends electrical/radio signals

Instead, networking is divided into **layers**.

Each layer has a specific responsibility and provides services to the layer above it.

```text
Application
    ↓
Transport
    ↓
Network
    ↓
Data Link
    ↓
Physical
```

This is called **layering**.

### Benefits

- **Separation of responsibilities** — each layer solves a specific problem.
- **Replaceability** — one layer can often change without changing the application.
- **Troubleshooting** — failures can be isolated to a layer.

For example:

```text
DNS?
 ↓
TCP connection?
 ↓
TLS?
 ↓
HTTP?
 ↓
Application?
```

---

# 2. OSI Model

The **OSI (Open Systems Interconnection)** model is a conceptual model that divides networking into seven layers.

```text
7. Application
6. Presentation
5. Session
4. Transport
3. Network
2. Data Link
1. Physical
```

Mnemonic:

```text
All
People
Seem
To
Need
Data
Processing
```

The OSI model is mainly useful as a **conceptual framework**. Real Internet networking generally follows TCP/IP rather than implementing seven separate OSI layers.

---

# 3. OSI Layer 7 — Application

The Application layer contains network-aware application protocols.

Examples:

- HTTP
- DNS
- SMTP
- FTP
- SSH

Example:

```text
GET /users HTTP/1.1
Host: example.com
```

The browser uses HTTP rather than directly manipulating Ethernet frames.

> The OSI Application layer does not mean the entire application such as Chrome or a backend service. It refers to the networking protocols used by applications.

---

# 4. OSI Layer 6 — Presentation

The Presentation layer conceptually handles how data is represented.

Responsibilities can include:

- Data format translation
- Character encoding
- Compression
- Encryption

Examples:

```text
JSON
UTF-8
Compression
Encryption
```

Modern Internet stacks do not normally have a separate Presentation layer. These responsibilities may be handled by application protocols, libraries, or TLS.

---

# 5. OSI Layer 5 — Session

The Session layer conceptually manages communication sessions.

Responsibilities can include:

- Establishing sessions
- Maintaining sessions
- Synchronization
- Terminating sessions

Modern TCP/IP networking does not normally have a separate Session layer. Session-related functionality can be implemented by application protocols, TLS, TCP, or frameworks.

---

# 6. OSI Layer 4 — Transport

The Transport layer provides communication between application endpoints.

Main protocols:

```text
TCP
UDP
```

Responsibilities can include:

- End-to-end communication
- Port numbers
- Segmentation
- Reassembly
- Reliability
- Ordering
- Flow control
- Congestion control

TCP provides reliable, ordered byte-stream delivery with retransmission, flow control, and congestion control.

UDP provides connectionless datagrams without built-in guarantees of delivery or ordering.

---

# 7. OSI Layer 3 — Network

The Network layer handles communication between different networks.

The most important protocol is:

```text
IP
```

Responsibilities:

- Logical addressing
- Routing
- Packet forwarding
- Inter-network communication

Example:

```text
Network A
10.0.1.0/24
     |
   Router
     |
Network B
10.0.2.0/24
```

The router uses IP addressing and routing information to determine where packets should go.

---

# 8. OSI Layer 2 — Data Link

The Data Link layer handles communication over a local link/network.

Examples:

- Ethernet
- Wi-Fi

Important concepts:

- MAC addresses
- Frames
- Local delivery
- Error detection
- Switching

A switch primarily operates at Layer 2.

```text
PC A
  |
  | Ethernet frame
  ↓
Switch
  |
  +------> PC B
```

---

# 9. OSI Layer 1 — Physical

The Physical layer is responsible for transmitting raw bits over a physical medium.

Examples:

- Electrical signals over copper
- Light through fiber
- Radio waves through Wi-Fi

Conceptually:

```text
0 1 0 1 1 0 0 1 ...
```

Physical media includes:

- Copper
- Fiber
- Wireless radio

---

# 10. OSI Model Summary

| Layer | Name | Main Responsibility | Examples |
|---:|---|---|---|
| 7 | Application | Network services/protocols | HTTP, DNS, SMTP |
| 6 | Presentation | Data representation | Encoding, compression, encryption |
| 5 | Session | Session management | Session-related application functionality |
| 4 | Transport | End-to-end transport | TCP, UDP |
| 3 | Network | Routing and logical addressing | IP |
| 2 | Data Link | Local/link communication | Ethernet, Wi-Fi |
| 1 | Physical | Transmission of bits | Copper, fiber, radio |

---

# 11. TCP/IP Model

The TCP/IP model is the practical model associated with Internet networking.

A common four-layer representation is:

```text
Application
Transport
Internet
Network Access
```

Approximate mapping:

```text
OSI                         TCP/IP

Application ───────┐
Presentation ──────┼────── Application
Session ───────────┘

Transport ───────────────── Transport

Network ─────────────────── Internet

Data Link ─────────┐
Physical ──────────┴─────── Network Access
```

Some sources use a five-layer model:

```text
Application
Transport
Network
Data Link
Physical
```

The responsibilities matter more than the exact naming.

---

# 12. TCP/IP Application Layer

Contains protocols used directly by applications.

Examples:

```text
HTTP
DNS
SMTP
SSH
FTP
```

---

# 13. TCP/IP Transport Layer

Main protocols:

```text
TCP
UDP
```

The transport layer allows multiple applications to communicate using ports.

Example:

```text
Server
 ├── HTTPS :443
 ├── SSH   :22
 └── DNS   :53
```

---

# 14. TCP/IP Internet Layer

The Internet layer primarily deals with IP.

Responsibilities:

- IP addressing
- Packet forwarding
- Routing between networks

Example:

```text
Client
192.168.1.10
      |
    Router
      |
Server
142.250.x.x
```

---

# 15. TCP/IP Network Access Layer

This covers local-link communication.

Examples:

```text
Ethernet
Wi-Fi
```

It deals with:

- Frames
- MAC addresses
- Local delivery
- Physical transmission

---

# 16. OSI vs TCP/IP

OSI has seven conceptual layers:

```text
Application
Presentation
Session
Transport
Network
Data Link
Physical
```

A common TCP/IP representation has four:

```text
Application
Transport
Internet
Network Access
```

The TCP/IP Application layer combines the functionality conceptually represented by:

```text
OSI Application
OSI Presentation
OSI Session
```

The TCP/IP Network Access layer commonly combines:

```text
OSI Data Link
OSI Physical
```

---

# 17. Protocol Data Units (PDU)

Different layers use different names for the data they handle.

```text
Application
    ↓
Data

Transport
    ↓
TCP Segment / UDP Datagram

Network
    ↓
IP Packet

Data Link
    ↓
Frame

Physical
    ↓
Bits
```

A simplified flow:

```text
Application Data
       ↓
TCP Segment
       ↓
IP Packet
       ↓
Ethernet Frame
       ↓
Bits
```

---

# 18. Encapsulation

When data moves **down** the networking stack, each layer adds information needed by that layer.

Suppose the application wants to send:

```text
Hello
```

### Application

```text
Hello
```

### Transport

TCP adds a header:

```text
+-------------+---------+
| TCP Header  |  Hello  |
+-------------+---------+
```

This becomes a TCP segment.

### Network

IP adds a header:

```text
+-----------+-------------+---------+
| IP Header | TCP Header  |  Hello  |
+-----------+-------------+---------+
```

This becomes an IP packet.

### Data Link

Ethernet adds a header and trailer:

```text
+-------------+-----------+-------------+---------+-------------+
| ETH Header  | IP Header | TCP Header  |  Hello  | ETH Trailer|
+-------------+-----------+-------------+---------+-------------+
```

This becomes an Ethernet frame.

### Physical

The frame is transmitted as physical signals/bits.

```text
101101001011010...
```

This process is **encapsulation**.

---

# 19. Decapsulation

The receiver performs the reverse process:

```text
Bits
 ↓
Ethernet Frame
 ↓
IP Packet
 ↓
TCP Segment
 ↓
Application Data
```

Conceptually:

```text
Ethernet
    ↓
IP
    ↓
TCP
    ↓
Application
```

This process is **decapsulation**.

---

# 20. Layer 2 Frame vs Layer 3 Packet

Suppose traffic crosses multiple networks:

```text
Host A
   |
Network A
   |
Router 1
   |
Network B
   |
Router 2
   |
Network C
   |
Host B
```

The Layer 2 frame is associated with the **current link**.

Conceptually:

```text
Host A
   |
   | Ethernet Frame #1
   ↓
Router 1
   |
   | Ethernet Frame #2
   ↓
Router 2
   |
   | Ethernet Frame #3
   ↓
Host B
```

The link-layer frame can change at every hop.

The IP packet is the network-layer object being routed across networks.

This distinction is fundamental to understanding routing.

---

# 21. Which Devices Operate at Which Layer?

This is a useful starting model, although modern devices can operate at multiple layers.

### Hub

Primarily:

```text
Layer 1 — Physical
```

A hub repeats signals.

### Switch

Primarily:

```text
Layer 2 — Data Link
```

A switch forwards frames using MAC addresses.

Modern switches can also perform Layer 3 routing.

### Router

Primarily:

```text
Layer 3 — Network
```

A router forwards IP packets between networks.

### Load Balancer

Can operate at multiple layers:

```text
L4 → TCP/UDP
L7 → HTTP/application
```

### Reverse Proxy

Typically operates at Layer 7, although some proxies operate at lower layers.

---

# 22. Layer 2 vs Layer 3

## Layer 2

Uses:

```text
MAC address
```

Concerned primarily with:

```text
Local network/link
```

Example:

```text
PC → Switch → PC
```

## Layer 3

Uses:

```text
IP address
```

Concerned with:

```text
Communication between networks
```

Example:

```text
Network A
   |
 Router
   |
Network B
```

Useful mental model:

```text
MAC → local delivery
IP  → network-to-network delivery
```

---

# 23. Layer 3 vs Layer 4

### Layer 3 — IP

Answers:

> Which host/network should this packet go toward?

### Layer 4 — TCP/UDP

Answers:

> Which transport endpoint should receive this traffic?

Example:

```text
192.168.1.20:53124
        ↓
142.250.195.14:443
```

Here:

```text
Source IP       = 192.168.1.20
Source Port     = 53124

Destination IP  = 142.250.195.14
Destination Port= 443
```

---

# 24. Layer 4 vs Layer 7 Load Balancing

### L4 Load Balancer

Primarily works with:

```text
IP
Port
TCP
UDP
```

Example:

```text
Client
  |
  | TCP :443
  ↓
L4 Load Balancer
  |
  ├── Server A
  └── Server B
```

### L7 Load Balancer

Understands application-level protocols such as HTTP.

It can route based on:

- URL path
- Host
- HTTP headers
- Cookies
- HTTP method

Example:

```text
/users
   ↓
User Service

/orders
   ↓
Order Service
```

---

# 25. Real HTTPS Request — Layer by Layer

Suppose a browser requests:

```text
https://example.com/users
```

A simplified stack is:

```text
HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP
 ↓
Ethernet/Wi-Fi
```

### Application

```text
GET /users
Host: example.com
```

### TLS

The HTTP data is protected by TLS.

```text
Encrypted application data
```

### TCP

TCP provides the transport.

```text
TCP Segment
```

### IP

IP provides network addressing.

```text
IP Packet
```

### Ethernet/Wi-Fi

The packet is carried over the local link.

```text
Frame
```

### Physical

The frame is ultimately transmitted as physical signals.

---

# 26. TLS Does Not Map Perfectly to One OSI Layer

TLS is often shown between the application and transport layers:

```text
HTTP
 ↓
TLS
 ↓
TCP
```

But TLS does not map perfectly to one OSI layer.

It provides security for application communication and is implemented through protocols/libraries that sit between the application protocol and transport.

For HTTPS:

```text
HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP
 ↓
Ethernet
```

---

# 27. HTTP/3 Changes the Stack

HTTP/3 uses QUIC instead of TCP.

A simplified stack is:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
   ↓
Ethernet/Wi-Fi
```

This is an important reminder:

> Networking layers describe responsibilities and protocol relationships; they are not a rigid requirement that every application must use exactly one specific protocol at every layer.

---

# 28. TCP/IP Five-Layer Model

A very useful practical model is:

```text
5. Application
4. Transport
3. Network
2. Data Link
1. Physical
```

Examples:

| Layer | Examples |
|---|---|
| Application | HTTP, DNS, SSH |
| Transport | TCP, UDP |
| Network | IP, ICMP |
| Data Link | Ethernet, Wi-Fi |
| Physical | Copper, Fiber, Radio |

For practical packet reasoning, this five-layer model is often easier than the seven-layer OSI model.

---

# 29. Three Models You May Encounter

### OSI — 7 layers

```text
7 Application
6 Presentation
5 Session
4 Transport
3 Network
2 Data Link
1 Physical
```

### TCP/IP — 4 layers

```text
Application
Transport
Internet
Network Access
```

### Five-layer Internet model

```text
Application
Transport
Network
Data Link
Physical
```

Do not focus on memorizing the names.

Focus on understanding:

```text
What responsibility does each layer provide?
```

---

# 30. Practical Command — `ipconfig`

On Windows:

```powershell
ipconfig
```

Example:

```text
Windows IP Configuration

Ethernet adapter Ethernet:

   IPv4 Address. . . . . . . : 192.168.1.20
   Subnet Mask . . . . . . . : 255.255.255.0
   Default Gateway . . . . . : 192.168.1.1
```

Meaning:

```text
IPv4 Address
192.168.1.20
```

This is the host's IPv4 address on that network.

```text
Subnet Mask
255.255.255.0
```

This corresponds to:

```text
/24
```

The host is therefore on a network such as:

```text
192.168.1.0/24
```

```text
Default Gateway
192.168.1.1
```

Traffic destined for other networks is normally sent toward this gateway.

---

# 31. Practical Command — `ping`

Run:

```powershell
ping google.com
```

Example:

```text
Pinging google.com [142.250.195.14] with 32 bytes of data:

Reply from 142.250.195.14: bytes=32 time=22ms TTL=117
Reply from 142.250.195.14: bytes=32 time=21ms TTL=117
Reply from 142.250.195.14: bytes=32 time=23ms TTL=117
Reply from 142.250.195.14: bytes=32 time=22ms TTL=117

Ping statistics for 142.250.195.14:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

Meaning:

```text
google.com
    ↓
DNS resolved the hostname
    ↓
142.250.195.14
```

Then ICMP echo requests were sent.

```text
time=22ms
```

Approximately represents the round-trip time for that request.

```text
TTL=117
```

TTL is an IP-layer field that limits how long a packet can remain in the network. Routers decrement it as the packet crosses hops.

```text
Lost = 0
```

No packet loss was observed in this test.

Important:

> `ping` tests ICMP connectivity/reachability. A failed ping does not automatically mean TCP/HTTPS connectivity is broken because ICMP may be filtered.

---

# 32. Practical Command — `tracert`

On Windows:

```powershell
tracert google.com
```

Example:

```text
Tracing route to google.com [142.250.195.14]

  1     2 ms     1 ms     2 ms  192.168.1.1
  2    10 ms     9 ms    11 ms  10.20.0.1
  3    18 ms    17 ms    19 ms  100.64.0.1
  4    25 ms    24 ms    26 ms  203.0.113.10
  5    30 ms    29 ms    31 ms  142.250.195.14
```

Meaning:

```text
Your computer
     ↓
192.168.1.1
     ↓
10.20.0.1
     ↓
100.64.0.1
     ↓
203.0.113.10
     ↓
Destination
```

Each line represents a hop that responded to the trace.

The exact output varies based on routing and network configuration.

You may see:

```text
* * *
```

A missing response does not necessarily mean the path is broken. A router may simply not respond to the probe.

---

# 33. Practical Command — `nslookup`

Run:

```powershell
nslookup google.com
```

Example:

```text
Server:  router.local
Address:  192.168.1.1

Non-authoritative answer:
Name:    google.com
Addresses: 142.250.195.14
```

Meaning:

```text
DNS Server:
192.168.1.1
```

Your machine asked this DNS server to resolve the hostname.

```text
google.com
     ↓
142.250.195.14
```

This demonstrates DNS at the application layer.

---

# 34. Practical Command — `curl -v`

Run:

```powershell
curl.exe -v https://example.com
```

Example output will look approximately like:

```text
* Host example.com:443 was resolved.
* IPv4: 93.184.216.34
* Trying 93.184.216.34:443...
* Connected to example.com (93.184.216.34) port 443
* TLS 1.3 connection established
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: text/html
< Content-Length: 1256
```

This exposes several layers:

```text
Host resolved
     ↓
DNS

Connected to ... port 443
     ↓
TCP

TLS 1.3 connection established
     ↓
TLS

GET / HTTP/1.1
     ↓
HTTP request

HTTP/1.1 200 OK
     ↓
HTTP response
```

This is one of the best practical demonstrations of the layered architecture.

---

# 35. Practical Command — `arp`

On Windows:

```powershell
arp -a
```

Example:

```text
Interface: 192.168.1.20

Internet Address      Physical Address      Type
192.168.1.1           34-97-f6-aa-bb-cc     dynamic
192.168.1.5           08-00-27-11-22-33     dynamic
```

Meaning:

```text
192.168.1.1
      ↓
34-97-f6-aa-bb-cc
```

The ARP cache maps a local IPv4 address to a MAC address.

This demonstrates the relationship between:

```text
Layer 3 → IP
Layer 2 → MAC
```

---

# 36. Practical Command — `netstat`

On Windows:

```powershell
netstat -ano
```

Example:

```text
Proto  Local Address       Foreign Address       State
TCP    192.168.1.20:53124  142.250.195.14:443   ESTABLISHED
TCP    0.0.0.0:8080        0.0.0.0:0            LISTENING
```

Meaning:

```text
192.168.1.20:53124
        ↓
Local endpoint

142.250.195.14:443
        ↓
Remote endpoint
```

The connection is:

```text
TCP
192.168.1.20:53124
        ↓
142.250.195.14:443
```

The second line means a local application is listening on port `8080` on the available IPv4 interfaces.

This connects:

```text
Transport
   ↓
TCP
   ↓
Port
   ↓
Socket
```

---

# 37. Reading a Network Problem by Layers

Suppose an application cannot access:

```text
https://api.example.com
```

A layered approach is:

```text
Application
    ↓
Is the HTTP request valid?

TLS
    ↓
Can the TLS handshake complete?

Transport
    ↓
Can TCP connect to port 443?

Network
    ↓
Can IP packets reach the destination?

Data Link
    ↓
Can the host reach its local gateway?

Physical
    ↓
Is the network interface/link working?
```

This is much more useful than simply saying:

> "The network is down."

---

# 38. Complete Mental Model

```text
┌──────────────────────────────┐
│ Application                  │
│ HTTP / DNS / SSH             │
├──────────────────────────────┤
│ Transport                    │
│ TCP / UDP                    │
├──────────────────────────────┤
│ Network / Internet           │
│ IP / ICMP                    │
├──────────────────────────────┤
│ Data Link                    │
│ Ethernet / Wi-Fi             │
├──────────────────────────────┤
│ Physical                     │
│ Copper / Fiber / Radio       │
└──────────────────────────────┘
```

Sending:

```text
Application Data
       ↓
Transport Segment
       ↓
IP Packet
       ↓
Data-Link Frame
       ↓
Physical Signals
```

Receiving:

```text
Physical Signals
       ↓
Data-Link Frame
       ↓
IP Packet
       ↓
Transport Segment
       ↓
Application Data
```

---

# 39. What Each Layer Answers

A useful way to remember the layers is by the question each one answers.

```text
Application
→ What does the application want to communicate?

Transport
→ Which application endpoint should receive it?
→ Should delivery be reliable?

Network
→ Which host/network should the packet reach?
→ Which route should it take?

Data Link
→ How do I deliver this over the current local link?

Physical
→ How do I transmit the actual bits?
```

---

# 40. HTTPS Stack in One View

For a typical HTTPS request:

```text
                  Application
                       │
                      HTTP
                       │
                      TLS
                       │
                    Transport
                       │
                      TCP
                       │
                    Network
                       │
                       IP
                       │
                   Data Link
                       │
                  Ethernet/Wi-Fi
                       │
                   Physical
                       │
                Signals / Bits
```

For HTTP/3:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
   ↓
Ethernet/Wi-Fi
```

---

# 41. Key Takeaways

The most important concepts from this phase are:

```text
OSI
 ↓
7-layer conceptual model

TCP/IP
 ↓
Practical Internet architecture

Layering
 ↓
Separation of networking responsibilities

Encapsulation
 ↓
Data gains protocol information while moving down the stack

Decapsulation
 ↓
Data is processed while moving up the stack

Application
 ↓
HTTP, DNS, SSH

Transport
 ↓
TCP, UDP

Network
 ↓
IP, ICMP

Data Link
 ↓
Ethernet, Wi-Fi

Physical
 ↓
Signals / bits
```

The core mental model is:

```text
Application
    ↓
Transport
    ↓
Network
    ↓
Data Link
    ↓
Physical
```

and a real HTTPS communication can be visualized as:

```text
HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP
 ↓
Ethernet / Wi-Fi
 ↓
Physical network
```

The exact protocol stack can vary, but the key idea remains the same:

> **Each layer provides a specific networking capability and uses the services provided by lower layers.**

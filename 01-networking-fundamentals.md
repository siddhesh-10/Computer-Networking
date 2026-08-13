# Networking — Phase 1: Networking Fundamentals

> **Goal:** Build an intuitive understanding of what a computer network is and what happens when one computer communicates with another.
>
> **Target:** Senior Software Engineer / Backend / Distributed Systems interviews
>
> **Prerequisite:** None

---

## 1. What is a Computer Network?

A **computer network** is a collection of devices that can communicate with each other and exchange data.

Examples of networked devices:

- Laptop
- Mobile phone
- Server
- Router
- Switch
- Printer
- IoT device

At the simplest level:

```text
Computer A  ───────────────>  Computer B
             Data
```

The network provides the mechanisms needed to:

1. Identify the destination.
2. Decide where data should go.
3. Transport data.
4. Detect or recover from failures where required.
5. Allow different applications to communicate.

---

## 2. Why Do We Need Networking?

Without networking, computers would operate independently.

Networking allows us to:

- Access websites.
- Call APIs.
- Communicate between microservices.
- Access databases remotely.
- Transfer files.
- Stream video.
- Send emails.
- Connect cloud services.
- Build distributed systems.

For example:

```text
Browser
   |
   | HTTP request
   v
Web Server
   |
   | Database request
   v
Database
```

A modern application is usually a collection of communicating systems.

---

## 3. Networked Application vs Network

It is useful to separate two ideas.

### Network

The infrastructure that allows communication:

```text
NIC
Switch
Router
Internet
Cables
Wi-Fi
```

### Networked Application

The software that uses that infrastructure:

```text
Browser
Web Server
Database
Microservice
Redis
Kafka
```

For example:

```text
Browser
   |
   | HTTP
   v
Internet
   |
   v
Web Server
```

HTTP is an application protocol, while the Internet provides the underlying communication infrastructure.

---

# 4. Basic Terminology

## 4.1 Host

A **host** is a device connected to a network that can send or receive network traffic.

Examples:

- Laptop
- Server
- VM
- Container
- Phone

```text
Host A                Host B
Laptop                Server
```

---

## 4.2 Client

A **client** is a program that initiates communication with another system.

Examples:

- Browser
- Mobile app
- API client

```text
Client  ───── request ─────>  Server
```

---

## 4.3 Server

A **server** is a program that listens for requests and provides a service.

Examples:

- HTTP server
- Database server
- DNS server
- SSH server

Important:

> A server is primarily a role performed by software, not necessarily a special type of physical machine.

A normal computer can act as both client and server.

---

## 4.4 Network Interface / NIC

A **Network Interface Card (NIC)** is the hardware/software interface that allows a device to communicate over a network.

Modern computers may have:

- Ethernet interface
- Wi-Fi interface
- Virtual network interfaces

Example:

```text
Application
     |
Operating System
     |
Network Interface
     |
Wi-Fi / Ethernet
     |
Network
```

We will study the OS ↔ NIC interaction in much more detail later.

---

## 4.5 Packet

A **packet** is a unit of data transmitted over a network.

Conceptually:

```text
+-------------------+----------------------+
| Header            | Payload              |
+-------------------+----------------------+
```

The header contains control information such as:

- Source
- Destination
- Protocol information
- Other metadata

The exact structure depends on the protocol.

---

## 4.6 Frame

A **frame** is the unit of data used at the data-link layer.

For Ethernet:

```text
+----------------+----------------+
| Ethernet Header| Payload        |
+----------------+----------------+
```

Don't worry about the distinction between packet and frame yet. We will make it precise when we study the OSI/TCP-IP layers.

---

## 4.7 Segment

A **TCP segment** is the transport-layer representation of data carried by TCP.

Conceptually:

```text
TCP Header + Application Data
```

Similarly, UDP uses the term **datagram**.

So you may encounter:

```text
Application data
      ↓
TCP segment
      ↓
IP packet
      ↓
Ethernet frame
```

This layering is one of the most important ideas in networking.

---

# 5. Addressing

A network needs ways to identify communicating endpoints.

There are multiple kinds of addresses.

### MAC address

Used primarily for communication on the local network.

Example:

```text
3C:52:82:AB:12:34
```

### IP address

Used for network-level addressing.

IPv4 example:

```text
192.168.1.10
```

### Port

Identifies an application/service endpoint on a host.

Example:

```text
192.168.1.10:443
```

Here:

```text
192.168.1.10 → IP address
443          → port
```

A useful mental model:

```text
MAC  → Which network interface on the local network?
IP   → Which host/network?
Port → Which application/service?
```

This is simplified; we will refine it later.

---

# 6. What is a Port?

A single machine can run many network applications simultaneously.

For example:

```text
Server
 ├── Web server      :443
 ├── SSH             :22
 ├── PostgreSQL      :5432
 └── Application     :8080
```

The **port** helps the operating system deliver incoming traffic to the correct application/socket.

Ports range from:

```text
0 → 65535
```

Important categories:

- `0–1023`: well-known ports
- `1024–49151`: registered ports
- `49152–65535`: commonly used as dynamic/ephemeral ports

Examples:

| Protocol/Service | Typical Port |
|---|---:|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| DNS | 53 |
| PostgreSQL | 5432 |

These are conventions, not hard requirements.

For example, an HTTP server can technically listen on port `8080`.

---

# 7. Socket

A **socket** is an operating-system abstraction used by applications to communicate over a network.

A network connection is commonly identified using:

```text
Protocol + Source IP + Source Port + Destination IP + Destination Port
```

For example:

```text
TCP
192.168.1.20:53124
        ↓
142.250.195.14:443
```

The application normally does not manipulate Ethernet frames directly.

Instead:

```text
Application
     |
     | socket API
     v
Operating System
     |
     v
TCP / UDP
     |
     v
IP
     |
     v
Network Interface
```

We will study sockets deeply in the transport/networking phase.

---

# 8. Bandwidth

**Bandwidth** is the maximum amount of data that a network link can theoretically carry per unit of time.

Usually expressed as:

```text
Mbps
Gbps
```

Example:

```text
1 Gbps
```

means a link can theoretically transfer up to roughly 1 gigabit per second under the relevant conditions.

Bandwidth is not the same as latency.

---

# 9. Latency

**Latency** is the time required for data to travel from one point to another or for an operation to complete, depending on the context.

A common networking measurement is:

> Round-trip time (RTT)

Example:

```text
Client ───── request ─────> Server
Client <──── response ───── Server

RTT = time for the round trip
```

If RTT is 50 ms, the round trip takes approximately 50 milliseconds.

---

# 10. Throughput

**Throughput** is the actual amount of data successfully transferred per unit of time.

Example:

```text
Bandwidth = 1 Gbps
Actual throughput = 700 Mbps
```

The theoretical capacity and actual achieved throughput can differ.

Factors include:

- Network congestion
- Protocol overhead
- Packet loss
- Server performance
- Client performance
- TCP behavior
- Hardware limitations

---

# 11. Packet Loss

A packet may fail to reach its destination.

```text
Client
  |
  | Packet 1 ───────────────> Server
  |
  | Packet 2 ───X
  |
  | Packet 3 ───────────────> Server
```

This is **packet loss**.

Depending on the protocol:

- TCP can detect loss and retransmit.
- UDP itself does not guarantee retransmission.

Packet loss can cause:

- Increased latency
- Reduced throughput
- Retransmissions
- Application failures

---

# 12. Protocol

A **protocol** is a set of rules that defines how systems communicate.

Examples:

| Protocol | Purpose |
|---|---|
| HTTP | Web communication |
| DNS | Domain name resolution |
| TCP | Reliable transport |
| UDP | Datagram transport |
| IP | Network addressing/routing |
| Ethernet | Local network communication |
| TLS | Secure communication |

A protocol defines things such as:

- Message format
- Meaning of fields
- Communication sequence
- Error handling
- Expected behavior

---

# 13. Why Do We Need Layers?

Imagine an application wants to send:

```text
"Hello"
```

It should not need to know:

- How Ethernet works
- How routers forward packets
- How Wi-Fi transmits bits
- How TCP retransmits packets

Instead, networking is divided into layers.

Conceptually:

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

Each layer provides services to the layer above it.

This is called **layering**.

We will study the OSI and TCP/IP models in Phase 2.

---

# 14. Encapsulation — The Most Important Mental Model

Suppose an application wants to send:

```text
Hello
```

The application creates data.

The transport layer adds its header:

```text
[TCP Header][Hello]
```

The network layer adds an IP header:

```text
[IP Header][TCP Header][Hello]
```

The data-link layer adds its own header/trailer:

```text
[Ethernet Header][IP Header][TCP Header][Hello][Ethernet Trailer]
```

Conceptually:

```text
Application data
       ↓
TCP segment
       ↓
IP packet
       ↓
Ethernet frame
       ↓
Bits/signals
```

This process is called **encapsulation**.

At the receiver, the reverse process occurs:

```text
Bits
 ↓
Ethernet frame
 ↓
IP packet
 ↓
TCP segment
 ↓
Application data
```

This is called **decapsulation**.

---

# 15. Local Network vs Internet

A useful distinction:

### Local network

Example:

```text
Laptop
   |
Wi-Fi Router
   |
Printer
```

The devices may communicate within the same local network.

### Internet

The Internet connects many independent networks.

Conceptually:

```text
Home Network
      |
      v
    ISP
      |
      v
Internet
      |
      v
Cloud / Data Center
      |
      v
Server
```

The Internet is therefore not one giant physical network.

It is a **network of interconnected networks**.

---

# 16. Router vs Switch — Initial Mental Model

Don't memorize the details yet.

### Switch

Primarily connects devices within a local network.

```text
PC ──┐
PC ──┼── Switch
PC ──┘
```

### Router

Connects different networks.

```text
Network A
    |
  Router
    |
Network B
```

A router decides where packets should be forwarded based on routing information.

We will study both in depth later.

---

# 17. What Happens When Two Computers Communicate?

At a very high level:

```text
Computer A
    |
    | Application creates data
    ↓
Transport layer
    |
    | TCP/UDP
    ↓
Network layer
    |
    | IP
    ↓
Data Link
    |
    | Ethernet/Wi-Fi
    ↓
Physical network
    |
    ↓
Routers / Switches
    |
    ↓
Computer B
    |
    ↓
Operating System
    |
    ↓
Application
```

This is the foundation for almost everything we will study.

---

# 18. Example: Browser → Web Server

Suppose you open:

```text
https://example.com
```

At a high level:

```text
Browser
   |
   | Need IP address
   ↓
DNS
   |
   | IP address
   ↓
Browser
   |
   | Establish connection
   ↓
TCP / QUIC
   |
   | Secure connection
   ↓
TLS
   |
   | HTTP request
   ↓
Internet
   |
   ↓
Web Server
   |
   | HTTP response
   ↓
Browser
```

This single example contains many networking concepts:

- DNS
- IP
- Routing
- TCP/QUIC
- TLS
- HTTP
- Ports
- Sockets
- Packets
- Latency

We will unpack every part of this flow throughout the course.

---

# 19. Latency Breakdown

When a request takes time, "network latency" is not necessarily one thing.

A simplified request can involve:

```text
DNS lookup
   +
Connection establishment
   +
TLS handshake
   +
Request transmission
   +
Server processing
   +
Response transmission
```

For example:

```text
Total response time
=
DNS
+ TCP
+ TLS
+ Request
+ Server processing
+ Response
```

This becomes extremely important when debugging production APIs.

---

# 20. Important Performance Terms

### Latency

How long an operation takes.

### Bandwidth

Maximum capacity of a link.

### Throughput

Actual amount of data transferred per unit time.

### RTT

Round-trip time between endpoints.

### Jitter

Variation in packet delay.

Example:

```text
Packet 1 → 20 ms
Packet 2 → 21 ms
Packet 3 → 80 ms
Packet 4 → 22 ms
```

The delay is varying significantly.

### Packet Loss

Packets that fail to reach their destination.

---

# 21. A Useful Senior-Level Mental Model

When investigating a network problem, don't think:

> "The network is slow."

Break it down:

```text
DNS?
 ↓
Routing?
 ↓
TCP connection?
 ↓
TLS?
 ↓
HTTP?
 ↓
Load balancer?
 ↓
Server?
 ↓
Database?
```

Then ask:

```text
Latency?
Packet loss?
Bandwidth?
Congestion?
Connection exhaustion?
CPU?
Memory?
Queueing?
Timeout?
```

This mindset will become important in system design and production debugging.

---

# 22. Common Misconceptions

## "IP address identifies a process"

Not exactly.

An IP primarily identifies a network-layer endpoint/interface.

The **port** helps identify the service/application endpoint.

---

## "MAC address is used across the entire Internet"

No.

MAC addresses are primarily relevant to local link communication.

Routers operate at the network layer and forward traffic between networks.

---

## "Bandwidth = speed"

Not exactly.

Bandwidth describes capacity.

Latency describes delay.

A high-bandwidth connection can still have high latency.

---

## "TCP guarantees that the request will succeed"

No.

TCP provides reliable, ordered byte-stream delivery between endpoints.

It does not guarantee:

- The server application is running.
- The HTTP request succeeds.
- The database succeeds.
- The application returns a successful response.

---

## "The Internet is one network"

No.

The Internet is a network of interconnected networks.

---

# 23. Practical Commands

You should eventually be comfortable with these.

### Check IP configuration

Linux/macOS:

```bash
ip addr
```

or:

```bash
ifconfig
```

Windows:

```powershell
ipconfig
```

---

### Test reachability

```bash
ping example.com
```

This commonly uses ICMP and can help measure reachability and RTT.

---

### DNS lookup

```bash
nslookup example.com
```

or:

```bash
dig example.com
```

---

### Trace network path

Linux/macOS:

```bash
traceroute example.com
```

Windows:

```powershell
tracert example.com
```

---

### Inspect connections

Linux:

```bash
ss -tulnp
```

This can show listening sockets and network connections.

---

### HTTP debugging

```bash
curl -v https://example.com
```

The `-v` option is particularly useful for understanding:

- DNS
- TCP connection
- TLS
- HTTP request/response

We will use these commands more extensively later.

---

# 24. Key Takeaways

You should remember these concepts from Phase 1:

```text
Network
    ↓
Devices communicating with each other

Host
    ↓
Device participating in a network

IP
    ↓
Network-layer addressing

MAC
    ↓
Local-link addressing

Port
    ↓
Application/service endpoint

Socket
    ↓
OS abstraction used by applications for network communication

Packet
    ↓
Network-layer unit of data

Frame
    ↓
Data-link-layer unit of data

Segment
    ↓
TCP transport-layer unit

Protocol
    ↓
Rules for communication

Bandwidth
    ↓
Capacity

Throughput
    ↓
Actual transfer rate

Latency
    ↓
Delay

RTT
    ↓
Round-trip delay

Packet loss
    ↓
Packets that don't reach the destination

Encapsulation
    ↓
Headers added while moving down the networking stack

Decapsulation
    ↓
Headers processed/removed while moving up the stack
```

---

# 25. Interview Questions

## Beginner

1. What is a computer network?
2. What is a host?
3. What is an IP address?
4. What is a MAC address?
5. What is a port?
6. What is a socket?
7. What is a packet?
8. What is a frame?
9. What is a protocol?
10. What is the difference between bandwidth and latency?

## Intermediate

11. What happens when two computers communicate?
12. Why do we need IP addresses if we have MAC addresses?
13. Why do we need ports?
14. What is the difference between a switch and a router?
15. What is packet loss?
16. What is RTT?
17. What is throughput?
18. What is encapsulation?
19. What is decapsulation?
20. Why are networking protocols layered?

## Senior-level thinking

21. An API takes 2 seconds. How would you determine whether the problem is network latency or server processing?
22. Why can a high-bandwidth connection still have poor performance?
23. What happens when packets are lost?
24. What happens if a server has thousands of concurrent connections?
25. How does a request travel from a browser to a server?
26. Where can latency be introduced between a client and server?
27. Why does a distributed system need to care about network failures?
28. What is the difference between a network failure and an application failure?
29. Why can't we assume that a successful TCP connection means an HTTP request will succeed?
30. How would you troubleshoot intermittent API connectivity?

---

# 26. Connection to Later Topics

Phase 1 is the foundation.

```text
Phase 1
Networking Fundamentals
        |
        +----> Phase 2: OSI / TCP-IP
        |
        +----> TCP / UDP
        |
        +----> DNS
        |
        +----> HTTP
        |
        +----> TLS
        |
        +----> Load Balancing
        |
        +----> CDN
        |
        +----> Distributed Systems
        |
        +----> System Design
```

The most important thing is **not memorizing this document**.

The goal is to build a mental model that lets you reason about what happens when data moves between machines.

---

# 27. Next Phase

Next:

> **Phase 2 — OSI Model & TCP/IP Model**

We will answer:

- Why do networking layers exist?
- What exactly happens at each layer?
- OSI vs TCP/IP
- Encapsulation/decapsulation in detail
- Ethernet frame
- IP packet
- TCP segment
- HTTP message
- Which device operates at which layer?
- Where do switches, routers, load balancers and proxies fit?
- How the layers interact during a real HTTP request.

---

## Recommended Practice

Before moving to Phase 2, try these commands on your own machine:

```bash
ipconfig
ping google.com
nslookup google.com
tracert google.com
curl -v https://google.com
```

For each command, don't just run it.

Ask:

> **Which networking concept am I observing?**

That habit will make the later topics much easier.

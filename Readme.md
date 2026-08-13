# Computer Networking — Zero to Senior Software Engineer

A structured, practical journey to learn **computer networking from fundamentals to advanced production networking**, with a focus on:

- Backend engineering
- Distributed systems
- System design
- AWS networking
- CDN / Edge networking
- Senior Software Engineer interviews

The goal is not to memorize networking definitions. The goal is to build a mental model that lets me reason about **how data actually moves through a network**, troubleshoot production issues, and make better system-design decisions.

---

## 🎯 Learning Goal

By completing this repository, I should be able to confidently explain and reason about:

- What happens when a client calls an API
- How packets travel between machines
- How TCP establishes and maintains connections
- How DNS resolves domains
- How HTTP and HTTPS work internally
- How routers and switches forward traffic
- How load balancers work
- How CDNs route and cache traffic
- How BGP and Anycast enable global networking
- How AWS networking is implemented
- How network failures affect distributed systems
- How to troubleshoot production networking problems
- How networking concepts influence system design

---

# 📚 Curriculum

## Part 1 — Networking Foundations

### 01. Networking Fundamentals

Basic concepts required to understand everything else.

Topics:

- What is a network?
- Host
- Client / Server
- NIC
- Packet
- Frame
- Segment
- Protocol
- IP address
- MAC address
- Port
- Socket
- Bandwidth
- Latency
- Throughput
- RTT
- Packet loss
- Encapsulation
- Decapsulation

📖 [Read Phase 01 →](01-networking-fundamentals.md)

---

### 02. OSI & TCP/IP Models

Understanding networking layers and how they interact.

Topics:

- OSI model
- TCP/IP model
- Layer responsibilities
- Protocols at each layer
- PDU
- Encapsulation
- Decapsulation
- OSI vs TCP/IP

📖 [Read Phase 02 →](02-osi-and-tcp-ip.md)

---

### 03. Ethernet & Local Networking

Understanding communication inside a local network.

Topics:

- Ethernet
- MAC addresses
- Ethernet frames
- Switches
- Hubs
- Bridges
- Broadcast
- Unicast
- Multicast
- VLANs

📖 [Read Phase 03 →](03-ethernet-and-local-networking.md)

---

### 04. ARP & Address Resolution

How devices discover each other on a local network.

Topics:

- ARP
- ARP request
- ARP response
- ARP cache
- IP → MAC resolution
- Local network delivery

📖 [Read Phase 04 →](04-arp-and-address-resolution.md)

---

### 05. IP Addressing & Subnetting

Understanding network addressing.

Topics:

- IPv4
- IPv6 basics
- Public IP
- Private IP
- Loopback
- CIDR
- Subnet masks
- Subnetting
- Default gateway
- Network address
- Broadcast address

📖 [Read Phase 05 →](05-ip-addressing-and-subnetting.md)

---

### 06. Routing & ICMP

Understanding how packets move between networks.

Topics:

- Routing
- Routing tables
- Router
- Next hop
- Default route
- TTL
- ICMP
- Ping
- Traceroute

📖 [Read Phase 06 →](06-routing-and-icmp.md)

---

## Part 2 — Transport Layer

### 07. UDP

Understanding connectionless transport.

Topics:

- Datagram
- Connectionless communication
- UDP headers
- Reliability limitations
- UDP use cases
- TCP vs UDP

📖 [Read Phase 07 →](07-udp.md)

---

### 08. TCP Fundamentals

Understanding reliable communication.

Topics:

- TCP
- Three-way handshake
- SYN
- SYN-ACK
- ACK
- Sequence numbers
- Acknowledgements
- Ordered delivery
- Reliable delivery

📖 [Read Phase 08 →](08-tcp-fundamentals.md)

---

### 09. TCP Deep Dive

Understanding how TCP handles reliability and congestion.

Topics:

- Sliding window
- Flow control
- Receive window
- Retransmission
- Timeout
- Congestion control
- Slow start
- Congestion avoidance
- Fast retransmit
- Fast recovery

📖 [Read Phase 09 →](09-tcp-deep-dive.md)

---

### 10. TCP Lifecycle & Performance

Understanding the TCP connection lifecycle.

Topics:

- FIN
- RST
- Connection termination
- TIME_WAIT
- Keep-alive
- Connection pooling
- TCP buffers
- Timeouts
- Connection exhaustion

📖 [Read Phase 10 →](10-tcp-lifecycle-and-performance.md)

---

### 11. Sockets & OS Networking

Connecting networking with operating systems.

Topics:

- Socket
- Socket API
- bind()
- listen()
- accept()
- connect()
- send()
- recv()
- Blocking sockets
- Non-blocking sockets
- Ephemeral ports
- Connection lifecycle

📖 [Read Phase 11 →](11-sockets-and-os-networking.md)

---

## Part 3 — Application Networking

### 12. DNS

Understanding how domain names become IP addresses.

Topics:

- DNS
- Resolver
- Recursive DNS
- Authoritative DNS
- Root servers
- TLD servers
- DNS records
- A
- AAAA
- CNAME
- NS
- TXT
- PTR
- DNS caching
- TTL
- DNS failures

📖 [Read Phase 12 →](12-dns.md)

---

### 13. HTTP/1.1

Understanding web communication.

Topics:

- HTTP request
- HTTP response
- Methods
- Status codes
- Headers
- Cookies
- Persistent connections
- Keep-alive
- Chunked encoding
- Content-Length
- HTTP caching

📖 [Read Phase 13 →](13-http-1-1.md)

---

### 14. HTTP/2 & HTTP/3

Modern HTTP protocols.

Topics:

- HTTP/2
- Binary framing
- Streams
- Multiplexing
- HPACK
- Head-of-line blocking
- HTTP/3
- QUIC
- UDP-based transport

📖 [Read Phase 14 →](14-http2-and-http3.md)

---

### 15. TLS & HTTPS

Understanding secure communication.

Topics:

- Encryption
- Symmetric encryption
- Asymmetric encryption
- Hashing
- Digital signatures
- Certificates
- Certificate authorities
- Certificate chain
- TLS handshake
- TLS 1.2
- TLS 1.3
- HTTPS
- SNI
- ALPN

📖 [Read Phase 15 →](15-tls-and-https.md)

---

## Part 4 — Infrastructure Networking

### 16. NAT, Firewalls & Security

Topics:

- NAT
- PAT
- SNAT
- DNAT
- Port forwarding
- Stateful firewall
- Stateless firewall
- Network filtering

📖 [Read Phase 16 →](16-nat-firewalls-and-security.md)

---

### 17. Proxies & Reverse Proxies

Topics:

- Forward proxy
- Reverse proxy
- L4 proxy
- L7 proxy
- TLS termination
- Proxy use cases
- API gateway basics

📖 [Read Phase 17 →](17-proxies-and-reverse-proxies.md)

---

### 18. Load Balancing

Topics:

- L4 load balancing
- L7 load balancing
- Round robin
- Weighted round robin
- Least connections
- Consistent hashing
- Health checks
- Sticky sessions
- Connection draining
- TLS termination

📖 [Read Phase 18 →](18-load-balancing.md)

---

### 19. WebSockets, gRPC & Streaming

Topics:

- WebSockets
- SSE
- Long polling
- gRPC
- HTTP/2
- Streaming
- Bidirectional communication
- Persistent connections

📖 [Read Phase 19 →](19-websockets-grpc-and-streaming.md)

---

## Part 5 — CDN & Internet Architecture

### 20. CDN & Caching

Topics:

- CDN
- Edge server
- Origin server
- Cache
- Cache hit
- Cache miss
- TTL
- Cache-Control
- ETag
- Cache invalidation
- Origin shield
- Edge caching

📖 [Read Phase 20 →](20-cdn-and-caching.md)

---

### 21. Internet Architecture

Topics:

- ISP
- Autonomous Systems
- AS numbers
- Internet backbone
- Peering
- Transit
- Data centers
- Internet exchange points
- Global Internet architecture

📖 [Read Phase 21 →](21-internet-architecture.md)

---

### 22. BGP & Anycast

Topics:

- BGP
- AS_PATH
- NEXT_HOP
- LOCAL_PREF
- MED
- Route advertisement
- Route selection
- eBGP
- iBGP
- Anycast
- Global traffic routing

📖 [Read Phase 22 →](22-bgp-and-anycast.md)

---

### 23. Advanced IP Networking

Topics:

- IPv6
- MTU
- MSS
- IP fragmentation
- Path MTU Discovery
- ECMP
- Jumbo frames
- IPv6 addressing

📖 [Read Phase 23 →](23-advanced-ip-networking.md)

---

### 24. Network Performance

Topics:

- Latency
- Bandwidth
- Throughput
- RTT
- Jitter
- Packet loss
- Congestion
- Queueing
- Bufferbloat
- Network bottlenecks
- Performance optimization

📖 [Read Phase 24 →](24-network-performance.md)

---

### 25. Network Troubleshooting

Practical production troubleshooting.

Topics:

- ping
- traceroute
- dig
- nslookup
- curl
- ss
- netstat
- tcpdump
- Wireshark
- DNS debugging
- TCP debugging
- TLS debugging
- HTTP debugging
- Packet loss
- Timeouts
- Connection failures
- 4xx / 5xx investigation

📖 [Read Phase 25 →](25-network-troubleshooting.md)

---

# Part 6 — AWS Networking

AWS networking will be studied after understanding the underlying networking concepts.

### 26. AWS Networking Fundamentals

Topics:

- AWS global infrastructure
- Region
- Availability Zone
- VPC
- VPC CIDR
- Subnets
- Public subnet
- Private subnet
- Availability Zones
- Network interfaces

📖 [Read Phase 26 →](26-aws-networking-fundamentals.md)

---

### 27. AWS Routing & Connectivity

Topics:

- Route tables
- Internet Gateway
- NAT Gateway
- ENI
- Elastic IP
- Public IP
- Private IP
- Public vs private subnet
- Routing between subnets

📖 [Read Phase 27 →](27-aws-routing-and-connectivity.md)

---

### 28. AWS Network Security

Topics:

- Security Groups
- Network ACLs
- Stateful vs stateless filtering
- VPC Flow Logs
- AWS Network Firewall
- Network security architecture

📖 [Read Phase 28 →](28-aws-network-security.md)

---

### 29. AWS Private Networking

Topics:

- VPC Peering
- Transit Gateway
- VPC Endpoints
- Gateway endpoints
- Interface endpoints
- AWS PrivateLink
- Cross-VPC communication

📖 [Read Phase 29 →](29-aws-private-networking.md)

---

### 30. AWS DNS & Traffic Routing

Topics:

- Route 53
- Hosted zones
- DNS records
- Health checks
- Simple routing
- Weighted routing
- Latency-based routing
- Failover routing
- Geolocation routing
- DNS-based traffic management

📖 [Read Phase 30 →](30-aws-dns-and-traffic-routing.md)

---

### 31. AWS Load Balancing & CDN

Topics:

- Application Load Balancer
- Network Load Balancer
- Gateway Load Balancer
- Layer 4 vs Layer 7
- CloudFront
- CloudFront origins
- Edge locations
- CloudFront caching
- CloudFront TLS
- CloudFront + S3
- CloudFront + ALB

📖 [Read Phase 31 →](31-aws-load-balancing-and-cdn.md)

---

### 32. AWS Global & Hybrid Networking

Topics:

- AWS Global Accelerator
- VPN
- Site-to-Site VPN
- Client VPN
- Direct Connect
- Hybrid networking
- On-premise ↔ AWS
- Global traffic architecture

📖 [Read Phase 32 →](32-aws-global-and-hybrid-networking.md)

---

### 33. AWS Network Architecture & Troubleshooting

Real-world AWS networking scenarios.

Topics:

- Production VPC architecture
- Multi-AZ architecture
- Multi-account networking
- Network failures
- Connectivity debugging
- DNS failures
- Security Group issues
- Route table issues
- NAT Gateway issues
- VPC Flow Logs
- Production troubleshooting

📖 [Read Phase 33 →](33-aws-network-architecture-and-troubleshooting.md)

---

# Part 7 — Akamai & Distributed Systems

### 34. Akamai & Edge Networking

Akamai-focused networking concepts.

Topics:

- CDN architecture
- Edge servers
- Origin servers
- Edge caching
- Global traffic management
- DNS-based routing
- Anycast
- BGP
- Edge TLS
- Origin → Edge communication
- Cache invalidation
- Edge computing
- Global distribution

📖 [Read Phase 34 →](34-akamai-and-edge-networking.md)

---

### 35. Networking for Distributed Systems

Connecting networking to distributed systems and system design.

Topics:

- Network partitions
- Partial failures
- Timeouts
- Retries
- Exponential backoff
- Jitter
- Idempotency
- Backpressure
- Service discovery
- Connection pools
- Circuit breakers
- Replication
- Consistency
- CAP theorem
- Network reliability

📖 [Read Phase 35 →](35-networking-for-distributed-systems.md)

---

### 36. Senior Software Engineer Networking Interview

Final revision and interview preparation.

Topics:

- Networking fundamentals
- TCP deep-dive questions
- DNS questions
- HTTP questions
- TLS questions
- Load balancing
- CDN
- AWS networking
- BGP / Anycast
- Production troubleshooting
- Network failure scenarios
- System design questions
- Senior-level scenario questions

📖 [Read Phase 36 →](36-senior-networking-interview.md)

---

# 🧠 How to Study

Each phase should be studied using the following approach:

```text
1. Understand the intuition
        ↓
2. Understand why the concept exists
        ↓
3. Understand how it works internally
        ↓
4. Visualize the packet/data flow
        ↓
5. Try a practical example
        ↓
6. Use Linux/networking tools where possible
        ↓
7. Solve interview questions
        ↓
8. Connect it to AWS / distributed systems
        ↓
9. Revise from the short notes
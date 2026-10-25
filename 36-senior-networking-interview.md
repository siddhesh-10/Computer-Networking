# Networking — Phase 36: Senior Software Engineer Networking Interview

> **Goal:** Revise the major networking ideas, connect them to real system design decisions, and practice senior-level reasoning under production failure conditions.

## 1. What senior networking interviews are really testing

At the senior level, interviewers are usually not testing memorization alone. They are checking whether you can reason about:
- how traffic moves through a system
- why a network problem appears as an app error
- how reliability is designed across layers
- how TCP, DNS, HTTP, TLS, and routing interact in real systems
- how to debug production issues under uncertainty

A good senior answer usually includes:
- the core mechanism
- the real-world failure mode
- the trade-offs
- the operations and debugging implications

---

## 2. Foundations you should be able to explain clearly

You should be able to explain these without hesitation:
- IP, MAC, ports, sockets
- segments, packets, frames
- TCP handshake and connection lifecycle
- UDP vs TCP
- router, switch, route table, default gateway
- NAT, firewalls, subnetting, CIDR
- DNS resolution and caching
- HTTP request/response flow
- TLS handshake and certificate trust
- load balancing and reverse proxies
- CDN edge architecture and cache behavior
- BGP, Anycast, and global routing

If you can explain these in plain English and tie them to a real request flow, you are in a strong position.

---

## 3. Typical request-flow explanation

A common senior question is:

“What happens when a user loads a website?”

A solid answer:

```text
User browser
  ↓
DNS resolves hostname
  ↓
TCP connection to edge or load balancer
  ↓
TLS handshake
  ↓
HTTP request sent
  ↓
Load balancer routes to app
  ↓
App queries database/cache
  ↓
Response returns through reverse proxy / edge / client
```

The key is not just naming each step, but being able to say why each layer exists:
- DNS translates names to addresses
- TCP handles reliability and ordered delivery
- TLS protects the connection
- HTTP encodes the actual request/response
- load balancers distribute based on health and policy
- caches reduce latency and origin load

---

## 4. TCP deep-dive questions

Senior interviewers often test that you understand TCP beyond “three-way handshake.”

### Core TCP topics
- SYN, SYN-ACK, ACK
- sequence numbers and ACK numbers
- sliding window
- flow control
- congestion control
- slow start and congestion avoidance
- retransmission timeout
- fast retransmit / fast recovery
- TIME_WAIT and connection close behavior

### Example question
Why does TCP need both sequence numbers and acknowledgements?

Answer:
- sequence numbers let the receiver reassemble data in order
- acknowledgements tell the sender what bytes were received
- together they support reliability, reordering recovery, flow control, and retransmission

### Example question
Why is TIME_WAIT important?

Answer:
- it helps ensure delayed packets from a completed TCP connection do not get confused with a new connection
- it makes connection reuse and teardown safer
- it is a trade-off between connection cleanup and resource usage

---

## 5. DNS interview questions

Common DNS questions:
- What happens when a browser resolves a domain?
- Why do we need recursive and authoritative DNS?
- What is TTL and why does it matter?
- Why can DNS fail even when the app server is healthy?
- What is the difference between NXDOMAIN and SERVFAIL?

### Good answer pattern
Explain the flow:
```text
Browser → recursive resolver → root → TLD → authoritative -> IP answer -> cache
```

Then mention:
- TTL controls caching lifetime
- DNS cache can make stale records persist
- health checks and traffic routing may change answer selection
- DNS failures often look like app outages even when backend services are healthy

---

## 6. HTTP and HTTPS questions

You should be able to explain:
- request methods and semantics
- status codes
- headers and caching behavior
- persistent connections and keep-alive
- HTTP/1.1 vs HTTP/2 vs HTTP/3
- TLS handshake and certificate validation
- SNI and ALPN
- why HTTPS is not just encryption but also trust and authentication

### Example question
Why use HTTP/2 or HTTP/3?

Answer:
- reduce head-of-line blocking
- multiplex requests over one connection
- improve throughput and latency
- keep connections efficient under high concurrency

### Example question
What is the difference between a reverse proxy and a load balancer?

Answer:
- a load balancer distributes traffic among backend instances
- a reverse proxy often terminates TLS, handles caching, or adds filtering/application logic
- the same component can perform both roles depending on architecture

---

## 7. TLS and security interview questions

Senior-level TLS questions often explore trust and behavior rather than just terminology.

Understand:
- symmetric vs asymmetric encryption
- certificate chain and CA trust
- handshake flow
- why certificates matter
- how TLS termination at the edge changes architecture
- SNI and ALPN use cases

### Example question
Why terminate TLS at a load balancer or CDN?

Answer:
- offloads CPU work from application servers
- centralizes certificate management
- allows edge security policies and filtering
- can improve global performance through CDN routing

### Example question
Why do we still need backend TLS even if the edge terminates TLS?

Answer:
- the connection from edge to origin is still a network hop and should not be trusted by default
- origin services may need end-to-end confidentiality and integrity
- internal traffic may require separate certificate validation and security policy

---

## 8. Load balancing, CDN, and edge questions

Typical questions:
- What is the difference between L4 and L7 load balancing?
- How does a CDN improve latency?
- Why do edge networks use Anycast and BGP?
- Why do we need health checks?
- What is cache invalidation and why is it hard?

### Strong answer pattern
Mention the actual path:
```text
Client → edge / LB → app tier → DB
```

Then explain:
- edge caches serve content quickly and reduce origin load
- load balancers distribute traffic among healthy backends
- health checks remove unhealthy nodes from routing
- global traffic management directs users to the best region
- failures are isolated geographically and operationally

---

## 9. BGP and Anycast explanation

You should be able to explain BGP in a practical way.

### BGP basics
- BGP is the routing protocol used between autonomous systems
- it exchanges reachability information
- route selection depends on policy, local preference, AS_PATH, and metrics

### Anycast basics
- same IP advertised from multiple locations
- client traffic is routed to the nearest or best path
- useful for global services and DNS/CDN delivery

### Senior-level explanation
You are not just memorizing BGP syntax. You are describing how global Internet routing allows a service to remain reachable even when some regions or upstream links fail.

---

## 10. Production troubleshooting questions

These are extremely common in senior interviews.

Example scenarios:
- users in one region cannot reach the app
- intermittent 5xxs across a service
- DNS resolves but TCP connection hangs
- TLS handshake fails for some clients
- CDN cache returns stale content
- one AZ is degraded but others are healthy

### Good debugging flow
1. Identify the scope of the issue
2. Verify the exact request path
3. Check DNS and routing
4. Check the application and backend dependencies
5. Check socket/connectivity and load balancer health
6. Check security groups, NACLs, and network path
7. Inspect logs, metrics, and traces
8. Compare healthy vs failing paths

This is often what senior candidates are graded on: calm, structured debugging under constraints.

---

## 11. Distributed systems networking questions

These are especially important for senior engineering roles.

### Common topics
- network partitions
- partial failures
- retry storms
- exponential backoff and jitter
- timeouts
- idempotency
- circuit breakers
- backpressure
- consistency trade-offs
- replication over the network

### Example question
Why are retries dangerous without idempotency?

Answer:
- network retries can resend requests after the first call actually succeeded
- this can cause duplicate writes, duplicate charges, or repeated side effects
- idempotency keys or deduplication are needed for safety

### Example question
What happens when a dependency is slow?

Answer:
- latency increases upstream
- timeouts fire
- retry traffic multiplies
- queues grow
- circuit breakers may open
- user-facing performance degrades

This is the point: network issues become system design issues.

---

## 12. System design connection

Senior networking interviews often connect to architecture questions such as:
- designing a global API platform
- designing a high-availability multi-region system
- choosing between edge cache, load balancer, and app server placement
- routing traffic around a regional outage
- protecting against DDoS and bot abuse

The right answer usually includes:
- region-aware routing
- health-aware traffic allocation
- cache at the edge
- private backends
- observability and alerting
- graceful degradation under failure

---

## 13. Real-world “why” questions

Interviewer may ask:
- Why not just make every request one big TCP connection?
- Why use DNS TTL at all?
- Why do we have both public and private subnets?
- Why do CDNs use cache invalidation rules instead of always serving fresh content?
- Why do we design for retries and timeouts instead of assuming perfect connectivity?

The answer is always the same: the network is unreliable, diverse, and geographically distributed. Systems must tolerate this reality.

---

## 14. Strong answer format for interviews

When answering networking questions, use this structure:

1. Define the concept simply
2. Explain why it exists
3. Show the data path
4. Describe the failure modes
5. Mention trade-offs and real-world implications

Example structure:

“TCP provides ordered, reliable delivery by using sequence numbers, acknowledgements, and retransmission. It exists because IP alone is unreliable and packets can be lost or reordered. In practice, this means a service can trust that bytes arrive in order and can recover from packet loss, but it also means we must handle timeouts, flow control, and connection teardown correctly.”

This is the pattern you want to repeat.

---

## 15. High-value topic checklist before interviews

Make sure you can speak confidently about:
- TCP handshake and close behavior
- Congestion control and timeout semantics
- DNS resolution and caching
- HTTP request/response lifecycle
- TLS handshake and certificate validation
- NAT and subnetting reasoning
- Load balancing types and routing strategies
- CDN architecture and cache behavior
- BGP/Anycast/global traffic routing
- Cloud and hybrid networking basics
- distributed failures and resilience patterns

If you can teach these at a high level, the interview is much easier.

---

## 16. Final senior-level summary

A senior networking engineer does not just know the definitions. They can reason about:
- the full path of a request across networks
- why the network and the app fail in correlated ways
- how traffic is routed around outages and congestion
- how caches, load balancers, and DNS influence latency and reliability
- how to debug a failure from end-to-end without guessing

That is the real skill: understanding the system as a whole, not just isolated concepts.

---

## Key takeaways

- Networking is the foundation of distributed systems, not a separate specialty.
- The strongest candidates explain mechanisms, trade-offs, and production behavior.
- TCP, DNS, HTTP, TLS, CDN, and routing all interact in real request paths.
- Senior-level answers should reason about failure, resilience, latency, and operations.
- The goal is not to memorize every fact; it is to reason clearly under uncertainty.

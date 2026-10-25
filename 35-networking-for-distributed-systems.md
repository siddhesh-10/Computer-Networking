# Networking — Phase 35: Networking for Distributed Systems

> **Goal:** Connect networking fundamentals to distributed systems design, failure handling, and production reliability.

## 1. Why distributed systems are fundamentally networked

A distributed system is a system where multiple machines work together over a network. That means the network is not an optional layer; it is a core part of correctness and reliability.

```text
Client
  ↓
Load balancer
  ↓
Service A
  ↓
Service B
  ↓
Database / Cache / Queue
```

Every hop introduces:
- latency
- packet loss risk
- retries
- partial failures
- timeouts
- dependency on remote health

A distributed system is often more robust than a single machine, but it is also more sensitive to network problems.

---

## 2. Network partitions

A **network partition** happens when different parts of a distributed system cannot communicate with each other even though the machines remain alive.

```text
Service A   ─────── disconnected ───────   Service B
     │                                       │
     └────────────── same cluster ──────────┘
```

This can happen because of:
- switch failures
- router issues
- ISP problems
- misconfigured subnets
- overloaded network links
- DDoS or degraded connectivity

A system must continue functioning safely even when it cannot fully trust the network.

---

## 3. Partial failures

A **partial failure** means only some components fail while others continue working.

Example:
- one DB replica is slow
- one service instance is overloaded
- one region is unreachable
- a dependency times out while the rest of the system is healthy

This is one of the hardest parts of distributed systems because the failure is not always obvious. A service may appear alive while actually degraded.

```text
Healthy request flow
Client → API → DB

Partial failure
Client → API → DB timeout
```

The system must treat remote calls as unreliable by default.

---

## 4. Timeouts

A **timeout** is the limit we wait before deciding that a network request has failed or stalled.

If a timeout is too short:
- retries happen too aggressively
- clients see false failures
- load increases

If it is too long:
- users wait longer
- blocked threads or goroutines accumulate
- system resources get exhausted

A good timeout is usually tied to the expected latency and the call's criticality.

```text
Call starts
   ↓
Wait for response
   ↓
If deadline exceeded → timeout
```

Timeouts are required because the network is not guaranteed to deliver on time.

---

## 5. Retries

A **retry** resends a request after a failure or timeout.

Retries help when:
- a packet was lost
- a server briefly restarted
- a connection was dropped
- a load balancer redirected traffic

But retries can create damage if used blindly:
- duplicate writes
- duplicate charges
- repeated expensive operations
- cascading overload

```text
Request sent
   ↓
Timeout / failure
   ↓
Retry
```

Retries should usually be combined with jitter, idempotency checks, and backoff.

---

## 6. Exponential backoff

**Exponential backoff** increases the delay between retries over time.

Example:
- 100ms
- 200ms
- 400ms
- 800ms
- 1.6s

```text
retry 1 → 100ms
retry 2 → 200ms
retry 3 → 400ms
retry 4 → 800ms
```

This helps because repeated retries can amplify load during outages. Backoff helps the system recover instead of hammering a failing dependency.

---

## 7. Jitter

**Jitter** is a small randomization added to retry timing.

Without jitter:
```text
all clients retry at the same time
```

With jitter:
```text
clients retry at slightly different times
```

This prevents retry storms, where many clients simultaneously hit the same failing service and make the outage worse.

Jitter is especially important in distributed systems during incident recovery and traffic rebalancing.

---

## 8. Idempotency

An operation is **idempotent** if repeating it has the same effect as doing it once.

Examples:
- `SET user_id=42` is idempotent
- `charge_credit_card(100)` is not idempotent unless tied to a unique idempotency key

```text
Request A
  repeated twice
  should not create duplicate side effects
```

For network retries, idempotency is essential. Without it, retries may accidentally cause duplicate writes or charges.

---

## 9. Backpressure

**Backpressure** is the system’s way of telling upstream callers: “I am overloaded, slow down.”

```text
Producer → Queue → Consumer
        ↑
   too much load
```

If consumers cannot keep up:
- queues grow
- latencies rise
- memory increases
- the system eventually fails harder

Backpressure may appear as:
- rate limiting
- queueing
- connection refusal
- request rejection
- reduced concurrency

Healthy systems shed load before they collapse.

---

## 10. Service discovery

**Service discovery** is how services find each other in a dynamic cluster.

Examples:
- DNS-based service lookup
- Consul / etcd / ZooKeeper
- Kubernetes service endpoints
- load balancers with dynamic backend registration

```text
API service
   ↓
discovers order service
   ↓
connects to healthy instance
```

In a distributed system, service instances come and go. Hardcoded IPs do not scale well.

Service discovery is required for resilience, scaling, and rolling deployments.

---

## 11. Connection pools

A **connection pool** keeps a set of reusable network connections instead of creating a new one for every request.

```text
Request
  ↓
Pool of open connections
  ↓
Target service
```

Benefits:
- lower connection setup cost
- reduced latency
- better reuse of sockets
- improved throughput

Risks:
- connection exhaustion
- stale connections
- thread pool saturation
- hidden bottlenecks

Connection pooling is common for DBs, HTTP clients, and service-to-service communication.

---

## 12. Circuit breakers

A **circuit breaker** detects repeated failures and stops sending requests to a failing dependency for a while.

```text
Calls to dependency
  ↓
Failure threshold reached
  ↓
Open circuit: fail fast
  ↓
After timeout: half-open, test recovery
```

Circuit breakers help prevent:
- cascading failure
- repeated retries to an already failing service
- load amplification

They are essential when a service depends on remote systems that may be unstable or overloaded.

---

## 13. Replication

**Replication** means copying data or functionality across multiple nodes.

Common types:
- database replication
- cache replication
- leader/follower pattern
- multi-region replication

```text
Primary DB
   ├─ replica A
   ├─ replica B
   └─ replica C
```

Replication improves:
- availability
- read scaling
- resilience to node failure

But it also introduces:
- replication lag
- consistency trade-offs
- conflict resolution complexity

---

## 14. Consistency

**Consistency** refers to how visible a system’s data is across replicas and clients.

Examples:
- strong consistency: reads see the latest value immediately
- eventual consistency: writes spread out eventually; reads may be stale temporarily

```text
write to primary
   ↓
replicas catch up
   ↓
eventual consistency window
```

The more consistency you want, the more coordination and latency you usually pay. This is a core design trade-off in distributed systems.

---

## 15. CAP theorem

The **CAP theorem** says that a distributed system can have at most two of these three properties:
- **Consistency**
- **Availability**
- **Partition tolerance**

During a network partition, a system usually must choose between:
- consistency
- availability

```text
CAP
├─ Consistency
├─ Availability
└─ Partition tolerance
```

The practical lesson is that distributed systems cannot be perfectly consistent, perfectly available, and perfectly partition-tolerant at the same time.

---

## 16. Network reliability and failure domains

In distributed systems, every service has a **failure domain**.

Examples:
- one process
- one host
- one AZ
- one region
- one network provider

A dependency can fail without taking the whole system down.

```text
System A
  ├─ dependency 1: healthy
  ├─ dependency 2: slow
  └─ dependency 3: down
```

Good design reduces blast radius by isolating failures and keeping dependencies small and replaceable.

---

## 17. Why networking is central to distributed design

Distributed systems are designed under assumptions like:
- requests can be dropped
- peers can disappear
- clocks are not perfectly synchronized
- TCP connections can break
- remote latency varies
- retries may happen multiple times

The networking layer influences:
- system throughput
- correctness
- latency
- observability
- resilience

If a team ignores the network, they often build a system that looks correct in tests but fails in production.

---

## 18. Time and clocks

Distributed systems often depend on time, but clocks are not perfectly aligned across machines.

Problems include:
- clock skew
- stale metadata
- inconsistent ordering
- time-based cache expiry mismatches

This affects:
- lease expiration
- log ordering
- cache invalidation
- distributed coordination

Network timing and clock synchronization matter even when the system is otherwise healthy.

---

## 19. TCP behavior in distributed systems

TCP is widely used inside services because it offers reliability and ordered delivery, but it still has failure modes:
- broken connections
- half-open connections
- keep-alive exhaustion
- backpressure from peers
- delayed or lost segments

A distributed system may think a connection is healthy while it is silently stalled. That is why timeouts, health checks, and retry logic matter.

---

## 20. DNS in distributed systems

DNS is often the first thing a client touches when connecting to a service.

It matters for:
- load balancing
- failover across regions
- service discovery
- health-aware routing
- traffic shifting during deploys

```text
Client → DNS → target service IP → connection
```

If TTL, resolver behavior, or health checks are wrong, a system may continue sending traffic to dead endpoints.

---

## 21. Load balancing and distributed traffic

A distributed system often has multiple instances behind a load balancer.

The balancer can distribute based on:
- round robin
- least connections
- weighted routing
- health checks
- session affinity

```text
Users
  ↓
LB
  ├─ instance A
  ├─ instance B
  └─ instance C
```

This improves availability but requires careful handling of retries, connection reuse, and sticky sessions when needed.

---

## 22. Partial success and user experience

In distributed systems, not every request succeeds completely.

Examples:
- DB commit succeeded, email send failed
- request reached service, response lost to client
- client retries but backend only processed once

This is why distributed systems often need:
- idempotency keys
- compensating actions
- retries with caution
- clear failure semantics

The network should be treated as a source of uncertainty, not a perfect channel.

---

## 23. Observability is part of reliability

A distributed system is only as reliable as the observability around it.

Need to monitor:
- request latency
- timeout rates
- retries
- queue depth
- connection counts
- error ratios
- dependency health
- packet drops / network issues

Without good metrics and logs, failures are hard to diagnose and easy to misinterpret.

---

## 24. Retry storms and cascades

A common production problem is a **retry storm**.

```text
Dependency fails
  ↓
many clients retry at once
  ↓
dependency is hammered harder
  ↓
system crashes or saturates
```

This occurs because distributed systems fail in waves. Jitter, rate limiting, and circuit breakers are used to prevent a small issue from becoming a major incident.

---

## 25. SLOs, SLAs, and error budgets

System design depends on acceptable failure rates.

Common terms:
- **SLO**: service level objective
- **SLA**: service level agreement
- **Error budget**: allowed failure percentage before a service breaches its SLO

If a service is latency-sensitive, the network budget may be tiny. That can force stronger caching, regional deployment, or connection optimization.

---

## 26. Queueing and latency amplification

A request may pass through multiple services, each adding queueing and retry delay.

```text
Client
  ↓
Service A
  ↓
Service B
  ↓
Database
```

If one service slows down:
- requests pile up
- latency compounds
- downstream dependencies feel it
- retries multiply

This is called **latency amplification**. It is often the reason distributed systems fail under load even when no single component is broken.

---

## 27. TCP vs HTTP semantics in distributed systems

At the HTTP layer, clients and servers care about application semantics, while TCP delivers packets reliably. The application may still face:
- connection resets
- half-closed streams
- stale connection reuse
- missing request/response pairing

A distributed system must handle both the transport layer and the application semantics correctly.

---

## 28. Failure handling patterns

Good distributed systems usually combine several patterns:
- retries with backoff and jitter
- timeouts with realistic budgets
- circuit breakers
- graceful degradation
- caching
- queueing and buffering
- rate limits
- health checks

These patterns are not only coding tricks; they are network-aware defensive design.

---

## 29. Connection reuse and keep-alives

Keep-alive connections are important for reducing handshake overhead and reusing established TCP sessions.

Benefits:
- fewer connects
- lower latency for repeated requests
- better throughput

But if a keep-alive connection goes stale, clients must detect and recover. This is another place where network reliability and client robustness meet.

---

## 30. Client-side and server-side resilience

Modern systems require both client and server resilience.

Client side:
- handle timeouts
- retry on transient failures
- respect rate limits
- handle 429 / 5xx gracefully

Server side:
- detect slow dependencies
- fail fast when overloaded
- shed traffic with queues or load limits
- maintain an accurate health signal

The network is not a safe assumption. Reliability must be designed in.

---

## 31. Data consistency and replication lag

In a distributed database or cache system, replicas may lag behind the leader.

If reads go to a lagging replica:
- users may see stale data
- a request may fail if the write is not visible yet

This is why many systems choose:
- read-after-write consistency for some paths
- leader-based reads for critical writes
- quorum-based coordination for strong semantics

The right choice depends on the product, not just the networking stack.

---

## 32. Regional and multi-zone systems

Distributed systems often run across multiple AZs or regions.

This helps with:
- outage isolation
- disaster recovery
- lower latency for local users
- service continuity under regional failure

But it adds complexity:
- cross-region traffic cost
- latency increases
- DNS and routing changes
- data replication conflicts

Network design becomes a first-class part of reliability engineering.

---

## 33. Production incident mindset

In production, when a system fails, ask:
- Is this a dependency problem or a code bug?
- Is the network partitioned or just slow?
- Are retries amplifying the issue?
- Is a timeout too low or too high?
- Are we exhausting connections or queues?
- Did a dependency fail in one AZ or region?

The right debugging path is usually:
- identify the failing dependency
- inspect latency and error rates
- confirm the network path and health checks
- evaluate retry and backoff behavior
- verify the effect of load and circuit breaker state

---

## 34. A practical distributed-systems mental model

A useful mental model is:

```text
Everything is remote
Everything can fail
Everything can be slow
Everything can be retried
Everything must be designed around uncertainty
```

A distributed system is not “just compute”; it is compute plus communication, failure handling, and coordination.

---

## 35. Interview-style summary

A strong interview answer is:

“Distributed systems rely on the network for every interaction, so reliability depends on handling timeouts, partial failures, retries, partitions, and degraded dependencies. Concepts like exponential backoff, jitter, idempotency, circuit breakers, and backpressure are essential because the network is unreliable by default. We also need to reason about consistency, replication, service discovery, and load balancing, since the system must stay correct and available even when some machines or regions fail.”

This explains why networking is central to system design and operational resilience.

---

## Key takeaways

- Distributed systems are networked systems; the network is part of the design, not a side detail.
- Network partitions and partial failures are normal engineering concerns.
- Timeouts, retries, backoff, and jitter are required to keep systems stable.
- Idempotency and circuit breakers protect services from repeated failure effects.
- Consistency, replication, and service discovery determine correctness and availability.
- Good observability and defensive traffic patterns are essential in production.
- The real design goal is not “no failures,” but graceful behavior under failure.

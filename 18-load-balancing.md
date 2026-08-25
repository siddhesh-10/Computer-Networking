# Networking — Phase 18: Load Balancing

> **Goal:** Understand how load balancers distribute traffic across multiple backend servers, how L4 and L7 load balancing differ, and how algorithms, health checks, sessions, and connection draining affect traffic distribution.

## 1. What Is Load Balancing?

**Load balancing** distributes incoming network traffic across multiple backend servers.

```text
Clients
   ↓
Load Balancer
   ↓
┌────────┬────────┬────────┐
Server A Server B Server C
```

It helps provide:

```text
Scalability
High availability
Fault tolerance
Better resource utilization
```

---

## 2. Where Does a Load Balancer Fit?

A load balancer can operate at different layers.

### L4

```text
Client
   ↓
L4 Load Balancer
   ↓
TCP / UDP
   ↓
Backend
```

It primarily uses:

```text
IP addresses
Ports
Protocol
Connection state
```

### L7

```text
Client
   ↓
L7 Load Balancer
   ↓
HTTP / HTTPS
   ↓
Backend
```

It can understand:

```text
Host
URL/path
HTTP method
Headers
Cookies
```

---

## 3. L4 vs L7 Load Balancing

| | L4 | L7 |
|---|---|---|
| Layer | Transport | Application |
| Understands HTTP | No | Yes |
| Routing by IP/port | Yes | Yes |
| Routing by URL | No | Yes |
| Routing by headers | No | Yes |
| Routing by cookies | No | Yes |
| Protocol examples | TCP/UDP | HTTP/gRPC |

```text
L4 → "Which backend should handle this connection?"
L7 → "Which backend should handle this application request?"
```

---

## 4. Basic Request Flow

```text
Client
   ↓
Load Balancer
   ↓
Server B
   ↓
Response
   ↓
Load Balancer
   ↓
Client
```

The load balancer selects an eligible backend from its pool.

---

## 5. Round Robin

**Round robin** sends requests/connections to servers sequentially.

```text
Request 1 → A
Request 2 → B
Request 3 → C
Request 4 → A
Request 5 → B
```

Simple and predictable.

Limitation:

> It does not consider different server capacities or current workload.

---

## 6. Weighted Round Robin

Servers receive different proportions of traffic.

Example:

```text
Server A → weight 3
Server B → weight 1
```

Conceptually:

```text
A → A → A → B → A → A → A → B
```

A receives roughly three times as much traffic as B.

Useful when servers have different capacities.

---

## 7. Least Connections

The load balancer sends a new connection to the backend with the fewest active connections.

```text
Server A → 10 connections
Server B → 4 connections
Server C → 7 connections
```

New connection:

```text
New connection
      ↓
Server B
```

Useful when connection durations vary.

---

## 8. Consistent Hashing

**Consistent hashing** maps a key to a backend while minimizing remapping when servers are added or removed.

Example:

```text
user-123
   ↓
Hash
   ↓
Server B
```

Common keys:

```text
User ID
Session ID
Cache key
Object ID
```

Useful for:

```text
Distributed caches
Stateful services
Session affinity
Data partitioning
```

---

## 9. Health Checks

A load balancer should avoid sending traffic to unhealthy backends.

Example:

```text
Server A → healthy
Server B → unhealthy
Server C → healthy
```

An HTTP health check might be:

```http
GET /health
```

Expected:

```text
HTTP/1.1 200 OK
```

If B fails:

```text
Load Balancer
   ↓
┌────────┬────────┐
Server A Server C
```

B is temporarily removed from the active pool.

---

## 10. Active vs Passive Health Checks

### Active

The load balancer sends checks:

```text
Load Balancer
      ↓
GET /health
      ↓
Server
```

### Passive

The load balancer observes real traffic:

```text
Connection failures
Timeouts
5xx responses
```

Implementations may use either or both approaches.

---

## 11. Sticky Sessions

Without affinity:

```text
User X
 ↓
Server A

User X
 ↓
Server B
```

With **sticky sessions**:

```text
User X
  ↓
Server A
  ↓
Server A
  ↓
Server A
```

Common mechanisms:

```text
Cookie-based affinity
Source-IP affinity
Session identifiers
```

---

## 12. Why Sticky Sessions?

Suppose session state exists only on Server A:

```text
Server A
 └── User session
```

If the next request reaches B:

```text
User
 ↓
Server B
 ↓
Session not found
```

Sticky sessions can avoid this.

A more scalable design often externalizes state:

```text
Server A ─┐
Server B ─┼──→ Shared session store
Server C ─┘
```

For example:

```text
Database
Distributed cache
External session store
```

---

## 13. Connection Draining

When a backend is being removed, immediately terminating active connections can cause failures.

**Connection draining** lets existing connections finish.

```text
Server B
   ↓
Marked for removal
   ↓
No new connections
   ↓
Existing connections continue
   ↓
Connections finish
   ↓
Server removed
```

Useful during:

```text
Deployments
Maintenance
Scaling in
Backend replacement
```

---

## 14. TLS Termination

A load balancer can terminate TLS:

```text
Client
   ↓ HTTPS
Load Balancer
   ↓ HTTP / HTTPS
Backend
```

The load balancer handles:

```text
TLS handshake
Certificate
Encryption/decryption
```

This can centralize certificate management and enables L7 inspection after decryption.

---

## 15. TLS Passthrough

The load balancer can forward encrypted traffic without terminating TLS:

```text
Client
   ↓ HTTPS
Load Balancer
   ↓ HTTPS
Backend
```

Backend performs TLS termination.

```text
TLS termination
 ↓
LB can inspect HTTP and perform L7 routing

TLS passthrough
 ↓
LB cannot inspect encrypted HTTP content
```

---

## 16. Combining Algorithms and Health Checks

A real load balancer typically does:

```text
Health checks
     ↓
Healthy backend pool
     ↓
Load-balancing algorithm
     ↓
Select backend
```

Example:

```text
A = healthy
B = unhealthy
C = healthy
     ↓
Round robin
     ↓
A → C → A → C
```

---

## 17. Example Architecture

```text
                 Internet
                    |
                    ↓
                   DNS
                    |
                    ↓
             Load Balancer
              /     |                  ↓      ↓      ↓
          App A   App B   App C
             \      |      /
              \     |     /
                Database
```

The load balancer can provide:

```text
Traffic distribution
Health checks
TLS termination
Connection management
```

---

## 18. Practical: Test Endpoint Connectivity

Run:

```powershell
Test-NetConnection example.com -Port 443
```

Example:

```text
ComputerName     : example.com
RemotePort       : 443
TcpTestSucceeded : True
```

Meaning:

```text
TCP connection to example.com:443
was successfully established.
```

This tests endpoint connectivity; it does not reveal the balancing algorithm.

---

## 19. Practical: Inspect HTTP Response

```powershell
curl.exe -I https://example.com
```

Example:

```text
HTTP/1.1 200 OK
Content-Type: text/html
```

Headers may reveal load-balancer/proxy information, but this depends on the implementation.

---

## 20. Algorithm Comparison

| Algorithm | Main idea | Useful when |
|---|---|---|
| Round robin | Rotate through servers | Similar servers/workloads |
| Weighted round robin | Weight traffic | Different server capacity |
| Least connections | Fewest active connections | Connection duration varies |
| Consistent hashing | Stable key → backend | Caches/stateful workloads |

---

## 21. Load Balancing Mental Model

```text
                    Clients
                       |
                       ↓
                 Load Balancer
                       |
                 Health checks
                       |
              Healthy backend pool
                       |
              Selection algorithm
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Server A      Server B      Server C
```

Flow:

```text
Traffic
   ↓
Healthy backends
   ↓
Algorithm
   ↓
Selected backend
   ↓
Response
   ↓
Client
```

---

## 22. Summary

```text
Load Balancer
 ↓
Distributes traffic across backends

L4
 ↓
Balances TCP/UDP connections

L7
 ↓
Balances application requests

Round robin
 ↓
Sequential distribution

Weighted round robin
 ↓
Distribution based on weights

Least connections
 ↓
Choose backend with fewer active connections

Consistent hashing
 ↓
Stable key → backend mapping

Health checks
 ↓
Remove unhealthy backends

Sticky sessions
 ↓
Keep a client/session associated with a backend

Connection draining
 ↓
Finish existing work before backend removal

TLS termination
 ↓
Load balancer handles TLS
```

Core idea:

> A load balancer distributes traffic among **eligible/healthy backends**, using a selection algorithm appropriate for the workload.

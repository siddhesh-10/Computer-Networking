# Networking — Phase 24: Network Performance

> **Goal:** Understand latency, bandwidth, throughput, RTT, jitter, packet loss, congestion, queueing, bufferbloat, bottlenecks, and practical performance optimization.

## 1. Network Performance Basics

Network performance describes how efficiently data moves between endpoints.

```text
Latency       → How long data takes to travel
Bandwidth     → How much capacity a link has
Throughput    → How much data is actually delivered
RTT           → Time for request + response
Jitter        → Variation in packet delay
Packet loss   → Packets that never arrive
Congestion    → Too much traffic for available capacity
Queueing      → Packets waiting for transmission
```

Performance is often limited by the slowest or most congested part of the path.

## 2. Latency

**Latency** is the time required for data to travel from one point to another.

- **Layer:** End-to-end performance property, not a single protocol-layer feature
- Measured in milliseconds (`ms`)
- Affects TCP, HTTP, DNS, databases, APIs, and distributed systems

```text
Processing + Serialization + Propagation + Queueing + Protocol delays
                         = End-to-end latency
```

Latency is not the same as bandwidth. A link can have high bandwidth and high latency, or low bandwidth and low latency.

## 3. RTT

**RTT (Round-Trip Time)** is the time for a packet/request to reach a destination and for a response to return.

```text
Client                    Server
  |-------- Request ------->|
  |<------- Response -------|
          ← RTT →
```

Example:

```text
Reply from 8.8.8.8: bytes=32 time=22ms TTL=117
```

`22ms` is the approximate round-trip delay.

## 4. Bandwidth

**Bandwidth** is the maximum data-carrying capacity of a link.

```text
Link capacity = 1 Gbps
```

Think:

```text
Bandwidth = size of the pipe
Throughput = data actually flowing through it
```

## 5. Throughput

**Throughput** is the actual rate at which data is successfully delivered.

```text
Link bandwidth = 1 Gbps
Actual throughput = 650 Mbps
```

Possible causes of the difference:

- Congestion
- Packet loss
- Protocol overhead
- Slow sender/receiver
- CPU or disk limits
- TCP window limitations
- Network bottlenecks

## 6. Bandwidth vs Throughput

| Concept | Meaning |
|---|---|
| Bandwidth | Maximum link capacity |
| Throughput | Actual delivered data rate |
| Goodput | Useful application data rate |

```text
1 Gbps link
    ↓
800 Mbps TCP throughput
    ↓
760 Mbps application goodput
```

## 7. Bandwidth-Delay Product

**BDP (Bandwidth-Delay Product)** estimates how much data can be in flight for a path.

```text
BDP = Bandwidth × RTT
```

Example:

```text
Bandwidth = 1 Gbps
RTT = 100 ms

BDP = 1,000,000,000 × 0.1
    = 100,000,000 bits
    ≈ 12.5 MB
```

High-bandwidth, high-latency paths may need a large TCP congestion window to fully utilize the link.

## 8. Jitter

**Jitter** is variation in packet delay.

```text
Packet 1 → 20 ms
Packet 2 → 21 ms
Packet 3 → 45 ms
Packet 4 → 22 ms
```

Jitter matters especially for voice, video, gaming, and other real-time traffic.

A network can have acceptable average latency but poor real-time behavior if delay varies significantly.

## 9. Packet Loss

**Packet loss** occurs when packets fail to reach the destination.

```text
Sent:     1000 packets
Received: 995 packets
Lost:        5 packets

Loss = 0.5%
```

Causes include congestion, faulty links, hardware problems, wireless interference, queue overflow, and routing problems.

TCP generally responds to loss with retransmissions and congestion control, so loss can reduce effective throughput.

## 10. Congestion

**Congestion** occurs when offered traffic exceeds available capacity.

```text
Traffic in
   ↓
+---------+
| Router  |
| Queue   |
+---------+
   ↓
Link capacity
```

If packets arrive faster than the link can transmit them, they accumulate in a queue. A full queue can cause packet drops.

## 11. Queueing

Packets often wait in buffers before transmission.

```text
Packets
  ↓
[ Queue ]
  ↓
Network link
  ↓
Destination
```

Queueing adds latency.

```text
Low load  → small queue → low latency
High load → large queue → high latency
```

## 12. Bufferbloat

**Bufferbloat** occurs when excessively large buffers allow queues to grow very large.

```text
Traffic burst
     ↓
Large buffer
     ↓
Huge queue
     ↓
Long wait
     ↓
High latency
```

Example:

```text
Idle RTT   = 20 ms
Loaded RTT = 250 ms
```

Bandwidth may remain high while interactive traffic becomes slow.

Active Queue Management (AQM) techniques such as CoDel, FQ-CoDel, and RED can help control queue growth.

## 13. Network Bottlenecks

A **bottleneck** is the component limiting end-to-end performance.

```text
Server
  |
  | 10 Gbps
  v
Switch
  |
  | 1 Gbps   ← Bottleneck
  v
Router
  |
  | 10 Gbps
  v
Client
```

Common bottlenecks include network links, routers, firewalls, load balancers, CPU, memory, disks, databases, and application servers.

## 14. Serialization Delay

**Serialization delay** is the time required to place a packet's bits onto a link.

```text
Serialization time = Packet size / Link rate
```

Example:

```text
Packet = 1,500 bytes
Link   = 100 Mbps

Time ≈ 1500 × 8 / 100,000,000
     ≈ 0.12 ms
```

Higher link speeds reduce serialization delay.

## 15. Performance Model

A simplified model is:

```text
End-to-end latency
    = Processing
    + Serialization
    + Propagation
    + Queueing
    + Protocol/application delays
```

Queueing is especially sensitive to traffic load.

## 16. TCP and Network Performance

TCP performance depends on more than link bandwidth.

```text
TCP throughput
    ↓
Congestion window
    ↓
RTT
    ↓
Packet loss
    ↓
Receive window
    ↓
Available bandwidth
```

A useful intuition is:

```text
Throughput ≈ Window / RTT
```

A high RTT can limit throughput if the sender cannot maintain enough data in flight.

## 17. Performance Optimization

Use this process:

```text
Measure
  ↓
Identify bottleneck
  ↓
Change one important variable
  ↓
Measure again
  ↓
Verify improvement
```

Common approaches:

**Reduce latency**
- Place services closer together
- Reuse connections
- Use connection pooling
- Reduce unnecessary round trips
- Cache data

**Increase throughput**
- Increase link capacity
- Increase appropriate parallelism
- Tune TCP/window behavior
- Remove bottlenecks

**Reduce packet loss**
- Fix faulty links
- Address congestion
- Check interface errors
- Improve wireless conditions

**Reduce queueing**
- Control traffic bursts
- Use appropriate queue management
- Avoid unnecessarily large buffers
- Apply traffic shaping where appropriate

## 18. Practical Commands

### Measure RTT and packet loss

```powershell
ping example.com -n 10
```

Example output:

```text
Packets: Sent = 10, Received = 10, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 18ms, Maximum = 24ms, Average = 20ms
```

Meaning:

- `0% loss` → all tested packets returned
- `Minimum` → fastest RTT
- `Maximum` → slowest RTT
- `Average` → average RTT

### Trace the network path

```powershell
tracert example.com
```

Example:

```text
  1    2 ms    1 ms    2 ms  192.168.1.1
  2   10 ms    9 ms   11 ms  10.0.0.1
  3   20 ms   19 ms   21 ms  203.0.113.1
```

Meaning:

- Each row is a routing hop
- The three times are separate probes
- A single slow intermediate hop does not automatically mean that hop is the bottleneck; later hops may also respond slowly because of routing or ICMP handling

### Test TCP connectivity

```powershell
Test-NetConnection example.com -Port 443
```

Example:

```text
ComputerName     : example.com
RemotePort       : 443
TcpTestSucceeded : True
```

Meaning: `True` means a TCP connection to port 443 succeeded.

### Check interface statistics

```powershell
Get-NetAdapterStatistics
```

Example:

```text
Name      ReceivedBytes   SentBytes   ReceivedDiscardedPackets
Ethernet  524288000       104857600   12
```

Meaning:

- `ReceivedBytes` → bytes received
- `SentBytes` → bytes transmitted
- `ReceivedDiscardedPackets` → packets discarded at the interface/stack

## 19. Performance Troubleshooting Mental Model

When an application is slow:

```text
Application
    ↓
Is the server slow?
    ↓
Is the network slow?
    ↓
    ├── High RTT?
    ├── Packet loss?
    ├── High jitter?
    ├── Queueing?
    ├── Congestion?
    └── Bottleneck?
    ↓
Is the client/resource slow?
```

Do not assume that "slow network" means low bandwidth.

For example:

```text
1 Gbps link + 200 ms RTT + many request/response round trips
```

can feel slower for interactive workloads than:

```text
100 Mbps link + 10 ms RTT
```

## 20. Summary

- **Latency** is the time required for data to travel.
- **RTT** measures round-trip delay.
- **Bandwidth** is link capacity.
- **Throughput** is actual delivered data rate.
- **Jitter** is variation in packet delay.
- **Packet loss** can cause retransmissions and reduced throughput.
- **Congestion** occurs when offered traffic exceeds capacity.
- **Queueing** adds delay as packets wait for transmission.
- **Bufferbloat** is excessive queueing caused by oversized buffers.
- **Bottlenecks** limit end-to-end performance.
- **Optimization should start with measurement and bottleneck identification.**

> **Key idea:** Network performance is not just bandwidth. Latency, RTT, loss, jitter, queueing, and bottlenecks can all determine how fast an application actually feels.

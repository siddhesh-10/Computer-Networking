# Networking — Phase 6: Routing & ICMP

> **Goal:** Understand how IP packets move between networks, how routers choose the next hop, and how ICMP tools such as `ping` and `tracert` reveal network paths.

## 1. Routing

**Routing** is deciding where an IP packet should go next.

```text
PC A → Router A → Router B → Server
```

Each router examines the **destination IP** and uses its routing table to choose the next hop.

```text
Receive packet
      ↓
Read destination IP
      ↓
Look up routing table
      ↓
Choose route
      ↓
Forward packet
```

---

## 2. Router

A **router** connects different IP networks.

```text
192.168.1.0/24
       |
     Router
       |
10.0.0.0/24
```

A router typically has an interface in each network.

Its basic job is:

```text
Destination IP
      ↓
Route lookup
      ↓
Next hop / interface
      ↓
Forward
```

---

## 3. Routing Table

A routing table contains routes describing where packets should be sent.

Example:

```text
Destination       Gateway       Interface
192.168.1.0/24    On-link       eth0
10.0.0.0/24       192.168.1.1   eth0
0.0.0.0/0         192.168.1.1   eth0
```

Meaning:

```text
192.168.1.0/24 → directly connected
10.0.0.0/24    → send to 192.168.1.1
0.0.0.0/0      → default route
```

---

## 4. Longest Prefix Match

A destination can match multiple routes.

Example:

```text
10.0.0.0/8
10.1.0.0/16
10.1.2.0/24
0.0.0.0/0
```

For:

```text
10.1.2.50
```

the router chooses:

```text
10.1.2.0/24
```

because it is the **most specific matching route**.

```text
More specific prefix
        ↓
Preferred route
```

---

## 5. Next Hop

The **next hop** is the immediate router/device to which a packet is forwarded.

Example:

```text
PC → Router A → Router B → Server
```

Router A might have:

```text
Destination = 10.0.0.0/24
Next hop    = Router B
```

Router A does not need to know the entire path. It only needs the appropriate next hop.

---

## 6. Default Route

The default IPv4 route is:

```text
0.0.0.0/0
```

It means:

> If no more specific route matches, use this route.

Example:

```text
Destination        Next Hop
192.168.1.0/24     On-link
10.0.0.0/8         Router A
0.0.0.0/0          Router A
```

A packet for:

```text
8.8.8.8
```

uses the default route if no more specific route matches.

---

## 7. Packet Routing

Suppose:

```text
Source      = 192.168.1.10
Destination = 10.0.0.20
```

Path:

```text
PC
 ↓
Router A
 ↓
Router B
 ↓
Server
```

At Router A:

```text
Destination IP
      ↓
Routing table
      ↓
Next hop = Router B
      ↓
Forward
```

At Router B:

```text
Destination IP
      ↓
Routing table
      ↓
10.0.0.0/24 is directly connected
      ↓
Forward to Server
```

Each router makes its own forwarding decision.

---

## 8. Layer 2 Changes at Each Hop

The IP packet travels through multiple networks, but the Ethernet frame is normally replaced at each router hop.

```text
PC ── Router A ── Router B ── Server
```

First link:

```text
Source MAC      = PC MAC
Destination MAC = Router A MAC
```

Next link:

```text
Source MAC      = Router A MAC
Destination MAC = Router B MAC
```

Final link:

```text
Source MAC      = Router B MAC
Destination MAC = Server MAC
```

The IP destination generally remains:

```text
10.0.0.20
```

So:

```text
MAC addresses → change at each Layer 2 hop
IP addresses  → generally remain end-to-end
```

---

## 9. TTL

**TTL (Time To Live)** prevents packets from circulating forever.

Example:

```text
TTL = 64
```

Each forwarding router decrements it:

```text
Router A: 64 → 63
Router B: 63 → 62
Router C: 62 → 61
```

When TTL reaches zero, the router discards the packet.

This prevents an accidental routing loop from keeping a packet alive indefinitely.

---

## 10. ICMP Time Exceeded

When a router discards a packet because TTL reached zero, it can send an ICMP **Time Exceeded** message to the sender.

```text
Packet
TTL = 1
 ↓
Router
 ↓
TTL → 0
 ↓
Packet discarded
 ↓
ICMP Time Exceeded
 ↓
Sender
```

Traceroute uses this behavior.

---

## 11. ICMP

**ICMP (Internet Control Message Protocol)** is used for network diagnostics, control, and error reporting.

Examples:

```text
Echo Request
Echo Reply
Time Exceeded
Destination Unreachable
```

ICMP is associated with Layer 3 and is carried inside IP.

It is **not TCP or UDP**.

---

## 12. Ping

`ping` commonly uses:

```text
ICMP Echo Request
```

and expects:

```text
ICMP Echo Reply
```

```text
PC ── Echo Request ──→ Server
PC ←── Echo Reply ──── Server
```

Run:

```powershell
ping 8.8.8.8
```

Example:

```text
Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=20ms TTL=117
Reply from 8.8.8.8: bytes=32 time=19ms TTL=117
```

Meaning:

```text
bytes=32
    ↓
ICMP payload size

time=20ms
    ↓
Approximate round-trip time

TTL=117
    ↓
TTL remaining in the reply
```

If you see:

```text
Request timed out.
```

it does not necessarily mean the destination is down. ICMP can be blocked or filtered.

---

## 13. Traceroute / Tracert

On Windows:

```powershell
tracert google.com
```

Example:

```text
Tracing route to google.com

  1    <1 ms    <1 ms    <1 ms    192.168.1.1
  2     5 ms     4 ms     5 ms    10.20.0.1
  3    12 ms    11 ms    12 ms    172.16.0.1
  4    20 ms    19 ms    21 ms    ...
```

This shows responding hops:

```text
Hop 1 → 192.168.1.1
Hop 2 → 10.20.0.1
Hop 3 → 172.16.0.1
```

The times are approximate round-trip measurements to each hop.

---

## 14. How Traceroute Uses TTL

Traceroute sends probes with increasing TTL values.

### Probe 1

```text
TTL = 1
```

First router:

```text
TTL → 0
```

It sends:

```text
ICMP Time Exceeded
```

Traceroute learns:

```text
Hop 1 = Router A
```

### Probe 2

```text
TTL = 2
```

Router A:

```text
2 → 1
```

Router B:

```text
1 → 0
```

Router B responds.

Therefore:

```text
Hop 2 = Router B
```

Then:

```text
TTL = 3
TTL = 4
TTL = 5
...
```

until the destination responds or the maximum hop count is reached.

---

## 15. Why `* * *` Appears

Example:

```text
  1    2 ms    2 ms    2 ms    192.168.1.1
  2    *       *       *       Request timed out.
  3   20 ms   19 ms   21 ms    172.16.0.1
```

`*` means a probe did not receive the expected response in time.

Possible reasons:

- Router/firewall filters the response
- ICMP is rate-limited
- Packet was lost

It does **not automatically mean that router is broken**.

---

## 16. Practical: View Routing Table

Run:

```powershell
route print
```

Example:

```text
Network Destination    Netmask          Gateway
0.0.0.0                0.0.0.0          192.168.1.1
192.168.1.0            255.255.255.0    On-link
```

Meaning:

```text
0.0.0.0/0
    ↓
Default route
    ↓
192.168.1.1
```

and:

```text
192.168.1.0/24
    ↓
Local network
    ↓
On-link
```

---

## 17. Practical: Ping the Gateway

Run:

```powershell
ping 192.168.1.1
```

Example:

```text
Reply from 192.168.1.1: bytes=32 time<1ms TTL=64
Reply from 192.168.1.1: bytes=32 time<1ms TTL=64
```

This tests connectivity between your machine and its gateway.

If this works but an Internet destination does not, the problem may be farther along the path.

---

## 18. Practical: Trace a Destination

Run:

```powershell
tracert 8.8.8.8
```

Example:

```text
Tracing route to 8.8.8.8

  1    <1 ms    <1 ms    <1 ms    192.168.1.1
  2     5 ms     5 ms     4 ms    10.20.0.1
  3    12 ms    11 ms    13 ms    ...
```

Think of the result as:

```text
Your PC
  ↓
Gateway
  ↓
Next router
  ↓
Next router
  ↓
...
  ↓
Destination
```

---

## 19. Routing vs Forwarding

### Routing

Deciding:

```text
Which route should be used?
```

### Forwarding

Actually sending the packet using that route.

```text
Routing table
      ↓
Route selection
      ↓
Forwarding decision
      ↓
Output interface
      ↓
Next hop
```

---

## 20. Key Mental Model

For every router:

```text
Incoming IP packet
        ↓
Read destination IP
        ↓
Routing table
        ↓
Longest-prefix match
        ↓
Choose next hop/interface
        ↓
Decrement TTL
        ↓
Create Layer 2 frame
        ↓
Forward
```

For diagnostics:

```text
ping
 ↓
ICMP Echo Request/Reply
 ↓
Tests reachability + RTT

tracert
 ↓
Increasing TTL
 ↓
ICMP Time Exceeded
 ↓
Reveals responding hops
```

---

## 21. Summary

```text
Routing
 ↓
Choosing where an IP packet should go

Router
 ↓
Connects different IP networks

Routing table
 ↓
Contains routes

Next hop
 ↓
Immediate forwarding destination

Default route
 ↓
0.0.0.0/0

TTL
 ↓
Decreases at each router
 ↓
Prevents endless routing loops

ICMP
 ↓
Network diagnostics and control

Ping
 ↓
ICMP Echo Request/Reply

Traceroute
 ↓
Uses TTL + ICMP responses to discover the path
```

Core flow:

```text
Destination IP
      ↓
Routing table
      ↓
Longest-prefix match
      ↓
Next hop
      ↓
Decrement TTL
      ↓
Forward
      ↓
Next router
      ↓
Repeat
```

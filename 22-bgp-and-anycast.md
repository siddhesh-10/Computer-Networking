# Networking — Phase 22: BGP & Anycast

> **Goal:** Understand how networks exchange routes across the Internet using BGP, how BGP selects routes, and how Anycast uses the same IP prefix from multiple locations.

## 1. Where BGP Fits

**BGP (Border Gateway Protocol)** is a routing protocol used to exchange reachability information between Autonomous Systems.

```text
BGP
 ↓
TCP port 179
 ↓
IP
```

Its basic message is:

```text
"I can reach this IP prefix."
```

Example:

```text
AS 64510
   |
   | BGP advertisement
   | 203.0.113.0/24
   ↓
AS 64520
```

BGP carries routing information, not application data.

---

## 2. Why BGP Is Needed

The Internet contains many independently operated networks:

```text
AS 100
AS 200
AS 300
AS 400
...
```

BGP allows these networks to exchange reachability information.

```text
AS 100
   ↕
  BGP
   ↕
AS 200
```

BGP is the primary **inter-domain routing protocol** of the Internet.

---

## 3. Route Advertisement

Suppose AS 64510 owns:

```text
203.0.113.0/24
```

It can advertise that prefix to another AS:

```text
AS 64510
   |
   | "203.0.113.0/24 is reachable through me"
   ↓
AS 64520
```

AS 64520 may then advertise the route onward:

```text
AS 64530
   ↓
AS 64520
   ↓
AS 64510
   ↓
203.0.113.0/24
```

---

## 4. Important BGP Attributes

Important attributes for this phase:

```text
AS_PATH
NEXT_HOP
LOCAL_PREF
MED
```

They help routers answer:

```text
"Which route should I use?"
```

---

## 5. AS_PATH

**AS_PATH** records the Autonomous Systems through which a route has passed.

Example:

```text
Destination: 203.0.113.0/24

AS_PATH:
64510 64520 64530
```

A shorter AS path is often preferred, but it is **not automatically the first or most important criterion**.

### Loop Prevention

If an AS receives a route containing its own ASN:

```text
64510 64520 64530 64510
```

it can reject the route because it indicates a routing loop.

---

## 6. NEXT_HOP

`NEXT_HOP` tells the router where traffic should be forwarded for the advertised prefix.

Example:

```text
Destination:
203.0.113.0/24

NEXT_HOP:
192.0.2.1
```

```text
203.0.113.0/24
        ↓
192.0.2.1
        ↓
Forward traffic
```

> `NEXT_HOP` is the next routing destination, not necessarily the final server.

---

## 7. LOCAL_PREF

**LOCAL_PREF (Local Preference)** is used inside an AS to influence which route should be preferred for outbound traffic.

Higher `LOCAL_PREF` is normally preferred.

Example:

```text
Route A → LOCAL_PREF 200
Route B → LOCAL_PREF 100
```

Route A is normally preferred.

Common use:

```text
Prefer ISP A over ISP B
Control outbound traffic
```

Mental model:

```text
LOCAL_PREF
"I prefer leaving through this path."
```

---

## 8. MED

**MED (Multi-Exit Discriminator)** can influence which entry point another AS should prefer when multiple links exist.

Example:

```text
             ISP
           /           Link A     Link B
       MED 50    MED 100
           \     /
            Your AS
```

Lower MED is generally preferred when comparing applicable routes.

Mental model:

```text
MED
"I would prefer you enter through this link."
```

MED is commonly used to influence inbound traffic, but actual behavior depends on routing policy.

---

## 9. LOCAL_PREF vs MED

| Attribute | Main purpose | Preference |
|---|---|---|
| LOCAL_PREF | Influence outbound path inside an AS | Higher |
| MED | Suggest preferred entry point to a neighboring AS | Lower |

These are simplified mental models; real BGP policy can override or combine multiple attributes.

---

## 10. BGP Route Selection

A router may receive multiple routes to the same prefix.

Example:

```text
203.0.113.0/24

Route A
LOCAL_PREF = 200
AS_PATH = 3 hops

Route B
LOCAL_PREF = 100
AS_PATH = 2 hops
```

Route A is normally preferred because `LOCAL_PREF` is considered before AS path length.

Simplified mental model:

```text
Candidate routes
      ↓
BGP policy/attributes
      ↓
LOCAL_PREF
      ↓
AS_PATH
      ↓
MED
      ↓
Other tie-breakers
      ↓
Best route
```

The exact decision process depends on the implementation and configuration.

---

## 11. eBGP

**eBGP (External BGP)** runs between different Autonomous Systems.

```text
AS 64510
   |
  eBGP
   |
AS 64520
```

Examples:

```text
ISP ↔ ISP
Enterprise ↔ ISP
Cloud provider ↔ ISP
Content provider ↔ ISP
```

---

## 12. iBGP

**iBGP (Internal BGP)** runs between routers inside the same Autonomous System.

```text
             AS 64510
          /                   Router A         Router B
          \             /
             iBGP
```

It can distribute BGP-learned routes internally.

Important:

```text
eBGP
 ↓
Different ASes

iBGP
 ↓
Same AS
```

---

## 13. eBGP vs iBGP

| | eBGP | iBGP |
|---|---|---|
| AS relationship | Different ASes | Same AS |
| Main purpose | Exchange routes between networks | Distribute BGP routes inside an AS |
| Example | ISP ↔ ISP | Router ↔ Router inside ISP |

---

## 14. BGP Route Propagation

Suppose:

```text
AS 64510 owns:
203.0.113.0/24
```

It advertises the route:

```text
AS 64510
   ↓
AS 64520
```

AS 64520 can advertise it onward:

```text
AS 64530
   ↓
AS 64520
   ↓
AS 64510
```

The AS path grows as the route crosses AS boundaries.

---

## 15. Anycast

**Anycast** allows the same IP address/prefix to be announced from multiple network locations.

Example:

```text
Location A → 203.0.113.10
Location B → 203.0.113.10
Location C → 203.0.113.10
```

Conceptually:

```text
              Same IP
             /   |               ↓    ↓    ↓
         Site A Site B Site C
```

Routing determines which location receives traffic.

---

## 16. How Anycast Works

Suppose three sites advertise:

```text
203.0.113.0/24
```

```text
Site A → advertises prefix
Site B → advertises prefix
Site C → advertises prefix
```

A user's network may learn multiple paths.

BGP routing then selects a path according to routing policy and topology.

> Anycast does **not** guarantee the geographically closest server. The selected path is determined by routing decisions.

---

## 17. Why Anycast Is Useful

Suppose a service exists in:

```text
Asia
Europe
USA
```

All locations advertise the same service IP.

Traffic may look like:

```text
User in Asia   → Asia site
User in Europe → Europe site
User in USA    → USA site
```

Potential benefits:

```text
Lower latency
Traffic distribution
Redundancy
Regional failover
```

---

## 18. Anycast for DNS

Anycast is commonly used for DNS infrastructure.

```text
             DNS IP
                |
       ┌────────┼────────┐
       ↓        ↓        ↓
     Site A   Site B   Site C
```

The client sends its query to the same IP address.

Routing determines which site receives it.

Benefits:

```text
Lower latency
Redundancy
Traffic distribution
Resilience
```

---

## 19. Anycast and Failure

Suppose:

```text
Site A → healthy
Site B → healthy
Site C → failed
```

Site C can withdraw its route:

```text
Site C
   ↓
Route withdrawal
```

Traffic can then converge toward another site.

```text
Before:
User → Site C

After:
User → Site A / Site B
```

BGP convergence takes time, so failover is not necessarily instantaneous.

---

## 20. Global Traffic Routing

Large Internet services can combine:

```text
DNS
CDN
Anycast
BGP
Load balancing
Health checks
```

Example:

```text
                    Global Service
                         |
              Same prefix advertised
                         |
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Region A        Region B       Region C
          |              |              |
       Load Balancer  Load Balancer  Load Balancer
          |              |              |
        Servers        Servers        Servers
```

BGP can influence which region receives traffic.

The regional load balancer then distributes traffic among local servers.

---

## 21. Anycast vs DNS Routing

### Anycast

```text
Same IP
   ↓
BGP
   ↓
Selected network location
```

### DNS-based routing

```text
Client
   ↓
DNS query
   ↓
DNS system chooses IP
   ↓
Client connects to that IP
```

DNS routing can make decisions using:

```text
Geography
Latency
Health
Policy
```

Anycast works at the routing level; DNS routing works at the name-resolution level.

They can also be combined.

---

## 22. Practical: View BGP Information

On a Linux router running a routing suite such as FRRouting:

```bash
vtysh -c "show ip bgp"
```

Example:

```text
Network          Next Hop        Metric LocPrf Weight Path
*> 203.0.113.0/24 192.0.2.1          0    200      0 64510 i
```

Meaning:

```text
203.0.113.0/24
 ↓
Destination prefix

192.0.2.1
 ↓
NEXT_HOP

200
 ↓
LOCAL_PREF

64510
 ↓
AS_PATH
```

`*>` commonly indicates that the route is valid and selected/best in FRRouting's display.

---

## 23. Practical: View the Network Path

Windows:

```powershell
tracert example.com
```

Example:

```text
Tracing route to example.com

  1    <1 ms    <1 ms    <1 ms    192.168.1.1
  2     5 ms     6 ms     5 ms    10.10.0.1
  3    12 ms    11 ms    13 ms    ...
```

Meaning:

```text
Hop 1
 ↓
Usually local router

Hop 2+
 ↓
Additional routers along the path
```

`tracert` shows the observed IP path, not the complete set of BGP attributes.

---

## 24. BGP Mental Model

```text
                 Internet
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      AS A        AS B        AS C
        \           |           /
         \         BGP         /
          └─────────┼─────────┘
                    ↓
              Route selection
                    ↓
                 Best path
```

Route advertisement:

```text
Prefix
  ↓
BGP advertisement
  ↓
AS_PATH
NEXT_HOP
LOCAL_PREF
MED
  ↓
Route selection
  ↓
Forward traffic
```

Anycast:

```text
             Same IP prefix
              /     |                  ↓      ↓      ↓
          Site A  Site B  Site C
             \      |      /
              BGP routing
                   ↓
             User reaches
             one location
```

---

## 25. Summary

```text
BGP
 ↓
Inter-AS routing protocol

AS_PATH
 ↓
Records ASes in the route path
 + helps prevent loops

NEXT_HOP
 ↓
Indicates the next routing destination

LOCAL_PREF
 ↓
Influences outbound path selection
 + higher is preferred

MED
 ↓
Can influence inbound path selection
 + lower is generally preferred

Route advertisement
 ↓
"I can reach this prefix"

Route selection
 ↓
Choose the preferred route

eBGP
 ↓
BGP between different ASes

iBGP
 ↓
BGP within the same AS

Anycast
 ↓
Same IP prefix announced from multiple locations

Global traffic routing
 ↓
BGP + Anycast + DNS + CDN + load balancing
```

> **Key idea:** BGP tells networks which IP prefixes are reachable and helps them choose paths; Anycast uses the same routing system to make one service address reachable from multiple locations.

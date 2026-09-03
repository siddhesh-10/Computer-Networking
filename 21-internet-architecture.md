# Networking --- Phase 21: Internet Architecture

> **Goal:** Understand how the global Internet is organized, how ISPs
> and Autonomous Systems connect, and how traffic moves through transit
> providers, peering links, Internet Exchange Points, and data centers.

## 1. What Is the Internet?

The Internet is a **network of interconnected networks**.

It is not one single network owned by one organization.

``` text
                 Global Internet
                       |
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
     ISP A           ISP B           ISP C
       |               |               |
      AS              AS              AS
       |               |               |
   Customers       Customers       Customers
```

Different organizations operate their own networks and connect them
together.

These independently operated networks are commonly represented by
**Autonomous Systems (ASes)**.

------------------------------------------------------------------------

## 2. ISP

**ISP (Internet Service Provider)** provides Internet connectivity to
customers.

Examples:

``` text
Home users
Businesses
Cloud providers
Universities
Data centers
Other ISPs
```

Simplified:

``` text
Customer
   ↓
ISP
   ↓
Internet
```

An ISP may operate:

``` text
Access networks
Routers
Data centers
Fiber links
International links
Peering connections
Transit connections
```

------------------------------------------------------------------------

## 3. Autonomous System

An **Autonomous System (AS)** is a network or group of networks operated
under a common routing policy.

Examples:

``` text
Large ISP
Cloud provider
Content provider
Enterprise
University
```

Conceptually:

``` text
AS 100
 ├── Network A
 ├── Network B
 └── Network C
```

The AS presents routes to other networks and applies its own routing
policies.

------------------------------------------------------------------------

## 4. AS Number

An **ASN (Autonomous System Number)** identifies an Autonomous System
for Internet routing.

Example:

``` text
AS64500
```

ASNs are used by **BGP (Border Gateway Protocol)** to exchange routing
information between autonomous systems.

``` text
AS 64500
    ↕
   BGP
    ↕
AS 64501
```

The actual Internet contains many thousands of autonomous systems.

------------------------------------------------------------------------

## 5. BGP and the Internet

**BGP** is the primary inter-domain routing protocol used to exchange
reachability information between Autonomous Systems.

Example:

``` text
AS 100
  |
  | "I can reach 203.0.113.0/24"
  |
AS 200
```

AS 200 can learn that traffic for:

``` text
203.0.113.0/24
```

can be sent toward AS 100.

BGP is **policy-driven**, not simply "choose the shortest physical
path."

Policies can consider:

``` text
AS path
Local preference
MED
Communities
Provider/customer relationships
Traffic engineering
```

------------------------------------------------------------------------

## 6. Internet Backbone

The **Internet backbone** consists of high-capacity networks and links
that carry large amounts of traffic across regions and countries.

``` text
Local ISP
    ↓
Regional Network
    ↓
Backbone
    ↓
Regional Network
    ↓
Destination ISP
```

Backbone infrastructure can include:

``` text
Long-distance fiber
Submarine cables
High-capacity routers
Major data centers
Carrier networks
```

There is not one single physical "Internet backbone."

Many large networks interconnect to form the global Internet.

------------------------------------------------------------------------

## 7. Transit

**IP transit** is a connectivity relationship where one network provides
access to other networks.

``` text
Customer AS
     ↓
Transit Provider
     ↓
Rest of Internet
```

The customer typically pays the transit provider.

Conceptually:

``` text
AS A
 ↓
Transit Provider
 ↓
AS B / AS C / AS D / ...
```

Transit providers can provide broad Internet reachability.

------------------------------------------------------------------------

## 8. Peering

**Peering** is when two networks directly exchange traffic.

``` text
ISP A
  ↕
Peering
  ↕
ISP B
```

Instead of:

``` text
ISP A
  ↓
Transit Provider
  ↓
ISP B
```

the networks can exchange traffic directly.

Potential benefits:

``` text
Lower latency
Lower transit cost
Better traffic control
Reduced transit congestion
```

Peering can be:

``` text
Private peering
Public peering
```

------------------------------------------------------------------------

## 9. Transit vs Peering

``` text
  -----------------------------------------------------------------------
                          Transit                 Peering
  ----------------------- ----------------------- -----------------------
  Relationship            Provider/customer       Usually peer-to-peer

  Purpose                 Reach broader Internet  Exchange traffic
                                                  between networks

  Payment                 Usually customer pays   Often settlement-free,
                          provider                but not always

  Scope                   Broad Internet          Usually specific
                          reachability            routes/traffic
  -----------------------------------------------------------------------
```

Simplified:

``` text
Transit:
"I pay you to reach the Internet."

Peering:
"We connect directly to exchange our traffic."
```

Commercial agreements can be more complex.

------------------------------------------------------------------------

## 10. Internet Exchange Point

An **IXP (Internet Exchange Point)** is a facility where multiple
networks interconnect to exchange traffic.

``` text
              IXP
        ┌──────┼──────┐
        ↓      ↓      ↓
      ISP A  ISP B  ISP C
```

An IXP commonly provides:

``` text
Shared switching infrastructure
Colocation
Peering connectivity
```

Participants establish peering sessions, commonly using BGP, across the
exchange fabric.

------------------------------------------------------------------------

## 11. Public Peering

At an IXP:

``` text
             IXP Switch
          /      |               /       |             ISP A    ISP B    ISP C
```

ISP A can establish a BGP peering session with ISP B.

Traffic can then flow:

``` text
ISP A → IXP → ISP B
```

instead of:

``` text
ISP A → Transit → ISP B
```

The IXP provides shared interconnection infrastructure; participants
decide which routes to exchange.

------------------------------------------------------------------------

## 12. Private Peering

Two networks can connect directly without using a shared public exchange
fabric.

``` text
AS A
 |
 | Direct link
 |
AS B
```

This is common between networks that exchange significant amounts of
traffic.

Examples:

``` text
Large ISP ↔ Content provider
Cloud provider ↔ ISP
Content network ↔ ISP
```

------------------------------------------------------------------------

## 13. Data Centers

A **data center** is a facility containing computing and networking
infrastructure.

Typical components:

``` text
Servers
Storage
Switches
Routers
Firewalls
Load balancers
Power systems
Cooling systems
```

Simplified:

``` text
             Internet
                |
             Routers
                |
             Switches
          /      |             Server  Server  Server
```

Data centers host:

``` text
Applications
Databases
APIs
CDNs
Cloud services
Network infrastructure
```

------------------------------------------------------------------------

## 14. Data Center Connectivity

A large data center may connect to multiple networks:

``` text
                Data Center
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        ISP A      ISP B      IXP
          |          |          |
       Internet   Internet   Peering
```

Multiple connections can provide:

``` text
Redundancy
Higher capacity
Better routing options
Lower latency
Provider diversity
```

------------------------------------------------------------------------

## 15. Global Internet Architecture

A simplified view:

``` text
                    Global Internet
                          |
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
     Region A          Region B          Region C
        |                 |                 |
      ISPs              ISPs              ISPs
        |                 |                 |
     Networks          Networks          Networks
        \                 |                 /
         \                |                /
              Transit / Peering
                     |
                   IXPs
                     |
               Data Centers
                     |
                  Services
```

The real Internet is much more interconnected and does not follow a
strict hierarchy.

------------------------------------------------------------------------

## 16. How a Packet Can Cross the Internet

Suppose:

``` text
User
 ↓
Local ISP
 ↓
Regional network
 ↓
Transit / Peering
 ↓
Destination network
 ↓
Data center
 ↓
Server
```

The packet may cross several autonomous systems:

``` text
AS 64510
   ↓
AS 64520
   ↓
AS 64530
   ↓
AS 64540
```

The exact path depends on:

``` text
BGP routing
Routing policy
Peering
Transit
Network failures
Traffic engineering
```

------------------------------------------------------------------------

## 17. AS Path

BGP can advertise routes with an **AS path**.

Example:

``` text
Destination: 203.0.113.0/24

AS Path:
64530 64520 64510
```

Conceptually:

``` text
Your AS
  ↓
64520
  ↓
64530
  ↓
Destination
```

The AS path helps BGP understand the route through autonomous systems
and helps prevent routing loops.

------------------------------------------------------------------------

## 18. Customer, Provider and Peer Relationships

A simplified model:

``` text
          Transit Provider
             /                   /                 Customer     Customer
```

Another relationship:

``` text
ISP A
  ↕
Peer
  ↕
ISP B
```

A network may simultaneously have:

``` text
Customers
Providers
Peers
```

Example:

``` text
              Transit Provider
                ↑        ↑
                |        |
              ISP A ─── ISP B
                 Peering
```

------------------------------------------------------------------------

## 19. Why Peering Matters for CDN Traffic

Suppose a user requests content from a CDN.

Without direct interconnection:

``` text
User
 ↓
ISP
 ↓
Transit
 ↓
CDN
```

With peering:

``` text
User
 ↓
ISP
 ↓
IXP / Private Peering
 ↓
CDN
```

Peering can reduce:

``` text
Path length
Latency
Transit usage
```

Large content networks therefore build extensive interconnection
networks.

------------------------------------------------------------------------

## 20. Practical: View Your Network Route

Windows:

``` powershell
tracert example.com
```

Example:

``` text
Tracing route to example.com

  1    <1 ms    <1 ms    <1 ms    192.168.1.1
  2     5 ms     6 ms     5 ms    10.10.0.1
  3    12 ms    11 ms    13 ms    ...
  ...
```

Meaning:

``` text
Hop 1
 ↓
Usually local router

Hop 2+
 ↓
Additional routers along the path
```

Some hops may show:

``` text
Request timed out.
```

This does not automatically mean the route is broken. Routers can filter
or deprioritize traceroute responses.

------------------------------------------------------------------------

## 21. Practical: View Route Table

Run:

``` powershell
route print
```

Example:

``` text
IPv4 Route Table

Network Destination    Netmask        Gateway
0.0.0.0                0.0.0.0        192.168.1.1
192.168.1.0            255.255.255.0  On-link
```

Meaning:

``` text
0.0.0.0 / 0
 ↓
Default route

192.168.1.0/24
 ↓
Local network
```

This shows your host's local routing decisions, not the entire Internet
routing table.

------------------------------------------------------------------------

## 22. Practical: Resolve a Host

Run:

``` powershell
Resolve-DnsName example.com
```

Example:

``` text
Name       Type  IPAddress
----       ----  ---------
example.com A     93.184.216.34
```

Meaning:

``` text
DNS name
   ↓
IP address
```

DNS resolution happens before the host can normally establish a
connection to the destination.

------------------------------------------------------------------------

## 23. Internet Architecture Mental Model

``` text
                    Internet
                       |
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
       ISP            ISP            ISP
        |              |              |
       AS             AS             AS
        \              |             /
         \             |            /
          \        Peering /        /
           \         Transit       /
            \          |          /
                    IXPs
                      |
                Data Centers
                      |
                  Services
```

At a higher level:

``` text
Users
  ↓
Access ISP
  ↓
ISP / Regional AS
  ↓
Peering or Transit
  ↓
Destination AS
  ↓
Data Center
  ↓
Server / Service
```

------------------------------------------------------------------------

## 24. Summary

``` text
ISP
 ↓
Provides Internet connectivity

Autonomous System
 ↓
Network operated under a common routing policy

ASN
 ↓
Identifier for an Autonomous System

BGP
 ↓
Exchanges reachability information between ASes

Internet backbone
 ↓
High-capacity interconnected networks

Transit
 ↓
Connectivity to broader networks

Peering
 ↓
Direct traffic exchange between networks

IXP
 ↓
Shared infrastructure for network interconnection

Data center
 ↓
Facility hosting compute/network infrastructure
```

Core idea:

> The Internet is a **network of independently operated networks** that
> interconnect through **transit, peering, and exchange points**, with
> BGP helping those networks determine how to reach each other's IP
> prefixes.

Simplified end-to-end path:

``` text
User
 ↓
Access ISP
 ↓
Autonomous System
 ↓
Peering / Transit / IXP
 ↓
Destination Autonomous System
 ↓
Data Center
 ↓
Server
```

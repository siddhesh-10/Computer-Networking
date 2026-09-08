# Networking --- Phase 23: Advanced IP Networking

> **Goal:** Understand IPv6, packet size limits, fragmentation, Path MTU
> Discovery, ECMP, and jumbo frames.

## 1. IPv6

**IPv6 (Internet Protocol version 6)** is a network-layer protocol used
to identify and route packets across IP networks.

-   **Layer:** Network layer (Layer 3)
-   **Address size:** 128 bits
-   **Base header:** 40 bytes
-   **Transport above it:** TCP, UDP, or other IP-aware transport
    protocols
-   **Purpose:** Global addressing and packet routing

``` text
Application
    ↓
TCP / UDP
    ↓
IPv6
    ↓
Ethernet / Wi-Fi
    ↓
Physical network
```

### IPv4 vs IPv6

  Feature         IPv4              IPv6
  --------------- ----------------- ------------------
  Address         32-bit            128-bit
  Example         `192.0.2.10`      `2001:db8::10`
  Broadcast       Yes               No
  Multicast       Yes               Yes
  Fragmentation   Hosts + routers   Source host only
  Base header     20+ bytes         40 bytes

------------------------------------------------------------------------

## 2. IPv6 Addressing

An IPv6 address has **128 bits**, written as eight hexadecimal groups.

``` text
2001:0db8:0000:0000:0000:0000:0000:0010
```

Compressed:

``` text
2001:db8::10
```

### Common address types

  Type        Meaning
  ----------- ---------------------------------------------------
  Unicast     One interface
  Multicast   Multiple interfaces
  Anycast     One of multiple interfaces using the same address
  `::1`       Loopback
  `::`        Unspecified address

### Prefix notation

``` text
2001:db8:1234:5678::/64
```

`/64` means the first 64 bits are the network prefix.

------------------------------------------------------------------------

## 3. MTU

**MTU (Maximum Transmission Unit)** is the largest IP packet a link can
carry without needing fragmentation at that link.

-   **Layer:** Network/link boundary
-   Typical Ethernet MTU: **1500 bytes**
-   Jumbo-frame environments commonly use an MTU around **9000 bytes**

``` text
Host A
  |
  | MTU 1500
  v
Router
  |
  | MTU 1400
  v
Host B

Path MTU = 1400
```

The end-to-end path is constrained by the **smallest MTU**.

------------------------------------------------------------------------

## 4. MSS

**MSS (Maximum Segment Size)** is the largest TCP payload a host
advertises for one TCP segment.

-   **Layer:** Transport layer
-   Applies to **TCP**, not UDP
-   MSS describes payload; MTU describes the IP packet size

For typical IPv4 Ethernet:

``` text
MTU = 1500
IPv4 header = 20
TCP header = 20

MSS = 1500 - 20 - 20
    = 1460 bytes
```

For IPv6:

``` text
MTU = 1500
IPv6 base header = 40
TCP header = 20

MSS = 1440 bytes
```

TCP options can reduce usable payload.

------------------------------------------------------------------------

## 5. MTU vs MSS

  Concept                    MTU                     MSS
  -------------------------- ----------------------- ------------------------
  Layer                      Network/link boundary   Transport
  Applies to                 IP packet               TCP payload
  Includes IP/TCP headers?   Yes                     No
  Typical example            1500 bytes              1460 bytes
  Purpose                    Packet-size limit       TCP payload-size limit

``` text
IP packet
+-------------------------------+
| IP header | TCP | TCP payload |
+-------------------------------+
<----------- MTU -------------->

                 <--- MSS --->
```

------------------------------------------------------------------------

## 6. IP Fragmentation

Fragmentation happens when an IP packet is larger than a link can carry.

### IPv4

IPv4 routers can fragment packets when fragmentation is permitted.

``` text
Large IPv4 packet
        ↓
     Router
        ↓
  +-----+-----+-----+
  | F1  | F2  | F3  |
  +-----+-----+-----+
        ↓
    Destination
        ↓
    Reassembly
```

### IPv6

IPv6 routers **do not fragment packets**.

If a packet is too large, the sender must send smaller packets.

IPv6 can use a **Fragment extension header**, but fragmentation is
performed by the source rather than intermediate routers.

------------------------------------------------------------------------

## 7. Path MTU Discovery

**Path MTU Discovery (PMTUD)** determines the largest packet that can
travel from source to destination without fragmentation.

``` text
Host A
  |
  | 1500
  v
Router
  |
  | 1400
  v
Router
  |
  | 1500
  v
Host B

Path MTU = 1400
```

### IPv4 PMTUD

A sender can set the **Don't Fragment (DF)** bit.

If a router cannot forward an oversized packet, it can send an ICMP
message indicating that the packet is too large and, where supported,
the next-hop MTU.

### IPv6 PMTUD

IPv6 relies on ICMPv6 **Packet Too Big** messages.

The sender then reduces its packet size.

### Why PMTUD matters

Without working PMTUD, a connection can establish successfully but fail
when larger packets are sent.

------------------------------------------------------------------------

## 8. PMTUD and TCP

TCP learns MSS during the handshake.

``` text
Client                         Server
  | ---- SYN, MSS=1460 ------> |
  | <--- SYN-ACK, MSS=1460 --- |
  |                            |
  | ===== TCP data ===========>|
```

MSS helps TCP avoid creating IP packets that are too large.

However, advertised MSS is not a complete guarantee of the path MTU.
Tunnels can add overhead, paths can change, and another link may have a
smaller MTU.

------------------------------------------------------------------------

## 9. ECMP

**ECMP (Equal-Cost Multi-Path)** allows a router to use multiple paths
with the same routing cost.

-   **Layer:** Network layer
-   Routing protocols such as OSPF, IS-IS, and BGP can provide multiple
    eligible paths
-   The router selects among equal-cost next hops

``` text
             +--> Router B --+
Host A --> R1                --> Destination
             +--> Router C --+
```

### Benefits

-   Higher aggregate capacity
-   Redundancy
-   Better link utilization
-   Common in data-center networks

------------------------------------------------------------------------

## 10. ECMP Hashing

Routers commonly hash flow fields to choose a next hop.

Typical inputs:

``` text
Source IP
Destination IP
Source port
Destination port
Protocol
```

Conceptually:

``` text
Flow
 ↓
Hash
 ↓
Next-hop selection
```

Keeping a flow on one path reduces packet reordering.

------------------------------------------------------------------------

## 11. ECMP vs Load Balancing

  ECMP                    Load Balancer
  ----------------------- ---------------------------------------
  Routing mechanism       Traffic distribution system
  Usually Layer 3         Often Layer 4 or Layer 7
  Chooses next hop/path   Chooses backend/server
  Uses routing topology   Can use health and application policy

------------------------------------------------------------------------

## 12. Jumbo Frames

A **jumbo frame** is an Ethernet frame larger than the traditional
1500-byte MTU.

A common configuration is:

``` text
MTU ≈ 9000 bytes
```

They are useful in controlled environments such as data centers and
storage networks.

### Benefits

-   More application data per packet
-   Fewer packets for large transfers
-   Lower packet-processing overhead

### Trade-off

Every relevant device and link must support the chosen frame size.

``` text
Host → Switch → Router → Switch → Host
        9000     1500

          ↓

9000-byte packets cannot cross unchanged
```

------------------------------------------------------------------------

## 13. Jumbo Frames and MTU

Jumbo frames are an **end-to-end configuration concern**.

If one device uses:

``` text
MTU = 9000
```

but a later device only supports:

``` text
MTU = 1500
```

the path cannot carry 9000-byte packets unchanged.

This is why jumbo frames are normally configured consistently across the
relevant network segment.

------------------------------------------------------------------------

## 14. Practical Commands

### Check IPv4/IPv6 configuration

``` powershell
ipconfig
```

Example output:

``` text
Ethernet adapter Ethernet:

   IPv4 Address. . . . . . : 192.168.1.20
   IPv6 Address. . . . . . : 2001:db8::20
   Default Gateway . . . . : 192.168.1.1
```

Meaning:

-   `IPv4 Address` → IPv4 address assigned to the interface
-   `IPv6 Address` → IPv6 address assigned to the interface
-   `Default Gateway` → router used for non-local destinations

### Test IPv6 connectivity

``` powershell
ping -6 example.com
```

Example output:

``` text
Reply from 2001:db8::10: time=18ms
Reply from 2001:db8::10: time=17ms
```

Meaning:

-   `-6` forces IPv6
-   Replies confirm IPv6 connectivity
-   `time` is the measured round-trip time

### Inspect routing table

``` powershell
route print
```

Example:

``` text
IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
 12    256  ::/0                   fe80::1
 12    256  2001:db8::/64          On-link
```

Meaning:

-   `::/0` → default IPv6 route
-   `fe80::1` → next-hop gateway
-   `2001:db8::/64` → directly connected IPv6 network

### Test IPv4 packet size

``` powershell
ping 192.168.1.1 -f -l 1472
```

Example output:

``` text
Reply from 192.168.1.1: bytes=1472 time<1ms TTL=64
```

For IPv4:

``` text
1472 ICMP payload
+ 8 ICMP header
+ 20 IPv4 header
= 1500 bytes
```

If the packet is too large:

``` text
Packet needs to be fragmented but DF set.
```

This indicates that the tested packet does not fit without
fragmentation.

------------------------------------------------------------------------

## 15. Mental Model

``` text
IPv6
  │
  ├── 128-bit addresses
  ├── No router fragmentation
  └── ICMPv6 provides PMTUD feedback

MTU
  │
  └── Maximum IP packet size on a link

MSS
  │
  └── Maximum TCP payload

PMTUD
  │
  └── Finds the usable MTU across the path

ECMP
  │
  └── Uses multiple equal-cost next hops

Jumbo Frames
  │
  └── Larger Ethernet frames, commonly ~9000-byte MTU
```

## 16. Summary

-   **IPv6** provides 128-bit addressing and modern IP routing.
-   **IPv6 routers do not fragment packets.**
-   **MTU** is the maximum packet size a link can carry.
-   **MSS** is the maximum TCP payload size.
-   **PMTUD** discovers the usable path MTU.
-   **ECMP** distributes flows across equal-cost routing paths.
-   **Jumbo frames** allow larger Ethernet frames but require compatible
    path configuration.

> **Key idea:** Packet size is constrained by the path, TCP controls its
> payload with MSS, PMTUD discovers the usable path size, and ECMP
> provides multiple equal-cost network paths.

# Networking — Phase 5: IP Addressing & Subnetting

> **Goal:** Understand how IP addresses identify networks and hosts, how subnetting divides networks, and how a host decides whether a destination is local or remote.

## 1. IPv4

IPv4 uses a **32-bit address**.

Example:

```text
192.168.1.10
```

It contains four 8-bit octets:

```text
192 . 168 . 1 . 10
 8     8    8    8  = 32 bits
```

An IPv4 address has a logical:

```text
Network portion | Host portion
```

The subnet mask or CIDR prefix determines the boundary.

---

## 2. IPv4 Example

Consider:

```text
192.168.1.10/24
```

`/24` means:

```text
24 bits → Network
8 bits  → Host
```

Therefore:

```text
Network = 192.168.1.0
Host    = 10
```

---

## 3. Private and Public IPs

### Private IPv4 ranges

Private addresses are used inside networks and are not directly Internet-routable.

| Range | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | `10.0.0.0/8` |
| 172.16.0.0 – 172.31.255.255 | `172.16.0.0/12` |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` |

Example:

```text
192.168.1.10
```

### Public IP

A public IP is globally routable on the Internet.

Typical home network:

```text
PC
192.168.1.10
    |
    ↓
Router
Private: 192.168.1.1
Public : 203.x.x.x
    |
    ↓
Internet
```

Multiple private devices can share the public IP using NAT.

---

## 4. Loopback

Loopback refers to the local machine.

IPv4 loopback:

```text
127.0.0.0/8
```

Most commonly:

```text
127.0.0.1
```

Run:

```powershell
ping 127.0.0.1
```

Example:

```text
Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
```

Meaning:

```text
Your machine → Your machine
```

IPv6 loopback is:

```text
::1
```

---

## 5. IPv6 Basics

IPv6 uses **128-bit addresses**, compared with IPv4's 32 bits.

Example:

```text
2001:db8:1234:5678::10
```

IPv6 uses hexadecimal groups separated by `:`.

Zero groups can be compressed:

```text
2001:0db8:0000:0000:0000:0000:0000:0010
```

becomes:

```text
2001:db8::10
```

Important addresses:

```text
::1       → Loopback
::        → Unspecified
fe80::/10 → Link-local
```

IPv6 does **not** use broadcast; multicast is used instead.

---

## 6. CIDR

**CIDR (Classless Inter-Domain Routing)** represents a network as:

```text
IP/prefix-length
```

Example:

```text
192.168.1.0/24
```

`/24` means:

```text
24 bits = network
8 bits  = host
```

Common IPv4 sizes:

| CIDR | Total addresses |
|---|---:|
| `/16` | 65,536 |
| `/24` | 256 |
| `/25` | 128 |
| `/26` | 64 |
| `/27` | 32 |
| `/28` | 16 |
| `/30` | 4 |

For traditional IPv4 host subnets, usable hosts are commonly:

```text
Total addresses - 2
```

because network and broadcast addresses are reserved. There are exceptions, especially in point-to-point and cloud networking.

---

## 7. Subnet Mask

A subnet mask is another representation of the prefix.

```text
192.168.1.10/24

Subnet mask:
255.255.255.0
```

Binary:

```text
11111111.11111111.11111111.00000000
```

`1` bits represent the network portion and `0` bits represent the host portion.

Therefore:

```text
192.168.1.10/24
        ↓
Network = 192.168.1.0
```

---

## 8. Network Address

The **network address** identifies the subnet.

For:

```text
192.168.1.10/24
```

the network address is:

```text
192.168.1.0
```

The subnet is:

```text
192.168.1.0/24
```

---

## 9. Broadcast Address

The IPv4 **broadcast address** sends traffic to all hosts in the subnet.

For:

```text
192.168.1.0/24
```

we have:

```text
Network   = 192.168.1.0
Hosts     = 192.168.1.1 - 192.168.1.254
Broadcast = 192.168.1.255
```

IPv6 does not use broadcast.

---

## 10. Subnetting

Subnetting divides a larger network into smaller networks.

Start with:

```text
192.168.1.0/24
```

Split it into `/26` networks:

```text
/24 → /26
```

We borrow 2 host bits:

```text
2² = 4 subnets
```

The subnets are:

```text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

Each has:

```text
64 total addresses
62 traditional usable host addresses
```

---

## 11. Finding a Subnet Range

Example:

```text
192.168.1.70/26
```

A `/26` has blocks of 64:

```text
0   - 63
64  - 127
128 - 191
192 - 255
```

`70` belongs to `64 - 127`.

Therefore:

```text
Network   = 192.168.1.64
First host = 192.168.1.65
Last host  = 192.168.1.126
Broadcast  = 192.168.1.127
```

---

## 12. Local vs Remote Destination

Suppose:

```text
Host:
192.168.1.10/24

Destination:
192.168.1.20
```

Both are in:

```text
192.168.1.0/24
```

So the destination is local.

```text
Destination IP
      ↓
Same subnet?
      ↓
Yes
      ↓
ARP for destination MAC
      ↓
Ethernet frame → destination
```

Now:

```text
Host:
192.168.1.10/24

Destination:
10.0.0.20
```

The destination is remote:

```text
Destination IP
      ↓
Same subnet?
      ↓
No
      ↓
Default gateway
      ↓
Router
```

This is one of the most important uses of subnetting.

---

## 13. Default Gateway

The **default gateway** is the router used when the destination is outside the local network.

Example:

```text
IP      = 192.168.1.10
Mask    = 255.255.255.0
Gateway = 192.168.1.1
```

For:

```text
192.168.1.20
```

the host sends directly to the local destination.

For:

```text
8.8.8.8
```

the host sends the frame to:

```text
192.168.1.1
```

The router then forwards the packet.

---

## 14. Practical: View IP Configuration

Run:

```powershell
ipconfig
```

Example:

```text
IPv4 Address. . . . . . . . . : 192.168.1.10
Subnet Mask . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . : 192.168.1.1
```

Meaning:

```text
IP      = 192.168.1.10
Network = 192.168.1.0/24
Gateway = 192.168.1.1
```

---

## 15. Practical: View Full Interface Information

Run:

```powershell
ipconfig /all
```

Example:

```text
IPv4 Address. . . . . . . . . : 192.168.1.10
Subnet Mask . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . : 192.168.1.1
DNS Servers . . . . . . . . . : 192.168.1.1
```

The important fields are:

```text
IPv4 Address
Subnet Mask
Default Gateway
DNS Servers
```

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

The routing table helps the OS decide where an IP packet should be sent.

---

## 17. Special IPv4 Addresses

| Range / Address | Purpose |
|---|---|
| `127.0.0.0/8` | Loopback |
| `169.254.0.0/16` | Link-local / APIPA |
| `0.0.0.0` | Unspecified / default-route context |
| `255.255.255.255` | Limited broadcast |
| `10.0.0.0/8` | Private |
| `172.16.0.0/12` | Private |
| `192.168.0.0/16` | Private |

For example, a Windows host getting:

```text
169.254.x.x
```

can indicate that normal address configuration failed and Windows assigned an APIPA address.

---

## 18. Core Mental Model

Think of an IP address as:

```text
IP Address
    │
    ↓
Network prefix + Host portion
```

For:

```text
192.168.1.10/24
```

```text
Network = 192.168.1.0
Host    = 10
```

When sending:

```text
Destination IP
      ↓
Same subnet?
   /        Yes        No
  ↓          ↓
ARP for     Default
destination gateway
MAC           ↓
  ↓         Router
Switch
```

---

## 19. Summary

```text
IPv4
 ↓
32-bit address

IPv6
 ↓
128-bit address

Private IP
 ↓
Internal addressing

Public IP
 ↓
Internet-routable addressing

CIDR
 ↓
IP/prefix length

Subnet mask
 ↓
Defines network/host boundary

Network address
 ↓
Identifies the subnet

Broadcast address
 ↓
IPv4 → all hosts in the subnet

Subnetting
 ↓
Divides a network into smaller networks

Default gateway
 ↓
Next hop for destinations outside the local subnet
```

Core example:

```text
192.168.1.10/24

Network:
192.168.1.0

Usable hosts:
192.168.1.1 - 192.168.1.254

Broadcast:
192.168.1.255
```

Key decision:

```text
Destination IP
      ↓
Same subnet?
   /        Yes        No
  ↓          ↓
ARP         Gateway
  ↓           ↓
Switch      Router
```

# Networking — Phase 3: Ethernet & Local Networking

> **Goal:** Understand how devices communicate inside the same local network and how Ethernet, MAC addresses, frames, and switches make that communication possible.

## 1. Local Network

A **LAN (Local Area Network)** connects devices within a limited network such as a home, office, or data center.

```text
PC A ───┐
PC B ───┼── Switch ─── Router ─── Internet
PC C ───┘
```

When PC A communicates with PC B on the same LAN, traffic can be delivered locally through the switch.

---

## 2. Ethernet

**Ethernet** is a family of technologies used for communication over wired local networks.

It defines things such as:

- Frame format
- MAC addressing
- Local-link communication
- Error detection

Simplified stack:

```text
TCP/UDP
   ↓
IP packet
   ↓
Ethernet frame
   ↓
Physical medium
```

Ethernet is primarily associated with **Layer 2 (Data Link)**, while its physical transmission involves Layer 1.

---

## 3. MAC Address

A **MAC address** identifies a network interface on a local network.

Example:

```text
34:97:f6:aa:bb:cc
```

A typical Ethernet MAC address is **48 bits = 6 bytes**.

### MAC vs IP

```text
MAC → local/link-level identity
IP  → network-level addressing
```

Example:

```text
IP  : 192.168.1.20
MAC : 34:97:f6:aa:bb:cc
```

The IP address can change when a device joins another network; the MAC normally identifies the network interface.

---

## 4. Ethernet Frame

An Ethernet frame carries a network-layer packet over an Ethernet link.

Simplified:

```text
┌──────────────┬──────────────┬──────────┬─────────────┐
│ Dest MAC     │ Source MAC   │ EtherType│ Payload     │
├──────────────┴──────────────┴──────────┴─────────────┤
│                    IP Packet                         │
├─────────────────────────────────────────────────────┤
│                    FCS / Trailer                     │
└─────────────────────────────────────────────────────┘
```

Important fields:

- **Destination MAC** — intended receiver on the current link.
- **Source MAC** — sender on the current link.
- **EtherType** — identifies the payload, such as IPv4, IPv6, or ARP.
- **Payload** — often an IP packet.
- **FCS** — used for error detection.

---

## 5. Frame vs Packet

A simplified encapsulation chain is:

```text
Application Data
       ↓
TCP Segment
       ↓
IP Packet
       ↓
Ethernet Frame
```

The **IP packet** is a Layer 3 object.

The **Ethernet frame** is a Layer 2 object carrying that packet.

```text
Ethernet Frame
┌──────────────────────────────┐
│ Ethernet Header              │
│                              │
│    IP Packet                 │
│    ┌──────────────────────┐  │
│    │ IP Header            │  │
│    │ TCP Segment          │  │
│    └──────────────────────┘  │
│                              │
│ Ethernet Trailer             │
└──────────────────────────────┘
```

---

## 6. Switch

A **network switch** connects devices within a LAN.

```text
PC A ───┐
PC B ───┼── Switch
PC C ───┘
```

The switch maintains a **MAC address table**:

| MAC Address | Port |
|---|---:|
| AA:AA:AA:AA:AA:AA | 1 |
| BB:BB:BB:BB:BB:BB | 2 |
| CC:CC:CC:CC:CC:CC | 3 |

If a frame has:

```text
Destination MAC = BB:BB:BB:BB:BB:BB
```

the switch can send it to Port 2.

---

## 7. MAC Address Learning

Suppose PC A sends a frame through Port 1.

The switch sees:

```text
Source MAC = MAC_A
Incoming Port = 1
```

It learns:

```text
MAC_A → Port 1
```

If PC B later sends through Port 2:

```text
MAC_B → Port 2
```

The switch gradually builds its MAC table.

Entries can expire after inactivity.

---

## 8. Unknown Destination

If the switch doesn't know the destination MAC, it generally **floods** the frame out the other relevant ports.

```text
          ┌── PC B
          │
PC A ─ Switch ─ PC C
          │
          └── PC D
```

Once the destination responds, the switch can learn its MAC address.

---

## 9. Unicast, Broadcast, Multicast

### Unicast

One sender → one receiver.

```text
A ─────→ B
```

### Broadcast

One sender → all devices in the broadcast domain.

IPv4 Ethernet broadcast:

```text
FF:FF:FF:FF:FF:FF
```

ARP commonly uses broadcasts to discover local IPv4-to-MAC mappings.

### Multicast

One sender → a group of interested receivers.

```text
        ┌── B
        ├── C
A ──────┤
        └── D
```

---

## 10. Collision Domain

A **collision domain** is an area where simultaneous transmissions can potentially interfere.

Old hub-based Ethernet:

```text
       ┌── PC A
       │
Hub ───┼── PC B
       │
       └── PC C
```

Modern switched Ethernet typically gives each switch port its own collision domain.

```text
PC A ── Port 1
PC B ── Port 2
PC C ── Port 3
          │
        Switch
```

---

## 11. Broadcast Domain

A **broadcast domain** is the set of devices that receive a Layer 2 broadcast.

```text
PC A ─┐
PC B ─┼── Switch
PC C ─┘
```

A broadcast can reach A, B, and C.

Routers normally separate broadcast domains:

```text
Network A                 Network B
PC ─ Switch ─ Router ─ Switch ─ PC
```

A Layer 2 broadcast from Network A normally does not cross the router.

---

## 12. VLAN

A **VLAN (Virtual LAN)** logically separates networks at Layer 2.

Without VLANs:

```text
Switch
 ├── PC A
 ├── PC B
 ├── PC C
 └── PC D
```

With VLANs:

```text
VLAN 10             VLAN 20
 ├── PC A             ├── PC C
 └── PC B             └── PC D
```

Devices in different VLANs are logically separated and normally need Layer 3 routing to communicate.

---

## 13. Access Port vs Trunk Port

An **access port** normally carries one VLAN:

```text
PC
 |
Access Port
 |
Switch
VLAN 10
```

A **trunk port** can carry multiple VLANs, usually using VLAN tags:

```text
Switch A
   |
 Trunk
   |
Switch B
```

The trunk can carry:

```text
VLAN 10
VLAN 20
VLAN 30
```

---

## 14. Same-Subnet Communication

Suppose:

```text
PC A
IP  = 192.168.1.10
MAC = AA:AA:AA:AA:AA:AA

PC B
IP  = 192.168.1.20
MAC = BB:BB:BB:BB:BB:BB
```

Both belong to:

```text
192.168.1.0/24
```

PC A sends:

```text
Application
    ↓
TCP/UDP
    ↓
IP packet
Destination IP = 192.168.1.20
    ↓
Find destination MAC
    ↓
Ethernet frame
Destination MAC = BB:BB:BB:BB:BB:BB
    ↓
Switch
    ↓
PC B
```

The switch forwards the frame using the destination MAC.

---

## 15. Different-Subnet Communication

Suppose:

```text
PC A
192.168.1.10
```

wants to communicate with:

```text
Server
10.0.0.20
```

The server is remote from PC A's subnet.

The frame is sent toward the **default gateway**:

```text
PC A
     |
     | Ethernet frame
     | Destination MAC = Gateway MAC
     ↓
Switch
     ↓
Router
     |
     | New Layer 2 frame
     ↓
Next network
     ↓
Server
```

Important:

```text
Destination IP = 10.0.0.20
```

is still the destination of the IP packet.

But:

```text
Destination MAC = Gateway MAC
```

on the first local link.

At the next link, the Layer 2 frame can have different MAC addresses.

This is the key distinction:

```text
IP  → network-to-network addressing/routing
MAC → local-link delivery
```

---

## 16. Practical: Find Your MAC Address

On Windows:

```powershell
ipconfig /all
```

Example:

```text
Ethernet adapter Ethernet:

   Physical Address. . . . . . : 34-97-F6-AA-BB-CC
   IPv4 Address. . . . . . . . : 192.168.1.20
   Subnet Mask . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . : 192.168.1.1
```

Meaning:

```text
MAC = 34-97-F6-AA-BB-CC
IP  = 192.168.1.20
```

The MAC is used for local Layer 2 communication; the IP is used for Layer 3 addressing.

---

## 17. Practical: View ARP Cache

Run:

```powershell
arp -a
```

Example:

```text
Interface: 192.168.1.20

Internet Address      Physical Address      Type
192.168.1.1           34-97-f6-aa-bb-cc     dynamic
192.168.1.5           08-00-27-11-22-33     dynamic
```

Meaning:

```text
192.168.1.1 → 34-97-f6-aa-bb-cc
```

The host has learned the MAC associated with that local IPv4 address.

Conceptually:

```text
IP address
    ↓
ARP lookup
    ↓
MAC address
    ↓
Ethernet frame
```

ARP itself will be covered separately; here the important connection is between Layer 3 IP addressing and Layer 2 MAC addressing.

---

## 18. Practical: View Network MAC Addresses

Run:

```powershell
getmac
```

Example:

```text
Physical Address    Transport Name
=================== =========================================================
34-97-F6-AA-BB-CC   \Device\Tcpip_{...}
```

Meaning:

```text
34-97-F6-AA-BB-CC
```

is the MAC address associated with that network interface.

---

## 19. Key Mental Model

### Same subnet

```text
Application
    ↓
TCP/UDP
    ↓
IP
    ↓
Destination MAC
    ↓
Ethernet Frame
    ↓
Switch
    ↓
Destination
```

### Different subnet

```text
Application
    ↓
TCP/UDP
    ↓
IP
    ↓
Destination is remote
    ↓
Gateway MAC
    ↓
Ethernet Frame
    ↓
Switch
    ↓
Router
    ↓
New Layer 2 frame
    ↓
Next hop
    ↓
Destination
```

---

## 20. Summary

```text
Ethernet
    ↓
Layer 2 local networking

MAC address
    ↓
Network-interface address used on the local link

Frame
    ↓
Layer 2 unit carrying a packet

Switch
    ↓
Forwards frames using MAC addresses

Broadcast
    ↓
One sender → all devices in the broadcast domain

VLAN
    ↓
Logical Layer 2 separation

Router
    ↓
Connects different IP networks

Same subnet
    ↓
Frame targets the destination MAC

Different subnet
    ↓
Frame initially targets the default gateway MAC
```

The core relationship:

```text
                IP Packet
                    │
                    │ carried inside
                    ↓
             Ethernet Frame
                    │
                    │ forwarded by
                    ↓
                 Switch
                    │
                    ↓
             Local destination
```

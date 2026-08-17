# Networking — Phase 4: ARP & Address Resolution

> **Goal:** Understand how a device discovers the MAC address corresponding to an IPv4 address on the local network.

## 1. Why Do We Need ARP?

IP and MAC addresses serve different purposes:

```text
IP  → Layer 3 → network/host addressing
MAC → Layer 2 → local-link delivery
```

Suppose PC A knows:

```text
Destination IP = 192.168.1.20
```

but needs to build an Ethernet frame. It also needs:

```text
Destination MAC = BB:BB:BB:BB:BB:BB
```

**ARP (Address Resolution Protocol)** provides:

```text
IPv4 address
     ↓
    ARP
     ↓
MAC address
```

---

## 2. ARP Request

If PC A does not know the MAC for `192.168.1.20`, it sends an ARP Request:

```text
PC A
192.168.1.10
     |
     | Broadcast
     | "Who has 192.168.1.20?"
     ↓
   Switch
   ├── PC B
   ├── PC C
   └── PC D
```

The Ethernet destination MAC is:

```text
FF:FF:FF:FF:FF:FF
```

The switch floods the frame within the broadcast domain.

---

## 3. ARP Reply

PC B owns `192.168.1.20`, so it replies:

```text
PC B
     |
     | ARP Reply
     | "192.168.1.20 is BB:BB:BB:BB:BB:BB"
     ↓
PC A
```

PC A can now store:

```text
192.168.1.20
      ↓
BB:BB:BB:BB:BB:BB
```

It can then send the actual data frame.

---

## 4. Complete Local Communication

```text
Application
    ↓
TCP/UDP
    ↓
Destination IP = 192.168.1.20
    ↓
Check ARP cache
    ↓
MAC known?
   /  Yes  No
  |    |
  |   ARP Request
  |      ↓
  |   ARP Reply
  |      ↓
  └──────┘
    ↓
Ethernet Frame
    ↓
Switch
    ↓
PC B
```

The important point is:

> ARP happens before the host can normally send an IPv4 packet to a local destination using Ethernet, if the required MAC mapping is not already cached.

---

## 5. ARP Cache

Hosts store recently learned mappings:

| IPv4 Address | MAC Address |
|---|---|
| 192.168.1.1 | 34:97:f6:aa:bb:cc |
| 192.168.1.20 | bb:bb:bb:bb:bb:bb |

Entries eventually expire.

This avoids performing an ARP broadcast for every packet.

---

## 6. Practical: View ARP Cache

On Windows:

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
192.168.1.5 → 08-00-27-11-22-33
```

`dynamic` generally means the mapping was learned automatically.

---

## 7. Practical: Generate ARP Traffic

Try:

```powershell
ping 192.168.1.1
```

Then:

```powershell
arp -a
```

You may see:

```text
Internet Address      Physical Address      Type
192.168.1.1           34-97-f6-aa-bb-cc     dynamic
```

Conceptually:

```text
Ping destination
      ↓
Need destination MAC
      ↓
ARP lookup
      ↓
If missing → ARP Request
      ↓
ARP Reply
      ↓
MAC added to cache
      ↓
ICMP packet sent
```

If the entry was already cached, a new ARP request may not occur.

---

## 8. ARP Only Resolves Local IPv4 Destinations

ARP does **not** find the MAC address of a remote Internet server.

Suppose:

```text
PC
192.168.1.10
   |
   | wants to reach
   ↓
8.8.8.8
```

If `8.8.8.8` is outside the local subnet, the PC needs the MAC of its **default gateway**, not the MAC of `8.8.8.8`.

```text
PC
   |
   | Destination MAC = Gateway MAC
   ↓
Router
   |
   ↓
Internet
   |
   ↓
8.8.8.8
```

The IP packet still has:

```text
Destination IP = 8.8.8.8
```

but the first Ethernet frame has:

```text
Destination MAC = Gateway MAC
```

---

## 9. IP Destination vs MAC Destination

For remote communication:

```text
IP Packet
Destination IP = 8.8.8.8
       │
       ↓
Ethernet Frame
Destination MAC = Gateway MAC
```

When the router forwards the packet, it creates a new Layer 2 frame for the next link.

Therefore:

```text
IP destination
    ↓
Usually remains the final destination

MAC destination
    ↓
Changes at each Layer 2 hop
```

This is a key Layer 2 vs Layer 3 concept.

---

## 10. ARP Request and Reply

A request contains information such as:

```text
Sender IP
Sender MAC

Target IP
Target MAC
```

Example:

```text
Sender IP  = 192.168.1.10
Sender MAC = AA:AA:AA:AA:AA:AA

Target IP  = 192.168.1.20
Target MAC = unknown
```

Effectively:

```text
"Who has 192.168.1.20?"
```

The Ethernet frame is broadcast:

```text
FF:FF:FF:FF:FF:FF
```

The reply provides:

```text
192.168.1.20
      ↓
BB:BB:BB:BB:BB:BB
```

and is normally sent directly to the requester.

---

## 11. Gratuitous ARP

A **Gratuitous ARP (GARP)** is an ARP message sent without a normal request from another host.

It can be used to:

- Announce an IP/MAC mapping
- Update ARP caches
- Detect duplicate IP addresses
- Help with failover

For example, after a failover:

```text
192.168.1.20
      ↓
New MAC
```

A device can announce this mapping so other hosts can update their caches.

---

## 12. ARP Spoofing

Traditional ARP does not authenticate replies.

An attacker could falsely claim:

```text
192.168.1.1
      ↓
Attacker MAC
```

A victim might then incorrectly store:

```text
Gateway IP → Attacker MAC
```

This is called **ARP spoofing / ARP poisoning**.

Network protections can include:

- Dynamic ARP Inspection
- DHCP snooping
- Network segmentation
- Appropriate static mappings in specific environments

---

## 13. ARP and the Switch

The host runs ARP; the switch normally just forwards the Ethernet frames.

```text
Host
 ↓
Creates ARP Request
 ↓
Switch
 ↓
Floods broadcast
```

So:

```text
Host
 ↓
Runs ARP

Switch
 ↓
Forwards Layer 2 frames
```

A router or Layer 3 switch can itself use ARP when it needs to resolve a next-hop IPv4 address.

---

## 14. ARP and the OSI Model

ARP connects Layer 3 IPv4 addressing with Layer 2 MAC addressing:

```text
IPv4 address
      ↓
     ARP
      ↓
MAC address
      ↓
Ethernet
```

ARP messages are carried directly inside Ethernet frames rather than inside IP packets.

---

## 15. ARP vs DNS

These solve different problems.

### DNS

```text
Domain name
     ↓
IP address
```

Example:

```text
google.com
     ↓
142.x.x.x
```

### ARP

```text
IPv4 address
     ↓
MAC address
```

Example:

```text
192.168.1.20
     ↓
BB:BB:BB:BB:BB:BB
```

Remember:

```text
DNS → Name → IP
ARP → IP → MAC
```

---

## 16. Key Mental Model

### Same subnet

```text
Destination IP
      ↓
ARP cache
      ↓
Destination MAC
      ↓
Ethernet frame
      ↓
Switch
      ↓
Destination host
```

### Different subnet

```text
Destination IP is remote
          ↓
Find default gateway
          ↓
ARP for gateway MAC
          ↓
Ethernet frame → Gateway
          ↓
Router forwards IP packet
          ↓
New Layer 2 frame
```

---

## 17. Summary

```text
ARP
 ↓
IPv4 → MAC on the local network

ARP Request
 ↓
Broadcast

ARP Reply
 ↓
Normally unicast to requester

ARP Cache
 ↓
Stores learned mappings

Same subnet
 ↓
ARP for destination host

Different subnet
 ↓
ARP for default gateway

DNS
 ↓
Domain → IP

ARP
 ↓
IP → MAC
```

Core relationship:

```text
          DNS
Domain ───────→ IP
                │
                │ ARP
                ↓
               MAC
                │
                ↓
             Ethernet
                │
                ↓
              Switch
```

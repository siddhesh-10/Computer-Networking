# Networking — Phase 7: UDP

> **Goal:** Understand how UDP provides simple, connectionless transport between applications and why it is useful when low overhead and speed matter more than built-in reliability.

## 1. What Is UDP?

**UDP (User Datagram Protocol)** is a transport-layer protocol.

```text
Application
    ↓
UDP
    ↓
IP
    ↓
Ethernet / Wi-Fi
```

UDP provides:

- Port-based delivery
- Checksums
- Message/datagram boundaries

It does **not** provide a connection or guaranteed delivery.

---

## 2. Datagram

UDP sends data as independent **datagrams**.

```text
Application
   ↓
UDP Datagram 1
UDP Datagram 2
UDP Datagram 3
```

Each datagram is handled independently.

A datagram contains:

```text
UDP Header
    +
Application Data
```

Unlike TCP, UDP preserves message boundaries.

---

## 3. Connectionless Communication

UDP does not establish a connection before sending data.

TCP:

```text
Client
  ↓
Connection establishment
  ↓
Data
```

UDP:

```text
Client
  ↓
Send datagram
  ↓
Server
```

There is no TCP-style handshake, reducing overhead.

---

## 4. UDP Header

A UDP header is only **8 bytes**:

```text
+-------------------+-------------------+
| Source Port       | Destination Port  |
+-------------------+-------------------+
| Length            | Checksum          |
+-------------------+-------------------+
```

### Source Port
Identifies the sending application/process.

### Destination Port
Identifies the receiving application/process.

### Length
Size of UDP header + UDP data.

### Checksum
Used to detect corruption of the datagram.


UDP data means the data given to UDP by the application, not the entire IP packet.
```text
Think of encapsulation:

Application data
       ↓
   UDP header
       +
   UDP data
       ↓
     UDP
       ↓
   IP header
       +
   UDP segment
       ↓
     IP packet

For example, suppose a DNS application wants to send:

"google.com"

UDP creates:

┌─────────────────────────────┐
│ UDP Header                  │
│ Source Port                │
│ Destination Port           │
│ Length                     │
│ Checksum                   │
├─────────────────────────────┤
│ UDP Data                    │
│ "google.com"               │
└─────────────────────────────┘

This whole thing is called a UDP datagram (or UDP segment in some terminology).

Then IP wraps it:

┌─────────────────────────────┐
│ IP Header                   │
│ Source IP                   │
│ Destination IP              │
│ TTL / Hop Limit             │
│ Protocol = UDP              │
├─────────────────────────────┤
│ UDP Header                  │
│ UDP Data                    │
└─────────────────────────────┘

So:

IP data/payload
        =
    UDP datagram
        =
UDP header + UDP data

That's the important distinction.

Another way to visualize it
IP packet
┌───────────────────────────────┐
│ IP Header                     │
├───────────────────────────────┤
│                               │
│    UDP datagram               │
│    ┌───────────────────────┐  │
│    │ UDP Header            │  │
│    ├───────────────────────┤  │
│    │ Application Data      │  │
│    └───────────────────────┘  │
│                               │
└───────────────────────────────┘

So when you say "UDP data", think:

The application data carried inside UDP.
```
---

## 5. Ports

IP identifies the host; the port identifies the application on that host.

```text
IP Address + Port
       ↓
Application endpoint
```

Example:

```text
192.168.1.10:53
```

A server may listen on:

```text
0.0.0.0:53
```

meaning port 53 on its local interfaces.

---

## 6. UDP Is Not Reliable

UDP does not guarantee:

```text
Delivery
Ordering
Retransmission
Duplicate protection
Flow control
Congestion control
```

If:

```text
Sender → 1 → 2 → 3 → Network
```

and datagram 2 is lost:

```text
Receiver → 1 → 3
```

UDP itself does not retransmit datagram 2.

If reliability is required, the application or a protocol above UDP must provide it.


## Congestion control vs flow control

These are easy to confuse.
```text
Congestion control

Protects the network.

Sender → Internet → Receiver
            ↑
      Don't overload
       the network

Question:

"Can the network handle this much traffic?"
```

```text
Flow control

Protects the receiver.

Sender ─────────→ Receiver
                    ↑
              Receiver is slow

Question:

"Can the receiver handle this much data?"

For example, your PC might be capable of receiving only a certain amount of data at a time.
```

TCP handles both
TCP
├── Flow control
│     → Don't overwhelm receiver
│
└── Congestion control
      → Don't overwhelm network
What about UDP?

UDP itself has no built-in congestion control.

---

## 7. UDP Does Not Cause Packet Loss

The network can lose packets with either TCP or UDP.

The difference is:

```text
TCP
 ↓
Detect missing data
 ↓
Retransmit

UDP
 ↓
No built-in retransmission
 ↓
Application decides
```

---

## 8. UDP and Ordering

Sender:

```text
1 → 2 → 3
```

Receiver could get:

```text
1 → 3 → 2
```

UDP does not reorder the datagrams.

An application can implement sequence numbers if ordering is required.

---

## 9. UDP Use Cases

UDP is useful when low overhead, low latency, or application-controlled reliability matters.

### DNS

Traditional DNS commonly uses:

```text
UDP port 53
```

### DHCP

DHCP commonly uses:

```text
UDP 67 → Server
UDP 68 → Client
```

### Real-time traffic

Examples:

```text
Voice
Video
Online gaming
```

For real-time traffic, waiting for an old lost packet can sometimes be worse than continuing with newer data.

### QUIC

QUIC is built on UDP and adds features such as:

```text
Reliable delivery
Encryption
Stream multiplexing
Connection migration
```

So:

> UDP itself is not reliable, but protocols built on top of UDP can provide reliability.

---

## 10. TCP vs UDP

genui{"learning_viz":{"type_id":"TCP_VS_UDP"}}

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Handshake | Yes | No |
| Reliability | Built-in | No |
| Ordering | Built-in | No |
| Retransmission | Yes | No |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Data model | Byte stream | Datagrams |
| Minimum header | 20 bytes | 8 bytes |
| Broadcast/multicast | No | Yes |
| Typical use | HTTP/HTTPS, SSH | DNS, DHCP, real-time traffic |

---

## 11. TCP vs UDP Data Model

TCP provides a **byte stream**.

If an application writes:

```text
"Hello"
"World"
```

TCP does not preserve those as two messages. The receiver sees a stream of bytes.

UDP preserves datagram boundaries:

```text
Datagram 1 = "Hello"
Datagram 2 = "World"
```

This distinction is important.

---

## 12. Practical: View UDP Sockets

On Windows:

```powershell
netstat -ano -p udp
```

Example:

```text
Proto  Local Address      Foreign Address    State
UDP    0.0.0.0:5353       *:*
UDP    0.0.0.0:1900       *:*
```

Meaning:

```text
Protocol   = UDP
Local port = 5353 / 1900
```

`*:*` means there is no specific remote endpoint associated with that socket.

UDP does not require a TCP-style `ESTABLISHED` connection state.

The exact output depends on running applications.

---

## 13. Practical: Find a Specific UDP Port

Run:

```powershell
netstat -ano -p udp | findstr ":53"
```

Possible output:

```text
UDP    192.168.1.10:5353    *:*
```

Meaning:

```text
Local IP   = 192.168.1.10
Local port = 5353
Protocol   = UDP
```

---

## 14. UDP Encapsulation

When an application sends UDP data:

```text
Application Data
       ↓
+------------------+
| UDP Header       |
| Source Port      |
| Destination Port |
| Length            |
| Checksum          |
+------------------+
| Application Data |
+------------------+
       ↓
      IP
       ↓
Ethernet / Wi-Fi
```

At the receiver:

```text
Ethernet
   ↓
IP
   ↓
UDP
   ↓
Destination Port
   ↓
Correct application
```

---

## 15. Key Mental Model

UDP's job is intentionally small:

```text
Application
     ↓
Send datagram to IP:port
     ↓
UDP
     ↓
Ports + length + checksum
     ↓
IP
     ↓
Network
```

It provides:

```text
Process-to-process delivery
+
Datagram boundaries
+
Checksum
```

It does not try to provide every transport feature.

---

## 16. Summary

```text
UDP
 ↓
Transport-layer protocol

Connectionless
 ↓
No connection setup

Datagram
 ↓
Independent message

Header
 ↓
8 bytes

Provides
 ↓
Ports
 ↓
Length
 ↓
Checksum

Does NOT provide
 ↓
Guaranteed delivery
 ↓
Ordering
 ↓
Retransmission
 ↓
Flow control
 ↓
Congestion control
```

Core comparison:

```text
TCP
 ↓
Reliable + ordered byte stream
 ↓
More transport-layer mechanisms

UDP
 ↓
Connectionless datagrams
 ↓
Low overhead
 ↓
Application controls additional behavior
```

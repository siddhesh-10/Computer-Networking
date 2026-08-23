# Networking — Phase 14: HTTP/2 & HTTP/3

> **Goal:** Understand how HTTP/2 improves HTTP/1.1 connection usage and how HTTP/3 uses QUIC over UDP.

## 1. HTTP/2

HTTP/2 introduces:

```text
Binary framing
Streams
Multiplexing
Header compression
```

---

## 2. Binary Framing

HTTP/1.1 is text-oriented. HTTP/2 represents communication using **binary frames**.

```text
HTTP/2
 ↓
Binary frames
 ↓
TCP
```

The HTTP request/response semantics remain, but the wire format changes.

---

## 3. Streams

HTTP/2 divides one TCP connection into logical streams.

```text
One TCP connection
 ├── Stream 1
 ├── Stream 3
 ├── Stream 5
 └── Stream 7
```

Each stream can carry an HTTP request/response exchange.

---

## 4. Multiplexing

HTTP/2 can interleave frames from multiple streams:

```text
One TCP connection
 ├── Request A
 ├── Request B
 ├── Request C
 └── Request D
```

This reduces the need for many parallel TCP connections.

---

## 5. HPACK

HTTP/2 uses **HPACK** for header compression.

Repeated headers can be represented efficiently using a compressed representation and header tables.

This reduces repeated header overhead.

---

## 6. Head-of-Line Blocking

HTTP/2 multiplexes streams over one TCP connection.

TCP guarantees ordered byte delivery.

If a TCP packet is lost:

```text
TCP packet lost
      ↓
TCP retransmits
      ↓
Later bytes wait for missing bytes
      ↓
Multiple HTTP/2 streams can be affected
```

So HTTP/2 can still experience **transport-level head-of-line blocking**.

---

## 7. HTTP/3

HTTP/3 changes the transport:

```text
HTTP/1.1 → TCP
HTTP/2   → TCP
HTTP/3   → QUIC
```

QUIC runs over UDP:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
```

---

## 8. QUIC

QUIC is a transport protocol built over UDP.

It provides features including:

```text
Reliable delivery
Streams
Encryption
Connection migration
Congestion control
```

Raw UDP does not provide these guarantees; QUIC adds them.

---

## 9. HTTP/3 Multiplexing

One QUIC connection can contain independent streams:

```text
One QUIC connection
 ├── Stream A
 ├── Stream B
 ├── Stream C
 └── Stream D
```

Loss affecting one stream does not require unrelated streams to wait for TCP's single ordered byte stream.

---

## 10. HTTP/2 vs HTTP/3

| | HTTP/2 | HTTP/3 |
|---|---|---|
| Transport | TCP | QUIC |
| Underlying protocol | TCP | UDP |
| Multiplexing | Yes | Yes |
| Binary framing | Yes | Yes |
| Header compression | HPACK | QPACK |
| Transport HOL blocking | Possible | Avoided across streams |
| Encryption | TLS | TLS integrated with QUIC |

---

## 11. Practical: Check HTTP Version

With a curl build that supports it:

```powershell
curl.exe -I --http2 https://example.com
```

For HTTP/3:

```powershell
curl.exe -I --http3 https://example.com
```

Support depends on the installed curl build and destination server.

Browser developer tools can also show the protocol used for a request.

---

## 12. Mental Model

HTTP/1.1:

```text
HTTP
 ↓
TCP
 ↓
IP
```

HTTP/2:

```text
HTTP/2
 ↓
Binary frames
 ↓
Multiple streams
 ↓
TCP
 ↓
IP
```

HTTP/3:

```text
HTTP/3
 ↓
QUIC
 ↓
Multiple streams
 ↓
UDP
 ↓
IP
```

Core difference:

```text
HTTP/2
 ↓
Multiplexing over TCP
 ↓
TCP loss recovery can block streams

HTTP/3
 ↓
Multiplexing over QUIC
 ↓
Streams have independent transport-level progress

# Networking — Phase 19: WebSockets, gRPC & Streaming

> **Goal:** Understand different ways applications communicate beyond simple request/response APIs, including REST, WebSockets, SSE, long polling, gRPC, and streaming.

## 1. Where These Fit

These are primarily **application-layer communication mechanisms**.

```text
Application
 ├── REST / HTTP APIs
 ├── WebSockets
 ├── SSE
 └── gRPC
        ↓
   HTTP/1.1 or HTTP/2
        ↓
       TCP
        ↓
       IP
```

Modern HTTP/3 applications can use:

```text
Application
   ↓
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
```

The application protocol defines **how messages are exchanged**; the underlying transport provides network communication.

---

## 2. Request/Response vs Streaming

Traditional HTTP:

```text
Client
   ↓ Request
Server
   ↓ Response
Client
```

Streaming:

```text
Client
   ↓
Server
   ↓ data
   ↓ data
   ↓ data
```

Bidirectional communication:

```text
Client  ←────────→  Server
        messages
        in both
        directions
```

---

## 3. REST APIs

**REST** is an architectural style commonly implemented using HTTP.

Example:

```http
GET /users/123
```

Response:

```json
{
  "id": 123,
  "name": "Alice"
}
```

Common methods:

```text
GET
POST
PUT
PATCH
DELETE
```

Typical flow:

```text
Client
   ↓ HTTP request
API
   ↓ HTTP response
Client
```

Common uses:

```text
Web APIs
Mobile applications
Public APIs
CRUD operations
Browser-to-backend communication
```

> REST is an architectural style; HTTP is a protocol.

---

## 4. Other API Styles

### REST

```text
HTTP
 ↓
Resource-oriented API
```

### RPC

**RPC (Remote Procedure Call)** models communication as calling a remote service method.

```text
Client
   ↓
getUser(123)
   ↓
Remote service
```

Examples include:

```text
gRPC
JSON-RPC
XML-RPC
```

### GraphQL

GraphQL lets clients request the fields they need.

```graphql
query {
  user(id: 123) {
    name
    email
  }
}
```

It is commonly transported over HTTP.

### WebSocket

Designed for long-lived, bidirectional communication:

```text
Client ←────────→ Server
```

---

## 5. Long Polling

Long polling uses normal HTTP but keeps a request open until data is available.

```text
Client → Request
Server → Wait
Server → Response when data is available
Client → New Request
```

Flow:

```text
Client
  ↓ Request
Server
  ↓ waits
  ↓ data available
  ↓ Response
Client
  ↓
New Request
```

Useful when:

```text
Real-time updates are needed
WebSockets are unavailable
Simple HTTP infrastructure is preferred
```

Limitations:

```text
Repeated HTTP requests
Connection overhead
More server connection management
```

---

## 6. Server-Sent Events (SSE)

**SSE (Server-Sent Events)** allows a server to continuously send events to a browser over an HTTP connection.

```text
Client
   ↓ HTTP request
Server
   ↓ event
   ↓ event
   ↓ event
```

Communication is:

```text
Server → Client
```

So SSE is **unidirectional**.

Typical uses:

```text
Live notifications
Progress updates
Monitoring dashboards
Live feeds
Status changes
```

Example:

```http
Content-Type: text/event-stream
```

Possible body:

```text
data: update 1

data: update 2

data: update 3
```

Browsers can consume SSE using the `EventSource` API.

---

## 7. SSE vs Long Polling

| | Long Polling | SSE |
|---|---|---|
| Transport | HTTP | HTTP |
| Direction | Repeated server responses | Server → client stream |
| Persistent connection | Repeated requests | Yes |
| Browser API | Normal HTTP | `EventSource` |
| Server → client updates | Yes | Yes |
| Client → server | Separate request | Separate request |

For continuous server-to-browser events, SSE is generally simpler than repeatedly long-polling.

---

## 8. WebSockets

**WebSocket** provides a persistent, bidirectional communication channel.

```text
Client ←────────→ Server
       messages
       both ways
```

Unlike normal HTTP:

```text
Client → Request
Server → Response
```

either side can send messages when needed.

Common uses:

```text
Chat
Multiplayer games
Live dashboards
Collaborative editing
Real-time notifications
Trading interfaces
```

---

## 9. WebSocket Connection

A WebSocket connection traditionally starts with an HTTP upgrade.

Conceptually:

```text
Client
   ↓ HTTP Upgrade request
Server
   ↓ 101 Switching Protocols
WebSocket connection
   ↓
Bidirectional messages
```

After the upgrade:

```text
Client ←────────→ Server
```

The connection remains open until one side closes it or it fails.

---

## 10. WebSockets and TCP

Traditional WebSockets use TCP:

```text
WebSocket
    ↓
TCP
    ↓
IP
```

TCP provides:

```text
Reliable delivery
Ordered byte stream
Retransmission
Flow control
Congestion control
```

WebSocket provides the application-level messaging model on top.

---

## 11. Persistent Connections

HTTP keep-alive:

```text
TCP connection
 ├── HTTP request/response
 ├── HTTP request/response
 └── HTTP request/response
```

WebSocket:

```text
One connection
 ├── Client → Server
 ├── Server → Client
 ├── Server → Client
 ├── Client → Server
 └── ...
```

WebSocket is useful when communication must continue in both directions.

---

## 12. gRPC

**gRPC** is an RPC framework commonly used for service-to-service communication.

Typical stack:

```text
Application
   ↓
gRPC
   ↓
HTTP/2
   ↓
TCP
   ↓
IP
```

gRPC commonly uses:

```text
Protocol Buffers (Protobuf)
HTTP/2
Binary serialization
```

Example:

```text
UserService
    ↓
GetUser(123)
    ↓
User response
```

Instead of manually constructing URLs and JSON, clients can call generated service methods.

---

## 13. Protocol Buffers

gRPC commonly uses **Protocol Buffers** to define messages and services.

Example:

```protobuf
message User {
    int32 id = 1;
    string name = 2;
}
```

The `.proto` definition can generate client/server code.

Advantages:

```text
Compact binary representation
Strongly typed schemas
Code generation
Efficient serialization
```

---

## 14. gRPC Streaming

gRPC supports several patterns.

### Unary RPC

```text
Client → Request
Server → Response
```

### Server Streaming

```text
Client → Request
Server → Response 1
         Response 2
         Response 3
```

### Client Streaming

```text
Client → Request 1
       → Request 2
       → Request 3
Server → Response
```

### Bidirectional Streaming

```text
Client → Message
Server → Message
Client → Message
Client → Message
Server → Message
```

Both sides can continuously exchange messages.

---

## 15. HTTP/2 and gRPC

gRPC commonly uses HTTP/2 features:

```text
Streams
Multiplexing
Binary framing
Header compression
```

Conceptually:

```text
One TCP connection
 ├── gRPC stream A
 ├── gRPC stream B
 └── gRPC stream C
```

Multiple RPCs can share a connection.

---

## 16. Streaming

**Streaming** means data is processed incrementally instead of waiting for the complete result.

Without streaming:

```text
Request
   ↓
Wait for complete result
   ↓
Response
```

With streaming:

```text
Request
   ↓
Data 1
   ↓
Data 2
   ↓
Data 3
   ↓
Data 4
```

Useful for:

```text
Large files
Video/audio
Logs
Live metrics
AI/model responses
Database results
Event streams
```

---

## 17. Bidirectional Communication

Bidirectional communication means both sides can send data independently.

```text
Client                  Server

  |  Message A  -------->
  |
  |  <-------- Message B
  |
  |  Message C  -------->
  |
  |  <-------- Message D
```

Examples:

```text
WebSocket
gRPC bidirectional streaming
```

This differs from normal request/response communication.

---

## 18. Choosing the Right API / Communication Style

| Technology | Direction | Typical transport | Common use |
|---|---|---|---|
| REST | Request/response | HTTP | Public/web APIs |
| GraphQL | Request/response | HTTP | Flexible client queries |
| Long polling | Repeated server responses | HTTP | Simple real-time fallback |
| SSE | Server → client | HTTP | Notifications/live feeds |
| WebSocket | Bidirectional | TCP | Real-time applications |
| gRPC unary | Request/response | HTTP/2 | Service-to-service |
| gRPC streaming | Streaming/bidirectional | HTTP/2 | Service streams |

---

## 19. Practical: Test HTTP Streaming

If you have a streaming endpoint:

```powershell
curl.exe -N https://example.com/stream
```

Possible output:

```text
event: update
data: 1

event: update
data: 2

event: update
data: 3
```

Meaning:

```text
Server
 ↓
Keeps connection open
 ↓
Sends data incrementally
 ↓
Client processes events as they arrive
```

The exact endpoint and output depend on the server.

---

## 20. Practical: Inspect HTTP Version

```powershell
curl.exe -I https://example.com
```

Example:

```text
HTTP/1.1 200 OK
Content-Type: text/html
```

For HTTP/2-capable curl:

```powershell
curl.exe -I --http2 https://example.com
```

Support depends on the installed curl build and server.

---

## 21. Mental Model

REST:

```text
Client
  ↓
HTTP Request
  ↓
Server
  ↓
HTTP Response
```

SSE:

```text
Client
  ↓
HTTP connection
  ↓
Server
  ↓
Event
  ↓
Event
  ↓
Event
```

WebSocket:

```text
Client ←────────→ Server
       messages
```

gRPC:

```text
Client
   ↓
gRPC
   ↓
HTTP/2
   ↓
TCP
   ↓
Server
```

gRPC bidirectional streaming:

```text
Client ←────────────→ Server
       continuous
       messages
```

---

## 22. Summary

```text
REST
 ↓
Resource-oriented HTTP APIs

Long polling
 ↓
Repeated HTTP requests waiting for updates

SSE
 ↓
Server → client event stream

WebSocket
 ↓
Persistent bidirectional communication

gRPC
 ↓
RPC framework commonly using HTTP/2 + Protobuf

Streaming
 ↓
Process data incrementally

Bidirectional streaming
 ↓
Both sides continuously exchange messages
```

Key distinction:

```text
REST
"Request something and receive a response"

SSE
"Keep sending events from server to client"

WebSocket
"Keep a two-way communication channel open"

gRPC
"Call remote services using strongly typed RPCs"

Streaming
"Process data as it arrives instead of waiting for everything"
```

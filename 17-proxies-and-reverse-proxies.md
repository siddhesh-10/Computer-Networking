# Networking --- Phase 17: Proxies & Reverse Proxies

> **Goal:** Understand what proxies are, where they sit in a network,
> how forward and reverse proxies differ, and how L4/L7 proxies, TLS
> termination, and API gateways are used.

## 1. What Is a Proxy?

A **proxy** is an intermediary that receives network traffic from one
side and communicates with the other side.

``` text
Client
   ↓
Proxy
   ↓
Server
```

Instead of:

``` text
Client ─────────→ Server
```

traffic becomes:

``` text
Client ─→ Proxy ─→ Server
```

Common proxy functions include:

``` text
Traffic control
Security
Authentication
Caching
Logging
Load distribution
TLS handling
Protocol-aware routing
```

------------------------------------------------------------------------

## 2. Where Does a Proxy Operate?

A proxy can operate at different layers.

### L4 Proxy

**Layer 4 = Transport layer**

Usually works with:

``` text
TCP
UDP
IP addresses
Ports
Connection state
```

It can route based on:

``` text
Source IP
Destination IP
Source port
Destination port
Protocol
```

It does not need to understand HTTP URLs or headers.

### L7 Proxy

**Layer 7 = Application layer**

For HTTP, it can understand:

``` text
HTTP method
URL/path
Host
Headers
Cookies
HTTP status
```

Example:

``` text
/api/users     → API servers
/images/*      → Image servers
/static/*      → Static content servers
```

------------------------------------------------------------------------

## 3. Forward Proxy

A **forward proxy** represents the **client side**.

``` text
Client
   ↓
Forward Proxy
   ↓
Internet / Server
```

The server may see the proxy as the source rather than the original
client.

Common uses:

``` text
Corporate Internet access
Content filtering
Access control
Caching
Monitoring
Privacy
```

Example:

``` text
Employee laptop
      ↓
Corporate proxy
      ↓
Internet
```

The client is normally configured to use the proxy.

------------------------------------------------------------------------

## 4. Reverse Proxy

A **reverse proxy** represents the **server side**.

``` text
Client
   ↓
Reverse Proxy
   ↓
Backend servers
```

The client normally thinks it is communicating with the service.

Example:

``` text
Client
   ↓
api.example.com
   ↓
Reverse proxy
   ↓
┌────────┬────────┬────────┐
Backend  Backend  Backend
  A        B        C
```

Common uses:

``` text
Load balancing
TLS termination
Authentication
Caching
Routing
Rate limiting
Security
```

------------------------------------------------------------------------

## 5. Forward vs Reverse Proxy
``` text
  -----------------------------------------------------------------------
                          Forward Proxy           Reverse Proxy
  ----------------------- ----------------------- -----------------------
  Represents              Client                  Server

  Typical location        Client network          Service/data-center
                                                  edge

  Client knows it?        Usually yes             Usually no

  Main purpose            Control outbound        Protect/manage inbound
                          traffic                 services

  Example                 Corporate proxy         Web/API gateway
  -----------------------------------------------------------------------
```

``` text
Forward:

Client → Proxy → Internet


Reverse:

Internet → Proxy → Backend
```

------------------------------------------------------------------------

## 6. L4 Proxy

An **L4 proxy** works at the transport layer.

``` text
Client
192.168.1.10:50000
      ↓
L4 Proxy
      ↓
Server
10.0.0.20:443
```

It can make decisions using:

``` text
IP
Port
TCP/UDP
Connection state
```

For TCP, it does not need to understand:

``` http
GET /users
Host: example.com
Cookie: ...
```

It can forward connections without understanding the HTTP application
protocol.

------------------------------------------------------------------------

## 7. L7 Proxy

An **L7 proxy** understands the application protocol.

For HTTP:

``` text
Client
   ↓
L7 Proxy
   ↓
Backend
```

It can inspect:

``` text
Host
Path
Method
Headers
Cookies
```

Example routing:

``` text
/api/*
   ↓
API service

/images/*
   ↓
Image service

/static/*
   ↓
Static content service
```

This is called **content-based routing**.

------------------------------------------------------------------------

## 8. L4 vs L7
``` text
                         L4 Proxy    L7 Proxy
  ---------------------- ----------- ----------------
  Layer                  Transport   Application
  Understands HTTP       No          Yes
  Sees ports/IPs         Yes         Yes
  Path-based routing     No          Yes
  Header-based routing   No          Yes
  Typical protocols      TCP/UDP     HTTP/gRPC/etc.
```

Mental model:

``` text
L4
 ↓
"Where is this connection going?"

L7
 ↓
"What does this application request mean?"
```

------------------------------------------------------------------------

## 9. TLS Termination

A reverse proxy can terminate TLS.

Without TLS termination at the proxy:

``` text
Client
   ↓ HTTPS
Backend
```

With TLS termination:

``` text
Client
   ↓ HTTPS
Reverse Proxy
   ↓ HTTP / HTTPS
Backend
```

The proxy handles:

``` text
TLS handshake
Certificate
Encryption/decryption
```

Example:

``` text
Client
   ↓
https://api.example.com
   ↓
Reverse Proxy
   ↓
HTTP
   ↓
Backend
```

This can centralize certificate management.

------------------------------------------------------------------------

## 10. TLS Passthrough vs Termination

### TLS Termination

``` text
Client
   ↓ HTTPS
Proxy
   ↓ HTTP / HTTPS
Backend
```

The proxy decrypts TLS and can inspect HTTP when the backend connection
is HTTP or when TLS is re-established separately.

### TLS Passthrough

``` text
Client
   ↓ HTTPS
Proxy
   ↓ HTTPS
Backend
```

The proxy forwards encrypted traffic without decrypting the application
data.

Therefore:

``` text
Termination
 ↓
Proxy can inspect HTTP

Passthrough
 ↓
Proxy cannot inspect encrypted HTTP content
```

------------------------------------------------------------------------

## 11. Reverse Proxy as Load Balancer

A reverse proxy can distribute requests across multiple backends.

``` text
                 Reverse Proxy
                 /     |                     ↓      ↓      ↓
              App A  App B  App C
```

Possible algorithms include:

``` text
Round robin
Least connections
Weighted routing
Hash-based routing
```

Example:

``` text
Request 1 → App A
Request 2 → App B
Request 3 → App C
Request 4 → App A
```

------------------------------------------------------------------------

## 12. Health Checks

A reverse proxy/load balancer can check backend health.

``` text
Proxy
  |
  ├── App A → healthy
  ├── App B → unhealthy
  └── App C → healthy
```

If App B fails its health check:

``` text
New request
     ↓
Proxy
     ↓
App A / App C
```

Traffic can temporarily avoid the unhealthy backend.

------------------------------------------------------------------------

## 13. Proxy Use Cases

Reverse-proxy uses:

``` text
Load balancing
TLS termination
Authentication
Rate limiting
Caching
Compression
Routing
Observability
Security filtering
```

Forward-proxy uses:

``` text
Outbound access control
Content filtering
Caching
Corporate policy enforcement
Logging
```

------------------------------------------------------------------------

## 14. API Gateway Basics

An **API gateway** is a specialized entry point for APIs.

``` text
Clients
   |
   ↓
API Gateway
   |
   ├── User Service
   ├── Order Service
   ├── Payment Service
   └── Product Service
```

It can provide centralized functions such as:

``` text
Authentication
Authorization
Routing
Rate limiting
TLS termination
Request validation
Logging
Metrics
```

An API gateway often uses reverse-proxy functionality.

> A reverse proxy and an API gateway are related, but not identical. A
> gateway usually adds API-specific policies and management features.

------------------------------------------------------------------------

## 15. Example: Web Application

``` text
Internet
    ↓
DNS
    ↓
Public IP
    ↓
Reverse Proxy
    ↓
┌───────────────┐
│ Load Balancer │
└───────┬───────┘
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
App A  App B  App C
```

The reverse proxy can handle:

``` text
TLS
Routing
Health checks
Load balancing
Logging
```

------------------------------------------------------------------------

## 16. Example: Path-Based Routing

A single public hostname can expose multiple services:

``` text
api.example.com/users
api.example.com/orders
api.example.com/products
```

Reverse proxy:

``` text
/users/*
    ↓
User service

/orders/*
    ↓
Order service

/products/*
    ↓
Product service
```

The proxy inspects the HTTP path and routes accordingly.

This requires an L7-aware proxy.

------------------------------------------------------------------------

## 17. Practical: Use an HTTP Proxy

If an HTTP proxy is configured:

``` powershell
curl.exe -x http://proxy.example.com:8080 https://example.com
```

Meaning:

``` text
-x
 ↓
Use the specified proxy
```

The exact output depends on the proxy and destination.

------------------------------------------------------------------------

## 18. Practical: Inspect a Reverse Proxy Response

Run:

``` powershell
curl.exe -I https://example.com
```

Example:

``` text
HTTP/1.1 200 OK
Server: example-proxy
Content-Type: text/html
```

Proxy-specific headers vary by implementation, so do not assume a
particular `Server` header will always appear.

------------------------------------------------------------------------

## 19. Proxy Mental Model

Forward proxy:

``` text
                Internet
                   ↑
                   |
Client → Forward Proxy
```

Reverse proxy:

``` text
Client
  ↓
Reverse Proxy
  ↓
Backend servers
```

L4:

``` text
TCP/UDP connection
        ↓
       L4
        ↓
Backend
```

L7:

``` text
HTTP request
        ↓
       L7
        ↓
Inspect path/headers
        ↓
Backend
```

------------------------------------------------------------------------

## 20. Summary

``` text
Proxy
 ↓
Intermediary between client and server

Forward proxy
 ↓
Represents client

Reverse proxy
 ↓
Represents server

L4 proxy
 ↓
Works with TCP/UDP connections

L7 proxy
 ↓
Understands application protocols such as HTTP

TLS termination
 ↓
Proxy handles TLS encryption/decryption

Load balancing
 ↓
Distributes traffic across backends

API gateway
 ↓
Reverse-proxy-style API entry point
 + authentication
 + routing
 + rate limiting
 + other policies
```

Key distinction:

``` text
Forward Proxy
"Client → proxy → Internet"


Reverse Proxy
"Internet → proxy → service"


L4
"Understand connection"


L7
"Understand application request"
```

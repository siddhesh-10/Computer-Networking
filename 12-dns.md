# Networking --- Phase 12: DNS

> **Goal:** Understand how DNS translates domain names into IP addresses
> and how resolvers, recursive servers, authoritative servers, and DNS
> records work together.

## 1. What Is DNS?

**DNS (Domain Name System)** maps names to network information.

``` text
www.example.com
       ↓
      DNS
       ↓
93.184.216.34
```

DNS is a distributed, hierarchical system.

------------------------------------------------------------------------

## 2. DNS Resolver

A **DNS resolver** is the component that asks DNS servers for an answer.

``` text
Browser / Application
        ↓
OS DNS resolver
        ↓
DNS server
        ↓
IP address
```

The configured DNS server may be provided by an ISP, company, cloud
provider, or public DNS service.

------------------------------------------------------------------------

## 3. Recursive DNS

A **recursive resolver** performs DNS lookups on behalf of the client.

If the answer is not cached:

``` text
Client
  ↓
Recursive Resolver
  ↓
Root
  ↓
.com TLD
  ↓
example.com Authoritative DNS
  ↓
IP address
  ↓
Client
```

The client normally does not contact every DNS server directly.

------------------------------------------------------------------------

## 4. Authoritative DNS

An **authoritative DNS server** contains the official DNS records for a
domain/zone.

For example:

``` text
www.example.com → 93.184.216.34
```

Key distinction:

``` text
Recursive resolver
 ↓
Finds the answer

Authoritative server
 ↓
Provides authoritative zone data
```

------------------------------------------------------------------------

## 5. Root DNS Servers

At the top of the DNS hierarchy are the **root DNS servers**.

They normally do not return the final IP for:

``` text
www.example.com
```

Instead, they direct the resolver toward the appropriate TLD
infrastructure.

``` text
Root
 ↓
.com
 ↓
example.com
 ↓
www.example.com
```

The root DNS system is distributed globally.

------------------------------------------------------------------------

## 6. TLD Servers

**TLD (Top-Level Domain)** servers handle domains such as:

``` text
.com
.org
.net
.in
```

For:

``` text
www.example.com
```

the resolver can ask the `.com` infrastructure:

``` text
"Where are the authoritative servers for example.com?"
```

The answer points toward the authoritative name servers.

------------------------------------------------------------------------

## 7. Complete DNS Resolution

Simplified lookup:

``` text
1. Application
       ↓
2. Recursive resolver
       ↓
3. Root DNS
       ↓
4. .com TLD DNS
       ↓
5. example.com authoritative DNS
       ↓
6. IP address
       ↓
7. Resolver caches answer
       ↓
8. Application receives IP
```

If the resolver already has a valid cached answer, most of these steps
are skipped.

------------------------------------------------------------------------

## 8. DNS Records

Common DNS record types:

``` text
A
AAAA
CNAME
NS
TXT
PTR
```

------------------------------------------------------------------------

## 9. A Record

An **A record** maps a hostname to an IPv4 address.

``` text
www.example.com
       ↓
A
       ↓
192.0.2.10
```

So:

``` text
Hostname → IPv4
```

------------------------------------------------------------------------

## 10. AAAA Record

An **AAAA record** maps a hostname to an IPv6 address.

``` text
www.example.com
       ↓
AAAA
       ↓
2001:db8::10
```

Remember:

``` text
A    → IPv4
AAAA → IPv6
```

------------------------------------------------------------------------

## 11. CNAME Record

A **CNAME (Canonical Name)** creates an alias from one DNS name to
another.

``` text
www.example.com
       ↓
CNAME
       ↓
example.com
       ↓
A / AAAA
       ↓
IP address
```

A CNAME points to another **name**, not directly to an IP address.

------------------------------------------------------------------------

## 12. NS Record

An **NS (Name Server)** record identifies authoritative name servers for
a DNS zone.

Example:

``` text
example.com
    ↓
NS
    ↓
ns1.dns-provider.example
ns2.dns-provider.example
```

------------------------------------------------------------------------

## 13. TXT Record

A **TXT record** stores text associated with a DNS name.

Common uses include:

``` text
Domain verification
SPF information
DKIM-related information
DMARC-related information
Service configuration
```

Example:

``` text
example.com
    ↓
TXT
    ↓
"verification=abc123"
```

The meaning depends on the service using it.

------------------------------------------------------------------------

## 14. PTR Record

A **PTR (Pointer)** record is commonly used for reverse DNS.

Normal DNS:

``` text
Name → IP
```

PTR:

``` text
IP → Name
```

Example:

``` text
192.0.2.10
    ↓
PTR
    ↓
www.example.com
```

Reverse IPv4 DNS commonly uses:

``` text
in-addr.arpa
```

------------------------------------------------------------------------

## 15. DNS Caching

DNS answers can be cached by:

``` text
Browser
Operating system
Local DNS service
Recursive resolver
```

Example:

``` text
First lookup
Client → Resolver → Authoritative DNS
                       ↓
                    IP + TTL
                       ↓
                    Cache

Later lookup
Client → Resolver
          ↓
       Cached answer
```

Caching reduces latency, DNS traffic, and load on authoritative servers.

------------------------------------------------------------------------

## 16. DNS TTL

DNS **TTL (Time To Live)** specifies how long a cached DNS answer may be
retained.

Example:

``` text
A record:

www.example.com
TTL = 300
IP  = 192.0.2.10
```

Conceptually:

``` text
300 seconds
     ↓
Cache can use answer
     ↓
TTL expires
     ↓
Resolver may query again
```

Do not confuse DNS TTL with the IP packet TTL:

``` text
DNS TTL
 ↓
DNS cache lifetime

IP TTL
 ↓
Packet lifetime across routers
```

------------------------------------------------------------------------

## 17. DNS Changes and Caching

Suppose:

``` text
Old IP = 192.0.2.10
New IP = 192.0.2.20
```

A resolver may continue returning the old IP while its cached record is
still valid.

``` text
Change DNS record
      ↓
Existing cached answers remain
      ↓
TTL expires
      ↓
Resolver obtains new answer
```

Lowering TTL before a planned change can reduce how long old answers
remain cached.

------------------------------------------------------------------------

## 18. DNS Failures

Common DNS failures include:

``` text
Name does not resolve
Wrong IP returned
DNS server unreachable
Timeout
NXDOMAIN
SERVFAIL
```

### NXDOMAIN

The requested DNS name does not exist according to the responding DNS
infrastructure.

``` text
unknown.example.com
        ↓
NXDOMAIN
```

### SERVFAIL

The resolver failed to obtain a usable answer.

Possible causes include:

``` text
Authoritative DNS failure
DNSSEC validation failure
Timeouts
Misconfiguration
```

------------------------------------------------------------------------

## 19. Practical: Resolve a Domain

On Windows:

``` powershell
nslookup example.com
```

Example:

``` text
Server:  dns.example.net
Address: 192.168.1.1

Non-authoritative answer:
Name:    example.com
Address: 93.184.216.34
```

Meaning:

``` text
Server
 ↓
DNS server used for the query

Name
 ↓
Requested domain

Address
 ↓
Resolved IP
```

`Non-authoritative answer` commonly means the response came from a
recursive resolver/cache rather than directly from the authoritative
server.

------------------------------------------------------------------------

## 20. Practical: Query Record Types

A record:

``` powershell
nslookup -type=A example.com
```

AAAA:

``` powershell
nslookup -type=AAAA example.com
```

NS:

``` powershell
nslookup -type=NS example.com
```

TXT:

``` powershell
nslookup -type=TXT example.com
```

The exact output depends on the domain.

------------------------------------------------------------------------

## 21. Practical: PowerShell DNS Lookup

Run:

``` powershell
Resolve-DnsName example.com
```

Example:

``` text
Name        Type  TTL  Section  IPAddress
----        ----  ---  -------  ---------
example.com A     300  Answer   93.184.216.34
```

Meaning:

``` text
Name
 ↓
DNS name

Type
 ↓
Record type

TTL
 ↓
Cache lifetime information

IPAddress
 ↓
Resolved IP
```

For IPv6:

``` powershell
Resolve-DnsName example.com -Type AAAA
```

------------------------------------------------------------------------

## 22. Practical: Reverse DNS

Run:

``` powershell
nslookup 8.8.8.8
```

Possible output:

``` text
Name:    dns.google
Address: 8.8.8.8
```

Conceptually:

``` text
IP address
    ↓
PTR lookup
    ↓
Hostname
```

A reverse lookup can fail if no PTR record exists.

------------------------------------------------------------------------

## 23. DNS Troubleshooting

When a domain does not work:

``` text
Application
    ↓
DNS lookup
    ↓
Does name resolve?
   /  No   Yes
 ↓     ↓
Check   Connect to
DNS     returned IP
```

Useful commands:

``` powershell
nslookup example.com
Resolve-DnsName example.com
```

If DNS resolves but the application still fails, the problem may instead
be:

``` text
Routing
Firewall
TCP
TLS
Application
```

DNS success does not guarantee application connectivity.

------------------------------------------------------------------------

------------------------------------------------------------------------

## 24. What Happens When You Enter a URL in a Browser?

A URL such as:

``` text
https://www.example.com/products
```

triggers several networking steps.

### Step 1 --- Parse the URL

The browser identifies:

``` text
Scheme   = https
Host     = www.example.com
Path     = /products
Port     = 443 (default for HTTPS)
```

### Step 2 --- Check Local Caches

Before asking a DNS resolver, the browser/OS may already know the IP
through:

``` text
Browser DNS cache
OS DNS cache
Local resolver cache
```

If there is a valid cached answer:

``` text
www.example.com
       ↓
Cached IP
```

DNS lookup can be avoided.

### Step 3 --- DNS Resolution

If the IP is not cached:

``` text
Browser
   ↓
OS resolver
   ↓
Recursive DNS resolver
   ↓
Root
   ↓
TLD (.com)
   ↓
Authoritative DNS
   ↓
A / AAAA record
   ↓
IP address
```

Example:

``` text
www.example.com
       ↓
93.184.216.34
```

### Step 4 --- Decide the Network Path

The OS now has a destination IP.

It checks the routing table:

``` text
Destination IP
      ↓
Routing table
      ↓
Local network?
   /        \
 Yes         No
  ↓           ↓
Direct      Default
delivery    gateway
```

If the destination is remote, the host sends the first Ethernet frame
toward its router/default gateway.

### Step 5 --- ARP / Neighbor Resolution

For IPv4, the host may need ARP to find the MAC address of the next
local hop:

``` text
Destination is remote
        ↓
Need gateway MAC
        ↓
ARP
        ↓
Gateway MAC
```

For IPv6, Neighbor Discovery is used instead of ARP.

### Step 6 --- TCP Connection

For traditional HTTPS over TCP:

``` text
Client                         Server

SYN ------------------------->

    <------------------- SYN-ACK

ACK -------------------------->
```

Now the TCP connection is established.

> Modern HTTP/3 uses QUIC over UDP instead of TCP, so this step differs
> for HTTP/3.

### Step 7 --- TLS Handshake

Because the URL uses:

``` text
https://
```

the client establishes TLS security.

Conceptually:

``` text
TCP connection
      ↓
TLS handshake
      ↓
Certificate validation
      ↓
Session keys established
      ↓
Encrypted HTTP
```

The exact TLS exchange depends on the TLS version and connection state.

### Step 8 --- HTTP Request

The browser sends an HTTP request over the encrypted connection.

Simplified:

``` http
GET /products HTTP/1.1
Host: www.example.com
```

For HTTP/2 or HTTP/3, the wire representation is different, but the
application-level request concept remains.

### Step 9 --- Server Processing

The request may travel through:

``` text
Internet
   ↓
Load balancer
   ↓
Web server
   ↓
Application server
   ↓
Database / cache / other services
```

The server generates a response.

### Step 10 --- HTTP Response

Example:

``` text
HTTP/1.1 200 OK
Content-Type: text/html
```

followed by the response body.

``` text
Server
  ↓
HTTP response
  ↓
TLS encryption
  ↓
TCP/IP
  ↓
Browser
```

### Step 11 --- Browser Renders the Page

The browser receives the HTML and discovers additional resources:

``` text
HTML
 ↓
CSS
JavaScript
Images
Fonts
APIs
```

These can trigger additional DNS lookups and network requests.

For example:

``` text
www.example.com
      ↓
HTML
      ↓
cdn.example.com
api.example.com
images.example.com
```

Connections may be reused through HTTP keep-alive/connection pooling.

### Complete Mental Model

``` text
Enter URL
   ↓
Parse URL
   ↓
Check browser/OS DNS cache
   ↓
DNS resolution
   ↓
Get IP address
   ↓
Routing decision
   ↓
ARP / IPv6 Neighbor Discovery
   ↓
TCP 3-way handshake
   ↓
TLS handshake
   ↓
HTTP request
   ↓
Server / load balancer / application
   ↓
HTTP response
   ↓
Browser downloads resources
   ↓
Browser renders page
```

This is a useful end-to-end model because it connects the concepts
learned so far:

``` text
DNS
 ↓
IP addressing
 ↓
Routing
 ↓
ARP
 ↓
TCP
 ↓
TLS
 ↓
HTTP
```

## 25. DNS Mental Model

``` text
                 Root
                  ↓
                .com
                  ↓
             example.com
                  ↓
       Authoritative DNS servers
                  ↓
          www.example.com
                  ↓
             A / AAAA
                  ↓
                IP
```

Client-side path:

``` text
Application
    ↓
Resolver
    ↓
Cache hit?
   /  Yes  No
  ↓    ↓
 IP   DNS hierarchy
       ↓
      IP
       ↓
     Cache
       ↓
   Application
```

------------------------------------------------------------------------

## 26. Summary

``` text
DNS
 ↓
Name → network information

Resolver
 ↓
Requests/performs DNS resolution

Recursive DNS
 ↓
Finds answers on behalf of clients

Authoritative DNS
 ↓
Provides authoritative zone data

Root
 ↓
Directs queries toward TLD infrastructure

TLD
 ↓
Directs queries toward authoritative servers

A
 ↓
Hostname → IPv4

AAAA
 ↓
Hostname → IPv6

CNAME
 ↓
Name → another name

NS
 ↓
Authoritative name servers

TXT
 ↓
Text/service metadata

PTR
 ↓
IP → hostname

DNS cache
 ↓
Stores answers

DNS TTL
 ↓
Controls cache lifetime
```

Core flow:

``` text
                 DNS
                  |
          ┌───────┴────────┐
          ↓                ↓
       Client            Records
          ↓
       Resolver
          ↓
       Cache?
       /          Yes      No
      ↓        ↓
     IP      Root
              ↓
             TLD
              ↓
        Authoritative
              ↓
             IP
              ↓
            Cache
              ↓
           Client
```

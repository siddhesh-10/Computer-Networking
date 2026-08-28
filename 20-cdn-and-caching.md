# Networking — Phase 20: CDN & Caching

> **Goal:** Understand how CDNs and caches reduce latency, protect origin infrastructure, and serve content from locations closer to users.

## 1. What Is a CDN?

A **CDN (Content Delivery Network)** is a distributed network of servers that caches and serves content closer to users.

```text
                 Origin Server
                      |
              ┌───────┴───────┐
              ↓       ↓       ↓
           Edge A   Edge B   Edge C
              ↑       ↑       ↑
            Users   Users   Users
```

A CDN commonly sits between clients and the origin:

```text
Client
  ↓
CDN / Edge
  ↓
Origin
```

If the content is cached, the CDN can serve it without contacting the origin.

---

## 2. Where Does a CDN Fit?

A CDN is primarily an **application-layer / HTTP delivery system**, using lower networking layers underneath.

Typical HTTPS path:

```text
Browser
   ↓
HTTP
   ↓
TLS
   ↓
TCP
   ↓
IP
   ↓
CDN Edge
   ↓
Origin
```

Modern deployments may use:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
```

A CDN is infrastructure for delivering application content; it is not a new transport protocol.

---

## 3. Edge Server

An **edge server** is a CDN server located closer to users.

```text
User
 ↓
Nearby CDN Edge
 ↓
Origin
```

If the requested object is cached:

```text
User
 ↓
Edge
 ↓
Cached response
```

The origin is not contacted.

Benefits:

```text
Lower latency
Reduced origin traffic
Better scalability
Faster content delivery
```

---

## 4. Origin Server

The **origin server** is the authoritative backend that generates or stores the original content.

```text
Client
   ↓
CDN Edge
   ↓
Origin
   ↓
Application / Storage
```

Examples:

```text
HTML
Images
JavaScript
CSS
API responses
Video segments
Files
```

If the CDN cannot satisfy a request from cache, it may fetch the content from the origin.

---

## 5. Cache

A **cache** stores previously retrieved data so it can be reused.

Without cache:

```text
Client → Origin
Client → Origin
Client → Origin
```

With cache:

```text
Client → CDN Edge
             ↓
           Cache
```

The cache avoids unnecessary origin requests.

---

## 6. Cache Hit

A **cache hit** occurs when the requested object is available and usable in the cache.

```text
Client
   ↓
CDN Edge
   ↓
Cache HIT
   ↓
Cached object
   ↓
Client
```

Example:

```text
GET /logo.png

Cache: HIT
Origin: not contacted
```

This is generally faster than fetching the object from the origin.

---

## 7. Cache Miss

A **cache miss** occurs when the requested object is not available as a usable cached response.

```text
Client
   ↓
CDN Edge
   ↓
Cache MISS
   ↓
Origin
   ↓
Response
   ↓
CDN Cache
   ↓
Client
```

Conceptually:

```text
First request:
MISS → Origin → Cache

Later request:
HIT → Cache
```

Exact behavior depends on HTTP and CDN configuration.

---

## 8. Cache Hit Ratio

A useful CDN metric is the **cache hit ratio**.

```text
Cache hit ratio =
Cache hits / Total cacheable requests
```

Example:

```text
1000 cacheable requests
800 cache hits
200 cache misses
```

```text
Hit ratio = 800 / 1000 = 80%
```

A higher hit ratio generally means more requests are served without reaching the origin.

---

## 9. TTL

**TTL (Time To Live)** determines how long cached content can remain fresh before it needs validation or re-fetching, depending on caching rules.

Example:

```http
Cache-Control: max-age=300
```

This allows the response to be considered fresh for:

```text
300 seconds = 5 minutes
```

Conceptually:

```text
Object cached
     ↓
Fresh for 300 seconds
     ↓
Freshness expires
     ↓
Validation / re-fetch may occur
```

TTL is not necessarily the only factor; cache-control directives and CDN configuration also matter.

---

## 10. Cache-Control

`Cache-Control` defines how a response may be cached and reused.

Example:

```http
Cache-Control: public, max-age=3600
```

Meaning:

```text
public
 ↓
May be stored by shared caches

max-age=3600
 ↓
Fresh for up to 3600 seconds
```

Important directives:

```text
private
no-cache
no-store
must-revalidate
s-maxage

Directive	Simple meaning
private	Only private/browser caches may store it
no-store	Don't store it
no-cache	Can store, but must validate before reuse
must-revalidate	Once stale, must validate before reuse
s-maxage	Freshness lifetime specifically for shared caches/CDNs
```

Important distinction:

```text
no-cache
 ↓
May store, but must validate before reuse

no-store
 ↓
Do not store the response
```

---

## 11. Browser Cache vs CDN Cache

There can be multiple cache layers:

```text
Browser Cache
      ↓
CDN Edge Cache
      ↓
Origin
```

If the browser has a fresh object:

```text
Browser
 ↓
Cached response
```

No CDN request is required.

If browser cache misses:

```text
Browser
   ↓
CDN
   ↓
Cache HIT
```

If the CDN also misses:

```text
Browser
   ↓
CDN
   ↓
Cache MISS
   ↓
Origin
```

---

## 12. ETag

An **ETag** identifies a particular representation of a resource.

Response:

```http
ETag: "abc123"
```

Later the client can send:

```http
If-None-Match: "abc123"
```

If the resource has not changed:

```text
HTTP/1.1 304 Not Modified
```

No new response body is required.

Flow:

```text
Client
  ↓
If-None-Match: "abc123"
  ↓
Server / CDN
  ↓
304 Not Modified
```

This reduces unnecessary data transfer.

---

## 13. Conditional Requests

Common validators:

```text
ETag
Last-Modified
```

Example:

```http
If-None-Match: "abc123"
```

or:

```http
If-Modified-Since: Wed, 26 Aug 2026 10:00:00 GMT
```

Possible result:

```text
304 Not Modified
```

Meaning:

```text
Cached copy is still valid.
```

---

## 14. Cache Invalidation

**Cache invalidation** removes or makes cached content unusable before normal expiration.

Problem:

```text
Old content
    ↓
Application updated
    ↓
Cached content still exists
    ↓
Users may receive old content
```

Common approaches:

```text
Wait for TTL
Explicit purge
Versioned URLs
Cache tags/keys
Revalidation
```

---

## 15. Versioned URLs

A common way to avoid stale static assets is changing the URL when the content changes.

Instead of:

```text
/app.js
```

use:

```text
/app.a81f3c.js
```

After a new release:

```text
/app.b91d22.js
```

The URL changes, so the CDN treats it as a different cache object.

This is commonly called **cache busting**.

---

## 16. Origin Shield

An **origin shield** is an additional caching layer between CDN edges and the origin.

Without a shield:

```text
Edge A ─┐
Edge B ─┼──→ Origin
Edge C ─┘
```

With a shield:

```text
Edge A ─┐
Edge B ─┼──→ Origin Shield ──→ Origin
Edge C ─┘
```

If many edges miss the same object, the shield can reduce repeated origin requests.

```text
Many edge misses
       ↓
One shield cache
       ↓
Fewer origin requests
```

---

## 17. Edge Caching

Static content is a common CDN caching candidate.

Examples:

```text
Images
CSS
JavaScript
Fonts
Videos
Downloads
```

Flow:

```text
User
 ↓
Edge
 ↓
Cache HIT
 ↓
Content
```

Dynamic or personalized content may require different caching rules.

For example:

```http
GET /profile
Cookie: user=123
```

should not automatically be shared across users.

---

## 18. What Determines a Cache Key?

A cache needs to identify objects.

A simplified cache key may contain:

```text
Scheme
Host
Path
Query string
```

Example:

```text
https://example.com/images/a.png
```

may map to:

```text
example.com + /images/a.png
```

Depending on configuration, query parameters or other request attributes can also affect the key.

Therefore:

```text
/image.png?v=1
/image.png?v=2
```

may be treated as different cache objects.

---

## 19. CDN Request Flow

### Cache Hit

```text
User
 ↓
Edge
 ↓
Cache HIT
 ↓
Cached response
 ↓
User
```

Origin is not contacted.

### Cache Miss

```text
User
 ↓
Edge
 ↓ MISS
Origin
 ↓
Response
 ↓
Edge caches response
 ↓
User
```

---

## 20. Practical: Inspect Cache Headers

Run:

```powershell
curl.exe -I https://example.com
```

Example:

```text
HTTP/1.1 200 OK
Cache-Control: public, max-age=300
ETag: "abc123"
Age: 120
```

Meaning:

```text
Cache-Control
 ↓
Defines caching behavior

ETag
 ↓
Resource validator

Age: 120
 ↓
Response has been in a shared cache for about 120 seconds
```

Actual headers vary by server/CDN.

---

## 21. Practical: Inspect a Response Verbosely

```powershell
curl.exe -I -v https://example.com
```

Possible output:

```text
> HEAD / HTTP/1.1
> Host: example.com

< HTTP/1.1 200 OK
< Cache-Control: public, max-age=300
< ETag: "abc123"
```

The `-v` option shows additional connection, request, and response details.

Exact output depends on the endpoint.

---

## 22. Practical: Test Conditional Requests

First inspect the ETag:

```powershell
curl.exe -I https://example.com
```

Suppose:

```text
ETag: "abc123"
```

Then send:

```powershell
curl.exe -I -H 'If-None-Match: "abc123"' https://example.com
```

Possible result:

```text
HTTP/1.1 304 Not Modified
```

Meaning:

```text
The cached representation is still valid.
```

A server may instead return `200 OK` if the validator is not accepted or the resource has changed.

---

## 23. CDN vs Origin

| | CDN / Edge | Origin |
|---|---|---|
| Location | Distributed | Central/backend |
| Main purpose | Fast delivery | Generate/store content |
| Cache content | Yes | Source of truth |
| User proximity | Usually close | May be far away |
| Handles every request? | Ideally many | Fewer with effective caching |

Mental model:

```text
Origin
 ↓
CDN
 ↓
Edge
 ↓
Users
```

---

## 24. CDN Benefits

CDNs can provide:

```text
Lower latency
Higher scalability
Reduced origin bandwidth
Better availability
Traffic absorption
Edge caching
TLS termination
DDoS protection
```

Exact capabilities depend on the CDN provider and configuration.

---

## 25. CDN Mental Model

```text
                         Origin
                           |
                    Origin Shield
                           |
              ┌────────────┼────────────┐
              ↓            ↓            ↓
           Edge A        Edge B        Edge C
              ↑            ↑            ↑
           Users        Users         Users
```

Request:

```text
User
 ↓
Selected Edge
 ↓
Cache?
 ├── HIT  → Return cached object
 │
 └── MISS → Shield/origin
              ↓
           Response
              ↓
           Cache
              ↓
            User
```

---

## 26. Summary

```text
CDN
 ↓
Distributed content-delivery infrastructure

Edge server
 ↓
CDN server closer to users

Origin server
 ↓
Authoritative backend/content source

Cache hit
 ↓
Object served from cache

Cache miss
 ↓
Object fetched from upstream/origin

TTL
 ↓
Controls freshness lifetime

Cache-Control
 ↓
HTTP caching rules

ETag
 ↓
Resource validator

Cache invalidation
 ↓
Remove or invalidate cached content

Origin shield
 ↓
Intermediate cache protecting origin

Edge caching
 ↓
Store content at CDN edge
```

Core flow:

```text
User
  ↓
CDN Edge
  ↓
Cache HIT?
 ├── Yes → Return cached content
 │
 └── No
      ↓
   Origin Shield
      ↓
    Origin
      ↓
   Cache response
      ↓
     User
```

> **Key idea:** A CDN moves frequently requested content closer to users and reduces how often requests have to travel back to the origin.

# Networking — Phase 31: AWS Load Balancing & CDN

> **Goal:** Understand how AWS distributes traffic across targets and how CloudFront delivers content from edge locations.

## 1. Load Balancing

A load balancer distributes incoming traffic across multiple targets, such as EC2 instances, containers, or IP addresses.

```text
Clients
   ↓
Load Balancer
   ├── Target A
   ├── Target B
   └── Target C
```

Benefits include higher availability, scalability, and health-based traffic distribution.

---

## 2. ALB vs NLB vs GWLB

| AWS service | Layer | Main purpose |
|---|---|---|
| **Application Load Balancer (ALB)** | Layer 7 | HTTP/HTTPS routing using host, path, headers, and other request properties |
| **Network Load Balancer (NLB)** | Layer 4 | High-performance TCP, UDP, and TLS traffic handling |
| **Gateway Load Balancer (GWLB)** | Layer 3 integration / Layer 4 traffic handling | Deploy and scale virtual network appliances, such as firewalls |

### ALB example

```text
example.com/api/*  → API targets
example.com/admin/* → Admin targets
```

ALB understands HTTP requests, so it can route based on URL paths or hostnames.

### NLB example

```text
TCP :443 → Target group
UDP :5000 → Target group
```

NLB operates at the connection/transport level and does not route based on HTTP paths.

### GWLB example

```text
Incoming traffic
       ↓
Gateway Load Balancer
       ↓
Firewall / inspection appliance
       ↓
Destination
```

GWLB helps transparently insert virtual appliances into network traffic paths.

---

## 3. Layer 4 vs Layer 7

### Layer 4 — Transport

Uses information such as:

```text
Source/destination IP
Source/destination port
TCP / UDP
```

Example: route TCP traffic arriving on port 443 to a target group.

### Layer 7 — Application

Can inspect application-protocol information, such as HTTP:

```text
Host: api.example.com
Path: /users
Method: GET
Headers
```

Example:

```text
/api/*     → API service
/images/*  → Image service
```

Remember:

```text
NLB → Layer 4
ALB → Layer 7
```

GWLB is designed for appliance insertion and traffic inspection rather than ordinary HTTP request routing.

---

## 4. Target Groups and Health Checks

ALB and NLB forward traffic to **target groups**. Targets can include supported compute endpoints such as EC2 instances, IP addresses, or Lambda functions where supported.

A health check tests whether a target is ready to receive traffic.

Example:

```text
Load Balancer
   ├── Target A: Healthy   → receives traffic
   ├── Target B: Unhealthy → normally excluded
   └── Target C: Healthy   → receives traffic
```

For an ALB HTTP health check, the load balancer can request a configured path such as:

```text
/health
```

A healthy response helps AWS consider the target eligible for traffic. Configure health checks to test application readiness, not merely whether a process exists.

---

## 5. What Is a CDN?

A **Content Delivery Network (CDN)** delivers content through geographically distributed edge locations.

Without a CDN:

```text
User in India ─────────→ Origin in the US
```

With a CDN:

```text
User in India
      ↓
Nearby CloudFront edge location
      ↓ cache miss, if needed
Origin
```

If the content is cached at the edge, CloudFront can return it without fetching it from the origin.

Benefits:

- Lower latency for cached content
- Reduced origin load
- Better global content delivery
- Integration with AWS security and access controls

---

## 6. Amazon CloudFront

**Amazon CloudFront** is AWS's CDN.

It can deliver content from origins such as:

- Amazon S3
- Application Load Balancer
- EC2-hosted web servers
- Other publicly reachable HTTP servers
- Certain AWS services and custom origins

A **distribution** is the CloudFront configuration that defines origins, cache behaviors, TLS settings, and related delivery rules.

```text
Viewer / browser
      ↓ HTTPS
CloudFront edge location
      ↓ cache miss
Configured origin
```

---

## 7. Edge Locations and Caching

An **edge location** is a CloudFront point of presence that serves requests close to viewers. CloudFront also uses regional edge caches in its content-delivery hierarchy.

Simplified request flow:

```text
Viewer requests /logo.png
          ↓
CloudFront checks cache
      ┌───┴────┐
      ↓        ↓
   Hit       Miss
      ↓        ↓
Return      Fetch from
cached      origin
object        ↓
      └── Cache and return
```

A cache hit avoids an origin fetch. On a miss, CloudFront retrieves the object from the origin and may cache it according to the behavior and cache policy.

### Cache behavior depends on

- Cache policy and TTLs
- Cache key, including configured query strings, headers, and cookies
- Origin response cache headers, depending on policy
- Whether the response is eligible for caching

Dynamic and personalized responses need careful cache configuration to avoid serving one user's content to another.

---

## 8. CloudFront Origins

An **origin** is the source from which CloudFront retrieves content when it cannot serve a request from cache.

### S3 origin

Good for static assets:

```text
Browser
   ↓
CloudFront
   ↓
Private S3 bucket
```

Use **Origin Access Control (OAC)** to let CloudFront access a private S3 bucket without making the bucket publicly readable.

### ALB origin

Useful for dynamic applications:

```text
Browser
   ↓
CloudFront
   ↓
ALB
   ↓
Application targets
```

CloudFront can cache eligible responses, while uncached requests reach the application through the ALB.

---

## 9. CloudFront + S3

Typical architecture for a static website or frontend assets:

```text
User
  ↓ HTTPS
CloudFront
  ↓
S3 bucket
```

Common setup:

1. Store static files in S3.
2. Configure a CloudFront distribution with the S3 bucket as origin.
3. Configure OAC and the S3 bucket policy.
4. Attach a trusted TLS certificate for a custom domain, if needed.
5. Set cache policies and invalidate objects after deployments when necessary.

**Important:** For private S3 content, prefer the S3 REST origin with OAC. S3 website endpoints have different behavior and do not support OAC in the same way.

---

## 10. CloudFront + ALB

Typical architecture for a web application or API:

```text
User
  ↓ HTTPS
CloudFront
  ↓ HTTPS
Application Load Balancer
  ↓
EC2 / ECS / other targets
```

CloudFront can handle edge delivery and caching; ALB handles application-aware routing to targets.

Consider:

- Use HTTPS between CloudFront and the ALB.
- Configure the ALB to accept traffic only from intended sources where feasible.
- Use AWS WAF at the appropriate layer if web filtering is required.
- Forward only the headers, cookies, and query strings the application needs.
- Avoid caching authenticated or personalized responses unless the cache key and policy are designed for them.

CloudFront does not replace ALB health checks or application-level health management.

---

## 11. CloudFront TLS

TLS protects data in transit using HTTPS.

CloudFront has two relevant connections:

```text
Viewer ── TLS ──→ CloudFront ── TLS ──→ Origin
```

These are separate TLS connections.

### Viewer to CloudFront

For a custom CloudFront hostname such as:

```text
cdn.example.com
```

configure an appropriate certificate in **AWS Certificate Manager (ACM), in us-east-1 (N. Virginia)** for use with CloudFront.

### CloudFront to origin

Configure the origin protocol policy to use HTTPS where supported. For an HTTPS custom origin, the origin must present a certificate that CloudFront can validate and that matches the configured origin domain.

TLS from the viewer to CloudFront does not automatically mean the CloudFront-to-origin connection is encrypted.

---

## 12. Practical AWS Checks

These AWS CLI commands require AWS CLI configuration and suitable IAM permissions.

### Describe an ALB

```bash
aws elbv2 describe-load-balancers
```

Example-shaped output:

```json
{
  "LoadBalancers": [
    {
      "Type": "application",
      "Scheme": "internet-facing",
      "State": {"Code": "active"}
    }
  ]
}
```

Meaning:

- `Type: application` — this is an ALB.
- `Scheme: internet-facing` — it is configured for internet-facing access.
- `State.Code: active` — AWS reports the load balancer as active.

Actual output contains additional fields and depends on your account.

### List CloudFront distributions

```bash
aws cloudfront list-distributions
```

Example-shaped output:

```json
{
  "DistributionList": {
    "Quantity": 1,
    "Items": [
      {
        "DomainName": "d123example.cloudfront.net",
        "Enabled": true
      }
    ]
  }
}
```

Meaning:

- `DomainName` — the distribution's CloudFront hostname.
- `Enabled: true` — the distribution is configured as enabled; deployment status should also be checked before assuming changes are live.

---

## 13. Quick Comparison

| Requirement | Typical choice |
|---|---|
| Route HTTP requests by host/path | ALB |
| Handle TCP/UDP traffic at transport level | NLB |
| Insert scalable firewall/inspection appliances | GWLB |
| Cache and deliver global content | CloudFront |
| Serve static files from private S3 | CloudFront + S3 + OAC |
| Deliver web app traffic to backend targets | CloudFront + ALB |

---

## 14. Summary

```text
ALB
 ↓
Layer 7: HTTP/HTTPS-aware routing

NLB
 ↓
Layer 4: TCP/UDP/TLS traffic handling

GWLB
 ↓
Scale and insert virtual network appliances

CloudFront
 ↓
CDN with edge delivery and caching

CloudFront + S3
 ↓
Static content delivery

CloudFront + ALB
 ↓
Edge delivery + application routing

TLS
 ↓
Can protect viewer-to-CloudFront and CloudFront-to-origin separately
```

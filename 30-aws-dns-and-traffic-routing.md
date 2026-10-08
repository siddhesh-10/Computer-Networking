# Networking — Phase 30: AWS DNS & Traffic Routing

> **Goal:** Understand how Amazon Route 53 resolves DNS names and uses routing policies to direct users toward healthy or preferred endpoints.

## 1. Route 53

**Amazon Route 53** is AWS's managed DNS service.

**Layer:** Application layer / DNS.

DNS maps names to addresses or service endpoints:

```text
User
  │
  │ www.example.com?
  ▼
Route 53
  │
  │ DNS answer
  ▼
IP / AWS endpoint
  │
  ▼
Application
```

Route 53 can also provide:
- Domain registration
- DNS hosting
- Health checks
- Traffic routing policies

## 2. Hosted zones

A **hosted zone** is a container for DNS records for a domain.

Example:

```text
Hosted zone: example.com

example.com
├── A      → 203.0.113.10
├── AAAA   → IPv6 address
├── www    → application endpoint
└── api    → API endpoint
```

Two common types:

| Type | Purpose |
|---|---|
| Public hosted zone | DNS records publicly resolvable on the internet |
| Private hosted zone | DNS records resolvable inside associated VPCs |

A private hosted zone is useful for internal names:

```text
db.internal.example.com
        ↓
10.0.2.20
```

## 3. DNS records

Common records:

| Record | Purpose |
|---|---|
| A | Name → IPv4 address |
| AAAA | Name → IPv6 address |
| CNAME | Name → another DNS name |
| MX | Mail servers |
| TXT | Text / verification / policy data |
| NS | Authoritative name servers |
| Alias | Route 53-specific mapping to supported AWS resources |

Example:

```text
www.example.com
      │
      ▼
A / Alias record
      │
      ▼
ALB / CloudFront / IP
```

An **Alias** record is especially useful for supported AWS resources because it can point directly to AWS resources without requiring a CNAME at the zone apex.

## 4. Health checks

Route 53 health checks determine whether an endpoint is healthy according to configured checks.

Example:

```text
Route 53
   │
   ├── Health check → Region A → healthy
   │
   └── Health check → Region B → unhealthy
```

Health checks can monitor endpoints and can be combined with routing policies such as failover.

A health check does not fix an unhealthy server; it allows DNS routing decisions to avoid an endpoint when the configured health condition fails.

## 5. Simple routing

**Simple routing** returns a single resource or set of records without sophisticated traffic distribution.

```text
www.example.com
       │
       ▼
203.0.113.10
```

Use it when there is no need for weighted, latency, geographic, or failover-based decisions.

## 6. Weighted routing

**Weighted routing** distributes DNS responses according to configured weights.

Example:

```text
www.example.com

Region A → weight 90
Region B → weight 10
```

Conceptually:

```text
                 Route 53
                    │
             ┌──────┴──────┐
             ▼             ▼
        Endpoint A      Endpoint B
           90%             10%
```

Useful for:
- Canary releases
- Gradual migrations
- A/B-style traffic distribution
- Blue/green deployments

Weights control the probability of receiving an answer; they are not a guarantee that exactly that percentage of live requests will reach each endpoint.

## 7. Latency-based routing

**Latency-based routing** directs users to the AWS Region that Route 53 determines will provide the lowest network latency among configured resources.

```text
User in India
      │
      ▼
   Route 53
      │
      ├── us-east-1
      ├── eu-west-1
      └── ap-south-1  ← lowest measured latency
                         │
                         ▼
                      Endpoint
```

This is useful for globally distributed applications where user experience depends on network latency.

**Important:** It chooses based on measured network latency, not simply geographic distance.

## 8. Failover routing

**Failover routing** provides primary/secondary behavior.

```text
             Route 53
                │
        Health check
           │       │
        healthy   failed
           │       │
           ▼       ▼
        Primary  Secondary
```

Example:

```text
Primary:
  us-east-1

Secondary:
  eu-west-1
```

If the primary is considered unhealthy, Route 53 can return the secondary endpoint.

This is commonly used for disaster recovery.

## 9. Geolocation routing

**Geolocation routing** selects records based on the geographic location associated with the DNS query.

Example:

```text
India      → ap-south-1
Europe     → eu-west-1
North America → us-east-1
```

```text
                 Route 53
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      India       Europe      USA
        │           │           │
        ▼           ▼           ▼
    Endpoint A  Endpoint B  Endpoint C
```

Unlike latency routing, geolocation routing is based on location rather than measured network latency.

A default record can be configured for locations that do not match a more specific geolocation rule.

## 10. DNS-based traffic management

These policies can be combined with health checks and different endpoints:

```text
                       Route 53
                          │
                ┌─────────┴─────────┐
                │ Routing policy    │
                └─────────┬─────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
     Weighted           Latency          Failover
        │                 │                 │
        ▼                 ▼                 ▼
    Endpoint A        Region A/B       Primary/DR
```

A common global architecture:

```text
                     Users
                       │
                       ▼
                    Route 53
                       │
              Latency-based routing
                ┌──────┴──────┐
                ▼             ▼
             Region A       Region B
                │             │
               ALB           ALB
                │             │
             App tier       App tier
```

Route 53 makes the **DNS decision**. It does not proxy application traffic like a load balancer.

## 11. Practical AWS CLI

### List hosted zones

```powershell
aws route53 list-hosted-zones --query "HostedZones[*].[Id,Name,Config.PrivateZone]" --output table
```

Example:

```text
------------------------------------------------
|              ListHostedZones                 |
+------------------+----------------+-----------+
| /hostedzone/Z123 | example.com.   | False     |
| /hostedzone/Z456 | internal.local.| True      |
+------------------+----------------+-----------+
```

**Interpretation:** `False` indicates a public hosted zone; `True` indicates a private hosted zone.

### List DNS records

```powershell
aws route53 list-resource-record-sets --hosted-zone-id Z123456789 --output table
```

Example:

```text
------------------------------------------------
|          ListResourceRecordSets              |
+------------------+------+---------------------+
| example.com.     | A    | 203.0.113.10        |
| www.example.com. | A    | 203.0.113.20        |
+------------------+------+---------------------+
```

**Interpretation:** These records define the DNS answers returned for the names.

### Inspect health checks

```powershell
aws route53 list-health-checks --query "HealthChecks[*].[Id,HealthCheckConfig.Type,HealthCheckConfig.FullyQualifiedDomainName]" --output table
```

Example:

```text
-------------------------------------------------------
|                 ListHealthChecks                    |
+------------------+---------+------------------------+
| abc123           | HTTPS   | api.example.com        |
+------------------+---------+------------------------+
```

**Interpretation:** Route 53 is configured to monitor the HTTPS endpoint `api.example.com`.

## Key idea

Think of Route 53 as a **DNS decision engine**:

```text
DNS query
   ↓
Route 53
   ↓
Routing policy
   ↓
Optional health evaluation
   ↓
DNS answer
   ↓
Client connects to selected endpoint
```

Policy selection:

```text
One endpoint          → Simple
Traffic percentages   → Weighted
Lowest latency        → Latency-based
Primary + DR          → Failover
User geography        → Geolocation
```

DNS routing controls **where clients are directed**; the actual application connection happens afterward.

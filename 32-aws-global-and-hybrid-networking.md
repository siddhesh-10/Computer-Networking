# Phase 32: AWS Global & Hybrid Networking

## 1. AWS Global Accelerator

AWS Global Accelerator provides static anycast IP addresses and routes user traffic through the AWS global network to healthy endpoints.

- **Use it for:** globally distributed applications that need improved path performance and fast regional failover.
- **Endpoints:** Application Load Balancers, Network Load Balancers, EC2 instances, and Elastic IP addresses in supported configurations.
- **Health checks:** traffic is directed to healthy endpoints based on endpoint health and traffic dials.
- **Global Accelerator vs CloudFront:** Global Accelerator improves network-path routing for TCP/UDP traffic; CloudFront is a content-delivery network with HTTP caching and edge processing.

```text
User → Global Accelerator anycast IP → AWS global network
     → healthy regional endpoint → application
```

## 2. VPN fundamentals

A VPN creates an encrypted tunnel across an untrusted network, commonly the public internet.

- **Encryption** protects data in transit.
- **Authentication** verifies the tunnel peer or user.
- **Routing** determines which traffic uses the tunnel.
- **IPsec** is commonly used for site-to-site tunnels; client VPN services use a client connection and service-specific protocols.

A VPN protects traffic in transit; it does not automatically make the destination application secure.

## 3. AWS Site-to-Site VPN

Site-to-Site VPN connects an on-premises network to an AWS VPC over IPsec tunnels.

**Main components**
- **Customer gateway (CGW):** AWS representation of the on-premises VPN device and its public IP.
- **Virtual private gateway (VGW):** AWS VPN endpoint attached to a VPC.
- **Transit gateway (TGW):** central routing hub that can connect multiple VPCs and VPNs.
- **VPN tunnels:** AWS provides two tunnels per connection for availability.

**Typical traffic path**

```text
On-prem host → local router/firewall → IPsec tunnel over internet
             → VGW or TGW → VPC route table → EC2/private resource
```

Routes can be configured statically or exchanged using BGP, depending on the design. Security groups, network ACLs, and host firewalls still control access after routing is established.

## 4. AWS Client VPN

Client VPN gives individual users remote access to AWS or on-premises networks through a managed VPN endpoint.

- Common for employee and administrator access.
- Uses client authentication and authorization rules.
- Route entries determine reachable networks.
- Split-tunnel configuration controls whether all client traffic or only specified routes use the VPN.

**Site-to-Site vs Client VPN**

| Feature | Site-to-Site VPN | Client VPN |
|---|---|---|
| Connects | Networks | Individual client devices |
| Typical use | Office/data center to AWS | Remote employees to private resources |
| Tunnel peer | Network VPN device | VPN client |
| Routing scope | Network prefixes | Client routes and authorization rules |

## 5. AWS Direct Connect

Direct Connect provides a dedicated network connection from a customer location or colocation facility to AWS. It is not, by itself, an encrypted connection.

- **Private virtual interface (VIF):** access private AWS resources using private connectivity.
- **Public VIF:** access supported AWS public service endpoints over the connection.
- **Transit VIF:** connect through a Direct Connect gateway to supported transit architectures.
- **Direct Connect gateway:** helps connect a Direct Connect connection to supported virtual private gateways or transit gateways across Regions.
- **BGP:** exchanges routes across the connection.

**Why use it?**
- More consistent network performance than internet-based connectivity.
- Useful for large data transfers and hybrid workloads.
- Can reduce dependence on the public internet.

Plan for resilience with independent connections and locations where required. Direct Connect does not automatically provide encryption; use IPsec over the connection or another suitable encryption method when required.

## 6. Hybrid networking: on-premises ↔ AWS

Hybrid networking joins private on-premises networks with AWS VPCs.

```text
On-premises data center
        │
   Router / firewall
        │
  ┌─────┴───────────────┐
  │                     │
Site-to-Site VPN   Direct Connect
  │                     │
  └──────────┬──────────┘
             │
       VGW or Transit Gateway
             │
          VPC routes
             │
     Private AWS resources
```

Common design choices:
- **VPN only:** fast to deploy; traffic uses the internet as the underlay.
- **Direct Connect only:** dedicated connectivity; plan separate resilient paths.
- **Direct Connect + VPN:** dedicated primary path plus an encrypted or backup path, depending on architecture.
- **Transit Gateway:** central hub for many VPCs, VPN connections, and supported Direct Connect integrations.

Check both sides of the connection: AWS route tables, on-premises routes, BGP advertisements, firewall rules, security groups, and return routes. A one-way route is not enough for a working connection.

## 7. Global traffic architecture

A global application may combine DNS, edge services, regional load balancers, and private connectivity.

```text
Global users
    │
Route 53 DNS
    │
    ├── CloudFront → cached content / regional origin
    │
    └── Global Accelerator → healthy regional ALB or NLB
                                      │
                              Regional application
                                      │
                               Private data layer
```

Use each component for its specific job:
- **Route 53:** DNS-based routing and health-aware answers; clients still resolve and connect to the returned address.
- **CloudFront:** caches content at edge locations and can run supported edge functions.
- **Global Accelerator:** routes TCP/UDP traffic through the AWS global network to healthy endpoints using static anycast IPs.
- **ALB/NLB:** distributes traffic within a Region.
- **VPN / Direct Connect:** connects private networks to AWS; these are not substitutes for public-facing global traffic routing.

A global design also needs application-level decisions: data replication, session handling, failover policy, health-check behavior, and consistency between Regions.

## 8. Useful AWS CLI checks

Commands below are examples; replace placeholder IDs and Regions with your own. Permissions and available fields depend on the resource.

### List Site-to-Site VPN connections

```bash
aws ec2 describe-vpn-connections --region us-east-1
```

Example excerpt:

```json
{
  "VpnConnections": [
    {
      "VpnConnectionId": "vpn-0123456789abcdef0",
      "State": "available",
      "Type": "ipsec.1"
    }
  ]
}
```

Meaning: the VPN connection resource is available; verify tunnel status and route propagation separately.

### Inspect Direct Connect connections

```bash
aws directconnect describe-connections
```

Example excerpt:

```json
{
  "connections": [
    {
      "connectionId": "dxcon-example",
      "connectionState": "available",
      "bandwidth": "1Gbps"
    }
  ]
}
```

Meaning: the connection resource is available. This alone does not prove that BGP sessions and end-to-end routes are working.

### List Global Accelerator resources

```bash
aws globalaccelerator list-accelerators
```

Example excerpt:

```json
{
  "Accelerators": [
    {
      "Name": "global-app",
      "Enabled": true,
      "Status": "IN_PROGRESS"
    }
  ]
}
```

Meaning: the accelerator is enabled, but its status indicates provisioning or updating is still in progress.

## 9. Troubleshooting checklist

1. Confirm tunnel or physical connection state.
2. Verify BGP sessions and advertised prefixes, or check static routes.
3. Confirm VPC and on-premises routes in both directions.
4. Check security groups, network ACLs, and firewalls.
5. Verify DNS resolves to the intended global endpoint.
6. Check endpoint health and regional failover configuration.
7. Test application ports and return traffic—not only ICMP.

## Key takeaways

- Global Accelerator routes traffic over the AWS global network; CloudFront focuses on content delivery and caching.
- Site-to-Site VPN connects networks; Client VPN connects individual users.
- Direct Connect is dedicated connectivity, not automatic encryption.
- Hybrid connectivity requires correct routes and security controls on both sides.
- Global availability depends on healthy endpoints, resilient connectivity, and application/data-layer failover.

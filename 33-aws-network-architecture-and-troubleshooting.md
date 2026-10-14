# Phase 33: AWS Network Architecture & Troubleshooting

## 1. Production VPC architecture

A production VPC usually separates public entry points from private application and data tiers.

```text
                         Internet
                            │
                      Internet Gateway
                            │
                 Public subnets (AZ-a / AZ-b)
                    ALB / NAT Gateways
                            │
                 Private app subnets (AZ-a / AZ-b)
                    EC2 / containers / services
                            │
                  Private data subnets
                     databases / cache
```

Core design points:
- Use multiple Availability Zones (AZs) for resilience.
- Place internet-facing load balancers in public subnets; keep application and database resources private unless public access is explicitly required.
- Use one NAT Gateway per AZ when availability and AZ-independent egress justify the cost.
- Keep route tables, security groups, and network ACLs aligned with intended traffic paths.
- Use VPC endpoints for supported AWS services when they reduce public egress or simplify private access.

## 2. Multi-AZ architecture

Each AZ should have the subnets and routes required for its role. Avoid making an application tier depend on a single-AZ component if the service must survive an AZ failure.

Example:
- ALB spans public subnets in at least two AZs.
- App instances run in private subnets in multiple AZs.
- A NAT Gateway in each AZ gives local egress without relying on another AZ's NAT.
- Database resilience depends on the database service's Multi-AZ design and failover behavior.

**Important:** Multi-AZ improves resilience but does not guarantee application health. Check target health, dependencies, capacity, and failover behavior.

## 3. Multi-account networking

AWS Organizations environments commonly separate workloads into accounts while centralizing shared network services.

Common patterns:
- **Transit Gateway:** hub for routing among VPCs and supported VPN/Direct Connect connections.
- **VPC peering:** direct connection between two VPCs; it is not transitive.
- **AWS RAM:** shares supported resources, such as Transit Gateway attachments, across accounts.
- **Central egress/inspection VPC:** routes selected traffic through shared NAT or inspection appliances.
- **Cloud WAN:** managed option for building and operating larger global networks.

Plan CIDR ranges centrally. Overlapping VPC CIDRs complicate or prevent many routing patterns. Separate Transit Gateway route tables can segment environments, but attachments and routes must be explicitly associated and propagated as intended.

## 4. A systematic connectivity-debugging flow

Troubleshoot from source to destination, checking each layer in order.

1. **Name resolution:** does the hostname resolve to the expected IP?
2. **Route selection:** does the source subnet route table contain a route for the destination?
3. **Network path:** are IGW, NAT Gateway, Transit Gateway, peering, VPN, or endpoint paths configured correctly?
4. **Security controls:** do security groups, network ACLs, and host firewalls permit the flow?
5. **Listener and target:** is the destination listening on the expected IP and port? Is the load balancer target healthy?
6. **Return path:** can replies route back to the original source?
7. **Evidence:** use VPC Flow Logs, Reachability Analyzer, service metrics, and host-level tests.

A successful ping is not proof that a TCP application port works. ICMP may be blocked while TCP is allowed, or the reverse.

## 5. DNS failures

Check these in order:
- Is the hostname spelled correctly, and is the intended DNS zone authoritative?
- Does the record point to the expected target?
- Is the record stale due to caching or TTL?
- For private hosted zones, is the VPC association correct?
- Are VPC DNS support and DNS hostnames enabled where required?
- Do security rules allow DNS traffic to custom resolvers?
- If using Route 53 Resolver endpoints or forwarding rules, are routes and rules correct?

Useful commands:

```bash
nslookup app.example.com
```

Example output:

```text
Name:    app.example.com
Address: 203.0.113.20
```

Meaning: the resolver returned an address. Confirm that it is the expected address and that the endpoint is reachable.

```bash
dig app.example.com
```

Inspect the answer, status code, and TTL. `NXDOMAIN` means the responding DNS server says the name does not exist; `SERVFAIL` indicates the server failed to complete resolution.

## 6. Security Group issues

Security groups are stateful and are attached to supported network interfaces/resources.

Common causes:
- Inbound rule missing for the application port.
- Source CIDR or source security group is incorrect.
- A load balancer can reach a target only if target rules allow traffic from the load balancer's security group.
- Egress rules do not permit the required outbound connection.
- The wrong security group is attached to the resource.

Example for an ALB-backed application:
- ALB security group: inbound HTTPS from intended clients.
- App security group: inbound application port from the ALB security group, not from the entire internet.
- Database security group: inbound database port from the app security group.

Security groups do not replace route tables. A permitted security-group rule cannot create a network route.

## 7. Route table issues

For each subnet, identify its associated route table and find the most specific matching destination prefix.

Typical patterns:
- Public subnet: `0.0.0.0/0 → Internet Gateway` for IPv4 internet routing.
- Private subnet with internet egress: `0.0.0.0/0 → NAT Gateway`.
- VPC-to-VPC traffic: route to the intended peering, Transit Gateway, or other supported target.
- On-premises traffic: route to the VPN or Direct Connect/Transit Gateway path as designed.

Common mistakes:
- Updating the wrong route table.
- Missing the return route.
- Missing route propagation or Transit Gateway route-table association/propagation.
- Overlapping CIDRs.
- Sending private-subnet internet traffic directly to an Internet Gateway instead of through NAT.

## 8. NAT Gateway issues

A NAT Gateway allows resources in a private subnet to initiate supported outbound IPv4 connections; it does not allow unsolicited inbound connections from the internet.

Verify:
1. NAT Gateway state is `Available`.
2. The NAT Gateway is in a public subnet.
3. That public subnet has a default route to an Internet Gateway.
4. The private subnet route table sends internet-bound IPv4 traffic to the NAT Gateway.
5. The NAT Gateway has an Elastic IP and the VPC has the required internet path.
6. Network ACLs allow the required traffic and return ephemeral ports.
7. NAT capacity, connection counts, and error metrics are healthy.

For AZ resilience, use a NAT Gateway per AZ when appropriate and route each private subnet to its local NAT Gateway. Cross-AZ NAT routing may work but can add cross-AZ dependency and charges.

For IPv6 outbound-only internet access, evaluate an egress-only Internet Gateway; NAT Gateway is primarily for IPv4 translation.

## 9. VPC Flow Logs

VPC Flow Logs capture metadata about IP traffic to and from network interfaces, subnets, or VPCs. They help diagnose accepted and rejected traffic.

- Destinations include CloudWatch Logs, Amazon S3, and supported delivery integrations.
- Records can include source/destination IPs, ports, protocol, packet/byte counts, and an action such as `ACCEPT` or `REJECT`.
- Flow Logs are not packet captures and do not show application payloads.
- Delivery is not instantaneous; do not treat them as a real-time packet trace.

Example record excerpt:

```text
2 123456789012 eni-abc123 10.0.1.25 10.0.2.40 49152 443 6 10 840 1728900000 1728900060 ACCEPT OK
```

Interpretation: a flow record indicates accepted TCP traffic from `10.0.1.25:49152` to `10.0.2.40:443` during the recorded interval. Confirm exact field positions against the configured Flow Logs format.

An `ACCEPT` record does not prove the application returned a successful response. A `REJECT` record helps narrow down filtering but should be correlated with route and security configuration.

## 10. Useful AWS CLI checks

Replace example IDs and Region with your own. The snippets show example output, not live account state.

### Inspect VPC route tables

```bash
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-0123456789abcdef0
```

Example excerpt:

```json
{
  "RouteTables": [
    {
      "RouteTableId": "rtb-0123456789abcdef0",
      "Routes": [
        {
          "DestinationCidrBlock": "0.0.0.0/0",
          "GatewayId": "igw-0123456789abcdef0",
          "State": "active"
        }
      ]
    }
  ]
}
```

Meaning: this route table has an active IPv4 default route to an Internet Gateway. Confirm which subnets are associated with it.

### Check NAT Gateway state

```bash
aws ec2 describe-nat-gateways --filter Name=vpc-id,Values=vpc-0123456789abcdef0
```

Look for `State: available`. Also inspect subnet placement, Elastic IP allocation, and the private subnet's route table.

### Inspect security groups

```bash
aws ec2 describe-security-groups --group-ids sg-0123456789abcdef0
```

Review inbound and outbound permissions, then verify the group is attached to the intended resource or network interface.

### Inspect Flow Logs

```bash
aws ec2 describe-flow-logs --filter Name=resource-id,Values=vpc-0123456789abcdef0
```

Confirm the Flow Log exists, its traffic type, destination, and delivery status.

## 11. Production incident checklist

During an incident:
1. Establish scope: one host, one subnet, one AZ, one VPC, or multiple Regions?
2. Identify what changed: routes, security groups, DNS, deployments, certificates, or network appliances.
3. Test name resolution and the exact destination IP/port.
4. Verify routes in both directions and the relevant network path.
5. Check load balancer target health and application listener state.
6. Inspect VPC Flow Logs, Reachability Analyzer results, NAT metrics, and service logs.
7. Compare a working path with a failing path.
8. Apply the smallest reversible fix, then verify recovery from the original source.
9. Record root cause, impact, and preventive action.

Avoid broad emergency rules such as opening all ports to `0.0.0.0/0` unless there is a tightly controlled, justified reason. Prefer a narrow change with a rollback plan.

## Key takeaways

- Debug in order: DNS → routes → network path → security controls → listener/target → return path.
- Security groups are stateful; network ACLs are stateless.
- NAT Gateway enables outbound IPv4 connectivity from private subnets, not unsolicited inbound access.
- VPC Flow Logs provide flow metadata, not packet contents.
- Production resilience depends on multi-AZ design, correct routing, observability, and tested recovery procedures.

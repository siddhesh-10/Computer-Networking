# Networking — Phase 27: AWS Routing & Connectivity

> **Goal:** Understand how traffic enters, leaves, and moves within an AWS VPC.

## 1. The basic model

An AWS VPC is a logically isolated network in one Region. Each subnet belongs to one Availability Zone. **Route tables decide where packets go; security groups and network ACLs decide whether traffic is allowed.**

```text
Internet
   |
Internet Gateway (IGW)
   |
Public subnet ─── Route table: 0.0.0.0/0 → IGW
   |
   +── Private subnet ─── Route table: 0.0.0.0/0 → NAT Gateway
```

A NAT Gateway is placed in a public subnet; private-subnet outbound Internet traffic is routed through it.

## 2. Route tables

A route table maps destination CIDR ranges to targets. AWS uses the **most specific matching route** (longest-prefix match). Every subnet is associated with a route table; if not explicitly associated, it uses the VPC's main route table.

| Destination | Target | Meaning |
|---|---|---|
| `10.0.0.0/16` | `local` | Route within the VPC |
| `0.0.0.0/0` | Internet Gateway | Default IPv4 route to the Internet |
| `0.0.0.0/0` | NAT Gateway | Default IPv4 route via NAT for outbound access |

A route selects the path. It does **not** override security-group rules, network ACLs, or the destination's firewall.

## 3. Internet Gateway (IGW)

An IGW connects a VPC to the Internet. For an EC2 instance to communicate directly with the public IPv4 Internet, it generally needs:
- A subnet route such as `0.0.0.0/0 → IGW`.
- A public IPv4 address or Elastic IP.
- Security-group and network-ACL rules that permit the traffic.
- An application listening on the relevant port.

An IGW does not automatically make every instance public.

## 4. NAT Gateway

A NAT Gateway lets instances in a private subnet initiate connections to the public IPv4 Internet without accepting unsolicited inbound Internet connections through that NAT path.

```text
Private EC2
  → private route table: 0.0.0.0/0 → NAT Gateway
  → NAT Gateway in public subnet
  → public route table: 0.0.0.0/0 → IGW
  → Internet
```

The NAT Gateway translates addresses. Return traffic for connections initiated from inside can flow back; external hosts cannot normally start a new connection to a private instance through the NAT Gateway. A public NAT Gateway needs an Elastic IP and a route to an IGW. It is managed by AWS and billed based on usage.

## 5. Public and private IP addresses

| Address type | Meaning |
|---|---|
| Private IPv4 | Used for communication inside private networks, including VPC traffic |
| Public IPv4 | Publicly routable address associated with an AWS resource |
| Elastic IP (EIP) | Static public IPv4 address allocated to your AWS account and associated with a supported resource, such as an instance or NAT Gateway |

An EC2 instance can have both private and public IPv4 addresses: its private address is used within the VPC, while its public address provides Internet-facing IPv4 connectivity. An auto-assigned public IPv4 address can change after a stop/start; an Elastic IP remains allocated until you release it.

IPv6 uses different routing rules. AWS does not use a NAT Gateway for ordinary IPv6 Internet egress in the same way as IPv4; an egress-only Internet Gateway can support outbound-only IPv6 Internet access.

## 6. Elastic Network Interface (ENI)

An ENI is a virtual network interface attached to an instance or another supported resource. It can have:
- A MAC address and one or more private IP addresses.
- Security groups.
- A subnet and Availability Zone association.
- Optional public IPv4/EIP associations, depending on resource and configuration.

Think of an ENI as the instance's network card. The subnet and route table determine network paths; the ENI provides the interface and its network identity.

## 7. What makes a subnet public or private?

**The route table is the key distinction**, not the subnet's name.

- **Public subnet:** has a route to an Internet Gateway, commonly `0.0.0.0/0 → igw-...`.
- **Private subnet:** has no direct default route to an IGW for public Internet access. It may route IPv4 Internet-bound traffic to a NAT Gateway.
- A subnet can have local VPC routes even when it is private.
- An instance in a public subnet is not automatically reachable: it also needs a public address and suitable security rules.

## 8. Routing between subnets

Subnets in the same VPC can usually communicate through the automatically provided `local` route, provided security groups, network ACLs, and host firewalls allow traffic.

```text
VPC: 10.0.0.0/16
  ├── Subnet A: 10.0.1.0/24
  └── Subnet B: 10.0.2.0/24

Route in both subnet route tables:
  10.0.0.0/16 → local
```

An instance at `10.0.1.10` can route traffic to `10.0.2.20` through the VPC local route. No IGW or NAT Gateway is needed for this same-VPC path.

Traffic between separate VPCs needs additional connectivity, such as VPC peering or Transit Gateway, plus correct routes and security rules.

## 9. Practical AWS CLI checks

These commands require AWS CLI credentials and permission to describe EC2 networking resources.

### List VPCs

```powershell
aws ec2 describe-vpcs --query "Vpcs[*].[VpcId,CidrBlock,State]" --output table
```

Example output:

```text
-----------------------------------------------
|                 DescribeVpcs                |
+----------------------+-------------+--------+
| vpc-0123456789abcdef0| 10.0.0.0/16 | available |
+----------------------+-------------+--------+
```

Meaning: the VPC has CIDR `10.0.0.0/16` and is in the `available` state.

### List subnets and Availability Zones

```powershell
aws ec2 describe-subnets --query "Subnets[*].[SubnetId,VpcId,AvailabilityZone,CidrBlock]" --output table
```

Example output:

```text
| subnet-aaa | vpc-0123 | us-east-1a | 10.0.1.0/24 |
| subnet-bbb | vpc-0123 | us-east-1b | 10.0.2.0/24 |
```

Meaning: both subnets belong to the same VPC but are in different Availability Zones.

### Inspect route tables

```powershell
aws ec2 describe-route-tables --query "RouteTables[*].[RouteTableId,Routes[*].[DestinationCidrBlock,GatewayId,NatGatewayId]]" --output json
```

Example route entry:

```json
{
  "DestinationCidrBlock": "0.0.0.0/0",
  "GatewayId": "igw-0123456789abcdef0"
}
```

Meaning: unmatched IPv4 destinations use the Internet Gateway as the default target. If the target is a NAT Gateway instead, outbound IPv4 traffic follows that NAT path.

## Key idea

**Routes choose the destination path; gateways provide connectivity; IP addresses identify endpoints; security controls decide whether traffic is permitted.** A public subnet has an IGW route, while a private subnet commonly uses a NAT Gateway for outbound IPv4 Internet access.

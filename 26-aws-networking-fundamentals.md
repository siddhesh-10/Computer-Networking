# Networking — Phase 26: AWS Networking Fundamentals

> **Goal:** Understand AWS global infrastructure, Regions, Availability Zones, VPCs, CIDR ranges, subnets, and network interfaces.

## 1. AWS Global Infrastructure

AWS operates infrastructure in multiple geographic areas. Choose where resources run based on latency, availability, compliance, service availability, and cost.

```text
AWS Global Infrastructure
  └── Region
       ├── Availability Zone
       │    ├── Subnets
       │    └── Resources / Network Interfaces
       └── Availability Zone
```

AWS networking builds on IP addressing, subnetting, routing, and network interfaces.

## 2. Region

A **Region** is a separate geographic area where AWS provides services. Examples include `us-east-1` and `eu-west-1`.

Choose a Region based on:
- Distance to users and latency
- Required AWS services and instance types
- Data residency and compliance
- Cost and disaster-recovery needs

Resources in different Regions are separate unless connected or replicated using an appropriate design.

## 3. Availability Zone (AZ)

An **Availability Zone** is an isolated location within a Region, designed to reduce the impact of failures in another AZ. An AZ may consist of one or more data centers.

```text
AWS Region
  ├── AZ-a → subnets and resources
  ├── AZ-b → subnets and resources
  └── AZ-c → subnets and resources
```

Deploying across multiple AZs can improve resilience, but applications and dependencies must also support failover.

## 4. Region vs Availability Zone

| Concept | Region | Availability Zone |
|---|---|---|
| Scope | Geographic area | Isolated location within a Region |
| Purpose | Geographic placement and separation | Availability and fault isolation |
| Example | `us-east-1` | An AZ identifier within `us-east-1` |

AZ names such as `us-east-1a` are not guaranteed to map to the same physical AZ across AWS accounts. Use AZ IDs when consistent physical-zone identification across accounts is needed.

## 5. VPC

A **Virtual Private Cloud (VPC)** is a logically isolated virtual network in an AWS Region that you define.

You control its IP ranges, subnets, route tables, gateways, connectivity, and security controls.

```text
Region
└── VPC: 10.0.0.0/16
    ├── Subnet in AZ-a
    ├── Subnet in AZ-b
    └── Subnet in AZ-c
```

A VPC spans multiple AZs in one Region. **A subnet belongs to exactly one AZ.**

## 6. VPC CIDR

A VPC CIDR block defines the IP address range allocated to the VPC.

```text
10.0.0.0/16
         ^^
         16 network-prefix bits
```

The shorter the prefix, the larger the IPv4 range.

| CIDR | Total IPv4 addresses |
|---|---:|
| `10.0.0.0/16` | 65,536 |
| `10.0.1.0/24` | 256 |
| `10.0.1.0/28` | 16 |

AWS reserves some IPv4 addresses in each subnet, so the number available to resources is lower than the total.

Avoid overlapping CIDRs between VPCs, on-premises networks, and connected partner networks. Overlap complicates routing and can require renumbering or translation.

## 7. Subnets

A **subnet** is a smaller IP range inside a VPC. Each subnet belongs to one AZ and is associated with a route table.

```text
VPC: 10.0.0.0/16
  ├── 10.0.1.0/24  (AZ-a)
  ├── 10.0.2.0/24  (AZ-b)
  └── 10.0.3.0/24  (AZ-c)
```

Subnets organize IP allocation, routing, workloads, and security boundaries.

## 8. Public vs Private Subnets

In AWS, **public or private is determined primarily by routing**, not by the subnet's name or CIDR.

### Public subnet

A subnet is public when its route table has a route to an Internet Gateway (IGW), typically:

```text
0.0.0.0/0 → Internet Gateway
```

For an IPv4 resource to communicate directly with the public Internet, it generally also needs a public IPv4 address or Elastic IP and suitable security rules.

### Private subnet

A private subnet has no direct route to an Internet Gateway for direct public Internet access. It may still have outbound Internet access through a NAT Gateway or private connectivity through a VPN, Direct Connect, VPC peering, or Transit Gateway.

```text
Internet
   |
Internet Gateway
   |
Public subnet
   └── Public-facing resource
        |
        | VPC routing
        v
Private subnet
   └── Application / database
```

A public subnet does **not** automatically make every resource public. Addressing, route tables, security groups, network ACLs, and service configuration all matter.

## 9. Route Tables

A **route table** determines where traffic is sent based on its destination IP prefix.

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | `local` | Routing within the VPC |
| `0.0.0.0/0` | Internet Gateway | Default IPv4 route to the Internet |

A subnet uses an associated route table. The VPC's local route allows routing between VPC subnets, subject to security controls and other configuration.

```text
Resource → Subnet route table → Longest-prefix match → Route target
```

## 10. Elastic Network Interfaces (ENIs)

An **Elastic Network Interface (ENI)** is a virtual network interface attached to an EC2 instance or used by supported AWS services.

An ENI can have:
- Private IPv4 addresses and, when configured, IPv6 addresses
- A MAC address
- Security groups
- A subnet association
- A primary private IP and optional secondary private IPs

```text
EC2 instance
  ├── ENI 1
  │    ├── Private IP
  │    ├── MAC address
  │    └── Security groups
  └── ENI 2 (if supported/configured)
       └── Additional network identity
```

ENIs connect compute resources to VPC networking. Attachment and movement capabilities depend on the instance and interface type.

## 11. Private IP vs Public IP

- **Private IP:** Used within private networks and connected networks.
- **Public IPv4:** Used for Internet-facing IPv4 communication when routing and security allow it.
- **Elastic IP:** A static public IPv4 address that can be associated with supported AWS resources.

A public IP does not bypass security groups, network ACLs, or routing requirements. IPv6 has different addressing and routing behavior; IPv6 resources commonly use globally routable addresses controlled by routes and security rules.

## 12. Example VPC Layout

```text
Region
└── VPC: 10.0.0.0/16
    ├── AZ-a
    │   ├── Public subnet:  10.0.1.0/24
    │   └── Private subnet: 10.0.11.0/24
    └── AZ-b
        ├── Public subnet:  10.0.2.0/24
        └── Private subnet: 10.0.12.0/24
```

A common design places public-facing load balancers in public subnets and application/database resources in private subnets. The exact design depends on required connectivity and security.

## 13. Practical AWS CLI Commands

These commands require AWS CLI, configured credentials, and appropriate permissions. Replace example values with your own.

### Describe VPCs

```bash
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock,State]' --output table
```

Example output:

```text
---------------------------------------
|             DescribeVpcs            |
+----------------------+--------------+
| vpc-0123456789abcdef0 | 10.0.0.0/16 |
+----------------------+--------------+
```

Meaning: shows the VPC ID and its IPv4 CIDR.

### List subnets

```bash
aws ec2 describe-subnets --query 'Subnets[*].[SubnetId,VpcId,AvailabilityZone,CidrBlock]' --output table
```

Example output:

```text
| subnet-aaa | vpc-0123 | us-east-1a | 10.0.1.0/24 |
| subnet-bbb | vpc-0123 | us-east-1b | 10.0.2.0/24 |
```

Meaning: each subnet belongs to a VPC, has its own CIDR, and is placed in one AZ.

### List network interfaces

```bash
aws ec2 describe-network-interfaces --query 'NetworkInterfaces[*].[NetworkInterfaceId,SubnetId,PrivateIpAddress,Status]' --output table
```

Example output:

```text
| eni-0123 | subnet-aaa | 10.0.1.25 | in-use |
```

Meaning: the ENI is in use and has private IP `10.0.1.25` in the listed subnet.

## 14. Summary

- **Region:** Geographic AWS area.
- **Availability Zone:** Isolated location within a Region.
- **VPC:** Logically isolated virtual network in one Region.
- **VPC CIDR:** IP range allocated to the VPC.
- **Subnet:** Smaller IP range belonging to one AZ.
- **Public subnet:** Has a route to an Internet Gateway; resources still need suitable addressing and security.
- **Private subnet:** Has no direct route to an Internet Gateway for direct public Internet access; NAT or private connectivity may still exist.
- **Route table:** Determines where traffic is sent.
- **ENI:** Virtual network interface connecting a resource to VPC networking.

> **Key idea:** A VPC defines the network, CIDRs define address space, subnets divide that space per AZ, route tables determine paths, and ENIs attach workloads to the network.

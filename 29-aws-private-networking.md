# Networking — Phase 29: AWS Private Networking

> **Goal:** Understand how AWS resources communicate privately across VPCs and how services can be reached without traversing the public internet.

## 1. VPC Peering

**VPC Peering** creates a private network connection between two VPCs.

```text
VPC A                         VPC B
10.0.0.0/16                  10.1.0.0/16
   │                             │
   └────── VPC Peering ─────────┘
```

Traffic uses private IP addresses and does not require an Internet Gateway, NAT Gateway, or public IP.

Important:
- Add routes in both VPCs' route tables.
- CIDR ranges must not overlap.
- Security groups and NACLs still apply.
- Peering is **not transitive**.

Example:

```text
A ↔ B
B ↔ C

A ↛ C automatically
```

For many-to-many VPC connectivity, Transit Gateway is usually a better fit.

## 2. Transit Gateway

**AWS Transit Gateway (TGW)** acts as a regional network router connecting multiple VPCs and other networks.

```text
             ┌─────────────┐
VPC A ───────┤             │
VPC B ───────┤ Transit GW  ├──── VPN / Direct Connect
VPC C ───────┤             │
             └─────────────┘
```

Instead of creating many individual peering connections:

```text
Without TGW:
A ↔ B
A ↔ C
A ↔ D
B ↔ C
B ↔ D
C ↔ D
```

With TGW:

```text
A ─┐
B ─┤
C ─┼── Transit Gateway
D ─┘
```

TGW uses route tables to control which attachments can communicate.

## 3. VPC Endpoints

A **VPC endpoint** provides private connectivity from a VPC to supported AWS services or endpoint-enabled services.

This can avoid sending traffic through an Internet Gateway, NAT Gateway, or public IP.

```text
Private EC2
    │
    ▼
VPC Endpoint
    │
    ▼
AWS Service
```

The two main endpoint types here are **Gateway endpoints** and **Interface endpoints**.

## 4. Gateway Endpoints

Gateway endpoints provide private access to:

- **Amazon S3**
- **Amazon DynamoDB**

They work through route tables rather than ENIs.

```text
Private subnet
      │
      ▼
Route table
      │
      ▼
Gateway Endpoint
      │
      ▼
S3 / DynamoDB
```

A gateway endpoint can allow a private EC2 instance to access S3 without requiring a NAT Gateway.

## 5. Interface Endpoints

An **Interface VPC Endpoint** creates one or more private **ENIs** in selected subnets.

```text
Private EC2
    │
    ▼
Endpoint ENI
10.0.2.50
    │
    ▼
AWS service / endpoint service
```

Interface endpoints use **AWS PrivateLink** technology.

Important:
- Endpoint ENIs receive private IP addresses.
- Security groups can control traffic to the endpoint ENIs.
- Private DNS can resolve a service name to the private endpoint IPs.
- They support many AWS services and supported partner/customer services.

## 6. AWS PrivateLink

**AWS PrivateLink** provides private, service-oriented connectivity between a service provider and service consumers.

```text
Consumer VPC
    │
    ▼
Interface Endpoint
    │
    │ PrivateLink
    ▼
Provider VPC
    │
    ▼
Service
```

The consumer does not need direct network-level access to the provider VPC.

This differs from VPC Peering:

| VPC Peering | PrivateLink |
|---|---|
| Network-to-network connectivity | Service-to-consumer connectivity |
| Broad private routing | Access to a specific service |
| Routes between VPC CIDRs | Endpoint-based access |
| CIDRs must not overlap | Consumer/provider CIDRs can be independent |
| More network access | More isolated service exposure |

## 7. Cross-VPC communication

```text
                    Cross-VPC
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
 VPC Peering      Transit Gateway  PrivateLink
 network-wide      many networks    specific service
```

### VPC Peering

Use when:
- A small number of VPCs need direct connectivity.
- Private network access is required.
- CIDRs do not overlap.

### Transit Gateway

Use when:
- Many VPCs need connectivity.
- Centralized routing is useful.
- VPN or Direct Connect networks also need to connect.

### PrivateLink

Use when:
- One VPC needs a specific service from another VPC.
- The provider should not expose its whole VPC.
- Consumer and provider networks should remain loosely coupled.

## 8. Practical architecture

```text
                 ┌───────────────┐
                 │ Transit GW    │
                 └───────┬───────┘
                    ┌────┴────┐
                    ▼         ▼
                 VPC A      VPC B
                    │         │
                    │         └── Interface Endpoint
                    │                    │
                    ▼                    ▼
               App services         AWS / SaaS service

VPC A ── VPC Peering ── VPC C

Private subnet ── Gateway Endpoint ── S3
```

These mechanisms can be combined because they solve different connectivity problems.

## 9. Practical AWS CLI

### List VPC peering connections

```powershell
aws ec2 describe-vpc-peering-connections --query "VpcPeeringConnections[*].[VpcPeeringConnectionId,Status.Code,RequesterVpcInfo.VpcId,AccepterVpcInfo.VpcId]" --output table
```

Example:

```text
----------------------------------------------------------------
|              DescribeVpcPeeringConnections                   |
+------------------+-----------+-----------+-------------------+
| pcx-0123abcd     | active    | vpc-aaaa  | vpc-bbbb          |
+------------------+-----------+-----------+-------------------+
```

**Interpretation:** `active` means the peering connection is established. Routes are still required for traffic to flow.

### List Transit Gateways

```powershell
aws ec2 describe-transit-gateways --query "TransitGateways[*].[TransitGatewayId,State,Description]" --output table
```

Example:

```text
------------------------------------------------
|              DescribeTransitGateways          |
+------------------+-----------+-----------------+
| tgw-0123abcd     | available | prod-network    |
+------------------+-----------+-----------------+
```

**Interpretation:** The TGW is available. VPC attachments and appropriate TGW/VPC routes are still required.

### List VPC endpoints

```powershell
aws ec2 describe-vpc-endpoints --query "VpcEndpoints[*].[VpcEndpointId,VpcId,VpcEndpointType,ServiceName,State]" --output table
```

Example:

```text
----------------------------------------------------------------
|                    DescribeVpcEndpoints                       |
+------------+------------+---------+------------------+--------+
| vpce-abc123| vpc-aaaa   | Gateway | com.amazonaws...s3| Available |
+------------+------------+---------+------------------+--------+
```

**Interpretation:** `Gateway` indicates a gateway endpoint, commonly used for S3 or DynamoDB. `Available` means the endpoint is ready.

## Key idea

Choose private connectivity based on **what needs to be connected**:

```text
VPC ↔ VPC
   → VPC Peering

Many VPCs / hybrid networks
   → Transit Gateway

VPC → S3 / DynamoDB
   → Gateway Endpoint

VPC → AWS / partner service
   → Interface Endpoint

Consumer → specific service
   → AWS PrivateLink
```

Private networking reduces public exposure, but routing, DNS, Security Groups, NACLs, and endpoint policies still need to be configured correctly.

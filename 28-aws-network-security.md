# Networking — Phase 28: AWS Network Security

> **Goal:** Understand how AWS controls network traffic using Security Groups, Network ACLs, Flow Logs, and AWS Network Firewall.

## 1. Security Groups

A **Security Group (SG)** is a virtual firewall attached to resources such as EC2 network interfaces.

**Layer:** L3/L4 filtering using IPs, protocols, and ports.

Key properties:
- **Stateful:** Return traffic is automatically allowed for an allowed connection.
- **Allow rules only:** No explicit deny rules.
- Rules can reference another Security Group.
- Rules apply to traffic reaching the associated network interface.

Example:

```text
Internet
   │ TCP 443
   ▼
[Load Balancer SG]
   │ TCP 8080
   ▼
[App SG]
   │ TCP 5432
   ▼
[DB SG]
```

A strong design is to allow each tier to talk only to the required next tier.

```text
App SG:
  Inbound TCP 8080 ← LoadBalancer-SG

DB SG:
  Inbound TCP 5432 ← App-SG
```

## 2. Network ACLs

A **Network Access Control List (NACL)** is a firewall applied at the **subnet** level.

**Layer:** L3/L4 filtering.

Key properties:
- **Stateless:** Return traffic must be explicitly allowed.
- Supports **allow and deny** rules.
- Rules are evaluated in **rule-number order**; the first matching rule wins.
- Applies to traffic entering or leaving the subnet.

```text
                 Subnet
        ┌─────────────────────┐
Internet → NACL → EC2 / ENI   │
Internet ← NACL ← EC2 / ENI   │
        └─────────────────────┘
```

NACLs are useful as an additional subnet-level security boundary, especially for broad deny rules.

## 3. Stateful vs. stateless filtering

### Stateful — Security Group

If an instance receives an allowed connection:

```text
Client ── TCP 443 ──> Server
Client <─ response ── Server
```

The return traffic is automatically permitted by the SG's state tracking.

### Stateless — Network ACL

Both directions need matching rules:

```text
Client ──> Server
        NACL inbound: allow TCP 443

Server ──> Client
        NACL outbound: allow ephemeral return port
```

This is why NACL rules often need to account for **ephemeral ports** used by client-side TCP connections.

## 4. Security Group vs NACL

| Feature | Security Group | Network ACL |
|---|---|---|
| Scope | ENI/resource | Subnet |
| Stateful | Yes | No |
| Rules | Allow only | Allow + deny |
| Rule evaluation | Matching rules collectively apply | Lowest matching rule number wins |
| Typical use | Workload-level access control | Subnet boundary / broad filtering |

A common architecture uses **Security Groups as the primary workload firewall** and NACLs as an additional coarse-grained subnet control.

## 5. VPC Flow Logs

**VPC Flow Logs** capture metadata about network traffic to and from network interfaces, subnets, or VPCs.

They help answer:
- Was traffic accepted or rejected?
- Which source/destination IPs were involved?
- Which port and protocol were used?
- How many bytes/packets were transferred?

Example record:

```text
2 123456789012 eni-abc123 10.0.1.10 10.0.2.20 49152 5432 6 10 840 ACCEPT OK
```

Important fields:

```text
srcaddr = 10.0.1.10
dstaddr = 10.0.2.20
srcport = 49152
dstport = 5432
protocol = 6       (TCP)
action = ACCEPT
```

Flow Logs are **observability data**, not a firewall. They record traffic metadata; they do not themselves block traffic.

Typical troubleshooting:

```text
Application timeout
       │
       ▼
Check route
       │
       ▼
Check Security Group / NACL
       │
       ▼
Check VPC Flow Logs
       │
       ▼
Identify ACCEPT / REJECT traffic
```

## 6. AWS Network Firewall

**AWS Network Firewall** is a managed network firewall service for inspecting and filtering VPC traffic.

It can provide:
- Stateful traffic inspection
- Stateless rule groups
- Domain-based filtering
- Intrusion-prevention capabilities
- Centralized inspection architectures

Typical placement:

```text
Internet
   │
   ▼
IGW
   │
   ▼
Network Firewall
   │
   ▼
Application VPC
```

Traffic must be deliberately routed through the firewall. Deploying the firewall alone does not automatically inspect every packet in the VPC.

## 7. Network security architecture

A common layered architecture:

```text
                    Internet
                       │
                       ▼
              ┌─────────────────┐
              │ AWS WAF / ALB   │
              └────────┬────────┘
                       │
                Security Group
                       │
                       ▼
              ┌─────────────────┐
              │   Web Subnet    │
              └────────┬────────┘
                       │
                 Security Group
                       │
                       ▼
              ┌─────────────────┐
              │   App Subnet    │
              └────────┬────────┘
                       │
                 Security Group
                       │
                       ▼
              ┌─────────────────┐
              │   DB Subnet     │
              └─────────────────┘

        NACLs → subnet-level boundary
        Flow Logs → traffic visibility
        Firewall → centralized inspection
```

The principle is **defense in depth**:

```text
Routing
   ↓
NACL
   ↓
Security Group
   ↓
Host/application controls
   ↓
Logging & monitoring
```

## 8. Practical AWS CLI

These commands require AWS CLI credentials and appropriate EC2 permissions.

### Inspect Security Groups

```powershell
aws ec2 describe-security-groups --query "SecurityGroups[*].[GroupId,GroupName,VpcId]" --output table
```

Example output:

```text
---------------------------------------------------
|              DescribeSecurityGroups             |
+----------------+-----------+--------------------+
| sg-0123abcd    | app-sg    | vpc-0123abcd       |
| sg-0456efgh    | db-sg     | vpc-0123abcd       |
+----------------+-----------+--------------------+
```

**Interpretation:** Shows Security Groups and the VPC they belong to.

### Inspect a specific Security Group

```powershell
aws ec2 describe-security-groups --group-ids sg-0123abcd --output json
```

Look at `IpPermissions` for inbound rules and `IpPermissionsEgress` for outbound rules.

### Inspect NACLs

```powershell
aws ec2 describe-network-acls --query "NetworkAcls[*].[NetworkAclId,VpcId,IsDefault]" --output table
```

Example:

```text
-----------------------------------------------
|              DescribeNetworkAcls            |
+----------------+----------------+------------+
| acl-0123abcd   | vpc-0123abcd   | False      |
+----------------+----------------+------------+
```

**Interpretation:** Identifies the NACL and whether it is the VPC's default NACL.

### Inspect Flow Logs

```powershell
aws ec2 describe-flow-logs --query "FlowLogs[*].[FlowLogId,ResourceType,TrafficType,LogDestinationType,LogDestination]" --output table
```

Example:

```text
----------------------------------------------------------------
|                       DescribeFlowLogs                        |
+------------+------+---------+-------------------+-------------+
| fl-0123abc | VPC  | ALL     | cloud-watch-logs  | /aws/vpc/...|
+------------+------+---------+-------------------+-------------+
```

**Interpretation:** `ALL` means both accepted and rejected traffic is captured.

## Key idea

Think of AWS network security as layers:

```text
NACL             → subnet-level, stateless
Security Group  → resource-level, stateful
Flow Logs        → visibility
Network Firewall → advanced traffic inspection
```

For troubleshooting, separate **routing problems** from **security problems**:

```text
Can packet reach the destination?
        ↓
      Routing
        ↓
Is traffic permitted?
        ↓
 SG / NACL / Firewall
        ↓
Can the application respond?
        ↓
 Application / host firewall
```

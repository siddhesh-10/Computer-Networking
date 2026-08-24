# Networking --- Phase 16: NAT, Firewalls & Security

> **Goal:** Understand how NAT translates addresses/ports and how
> firewalls control which network traffic is allowed.

## 1. Where This Fits

These concepts operate mainly around the **network and transport
layers**.

``` text
Application
    ↓
TCP / UDP
    ↓
IP
    ↓
NAT / Firewall
    ↓
Network
```

They are commonly implemented by:

``` text
Home routers
Cloud gateways
Enterprise firewalls
Network appliances
```

------------------------------------------------------------------------

## 2. NAT

**NAT (Network Address Translation)** changes IP addressing as packets
pass through a device.

A common use is allowing private hosts to access the Internet through a
public IP.

``` text
Private network                 Internet

192.168.1.10
      ↓
192.168.1.1  NAT
      ↓
Public IP
      ↓
Internet
```

Example:

``` text
Client:
192.168.1.10:50000

NAT:
203.0.113.10:40001

Server:
93.184.216.34:443
```

The NAT device maintains a translation so return traffic can be mapped
back to the correct private host.

------------------------------------------------------------------------

## 3. Why NAT Is Used

Private IPv4 addresses are not globally routable.

Common private ranges:

``` text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

NAT allows many private hosts to share public IPv4 addresses.

``` text
192.168.1.10 ─┐
192.168.1.11 ─┼── NAT ──→ Public IP
192.168.1.12 ─┘
```

NAT helps conserve IPv4 addresses, but **NAT itself is not a
firewall/security mechanism**.

------------------------------------------------------------------------

## 4. PAT

**PAT (Port Address Translation)** translates IP addresses and
transport-layer ports.

It is commonly used for many-to-one IPv4 Internet access.

``` text
Client 1:
192.168.1.10:50000
        ↓
203.0.113.10:40001

Client 2:
192.168.1.11:50000
        ↓
203.0.113.10:40002
```

Both clients can use the same public IP because different translated
ports distinguish the connections.

``` text
Private IP:Port
      ↓
Public IP:Translated Port
```

Many home NAT configurations are effectively PAT.

------------------------------------------------------------------------

## 5. NAT vs PAT

``` text
NAT
 ↓
Translates IP addresses

PAT
 ↓
Translates IP addresses + ports
```

Example:

``` text
NAT:

192.168.1.10
      ↓
203.0.113.10


PAT:

192.168.1.10:50000
      ↓
203.0.113.10:40001
```

In practice, "NAT" is often used as a general term for both.

------------------------------------------------------------------------

## 6. SNAT

**SNAT (Source NAT)** changes the **source** address of a packet.

Before:

``` text
Source      = 192.168.1.10:50000
Destination = 93.184.216.34:443
```

After SNAT:

``` text
Source      = 203.0.113.10:40001
Destination = 93.184.216.34:443
```

Typical use:

``` text
Private client
     ↓
SNAT / PAT
     ↓
Internet
```

------------------------------------------------------------------------

## 7. DNAT

**DNAT (Destination NAT)** changes the **destination** address, commonly
for inbound traffic.

``` text
Internet client
     ↓
203.0.113.10:443
     ↓
DNAT
     ↓
192.168.1.10:443
```

Typical use:

``` text
Public IP:443
     ↓
Private server:443
```

------------------------------------------------------------------------

## 8. SNAT vs DNAT

``` text
                     SNAT               DNAT
  ------------------ ------------------ ------------------
  Changes            Source             Destination
  Common direction   Outbound           Inbound
  Example            Private → Public   Public → Private
  Common use         Internet access    Port forwarding
```

``` text
Outbound:

Private client
     ↓
SNAT
     ↓
Internet


Inbound:

Internet
     ↓
DNAT
     ↓
Private server
```

------------------------------------------------------------------------

## 9. Port Forwarding

**Port forwarding** maps an external IP/port to an internal server.

``` text
Internet
   |
   | 203.0.113.10:443
   ↓
Router / Firewall
   |
   | DNAT
   ↓
192.168.1.10:443
```

Example rule:

``` text
Public TCP 443
       ↓
192.168.1.10 TCP 443
```

The firewall normally must also allow the traffic.

> NAT and firewalling are separate functions.

------------------------------------------------------------------------

## 10. Stateful Firewall

A **stateful firewall** tracks connection/flow state.

For TCP:

``` text
Client
  ↓
SYN
  ↓
Firewall
  ↓
Track connection
  ↓
Server
```

Conceptually:

``` text
NEW
 ↓
ESTABLISHED
 ↓
FIN / close
 ↓
CLOSED
```

This allows rules such as:

``` text
Allow outbound connections
Allow their return traffic
Block unrelated inbound traffic
```

A stateful device commonly maintains a connection/flow table.

------------------------------------------------------------------------

## 11. Stateless Firewall

A **stateless firewall** evaluates packets independently.

It can inspect:

``` text
Source IP
Destination IP
Protocol
Source port
Destination port
```

Example:

``` text
ALLOW TCP destination port 443
DENY  TCP destination port 23
```

Each packet is evaluated against the configured rules without relying on
connection state.

------------------------------------------------------------------------

## 12. Stateful vs Stateless
``` text

                                   Stateful                        Stateless
  -------------------------------- ------------------------------- -------------------------
  Tracks connections               Yes                             No
  Packet evaluated independently   No, state can affect decision   Yes
  Recognizes established flows     Yes                             No
  Return traffic rules             Often simpler                   May need explicit rules
  Typical use                      Firewalls                       ACLs / packet filters
```

Mental model:

``` text
Stateful:

Packet
 ↓
Connection state?
 ↓
Allow / deny


Stateless:

Packet
 ↓
Rule match?
 ↓
Allow / deny
```

------------------------------------------------------------------------

## 13. Network Filtering

A firewall can filter using:

``` text
Source IP
Destination IP
Source port
Destination port
Protocol
Connection state
Interface
Direction
```

Example:

``` text
ALLOW TCP 443
DENY  TCP 23
DENY  other inbound
```

Meaning:

``` text
HTTPS → allowed
Telnet → blocked
Other inbound traffic → blocked
```

Filtering answers:

``` text
Should this traffic be allowed?
```

------------------------------------------------------------------------

## 14. NAT Is Not a Firewall

A common misconception is:

> "NAT provides security."

NAT can make unsolicited inbound connections harder in typical
configurations, but NAT and firewalling solve different problems.

``` text
NAT
 ↓
Translates addresses/ports

Firewall
 ↓
Allows or blocks traffic
```

A device can perform both:

``` text
Router / Gateway
 ├── NAT
 └── Firewall
```

------------------------------------------------------------------------

## 15. Example: Outbound HTTPS

Suppose:

``` text
Client:
192.168.1.10:50000

Server:
93.184.216.34:443
```

Outgoing packet:

``` text
192.168.1.10:50000
        ↓
      PAT
        ↓
203.0.113.10:40001
        ↓
93.184.216.34:443
```

The firewall/NAT device can track:

``` text
203.0.113.10:40001
        ↕
93.184.216.34:443
```

Return traffic:

``` text
93.184.216.34:443
        ↓
203.0.113.10:40001
        ↓
NAT translation
        ↓
192.168.1.10:50000
```

------------------------------------------------------------------------

## 16. Example: Inbound Port Forwarding

Suppose:

``` text
Public IP:
203.0.113.10

Internal server:
192.168.1.20
```

Rule:

``` text
203.0.113.10:443
        ↓
192.168.1.20:443
```

Flow:

``` text
Internet client
      ↓
203.0.113.10:443
      ↓
Firewall rule
      ↓
DNAT
      ↓
192.168.1.20:443
      ↓
Internal server
```

The firewall must permit the connection.

------------------------------------------------------------------------

## 17. Connection Tracking

A stateful firewall/NAT device may maintain a table like:

``` text
Protocol  Internal              Public              Remote
TCP       192.168.1.10:50000    203.0.113.10:40001   93.184.216.34:443
```

This allows return packets to be associated with the correct internal
connection.

The exact fields depend on the implementation.

------------------------------------------------------------------------

## 18. Practical: Windows Firewall Rules

Run:

``` powershell
Get-NetFirewallRule | Select-Object -First 5
```

Example:

``` text
Name                         Enabled Direction Action
----                         ------- --------- ------
CoreNetworking-AllowICMP     True    Inbound   Allow
...
```

Meaning:

``` text
Enabled
 ↓
Rule is active

Direction
 ↓
Inbound or outbound

Action
 ↓
Allow or block
```

Exact rules vary by Windows configuration.

------------------------------------------------------------------------

## 19. Practical: Test a Port

Run:

``` powershell
Test-NetConnection example.com -Port 443
```

Example:

``` text
ComputerName     : example.com
RemotePort       : 443
TcpTestSucceeded : True
```

Meaning:

``` text
TCP connection to
example.com:443
was successful.
```

If:

``` text
TcpTestSucceeded : False
```

the connection could not be established from that host.

Possible causes include:

``` text
Firewall
Routing
Server not listening
NAT
Network failure
```

------------------------------------------------------------------------

## 20. NAT + Firewall Mental Model

``` text
                 Internet
                    |
                    ↓
            Public IP:443
                    |
             ┌──────┴──────┐
             │ Firewall    │
             │ + NAT       │
             └──────┬──────┘
                    |
              Private network
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Host A     Host B    Host C
```

Outbound:

``` text
Private host
    ↓
SNAT / PAT
    ↓
Firewall
    ↓
Internet
```

Inbound:

``` text
Internet
    ↓
Firewall rule
    ↓
DNAT / port forwarding
    ↓
Private server
```

------------------------------------------------------------------------

## 21. Summary

``` text
NAT
 ↓
Translates IP addresses

PAT
 ↓
Translates IP addresses + ports

SNAT
 ↓
Changes source address

DNAT
 ↓
Changes destination address

Port forwarding
 ↓
Maps public address/port to internal server

Stateful firewall
 ↓
Tracks connection state

Stateless firewall
 ↓
Evaluates packets independently

Network filtering
 ↓
Allows or blocks traffic using rules
```

Core distinction:

``` text
NAT
 ↓
"Where should this address/port be translated?"

Firewall
 ↓
"Should this traffic be allowed?"
```

Typical Internet access:

``` text
192.168.1.10:50000
        ↓
      PAT
        ↓
203.0.113.10:40001
        ↓
      Internet
```

Typical inbound service:

``` text
203.0.113.10:443
        ↓
Firewall allows
        ↓
DNAT / Port Forward
        ↓
192.168.1.20:443
```

# Networking — Phase 25: Network Troubleshooting

> **Goal:** Troubleshoot real production network failures systematically using common network, DNS, TCP, TLS, and HTTP tools.

## 1. Production Troubleshooting Mindset

Do not start by changing configuration.

```text
Symptom → Scope → Layer → Evidence → Root cause → Fix → Verify
```

First determine:

- Is the problem DNS, network, TCP, TLS, HTTP, or the application?
- Does it affect one host, one service, one subnet, or everyone?
- Is it intermittent or constant?
- Did anything change recently?

Useful stack:

```text
HTTP / API
    ↓
TLS
    ↓
TCP
    ↓
IP
    ↓
Ethernet
    ↓
Physical / Cloud network
```

## 2. `ping`

`ping` tests IP reachability using **ICMP**.

- **Layer:** Network layer
- **Protocol:** ICMP / ICMPv6
- Does **not** test TCP, TLS, or HTTP

```powershell
ping example.com
```

Example:

```text
Reply from 93.184.216.34: bytes=32 time=21ms TTL=56
Reply from 93.184.216.34: bytes=32 time=20ms TTL=56

Packets: Sent = 2, Received = 2, Lost = 0 (0% loss)
```

Meaning:
- Reply received → ICMP reached the destination and returned
- `time` → approximate RTT
- `TTL` → remaining IP hop limit
- `0% loss` → no loss in this test

```text
ping succeeds ≠ TCP/443 succeeds
ping fails  ≠ application is definitely down
```

Firewalls commonly block ICMP.

## 3. `traceroute` / `tracert`

Traceroute shows the path toward a destination.

- **Layer:** Network layer
- Uses TTL/hop-limit behavior and probe packets
- Windows: `tracert`
- Linux/macOS: `traceroute`

```powershell
tracert example.com
```

Example:

```text
  1     1 ms     1 ms     2 ms  192.168.1.1
  2    10 ms     9 ms    11 ms  10.0.0.1
  3    21 ms    20 ms    22 ms  203.0.113.1
```

Meaning:
- Each row represents a routing hop
- The times are probe RTTs
- `*` can mean a router did not return a response
- A missing response does not automatically mean traffic is broken

## 4. `dig`

`dig` is a DNS diagnostic tool.

- **Layer:** Application layer
- **Protocol:** DNS, usually UDP/TCP port 53

```bash
dig example.com
```

Example:

```text
;; ANSWER SECTION:
example.com.    300    IN    A    93.184.216.34
```

Meaning:
- `300` → TTL in seconds
- `A` → IPv4 address record
- `93.184.216.34` → returned address

Useful queries:

```bash
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig @8.8.8.8 example.com
```

## 5. `nslookup`

`nslookup` is another DNS troubleshooting utility.

```powershell
nslookup example.com
```

Example:

```text
Server:  dns.example.local
Address: 192.168.1.1

Non-authoritative answer:
Name:    example.com
Address: 93.184.216.34
```

Meaning:
- `Server` → DNS resolver queried
- `Non-authoritative` → answer came from a recursive resolver/cache rather than the authoritative server
- `Address` → resolved IP address

Use `dig` when you need more detailed DNS response information; `nslookup` is commonly available on Windows.

## 6. DNS Debugging

```text
Application
    ↓
DNS resolution?
    ↓
   Yes → IP connectivity
    No
    ↓
DNS troubleshooting
```

Check:

```powershell
nslookup api.example.com
```

Then query a specific resolver:

```bash
dig @8.8.8.8 api.example.com
```

Check different record types:

```bash
dig api.example.com A
dig api.example.com AAAA
```

Common failures:

- NXDOMAIN
- SERVFAIL
- Timeout
- Wrong record
- Stale cache
- Broken delegation
- Incorrect DNS server
- IPv6 record pointing to an unreachable address

## 7. `curl`

`curl` is extremely useful for HTTP and TLS troubleshooting.

- **Layer:** Application layer
- Tests HTTP/HTTPS
- HTTPS additionally exercises TLS and TCP

Windows:

```powershell
curl.exe -I https://example.com
```

Example:

```text
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
```

Meaning:
- `200` → request succeeded
- `Content-Type` → response media type
- `Content-Length` → response body size

### Verbose mode

```powershell
curl.exe -v https://example.com
```

It can show:

```text
* Connected to example.com
* TLS handshake
> GET /
< HTTP/1.1 200 OK
```

This helps separate:

```text
DNS → TCP → TLS → HTTP
```

## 8. `ss`

`ss` displays socket and connection information on Linux.

- **Layer:** Transport/socket layer
- Useful for TCP/UDP connection state

```bash
ss -tulnp
```

Example:

```text
LISTEN 0 128 0.0.0.0:443 0.0.0.0:* users:(('nginx',pid=1200))
ESTAB  0 0   10.0.0.10:443 10.0.0.20:51522
```

Meaning:
- `LISTEN` → service is waiting for connections
- `ESTAB` → TCP connection is established
- `:443` → HTTPS service
- `nginx` → process owning the listening socket

## 9. `netstat`

`netstat` shows network connections, listening ports, and statistics.

```powershell
netstat -ano
```

Example:

```text
TCP    0.0.0.0:443    0.0.0.0:0    LISTENING    1200
TCP    10.0.0.10:443  10.0.0.20:51522 ESTABLISHED 1200
```

Meaning:
- `LISTENING` → service accepts connections
- `ESTABLISHED` → TCP connection exists
- `1200` → process ID (PID)

`ss` is generally preferred on modern Linux systems, but `netstat` remains useful on Windows and older systems.

## 10. `tcpdump`

`tcpdump` captures and displays packets.

- **Layer:** Packet-level observation across multiple layers
- Useful for DNS, TCP, ICMP, HTTP, and other traffic

```bash
sudo tcpdump -i eth0 port 443
```

Example:

```text
10.0.0.10.51522 > 203.0.113.10.443: Flags [S], seq 1000
203.0.113.10.443 > 10.0.0.10.51522: Flags [S.], seq 5000, ack 1001
10.0.0.10.51522 > 203.0.113.10.443: Flags [.], ack 5001
```

This represents:

```text
Client → SYN
Server → SYN-ACK
Client → ACK
```

That is the TCP three-way handshake.

Useful filters:

```bash
tcpdump -i eth0 host 10.0.0.20
tcpdump -i eth0 port 53
tcpdump -i eth0 port 443
tcpdump -i eth0 tcp
tcpdump -i eth0 icmp
```

## 11. Wireshark

**Wireshark** is a graphical packet analyzer.

It can decode:

```text
Ethernet → IP → TCP → TLS → HTTP
```

Useful display filters:

```text
dns
tcp
tls
http
icmp
tcp.port == 443
ip.addr == 10.0.0.20
tcp.flags.syn == 1
```

Wireshark is useful when packet-by-packet behavior needs inspection.

## 12. TCP Debugging

For a TCP connection, check the handshake first:

```text
Client                    Server
  | ---- SYN ------------> |
  | <--- SYN-ACK ----------|
  | ---- ACK ------------->|
```

### No SYN-ACK

```text
Client → SYN → Server
Client ←     no response
```

Possible causes:
- Firewall
- Routing problem
- Security group/network ACL
- Server not reachable
- Server silently dropping packets

### RST

```text
Client → SYN
Server → RST
```

Possible causes include a closed port or a service/device actively rejecting the connection.

### Handshake succeeds but application hangs

Then investigate:
- TLS
- HTTP
- Server processing
- Application dependencies
- Packet loss/retransmissions

## 13. TCP Connection States

| State | Meaning |
|---|---|
| LISTEN | Waiting for connections |
| SYN-SENT | Client sent SYN |
| SYN-RECEIVED | Server received SYN and replied |
| ESTABLISHED | Connection established |
| FIN-WAIT | Connection is being closed |
| TIME-WAIT | Waiting after connection close |

A large number of connections in one state can provide clues, but the state alone does not prove the root cause.

## 14. TLS Debugging

HTTPS normally follows:

```text
DNS → TCP → TLS → HTTP
```

Useful command:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Example:

```text
CONNECTED(00000003)
Protocol  : TLSv1.3
Cipher    : TLS_AES_256_GCM_SHA384
Verify return code: 0 (ok)
```

Meaning:
- `CONNECTED` → TCP connection succeeded
- `TLSv1.3` → negotiated TLS version
- `Cipher` → negotiated cryptographic suite
- `Verify return code: 0` → certificate verification succeeded in this test

Common TLS problems:
- Expired certificate
- Wrong hostname/SNI
- Missing intermediate certificate
- Unsupported TLS version
- Unsupported cipher
- Client/server trust mismatch

## 15. HTTP Debugging

Once TCP and TLS work, inspect HTTP:

```powershell
curl.exe -v https://example.com/api
```

Look for:

```text
> GET /api HTTP/1.1
> Host: example.com

< HTTP/1.1 200 OK
< Content-Type: application/json
```

If the response is slow, separate:

```text
DNS time
+ TCP connection time
+ TLS handshake time
+ Server processing time
+ Response transfer time
```

This prevents treating every HTTP delay as a network problem.

## 16. HTTP Status Code Investigation

### 4xx — Client/request-side errors

| Code | Meaning |
|---|---|
| 400 | Bad Request |
| 401 | Authentication required/failed |
| 403 | Forbidden |
| 404 | Resource not found |
| 408 | Request Timeout |
| 429 | Too Many Requests |

Typical investigation:

```text
Request
 ↓
Authentication?
 ↓
Authorization?
 ↓
Correct URL?
 ↓
Required headers/body?
 ↓
Rate limit?
```

### 5xx — Server/proxy-side errors

| Code | Meaning |
|---|---|
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

For a reverse-proxy architecture:

```text
Client → Load Balancer / Proxy → Application → Database / Dependency
```

A `502` or `504` may indicate a problem between the proxy and its upstream, not necessarily between the client and proxy.

## 17. Timeouts

A timeout means an expected operation did not complete within its configured time.

```text
DNS timeout
   ↓
TCP connect timeout
   ↓
TLS handshake timeout
   ↓
HTTP response timeout
   ↓
Application dependency timeout
```

Always identify **which operation timed out**.

## 18. Connection Failures

A connection failure can occur before HTTP starts.

```text
DNS → IP → TCP connection → TLS → HTTP
```

Troubleshoot in order:

```text
1. Does DNS resolve?
2. Is the destination IP correct?
3. Is the route available?
4. Is TCP/port reachable?
5. Does TLS succeed?
6. Does HTTP return the expected response?
```

Useful tools:

```text
DNS  → dig / nslookup
IP   → ping / traceroute
TCP  → ss / netstat / tcpdump / Test-NetConnection
TLS  → openssl / curl -v
HTTP → curl -v
```

## 19. Packet Loss Investigation

Start with:

```powershell
ping example.com -n 20
```

Example:

```text
Packets: Sent = 20, Received = 18, Lost = 2 (10% loss)
```

Then trace the path:

```powershell
tracert example.com
```

For deeper analysis:

```bash
sudo tcpdump -i eth0 host 203.0.113.10
```

Look for:
- Retransmissions
- Missing acknowledgements
- Duplicate ACKs
- TCP resets
- ICMP errors

Important: packet loss reported at one intermediate router does not necessarily mean end-to-end packet loss. Some routers deprioritize or filter diagnostic probes.

## 20. Production Troubleshooting Flow

```text
User reports "API is down"
            ↓
      DNS resolution?
       /          \\
     No            Yes
     ↓              ↓
  Fix DNS       TCP/443 works?
                  /       \\
                No         Yes
                ↓           ↓
          Routing, ACL,   TLS works?
          firewall, etc.    /    \\
                            No     Yes
                            ↓       ↓
                         TLS      HTTP status?
                       debugging   /       \\
                                  4xx       5xx
                                   ↓         ↓
                              Request    Server/proxy/
                              problem    dependency
```

## 21. Practical Tool Selection

| Problem | First tools |
|---|---|
| DNS resolution | `dig`, `nslookup` |
| Basic reachability | `ping` |
| Routing path | `tracert`, `traceroute` |
| TCP port | `Test-NetConnection`, `ss`, `netstat` |
| TCP packets | `tcpdump`, Wireshark |
| TLS | `openssl`, `curl -v` |
| HTTP | `curl -v` |
| Listening service | `ss`, `netstat` |
| Packet loss | `ping`, `tcpdump`, Wireshark` |
| 4xx/5xx | `curl`, application/proxy logs |

## 22. Mental Model

```text
                    Production failure
                           │
                           ▼
                         DNS?
                           │
                           ▼
                       Routing?
                           │
                           ▼
                      TCP / Port?
                           │
                           ▼
                         TLS?
                           │
                           ▼
                         HTTP?
                           │
                           ▼
                    Application /
                    dependency?
```

The key is to identify the **first layer where expected behavior stops**.

## 23. Summary

- **ping** checks ICMP reachability, not application health.
- **traceroute/tracert** shows the network path using hop-by-hop probes.
- **dig/nslookup** diagnose DNS.
- **curl** is useful for HTTP and HTTPS debugging.
- **ss/netstat** show sockets, listening ports, and connection states.
- **tcpdump/Wireshark** provide packet-level evidence.
- **TCP debugging** starts with the SYN/SYN-ACK/ACK handshake.
- **TLS debugging** starts after TCP succeeds.
- **HTTP debugging** starts after TLS succeeds for HTTPS.
- **4xx** generally indicates a client/request-side problem; **5xx** generally indicates a server/proxy-side problem.
- **Timeouts** must be tied to the specific operation that timed out.
- **Packet loss** should be investigated end-to-end, not inferred from one intermediate hop.

> **Key idea:** Troubleshoot from the bottom of the connection stack upward: **DNS → IP/routing → TCP → TLS → HTTP → application**, and use packet captures when higher-level evidence is insufficient.

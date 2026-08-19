# Networking --- Phase 10: TCP Lifecycle & Performance

> **Goal:** Understand how TCP connections close, why `TIME_WAIT`
> exists, and how buffers, keep-alive, pooling, timeouts, and connection
> limits affect application performance.

## 1. TCP Connection Lifecycle

A simplified TCP lifecycle:

``` text
          Connection setup
        SYN → SYN-ACK → ACK
                 ↓
             ESTABLISHED
                 ↓
              Data
                 ↓
          Connection close
```

A TCP connection can end normally with `FIN` or be terminated
immediately with `RST`.

------------------------------------------------------------------------

## 2. FIN

**FIN (Finish)** means:

> "I have no more data to send."

A normal TCP close is usually a **four-step exchange** because each
direction closes independently.

``` text
Client                         Server

FIN ------------------------->

    <---------------------- ACK

    <---------------------- FIN

ACK ------------------------->
```

This is called a **graceful close**.

Important:

``` text
FIN = close my sending direction
```

It does not mean the entire connection disappears instantly.

------------------------------------------------------------------------

## 3. Half-Close

TCP is full-duplex, so each direction can close independently.

After the client sends:

``` text
FIN
```

the client cannot send more application data, but it can still receive
data from the server.

``` text
Client  ←────── Server
  X
cannot send

can still receive
```

The server can later send its own:

``` text
FIN
```

------------------------------------------------------------------------

## 4. RST

**RST (Reset)** immediately terminates or rejects a TCP connection.

Common situations include:

``` text
Connection to a closed port
Unexpected TCP segment
Application forcibly aborting a connection
```

Conceptually:

``` text
Client ─────────────→ Server
       <──── RST
```

Difference:

``` text
FIN
 ↓
Graceful shutdown

RST
 ↓
Immediate/reset shutdown
```

An RST can cause data that has not been successfully delivered to be
discarded.

------------------------------------------------------------------------

## 5. TIME_WAIT

After a TCP connection is closed, one endpoint can enter:

``` text
TIME_WAIT
```

Its purpose includes allowing delayed packets from the old connection to
expire and ensuring the final TCP termination exchange can be completed
if needed.

A simplified close:

``` text
Client
  |
  | FIN
  ↓
Server
  |
  | FIN
  ↓
Client
  |
  | ACK
  ↓
TIME_WAIT
```

`TIME_WAIT` is therefore intentional TCP behavior, not automatically a
problem.

The exact duration depends on the TCP implementation and operating
system.

------------------------------------------------------------------------

## 6. Why TIME_WAIT Can Matter

A busy server can create many short-lived TCP connections:

``` text
Request 1 → connection → close
Request 2 → connection → close
Request 3 → connection → close
...
```

This can produce many sockets in:

``` text
TIME_WAIT
```

Large numbers can consume local networking resources and ephemeral
ports.

This is one reason long-lived connections and connection pooling can
improve performance.

------------------------------------------------------------------------

## 7. Keep-Alive

There are two related concepts that should not be confused.

### TCP Keepalive

TCP can send periodic probes on an otherwise idle connection to detect
whether the peer is still reachable.

``` text
Idle TCP connection
        ↓
Keepalive probe
        ↓
Peer responds?
```

The exact intervals and behavior depend on the OS/application
configuration.

### Application-Level Keep-Alive

Protocols such as HTTP can reuse a TCP connection for multiple requests
instead of creating a new connection for every request.

``` text
TCP connection
 ├── HTTP request 1
 ├── HTTP response 1
 ├── HTTP request 2
 └── HTTP response 2
```

The main benefit is avoiding repeated TCP connection setup.

------------------------------------------------------------------------

## 8. Connection Pooling

A **connection pool** keeps a set of reusable connections.

Without pooling:

``` text
Request
  ↓
Open TCP connection
  ↓
Use it
  ↓
Close
```

With pooling:

``` text
        Connection Pool
       ┌─────┬─────┬─────┐
Request → C1  │ C2  │ C3  │
       └─────┴─────┴─────┘
             ↓
         Reuse connection
```

Benefits:

-   Avoids repeated handshakes
-   Reduces connection setup latency
-   Reduces connection churn
-   Can reduce `TIME_WAIT` creation
-   Improves throughput for many requests

A pool normally has limits such as:

``` text
Maximum connections
Minimum idle connections
Idle timeout
Connection lifetime
```

------------------------------------------------------------------------

## 9. TCP Buffers

TCP uses buffers on both sides.

``` text
Application
    ↓
Send buffer
    ↓
TCP
    ↓
Network
    ↓
TCP
    ↓
Receive buffer
    ↓
Application
```

### Send buffer

Stores application data waiting to be transmitted or acknowledged.

### Receive buffer

Stores received data until the application reads it.

If the receiving application reads slowly:

``` text
Receive buffer fills
      ↓
Advertised receive window decreases
      ↓
Sender must slow down
```

This connects TCP buffers directly to **flow control**.

------------------------------------------------------------------------

## 10. Timeouts

Applications usually need timeouts so they do not wait forever.

Common examples:

``` text
Connection timeout
Read timeout
Write timeout
Idle connection timeout
Pool acquisition timeout
```

Example:

``` text
Application
   |
   | connect()
   ↓
Server unavailable
   |
   | wait...
   |
Timeout
   ↓
Fail request
```

Timeouts are application/configuration concepts and should be chosen
according to the expected network and service behavior.

------------------------------------------------------------------------

## 11. Connection Exhaustion

A service can run out of available connections or local networking
resources.

Possible causes:

``` text
Too many concurrent clients
+
Connections held too long
+
Poor connection pooling
+
Slow downstream services
+
Connection leaks
```

Example:

``` text
Connection pool max = 100

100 connections
      ↓
All busy
      ↓
New request
      ↓
Waits for connection
      ↓
Pool timeout
```

This can appear as application latency even when CPU usage is low.

------------------------------------------------------------------------

## 12. Connection Churn

Creating and closing connections repeatedly is called **connection
churn**.

``` text
Open → Use → Close
Open → Use → Close
Open → Use → Close
```

High churn can cause:

-   More TCP handshakes
-   More TLS handshakes when HTTPS is used
-   More CPU overhead
-   More `TIME_WAIT` sockets
-   Higher latency
-   Ephemeral port pressure

Pooling and connection reuse reduce this churn.

------------------------------------------------------------------------

## 13. Practical: View TCP States

On Windows:

``` powershell
netstat -ano -p tcp
```

Example:

``` text
Proto  Local Address       Foreign Address      State
TCP    192.168.1.10:52001  142.250.72.14:443    ESTABLISHED
TCP    192.168.1.10:52002  142.250.72.14:443    TIME_WAIT
TCP    192.168.1.10:52003  142.250.72.14:443    CLOSE_WAIT
```

Meaning:

``` text
ESTABLISHED
    ↓
Active TCP connection

TIME_WAIT
    ↓
Connection has closed and the endpoint is waiting as required by TCP

CLOSE_WAIT
    ↓
Peer has closed its sending side;
local application has not fully closed its socket yet
```

A large `CLOSE_WAIT` count can be a useful clue that an application is
not closing connections properly.

------------------------------------------------------------------------

## 14. Practical: Count TIME_WAIT Connections

You can use PowerShell:

``` powershell
Get-NetTCPConnection -State TimeWait | Measure-Object
```

Example:

``` text
Count : 247
```

Meaning:

``` text
247 TCP connections
are currently in TIME_WAIT
```

The number changes with traffic and the OS's TCP behavior.

------------------------------------------------------------------------

## 15. Practical: Count Established Connections

Run:

``` powershell
Get-NetTCPConnection -State Established | Measure-Object
```

Example:

``` text
Count : 84
```

Meaning:

``` text
84 TCP connections
are currently ESTABLISHED
```

This can help during troubleshooting when a service appears to have too
many active connections.

------------------------------------------------------------------------

## 16. Lifecycle Mental Model

``` text
          SYN
Client ────────→ Server
       ← SYN-ACK
Client ────────→ ACK
        ESTABLISHED
             ↓
            DATA
             ↓
          FIN/ACK
             ↓
        Graceful close
             ↓
         TIME_WAIT
```

For an abnormal/reset close:

``` text
Connection
    ↓
RST
    ↓
Immediate termination
```

For performance:

``` text
Short-lived connections
        ↓
Handshake + close repeatedly
        ↓
Connection churn
        ↓
TIME_WAIT + latency + resource usage

Connection pooling
        ↓
Reuse established connections
        ↓
Less churn
        ↓
Better efficiency
```

------------------------------------------------------------------------

## 17. Summary

``` text
FIN
 ↓
Graceful connection shutdown

RST
 ↓
Immediate/reset termination

TIME_WAIT
 ↓
Allows old packets to expire and helps complete TCP termination safely

Keep-alive
 ↓
Detect idle peer failure or reuse connections depending on context

Connection pooling
 ↓
Reuse TCP connections

TCP buffers
 ↓
Store data between application and network

Timeouts
 ↓
Prevent indefinite waiting

Connection exhaustion
 ↓
Too many active/held connections or insufficient resources
```

Core performance relationship:

``` text
        TCP Performance
              |
      ┌───────┼────────┐
      ↓       ↓        ↓
   Buffers  Pooling  Timeouts
      ↓       ↓        ↓
   Flow     Less      Bounded
  control   churn     waiting
      └───────┼────────┘
              ↓
       Connection health
              ↓
        Application latency
```

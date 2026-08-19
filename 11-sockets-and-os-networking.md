# Networking --- Phase 11: Sockets & OS Networking

> **Goal:** Understand how applications use the operating system's
> socket API to communicate over TCP/IP.

## 1. What Is a Socket?

A **socket** is an OS-managed communication endpoint.

Think:

``` text
Application
    ↓
Socket API
    ↓
Operating System
    ↓
TCP / UDP
    ↓
IP
    ↓
Network
```

A TCP socket is associated with local and remote endpoints:

``` text
Local IP + Local Port
Remote IP + Remote Port
```

Example:

``` text
192.168.1.10:50000
        ↕
10.0.0.20:443
```

------------------------------------------------------------------------

## 2. Socket API

Applications normally do not manipulate TCP packets directly.

They use OS APIs such as:

``` text
socket()
bind()
listen()
accept()
connect()
send()
recv()
close()
```

The OS handles much of:

``` text
TCP state
Buffers
Sequence numbers
Retransmission
IP routing
Network interface
```

------------------------------------------------------------------------

## 3. TCP Server Lifecycle

A typical TCP server does:

``` text
socket()
   ↓
bind()
   ↓
listen()
   ↓
accept()
   ↓
recv()/send()
   ↓
close()
```

Conceptually:

``` text
Create socket
     ↓
Assign local IP/port
     ↓
Listen for connections
     ↓
Accept client
     ↓
Exchange data
     ↓
Close
```

------------------------------------------------------------------------

## 4. `socket()`

`socket()` creates a socket.

Conceptually:

``` text
socket(AF_INET, SOCK_STREAM, 0)
```

Meaning:

``` text
AF_INET
   ↓
IPv4

SOCK_STREAM
   ↓
TCP-style byte stream

0
   ↓
Use the appropriate protocol for the socket type
```

For UDP, applications commonly use:

``` text
SOCK_DGRAM
```

------------------------------------------------------------------------

## 5. `bind()`

`bind()` assigns a local address to a socket.

Example:

``` text
bind(
    IP   = 192.168.1.10,
    port = 8080
)
```

The server is saying:

``` text
"I want this socket associated with
192.168.1.10:8080."
```

A server may bind to:

``` text
0.0.0.0:8080
```

meaning port `8080` on the host's local IPv4 interfaces.

------------------------------------------------------------------------

## 6. `listen()`

`listen()` puts a TCP socket into a state where it can accept incoming
connections.

``` text
socket()
   ↓
bind()
   ↓
listen()
```

Conceptually:

``` text
Server
192.168.1.10:8080
        ↓
Waiting for TCP clients
```

The listening socket is not the same thing as the connected socket used
to exchange application data.

------------------------------------------------------------------------

## 7. `accept()`

`accept()` accepts an incoming TCP connection.

``` text
Listening socket
       ↓
   accept()
       ↓
Connected socket
```

Example:

``` text
Server listening:
192.168.1.10:8080

Client:
192.168.1.20:52341
```

After `accept()`:

``` text
Connected socket:

192.168.1.10:8080
        ↕
192.168.1.20:52341
```

The listening socket can continue accepting other clients.

This is why a server can have:

``` text
Listening socket
       +
Client connection 1
Client connection 2
Client connection 3
...
```

------------------------------------------------------------------------

## 8. `connect()`

A client uses `connect()` to establish a TCP connection.

``` text
Client
   ↓
connect(server_ip, server_port)
   ↓
TCP handshake
   ↓
Connected socket
```

Example:

``` text
connect(10.0.0.20, 443)
```

The OS performs the TCP connection establishment.

------------------------------------------------------------------------

## 9. `send()` and `recv()`

Once connected:

``` text
send()
  ↓
TCP send buffer
  ↓
Network
  ↓
TCP receive buffer
  ↓
recv()
```

Example:

``` text
Client                  Server

send("Hello") -------->

          <----------- recv()
```

Important:

> TCP is a byte stream. One `send()` does not necessarily correspond to
> one `recv()`.

For example:

``` text
send("Hello")
send("World")
```

The receiver might read:

``` text
"HelloWorld"
```

or:

``` text
"Hel"
"loWo"
"rld"
```

The application protocol must define message boundaries.

------------------------------------------------------------------------

## 10. Blocking Sockets

A **blocking** socket call can wait until the requested operation can
proceed.

Example:

``` text
recv()
  ↓
No data available
  ↓
Thread waits
  ↓
Data arrives
  ↓
recv() returns
```

Simple model:

``` text
Application thread
       ↓
     recv()
       ↓
    BLOCKED
       ↓
Data arrives
       ↓
    Continue
```

Blocking APIs are simple to program but can tie up threads when many
connections are waiting.

------------------------------------------------------------------------

## 11. Non-Blocking Sockets

A **non-blocking** socket returns control to the application instead of
waiting indefinitely.

Conceptually:

``` text
recv()
  ↓
No data available
  ↓
Return immediately
```

The application can then:

``` text
Do other work
      ↓
Check socket later
```

Non-blocking sockets are commonly used with event mechanisms such as:

``` text
select()
poll()
epoll()
kqueue()
IOCP
```

The exact mechanism depends on the operating system.

------------------------------------------------------------------------

## 12. Blocking vs Non-Blocking

  -----------------------------------------------------------------------
                          Blocking                Non-blocking
  ----------------------- ----------------------- -----------------------
  Waits for I/O           Yes                     No

  Programming model       Simpler                 More complex

  Many connections        Often needs             Efficient with event
                          threads/processes       loops

  Typical use             Simple servers/apps     High-concurrency
                                                  servers
  -----------------------------------------------------------------------

Mental model:

``` text
Blocking:

recv()
  ↓
WAIT
  ↓
data
  ↓
continue


Non-blocking:

recv()
  ↓
No data?
  ↓
Return
  ↓
Do other work
```

------------------------------------------------------------------------

## 13. Ephemeral Ports

Clients usually need a local source port.

Instead of choosing a fixed application port, the OS can assign an
**ephemeral port**.

Example:

``` text
Client:
192.168.1.10:52341

Server:
142.250.72.14:443
```

Here:

``` text
52341
 ↓
Ephemeral client port
```

This allows many client connections to the same server port.

For example:

``` text
192.168.1.10:52341 → Server:443
192.168.1.10:52342 → Server:443
192.168.1.10:52343 → Server:443
```

The OS manages the ephemeral port range; the exact range depends on the
operating system.

------------------------------------------------------------------------

## 14. Connection Lifecycle

Typical TCP client:

``` text
socket()
   ↓
connect()
   ↓
send()/recv()
   ↓
close()
```

Typical TCP server:

``` text
socket()
   ↓
bind()
   ↓
listen()
   ↓
accept()
   ↓
send()/recv()
   ↓
close()
```

Important distinction:

``` text
Listening socket
       ↓
Accepts connections

Connected socket
       ↓
Exchanges application data
```

------------------------------------------------------------------------

## 15. Practical: View Listening Ports

On Windows:

``` powershell
netstat -ano -p tcp
```

Example:

``` text
Proto  Local Address      Foreign Address    State
TCP    0.0.0.0:8080       0.0.0.0:0          LISTENING
```

Meaning:

``` text
0.0.0.0:8080
    ↓
TCP port 8080 is listening
on the host's local IPv4 interfaces.
```

------------------------------------------------------------------------

## 16. Practical: Find Which Process Owns a Port

The `-o` option shows the PID.

Example:

``` powershell
netstat -ano | findstr ":8080"
```

Output:

``` text
TCP    0.0.0.0:8080    0.0.0.0:0    LISTENING    1234
```

Meaning:

``` text
Port   = 8080
State  = LISTENING
PID    = 1234
```

Then:

``` powershell
tasklist /FI "PID eq 1234"
```

Example:

``` text
Image Name      PID
java.exe        1234
```

Meaning:

``` text
java.exe
   ↓
owns PID 1234
   ↓
listening on port 8080
```

------------------------------------------------------------------------

## 17. Practical: Test a TCP Port

PowerShell provides:

``` powershell
Test-NetConnection google.com -Port 443
```

Example:

``` text
ComputerName     : google.com
RemotePort       : 443
TcpTestSucceeded : True
```

Meaning:

``` text
TCP connection to
google.com:443
was successfully established.
```

If:

``` text
TcpTestSucceeded : False
```

the TCP connection could not be established from your machine at that
time.

------------------------------------------------------------------------

## 18. Socket and TCP Relationship

The application sees:

``` text
Socket API
```

The OS implements:

``` text
TCP
IP
Network interface
```

So:

``` text
Application
     ↓
send()/recv()
     ↓
Socket
     ↓
TCP
     ↓
IP
     ↓
Network
```

This abstraction lets applications communicate without implementing TCP
themselves.

------------------------------------------------------------------------

## 19. Important Mental Model

Think of a socket as the application's handle to network communication.

``` text
Application
     ↓
Socket
     ↓
OS networking stack
     ↓
TCP/UDP
     ↓
IP
     ↓
Network
```

TCP server:

``` text
socket
   ↓
bind
   ↓
listen
   ↓
accept
   ↓
connected socket
   ↓
send / recv
   ↓
close
```

TCP client:

``` text
socket
   ↓
connect
   ↓
send / recv
   ↓
close
```

------------------------------------------------------------------------

## 20. Summary

``` text
Socket
 ↓
OS communication endpoint

socket()
 ↓
Create socket

bind()
 ↓
Assign local IP/port

listen()
 ↓
Wait for TCP connections

accept()
 ↓
Create/return connected socket

connect()
 ↓
Establish connection to server

send()
 ↓
Send bytes

recv()
 ↓
Receive bytes

Blocking
 ↓
Wait for I/O

Non-blocking
 ↓
Return without waiting

Ephemeral port
 ↓
Temporary client-side port

close()
 ↓
Release socket / terminate communication
```

Core server flow:

``` text
socket()
   ↓
bind()
   ↓
listen()
   ↓
accept()
   ↓
send()/recv()
   ↓
close()
```

Core client flow:

``` text
socket()
   ↓
connect()
   ↓
send()/recv()
   ↓
close()
```

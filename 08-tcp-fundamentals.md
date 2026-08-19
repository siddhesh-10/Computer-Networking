# Networking --- Phase 8: TCP Fundamentals

> **Goal:** Understand how TCP establishes communication and provides
> reliable, ordered delivery between applications.

## 1. What Is TCP?

**TCP (Transmission Control Protocol)** is a transport-layer protocol.

``` text
Application
    ↓
TCP
    ↓
IP
    ↓
Ethernet / Wi-Fi
```

TCP provides:

-   Connection-oriented communication
-   Reliable delivery
-   Ordered delivery
-   Sequence numbers
-   Acknowledgements
-   Retransmission

------------------------------------------------------------------------

## 2. TCP Connection

TCP is **connection-oriented**.

Before application data is exchanged, TCP establishes a connection
between two endpoints.

``` text
Client
  ↓
TCP connection
  ↓
Server
```

A connection is identified by:

``` text
Source IP + Source Port
Destination IP + Destination Port
```

Example:

``` text
192.168.1.10:50000
        ↕
142.250.72.14:443
```
``` text
The TCP connection is a logical connection between two endpoints, identified mainly by:

Source IP + Source Port
Destination IP + Destination Port

For example:

PC                         Server
192.168.1.10:50000  →  142.250.x.x:443

After the TCP 3-way handshake:

PC                         Server
 │                            │
 │── SYN ────────────────────→│
 │←── SYN-ACK ────────────────│
 │── ACK ─────────────────────→│
 │                            │
 │====== TCP connection ======│

Now application data can flow.

But does it skip IP/Ethernet?

No. Every packet still goes through the networking layers.

For every piece of data:

Application
    ↓
TCP
    ↓
IP
    ↓
Ethernet / Wi-Fi
    ↓
Router
    ↓
...
    ↓
Server

TCP knows:

"I am communicating with 142.250.x.x:443"

But TCP doesn't decide how to physically reach that server.

The IP layer still does:

Destination IP
      ↓
Routing table
      ↓
Next hop

And each local link still uses Layer 2:

PC → Router A
MAC: PC → Router A


Router A → Router B
MAC: Router A → Router B


Router B → Server
MAC: Router B → Server
What does the TCP connection actually establish then?

It establishes state at both endpoints.

For example, TCP remembers things such as:

Connection established
Sequence numbers
Acknowledgments
Window size
Retransmission state
Congestion-control state

So after the handshake, when you send:

"Hello"

TCP doesn't need another handshake. It creates TCP segments and sends them through the normal IP routing system.

One subtle point

The route can even change during an established TCP connection.

For example:

Before:
PC → Router A → Router B → Server


Later:
PC → Router A → Router C → Router D → Server

The TCP connection can continue because the endpoints are still communicating with the same IP/port pair.

So remember:

TCP connection = logical relationship between two applications.

IP routing = determines how each packet gets toward the destination.

Ethernet/Wi-Fi = delivers the packet to the next hop on each local link.
``` 
------------------------------------------------------------------------

## 3. Three-Way Handshake

TCP normally establishes a connection using three messages:

``` text
Client                     Server

   SYN  -------------------->
        <---------------- SYN-ACK
   ACK  -------------------->
```

After this:

``` text
TCP connection established
```

------------------------------------------------------------------------

## 4. SYN

**SYN** means synchronize.

The client sends a SYN to request a TCP connection.

Example:

``` text
Client → Server

SYN
Sequence Number = 100
```

The sequence number is the starting sequence number chosen by the
client.

Conceptually:

``` text
"I want to establish a connection.
 My initial sequence number is 100."
```

------------------------------------------------------------------------

## 5. SYN-ACK

The server responds with:

``` text
SYN + ACK
```

Example:

``` text
Server → Client

SYN
Sequence Number = 500

ACK
Acknowledgement Number = 101
```

The server is saying:

``` text
"I received your SYN with sequence number 100.
 I expect your next sequence number to be 101.
 My sequence number starts at 500."
```

------------------------------------------------------------------------

## 6. Final ACK

The client sends:

``` text
ACK
Acknowledgement Number = 501
```

So:

``` text
Client                     Server

SYN, Seq=100 ------------>

       <------------- SYN-ACK
                    Seq=500
                    Ack=101

ACK, Ack=501 ------------>

        Connection established
```

Both sides have now synchronized their initial sequence numbers.

------------------------------------------------------------------------

## 7. Why Three Messages?

Both sides need to establish their sequence-number state.

``` text
Client → SYN
"My starting sequence number is 100"

Server → SYN-ACK
"I received yours.
 Mine is 500."

Client → ACK
"I received yours."
```

Now both directions are ready for data transfer.

------------------------------------------------------------------------

## 8. Sequence Numbers

TCP treats transmitted data as an ordered **sequence of bytes**.

Example:

``` text
Application data:

HELLO
```

Conceptually:

``` text
H E L L O
↓ ↓ ↓ ↓ ↓
100 101 102 103 104
```

The sequence number identifies the position of data in the TCP byte
stream.

> TCP sequence numbers track **bytes**, not individual packets.

------------------------------------------------------------------------

## 9. Acknowledgements

An ACK tells the sender which data has been successfully received.

Suppose:

``` text
Sender → Receiver

Seq = 100
Data = 10 bytes
```

The receiver can respond:

``` text
ACK = 110
```

Meaning:

``` text
"I have received bytes 100–109.
 I expect byte 110 next."
```

TCP acknowledgements are generally **cumulative**.

------------------------------------------------------------------------

## 10. Reliable Delivery

Suppose:

``` text
Sender
  |
  | Segment 1
  | Segment 2
  | Segment 3
  ↓
Receiver
```

If Segment 2 is lost:

``` text
Segment 1 → received
Segment 2 → LOST
Segment 3 → received
```

TCP can detect the missing data and retransmit it.

``` text
Sender → Segment 2 → X
                  packet lost

Receiver → ACKs indicate missing progress

Sender → Retransmit Segment 2
```

This allows TCP to provide reliable delivery over IP's best-effort
network.

------------------------------------------------------------------------

## 11. Ordered Delivery

IP packets can arrive out of order.

``` text
Sender:
1 → 2 → 3

Network:
1 → Receiver
3 → Receiver
2 → Receiver
```

TCP uses sequence numbers to reconstruct the correct order:

``` text
1
2
3
```

The application receives an ordered byte stream.

------------------------------------------------------------------------

## 12. TCP Segment

TCP data is carried in a TCP segment.

``` text
+----------------------+
| TCP Header           |
+----------------------+
| Application Data     |
+----------------------+
```

Important TCP header fields include:

``` text
Source Port
Destination Port
Sequence Number
Acknowledgement Number
Flags
Window
Checksum
```

For this phase, focus on:

``` text
Sequence Number
Acknowledgement Number
SYN
ACK
```

------------------------------------------------------------------------

## 13. TCP Flags

Important flags:

``` text
SYN → Start/synchronize connection
ACK → Acknowledgement field is valid
FIN → Gracefully close connection
RST → Reset connection
```

The three-way handshake uses:

``` text
SYN
SYN + ACK
ACK
```

------------------------------------------------------------------------

## 14. TCP vs UDP

  Feature             TCP           UDP
  ------------------- ------------- -----------
  Connection          Yes           No
  Handshake           Yes           No
  Reliable delivery   Yes           No
  Ordered delivery    Yes           No
  Sequence numbers    Yes           No
  Acknowledgements    Yes           No
  Retransmission      Yes           No
  Data model          Byte stream   Datagrams

Key difference:

``` text
TCP
 ↓
Reliable ordered byte stream

UDP
 ↓
Independent datagrams
```

------------------------------------------------------------------------

## 15. TCP vs UDP Data Model

TCP provides a **byte stream**.

If an application writes:

``` text
"Hello"
"World"
```

TCP does not preserve those as two separate messages.

The receiver sees a stream of bytes:

``` text
HelloWorld
```

UDP preserves datagram boundaries:

``` text
Datagram 1 = "Hello"
Datagram 2 = "World"
```

------------------------------------------------------------------------

## 16. Practical: Check TCP Connections

On Windows:

``` powershell
netstat -ano -p tcp
```

Example:

``` text
Proto  Local Address       Foreign Address      State
TCP    192.168.1.10:52341  142.250.72.14:443    ESTABLISHED
```

Meaning:

``` text
Protocol        = TCP
Local endpoint  = 192.168.1.10:52341
Remote endpoint = 142.250.72.14:443
State           = ESTABLISHED
```

`ESTABLISHED` means the TCP connection has been successfully
established.

------------------------------------------------------------------------

## 17. Practical: Find HTTPS Connections

Run:

``` powershell
netstat -ano -p tcp | findstr ":443"
```

Possible output:

``` text
TCP    192.168.1.10:52341    142.250.72.14:443    ESTABLISHED
```

Here:

``` text
443
 ↓
HTTPS server port

52341
 ↓
Client ephemeral port
```

The exact output depends on running applications.

------------------------------------------------------------------------

## 18. TCP Flow

Once the handshake completes:

``` text
Client                         Server

SYN ------------------------->

    <------------------- SYN-ACK

ACK -------------------------->

Data ------------------------->

    <------------------------- ACK

More data ------------------->

    <------------------------- ACK
```

TCP continuously uses:

``` text
Sequence numbers
+
Acknowledgements
```

to maintain reliable ordered delivery.

------------------------------------------------------------------------

## 19. Important Mental Model

Think of TCP as creating a reliable byte stream on top of IP.

``` text
Application
    ↓
"I want to send these bytes"
    ↓
TCP
    ↓
Add sequence numbers
    ↓
Send segments
    ↓
Receiver acknowledges data
    ↓
Missing data is retransmitted
    ↓
Data is reordered if necessary
    ↓
Application receives ordered stream
```

IP itself does not provide these guarantees.

``` text
IP
 ↓
Best-effort packet delivery

TCP
 ↓
Reliable + ordered byte stream
```

------------------------------------------------------------------------

## 20. Summary

``` text
TCP
 ↓
Connection-oriented transport

Three-way handshake
 ↓
SYN
 ↓
SYN-ACK
 ↓
ACK
 ↓
Connection established

Sequence numbers
 ↓
Identify byte positions

Acknowledgements
 ↓
Confirm received data

Retransmission
 ↓
Recover lost data

Ordering
 ↓
Reconstruct correct byte stream
```

Core flow:

``` text
       TCP CONNECTION
             ↓
SYN → SYN-ACK → ACK
             ↓
       Data transfer
             ↓
Sequence numbers
             +
Acknowledgements
             ↓
       Reliable delivery
             ↓
        Ordered stream
```

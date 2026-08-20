# Networking --- Phase 9: TCP Deep Dive

> **Goal:** Understand how TCP controls how much data can be in flight,
> detects loss, retransmits data, and adapts its sending rate to network
> conditions.

## 1. Sliding Window

TCP can send multiple segments before waiting for an ACK.

Without a window:

``` text
Send → ACK → Send → ACK → Send
```

With a sliding window:

``` text
Send 1
Send 2
Send 3
Send 4
      ↓
Receive ACKs
      ↓
Window moves forward
```

This keeps the network busy instead of waiting after every segment.

Conceptually:

``` text
[ACKed][ACKed][In flight][In flight][Can send][Can send]
   ←──────────── TCP window ─────────────→
```

As ACKs arrive, the window **slides forward**.

------------------------------------------------------------------------

## 2. Flow Control

Flow control protects the **receiver** from being overwhelmed.

The receiver advertises how much additional data it can accept.

``` text
Sender
  ↓
TCP data
  ↓
Receiver buffer
```

If the receiver is busy:

``` text
Available buffer ↓
Receive window ↓
Sender slows down
```

This is different from congestion control.

``` text
Flow control
 ↓
Protects receiver

Congestion control
 ↓
Protects network
```

------------------------------------------------------------------------

## 3. Receive Window

TCP uses the **receive window (`rwnd`)** to tell the sender how much
data the receiver can currently accept.

Example:

``` text
Receiver advertises:

rwnd = 20 KB
```

The sender should not have more than roughly 20 KB of unacknowledged
data outstanding because of receiver capacity.

As the application reads data from the receive buffer:

``` text
Buffer space increases
      ↓
Advertised window increases
      ↓
Sender can send more
```

If:

``` text
rwnd = 0
```

the receiver is temporarily unable to accept more data. TCP can use
window-update mechanisms to resume transmission when space becomes
available.

------------------------------------------------------------------------

## 4. Retransmission

TCP retransmits data when it believes a segment was lost.

Example:

``` text
Sender        Receiver

Seg 1 -------->

Seg 2 --------> X

Seg 3 -------->

ACK ---------->

ACK ---------->

        Seg 2 lost
```

TCP can infer that data is missing and retransmit it:

``` text
Sender
  |
  └── Retransmit Seg 2 ──→ Receiver
```

Retransmission is one of the mechanisms that gives TCP reliable
delivery.

------------------------------------------------------------------------

## 5. Retransmission Timeout

If an expected ACK does not arrive within the estimated retransmission
timeout (**RTO**), TCP can retransmit the outstanding data.

Conceptually:

``` text
Send segment
     ↓
Start/maintain timer
     ↓
ACK arrives?
   /      Yes      No
  ↓        ↓
Continue  RTO expires
           ↓
       Retransmit
```

TCP does not use one fixed timeout for every network.

It estimates network round-trip behavior and adapts the retransmission
timeout.

------------------------------------------------------------------------

## 6. RTT and RTO

**RTT (Round-Trip Time)** is approximately the time between sending data
and receiving the corresponding acknowledgement.

Example:

``` text
Sender ───────→ Receiver
       40 ms
Sender ←─────── Receiver
       40 ms

RTT ≈ 80 ms
```

TCP measures RTT samples and uses them to estimate an appropriate RTO.

A very short RTO on a high-latency network could cause unnecessary
retransmissions.

A very long RTO could delay recovery from real packet loss.

------------------------------------------------------------------------

## 7. Congestion Control

Flow control protects the receiver.

Congestion control protects the **network**.

A network can become overloaded when senders inject more traffic than
the path can handle.

``` text
Many senders
     ↓
Routers / links
     ↓
Congestion
     ↓
Queueing / packet loss
```

TCP responds by adjusting its sending rate.

The main control variable is the **congestion window (`cwnd`)**.

The sender's effective amount of in-flight data is constrained by both:

``` text
Receive window
+
Congestion window
```

A useful simplified model is:

``` text
send window ≈ min(rwnd, cwnd)
```

------------------------------------------------------------------------

## 8. Slow Start

Despite its name, **slow start** can increase the sending rate quickly.

At the beginning of a connection, TCP starts with a relatively small
congestion window.

As ACKs arrive, `cwnd` grows rapidly, approximately exponentially per
RTT during slow start.

Conceptually:

``` text
RTT 1 → 1 unit
RTT 2 → 2 units
RTT 3 → 4 units
RTT 4 → 8 units
```

The exact behavior depends on the TCP implementation and protocol
version.

Slow start continues until a threshold or congestion signal causes TCP
to change behavior.

------------------------------------------------------------------------

## 9. Congestion Avoidance

After slow start reaches the **slow-start threshold (`ssthresh`)**, TCP
generally enters congestion avoidance.

The growth becomes much more conservative.

Simplified:

``` text
Slow start
   ↓
Rapid growth

ssthresh
   ↓

Congestion avoidance
   ↓
More gradual growth
```

A common mental model is:

``` text
Slow start       → exponential-ish growth
Congestion avoid → linear-ish growth
```

Exact algorithms vary between TCP implementations.

------------------------------------------------------------------------

## 10. Packet Loss as a Congestion Signal

TCP historically treats packet loss as an indication that congestion may
exist.

Example:

``` text
cwnd increasing
      ↓
Packet loss
      ↓
TCP reduces sending rate
      ↓
cwnd grows again
```

Modern TCP implementations can also use other signals, such as ECN, but
loss-based congestion control remains fundamental.

------------------------------------------------------------------------

## 11. Fast Retransmit

Waiting for an RTO is slow.

TCP can often detect loss earlier through **duplicate ACKs**.

Suppose:

``` text
Sender:

Segment 1
Segment 2
Segment 3
Segment 4
```

Segment 2 is lost:

``` text
Receiver gets:
1
3
4
```

Because the receiver is still missing Segment 2, it can repeatedly
acknowledge the next expected data.

Conceptually:

``` text
ACK 2
ACK 2
ACK 2
```

Multiple duplicate ACKs can indicate that a segment was lost.

TCP can then:

``` text
Fast retransmit
      ↓
Retransmit missing segment
```

without waiting for the retransmission timeout.

------------------------------------------------------------------------

## 12. Fast Recovery

After fast retransmit, TCP should reduce its sending rate, but it does
not necessarily need to fall all the way back to the beginning of slow
start.

**Fast recovery** is associated with recovering from loss detected
through duplicate ACKs while maintaining useful in-flight data.

Conceptually:

``` text
Normal transmission
        ↓
Duplicate ACKs
        ↓
Fast retransmit
        ↓
Reduce congestion window
        ↓
Fast recovery
        ↓
Congestion avoidance
```

The exact behavior depends on the TCP congestion-control algorithm.

------------------------------------------------------------------------

## 13. Flow Control vs Congestion Control

This distinction is extremely important.

``` text
                  Flow Control              Congestion Control
  --------------- ------------------------- ----------------------
  Protects        Receiver                  Network
  Main signal     `rwnd`                    `cwnd`
  Problem         Receiver buffer filling   Network overload
  Controlled by   Receiver                  Sender/path feedback
```

Simplified:

``` text
Receiver capacity
      ↓
     rwnd
      ↓
Protect receiver


Network capacity
      ↓
     cwnd
      ↓
Protect network
```

The sender is constrained by both.

``` text
Effective sending window
        ≈
min(rwnd, cwnd)
```

------------------------------------------------------------------------

## 14. Practical: Observe TCP Connections

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

`netstat` shows connection state, but it does **not** expose all TCP
internals such as `cwnd`, `rwnd`, or every retransmission decision.

------------------------------------------------------------------------

## 15. Practical: Inspect TCP Statistics

Run:

``` powershell
netstat -s -p tcp
```

Example output can contain:

``` text
TCP Statistics for IPv4

  Segments Received = 123456
  Segments Sent = 120321
  Segments Retransmitted = 142
```

Meaning:

``` text
Segments Received
    ↓
TCP segments received by the host

Segments Sent
    ↓
TCP segments transmitted

Segments Retransmitted
    ↓
TCP segments sent again after loss/recovery detection
```

Exact counters and formatting depend on the Windows version and current
traffic.

------------------------------------------------------------------------

## 16. TCP Deep-Dive Mental Model

Think of TCP as balancing three things:

``` text
              TCP Sender
                  |
        ┌─────────┴─────────┐
        ↓                   ↓
   Receiver capacity    Network capacity
        ↓                   ↓
       rwnd                cwnd
        └─────────┬─────────┘
                  ↓
        Effective send window
                  ↓
             TCP segments
                  ↓
               Network
                  ↓
        ACKs / loss signals
                  ↓
        Adjust transmission
```

For loss:

``` text
Packet loss
    ↓
Duplicate ACKs or timeout
    ↓
Retransmission
    ↓
Congestion response
    ↓
Reduce sending rate
    ↓
Recover and increase again
```

------------------------------------------------------------------------

## 17. Summary

``` text
Sliding window
 ↓
Allows multiple bytes to be in flight

Flow control
 ↓
Protects receiver
 ↓
rwnd

Retransmission
 ↓
Recovers lost data

RTO
 ↓
Triggers retransmission when ACKs take too long

Congestion control
 ↓
Protects network
 ↓
cwnd

Slow start
 ↓
Rapid initial growth

Congestion avoidance
 ↓
More conservative growth

Fast retransmit
 ↓
Uses duplicate ACKs to detect loss early

Fast recovery
 ↓
Reduces sending rate and continues recovery
```

Core relationship:

``` text
                 TCP
                  |
        ┌─────────┴─────────┐
        ↓                   ↓
   Flow control       Congestion control
        ↓                   ↓
      rwnd                cwnd
        └─────────┬─────────┘
                  ↓
       Effective window
          min(rwnd,cwnd)
                  ↓
            Data in flight
                  ↓
             ACK / loss
                  ↓
        Adjust sending rate
```

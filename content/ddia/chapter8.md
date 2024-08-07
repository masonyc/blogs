---
date created: 2024-08-08T08:00
date modified: 2024-08-08T08:00
---

# Unreliable Networks

> When one part of the network is cut off from the rest due to a network fault, that is sometimes called a **network partition** or **netsplit**

TCP performs flow control (also known as congestion avoidance or backpressure), in which a node limits its own rate of sending in order to avoid overloading a network link or the receiving node

## UDP vs TCP

It’s a trade-off between **reliability** and **variability** of delays: as UDP does not perform flow control and does not retransmit lost packets. UDP is a good choice in situations where delayed data is worthless

# Unreliable Clocks

Clock is not reliable, each machine has its own notion of time

It is possible to synchronize clocks to some degree: the most commonly used mechanism is the **Network Time Protocol (NTP)**, which allows the computer clock to be adjusted according to the time reported by a group of servers

# Monotonic vs Time-of-Day Clocks

Time-of-Day clock - can jump back to a previous old point in time after NTP synchronization

Monotonic clock - guaranteed to always move forward, suitable for checking how much time elapesed. However, it does **not make sense to compare monotonic clock values from two different computers**, because they don't mean the same thing

## Problem with Last Write Wins

- Database writes can mysteriously disappear: a node with a lagging clock is unable to overwrite
  values previously written by a node with a fast clock until the clock skew between the nodes has
  elapsed

- LWW cannot distinguish between writes that occurred sequentially in quick succession (client B’s increment occurs after client A’s write in a short gap) and writes that were truly concurrent (neither writer was aware of the other). Additional causality tracking mechanisms, such as version vectors, are needed in order to prevent violations of causality

- It is possible for two nodes to independently generate writes with the same timestamp, especially when the clock only has millisecond resolution. An additional tiebreaker value (which can simply be a large random number) is required to resolve such conflicts, but this approach can also lead to violations of causality

**Logical clocks**, which are based on incrementing counters rather than an oscillating quartz crystal, are a safer alternative for ordering events. Logical clocks do not measure the time of day or the number of seconds elapsed, only the relative ordering of events (whether one event happened before or after another). In contrast, time-of-day and monotonic clocks, which measure actual elapsed time, are also known as **physical clocks**

> !!! Physical clock is not reliable even with NTP clock server. Because it will still have the network delay

## Process Pauses

Single leader sceario - uses a lease to identify self as leader is dangerous

1. Lease time is from another server which has a differen clock -- Unreliable
2. If using local monotonic clocks
   - The process can pause after <code>lease.isValid()</code>, and when the process resume, it will no longer be a leader as lease will be expired,
     but it will not aware of the situation and continue process until it checks the clock again

```
while (true) {
    request = getIncomingRequest();

    // Ensure that the lease always has at least 10 seconds remaining
    if (lease.expiryTimeMillis - System.currentTimeMillis() < 10000) {
        lease = lease.renew();
    }

    if (lease.isValid()) {
        process(request);
    }
}

```

# Knowledge, Truth and Lies
## Fencing
Let’s assume that every time the lock server grants a lock or lease, it also returns a fencing token, which is a number that increases every time a lock is granted (e.g., incremented by the lock service). We can then require that every time a client sends a write request to the storage service, it must include its current fencing token

Checking a token on the server side may seem like a downside, but it is arguably a good thing: it is unwise for a service to assume that its clients will always be well behaved, because the clients are often run by people whose priorities are very different from the priorities of the people running the service

## Byzantine Faults
Fencing tokens can detect and block a node that is inadvertently acting in error (e.g., because it hasn’t yet found out that its lease has expired). However, if the node deliberately wanted to subvert the system’s guarantees, it could easily do so by sending messages with a fake fencing token

If a node may claim to have received a particular message when in fact it didn’t. Such behavior is known as a Byzantine fault, and the problem of reaching consensus in this untrusting environment is known as the **Byzantine Generals Problem**

A system is **Byzantine fault-tolerant** if it continues to operate correctly even if some of the nodes are malfunctioning and not obeying the protocol, or if malicious attackers are interfering with the network. This concern is relevant in certain specific circumstances

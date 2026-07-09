# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 08 --- Network Connectivity (Part 2)

------------------------------------------------------------------------

# Understanding `ping` Output

The `ping` command provides valuable information about network
communication.

Example:

``` text
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=15.3 ms
```

Every field tells us something about the connection.

------------------------------------------------------------------------

# Bytes

    64 bytes

This is the size of the ICMP reply received from the destination.

By default, Linux sends small packets because the goal is to test
connectivity rather than transfer data.

------------------------------------------------------------------------

# ICMP Sequence Number

    icmp_seq=1
    icmp_seq=2
    icmp_seq=3

Each packet receives a sequence number.

These numbers help identify:

-   Missing packets
-   Delayed packets
-   Packet order

If packet 4 never appears, packet loss has occurred.

------------------------------------------------------------------------

# Time To Live (TTL)

    ttl=117

TTL stands for **Time To Live**.

Despite its name, TTL is **not measured in seconds**.

Instead, TTL represents the maximum number of routers (hops) a packet
may pass through before it is discarded.

Every router decreases the TTL by one.

Example:

    Client

    TTL = 64
          │
    Router 1 → TTL = 63
          │
    Router 2 → TTL = 62
          │
    Destination

If TTL reaches zero before arriving at the destination, the packet is
discarded.

TTL prevents packets from circulating forever because of routing loops.

------------------------------------------------------------------------

# Round-Trip Time

    time=15.3 ms

This is the **round-trip time (RTT)**.

It measures how long it takes for a packet to:

1.  Leave your computer.
2.  Reach the destination.
3.  Return to your computer.

Lower values generally indicate better network performance.

------------------------------------------------------------------------

# Ping Statistics

When you stop `ping` using:

``` text
Ctrl + C
```

Linux displays statistics.

Example:

``` text
5 packets transmitted
5 received
0% packet loss
time 4005ms

rtt min/avg/max/mdev = 12.1/13.5/15.2/0.9 ms
```

------------------------------------------------------------------------

# Packets Transmitted

    5 packets transmitted

Number of requests sent.

------------------------------------------------------------------------

# Packets Received

    5 received

Number of replies received.

If this number is smaller than the transmitted count, packet loss
occurred.

------------------------------------------------------------------------

# Packet Loss

    0% packet loss

Ideal result.

Higher percentages indicate communication problems.

------------------------------------------------------------------------

# RTT Statistics

    min / avg / max / mdev

Meaning:

-   **min** → Fastest response
-   **avg** → Average response
-   **max** → Slowest response
-   **mdev** → Variation in response times (jitter)

Consistently low RTT values indicate a stable connection.

------------------------------------------------------------------------

# When Ping Succeeds but the Application Fails

This is a very common production situation.

Example:

``` bash
ping server.example.com
```

Succeeds.

However:

``` bash
curl http://server.example.com
```

Fails.

Possible causes:

-   Web server stopped
-   Wrong listening port
-   Firewall
-   Application crash
-   Reverse proxy issue

Ping only proves that the host responded to ICMP.

It does **not** prove that application services are available.

------------------------------------------------------------------------

# Linux Perspective

Professional administrators never stop after a successful ping.

They continue verifying:

-   DNS
-   Routing
-   Open ports
-   Listening services
-   Application health

Connectivity is only the first step.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Every field of `ping` output
-   TTL
-   RTT
-   Packet statistics
-   Why successful ping does not guarantee application availability

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn `traceroute`, `tracepath`, and `mtr` to discover how packets
travel across networks and identify where connectivity problems occur.

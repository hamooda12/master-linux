# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 08 --- Network Connectivity (Part 3)

------------------------------------------------------------------------

# Understanding the Network Path

Knowing that a destination is unreachable is useful.

Knowing **where** communication fails is even more valuable.

Linux provides several tools that show the path packets take across a
network.

These tools help answer questions such as:

-   Which routers does my traffic pass through?
-   Where is communication failing?
-   Which network link introduces high latency?

------------------------------------------------------------------------

# The `traceroute` Command

## Purpose

`traceroute` displays the route packets take from your computer to a
destination.

Instead of showing only whether the destination is reachable, it
displays every intermediate router (hop).

Example:

``` bash
traceroute google.com
```

------------------------------------------------------------------------

# Example Output

``` text
1  192.168.1.1      1.2 ms
2  10.0.0.1         6.5 ms
3  172.16.5.1      11.8 ms
4  142.250.190.78  18.4 ms
```

Each line represents one router.

------------------------------------------------------------------------

# Understanding the Output

## Hop Number

    1
    2
    3
    4

Each router along the path is called a **hop**.

Hop 1 is usually your default gateway.

------------------------------------------------------------------------

## Router Address

    192.168.1.1

The IP address of the router that forwarded the packet.

------------------------------------------------------------------------

## Response Time

    1.2 ms

The time required for that router to respond.

Increasing values often indicate increasing distance.

------------------------------------------------------------------------

# How `traceroute` Works

`traceroute` sends packets with gradually increasing TTL values.

Example:

    TTL = 1

    ↓

    Router 1 replies

    TTL = 2

    ↓

    Router 1

    ↓

    Router 2 replies

    TTL = 3

    ↓

    Router 1

    ↓

    Router 2

    ↓

    Router 3 replies

By increasing TTL one hop at a time, Linux discovers the complete route.

------------------------------------------------------------------------

# The `tracepath` Command

`tracepath` performs a similar function to `traceroute`.

Example:

``` bash
tracepath google.com
```

Advantages:

-   Usually requires fewer privileges.
-   Automatically discovers the network MTU.
-   Simpler output.

Many Linux administrators use `tracepath` for quick investigations.

------------------------------------------------------------------------

# The `mtr` Command

`mtr` (My Traceroute) combines the functionality of:

-   `ping`
-   `traceroute`

Example:

``` bash
mtr google.com
```

Unlike `traceroute`, `mtr` continuously updates statistics.

It displays:

-   Packet loss
-   Latency
-   Route changes

This makes it excellent for diagnosing intermittent network problems.

------------------------------------------------------------------------

# Comparing Connectivity Tools

  Command        Purpose
  -------------- --------------------------------
  `ping`         Verify connectivity
  `traceroute`   Display packet path
  `tracepath`    Trace route with MTU discovery
  `mtr`          Continuous route analysis

------------------------------------------------------------------------

# Linux Perspective

Professional administrators often use these tools together.

Typical workflow:

1.  Verify connectivity using `ping`.
2.  If communication fails, inspect the route using `traceroute` or
    `tracepath`.
3.  Use `mtr` to monitor packet loss and latency over time.

------------------------------------------------------------------------

# Production Example

Users report that a cloud-hosted application is extremely slow.

`ping` succeeds.

However:

``` bash
mtr app.example.com
```

reveals severe packet loss beginning at the seventh hop.

The application is healthy.

The network path is causing the problem.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   `traceroute`
-   `tracepath`
-   `mtr`
-   Hops
-   TTL-based route discovery
-   Route analysis

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the lesson with production troubleshooting scenarios,
interview questions, hands-on labs, challenge exercises, and a complete
network connectivity cheat sheet.

# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 06 --- Routing (Part 3)

------------------------------------------------------------------------

# Route Selection

Linux may have multiple routes that could potentially match a
destination.

How does it decide which one to use?

The Linux kernel follows a simple rule:

> **Choose the most specific matching route.**

This is called the **Longest Prefix Match**.

------------------------------------------------------------------------

## Example

Routing table:

``` text
10.0.0.0/8          via 192.168.1.1
10.10.0.0/16        via 192.168.1.2
10.10.20.0/24       via 192.168.1.3
default             via 192.168.1.254
```

Destination:

``` text
10.10.20.55
```

Linux checks every route.

The matching routes are:

    10.0.0.0/8
    10.10.0.0/16
    10.10.20.0/24

The most specific match is:

``` text
10.10.20.0/24
```

Therefore Linux forwards the packet through:

    192.168.1.3

------------------------------------------------------------------------

# Why Longest Prefix Match?

Imagine the following routes:

    City

    ↓

    Neighborhood

    ↓

    Street

    ↓

    House

The house address is more specific than the city.

Likewise:

    /24

is more specific than:

    /16

Linux always prefers the most precise route.

------------------------------------------------------------------------

# Static Routes

A **static route** is manually configured by an administrator.

Example:

``` text
172.20.0.0/16 via 192.168.1.10
```

This instructs Linux:

> "To reach the 172.20.0.0/16 network, always send packets to
> 192.168.1.10."

Static routes are common in:

-   Enterprise networks
-   Data centers
-   Cloud environments
-   Lab environments

------------------------------------------------------------------------

# Dynamic Routing (Overview)

Large organizations usually do not configure every route manually.

Instead, routers exchange routing information automatically using
routing protocols.

Examples include:

-   OSPF
-   BGP
-   RIP

Linux servers usually do **not** participate directly in these protocols
unless they are configured as routers.

For most Linux administrators and DevOps engineers, understanding
routing tables is more important than configuring dynamic routing
protocols.

------------------------------------------------------------------------

# Linux Perspective

The routing table is maintained by the Linux kernel.

Applications never choose routes.

Applications simply send data.

The kernel decides:

-   Which interface to use.
-   Whether to use a gateway.
-   Which route is the best match.

------------------------------------------------------------------------

# Production Example

A company introduces a new internal network:

``` text
10.50.0.0/16
```

Existing Linux servers cannot reach it.

Investigation shows:

``` bash
ip route
```

contains no route for:

``` text
10.50.0.0/16
```

Adding the correct static route immediately restores connectivity.

------------------------------------------------------------------------

# Common Mistakes

## Assuming Linux Always Uses the Default Gateway

The default gateway is used **only** when no more specific route exists.

------------------------------------------------------------------------

## Ignoring Prefix Length

A `/24` route is preferred over a `/16` route when both match.

------------------------------------------------------------------------

## Editing Applications Instead of Routes

Routing decisions belong to the Linux kernel.

Changing application configuration will not fix a missing route.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Route selection
-   Longest Prefix Match
-   Static routes
-   Dynamic routing overview
-   Kernel routing decisions

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the routing lesson with production troubleshooting,
interview questions, labs, challenge exercises, and a routing cheat
sheet.

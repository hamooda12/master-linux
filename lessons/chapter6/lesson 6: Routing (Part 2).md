# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 06 --- Routing (Part 2)

------------------------------------------------------------------------

# The Routing Table

Every Linux system maintains a **routing table**.

A routing table is a list of rules that tells the Linux kernel where
packets should be sent.

Whenever an application sends data, Linux consults this table before
transmitting the packet.

    Application
          |
    Linux Kernel
          |
    Routing Table
          |
    Selected Interface
          |
    Destination

------------------------------------------------------------------------

# Viewing the Routing Table

The standard command is:

``` bash
ip route
```

Example output:

``` text
default via 192.168.1.1 dev enp0s3

192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.20
```

------------------------------------------------------------------------

# Understanding the Output

## Default Route

``` text
default via 192.168.1.1 dev enp0s3
```

Meaning:

-   `default` → Used when no more specific route exists.
-   `via 192.168.1.1` → Send packets to the default gateway.
-   `dev enp0s3` → Use the `enp0s3` interface.

This is the route Linux uses to reach the Internet.

------------------------------------------------------------------------

## Local Network Route

``` text
192.168.1.0/24 dev enp0s3
```

Meaning:

-   The network is directly connected.
-   No router is required.
-   Linux sends packets directly through `enp0s3`.

------------------------------------------------------------------------

# How Linux Chooses a Route

Linux compares the destination IP address against the routing table.

If the destination belongs to a directly connected network, Linux sends
the packet directly.

Otherwise, it looks for another matching route.

If none exists, Linux uses the **default route**.

    Destination IP
          |
    Match Local Route?
          |
     Yes ----------> Send Directly
     |
     No
     |
    Use Default Gateway

------------------------------------------------------------------------

# Static Routes

Most small networks only need a default route.

Larger environments may also contain **static routes**.

Example:

``` text
10.20.0.0/16 via 192.168.1.254
```

Meaning:

To reach the `10.20.0.0/16` network, send packets to the router at
`192.168.1.254`.

------------------------------------------------------------------------

# Linux Perspective

Future commands include:

``` bash
ip route

ip route show
```

Both display the current routing table.

Administrators frequently run these commands during connectivity
investigations.

------------------------------------------------------------------------

# Production Example

A Linux server can reach devices on:

    192.168.1.0/24

but cannot reach:

    8.8.8.8

Running:

``` bash
ip route
```

reveals that no default route exists.

Without a default gateway, Linux has no idea where Internet-bound
packets should go.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   What a routing table is.
-   How to inspect it.
-   Default routes.
-   Directly connected routes.
-   Static routes.
-   How Linux selects routes.

------------------------------------------------------------------------

# ➡️ Next Part

We'll examine real routing problems, troubleshoot missing routes and
gateways, and finish the lesson with interview questions, labs, and a
routing cheat sheet.

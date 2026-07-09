# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 04 --- IP Addressing (Part 4)

------------------------------------------------------------------------

# IPv4 vs IPv6 in Production

Although IPv6 is the future of networking, both IPv4 and IPv6 are widely
used today.

Most organizations operate in a **dual-stack** environment, where
devices support both protocols.

Example:

    IPv4 : 192.168.1.25

    IPv6 : 2001:db8::25

Linux can assign both addresses to the same network interface.

------------------------------------------------------------------------

# How Linux Uses IP Addresses

Every network packet contains:

-   Source IP address
-   Destination IP address

Example:

    Laptop

    192.168.1.20

          |

          v

    Server

    192.168.1.50

The source identifies who sent the packet.

The destination identifies who should receive it.

Routers examine only the destination IP address to decide where packets
should travel.

------------------------------------------------------------------------

# Production Troubleshooting Scenarios

## Scenario 1 --- Duplicate IP Address

Two servers accidentally receive:

    192.168.1.50

Symptoms:

-   Random connectivity failures
-   Intermittent communication
-   Unstable applications

Root Cause:

Two devices cannot reliably share the same IP address.

------------------------------------------------------------------------

## Scenario 2 --- Wrong Subnet

Server A

    192.168.1.10/24

Server B

    192.168.2.20/24

Although both addresses look similar, they belong to different networks.

Without a router, communication fails.

------------------------------------------------------------------------

## Scenario 3 --- Wrong Default Gateway

The server has the correct IP address but an incorrect gateway.

Result:

-   Local communication succeeds.
-   Internet communication fails.
-   Remote servers cannot be reached.

------------------------------------------------------------------------

## Scenario 4 --- Private Address on the Internet

Suppose a public web server is configured with:

    192.168.1.25

Users on the Internet cannot reach it because this is a private address.

Public services require public IP addresses (or a reverse proxy/load
balancer using one).

------------------------------------------------------------------------

# Troubleshooting Methodology

When an IP-related issue occurs:

    Check IP address
            |
            v
    Check subnet
            |
            v
    Check default gateway
            |
            v
    Check routing
            |
            v
    Verify connectivity

Always verify configuration before changing anything.

------------------------------------------------------------------------

# Common Mistakes

## Confusing Public and Private Addresses

Private addresses cannot be reached directly from the Internet.

------------------------------------------------------------------------

## Ignoring the Subnet Prefix

Two servers may have similar addresses but belong to different networks
because of different prefixes.

------------------------------------------------------------------------

## Assuming an IP Address Is Enough

Communication also depends on:

-   Subnet
-   Gateway
-   Routing
-   Network connectivity

------------------------------------------------------------------------

# Best Practices

-   Assign unique IP addresses.
-   Document addressing plans.
-   Use CIDR notation consistently.
-   Verify network settings after configuration changes.
-   Troubleshoot logically rather than guessing.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is an IP address?
2.  What is the difference between IPv4 and IPv6?
3.  What is the purpose of a subnet mask?

## Intermediate

1.  Explain CIDR notation.
2.  Why can't devices use the network address?
3.  What is a broadcast address?

## Advanced

1.  Two servers cannot communicate. How would you determine whether they
    belong to the same subnet?
2.  Why is CIDR preferred over traditional subnet masks in modern
    systems?

------------------------------------------------------------------------

# Hands-on Lab

Answer the following:

1.  Identify whether each address is public or private.

```{=html}
<!-- -->
```
    10.1.5.20

    192.168.5.8

    8.8.8.8

    172.20.10.15

2.  For the following network:

```{=html}
<!-- -->
```
    192.168.1.0/24

Identify:

-   Network address
-   Broadcast address
-   First usable host
-   Last usable host

------------------------------------------------------------------------

# Challenge Exercise

A colleague configures a Linux server with:

    IP Address:

    192.168.10.25/24

The server cannot communicate with another server on:

    192.168.20.15/24

Explain why this happens and describe what networking component is
required for successful communication.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 04 you understand:

-   IPv4
-   IPv6
-   Public and private addressing
-   Subnet masks
-   CIDR notation
-   Network addresses
-   Broadcast addresses
-   Host addresses
-   IP troubleshooting fundamentals

These concepts form the foundation for every Linux networking task.

------------------------------------------------------------------------

# Cheat Sheet

  Concept             Example
  ------------------- -------------------------------------------
  IPv4                192.168.1.10
  IPv6                2001:db8::1
  Private Networks    10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
  CIDR                /8, /16, /24, /32
  Network Address     192.168.1.0
  Broadcast Address   192.168.1.255
  Host Range          192.168.1.1--192.168.1.254

------------------------------------------------------------------------

# What's Next

Lesson 05 --- Network Interfaces

You'll learn how Linux represents network interfaces, inspect them using
the `ip` command, understand MAC addresses, loopback interfaces,
interface states, and prepare for real network configuration and
troubleshooting.

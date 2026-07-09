# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 04 --- IP Addressing (Part 3)

------------------------------------------------------------------------

# CIDR Notation

## Definition

CIDR stands for **Classless Inter-Domain Routing**.

Instead of writing a subnet mask like:

    255.255.255.0

modern operating systems simply write:

    /24

CIDR is shorter, easier to read, and is the standard notation used by
Linux, cloud providers, Docker, and Kubernetes.

------------------------------------------------------------------------

# How CIDR Works

The number after the slash represents the number of **network bits**.

Example:

    192.168.1.25/24

means:

-   24 bits identify the network.
-   Remaining bits identify hosts.

------------------------------------------------------------------------

# Common CIDR Prefixes

  CIDR   Subnet Mask       Typical Size
  ------ ----------------- --------------------
  /8     255.0.0.0         Very large network
  /16    255.255.0.0       Medium network
  /24    255.255.255.0     Small LAN
  /32    255.255.255.255   Single host

You do **not** need to memorize every possible prefix.

The most common ones are:

-   /8
-   /16
-   /24
-   /32

------------------------------------------------------------------------

# Reading a CIDR Address

Example:

    10.0.5.20/8

Network:

    10.0.0.0

Host:

    0.5.20

------------------------------------------------------------------------

Another example:

    172.16.8.15/16

Network:

    172.16.0.0

Host:

    8.15

------------------------------------------------------------------------

Another example:

    192.168.1.50/24

Network:

    192.168.1.0

Host:

    50

------------------------------------------------------------------------

# CIDR in Linux

Linux displays addresses using CIDR notation.

Example:

``` text
192.168.1.25/24
```

instead of

``` text
192.168.1.25
255.255.255.0
```

This notation appears throughout Linux networking tools.

Future commands include:

``` bash
ip addr
ip route
hostname -I
```

------------------------------------------------------------------------

# CIDR in Cloud Computing

Cloud providers also use CIDR.

Examples:

-   AWS VPC
-   Azure Virtual Networks
-   Google Cloud VPC
-   Docker bridge networks
-   Kubernetes Pod networks

Typical examples:

    10.0.0.0/16

    192.168.0.0/24

Understanding CIDR is essential when working with cloud infrastructure.

------------------------------------------------------------------------

# Production Example

Suppose two servers have the following addresses:

    Server A

    192.168.10.20/24

    Server B

    192.168.10.45/24

Both belong to the same subnet.

Now change Server B to:

    192.168.20.45/24

The servers are now on different networks.

Traffic must pass through a router.

------------------------------------------------------------------------

# Common Mistakes

## Thinking /24 Is Part of the IP Address

Incorrect:

    192.168.1.10/24

The `/24` is **not** part of the IP address.

It describes the network prefix.

------------------------------------------------------------------------

## Confusing CIDR with Host Count

CIDR defines the network size.

It is not simply "the number of hosts."

------------------------------------------------------------------------

## Ignoring the Prefix

Two identical IP addresses with different prefixes may belong to
completely different networks.

Always consider both the IP address and the CIDR prefix.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   CIDR notation
-   Why Linux uses CIDR
-   Common prefixes
-   Reading network prefixes
-   CIDR in production and cloud environments

------------------------------------------------------------------------

# ➡️ Next Part

We'll combine everything you've learned about IP addressing, compare
IPv4 and IPv6 in production, and finish the lesson with troubleshooting
scenarios, interview questions, labs, and a cheat sheet.

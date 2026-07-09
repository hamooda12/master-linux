# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 05 --- Network Interfaces (Part 3)

------------------------------------------------------------------------

# Understanding `ip link` Output

One of the first commands a Linux administrator runs during network
troubleshooting is:

``` bash
ip link
```

Example output:

``` text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default
```

Let's understand every important field.

------------------------------------------------------------------------

# Interface Number

    1:
    2:

Linux assigns every network interface an index.

This number uniquely identifies the interface inside the kernel.

------------------------------------------------------------------------

# Interface Name

Examples:

    lo

    enp0s3

    wlp2s0

These names identify individual interfaces.

------------------------------------------------------------------------

# Interface Flags

Example:

    <UP,BROADCAST,RUNNING,MULTICAST>

Common flags include:

  Flag        Meaning
  ----------- ----------------------------------
  UP          Interface is enabled
  BROADCAST   Supports broadcast traffic
  MULTICAST   Supports multicast communication
  LOOPBACK    Internal loopback interface
  LOWER_UP    Physical connection detected

Flags describe the interface's capabilities and current condition.

------------------------------------------------------------------------

# MTU

Example:

    mtu 1500

MTU stands for **Maximum Transmission Unit**.

It defines the maximum amount of data that can be transmitted in a
single frame without fragmentation.

Typical values:

  MTU    Usage
  ------ -----------------------------
  1500   Standard Ethernet
  9000   Jumbo Frames (Data Centers)

------------------------------------------------------------------------

# State

Example:

    state UP

Possible values include:

-   UP
-   DOWN
-   UNKNOWN

Remember:

An interface may be **UP** while the application using it is still
unavailable.

------------------------------------------------------------------------

# Understanding `ip addr`

Another essential command is:

``` bash
ip addr
```

Example:

``` text
2: enp0s3

inet 192.168.1.25/24

inet6 fe80::5054:ff:fe12:3456/64
```

------------------------------------------------------------------------

# IPv4 Address

    inet 192.168.1.25/24

This displays:

-   IPv4 address
-   CIDR prefix

------------------------------------------------------------------------

# IPv6 Address

    inet6 fe80::5054:ff:fe12:3456/64

Linux often assigns IPv6 addresses automatically.

Many modern cloud environments use IPv6 extensively.

------------------------------------------------------------------------

# Multiple Addresses

A single interface may have:

-   IPv4 address
-   IPv6 address
-   Additional secondary addresses

This is completely normal.

------------------------------------------------------------------------

# Linux Perspective

Most Linux network troubleshooting begins with:

``` bash
ip link

ip addr
```

These commands answer questions such as:

-   Does the interface exist?
-   Is it enabled?
-   Does it have an IP address?
-   Which addresses are assigned?

------------------------------------------------------------------------

# Production Example

Users cannot reach a web server.

Investigation begins:

``` bash
ip link
```

Result:

    state DOWN

Root Cause:

The network interface is disabled.

The web application itself may be functioning perfectly, but no packets
can reach it.

------------------------------------------------------------------------

# Common Mistakes

## Confusing UP with Internet Access

An interface being UP only means Linux has enabled it.

It does **not** guarantee:

-   Internet access
-   DNS resolution
-   Routing
-   Application availability

------------------------------------------------------------------------

## Ignoring IPv6

Even if you mainly use IPv4, Linux often configures IPv6 automatically.

Production environments frequently support both.

------------------------------------------------------------------------

## Looking Only at the IP Address

Always inspect:

-   Interface state
-   Assigned addresses
-   Interface name

Together they provide a more complete picture.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   How to interpret `ip link`
-   How to interpret `ip addr`
-   Interface flags
-   MTU
-   Interface states
-   IPv4 and IPv6 addresses

------------------------------------------------------------------------

# ➡️ Next Part

We'll finish the lesson with production troubleshooting, interview
questions, hands-on labs, challenge exercises, and a complete cheat
sheet for Linux network interfaces.

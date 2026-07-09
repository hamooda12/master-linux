# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 06 --- Routing (Part 1)

> **Goal:** Understand how Linux decides where to send packets and why
> routing is essential for communication beyond the local network.

------------------------------------------------------------------------

# 📚 Overview

Every packet sent by a Linux system must have a destination.

If the destination is on the same local network, Linux can send the
packet directly.

If the destination is on another network, Linux must know **where to
forward the packet**.

This decision-making process is called **routing**.

Without routing, the Internet would not exist.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain what routing is.
-   Understand why routing is necessary.
-   Explain the role of routers.
-   Understand the default gateway.
-   Read basic routing concepts used by Linux.

------------------------------------------------------------------------

# 🌍 What Is Routing?

Routing is the process of selecting the path that packets take from a
source to a destination.

    Laptop
       |
    Router
       |
    Internet
       |
    Cloud Server

The router decides where packets should go next.

------------------------------------------------------------------------

# Why Is Routing Needed?

Imagine two different office networks.

    Office A

    192.168.1.0/24


    Office B

    192.168.2.0/24

A computer in Office A cannot directly communicate with a computer in
Office B.

A router is required to forward traffic between the two networks.

------------------------------------------------------------------------

# Router vs Switch

Many beginners confuse these devices.

## Switch

-   Connects devices inside one LAN.
-   Uses MAC addresses.
-   Operates mainly at Layer 2.

## Router

-   Connects different networks.
-   Uses IP addresses.
-   Operates mainly at Layer 3.

```{=html}
<!-- -->
```
    PC ----\
             \
    PC ---- Switch ---- Router ---- Internet
             /
    Server --/

------------------------------------------------------------------------

# Default Gateway

The **default gateway** is the router a Linux system uses when the
destination is outside its local network.

Example:

    Computer

    192.168.1.20

    Gateway

    192.168.1.1

    Destination

    8.8.8.8

Linux recognizes that `8.8.8.8` is not on the local network.

Instead of sending packets directly, it forwards them to the default
gateway.

------------------------------------------------------------------------

# Real-World Analogy

Imagine sending a letter.

If the recipient lives in your apartment building, you can deliver it
yourself.

If they live in another city, you give the letter to the postal service.

The postal service is like the default gateway.

It knows how to forward your message toward its destination.

------------------------------------------------------------------------

# Linux Perspective

Linux maintains a **routing table**.

The routing table tells the kernel where packets should be sent.

Later you'll inspect it using:

``` bash
ip route
```

Every packet leaving the system is compared against this table.

------------------------------------------------------------------------

# Production Example

A web server can communicate with machines inside its local network but
cannot reach the Internet.

Possible causes include:

-   Missing default gateway.
-   Incorrect routing table.
-   Router failure.

These are routing problems, not application problems.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   What routing is.
-   Why routing exists.
-   Router vs switch.
-   Default gateway.
-   Why Linux maintains a routing table.

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn how Linux stores routes, inspect routing tables using
`ip route`, and understand route selection.

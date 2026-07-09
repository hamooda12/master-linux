# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 02 --- The OSI Model (Part 4)

------------------------------------------------------------------------

# Mapping the OSI Model to Linux

The OSI model is not just a theory taught in networking classes.

Linux administrators use it as a troubleshooting framework.

When something fails, ask:

**Which OSI layer is most likely responsible?**

------------------------------------------------------------------------

## Layer 1 --- Physical

Typical issues:

-   Ethernet cable unplugged
-   Wi-Fi disabled
-   Faulty network card
-   Switch power failure

Future Linux commands:

``` bash
ip link
```

Question:

**Is the machine physically connected?**

------------------------------------------------------------------------

## Layer 2 --- Data Link

Typical issues:

-   Incorrect MAC communication
-   Switch problems
-   Local network communication failure

Future Linux commands:

``` bash
ip link
```

Question:

**Can devices communicate inside the same LAN?**

------------------------------------------------------------------------

## Layer 3 --- Network

Typical issues:

-   Wrong IP address
-   Wrong subnet
-   Wrong default gateway
-   Routing failures

Future Linux commands:

``` bash
ip addr
ip route
ping
```

Question:

**Can packets reach the destination network?**

------------------------------------------------------------------------

## Layer 4 --- Transport

Typical issues:

-   Closed TCP port
-   Firewall blocking traffic
-   Service not listening

Future Linux commands:

``` bash
ss
netstat
lsof -i
```

Question:

**Is the application listening on the expected port?**

------------------------------------------------------------------------

## Layer 5 --- Session

Typical issues:

-   Lost connection
-   Interrupted communication
-   Session timeout

Applications usually manage these problems automatically.

------------------------------------------------------------------------

## Layer 6 --- Presentation

Typical issues:

-   TLS certificate problems
-   Unsupported encoding
-   Incorrect data format

Examples:

-   HTTPS certificate errors
-   Invalid JSON
-   Character encoding issues

------------------------------------------------------------------------

## Layer 7 --- Application

Typical issues:

-   Spring Boot crashed
-   Nginx configuration error
-   REST API bug
-   HTTP 500 error

Future Linux commands:

``` bash
curl
wget
```

Question:

**Is the application functioning correctly?**

------------------------------------------------------------------------

# Production Troubleshooting Example

A user reports:

> "The website isn't working."

Think in layers.

    Layer 1
    ↓

    Cable connected?

    ↓

    Layer 2

    Local communication?

    ↓

    Layer 3

    Correct IP?

    ↓

    Layer 4

    Port listening?

    ↓

    Layer 7

    Application healthy?

This structured approach prevents random guessing.

------------------------------------------------------------------------

# Common Mistakes

## Memorizing Without Understanding

The OSI model is a thinking tool, not a memorization exercise.

------------------------------------------------------------------------

## Jumping Directly to the Application

Many engineers restart services immediately.

Experienced administrators first determine **which layer is failing**.

------------------------------------------------------------------------

## Assuming Ping Proves Everything

Ping only verifies certain aspects of connectivity.

An application can still fail even when ping succeeds.

------------------------------------------------------------------------

# Best Practices

-   Troubleshoot from lower layers upward.
-   Gather evidence before making changes.
-   Verify every fix.
-   Avoid restarting services unnecessarily.
-   Document findings during production incidents.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  Why does the OSI model exist?
2.  Which layer uses IP addresses?
3.  Which layer uses MAC addresses?

## Intermediate

1.  Explain encapsulation.
2.  Which layer handles ports?
3.  Which layer is responsible for routing?

## Advanced

1.  A website returns HTTP 500. Which OSI layer is failing?
2.  Users cannot reach a server because of a wrong gateway. Which layer
    is affected?

------------------------------------------------------------------------

# Hands-on Lab

For each issue below, identify the OSI layer involved.

-   Broken Ethernet cable
-   Wrong IP address
-   Closed SSH port
-   Invalid TLS certificate
-   Spring Boot application crash

------------------------------------------------------------------------

# Challenge Exercise

A user cannot access your application.

Write a troubleshooting plan that investigates the problem layer by
layer instead of guessing.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 02 you understand:

-   All seven OSI layers
-   The responsibility of each layer
-   Encapsulation
-   Linux examples for every layer
-   How the OSI model supports production troubleshooting

The OSI model is now your mental framework for analyzing networking
problems.

------------------------------------------------------------------------

# Cheat Sheet

    Layer Responsibility   Example
  ------- ---------------- -----------------------
        7 Application      HTTP, SSH
        6 Presentation     TLS, UTF-8
        5 Session          Connection management
        4 Transport        TCP, UDP, Ports
        3 Network          IP, Routing
        2 Data Link        MAC, Frames
        1 Physical         Cable, Wi-Fi

------------------------------------------------------------------------

# What's Next

Lesson 03 --- TCP/IP Model

You'll learn the practical networking model used by Linux, the Internet,
cloud platforms, and modern applications, and compare it directly with
the OSI model.

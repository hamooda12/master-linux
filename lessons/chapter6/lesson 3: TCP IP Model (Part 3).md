# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 03 --- TCP/IP Model (Part 3)

------------------------------------------------------------------------

# Layer 3 --- Transport Layer

## Definition

The Transport Layer is responsible for communication between
applications running on different computers.

The Internet Layer successfully delivers data to the destination
computer.

The Transport Layer ensures the data reaches the correct application.

    Client
       |
    Transport Layer
       |
    Spring Boot Application

------------------------------------------------------------------------

# Responsibilities

-   End-to-end communication
-   Port numbers
-   Reliable delivery
-   Error detection
-   Flow control
-   Data segmentation

------------------------------------------------------------------------

# TCP and UDP

The two primary Transport Layer protocols are:

-   TCP (Transmission Control Protocol)
-   UDP (User Datagram Protocol)

Both transport application data, but they work differently.

------------------------------------------------------------------------

# TCP

## Definition

TCP is a **connection-oriented** protocol.

Before data is exchanged, TCP establishes a connection between the
client and server.

    Client
       |
    Connection Established
       |
    Server

------------------------------------------------------------------------

## Characteristics

-   Reliable
-   Ordered delivery
-   Error checking
-   Lost packet recovery
-   Slower than UDP

TCP guarantees that data arrives correctly.

------------------------------------------------------------------------

## Common TCP Applications

-   HTTP
-   HTTPS
-   SSH
-   FTP
-   Email protocols

Most business applications use TCP because reliability is more important
than speed.

------------------------------------------------------------------------

# UDP

## Definition

UDP is a **connectionless** protocol.

It sends data without establishing a connection first.

    Client
       |
    Send Data
       |
    Server

No confirmation is required.

------------------------------------------------------------------------

## Characteristics

-   Faster
-   Lower overhead
-   No guaranteed delivery
-   No retransmission
-   Packets may arrive out of order

------------------------------------------------------------------------

## Common UDP Applications

-   DNS
-   Voice over IP (VoIP)
-   Video streaming
-   Online gaming
-   Live broadcasts

These applications value speed over perfect reliability.

------------------------------------------------------------------------

# TCP vs UDP

  TCP                   UDP
  --------------------- ----------------------
  Reliable              Best effort
  Connection-oriented   Connectionless
  Error recovery        No recovery
  Ordered delivery      Order not guaranteed
  Slower                Faster

------------------------------------------------------------------------

# Linux Perspective

Many Linux services listen using either TCP or UDP.

Examples include:

-   SSH → TCP
-   HTTP → TCP
-   HTTPS → TCP
-   DNS → UDP (and sometimes TCP)

Later, you'll inspect these services with:

``` bash
ss
```

------------------------------------------------------------------------

# Production Example

A user opens a website.

The browser communicates with the web server using TCP because every
byte of the webpage must arrive correctly.

However, when the browser performs a DNS lookup, UDP is usually used
because the request is very small and speed is important.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Transport Layer
-   TCP
-   UDP
-   Differences between TCP and UDP
-   Why different applications choose different transport protocols

------------------------------------------------------------------------

# Next Part

We'll study the Application Layer, compare TCP/IP with the OSI model,
and see how Linux uses the complete networking stack in production.

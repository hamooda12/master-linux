# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 02 --- The OSI Model (Part 2)

------------------------------------------------------------------------

# Layer 1 --- Physical Layer

## Definition

The Physical Layer is responsible for transmitting raw bits from one
device to another.

It deals with the actual physical medium used for communication.

Examples include:

-   Ethernet cables
-   Fiber optic cables
-   Wireless radio signals (Wi-Fi)
-   Network connectors
-   Electrical signals
-   Light pulses

```{=html}
<!-- -->
```
    Computer ===== Ethernet Cable ===== Switch

### Responsibilities

-   Sending bits
-   Receiving bits
-   Electrical signaling
-   Physical connections

### Common Problems

-   Broken cable
-   Loose connector
-   Damaged network card
-   Wi-Fi disabled
-   Hardware failure

------------------------------------------------------------------------

# Linux Perspective

Linux cannot repair a damaged cable.

However, Linux can detect whether an interface is physically connected.

Later you will learn commands such as:

``` bash
ip link
```

------------------------------------------------------------------------

# Layer 2 --- Data Link Layer

## Definition

The Data Link Layer allows devices on the **same local network (LAN)**
to communicate.

Instead of using IP addresses, it uses **MAC addresses**.

    Laptop
       |
    Switch
       |
    Server

Each network card has a unique MAC address.

Example:

    08:00:27:3A:91:BC

### Responsibilities

-   Local communication
-   MAC addressing
-   Frame creation
-   Error detection

------------------------------------------------------------------------

# Frame

The Data Link layer wraps packets into frames before transmission.

    +-----------------------+
    | Frame Header          |
    | Packet                |
    | Frame Trailer         |
    +-----------------------+

Frames exist only within a local network.

Routers remove old frames and create new ones when forwarding traffic.

------------------------------------------------------------------------

# Switches and Layer 2

Switches operate primarily at Layer 2.

They forward frames based on MAC addresses instead of IP addresses.

This allows efficient communication inside a LAN.

------------------------------------------------------------------------

# Layer 3 --- Network Layer

## Definition

The Network Layer is responsible for moving data between different
networks.

Unlike Layer 2, it uses **IP addresses**.

    Home LAN
        |
     Router
        |
    Internet
        |
    Cloud Server

### Responsibilities

-   Logical addressing
-   Routing
-   Packet forwarding
-   Path selection

------------------------------------------------------------------------

# IP Address

Every device communicating at Layer 3 requires an IP address.

Example:

    192.168.1.10

The router examines destination IP addresses to determine where packets
should be sent.

------------------------------------------------------------------------

# Router and Layer 3

Routers operate primarily at Layer 3.

They connect different networks together.

Without routers, communication beyond the local network would not be
possible.

------------------------------------------------------------------------

# Linux Perspective

Linux uses the Network Layer continuously.

Every server has:

-   IP addresses
-   Routing tables
-   Default gateways

Future lessons will introduce:

``` bash
ip addr
ip route
```

------------------------------------------------------------------------

# Production Example

Suppose users cannot reach your server.

Possible investigations:

-   Is the network cable connected? (Layer 1)
-   Is communication working inside the LAN? (Layer 2)
-   Does the server have the correct IP address? (Layer 3)
-   Is the router forwarding traffic? (Layer 3)

Thinking in layers helps isolate the problem quickly.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Physical Layer
-   Data Link Layer
-   Network Layer
-   MAC addresses
-   Frames
-   IP addresses
-   Router vs Switch responsibilities

------------------------------------------------------------------------

# Next Part

We'll study:

-   Transport Layer
-   Session Layer
-   Presentation Layer
-   Application Layer

These are the layers that applications interact with most frequently.

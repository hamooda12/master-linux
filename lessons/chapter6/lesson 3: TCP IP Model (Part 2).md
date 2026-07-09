# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 03 --- TCP/IP Model (Part 2)

------------------------------------------------------------------------

# Layer 1 --- Network Access Layer

## Definition

The Network Access Layer is responsible for transmitting data across the
physical network.

It combines the responsibilities of the Physical and Data Link layers
from the OSI model.

    Computer
        |
    Ethernet / Wi-Fi
        |
    Switch

This layer determines how data is actually placed onto the network
medium.

------------------------------------------------------------------------

## Responsibilities

-   Physical transmission
-   MAC addressing
-   Frame creation
-   Error detection
-   Communication inside a LAN

------------------------------------------------------------------------

## Common Technologies

-   Ethernet
-   Wi-Fi
-   Fiber
-   MAC Addresses
-   Network Interface Cards (NICs)

------------------------------------------------------------------------

## Linux Perspective

Linux network interfaces belong to this layer.

Examples you'll use later:

``` bash
ip link
ip addr
```

------------------------------------------------------------------------

# Layer 2 --- Internet Layer

## Definition

The Internet Layer is responsible for delivering packets between
different networks.

Unlike the Network Access Layer, it focuses on **logical addressing**
rather than physical hardware.

    Home Network
          |
       Router
          |
    Internet
          |
    Cloud Server

------------------------------------------------------------------------

## Responsibilities

-   IP addressing
-   Routing
-   Packet forwarding
-   Selecting paths between networks

------------------------------------------------------------------------

## Protocol

The most important protocol here is:

-   IP (Internet Protocol)

Versions include:

-   IPv4
-   IPv6

------------------------------------------------------------------------

## Linux Perspective

Linux maintains:

-   IP addresses
-   Routing tables
-   Default gateways

Commands you'll learn later:

``` bash
ip addr
ip route
ping
```

------------------------------------------------------------------------

# Packet Flow

Imagine a browser contacting a web server.

    Browser
        |
    Internet Layer
        |
    Router
        |
    Internet
        |
    Server

Every router examines the destination IP address and forwards the packet
toward its destination.

------------------------------------------------------------------------

# Production Example

A server cannot reach a database hosted in another network.

Possible causes:

-   Wrong IP address
-   Missing route
-   Incorrect gateway
-   Router failure

Notice that all of these belong primarily to the Internet Layer.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Network Access Layer
-   Internet Layer
-   Their responsibilities
-   Their relationship to Linux networking

------------------------------------------------------------------------

# Next Part

We'll study:

-   Transport Layer
-   Application Layer
-   TCP
-   UDP
-   End-to-end communication

# Chapter 06 --- Lesson 01

# Networking Fundamentals

> **Objective:** Understand what a network is, why it exists, and the
> core concepts every DevOps engineer must master before learning Linux
> networking commands.

------------------------------------------------------------------------

# Why Should a DevOps Engineer Learn Networking?

Almost every production problem eventually becomes a networking problem.

Examples:

-   A web application is running, but users cannot access it.
-   A database refuses connections.
-   A Kubernetes Pod cannot reach another service.
-   DNS resolves the wrong IP.
-   A firewall blocks traffic.
-   A load balancer forwards requests to the wrong backend.

To solve these problems, you need networking fundamentals---not just
Linux commands.

------------------------------------------------------------------------

# What Is a Network?

A network is a group of devices that communicate and exchange data.

Examples of devices:

-   Laptops
-   Servers
-   Phones
-   Routers
-   Switches
-   Printers

Instead of copying files using a USB drive, devices send data across the
network.

------------------------------------------------------------------------

# Why Do Networks Exist?

Without networking:

-   Websites wouldn't exist.
-   Cloud computing wouldn't exist.
-   SSH wouldn't work.
-   APIs couldn't communicate.
-   Databases couldn't serve applications.

Networking allows computers to share information regardless of physical
distance.

------------------------------------------------------------------------

# A Real Production Example

Imagine a user opens:

    https://example.com

Behind the scenes:

1.  The browser discovers the server's IP using DNS.
2.  A connection is established.
3.  Packets travel through many routers.
4.  The server processes the request.
5.  The response returns to the browser.

All of this usually happens in well under one second.

------------------------------------------------------------------------

# Client and Server

## Client

A client requests a service.

Examples:

-   Web browser
-   Mobile app
-   curl
-   React frontend

## Server

A server provides a service.

Examples:

-   Nginx
-   Apache
-   Spring Boot
-   MySQL
-   PostgreSQL

Example:

    Browser  ------------>  Web Server
     Request                Response

A single machine can act as both a client and a server.

------------------------------------------------------------------------

# What Is a Packet?

Computers do not send large files as one huge block.

Instead, data is divided into small pieces called **packets**.

Example:

A 100 MB file becomes thousands of packets.

Advantages:

-   Reliable transmission
-   Easier error recovery
-   Efficient routing
-   Multiple conversations can share the same network

Think of packets like individual parcels shipped through a delivery
company.

------------------------------------------------------------------------

# Network Interface

A network interface is the connection between your computer and the
network.

Examples:

-   Ethernet
-   Wi-Fi
-   Virtual interface (Docker, VPN)

Linux usually names interfaces like:

    eth0
    ens33
    enp0s3
    wlan0

------------------------------------------------------------------------

# MAC Address

A MAC address uniquely identifies a network interface on a local
network.

Example:

    00:1A:2B:3C:4D:5E

Characteristics:

-   Layer 2 identifier
-   Usually assigned by the manufacturer
-   Used only inside the local network

------------------------------------------------------------------------

# IP Address

An IP address identifies a device on an IP network.

Example:

    192.168.1.20

Unlike a MAC address, an IP address allows communication beyond the
local network.

------------------------------------------------------------------------

# IPv4 vs IPv6

IPv4 example:

    192.168.1.20

IPv6 example:

    2001:db8::1

IPv6 provides a vastly larger address space than IPv4.

------------------------------------------------------------------------

# Public vs Private IP

Private IPs are used inside internal networks.

Common private ranges:

    10.0.0.0/8
    172.16.0.0/12
    192.168.0.0/16

Public IPs are globally reachable on the Internet.

------------------------------------------------------------------------

# Loopback Address

    127.0.0.1

Loopback always points to your own machine.

When an application connects to 127.0.0.1, no traffic leaves the
computer.

This is heavily used for testing and local development.

------------------------------------------------------------------------

# Hostname

A hostname is the human-readable name of a computer.

Example:

    web-server-01

Hostnames make systems easier to identify than remembering IP addresses.

------------------------------------------------------------------------

# Ports

Many services can run on one computer because each service listens on a
different port.

Examples:

  Service        Port
  ------------ ------
  SSH              22
  HTTP             80
  HTTPS           443
  MySQL          3306
  PostgreSQL     5432

Think of an IP address as a building and a port as an apartment number.

------------------------------------------------------------------------

# TCP vs UDP

## TCP

-   Reliable
-   Ordered delivery
-   Error checking
-   Used for HTTP, SSH, databases

## UDP

-   Faster
-   No guaranteed delivery
-   Used for streaming, VoIP, DNS queries

------------------------------------------------------------------------

# OSI Model (Practical View)

Remember this model as a troubleshooting framework.

1.  Physical
2.  Data Link
3.  Network
4.  Transport
5.  Session
6.  Presentation
7.  Application

For Linux administrators, Layers 2--7 are encountered most often.

------------------------------------------------------------------------

# Key Takeaways

-   Networks connect devices.
-   Clients request; servers provide.
-   Data travels as packets.
-   Every interface has a MAC address.
-   Every networked device has an IP address.
-   Ports identify services.
-   TCP prioritizes reliability; UDP prioritizes speed.
-   Understanding these concepts is essential before learning Linux
    networking commands.

------------------------------------------------------------------------

# Next Lesson

**Lesson 02 --- Inspecting Network Configuration with Linux (`ip`,
`hostname`, and routing).**

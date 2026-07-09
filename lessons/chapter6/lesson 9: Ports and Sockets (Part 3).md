# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 09 --- Ports and Sockets (Part 3)

------------------------------------------------------------------------

# Established Connections

A listening socket waits for incoming connections.

Once a client connects, Linux creates an **established connection**.

    Browser
        |
    TCP Connection
        |
    Spring Boot

The listening socket remains available for new clients while each
connected client receives its own established connection.

------------------------------------------------------------------------

# Viewing Established Connections

Display established TCP connections:

``` bash
ss -t
```

Or display all TCP sockets with process information:

``` bash
ss -tp
```

Example:

``` text
State  Recv-Q Send-Q Local Address:Port   Peer Address:Port
ESTAB  0      0      192.168.1.20:8080    192.168.1.50:52344
```

------------------------------------------------------------------------

# Local Address

The **Local Address** identifies the interface and port on your Linux
machine.

Example:

``` text
192.168.1.20:8080
```

Meaning:

-   Local IP: `192.168.1.20`
-   Local Port: `8080`

This is where your application is listening or communicating.

------------------------------------------------------------------------

# Peer Address

The **Peer Address** represents the remote system.

Example:

``` text
192.168.1.50:52344
```

Meaning:

-   Client IP: `192.168.1.50`
-   Client Port: `52344`

This identifies the remote endpoint of the connection.

------------------------------------------------------------------------

# Ephemeral Ports

When a client initiates a connection, Linux automatically chooses a
temporary source port.

These are called **ephemeral ports**.

Example:

``` text
Client

192.168.1.50:52344

        │

        ▼

Server

192.168.1.20:8080
```

The client did not manually choose `52344`.

Linux selected it automatically.

------------------------------------------------------------------------

# Why Are Ephemeral Ports Needed?

Imagine a browser opening three tabs to the same website.

    Browser

    52344

    52345

    52346

            │

            ▼

    Server

    Port 443

Each connection needs its own unique client port so Linux can
distinguish between them.

------------------------------------------------------------------------

# Socket States

Common TCP states include:

  State        Meaning
  ------------ ----------------------------------
  LISTEN       Waiting for incoming connections
  ESTAB        Active connection established
  TIME-WAIT    Connection recently closed
  CLOSE-WAIT   Waiting for application to close

These states help administrators understand the lifecycle of network
connections.

------------------------------------------------------------------------

# Linux Perspective

Running:

``` bash
ss -tan
```

shows both listening and established TCP sockets.

During troubleshooting this helps answer:

-   Is anyone connected?
-   Which clients are connected?
-   Is the application accepting requests?

------------------------------------------------------------------------

# Production Example

Users report that the application is slow.

Running:

``` bash
ss -tan
```

reveals thousands of established connections.

The service is reachable, but it may be overloaded by client traffic.

This information guides further investigation.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Established connections
-   Local addresses
-   Peer addresses
-   Ephemeral ports
-   Common TCP socket states
-   Interpreting active socket information

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the lesson with production troubleshooting scenarios,
interview questions, hands-on labs, challenge exercises, and a complete
ports and sockets cheat sheet.

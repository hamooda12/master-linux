# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 09 --- Ports and Sockets (Part 2)

------------------------------------------------------------------------

# Inspecting Sockets with `ss`

Modern Linux systems use the `ss` command to inspect sockets.

``` bash
ss
```

To display only listening TCP sockets:

``` bash
ss -tl
```

To display listening TCP sockets with process information:

``` bash
ss -tlp
```

To display both TCP and UDP listening sockets:

``` bash
ss -tulpn
```

This is one of the most common networking commands used by Linux
administrators.

------------------------------------------------------------------------

# Understanding `ss -tulpn`

Example:

``` text
Netid State  Local Address:Port  Peer Address:Port Process
tcp   LISTEN 0.0.0.0:22      0.0.0.0:*        users:(("sshd",pid=821))
tcp   LISTEN 0.0.0.0:8080    0.0.0.0:*        users:(("java",pid=2401))
udp   UNCONN 0.0.0.0:53      0.0.0.0:*        users:(("systemd-resolved",pid=611))
```

------------------------------------------------------------------------

# Output Analysis

## Netid

Protocol in use.

Examples:

-   tcp
-   udp

------------------------------------------------------------------------

## State

Typical values:

-   LISTEN
-   ESTAB
-   UNCONN

`LISTEN` means the application is waiting for incoming connections.

------------------------------------------------------------------------

## Local Address

Example:

``` text
0.0.0.0:8080
```

This is the address and port on which the application is listening.

------------------------------------------------------------------------

## Peer Address

Example:

``` text
0.0.0.0:*
```

For listening sockets there is no connected remote client yet.

------------------------------------------------------------------------

## Process

Example:

``` text
java
```

Shows which process owns the socket.

This is extremely useful during troubleshooting.

------------------------------------------------------------------------

# TCP vs UDP Sockets

TCP sockets:

-   Reliable
-   Connection-oriented
-   Used by HTTP, HTTPS, SSH

UDP sockets:

-   Connectionless
-   Lower overhead
-   Used by DNS and streaming applications

------------------------------------------------------------------------

# The `netstat` Command

Older Linux systems commonly use:

``` bash
netstat -tulpn
```

Although still encountered, modern Linux distributions recommend using
`ss` because it is faster and obtains information directly from the
kernel.

------------------------------------------------------------------------

# The `lsof -i` Command

`lsof` lists open files.

Since Linux treats sockets as files, it can also display network
connections.

``` bash
lsof -i
```

Example:

``` text
COMMAND PID USER FD TYPE DEVICE NODE NAME
java    2401 app  45u IPv4       TCP *:8080 (LISTEN)
```

Useful for identifying which application owns a particular port.

------------------------------------------------------------------------

# Linux Perspective

A common investigation sequence is:

``` bash
ss -tulpn
```

If a port is missing, verify:

-   Is the process running?
-   Is the service configured correctly?
-   Did the application fail to start?

------------------------------------------------------------------------

# Production Example

Users report they cannot access:

``` text
http://server:8080
```

Running:

``` bash
ss -tulpn
```

shows no service listening on port 8080.

The problem is not routing or DNS.

The application never opened the required socket.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   `ss`
-   `netstat`
-   `lsof -i`
-   Listening sockets
-   TCP and UDP sockets
-   Reading socket information

------------------------------------------------------------------------

# ➡️ Next Part

We'll study established connections, local and remote addresses,
ephemeral ports, and advanced socket analysis.

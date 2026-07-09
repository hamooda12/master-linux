# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 01 --- Networking Fundamentals (Part 2)

------------------------------------------------------------------------

# 🌐 LAN (Local Area Network)

## Definition

A LAN is a network that connects devices within a limited geographic
area.

Examples:

-   Home network
-   Office network
-   School
-   University
-   Data center rack

```{=html}
<!-- -->
```
    Laptop ---- Switch ---- Server
          \         |
           \---- Printer

### Characteristics

-   High speed
-   Low latency
-   Privately managed
-   Usually Ethernet or Wi-Fi

### DevOps Example

Your Spring Boot application, database, and reverse proxy often
communicate over a LAN.

------------------------------------------------------------------------

# 🌎 WAN (Wide Area Network)

## Definition

A WAN connects multiple LANs over large distances.

    Office A -------- Internet -------- Office B

Examples:

-   Company branches
-   Cloud regions
-   MPLS networks
-   The Internet

### Characteristics

-   Larger geographic coverage
-   Higher latency than LAN
-   Managed by ISPs or cloud providers

------------------------------------------------------------------------

# 🌍 Internet

The Internet is a global collection of interconnected networks.

It is **not** a single network.

    LAN --> ISP --> Internet --> ISP --> LAN

Every website, cloud platform, and online service depends on this
interconnected structure.

------------------------------------------------------------------------

# 👤 Client

A client requests a service from another computer.

Examples:

-   Web browser
-   curl
-   Mobile app
-   React frontend

```{=html}
<!-- -->
```
    Client ---- Request ---->

------------------------------------------------------------------------

# 🖥 Server

A server provides services to clients.

Examples:

-   Spring Boot API
-   Nginx
-   Database
-   SSH server

```{=html}
<!-- -->
```
    <---- Response ---- Server

A single server can handle many clients simultaneously.

------------------------------------------------------------------------

# 🤝 Client--Server Model

    Browser
       |
    HTTP Request
       |
    Spring Boot
       |
    Database

The client starts the conversation.

The server waits for requests and responds.

------------------------------------------------------------------------

# 🔄 Request--Response Cycle

1.  Client sends a request.
2.  Network transports the request.
3.  Server processes it.
4.  Server sends a response.
5.  Client displays the result.

Almost every modern application follows this model.

------------------------------------------------------------------------

# 🐧 Linux Perspective

Linux can act as:

-   A client
-   A server
-   Both simultaneously

Example:

-   Your Linux VM uses `apt` as a client to download packages.
-   The same VM may run Nginx as a web server.

------------------------------------------------------------------------

# 🏗 Production Example

A user opens your website.

    Browser
       |
    Internet
       |
    Nginx
       |
    Spring Boot
       |
    MySQL

If any communication fails, users experience an outage even though other
components may still be healthy.

------------------------------------------------------------------------

# 📌 Lesson Summary

You learned:

-   LAN
-   WAN
-   Internet
-   Client
-   Server
-   Client--Server architecture
-   Request--Response communication

These concepts form the vocabulary used throughout networking and
DevOps.

------------------------------------------------------------------------

# ➡️ Next Part

Router, Switch, Gateway, Packet, Frame, Bandwidth, and Latency.

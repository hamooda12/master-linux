# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 02 --- The OSI Model (Part 3)

------------------------------------------------------------------------

# Layer 4 --- Transport Layer

## Definition

The Transport Layer is responsible for delivering data between
applications running on different devices.

Unlike the Network Layer, which gets data to the correct computer, the
Transport Layer gets data to the correct application.

    Client
       |
    Transport Layer
       |
    Server Application

### Responsibilities

-   End-to-end communication
-   Reliable delivery
-   Error recovery
-   Data segmentation
-   Flow control

------------------------------------------------------------------------

# Ports

Applications share one network connection.

Ports allow Linux to determine which application should receive incoming
data.

Examples:

  Service              Default Port
  ------------------ --------------
  SSH                            22
  HTTP                           80
  HTTPS                         443
  Spring Boot                  8080
  React Dev Server             5173

Later you will inspect ports using:

``` bash
ss
```

------------------------------------------------------------------------

# Layer 5 --- Session Layer

## Definition

The Session Layer establishes, manages, and terminates communication
sessions between applications.

Think of it as the manager of a conversation.

Responsibilities include:

-   Starting sessions
-   Maintaining sessions
-   Ending sessions
-   Recovering interrupted sessions

------------------------------------------------------------------------

# Real-World Analogy

Imagine a phone call.

1.  Dial the number.
2.  Talk.
3.  Hang up.

The Session Layer manages the beginning, continuation, and end of the
conversation.

------------------------------------------------------------------------

# Layer 6 --- Presentation Layer

## Definition

The Presentation Layer ensures both systems understand the exchanged
data.

It handles:

-   Data formatting
-   Character encoding
-   Compression
-   Encryption

Examples include:

-   UTF-8 text
-   JSON
-   XML
-   TLS encryption

Without this layer, two systems might exchange data in incompatible
formats.

------------------------------------------------------------------------

# Layer 7 --- Application Layer

## Definition

The Application Layer is the closest layer to the user.

Applications interact with the network through this layer.

Examples:

-   Web browsers
-   SSH clients
-   Email clients
-   curl
-   wget

Protocols commonly associated with this layer include:

-   HTTP
-   HTTPS
-   SSH
-   FTP
-   DNS
-   SMTP

------------------------------------------------------------------------

# Linux Perspective

Many Linux networking tools operate at or near the Application Layer.

Examples:

``` bash
curl
wget
dig
host
```

These tools allow administrators to communicate with remote services.

------------------------------------------------------------------------

# Production Example

A user opens:

    https://example.com

The browser communicates with the web server using the Application
Layer.

If the application returns:

    HTTP 500 Internal Server Error

The network is functioning.

The application itself has encountered an error.

Understanding the OSI layers helps identify where the failure occurs.

------------------------------------------------------------------------

# Putting the Layers Together

    7. Application
    6. Presentation
    5. Session
    4. Transport
    3. Network
    2. Data Link
    1. Physical

Each layer performs one responsibility.

Each layer depends on the layers below it.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Transport Layer
-   Session Layer
-   Presentation Layer
-   Application Layer
-   Ports
-   Common application protocols
-   How Linux applications use the OSI model

------------------------------------------------------------------------

# Next Part

We'll connect every OSI layer to real Linux commands and DevOps
troubleshooting scenarios.

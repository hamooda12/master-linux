# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 01 --- Networking Fundamentals (Part 3)

------------------------------------------------------------------------

# 🛣 Router

## Definition

A router is a networking device that connects different networks
together and decides the best path for data to travel.

Without routers, your home network could never communicate with the
Internet.

    Home Network
          |
       Router
          |
       Internet

### Responsibilities

-   Connect different networks
-   Forward packets
-   Choose routes
-   Connect private networks to the Internet

### Real-World Analogy

A router is like a highway interchange.

It decides which road a vehicle should take to reach its destination.

------------------------------------------------------------------------

# 🔀 Switch

## Definition

A switch connects devices **inside the same local network (LAN).**

    PC A
       |
    PC B --- Switch --- Printer
       |
    Server

Unlike a router, a switch does **not** connect different networks.

It only allows devices in the same LAN to communicate efficiently.

### Responsibilities

-   Connect local devices
-   Forward frames to the correct device
-   Reduce unnecessary network traffic

------------------------------------------------------------------------

# 🚪 Gateway

A gateway is the device that allows your network to reach another
network.

Most home networks use the router as the default gateway.

    Laptop
       |
    Switch
       |
    Default Gateway
       |
    Internet

Whenever your computer wants to reach Google, GitHub, or AWS, it sends
the traffic to the default gateway.

------------------------------------------------------------------------

# 📦 Packet

## Definition

A packet is a small unit of data transmitted across a network.

Instead of sending one huge file at once, computers divide it into many
packets.

    Large File

    ↓

    Packet 1
    Packet 2
    Packet 3
    Packet 4

Each packet travels independently and may even take different routes.

The destination device rebuilds the original data.

------------------------------------------------------------------------

# 📨 Frame

Many beginners confuse packets and frames.

A frame exists only while data travels across a local network.

    Frame

    +----------------+
    | Header         |
    | Payload        |
    | Trailer        |
    +----------------+

A packet is wrapped inside a frame before being transmitted across the
local network.

------------------------------------------------------------------------

# 📊 Bandwidth

Bandwidth is the maximum amount of data that can be transferred over a
network during a given period.

Examples:

-   100 Mbps
-   1 Gbps
-   10 Gbps

Higher bandwidth means more data can be transferred simultaneously.

Think of bandwidth as the width of a highway.

    Two-lane road

    ↓

    Four-lane highway

    ↓

    Eight-lane highway

More lanes allow more cars to travel at the same time.

------------------------------------------------------------------------

# ⏱ Latency

Latency is the time required for data to travel from one device to
another.

Usually measured in milliseconds (ms).

Examples:

-   2 ms
-   10 ms
-   80 ms
-   200 ms

Low latency means faster communication.

High latency causes delays.

### Real-World Analogy

Bandwidth answers:

> How much water can flow through a pipe?

Latency answers:

> How long does the first drop take to arrive?

These are different concepts.

------------------------------------------------------------------------

# 📈 Bandwidth vs Latency

  Bandwidth        Latency
  ---------------- --------------
  Amount of data   Time delay
  Mbps/Gbps        Milliseconds
  Highway width    Travel time

A network may have:

-   High bandwidth but high latency.
-   Low bandwidth but low latency.

Both affect application performance differently.

------------------------------------------------------------------------

# 🏗 Production Example

A company deploys a Spring Boot application.

Users complain that pages load slowly.

The server CPU is healthy.

Memory is healthy.

Disk is healthy.

Investigation shows latency between the application server and database
increased from **2 ms** to **150 ms**.

The application isn't overloaded.

The network is delaying every request.

------------------------------------------------------------------------

# 🐧 Linux Perspective

Linux exposes these networking concepts through tools you'll learn
later:

``` bash
ip
ping
tracepath
ss
ip route
```

These commands help administrators understand how packets travel and
where communication problems occur.

------------------------------------------------------------------------

# 📌 Lesson Summary

You learned:

-   Router
-   Switch
-   Gateway
-   Packet
-   Frame
-   Bandwidth
-   Latency

Together with the previous parts, you now understand the fundamental
building blocks of computer networking.

------------------------------------------------------------------------

# ➡️ Next Lesson

Lesson 02 --- The OSI Model

You'll learn how networking is divided into layers and how each layer
contributes to successful communication.

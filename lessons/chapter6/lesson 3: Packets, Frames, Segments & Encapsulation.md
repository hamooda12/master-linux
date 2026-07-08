# Chapter 06 --- Networking Fundamentals

# Lesson 01 --- How the Internet Actually Works

## Part 3 --- Packets, Frames, Segments & Encapsulation

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will understand:

-   Why computers split data into packets
-   The difference between application data, segments, packets and
    frames
-   What encapsulation means
-   What decapsulation means
-   Why this layered design makes the Internet possible

------------------------------------------------------------------------

# The Problem

Imagine you want to send a 2 GB backup file to another server.

Would the operating system send one gigantic block?

No.

If the transmission failed at 99%, the entire file would need to be sent
again.

That would be slow, unreliable and waste bandwidth.

Instead, computers split data into many small pieces.

------------------------------------------------------------------------

# Why Packets Exist

Suppose your application sends a large file.

Instead of:

``` text
2 GB
```

The operating system creates something like:

``` text
Packet 1
Packet 2
Packet 3
...
Packet 150000
```

Advantages:

-   Easier recovery from errors
-   Better use of network bandwidth
-   Multiple applications can share the network
-   Routers can forward data efficiently

Think of moving a house.

You don't put everything into one enormous truck.

You use many boxes.

If one box is damaged, you replace only that box.

------------------------------------------------------------------------

# Layers Build Upon Each Other

When your browser creates an HTTP request, it is only application data.

It cannot travel across the network by itself.

Each networking layer adds information.

``` text
Application Data
        │
        ▼
TCP Segment
        │
        ▼
IP Packet
        │
        ▼
Ethernet Frame
        │
        ▼
Bits on Cable or Wi-Fi
```

Each layer has one responsibility.

------------------------------------------------------------------------

# Segment

The Transport Layer (TCP or UDP) wraps the application data.

It adds information such as:

-   Source Port
-   Destination Port
-   Reliability (TCP)
-   Sequence Numbers

Now the data is called a **segment** (for TCP).

------------------------------------------------------------------------

# Packet

Next, the Network Layer adds IP information.

It includes:

-   Source IP Address
-   Destination IP Address

Now it becomes an **IP packet**.

Routers use this information to move the packet between networks.

------------------------------------------------------------------------

# Frame

Finally, the Data Link Layer prepares the packet for the local network.

It adds:

-   Source MAC Address
-   Destination MAC Address

Now it becomes an **Ethernet frame**.

Switches understand frames.

------------------------------------------------------------------------

# Encapsulation

The process of wrapping data with headers is called **encapsulation**.

``` text
HTTP Request
      │
      ▼
+ TCP Header
      │
      ▼
+ IP Header
      │
      ▼
+ Ethernet Header
      │
      ▼
Sent Across the Network
```

Each layer wraps the previous one.

Like placing:

-   A letter
-   Inside an envelope
-   Inside a shipping box
-   Inside a delivery truck

------------------------------------------------------------------------

# Decapsulation

On the destination server, the opposite happens.

``` text
Ethernet Frame
       │
Remove Ethernet Header
       ▼
IP Packet
       │
Remove IP Header
       ▼
TCP Segment
       │
Remove TCP Header
       ▼
HTTP Request
       │
       ▼
Nginx
```

Each layer removes only its own header and passes the remaining data
upward.

------------------------------------------------------------------------

# Production Example

A browser sends:

``` http
GET /api/users
```

Before it reaches Nginx, it has been transformed into:

``` text
Ethernet Frame
 └── IP Packet
      └── TCP Segment
           └── HTTP Request
```

Nginx never sees the Ethernet frame.

It receives only the HTTP request after lower layers finish their work.

------------------------------------------------------------------------

# Why DevOps Engineers Care

Tools inspect different layers.

Examples:

  Tool      What It Helps Inspect
  --------- -----------------------------
  ip        Network layer
  ss        Transport layer
  tcpdump   Frames, packets and traffic
  curl      Application requests

Understanding the layers tells you which tool to use.

------------------------------------------------------------------------

# Common Mistakes

❌ Calling everything a "packet".

❌ Thinking applications send Ethernet frames directly.

❌ Believing routers read HTTP requests.

Routers forward packets based on IP addresses---not web content.

------------------------------------------------------------------------

# Summary

-   Applications create data.
-   TCP/UDP creates segments.
-   IP creates packets.
-   Ethernet creates frames.
-   Encapsulation wraps data.
-   Decapsulation unwraps it.
-   Every layer has a specific responsibility.

------------------------------------------------------------------------

# Next Part

In Part 4 we'll learn:

-   Network Interfaces
-   MAC Addresses
-   Switches
-   ARP
-   How computers communicate on the same local network

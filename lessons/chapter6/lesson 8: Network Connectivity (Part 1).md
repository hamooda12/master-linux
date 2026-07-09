# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 08 --- Network Connectivity (Part 1)

> **Goal:** Learn how Linux tests connectivity between devices and
> understand the tools used to verify whether a host is reachable.

------------------------------------------------------------------------

# 📚 Overview

One of the first questions a Linux administrator asks during a network
outage is:

> **"Can these two systems communicate?"**

Before checking applications, databases, or APIs, you must first verify
basic network connectivity.

Linux provides several tools for this purpose, including:

-   `ping`
-   `traceroute`
-   `tracepath`
-   `mtr`

This lesson begins with the most fundamental tool: **ping**.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain what network connectivity means.
-   Understand the purpose of ICMP.
-   Explain how `ping` works.
-   Interpret basic `ping` output.
-   Understand latency and packet loss.

------------------------------------------------------------------------

# 🌍 What Is Network Connectivity?

Network connectivity simply means that two devices can communicate
across a network.

Example:

    Laptop
       |
    Router
       |
    Linux Server

If packets successfully travel between them, the devices have network
connectivity.

------------------------------------------------------------------------

# ICMP

## Definition

`ping` uses the **Internet Control Message Protocol (ICMP)**.

ICMP is not used to transfer application data.

Instead, it is used to:

-   Test connectivity
-   Report network errors
-   Help diagnose networking problems

Think of ICMP as the network's diagnostic protocol.

------------------------------------------------------------------------

# The `ping` Command

The simplest connectivity test is:

``` bash
ping google.com
```

or

``` bash
ping 8.8.8.8
```

Linux repeatedly sends small ICMP Echo Request packets.

If the destination responds, connectivity exists.

------------------------------------------------------------------------

# How `ping` Works

    Computer
         |
    Echo Request
         |
    Destination
         |
    Echo Reply
         |
    Computer

Each successful reply confirms that the destination responded.

------------------------------------------------------------------------

# Example Output

``` text
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.4 ms
```

Important fields:

-   `64 bytes` → Size of the reply.
-   `icmp_seq=1` → Packet sequence number.
-   `ttl=117` → Time To Live value.
-   `time=12.4 ms` → Round-trip latency.

------------------------------------------------------------------------

# Latency

Latency measures how long it takes a packet to travel to the destination
and back.

Example:

    time=0.5 ms

Very fast.

    time=120 ms

Noticeably slower.

Lower latency generally means faster communication.

------------------------------------------------------------------------

# Packet Loss

Sometimes packets never reach the destination.

Example:

    10 packets transmitted
    8 received
    20% packet loss

Packet loss may indicate:

-   Network congestion
-   Hardware failures
-   Wireless interference
-   Router problems
-   Firewall filtering

------------------------------------------------------------------------

# Linux Perspective

`ping` is often the first command used during troubleshooting because it
quickly answers a simple question:

> Can I reach the destination?

However, a successful ping does **not** guarantee that applications such
as HTTP or SSH are working.

------------------------------------------------------------------------

# Production Example

Users report they cannot access a website.

First investigation:

``` bash
ping server.example.com
```

If there is no reply, continue investigating the network before
restarting applications.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Network connectivity
-   ICMP
-   `ping`
-   Latency
-   Packet loss
-   Basic ping output

------------------------------------------------------------------------

# ➡️ Next Part

We'll analyze `ping` output in detail, understand Time To Live (TTL),
and learn when ping succeeds but applications still fail.

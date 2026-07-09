# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 05 --- Network Interfaces (Part 2)

------------------------------------------------------------------------

# 🔄 The Loopback Interface

## Definition

The **loopback interface** is a special virtual network interface that
allows a computer to communicate with itself.

Unlike Ethernet or Wi-Fi interfaces, the loopback interface never sends
traffic onto a physical network.

It exists entirely inside the operating system.

    Application
         |
      Loopback
         |
    Application

------------------------------------------------------------------------

## Why Does Linux Need a Loopback Interface?

Many applications communicate with services running on the same machine.

For example:

-   A browser testing a local web server
-   Spring Boot connecting to another local service
-   Monitoring software checking local applications
-   Databases accepting local connections

Without the loopback interface, local network communication would be
much more difficult.

------------------------------------------------------------------------

## Loopback Address

The standard IPv4 loopback address is:

    127.0.0.1

The hostname:

    localhost

normally resolves to:

    127.0.0.1

For IPv6, the loopback address is:

    ::1

------------------------------------------------------------------------

## Example

    Browser

          |

    http://localhost:8080

          |

    Spring Boot

The request never leaves your computer.

No router or switch is involved.

------------------------------------------------------------------------

# 🔑 MAC Address

## Definition

Every network interface has a unique **MAC (Media Access Control)
address**.

Example:

    08:00:27:4A:91:BC

Unlike an IP address, a MAC address identifies the physical network
interface.

------------------------------------------------------------------------

## IP Address vs MAC Address

  IP Address              MAC Address
  ----------------------- -----------------------------
  Logical address         ----------------- Physical hardware address
  
  Used between networks   ----------------- Used inside a local network
  
  Can change              ----------------- Usually fixed by hardware
  
  Example: 192.168.1.20   ----------------- Example: 08:00:27:4A:91:BC

Both are important but serve different purposes.

------------------------------------------------------------------------

# 🖥 Inspecting Interfaces with Linux

Linux provides the **ip** command for viewing network interfaces.

## Display All Interfaces

``` bash
ip link
```

Example output:

``` text
1: lo: <LOOPBACK,UP> mtu 65536 state UNKNOWN
2: enp0s3: <BROADCAST,MULTICAST,UP> mtu 1500 state UP
```

------------------------------------------------------------------------

# Understanding the Output

    lo

The loopback interface.

------------------------------------------------------------------------

    enp0s3

The primary Ethernet interface.

------------------------------------------------------------------------

    UP

The interface is enabled.

------------------------------------------------------------------------

    mtu 1500

The Maximum Transmission Unit.

It represents the largest packet size that can be transmitted without
fragmentation.

------------------------------------------------------------------------

# Viewing IP Addresses

Linux also allows you to display IP addresses assigned to interfaces.

``` bash
ip addr
```

Example:

``` text
2: enp0s3

inet 192.168.1.20/24
```

This shows:

-   Interface name
-   IPv4 address
-   Network prefix

------------------------------------------------------------------------

# Linux Perspective

The `ip` command replaces many older networking utilities.

It is the preferred tool for modern Linux systems.

Future lessons will use it extensively.

------------------------------------------------------------------------

# Production Example

Suppose users cannot access your web application.

The first investigation might be:

``` bash
ip link
```

If the interface is DOWN, no further network troubleshooting is
meaningful until connectivity is restored.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Loopback interface
-   localhost
-   127.0.0.1
-   MAC addresses
-   Difference between MAC and IP addresses
-   ip link
-   ip addr
-   Reading basic interface information

------------------------------------------------------------------------

# ➡️ Next Part

We'll analyze command output in detail, explore interface states
further, and connect network interfaces to real production
troubleshooting.

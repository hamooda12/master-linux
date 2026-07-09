# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 04 --- IP Addressing (Part 2)

------------------------------------------------------------------------

# 🧮 Subnet Mask

## Definition

A subnet mask determines which part of an IP address identifies the
**network** and which part identifies the **host**.

Without a subnet mask, a computer cannot determine whether another
device is on the same network or on a different network.

Example:

    IP Address : 192.168.1.25
    Subnet Mask: 255.255.255.0

The subnet mask tells Linux:

-   Network portion → `192.168.1`
-   Host portion → `25`

------------------------------------------------------------------------

# Why Do We Need a Subnet Mask?

Imagine an apartment building.

    Building 12
     ├── Apartment 1
     ├── Apartment 2
     ├── Apartment 3

The building number is like the **network**.

The apartment number is like the **host**.

The subnet mask separates these two parts.

------------------------------------------------------------------------

# 🌐 Network Address

The network address identifies the network itself.

Devices **cannot** use the network address as their own IP.

Example:

    Network: 192.168.1.0/24

Here:

    192.168.1.0

represents the network.

------------------------------------------------------------------------

# 🖥 Host Address

Host addresses belong to individual devices.

Example:

    192.168.1.10
    192.168.1.20
    192.168.1.30

Each host on the network must have a unique address.

Duplicate IP addresses cause communication problems.

------------------------------------------------------------------------

# 📢 Broadcast Address

The broadcast address targets **every host** on the same network.

Example:

    Network:   192.168.1.0/24

    Broadcast: 192.168.1.255

Packets sent to the broadcast address are delivered to every device in
that subnet.

------------------------------------------------------------------------

# Valid Host Range

For the network:

    192.168.1.0/24

  Address                       Purpose
  ----------------------------- -----------
  192.168.1.0                   Network
  192.168.1.1 - 192.168.1.254   Hosts
  192.168.1.255                 Broadcast

Only host addresses can be assigned to computers.

------------------------------------------------------------------------

# Linux Perspective

When Linux receives an IP address and subnet mask, it automatically
determines:

-   Network address
-   Host address
-   Broadcast address

You don't calculate these manually every day, but understanding them
helps during troubleshooting.

------------------------------------------------------------------------

# Production Example

Suppose two servers should communicate.

Server A

    192.168.1.20/24

Server B

    192.168.1.35/24

Both belong to the same network.

No router is required for them to communicate.

Now change Server B to:

    192.168.2.35/24

The servers are now on different networks.

A router is required.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Subnet mask
-   Network address
-   Host address
-   Broadcast address
-   Valid host ranges

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn CIDR notation and how modern Linux systems represent
networks using prefixes like `/24`, `/16`, and `/8`.

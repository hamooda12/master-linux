# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 04 --- IP Addressing (Part 1)

> **Goal:** Understand what an IP address is, why every networked device
> needs one, and how IPv4 and IPv6 enable communication across networks.

------------------------------------------------------------------------

# 📚 Overview

Computers cannot communicate simply because they are connected to a
network.

Each device must have a unique logical address so other devices know
where to send data.

That logical address is called an **IP address**.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain the purpose of an IP address.
-   Distinguish between IPv4 and IPv6.
-   Understand public and private IP addresses.
-   Recognize common IPv4 address formats.

------------------------------------------------------------------------

# 🌍 Why Do We Need IP Addresses?

Imagine a city with thousands of houses but no street names or house
numbers.

A delivery driver would never know where to deliver a package.

Networks have the same problem.

Without addresses, routers would not know where packets should go.

    Client
       |
    Internet
       |
    Destination IP Address
       |
    Server

------------------------------------------------------------------------

# 📖 What Is an IP Address?

An **Internet Protocol (IP) address** is a logical address assigned to a
network device.

Its primary purpose is to identify a device and allow packets to be
delivered to the correct destination.

Examples:

    192.168.1.25
    10.0.0.15
    172.16.5.10

------------------------------------------------------------------------

# IPv4

IPv4 is the most widely used addressing system.

It contains **32 bits** divided into four numbers called octets.

Example:

    192.168.1.25

Each octet ranges from **0 to 255**.

    192 . 168 . 1 . 25

------------------------------------------------------------------------

# IPv6

IPv6 was introduced because IPv4 addresses are limited.

IPv6 uses **128 bits**, allowing an enormous number of unique addresses.

Example:

    2001:db8:85a3::8a2e:370:7334

You will see IPv6 frequently on modern Linux systems and cloud
platforms.

------------------------------------------------------------------------

# IPv4 vs IPv6

  Feature          IPv4           IPv6
  ---------------- -------------- -------------
  Address Length   32 bits        128 bits
  Format           Decimal        Hexadecimal
  Example          192.168.1.10   2001:db8::1

------------------------------------------------------------------------

# Public IP Address

A public IP address is reachable over the Internet.

Examples include:

-   Public websites
-   Cloud virtual machines
-   Public APIs

Your Internet Service Provider (ISP) typically assigns your router a
public IP address.

------------------------------------------------------------------------

# Private IP Address

Private IP addresses are used only inside private networks.

They are **not directly reachable from the Internet**.

Common private ranges:

    10.0.0.0/8
    172.16.0.0/12
    192.168.0.0/16

Home routers commonly assign addresses such as:

    192.168.1.100

------------------------------------------------------------------------

# Linux Perspective

Every Linux server has at least one IP address.

Later lessons will show how to inspect them using:

``` bash
ip addr
hostname -I
```

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Why IP addresses exist
-   IPv4
-   IPv6
-   Public IP addresses
-   Private IP addresses

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn subnet masks, CIDR notation, network addresses, broadcast
addresses, and host addresses.

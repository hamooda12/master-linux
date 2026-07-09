# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 13 --- Network Configuration (Part 1)

> **Goal:** Understand how Linux obtains network configuration and how
> IP addresses, gateways, and DNS settings are assigned to network
> interfaces.

------------------------------------------------------------------------

# 📚 Overview

Up to this point, you've learned how Linux communicates over a network.

Now it's time to learn **how Linux is configured to communicate**.

A network interface is not useful by itself. It must be configured with:

-   An IP address
-   A subnet prefix
-   A default gateway
-   DNS servers

Without these settings, a Linux system cannot properly communicate with
other devices.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain what network configuration is.
-   Understand the difference between dynamic and static configuration.
-   Explain DHCP.
-   Explain static IP addressing.
-   Identify the basic components of a Linux network configuration.

------------------------------------------------------------------------

# What Is Network Configuration?

Network configuration is the process of assigning the information a
computer needs to communicate on a network.

Typical configuration includes:

    IP Address

    192.168.1.20

    Subnet Prefix

    /24

    Default Gateway

    192.168.1.1

    DNS Server

    8.8.8.8

------------------------------------------------------------------------

# Two Ways to Configure a Network

Linux systems generally receive network settings in one of two ways:

1.  Dynamic configuration (DHCP)
2.  Static configuration

------------------------------------------------------------------------

# Dynamic Configuration (DHCP)

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns
network settings.

Typical information provided by a DHCP server:

-   IP address
-   Subnet prefix
-   Default gateway
-   DNS servers
-   Lease duration

```{=html}
<!-- -->
```
    Linux Client
          │
    DHCP Request
          │
    DHCP Server
          │
    Configuration Assigned

Most laptops and desktop computers use DHCP.

------------------------------------------------------------------------

# Static Configuration

With static configuration, the administrator manually specifies the
settings.

Example:

    IP Address

    192.168.1.50

    Gateway

    192.168.1.1

    DNS

    1.1.1.1

These settings remain unchanged until an administrator modifies them.

------------------------------------------------------------------------

# DHCP vs Static Addressing

  DHCP                 Static
  -------------------- ------------------------
  Automatic            Manual
  Easy to manage       Requires configuration
  IP may change        IP remains fixed
  Common for clients   Common for servers

------------------------------------------------------------------------

# When Should You Use Static IPs?

Servers commonly use static addresses because clients must always know
how to reach them.

Examples:

-   Web servers
-   Database servers
-   DNS servers
-   Load balancers
-   Monitoring servers

Changing these addresses unexpectedly could interrupt services.

------------------------------------------------------------------------

# Linux Perspective

Modern Linux systems commonly use:

``` bash
ip addr
```

to verify assigned addresses.

Later in this lesson you'll also learn how Ubuntu manages persistent
network configuration using **Netplan**.

------------------------------------------------------------------------

# Production Example

A company deploys a new database server.

If its IP address changes every reboot because of DHCP, applications may
lose connectivity.

Assigning a static IP ensures the database remains reachable at the same
address.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   What network configuration is.
-   DHCP.
-   Static IP addressing.
-   When each approach is appropriate.
-   The essential components of network configuration.

------------------------------------------------------------------------

# ➡️ Next Part

We'll explore Netplan, inspect current network settings, and learn how
modern Ubuntu systems store persistent network configuration.

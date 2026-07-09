# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 05 --- Network Interfaces (Part 4)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Interface is DOWN

Users cannot reach a Linux server.

First command:

``` bash
ip link
```

Output:

``` text
2: enp0s3: <BROADCAST,MULTICAST>
state DOWN
```

### Investigation

The operating system detects the interface, but it is disabled.

Before checking DNS, routing, or applications, restore the network
interface.

------------------------------------------------------------------------

## Scenario 2 --- Interface Has No IP Address

``` bash
ip addr
```

Output:

``` text
2: enp0s3

<no inet address>
```

### Investigation

The interface exists but has no IPv4 address.

Possible causes:

-   DHCP failure
-   Static IP not configured
-   Configuration error

Without an IP address, other devices cannot communicate with this
server.

------------------------------------------------------------------------

## Scenario 3 --- Wrong Interface

A server contains:

``` text
enp0s3

enp0s8
```

The application administrator configures the service to use the wrong
interface.

Result:

-   Application starts successfully.
-   Users cannot reach it through the expected network.

Always verify which interface is connected to the production network.

------------------------------------------------------------------------

# Troubleshooting Workflow

When network communication fails:

    Is the interface present?

            ↓

    Is it UP?

            ↓

    Does it have an IP address?

            ↓

    Is the subnet correct?

            ↓

    Check routing

            ↓

    Check DNS

            ↓

    Check applications

This systematic approach minimizes unnecessary changes.

------------------------------------------------------------------------

# Common Mistakes

## Ignoring the Interface State

Many administrators immediately investigate the application.

If the interface is DOWN, application debugging is pointless.

------------------------------------------------------------------------

## Confusing Loopback with Physical Interfaces

Traffic sent to:

    127.0.0.1

never leaves the computer.

Traffic sent to:

    192.168.x.x

uses a real network interface.

------------------------------------------------------------------------

## Assuming Every Interface Is Physical

Modern Linux systems frequently contain virtual interfaces created by:

-   Docker
-   Virtual machines
-   VPN software
-   Cloud networking

------------------------------------------------------------------------

# Best Practices

-   Learn interface names before making changes.
-   Verify interface status before troubleshooting higher networking
    layers.
-   Prefer the `ip` command over older networking utilities.
-   Always verify changes after modifying network settings.
-   Document interface configurations in production environments.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is a network interface?
2.  What is the purpose of a NIC?
3.  What does the loopback interface do?

## Intermediate

1.  Explain the difference between a MAC address and an IP address.
2.  Why might a Linux server have multiple interfaces?
3.  What information does `ip addr` display?

## Advanced

1.  Why can an interface be UP while users still cannot access the
    application?
2.  Describe a logical troubleshooting process when a server becomes
    unreachable.

------------------------------------------------------------------------

# Hands-on Lab

Run the following commands on your Linux machine:

``` bash
ip link
```

``` bash
ip addr
```

Answer:

1.  How many interfaces exist?
2.  Which interface is the loopback interface?
3.  Which interface has an IPv4 address?
4.  Does your system have an IPv6 address?
5.  Which interface is currently UP?

------------------------------------------------------------------------

# Challenge Exercise

A Linux server suddenly becomes unreachable.

Without restarting any services, write the first five investigation
steps you would perform using only the concepts learned so far.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 05 you understand:

-   Network interfaces
-   NICs
-   Physical and virtual interfaces
-   Loopback
-   localhost
-   MAC addresses
-   Interface states
-   `ip link`
-   `ip addr`
-   Reading interface information
-   Basic interface troubleshooting

These concepts prepare you for routing, DNS, and production network
troubleshooting.

------------------------------------------------------------------------

# Cheat Sheet

  Concept             Purpose
  ------------------- -----------------------------------
  Network Interface   Connects Linux to a network
  NIC                 Network hardware
  Loopback            Local communication (`127.0.0.1`)
  MAC Address         Physical interface identifier
  `ip link`           View interfaces and states
  `ip addr`           View IP addresses
  UP                  Interface enabled
  DOWN                Interface disabled
  MTU                 Maximum Transmission Unit

------------------------------------------------------------------------

# What's Next

Lesson 06 --- Routing

You'll learn how Linux decides where packets should go, what a routing
table is, how the default gateway works, and how to inspect routes using
the `ip route` command.

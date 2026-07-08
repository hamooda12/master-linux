# Chapter 06 --- Networking Fundamentals

# Lesson 12 --- Linux Network Configuration for DevOps Engineers

> **Objective:** Learn how Linux represents, configures, and uses
> networking so you can confidently troubleshoot servers in production.

------------------------------------------------------------------------

# Learning Objectives

By the end of this lesson you will be able to:

-   Explain what a network interface is.
-   Distinguish physical and virtual interfaces.
-   Understand the loopback interface.
-   Read Linux IP configuration.
-   Explain DHCP vs Static IP.
-   Understand the default gateway.
-   Read the routing table.
-   Understand hostname and DNS resolution.
-   Use the `ip` command confidently.
-   Troubleshoot common Linux networking issues.

------------------------------------------------------------------------

# Why This Matters

Every network problem eventually reaches the operating system.

Before checking Docker, Kubernetes, Nginx, AWS, or your application, you
must first answer:

-   Does the server have the correct IP?
-   Is the interface up?
-   Does it know where to send packets?
-   Can it resolve hostnames?
-   Is the service listening?

If the Linux network configuration is wrong, nothing above it will work.

------------------------------------------------------------------------

# Linux Networking Stack

``` text
Application
      │
TCP / UDP
      │
IP
      │
Network Interface
      │
NIC (Ethernet / Wi-Fi / Virtual)
      │
Switch / Router
```

------------------------------------------------------------------------

# What Is a Network Interface?

A network interface is the operating system's connection to a network.

Think of it as a door through which packets enter and leave.

Examples:

-   Ethernet card
-   Wi-Fi adapter
-   Virtual machine adapter
-   Docker bridge
-   VPN tunnel

Each interface has:

-   Name
-   MAC address
-   IP address(es)
-   State (UP/DOWN)

------------------------------------------------------------------------

# Physical vs Virtual Interfaces

## Physical

``` text
eth0
ens33
enp0s3
wlan0
```

Connected to real hardware.

## Virtual

``` text
lo
docker0
vethXXXX
tun0
br-xxxxxxxx
```

Created by Linux, Docker, Kubernetes, VPNs, or virtualization software.

------------------------------------------------------------------------

# The Loopback Interface

``` text
lo
```

Address:

``` text
127.0.0.1
```

Purpose:

Allows a machine to communicate with itself.

Examples:

``` bash
curl http://127.0.0.1:8080
```

Even with no internet connection, loopback still works.

------------------------------------------------------------------------

# Viewing Interfaces

``` bash
ip link
```

Example:

``` text
1: lo
2: enp0s3
3: docker0
```

Show addresses:

``` bash
ip addr
```

------------------------------------------------------------------------

# Understanding an IP Address

Example:

``` text
192.168.1.20/24
```

-   Network: 192.168.1.0
-   Host: 20
-   Prefix: /24

The interface may have multiple IPv4 and IPv6 addresses.

------------------------------------------------------------------------

# Interface States

``` text
UP
```

Ready to send and receive traffic.

``` text
DOWN
```

Disabled.

Bring an interface down:

``` bash
sudo ip link set enp0s3 down
```

Bring it back:

``` bash
sudo ip link set enp0s3 up
```

------------------------------------------------------------------------

# DHCP vs Static IP

## DHCP

A DHCP server automatically assigns:

-   IP address
-   Gateway
-   DNS
-   Subnet mask

Advantages:

-   Easy management
-   Automatic configuration

## Static IP

Configured manually.

Common for:

-   Servers
-   Databases
-   Load balancers

------------------------------------------------------------------------

# The Default Gateway

Your server only knows its local network.

To reach another network, packets are sent to the default gateway.

View it:

``` bash
ip route
```

Example:

``` text
default via 192.168.1.1 dev enp0s3
```

Meaning:

"If I don't know where the destination is, send the packet to
192.168.1.1."

------------------------------------------------------------------------

# Reading the Routing Table

``` bash
ip route
```

Example:

``` text
default via 192.168.1.1 dev enp0s3
192.168.1.0/24 dev enp0s3
172.17.0.0/16 dev docker0
```

Interpretation:

-   Unknown destinations → gateway
-   Local LAN → directly through enp0s3
-   Docker network → docker0

------------------------------------------------------------------------

# Hostnames

Current hostname:

``` bash
hostname
```

Detailed information:

``` bash
hostnamectl
```

------------------------------------------------------------------------

# Local Name Resolution

File:

``` text
/etc/hosts
```

Example:

``` text
127.0.0.1 localhost
192.168.1.50 database
```

Linux checks this file before querying DNS.

------------------------------------------------------------------------

# DNS Configuration

File:

``` text
/etc/resolv.conf
```

Example:

``` text
nameserver 8.8.8.8
nameserver 1.1.1.1
```

These are the DNS servers Linux queries.

------------------------------------------------------------------------

# NetworkManager

Ubuntu often uses NetworkManager.

Useful command:

``` bash
nmcli
```

Examples:

``` bash
nmcli device status
nmcli connection show
```

------------------------------------------------------------------------

# Essential Commands

``` bash
ip addr
ip link
ip route
hostname
hostnamectl
ss -tulpen
ping 8.8.8.8
ping google.com
cat /etc/hosts
cat /etc/resolv.conf
nmcli
```

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- No Internet

Checklist:

1.  Interface UP?
2.  IP assigned?
3.  Default gateway exists?
4.  Can ping gateway?
5.  DNS configured?

Commands:

``` bash
ip addr
ip route
ping 192.168.1.1
ping 8.8.8.8
ping google.com
```

Interpretation:

-   Gateway fails → local network issue.
-   Gateway works, 8.8.8.8 fails → routing.
-   8.8.8.8 works, hostname fails → DNS.

------------------------------------------------------------------------

## Scenario 2 --- Website Works Locally but Not Remotely

Verify:

``` bash
ss -tulpen
ip addr
ip route
```

Possible causes:

-   Service bound to 127.0.0.1
-   Firewall
-   Wrong IP
-   Missing route

------------------------------------------------------------------------

## Scenario 3 --- Wrong Hostname Resolution

``` bash
cat /etc/hosts
cat /etc/resolv.conf
```

Incorrect entries can cause applications to connect to the wrong server.

------------------------------------------------------------------------

# Common Mistakes

-   Confusing interface DOWN with cable problems.
-   Assuming ping proves an application works.
-   Forgetting the default gateway.
-   Editing `/etc/hosts` without understanding precedence.
-   Thinking Docker interfaces are physical NICs.

------------------------------------------------------------------------

# Best Practices

-   Use static IPs for infrastructure servers.
-   Document every network interface.
-   Audit routing after network changes.
-   Keep DNS configuration consistent.
-   Learn the `ip` command instead of relying on deprecated tools.

------------------------------------------------------------------------

# Interview Questions

1.  What is a network interface?
2.  What is the purpose of the loopback interface?
3.  Difference between DHCP and Static IP?
4.  What is a default gateway?
5.  What does `ip route` display?
6.  Difference between `/etc/hosts` and DNS?
7.  Why might `ping 8.8.8.8` work while `ping google.com` fails?

------------------------------------------------------------------------

# Cheat Sheet

``` bash
ip addr
ip link
ip route
hostname
hostnamectl
nmcli device status
ss -tulpen
ping 8.8.8.8
ping google.com
cat /etc/hosts
cat /etc/resolv.conf
```

------------------------------------------------------------------------

# Summary

Linux networking begins with correctly configured interfaces, IP
addresses, routes, gateways, and DNS. Before investigating firewalls,
reverse proxies, or applications, always verify the operating system's
network configuration. Mastering these fundamentals will make diagnosing
production incidents much faster and more reliable.

------------------------------------------------------------------------

# Next Lesson

**Lesson 13 --- Routing Basics for DevOps Engineers**

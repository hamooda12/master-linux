# 🐧 Linux Networking Command Reference (README)

> **Companion guide for Chapter 06 -- Linux Networking Fundamentals**
>
> This document focuses on **how to use the networking commands** you
> learned in the chapter. It is intended as a practical reference after
> understanding the networking concepts.

------------------------------------------------------------------------

# 📋 Command Categories

-   Interface Inspection
-   IP Address Management
-   Routing
-   DNS
-   Connectivity Testing
-   Ports & Sockets
-   Network Statistics
-   ARP / Neighbor Cache

------------------------------------------------------------------------

# 🌐 Interface Inspection

## `ip link`

**Purpose**

Display or manage network interfaces.

**Common Usage**

``` bash
ip link
ip link show
ip link show wlo1
```

**Use When**

-   Listing interfaces
-   Checking interface names
-   Viewing interface state (UP/DOWN)

------------------------------------------------------------------------

## `ip addr`

**Purpose**

Display IP addresses assigned to interfaces.

**Common Usage**

``` bash
ip addr
ip addr show
ip addr show wlo1
```

Shows:

-   IPv4
-   IPv6
-   Prefix length
-   Interface information

------------------------------------------------------------------------

# 🛰 IP Address Configuration

## Add an IP

``` bash
sudo ip addr add 192.168.1.100/24 dev eth0
```

## Remove an IP

``` bash
sudo ip addr del 192.168.1.100/24 dev eth0
```

> Temporary changes only.

------------------------------------------------------------------------

# 🛣 Routing

## Show routing table

``` bash
ip route
```

## Add a route

``` bash
sudo ip route add 10.20.0.0/16 via 192.168.1.254
```

## Add a default gateway

``` bash
sudo ip route add default via 192.168.1.1
```

## Delete a route

``` bash
sudo ip route del 10.20.0.0/16
```

------------------------------------------------------------------------

# 🔎 DNS

## Query DNS

``` bash
dig google.com
dig google.com MX
dig google.com TXT
```

## Simple lookup

``` bash
nslookup google.com
```

## Legacy lookup

``` bash
host google.com
```

## Resolver configuration

``` bash
cat /etc/resolv.conf
```

------------------------------------------------------------------------

# 📡 Connectivity Testing

## Ping

``` bash
ping google.com
ping -c 4 google.com
```

## Trace packet path

``` bash
traceroute google.com
```

or

``` bash
tracepath google.com
```

------------------------------------------------------------------------

# 🚪 Ports & Sockets

## Display listening sockets

``` bash
ss -tuln
```

## Display processes

``` bash
ss -tulpn
```

Useful options:

-   `-t` TCP
-   `-u` UDP
-   `-l` Listening
-   `-n` Numeric
-   `-p` Process

------------------------------------------------------------------------

# 📈 Network Statistics

## Interface statistics

``` bash
ip -s link
```

## Connection statistics

``` bash
ss -s
```

------------------------------------------------------------------------

# 🤝 ARP / Neighbor Cache

## Show neighbor table

``` bash
ip neigh
```

## Flush cache

``` bash
sudo ip neigh flush all
```

------------------------------------------------------------------------

# 🛠 Troubleshooting Workflow

1.  Check interface

``` bash
ip link
```

2.  Check IP

``` bash
ip addr
```

3.  Check routing

``` bash
ip route
```

4.  Test gateway

``` bash
ping <gateway-ip>
```

5.  Test Internet

``` bash
ping 8.8.8.8
```

6.  Test DNS

``` bash
ping google.com
dig google.com
```

7.  Check listening ports

``` bash
ss -tulpn
```

------------------------------------------------------------------------

# ⚠ Common Mistakes

-   Confusing **interface state** with **IP configuration**.
-   Adding routes with an unreachable gateway.
-   Forgetting that `ip` changes are temporary.
-   Assuming `localhost` tests external connectivity.
-   Reading `ip route` without identifying the matching destination
    network.

------------------------------------------------------------------------

# 📚 Quick Cheat Sheet

  Command        Purpose
  -------------- -------------------------------
  `ip link`      Network interfaces
  `ip addr`      IP addresses
  `ip route`     Routing table
  `ip neigh`     ARP/neighbor cache
  `ping`         Connectivity
  `traceroute`   Packet path
  `dig`          DNS queries
  `nslookup`     DNS lookup
  `ss -tulpn`    Listening ports and processes
  `ip -s link`   Interface statistics

------------------------------------------------------------------------

**Recommended Practice**

Run every command on your own machine, change one thing at a time, and
compare the output before and after. Understanding the output is more
valuable than memorizing the command syntax.

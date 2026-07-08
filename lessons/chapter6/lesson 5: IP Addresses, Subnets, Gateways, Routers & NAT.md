
# Chapter 06 — Networking Fundamentals

# Lesson 01 — How the Internet Actually Works
## Part 5 — IP Addresses, Subnets, Gateways, Routers & NAT

---

# Learning Objectives

After this lesson you will understand:

- What an IP address really does
- Why IP addresses are different from MAC addresses
- What a subnet is
- What a gateway is
- What a router does
- Why private IP addresses exist
- Why public IP addresses exist
- What NAT is
- How home, cloud, Docker and Kubernetes networking depend on routing

---

# Where We Are in the Journey

So far, our request looks like this:

```text
User
 ↓
Browser
 ↓
HTTP Request
 ↓
TCP Segment
 ↓
IP Packet
 ↓
Ethernet Frame
 ↓
Local Network
```

In Part 4 we learned how the frame reaches the **next local device** using MAC addresses and ARP.

Now we need to answer:

> How does traffic leave the local network and reach another network?

That is where IP addressing and routing become important.

---

# What Is an IP Address?

An **IP address** is a logical address used to identify a device on an IP network.

Example:

```text
192.168.1.20
```

IP addresses are used by routers to move packets between networks.

A simple way to remember it:

```text
MAC Address = local delivery
IP Address  = network-to-network delivery
```

MAC addresses help inside one local network.

IP addresses help across many networks.

---

# IP Address as a Location

Think about a delivery system.

A MAC address is like a local door identifier.

An IP address is more like:

```text
Country → City → Street → Building
```

It helps routers decide which direction the packet should travel.

---

# IPv4

IPv4 addresses look like this:

```text
192.168.1.20
```

They are made of four numbers separated by dots.

Each number ranges from:

```text
0 to 255
```

Examples:

```text
10.0.0.5
172.16.10.20
192.168.1.50
8.8.8.8
1.1.1.1
```

IPv4 is still heavily used in Linux, cloud, Docker and Kubernetes networking.

---

# IPv6

IPv6 addresses look like this:

```text
2001:db8::1
```

IPv6 was created because IPv4 does not have enough addresses for the modern Internet.

IPv6 has a much larger address space.

For this chapter, we will focus mostly on IPv4 first because it is still the most common starting point for Linux and DevOps troubleshooting.

---

# Network and Host Parts

An IP address has two logical parts:

```text
Network Part + Host Part
```

Example:

```text
192.168.1.20/24
```

This usually means:

```text
Network: 192.168.1.0
Host:    .20
```

The `/24` tells us how much of the address belongs to the network part.

This notation is called **CIDR**.

---

# What Is CIDR?

CIDR means:

```text
Classless Inter-Domain Routing
```

But don't worry about the name first.

Practically, CIDR tells us the size of the network.

Example:

```text
192.168.1.20/24
```

The `/24` means the first 24 bits represent the network.

In common beginner terms:

```text
192.168.1.0/24
```

contains addresses like:

```text
192.168.1.1
192.168.1.2
192.168.1.3
...
192.168.1.254
```

Devices in this range are usually considered part of the same subnet.

---

# What Is a Subnet?

A **subnet** is a smaller logical network inside a bigger network.

Example:

```text
192.168.1.0/24
```

This subnet may contain devices like:

```text
192.168.1.10  Laptop
192.168.1.20  Server
192.168.1.30  Printer
192.168.1.1   Router
```

These devices can usually communicate locally.

---

# Why Subnets Exist

Subnets help organize networks.

Without subnets, networks would become chaotic and inefficient.

Subnets allow engineers to separate:

- Office devices
- Servers
- Databases
- Kubernetes nodes
- Public-facing systems
- Private internal systems

Example production layout:

```text
10.0.1.0/24   Public subnet
10.0.2.0/24   Private application subnet
10.0.3.0/24   Database subnet
```

This is exactly how cloud networking is often designed.

---

# Same Subnet vs Different Subnet

If two devices are in the same subnet, they can usually communicate directly using local delivery.

Example:

```text
Laptop: 192.168.1.10/24
Server: 192.168.1.20/24
```

Both are in:

```text
192.168.1.0/24
```

So the laptop can use ARP to find the server's MAC address.

---

If the destination is outside the subnet:

```text
Laptop: 192.168.1.10/24
Google: 8.8.8.8
```

The laptop cannot ARP for Google.

Instead, it sends the packet to the default gateway.

---

# What Is a Gateway?

A **gateway** is the device used to reach other networks.

In most home networks, the gateway is your router.

Example:

```text
Laptop IP:      192.168.1.10
Gateway IP:     192.168.1.1
Destination IP: 8.8.8.8
```

The laptop says:

> 8.8.8.8 is not in my subnet, so I will send this packet to my gateway.

---

# Default Gateway

The **default gateway** is used when the system does not have a more specific route.

You can think of it as:

> If you don't know where to send this packet, send it here.

On Linux, you can later see it with:

```bash
ip route
```

Example output:

```text
default via 192.168.1.1 dev wlan0
```

Meaning:

```text
For unknown external destinations, send traffic to 192.168.1.1 using wlan0.
```

---

# What Is a Router?

A router connects different networks.

Example:

```text
192.168.1.0/24
      │
      │
   Router
      │
      │
Internet
```

A router receives packets and decides where to forward them next.

Routers mainly care about IP addresses, not HTTP content.

---

# Switch vs Router

A switch connects devices inside the same local network.

A router connects different networks.

```text
Same Network:

Laptop ── Switch ── Server
```

```text
Different Networks:

Home Network ── Router ── Internet
```

Switches use MAC addresses.

Routers use IP addresses.

---

# Routing Table

Every Linux machine has a routing table.

The routing table answers:

> Where should packets go?

Example:

```text
Destination        Next Hop
192.168.1.0/24     directly connected
default            192.168.1.1
```

Meaning:

- For local network traffic, send directly.
- For everything else, send to the gateway.

Later you will inspect this with:

```bash
ip route
```

---

# Packet Journey Outside the Local Network

Suppose your laptop wants to reach:

```text
8.8.8.8
```

Your laptop checks:

```text
Is 8.8.8.8 inside my subnet?
```

No.

So it sends the packet to the default gateway.

```text
Laptop
  │
  ▼
Default Gateway
  │
  ▼
ISP Router
  │
  ▼
Internet
  │
  ▼
8.8.8.8
```

At each router, the packet moves closer to the destination.

---

# Important Detail

The IP destination usually stays the same:

```text
Destination IP: 8.8.8.8
```

But the Layer 2 MAC destination changes hop by hop.

```text
Hop 1: Laptop → Home Router
Hop 2: Home Router → ISP Router
Hop 3: ISP Router → Next Router
...
```

This is why understanding both IP and MAC is essential.

---

# Public IP Addresses

A **public IP address** is reachable on the Internet.

Examples:

```text
8.8.8.8
1.1.1.1
```

Public IPs must be globally unique.

If two Internet-facing systems had the same public IP, routing would break.

---

# Private IP Addresses

Private IP addresses are used inside internal networks.

Common private ranges:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Examples:

```text
192.168.1.10
10.0.2.15
172.16.5.20
```

These addresses are not directly reachable from the public Internet.

---

# Why Private IPs Exist

There are not enough IPv4 addresses for every device in the world to have a unique public IP.

Private IPs allow many internal devices to share network space.

Example:

```text
Home Network
 ├── Laptop       192.168.1.10
 ├── Phone        192.168.1.11
 ├── Printer      192.168.1.12
 └── Smart TV     192.168.1.13
```

All of them may share one public IP when accessing the Internet.

---

# The Problem Private IPs Create

Private IPs are reusable.

Your laptop may have:

```text
192.168.1.10
```

My laptop may also have:

```text
192.168.1.10
```

That is fine because they are in different private networks.

But the public Internet cannot route directly to both of them.

So how do private devices access the Internet?

That is where NAT comes in.

---

# What Is NAT?

NAT means:

```text
Network Address Translation
```

NAT changes IP address information as traffic passes through a router.

In home networks, NAT usually translates:

```text
Private IP → Public IP
```

Example:

```text
192.168.1.10 → 203.0.113.5
```

The website sees the router's public IP, not your laptop's private IP.

---

# NAT Example

Your laptop sends a request:

```text
Source IP:      192.168.1.10
Destination IP: 8.8.8.8
```

Your router changes it to:

```text
Source IP:      Router Public IP
Destination IP: 8.8.8.8
```

When the response comes back, the router remembers which internal device started the connection and sends the response back to your laptop.

---

# PAT / Port Address Translation

Many devices can share one public IP because the router tracks connections using ports.

Example:

```text
Laptop      192.168.1.10:51000
Phone       192.168.1.11:51001
Smart TV    192.168.1.12:51002
```

All may appear externally as:

```text
203.0.113.5
```

But with different port mappings.

This is often called PAT or NAT overload.

---

# NAT Table Simplified

The router may keep a table like this:

```text
Internal Device          Public Mapping
192.168.1.10:51000  →    203.0.113.5:40001
192.168.1.11:51001  →    203.0.113.5:40002
192.168.1.12:51002  →    203.0.113.5:40003
```

When responses return, the router knows where to send them internally.

---

# DevOps Production Example

Cloud environments use similar ideas.

Example AWS-style design:

```text
Internet
   │
Internet Gateway
   │
Public Subnet
   │
Load Balancer
   │
Private Subnet
   │
Application Server
   │
Private Database Subnet
   │
PostgreSQL
```

The application server may have a private IP only.

It may reach the Internet through a NAT Gateway.

The database should usually not have a public IP.

---

# Why Databases Usually Stay Private

A production database should normally not be directly reachable from the Internet.

Better design:

```text
Internet
   │
Load Balancer
   │
Application Server
   │
Database
```

Only the application talks to the database.

The public Internet talks only to the load balancer.

This reduces attack surface.

---

# Docker Networking Connection

Docker commonly creates private networks.

Example:

```text
Container IP: 172.17.0.2
Host IP:      192.168.1.10
```

The container may access the Internet through NAT performed by the Docker host.

Simplified:

```text
Container
   │
Docker Bridge
   │
Host NAT
   │
Router
   │
Internet
```

Understanding NAT makes Docker networking much less confusing.

---

# Kubernetes Networking Connection

Kubernetes also uses IP routing concepts heavily.

Example:

```text
Pod IP
  │
Node Network
  │
Cluster Network
  │
Service
  │
Other Pod
```

Kubernetes hides much of the routing complexity, but under the hood:

- Pods have IPs
- Nodes have IPs
- Services have virtual IPs
- Traffic is routed or translated
- Network policies may allow or deny communication

The fundamentals remain the same.

---

# Common Mistakes

❌ Thinking private IPs are unique across the world.

❌ Thinking private IPs are directly reachable from the Internet.

❌ Confusing a switch with a router.

❌ Forgetting that the default gateway is required to leave the subnet.

❌ Thinking NAT is only a home router concept.

❌ Exposing databases publicly without a strong reason.

---

# Best Practices for DevOps Engineers

- Always know whether an IP is public or private.
- Always check the subnet before assuming routing is needed.
- Always verify the default gateway when Internet access fails.
- Keep databases and internal services on private networks.
- Use public IPs only where necessary.
- Understand NAT before troubleshooting Docker, cloud or Kubernetes networking.
- Draw the request path before debugging complex network issues.

---

# Troubleshooting Mindset

When a server cannot reach another system, ask:

1. Does it have an IP address?
2. Is the destination in the same subnet?
3. If not, does it have a default gateway?
4. Does the routing table know where to send traffic?
5. Is NAT involved?
6. Is a firewall or security group blocking traffic?
7. Is the destination service actually listening?

This order prevents random troubleshooting.

---

# Linux Command Preview

Later you will use:

```bash
ip addr
```

To inspect IP addresses.

```bash
ip route
```

To inspect routes and gateways.

```bash
ping
```

To test reachability.

```bash
traceroute
```

To inspect the path traffic takes.

```bash
ss -tulnp
```

To inspect listening ports.

---

# Cheat Sheet

| Concept | Meaning |
|--------|---------|
| IP Address | Logical network address |
| IPv4 | Common IP format like 192.168.1.10 |
| IPv6 | Newer IP format with much larger address space |
| Subnet | Smaller logical network |
| CIDR | Notation showing network size, like /24 |
| Gateway | Device used to reach other networks |
| Default Gateway | Route used when no more specific route exists |
| Router | Device that forwards packets between networks |
| Public IP | Globally reachable Internet IP |
| Private IP | Internal IP not directly reachable from Internet |
| NAT | Translates private IPs to public IPs |
| PAT | Uses ports so many devices can share one public IP |

---

# Interview Questions

## 1. What is the difference between a MAC address and an IP address?

A MAC address is used for local delivery on a local network. An IP address is used for logical addressing and routing between networks.

---

## 2. What is a subnet?

A subnet is a logical subdivision of a network.

---

## 3. What is a default gateway?

A default gateway is the router used when the destination is outside the local subnet and no more specific route exists.

---

## 4. Why do private IP addresses exist?

They allow internal networks to use reusable address ranges and reduce the need for every device to have a public IP address.

---

## 5. What does NAT do?

NAT translates private IP addresses to public IP addresses, allowing private devices to access external networks.

---

## 6. Why should databases usually not have public IP addresses?

Because databases should normally be reachable only by trusted internal application services, reducing exposure to attacks.

---

# Summary

In this lesson you learned:

- IP addresses are used for logical network communication.
- Subnets define which devices are local to each other.
- Gateways allow devices to reach other networks.
- Routers forward packets between networks.
- Public IPs are globally reachable.
- Private IPs are used internally.
- NAT allows private devices to access the Internet.
- Docker, Kubernetes and cloud networking rely heavily on these same concepts.

---

# Next Part

In Part 6 we will learn:

- Ports
- Sockets
- TCP
- UDP
- How applications share one IP address
- How Linux maps network traffic to processes
- The complete request walkthrough from browser to production service

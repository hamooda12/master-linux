
# Chapter 06 — Networking Fundamentals

# Lesson 01 — How the Internet Actually Works
## Part 4 — Network Interfaces, MAC Addresses, Switches & ARP

---

# Learning Objectives

After this lesson you will understand:

- What a network interface is
- Why every network interface has a MAC address
- Why we need MAC addresses if IP addresses already exist
- How devices communicate inside the same local network
- What switches do
- How switches learn where devices are
- What ARP is
- How your laptop discovers the MAC address of the default gateway

---

# The Big Question

In Part 3 we learned that data is wrapped like this:

```text
HTTP Request
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
Network
```

Now we need to answer a very important question:

> If the packet already has a destination IP address, why do we also need a MAC address?

This is one of the most important networking questions for DevOps engineers.

---

# Network Interface

A **network interface** is the connection point between a machine and a network.

Examples:

- Ethernet card
- Wi-Fi adapter
- Virtual machine adapter
- Docker bridge interface
- VPN interface
- Kubernetes CNI interface

On Linux, interfaces may look like:

```text
eth0
ens33
enp0s3
wlan0
docker0
vethabc123
```

The interface is where network traffic enters and leaves the machine.

---

# Physical and Virtual Interfaces

Not every network interface is physical.

A laptop Wi-Fi card is physical.

But Docker and Kubernetes create virtual interfaces.

Example:

```text
Linux Host
   │
   ├── eth0       → physical network
   ├── docker0    → Docker bridge
   ├── veth123    → container connection
   └── tun0       → VPN tunnel
```

This is why DevOps engineers must understand interfaces deeply.

Containers, VMs, VPNs and cloud networking all depend on virtual interfaces.

---

# MAC Address

A **MAC address** identifies a network interface on the local network.

Example:

```text
00:1A:2B:3C:4D:5E
```

A MAC address belongs to the **Data Link Layer**.

It is used for local delivery.

That means:

> MAC addresses are used to move frames inside the local network.

---

# IP Address vs MAC Address

An IP address answers:

> Where is the destination network or host?

A MAC address answers:

> Which local network interface should receive this frame next?

Think of it like this:

```text
IP Address  = city + street + building
MAC Address = exact door inside the current local area
```

The IP address helps with long-distance routing.

The MAC address helps with local delivery.

---

# Why IP Is Not Enough

Imagine your laptop wants to reach:

```text
8.8.8.8
```

That IP address is not on your local network.

Your laptop cannot send the frame directly to Google's server.

Instead, it sends the frame to the **default gateway** — usually your router.

So your laptop needs:

```text
Destination IP: 8.8.8.8
Destination MAC: Router's MAC address
```

Important:

The IP destination remains Google's IP.

But the MAC destination is the next local device.

---

# The Next Hop Idea

Networking often works hop by hop.

Example:

```text
Laptop
  │
  ▼
Home Router
  │
  ▼
ISP Router
  │
  ▼
Internet Router
  │
  ▼
Google Server
```

The IP destination may stay the same:

```text
8.8.8.8
```

But the MAC address changes at every local hop.

Each local network only needs to know the next device.

---

# Local Network Communication

Imagine two devices on the same network:

```text
Laptop A
IP: 192.168.1.10
MAC: AA:AA:AA:AA:AA:AA

Laptop B
IP: 192.168.1.20
MAC: BB:BB:BB:BB:BB:BB
```

If Laptop A wants to talk to Laptop B, it sends a frame directly to Laptop B's MAC address.

```text
Source MAC:      AA:AA:AA:AA:AA:AA
Destination MAC: BB:BB:BB:BB:BB:BB
Source IP:       192.168.1.10
Destination IP:  192.168.1.20
```

No router is needed because both devices are on the same local network.

---

# Communication Outside the Local Network

Now Laptop A wants to reach:

```text
142.250.200.14
```

That IP is outside the local network.

Laptop A sends the frame to the router.

```text
Source MAC:      Laptop MAC
Destination MAC: Router MAC
Source IP:       192.168.1.10
Destination IP:  142.250.200.14
```

The destination IP is the real target.

The destination MAC is only the next local hop.

This difference is critical.

---

# What Is a Switch?

A switch connects devices inside a local network.

Example:

```text
          Switch
        /   |    \
       /    |     \
 Laptop   Server   Printer
```

A switch works mainly with MAC addresses.

It forwards Ethernet frames to the correct device.

---

# How a Switch Learns

A switch builds a MAC address table.

Example:

```text
MAC Address              Port
AA:AA:AA:AA:AA:AA        Port 1
BB:BB:BB:BB:BB:BB        Port 2
CC:CC:CC:CC:CC:CC        Port 3
```

The switch learns by watching traffic.

When a frame arrives, the switch looks at the source MAC address and remembers:

> This MAC address lives on this port.

No one usually configures this manually.

The switch learns automatically.

---

# What Happens If the Switch Does Not Know the Destination?

If the switch does not know where a MAC address is, it floods the frame.

That means it sends the frame out multiple ports except the one it came from.

Example:

```text
Unknown destination MAC

Switch sends frame to:
- Port 2
- Port 3
- Port 4
```

When the correct device responds, the switch learns its MAC address.

---

# What Is ARP?

ARP stands for:

```text
Address Resolution Protocol
```

ARP answers this question:

> I know the IP address, but what is the MAC address?

Example:

```text
Who has 192.168.1.1?
Tell 192.168.1.10.
```

The router replies:

```text
192.168.1.1 is at 11:22:33:44:55:66
```

Now the laptop can send Ethernet frames to the router.

---

# ARP Example

Your laptop has:

```text
IP: 192.168.1.10
Gateway: 192.168.1.1
```

It wants to access:

```text
https://example.com
```

The server is outside your local network.

So your laptop needs to send traffic to the gateway.

But it only knows:

```text
Gateway IP: 192.168.1.1
```

It still needs:

```text
Gateway MAC: ?
```

So it sends an ARP request.

```text
Laptop → Broadcast:
Who has 192.168.1.1?
```

Router replies:

```text
Router → Laptop:
192.168.1.1 is at 11:22:33:44:55:66
```

Now the laptop can send frames to the router.

---

# Broadcast

An ARP request is usually sent as a broadcast.

Broadcast means:

> Send this message to everyone on the local network.

The broadcast MAC address is:

```text
ff:ff:ff:ff:ff:ff
```

Only the device with the requested IP should answer.

---

# ARP Cache

After learning a MAC address, the operating system stores it temporarily in the ARP cache.

This avoids asking the same question repeatedly.

Example:

```text
IP Address       MAC Address
192.168.1.1      11:22:33:44:55:66
192.168.1.20     BB:BB:BB:BB:BB:BB
```

Later in Linux, you can inspect neighbor information using:

```bash
ip neigh
```

---

# Complete Local Delivery Example

A browser wants to reach a public website.

```text
Browser wants:
https://example.com
```

After DNS, the browser knows:

```text
Destination IP: 93.184.216.34
```

The operating system sees that this IP is outside the local network.

So it chooses the default gateway:

```text
Gateway IP: 192.168.1.1
```

Then it checks the ARP cache.

If the router MAC is missing, it sends ARP.

After ARP succeeds, the Ethernet frame is created:

```text
Ethernet Frame
  Source MAC:      Laptop MAC
  Destination MAC: Router MAC

IP Packet
  Source IP:       Laptop IP
  Destination IP:  Website IP
```

This is how your request leaves your local network.

---

# Very Important DevOps Rule

For traffic going outside your local network:

```text
Destination IP  = final server
Destination MAC = next hop/router
```

This explains many confusing networking problems.

---

# Production Troubleshooting Example

Problem:

```text
Server cannot reach the Internet.
```

Possible checks:

```bash
ip addr
ip route
ip neigh
ping 192.168.1.1
```

What are we checking?

- Does the server have an interface?
- Does it have an IP address?
- Does it have a default gateway?
- Can it resolve the gateway's MAC address?
- Can it reach the gateway?

If ARP fails, the server may never even reach the router.

---

# How This Applies to Cloud

In cloud environments, you may not see a physical switch.

But the concepts still exist.

Cloud networking uses virtual versions of the same ideas:

```text
VM / EC2 Instance
      │
Virtual Network Interface
      │
Virtual Switch
      │
Virtual Router
      │
VPC / Internet Gateway
```

AWS, Azure and GCP hide much of the physical hardware, but the logic remains.

---

# How This Applies to Docker

Docker creates virtual interfaces.

Example:

```text
Container
   │
veth pair
   │
docker0 bridge
   │
host eth0
   │
network
```

The container has its own IP.

Traffic moves through virtual interfaces.

Docker networking is much easier once you understand interfaces and local delivery.

---

# How This Applies to Kubernetes

Kubernetes networking also depends on virtual interfaces.

Example:

```text
Pod
 │
veth
 │
CNI bridge / overlay
 │
Node network
 │
Other Pod or Service
```

Later, when we study Kubernetes networking, concepts like MAC, IP, routing and virtual interfaces will return.

---

# Common Mistakes

❌ Thinking MAC addresses route traffic across the Internet.

❌ Thinking IP addresses are enough for local delivery.

❌ Thinking switches use IP addresses like routers.

❌ Forgetting that destination MAC changes hop by hop.

❌ Forgetting that ARP only works inside the local network.

---

# Best Practices for DevOps Engineers

- Always identify the interface involved.
- Always distinguish local network problems from routing problems.
- Check the default gateway when Internet access fails.
- Check ARP/neighbor entries when local delivery fails.
- Remember that Docker and Kubernetes create virtual interfaces.
- Do not treat networking as magic just because it is virtualized in cloud environments.

---

# Interview Questions

## 1. Why do we need MAC addresses if we already have IP addresses?

Because IP addresses identify the final logical destination, while MAC addresses are used for local delivery on the current network segment.

---

## 2. What does ARP do?

ARP maps an IP address to a MAC address on the local network.

---

## 3. Does ARP work across the Internet?

No. ARP works only inside the local network or broadcast domain.

---

## 4. What does a switch use to forward frames?

A switch uses MAC addresses.

---

## 5. What happens when a switch does not know a destination MAC?

It floods the frame out multiple ports, except the port where the frame came from.

---

## 6. When sending traffic to a public IP, what is the destination MAC address?

The destination MAC is the MAC address of the next hop, usually the default gateway.

---

# Mini Lab Preview

In the Linux command lessons, you will inspect these concepts using:

```bash
ip link
ip addr
ip route
ip neigh
```

Example:

```bash
ip link
```

Shows interfaces.

```bash
ip addr
```

Shows IP addresses.

```bash
ip route
```

Shows routing table and default gateway.

```bash
ip neigh
```

Shows ARP/neighbor cache.

---

# Cheat Sheet

| Concept | Meaning |
|--------|---------|
| Interface | Connection point to a network |
| MAC Address | Local network interface identifier |
| Switch | Connects devices in a local network |
| Frame | Data link layer unit |
| ARP | Maps IP address to MAC address |
| Broadcast | Message sent to all local devices |
| Default Gateway | Router used to leave the local network |
| ARP Cache | Temporary table of IP-to-MAC mappings |

---

# Summary

In this lesson you learned:

- A network interface connects a machine to a network.
- MAC addresses identify interfaces locally.
- IP addresses identify logical destinations.
- Switches forward frames using MAC addresses.
- ARP discovers MAC addresses from IP addresses.
- ARP works only inside the local network.
- Traffic outside the local network is sent to the default gateway.
- Docker, Kubernetes and cloud networking use virtual versions of these same ideas.

---

# Next Part

In Part 5 we will learn:

- IP addresses
- Subnets
- Gateways
- Routers
- Public vs private IPs
- NAT
- How traffic moves beyond the local network

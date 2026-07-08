# Chapter 06 — Networking Fundamentals

# Lesson 13 — Routing Basics for DevOps Engineers

> **Objective:** Understand how Linux decides where packets go, how routing tables work, what a default gateway really does, and how to troubleshoot routing problems in production.

---

# Why Routing Exists

Imagine your server has the IP:

```text
192.168.1.20/24
```

It wants to send data to:

```text
8.8.8.8
```

How does Linux know where to send the packet?

The answer is **routing**.

A routing table is simply a list of rules that tells the Linux kernel:

> "For this destination network, use this interface and (if needed) this next hop."

---

# The Journey of an Outgoing Packet

```text
Application
      │
      ▼
Socket
      │
      ▼
Linux Kernel
      │
      ▼
Destination IP?
      │
      ▼
Routing Table Lookup
      │
      ├── Local Network ──► Send directly
      │
      └── Remote Network ─► Send to Default Gateway
```

Linux always checks the routing table before transmitting a packet.

---

# Local Network vs Remote Network

Suppose:

```text
Server IP : 192.168.1.20/24
```

These are local:

```text
192.168.1.50
192.168.1.100
192.168.1.200
```

This is remote:

```text
10.0.0.5
```

And this is remote:

```text
8.8.8.8
```

Local traffic is sent directly after ARP resolution.

Remote traffic is forwarded to the default gateway.

---

# The Default Gateway

Think of your local network as a neighborhood.

If the destination house is inside the neighborhood:

You drive directly there.

If the destination is in another city:

You first drive to the highway entrance.

The **default gateway** is that highway entrance.

Example:

```text
default via 192.168.1.1 dev enp0s3
```

Meaning:

Unknown destinations are sent to `192.168.1.1` using interface `enp0s3`.

---

# Inspecting the Routing Table

Command:

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev enp0s3 proto dhcp metric 100
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.20
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
```

---

# Understanding Each Route

## Default Route

```text
default via 192.168.1.1 dev enp0s3
```

Used for every destination that doesn't match a more specific route.

---

## Connected Route

```text
192.168.1.0/24 dev enp0s3
```

The kernel knows this network is directly attached.

No router required.

---

## Docker Route

```text
172.17.0.0/16 dev docker0
```

Traffic to Docker containers stays on the Docker bridge.

---

# Longest Prefix Match

Linux may have multiple matching routes.

It always chooses the **most specific** route.

Example:

```text
default via 192.168.1.1
192.168.1.0/24 dev enp0s3
192.168.1.50/32 dev special0
```

Destination:

```text
192.168.1.50
```

Linux chooses:

```text
192.168.1.50/32
```

because `/32` is more specific than `/24`.

This rule is called **Longest Prefix Match**.

---

# Route Metrics

Sometimes two routes reach the same destination.

Linux uses a **metric**.

Lower metric = preferred route.

Example:

```text
default via 192.168.1.1 metric 100
default via 10.0.0.1 metric 200
```

Linux prefers the first route.

---

# Adding a Temporary Route

Example:

```bash
sudo ip route add 10.10.0.0/16 via 192.168.1.254
```

Delete it:

```bash
sudo ip route del 10.10.0.0/16
```

These changes are temporary and disappear after reboot unless saved by your network configuration system.

---

# Tracing the Path

Useful commands:

```bash
tracepath 8.8.8.8
traceroute 8.8.8.8
```

These show the routers (hops) a packet traverses.

Example:

```text
Server
  │
Gateway
  │
ISP Router
  │
Regional Router
  │
Google Network
```

---

# Production Scenario 1 — No Internet

Symptoms:

```text
Local LAN works.
Internet does not.
```

Checklist:

```bash
ip addr
ip route
ping <gateway>
ping 8.8.8.8
```

If the default route is missing, remote traffic cannot leave the server.

---

# Production Scenario 2 — Wrong Gateway

Expected:

```text
default via 192.168.1.1
```

Actual:

```text
default via 192.168.2.1
```

The server sends packets to a gateway that is not reachable.

Result:

No internet connectivity.

---

# Production Scenario 3 — Docker Networking

Your application cannot reach a container.

Check:

```bash
ip route
docker network ls
docker inspect <container>
```

Verify the Docker route exists and points to `docker0` or the correct bridge.

---

# Troubleshooting Workflow

```text
1. Check interface
        │
2. Check IP address
        │
3. Check routing table
        │
4. Check default gateway
        │
5. Ping gateway
        │
6. Ping remote IP
        │
7. Test DNS
        │
8. Test application
```

---

# Hands-On Lab

Run:

```bash
ip route
```

Answer:

- What is your default gateway?
- Which interface is used?
- Are there Docker routes?
- Which routes are directly connected?

Run:

```bash
tracepath 8.8.8.8
```

Observe how packets move through multiple routers.

---

# Common Mistakes

- Assuming every destination is directly reachable.
- Deleting the default route accidentally.
- Forgetting that Docker adds routes.
- Ignoring route metrics.
- Confusing routing issues with DNS problems.

---

# Cheat Sheet

```bash
ip route
ip route show
ip route add <network> via <gateway>
ip route del <network>
tracepath 8.8.8.8
traceroute 8.8.8.8
ping <gateway>
```

---

# Interview Questions

1. What is a routing table?
2. What is a default gateway?
3. What is Longest Prefix Match?
4. What is a route metric?
5. Why is a directly connected route different from a default route?
6. How do you inspect the routing table?
7. How would you troubleshoot a missing default route?

---

# Summary

Routing is the decision-making process that tells Linux where every packet should go. Understanding routing tables, gateways, and route selection is essential for troubleshooting production systems, cloud networking, Docker, Kubernetes, and load balancers.

---

# Next Lesson

**Lesson 14 — Reverse Proxy & Load Balancer for DevOps Engineers**

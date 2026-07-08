# Chapter 06 — Networking Fundamentals

# Lesson 09 — The Complete Journey of an HTTPS Request

> **Objective:** Follow a single HTTPS request from a user's browser to a production application and back.

---

# Learning Objectives

By the end of this lesson you will be able to:

- Explain every major step of an HTTPS request.
- Connect DNS, ARP, TCP, IP, routing, NAT, and HTTP together.
- Understand where common production failures occur.
- Map each stage to Linux troubleshooting tools.

---

# Production Architecture

```text
User
 │
 ▼
Browser
 │
 ▼
DNS
 │
 ▼
Local Network
 │
 ▼
Default Gateway
 │
 ▼
ISP
 │
 ▼
Internet
 │
 ▼
Cloud Firewall
 │
 ▼
Load Balancer
 │
 ▼
Nginx
 │
 ▼
Spring Boot
 │
 ▼
PostgreSQL
```

---

# Step 1 — User Enters a URL

Example:

```text
https://mycompany.com
```

The browser has a domain name, not an IP address.

---

# Step 2 — DNS Resolution

DNS translates:

```text
mycompany.com
```

into:

```text
203.0.113.20
```

Now the browser knows the destination IP.

---

# Step 3 — Routing Decision

Linux checks:

- Is the destination on my local subnet?

If not, it chooses the default gateway.

---

# Step 4 — ARP

If the gateway MAC address is unknown:

```text
Who has 192.168.1.1?
```

The router replies with its MAC address.

---

# Step 5 — TCP Handshake

Before sending HTTP data:

```text
Client          Server

SYN  --------->

      <--------- SYN-ACK

ACK  --------->
```

The connection is now established.

---

# Step 6 — TLS Handshake

Because HTTPS is used:

- Certificates are exchanged.
- Encryption keys are negotiated.
- A secure channel is created.

Only then can encrypted HTTP data be exchanged.

---

# Step 7 — Encapsulation

The request is wrapped:

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
```

---

# Step 8 — Leaving the Local Network

The frame is sent to the default gateway.

Remember:

- Destination IP = Final server
- Destination MAC = Router

---

# Step 9 — Across the Internet

Routers forward the packet hop by hop until it reaches the cloud provider.

---

# Step 10 — Cloud Infrastructure

Typical flow:

```text
Firewall
   │
Load Balancer
   │
Nginx
   │
Spring Boot
```

The firewall filters traffic.

The load balancer selects a healthy backend.

---

# Step 11 — Application Processing

Spring Boot:

- Validates the request.
- Executes business logic.
- Queries PostgreSQL if needed.
- Builds the HTTP response.

---

# Step 12 — Response

The response travels back using the same networking principles until the browser renders the page.

---

# Linux Tools

| Stage | Tool |
|------|------|
| Interfaces | `ip link` |
| IP Address | `ip addr` |
| Routes | `ip route` |
| Neighbors | `ip neigh` |
| Connectivity | `ping` |
| Route Path | `traceroute` |
| DNS | `dig`, `nslookup` |
| Ports | `ss -tulnp` |
| HTTP | `curl` |
| Packets | `tcpdump` |

---

# Production Troubleshooting Flow

1. DNS
2. Routing
3. Gateway
4. Firewall
5. Load Balancer
6. Nginx
7. Application
8. Database

Always investigate in this order.

---

# Hands-On Lab

1. Run `ip addr`.
2. Run `ip route`.
3. Run `ip neigh`.
4. Run `ping 8.8.8.8`.
5. Run `dig example.com`.
6. Run `curl https://example.com`.
7. Run `ss -tulnp`.

---

# Interview Questions

1. Why is DNS required before TCP?
2. Why are both IP and MAC addresses needed?
3. Why does HTTPS require TLS?
4. What is the role of the load balancer?
5. Which Linux command shows listening ports?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| DNS | Domain → IP |
| ARP | IP → MAC |
| TCP | Reliable transport |
| TLS | Encryption |
| Router | Forward packets |
| Load Balancer | Distribute traffic |
| Nginx | Reverse proxy |
| Spring Boot | Business logic |
| PostgreSQL | Data storage |

---

# Summary

A single HTTPS request passes through many networking layers and infrastructure components before reaching the application. Understanding this complete flow allows DevOps engineers to troubleshoot production systems methodically.

---

# Next Lesson

**Lesson 10 — Linux Networking Toolkit (`ip`, `hostname`, `ping`, `ss`, `dig`, `curl`)**

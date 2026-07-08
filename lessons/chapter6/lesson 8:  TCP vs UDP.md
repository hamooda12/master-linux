
# Chapter 06 — Networking Fundamentals

# Lesson 08 — TCP vs UDP

> **Objective:** Understand why two transport protocols exist, how TCP guarantees reliable communication, when UDP is the better choice, and how DevOps engineers troubleshoot TCP/UDP issues.

---

# Learning Objectives

By the end of this lesson you will:

- Understand the role of the Transport Layer.
- Compare TCP and UDP.
- Learn the TCP three-way handshake.
- Understand reliability, ordering and retransmission.
- Know when UDP is preferable.
- Connect these concepts to Linux, Docker, Kubernetes and production systems.

---

# Why Do We Need Transport Protocols?

Imagine sending application data across the network.

IP can deliver packets to the correct machine, but it does **not** guarantee:

- Delivery
- Order
- Duplicate protection

Transport protocols solve these problems.

The two most common are:

- TCP
- UDP

---

# TCP

TCP stands for **Transmission Control Protocol**.

Its goal is reliability.

Features:

- Connection-oriented
- Reliable delivery
- Ordered packets
- Retransmission
- Flow control
- Congestion control

Typical uses:

- HTTPS
- SSH
- APIs
- PostgreSQL
- MySQL

---

# TCP Three-Way Handshake

Before sending data:

```text
Client                  Server

SYN  ------------------>

      <----------------  SYN-ACK

ACK  ------------------>

Connection Established
```

Only after this handshake does application data begin to flow.

---

# Why TCP Is Reliable

TCP numbers every byte it sends.

If data is lost:

```text
Packet 1 ✓
Packet 2 ✓
Packet 3 ✗
Packet 4 ✓
```

The receiver requests Packet 3 again.

Applications receive the data in the correct order.

---

# UDP

UDP stands for **User Datagram Protocol**.

UDP is much simpler.

Characteristics:

- No handshake
- No retransmission
- No ordering
- Lower overhead
- Faster

Typical uses:

- DNS
- Voice calls
- Video streaming
- Online gaming

---

# TCP vs UDP

| TCP | UDP |
|------|-----|
| Reliable | Best effort |
| Ordered | Unordered |
| Connection-oriented | Connectionless |
| Handshake | No handshake |
| Higher overhead | Lower overhead |

---

# Production Examples

## HTTPS

Browser → Nginx

Uses TCP because every byte matters.

---

## PostgreSQL

Application → Database

Uses TCP because missing or reordered data would corrupt communication.

---

## DNS

Most DNS queries use UDP because:

- Queries are small.
- Speed is important.
- A lost query can simply be retried.

---

# Docker & Kubernetes

Containers and Pods still use TCP and UDP.

Example:

```text
Browser
   │
TCP 443
   │
Ingress
   │
TCP 8080
   │
Pod
```

Or:

```text
Application
   │
UDP 53
   │
DNS Server
```

---

# Linux Tools

Useful commands:

```bash
ss -t
```

Show TCP sockets.

```bash
ss -u
```

Show UDP sockets.

```bash
ss -tulnp
```

Show listening TCP and UDP services.

---

# DevOps Troubleshooting

Questions to ask:

1. Is the application using TCP or UDP?
2. Is the correct port open?
3. Is the service listening?
4. Is the firewall blocking the protocol?
5. Is packet loss affecting TCP performance?

---

# Common Mistakes

- Assuming every service uses TCP.
- Forgetting that DNS commonly uses UDP.
- Thinking UDP is "bad" because it is unreliable.
- Using `ping` to test an application.

---

# Hands-on Lab

Run:

```bash
ss -t
```

Then:

```bash
ss -u
```

Finally:

```bash
ss -tulnp
```

Compare TCP and UDP sockets.

---

# Interview Questions

1. Why does TCP use a handshake?
2. Why is TCP considered reliable?
3. Why does DNS usually use UDP?
4. Give two applications that commonly use TCP.
5. Give two applications that commonly use UDP.

---

# Cheat Sheet

| Protocol | Best For |
|----------|----------|
| TCP | Reliable communication |
| UDP | Low-latency communication |
| SYN | Start a TCP connection |
| ACK | Acknowledge received data |
| Retransmission | Resend lost data |

---

# Summary

You learned:

- Why transport protocols exist.
- The differences between TCP and UDP.
- How the TCP handshake works.
- Why TCP guarantees reliability.
- Why UDP is useful for speed-sensitive applications.
- How Linux displays TCP and UDP sockets.

---

# Next Lesson

**Lesson 09 — The Complete Journey of an HTTPS Request**

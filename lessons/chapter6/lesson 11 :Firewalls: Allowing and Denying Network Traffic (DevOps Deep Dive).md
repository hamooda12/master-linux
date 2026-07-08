
# Chapter 06 — Networking Fundamentals

# Lesson 12 — Firewalls: Allowing & Denying Network Traffic

> **Objective:** Understand how firewalls control network traffic, why they are essential in production, and how Linux administrators and DevOps engineers troubleshoot firewall issues.

---

# Learning Objectives

By the end of this lesson you will:

- Understand what a firewall is.
- Explain inbound and outbound traffic.
- Understand allow and deny rules.
- Distinguish stateful and stateless firewalls.
- Know the role of Linux firewalls and cloud firewalls.
- Troubleshoot common firewall problems.

---

# What Is a Firewall?

A firewall is a security system that examines network traffic and decides whether to:

- Allow it
- Deny it

Think of a firewall as a security guard standing at the entrance of a building.

---

# Why Firewalls Exist

Without a firewall:

- Anyone could try to access your services.
- Internal services could be exposed accidentally.
- Attackers would have far fewer obstacles.

Firewalls reduce the attack surface.

---

# Inbound vs Outbound

## Inbound

Traffic entering your server.

Example:

```text
Internet
   │
   ▼
Web Server
```

Example rule:

```text
ALLOW TCP 443
```

---

## Outbound

Traffic leaving your server.

Example:

```text
Application Server
      │
      ▼
Database
```

Example rule:

```text
ALLOW TCP 5432
```

---

# Allow vs Deny

Example policy:

```text
ALLOW TCP 22
ALLOW TCP 443
DENY EVERYTHING ELSE
```

This is a common production configuration.

---

# Stateful Firewall

A stateful firewall remembers existing connections.

Example:

```text
Browser
   │
HTTPS Request
   ▼
Server
```

The response is automatically allowed because it belongs to an existing connection.

---

# Stateless Firewall

A stateless firewall evaluates every packet independently.

It does not remember previous traffic.

---

# Linux Firewalls

Common Linux firewall technologies:

- netfilter
- iptables
- nftables
- ufw (Ubuntu)

Later you will learn practical commands for each.

---

# Cloud Firewalls

Cloud providers offer firewall features too.

Examples:

- AWS Security Groups
- AWS Network ACLs
- Azure Network Security Groups
- Google Cloud Firewall Rules

These work together with the Linux firewall.

---

# Production Example

A web server is running correctly.

Nginx listens on:

```text
0.0.0.0:443
```

Users still cannot connect.

Possible causes:

- Linux firewall blocks TCP 443.
- Cloud security group blocks TCP 443.
- Corporate firewall blocks HTTPS.

Always verify the network path.

---

# Linux Commands Preview

Ubuntu UFW:

```bash
sudo ufw status
```

iptables:

```bash
sudo iptables -L
```

nftables:

```bash
sudo nft list ruleset
```

---

# DevOps Troubleshooting Checklist

1. Is the application running?
2. Is it listening on the correct port?
3. Is the Linux firewall allowing the traffic?
4. Is the cloud firewall allowing the traffic?
5. Is the client using the correct IP and port?

---

# Hands-On Lab

Run:

```bash
sudo ufw status
```

Then inspect listening services:

```bash
ss -tulnp
```

Compare open ports with firewall rules.

---

# Common Mistakes

- Opening more ports than necessary.
- Forgetting outbound rules.
- Assuming a listening service is automatically reachable.
- Ignoring cloud firewall rules.

---

# Cheat Sheet

| Concept | Meaning |
|---------|---------|
| Firewall | Filters traffic |
| Inbound | Traffic entering a host |
| Outbound | Traffic leaving a host |
| Allow | Permit traffic |
| Deny | Block traffic |
| Stateful | Remembers connections |
| Stateless | Treats each packet independently |

---

# Interview Questions

1. What is a firewall?
2. What is the difference between inbound and outbound traffic?
3. What is the difference between stateful and stateless firewalls?
4. Name two Linux firewall technologies.
5. Why can a service be listening but still unreachable?

---

# Summary

Firewalls control which network traffic is allowed or denied. Understanding firewall rules is essential for Linux administration, cloud infrastructure, Docker, Kubernetes, and production troubleshooting.

---

# Next Lesson

**Lesson 13 — Configuring Network Interfaces with the `ip` Command**

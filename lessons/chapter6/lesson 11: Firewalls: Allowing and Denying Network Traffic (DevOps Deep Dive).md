# Chapter 06 --- Networking Fundamentals

# Lesson 11 --- Firewalls: Allowing and Denying Network Traffic (DevOps Deep Dive)

> **Objective:** Understand how firewalls control network traffic, how
> Linux firewalls work, and how to troubleshoot connectivity problems in
> production environments.

------------------------------------------------------------------------

# Why Firewalls Matter

A firewall is a traffic filter.

It decides whether a packet is allowed to pass or should be blocked.

Think of a firewall as the security guard standing at the entrance of a
building.

Every packet asks:

``` text
May I enter?
```

The firewall checks its rules and either:

``` text
ALLOW
```

or

``` text
DENY
```

------------------------------------------------------------------------

# Where Can Firewalls Exist?

``` text
Internet
   │
Cloud Firewall (AWS Security Group)
   │
Router Firewall
   │
Linux Firewall (iptables/nftables/ufw)
   │
Docker Rules
   │
Application
```

A packet may pass several firewalls before reaching your application.

------------------------------------------------------------------------

# Stateful vs Stateless

## Stateless

Every packet is checked independently.

## Stateful (most Linux firewalls)

The firewall remembers connections.

Example:

``` text
Client → Server:443
```

Reply traffic is automatically allowed because it belongs to an existing
connection.

------------------------------------------------------------------------

# Inbound vs Outbound

Inbound = traffic entering your machine.

Examples:

-   SSH (22)
-   HTTP (80)
-   HTTPS (443)

Outbound = traffic leaving your machine.

Examples:

-   Database queries
-   API calls
-   DNS lookups

------------------------------------------------------------------------

# Default Policies

Three common strategies:

``` text
ALLOW ALL
```

Not secure.

``` text
DENY ALL
```

Very secure but nothing works until rules are added.

``` text
DENY BY DEFAULT + ALLOW REQUIRED PORTS
```

Production best practice.

------------------------------------------------------------------------

# Important Ports

  Port   Service
  ------ -------------
  22     SSH
  53     DNS
  80     HTTP
  443    HTTPS
  3306   MySQL
  5432   PostgreSQL
  6379   Redis
  8080   Spring Boot
  9090   Prometheus

------------------------------------------------------------------------

# Packet Journey

``` text
Internet
   │
Firewall
   │
Is port allowed?
   │
YES ─────► Application
NO  ─────► Packet Dropped
```

------------------------------------------------------------------------

# Linux Firewalls

Modern Linux uses **nftables**.

Older systems often use **iptables**.

Ubuntu commonly provides **ufw** as a simple interface.

Relationship:

``` text
ufw
 ↓
iptables / nftables
 ↓
Kernel
```

------------------------------------------------------------------------

# UFW Commands

Check status:

``` bash
sudo ufw status verbose
```

Allow HTTPS:

``` bash
sudo ufw allow 443/tcp
```

Allow SSH:

``` bash
sudo ufw allow ssh
```

Delete rule:

``` bash
sudo ufw delete allow 443/tcp
```

Enable firewall:

``` bash
sudo ufw enable
```

------------------------------------------------------------------------

# Viewing Listening Ports

``` bash
ss -tulpen
```

Important distinction:

Application listening ≠ firewall allowing.

------------------------------------------------------------------------

# Production Troubleshooting

## Incident 1

Users cannot open:

``` text
https://app.example.com
```

Checklist:

1.  DNS works?
2.  Server reachable?
3.  Port 443 listening?
4.  Firewall open?
5.  TLS certificate valid?
6.  Reverse proxy healthy?
7.  Backend healthy?

Commands:

``` bash
ss -tulpen
sudo ufw status
curl -vk https://localhost
journalctl -u nginx -n 50
```

------------------------------------------------------------------------

## Incident 2

SSH suddenly stopped working.

Possible causes:

-   Port 22 blocked
-   sshd stopped
-   Cloud firewall closed
-   Wrong IP restriction

Commands:

``` bash
systemctl status ssh
sudo ufw status
ss -tln | grep 22
```

------------------------------------------------------------------------

## Incident 3

Spring Boot works locally but not remotely.

Check:

-   App bound to 127.0.0.1 instead of 0.0.0.0
-   Port 8080 blocked
-   Reverse proxy misconfigured

------------------------------------------------------------------------

# Docker Considerations

Docker modifies firewall rules automatically.

Verify:

``` bash
docker ps
docker port <container>
ss -tulpen
```

Always confirm published ports and firewall rules match.

------------------------------------------------------------------------

# Kubernetes Considerations

Traffic may be filtered by:

-   NetworkPolicy
-   Service
-   Ingress
-   Cloud Load Balancer
-   Node Firewall

Debug layer by layer.

------------------------------------------------------------------------

# Best Practices

-   Deny by default.
-   Open only required ports.
-   Never expose databases publicly unless required.
-   Restrict SSH by IP when possible.
-   Regularly audit firewall rules.
-   Document every opened port.

------------------------------------------------------------------------

# Interview Questions

1.  What is a firewall?
2.  Difference between stateful and stateless firewalls?
3.  Difference between inbound and outbound rules?
4.  Why can an application still be unreachable if it is listening?
5.  Why is "deny by default" recommended?
6.  Difference between ufw, iptables and nftables?
7.  How would you troubleshoot "connection refused" vs "connection timed
    out"?

------------------------------------------------------------------------

# Cheat Sheet

``` bash
sudo ufw status verbose
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow ssh
sudo ufw delete allow 80/tcp
sudo ufw enable
sudo ufw disable

ss -tulpen
curl -v http://localhost
curl -vk https://localhost
journalctl -u nginx
systemctl status ssh
```

------------------------------------------------------------------------

# Summary

Firewalls are one of the first layers of defense in every production
environment. A DevOps engineer must understand where firewalls exist,
how they evaluate traffic, how Linux firewall tools work, and how to
distinguish firewall problems from application, DNS, or service
failures.

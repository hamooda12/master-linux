# Chapter 06 --- Networking Fundamentals

> **Goal:** Build a deep understanding of Linux networking from a DevOps
> perspective. This chapter emphasizes *how networking works*, not just
> which commands to run. Every lesson connects theory with production
> troubleshooting.

------------------------------------------------------------------------

# Learning Objectives

By the end of this chapter you will be able to:

-   Explain how hosts communicate on a network.
-   Understand IP addresses, subnets, gateways, DNS, and routing.
-   Inspect Linux network configuration.
-   Test connectivity and diagnose failures.
-   Investigate listening ports and services.
-   Understand firewall fundamentals.
-   Connect securely using SSH.
-   Troubleshoot common production networking incidents.

------------------------------------------------------------------------

# Roadmap

## Lesson 1 --- Networking Fundamentals

-   What is a computer network?
-   LAN, WAN, Internet
-   Client vs Server
-   Packets
-   Network interfaces
-   MAC Address
-   IP Address (IPv4 & IPv6)
-   Public vs Private IP
-   Loopback (`127.0.0.1`)
-   Hostname
-   Ports
-   TCP vs UDP
-   OSI model (practical)
-   TCP/IP model

## Lesson 2 --- Viewing Network Configuration

Commands: - `ip addr` - `ip link` - `ip route` - `hostname` -
`hostnamectl`

Concepts: - Interfaces - Link state - Routes - Default gateway

## Lesson 3 --- DNS

Commands: - `dig` - `nslookup` - `host` - `resolvectl`

Files: - `/etc/hosts` - `/etc/resolv.conf`

Concepts: - Domain names - Name resolution - DNS cache - Recursive
lookup

## Lesson 4 --- Connectivity Testing

Commands: - `ping` - `curl` - `wget`

Concepts: - ICMP - HTTP vs HTTPS - Latency - Packet loss - Timeouts

## Lesson 5 --- Listening Ports & Sockets

Commands: - `ss` - `netstat` (legacy) - `lsof`

Concepts: - Listening sockets - Established connections - Bound
address - `0.0.0.0` vs `127.0.0.1`

## Lesson 6 --- Firewall Basics

Commands: - `ufw status` - `ufw allow` - `ufw deny` - `ufw delete`

Concepts: - Inbound vs outbound rules - Allow vs deny - Least privilege

## Lesson 7 --- SSH

Commands: - `ssh` - `scp` - `ssh-keygen`

Concepts: - Password vs key authentication - Host keys - Authorized keys

## Lesson 8 --- Production Troubleshooting

Investigate scenarios such as:

-   Service unreachable
-   Wrong listening address
-   DNS failure
-   Incorrect gateway
-   Closed firewall port
-   Backend reachable locally but not remotely
-   SSH connection failures

------------------------------------------------------------------------

# Teaching Style

Each lesson will follow the same structure:

1.  Theory
2.  Why the concept exists
3.  Real-world examples
4.  ASCII diagrams
5.  Command explanations
6.  Hands-on lab
7.  Common mistakes
8.  Best practices
9.  Interview questions
10. Summary
11. Cheat sheet updates

------------------------------------------------------------------------

# Practical Bootcamp

After completing the lessons, we'll complete a production-style bootcamp
with progressively harder networking incidents, following the same
investigation-first methodology used in previous chapters.

------------------------------------------------------------------------

# Expected Outcome

By the end of Chapter 06 you should be able to:

-   Read and understand Linux network configuration.
-   Explain how packets travel between machines.
-   Diagnose connectivity problems methodically.
-   Debug DNS issues.
-   Verify application reachability.
-   Interpret `ss` output confidently.
-   Understand networking concepts required for Linux administration,
    DevOps, cloud platforms, containers, and Kubernetes.

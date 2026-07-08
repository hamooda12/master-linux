
# Chapter 06 — Networking Fundamentals

# Lesson 07 — Ports & Sockets

> **Objective:** Understand how one machine can run many network services at the same time, how Linux connects traffic to the correct process, and why ports and sockets are essential for DevOps troubleshooting.

---

# 1. Why This Lesson Matters

In the previous lessons, we learned how traffic reaches the correct machine.

But reaching the machine is not enough.

A single Linux server may run many services:

```text
Linux Server
 ├── SSH
 ├── Nginx
 ├── Spring Boot API
 ├── PostgreSQL
 └── Redis
```

All of these services may use the same IP address.

So the big question is:

> **When traffic reaches the server, how does Linux know which application should receive it?**

The answer is:

```text
Ports
```

---

# 2. IP Address vs Port

An IP address identifies the machine.

A port identifies the service running on that machine.

Think of it like this:

```text
IP Address = Building address
Port       = Apartment number
```

Example:

```text
203.0.113.10:443
```

Means:

```text
Server IP: 203.0.113.10
Service:   Port 443
```

Usually, port `443` means HTTPS.

---

# 3. What Is a Port?

A port is a logical number used by the operating system to identify a network service.

Ports range from:

```text
0 to 65535
```

Examples:

| Service | Port |
|---|---:|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Redis | 6379 |

When a service starts, it may listen on a port.

Example:

```text
Nginx listens on port 80 and 443.
PostgreSQL listens on port 5432.
SSH listens on port 22.
```

---

# 4. Listening Port

A service that is waiting for connections is said to be **listening**.

Example:

```text
Nginx listening on 0.0.0.0:80
```

Means:

```text
Nginx accepts HTTP connections on port 80 from all network interfaces.
```

Example:

```text
PostgreSQL listening on 127.0.0.1:5432
```

Means:

```text
PostgreSQL accepts connections only from the same machine.
```

This difference is very important.

---

# 5. 127.0.0.1 vs 0.0.0.0

## 127.0.0.1

```text
127.0.0.1
```

Means:

```text
Only this machine.
```

If a service listens on `127.0.0.1`, remote machines cannot connect to it.

Example:

```text
127.0.0.1:5432
```

PostgreSQL is reachable only locally.

---

## 0.0.0.0

```text
0.0.0.0
```

Means:

```text
Listen on all available IPv4 interfaces.
```

Example:

```text
0.0.0.0:80
```

Nginx accepts traffic from any network interface, if firewall rules allow it.

---

# 6. Localhost Example

Imagine this server:

```text
Server IP: 10.0.1.20
```

Service A:

```text
127.0.0.1:8080
```

Service B:

```text
0.0.0.0:80
```

Result:

```text
Remote user → 10.0.1.20:80     Works
Remote user → 10.0.1.20:8080   Does not work
Same server → 127.0.0.1:8080   Works
```

This is a common production issue.

---

# 7. What Is a Socket?

A socket is one endpoint of a network conversation.

A TCP connection is uniquely identified by four values:

```text
Source IP
Source Port
Destination IP
Destination Port
```

This is called a socket pair or 4-tuple.

Example:

```text
Client: 192.168.1.10:51520
Server: 203.0.113.10:443
```

Full connection:

```text
192.168.1.10:51520 → 203.0.113.10:443
```

---

# 8. Why Client Ports Look Random

When your browser connects to HTTPS:

```text
Destination: 203.0.113.10:443
```

Your machine also needs a source port.

Linux chooses a temporary high-numbered port.

Example:

```text
192.168.1.10:51520 → 203.0.113.10:443
```

This temporary port is called an **ephemeral port**.

It allows many connections to the same server.

---

# 9. Many Browser Tabs, Same Server

Suppose you open three browser tabs to the same website.

```text
192.168.1.10:51520 → 203.0.113.10:443
192.168.1.10:51521 → 203.0.113.10:443
192.168.1.10:51522 → 203.0.113.10:443
```

Same destination IP.

Same destination port.

Different source ports.

That is how Linux keeps the connections separate.

---

# 10. Port Categories

Ports are usually grouped like this:

| Range | Name | Usage |
|---|---|---|
| 0–1023 | Well-known ports | Common services like SSH, HTTP, HTTPS |
| 1024–49151 | Registered ports | Application/service ports |
| 49152–65535 | Dynamic/Ephemeral ports | Temporary client-side ports |

On Linux, binding to ports below `1024` usually requires root privileges or special capabilities.

Example:

```text
Port 80 requires elevated permission.
Port 8080 usually does not.
```

That is why developers often run apps on:

```text
8080
3000
5000
8000
```

---

# 11. Ports and Linux Processes

A port is not useful by itself.

A process must listen on it.

Example:

```text
Port 80  → nginx process
Port 22  → sshd process
Port 5432 → postgres process
```

If no process is listening, connection attempts fail.

This is why DevOps engineers often ask:

> Is the service listening on the expected port?

---

# 12. Important Linux Command Preview: ss

Later, we will use:

```bash
ss -tulnp
```

Meaning:

```text
-t  TCP
-u  UDP
-l  Listening
-n  Numeric output
-p  Show process
```

Example output:

```text
State   Local Address:Port     Process
LISTEN  0.0.0.0:80             nginx
LISTEN  127.0.0.1:5432         postgres
LISTEN  0.0.0.0:22             sshd
```

This command is one of the most important networking troubleshooting tools in Linux.

---

# 13. DevOps Production Scenario

## Incident

Users report:

```text
The API is down.
```

You SSH into the server.

The Spring Boot process is running.

But users still cannot connect.

You check:

```bash
ss -tulnp
```

You find:

```text
LISTEN 127.0.0.1:8080 java
```

The application is listening only on localhost.

That means:

```text
Nginx on the same machine can reach it.
Remote users cannot reach it directly.
```

If the architecture expects Nginx to proxy traffic, this may be correct.

If users are supposed to connect directly to port `8080`, this is a problem.

---

# 14. Another Production Scenario

Nginx is supposed to listen on HTTPS.

Expected:

```text
0.0.0.0:443
```

Actual:

```text
0.0.0.0:80
```

Result:

```text
http://example.com  works
https://example.com fails
```

Possible causes:

- TLS configuration missing
- Certificate problem
- Wrong Nginx server block
- Firewall blocking 443
- Load balancer forwarding to wrong port

---

# 15. Ports and Firewalls

A service can be listening correctly, but traffic may still fail.

Example:

```text
Nginx listens on 0.0.0.0:443
```

But firewall blocks port `443`.

Result:

```text
Service is running.
Port is listening.
Users still cannot connect.
```

So troubleshooting must check both:

```text
Is the service listening?
Is traffic allowed?
```

---

# 16. Ports in Docker

Docker commonly maps host ports to container ports.

Example:

```bash
docker run -p 8080:80 nginx
```

Means:

```text
Host port 8080 → Container port 80
```

Diagram:

```text
Browser
  │
  ▼
Host IP:8080
  │
  ▼
Docker NAT
  │
  ▼
Container IP:80
```

This explains why a container can run Nginx on port `80`, while the host exposes it on port `8080`.

---

# 17. Ports in Kubernetes

Kubernetes has several port concepts.

Example Service:

```yaml
port: 80
targetPort: 8080
```

Meaning:

```text
Service port:     80
Container port:   8080
```

Diagram:

```text
Client
  │
  ▼
Kubernetes Service:80
  │
  ▼
Pod:8080
```

This is a major source of confusion for beginners.

Once you understand ports and sockets, Kubernetes networking becomes easier.

---

# 18. Ports in Cloud Load Balancers

A cloud load balancer may listen on one port and forward to another.

Example:

```text
User → Load Balancer:443
Load Balancer → Backend Server:8080
```

This is common in production.

TLS may terminate at the load balancer, then forward plain HTTP to the backend.

---

# 19. Important Mental Model

When debugging ports, always ask:

```text
Client wants to connect to which IP and which port?
Is a process listening there?
Is it listening on the correct interface?
Is firewall/security group allowing it?
Is a proxy/load balancer forwarding correctly?
```

Never check only one layer.

---

# 20. Common Mistakes

❌ Thinking an IP address alone is enough.

❌ Forgetting that services must listen on ports.

❌ Confusing `127.0.0.1` with `0.0.0.0`.

❌ Exposing databases publicly.

❌ Forgetting Docker port mapping.

❌ Confusing Kubernetes `port` and `targetPort`.

❌ Assuming a running process means the network service is reachable.

---

# 21. Best Practices

- Bind public-facing services only when needed.
- Keep databases private whenever possible.
- Use `ss -tulnp` to verify listening services.
- Know the difference between local-only and public listening.
- Document expected ports for every service.
- In production, expose apps through reverse proxies or load balancers.
- Avoid opening unnecessary firewall ports.
- Be careful when binding services to `0.0.0.0`.

---

# 22. Hands-On Lab

Run these commands on your Linux machine.

## Task 1 — View listening TCP/UDP ports

```bash
ss -tulnp
```

Observe:

- Local address
- Port
- Process name
- Listening state

---

## Task 2 — View TCP listening ports only

```bash
ss -tlnp
```

---

## Task 3 — Find SSH listening port

```bash
ss -tlnp | grep ':22'
```

---

## Task 4 — Find services listening only on localhost

```bash
ss -tulnp | grep '127.0.0.1'
```

---

## Task 5 — Find services listening on all interfaces

```bash
ss -tulnp | grep '0.0.0.0'
```

---

# 23. Troubleshooting Checklist

When a service is unreachable:

```text
1. Is the process running?
2. Is it listening on the expected port?
3. Is it listening on 127.0.0.1 or 0.0.0.0?
4. Is the firewall allowing the port?
5. Is the load balancer forwarding to the correct port?
6. Is Docker/Kubernetes mapping the port correctly?
7. Is the client using the correct protocol?
```

Example:

```text
Using HTTPS against an HTTP-only port will fail.
```

---

# 24. Interview Questions

## 1. What is the difference between an IP address and a port?

An IP address identifies the machine. A port identifies the service or application on that machine.

---

## 2. What does it mean when a service listens on `127.0.0.1`?

It accepts connections only from the local machine.

---

## 3. What does it mean when a service listens on `0.0.0.0`?

It listens on all available IPv4 interfaces.

---

## 4. What is an ephemeral port?

A temporary client-side port chosen by the operating system for outgoing connections.

---

## 5. What command shows listening ports in Linux?

```bash
ss -tulnp
```

---

## 6. Why can a service be running but unreachable?

Because it may not be listening on the expected port, may be bound only to localhost, or traffic may be blocked by firewall/security rules.

---

## 7. What is Docker port mapping?

It maps a port on the host machine to a port inside the container.

Example:

```bash
-p 8080:80
```

Means host port `8080` forwards to container port `80`.

---

## 8. What is the difference between Kubernetes `port` and `targetPort`?

`port` is the Service port. `targetPort` is the port inside the Pod/container.

---

# 25. Cheat Sheet

| Concept | Meaning |
|---|---|
| IP Address | Identifies a machine/network endpoint |
| Port | Identifies a service on a machine |
| Socket | Endpoint of a network conversation |
| Listening Port | Port waiting for connections |
| 127.0.0.1 | Local machine only |
| 0.0.0.0 | All IPv4 interfaces |
| Ephemeral Port | Temporary client-side port |
| `ss -tulnp` | Show listening TCP/UDP ports and processes |
| Docker `-p 8080:80` | Host 8080 maps to container 80 |
| Kubernetes `targetPort` | Port inside the Pod/container |

---

# 26. Summary

In this lesson, you learned:

- An IP address identifies a machine.
- A port identifies a service.
- A socket represents one endpoint of communication.
- Linux uses ports to deliver traffic to the correct process.
- `127.0.0.1` means local-only.
- `0.0.0.0` means all IPv4 interfaces.
- Ephemeral ports allow many client connections.
- Docker, Kubernetes and cloud load balancers all depend heavily on port mapping.
- A running process is not enough; it must listen on the correct address and port.

---

# Next Lesson

**Lesson 08 — TCP vs UDP**

In the next lesson, you will learn:

- Why TCP exists
- How the TCP handshake works
- Why TCP is reliable
- Why UDP exists
- When UDP is better than TCP
- Why DNS often uses UDP
- How DevOps engineers troubleshoot TCP and UDP problems

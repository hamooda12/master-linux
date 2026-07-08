
# Chapter 06 — Networking Fundamentals

# Lesson 15 — Production Network Troubleshooting Methodology for DevOps Engineers

> **Objective:** Learn a repeatable troubleshooting methodology used by experienced DevOps engineers to diagnose network problems from the operating system up to the application.

---

# Introduction

One of the biggest differences between a junior and a senior DevOps engineer is **not** knowing more commands.

It is knowing **where to start**.

Junior engineers often guess:

- Restart Nginx
- Restart Docker
- Restart Kubernetes Pods
- Restart the VM

Sometimes it works.

Often it hides the real problem.

Senior engineers instead follow a process.

This lesson teaches that process.

---

# The Golden Rule

**Always troubleshoot from the bottom up.**

Never jump directly into the application.

Follow the networking stack.

```text
Physical Link
      │
Network Interface
      │
IP Address
      │
Routing
      │
DNS
      │
TCP Connection
      │
Firewall
      │
TLS
      │
HTTP
      │
Reverse Proxy
      │
Backend
      │
Database
```

---

# Incident 1 — "The Website Is Down"

Users report:

```text
https://app.example.com
```

does not open.

Do NOT restart services.

Start asking questions.

---

# Step 1 — Can the Server Reach the Network?

Check interfaces:

```bash
ip link
ip addr
```

Look for:

- Correct interface
- UP
- LOWER_UP
- Correct IP address

If the interface is DOWN, nothing else matters.

---

# Step 2 — Check the Routing Table

```bash
ip route
```

Verify:

- Default route exists
- Correct gateway
- Docker routes
- VPN routes

If there is no default route, the server cannot reach remote networks.

---

# Step 3 — Test Local Connectivity

Ping the gateway:

```bash
ping <gateway-ip>
```

Results:

- Success → Local network works.
- Failure → Cable, switch, VLAN, hypervisor, cloud network, or gateway issue.

---

# Step 4 — Test Internet Connectivity

```bash
ping 8.8.8.8
```

If this fails but the gateway works:

Possible causes:

- ISP problem
- Cloud routing
- Firewall
- Missing upstream route

---

# Step 5 — Test DNS

```bash
ping google.com
```

Interpretation:

| 8.8.8.8 | google.com | Meaning |
|----------|------------|---------|
| ✔ | ✔ | Network + DNS OK |
| ✔ | ✘ | DNS problem |
| ✘ | ✘ | Connectivity problem |

Also inspect:

```bash
cat /etc/resolv.conf
cat /etc/hosts
```

---

# Step 6 — Is the Service Listening?

```bash
ss -tulpen
```

Example:

```text
LISTEN 0 511 0.0.0.0:443
```

Good.

Example:

```text
127.0.0.1:8080
```

Only local clients can connect.

---

# Step 7 — Test Locally

```bash
curl http://localhost:8080
```

If this fails:

The backend is probably not healthy.

Check:

```bash
systemctl status backend
journalctl -u backend -n 50
```

---

# Step 8 — Test Through the Reverse Proxy

```bash
curl -vk https://localhost
```

If backend works but HTTPS fails:

Investigate:

- Nginx
- TLS
- Certificates
- Proxy configuration

---

# Step 9 — Check Nginx

```bash
nginx -t
systemctl status nginx
journalctl -u nginx -n 100
```

Common issues:

- Syntax error
- Wrong upstream
- Wrong port
- Missing certificate
- Bad permissions

---

# Step 10 — Check the Backend

Questions:

- Running?
- Listening?
- Correct port?
- Bound to 0.0.0.0?
- Database reachable?

Useful commands:

```bash
systemctl status backend
ss -tulpen
curl http://127.0.0.1:8080
```

---

# Step 11 — Check the Database

Application logs often show:

```text
Connection refused
```

or

```text
Timeout
```

Verify database service:

```bash
systemctl status postgresql
```

or

```bash
systemctl status mysql
```

---

# Understanding Common Errors

## Connection Refused

Meaning:

The destination was reached, but nothing accepted the connection.

Usually:

- Service stopped
- Wrong port
- Not listening

---

## Connection Timed Out

Meaning:

No response returned.

Possible causes:

- Firewall
- Routing
- Security Group
- Packet dropped

---

## 404 Not Found

Network is working.

Application handled the request.

Wrong URL or missing resource.

---

## 502 Bad Gateway

Proxy cannot communicate with backend.

---

## 503 Service Unavailable

Backend unavailable or overloaded.

---

## 504 Gateway Timeout

Backend took too long.

---

# Production Decision Tree

```text
Website Down
      │
      ▼
Interface OK?
      │
No → Fix interface
      │
Yes
      ▼
IP Correct?
      │
No → Fix IP
      │
Yes
      ▼
Gateway Reachable?
      │
No → Fix routing/network
      │
Yes
      ▼
DNS Works?
      │
No → Fix DNS
      │
Yes
      ▼
Port Listening?
      │
No → Fix application
      │
Yes
      ▼
Firewall Open?
      │
No → Fix firewall
      │
Yes
      ▼
TLS Works?
      │
No → Fix certificates/Nginx
      │
Yes
      ▼
HTTP Response?
      │
5xx → Backend
4xx → Client/Application
2xx → Healthy
```

---

# Real Incident 1

Users:

```text
Cannot open website.
```

Findings:

```bash
ss -tulpen
```

No port 443.

Root cause:

Nginx stopped.

---

# Real Incident 2

Port 443 listening.

Backend:

```bash
curl localhost:8080
```

fails.

Root cause:

Backend crashed.

---

# Real Incident 3

Backend healthy.

HTTPS unhealthy.

```bash
nginx -t
```

returns syntax error.

Root cause:

Broken deployment.

---

# Real Incident 4

Everything healthy locally.

Remote users fail.

```bash
ufw status
```

Port 443 blocked.

Root cause:

Firewall.

---

# Real Incident 5

Everything healthy.

Remote still fails.

Security Group blocks port 443.

Root cause:

Cloud firewall.

---

# Real Incident 6

Everything healthy.

Only one office cannot connect.

Root cause:

Corporate firewall.

---

# Command Reference

```bash
# Interfaces
ip link
ip addr

# Routes
ip route

# DNS
cat /etc/resolv.conf
cat /etc/hosts

# Connectivity
ping
tracepath

# Ports
ss -tulpen

# HTTP
curl -I
curl -v
curl -vk

# TLS
openssl s_client -connect example.com:443

# Services
systemctl status
journalctl

# Firewall
ufw status
```

---

# Troubleshooting Checklist

- Is the interface UP?
- Does it have the correct IP?
- Is the gateway reachable?
- Does routing look correct?
- Does DNS resolve?
- Is the service listening?
- Is it listening on the correct address?
- Is the firewall open?
- Does TLS succeed?
- Does HTTP return a response?
- Is the reverse proxy healthy?
- Is the backend healthy?
- Is the database healthy?

---

# Common Mistakes

- Restarting services before investigating.
- Ignoring logs.
- Assuming DNS is always the problem.
- Confusing timeout with connection refused.
- Forgetting cloud firewalls.
- Testing only from localhost.
- Ignoring listening addresses.

---

# Interview Questions

1. Describe your troubleshooting methodology when a website is unreachable.
2. Difference between "Connection Refused" and "Connection Timed Out"?
3. How do you verify a service is listening?
4. Why test with localhost first?
5. How do you isolate DNS issues?
6. What causes HTTP 502?
7. Which layer do you investigate first?

---

# Final DevOps Troubleshooting Mindset

Never guess.

Observe.

Measure.

Verify.

Change **one thing at a time**.

After every change, verify again.

This disciplined approach is what allows experienced DevOps engineers to solve incidents quickly without creating new ones.

---

# Summary

A production incident is rarely solved by memorizing commands. It is solved by following a structured process.

When you consistently verify:

1. Network interface
2. IP address
3. Routing
4. DNS
5. Listening ports
6. Firewall
7. TLS
8. Reverse proxy
9. Backend
10. Database

you can isolate almost any networking problem efficiently and confidently.

# Chapter 06 — Networking Fundamentals

# Lesson 14 — Reverse Proxy & Load Balancer for DevOps Engineers

> **Objective:** Understand how reverse proxies and load balancers work, why almost every production environment uses them, and how to troubleshoot the most common failures.

---

# Why This Lesson Matters

Most production architectures are **not**:

```text
Browser
   │
   ▼
Application
```

Instead they look like:

```text
Browser
   │ HTTPS
   ▼
Load Balancer
   │
   ▼
Reverse Proxy (Nginx)
   │
   ▼
Application (Spring Boot / Node.js / Laravel)
   │
   ▼
Database
```

As a DevOps engineer, you'll spend much of your time configuring, monitoring, and troubleshooting these layers.

---

# Forward Proxy vs Reverse Proxy

## Forward Proxy

A forward proxy sits **in front of clients**.

```text
Client
   │
Forward Proxy
   │
Internet
```

Common uses:

- Hide client identity
- Content filtering
- Corporate internet access
- Caching

---

## Reverse Proxy

A reverse proxy sits **in front of servers**.

```text
Internet
   │
Reverse Proxy
   │
Backend Servers
```

Clients usually don't know the backend servers exist.

Benefits:

- Security
- TLS termination
- Load balancing
- Caching
- Compression
- Central logging

---

# Why Not Expose the Application Directly?

Without a reverse proxy:

```text
Internet
   │
Spring Boot :8080
```

Problems:

- No TLS management
- Weak request filtering
- No centralized logging
- No load balancing
- Every app manages HTTPS itself

With Nginx:

```text
Internet
   │
Nginx :443
   │
Spring Boot :8080
```

Nginx becomes the public entry point.

---

# What Is a Load Balancer?

A load balancer distributes incoming requests across multiple backend servers.

Example:

```text
           Client Requests
                 │
                 ▼
          Load Balancer
        ┌─────┼─────┐
        ▼     ▼     ▼
      App1  App2  App3
```

Benefits:

- Higher availability
- Better scalability
- Lower response time
- No single overloaded backend

---

# Common Load-Balancing Algorithms

## Round Robin

Requests are distributed one after another.

```text
1 → App1
2 → App2
3 → App3
4 → App1
```

Simple and widely used.

---

## Least Connections

Send new requests to the server with the fewest active connections.

Useful when requests have different durations.

---

## IP Hash

The client IP determines the backend.

Useful for applications that keep session state in memory.

---

## Weighted Round Robin

More powerful servers receive more traffic.

Example:

```text
App1 weight 3
App2 weight 1
```

App1 receives roughly three times as many requests.

---

# TLS Termination

HTTPS is computationally expensive.

Instead of every backend handling TLS:

```text
Browser
   │ HTTPS
   ▼
Nginx
   │ HTTP
   ▼
Backend
```

Nginx decrypts HTTPS and forwards HTTP internally.

This process is called **TLS termination**.

---

# Important Reverse Proxy Headers

When Nginx forwards requests, it often adds headers.

## Host

Original hostname requested by the client.

## X-Forwarded-For

Original client IP.

Without this, the backend may only see the proxy IP.

## X-Forwarded-Proto

Indicates whether the original request used HTTP or HTTPS.

## X-Real-IP

Another commonly used client IP header.

---

# Example Nginx Configuration

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Explanation:

- `listen 443 ssl` → accept HTTPS.
- `proxy_pass` → forward traffic to the backend.
- `proxy_set_header` → preserve important request information.

---

# Health Checks

A load balancer should not send traffic to unhealthy servers.

Example:

```text
App1 ✓
App2 ✓
App3 ✗
```

Requests go only to App1 and App2 until App3 recovers.

---

# Sticky Sessions

Some applications store user sessions in memory.

If request 1 goes to App1 and request 2 goes to App2, the session may be lost.

Sticky sessions keep the same client on the same backend.

Modern applications often avoid this by storing sessions in Redis or a database.

---

# Common Production Errors

## 502 Bad Gateway

Meaning:

The proxy could not get a valid response from the backend.

Possible causes:

- Backend crashed
- Wrong backend port
- Backend not listening
- Firewall between proxy and backend

Check:

```bash
systemctl status backend
ss -tulpen
journalctl -u nginx -n 50
```

---

## 503 Service Unavailable

Meaning:

The service is temporarily unavailable.

Possible causes:

- No healthy backends
- Maintenance
- Application overloaded

---

## 504 Gateway Timeout

Meaning:

The backend took too long to respond.

Possible causes:

- Slow database
- Long-running query
- Network latency
- Application deadlock

---

# Troubleshooting Workflow

```text
Browser
   │
DNS OK?
   │
TCP OK?
   │
TLS OK?
   │
Nginx Listening?
   │
Backend Listening?
   │
Backend Healthy?
   │
Database Healthy?
```

Useful commands:

```bash
curl -v https://app.example.com
ss -tulpen
nginx -t
systemctl status nginx
journalctl -u nginx -n 50
curl http://127.0.0.1:8080
```

---

# Hands-On Lab

1. Verify Nginx configuration:

```bash
sudo nginx -t
```

2. Check listening ports:

```bash
ss -tulpen
```

3. Test backend directly:

```bash
curl http://127.0.0.1:8080
```

4. Test through the reverse proxy:

```bash
curl -vk https://localhost
```

Observe where failures occur.

---

# Best Practices

- Terminate TLS at the reverse proxy.
- Preserve `X-Forwarded-*` headers.
- Enable health checks.
- Avoid exposing backend services directly.
- Monitor latency and error rates.
- Keep configurations under version control.

---

# Interview Questions

1. What is a reverse proxy?
2. What is the difference between a reverse proxy and a forward proxy?
3. Why use a load balancer?
4. Explain Round Robin and Least Connections.
5. What is TLS termination?
6. What causes HTTP 502, 503, and 504?
7. Why are `X-Forwarded-For` and `Host` important?

---

# Cheat Sheet

```bash
nginx -t
systemctl status nginx
journalctl -u nginx
ss -tulpen
curl -I https://example.com
curl -vk https://localhost
```

---

# Summary

Reverse proxies and load balancers are core building blocks of modern production systems. They secure applications, distribute traffic, terminate TLS, and provide a central point for routing and monitoring. Understanding how requests flow through these components is essential for every DevOps engineer.

---

# Next Lesson

**Lesson 15 — Production Network Troubleshooting Methodology**

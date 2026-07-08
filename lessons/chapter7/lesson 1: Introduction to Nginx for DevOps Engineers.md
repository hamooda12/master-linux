
# Chapter 07 — Nginx & Production Web Infrastructure

# Lesson 01 — Introduction to Nginx for DevOps Engineers

> **Objective:** Understand what Nginx is, why it became one of the most widely used web servers and reverse proxies, and where it fits in a modern production architecture.

---

# Why Learn Nginx?

Almost every production environment contains one of these:

- Nginx
- Apache
- HAProxy
- Envoy

Among them, **Nginx** is one of the most common.

Even if your application is written in:

- Spring Boot
- Laravel
- Node.js
- Django
- Go

there is a very good chance that Nginx sits in front of it.

---

# What Is Nginx?

Nginx (pronounced **"Engine-X"**) is software that can act as:

- Web Server
- Reverse Proxy
- Load Balancer
- TLS/SSL Terminator
- Static File Server
- HTTP Cache

One program can perform all of these roles.

---

# Where Nginx Sits

A simple production architecture:

```text
Internet
    │
 HTTPS
    │
    ▼
+-------------+
|    Nginx    |
+-------------+
    │
 HTTP
    │
    ▼
+------------------+
| Spring Boot App  |
+------------------+
    │
    ▼
+-------------+
| PostgreSQL  |
+-------------+
```

Users never connect directly to your application.

They connect to **Nginx**.

---

# Why Not Expose Spring Boot Directly?

Suppose your application listens on:

```text
0.0.0.0:8080
```

If you expose it directly:

```text
Internet
   │
Spring Boot
```

You lose many production features:

- Central TLS management
- Reverse proxying
- Request filtering
- Compression
- Caching
- Access logging
- Load balancing

Instead:

```text
Internet
    │
Nginx :443
    │
Spring Boot :8080
```

Nginx becomes the secure public entry point.

---

# Nginx vs Apache

| Feature | Nginx | Apache |
|---------|--------|---------|
| Architecture | Event-driven | Process/Thread based (traditional) |
| Memory Usage | Lower | Higher |
| Static Files | Excellent | Good |
| Reverse Proxy | Excellent | Good |
| High Concurrency | Excellent | Good |

Both are excellent servers, but Nginx became famous for handling large numbers of concurrent connections efficiently.

---

# Installing Nginx (Ubuntu)

Update packages:

```bash
sudo apt update
```

Install:

```bash
sudo apt install nginx
```

Check service:

```bash
systemctl status nginx
```

Start manually:

```bash
sudo systemctl start nginx
```

Enable on boot:

```bash
sudo systemctl enable nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Reload configuration (preferred after config changes):

```bash
sudo systemctl reload nginx
```

Why reload instead of restart?

A reload tells Nginx to reread its configuration without dropping existing client connections.

---

# Verify Nginx Is Listening

Run:

```bash
ss -tulpen | grep :80
```

Typical output:

```text
tcp LISTEN 0 511 0.0.0.0:80
```

This tells us:

- TCP socket
- Listening
- Port 80
- Accepting connections on all IPv4 interfaces

---

# Test the Default Website

From the server:

```bash
curl http://localhost
```

From another machine:

```bash
curl http://SERVER_IP
```

Or open the server IP in a browser.

You should see:

```text
Welcome to nginx!
```

---

# Where Are Nginx Files?

Configuration:

```text
/etc/nginx/
```

Main configuration:

```text
/etc/nginx/nginx.conf
```

Virtual hosts:

```text
/etc/nginx/sites-available/
```

Enabled sites:

```text
/etc/nginx/sites-enabled/
```

Logs:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Default web root:

```text
/var/www/html/
```

---

# Validate Configuration

Before restarting Nginx:

```bash
sudo nginx -t
```

Example:

```text
syntax is ok
test is successful
```

Never restart production Nginx without validating the configuration first.

---

# Common Production Problems

## Port 80 Already in Use

Check:

```bash
ss -tulpen | grep :80
```

Another web server may already be listening.

---

## Nginx Won't Start

Check:

```bash
systemctl status nginx
journalctl -u nginx -n 50
```

Often caused by:

- Syntax errors
- Port conflicts
- Invalid certificates
- Permission problems

---

# Hands-On Lab

1. Install Nginx.
2. Start the service.
3. Verify it is running.
4. Confirm it listens on port 80.
5. Open the default page.
6. Locate:
   - nginx.conf
   - access.log
   - error.log
   - sites-available
   - sites-enabled

---

# Best Practices

- Always run `nginx -t` before reloading.
- Prefer `reload` over `restart` after configuration changes.
- Keep configuration files under version control.
- Monitor both access and error logs.
- Never expose backend applications directly when Nginx can sit in front of them.

---

# Cheat Sheet

```bash
sudo apt install nginx
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
sudo nginx -t
ss -tulpen | grep :80
curl http://localhost
```

---

# Interview Questions

1. What is Nginx?
2. What roles can Nginx perform?
3. Why place Nginx in front of an application?
4. Why use `reload` instead of `restart`?
5. Where is the main Nginx configuration file?
6. How do you validate an Nginx configuration?

---

# Summary

Nginx is far more than a web server. It is the front door to many production systems, providing security, performance, TLS termination, reverse proxying, and load balancing. Mastering Nginx begins with understanding its role in the architecture before diving into its configuration.

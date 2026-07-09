# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 12 --- Linux Firewall Fundamentals (Part 3)

------------------------------------------------------------------------

# Allowing Traffic

The primary purpose of a firewall is to decide which traffic is allowed.

With UFW, allowing services is straightforward.

------------------------------------------------------------------------

# Allowing SSH

Before enabling the firewall on a remote server, always allow SSH.

``` bash
sudo ufw allow 22/tcp
```

Or by service name:

``` bash
sudo ufw allow OpenSSH
```

Using the service name reduces mistakes because UFW already knows the
correct port.

------------------------------------------------------------------------

# Allowing HTTP

To allow a web server:

``` bash
sudo ufw allow 80/tcp
```

This permits incoming HTTP traffic.

------------------------------------------------------------------------

# Allowing HTTPS

For secure websites:

``` bash
sudo ufw allow 443/tcp
```

------------------------------------------------------------------------

# Allowing Custom Ports

Applications often use custom ports.

Example:

``` bash
sudo ufw allow 8080/tcp
```

Spring Boot applications commonly use port **8080** during development.

------------------------------------------------------------------------

# Allowing UDP

Not every application uses TCP.

DNS commonly uses UDP.

Example:

``` bash
sudo ufw allow 53/udp
```

Always allow the protocol actually used by the application.

------------------------------------------------------------------------

# Denying Traffic

To block a service:

``` bash
sudo ufw deny 8080/tcp
```

Future connection attempts to this port are blocked.

------------------------------------------------------------------------

# Removing Rules

If a rule is no longer needed:

``` bash
sudo ufw delete allow 8080/tcp
```

Example output:

``` text
Rule deleted
```

------------------------------------------------------------------------

# Listing Numbered Rules

Numbered rules make deletion easier.

``` bash
sudo ufw status numbered
```

Example:

``` text
[1] 22/tcp   ALLOW IN Anywhere
[2] 80/tcp   ALLOW IN Anywhere
[3] 443/tcp  ALLOW IN Anywhere
```

Delete by number:

``` bash
sudo ufw delete 2
```

------------------------------------------------------------------------

# Application Profiles

Many packages install predefined UFW profiles.

List them:

``` bash
sudo ufw app list
```

Example:

``` text
Available applications:

OpenSSH
Nginx Full
Nginx HTTP
Nginx HTTPS
```

Allow a profile:

``` bash
sudo ufw allow "Nginx Full"
```

This is often easier than remembering individual ports.

------------------------------------------------------------------------

# Linux Perspective

Typical deployment sequence:

1.  Verify the service is listening.

``` bash
ss -tulpn
```

2.  Allow the required firewall rule.

``` bash
sudo ufw allow 8080/tcp
```

3.  Verify the firewall configuration.

``` bash
sudo ufw status
```

------------------------------------------------------------------------

# Production Example

A Java application listens on:

``` text
0.0.0.0:8080
```

`ss` confirms the socket exists.

Remote clients still cannot connect.

Running:

``` bash
sudo ufw status
```

shows that TCP port **8080** is not allowed.

Adding the correct firewall rule immediately restores connectivity.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Allowing TCP ports
-   Allowing UDP ports
-   Denying traffic
-   Removing firewall rules
-   Numbered rules
-   Application profiles

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete the firewall lesson with production troubleshooting
scenarios, interview questions, hands-on labs, challenge exercises, and
a firewall cheat sheet.

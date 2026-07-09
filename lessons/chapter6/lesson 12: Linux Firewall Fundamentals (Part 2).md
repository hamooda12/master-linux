# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 12 --- Linux Firewall Fundamentals (Part 2)

------------------------------------------------------------------------

# Introducing UFW

## What Is UFW?

**UFW (Uncomplicated Firewall)** is a user-friendly interface for
managing the Linux firewall.

Instead of writing complex firewall rules directly, UFW provides simple
commands that are easy to remember and use.

Ubuntu installs and documents UFW as the recommended firewall management
tool.

------------------------------------------------------------------------

# Why Use UFW?

Without UFW, administrators often interact directly with:

-   iptables (legacy)
-   nftables

These tools are powerful but can be difficult for beginners.

UFW simplifies common firewall tasks while using the underlying firewall
framework.

------------------------------------------------------------------------

# Checking Firewall Status

The first command every administrator should know is:

``` bash
sudo ufw status
```

Example output:

``` text
Status: inactive
```

or

``` text
Status: active
```

This tells you whether the firewall is currently enforcing rules.

------------------------------------------------------------------------

# Enabling the Firewall

To enable UFW:

``` bash
sudo ufw enable
```

Example output:

``` text
Firewall is active and enabled on system startup
```

Once enabled, firewall rules begin protecting the system.

> ⚠️ **Important**
>
> On remote servers, always allow SSH **before** enabling the firewall.
>
> Otherwise, you may lock yourself out of the server.

------------------------------------------------------------------------

# Disabling the Firewall

To temporarily disable UFW:

``` bash
sudo ufw disable
```

Example:

``` text
Firewall stopped and disabled on system startup
```

Disabling a firewall removes an important layer of protection and should
only be done when necessary.

------------------------------------------------------------------------

# Viewing Firewall Rules

Display all configured rules:

``` bash
sudo ufw status verbose
```

Example:

``` text
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

This output shows:

-   Which ports are allowed.
-   Which protocol is used.
-   Where connections are allowed from.

------------------------------------------------------------------------

# Default Policies

Every firewall has default behavior.

Typical defaults are:

``` text
Incoming: deny

Outgoing: allow
```

Meaning:

-   Block unsolicited incoming connections.
-   Allow applications on the server to communicate outward.

This is a secure starting point for most Linux servers.

------------------------------------------------------------------------

# Linux Perspective

When troubleshooting connectivity problems, always verify both:

1.  Is the application listening?

``` bash
ss -tulpn
```

2.  Is the firewall allowing the connection?

``` bash
sudo ufw status
```

Both conditions must be true before clients can connect successfully.

------------------------------------------------------------------------

# Production Example

An administrator deploys an SSH server.

The SSH service is listening on port 22:

``` bash
ss -tulpn
```

However, remote clients still cannot connect.

Running:

``` bash
sudo ufw status
```

reveals that SSH traffic is blocked by the firewall.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   What UFW is.
-   Why Linux uses UFW.
-   Checking firewall status.
-   Enabling and disabling UFW.
-   Viewing firewall rules.
-   Default firewall policies.

------------------------------------------------------------------------

# ➡️ Next Part

We'll learn how to allow and deny ports, remove firewall rules, and
safely configure access for SSH, web servers, and applications.

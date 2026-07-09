# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 11 --- SSH Fundamentals (Part 1)

> **Goal:** Understand what SSH is, why it is essential for Linux
> administration, and how secure remote access works.

------------------------------------------------------------------------

# 📚 Overview

Imagine your Linux server is running in another room, another city, or
another country.

How do you manage it without sitting in front of it?

The answer is **SSH (Secure Shell)**.

SSH is one of the most important tools used by Linux administrators,
DevOps engineers, cloud engineers, and system administrators. Nearly
every Linux server in production is managed through SSH.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain what SSH is.
-   Understand why SSH is used.
-   Compare SSH with older remote access methods.
-   Understand the client-server model.
-   Recognize the default SSH port.

------------------------------------------------------------------------

# 🌍 What Is SSH?

**SSH (Secure Shell)** is a secure network protocol used to remotely
access and manage another computer over a network.

Instead of physically connecting a keyboard and monitor to a server, you
open an encrypted SSH session.

    Administrator
          │
          │ SSH
          ▼
    Linux Server

Anything you type is securely transmitted to the remote machine.

------------------------------------------------------------------------

# Why Do We Need SSH?

Without SSH, administrators would need physical access to every server.

In modern environments, servers may be located in:

-   Data centers
-   Cloud platforms
-   Remote offices
-   Different countries

SSH allows administrators to manage them from anywhere with network
access.

------------------------------------------------------------------------

# SSH Client and SSH Server

SSH communication involves two components.

## SSH Client

The program used to initiate a remote connection.

Example:

``` bash
ssh user@server
```

Your laptop usually acts as the client.

------------------------------------------------------------------------

## SSH Server

The service running on the remote Linux machine that accepts incoming
SSH connections.

    Laptop
       │
    SSH Client
       │
    ════════════ Network ════════════
       │
    SSH Server
       │
    Linux Server

------------------------------------------------------------------------

# Why SSH Is Secure

SSH encrypts all communication between the client and server.

Encryption protects:

-   Usernames
-   Passwords
-   Commands
-   Files transferred through SSH
-   Command output

Anyone intercepting the traffic sees encrypted data rather than readable
text.

------------------------------------------------------------------------

# SSH vs Telnet

  SSH               Telnet
  ----------------- -------------------
  Encrypted         Unencrypted
  Secure            Insecure
  Recommended       Rarely used today
  Default Port 22   Default Port 23

Today, SSH has almost completely replaced Telnet for remote
administration.

------------------------------------------------------------------------

# Default SSH Port

By default, SSH listens on:

``` text
22/TCP
```

This means SSH clients connect to TCP port **22** unless another port
has been configured.

------------------------------------------------------------------------

# Linux Perspective

Most Linux distributions use **OpenSSH**.

Later in this lesson you'll learn commands such as:

``` bash
ssh
```

``` bash
scp
```

``` bash
sftp
```

and inspect the SSH service using:

``` bash
ss -tulpn
```

------------------------------------------------------------------------

# Production Example

A DevOps engineer deploys a new application to a cloud server.

Instead of visiting the data center, they connect using SSH, review
logs, restart services, and verify the deployment---all remotely and
securely.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   What SSH is.
-   Why SSH exists.
-   SSH client and server.
-   Why SSH is secure.
-   SSH vs Telnet.
-   The default SSH port.

------------------------------------------------------------------------

# ➡️ Next Part

We'll establish SSH connections, understand SSH command syntax, host
keys, authentication methods, and safely connect to Linux servers.

# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 03 --- TCP/IP Model (Part 1)

> **Goal:** Understand the networking model that Linux, the Internet,
> cloud providers, Docker, Kubernetes, and modern applications actually
> use.

------------------------------------------------------------------------

# 📚 Overview

Although the OSI model is an excellent learning tool, real-world
networking is based on the **TCP/IP Model**.

Linux networking tools, web browsers, cloud platforms, and network
devices all implement the TCP/IP model.

Understanding this model is essential for every Linux administrator and
DevOps engineer.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain the purpose of the TCP/IP model.
-   List its four layers.
-   Compare it with the OSI model.
-   Understand why it is the practical networking model.

------------------------------------------------------------------------

# 🌍 Why Another Networking Model?

The OSI model explains networking conceptually.

The TCP/IP model explains how networking is actually implemented.

Think of it this way:

-   **OSI** teaches *how networking is organized*.
-   **TCP/IP** shows *how networking works in practice*.

------------------------------------------------------------------------

# 🏗 The Four TCP/IP Layers

    +----------------------+
    | Application          |
    +----------------------+
    | Transport            |
    +----------------------+
    | Internet             |
    +----------------------+
    | Network Access       |
    +----------------------+

Compared to OSI, several layers are combined.

------------------------------------------------------------------------

# Mapping TCP/IP to OSI

  TCP/IP Layer     Corresponding OSI Layers
  ---------------- ------------------------------------
  Application      Application, Presentation, Session
  Transport        Transport
  Internet         Network
  Network Access   Data Link + Physical

The TCP/IP model is simpler because it groups related responsibilities
together.

------------------------------------------------------------------------

# Why Linux Uses TCP/IP

Linux networking is built around TCP/IP.

Examples include:

-   IP addresses
-   TCP connections
-   UDP communication
-   HTTP
-   HTTPS
-   SSH
-   DNS

Every networking command you learn later ultimately works with these
layers.

------------------------------------------------------------------------

# Linux Perspective

When you run:

``` bash
curl https://example.com
```

Linux uses every TCP/IP layer automatically:

1.  Application creates an HTTP request.
2.  Transport establishes communication.
3.  Internet layer routes packets.
4.  Network Access transmits frames over the physical network.

You only see one command, but many networking layers cooperate behind
the scenes.

------------------------------------------------------------------------

# 📌 Lesson Summary

You learned:

-   Why the TCP/IP model exists.
-   The four TCP/IP layers.
-   How it differs from the OSI model.
-   Why Linux and the Internet use TCP/IP.

------------------------------------------------------------------------

# ➡️ Next Part

We'll begin with the **Application Layer**, the layer where users and
applications interact with the network.

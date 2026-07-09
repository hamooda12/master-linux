# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 09 --- Ports and Sockets (Part 1)

> **Goal:** Understand how Linux applications receive network
> connections and why ports and sockets are fundamental to every server
> application.

------------------------------------------------------------------------

# 📚 Overview

A computer can run many network applications simultaneously:

-   SSH
-   Nginx
-   Spring Boot
-   MySQL
-   Redis

All of them share the same network interface.

How does Linux know which application should receive an incoming
connection?

The answer is **ports** and **sockets**.

------------------------------------------------------------------------

# 🎯 Learning Objectives

After this lesson you will be able to:

-   Explain what a port is.
-   Explain what a socket is.
-   Distinguish between ports and IP addresses.
-   Understand why applications listen on ports.
-   Recognize common service ports.

------------------------------------------------------------------------

# 🌍 Why Do We Need Ports?

An IP address identifies a computer.

A port identifies a specific application running on that computer.

Example:

    Server

    IP: 192.168.1.20

     ├── SSH      Port 22
     ├── HTTP     Port 80
     ├── HTTPS    Port 443
     ├── Spring   Port 8080
     └── MySQL    Port 3306

Without ports, Linux would not know which application should receive
incoming packets.

------------------------------------------------------------------------

# 📖 What Is a Port?

A port is a logical communication endpoint used by the Transport Layer.

Applications bind to ports and wait for incoming connections.

Think of:

-   IP address = Apartment building
-   Port = Apartment number

The mail reaches the correct apartment because it has both.

------------------------------------------------------------------------

# Well-Known Ports

  Service              Default Port
  ------------------ --------------
  SSH                            22
  HTTP                           80
  HTTPS                         443
  DNS                            53
  SMTP                           25
  MySQL                        3306
  PostgreSQL                   5432
  Spring Boot                  8080
  React Dev Server             5173

Applications may use different ports, but these are common defaults.

------------------------------------------------------------------------

# What Is a Socket?

A socket is an endpoint created by the operating system that combines:

-   Protocol (TCP or UDP)
-   IP address
-   Port

A socket allows an application to communicate over the network.

Example:

    TCP
    192.168.1.20
    Port 8080

Together these form a socket.

------------------------------------------------------------------------

# Listening Socket

When an application starts, it often creates a **listening socket**.

    Spring Boot
          |
    Listening Socket
          |
    Port 8080

The application waits until a client connects.

------------------------------------------------------------------------

# Linux Perspective

Linux manages sockets inside the kernel.

Later you'll inspect them using:

``` bash
ss
netstat
lsof -i
```

These tools help determine:

-   Which ports are open.
-   Which application owns a port.
-   Whether a service is listening.

------------------------------------------------------------------------

# Production Example

Users cannot access a Spring Boot application.

The application process is running.

The network interface is healthy.

The next question is:

> Is the application actually listening on port **8080**?

This is where socket inspection becomes essential.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Why ports exist.
-   What a port is.
-   What a socket is.
-   Listening sockets.
-   Common service ports.

------------------------------------------------------------------------

# ➡️ Next Part

We'll inspect listening sockets using Linux commands, understand TCP and
UDP sockets, and analyze real command output.

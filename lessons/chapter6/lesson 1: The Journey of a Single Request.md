# Chapter 06 --- Networking Fundamentals

# Lesson 01 --- How the Internet Actually Works: The Journey of a Single Request

> **Part 1 --- Why Networks Exist & The Big Picture**

------------------------------------------------------------------------

# Lesson Objectives

By the end of this lesson, you will be able to:

-   Explain why computer networks exist.
-   Understand why networking is one of the most important skills for a
    DevOps engineer.
-   Visualize how devices communicate across the Internet.
-   Understand the difference between local communication and Internet
    communication.
-   Build the mental model that every future networking lesson depends
    on.

------------------------------------------------------------------------

# Before We Learn Networking

Forget the networking definitions you've memorized.

For now, ignore terms like:

-   IP Address
-   MAC Address
-   DNS
-   TCP
-   UDP
-   Router
-   Switch
-   Port

We'll first understand **why networking exists**. Once that makes sense,
every other concept will fit naturally.

------------------------------------------------------------------------

# Imagine There Is No Internet

Imagine there is no networking.

You develop a Spring Boot application.

It runs perfectly on your laptop.

Now your teammate wants to use it.

How do you send it?

-   USB drive
-   External disk
-   Email attachment (which also requires networking!)

Now imagine a customer wants to visit your website.

They simply can't.

Without networking:

-   GitHub wouldn't exist.
-   Docker Hub wouldn't exist.
-   Cloud platforms wouldn't exist.
-   SSH wouldn't exist.
-   APIs couldn't communicate.
-   Databases couldn't serve applications.
-   Kubernetes clusters couldn't work.

Modern software engineering depends entirely on networking.

------------------------------------------------------------------------

# What Is the Purpose of Networking?

A common definition is:

> A network connects computers.

While correct, it misses the real purpose.

A better definition is:

> **Networking allows computers to exchange information reliably,
> efficiently, and securely regardless of where they are located.**

Networking is fundamentally about **communication**.

------------------------------------------------------------------------

# What Is a Network?

A network is a collection of devices capable of exchanging data.

Examples include:

-   Laptops
-   Servers
-   Phones
-   Routers
-   Switches
-   Firewalls
-   Virtual Machines
-   Containers
-   Kubernetes Pods

The important idea is not the device itself.

The important idea is that devices can communicate.

------------------------------------------------------------------------

# A Production Example

Imagine TechCorp's infrastructure.

``` text
Employees

 Alice      Bob      Charlie
   │          │          │
   └──────────┼──────────┘
              │
        Office Network
              │
        Web Application
              │
          PostgreSQL
```

Employees send requests.

The web application processes them.

The database stores and returns data.

Communication is what makes the business function.

------------------------------------------------------------------------

# The Internet Is a Network of Networks

The Internet is not one giant computer.

Instead, it is millions of independent networks connected together.

``` text
Home Network
      │
ISP Network
      │
Regional Network
      │
Internet Backbone
      │
Cloud Provider
      │
Company Network
```

The word **Internet** literally comes from **interconnected networks**.

------------------------------------------------------------------------

# A Request's Journey

Throughout this chapter we'll follow one request:

``` text
https://mycompany.com
```

Eventually you'll understand every step of this journey:

``` text
User
 ↓
Browser
 ↓
Operating System
 ↓
DNS
 ↓
Network Interface
 ↓
Switch
 ↓
Router
 ↓
Internet
 ↓
Cloud Firewall
 ↓
Load Balancer
 ↓
Nginx
 ↓
Spring Boot
 ↓
PostgreSQL
 ↓
Response
```

By the end of Chapter 06 every box in this diagram will be familiar.

------------------------------------------------------------------------

# Why DevOps Engineers Must Understand Networking

Imagine you receive an alert:

    Production API is unreachable.

The application is running.

CPU is healthy.

Memory is healthy.

Disk is healthy.

Yet users cannot connect.

Possible causes include:

-   DNS
-   Routing
-   Firewall
-   Load Balancer
-   Wrong IP
-   Wrong Port
-   TLS
-   Kubernetes Service
-   Container Networking

Experienced DevOps engineers don't immediately blame the application.

Instead they ask:

> **Where did communication stop?**

That question drives production troubleshooting.

------------------------------------------------------------------------

# Key Takeaways

-   Networking exists to enable communication.
-   The Internet is a network of networks.
-   Every request follows a journey.
-   DevOps troubleshooting is often networking troubleshooting.
-   Understanding the journey is more valuable than memorizing
    definitions.

------------------------------------------------------------------------

# Next Part

In Part 2 we will answer:

-   What is a client?
-   What is a server?
-   What is a service?
-   What is a request?
-   What is a response?
-   How do browsers and servers communicate?
-   How does this map to a real production architecture?

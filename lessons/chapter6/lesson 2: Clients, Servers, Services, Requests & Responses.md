# Chapter 06 --- Networking Fundamentals

# Lesson 01 --- How the Internet Actually Works

## Part 2 --- Clients, Servers, Services, Requests & Responses

------------------------------------------------------------------------

# Learning Objectives

After this lesson you will understand:

-   What a client really is
-   What a server really is
-   What a service is
-   How requests and responses work
-   Why HTTP is stateless
-   How a real production architecture communicates

------------------------------------------------------------------------

# The First Conversation

When you visit:

``` text
https://mycompany.com
```

your browser is **starting a conversation**.

Every network interaction is simply one computer asking another computer
to do something.

------------------------------------------------------------------------

# What Is a Client?

A **client** is the device or program that **initiates communication**.

Examples:

-   Web Browser
-   Mobile App
-   curl
-   React Frontend
-   Another backend service
-   Kubernetes Pod

A client is not necessarily a person's computer.

In microservices, one server can become the client of another server.

Example:

``` text
Browser
   │
Request
   ▼
Nginx
   │
Request
   ▼
Spring Boot
   │
Request
   ▼
PostgreSQL
```

Spring Boot is a **server** for Nginx, but a **client** for PostgreSQL.

Roles depend on who starts the conversation.

------------------------------------------------------------------------

# What Is a Server?

A server is simply software waiting for requests.

Examples:

-   Nginx
-   Apache
-   Spring Boot
-   MySQL
-   PostgreSQL
-   Redis

Notice something important:

A server is **not** a powerful computer.

A laptop can run server software.

A cloud VM can run server software.

A Raspberry Pi can run server software.

The word "server" usually describes the software's role.

------------------------------------------------------------------------

# What Is a Service?

A service is a program that provides one specific capability.

Examples:

  Service      Purpose
  ------------ -------------------
  SSH          Remote login
  Nginx        Serve web traffic
  PostgreSQL   Store data
  Redis        Cache data

In Linux, services usually run continuously in the background.

Later in this chapter you'll inspect them using:

``` bash
systemctl
ss
ip
```

------------------------------------------------------------------------

# Request and Response

Communication follows a simple pattern.

``` text
Client
   │
   │ Request
   ▼
Server
   │
   │ Response
   ▼
Client
```

The client asks.

The server answers.

Examples:

Request:

``` http
GET /products
```

Response:

``` json
[
  {"id":1,"name":"Laptop"}
]
```

------------------------------------------------------------------------

# Stateless Communication

HTTP is called **stateless**.

That means each request is independent.

The server does not automatically remember previous requests.

Example:

``` text
Request #1
Login

✓ Completed

Request #2
View Dashboard

The server treats it as a brand-new request unless authentication
information (such as a session cookie or token) is sent again.
```

This design makes web applications scalable.

------------------------------------------------------------------------

# Production Example

A typical DevOps architecture:

``` text
User
  │
Browser
  │
Internet
  │
Load Balancer
  │
Nginx
  │
Spring Boot API
  │
Redis
  │
PostgreSQL
```

Who is the client?

-   Browser → client
-   Nginx → client (when calling Spring Boot)
-   Spring Boot → client (when calling Redis)
-   Spring Boot → client (when calling PostgreSQL)

The same application can be both a client and a server depending on the
conversation.

------------------------------------------------------------------------

# DevOps Insight

When troubleshooting, ask:

-   Which component started the request?
-   Which component should answer?
-   Did the request arrive?
-   Did the response leave?

These four questions solve many production incidents.

------------------------------------------------------------------------

# Common Mistakes

❌ Thinking a server must be a powerful machine.

❌ Thinking browsers are the only clients.

❌ Believing one machine can only be a client or only be a server.

------------------------------------------------------------------------

# Summary

-   Clients initiate communication.
-   Servers provide services.
-   Services perform specific jobs.
-   Communication uses requests and responses.
-   HTTP is stateless.
-   A single machine can be both a client and a server.

------------------------------------------------------------------------

# Next Part

Next we'll answer:

-   How does data become packets?
-   Why don't computers send one giant file?
-   What are frames, packets and segments?
-   What is encapsulation?

# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 09 --- Ports and Sockets (Part 4)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Service Is Running but Users Cannot Connect

The application process is running:

``` bash
ps aux | grep java
```

However:

``` bash
ss -tulpn
```

shows:

``` text
(no service listening on port 8080)
```

### Investigation

The application started, but it never opened the expected listening
socket.

Possible causes:

-   Startup failure
-   Wrong configuration
-   Different listening port
-   Application crash during initialization

------------------------------------------------------------------------

## Scenario 2 --- Wrong Listening Port

The documentation says the application uses:

``` text
Port 8080
```

But `ss` shows:

``` text
LISTEN 0.0.0.0:9090
```

Users connect to the wrong port.

The network is healthy---the application configuration is incorrect.

------------------------------------------------------------------------

## Scenario 3 --- Port Already in Use

A service refuses to start.

Running:

``` bash
ss -tulpn
```

reveals:

``` text
LISTEN 0.0.0.0:8080 users:(("java",pid=2150))
```

Another process already owns the port.

Linux allows only one application to bind to the same TCP port and
address at a time.

------------------------------------------------------------------------

## Scenario 4 --- Service Listening Only on Localhost

`ss -tulpn` shows:

``` text
LISTEN 127.0.0.1:8080
```

Local testing succeeds:

``` bash
curl http://127.0.0.1:8080
```

Remote users cannot connect.

The service is listening only on the loopback interface instead of the
external network interface.

------------------------------------------------------------------------

# Troubleshooting Workflow

``` text
Application running?
        │
        ▼
Is the correct port listening?
        │
        ▼
Which process owns the port?
        │
        ▼
Is it bound to the correct interface?
        │
        ▼
Verify client connectivity
```

------------------------------------------------------------------------

# Common Mistakes

## Assuming the Process Automatically Opens a Port

A running process is not always listening for network connections.

Always verify using `ss`.

------------------------------------------------------------------------

## Ignoring the Listening Address

There is a major difference between:

``` text
127.0.0.1:8080
```

and

``` text
0.0.0.0:8080
```

The first accepts only local connections.

The second accepts connections from other hosts (subject to firewall and
routing).

------------------------------------------------------------------------

## Forgetting Which Process Owns the Port

Multiple applications may run on the same server.

Always identify the owning process before stopping services.

------------------------------------------------------------------------

# Best Practices

-   Prefer `ss` over older networking tools.
-   Verify both the listening port and listening address.
-   Confirm which process owns each socket.
-   Investigate before restarting services.
-   Document application port assignments.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is a port?
2.  What is a socket?
3.  What does a listening socket do?

## Intermediate

1.  Explain the difference between a listening socket and an established
    connection.
2.  What are ephemeral ports?
3.  Why can multiple clients connect to the same server port?

## Advanced

1.  A process is running but `ss` shows no listening socket. What does
    that indicate?
2.  Why is a service bound to `127.0.0.1` unreachable from another
    computer?

------------------------------------------------------------------------

# Hands-on Lab

Run the following commands:

``` bash
ss -tulpn
```

``` bash
ss -tan
```

``` bash
lsof -i
```

Answer:

1.  Which services are listening?
2.  Which process owns port 22?
3.  Are any established TCP connections present?
4.  Which local ports are currently open?

------------------------------------------------------------------------

# Challenge Exercise

A Spring Boot application starts successfully, but users cannot access
it.

Describe an investigation using only:

-   `ss`
-   `lsof -i`
-   `ps`

Explain how each command contributes to identifying the problem.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 09 you understand:

-   Ports
-   Sockets
-   Listening sockets
-   Established connections
-   Local and peer addresses
-   Ephemeral ports
-   `ss`
-   `netstat`
-   `lsof -i`
-   Socket troubleshooting methodology

These concepts are fundamental for diagnosing application connectivity
issues on Linux systems.

------------------------------------------------------------------------

# Cheat Sheet

  Command            Purpose
  ------------------ -----------------------------------------------
  `ss -tulpn`        Show listening TCP/UDP sockets with processes
  `ss -tan`          Show TCP socket states
  `ss -tp`           Show TCP connections with process info
  `netstat -tulpn`   Legacy socket inspection
  `lsof -i`          Show processes using network sockets

------------------------------------------------------------------------

# What's Next

Lesson 10 --- Testing Network Services

You'll learn how to test real network services using `curl`, `wget`,
`nc`, and `telnet`, verify REST APIs, check open ports, and diagnose
application connectivity from a client's perspective.

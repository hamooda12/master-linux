# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 06 --- Routing (Part 4)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Missing Default Route

A Linux server can communicate with other devices on the local network
but cannot access the Internet.

Investigation:

``` bash
ip route
```

Output:

``` text
192.168.1.0/24 dev enp0s3
```

Notice that no **default** route exists.

### Root Cause

Linux knows how to reach the local network but has no route for all
other destinations.

------------------------------------------------------------------------

## Scenario 2 --- Wrong Default Gateway

Routing table:

``` text
default via 192.168.1.250
```

The actual router is:

``` text
192.168.1.1
```

Result:

-   Local communication works.
-   Remote communication fails.

Always verify the gateway address.

------------------------------------------------------------------------

## Scenario 3 --- Incorrect Static Route

A static route points to the wrong router.

Result:

-   Some networks are reachable.
-   Others always fail.

Checking the routing table quickly reveals the incorrect configuration.

------------------------------------------------------------------------

# Troubleshooting Workflow

Whenever routing problems are suspected:

``` text
Check interface status
        │
        ▼
Verify IP address
        │
        ▼
Inspect routing table
        │
        ▼
Verify default gateway
        │
        ▼
Check destination route
        │
        ▼
Test connectivity
```

Investigate methodically rather than making assumptions.

------------------------------------------------------------------------

# Common Mistakes

## Assuming DNS Is the Problem

If Linux cannot route packets, DNS is not the first thing to
investigate.

Routing occurs before application communication.

------------------------------------------------------------------------

## Forgetting the Default Gateway

Without a default gateway, Linux cannot reach networks outside the local
subnet.

------------------------------------------------------------------------

## Ignoring Route Specificity

Linux always chooses the most specific matching route.

Understanding prefix length is essential.

------------------------------------------------------------------------

# Best Practices

-   Verify the routing table before modifying applications.
-   Document custom static routes.
-   Keep gateway configurations consistent.
-   Test connectivity after routing changes.
-   Prefer evidence over assumptions during troubleshooting.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is routing?
2.  What is a routing table?
3.  What is a default gateway?

## Intermediate

1.  Explain the difference between a directly connected route and a
    static route.
2.  How does Linux choose which route to use?

## Advanced

1.  Explain Longest Prefix Match.
2.  A server reaches its local network but not the Internet. What
    routing issues would you investigate first?

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
ip route
```

Answer the following:

1.  What is your default gateway?
2.  Which interface is used?
3.  Which local network is directly connected?
4.  Is there more than one route?

------------------------------------------------------------------------

# Challenge Exercise

A Linux server has the following routing table:

``` text
default via 10.0.0.1

192.168.1.0/24 dev enp0s3

172.16.0.0/16 via 192.168.1.254
```

Determine which route Linux will use for each destination:

-   192.168.1.25
-   172.16.10.50
-   8.8.8.8

Explain your reasoning.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 06 you understand:

-   Routing
-   Routers
-   Default gateways
-   Routing tables
-   Directly connected routes
-   Static routes
-   Route selection
-   Longest Prefix Match
-   Linux routing fundamentals
-   Basic routing troubleshooting

These concepts are essential for diagnosing connectivity problems in
Linux and cloud environments.

------------------------------------------------------------------------

# Cheat Sheet

  Concept                Purpose
  ---------------------- ------------------------------------------
  Router                 Connects different networks
  Routing                Determines packet path
  Routing Table          List of routing rules
  Default Gateway        Route used when no specific route exists
  Direct Route           Local network communication
  Static Route           Administrator-defined route
  `ip route`             Display routing table
  Longest Prefix Match   Linux route selection rule

------------------------------------------------------------------------

# What's Next

Lesson 07 --- DNS (Domain Name System)

You'll learn how Linux converts domain names into IP addresses,
understand DNS records, inspect DNS configuration, and troubleshoot name
resolution problems using professional Linux tools.

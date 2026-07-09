# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 13 --- Network Configuration (Part 4)

------------------------------------------------------------------------

# Production Deployment Checklist

Before placing a Linux server into production, verify its network
configuration.

## Step 1 --- Verify the Interface

``` bash
ip link
```

Ensure the correct interface exists and is **UP**.

------------------------------------------------------------------------

## Step 2 --- Verify the IP Address

``` bash
ip addr
```

Confirm:

-   Correct IPv4/IPv6 address
-   Correct CIDR prefix
-   Address assigned to the correct interface

------------------------------------------------------------------------

## Step 3 --- Verify the Routing Table

``` bash
ip route
```

Confirm:

-   Default gateway exists.
-   Local routes are correct.

------------------------------------------------------------------------

## Step 4 --- Verify DNS

``` bash
cat /etc/resolv.conf
```

Confirm:

-   Correct DNS servers
-   No unexpected entries

------------------------------------------------------------------------

## Step 5 --- Test Connectivity

``` bash
ping 8.8.8.8
```

Tests basic IP connectivity.

Then test DNS:

``` bash
ping google.com
```

If the first succeeds but the second fails, investigate DNS.

------------------------------------------------------------------------

## Step 6 --- Verify the Application

Finally, verify the actual service.

Example:

``` bash
curl http://localhost:8080
```

A healthy network configuration is only valuable if the application also
responds correctly.

------------------------------------------------------------------------

# Complete Troubleshooting Workflow

``` text
Interface UP?
      │
      ▼
Correct IP Address?
      │
      ▼
Correct Routing?
      │
      ▼
DNS Working?
      │
      ▼
Firewall Allows Traffic?
      │
      ▼
Service Listening?
      │
      ▼
Application Responds?
```

This workflow combines the concepts from the entire networking chapter.

------------------------------------------------------------------------

# Common Mistakes

## Editing the Wrong Netplan File

Some systems contain multiple YAML files in:

``` text
/etc/netplan/
```

Always identify which file is actually used.

------------------------------------------------------------------------

## Forgetting to Test After Changes

Never assume a configuration works.

Always verify:

-   IP address
-   Gateway
-   DNS
-   Application connectivity

------------------------------------------------------------------------

## Locking Yourself Out

When changing networking on a remote server:

-   Prefer `netplan try`.
-   Keep an active SSH session open.
-   Verify connectivity before disconnecting.

------------------------------------------------------------------------

# Best Practices

-   Document static IP assignments.
-   Keep DNS configuration consistent.
-   Back up Netplan files before editing.
-   Test changes immediately.
-   Use a structured troubleshooting process instead of guessing.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What information is required for a Linux host to communicate on a
    network?
2.  What is DHCP?
3.  Why do servers often use static IP addresses?

## Intermediate

1.  Explain the purpose of Netplan.
2.  What does `netplan apply` do?
3.  Why should configuration changes always be verified?

## Advanced

1.  Describe a complete troubleshooting process for a Linux server that
    cannot reach the Internet.
2.  Why is `netplan try` recommended for remote administration?

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
ip link
ip addr
ip route
cat /etc/resolv.conf
```

If using Ubuntu:

``` bash
ls /etc/netplan
cat /etc/netplan/*.yaml
```

Answer:

1.  Which interface is active?
2.  Which IP address is assigned?
3.  What is the default gateway?
4.  Which DNS servers are configured?
5.  Is DHCP or static addressing used?

------------------------------------------------------------------------

# Challenge Exercise

A newly deployed Ubuntu server is unreachable after a network
configuration change.

Create a troubleshooting plan using only:

-   `ip link`
-   `ip addr`
-   `ip route`
-   `cat /etc/resolv.conf`
-   `cat /etc/netplan/*.yaml`
-   `sudo netplan try`

Explain what each step verifies and how it helps isolate the problem.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 13 you understand:

-   Dynamic and static network configuration
-   DHCP
-   Static IP addressing
-   Netplan
-   Persistent network configuration
-   Verifying interfaces, routes, and DNS
-   Safe deployment practices
-   Production troubleshooting workflow

You now have the knowledge to configure Linux networking for development
machines, cloud instances, and production servers.

------------------------------------------------------------------------

# Cheat Sheet

  Command                     Purpose
  --------------------------- ----------------------------
  `ip link`                   Show network interfaces
  `ip addr`                   Show IP addresses
  `ip route`                  Show routing table
  `cat /etc/resolv.conf`      Show DNS configuration
  `ls /etc/netplan`           List Netplan files
  `cat /etc/netplan/*.yaml`   View Netplan configuration
  `sudo netplan try`          Safely test configuration
  `sudo netplan apply`        Apply configuration

------------------------------------------------------------------------

# What's Next

The networking fundamentals chapter is complete. The next chapter builds
on these concepts by applying them to container networking,
virtualization, cloud infrastructure, and production DevOps
environments.

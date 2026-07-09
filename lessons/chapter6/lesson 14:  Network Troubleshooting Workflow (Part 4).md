# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 14 --- Network Troubleshooting Workflow (Part 4)

------------------------------------------------------------------------

# Complete Troubleshooting Checklist

When a networking issue is reported, investigate in this order:

## 1. Verify the Interface

``` bash
ip link
```

-   Does the interface exist?
-   Is it UP?

------------------------------------------------------------------------

## 2. Verify IP Configuration

``` bash
ip addr
```

-   Correct IP address?
-   Correct subnet prefix?

------------------------------------------------------------------------

## 3. Verify Routing

``` bash
ip route
```

-   Default gateway exists?
-   Static routes correct?

------------------------------------------------------------------------

## 4. Verify DNS

``` bash
cat /etc/resolv.conf
```

``` bash
dig example.com
```

-   Correct DNS server?
-   Domain resolves successfully?

------------------------------------------------------------------------

## 5. Verify Connectivity

``` bash
ping 8.8.8.8
```

``` bash
tracepath example.com
```

-   Reachable?
-   Packet loss?
-   Unexpected routing?

------------------------------------------------------------------------

## 6. Verify Firewall

``` bash
sudo ufw status
```

-   Firewall active?
-   Correct ports allowed?
-   Correct protocol?

------------------------------------------------------------------------

## 7. Verify Listening Services

``` bash
ss -tulpn
```

-   Application listening?
-   Correct port?
-   Correct interface?

------------------------------------------------------------------------

## 8. Verify the Application

``` bash
curl http://localhost:8080
```

Does the application return a healthy response?

------------------------------------------------------------------------

# The Complete Investigation Flow

``` text
Problem Report
       │
       ▼
ip link
       │
       ▼
ip addr
       │
       ▼
ip route
       │
       ▼
DNS
       │
       ▼
ping
       │
       ▼
tracepath
       │
       ▼
Firewall
       │
       ▼
ss
       │
       ▼
curl
       │
       ▼
Application Logs
```

Each layer depends on the layers beneath it.

------------------------------------------------------------------------

# Common Mistakes

## Restarting Services Too Early

Restarting a healthy application does not solve:

-   Wrong routing
-   DNS failures
-   Firewall blocks
-   Incorrect IP configuration

Always identify the failing layer first.

------------------------------------------------------------------------

## Making Multiple Changes at Once

If you change several settings simultaneously, it becomes difficult to
determine which change fixed---or caused---the problem.

Make one change, then verify the result.

------------------------------------------------------------------------

## Ignoring Evidence

Base every decision on command output, not assumptions.

------------------------------------------------------------------------

# Best Practices

-   Investigate from lower layers to higher layers.
-   Collect evidence before changing anything.
-   Verify each fix before continuing.
-   Document configuration changes.
-   Keep troubleshooting simple and systematic.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is the first networking command you would run when a server is
    unreachable?
2.  Why is `ping` usually performed before `curl`?
3.  What does `ss` show?

## Intermediate

1.  Explain a structured Linux networking troubleshooting workflow.
2.  Why should routing be verified before testing applications?
3.  How would you distinguish a DNS problem from a routing problem?

## Advanced

1.  A web application is unreachable. Describe your investigation from
    the network interface up to the application.
2.  Why is evidence-based troubleshooting critical in production
    environments?

------------------------------------------------------------------------

# Hands-on Lab

Perform a complete health check on your Linux machine:

``` bash
ip link
ip addr
ip route
cat /etc/resolv.conf
ping 8.8.8.8
ping google.com
tracepath google.com
sudo ufw status
ss -tulpn
curl http://localhost
```

For each command, write:

-   What it verifies.
-   Whether the result is healthy.
-   What your next step would be if it failed.

------------------------------------------------------------------------

# Final Challenge

A production server cannot be reached by users.

Using only the tools learned in this chapter, create a complete
investigation plan.

Your plan should include:

-   The order of commands.
-   What each command verifies.
-   Possible root causes.
-   How you would decide whether the issue is related to:
    -   Network interfaces
    -   IP addressing
    -   Routing
    -   DNS
    -   Firewall
    -   Listening services
    -   The application itself

------------------------------------------------------------------------

# Chapter Summary

Congratulations!

You have completed **Chapter 06 --- Linux Networking Fundamentals**.

You now understand:

-   Networking fundamentals
-   IP addressing
-   Network interfaces
-   Routing
-   DNS
-   Network connectivity
-   Ports and sockets
-   Service testing
-   SSH fundamentals
-   Linux firewall fundamentals
-   Network configuration
-   Professional troubleshooting workflows

These concepts form the networking foundation required for Docker,
Kubernetes, cloud platforms, DevOps, Linux system administration, and
production troubleshooting.

------------------------------------------------------------------------

# Chapter Cheat Sheet

  Command                  Purpose
  ------------------------ -----------------------------
  `ip link`                Show interfaces
  `ip addr`                Show IP addresses
  `ip route`               Show routing table
  `cat /etc/resolv.conf`   Show DNS servers
  `ping`                   Test connectivity
  `tracepath`              Show network path
  `dig`                    Query DNS
  `ss -tulpn`              Show listening sockets
  `curl`                   Test HTTP services
  `nc -zv`                 Test TCP ports
  `ssh`                    Secure remote login
  `scp`                    Secure file copy
  `sudo ufw status`        Show firewall status
  `sudo netplan apply`     Apply network configuration

------------------------------------------------------------------------


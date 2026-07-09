# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 13 --- Network Configuration (Part 3)

------------------------------------------------------------------------

# Production Troubleshooting

## Scenario 1 --- Wrong Static IP Address

A server is configured with:

``` text
192.168.2.50/24
```

The rest of the servers use:

``` text
192.168.1.0/24
```

### Symptoms

-   Cannot communicate with local servers.
-   Gateway cannot be reached.
-   Applications fail to connect.

### Investigation

Verify the assigned address:

``` bash
ip addr
```

------------------------------------------------------------------------

## Scenario 2 --- Missing Default Gateway

The interface has the correct IP address.

However:

``` bash
ip route
```

shows no default route.

### Result

-   Local communication works.
-   Internet access fails.
-   Remote APIs cannot be reached.

------------------------------------------------------------------------

## Scenario 3 --- Incorrect DNS Server

The network configuration specifies:

``` text
nameserver 192.168.1.250
```

The DNS server does not exist.

### Symptoms

-   `ping 8.8.8.8` succeeds.
-   `ping google.com` fails.

Verify DNS:

``` bash
cat /etc/resolv.conf
```

------------------------------------------------------------------------

## Scenario 4 --- Netplan Syntax Error

After editing a Netplan YAML file:

``` bash
sudo netplan apply
```

returns an error.

### Root Cause

YAML is indentation-sensitive.

Even one misplaced space can invalidate the configuration.

Always validate carefully and prefer:

``` bash
sudo netplan try
```

before applying permanent changes.

------------------------------------------------------------------------

# Troubleshooting Workflow

``` text
Verify interface (ip addr)
        │
        ▼
Verify routing (ip route)
        │
        ▼
Verify DNS (/etc/resolv.conf)
        │
        ▼
Review Netplan configuration
        │
        ▼
Apply and verify changes
```

------------------------------------------------------------------------

# Common Mistakes

## Mixing DHCP and Static Settings

Avoid defining static addresses while DHCP remains enabled unless you
intentionally understand the behavior.

------------------------------------------------------------------------

## Forgetting to Apply Changes

Editing a Netplan file alone changes nothing.

Run:

``` bash
sudo netplan apply
```

------------------------------------------------------------------------

## Not Verifying the Result

Always verify with:

``` bash
ip addr
ip route
```

after making configuration changes.

------------------------------------------------------------------------

# Best Practices

-   Use static addresses for production servers.
-   Use DHCP for most client systems.
-   Back up configuration files before editing.
-   Use `netplan try` when working remotely.
-   Verify every networking change before ending maintenance.

------------------------------------------------------------------------

# Interview Questions

## Beginner

1.  What is DHCP?
2.  What is a static IP address?
3.  Where are Netplan files stored?

## Intermediate

1.  Why do production servers commonly use static IP addresses?
2.  What is the difference between `netplan apply` and `netplan try`?

## Advanced

1.  A server can reach local hosts but not the Internet. Which
    configuration would you investigate first?
2.  Why is `netplan try` safer on remote servers?

------------------------------------------------------------------------

# Hands-on Lab

Run:

``` bash
ip addr
```

``` bash
ip route
```

``` bash
cat /etc/resolv.conf
```

If using Ubuntu with Netplan:

``` bash
ls /etc/netplan
```

Answer:

1.  Which interface has an IP address?
2.  What is the default gateway?
3.  Which DNS servers are configured?
4.  Does your system use DHCP or a static IP?

------------------------------------------------------------------------

# Challenge Exercise

A Linux server was reconfigured with a static IP.

After rebooting:

-   Local hosts are unreachable.
-   Internet access is unavailable.
-   DNS resolution fails.

Design a step-by-step investigation using only:

-   `ip addr`
-   `ip route`
-   `cat /etc/resolv.conf`
-   Netplan configuration files

Explain what you would verify at each step.

------------------------------------------------------------------------

# Lesson Summary

After completing Lesson 13 you understand:

-   DHCP
-   Static IP addressing
-   Netplan
-   Persistent network configuration
-   `netplan apply`
-   `netplan try`
-   Network configuration verification
-   Troubleshooting configuration problems

These skills are essential for configuring Linux servers in data
centers, cloud platforms, and production environments.

------------------------------------------------------------------------

# Cheat Sheet

  Command                  Purpose
  ------------------------ ---------------------------
  `ip addr`                Display IP addresses
  `ip route`               Display routing table
  `cat /etc/resolv.conf`   View DNS configuration
  `ls /etc/netplan`        List Netplan files
  `sudo netplan try`       Test configuration safely
  `sudo netplan apply`     Apply configuration

------------------------------------------------------------------------

# What's Next

Continue with the remaining networking roadmap, moving into advanced
Linux networking administration and practical production scenarios.

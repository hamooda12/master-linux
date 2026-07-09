# 📘 Chapter 06 --- Linux Networking Fundamentals

# Lesson 13 --- Network Configuration (Part 2)

------------------------------------------------------------------------

# Netplan

Modern Ubuntu systems use **Netplan** to configure networking.

Instead of editing many configuration files, administrators define
network settings in a YAML file.

Netplan then generates the appropriate configuration for the underlying
networking service.

------------------------------------------------------------------------

# Where Are Netplan Files Stored?

Netplan configuration files are located in:

``` text
/etc/netplan/
```

Typical files include:

``` text
01-network-manager-all.yaml

50-cloud-init.yaml
```

The exact filename depends on how Ubuntu was installed.

------------------------------------------------------------------------

# Viewing the Configuration

Display the configuration:

``` bash
cat /etc/netplan/*.yaml
```

Or edit a specific file:

``` bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

------------------------------------------------------------------------

# Basic Netplan Structure

Example:

``` yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

This tells Ubuntu:

-   Use interface `enp0s3`
-   Obtain an IPv4 address using DHCP

------------------------------------------------------------------------

# Static IP Configuration

Example:

``` yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

This configuration defines:

-   Static IP address
-   Default gateway
-   DNS servers

------------------------------------------------------------------------

# Applying Changes

After editing a Netplan file:

``` bash
sudo netplan apply
```

Ubuntu immediately applies the new configuration.

To test changes safely before committing them:

``` bash
sudo netplan try
```

If networking stops working, Netplan automatically rolls back unless you
confirm the configuration.

------------------------------------------------------------------------

# Verifying the Configuration

After applying changes, verify them using:

``` bash
ip addr
```

``` bash
ip route
```

``` bash
cat /etc/resolv.conf
```

Always verify that:

-   The interface has the correct IP address.
-   The default route exists.
-   DNS servers are configured correctly.

------------------------------------------------------------------------

# DHCP with Netplan

To enable DHCP again:

``` yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

This instructs Ubuntu to obtain its network configuration automatically.

------------------------------------------------------------------------

# Linux Perspective

Netplan does not directly configure the network.

Instead, it generates configuration for a backend renderer such as:

-   systemd-networkd
-   NetworkManager

The renderer then configures the interfaces.

------------------------------------------------------------------------

# Production Example

An administrator assigns a static IP to a production web server.

After editing the Netplan configuration and running:

``` bash
sudo netplan apply
```

they verify:

-   IP address
-   Default gateway
-   DNS
-   Application connectivity

before declaring the deployment successful.

------------------------------------------------------------------------

# Lesson Summary

You learned:

-   Netplan
-   Netplan configuration files
-   DHCP configuration
-   Static IP configuration
-   `netplan apply`
-   `netplan try`
-   Verifying network configuration

------------------------------------------------------------------------

# ➡️ Next Part

We'll complete network configuration with troubleshooting scenarios,
interview questions, hands-on labs, challenge exercises, and a complete
configuration cheat sheet.

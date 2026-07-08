# Chapter 06 — Networking Fundamentals

# Lesson 12 — Linux Network Configuration for DevOps Engineers (Part 1)

> **Objective:** Build a deep understanding of how Linux represents networking before learning routing, firewalls, Docker, or Kubernetes.

---

# Learning Objectives

By the end of this lesson you will be able to:

- Understand why Linux networking knowledge is essential for DevOps engineers.
- Explain how Linux processes incoming and outgoing network traffic.
- Understand what a network interface is.
- Identify different types of Linux network interfaces.
- Use the modern `ip` command to inspect network configuration.
- Read and understand the output of `ip addr`.
- Diagnose basic interface-related networking problems.

---

# Why Linux Networking Matters

Every request sent to your application first arrives at the **Linux kernel**.

Before **Nginx**, **Apache**, **Docker**, **Kubernetes**, or your application can process a packet, Linux must decide:

- Which network interface received it.
- Whether the interface is operational.
- Which IP address the packet is destined for.
- Which routing table entry should be used.
- Whether a socket is listening on the destination port.
- Which process should receive the data.

In other words:

```
Internet
    │
    ▼
Network Card (NIC)
    │
    ▼
Linux Kernel
    │
    ├── Validate Interface
    ├── Check Routing Table
    ├── Process TCP/UDP
    ├── Find Listening Socket
    │
    ▼
Application (Nginx, SSH, Spring Boot, etc.)
```

When a production incident occurs, experienced DevOps engineers usually verify the operating system before blaming the application.

Typical checklist:

- Is the interface up?
- Does it have the correct IP address?
- Is there a default gateway?
- Can DNS resolve hostnames?
- Is the application listening on the expected port?

---

# How Linux Sees the Network

Linux networking is built as a stack of layers.

```
Application
     │
     ▼
Socket
     │
     ▼
TCP / UDP
     │
     ▼
IP
     │
     ▼
Network Interface
     │
     ▼
Network Card (Physical or Virtual)
```

Applications **never communicate directly with the network card**.

Instead:

1. The application writes data to a socket.
2. The kernel encapsulates it into TCP or UDP.
3. An IP packet is created.
4. Linux consults the routing table.
5. The correct interface is selected.
6. The NIC transmits the frame.

The reverse process occurs for incoming packets.

---

# What Is a Network Interface?

A **network interface** is Linux's representation of a connection to a network.

Common interfaces include:

| Interface | Purpose |
|----------|---------|
| `enp0s3` | Ethernet |
| `wlan0` | Wi‑Fi |
| `lo` | Loopback |
| `docker0` | Docker bridge |
| `tun0` | VPN tunnel |

Each interface has:

- A unique name
- One or more IP addresses
- A MAC address
- An operational state (`UP` or `DOWN`)
- Statistics such as RX/TX packets, errors, and drops

---

# The `ip` Command

Older Linux systems used `ifconfig`.

Modern Linux uses the **`ip` utility** from the **iproute2** package.

Advantages include:

- Unified interface management
- Routing management
- Address management
- Neighbor (ARP) management
- Tunnel management

The `ip` command communicates directly with the Linux kernel using **Netlink**, making it the recommended networking tool on modern Linux systems.

---

# Deep Dive: `ip addr`

Display interface information:

```bash
ip addr
```

Example output:

```text
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP
    link/ether 52:54:00:12:34:56
    inet 192.168.1.20/24
    inet6 fe80::5054:ff:fe12:3456/64
```

## Understanding the Output

### `BROADCAST`

The interface can send broadcast frames.

Example:

A DHCP Discover packet is broadcast because the client does not yet know the DHCP server.

---

### `MULTICAST`

The interface supports multicast traffic.

Used by protocols such as:

- mDNS
- IPv6 Neighbor Discovery
- Routing protocols

---

### `UP`

The interface has been administratively enabled.

Usually enabled with:

```bash
sudo ip link set enp0s3 up
```

---

### `LOWER_UP`

The physical or virtual connection is functioning correctly.

If you see:

```
UP
```

but **not** `LOWER_UP`, possible causes include:

- Ethernet cable unplugged
- Switch port disabled
- Virtual NIC disconnected
- Hypervisor networking issue

---

### `mtu 1500`

The **Maximum Transmission Unit (MTU)**.

It defines the maximum packet size that can be transmitted without fragmentation.

Common values:

| Network | MTU |
|---------|----:|
| Ethernet | 1500 |
| VPN | 1400–1450 |
| Jumbo Frames | 9000 |

An incorrect MTU can cause:

- Slow connections
- Packet fragmentation
- Hanging SSH sessions
- Kubernetes networking issues

---

### `state UP`

The interface is operational and ready to transmit traffic.

If the state is `DOWN`, the interface cannot send or receive packets.

---

### `link/ether`

The MAC address of the interface.

Example:

```
52:54:00:12:34:56
```

The MAC address is used for communication within the local network.

---

### `inet`

The IPv4 address and prefix length.

Example:

```
192.168.1.20/24
```

This means:

- IPv4 address: `192.168.1.20`
- Prefix: `/24`
- Network: `192.168.1.0/24`

---

### `inet6`

The IPv6 address assigned to the interface.

Example:

```
fe80::5054:ff:fe12:3456/64
```

Linux systems commonly have IPv6 enabled even if it is not actively used.

---

# Production Example

Suppose your web server cannot be reached.

You run:

```bash
ip addr
```

Output:

```text
state DOWN
```

Even if:

- Nginx is running
- Firewall rules are correct
- DNS records are valid

the server will still be unreachable because the interface itself is not operational.

Always verify the interface before troubleshooting higher layers.

---

# Hands-On Exercises

## Exercise 1

Run:

```bash
ip addr
```

Identify:

- Your active interface
- Its IPv4 address
- Its MAC address
- Whether IPv6 is configured

---

## Exercise 2

Compare:

- `lo`
- Your Ethernet or Wi‑Fi interface

Questions:

- Which one has a physical MAC address?
- Which one always exists?
- Which one remains available even without a network cable?

---

## Exercise 3

Predict what happens if your interface loses its IPv4 address.

Consider:

- SSH access
- Internet connectivity
- Communication with other machines
- Local applications

---

## Exercise 4

Explain why `localhost` continues to work even if the network cable is unplugged.

Hint:

The loopback interface (`lo`) is an internal virtual interface handled entirely by the Linux kernel.

---

# Key Takeaways

- Every network packet passes through the Linux kernel before reaching an application.
- Applications communicate through sockets, not directly with network hardware.
- Network interfaces represent physical or virtual network connections.
- The `ip` command is the modern standard for Linux networking.
- `ip addr` provides essential information such as IP addresses, MAC addresses, interface state, and MTU.
- Always verify interface status before troubleshooting firewalls or applications.

---

# Coming in Part 2

- `ip link` in depth
- Interface states
- DHCP vs Static IP
- Default gateway
- `ip route` explained line by line
- `/etc/hosts`
- `/etc/resolv.conf`
- `hostname` and `hostnamectl`
- `nmcli`
- Production troubleshooting labs

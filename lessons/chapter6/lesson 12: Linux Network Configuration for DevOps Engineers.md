
# Chapter 06 — Networking Fundamentals

# Lesson 12 — Linux Network Configuration for DevOps Engineers

> **Objective:** Learn how Linux represents, configures, and troubleshoots networking at the operating-system level before moving into routing, firewalls, reverse proxies, Docker, Kubernetes, and cloud networking.

---

# Why This Lesson Matters

Every production network issue eventually touches the operating system.

Before you blame:

- Nginx
- Docker
- Kubernetes
- AWS Security Groups
- DNS
- Load Balancer
- Backend application
- Database

you must first answer these Linux-level questions:

```text
Does the server have an IP address?
Is the interface UP?
Is the physical/virtual link working?
Does Linux know where to send packets?
Can Linux resolve hostnames?
Is the application listening?
Is it listening on the correct address?
```

If Linux networking is broken, everything above it will fail.

---

# Big Picture: How Linux Sends Network Traffic

When an application sends data, it does not talk directly to the network card.

Example:

```text
Spring Boot Application
        │
        ▼
Socket
        │
        ▼
Linux Kernel
        │
        ▼
TCP / UDP
        │
        ▼
IP Layer
        │
        ▼
Routing Decision
        │
        ▼
Network Interface
        │
        ▼
Network Card / Virtual Adapter
        │
        ▼
Switch / Router / Internet
```

The Linux kernel is responsible for:

- Managing network interfaces
- Assigning IP addresses
- Maintaining the routing table
- Sending packets through the correct interface
- Receiving packets from the network
- Delivering traffic to the correct process/socket

---

# What Is a Network Interface?

A network interface is Linux's representation of a network connection.

Think of it as a **door** used by packets to enter or leave the machine.

Examples:

```text
lo        → loopback interface
eth0      → old-style Ethernet interface
enp0s3    → modern Ethernet interface
ens33     → common VM Ethernet interface
wlan0     → Wi-Fi interface
docker0   → Docker bridge interface
vethxxxx  → virtual Ethernet pair
tun0      → VPN tunnel interface
br-xxxx   → Linux bridge / Docker network bridge
```

To Linux, all of these are network interfaces.

Some are physical.

Some are virtual.

But the kernel manages them using the same networking system.

---

# Physical vs Virtual Interfaces

## Physical Interfaces

Physical interfaces are connected to real hardware.

Examples:

```text
Ethernet card
Wi-Fi adapter
USB network adapter
```

Typical names:

```text
eth0
enp0s3
ens33
eno1
wlan0
```

## Virtual Interfaces

Virtual interfaces are created by software.

Examples:

```text
lo        → loopback
docker0   → Docker bridge
vethxxx   → container connection
tun0      → VPN tunnel
br-xxx    → bridge network
```

DevOps engineers see virtual interfaces all the time when working with:

- Docker
- Kubernetes
- VPNs
- Virtual machines
- Cloud networking
- Linux bridges

---

# The Loopback Interface: `lo`

Every Linux system has a loopback interface:

```text
lo
```

Its main IPv4 address is:

```text
127.0.0.1
```

Also known as:

```text
localhost
```

The loopback interface allows the machine to communicate with itself.

Example:

```bash
curl http://127.0.0.1:8080
```

This request does **not** leave the machine.

It follows this path:

```text
curl
 │
 ▼
Linux Kernel
 │
 ▼
lo interface
 │
 ▼
Linux Kernel
 │
 ▼
Application listening on 127.0.0.1:8080
```

No switch.

No router.

No Wi-Fi.

No internet.

Everything stays inside the same Linux machine.

---

# Why `localhost` Can Be Dangerous in Production

Suppose your backend runs on port `8080`.

You test locally:

```bash
curl http://localhost:8080
```

It works.

But users cannot access:

```text
http://server-ip:8080
```

Possible reason:

The application is listening only on:

```text
127.0.0.1:8080
```

That means:

```text
Only this same machine can access it.
```

Remote clients cannot.

For remote access, the application usually needs to listen on:

```text
0.0.0.0:8080
```

or on the server's real IP address.

This is one of the most common DevOps troubleshooting issues.

---

# The `ip` Command Suite

Modern Linux uses the `ip` command from the `iproute2` package.

Old Linux tutorials often use:

```bash
ifconfig
route
netstat
```

Modern replacements:

| Old Command | Modern Command |
|------------|----------------|
| `ifconfig` | `ip addr`, `ip link` |
| `route` | `ip route` |
| `netstat` | `ss` |

Why modern Linux prefers `ip`:

- More accurate
- More powerful
- Supports modern kernel networking features
- Works better with virtual networking
- Used in production Linux systems

---

# Command 1: `ip link`

## What It Does

```bash
ip link
```

Shows network interfaces at the link layer.

It answers:

```text
What network interfaces does Linux know about?
Are they UP or DOWN?
What is their MAC address?
What is their MTU?
Is the physical/virtual link active?
```

It does **not** focus on IP addresses.

For IP addresses, use:

```bash
ip addr
```

---

# Example: `ip link`

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:6f:15:45 brd ff:ff:ff:ff:ff:ff
```

---

# `ip link` Field-by-Field

## Interface Index

```text
2:
```

Linux gives every interface an internal number.

Example:

```text
1 → lo
2 → enp0s3
3 → docker0
```

This number is mainly used internally by the kernel.

---

## Interface Name

```text
enp0s3
```

This is the interface name.

Older systems used names like:

```text
eth0
eth1
```

Modern systems use predictable names like:

```text
enp0s3
ens33
eno1
```

---

# Why Linux Stopped Using Only `eth0`

Old naming caused problems.

Imagine a server with two NICs:

```text
NIC A
NIC B
```

Before reboot:

```text
NIC A = eth0
NIC B = eth1
```

After reboot, the order could change:

```text
NIC A = eth1
NIC B = eth0
```

That is dangerous.

Firewall rules, routing rules, and automation could apply to the wrong interface.

Modern predictable names are based on hardware location, making them more stable.

---

# Interface Flags

Example:

```text
<BROADCAST,MULTICAST,UP,LOWER_UP>
```

These flags describe capabilities and current state.

---

## `BROADCAST`

The interface supports broadcast traffic.

Broadcast means sending to everyone on the local network.

Example use case:

```text
ARP: Who has 192.168.1.10?
```

This is sent to:

```text
ff:ff:ff:ff:ff:ff
```

Every device on the local network receives it.

Only the correct device replies.

---

## `MULTICAST`

The interface supports multicast traffic.

Multicast means:

```text
One sender → selected group of receivers
```

Used by:

- Some service discovery systems
- Routing protocols
- Streaming systems
- Some Kubernetes/networking components

---

## `UP`

The interface is administratively enabled.

Meaning:

```text
Linux is allowed to use this interface.
```

You can bring an interface down:

```bash
sudo ip link set enp0s3 down
```

Bring it back up:

```bash
sudo ip link set enp0s3 up
```

---

## `LOWER_UP`

This means the lower-layer connection is active.

For physical Ethernet, it usually means:

```text
Cable connected and link detected
```

For virtual interfaces, it means the virtual link is active.

Important difference:

```text
UP       → Linux enabled the interface
LOWER_UP → The actual link is working
```

An interface can be `UP` but not `LOWER_UP`.

Example causes:

- Ethernet cable unplugged
- Switch port disabled
- Hypervisor virtual NIC issue
- Cloud network attachment issue

---

# `mtu`

Example:

```text
mtu 1500
```

MTU means:

```text
Maximum Transmission Unit
```

It is the largest packet/frame payload size the interface can send without fragmentation.

Common value:

```text
1500 bytes
```

Why DevOps engineers care:

Wrong MTU can cause strange problems:

- VPN traffic works partially
- HTTPS hangs
- Kubernetes pods cannot communicate correctly
- Large responses fail but small pings work

---

# `qdisc`

Example:

```text
qdisc fq_codel
```

`qdisc` means:

```text
Queueing Discipline
```

When packets are waiting to be sent, Linux needs a queue.

`qdisc` controls how packets wait and are scheduled.

Most DevOps engineers do not modify this daily, but you should recognize it.

---

# `state`

Example:

```text
state UP
```

Common states:

```text
UP       → interface operational
DOWN     → interface disabled
UNKNOWN  → often seen on loopback
```

---

# `qlen`

Example:

```text
qlen 1000
```

Queue length.

It tells how many packets can wait in the transmit queue.

---

# Command 2: `ip addr`

## What It Does

```bash
ip addr
```

Shows network interfaces **and their IP addresses**.

It answers:

```text
Which IP addresses are assigned to this machine?
Which interface owns each IP?
Is the address IPv4 or IPv6?
Is the address dynamic or static?
What subnet is the interface in?
```

---

# Example: `ip addr`

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:6f:15:45 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.120/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 81234sec preferred_lft 81234sec
    inet6 fe80::a00:27ff:fe6f:1545/64 scope link
```

---

# `link/ether`

Example:

```text
link/ether 08:00:27:6f:15:45
```

This is the MAC address of the interface.

MAC address works at Layer 2.

It is used inside the local network.

IP answers:

```text
Which machine/network destination?
```

MAC answers:

```text
Which local interface should receive the frame?
```

---

# Broadcast MAC

Example:

```text
brd ff:ff:ff:ff:ff:ff
```

This is the broadcast MAC address.

It means:

```text
Send to everyone on the local network segment.
```

ARP uses this when it does not know the destination MAC.

---

# `inet`

Example:

```text
inet 192.168.1.120/24
```

This is the IPv4 address.

Breakdown:

```text
192.168.1.120 → IP address
/24           → prefix length
```

`/24` means:

```text
255.255.255.0
```

So the local network is:

```text
192.168.1.0/24
```

Usable host range is usually:

```text
192.168.1.1 - 192.168.1.254
```

Broadcast address:

```text
192.168.1.255
```

---

# Why `/24` Matters

If your server has:

```text
192.168.1.120/24
```

then Linux considers these local:

```text
192.168.1.1
192.168.1.50
192.168.1.200
```

But this is remote:

```text
192.168.2.10
```

For remote networks, Linux sends packets to the gateway.

This is why IP prefix matters for routing.

---

# `brd`

Example:

```text
brd 192.168.1.255
```

This is the IPv4 broadcast address for that subnet.

It is used when a packet should reach all hosts in the local subnet.

---

# `scope`

Example:

```text
scope global
```

Scope tells where the address is valid.

Common scopes:

```text
host    → valid only inside this machine
link    → valid only on local link
global  → usable beyond local host
```

Example:

```text
127.0.0.1 scope host
```

Only valid inside this machine.

Example:

```text
192.168.1.120 scope global
```

Usable for normal network communication.

---

# `dynamic`

Example:

```text
dynamic
```

This means the IP address was assigned automatically, usually by DHCP.

If there is no `dynamic`, it may be statically configured.

---

# `valid_lft`

Example:

```text
valid_lft 81234sec
```

Valid lifetime.

How long this address remains valid.

For DHCP addresses, this decreases over time and is renewed.

---

# `preferred_lft`

Example:

```text
preferred_lft 81234sec
```

Preferred lifetime.

How long Linux prefers using this address for new connections.

---

# `inet6`

Example:

```text
inet6 fe80::a00:27ff:fe6f:1545/64 scope link
```

This is an IPv6 address.

`fe80::/10` addresses are link-local.

They work only on the local network link.

You will often see IPv6 even if you did not configure it manually.

---

# Command 3: `ip route`

## What It Does

```bash
ip route
```

Shows the Linux routing table.

The routing table answers:

```text
Where should Linux send packets?
Which interface should be used?
Which gateway should receive remote traffic?
```

Example:

```text
default via 192.168.1.1 dev enp0s3 proto dhcp metric 100
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.120 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
```

---

# `default via`

Example:

```text
default via 192.168.1.1 dev enp0s3
```

This is the default gateway.

Meaning:

```text
If Linux does not have a more specific route, send packets to 192.168.1.1 through enp0s3.
```

Without a default route, the machine may communicate only with local networks.

---

# Local Route

Example:

```text
192.168.1.0/24 dev enp0s3
```

Meaning:

```text
The network 192.168.1.0/24 is directly reachable through enp0s3.
```

No gateway needed.

The destination is on the same LAN.

---

# Docker Route

Example:

```text
172.17.0.0/16 dev docker0
```

Docker often creates its own network route.

Meaning:

```text
Traffic to Docker container network goes through docker0.
```

This is why Docker changes the routing table.

---

# How Linux Chooses a Route

Linux uses the most specific matching route.

This is called:

```text
Longest Prefix Match
```

Example:

```text
default via 192.168.1.1
192.168.1.0/24 dev enp0s3
192.168.1.50/32 dev special0
```

For destination:

```text
192.168.1.50
```

Linux chooses:

```text
192.168.1.50/32
```

because it is the most specific.

For destination:

```text
192.168.1.80
```

Linux chooses:

```text
192.168.1.0/24
```

For destination:

```text
8.8.8.8
```

Linux chooses:

```text
default via 192.168.1.1
```

---

# Command 4: `hostname`

```bash
hostname
```

Shows the current system hostname.

Example:

```text
web-server-01
```

The hostname identifies the machine by name.

---

# Command 5: `hostnamectl`

```bash
hostnamectl
```

Shows more detailed host information.

Example fields:

```text
Static hostname
Operating System
Kernel
Architecture
Virtualization
```

Set hostname:

```bash
sudo hostnamectl set-hostname web-server-01
```

---

# `/etc/hosts`

File:

```text
/etc/hosts
```

Example:

```text
127.0.0.1 localhost
192.168.1.50 database
10.0.0.20 internal-api
```

Linux checks `/etc/hosts` before DNS in many common configurations.

So if you run:

```bash
ping database
```

Linux may resolve it to:

```text
192.168.1.50
```

without asking DNS.

---

# Why `/etc/hosts` Matters in DevOps

Common uses:

- Local testing
- Temporary overrides
- Internal lab environments
- Emergency workaround during DNS issues

Danger:

A wrong `/etc/hosts` entry can make an app connect to the wrong server.

Example:

```text
10.0.0.5 database
```

But the real database moved to:

```text
10.0.0.9
```

Your application may still connect to the old server.

---

# `/etc/resolv.conf`

File:

```text
/etc/resolv.conf
```

Example:

```text
nameserver 127.0.0.53
options edns0 trust-ad
search localdomain
```

This file tells Linux which DNS resolver to use.

On Ubuntu, you often see:

```text
127.0.0.53
```

This usually means `systemd-resolved` is handling DNS locally.

---

# DNS Troubleshooting Pattern

Run:

```bash
ping 8.8.8.8
```

Then:

```bash
ping google.com
```

Interpretation:

## Case 1

```text
ping 8.8.8.8 works
ping google.com fails
```

Likely DNS problem.

## Case 2

```text
ping 8.8.8.8 fails
ping gateway fails
```

Likely local network or gateway problem.

## Case 3

```text
ping google.com works
curl app fails
```

DNS is probably not the issue.

Move to TCP, firewall, TLS, HTTP, or application debugging.

---

# NetworkManager and `nmcli`

Ubuntu desktop and many servers use NetworkManager.

Command:

```bash
nmcli
```

Show devices:

```bash
nmcli device status
```

Example:

```text
DEVICE   TYPE      STATE      CONNECTION
enp0s3   ethernet  connected  Wired connection 1
lo       loopback  unmanaged  --
```

Show connections:

```bash
nmcli connection show
```

NetworkManager manages:

- IP addressing
- DNS
- gateways
- Wi-Fi
- connection profiles

---

# Command 6: `ss -tulpen`

## What It Does

```bash
ss -tulpen
```

Shows listening sockets and network connections.

Breakdown:

```text
-t  TCP
-u  UDP
-l  listening sockets
-p  process using socket
-e  extended information
-n  numeric addresses/ports
```

Example:

```text
tcp LISTEN 0 4096 127.0.0.1:8080 0.0.0.0:* users:(("java",pid=1234,fd=9))
tcp LISTEN 0 511 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=900,fd=6))
```

---

# `127.0.0.1` vs `0.0.0.0`

This is critical.

## Listening on `127.0.0.1`

```text
127.0.0.1:8080
```

Only local machine can connect.

Remote users cannot.

## Listening on `0.0.0.0`

```text
0.0.0.0:8080
```

Application accepts connections on all IPv4 interfaces.

Remote users may connect if firewall and routing allow it.

---

# Production Example: App Works Locally but Not Remotely

Local test:

```bash
curl http://localhost:8080
```

Works.

Remote test:

```bash
curl http://server-ip:8080
```

Fails.

Check:

```bash
ss -tulpen | grep 8080
```

If output shows:

```text
127.0.0.1:8080
```

then the app is listening only locally.

Fix depends on the application.

For Spring Boot:

```properties
server.address=0.0.0.0
server.port=8080
```

---

# Troubleshooting Scenario 1: Server Has No Internet

Symptoms:

```text
curl https://example.com fails
```

Step 1: Check interface

```bash
ip addr
```

Look for:

- correct IP
- interface UP
- LOWER_UP

Step 2: Check route

```bash
ip route
```

Look for:

```text
default via ...
```

Step 3: Test gateway

```bash
ping <gateway-ip>
```

Step 4: Test internet IP

```bash
ping 8.8.8.8
```

Step 5: Test DNS

```bash
ping google.com
```

Decision:

```text
Gateway fails      → local network issue
8.8.8.8 fails      → routing/internet issue
google.com fails   → DNS issue
```

---

# Troubleshooting Scenario 2: SSH Not Working

Check service:

```bash
systemctl status ssh
```

Check listening port:

```bash
ss -tulpen | grep :22
```

Check IP:

```bash
ip addr
```

Check firewall:

```bash
sudo ufw status
```

Possible causes:

- sshd stopped
- SSH not listening
- firewall blocking port 22
- wrong server IP
- cloud firewall/security group blocking traffic
- server has no route back to client

---

# Troubleshooting Scenario 3: DNS Broken

Symptoms:

```bash
ping 8.8.8.8
```

works.

But:

```bash
ping google.com
```

fails.

Check:

```bash
cat /etc/resolv.conf
```

Also check:

```bash
resolvectl status
```

Possible causes:

- wrong DNS server
- broken local resolver
- VPN changed DNS
- NetworkManager profile misconfigured
- `/etc/hosts` overriding expected DNS

---

# Troubleshooting Scenario 4: Wrong IP Address

Check:

```bash
ip addr
```

Maybe the server got an IP from the wrong network.

Example expected:

```text
10.0.1.20/24
```

Actual:

```text
192.168.1.44/24
```

Possible causes:

- wrong DHCP server
- wrong VLAN
- wrong VM network adapter
- wrong cloud subnet
- static configuration mistake

---

# Troubleshooting Scenario 5: Interface Is UP but No Connectivity

Check:

```bash
ip link
```

If you see:

```text
UP
```

but not:

```text
LOWER_UP
```

then Linux enabled the interface but the lower-layer link is not active.

Possible causes:

- unplugged cable
- disabled switch port
- VM adapter disconnected
- cloud ENI/network interface detached
- hypervisor issue

---

# Hands-On Lab

## Lab 1: Identify Your Interfaces

Run:

```bash
ip link
```

Answer:

```text
What interfaces exist?
Which one is loopback?
Which one is your main network interface?
Do you see Docker or virtual interfaces?
```

---

## Lab 2: Identify Your IP Address

Run:

```bash
ip addr
```

Answer:

```text
What is your IPv4 address?
What prefix length does it use?
Is it dynamic?
Does it have IPv6?
```

---

## Lab 3: Find Your Default Gateway

Run:

```bash
ip route
```

Answer:

```text
What is your default gateway?
Which interface is used to reach it?
```

---

## Lab 4: Test DNS vs Routing

Run:

```bash
ping 8.8.8.8
ping google.com
```

Answer:

```text
If the first works and the second fails, what is broken?
```

---

## Lab 5: Check Listening Services

Run:

```bash
ss -tulpen
```

Answer:

```text
Which ports are listening?
Which processes own them?
Are they listening on 127.0.0.1 or 0.0.0.0?
```

---

# Common Mistakes

## Mistake 1: Thinking `ip link` Shows IP Addresses

`ip link` shows interfaces.

Use:

```bash
ip addr
```

for addresses.

---

## Mistake 2: Ignoring `LOWER_UP`

`UP` alone is not enough.

You want to know whether the actual link is active.

---

## Mistake 3: Assuming Localhost Means Remote Access Works

This works only locally:

```bash
curl http://localhost:8080
```

Remote users need the service to listen on the server IP or `0.0.0.0`.

---

## Mistake 4: Blaming DNS Too Early

If IP connectivity fails, DNS is not the first problem.

Always test:

```bash
ping 8.8.8.8
```

before blaming hostname resolution.

---

## Mistake 5: Forgetting the Default Route

Without a default route, the server may only reach local networks.

Check:

```bash
ip route
```

---

# Best Practices

- Learn `ip`, not only old commands like `ifconfig`.
- Always check `ip addr` before debugging higher layers.
- Always check `ip route` when traffic cannot leave the server.
- Always compare local access vs remote access.
- Know the difference between `127.0.0.1` and `0.0.0.0`.
- Document server IPs, gateways, DNS, and interfaces.
- Be careful when editing `/etc/hosts`.
- Know that Docker and Kubernetes add virtual interfaces and routes.
- Do not restart services blindly before checking network state.

---

# DevOps Mental Model

When debugging Linux networking, think in this order:

```text
1. Interface exists?
2. Interface UP?
3. LOWER_UP active?
4. IP address assigned?
5. Correct subnet?
6. Default gateway exists?
7. Can reach gateway?
8. Can reach external IP?
9. Can resolve DNS?
10. Is service listening?
11. Is service listening on correct address?
12. Is firewall allowing traffic?
```

This order prevents guessing.

---

# Command Cheat Sheet

```bash
# Show interfaces
ip link

# Show IP addresses
ip addr

# Show routing table
ip route

# Show hostname
hostname

# Show host details
hostnamectl

# Set hostname
sudo hostnamectl set-hostname web-server-01

# Show local host mappings
cat /etc/hosts

# Show DNS resolver config
cat /etc/resolv.conf

# Show NetworkManager device status
nmcli device status

# Show NetworkManager connections
nmcli connection show

# Show listening TCP/UDP sockets
ss -tulpen

# Test IP connectivity
ping 8.8.8.8

# Test DNS resolution + connectivity
ping google.com

# Test local application
curl http://localhost:8080

# Test remote-style access from same server
curl http://SERVER_IP:8080
```

---

# Interview Questions

1. What is a Linux network interface?
2. What is the difference between a physical and virtual interface?
3. What is the loopback interface?
4. Why can `localhost` work while remote access fails?
5. What is the difference between `ip link` and `ip addr`?
6. What does `LOWER_UP` mean?
7. What is MTU?
8. What is the purpose of a default gateway?
9. What does `ip route` show?
10. What is `/etc/hosts` used for?
11. What is `/etc/resolv.conf` used for?
12. Why might `ping 8.8.8.8` work but `ping google.com` fail?
13. What is the difference between listening on `127.0.0.1` and `0.0.0.0`?
14. How do you check which process is listening on a port?
15. What is the first thing you check when a server has no network access?

---

# Final Summary

Linux networking starts with interfaces, addresses, routes, DNS, and sockets.

Before debugging advanced systems like Nginx, Docker, Kubernetes, or cloud load balancers, a DevOps engineer must understand what Linux itself is doing.

The most important commands from this lesson are:

```bash
ip link
ip addr
ip route
ss -tulpen
cat /etc/hosts
cat /etc/resolv.conf
nmcli device status
```

Master these commands deeply and you will be able to diagnose many real production networking problems without guessing.

---

# Next Lesson

**Lesson 13 — Routing Basics for DevOps Engineers**

In the next lesson, we will go deeper into:

- What routing really means
- Why default gateways exist
- How Linux chooses routes
- Longest Prefix Match
- Static routes
- Local vs remote networks
- `tracepath` and `traceroute`
- Real production routing failures

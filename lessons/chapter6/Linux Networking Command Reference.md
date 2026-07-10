# 🐧 Linux Networking Command Reference v2.0

> **Companion Guide for Chapter 06 -- Linux Networking Fundamentals**
>
> This reference covers the networking commands learned throughout
> Chapter 06 and focuses on real Linux/DevOps troubleshooting.

------------------------------------------------------------------------

# 🌐 Interface Inspection

## `ip link`

Purpose: Show or manage network interfaces.

``` bash
ip link
ip link show
ip link show wlo1
ip link set wlo1 up
ip link set wlo1 down
```

Use for: - Verify interface exists - Check UP/DOWN state -
Enable/disable interface

------------------------------------------------------------------------

## `ip addr`

Purpose: Show IP addresses.

``` bash
ip addr
ip addr show
ip addr show wlo1
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0
```

------------------------------------------------------------------------

# 🛣 Routing

## `ip route`

``` bash
ip route
ip route add 10.20.0.0/16 via 192.168.1.254
ip route add default via 192.168.1.1
ip route del 10.20.0.0/16
ip route get 8.8.8.8
```

Important: - `dev` = outgoing interface - `via` = next-hop gateway -
`scope link` = destination is directly reachable on this network -
`default` = used when no more specific route exists

------------------------------------------------------------------------

# 🤝 ARP / Neighbor Cache

``` bash
ip neigh
ip neigh show
sudo ip neigh flush all
```

Use to: - View learned MAC addresses - Troubleshoot ARP failures

------------------------------------------------------------------------

# 🔎 DNS Investigation

## dig

``` bash
dig google.com
dig google.com A
dig google.com AAAA
dig google.com MX
dig google.com TXT
dig google.com NS
dig google.com SOA

dig @8.8.8.8 google.com
dig +short google.com
```

Useful fields: - QUESTION - ANSWER - TTL - SERVER - Query time

------------------------------------------------------------------------

## nslookup

``` bash
nslookup google.com
nslookup google.com 8.8.8.8
```

Shows: - DNS server used - IPv4 (A) - IPv6 (AAAA)

------------------------------------------------------------------------

## host

``` bash
host google.com
host -t MX google.com
host -t TXT google.com
host -t NS google.com
```

Quick summary of common DNS records.

------------------------------------------------------------------------

## Resolver Configuration

``` bash
cat /etc/resolv.conf
resolvectl status
resolvectl query google.com
```

Know: - `127.0.0.53` = local systemd-resolved stub resolver -
`resolvectl status` = real upstream DNS servers and DNS configuration

------------------------------------------------------------------------

# 📡 Connectivity

## ping

``` bash
ping google.com
ping 8.8.8.8
ping -c 4 google.com
```

## tracepath

``` bash
tracepath google.com
```

## traceroute

``` bash
traceroute google.com
```

Use to discover packet path.

------------------------------------------------------------------------

# 🚪 Ports & Sockets

## ss

``` bash
ss -tuln
ss -tulpn
ss -tn
ss -un
ss -s
```

Flags:

-   `-t` TCP
-   `-u` UDP
-   `-l` Listening
-   `-p` Process
-   `-n` Numeric

------------------------------------------------------------------------

# 📈 Network Statistics

``` bash
ip -s link
ss -s
```

------------------------------------------------------------------------

# 🧠 Troubleshooting Workflow

``` text
1. ip link
2. ip addr
3. ip route
4. ip route get <destination>
5. ip neigh
6. ping <gateway>
7. ping 8.8.8.8
8. ping google.com
9. dig / nslookup / host
10. cat /etc/resolv.conf
11. resolvectl status
12. ss -tulpn
13. ip -s link
14. ss -s
```

------------------------------------------------------------------------

# ⚡ Interview Notes

-   `dig` is preferred over `nslookup` for troubleshooting.
-   `host` provides a concise DNS summary.
-   `scope link` means the destination is directly reachable.
-   `127.0.0.53` is not Google's DNS---it is Ubuntu's local DNS stub
    resolver.
-   `ip route get` reveals exactly which route Linux will use.

------------------------------------------------------------------------

# 🚀 Future DevOps Commands (Not Yet Covered)

``` text
curl
wget
tcpdump
nmap
nc
mtr
iperf3
iftop
ethtool
nmcli
journalctl -u systemd-resolved
```

These belong to later chapters and will be added after they are formally
studied.

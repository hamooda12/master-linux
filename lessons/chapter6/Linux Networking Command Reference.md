# 🐧 Linux Networking Command Reference (Complete)

> Companion guide for Chapter 06 --- Linux Networking Fundamentals

This document summarizes **every command introduced in Chapter 06**.

------------------------------------------------------------------------

# 🖥 Host Information

## hostname

Show or temporarily change the hostname.

``` bash
hostname
hostname new-host
```

## hostnamectl

Display or permanently manage hostname.

``` bash
hostnamectl
hostnamectl set-hostname web01
```

## uname -a

Display kernel and system information.

``` bash
uname -a
```

------------------------------------------------------------------------

# 🌐 Network Interfaces

## ip addr

``` bash
ip addr
ip addr show
ip addr show eth0
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0
```

## ip link

``` bash
ip link
ip link show
sudo ip link set eth0 up
sudo ip link set eth0 down
```

## hostname -I

``` bash
hostname -I
```

Shows all assigned IPv4 addresses.

## nmcli device status

``` bash
nmcli device status
```

Shows NetworkManager device state.

## ethtool

``` bash
sudo ethtool eth0
```

Shows NIC speed, duplex and link status.

------------------------------------------------------------------------

# 🛣 Routing

## ip route

``` bash
ip route
ip route add 10.20.0.0/16 via 192.168.1.254
ip route add default via 192.168.1.1
ip route del 10.20.0.0/16
```

## ip route get

``` bash
ip route get 8.8.8.8
```

Shows the exact route Linux will use.

## route -n

``` bash
route -n
```

Legacy routing table viewer.

------------------------------------------------------------------------

# 📡 Connectivity

## ping

``` bash
ping google.com
ping -c 4 8.8.8.8
```

## tracepath

``` bash
tracepath google.com
```

## traceroute

``` bash
traceroute google.com
```

## arp

``` bash
arp
arp -a
```

Legacy ARP table.

## ip neigh

``` bash
ip neigh
sudo ip neigh flush all
```

Modern ARP/Neighbor table.

------------------------------------------------------------------------

# 🔎 DNS

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

## host

``` bash
host google.com
host -t MX google.com
host -t TXT google.com
```

## nslookup

``` bash
nslookup google.com
nslookup google.com 8.8.8.8
```

## Resolver

``` bash
cat /etc/resolv.conf
resolvectl status
resolvectl query google.com
resolvectl flush-caches
```

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

## lsof

``` bash
sudo lsof -i
sudo lsof -i :80
```

------------------------------------------------------------------------

# 🧪 Testing Network Services

## curl

``` bash
curl http://localhost:8080
curl -I https://example.com
curl -v https://example.com
curl -L https://example.com
curl -X POST http://localhost/api -H "Content-Type: application/json" -d '{"name":"Alice"}'
```

## wget

``` bash
wget https://example.com/file.iso
wget -O custom.iso https://example.com/file.iso
wget -c https://example.com/file.iso
```

## nc (Netcat)

``` bash
nc -zv localhost 8080
nc -zvu dns.google 53
nc -l 9000
```

## telnet

``` bash
telnet google.com 80
```

## openssl s_client

``` bash
openssl s_client -connect google.com:443
```

Inspect TLS/SSL certificates.

------------------------------------------------------------------------

# 🔐 SSH

## ssh

``` bash
ssh user@server
ssh -p 2222 user@server
```

## scp

``` bash
scp file.txt user@server:/tmp
scp user@server:/tmp/file.txt .
```

## sftp

``` bash
sftp user@server
```

## ssh-keygen

``` bash
ssh-keygen
ssh-keygen -t ed25519
```

## SSH Service

``` bash
systemctl status ssh
systemctl restart ssh
journalctl -u ssh
```

------------------------------------------------------------------------

# 🔥 Linux Firewall

## UFW

``` bash
ufw status
ufw status verbose
sudo ufw enable
sudo ufw disable
sudo ufw reload
sudo ufw allow 22/tcp
sudo ufw deny 80/tcp
sudo ufw delete allow 22/tcp
```

## iptables

``` bash
iptables -L
iptables -L -n -v
iptables -S
iptables -F
```

## nftables

``` bash
nft list ruleset
systemctl status nftables
```

------------------------------------------------------------------------

# ⚙ Network Configuration

``` bash
nmcli
nmcli device status
ip addr
ip route
resolvectl status
cat /etc/resolv.conf
```

------------------------------------------------------------------------

# 🚑 DevOps Troubleshooting Workflow

``` text
1. hostnamectl
2. ip link
3. ip addr
4. hostname -I
5. nmcli device status
6. ethtool
7. ip route
8. ip route get <destination>
9. ping
10. tracepath / traceroute
11. ip neigh
12. dig / host / nslookup
13. resolvectl status
14. ss -tulpn
15. lsof -i
16. curl
17. nc
18. openssl s_client
19. ssh
20. ufw / iptables / nft
```

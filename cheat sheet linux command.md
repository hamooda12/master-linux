# 🐧 Linux Cheat Sheet

> **Personal Linux reference** built throughout my Linux Practice
> journey.
>
> **Current Progress:** ✅ Through **Chapter 06 --- Linux Networking
> Fundamentals**

------------------------------------------------------------------------

# 🔥 Daily Commands (Most Used)

``` bash
pwd
ls -lah
cd -
find . -name "*.conf"
grep -r "ERROR" .
tail -f app.log
ip addr
ip route
ping 8.8.8.8
ss -tulpn
curl http://localhost
systemctl status nginx
journalctl -fu nginx
id
stat file
getfacl file
```

> 💡 **Remember**
>
> -   `find` searches the filesystem.
> -   `grep` searches inside files.
> -   `locate` searches the indexed database.
> -   `which` finds the executable in your PATH.
> -   `whereis` finds binaries, source, and man pages.

------------------------------------------------------------------------

# 📂 Navigation

  Command     Description
  ----------- -------------------------
  `pwd`       Print current directory
  `ls -lah`   Long listing
  `tree`      Directory tree
  `cd -`      Previous directory

------------------------------------------------------------------------

# 📁 File & Directory Operations

`mkdir -p` • `touch` • `cp -r` • `mv` • `rm -r` • `find` • `locate` •
`which` • `whereis`

------------------------------------------------------------------------

# 📄 File Viewing

`cat` • `less` • `head` • `tail` • `tail -f`

------------------------------------------------------------------------

# 🔍 Text Processing

`grep` • `sort` • `uniq` • `cut` • `wc` • `tr` • `awk` • `sed`

------------------------------------------------------------------------

# 👤 Users & Groups

## User Management

`useradd` • `adduser` • `usermod` • `userdel` • `passwd` • `passwd -S` •
`chage`

## Group Management

`groupadd` • `groupmod` • `groupdel` • `gpasswd`

## Information

`whoami` • `id` • `groups` • `getent passwd` • `getent group`

------------------------------------------------------------------------

# 🔐 Permissions

`chmod` • `chown` • `chgrp` • `umask` • `stat` • `namei -l`

Common modes:

-   `600` Private file
-   `644` Regular file
-   `700` Private directory
-   `755` Executable / directory
-   `775` Shared directory

------------------------------------------------------------------------

# 🔑 ACL

`getfacl` • `setfacl`

------------------------------------------------------------------------

# 🛡️ sudo

`sudo` • `sudo -l` • `sudo -k` • `visudo`

------------------------------------------------------------------------

# 📌 File Attributes

`lsattr` • `chattr`

------------------------------------------------------------------------

# 📦 Archives & Compression

`tar` • `gzip` • `bzip2` • `xz` • `zip`

------------------------------------------------------------------------

# ⚙️ Process Management

## Monitoring

`ps` • `top` • `htop` • `pstree`

## Control

`kill` • `pkill` • `killall`

## Priority

`nice` • `renice`

------------------------------------------------------------------------

# 🛠️ Service Management

`systemctl status`

`systemctl start`

`systemctl stop`

`systemctl restart`

`systemctl reload`

`systemctl enable`

`systemctl disable`

------------------------------------------------------------------------

# 📜 System Logs

`journalctl`

`journalctl -u`

`journalctl -fu`

`journalctl -b`

`journalctl --since today`

`journalctl -p err`

------------------------------------------------------------------------

# 💼 Job Control

`command &` • `jobs` • `fg` • `bg`

------------------------------------------------------------------------

# 🖥️ Persistent Sessions

`nohup` • `screen` • `tmux`

------------------------------------------------------------------------

# 🌐 Networking

## Interface & Address

  Command          Purpose
  ---------------- --------------------
  `ip link`        Show interfaces
  
  `ip addr`        Show IP addresses
  
  `ip addr show`   Detailed addresses

## Routing

  Command           Purpose
  ----------------- --------------------
  `ip route`        Show routing table
  
  `ip route show`   Display routes

## DNS

  Command                  Purpose
  ------------------------ -----------------
  `cat /etc/resolv.conf`   DNS servers
  
  `host domain`            Simple lookup
  
  `dig domain`             Detailed lookup
  
  `nslookup domain`        DNS query
  
  `hostname`               System hostname

## Connectivity

  Command             Purpose
  ------------------- ---------------------------
  `ping host`         Connectivity test
  
  `traceroute host`   Show packet path
  
  `tracepath host`    Trace route
  
  `mtr host`          Continuous route analysis

## Ports & Sockets

  Command            Purpose
  ------------------ -------------------------
  `ss -tulpn`        Listening sockets
  
  `ss -tan`          TCP connections
  
  `ss -tp`           TCP with processes
  
  `lsof -i`          Processes using sockets
  
  `netstat -tulpn`   Legacy socket view

## Service Testing

  Command              Purpose
  -------------------- ------------------
  `curl URL`           Test HTTP/API
  
  `curl -I URL`        Headers only
  
  `curl -L URL`        Follow redirects
  
  `wget URL`           Download file
  
  `wget -O file URL`   Save with name
  
  `nc -zv host port`   Test TCP port
  
  `telnet host port`   Legacy TCP test

## SSH

  Command                   Purpose
  ------------------------- ----------------------
  `ssh user@host`           Remote login
  
  `ssh -p PORT user@host`   Custom port
  
  `ssh-keygen -t ed25519`   Generate key
  
  `ssh-copy-id user@host`   Install public key
  
  `scp`                     Secure copy
  
  `sftp`                    Secure file transfer

## Firewall (UFW)

  Command                            Purpose
  ---------------------------------- ------------------
  `sudo ufw status`                  Firewall status
  
  `sudo ufw status verbose`          Detailed status
  
  `sudo ufw enable`                  Enable firewall
  
  `sudo ufw disable`                 Disable firewall
  
  `sudo ufw allow OpenSSH`           Allow SSH
  
  `sudo ufw allow 80/tcp`            Allow HTTP
  
  `sudo ufw allow 443/tcp`           Allow HTTPS
  
  `sudo ufw deny PORT/tcp`           Block port
  
  `sudo ufw delete allow PORT/tcp`   Delete rule
  
  `sudo ufw status numbered`         Numbered rules
  
  `sudo ufw app list`                App profiles

## Network Configuration

  Command                     Purpose
  --------------------------- ---------------------
  `ls /etc/netplan`           List Netplan files
  
  `cat /etc/netplan/*.yaml`   View config
  
  `sudo netplan try`          Test configuration
  
  `sudo netplan apply`        Apply configuration

------------------------------------------------------------------------

# 🚦 Common Linux Signals

`SIGTERM` • `SIGKILL` • `SIGINT` • `SIGHUP` • `SIGSTOP` • `SIGCONT`

------------------------------------------------------------------------

# ⚠️ Common Mistakes

-   Using `kill -9` before `SIGTERM`
-   Forgetting `sort` before `uniq`
-   Editing `/etc/sudoers` without `visudo`
-   Forgetting `-a` with `usermod -aG`
-   Assuming `ping` proves an application is healthy
-   Forgetting to verify `ss -tulpn`
-   Ignoring firewall rules (`ufw`)
-   Changing Netplan without `netplan try`
-   Restarting services before collecting evidence

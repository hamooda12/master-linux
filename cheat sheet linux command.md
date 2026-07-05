# 🐧 Linux Cheat Sheet

> Personal Linux reference built throughout my Linux Practice journey.

**Current Progress:** Through **Chapter 04 -- Process Management &
System Services**

------------------------------------------------------------------------

# 📂 Navigation

  Command   Description
  --------- -------------------------
  `pwd`     Print current directory
  `ls`      List files
  `ls -a`   Show hidden files
  `ls -l`   Long listing
  `tree`    Display directory tree
  `cd`      Change directory

------------------------------------------------------------------------

# 📁 File & Directory Operations

  Command      Description
  ------------ ---------------------------
  `mkdir`      Create directory
  `mkdir -p`   Create nested directories
  `touch`      Create file
  `cp`         Copy files
  `mv`         Move/Rename
  `rm`         Remove files
  `rm -r`      Remove directories
  `find`       Search files
  `locate`     Fast file search
  `which`      Locate executable
  `whereis`    Locate binary/source/man

------------------------------------------------------------------------

# 📄 File Viewing

  Command     Description
  ----------- ------------------
  `cat`       Display file
  `less`      Scroll file
  `more`      Paginated output
  `head`      First lines
  `tail`      Last lines
  `tail -f`   Follow log file

------------------------------------------------------------------------

# 🔍 Text Processing

  Command   Description
  --------- -----------------------------
  `grep`    Search text
  `sort`    Sort lines
  `uniq`    Remove duplicates
  `cut`     Extract fields
  `wc`      Count lines/words
  `tr`      Translate/Delete characters
  `awk`     Pattern scanning
  `sed`     Stream editor

------------------------------------------------------------------------

# 🔗 Links

  Command   Description
  --------- ---------------
  `ln`      Hard link
  `ln -s`   Symbolic link

------------------------------------------------------------------------

# 🔐 Permissions

  Command   Description
  --------- --------------------
  `chmod`   Change permissions
  `chown`   Change owner
  `chgrp`   Change group

------------------------------------------------------------------------

# 📦 Archives & Compression

-   `tar -cf archive.tar dir/`
-   `tar -tf archive.tar`
-   `tar -xf archive.tar`
-   `tar -czf archive.tar.gz dir/`
-   `tar -cjf archive.tar.bz2 dir/`
-   `tar -cJf archive.tar.xz dir/`
-   `gzip`, `gunzip`
-   `bzip2`, `bunzip2`
-   `xz`, `unxz`
-   `zip`, `unzip`

------------------------------------------------------------------------

# 🔐 Checksums

-   `sha256sum file`
-   `sha256sum file > file.sha256`
-   `sha256sum -c file.sha256`

------------------------------------------------------------------------

# 💾 Backup Concepts

-   Archive
-   Compression
-   Full / Incremental / Differential Backup
-   3-2-1 Rule
-   RTO / RPO

------------------------------------------------------------------------

# 🚀 Bash Essentials

-   Variables
-   Command substitution
-   Functions
-   if statements
-   case statements
-   Exit codes (`$?`)

------------------------------------------------------------------------

# ⚙️ Process Management

## Monitoring

-   `ps`
-   `ps aux`
-   `ps -ef`
-   `top`
-   `htop`
-   `pstree`

## Control

-   `kill PID`
-   `kill -9 PID`
-   `kill -STOP PID`
-   `kill -CONT PID`
-   `kill -l`
-   `pkill process`
-   `killall process`

## Priority

-   `nice -n VALUE command`
-   `renice VALUE PID`

------------------------------------------------------------------------

# 🛠️ Service Management

-   `systemctl status SERVICE`
-   `systemctl start SERVICE`
-   `systemctl stop SERVICE`
-   `systemctl restart SERVICE`
-   `systemctl reload SERVICE`
-   `systemctl enable SERVICE`
-   `systemctl disable SERVICE`
-   `systemctl is-active SERVICE`
-   `systemctl is-enabled SERVICE`
-   `systemctl list-units --type=service`
-   `systemctl list-unit-files --type=service`

------------------------------------------------------------------------

# 📜 System Logs

-   `journalctl`
-   `journalctl -n 20`
-   `journalctl -u SERVICE`
-   `journalctl -fu SERVICE`
-   `journalctl -f`
-   `journalctl -b`
-   `journalctl -b -1`
-   `journalctl --since today`
-   `journalctl --since "1 hour ago"`
-   `journalctl -p err`

------------------------------------------------------------------------

# 💼 Job Control

-   `command &`
-   `jobs`
-   `fg`
-   `bg`
-   `Ctrl + Z`
-   `kill %1`

------------------------------------------------------------------------

# 🖥️ Long-Running Sessions

## nohup

-   `nohup command &`

## screen

-   `screen`
-   `screen -S session`
-   `screen -ls`
-   `screen -r session`

Detach: **Ctrl+A D**

## tmux

-   `tmux`
-   `tmux new -s session`
-   `tmux ls`
-   `tmux attach -t session`

Detach: **Ctrl+B D**

------------------------------------------------------------------------

# 🚦 Linux Signals

  Signal      Number Purpose
  --------- -------- --------------------------------
  SIGHUP           1 Hang up / reload configuration
  SIGINT           2 Interrupt (`Ctrl+C`)
  SIGKILL          9 Force terminate
  SIGTERM         15 Graceful terminate
  SIGCONT         18 Resume
  SIGSTOP         19 Pause

------------------------------------------------------------------------

# ⌨️ Keyboard Shortcuts

  Shortcut   Action
  ---------- -----------------
  `Ctrl+C`   Send SIGINT
  `Ctrl+Z`   Suspend process
  `Ctrl+D`   Logout / EOF
  `Ctrl+L`   Clear terminal

------------------------------------------------------------------------

# ⭐ Useful Combinations

``` bash
ps aux | grep nginx
systemctl status nginx
journalctl -u nginx
journalctl -fu nginx
nohup ./backup.sh > backup.log 2>&1 &
sleep 300 &
jobs
tmux new -s maintenance
```

------------------------------------------------------------------------

# 📌 Next Chapter

**Chapter 05 -- Users, Groups & Permissions**

Upcoming commands:

-   id
-   whoami
-   who
-   groups
-   useradd
-   usermod
-   userdel
-   passwd
-   groupadd
-   groupmod
-   groupdel
-   sudo
-   su
-   visudo

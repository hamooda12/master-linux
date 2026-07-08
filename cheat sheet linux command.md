# 🐧 Linux Cheat Sheet

> **Personal Linux reference** built throughout my Linux Practice
> journey.
>
> **Current Progress:** ✅ Through **Chapter 05 -- Users, Groups &
> Permissions**

------------------------------------------------------------------------

# 🔥 Daily Commands (Most Used)

``` bash
pwd
ls -lah
cd -
find . -name "*.conf"
grep -r "ERROR" .
tail -f app.log
ps aux
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

  Command     Description                      Example
  ----------- -------------------------------- ----------------
  `pwd`       Print current directory          `pwd`
  
  `ls`        List files                       `ls`
  
  `ls -lah`   Long + hidden + human-readable   `ls -lah`
  
  `tree`      Display directory tree           `tree`
  
  `cd dir`    Change directory                 `cd Documents`
  
  `cd ..`     Parent directory                 `cd ..`
  
  `cd -`      Previous directory               `cd -`

------------------------------------------------------------------------

# 📁 File & Directory Operations

  Command                  Description
  ------------------------ ----------------------------------
  `mkdir -p`               Create nested directories
  
  `touch`                  Create empty file
  
  `cp -r`                  Copy directory recursively
  
  `mv`                     Move / rename
  
  `rm -r`                  Remove directory recursively
  
  `find . -name "*.log"`   Search files
  
  `locate filename`        Fast indexed search
  
  `which command`          Locate executable
  
  `whereis command`        Locate binary, source & man page

------------------------------------------------------------------------

# 📄 File Viewing

  Command      Purpose
  ------------ -------------------------
  `cat`        Display entire file
  
  `less`       Scroll large files
  
  `head -20`   First 20 lines
  
  `tail -20`   Last 20 lines
  
  `tail -f`    Follow log in real time

------------------------------------------------------------------------

# 🔍 Text Processing

  Command   Purpose
  --------- -----------------------------
  `grep`    Search text
  
  `sort`    Sort lines
  
  `uniq`    Remove adjacent duplicates
  
  `cut`     Extract fields
  
  `wc`      Count lines, words & bytes
  
  `tr`      Translate/Delete characters
  
  `awk`     Field processing
  
  `sed`     Stream editor

------------------------------------------------------------------------

# 👤 Users & Groups

## User Information

  Command           Purpose
  ----------------- ---------------------
  `whoami`          Current user
  
  `id`              UID, GID and groups
  
  `groups`          Group membership
  
  `getent passwd`   User database
  
  `getent group`    Group database

## User Management

  Command       Purpose
  ------------- ----------------------
  `useradd`     Create user
  
  `adduser`     Interactive creation
  
  `usermod`     Modify user
  
  `userdel`     Delete user
  
  `passwd`      Change password
  
  `passwd -S`   Password status
  
  `chage`       Password aging

## Group Management

  Command      Purpose
  ------------ ----------------
  `groupadd`   Create group
  
  `groupmod`   Modify group
  
  `groupdel`   Delete group
  
  `gpasswd`    Manage members

------------------------------------------------------------------------

# 🔐 Permissions

  Command      Purpose
  ------------ --------------------------
  `chmod`      Change permissions
  
  `chown`      Change owner
  
  `chgrp`      Change group
  
  `umask`      Default permissions
  
  `stat`       File metadata
  
  `namei -l`   Inspect path permissions

Common Modes

-   `600` Private file
-   
-   `644` Regular file
-   
-   `700` Private directory
-   
-   `755` Executable / directory
-   
-   `775` Shared project directory

------------------------------------------------------------------------

# 🔑 ACL

  Command     Purpose
  ----------- ------------
  `getfacl`   View ACL
  
  `setfacl`   Manage ACL

------------------------------------------------------------------------

# 🛡️ sudo

  Command     Purpose
  ----------- --------------------------
  `sudo`      Run as another user
  
  `sudo -l`   List allowed commands
  
  `sudo -k`   Clear cached credentials
  
  `visudo`    Edit sudoers safely

------------------------------------------------------------------------

# 📌 File Attributes

  Command    Purpose
  ---------- -------------------
  `lsattr`   View attributes
  
  `chattr`   Change attributes

Examples

``` bash
chattr +i file

chattr -i file

chattr +a logfile

```

------------------------------------------------------------------------

# 📂 Important Account Files

-   `/etc/passwd`
-   `/etc/shadow`
-   `/etc/group`
-   `/etc/gshadow`
-   `/etc/sudoers`

------------------------------------------------------------------------

# 🧮 umask

``` bash
umask
umask -S
```

    umask  Files   Directories
  ------- ------- -------------
      022   644        755
      027   640        750
      077   600        700

------------------------------------------------------------------------

# 🔍 Production Investigation

``` bash
id username
groups username
getent passwd username
passwd -S username
chage -l username
ls -l
stat file
namei -l /path/file
getfacl file
sudo -l
```

------------------------------------------------------------------------

# 📦 Archives & Compression

`tar` • `gzip` • `bzip2` • `xz` • `zip`

------------------------------------------------------------------------

# ⚙️ Process Management

Monitor: `ps` • `top` • `htop` • `pstree`

Control: `kill` • `pkill` • `killall`

Priority: `nice` • `renice`

------------------------------------------------------------------------

# 🛠️ Service Management

`systemctl status` • `start` • `stop` • `restart` • `reload` • `enable`
• `disable`

------------------------------------------------------------------------

# 📜 System Logs

``` bash
journalctl
journalctl -u SERVICE
journalctl -fu SERVICE
journalctl -b
journalctl --since today
journalctl -p err
```

------------------------------------------------------------------------

# 💼 Job Control

`command &` • `jobs` • `fg` • `bg`

------------------------------------------------------------------------

# 🖥️ Persistent Sessions

`nohup` • `screen` • `tmux`

------------------------------------------------------------------------

# 🚦 Common Linux Signals

SIGTERM • SIGKILL • SIGINT • SIGHUP • SIGSTOP • SIGCONT

------------------------------------------------------------------------

# ⚠️ Common Mistakes

-   Using `kill -9` before `SIGTERM`
-   Forgetting `sort` before `uniq`
-   Editing `/etc/sudoers` without `visudo`
-   Changing ownership instead of using groups
-   Forgetting `-a` with `usermod -aG`
-   Ignoring ACLs when standard permissions aren't enough

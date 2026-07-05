
# 🐧 Linux Cheat Sheet

> **Personal Linux reference** built throughout my Linux Practice journey.
>
> **Current Progress:** ✅ Through **Chapter 04 – Process Management & System Services**

---

# 🔥 Daily Commands (Most Used)

```bash
pwd
ls -lah
cd -
find . -name "*.conf"
grep -r "ERROR" .
tail -f app.log
ps aux
systemctl status nginx
journalctl -fu nginx
```

> 💡 **Remember**
>
> - `find` searches the filesystem.
> - `grep` searches inside files.
> - `locate` searches the indexed database.
> - `which` finds the executable in your PATH.
> - `whereis` finds binaries, source, and man pages.

---

# 📂 Navigation

| Command | Description | Example |
|---------|-------------|---------|
| `pwd` | Print current directory | `pwd` |
| `ls` | List files | `ls` |
| `ls -lah` | Long + hidden + human-readable | `ls -lah` |
| `tree` | Display directory tree | `tree` |
| `cd dir` | Change directory | `cd Documents` |
| `cd ..` | Parent directory | `cd ..` |
| `cd -` | Previous directory | `cd -` |

---

# 📁 File & Directory Operations

| Command | Description |
|---------|-------------|
| `mkdir -p` | Create nested directories |
| `touch` | Create empty file |
| `cp -r` | Copy directory recursively |
| `mv` | Move / rename |
| `rm -r` | Remove directory recursively |
| `find . -name "*.log"` | Search files |
| `locate filename` | Fast indexed search |
| `which command` | Locate executable |
| `whereis command` | Locate binary, source & man page |

---

# 📄 File Viewing

| Command | Purpose |
|---------|---------|
| `cat` | Display entire file |
| `less` | Scroll large files |
| `head -20` | First 20 lines |
| `tail -20` | Last 20 lines |
| `tail -f` | Follow log in real time |

---

# 🔍 Text Processing

| Command | Purpose |
|---------|---------|
| `grep` | Search text |
| `sort` | Sort lines |
| `uniq` | Remove adjacent duplicates *(usually after `sort`)* |
| `cut` | Extract fields |
| `wc` | Count lines, words & bytes |
| `tr` | Translate/Delete characters |
| `awk` | Field processing |
| `sed` | Stream editor |

---

# 🔐 Permissions

| Command | Purpose |
|---------|---------|
| `chmod` | Change permissions |
| `chown` | Change owner |
| `chgrp` | Change group |

Common:
- `chmod 644 file`
- `chmod 755 script.sh`
- `chmod +x script.sh`

---

# 📦 Archives & Compression

| Task | Command |
|------|---------|
| Create tar | `tar -cf archive.tar dir/` |
| List archive | `tar -tf archive.tar` |
| Extract archive | `tar -xf archive.tar` |
| Create gzip archive | `tar -czf archive.tar.gz dir/` |
| Create bzip2 archive | `tar -cjf archive.tar.bz2 dir/` |
| Create xz archive | `tar -cJf archive.tar.xz dir/` |

Tools: `gzip`, `gunzip`, `bzip2`, `bunzip2`, `xz`, `unxz`, `zip`, `unzip`

---

# 🔐 Checksums

```bash
sha256sum file
sha256sum file > file.sha256
sha256sum -c file.sha256
```

---

# 💾 Backup Concepts

- Full Backup
- Incremental Backup
- Differential Backup
- 3-2-1 Rule
- RTO
- RPO

---

# 🚀 Bash Essentials

- Variables
- Command substitution `$(...)`
- Functions
- `if`
- `case`
- Exit status `$?`

---

# ⚙️ Process Management

### Monitor
`ps` • `ps aux` • `ps -ef` • `top` • `htop` • `pstree`

### Control
`kill` • `pkill` • `killall` • `kill -STOP` • `kill -CONT`

### Priority
`nice` • `renice`

---

# 🛠️ Service Management

| Action | Command |
|--------|---------|
| Status | `systemctl status SERVICE` |
| Start | `systemctl start SERVICE` |
| Stop | `systemctl stop SERVICE` |
| Restart | `systemctl restart SERVICE` |
| Reload | `systemctl reload SERVICE` |
| Enable | `systemctl enable SERVICE` |
| Disable | `systemctl disable SERVICE` |
| Active? | `systemctl is-active SERVICE` |
| Enabled? | `systemctl is-enabled SERVICE` |

---

# 📜 System Logs

```bash
journalctl
journalctl -u SERVICE
journalctl -fu SERVICE
journalctl -b
journalctl -b -1
journalctl --since today
journalctl --since "1 hour ago"
journalctl -p err
```

---

# 💼 Job Control

`command &` • `jobs` • `fg` • `bg` • `Ctrl+Z` • `kill %1`

---

# 🖥️ Persistent Sessions

### nohup
```bash
nohup command > output.log 2>&1 &
```

### screen
`screen` • `screen -S session` • `screen -ls` • `screen -r session`

Detach: **Ctrl+A D**

### tmux
`tmux new -s session` • `tmux ls` • `tmux attach -t session`

Detach: **Ctrl+B D**

---

# 🚦 Common Linux Signals

| Signal | No. | Purpose |
|--------|----:|---------|
| SIGHUP | 1 | Reload configuration |
| SIGINT | 2 | Ctrl+C |
| SIGKILL | 9 | Force kill *(cannot be caught)* |
| SIGTERM | 15 | Graceful termination *(preferred)* |
| SIGCONT | 18 | Resume |
| SIGSTOP | 19 | Pause *(cannot be ignored)* |

---

# ⚡ Production Command Recipes

```bash
ps aux | grep nginx
systemctl status nginx
journalctl -fu nginx
grep ERROR app.log | wc -l
grep ERROR app.log | sort | uniq -c | sort -nr
find . -type f | wc -l
du -sh * | sort -h
nohup ./backup.sh > backup.log 2>&1 &
tmux new -s maintenance
```

---

# ⚠️ Common Mistakes

- Running `rm -rf` without checking the path.
- Using `kill -9` before trying `SIGTERM`.
- Forgetting `sort` before `uniq`.
- Using `cat` on huge log files instead of `less`.
- Using `>` when you meant `>>`.

---

# 📌 Next Chapter

## Chapter 05 – Users, Groups & Permissions

`id` • `whoami` • `who` • `groups` • `useradd` • `usermod` • `userdel` • `passwd` • `groupadd` • `groupmod` • `groupdel` • `sudo` • `su` • `visudo`

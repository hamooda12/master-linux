# 🐧 Linux Cheat Sheet

> Personal Linux reference built throughout my Linux Practice journey.

**Current Progress:** Through **Chapter 03 – Archives, Compression & Backups**

---

# 📂 Navigation

| Command | Description             |
| ------- | ----------------------- |
| `pwd`   | Print current directory |
| `ls`    | List files              |
| `ls -a` | Show hidden files       |
| `ls -l` | Long listing            |
| `tree`  | Display directory tree  |
| `cd`    | Change directory        |

---

# 📁 File & Directory Operations

| Command    | Description               |
| ---------- | ------------------------- |
| `mkdir`    | Create directory          |
| `mkdir -p` | Create nested directories |
| `touch`    | Create file               |
| `cp`       | Copy files                |
| `mv`       | Move/Rename               |
| `rm`       | Remove files              |
| `rm -r`    | Remove directories        |
| `find`     | Search files              |
| `locate`   | Fast file search          |
| `which`    | Locate executable         |
| `whereis`  | Locate binary/source/man  |

---

# 📄 File Viewing

| Command   | Description      |
| --------- | ---------------- |
| `cat`     | Display file     |
| `less`    | Scroll file      |
| `more`    | Paginated output |
| `head`    | First lines      |
| `tail`    | Last lines       |
| `tail -f` | Follow log file  |

---

# 🔍 Text Processing

| Command | Description          |
| ------- | -------------------- |
| `grep`  | Search text          |
| `sort`  | Sort lines           |
| `uniq`  | Remove duplicates    |
| `cut`   | Extract fields       |
| `wc`    | Count lines/words    |
| `tr`    | Translate characters |
| `awk`   | Pattern scanning     |
| `sed`   | Stream editor        |

---

# 🔗 Links

| Command | Description   |
| ------- | ------------- |
| `ln`    | Hard link     |
| `ln -s` | Symbolic link |

---

# 🔐 Permissions

| Command | Description        |
| ------- | ------------------ |
| `chmod` | Change permissions |
| `chown` | Change owner       |
| `chgrp` | Change group       |

---

# 📦 Archives

| Command                         | Description          |
| ------------------------------- | -------------------- |
| `tar -cf archive.tar dir/`      | Create archive       |
| `tar -tf archive.tar`           | List archive         |
| `tar -xf archive.tar`           | Extract archive      |
| `tar -czf archive.tar.gz dir/`  | Create gzip archive  |
| `tar -cjf archive.tar.bz2 dir/` | Create bzip2 archive |
| `tar -cJf archive.tar.xz dir/`  | Create xz archive    |

---

# 🗜️ Compression

## gzip

| Command           | Description  |
| ----------------- | ------------ |
| `gzip file`       | Compress     |
| `gunzip file.gz`  | Decompress   |
| `gzip -t file.gz` | Test archive |

### bzip2

| Command             | Description  |
| ------------------- | ------------ |
| `bzip2 file`        | Compress     |
| `bunzip2 file.bz2`  | Decompress   |
| `bzip2 -t file.bz2` | Test archive |

### xz

| Command         | Description  |
| --------------- | ------------ |
| `xz file`       | Compress     |
| `unxz file.xz`  | Decompress   |
| `xz -t file.xz` | Test archive |

### zip

| Command                   | Description   |
| ------------------------- | ------------- |
| `zip -r archive.zip dir/` | Create zip    |
| `unzip archive.zip`       | Extract       |
| `unzip -l archive.zip`    | List contents |
| `unzip -t archive.zip`    | Test archive  |

---

# 🔐 Checksums

| Command                        | Description       |
| ------------------------------ | ----------------- |
| `sha256sum file`               | Generate checksum |
| `sha256sum file > file.sha256` | Save checksum     |
| `sha256sum -c file.sha256`     | Verify checksum   |

---

# 💾 Backup Concepts

| Concept      | Summary                        |
| ------------ | ------------------------------ |
| Archive      | Combine multiple files         |
| Compression  | Reduce file size               |
| Full Backup  | Backup everything              |
| Incremental  | Changes since last backup      |
| Differential | Changes since last full backup |
| 3-2-1 Rule   | 3 copies, 2 media, 1 off-site  |
| RTO          | Recovery Time Objective        |
| RPO          | Recovery Point Objective       |

---

# 🚀 Bash Essentials Used

| Feature              | Example         |
| -------------------- | --------------- |
| Variable             | `NAME=value`    |
| Command substitution | `DATE=$(date)`  |
| Function             | `log() { ... }` |
| Condition            | `if ... fi`     |
| Case statement       | `case ... esac` |
| Exit code            | `$?`            |

---

# ⭐ Useful Combinations

```bash
tar -czf backup.tar.gz project/
```

```bash
tar -tzf backup.tar.gz
```

```bash
sha256sum backup.tar.gz > backup.tar.gz.sha256
```

```bash
sha256sum -c backup.tar.gz.sha256
```

```bash
find backups -mtime +7 -delete
```

---

# 📌 Next Chapter

**Chapter 04 – Process Management & Job Control**

Upcoming commands:

* `ps`
* `top`
* `htop`
* `pstree`
* `kill`
* `killall`
* `pkill`
* `jobs`
* `bg`
* `fg`
* `nohup`
* `nice`
* `renice`

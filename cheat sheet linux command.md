# Linux Cheat Sheet

> Personal Linux command reference built during the Linux Practice journey.

**Current Coverage:** Chapter 01 — Linux Filesystem Fundamentals

---

# Filesystem Navigation

| Command | Description | Example |
|---|---|---|
| `pwd` | Print current working directory | `pwd` |
| `cd <dir>` | Change into a directory | `cd Documents` |
| `cd ..` | Move to parent directory | `cd ..` |
| `cd ~` | Go to home directory | `cd ~` |
| `cd` | Go to home directory | `cd` |
| `cd /` | Go to root directory | `cd /` |

---

# Listing Files

| Command | Description | Example |
|---|---|---|
| `ls` | List visible files and directories | `ls` |
| `ls -l` | Long listing format | `ls -l` |
| `ls -a` | Show hidden files | `ls -a` |
| `ls -la` | Long listing including hidden files | `ls -la` |
| `ls -lh` | Human-readable sizes | `ls -lh` |
| `ls -R` | Recursive listing | `ls -R` |
| `ls -t` | Sort by modification time | `ls -t` |
| `ls -li` | Show inode numbers | `ls -li` |
| `ls -ld <dir>` | Show directory itself, not contents | `ls -ld /tmp` |

---

# Creating Files and Directories

| Command | Description | Example |
|---|---|---|
| `mkdir <dir>` | Create directory | `mkdir projects` |
| `mkdir -p <path>` | Create nested directories | `mkdir -p app/src/config` |
| `touch <file>` | Create empty file or update timestamp | `touch notes.txt` |
| `touch file1 file2` | Create multiple files | `touch a.txt b.txt` |

---

# Copying, Moving, and Renaming

| Command | Description | Example |
|---|---|---|
| `cp source destination` | Copy file | `cp notes.txt backup.txt` |
| `cp file dir/` | Copy file into directory | `cp notes.txt backups/` |
| `cp -r dir copy` | Copy directory recursively | `cp -r project project-copy` |
| `mv file dir/` | Move file | `mv notes.txt docs/` |
| `mv old new` | Rename file or directory | `mv old.txt new.txt` |

---

# Deleting Files and Directories

| Command | Description | Example |
|---|---|---|
| `rm file` | Remove file | `rm notes.txt` |
| `rm file1 file2` | Remove multiple files | `rm a.txt b.txt` |
| `rm -r dir` | Remove directory and contents | `rm -r old-project` |
| `rm -f file` | Force remove file | `rm -f notes.txt` |
| `rm -rf dir` | Force remove directory recursively | `rm -rf temp/` |
| `rmdir dir` | Remove empty directory | `rmdir empty-folder` |

> Be careful with `rm -rf`. Always verify your path first using `pwd` and `ls`.

---

# Viewing File Contents

| Command | Description | Example |
|---|---|---|
| `cat file` | Display entire file | `cat README.md` |
| `less file` | View file page by page | `less /etc/services` |
| `head file` | Show first 10 lines | `head notes.txt` |
| `head -n 5 file` | Show first 5 lines | `head -n 5 notes.txt` |
| `tail file` | Show last 10 lines | `tail app.log` |
| `tail -n 20 file` | Show last 20 lines | `tail -n 20 app.log` |
| `tail -f file` | Follow file updates live | `tail -f /var/log/syslog` |

---

# Searching Files and Directories

| Command | Description | Example |
|---|---|---|
| `find . -name "file"` | Find by exact name | `find . -name "README.md"` |
| `find . -iname "file"` | Case-insensitive search | `find . -iname "readme.md"` |
| `find . -type f` | Find regular files | `find . -type f` |
| `find . -type d` | Find directories | `find . -type d` |
| `find . -name "*.md"` | Find by extension | `find . -name "*.md"` |
| `find / -name "file"` | Search from root | `find / -name "sshd_config"` |

---

# Wildcards / Globbing

| Pattern | Description | Example |
|---|---|---|
| `*` | Match zero or more characters | `ls *.txt` |
| `?` | Match exactly one character | `ls file?.txt` |
| `[abc]` | Match one character from set | `ls file[123].txt` |
| `[1-5]` | Match one character in range | `ls file[1-5].txt` |

---

# Command History and Completion

| Command / Key | Description | Example |
|---|---|---|
| `history` | Show command history | `history` |
| `!!` | Run previous command | `!!` |
| `!number` | Run command by history number | `!42` |
| `Ctrl + R` | Search command history | Press `Ctrl + R` |
| `Tab` | Auto-complete command/path | `cd Doc<Tab>` |

---

# Help and Documentation

| Command | Description | Example |
|---|---|---|
| `man command` | Open manual page | `man ls` |
| `command --help` | Quick help output | `ls --help` |
| `apropos keyword` | Search manual database | `apropos copy` |
| `/text` inside `man` | Search inside manual page | `/recursive` |
| `q` inside `man` or `less` | Quit viewer | `q` |

---

# Permissions

| Symbol | Meaning |
|---|---|
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `-` | No permission |

Permission groups:

```text
owner   group   others
rwx     r-x     r--
```

---

# `chmod`

| Command | Description | Example |
|---|---|---|
| `chmod u+x file` | Add execute for owner | `chmod u+x script.sh` |
| `chmod g-w file` | Remove write from group | `chmod g-w notes.txt` |
| `chmod a+r file` | Add read for everyone | `chmod a+r notes.txt` |
| `chmod 644 file` | Owner read/write, others read | `chmod 644 README.md` |
| `chmod 755 file` | Executable by everyone, writable by owner | `chmod 755 script.sh` |
| `chmod 600 file` | Private file for owner only | `chmod 600 secret.txt` |

---

# Ownership

| Command | Description | Example |
|---|---|---|
| `chown user file` | Change owner | `sudo chown root file.txt` |
| `chown user:group file` | Change owner and group | `sudo chown root:root file.txt` |
| `chown -R user:group dir` | Recursive ownership change | `sudo chown -R root:root project/` |
| `chgrp group file` | Change group | `sudo chgrp developers file.txt` |

---

# Hard Links and Soft Links

| Command | Description | Example |
|---|---|---|
| `ln file link` | Create hard link | `ln README.md README-hard.md` |
| `ln -s target link` | Create symbolic link | `ln -s README.md README-soft.md` |
| `ls -li` | View inode numbers | `ls -li` |
| `ls -l` | View symbolic link target | `ls -l` |

Key differences:

| Feature | Hard Link | Symbolic Link |
|---|---|---|
| Shares inode | Yes | No |
| Can break if target deleted | No | Yes |
| Can link directories | Usually no | Yes |
| Can cross filesystems | No | Yes |

---

# Special Permissions

| Permission | Symbol | Numeric Prefix | Purpose |
|---|---|---:|---|
| SetUID | `s` on owner execute | `4` | Run executable as file owner |
| SetGID | `s` on group execute | `2` | Inherit group on directories |
| Sticky Bit | `t` on others execute | `1` | Prevent users deleting others' files in shared dirs |

Examples:

| Command | Description |
|---|---|
| `chmod u+s file` | Enable SetUID |
| `chmod g+s dir` | Enable SetGID |
| `chmod +t dir` | Enable Sticky Bit |
| `chmod 4755 file` | SetUID + 755 |
| `chmod 2755 dir` | SetGID + 755 |
| `chmod 1777 dir` | Sticky Bit + 777 |

Check `/tmp`:

```bash
ls -ld /tmp
```

Expected style:

```text
drwxrwxrwt
```

---

# Safety Rules

- Always run `pwd` before destructive commands.
- Always inspect with `ls` before deleting.
- Avoid `chmod 777` unless you fully understand the risk.
- Be careful with recursive commands such as `rm -rf`, `chmod -R`, and `chown -R`.
- Use `man` and `--help` when unsure.

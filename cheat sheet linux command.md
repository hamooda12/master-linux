# Linux Cheat Sheet

> Complete personal Linux reference for the Linux Practice Book.  
> Covers Chapter 01 and Chapter 02.

---

# Chapter 01 – Filesystem & File Management

---

## Navigation

| Command | Description | Example |
|---|---|---|
| `pwd` | Print current working directory | `pwd` |
| `cd <dir>` | Change directory | `cd Documents` |
| `cd ..` | Move to parent directory | `cd ..` |
| `cd ~` | Go to home directory | `cd ~` |
| `cd` | Go to home directory | `cd` |
| `cd /path` | Navigate using absolute path | `cd /etc` |
| `cd -` | Return to previous directory | `cd -` |

---

## Listing Files

| Command | Description |
|---|---|
| `ls` | List directory contents |
| `ls -a` | Show hidden files |
| `ls -l` | Long listing format |
| `ls -la` | Long listing including hidden files |
| `ls -lh` | Human-readable sizes |
| `ls -R` | Recursive listing |

---

## Creating Files and Directories

| Command | Description |
|---|---|
| `touch file.txt` | Create an empty file |
| `mkdir dir` | Create a directory |
| `mkdir -p a/b/c` | Create nested directories |

---

## Copying Files and Directories

| Command | Description |
|---|---|
| `cp file copy` | Copy a file |
| `cp file dir/` | Copy file into directory |
| `cp -r dir copy-dir` | Copy directory recursively |
| `cp -i file copy` | Ask before overwrite |
| `cp -v file copy` | Show copy operation |

---

## Moving and Renaming

| Command | Description |
|---|---|
| `mv old new` | Rename file or directory |
| `mv file dir/` | Move file into directory |
| `mv -i old new` | Ask before overwrite |
| `mv -v old new` | Show move operation |

---

## Removing Files and Directories

| Command | Description |
|---|---|
| `rm file` | Remove file |
| `rm -i file` | Ask before deleting |
| `rm -r dir` | Remove directory recursively |
| `rm -rf dir` | Force remove recursively |
| `rmdir dir` | Remove empty directory |

> Be careful with `rm -rf`. It permanently deletes files.

---

## Viewing File Information

| Command | Description |
|---|---|
| `file filename` | Detect file type |
| `stat filename` | Show detailed file metadata |
| `du -sh dir` | Show directory size |
| `df -h` | Show filesystem disk usage |

---

## Linux File Paths

| Path Type | Example | Meaning |
|---|---|---|
| Absolute path | `/home/hamad/file.txt` | Starts from `/` |
| Relative path | `docs/file.txt` | Starts from current directory |
| Home shortcut | `~/file.txt` | Starts from user home |

---

## Important Linux Directories

| Directory | Purpose |
|---|---|
| `/` | Root directory |
| `/home` | User home directories |
| `/etc` | System configuration files |
| `/var` | Variable data such as logs |
| `/tmp` | Temporary files |
| `/bin` | Essential user commands |
| `/sbin` | Essential system commands |
| `/usr` | User applications and libraries |
| `/opt` | Optional third-party software |
| `/dev` | Device files |
| `/proc` | Kernel and process information |
| `/boot` | Boot files |
| `/root` | Root user's home directory |

---

## Permissions

Example:

```text
-rw-r--r--
```

Breakdown:

```text
-   rw-   r--   r--
│    │     │     │
│    │     │     └── Others
│    │     └──────── Group
│    └────────────── Owner
└─────────────────── File type
```

| Symbol | Meaning |
|---|---|
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `-` | Permission not granted |

---

## Permission Numbers

| Number | Permission |
|---|---|
| `0` | No permission |
| `1` | Execute |
| `2` | Write |
| `3` | Write + execute |
| `4` | Read |
| `5` | Read + execute |
| `6` | Read + write |
| `7` | Read + write + execute |

Common examples:

| Command | Meaning |
|---|---|
| `chmod 644 file` | Owner read/write, others read |
| `chmod 755 script.sh` | Owner full, others read/execute |
| `chmod 600 file` | Owner only read/write |
| `chmod +x script.sh` | Add execute permission |
| `chmod -w file` | Remove write permission |

---

## Ownership

| Command | Description |
|---|---|
| `chown user file` | Change file owner |
| `chown user:group file` | Change owner and group |
| `chgrp group file` | Change group |
| `chown -R user:group dir` | Change ownership recursively |

---

## Links

### Hard Link

```bash
ln original.txt hardlink.txt
```

- Points to the same inode.
- Works only within the same filesystem.
- Usually not used for directories.

### Symbolic Link

```bash
ln -s original.txt symlink.txt
```

- Points to a path.
- Can cross filesystems.
- Can link directories.
- Breaks if target is removed.

---

## Special Permissions

### Sticky Bit

```bash
chmod +t shared-dir
```

Common example:

```bash
ls -ld /tmp
```

Purpose:

- Users can create files in the directory.
- Users can only delete their own files.

---

### setUID

```bash
chmod u+s file
```

Purpose:

- Run executable with owner's privileges.

---

### setGID

```bash
chmod g+s dir
```

Purpose:

- New files inside directory inherit the directory group.

---

# Chapter 02 – Working with Text Files and Streams

---

## File Viewing

| Command | Description |
|---|---|
| `cat file` | Display entire file |
| `cat -n file` | Display file with line numbers |
| `more file` | View file page by page |
| `less file` | Interactive file viewer |

Useful `less` keys:

| Key | Action |
|---|---|
| `Space` | Next page |
| `b` | Previous page |
| `g` | Beginning of file |
| `G` | End of file |
| `/text` | Search |
| `n` | Next match |
| `q` | Quit |

---

## Reading Large Files

| Command | Description |
|---|---|
| `head file` | Show first 10 lines |
| `head -n 20 file` | Show first 20 lines |
| `head -20 file` | Same as `head -n 20` |
| `tail file` | Show last 10 lines |
| `tail -n 20 file` | Show last 20 lines |
| `tail -20 file` | Same as `tail -n 20` |
| `tail -f file` | Follow live updates |

---

## Searching with grep

| Command | Description |
|---|---|
| `grep text file` | Search for text |
| `grep -i text file` | Ignore case |
| `grep -n text file` | Show line numbers |
| `grep -v text file` | Invert match |
| `grep -c text file` | Count matching lines |
| `grep -w text file` | Match whole words |
| `grep -x "line" file` | Match entire line |
| `grep -o pattern file` | Print only matched text |
| `grep -l pattern *.log` | Show filenames with matches |
| `grep -L pattern *.log` | Show filenames without matches |
| `grep -A 2 pattern file` | Show 2 lines after match |
| `grep -B 2 pattern file` | Show 2 lines before match |
| `grep -C 2 pattern file` | Show 2 lines before and after |
| `grep -r pattern .` | Recursive search |

---

## Basic Regular Expressions

| Pattern | Meaning |
|---|---|
| `.` | Any single character |
| `*` | Repeat previous character zero or more times |
| `^` | Beginning of line |
| `$` | End of line |
| `[abc]` | Match one character from list |
| `[0-9]` | Match a digit |

Examples:

```bash
grep "^root" /etc/passwd
grep "failed$" log.txt
grep "[0-9]" file.txt
grep "c.t" words.txt
```

---

## Counting with wc

| Command | Description |
|---|---|
| `wc file` | Lines, words, bytes |
| `wc -l file` | Count lines |
| `wc -w file` | Count words |
| `wc -m file` | Count characters |
| `wc -c file` | Count bytes |

Examples:

```bash
grep ERROR app.log | wc -l
ps -ef | wc -l
ls | wc -l
```

---

## Sorting and Removing Duplicates

| Command | Description |
|---|---|
| `sort file` | Sort alphabetically |
| `sort -r file` | Reverse sort |
| `sort -n file` | Numeric sort |
| `sort -k2 file` | Sort by second column |
| `sort -u file` | Sort and remove duplicates |
| `sort -h file` | Sort human-readable sizes |
| `uniq file` | Remove adjacent duplicates |
| `uniq -c file` | Count duplicate lines |
| `uniq -d file` | Show only duplicate lines |
| `uniq -u file` | Show only unique lines |

Common pipelines:

```bash
sort file | uniq
sort file | uniq -c
sort file | uniq -c | sort -nr
du -sh * | sort -h
```

---

## Comparing Files

| Command | Description |
|---|---|
| `diff old new` | Compare files line by line |
| `diff -y old new` | Side-by-side comparison |
| `diff -u old new` | Unified diff format |
| `cmp file1 file2` | Byte-by-byte comparison |

---

## Standard Streams

| Stream | Number | Purpose |
|---|---|---|
| stdin | `0` | Input |
| stdout | `1` | Normal output |
| stderr | `2` | Error output |

---

## Redirection

| Operator | Description |
|---|---|
| `>` | Redirect stdout and overwrite |
| `>>` | Redirect stdout and append |
| `<` | Redirect input |
| `2>` | Redirect stderr and overwrite |
| `2>>` | Redirect stderr and append |
| `2>&1` | Redirect stderr to stdout |
| `&>` | Redirect stdout and stderr |
| `> /dev/null` | Discard normal output |
| `2> /dev/null` | Discard errors |
| `&> /dev/null` | Discard all output |

Examples:

```bash
echo "Linux" > notes.txt
echo "Practice" >> notes.txt
wc -l < notes.txt
ls missing.txt 2> errors.log
find /etc -name "*.conf" > results.txt 2> errors.txt
find /etc -name "*.conf" > output.log 2>&1
command &> output.log
command &> /dev/null
```

---

## tee

| Command | Description |
|---|---|
| `command | tee file` | Display and save output |
| `command | tee -a file` | Display and append output |

Examples:

```bash
ls | tee files.txt
grep ERROR app.log | tee error-report.txt
```

---

## Pipes

A pipe sends the output of one command into another command.

```bash
command1 | command2
```

Examples:

```bash
ls | wc -l
ps -ef | grep ssh
cat users.txt | sort | uniq
cat users.txt | sort | uniq -c | sort -nr
grep ERROR app.log | wc -l
grep ERROR app.log | sort | uniq -c | sort -nr
grep ERROR app.log | sort | uniq -c | sort -nr | head -1
```

---

# Practical Log Analysis Patterns

Count errors:

```bash
grep ERROR app.log | wc -l
```

Show errors with context:

```bash
grep -C 3 ERROR app.log
```

Find most common error:

```bash
grep ERROR app.log | sort | uniq -c | sort -nr
```

Save report:

```bash
grep ERROR app.log | sort | uniq -c | sort -nr > error-report.txt
```

View and save report:

```bash
grep ERROR app.log | sort | uniq -c | sort -nr | tee error-report.txt
```

---

# Useful Safety Reminders

- Use `less` instead of `cat` for large files.
- Use `>>` when appending to logs.
- Be careful with `>`, because it overwrites files.
- Use `sort` before `uniq`.
- Use `2> /dev/null` only when you intentionally want to hide errors.
- Test long pipelines step by step.

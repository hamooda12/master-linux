# Chapter 03 — Archives, Compression & Backups

# Lesson 02 — Creating Archives with tar

---

# Overview

In the previous lesson, you learned that an archive is simply a container that groups multiple files and directories into a single file.

In this lesson, you will learn how to create archives using the `tar` command. Although `tar` originally stood for **Tape Archive**, today it is the standard tool for packaging files on Linux systems.

Almost every Linux backup, software distribution, and deployment process uses `tar` in some form.

---

# Learning Objectives

After completing this lesson you will be able to:

- Understand the purpose of `tar`
- Create archives from files and directories
- Archive multiple directories at once
- Understand the most common `tar` options
- Preserve directory structures
- Verify that an archive was created successfully

---

# Prerequisites

Before starting this lesson you should understand:

- Files and directories
- Relative and absolute paths
- Difference between archives and compression

---

# What is tar?

`tar` creates a single archive file containing one or more files and directories.

Unlike `zip`, creating a tar archive **does not compress** the data.

Example:

```text
project/
logs/
configs/

↓

project.tar
```

The archive simply stores everything together while preserving:

- Directory hierarchy
- File names
- Permissions
- Ownership (when permitted)
- Modification times

---

# Basic Syntax

```bash
tar [OPTIONS] ARCHIVE_NAME FILES...
```

Example:

```bash
tar -cf backup.tar project
```

---

# Understanding the Most Common Options

| Option | Meaning |
|---------|----------|
| `-c` | Create a new archive |
| `-f` | Use the specified archive filename |
| `-v` | Verbose mode (show processed files) |
| `-t` | List archive contents |
| `-x` | Extract an archive |

For now, we'll focus on:

- `-c`
- `-f`
- `-v`

---

# Creating Your First Archive

Current directory:

```text
chapter-03
├── configs
├── logs
└── project
```

Create an archive:

```bash
tar -cf backup.tar project
```

Now list the files:

```bash
ls
```

Expected output:

```text
backup.tar
configs
logs
project
```

Notice that the original directory still exists.

`tar` copies data into the archive—it does not remove the source files.

---

# Viewing Archive Size

Use:

```bash
ls -lh backup.tar
```

Example:

```text
-rw-r--r-- 1 hamad hamad 10K backup.tar
```

The `-h` option displays file sizes in a human-readable format.

---

# Verbose Mode

Sometimes you want to see exactly what is being archived.

Use:

```bash
tar -cvf backup.tar project
```

Example output:

```text
project/
project/app.py
project/main.py
```

Verbose mode is especially useful for:

- Large backups
- Troubleshooting
- Automation logs

---

# Archiving Multiple Directories

You are not limited to one directory.

Example:

```bash
tar -cf server-backup.tar project logs configs
```

Result:

```text
server-backup.tar
```

Contains:

```text
project/
logs/
configs/
```

---

# Archiving Individual Files

You can archive files instead of directories.

Example:

```bash
tar -cf configs.tar app.conf nginx.conf
```

---

# Archiving Everything in a Directory

Use the wildcard (`*`):

```bash
tar -cf backup.tar *
```

Suppose your directory contains:

```text
configs
logs
project
README.md
notes.txt
```

Everything except hidden files will be archived.

---

# Including Hidden Files

Remember:

```text
*
```

does **not** include hidden files.

To archive everything, including hidden files, you can archive the current directory:

```bash
tar -cf backup.tar .
```

This includes:

- Hidden files
- Hidden directories
- Normal files
- Subdirectories

Be aware that this also includes the current directory itself.

---

# Preserving Metadata

One reason `tar` is widely used for backups is that it preserves important metadata:

- File permissions
- Directory structure
- Symbolic links
- Modification timestamps
- Ownership (when permissions allow)

This makes it ideal for system backups and migrations.

---

# Practical Scenario

You are about to upgrade a production web application.

Before making any changes, your team requires a complete backup.

Current structure:

```text
website/
database/
configs/
```

Create the backup:

```bash
tar -cvf backup-before-upgrade.tar website database configs
```

Now the entire project is safely stored in one archive.

---

# Hands-on Lab

## Step 1

Move into the practice directory.

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2

Verify the structure.

```bash
tree
```

---

## Step 3

Archive the project directory.

```bash
tar -cvf project.tar project
```

---

## Step 4

Archive all directories.

```bash
tar -cvf full-backup.tar project logs configs
```

---

## Step 5

Verify that the archives exist.

```bash
ls -lh
```

---

## Step 6

Compare their sizes.

```bash
du -h *.tar
```

---

# Challenge

Create an archive named:

```text
developer-backup.tar
```

that contains:

- project
- logs

but **not** configs.

Verify that the archive exists.

---

# Common Mistakes

## Forgetting `-f`

Wrong:

```bash
tar -cv backup.tar project
```

Correct:

```bash
tar -cvf backup.tar project
```

---

## Expecting Compression

Many beginners think:

```bash
backup.tar
```

is compressed.

It is **not**.

Compression will be introduced in the next lessons.

---

## Archiving the Wrong Directory

Always check your location:

```bash
pwd
```

before creating an archive.

---

# Best Practices

- Use descriptive archive names.
- Include dates in backup filenames.
- Keep original files until backups are verified.
- Use verbose mode for large backups.
- Store backups outside the source directory when possible.

Example:

```text
backup-2026-07-01.tar
```

---

# Interview Questions

1. What does `tar` stand for?

2. Does `tar` compress files?

3. What does the `-c` option do?

4. What is the purpose of `-f`?

5. Why is `-v` useful?

6. Why is `tar` commonly used for backups?

7. What metadata does `tar` preserve?

---

# Lesson Reflection

Before continuing, make sure you can answer:

- Can I create an archive?
- Can I archive multiple directories?
- Can I explain the difference between `tar` and compression?
- Can I identify the purpose of `-c`, `-f`, and `-v`?
- Do I understand why `tar` is preferred for Linux backups?

---

# Summary

In this lesson, you learned how to create archives using `tar`.

You practiced archiving files and directories, learned the most common command-line options, and saw why `tar` is the foundation of Linux backup workflows.

In the next lesson, you'll learn how to inspect and extract archive contents safely.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `tar -cf backup.tar project` | Create an archive |
| `tar -cvf backup.tar project` | Create archive with verbose output |
| `tar -cf backup.tar project logs` | Archive multiple directories |
| `tar -cf backup.tar *` | Archive all visible files |
| `tar -cf backup.tar .` | Archive current directory including hidden files |
| `ls -lh *.tar` | View archive sizes |
| `du -h *.tar` | Compare archive sizes |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-02/
        ├── tree-before.txt
        ├── tree-after.txt
        ├── archive-output.txt
        ├── archive-size.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- `tree` output before archiving
- `tar -cvf` output
- `ls -lh` results
- `du -h` comparison
- Screenshot of the generated `.tar` files

---

# Next Lesson

**Lesson 03 — Inspecting & Extracting Archives**

You will learn how to:

- View archive contents without extracting
- Extract archives safely
- Extract to another directory
- Prevent accidental overwrites
- Verify restored files

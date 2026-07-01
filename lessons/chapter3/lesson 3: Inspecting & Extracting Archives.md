# Chapter 03 — Archives, Compression & Backups

# Lesson 03 — Inspecting & Extracting Archives

---

# Overview

Creating an archive is only half of the backup process.

A backup is valuable only if it can be restored successfully.

Before extracting an archive, it is often useful to inspect its contents to verify that it contains the expected files and directories.

In this lesson, you'll learn how to:

- View archive contents
- Extract entire archives
- Extract specific files
- Extract to another directory
- Prevent accidental mistakes when restoring backups

---

# Learning Objectives

After completing this lesson you will be able to:

- List archive contents
- Extract archives safely
- Restore individual files
- Extract archives into another directory
- Verify restored files

---

# Prerequisites

Before beginning, you should know how to:

- Create archives using `tar`
- Navigate directories
- Understand Linux paths

---

# Listing Archive Contents

Suppose you have an archive named:

```text
backup.tar
```

Instead of extracting it immediately, inspect its contents.

```bash
tar -tf backup.tar
```

Example output:

```text
project/
project/app.py
project/main.py
logs/
logs/system.log
configs/
configs/nginx.conf
```

Notice that nothing has been extracted.

The archive is simply being read.

---

# Why Inspect an Archive?

Imagine receiving a backup from another administrator.

You probably want to verify:

- Does it contain the correct files?
- Is the expected directory structure present?
- Is anything missing?
- Is it safe to restore?

Listing the contents answers these questions before making changes to your filesystem.

---

# Verbose Listing

To display more information while listing:

```bash
tar -tvf backup.tar
```

Example:

```text
drwxr-xr-x root/root         0 2026-07-01 10:00 project/
-rw-r--r-- root/root       243 2026-07-01 10:00 project/app.py
-rw-r--r-- root/root       812 2026-07-01 10:00 project/main.py
```

The verbose listing shows:

- Permissions
- Owner
- Group
- File size
- Modification date
- Filename

This is useful when verifying backups.

---

# Extracting an Archive

To restore everything:

```bash
tar -xf backup.tar
```

After extraction:

```text
backup.tar
project/
logs/
configs/
```

The archive remains unchanged.

The files are copied out of it.

---

# Using Verbose Extraction

To see each restored file:

```bash
tar -xvf backup.tar
```

Example:

```text
project/
project/app.py
project/main.py
logs/system.log
configs/nginx.conf
```

---

# Where Are Files Extracted?

By default, extraction occurs in the current directory.

Check where you are:

```bash
pwd
```

Always verify your location before restoring a backup.

---

# Extracting into Another Directory

Suppose you want to restore files elsewhere.

Create a destination:

```bash
mkdir restore-test
```

Extract using:

```bash
tar -xf backup.tar -C restore-test
```

Notice the uppercase `-C`.

Result:

```text
restore-test/
├── project
├── logs
└── configs
```

The original working directory remains untouched.

---

# Extracting a Single File

Sometimes you need only one file.

Example:

```bash
tar -xf backup.tar project/main.py
```

Only that file is restored.

This is extremely useful when recovering accidentally deleted files.

---

# Extracting Multiple Files

You can specify several paths.

Example:

```bash
tar -xf backup.tar \
project/app.py \
configs/nginx.conf
```

Only the requested files are extracted.

---

# Viewing Before Extracting

A common workflow is:

```text
Receive Backup

↓

List Contents

↓

Verify Files

↓

Extract
```

This minimizes mistakes.

---

# Overwriting Existing Files

Suppose the destination already contains:

```text
project/app.py
```

Extracting the archive may overwrite it.

Therefore, administrators often restore into a temporary directory first:

```text
backup.tar

↓

restore-test/

↓

Verify

↓

Copy Needed Files
```

This approach is much safer.

---

# Practical Scenario

A production server crashes.

You have a backup named:

```text
production-backup.tar
```

Instead of restoring immediately:

1. Inspect the archive.

```bash
tar -tf production-backup.tar
```

2. Restore into a temporary directory.

```bash
mkdir recovery
tar -xf production-backup.tar -C recovery
```

3. Verify the files.

4. Replace only the required application data.

This workflow reduces the chance of damaging an already running system.

---

# Hands-on Lab

## Step 1

Move to your practice directory.

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2

Display available archives.

```bash
ls *.tar
```

---

## Step 3

Inspect an archive.

```bash
tar -tf project.tar
```

---

## Step 4

View detailed information.

```bash
tar -tvf project.tar
```

---

## Step 5

Create a restoration directory.

```bash
mkdir restore
```

---

## Step 6

Restore the archive.

```bash
tar -xf project.tar -C restore
```

---

## Step 7

Verify the restored files.

```bash
tree restore
```

---

## Step 8

Restore a single file.

```bash
tar -xf project.tar project/main.py
```

---

# Challenge

Create:

```text
test-restore/
```

Restore only:

```text
project/app.py
```

into the current directory.

Then verify it exists.

Bonus challenge:

Restore the entire archive into:

```text
backup-check/
```

without modifying your original project directory.

---

# Common Mistakes

## Forgetting `-C`

Wrong:

```bash
tar -xf backup.tar restore
```

This attempts to extract a file named `restore`.

Correct:

```bash
tar -xf backup.tar -C restore
```

---

## Restoring in the Wrong Directory

Always check:

```bash
pwd
```

before extracting.

---

## Skipping Inspection

Never assume an archive contains what you expect.

Always inspect it first.

---

# Best Practices

- Always inspect archives before extraction.
- Restore into a temporary directory when possible.
- Verify restored files before replacing production data.
- Keep original backups unchanged.
- Use verbose mode during troubleshooting.

---

# Interview Questions

1. How do you list the contents of a tar archive?

2. What is the difference between `-t` and `-x`?

3. What does the `-C` option do?

4. Why should backups be restored into a temporary directory?

5. How can you restore only one file from an archive?

6. Does extracting an archive delete the archive itself?

---

# Lesson Reflection

Before moving on, ensure you can answer:

- Can I inspect an archive without extracting it?
- Can I restore an archive into another directory?
- Can I recover a single file?
- Do I understand why administrators verify backups before restoring them?

---

# Summary

In this lesson, you learned how to inspect and restore archives using `tar`.

You practiced listing archive contents, extracting complete backups, restoring individual files, and safely recovering data into a separate directory.

These are essential skills for backup verification, disaster recovery, and day-to-day Linux administration.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `tar -tf backup.tar` | List archive contents |
| `tar -tvf backup.tar` | Detailed archive listing |
| `tar -xf backup.tar` | Extract archive |
| `tar -xvf backup.tar` | Extract with verbose output |
| `tar -xf backup.tar -C restore/` | Extract into another directory |
| `tar -xf backup.tar project/main.py` | Extract a single file |
| `pwd` | Verify current directory before extraction |
| `tree restore` | Verify restored files |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-03/
        ├── archive-list.txt
        ├── verbose-list.txt
        ├── restore-tree.txt
        ├── extracted-file.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- Output of `tar -tf`
- Output of `tar -tvf`
- `tree` after extraction
- Screenshot of the restored directory
- Notes comparing full extraction and single-file extraction

---

# Next Lesson

**Lesson 04 — Compression with gzip**

You'll learn:

- How `gzip` works
- Compressing files
- Decompressing files
- Compression ratios
- When to use `gzip`
- How `tar` and `gzip` work together to create `.tar.gz` archives

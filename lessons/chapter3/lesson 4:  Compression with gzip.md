# Chapter 03 — Archives, Compression & Backups

# Lesson 04 — Compression with gzip

---

# Overview

In the previous lessons, you learned how to create, inspect, and extract archives using `tar`.

Now we move to compression.

Compression reduces file size. This helps save disk space, reduce transfer time, and make backups easier to store.

In Linux, one of the most common compression tools is:

```bash
gzip
```

`gzip` is fast, reliable, and widely supported.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what `gzip` does
- Compress files using `gzip`
- Decompress `.gz` files
- View compressed file sizes
- Understand what happens to the original file
- Compare original and compressed sizes
- Understand when `gzip` is useful

---

# Prerequisites

Before this lesson, you should understand:

- What compression is
- How files and directories work
- How to use `ls`, `du`, and `cat`
- How archives differ from compression

---

# What Is gzip?

`gzip` is a Linux compression tool.

It compresses a single file and creates a new file ending with:

```text
.gz
```

Example:

```text
system.log
```

becomes:

```text
system.log.gz
```

Important:

`gzip` compresses files, not directories directly.

If you want to compress a directory, you usually archive it first with `tar`.

---

# Basic gzip Syntax

```bash
gzip filename
```

Example:

```bash
gzip system.log
```

Result:

```text
system.log.gz
```

By default, `gzip` replaces the original file with the compressed version.

---

# Example

Before compression:

```bash
ls
```

```text
system.log
```

Compress it:

```bash
gzip system.log
```

After compression:

```bash
ls
```

```text
system.log.gz
```

The original file is gone because `gzip` replaced it with the compressed version.

---

# Keeping the Original File

If you want to keep the original file, use:

```bash
gzip -k system.log
```

Now you will have:

```text
system.log
system.log.gz
```

The `-k` option means:

```text
keep original file
```

---

# Viewing File Sizes

Use:

```bash
ls -lh
```

Example:

```text
-rw-r--r-- 1 hamad hamad 100K system.log
-rw-r--r-- 1 hamad hamad 20K system.log.gz
```

You can also use:

```bash
du -h system.log*
```

---

# Decompressing with gunzip

To decompress a `.gz` file:

```bash
gunzip system.log.gz
```

Result:

```text
system.log
```

By default, `gunzip` removes the `.gz` file and restores the original file.

---

# Decompressing While Keeping the Compressed File

Use:

```bash
gunzip -k system.log.gz
```

Now both files remain:

```text
system.log
system.log.gz
```

---

# Using gzip -d

`gunzip` and `gzip -d` do the same basic job.

```bash
gzip -d system.log.gz
```

This decompresses the file.

---

# Testing a gzip File

To check whether a `.gz` file is valid:

```bash
gzip -t system.log.gz
```

If there is no output, the file is valid.

You can confirm with:

```bash
echo $?
```

Expected result:

```text
0
```

Exit code `0` means success.

---

# Viewing Compression Information

Use:

```bash
gzip -l system.log.gz
```

Example output:

```text
compressed        uncompressed  ratio uncompressed_name
20480             102400        80.0% system.log
```

This shows:

- Compressed size
- Original size
- Compression ratio
- Original filename

---

# Compression Levels

`gzip` supports compression levels from `1` to `9`.

| Level | Meaning |
|---|---|
| `-1` | Fastest, less compression |
| `-6` | Default balance |
| `-9` | Slowest, best compression |

Example:

```bash
gzip -9 system.log
```

This tries to produce the smallest file, but it may take longer.

---

# Why gzip Is Common

`gzip` is popular because it is:

- Fast
- Simple
- Installed on most Linux systems
- Good for logs and text files
- Common in backups and software packages

It is often used with `tar` to create:

```text
.tar.gz
```

---

# gzip and Directories

This will not work as expected:

```bash
gzip project/
```

`gzip` does not compress directories directly.

Correct workflow:

```text
project/
   ↓
project.tar
   ↓
project.tar.gz
```

Or using one command later:

```bash
tar -czf project.tar.gz project/
```

We will cover this fully in a later lesson.

---

# Practical Scenario

You are a Linux administrator.

The server has large log files:

```text
system.log
error.log
access.log
```

They are taking too much space.

Instead of deleting them, you compress old logs:

```bash
gzip access.log
gzip error.log
```

Now the logs are smaller but still recoverable.

This is common in real Linux servers.

---

# Hands-on Lab

## Step 1 — Move to the Chapter Directory

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2 — Create a gzip Practice Directory

```bash
mkdir -p gzip-lab
cd gzip-lab
```

---

## Step 3 — Create a Large Text File

```bash
for i in {1..1000}; do echo "This is a repeated log line for gzip practice." >> system.log; done
```

---

## Step 4 — Check the File Size

```bash
ls -lh system.log
```

---

## Step 5 — Compress the File

```bash
gzip system.log
```

---

## Step 6 — Verify the Result

```bash
ls -lh
```

Expected file:

```text
system.log.gz
```

---

## Step 7 — View Compression Information

```bash
gzip -l system.log.gz
```

---

## Step 8 — Test the Compressed File

```bash
gzip -t system.log.gz
echo $?
```

Expected output:

```text
0
```

---

## Step 9 — Decompress the File

```bash
gunzip system.log.gz
```

---

## Step 10 — Keep Original While Compressing

```bash
gzip -k system.log
```

Now verify:

```bash
ls -lh
```

You should see:

```text
system.log
system.log.gz
```

---

# Challenge

Create a file named:

```text
app.log
```

Add repeated content to it:

```bash
for i in {1..500}; do echo "Application backup log entry number $i" >> app.log; done
```

Then:

1. Check its size.
2. Compress it with `gzip -k`.
3. Verify the `.gz` file exists.
4. Show compression details using `gzip -l`.
5. Test the compressed file with `gzip -t`.

---

# Common Mistakes

## Mistake 1 — Thinking gzip Archives Directories

Wrong idea:

```text
gzip = archive + compression
```

Correct:

```text
gzip = compression only
```

---

## Mistake 2 — Forgetting gzip Removes the Original

This command:

```bash
gzip file.txt
```

replaces:

```text
file.txt
```

with:

```text
file.txt.gz
```

Use `-k` if you want to keep the original.

---

## Mistake 3 — Ignoring Backup Verification

Always test important compressed files:

```bash
gzip -t file.gz
```

---

# Best Practices

- Use `gzip -k` when practicing.
- Test compressed files with `gzip -t`.
- Use `gzip -l` to inspect compression results.
- Use `gzip` for text files, logs, and tar archives.
- Do not delete original data until the compressed file is verified.

---

# Interview Questions

1. What does `gzip` do?

2. Does `gzip` archive multiple files?

3. What file extension does `gzip` create?

4. What happens to the original file by default?

5. How do you keep the original file when compressing?

6. How do you decompress a `.gz` file?

7. How do you test whether a `.gz` file is valid?

8. Why is `gzip` often used with `tar`?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I compress a file with `gzip`?
- Can I decompress a `.gz` file?
- Can I keep the original file?
- Can I test a compressed file?
- Do I understand why directories need `tar` before `gzip`?

---

# Summary

In this lesson, you learned how to use `gzip` to compress files.

You practiced compressing, decompressing, keeping original files, checking compression ratios, and testing compressed files.

`gzip` is one of the most common Linux compression tools and is widely used in backups, log rotation, software packaging, and server administration.

---

# Cheat Sheet

| Command | Description |
|---|---|
| `gzip file.txt` | Compress file and remove original |
| `gzip -k file.txt` | Compress file and keep original |
| `gunzip file.txt.gz` | Decompress file |
| `gunzip -k file.txt.gz` | Decompress and keep `.gz` file |
| `gzip -d file.txt.gz` | Decompress using gzip |
| `gzip -t file.txt.gz` | Test compressed file integrity |
| `gzip -l file.txt.gz` | Show compression information |
| `gzip -1 file.txt` | Fast compression |
| `gzip -9 file.txt` | Best compression |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-04/
        ├── original-size.txt
        ├── compressed-size.txt
        ├── gzip-list-output.txt
        ├── gzip-test-output.txt
        ├── gzip-lab-tree.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- `ls -lh` before compression
- `ls -lh` after compression
- `gzip -l` output
- `gzip -t` result
- Screenshot of the `gzip-lab` directory
- Notes explaining what happened to the original file

---

# Next Lesson

**Lesson 05 — Compression with bzip2**

You will learn:

- How `bzip2` differs from `gzip`
- How to compress files with `bzip2`
- How to decompress `.bz2` files
- When to choose `bzip2`
- How to compare compression results

# Chapter 03 — Archives, Compression & Backups

# Lesson 05 — Compression with bzip2

---

# Overview

In the previous lesson, you learned how to compress files using `gzip`.

While `gzip` is fast and widely used, it is not the only compression utility available on Linux.

Another popular tool is **bzip2**.

Compared to `gzip`, `bzip2` usually produces **smaller compressed files**, but this comes at the cost of **slower compression and decompression**.

Choosing between `gzip` and `bzip2` depends on whether you prioritize speed or storage efficiency.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what `bzip2` is
- Compress files using `bzip2`
- Decompress `.bz2` files
- Test compressed files
- Compare `bzip2` with `gzip`
- Know when to use `bzip2`

---

# Prerequisites

Before this lesson, you should understand:

- File compression
- The `gzip` command
- Basic file management
- File size comparison using `ls -lh`

---

# What Is bzip2?

`bzip2` is a compression program for Linux and Unix systems.

Like `gzip`, it compresses **one file at a time**.

Unlike `tar`, it does **not** archive directories.

Compressed files receive the extension:

```text
.bz2
```

Example:

```text
report.txt
```

becomes

```text
report.txt.bz2
```

---

# Basic Syntax

Compress a file:

```bash
bzip2 filename
```

Example:

```bash
bzip2 report.txt
```

Result:

```text
report.txt.bz2
```

The original file is removed after compression.

---

# Keeping the Original File

To preserve the source file:

```bash
bzip2 -k report.txt
```

Now both files exist:

```text
report.txt
report.txt.bz2
```

---

# Viewing the Result

```bash
ls -lh
```

Example:

```text
-rw-r--r-- 1 hamad hamad 150K report.txt
-rw-r--r-- 1 hamad hamad  42K report.txt.bz2
```

---

# Decompressing Files

Use:

```bash
bunzip2 report.txt.bz2
```

Result:

```text
report.txt
```

The compressed file is removed by default.

---

# Keeping the Compressed File

```bash
bunzip2 -k report.txt.bz2
```

Now both versions remain.

---

# Using bzip2 -d

You can also decompress using:

```bash
bzip2 -d report.txt.bz2
```

This performs the same task as `bunzip2`.

---

# Testing a Compressed File

Verify integrity:

```bash
bzip2 -t report.txt.bz2
```

Check the exit code:

```bash
echo $?
```

Expected output:

```text
0
```

---

# Compression Levels

Unlike `gzip`, `bzip2` does not commonly expose multiple compression levels for everyday use.

Instead, it focuses on producing better compression than `gzip` while remaining relatively simple to use.

---

# Comparing gzip and bzip2

Suppose you compress the same log file.

Original:

```text
system.log
```

Using `gzip`:

```text
system.log.gz
```

Size:

```text
24 MB
```

Using `bzip2`:

```text
system.log.bz2
```

Size:

```text
20 MB
```

The exact result depends on the data being compressed, but `bzip2` often creates a smaller file.

---

# Speed Comparison

```text
Compression Speed

gzip
████████████████████

bzip2
██████████
```

```text
Compression Ratio

gzip
██████████████

bzip2
██████████████████
```

General rule:

- `gzip` is faster.
- `bzip2` usually compresses better.

---

# Typical Use Cases

Use `bzip2` when:

- Long-term backups
- Archiving documentation
- Source code releases
- Saving storage space is more important than speed

Use `gzip` when:

- Speed is important
- Frequently accessed files
- Log rotation
- Daily backups

---

# Practical Scenario

A company stores monthly database exports for several years.

These backups are rarely accessed.

Saving disk space is more important than compression speed.

The administrator chooses:

```bash
bzip2 monthly-backup.sql
```

to reduce storage requirements.

---

# Hands-on Lab

## Step 1 — Enter the Practice Directory

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2 — Create a New Lab

```bash
mkdir -p bzip2-lab
cd bzip2-lab
```

---

## Step 3 — Create a Test File

```bash
for i in {1..2000}; do
    echo "Linux backup practice using bzip2." >> report.txt
done
```

---

## Step 4 — View File Size

```bash
ls -lh report.txt
```

---

## Step 5 — Compress the File

```bash
bzip2 report.txt
```

---

## Step 6 — Verify the Result

```bash
ls -lh
```

Expected output:

```text
report.txt.bz2
```

---

## Step 7 — Test the Archive

```bash
bzip2 -t report.txt.bz2
echo $?
```

---

## Step 8 — Restore the File

```bash
bunzip2 report.txt.bz2
```

---

## Step 9 — Compress Again While Keeping the Original

```bash
bzip2 -k report.txt
```

Verify:

```bash
ls -lh
```

---

# Challenge

Create:

```text
logs.txt
```

Fill it with repeated content.

Then:

1. Compress it using `bzip2 -k`.
2. Verify both files exist.
3. Test the compressed file.
4. Restore it.
5. Compare its size with the original.

Bonus:

Compare the same file compressed with both:

```bash
gzip
```

and

```bash
bzip2
```

Which file is smaller?

---

# Common Mistakes

## Mistake 1

Thinking `bzip2` archives directories.

It does not.

Use:

```text
tar
```

first.

---

## Mistake 2

Forgetting that the original file is removed.

Unless you use:

```bash
-k
```

---

## Mistake 3

Assuming better compression always means better performance.

Better compression usually requires more CPU time.

---

# Best Practices

- Use `bzip2` for long-term storage.
- Test compressed files before deleting originals.
- Keep original files during practice.
- Archive directories with `tar` before compressing.
- Choose the compression tool based on your requirements.

---

# Interview Questions

1. What is the difference between `gzip` and `bzip2`?

2. Which one usually creates smaller files?

3. Which one is faster?

4. How do you keep the original file?

5. How do you test a `.bz2` file?

6. Why can't `bzip2` compress directories directly?

---

# Lesson Reflection

Before moving on, ensure you can answer:

- Can I compress a file using `bzip2`?
- Can I restore a `.bz2` file?
- Can I explain the trade-off between `gzip` and `bzip2`?
- Do I know when `bzip2` is a better choice?

---

# Summary

In this lesson, you learned how to use `bzip2` to compress and decompress files.

You compared it with `gzip` and saw that `bzip2` generally offers better compression at the cost of slower processing.

Understanding these trade-offs helps you choose the right compression tool for different workloads.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `bzip2 file.txt` | Compress a file |
| `bzip2 -k file.txt` | Compress and keep original |
| `bunzip2 file.txt.bz2` | Decompress |
| `bunzip2 -k file.txt.bz2` | Decompress and keep `.bz2` |
| `bzip2 -d file.txt.bz2` | Decompress using `bzip2` |
| `bzip2 -t file.txt.bz2` | Test compressed file |
| `ls -lh` | Compare file sizes |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-05/
        ├── original-size.txt
        ├── compressed-size.txt
        ├── bzip2-test-output.txt
        ├── comparison-notes.md
        ├── screenshots/
        └── lab-tree.txt
```

Capture:

- File size before compression
- File size after compression
- `bzip2 -t` output
- Screenshot of the lab directory
- Notes comparing `gzip` and `bzip2`

---

# Next Lesson

**Lesson 06 — Compression with xz**

You will learn:

- Why `xz` is considered one of the best compression algorithms available on Linux
- How to compress and decompress `.xz` files
- Performance vs compression trade-offs
- Why many Linux distributions use `.tar.xz` for software packages

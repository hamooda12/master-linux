# Chapter 03 — Archives, Compression & Backups

# Lesson 01 — Archives vs Compression

---

# Overview

When people first start using Linux, they often think that an archive and a compressed file are the same thing.

They are **not**.

Understanding the difference is essential because Linux typically performs these two operations separately.

Most Linux backup files are actually:

1. Archived
2. Compressed

This design gives Linux great flexibility and is one of the reasons why Linux backup tools are so powerful.

---

# Learning Objectives

After completing this lesson you will be able to:

- Explain what an archive is.
- Explain what compression is.
- Describe why Linux separates archiving from compression.
- Understand why `.tar.gz` is actually two operations.
- Choose the correct tool depending on the task.
- Recognize common archive file extensions.

---

# Prerequisites

Before starting this lesson, you should be comfortable with:

- Navigating the filesystem
- Creating directories
- Creating files
- Viewing directory trees
- Basic Linux commands

---

# Why Do We Need Archives?

Imagine you have a project like this:

```text
website/
├── index.html
├── style.css
├── script.js
├── images/
│   ├── logo.png
│   └── banner.jpg
└── config/
    └── nginx.conf
```

Suppose you need to send this project to another developer.

Sending every file individually would be inconvenient.

Instead, you package everything into **one file**.

That package is called an **archive**.

Think of an archive like a shipping box.

```
Many Files
      │
      ▼
+----------------+
|    Archive     |
|   website.tar  |
+----------------+
```

An archive simply collects multiple files and directories into a single file.

It does **not** necessarily reduce the file size.

---

# What Is Compression?

Compression is a completely different process.

Instead of collecting files, compression focuses on reducing storage space.

For example:

```text
100 MB
   │
Compression
   ▼
40 MB
```

Compression works by identifying repeated patterns within data and storing them more efficiently.

The result is a smaller file that saves:

- Disk space
- Network bandwidth
- Backup storage
- Download time

---

# Archive vs Compression

| Archive | Compression |
|----------|-------------|
| Combines multiple files | Reduces file size |
| Keeps directory structure | Does not organize files |
| Usually no size reduction | Produces a smaller file |
| Created with `tar` | Created with `gzip`, `bzip2`, `xz`, or `zip` |

---

# Why Linux Separates Them

Many operating systems combine these concepts into one tool.

Linux separates them intentionally.

This allows you to:

- Archive without compression
- Compress existing files
- Choose different compression algorithms
- Replace one compressor without changing the archive format

This modular approach follows the Unix philosophy:

> Do one thing and do it well.

---

# The Two-Step Process

When you create a `.tar.gz` file, Linux performs two separate operations.

```
Project Folder
      │
      ▼
Create Archive
      │
website.tar
      │
Compress
      ▼
website.tar.gz
```

Notice that the archive is created **before** compression.

---

# Common File Extensions

| Extension | Meaning |
|-----------|---------|
| `.tar` | Archive only |
| `.gz` | Gzip-compressed file |
| `.bz2` | Bzip2-compressed file |
| `.xz` | XZ-compressed file |
| `.tar.gz` | Tar archive compressed with Gzip |
| `.tar.bz2` | Tar archive compressed with Bzip2 |
| `.tar.xz` | Tar archive compressed with XZ |
| `.zip` | Archive and compression combined |

---

# Why Not Compress Every File Individually?

Suppose a project contains 5,000 files.

Without an archive:

```
5000 files

↓

Compress each one

↓

5000 compressed files
```

Managing thousands of compressed files would be difficult.

Instead:

```
5000 files

↓

One archive

↓

One compressed archive
```

This is easier to:

- Copy
- Move
- Share
- Store
- Restore

---

# Real-World Examples

## Example 1 – Source Code Backup

A software developer backs up an application before deploying a new release.

The entire project directory is archived and compressed into a single file for safe storage.

---

## Example 2 – Log Archiving

System administrators archive old log files to reduce disk usage while preserving historical records.

---

## Example 3 – Database Backup

Database dumps are often archived and compressed before being uploaded to cloud storage.

---

# Hands-on Lab

## Step 1

Create a practice directory.

```bash
mkdir -p ~/linux-practice/chapter-03/demo
```

---

## Step 2

Create some files.

```bash
touch file1.txt
touch file2.txt
touch file3.txt
```

---

## Step 3

Display the directory.

```bash
tree
```

Expected output:

```text
demo
├── file1.txt
├── file2.txt
└── file3.txt
```

No archive has been created yet.

No compression has been performed.

---

# Practical Scenario

You are a junior Linux administrator.

Before upgrading a production web server, your team lead asks you to create a backup of the entire application.

Instead of copying thousands of files individually, you first create an archive and then compress it.

This produces a single backup file that is easy to transfer and restore if the upgrade fails.

---

# Common Mistakes

### Mistake 1

Thinking `.tar` is compressed.

It is not.

---

### Mistake 2

Thinking `.gz` stores multiple files.

It normally compresses a single file.

---

### Mistake 3

Assuming every archive format performs compression.

Many archive formats simply package files together.

---

# Best Practices

- Archive before compressing.
- Keep original files until backups are verified.
- Use meaningful backup filenames.
- Include timestamps in backup names.
- Store backups in a dedicated backup directory.

---

# Interview Questions

### What is the difference between an archive and compression?

---

### Why is `.tar.gz` considered two operations?

---

### Why does Linux separate archiving from compression?

---

### When would you use a `.tar` file without compression?

---

### What are the advantages of storing backups as a single archive?

---

# Lesson Reflection

Before moving on, make sure you can answer these questions:

- Can I explain the difference between archiving and compression?
- Do I understand why Linux performs them separately?
- Can I identify the purpose of `.tar`, `.gz`, and `.tar.gz`?
- Do I understand why archives are commonly used for backups?

---

# Summary

In this lesson you learned that archives and compression solve different problems.

An archive combines many files into one container, while compression reduces the amount of storage required.

Linux keeps these operations separate, allowing administrators to combine tools in flexible ways.

This design follows the Unix philosophy and forms the basis of nearly every Linux backup workflow.

---

# Cheat Sheet

| File Type | Purpose |
|-----------|---------|
| `.tar` | Archive only |
| `.gz` | Compression |
| `.bz2` | Compression |
| `.xz` | Compression |
| `.tar.gz` | Archive + Gzip |
| `.tar.bz2` | Archive + Bzip2 |
| `.tar.xz` | Archive + XZ |
| `.zip` | Archive + Compression |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-01/
        ├── terminal-output.txt
        ├── directory-tree.txt
        ├── archive-vs-compression-diagram.txt
        └── screenshots/
```

Capture:

- Output of `tree`
- Screenshot of the practice directory
- Notes explaining the difference between archiving and compression
- The ASCII diagrams from this lesson

---

# Next Lesson

**Lesson 02 — Creating Archives with `tar`**

You will learn how to:

- Create archives
- Archive multiple directories
- Preserve file permissions
- Preserve ownership
- Verify archive contents
- Understand the most commonly used `tar` options

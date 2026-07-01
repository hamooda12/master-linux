# Chapter 03 — Archives, Compression & Backups

# Lesson 07 — Working with `.tar.gz`, `.tar.bz2`, and `.tar.xz`

---

# Overview

So far, you learned two separate skills:

1. Creating archives with `tar`
2. Compressing files with `gzip`, `bzip2`, and `xz`

Now we combine both skills.

In real Linux systems, backups are often stored as compressed tar archives:

```text
.tar.gz
.tar.bz2
.tar.xz
```

These files are common in:

- Server backups
- Source code packages
- Linux software downloads
- DevOps deployment bundles
- Long-term storage archives

---

# Learning Objectives

After completing this lesson, you will be able to:

- Create compressed tar archives.
- Extract compressed tar archives.
- List contents without extracting.
- Understand `tar` compression options.
- Choose between `.tar.gz`, `.tar.bz2`, and `.tar.xz`.

---

# Key Idea

A compressed tar archive is created in two logical steps:

```text
Directory
   ↓
.tar archive
   ↓
Compressed archive
```

But `tar` can do both in one command.

---

# Common tar Compression Options

| Option | Compression Tool | Extension |
|---|---|---|
| `-z` | gzip | `.tar.gz` |
| `-j` | bzip2 | `.tar.bz2` |
| `-J` | xz | `.tar.xz` |

Remember:

```text
-z = gzip
-j = bzip2
-J = xz
```

---

# Creating a `.tar.gz` Archive

```bash
tar -czvf project.tar.gz project/
```

Meaning:

| Option | Meaning |
|---|---|
| `-c` | Create archive |
| `-z` | Compress with gzip |
| `-v` | Verbose output |
| `-f` | Filename follows |

Result:

```text
project.tar.gz
```

---

# Creating a `.tar.bz2` Archive

```bash
tar -cjvf project.tar.bz2 project/
```

Meaning:

| Option | Meaning |
|---|---|
| `-c` | Create archive |
| `-j` | Compress with bzip2 |
| `-v` | Verbose output |
| `-f` | Filename follows |

---

# Creating a `.tar.xz` Archive

```bash
tar -cJvf project.tar.xz project/
```

Meaning:

| Option | Meaning |
|---|---|
| `-c` | Create archive |
| `-J` | Compress with xz |
| `-v` | Verbose output |
| `-f` | Filename follows |

Important:

```text
-J is uppercase.
```

Lowercase `-j` is for `bzip2`.

---

# Visual Workflow

```text
project/
├── app.py
├── main.py
└── README.md

        │
        ▼

tar creates archive

        │
        ▼

project.tar

        │
        ▼

gzip / bzip2 / xz compresses it

        │
        ▼

project.tar.gz
project.tar.bz2
project.tar.xz
```

---

# Listing Compressed Archive Contents

You can list contents without extracting.

## `.tar.gz`

```bash
tar -tzvf project.tar.gz
```

## `.tar.bz2`

```bash
tar -tjvf project.tar.bz2
```

## `.tar.xz`

```bash
tar -tJvf project.tar.xz
```

---

# Extracting Compressed Archives

## Extract `.tar.gz`

```bash
tar -xzvf project.tar.gz
```

## Extract `.tar.bz2`

```bash
tar -xjvf project.tar.bz2
```

## Extract `.tar.xz`

```bash
tar -xJvf project.tar.xz
```

---

# Extracting to Another Directory

Create restore directories:

```bash
mkdir restore-gz restore-bz2 restore-xz
```

Extract:

```bash
tar -xzvf project.tar.gz -C restore-gz
tar -xjvf project.tar.bz2 -C restore-bz2
tar -xJvf project.tar.xz -C restore-xz
```

---

# Modern tar Auto-Detection

Many modern versions of `tar` can detect compression automatically when extracting.

So this often works:

```bash
tar -xf project.tar.gz
tar -xf project.tar.bz2
tar -xf project.tar.xz
```

But as a beginner, it is useful to know the explicit options:

```text
-z
-j
-J
```

They help you understand what is happening.

---

# Choosing the Right Format

| Format | Speed | Compression | Best For |
|---|---|---|---|
| `.tar.gz` | Fast | Good | Daily backups, logs, quick transfers |
| `.tar.bz2` | Slower | Better | Older systems, long-term storage |
| `.tar.xz` | Slowest | Often best | Software packages, maximum space saving |

---

# Practical Scenario

You are preparing a deployment package for a web application.

The project directory is:

```text
webapp/
```

You want a compressed archive that is easy to upload and widely supported.

You choose:

```bash
tar -czvf webapp-release.tar.gz webapp/
```

For long-term storage, you might choose:

```bash
tar -cJvf webapp-archive.tar.xz webapp/
```

---

# Hands-on Lab

## Step 1 — Move to Chapter Directory

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2 — Create Lab Directory

```bash
mkdir -p compressed-tar-lab
cd compressed-tar-lab
```

---

## Step 3 — Create Project Files

```bash
mkdir -p project/src project/config project/logs

echo "print('Hello Linux backups')" > project/src/app.py
echo "APP_ENV=dev" > project/config/app.conf

for i in {1..1000}; do
    echo "Log entry number $i" >> project/logs/app.log
done
```

---

## Step 4 — View Project Structure

```bash
tree project
```

---

## Step 5 — Create `.tar.gz`

```bash
tar -czvf project.tar.gz project/
```

---

## Step 6 — Create `.tar.bz2`

```bash
tar -cjvf project.tar.bz2 project/
```

---

## Step 7 — Create `.tar.xz`

```bash
tar -cJvf project.tar.xz project/
```

---

## Step 8 — Compare Sizes

```bash
ls -lh project.tar.*
```

---

## Step 9 — List Contents

```bash
tar -tzvf project.tar.gz
tar -tjvf project.tar.bz2
tar -tJvf project.tar.xz
```

---

## Step 10 — Restore Each Archive

```bash
mkdir restore-gz restore-bz2 restore-xz

tar -xzvf project.tar.gz -C restore-gz
tar -xjvf project.tar.bz2 -C restore-bz2
tar -xJvf project.tar.xz -C restore-xz
```

---

## Step 11 — Verify Restores

```bash
tree restore-gz
tree restore-bz2
tree restore-xz
```

---

# Challenge

Create a directory named:

```text
server-config/
```

Inside it, create:

```text
nginx.conf
app.conf
database.conf
```

Then create three compressed archives:

```text
server-config.tar.gz
server-config.tar.bz2
server-config.tar.xz
```

After that:

1. Compare their sizes.
2. List the contents of each archive.
3. Restore each one into a different directory.
4. Verify the restored structure using `tree`.

---

# Common Mistakes

## Mistake 1 — Confusing `-j` and `-J`

```text
-j = bzip2
-J = xz
```

They are different.

---

## Mistake 2 — Forgetting `-f`

Wrong:

```bash
tar -czv project.tar.gz project/
```

Correct:

```bash
tar -czvf project.tar.gz project/
```

---

## Mistake 3 — Assuming Smallest Is Always Best

`.tar.xz` may be smallest, but it can be slower.

For daily backups, `.tar.gz` is often better.

---

# Best Practices

- Use `.tar.gz` for general-purpose backups.
- Use `.tar.xz` when storage space matters.
- Restore into a test directory before replacing real files.
- Always inspect important archives before extraction.
- Use clear filenames with dates.

Example:

```text
webapp-backup-2026-07-01.tar.gz
```

---

# Interview Questions

1. What does `.tar.gz` mean?

2. Which `tar` option enables gzip compression?

3. Which option enables bzip2 compression?

4. Which option enables xz compression?

5. What is the difference between `.tar` and `.tar.gz`?

6. How do you extract a `.tar.xz` file?

7. Why might `.tar.gz` be preferred for daily backups?

8. Why might `.tar.xz` be preferred for long-term storage?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I create `.tar.gz`, `.tar.bz2`, and `.tar.xz` files?
- Can I extract each compressed archive type?
- Can I list contents without extracting?
- Can I explain the trade-off between speed and compression ratio?

---

# Summary

In this lesson, you combined archiving and compression.

You learned how to create and extract:

```text
.tar.gz
.tar.bz2
.tar.xz
```

These formats are essential in Linux administration, DevOps, backups, and software distribution.

---

# Cheat Sheet

| Command | Description |
|---|---|
| `tar -czvf file.tar.gz dir/` | Create gzip-compressed archive |
| `tar -cjvf file.tar.bz2 dir/` | Create bzip2-compressed archive |
| `tar -cJvf file.tar.xz dir/` | Create xz-compressed archive |
| `tar -tzvf file.tar.gz` | List `.tar.gz` contents |
| `tar -tjvf file.tar.bz2` | List `.tar.bz2` contents |
| `tar -tJvf file.tar.xz` | List `.tar.xz` contents |
| `tar -xzvf file.tar.gz` | Extract `.tar.gz` |
| `tar -xjvf file.tar.bz2` | Extract `.tar.bz2` |
| `tar -xJvf file.tar.xz` | Extract `.tar.xz` |
| `tar -xf archive.tar.gz` | Extract with auto-detection |
| `tar -xf archive.tar.gz -C restore/` | Extract into specific directory |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-07/
        ├── project-tree.txt
        ├── archive-size-comparison.txt
        ├── tar-gz-list.txt
        ├── tar-bz2-list.txt
        ├── tar-xz-list.txt
        ├── restore-verification.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- `tree project`
- `ls -lh project.tar.*`
- Archive content listings
- Restore directory trees
- Notes comparing `.tar.gz`, `.tar.bz2`, and `.tar.xz`

---

# Next Lesson

**Lesson 08 — Working with `zip` and `unzip`**

You will learn when to use `zip` instead of `tar`, especially for cross-platform sharing between Linux, Windows, and macOS.

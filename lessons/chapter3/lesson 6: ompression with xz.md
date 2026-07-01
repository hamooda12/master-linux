# Chapter 03 — Archives, Compression & Backups

# Lesson 06 — Compression with xz

---

# Overview

In the previous lessons, you learned three important ideas:

- `tar` creates archives.
- `gzip` compresses files quickly.
- `bzip2` usually compresses better than `gzip`, but slower.

Now you will learn `xz`.

`xz` is a modern compression tool that usually creates very small compressed files. It is commonly used for software packages, source code archives, and long-term storage.

The trade-off is that `xz` can be slower and may use more CPU and memory than `gzip`.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what `xz` does.
- Compress files using `xz`.
- Decompress `.xz` files.
- Keep the original file during compression.
- Test `.xz` files for integrity.
- Compare `xz` with `gzip` and `bzip2`.
- Understand when to use `xz`.

---

# Prerequisites

Before this lesson, you should understand:

- Archives vs compression
- Basic `tar` usage
- `gzip`
- `bzip2`
- File size comparison with `ls -lh`

---

# What Is xz?

`xz` is a file compression tool.

It compresses a single file and creates a file ending with:

```text
.xz
```

Example:

```text
backup.log
```

becomes:

```text
backup.log.xz
```

Like `gzip` and `bzip2`, `xz` does **not** archive directories directly.

To compress a directory, archive it first with `tar`.

---

# Basic Syntax

```bash
xz filename
```

Example:

```bash
xz backup.log
```

Result:

```text
backup.log.xz
```

By default, `xz` removes the original file after compression.

---

# Keeping the Original File

Use:

```bash
xz -k backup.log
```

Now both files remain:

```text
backup.log
backup.log.xz
```

The `-k` option means:

```text
keep original file
```

---

# Decompressing Files

Use:

```bash
unxz backup.log.xz
```

Result:

```text
backup.log
```

By default, `unxz` removes the compressed `.xz` file after decompression.

---

# Decompressing While Keeping the Compressed File

```bash
unxz -k backup.log.xz
```

Now both files remain:

```text
backup.log
backup.log.xz
```

---

# Using xz -d

You can also decompress with:

```bash
xz -d backup.log.xz
```

This does the same job as:

```bash
unxz backup.log.xz
```

---

# Testing an xz File

To verify that an `.xz` file is valid:

```bash
xz -t backup.log.xz
```

If there is no output, the file is valid.

Confirm using:

```bash
echo $?
```

Expected output:

```text
0
```

---

# Viewing Compression Details

Use:

```bash
xz -l backup.log.xz
```

Example output:

```text
Strms  Blocks   Compressed Uncompressed  Ratio  Check   Filename
    1       1      8.0 KiB      64.0 KiB  0.125  CRC64   backup.log.xz
```

This shows:

- Compressed size
- Uncompressed size
- Compression ratio
- Integrity check type
- Filename

---

# Compression Levels

`xz` supports compression levels from `-0` to `-9`.

| Level | Meaning |
|---|---|
| `-0` | Fastest, lowest compression |
| `-6` | Default balance |
| `-9` | Strong compression, slower and more memory usage |

Example:

```bash
xz -9 backup.log
```

For normal use, the default is usually enough.

---

# Comparing gzip, bzip2, and xz

General rule:

| Tool | Speed | Compression Ratio | Common Use |
|---|---|---|---|
| `gzip` | Fast | Good | Logs, quick backups |
| `bzip2` | Medium/Slow | Better | Long-term archives |
| `xz` | Slowest | Often best | Software packages, long-term storage |

---

# Visual Comparison

```text
Speed

gzip
████████████████████

bzip2
████████████

xz
██████
```

```text
Compression Strength

gzip
████████████

bzip2
███████████████

xz
███████████████████
```

---

# Why xz Is Popular

`xz` is popular because it can produce very small files.

It is commonly used for:

- Linux package archives
- Source code releases
- Distribution images
- Long-term compressed backups

Examples you may see:

```text
package.tar.xz
source-code.tar.xz
linux-kernel.tar.xz
```

---

# Practical Scenario

You have a large archived source code backup:

```text
project.tar
```

You do not need to restore it often.

You want to save as much storage as possible.

You compress it using:

```bash
xz project.tar
```

Result:

```text
project.tar.xz
```

This may take longer than `gzip`, but the final file may be smaller.

---

# Hands-on Lab

## Step 1 — Enter the Chapter Directory

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2 — Create an xz Lab Directory

```bash
mkdir -p xz-lab
cd xz-lab
```

---

## Step 3 — Create a Test File

```bash
for i in {1..3000}; do
    echo "Linux backup compression practice using xz." >> archive-data.txt
done
```

---

## Step 4 — Check Original Size

```bash
ls -lh archive-data.txt
```

---

## Step 5 — Compress the File

```bash
xz archive-data.txt
```

---

## Step 6 — Verify the Result

```bash
ls -lh
```

Expected file:

```text
archive-data.txt.xz
```

---

## Step 7 — View Compression Details

```bash
xz -l archive-data.txt.xz
```

---

## Step 8 — Test the Compressed File

```bash
xz -t archive-data.txt.xz
echo $?
```

Expected output:

```text
0
```

---

## Step 9 — Decompress the File

```bash
unxz archive-data.txt.xz
```

---

## Step 10 — Compress Again While Keeping the Original

```bash
xz -k archive-data.txt
```

Verify:

```bash
ls -lh
```

---

# Challenge

Create a file named:

```text
monthly-report.log
```

Fill it with repeated text:

```bash
for i in {1..2000}; do
    echo "Monthly system report entry number $i" >> monthly-report.log
done
```

Then:

1. Compress it with `xz -k`.
2. Verify both files exist.
3. Test the `.xz` file.
4. Show compression details with `xz -l`.
5. Decompress it using `unxz -k`.

Bonus:

Compress the same file using:

```bash
gzip -k monthly-report.log
bzip2 -k monthly-report.log
xz -k monthly-report.log
```

Then compare:

```bash
ls -lh monthly-report.log*
```

---

# Common Mistakes

## Mistake 1 — Thinking xz Archives Directories

`xz` compresses one file.

It does not package directories.

Correct workflow:

```text
directory/
   ↓
directory.tar
   ↓
directory.tar.xz
```

---

## Mistake 2 — Using `-9` for Everything

Maximum compression is not always best.

It can be slower and use more memory.

Default compression is usually enough.

---

## Mistake 3 — Forgetting the Original Is Removed

This command:

```bash
xz file.txt
```

replaces the original file with:

```text
file.txt.xz
```

Use:

```bash
xz -k file.txt
```

to keep the original.

---

# Best Practices

- Use `xz` when storage space matters.
- Use `gzip` when speed matters.
- Use default compression unless you have a reason to change it.
- Test important files with `xz -t`.
- Keep original files during practice using `-k`.

---

# Interview Questions

1. What does `xz` do?

2. What extension does `xz` create?

3. Does `xz` archive directories?

4. How do you keep the original file?

5. How do you decompress a `.xz` file?

6. How do you test a compressed `.xz` file?

7. Why might `xz` be slower than `gzip`?

8. When would you choose `xz` over `gzip`?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I compress a file using `xz`?
- Can I decompress a `.xz` file?
- Can I test file integrity?
- Can I explain the trade-off between speed and compression ratio?
- Do I understand why `tar` is needed before compressing directories?

---

# Summary

In this lesson, you learned how to use `xz` for file compression.

You practiced compressing, decompressing, keeping original files, checking compression details, and testing integrity.

`xz` often provides excellent compression, making it useful for software archives and long-term storage, but it can be slower than `gzip`.

---

# Cheat Sheet

| Command | Description |
|---|---|
| `xz file.txt` | Compress file and remove original |
| `xz -k file.txt` | Compress and keep original |
| `unxz file.txt.xz` | Decompress file |
| `unxz -k file.txt.xz` | Decompress and keep `.xz` |
| `xz -d file.txt.xz` | Decompress using `xz` |
| `xz -t file.txt.xz` | Test compressed file |
| `xz -l file.txt.xz` | Show compression information |
| `xz -0 file.txt` | Fastest compression |
| `xz -9 file.txt` | Strongest compression |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-06/
        ├── original-size.txt
        ├── compressed-size.txt
        ├── xz-list-output.txt
        ├── xz-test-output.txt
        ├── compression-comparison.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- `ls -lh` before compression
- `ls -lh` after compression
- `xz -l` output
- `xz -t` result
- Optional comparison between `gzip`, `bzip2`, and `xz`
- Screenshot of the lab directory

---

# Next Lesson

**Lesson 07 — Working with `.tar.gz`, `.tar.bz2`, and `.tar.xz`**

You will learn how to combine archiving and compression into real backup formats used in Linux administration.

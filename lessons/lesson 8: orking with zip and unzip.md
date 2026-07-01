# Chapter 03 — Archives, Compression & Backups

# Lesson 08 — Working with `zip` and `unzip`

---

# Overview

So far, you learned the Linux-style backup workflow:

```text
tar + compression
```

But there is another very common format:

```text
.zip
```

`zip` is popular because it works easily across:

- Linux
- Windows
- macOS

Unlike `gzip`, `bzip2`, and `xz`, the `zip` format can both:

1. Archive multiple files
2. Compress them

That makes it useful for sharing files with non-Linux users.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Create `.zip` files.
- Extract `.zip` files.
- Zip files and directories.
- List zip contents without extracting.
- Understand when to use `zip` instead of `tar`.
- Use `zip` for cross-platform file sharing.

---

# Prerequisites

Before this lesson, you should understand:

- Archives vs compression
- Basic file and directory operations
- `tar`
- Compressed tar archives

---

# What Is zip?

`zip` is both an archive and compression format.

This means it can take many files:

```text
project/
├── app.py
├── config.conf
└── README.md
```

and create one compressed file:

```text
project.zip
```

Unlike `.tar.gz`, the archive and compression are handled together.

---

# Installing zip and unzip

On some Linux systems, `zip` and `unzip` may not be installed by default.

Install them with:

```bash
sudo apt update
sudo apt install zip unzip -y
```

Check versions:

```bash
zip -v
unzip -v
```

---

# Creating a Zip File from Files

```bash
zip documents.zip file1.txt file2.txt
```

Result:

```text
documents.zip
```

---

# Creating a Zip File from a Directory

To zip a directory recursively, use:

```bash
zip -r project.zip project/
```

The `-r` option means:

```text
recursive
```

Without `-r`, directories are not fully included.

---

# Listing Zip Contents

Before extracting, inspect the archive:

```bash
unzip -l project.zip
```

Example output:

```text
Archive:  project.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2026-07-01 12:00   project/
       24  2026-07-01 12:00   project/app.py
       12  2026-07-01 12:00   project/config.conf
---------                     -------
       36                     3 files
```

---

# Extracting a Zip File

```bash
unzip project.zip
```

This extracts the archive into the current directory.

---

# Extracting to Another Directory

Use:

```bash
unzip project.zip -d restore/
```

The `-d` option means:

```text
destination directory
```

---

# Overwrite Behavior

If files already exist, `unzip` may ask:

```text
replace project/app.py? [y]es, [n]o, [A]ll, [N]one
```

Useful options:

| Option | Meaning |
|---|---|
| `y` | Replace this file |
| `n` | Do not replace this file |
| `A` | Replace all |
| `N` | Replace none |

For safe practice, extract into a separate restore directory.

---

# Testing a Zip File

To verify zip archive integrity:

```bash
unzip -t project.zip
```

Example successful output:

```text
No errors detected in compressed data of project.zip.
```

---

# zip vs tar

| Feature | `zip` | `tar` |
|---|---|---|
| Archives multiple files | Yes | Yes |
| Compresses by default | Yes | No |
| Common on Windows | Yes | Less common |
| Preserves Linux metadata well | Less complete | Better |
| Best for Linux backups | Sometimes | Usually |
| Best for sharing with Windows users | Yes | Sometimes |

---

# When to Use zip

Use `zip` when:

- Sharing files with Windows users
- Sending files by email
- Creating simple compressed packages
- Cross-platform compatibility matters

---

# When to Use tar

Use `tar` when:

- Backing up Linux systems
- Preserving permissions matters
- Preserving ownership matters
- Working with Linux servers
- Creating deployment archives

---

# Practical Scenario

You finished a small Linux practice assignment and want to send it to someone using Windows.

Instead of creating:

```text
assignment.tar.gz
```

you create:

```bash
zip -r assignment.zip assignment/
```

This makes it easier for the receiver to open the file.

---

# Hands-on Lab

## Step 1 — Move to Chapter Directory

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2 — Create Lab Directory

```bash
mkdir -p zip-lab
cd zip-lab
```

---

## Step 3 — Create Practice Files

```bash
mkdir -p project/docs project/src

echo "Linux zip practice" > project/README.md
echo "print('Hello zip')" > project/src/app.py
echo "Documentation file" > project/docs/notes.txt
```

---

## Step 4 — View Structure

```bash
tree project
```

---

## Step 5 — Create a Zip Archive

```bash
zip -r project.zip project/
```

---

## Step 6 — Verify the Zip File

```bash
ls -lh project.zip
```

---

## Step 7 — List Contents

```bash
unzip -l project.zip
```

---

## Step 8 — Test the Archive

```bash
unzip -t project.zip
```

---

## Step 9 — Extract to Restore Directory

```bash
mkdir restore
unzip project.zip -d restore/
```

---

## Step 10 — Verify Restored Files

```bash
tree restore
```

---

# Challenge

Create a directory named:

```text
portfolio/
```

Inside it, create:

```text
README.md
projects.txt
contact.txt
```

Then:

1. Create `portfolio.zip`.
2. List its contents.
3. Test the zip file.
4. Extract it into `portfolio-restore/`.
5. Verify the restored files using `tree`.

---

# Common Mistakes

## Mistake 1 — Forgetting `-r`

Wrong:

```bash
zip project.zip project/
```

Correct:

```bash
zip -r project.zip project/
```

Without `-r`, the directory contents may not be fully archived.

---

## Mistake 2 — Extracting in the Wrong Place

Always check:

```bash
pwd
```

before extracting.

---

## Mistake 3 — Using zip for Full Linux System Backups

`zip` is not usually the best option for full Linux backups because it may not preserve Linux permissions and ownership as well as `tar`.

---

# Best Practices

- Use `zip` for cross-platform sharing.
- Use `tar` for Linux-native backups.
- Always test important zip archives.
- Extract into a restore directory first.
- Use meaningful names.

Example:

```text
portfolio-2026-07-01.zip
```

---

# Interview Questions

1. What is the difference between `zip` and `gzip`?

2. What does `zip -r` do?

3. How do you extract a `.zip` file?

4. How do you list zip contents without extracting?

5. How do you test a zip archive?

6. When is `zip` better than `tar`?

7. Why is `tar` usually better for Linux system backups?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I create a zip archive?
- Can I zip a full directory?
- Can I extract a zip archive safely?
- Can I test a zip file?
- Do I know when to choose `zip` instead of `tar`?

---

# Summary

In this lesson, you learned how to use `zip` and `unzip`.

You practiced creating zip archives, listing contents, testing integrity, and extracting files safely.

`zip` is especially useful when sharing files across Linux, Windows, and macOS, while `tar` remains the stronger choice for Linux-native backups.

---

# Cheat Sheet

| Command | Description |
|---|---|
| `zip files.zip file1 file2` | Create zip from files |
| `zip -r project.zip project/` | Zip a directory recursively |
| `unzip project.zip` | Extract zip in current directory |
| `unzip project.zip -d restore/` | Extract zip into destination |
| `unzip -l project.zip` | List zip contents |
| `unzip -t project.zip` | Test zip integrity |
| `ls -lh project.zip` | View zip file size |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-08/
        ├── project-tree.txt
        ├── zip-create-output.txt
        ├── zip-list-output.txt
        ├── zip-test-output.txt
        ├── restore-tree.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- `tree project`
- `zip -r` output
- `unzip -l` output
- `unzip -t` output
- `tree restore`
- Notes explaining when to use `zip` vs `tar`

---

# Next Lesson

**Lesson 09 — Comparing Compression Methods**

You will compare `gzip`, `bzip2`, `xz`, and `zip` using the same data and learn how to choose the right tool for real-world situations.

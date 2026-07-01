# Chapter 03 — Archives, Compression & Backups

# Lesson 14 — Building a Backup Utility (Version 1.0)

---

# Overview

Throughout this chapter, you have learned:

- Creating archives with `tar`
- Compressing files
- Backup strategies
- Verification
- Recovery testing

Now it's time to combine these skills into a practical project.

In this lesson, you will build Version 1.0 of a Linux backup utility.

Unlike previous lessons, this project focuses on writing a reusable Bash script rather than typing individual commands manually.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Write a reusable Bash backup script.
- Archive and compress directories.
- Create timestamped backup filenames.
- Organize backup directories.
- Display useful progress messages.
- Verify that a backup was created.

---

# Prerequisites

You should already understand:

- Bash basics
- Variables
- Command substitution
- `tar`
- Compression
- File permissions

---

# Project Goal

Build a utility that:

- Accepts a source directory.
- Stores backups in a backup directory.
- Uses timestamped filenames.
- Compresses using `tar.gz`.
- Reports success or failure.

---

# Final Directory Layout

```text
backup-project/
├── backup.sh
├── backups/
├── source/
│   ├── app.py
│   ├── config.ini
│   └── README.md
└── logs/
```

---

# Step 1 — Create the Project

```bash
mkdir -p ~/linux-practice/chapter-03/backup-project
cd ~/linux-practice/chapter-03/backup-project

mkdir backups
mkdir logs
mkdir source
```

---

# Step 2 — Create Sample Files

```bash
echo "print('Hello Linux')" > source/app.py
echo "ENV=development" > source/config.ini
echo "# Backup Demo" > source/README.md
```

Verify:

```bash
tree
```

---

# Step 3 — Create the Script

```bash
touch backup.sh
chmod +x backup.sh
```

---

# Step 4 — Script Header

```bash
#!/bin/bash
```

---

# Step 5 — Variables

```bash
SOURCE="./source"
DESTINATION="./backups"

DATE=$(date +"%Y-%m-%d_%H-%M-%S")

BACKUP_NAME="backup-$DATE.tar.gz"
```

Using variables makes the script easier to maintain.

---

# Step 6 — Create the Backup

```bash
tar -czf "$DESTINATION/$BACKUP_NAME" "$SOURCE"
```

---

# Step 7 — Verify Success

Immediately after `tar` finishes:

```bash
if [ $? -eq 0 ]; then
    echo "Backup completed successfully."
else
    echo "Backup failed."
fi
```

`$?` contains the exit status of the previous command.

- `0` means success.
- Non-zero indicates an error.

---

# Step 8 — Show Backup Information

Display useful information:

```bash
echo
echo "Backup Name : $BACKUP_NAME"
echo "Source      : $SOURCE"
echo "Destination : $DESTINATION"
```

---

# Complete Script (Version 1.0)

```bash
#!/bin/bash

SOURCE="./source"
DESTINATION="./backups"

DATE=$(date +"%Y-%m-%d_%H-%M-%S")

BACKUP_NAME="backup-$DATE.tar.gz"

echo "Creating backup..."

tar -czf "$DESTINATION/$BACKUP_NAME" "$SOURCE"

if [ $? -eq 0 ]; then
    echo
    echo "Backup completed successfully."
    echo
    echo "Backup Name : $BACKUP_NAME"
    echo "Source      : $SOURCE"
    echo "Destination : $DESTINATION"
else
    echo "Backup failed."
fi
```

---

# Running the Script

```bash
./backup.sh
```

Example output:

```text
Creating backup...

Backup completed successfully.

Backup Name : backup-2026-07-01_20-15-41.tar.gz
Source      : ./source
Destination : ./backups
```

---

# Verify the Backup

List the backup directory:

```bash
ls -lh backups
```

Inspect the archive:

```bash
tar -tzf backups/backup-*.tar.gz
```

---

# Timestamped Filenames

Why use timestamps?

Bad:

```text
backup.tar.gz
```

Every execution overwrites the previous backup.

Better:

```text
backup-2026-07-01_20-15-41.tar.gz
backup-2026-07-02_20-15-38.tar.gz
backup-2026-07-03_20-15-44.tar.gz
```

Each backup is preserved.

---

# Practical Scenario

A small company wants to back up its application directory every evening.

Instead of manually typing:

```bash
tar -czf ...
```

the administrator runs:

```bash
./backup.sh
```

or schedules it with a task scheduler in a later chapter.

---

# Hands-on Lab

## Step 1

Create the project exactly as shown.

---

## Step 2

Write `backup.sh`.

---

## Step 3

Run the script.

---

## Step 4

Modify one file inside:

```text
source/
```

---

## Step 5

Run the script again.

---

## Step 6

Verify that two different backup files now exist.

---

## Step 7

Extract one backup into:

```text
restore/
```

Verify the files.

---

# Challenge

Improve Version 1.0 by adding:

- A welcome message.
- The current date.
- The backup file size using:

```bash
du -h
```

Example:

```text
Backup Size:

24K
```

---

# Common Mistakes

## Mistake 1

Forgetting:

```bash
chmod +x backup.sh
```

---

## Mistake 2

Using hard-coded filenames.

Always generate unique backup names.

---

## Mistake 3

Not checking whether the backup succeeded.

Always inspect the exit status.

---

# Best Practices

- Use variables instead of repeating paths.
- Use timestamped filenames.
- Keep backups in a dedicated directory.
- Verify each backup after creation.
- Keep scripts readable and well commented.

---

# Interview Questions

1. Why use variables in a Bash script?

2. Why include timestamps in backup filenames?

3. What does `$?` represent?

4. Why should scripts check command exit codes?

5. How do you make a Bash script executable?

6. Why is automation better than manual backups?

---

# Lesson Reflection

Before moving on, ask yourself:

- Can I write a simple backup script?
- Can I create timestamped backups?
- Can I check whether a command succeeded?
- Can I explain how the script works?

---

# Summary

In this lesson, you built Version 1.0 of a reusable Linux backup utility.

The script archives and compresses a source directory, creates unique timestamped backup files, and reports whether the backup succeeded.

This is a solid foundation that you'll improve into a production-style tool in the next lesson.

---

# Cheat Sheet

| Command | Purpose |
|----------|---------|
| `chmod +x backup.sh` | Make the script executable |
| `./backup.sh` | Run the script |
| `date +"%Y-%m-%d_%H-%M-%S"` | Generate a timestamp |
| `tar -czf archive.tar.gz source/` | Create a compressed archive |
| `$?` | Previous command's exit status |
| `ls -lh backups` | List created backups |
| `tar -tzf backup.tar.gz` | Verify archive contents |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-14/
        ├── backup-script-v1.sh
        ├── backup-output.txt
        ├── backup-tree.txt
        ├── restore-results.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- The completed `backup.sh`
- Terminal output from running the script
- Directory tree before and after backups
- Restore verification
- Notes about possible improvements

---

# Next Lesson

**Lesson 15 — Refactoring the Backup Utility into a Professional Tool**

You'll enhance Version 1.0 by adding:

- Command-line arguments
- Configurable source and destination paths
- SHA-256 checksum generation
- Logging
- Error handling
- Automatic cleanup of old backups
- Colored terminal output
- Proper exit codes
- Professional project structure

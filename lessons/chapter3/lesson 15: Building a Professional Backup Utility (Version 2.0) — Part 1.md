# Chapter 03 — Archives, Compression & Backups

# Lesson 15 — Building a Professional Backup Utility (Version 2.0) — Part 1

---

# Overview

In Lesson 14, you created a working backup script.

It could:

- Create a compressed backup
- Use timestamps
- Report success

That script worked, but it was not yet suitable for production.

Professional tools are designed differently.

Instead of placing everything inside one file, they separate:

- Configuration
- Logs
- Backups
- Source data
- Documentation

In this lesson, you'll redesign your backup script into a professional utility.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Organize a Linux project professionally.
- Separate configuration from code.
- Build a reusable Bash application.
- Create a maintainable project layout.
- Prepare for adding advanced backup features.

---

# Prerequisites

You should understand:

- Bash scripting
- Variables
- tar
- gzip
- File permissions
- Exit codes

---

# Why Refactor?

Suppose your first script contains:

```bash
SOURCE="./source"
DESTINATION="./backups"
```

A month later, your manager says:

> "Store backups on another disk."

Now you must edit the script.

Later they say:

> "Keep backups for 30 days."

Again, you edit the script.

Eventually your script becomes difficult to maintain.

Professional software separates configuration from logic.

---

# Project Architecture

Version 1:

```text
backup-project/
├── backup.sh
├── backups/
└── source/
```

Version 2:

```text
backup-project/
├── backup.sh
├── README.md
├── LICENSE
├── config/
│   └── backup.conf
├── backups/
├── logs/
├── restore-test/
├── source/
└── assets/
```

Each directory has a specific responsibility.

---

# Project Directory Responsibilities

## backup.sh

The main application.

---

## config/

Stores configuration files.

---

## backups/

Stores generated backup archives.

---

## logs/

Stores execution logs.

---

## restore-test/

Temporary directory used to verify backups.

---

## source/

Sample data that will be backed up.

---

## assets/

Screenshots and documentation for your GitHub repository.

---

# Step 1 — Create the Project

```bash
mkdir -p ~/linux-practice/chapter-03/backup-project
cd ~/linux-practice/chapter-03/backup-project
```

Create the directories:

```bash
mkdir backups
mkdir config
mkdir logs
mkdir restore-test
mkdir source
mkdir assets
```

Create the project files:

```bash
touch backup.sh
touch README.md
touch LICENSE
touch config/backup.conf
touch logs/backup.log
```

---

# Step 2 — Verify the Structure

Run:

```bash
tree
```

Expected output:

```text
backup-project
├── assets
├── backup.sh
├── backups
├── config
│   └── backup.conf
├── LICENSE
├── logs
│   └── backup.log
├── README.md
├── restore-test
└── source
```

---

# Step 3 — Create Sample Source Files

```bash
echo "print('Linux Backup Utility')" > source/app.py

echo "APP_ENV=development" > source/app.conf

echo "# Linux Backup Utility" > source/README.md
```

Verify:

```bash
tree source
```

---

# Step 4 — Create the Configuration File

Open:

```bash
nano config/backup.conf
```

Add:

```bash
# Source Directory
SOURCE="./source"

# Backup Destination
DESTINATION="./backups"

# Log File
LOG_FILE="./logs/backup.log"

# Restore Test Directory
RESTORE_DIR="./restore-test"

# Backup Retention
RETENTION_DAYS=7

# Compression Method
COMPRESSION="gzip"
```

Save the file.

---

# Why Use a Configuration File?

Instead of writing:

```bash
SOURCE="./source"
```

inside every script,

we load everything from one place.

Benefits:

- Easier maintenance
- Cleaner code
- Easier customization
- No code modifications for configuration changes

---

# Step 5 — Create the Script Header

Edit:

```bash
nano backup.sh
```

Start with:

```bash
#!/bin/bash
```

---

# Step 6 — Load the Configuration

Immediately after the header:

```bash
source ./config/backup.conf
```

Now every variable inside:

```text
backup.conf
```

is automatically available.

Example:

```bash
echo "$SOURCE"
```

Output:

```text
./source
```

---

# Step 7 — Verify Required Directories

A professional application checks its environment before running.

Add:

```bash
mkdir -p "$DESTINATION"
mkdir -p "$RESTORE_DIR"
mkdir -p logs
```

This ensures required directories exist.

---

# Step 8 — Generate a Timestamp

```bash
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
```

Example:

```text
2026-07-01_20-41-58
```

---

# Step 9 — Create the Backup Filename

```bash
BACKUP_NAME="backup-$DATE.tar.gz"
```

Full path:

```bash
BACKUP_PATH="$DESTINATION/$BACKUP_NAME"
```

---

# Step 10 — Display Startup Information

```bash
echo
echo "======================================"
echo " Linux Backup Utility v2.0"
echo "======================================"

echo
echo "Source      : $SOURCE"
echo "Destination : $DESTINATION"
echo "Compression : $COMPRESSION"
echo
```

Example:

```text
======================================
 Linux Backup Utility v2.0
======================================

Source      : ./source
Destination : ./backups
Compression : gzip
```

---

# Step 11 — Create the Backup

```bash
echo "Creating archive..."

tar -czf "$BACKUP_PATH" "$SOURCE"
```

---

# Step 12 — Verify Success

```bash
if [ $? -ne 0 ]; then
    echo
    echo "Backup failed."
    exit 1
fi
```

---

# Step 13 — Success Message

```bash
echo
echo "Archive created successfully."

echo
echo "Archive:"
echo "$BACKUP_PATH"
```

---

# Complete Script (Version 2.0 — Part 1)

```bash
#!/bin/bash

source ./config/backup.conf

mkdir -p "$DESTINATION"
mkdir -p "$RESTORE_DIR"
mkdir -p logs

DATE=$(date +"%Y-%m-%d_%H-%M-%S")

BACKUP_NAME="backup-$DATE.tar.gz"

BACKUP_PATH="$DESTINATION/$BACKUP_NAME"

echo
echo "======================================"
echo " Linux Backup Utility v2.0"
echo "======================================"

echo
echo "Source      : $SOURCE"
echo "Destination : $DESTINATION"
echo "Compression : $COMPRESSION"

echo
echo "Creating archive..."

tar -czf "$BACKUP_PATH" "$SOURCE"

if [ $? -ne 0 ]; then
    echo
    echo "Backup failed."
    exit 1
fi

echo
echo "Archive created successfully."

echo
echo "Archive:"
echo "$BACKUP_PATH"
```

---

# Hands-on Lab

1. Create the complete project structure.
2. Create `backup.conf`.
3. Write the script.
4. Make it executable:

```bash
chmod +x backup.sh
```

5. Run:

```bash
./backup.sh
```

6. Verify:

```bash
ls -lh backups
```

---

# Challenge

Modify:

```text
backup.conf
```

Change:

```text
RETENTION_DAYS
```

from:

```text
7
```

to:

```text
30
```

Run the script again.

Notice that **no code inside `backup.sh` needed to change**.

This demonstrates why configuration files are useful.

---

# Common Mistakes

- Forgetting to load the configuration with `source`.
- Hardcoding values instead of using variables.
- Forgetting `chmod +x backup.sh`.
- Assuming required directories already exist.
- Ignoring command exit codes.

---

# Best Practices

- Keep configuration separate from application logic.
- Use descriptive variable names.
- Create required directories automatically.
- Never hardcode paths that may change.
- Fail early if a critical operation does not succeed.

---

# Interview Questions

1. Why should configuration be separated from code?
2. What does the `source` command do?
3. Why use `mkdir -p` instead of `mkdir`?
4. Why are timestamped filenames important?
5. Why should scripts check exit codes?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I organize a Bash project professionally?
- Can I load configuration from an external file?
- Can I explain the advantages of separating configuration from code?
- Can I generate unique backup filenames?

---

# Summary

In this lesson, you redesigned your backup project into a more professional structure.

You separated configuration from code, organized the project into dedicated directories, and created the foundation for a reusable backup utility.

In Part 2, you'll add logging, checksum generation, verification, restore testing, runtime measurement, and more advanced features.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `source config/backup.conf` | Load configuration variables |
| `mkdir -p directory` | Create directory if it doesn't exist |
| `date +"%Y-%m-%d_%H-%M-%S"` | Generate timestamp |
| `tar -czf archive.tar.gz source/` | Create compressed archive |
| `chmod +x backup.sh` | Make script executable |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-15-part1/
        ├── backup-project-tree.txt
        ├── backup.conf
        ├── backup.sh
        ├── terminal-output.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- Project directory tree
- Final `backup.conf`
- Final `backup.sh`
- Terminal output from a successful run
- Screenshot of the project structure

---

# Next Lesson

**Lesson 15 — Part 2: Professional Features**

You'll add:

- Logging
- SHA-256 checksum generation
- Backup verification
- Restore testing
- Runtime measurement
- Better status reporting

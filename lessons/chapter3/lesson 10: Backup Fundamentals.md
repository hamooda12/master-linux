# Chapter 03 — Archives, Compression & Backups

# Lesson 10 — Backup Fundamentals

---

# Overview

Creating a backup is easy.

Creating a backup that can actually save your business after a disaster is much harder.

Many organizations believe they have backups until they experience:

- A disk failure
- Accidental file deletion
- Ransomware
- Hardware damage
- Human error

Only then do they discover their backups are incomplete, corrupted, or impossible to restore.

This lesson introduces the principles behind professional backup systems before learning specific backup strategies.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what a backup really is.
- Understand why backups are critical.
- Identify common causes of data loss.
- Learn backup terminology.
- Understand backup objectives.
- Recognize characteristics of a good backup.

---

# Prerequisites

You should already know:

- Files and directories
- tar
- Compression
- Linux filesystem basics

---

# What Is a Backup?

A backup is:

> A separate copy of important data that can be used to restore information after data loss.

Notice two important words:

> Separate copy

If your only copy of a file is here:

```text
/home/hamad/project
```

that is **not** a backup.

Even if you copy it to:

```text
/home/hamad/project-copy
```

on the same hard drive...

...that is still not a proper backup.

---

# Why Are Backups Necessary?

Data can disappear for many reasons.

## Hardware Failure

Hard drives eventually fail.

SSD:

```text
██████████████

↓

Failure
```

HDD:

```text
██████████████

↓

Failure
```

No storage device lasts forever.

---

## Human Error

Examples:

```bash
rm -rf project/
```

or

```bash
mv important-folder /tmp
```

Mistakes happen.

Backups exist because humans make mistakes.

---

## Software Bugs

An application update may:

- Delete files
- Corrupt databases
- Damage configuration files

A recent backup allows recovery.

---

## Malware & Ransomware

Some malware encrypts or destroys files.

Without backups:

```text
Data Lost
```

With backups:

```text
Restore

↓

Business Continues
```

---

## Natural Disasters

Examples:

- Fire
- Flood
- Power surge
- Theft

These events can destroy entire servers.

---

# What Should Be Backed Up?

Not everything needs a backup.

Common examples include:

- Source code
- Databases
- Configuration files
- Virtual machine images
- User documents
- Application data
- SSL certificates

Examples:

```text
/etc/
/home/
/var/www/
/opt/
```

---

# What Usually Doesn't Need Backups?

Examples:

- Temporary files

```text
/tmp
```

- Package cache

```text
/var/cache
```

- Automatically generated files

These can often be recreated.

---

# Characteristics of a Good Backup

A professional backup should be:

## Complete

Everything important is included.

---

## Recoverable

You can restore it successfully.

---

## Verified

The backup has been tested.

---

## Secure

Sensitive data should be protected.

Examples:

- Encryption
- Restricted permissions

---

## Stored Separately

Never rely on a backup stored only on the same disk as the original data.

---

# Backup Terminology

## Source

The original data.

Example:

```text
/home/hamad/projects
```

---

## Destination

Where the backup is stored.

Example:

```text
/mnt/backups
```

---

## Archive

The backup file itself.

Example:

```text
backup.tar.gz
```

---

## Restore

Recovering data from a backup.

---

## Recovery

Returning a system to a working state.

---

# Backup Workflow

A typical backup process looks like this:

```text
Original Data
       │
       ▼
Create Archive
       │
       ▼
Compress
       │
       ▼
Verify
       │
       ▼
Store Backup
       │
       ▼
Test Restore
```

Skipping any step increases the risk of backup failure.

---

# Backup Objectives

Professional backups aim to:

- Minimize data loss.
- Reduce downtime.
- Restore services quickly.
- Protect business continuity.

A backup is successful only if it can be restored when needed.

---

# Common Backup Mistakes

## Mistake 1

Keeping backups on the same disk.

If the disk fails:

```text
Original Data

+

Backup

↓

Lost Together
```

---

## Mistake 2

Never testing restores.

Many organizations discover corrupted backups only during an emergency.

---

## Mistake 3

Backing up unnecessary files.

This wastes:

- Storage
- Time
- Bandwidth

---

## Mistake 4

Using inconsistent file names.

Bad:

```text
backup1.tar.gz
backup2.tar.gz
```

Better:

```text
backup-2026-07-01.tar.gz
backup-2026-07-02.tar.gz
```

---

# Practical Scenario

You are responsible for a web server.

Important data includes:

```text
/var/www/
/etc/nginx/
/etc/ssl/
/home/deploy
```

Temporary data:

```text
/tmp
```

Package cache:

```text
/var/cache
```

A good administrator backs up the important data but excludes temporary files.

---

# Hands-on Lab

## Step 1

Create a practice directory.

```bash
mkdir -p ~/linux-practice/chapter-03/backup-fundamentals
cd ~/linux-practice/chapter-03/backup-fundamentals
```

---

## Step 2

Create sample folders.

```bash
mkdir website
mkdir database
mkdir configs
mkdir tmp
```

---

## Step 3

Decide which directories should be backed up.

Create a file:

```bash
nano backup-plan.md
```

Write:

```text
Backup:
✓ website
✓ database
✓ configs

Exclude:
✗ tmp
```

---

## Step 4

Save the file.

Review your backup plan.

---

# Challenge

Imagine you administer a Linux server.

Classify each item as:

- Backup
- Do Not Backup

```text
/etc
/tmp
/home
/var/www
/var/cache
/opt
```

Explain **why** you made each choice.

---

# Common Mistakes

- Assuming copies on the same disk are backups.
- Never testing restores.
- Backing up unnecessary data.
- Forgetting configuration files.
- Ignoring security.

---

# Best Practices

- Keep backups separate from production data.
- Use meaningful names.
- Document your backup plan.
- Verify every backup.
- Perform test restores regularly.
- Automate repetitive backup tasks.

---

# Interview Questions

1. What is a backup?

2. Why isn't a copy on the same disk considered a proper backup?

3. Name five common causes of data loss.

4. What characteristics make a backup reliable?

5. What is the difference between a backup and a restore?

6. Why should backups be verified?

7. Which Linux directories are commonly backed up?

---

# Lesson Reflection

Before moving on, ask yourself:

- Can I define a backup?
- Do I understand why backups fail?
- Can I identify important system data?
- Can I explain why backup verification is essential?

---

# Summary

In this lesson, you learned the principles behind professional backup systems.

You explored why backups are necessary, what should be protected, common causes of data loss, and the qualities of a reliable backup.

These concepts provide the foundation for designing effective backup strategies, which you'll learn in the next lesson.

---

# Cheat Sheet

| Term | Meaning |
|------|---------|
| Backup | A separate copy of data for recovery |
| Restore | Recovering data from a backup |
| Source | Original data |
| Destination | Backup storage location |
| Archive | Backup file |
| Verify | Confirm the backup is usable |
| Recovery | Returning a system to operation |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-10/
        ├── backup-plan.md
        ├── backup-classification.txt
        ├── notes.md
        ├── workflow-diagram.txt
        └── screenshots/
```

Capture:

- Your backup plan
- Directory classification exercise
- Notes explaining what should and should not be backed up
- The backup workflow diagram
- Screenshot of the practice directory

---

# Next Lesson

**Lesson 11 — Backup Strategies: Full, Incremental & Differential Backups**

In the next lesson, you'll learn the three backup strategies used in professional IT environments, how they work, and when to use each one.

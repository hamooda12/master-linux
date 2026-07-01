# Chapter 03 — Archives, Compression & Backups

# Lesson 13 — Backup Verification & Recovery Testing

---

# Overview

Creating backups is only the first step.

A backup that has never been tested cannot be trusted.

Professional administrators always verify that backups:

- Were created successfully.
- Are not corrupted.
- Can be extracted.
- Contain the expected files.
- Can restore a working system.

Verification and recovery testing are critical parts of every backup strategy.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain why backup verification is important.
- Verify archive integrity.
- Generate and verify checksums.
- Perform recovery testing.
- Understand Recovery Time Objective (RTO).
- Understand Recovery Point Objective (RPO).
- Design a backup verification workflow.

---

# Prerequisites

You should already understand:

- tar
- gzip
- bzip2
- xz
- zip
- Backup strategies
- Backup policies

---

# Why Verification Matters

Imagine this situation.

Every night, your backup script runs successfully.

The logs say:

```text
Backup Completed Successfully
```

Three months later:

The production server fails.

You try to restore the backup.

Instead you receive:

```text
Archive is corrupted.
```

At that moment, the backup becomes worthless.

Verification exists to prevent this scenario.

---

# The Golden Rule

Never assume a backup works.

Always verify it.

Professional workflow:

```text
Create Backup

↓

Verify Backup

↓

Store Backup

↓

Test Restore

↓

Production Ready
```

---

# What Should Be Verified?

Every backup should answer these questions:

✓ Was the archive created?

✓ Is the archive readable?

✓ Is it corrupted?

✓ Does it contain the correct files?

✓ Can it actually be restored?

---

# Step 1 — Verify File Exists

After creating a backup:

```bash
ls -lh backup.tar.gz
```

Confirm:

- File exists
- Expected size
- Expected timestamp

---

# Step 2 — List Archive Contents

Instead of restoring immediately:

```bash
tar -tzf backup.tar.gz
```

Verify:

- Expected directories
- Expected files
- Correct structure

---

# Step 3 — Test Compression Integrity

Depending on the format:

## gzip

```bash
gzip -t backup.tar.gz
```

---

## bzip2

```bash
bzip2 -t backup.tar.bz2
```

---

## xz

```bash
xz -t backup.tar.xz
```

---

## zip

```bash
unzip -t backup.zip
```

These commands detect many forms of corruption without extracting the archive.

---

# Step 4 — Verify with Checksums

Checksums allow you to verify that a file has not changed.

Linux provides several tools:

```bash
md5sum
sha1sum
sha256sum
sha512sum
```

Today, SHA-256 is the recommended choice for most situations.

---

# Creating a SHA-256 Checksum

Generate a checksum:

```bash
sha256sum backup.tar.gz
```

Example:

```text
9d75d...e31d2  backup.tar.gz
```

Save it:

```bash
sha256sum backup.tar.gz > backup.tar.gz.sha256
```

Result:

```text
backup.tar.gz
backup.tar.gz.sha256
```

---

# Verifying a Checksum

Later, verify:

```bash
sha256sum -c backup.tar.gz.sha256
```

Expected output:

```text
backup.tar.gz: OK
```

If the file changed:

```text
FAILED
```

This helps detect:

- Corruption
- Incomplete transfers
- Accidental modification
- Storage problems

---

# Step 5 — Test Recovery

Never stop after verification.

Actually restore the backup.

Example:

```bash
mkdir restore-test

tar -xzf backup.tar.gz -C restore-test
```

Now compare:

```bash
tree project

tree restore-test/project
```

Everything should match.

---

# Recovery Testing

A recovery test confirms that:

- Files restore successfully.
- Applications start correctly.
- Configuration files are usable.
- Data is complete.

Large organizations perform scheduled recovery drills to ensure disaster recovery plans remain effective.

---

# Recovery Time Objective (RTO)

RTO answers:

> How quickly must the system be restored?

Example:

```text
Company Website

Maximum Downtime:

30 Minutes
```

Your backup strategy should support that recovery target.

---

# Recovery Point Objective (RPO)

RPO answers:

> How much data loss is acceptable?

Examples:

```text
Hourly Backup

↓

Maximum Data Loss

1 Hour
```

or

```text
Daily Backup

↓

Maximum Data Loss

24 Hours
```

A smaller RPO generally requires more frequent backups.

---

# Practical Example

Suppose:

```text
Database

Hourly Incremental

Daily Full
```

If the server fails at:

```text
3:15 PM
```

You restore:

- Daily Full Backup
- Hourly Incrementals

Maximum data loss depends on the timing of the most recent successful backup.

---

# Backup Verification Workflow

Professional workflow:

```text
Create Backup

↓

Verify Archive

↓

Generate SHA-256

↓

Store Backup

↓

Restore to Test Directory

↓

Compare Files

↓

Document Success
```

---

# Practical Scenario

You are responsible for nightly web server backups.

Your automated process performs:

1. Create archive.
2. Compress archive.
3. Generate SHA-256 checksum.
4. Verify checksum.
5. Restore into a temporary directory.
6. Log the result.
7. Upload the verified backup to cloud storage.

This gives confidence that backups are usable when needed.

---

# Hands-on Lab

## Step 1 — Move to the Chapter Directory

```bash
cd ~/linux-practice/chapter-03
```

---

## Step 2 — Create a Verification Lab

```bash
mkdir -p verification-lab
cd verification-lab
```

---

## Step 3 — Create Practice Files

```bash
mkdir project

echo "Linux Backup Verification" > project/readme.txt
echo "Configuration Data" > project/config.txt
```

---

## Step 4 — Create a Backup

```bash
tar -czf project.tar.gz project/
```

---

## Step 5 — Verify the Archive

```bash
tar -tzf project.tar.gz
```

---

## Step 6 — Test Compression

```bash
gzip -t project.tar.gz
```

---

## Step 7 — Generate SHA-256

```bash
sha256sum project.tar.gz > project.tar.gz.sha256
```

---

## Step 8 — Verify SHA-256

```bash
sha256sum -c project.tar.gz.sha256
```

Expected:

```text
project.tar.gz: OK
```

---

## Step 9 — Test Recovery

```bash
mkdir restore

tar -xzf project.tar.gz -C restore
```

---

## Step 10 — Compare Results

```bash
tree project

tree restore/project
```

---

# Challenge

Imagine you manage a production server.

Create a verification checklist covering:

- Archive creation
- Archive integrity
- Checksum generation
- Restore testing
- Documentation
- Log review

Save it as:

```text
backup-verification-checklist.md
```

Bonus:

Explain how RTO and RPO would influence your backup schedule for:

- A company website
- A banking database
- A personal Linux laptop

---

# Common Mistakes

## Mistake 1

Assuming backups are valid because the backup command completed.

---

## Mistake 2

Never testing restores.

---

## Mistake 3

Ignoring checksum verification after transferring backups.

---

## Mistake 4

Not documenting verification results.

---

# Best Practices

- Verify every backup.
- Generate SHA-256 checksums.
- Test restores regularly.
- Perform disaster recovery drills.
- Document verification results.
- Monitor backup logs.
- Review RTO and RPO requirements with stakeholders.

---

# Interview Questions

1. Why is backup verification important?

2. How do you verify a `.tar.gz` archive?

3. What is a checksum?

4. Why is SHA-256 commonly used?

5. What is the difference between RTO and RPO?

6. Why should backups be restored into a test directory?

7. What does `sha256sum -c` do?

8. What steps would you include in a backup verification process?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I verify a backup archive?
- Can I generate and verify SHA-256 checksums?
- Can I explain RTO and RPO?
- Can I perform a safe recovery test?
- Do I understand why backup verification is essential?

---

# Summary

In this lesson, you learned that a successful backup is not enough—it must also be verified and tested.

You practiced verifying archive integrity, generating SHA-256 checksums, restoring backups into a test directory, and understanding recovery objectives.

These practices are essential for reliable system administration and disaster recovery.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `tar -tzf backup.tar.gz` | List archive contents |
| `gzip -t backup.tar.gz` | Test gzip archive |
| `bzip2 -t backup.tar.bz2` | Test bzip2 archive |
| `xz -t backup.tar.xz` | Test xz archive |
| `unzip -t backup.zip` | Test zip archive |
| `sha256sum file` | Generate SHA-256 checksum |
| `sha256sum file > file.sha256` | Save checksum |
| `sha256sum -c file.sha256` | Verify checksum |
| `tar -xzf backup.tar.gz -C restore/` | Test restore |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-13/
        ├── verification-checklist.md
        ├── sha256-output.txt
        ├── restore-results.txt
        ├── rto-rpo-notes.md
        ├── backup-workflow.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- Archive verification output
- SHA-256 generation and verification
- Restore test results
- Comparison of original and restored directories
- RTO/RPO notes
- Screenshot of the verification workflow

---

# Next Lesson

**Lesson 14 — Automating Backups with Bash**

In this lesson, you'll build a professional backup script that:

- Creates timestamped backups
- Supports configurable source and destination paths
- Uses `tar` with compression
- Generates SHA-256 checksums
- Writes logs
- Handles common errors
- Returns proper exit codes

This will be one of the strongest portfolio projects in your Linux Practice repository.

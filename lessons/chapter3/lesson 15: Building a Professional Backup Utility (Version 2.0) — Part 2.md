# Chapter 03 — Archives, Compression & Backups

# Lesson 15 — Building a Professional Backup Utility (Version 2.0) — Part 2

---

# Overview

In Part 1, you created a professional project structure and separated configuration from code.

In this lesson, you'll enhance the backup utility by adding features commonly found in production backup tools:

- Logging
- SHA-256 checksum generation
- Backup verification
- Restore testing
- Runtime measurement
- Better terminal output

These improvements make the utility more reliable and easier to troubleshoot.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Record backup operations in log files.
- Generate SHA-256 checksums.
- Verify backup integrity.
- Test backup restoration automatically.
- Measure execution time.
- Produce professional status messages.

---

# Updated Project Structure

```text
backup-project/
├── backup.sh
├── backups/
├── config/
│   └── backup.conf
├── logs/
│   └── backup.log
├── restore-test/
├── source/
└── assets/
```

---

# Why Logging Matters

Imagine the backup runs automatically every night.

One morning someone asks:

> "Did last night's backup succeed?"

Without logs:

```text
Nobody knows.
```

With logs:

```text
2026-07-01 02:00:01
Backup Started

2026-07-01 02:00:03
Archive Created

2026-07-01 02:00:04
SHA256 Generated

2026-07-01 02:00:05
Verification Successful
```

Logs provide evidence of what happened.

---

# Step 1 — Record the Start Time

At the beginning of your script:

```bash
START_TIME=$(date +%s)
```

This stores the current Unix timestamp.

---

# Step 2 — Create a Logging Function

Instead of repeating `echo` commands, create a reusable function.

```bash
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') : $1" | tee -a "$LOG_FILE"
}
```

Now write logs like this:

```bash
log "Backup started."
```

Result:

```text
2026-07-01 20:45:03 : Backup started.
```

The `tee -a` command writes to both:

- The terminal
- The log file

---

# Step 3 — Log Each Major Step

Example:

```bash
log "Creating archive..."

tar -czf "$BACKUP_PATH" "$SOURCE"
```

If successful:

```bash
log "Archive created successfully."
```

---

# Step 4 — Generate a SHA-256 Checksum

Immediately after creating the archive:

```bash
log "Generating SHA-256 checksum..."

sha256sum "$BACKUP_PATH" > "$BACKUP_PATH.sha256"
```

This creates:

```text
backup-2026-07-01.tar.gz

backup-2026-07-01.tar.gz.sha256
```

---

# Step 5 — Verify the Checksum

Use:

```bash
log "Verifying checksum..."

sha256sum -c "$BACKUP_PATH.sha256"
```

If verification fails:

```bash
if [ $? -ne 0 ]; then
    log "Checksum verification failed."
    exit 1
fi
```

Otherwise:

```bash
log "Checksum verification passed."
```

---

# Step 6 — Test the Archive

Now verify that the archive itself can be extracted.

First, clear the restore directory:

```bash
rm -rf "$RESTORE_DIR"
mkdir -p "$RESTORE_DIR"
```

Then extract:

```bash
log "Testing restore..."

tar -xzf "$BACKUP_PATH" -C "$RESTORE_DIR"
```

If extraction fails:

```bash
if [ $? -ne 0 ]; then
    log "Restore test failed."
    exit 1
fi
```

Otherwise:

```bash
log "Restore test successful."
```

---

# Step 7 — Measure Runtime

At the end:

```bash
END_TIME=$(date +%s)

DURATION=$((END_TIME - START_TIME))
```

Display:

```bash
echo
echo "Backup Duration : ${DURATION} seconds"
```

---

# Step 8 — Display Backup Size

Use:

```bash
SIZE=$(du -h "$BACKUP_PATH" | cut -f1)
```

Display:

```bash
echo "Backup Size : $SIZE"
```

---

# Step 9 — Professional Completion Screen

Instead of a simple message:

```text
Backup completed.
```

Display:

```text
======================================

Backup Successful

Archive:
backups/backup-2026-07-01_20-45-03.tar.gz

Checksum:
backup-2026-07-01_20-45-03.tar.gz.sha256

Size:
24K

Duration:
1 second

======================================
```

This is much easier to read.

---

# Updated Script Flow

```text
Load Configuration
        │
        ▼
Create Archive
        │
        ▼
Generate SHA-256
        │
        ▼
Verify SHA-256
        │
        ▼
Restore Test
        │
        ▼
Calculate Size
        │
        ▼
Calculate Runtime
        │
        ▼
Log Success
```

---

# Practical Scenario

A company schedules backups every night.

The next morning, an administrator checks:

```text
logs/backup.log
```

They immediately see:

- When the backup started.
- Whether it succeeded.
- Whether verification passed.
- Whether the restore test succeeded.

No guesswork is required.

---

# Hands-on Lab

## Step 1

Add the logging function.

---

## Step 2

Generate SHA-256 checksums.

---

## Step 3

Verify the checksum.

---

## Step 4

Automatically restore the archive into:

```text
restore-test/
```

---

## Step 5

Measure execution time.

---

## Step 6

Display backup size.

---

## Step 7

Run the script several times.

Open:

```bash
cat logs/backup.log
```

Verify that every execution is recorded.

---

# Challenge

Improve the utility by adding:

```text
System Information

Hostname

Current User

Kernel Version
```

Hints:

```bash
hostname

whoami

uname -r
```

Include these details in the log file.

---

# Common Mistakes

## Mistake 1

Generating a checksum but never verifying it.

---

## Mistake 2

Writing logs only to the screen.

Always save them to a log file.

---

## Mistake 3

Assuming an archive is valid without testing extraction.

---

## Mistake 4

Forgetting to clear the restore-test directory before each verification.

---

# Best Practices

- Log every important operation.
- Verify every checksum.
- Test every backup by restoring it.
- Measure execution time.
- Record backup size.
- Fail immediately if verification fails.

---

# Interview Questions

1. Why should backup utilities generate checksums?

2. What does `sha256sum -c` do?

3. Why are logs important?

4. Why perform a restore test?

5. Why measure execution time?

6. What is the advantage of writing reusable functions?

7. Why is `tee` useful in Bash scripts?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I generate and verify SHA-256 checksums?
- Can I create reusable Bash functions?
- Can I automatically test backup restoration?
- Can I explain why logging is essential?

---

# Summary

In this lesson, you transformed your backup utility into a much more reliable tool.

It now logs its work, generates and verifies SHA-256 checksums, performs automatic restore testing, measures execution time, and reports useful information to the user.

These features significantly improve the trustworthiness and maintainability of the backup process.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `sha256sum file` | Generate SHA-256 checksum |
| `sha256sum -c file.sha256` | Verify checksum |
| `tee -a logfile` | Write to screen and log file |
| `date +%s` | Unix timestamp |
| `du -h file` | Human-readable file size |
| `tar -xzf archive.tar.gz -C dir` | Test restore |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-15-part2/
        ├── backup.log
        ├── checksum-output.txt
        ├── restore-test-results.txt
        ├── runtime-results.txt
        ├── backup-size.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- The updated `backup.log`
- SHA-256 generation and verification
- Restore test results
- Backup size and runtime output
- Screenshot of a successful backup run
- Notes describing each new feature

---

# Next Lesson

**Lesson 15 — Part 3: Production Features**

You'll complete the utility by adding:

- Command-line arguments
- Compression selection (`gzip`, `bzip2`, `xz`)
- Automatic cleanup of old backups
- Colored terminal output
- Improved error handling
- Help menu (`--help`)
- Professional code refactoring
- Final GitHub-ready version

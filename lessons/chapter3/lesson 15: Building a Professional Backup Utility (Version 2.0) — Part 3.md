# Chapter 03 — Archives, Compression & Backups

# Lesson 15 — Building a Professional Backup Utility (Version 2.0) — Part 3

---

# Overview

Your backup utility is now functional.

It can:

- Create backups
- Generate checksums
- Verify backups
- Test restoration
- Write logs

However, professional software is more than just functionality.

Good software is:

- Easy to use
- Easy to configure
- Easy to extend
- Easy to troubleshoot

In this lesson, you'll refactor your backup utility into a production-quality application by improving its architecture and user experience.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Refactor Bash scripts into reusable functions.
- Handle errors consistently.
- Add command-line arguments.
- Support multiple compression methods.
- Remove expired backups automatically.
- Improve terminal output.
- Build maintainable Bash applications.

---

# Why Refactor?

Imagine your script contains:

```bash
if [ $? -ne 0 ]; then
    ...
fi

...

if [ $? -ne 0 ]; then
    ...
fi

...

if [ $? -ne 0 ]; then
    ...
fi
```

After a few months the script becomes difficult to maintain.

Professional developers avoid duplicated code.

This principle is called:

> DRY — Don't Repeat Yourself

---

# Refactoring with Functions

Instead of repeating code, create reusable functions.

Example:

```bash
log() {
    echo "$(date '+%F %T') : $1" | tee -a "$LOG_FILE"
}
```

Now you simply write:

```bash
log "Backup started."
```

---

# Centralized Error Handling

Create:

```bash
check_status() {
    local EXIT_CODE=$1
    local MESSAGE=$2

    if [ "$EXIT_CODE" -ne 0 ]; then
        log "ERROR: $MESSAGE"
        exit 1
    fi
}
```

Usage:

```bash
tar -czf "$BACKUP_PATH" "$SOURCE"

check_status $? "Failed to create archive."
```

Instead of writing the same error handling repeatedly.

---

# Creating Reusable Functions

A good utility breaks work into small tasks.

Example architecture:

```text
main()

│

├── load_config()

├── create_backup()

├── generate_checksum()

├── verify_checksum()

├── test_restore()

├── cleanup_old_backups()

└── show_summary()
```

Each function has one responsibility.

---

# Command-Line Arguments

Users should not edit the script every time they want different behavior.

Example:

```bash
./backup.sh --help
```

```bash
./backup.sh --compression=xz
```

```bash
./backup.sh --source=/home/user/projects
```

```bash
./backup.sh --destination=/mnt/backups
```

This makes the utility much more flexible.

---

# Parsing Arguments

A simple approach:

```bash
for ARG in "$@"
do
    case $ARG in

        --help)
            show_help
            exit 0
            ;;

        --compression=*)
            COMPRESSION="${ARG#*=}"
            ;;

        --source=*)
            SOURCE="${ARG#*=}"
            ;;

        --destination=*)
            DESTINATION="${ARG#*=}"
            ;;

    esac
done
```

This overrides configuration values when requested.

---

# Help Menu

Every professional CLI tool should provide help.

Example:

```text
Linux Backup Utility

Usage:

./backup.sh [OPTIONS]

Options:

--help

--source=PATH

--destination=PATH

--compression=gzip

--compression=bzip2

--compression=xz
```

---

# Multiple Compression Methods

Instead of always using:

```bash
tar -czf
```

Support multiple formats.

Example:

```bash
case "$COMPRESSION" in

gzip)

tar -czf "$BACKUP_PATH" "$SOURCE"

;;

bzip2)

BACKUP_PATH="${BACKUP_PATH%.gz}.bz2"

tar -cjf "$BACKUP_PATH" "$SOURCE"

;;

xz)

BACKUP_PATH="${BACKUP_PATH%.gz}.xz"

tar -cJf "$BACKUP_PATH" "$SOURCE"

;;

*)

log "Unsupported compression."

exit 1

;;

esac
```

Now your utility becomes much more useful.

---

# Cleaning Old Backups

Instead of keeping backups forever:

```bash
find "$DESTINATION" \
-name "*.tar*" \
-mtime +"$RETENTION_DAYS" \
-delete
```

Log the cleanup:

```bash
log "Old backups cleaned."
```

This introduces automatic retention.

---

# Colored Output

Improve readability.

```bash
GREEN="\033[0;32m"

RED="\033[0;31m"

YELLOW="\033[1;33m"

NC="\033[0m"
```

Example:

```bash
echo -e "${GREEN}Backup completed successfully.${NC}"
```

Errors:

```bash
echo -e "${RED}Backup failed.${NC}"
```

Warnings:

```bash
echo -e "${YELLOW}Cleaning old backups...${NC}"
```

---

# Exit Codes

Professional tools return meaningful exit codes.

| Exit Code | Meaning |
|-----------|----------|
| 0 | Success |
| 1 | General Error |
| 2 | Invalid Configuration |
| 3 | Backup Failed |
| 4 | Verification Failed |
| 5 | Restore Failed |

This allows automation systems to react appropriately.

---

# Final Application Flow

```text
Load Configuration

↓

Parse Arguments

↓

Validate Configuration

↓

Create Backup

↓

Generate SHA-256

↓

Verify SHA-256

↓

Restore Test

↓

Cleanup Old Backups

↓

Display Summary

↓

Exit
```

---

# Practical Scenario

Your company stores backups on:

```text
Local NAS
```

Sometimes you also back up:

```text
External SSD
```

Instead of editing the script:

```bash
./backup.sh \
--destination=/media/backup-drive
```

No code changes required.

---

# Hands-on Lab

## Step 1

Implement:

```bash
check_status()
```

---

## Step 2

Convert repeated code into functions.

---

## Step 3

Implement:

```bash
show_help()
```

---

## Step 4

Support:

```bash
--compression=xz
```

---

## Step 5

Support:

```bash
--destination=/tmp/backups
```

---

## Step 6

Implement automatic cleanup.

---

## Step 7

Add colored output.

---

## Step 8

Test every supported compression type.

---

# Challenge

Implement one additional feature of your own.

Ideas:

- Display free disk space.
- Display hostname.
- Show current user.
- Display backup count.
- Ask for confirmation before deleting old backups.
- Support a dry-run mode that shows what would happen without creating a backup.

Explain why your feature improves the utility.

---

# Common Mistakes

- Mixing configuration with logic.
- Writing duplicate code.
- Forgetting to validate user input.
- Deleting backups without checking the path.
- Ignoring unsupported compression methods.

---

# Best Practices

- Keep functions small.
- Validate configuration before starting.
- Use descriptive log messages.
- Return meaningful exit codes.
- Support command-line arguments.
- Avoid duplicated logic.

---

# Interview Questions

1. What is DRY?

2. Why use functions in Bash?

3. Why are exit codes important?

4. Why support configuration files and command-line arguments?

5. Why should backup cleanup be automated?

6. Why is a help menu useful?

7. How would you extend this utility in the future?

---

# Lesson Reflection

Before moving on, ask yourself:

- Can I refactor duplicated Bash code?
- Can I organize a script into reusable functions?
- Can I support command-line arguments?
- Can I explain why professional CLI tools provide help menus?

---

# Summary

In this lesson, you transformed a working backup script into a professional command-line utility.

You introduced reusable functions, centralized error handling, configurable compression methods, automatic cleanup, command-line arguments, colored output, and meaningful exit codes.

The result is a much more maintainable and extensible application that reflects real-world Bash scripting practices.

---

# Cheat Sheet

| Feature | Purpose |
|----------|---------|
| `check_status()` | Centralized error handling |
| `log()` | Consistent logging |
| `show_help()` | Display CLI help |
| `case` | Parse arguments |
| `find -mtime` | Remove old backups |
| ANSI escape codes | Colored terminal output |
| Exit codes | Automation-friendly status reporting |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-15-part3/
        ├── backup-v2.sh
        ├── cleanup-test.txt
        ├── compression-tests.md
        ├── help-output.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- Help screen output
- Tests for each compression format
- Cleanup verification
- Screenshots of colored terminal output
- Notes describing your custom feature

---

# Next Lesson

**Lesson 16 — Chapter Review, Final Audit & Portfolio Preparation**

You'll finish Chapter 03 by creating:

- A professional `README.md`
- A complete Chapter 03 cheat sheet
- A command reference
- Top interview questions
- Troubleshooting guide
- Final chapter audit
- GitHub portfolio improvements

# Chapter 03 — Archives, Compression & Backups

# Lesson 16 — Chapter Review, Final Audit & Portfolio Preparation

---

# Overview

Congratulations!

You have completed Chapter 03.

This chapter took you from understanding the difference between archives and compression to building a professional backup utility suitable for a GitHub portfolio.

Before moving on to the next chapter, professional engineers perform one final step:

- Review
- Audit
- Document
- Improve

This lesson helps you consolidate everything you've learned and prepare your work for future reference and interviews.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Review all Chapter 03 concepts.
- Audit your knowledge.
- Organize your GitHub repository.
- Prepare for technical interviews.
- Verify your portfolio project.
- Identify areas for future improvement.

---

# Chapter Summary

During this chapter you learned:

## Archives

- `tar`
- Creating archives
- Listing archives
- Extracting archives

---

## Compression

- `gzip`
- `bzip2`
- `xz`
- `zip`

---

## Combined Archives

- `.tar.gz`
- `.tar.bz2`
- `.tar.xz`

---

## Backup Concepts

- Backup fundamentals
- Full backups
- Incremental backups
- Differential backups
- 3-2-1 Backup Rule
- Retention policies
- Backup scheduling

---

## Backup Verification

- SHA-256
- Archive verification
- Restore testing
- RTO
- RPO

---

## Bash Project

You built:

```text
Linux Backup Utility v2.0
```

Features:

- Timestamped backups
- Configuration file
- Logging
- Checksums
- Verification
- Restore testing
- Cleanup
- Command-line arguments
- Multiple compression methods
- Colored output

---

# Complete Command Reference

## tar

```bash
tar -cf archive.tar directory/
tar -tf archive.tar
tar -xf archive.tar
```

---

## gzip

```bash
gzip file
gunzip file.gz
gzip -t file.gz
gzip -l file.gz
```

---

## bzip2

```bash
bzip2 file
bunzip2 file.bz2
bzip2 -t file.bz2
```

---

## xz

```bash
xz file
unxz file.xz
xz -t file.xz
xz -l file.xz
```

---

## zip

```bash
zip -r project.zip project/
unzip project.zip
unzip -l project.zip
unzip -t project.zip
```

---

## Checksums

```bash
sha256sum file
sha256sum file > file.sha256
sha256sum -c file.sha256
```

---

# Decision Guide

| Situation | Best Tool |
|-----------|-----------|
| Daily Linux backups | `tar.gz` |
| Long-term storage | `tar.xz` |
| Cross-platform sharing | `zip` |
| Fast compression | `gzip` |
| Better compression | `bzip2` |
| Maximum compression | `xz` |

---

# Backup Design Checklist

Before deploying a backup system, ask:

- What data must be protected?
- How often should backups run?
- How much data loss is acceptable (RPO)?
- How quickly must systems recover (RTO)?
- Where will backups be stored?
- How long should backups be retained?
- How will backups be verified?
- How often will recovery tests be performed?

---

# Professional Backup Checklist

Every backup should include:

✓ Archive creation

✓ Compression

✓ Verification

✓ SHA-256 checksum

✓ Restore testing

✓ Logging

✓ Retention policy

✓ Off-site storage

✓ Documentation

---

# Troubleshooting Guide

## Archive Won't Extract

Possible causes:

- Corruption
- Incomplete download
- Wrong compression type

Check:

```bash
tar -tf archive.tar.gz
gzip -t archive.tar.gz
```

---

## SHA-256 Verification Failed

Possible causes:

- File modified
- Incomplete copy
- Disk corruption

Verify:

```bash
sha256sum -c backup.sha256
```

---

## Backup Script Fails

Check:

- Permissions
- Paths
- Available disk space
- Log file

---

## Backup Size Seems Wrong

Compare:

```bash
du -h

ls -lh
```

Verify:

```bash
tar -tf
```

---

# Top Interview Questions

## Basic

- What is the difference between an archive and compression?
- Why is `tar` used?
- Why isn't `tar` a compressor?
- What is `.tar.gz`?

---

## Intermediate

- Compare `gzip`, `bzip2`, and `xz`.
- Why use SHA-256?
- Explain Full vs Incremental backups.
- Explain Differential backups.
- What is the 3-2-1 Rule?

---

## Advanced

- What is RTO?
- What is RPO?
- How would you design a backup system for a production web server?
- How would you verify nightly backups?
- How would you automate backups?

---

# Portfolio Review

Your backup utility should now demonstrate:

✓ Bash scripting

✓ Linux administration

✓ Archives

✓ Compression

✓ Checksums

✓ Logging

✓ Error handling

✓ Configuration files

✓ CLI design

✓ Documentation

This is a strong portfolio project for a junior Linux Administrator or DevOps Engineer.

---

# GitHub Repository Structure

Your project should look similar to:

```text
backup-project/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── backup.sh
├── config/
│   └── backup.conf
├── backups/
├── logs/
│   └── backup.log
├── restore-test/
├── assets/
│   ├── screenshots/
│   └── diagrams/
└── docs/
    ├── INSTALL.md
    ├── CONFIGURATION.md
    └── USAGE.md
```

---

# Chapter Audit

## Commands

- [ ] tar
- [ ] gzip
- [ ] bzip2
- [ ] xz
- [ ] zip
- [ ] unzip
- [ ] sha256sum

---

## Concepts

- [ ] Archive
- [ ] Compression
- [ ] Backup
- [ ] Restore
- [ ] Verification
- [ ] Full Backup
- [ ] Incremental Backup
- [ ] Differential Backup
- [ ] 3-2-1 Rule
- [ ] RPO
- [ ] RTO
- [ ] Retention Policy

---

## Practical Skills

- [ ] Create archives
- [ ] Compress files
- [ ] Restore backups
- [ ] Verify archives
- [ ] Generate checksums
- [ ] Build Bash utilities
- [ ] Organize Linux projects

---

# Reflection

Ask yourself:

- Can I explain the difference between archiving and compression?
- Can I choose the right compression method?
- Can I design a backup strategy?
- Can I explain RTO and RPO?
- Can I verify a backup?
- Can I write a reusable Bash utility?
- Could I explain this project during a job interview?

If you can confidently answer "yes" to these questions, you've achieved the goals of this chapter.

---

# Final Challenge

Create a fresh Ubuntu virtual machine (or use your laptop) and, without referring to your notes:

1. Build the backup project from scratch.
2. Configure it.
3. Generate backups.
4. Verify them.
5. Restore them.
6. Demonstrate each feature.
7. Push the completed project to GitHub.

If you can complete this challenge, you've truly mastered the chapter.

---

# Chapter Cheat Sheet

| Task | Command |
|------|---------|
| Create archive | `tar -cf archive.tar dir/` |
| Create `.tar.gz` | `tar -czf archive.tar.gz dir/` |
| Create `.tar.bz2` | `tar -cjf archive.tar.bz2 dir/` |
| Create `.tar.xz` | `tar -cJf archive.tar.xz dir/` |
| Extract archive | `tar -xf archive.tar` |
| List archive | `tar -tf archive.tar` |
| Compress file | `gzip file` |
| Verify gzip | `gzip -t file.gz` |
| Generate checksum | `sha256sum file` |
| Verify checksum | `sha256sum -c file.sha256` |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── final/
        ├── chapter-audit.md
        ├── interview-notes.md
        ├── backup-project-final/
        ├── screenshots/
        ├── diagrams/
        └── reflection.md
```

Capture:

- Final backup utility screenshots
- Project directory tree
- Successful backup execution
- Successful restore verification
- GitHub repository screenshot
- Personal reflection on what you learned

---

# What's Next?

## Chapter 04 — Process Management & Job Control

You'll learn how Linux manages running programs and system resources.

Topics include:

- Process lifecycle
- `ps`
- `top`
- `htop`
- `pstree`
- `kill`
- `killall`
- `pkill`
- `nice`
- `renice`
- `jobs`
- `bg`
- `fg`
- `nohup`
- Signals
- Scheduling priorities
- Process monitoring

The chapter will conclude with another portfolio project:

> **Linux Process Monitoring Toolkit**

You'll build a Bash application that monitors CPU usage, memory, running processes, and system load, generating professional reports suitable for system administration tasks.

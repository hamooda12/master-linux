# Chapter 03 — Archives, Compression & Backups

# Lesson 12 — Backup Policies: The 3-2-1 Rule, Retention Policies & Scheduling

---

# Overview

Knowing how to create backups is only part of the job.

Professional administrators must also answer important questions such as:

- How many backup copies should we keep?
- Where should backups be stored?
- How often should backups run?
- How long should backups be retained?
- What happens if an entire data center is destroyed?

These questions are answered through backup policies.

In this lesson, you will learn three essential concepts:

- The 3-2-1 Backup Rule
- Backup Retention Policies
- Backup Scheduling

These concepts form the foundation of modern disaster recovery planning.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain the 3-2-1 Backup Rule.
- Design a backup retention policy.
- Create a professional backup schedule.
- Understand on-site and off-site backups.
- Explain why backup redundancy is critical.

---

# Prerequisites

You should understand:

- Backup fundamentals
- Full, Incremental, and Differential backups
- Basic Linux filesystem concepts

---

# Why Backup Policies Matter

Imagine this situation:

```text
Production Server
        │
        ▼
Backup Disk
```

Everything is stored in the same server room.

One fire destroys:

- Production server
- Backup disk

Result:

```text
Everything Lost
```

The backups existed…

…but they were stored incorrectly.

---

# The 3-2-1 Backup Rule

The industry-standard recommendation is:

```text
3 Copies

2 Different Media

1 Off-site Copy
```

Let's examine each part.

---

# Three Copies

Keep:

```text
Original Data

+

Backup Copy #1

+

Backup Copy #2
```

Never rely on only one backup.

---

# Two Different Storage Media

Store backups on different types of storage.

Examples:

```text
Production SSD

↓

External HDD

↓

Network Storage (NAS)
```

or

```text
Server Disk

↓

Cloud Storage
```

Using different media reduces the risk of hardware failure.

---

# One Off-site Copy

Keep at least one backup in another physical location.

Examples:

- Cloud storage
- Another office
- Another data center

If the entire building is lost, the off-site backup survives.

---

# Visual Diagram

```text
               Production Server
                      │
      ┌───────────────┼───────────────┐
      │                               │
Local Backup                    Cloud Backup
External HDD                   Object Storage
```

---

# Why the 3-2-1 Rule Works

It protects against:

- Disk failure
- Server failure
- Fire
- Flood
- Theft
- Ransomware
- Human error

One failure should never destroy every copy.

---

# Backup Retention Policy

Retention answers one question:

> How long should backups be kept?

Keeping backups forever is expensive.

Deleting them too early is risky.

Organizations define retention rules.

---

# Example Retention Policy

| Backup Type | Keep For |
|--------------|---------|
| Daily | 7 days |
| Weekly | 4 weeks |
| Monthly | 12 months |
| Yearly | 7 years |

This allows recovery from:

- Yesterday
- Last week
- Last month
- Last year

---

# Why Retention Matters

Suppose an employee accidentally deletes a document.

If the mistake is discovered:

Tomorrow:

```text
Yesterday's backup
```

works.

Six months later:

Yesterday's backup is no longer useful.

Long-term backups become important.

---

# Backup Rotation

Organizations remove old backups according to policy.

Example:

```text
backup-01

↓

backup-02

↓

backup-03

↓

backup-04

↓

Delete backup-01
```

This prevents unlimited storage growth.

---

# Backup Scheduling

A backup schedule defines:

- When backups run.
- Which type of backup runs.
- How frequently backups occur.

---

# Example Schedule

| Day | Backup |
|-----|--------|
| Sunday | Full |
| Monday | Incremental |
| Tuesday | Incremental |
| Wednesday | Incremental |
| Thursday | Incremental |
| Friday | Incremental |
| Saturday | Incremental |

Next Sunday:

```text
New Full Backup
```

---

# Another Example

A company with critical databases:

```text
Daily

Incremental
```

```text
Weekly

Full
```

```text
Monthly

Archive
```

---

# Business Requirements

Different systems require different schedules.

Examples:

## Personal Laptop

```text
Weekly
```

may be enough.

---

## Web Server

```text
Daily
```

is common.

---

## Financial Database

```text
Hourly Incremental

+

Daily Full
```

may be required.

---

# Backup Window

A backup should ideally run when system activity is low.

Examples:

```text
02:00 AM
```

instead of:

```text
11:00 AM
```

Running large backups during peak business hours may reduce system performance.

---

# Practical Scenario

A company hosts an online shopping website.

Policy:

```text
Daily Incremental

Weekly Full

Monthly Archive
```

Storage:

```text
Production

↓

NAS

↓

Cloud Storage
```

Retention:

```text
Daily
7 Days

Weekly
4 Weeks

Monthly
12 Months
```

This provides a balance between recovery speed and storage cost.

---

# Hands-on Lab

## Step 1

Create a planning directory.

```bash
mkdir -p ~/linux-practice/chapter-03/backup-policy
cd ~/linux-practice/chapter-03/backup-policy
```

---

## Step 2

Create a backup policy document.

```bash
nano backup-policy.md
```

Include:

```text
Backup Strategy:
Full + Incremental

Retention:
Daily: 7 days
Weekly: 4 weeks
Monthly: 12 months

Storage:
Local NAS
Cloud Storage

Backup Window:
02:00 AM
```

---

## Step 3

Create a weekly schedule.

```text
Sunday      Full
Monday      Incremental
Tuesday     Incremental
Wednesday   Incremental
Thursday    Incremental
Friday      Incremental
Saturday    Incremental
```

---

# Challenge

Design a backup policy for:

A company with:

- 10 TB of customer data
- 24/7 operations
- Daily database updates
- One office
- Cloud storage available

Answer:

1. Which backup strategy would you use?
2. How often would backups run?
3. What retention policy would you recommend?
4. Where would backups be stored?
5. How would you satisfy the 3-2-1 rule?

Explain your reasoning.

---

# Common Mistakes

## Mistake 1

Keeping all backups in one location.

---

## Mistake 2

Never deleting old backups.

Storage eventually fills up.

---

## Mistake 3

Scheduling backups during peak business hours.

---

## Mistake 4

Ignoring recovery requirements when designing retention.

---

# Best Practices

- Follow the 3-2-1 Backup Rule.
- Automate backup schedules.
- Keep at least one off-site copy.
- Review retention policies regularly.
- Match backup frequency to business needs.
- Monitor storage capacity.

---

# Interview Questions

1. What is the 3-2-1 Backup Rule?

2. Why are off-site backups important?

3. What is a backup retention policy?

4. Why shouldn't backups be kept forever?

5. What is a backup window?

6. Why are backups often scheduled overnight?

7. How would you design a backup schedule for a busy web server?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I explain the 3-2-1 Backup Rule?
- Can I create a retention policy?
- Can I design a weekly backup schedule?
- Do I understand why off-site storage is important?

---

# Summary

In this lesson, you learned how organizations design backup policies that balance reliability, storage costs, and recovery needs.

You explored the 3-2-1 Backup Rule, backup retention, scheduling, and storage planning.

These concepts are essential for building resilient systems that can recover from hardware failures, disasters, and human error.

---

# Cheat Sheet

| Concept | Recommendation |
|----------|----------------|
| Backup Rule | 3 copies, 2 media types, 1 off-site |
| Daily Backups | Short retention (e.g., 7 days) |
| Weekly Backups | Medium retention (e.g., 4 weeks) |
| Monthly Backups | Long retention (e.g., 12 months) |
| Backup Window | Low system activity (overnight) |
| Storage | Local + Off-site |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-12/
        ├── backup-policy.md
        ├── weekly-schedule.md
        ├── retention-policy.md
        ├── backup-architecture.txt
        ├── challenge-solution.md
        ├── screenshots/
        └── notes.md
```

Capture:

- Your backup policy
- Weekly backup schedule
- Retention table
- 3-2-1 architecture diagram
- Challenge solution
- Notes explaining your design decisions

---

# Next Lesson

**Lesson 13 — Backup Verification & Recovery Testing**

You'll learn why successful backups are meaningless unless they can be restored, how to verify backup integrity, perform recovery drills, generate checksums, and build confidence in your backup process.

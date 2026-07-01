# Chapter 03 — Archives, Compression & Backups

# Lesson 11 — Backup Strategies: Full, Incremental & Differential

---

# Overview

Not all backups are created the same way.

Imagine backing up a server containing 2 TB of data every single day.

It would:

- Consume large amounts of storage.
- Take a long time.
- Increase network usage.
- Waste system resources.

Professional backup systems solve this problem by using different backup strategies.

The three most common strategies are:

- Full Backup
- Incremental Backup
- Differential Backup

Each has different advantages and trade-offs.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain Full, Incremental, and Differential backups.
- Compare storage and recovery time.
- Choose the appropriate strategy.
- Understand how professional backup schedules work.
- Design a simple backup plan.

---

# Prerequisites

Before this lesson, you should understand:

- Backup fundamentals
- Archives
- Compression
- File management

---

# Why Backup Strategies Exist

Suppose your project contains:

```text
project/

100 GB
```

Backing up the entire project every day:

```text
Monday
100 GB

Tuesday
100 GB

Wednesday
100 GB

Thursday
100 GB
```

After four days:

```text
400 GB
```

Most of the files have not changed.

This wastes:

- Storage
- Time
- Network bandwidth

---

# Full Backup

A Full Backup copies **everything**.

Every file is backed up regardless of whether it changed.

Example:

```text
Monday

Project

↓

Full Backup

100 GB
```

Tuesday:

```text
Full Backup

100 GB
```

Wednesday:

```text
Full Backup

100 GB
```

Every backup is complete.

---

# Advantages of Full Backup

- Easy to understand.
- Fast recovery.
- Every backup is self-contained.
- Simplest restore process.

---

# Disadvantages

- Largest storage usage.
- Longest backup time.
- High bandwidth usage.

---

# Incremental Backup

An Incremental Backup stores **only changes since the last backup of any type**.

Example timeline:

```text
Monday

Full Backup

100 GB
```

Tuesday:

Only 3 GB changed.

```text
Incremental

3 GB
```

Wednesday:

Only 2 GB changed.

```text
Incremental

2 GB
```

Thursday:

Only 5 GB changed.

```text
Incremental

5 GB
```

Storage used:

```text
100 + 3 + 2 + 5

110 GB
```

instead of:

```text
400 GB
```

---

# Visual Timeline

```text
Monday

[FULL]

↓

Tuesday

[INC]

↓

Wednesday

[INC]

↓

Thursday

[INC]
```

---

# Restoring Incremental Backups

Suppose Thursday's server fails.

To recover Thursday's data, you need:

```text
Monday Full

+

Tuesday Incremental

+

Wednesday Incremental

+

Thursday Incremental
```

All backups must be available.

---

# Advantages

- Small backups.
- Fast backup process.
- Low storage usage.

---

# Disadvantages

- Slower recovery.
- More complex restoration.
- Missing one incremental backup may prevent a complete restore.

---

# Differential Backup

A Differential Backup stores **all changes since the last Full Backup**.

Example:

Monday:

```text
Full

100 GB
```

Tuesday:

3 GB changed.

```text
Differential

3 GB
```

Wednesday:

Tuesday + Wednesday changes.

```text
5 GB
```

Thursday:

Tuesday + Wednesday + Thursday changes.

```text
10 GB
```

Notice that each differential backup grows until the next Full Backup.

---

# Visual Timeline

```text
Monday

[FULL]

↓

Tuesday

[DIFF]

↓

Wednesday

[DIFF]

↓

Thursday

[DIFF]
```

---

# Restoring Differential Backups

To restore Thursday:

You only need:

```text
Monday Full

+

Thursday Differential
```

This makes recovery simpler than Incremental backups.

---

# Advantages

- Faster recovery than Incremental.
- Simpler restore process.
- Moderate storage usage.

---

# Disadvantages

- Larger backups over time.
- Uses more storage than Incremental.
- Slower backup than Incremental.

---

# Strategy Comparison

| Feature | Full | Incremental | Differential |
|----------|------|-------------|--------------|
| Backup Size | Largest | Smallest | Medium |
| Backup Speed | Slowest | Fastest | Medium |
| Storage Usage | Highest | Lowest | Medium |
| Restore Speed | Fastest | Slowest | Faster |
| Complexity | Low | High | Medium |

---

# Recovery Comparison

Recovering Thursday:

## Full

```text
Thursday Full
```

One file.

---

## Incremental

```text
Monday Full

+

Tuesday

+

Wednesday

+

Thursday
```

Four backups.

---

## Differential

```text
Monday Full

+

Thursday Differential
```

Two backups.

---

# Which Strategy Should You Choose?

## Small Business

Often:

```text
Full
```

Simple.

---

## Large Enterprise

Often:

```text
Full

+

Incremental
```

Storage savings are important.

---

## Medium Environment

Often:

```text
Full

+

Differential
```

Balances storage and recovery time.

---

# Real-World Backup Schedule

Example:

```text
Sunday

Full Backup
```

```text
Monday

Incremental
```

```text
Tuesday

Incremental
```

```text
Wednesday

Incremental
```

```text
Thursday

Incremental
```

```text
Friday

Incremental
```

```text
Saturday

Incremental
```

Next Sunday:

```text
New Full Backup
```

This schedule is very common.

---

# Practical Scenario

You administer a company file server.

Storage is expensive.

The server changes every day.

Your backup policy:

```text
Sunday

Full Backup
```

```text
Monday–Saturday

Incremental Backups
```

This minimizes storage while still allowing recovery.

---

# Hands-on Lab

## Scenario

A project starts at:

```text
100 GB
```

Daily changes:

```text
Tuesday

4 GB
```

```text
Wednesday

2 GB
```

```text
Thursday

7 GB
```

### Exercise 1

Calculate total storage required using:

- Full backups
- Incremental backups
- Differential backups

Create:

```text
backup-comparison.md
```

Document your calculations.

---

### Exercise 2

Draw the backup timeline using ASCII diagrams.

---

### Exercise 3

Explain how you would restore Thursday's backup for each strategy.

---

# Challenge

A company has:

- 5 TB database
- 100 GB changes per day

Answer:

1. Would Full backups every day be practical?
2. Would Incremental backups save storage?
3. Would Differential backups simplify recovery?
4. Which strategy would you recommend and why?

---

# Common Mistakes

## Mistake 1

Thinking Incremental and Differential are the same.

They are not.

---

## Mistake 2

Ignoring restore complexity.

A fast backup is useless if recovery is too slow during an emergency.

---

## Mistake 3

Choosing based only on storage.

Recovery time is equally important.

---

# Best Practices

- Perform regular Full backups.
- Use Incremental or Differential between Full backups.
- Document your backup schedule.
- Test recovery procedures regularly.
- Balance storage, backup time, and recovery time.

---

# Interview Questions

1. What is a Full Backup?

2. What is an Incremental Backup?

3. What is a Differential Backup?

4. Which strategy uses the least storage?

5. Which strategy restores the fastest?

6. Which strategy requires the most backup files during recovery?

7. Why do enterprises often combine Full and Incremental backups?

8. Which strategy would you recommend for a small company?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I explain all three backup strategies?
- Can I compare their advantages and disadvantages?
- Can I design a weekly backup schedule?
- Can I recommend a strategy based on business needs?

---

# Summary

In this lesson, you learned the three primary backup strategies used in professional environments.

You compared their storage requirements, backup speed, recovery complexity, and practical use cases.

Choosing the right backup strategy is a critical responsibility for Linux administrators and directly impacts business continuity.

---

# Cheat Sheet

| Strategy | Backup Size | Restore Speed | Storage Usage | Best For |
|----------|-------------|---------------|---------------|-----------|
| Full | Large | Fast | High | Small environments |
| Incremental | Small | Slow | Low | Large environments |
| Differential | Medium | Medium/Fast | Medium | Balanced workloads |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-03/
    └── lesson-11/
        ├── backup-comparison.md
        ├── backup-timeline.txt
        ├── recovery-plan.md
        ├── storage-calculations.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- Storage calculations
- ASCII backup timeline
- Recovery plans for each strategy
- Notes explaining your recommended backup strategy
- Any diagrams you created

---

# Next Lesson

**Lesson 12 — The 3-2-1 Backup Rule, Retention Policies & Backup Scheduling**

You'll learn how organizations protect backups against disasters, how long backups should be kept, and how to design a professional backup policy.

# Chapter 04 — Process Management & System Monitoring

# Lesson 04 — Managing Process Priority with `nice` & `renice`

---

# Overview

Not all processes are equally important.

Imagine you are running a production web server while also compressing a large backup archive.

Both tasks require CPU time.

Should they receive the same priority?

Probably not.

Linux solves this problem using **process priorities**.

By adjusting a process's priority, you can influence how much CPU time the Linux scheduler allocates to it relative to other processes.

This lesson introduces the concepts of **nice values** and the `nice` and `renice` commands, which allow administrators to control process scheduling.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Understand how Linux schedules CPU time.
- Explain process priority.
- Understand nice values.
- Launch processes with custom priorities.
- Change the priority of running processes.
- Identify process priorities using `ps` and `top`.
- Apply best practices when changing priorities.

---

# Prerequisites

Before continuing, you should understand:

- Processes
- Process IDs (PID)
- Real-time monitoring with `top` and `htop`
- Basic Linux permissions

---

# Why Process Priority Exists

A Linux system may run hundreds of processes simultaneously.

Examples include:

- Web servers
- Databases
- Docker containers
- Backup jobs
- Video rendering
- Software compilation
- User applications

The CPU can execute only a limited number of instructions at any instant.

The Linux scheduler decides which process gets CPU time next.

Priority helps the scheduler make better decisions.

---

# What is a Nice Value?

Linux uses a value called the **nice value** (or niceness) to influence scheduling.

The range is:

```text
-20 .............................. 19
```

Where:

```text
-20
```

Highest priority

```text
0
```

Default priority

```text
19
```

Lowest priority

Think of it this way:

> A "nice" process is polite—it willingly gives CPU time to others.

Therefore:

Higher nice value → Lower CPU priority

Lower nice value → Higher CPU priority

---

# Relationship Between Priority and Nice

| Nice Value | CPU Priority |
|------------|--------------|
| -20 | Highest |
| -10 | Very High |
| 0 | Default |
| 10 | Low |
| 19 | Lowest |

Remember:

Higher nice = Lower priority.

---

# Viewing Nice Values

Use `ps`:

```bash
ps -eo pid,ni,comm
```

Example:

```text
PID  NI COMMAND

1     0 systemd
432   0 sshd
801   0 bash
912  10 backup.sh
```

Column meanings:

- PID → Process ID
- NI → Nice value
- COMMAND → Executable

---

# Viewing Priority in top

Launch:

```bash
top
```

Notice the columns:

```text
PR
NI
```

Example:

```text
PID USER PR NI COMMAND

4012 hamad 20 0 firefox
```

Where:

- PR → Scheduler priority
- NI → Nice value

---

# Starting a Process with nice

Syntax:

```bash
nice -n <nice_value> command
```

Example:

```bash
nice -n 10 yes > /dev/null
```

The process starts with a lower CPU priority.

Verify:

```bash
ps -eo pid,ni,comm | grep yes
```

Example:

```text
4321 10 yes
```

---

# Starting a High-Priority Process

Only the root user can decrease the nice value (increase priority).

Example:

```bash
sudo nice -n -10 command
```

Without sufficient privileges:

```text
nice: cannot set niceness: Permission denied
```

---

# Changing Priority of a Running Process

Suppose a process is already running.

Find its PID:

```bash
ps aux | grep firefox
```

Example:

```text
4521 firefox
```

Change its nice value:

```bash
renice 10 -p 4521
```

Output:

```text
4521 (process ID) old priority 0, new priority 10
```

---

# Increasing Priority

Only root can assign negative nice values.

Example:

```bash
sudo renice -5 -p 4521
```

---

# Verifying the Change

Run:

```bash
ps -o pid,ni,comm -p 4521
```

Example:

```text
PID NI COMMAND

4521 -5 firefox
```

---

# Practical Scenario

A nightly backup script consumes too much CPU during business hours.

Instead of stopping it, you lower its priority.

Start the backup:

```bash
nice -n 15 tar -czf backup.tar.gz project/
```

The backup still runs, but interactive applications receive CPU time first.

This improves the responsiveness of the server for users.

---

# Hands-on Lab

## Step 1

Open Terminal A.

Run:

```bash
yes > /dev/null
```

This creates a CPU-intensive process.

---

## Step 2

Open Terminal B.

Find the process:

```bash
ps aux | grep yes
```

Record the PID.

---

## Step 3

Check its nice value:

```bash
ps -o pid,ni,comm -p <PID>
```

Expected:

```text
NI = 0
```

---

## Step 4

Lower its priority:

```bash
renice 15 -p <PID>
```

---

## Step 5

Verify:

```bash
ps -o pid,ni,comm -p <PID>
```

Expected:

```text
NI = 15
```

---

## Step 6

Open another terminal.

Launch:

```bash
top
```

Observe:

- NI column
- PR column

---

## Step 7

Terminate the process:

```bash
kill <PID>
```

---

# Understanding PR vs NI

Many beginners confuse these columns.

| Column | Meaning |
|----------|----------|
| NI | Nice value assigned by the user |
| PR | Scheduler priority calculated by Linux |

Linux calculates the effective priority using several internal factors, including the nice value.

Therefore:

Changing NI usually changes PR, but they are not identical.

---

# Common Mistakes

### Mistake 1

Thinking nice values guarantee CPU time.

Reality:

They only influence scheduling.

---

### Mistake 2

Trying to assign negative nice values as a normal user.

Only root can increase process priority.

---

### Mistake 3

Using high-priority values for every process.

Doing so defeats the purpose of the scheduler and may reduce system responsiveness.

---

# Best Practices

- Keep the default priority unless there is a clear reason to change it.
- Lower the priority of CPU-intensive background jobs.
- Increase priority only for critical services and only when necessary.
- Always monitor system behavior after changing priorities.
- Document any permanent priority adjustments on production servers.

---

# Interview Questions

1. What is the purpose of a nice value?
2. What is the valid range of nice values?
3. Which nice value has the highest priority?
4. Why can only root assign negative nice values?
5. What is the difference between `nice` and `renice`?
6. How can you display the nice value of a running process?
7. What is the difference between the `PR` and `NI` columns in `top`?

---

# Lesson Reflection

After completing this lesson, you should be able to answer:

- How does Linux decide which process receives CPU time?
- When should a process have a higher or lower priority?
- How do I launch a process with a custom nice value?
- How do I change the priority of an existing process?
- Why are negative nice values restricted to privileged users?

---

# Summary

In this lesson, you learned how Linux process priorities work through the concept of nice values. You practiced launching processes with custom priorities using `nice`, modifying running processes with `renice`, and verifying changes using `ps` and `top`. You also learned the difference between scheduler priority (`PR`) and user-controlled nice values (`NI`), and explored practical scenarios where adjusting process priority improves overall system performance.

---

# Cheat Sheet

```bash
# View process nice values
ps -eo pid,ni,comm

# Start process with lower priority
nice -n 10 command

# Start process with higher priority (root)
sudo nice -n -10 command

# Change priority of running process
renice 10 -p PID

# Increase priority (root)
sudo renice -5 -p PID

# Verify nice value
ps -o pid,ni,comm -p PID

# View priorities in top
top
```

---

# Lab Assets

Create the following structure:

```text
assets/
└── chapter-04/
    └── lesson-04/
        ├── ps-nice-values.png
        ├── top-pr-ni-columns.png
        ├── renice-before-after.png
        ├── yes-process.png
        ├── process-after-renice.png
        └── notes.md
```

Capture:

- `ps` output showing the `NI` column.
- `top` displaying `PR` and `NI`.
- Output before and after using `renice`.
- CPU-intensive `yes` process.
- Personal observations about scheduler behavior.

---

# Next Lesson

**Lesson 05 — Signals, `kill`, `pkill`, `killall` & Graceful Process Termination**

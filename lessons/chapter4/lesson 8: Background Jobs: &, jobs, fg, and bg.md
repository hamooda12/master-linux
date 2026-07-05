# Chapter 04 — Process Management & System Monitoring

# Lesson 08 — Background Jobs: `&`, `jobs`, `fg`, and `bg`

---

# Overview

When working in the Linux terminal, every command you execute normally runs in the **foreground**.

This means the terminal is occupied until the command finishes.

For short commands like `ls` or `pwd`, this is not a problem because they complete almost instantly.

However, many administrative tasks take much longer:

- Copying large files
- Compressing backups
- Downloading software
- Running scripts
- Compiling programs

If these commands always occupied the terminal, administrators would have to wait before doing anything else.

Linux solves this problem with **job control**.

Job control allows you to:

- Run commands in the background.
- Continue using the same terminal.
- Pause running jobs.
- Resume paused jobs.
- Bring jobs back to the foreground.

These features make administrators much more productive and are used daily when managing Linux systems.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain the difference between foreground and background processes.
- Run commands in the background.
- View active jobs.
- Move jobs between the foreground and background.
- Suspend and resume running commands.
- Understand when background jobs should and should not be used.

---

# Prerequisites

Before continuing, you should understand:

- Linux processes
- Process IDs (PID)
- Signals
- Basic terminal commands

---

# Why Job Control Exists

Imagine you are compressing a 100 GB backup:

```bash
tar -czf backup.tar.gz project/
```

The command may take several minutes.

During that time, your terminal becomes unavailable.

You cannot type another command until it finishes.

Instead of opening another terminal, Linux allows you to place long-running commands in the background so you can continue working.

---

# Foreground vs Background

## Foreground Process

A foreground process has control of the terminal.

You must wait until it finishes before entering another command.

Example:

```bash
sleep 30
```

The terminal waits for 30 seconds.

---

## Background Process

A background process continues running while the terminal remains available.

Example:

```bash
sleep 30 &
```

Notice the `&` at the end.

Linux immediately returns the prompt so you can continue working.

---

# Running a Command in the Background

Simply add an ampersand (`&`) to the end of the command.

Example:

```bash
sleep 300 &
```

Example output:

```text
[1] 5423
```

Meaning:

- `[1]` → Job number
- `5423` → Process ID (PID)

---

# Viewing Background Jobs

Use:

```bash
jobs
```

Example:

```text
[1]+ Running    sleep 300 &
```

This command lists jobs managed by the current shell.

---

# Job Number vs Process ID

These are different.

Example:

```text
[1] Running sleep 300
```

`1` is the **job number**.

The process also has a real PID:

```bash
ps
```

Example:

```text
PID     COMMAND
5423    sleep
```

Linux commands such as `kill` use the **PID**, while `fg` and `bg` use the **job number**.

---

# Suspending a Foreground Process

Suppose you run:

```bash
sleep 300
```

While it is running, press:

```text
Ctrl + Z
```

Output:

```text
[1]+ Stopped sleep 300
```

The process is paused but still exists in memory.

---

# Viewing Suspended Jobs

Run:

```bash
jobs
```

Example:

```text
[1]+ Stopped sleep 300
```

---

# Resuming a Job in the Background

Resume execution:

```bash
bg %1
```

Output:

```text
[1]+ sleep 300 &
```

The job continues running in the background.

---

# Bringing a Job to the Foreground

Bring the background job back:

```bash
fg %1
```

Now the process once again controls the terminal.

---

# Killing a Background Job

You can terminate it using its PID:

```bash
kill PID
```

Or by using the job number:

```bash
kill %1
```

---

# Multiple Background Jobs

Example:

```bash
sleep 300 &
sleep 400 &
sleep 500 &
```

Display them:

```bash
jobs
```

Output:

```text
[1]- Running sleep 300 &
[2]+ Running sleep 400 &
[3]  Running sleep 500 &
```

Each job has its own job number.

---

# Practical Scenario

A system administrator starts a large backup:

```bash
tar -czf backup.tar.gz project/
```

After a few seconds, they realize they need the terminal.

Instead of canceling the command:

1. Press `Ctrl + Z`
2. Resume it:

```bash
bg
```

Now the backup continues running while the administrator performs other tasks.

Later, if they want to monitor it:

```bash
fg
```

The backup returns to the foreground.

---

# Hands-on Lab

## Step 1

Run:

```bash
sleep 300 &
```

Verify:

```bash
jobs
```

---

## Step 2

Run:

```bash
sleep 300
```

Press:

```text
Ctrl + Z
```

Verify:

```bash
jobs
```

---

## Step 3

Resume it:

```bash
bg
```

---

## Step 4

Bring it back:

```bash
fg
```

---

## Step 5

Start several background jobs:

```bash
sleep 100 &
sleep 200 &
sleep 300 &
```

View:

```bash
jobs
```

Terminate one:

```bash
kill %1
```

Verify:

```bash
jobs
```

---

# Common Mistakes

## Mistake 1

Confusing job numbers with PIDs.

Remember:

```text
[1]
```

is **not** the PID.

---

## Mistake 2

Closing the terminal.

Background jobs started this way usually terminate when the shell exits.

For jobs that must survive logout, use **`nohup`**, **`screen`**, or **`tmux`**.

---

## Mistake 3

Assuming `jobs` shows every process.

It only shows jobs started by the current shell.

---

# Best Practices

- Use background jobs for temporary long-running commands.
- Verify jobs using `jobs`.
- Use `fg` when interaction is required.
- Use `bg` after suspending a command.
- For production workloads that must continue after logout, use `nohup` or a terminal multiplexer.

---

# Interview Questions

1. What is the difference between a foreground and background process?
2. What does the `&` symbol do?
3. How do you list background jobs?
4. What does `Ctrl + Z` do?
5. What is the difference between `fg` and `bg`?
6. Why is a job number different from a PID?
7. Why shouldn't production services rely on shell background jobs?

---

# Lesson Reflection

After completing this lesson, you should be able to answer:

- What is job control?
- When should I run a process in the background?
- How do I suspend and resume a job?
- When should I use `fg` or `bg`?
- What are the limitations of shell background jobs?

---

# Summary

In this lesson, you learned how Linux job control allows commands to run in the foreground or background. You practiced starting background jobs with `&`, listing them with `jobs`, suspending execution with `Ctrl + Z`, resuming jobs with `bg`, and bringing them back to the foreground with `fg`. These commands improve productivity when working in the terminal and prepare you for more advanced process management.

---

# Cheat Sheet

```bash
# Run in background
command &

# Show jobs
jobs

# Suspend current process
Ctrl + Z

# Resume in background
bg

# Resume specific job
bg %1

# Bring to foreground
fg

# Bring specific job
fg %1

# Kill a job
kill %1
```

---

# Next Lesson

**Lesson 09 — Long-Running Commands with `nohup`, `screen`, and `tmux`**

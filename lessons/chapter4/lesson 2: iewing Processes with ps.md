# Chapter 04 — Process Management & Job Control

# Lesson 02 — Viewing Processes with `ps`

---

# Overview

In the previous lesson, you learned what a process is.

Now you will learn how to view real running processes using:

```bash
ps
```

The `ps` command shows a snapshot of processes at a specific moment.

It helps administrators answer questions like:

- What is running?
- Who started it?
- What is its PID?
- How much CPU or memory is it using?
- What command started the process?

---

# Learning Objectives

After completing this lesson, you will be able to:

- Use `ps` to view running processes.
- Understand PID, TTY, TIME, and CMD.
- View all system processes.
- Find a specific process.
- Understand parent-child process relationships.
- Use `ps` in troubleshooting workflows.

---

# Prerequisites

You should understand:

- What a process is
- PID and PPID
- Basic terminal commands
- Standard output and pipes

---

# What Does `ps` Do?

`ps` stands for:

```text
process status
```

It displays information about running processes.

Unlike `top` or `htop`, `ps` does not continuously update.

It shows a snapshot.

```text
System Processes
       │
       ▼
ps command
       │
       ▼
Process snapshot
```

---

# Basic `ps`

Run:

```bash
ps
```

Example output:

```text
PID    TTY          TIME CMD
2451   pts/0    00:00:00 bash
2510   pts/0    00:00:00 ps
```

This shows processes connected to your current terminal session.

---

# Understanding the Output

| Column | Meaning |
|--------|---------|
| PID | Process ID |
| TTY | Terminal associated with the process |
| TIME | CPU time used |
| CMD | Command that started the process |

---

# Why Does `ps` Show Itself?

When you run:

```bash
ps
```

Linux creates a process for `ps`.

So `ps` appears in its own output.

That is normal.

---

# Viewing More Detail

Use:

```bash
ps -f
```

Example:

```text
UID        PID  PPID  C STIME TTY          TIME CMD
hamad     2451  2449  0 10:05 pts/0    00:00:00 bash
hamad     2540  2451  0 10:08 pts/0    00:00:00 ps -f
```

New columns include:

| Column | Meaning |
|--------|---------|
| UID | User who owns the process |
| PPID | Parent Process ID |
| C | CPU usage indicator |
| STIME | Start time |

---

# Parent and Child Processes

Example:

```text
bash
 └── ps -f
```

Your shell starts the `ps` command.

So:

```text
bash = parent
ps = child
```

---

# Viewing All Processes

To see all processes:

```bash
ps aux
```

This is one of the most common process commands on Linux.

Example:

```text
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2 167000 12000 ?        Ss   09:00   0:01 /sbin/init
hamad     2451  0.0  0.1  10000  5000 pts/0    Ss   10:05   0:00 bash
```

---

# Understanding `ps aux`

| Column | Meaning |
|--------|---------|
| USER | Process owner |
| PID | Process ID |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| VSZ | Virtual memory size |
| RSS | Physical memory used |
| TTY | Terminal |
| STAT | Process state |
| START | Start time |
| TIME | CPU time |
| COMMAND | Full command |

---

# Common Process States

In the `STAT` column, you may see:

| State | Meaning |
|-------|---------|
| R | Running |
| S | Sleeping |
| D | Uninterruptible sleep |
| T | Stopped |
| Z | Zombie |
| Ss | Sleeping, session leader |
| R+ | Running in foreground |

Most normal processes are often in `S` state.

---

# Finding a Process with `grep`

To find a specific process:

```bash
ps aux | grep sleep
```

Example:

```text
hamad  3001  0.0  0.0  8080  1000 pts/0  S  10:20  0:00 sleep 60
hamad  3005  0.0  0.0  7000   900 pts/1  S+ 10:20  0:00 grep sleep
```

Notice that `grep sleep` appears too.

That happens because `grep` itself is a process.

---

# Avoiding the grep Line

A common trick:

```bash
ps aux | grep '[s]leep'
```

This finds `sleep` without matching the `grep` command itself.

---

# Viewing a Process by PID

If you know the PID:

```bash
ps -p 3001
```

Detailed:

```bash
ps -fp 3001
```

---

# Custom Output Columns

You can choose exactly which fields to show.

Example:

```bash
ps -eo pid,ppid,user,stat,comm
```

Output:

```text
PID   PPID USER     STAT COMMAND
1     0    root     Ss   systemd
2451  2449 hamad    Ss   bash
3001  2451 hamad    S    sleep
```

This is very useful for scripts.

---

# Sorting Processes

Sort by CPU:

```bash
ps aux --sort=-%cpu
```

Sort by memory:

```bash
ps aux --sort=-%mem
```

Show top results:

```bash
ps aux --sort=-%mem | head
```

---

# Practical Scenario

A server feels slow.

You want to know which processes are consuming the most memory.

Run:

```bash
ps aux --sort=-%mem | head
```

This shows the heaviest memory users first.

---

# Hands-on Lab

## Step 1 — Start a Test Process

Open one terminal and run:

```bash
sleep 300
```

---

## Step 2 — Open Another Terminal

Run:

```bash
ps
```

Then:

```bash
ps -f
```

---

## Step 3 — Find the Sleep Process

```bash
ps aux | grep sleep
```

---

## Step 4 — View It by PID

Use the PID from the output:

```bash
ps -fp <PID>
```

---

## Step 5 — Show Custom Columns

```bash
ps -eo pid,ppid,user,stat,comm | head
```

---

## Step 6 — Sort by Memory

```bash
ps aux --sort=-%mem | head
```

---

# Challenge

Start three processes:

```bash
sleep 100
sleep 200
sleep 300
```

Then use `ps` commands to answer:

1. What are their PIDs?
2. What is their parent process?
3. What state are they in?
4. Which terminal are they attached to?

Save your answers in:

```text
process-investigation.md
```

---

# Common Mistakes

## Mistake 1

Thinking `ps` updates live.

It does not.

Use `top` or `htop` for live monitoring.

---

## Mistake 2

Ignoring PPID.

PPID helps you understand which process started another process.

---

## Mistake 3

Confusing TIME with real elapsed time.

`TIME` shows CPU time used, not how long the process has existed.

---

# Best Practices

- Use `ps aux` for broad inspection.
- Use `ps -fp PID` for one process.
- Use custom columns for scripts.
- Sort by CPU or memory when troubleshooting.
- Always verify before killing a process.

---

# Interview Questions

1. What does `ps` show?

2. Why does `ps` appear in its own output?

3. What is the difference between PID and PPID?

4. What does `ps aux` show?

5. What does the STAT column represent?

6. How do you find a process by name?

7. How do you sort processes by memory usage?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I inspect running processes?
- Can I identify a PID and PPID?
- Can I find a specific process?
- Can I sort processes by CPU or memory?
- Can I explain why `ps` is a snapshot tool?

---

# Summary

In this lesson, you learned how to inspect Linux processes using `ps`.

You practiced viewing your current session, showing all processes, finding specific commands, inspecting PIDs, customizing output, and sorting by resource usage.

This prepares you for the next lesson, where you will monitor processes live using `top` and `htop`.

---

# Cheat Sheet

| Command | Description |
|----------|-------------|
| `ps` | Show processes in current terminal |
| `ps -f` | Full-format listing |
| `ps aux` | Show all processes |
| `ps -p PID` | Show process by PID |
| `ps -fp PID` | Full info for PID |
| `ps aux | grep name` | Find process by name |
| `ps -eo pid,ppid,user,stat,comm` | Custom process columns |
| `ps aux --sort=-%cpu` | Sort by CPU usage |
| `ps aux --sort=-%mem` | Sort by memory usage |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-04/
    └── lesson-02/
        ├── ps-basic-output.txt
        ├── ps-full-output.txt
        ├── sleep-process.txt
        ├── process-investigation.md
        ├── sorted-processes.txt
        ├── screenshots/
        └── notes.md
```

Capture:

- Output of `ps`
- Output of `ps -f`
- Output showing the `sleep` process
- Sorted memory or CPU process list
- Challenge answers
- Screenshot of the process inspection workflow

---

# Next Lesson

**Lesson 03 — Real-Time Monitoring with `top` and `htop`**

You will learn how to monitor processes live, understand CPU and memory usage, identify resource-heavy processes, and interactively manage process views.

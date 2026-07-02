# Chapter 04 — Process Management & System Monitoring

# Lesson 03 — Real-Time Monitoring with `top` & `htop`

---

# Overview

Linux systems usually run many processes at the same time.

Some processes use very little CPU or memory.

Other processes may consume a lot of resources and slow down the system.

To troubleshoot this, Linux provides real-time monitoring tools.

The two most important tools are:

- `top`
- `htop`

`top` is installed by default on most Linux systems.

`htop` is more visual and interactive, but it may need to be installed manually.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Monitor processes in real time.
- Understand the main sections of `top`.
- Read CPU and memory usage.
- Sort processes by CPU or memory.
- Identify high-resource processes.
- Use `htop` for easier process monitoring.
- Compare `top` and `htop`.

---

# Prerequisites

Before this lesson, you should understand:

- What a process is.
- What a PID is.
- How to use `ps`.
- Basic terminal navigation.

---

# Why Real-Time Monitoring Matters

The `ps` command shows a snapshot of processes.

Example:

```bash
ps aux
```

This shows what is happening at that exact moment.

But sometimes the problem changes quickly.

For example:

- A process uses high CPU for a few seconds.
- Memory usage increases slowly over time.
- A service starts and stops repeatedly.
- The system becomes slow only under load.

In these cases, you need a live view.

That is where `top` and `htop` are useful.

---

# The `top` Command

`top` shows real-time information about:

- System uptime
- Logged-in users
- Load average
- CPU usage
- Memory usage
- Running processes
- Sleeping processes
- Process IDs
- Process owners
- Process states

Start it with:

```bash
top
```

To exit:

```text
q
```

---

# Example `top` Output

```text
top - 14:20:33 up 2 hours,  1 user,  load average: 0.25, 0.31, 0.28
Tasks: 215 total,   1 running, 214 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.0 us,  2.0 sy,  0.0 ni, 93.0 id,  0.0 wa,  0.0 hi,  0.0 si
MiB Mem :   7850.0 total,   3200.0 free,   2500.0 used,   2150.0 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used

PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
1234 hamad     20   0  500000  90000  30000 R  45.0   1.1   0:10.22 firefox
2222 mysql     20   0 1200000 250000  40000 S  10.0   3.2   2:15.10 mysqld
```

---

# Understanding the Header

The top part of `top` gives system-wide information.

---

## Uptime

```text
up 2 hours
```

This means the system has been running for 2 hours without rebooting.

---

## Users

```text
1 user
```

This means one user is currently logged in.

---

## Load Average

```text
load average: 0.25, 0.31, 0.28
```

These numbers represent system load during:

- Last 1 minute
- Last 5 minutes
- Last 15 minutes

Example:

```text
0.25, 0.31, 0.28
```

This means the system is not under heavy load.

A simple rule:

If your CPU has 4 cores, then a load average around `4.00` means the CPU is fully busy.

If the load average is much higher than the number of CPU cores, the system may be overloaded.

---

# Understanding CPU Usage

Example:

```text
%Cpu(s):  5.0 us,  2.0 sy,  0.0 ni, 93.0 id,  0.0 wa
```

Important fields:

| Field | Meaning |
|---|---|
| `us` | CPU used by user processes |
| `sy` | CPU used by the kernel/system |
| `ni` | CPU used by processes with nice priority |
| `id` | Idle CPU |
| `wa` | Waiting for I/O |
| `hi` | Hardware interrupts |
| `si` | Software interrupts |
| `st` | Stolen CPU time in virtual machines |

---

## Important CPU Examples

If you see:

```text
93.0 id
```

The CPU is mostly idle.

That is usually good.

If you see:

```text
2.0 id
```

The CPU is almost fully busy.

You should investigate which process is using CPU.

If you see high:

```text
wa
```

The system may be waiting on disk I/O.

This can mean:

- Slow disk
- Heavy file operations
- Database pressure
- Backup job
- Swap usage

---

# Understanding Memory Usage

Example:

```text
MiB Mem :   7850.0 total,   3200.0 free,   2500.0 used,   2150.0 buff/cache
```

Meaning:

| Field | Meaning |
|---|---|
| `total` | Total RAM |
| `free` | Completely unused RAM |
| `used` | RAM currently used |
| `buff/cache` | RAM used for buffers and cache |

Important note:

Linux uses free memory for caching.

So high memory usage does not always mean there is a problem.

Cached memory can be released when applications need it.

---

# Understanding the Process Table

The lower part of `top` shows running processes.

Example:

```text
PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
1234 hamad     20   0  500000  90000  30000 R  45.0   1.1   0:10.22 firefox
```

| Column | Meaning |
|---|---|
| `PID` | Process ID |
| `USER` | User who owns the process |
| `PR` | Process priority |
| `NI` | Nice value |
| `VIRT` | Virtual memory |
| `RES` | Real physical memory used |
| `SHR` | Shared memory |
| `S` | Process state |
| `%CPU` | CPU usage |
| `%MEM` | Memory usage |
| `TIME+` | Total CPU time used |
| `COMMAND` | Command/process name |

---

# Process States in `top`

The `S` column shows process state.

| State | Meaning |
|---|---|
| `R` | Running |
| `S` | Sleeping |
| `D` | Uninterruptible sleep, usually waiting for I/O |
| `T` | Stopped |
| `Z` | Zombie process |

Example:

```text
R
```

The process is currently running.

Example:

```text
S
```

The process is sleeping, waiting for something.

Most processes are usually sleeping.

That is normal.

---

# Useful `top` Keyboard Shortcuts

While inside `top`, use these keys:

| Key | Action |
|---|---|
| `q` | Quit |
| `P` | Sort by CPU usage |
| `M` | Sort by memory usage |
| `T` | Sort by total CPU time |
| `k` | Kill a process |
| `r` | Renice a process |
| `u` | Filter by user |
| `1` | Show each CPU core |
| `h` | Help |
| `d` | Change refresh delay |

---

# Sorting by CPU

Inside `top`, press:

```text
P
```

This sorts processes by CPU usage.

This is useful when the system feels slow.

---

# Sorting by Memory

Inside `top`, press:

```text
M
```

This sorts processes by memory usage.

This is useful when RAM usage is high.

---

# Showing CPU Cores Individually

Inside `top`, press:

```text
1
```

Instead of seeing one combined CPU line, you will see each CPU core separately.

Example:

```text
%Cpu0  : 10.0 us, 90.0 id
%Cpu1  : 80.0 us, 20.0 id
%Cpu2  :  5.0 us, 95.0 id
%Cpu3  :  7.0 us, 93.0 id
```

This helps you understand if one core is overloaded.

---

# Filtering by User

Inside `top`, press:

```text
u
```

Then type the username.

Example:

```text
hamad
```

Now `top` shows only processes owned by that user.

---

# Killing a Process from `top`

Inside `top`, press:

```text
k
```

Then enter the PID.

Example:

```text
1234
```

Then enter the signal.

Default is usually:

```text
15
```

Signal `15` asks the process to terminate gracefully.

If the process refuses to stop, signal `9` forcefully kills it.

Use signal `9` carefully.

---

# The `htop` Command

`htop` is an improved interactive process viewer.

It is easier to read than `top`.

It provides:

- Colored CPU and memory bars
- Mouse support
- Easy sorting
- Easy searching
- Tree view
- Simple process killing
- Better navigation

---

# Installing `htop`

On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install htop
```

On Fedora:

```bash
sudo dnf install htop
```

On Arch:

```bash
sudo pacman -S htop
```

---

# Starting `htop`

Run:

```bash
htop
```

To exit:

```text
F10
```

or:

```text
q
```

---

# Useful `htop` Keys

| Key | Action |
|---|---|
| `F1` | Help |
| `F2` | Setup |
| `F3` | Search |
| `F4` | Filter |
| `F5` | Tree view |
| `F6` | Sort |
| `F7` | Increase priority |
| `F8` | Decrease priority |
| `F9` | Kill process |
| `F10` | Quit |

---

# `htop` Tree View

Press:

```text
F5
```

Tree view shows parent-child relationships.

Example:

```text
systemd
├── sshd
│   └── bash
│       └── htop
└── nginx
    ├── nginx worker
    └── nginx worker
```

This helps you understand which process started another process.

---

# Comparing `top` and `htop`

| Feature | `top` | `htop` |
|---|---|---|
| Installed by default | Usually yes | Usually no |
| Real-time monitoring | Yes | Yes |
| Easy interface | Medium | Easier |
| Colors | Limited | Yes |
| Mouse support | No | Yes |
| Tree view | Limited | Yes |
| Easy process killing | Medium | Easy |
| Best for servers | Yes | Yes |

---

# Practical Scenario

Imagine your laptop becomes slow.

You open:

```bash
top
```

Then press:

```text
P
```

You see:

```text
PID USER   %CPU COMMAND
4432 hamad 99.0 python3
```

This means `python3` is using almost one full CPU core.

You can investigate:

```bash
ps -fp 4432
```

If it is safe to stop it:

```bash
kill 4432
```

If it refuses to stop:

```bash
kill -9 4432
```

But always investigate before killing.

---

# Hands-on Lab

## Step 1 — Open `top`

Run:

```bash
top
```

Observe:

- Load average
- CPU usage
- Memory usage
- Process list

Exit with:

```text
q
```

---

## Step 2 — Sort by CPU

Run:

```bash
top
```

Press:

```text
P
```

Notice which process is using the most CPU.

---

## Step 3 — Sort by Memory

Inside `top`, press:

```text
M
```

Notice which process is using the most memory.

---

## Step 4 — Show CPU Cores

Inside `top`, press:

```text
1
```

Observe each CPU core separately.

---

## Step 5 — Create CPU Load

Open Terminal A:

```bash
yes > /dev/null
```

This command creates continuous CPU usage.

Open Terminal B:

```bash
top
```

Press:

```text
P
```

You should see `yes` near the top.

---

## Step 6 — Stop the Load

In Terminal A, press:

```text
Ctrl + C
```

Or kill it from another terminal:

```bash
pkill yes
```

---

## Step 7 — Install and Open `htop`

Run:

```bash
sudo apt install htop
htop
```

Try:

- `F3` search
- `F4` filter
- `F5` tree view
- `F6` sort
- `F10` quit

---

# Challenge

Complete the following tasks:

1. Open `top`.
2. Sort processes by CPU.
3. Sort processes by memory.
4. Show individual CPU cores.
5. Filter processes by your username.
6. Start a CPU load using `yes > /dev/null`.
7. Find the `yes` process in `top`.
8. Stop the `yes` process.
9. Install and open `htop`.
10. Enable tree view in `htop`.

---

# Common Mistakes

## Mistake 1 — Thinking Sleeping Processes Are Bad

Most processes are sleeping most of the time.

This is normal.

Sleeping means the process is waiting for work.

---

## Mistake 2 — Panicking About Used Memory

Linux uses RAM for cache.

High used memory does not always mean a problem.

Check swap usage and application behavior before deciding.

---

## Mistake 3 — Killing Processes Too Quickly

Do not kill a process just because it uses CPU.

First ask:

- What is this process?
- Who started it?
- Is it important?
- Is it safe to stop?

---

## Mistake 4 — Ignoring Load Average

CPU percentage is important, but load average shows pressure over time.

Always check both.

---

# Best Practices

- Use `top` first because it is usually installed by default.
- Use `htop` when you want easier navigation.
- Sort by CPU when the system feels slow.
- Sort by memory when RAM usage is high.
- Use `kill` carefully.
- Prefer graceful termination before force killing.
- Watch trends, not just one moment.
- Investigate the process before stopping it.

---

# Interview Questions

1. What is the difference between `ps` and `top`?
2. Why is real-time monitoring useful?
3. What does load average mean?
4. What does the `%CPU` column show?
5. What does the `%MEM` column show?
6. What is the difference between `top` and `htop`?
7. How do you sort by CPU in `top`?
8. How do you sort by memory in `top`?
9. What does process state `S` mean?
10. Why should you avoid using `kill -9` immediately?

---

# Lesson Reflection

After this lesson, you should be able to explain:

- Why `ps` is not enough for live troubleshooting.
- How `top` helps you monitor system activity.
- How to identify CPU-heavy processes.
- How to identify memory-heavy processes.
- Why `htop` is easier for interactive monitoring.
- How to safely investigate a suspicious process.

---

# Summary

In this lesson, you learned how to monitor Linux processes in real time using `top` and `htop`.

You learned how to read CPU usage, memory usage, load average, process states, and process resource consumption.

You also practiced sorting processes, filtering by user, showing CPU cores, creating test CPU load, and using `htop` for easier process monitoring.

These tools are essential for Linux troubleshooting, system administration, DevOps, and production server monitoring.

---

# Cheat Sheet

```bash
# Start top
top

# Quit top
q

# Sort by CPU inside top
P

# Sort by memory inside top
M

# Show CPU cores inside top
1

# Filter by user inside top
u

# Kill process inside top
k

# Change refresh delay inside top
d

# Install htop on Ubuntu/Debian
sudo apt update
sudo apt install htop

# Start htop
htop

# Search in htop
F3

# Filter in htop
F4

# Tree view in htop
F5

# Sort in htop
F6

# Kill process in htop
F9

# Quit htop
F10

# Create CPU load for testing
yes > /dev/null

# Stop yes process
pkill yes
```

---

# Lab Assets

Create this folder:

```text
assets/
└── chapter-04/
    └── lesson-03/
        ├── top-default.png
        ├── top-sort-cpu.png
        ├── top-sort-memory.png
        ├── top-cpu-cores.png
        ├── top-yes-process.png
        ├── htop-default.png
        ├── htop-tree-view.png
        └── notes.md
```

Capture:

- `top` default view.
- `top` sorted by CPU.
- `top` sorted by memory.
- `top` showing individual CPU cores.
- `yes > /dev/null` visible in `top`.
- `htop` default view.
- `htop` tree view.
- Your notes about what each section means.

---

# Next Lesson

Lesson 04 — Managing Process Priority with `nice` & `renice`

# Chapter 04 — Process Management & System Monitoring

# Lesson 05 — Signals, `kill`, `pkill`, `killall` & Graceful Process Termination

---

# Overview

Every running process eventually stops.

Sometimes a process finishes its work naturally.

Other times, a user closes an application.

In more serious situations, a process may freeze, become unresponsive, or consume excessive system resources.

Linux provides a mechanism called **signals** to communicate with running processes.

A signal is a software interrupt sent to a process to notify it that an event has occurred or to request that it perform a specific action, such as terminating, pausing, or reloading its configuration.

Understanding signals is essential for Linux administrators because stopping a process incorrectly can lead to data loss, corrupted files, or service interruptions.

In this lesson, you will learn how Linux signals work and how to manage processes safely using `kill`, `pkill`, and `killall`.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what a Linux signal is.
- Distinguish between graceful and forceful termination.
- List common Linux signals.
- Send signals using `kill`.
- Terminate processes by name with `pkill`.
- Terminate multiple processes using `killall`.
- List available signals.
- Apply best practices when stopping processes.

---

# Prerequisites

Before continuing, you should understand:

- Linux processes
- Process IDs (PID)
- Process monitoring with `ps`
- Real-time monitoring using `top` and `htop`

---

# Why Signals Exist

Imagine a running application.

How can Linux tell it to:

- Save its work?
- Reload its configuration?
- Pause execution?
- Resume execution?
- Shut down cleanly?
- Stop immediately?

Linux uses **signals**.

A signal is a small message sent by the kernel or another process.

The receiving process decides how to respond.

Some signals can be handled.

Some cannot.

---

# What is a Signal?

A signal is an asynchronous notification delivered to a process.

Signals allow the operating system or another process to request that a running process perform an action.

Examples include:

- Terminate
- Pause
- Continue
- Reload configuration
- Interrupt

---

# Viewing Available Signals

Run:

```bash
kill -l
```

Example output:

```text
 1) SIGHUP
 2) SIGINT
 3) SIGQUIT
 9) SIGKILL
15) SIGTERM
18) SIGCONT
19) SIGSTOP
20) SIGTSTP
```

The exact list may vary slightly between Linux distributions.

---

# Common Linux Signals

| Signal | Number | Description |
|---------|---------|-------------|
| SIGHUP | 1 | Reload configuration or terminal disconnect |
| SIGINT | 2 | Interrupt process (`Ctrl + C`) |
| SIGQUIT | 3 | Quit and create core dump |
| SIGKILL | 9 | Force immediate termination |
| SIGTERM | 15 | Graceful termination (default) |
| SIGSTOP | 19 | Pause a process (cannot be ignored) |
| SIGCONT | 18 | Resume a paused process |
| SIGTSTP | 20 | Terminal stop (`Ctrl + Z`) |

---

# Graceful vs Forceful Termination

## Graceful Termination

The process receives:

```text
SIGTERM
```

The application has an opportunity to:

- Save data.
- Close files.
- Release resources.
- Finish current work.
- Exit cleanly.

This is the preferred method.

---

## Forceful Termination

The process receives:

```text
SIGKILL
```

The kernel immediately stops the process.

The process cannot:

- Save files.
- Handle cleanup.
- Ignore the signal.

Use `SIGKILL` only when a process refuses to terminate normally.

---

# The `kill` Command

Despite its name, `kill` does not always "kill" a process.

It sends a signal.

Syntax:

```bash
kill [signal] PID
```

Example:

```bash
kill 4321
```

This sends:

```text
SIGTERM
```

because `SIGTERM` is the default signal.

---

# Sending SIGTERM

Find a process:

```bash
ps aux | grep firefox
```

Example:

```text
hamad   4321 firefox
```

Terminate gracefully:

```bash
kill 4321
```

---

# Sending SIGKILL

If the process ignores `SIGTERM`:

```bash
kill -9 4321
```

or

```bash
kill -SIGKILL 4321
```

Remember:

Only use `SIGKILL` when absolutely necessary.

---

# Sending Signals by Name

Instead of numbers:

```bash
kill -SIGTERM 4321
```

or

```bash
kill -TERM 4321
```

Both are equivalent.

---

# Pausing a Process

Send:

```bash
kill -STOP 4321
```

The process stops executing but remains in memory.

Verify:

```bash
ps -o pid,state,comm -p 4321
```

Expected state:

```text
T
```

---

# Resuming a Process

Continue execution:

```bash
kill -CONT 4321
```

The process resumes where it left off.

---

# The `pkill` Command

`pkill` sends signals using the process name instead of the PID.

Syntax:

```bash
pkill process_name
```

Example:

```bash
pkill firefox
```

This sends `SIGTERM` to every process named `firefox`.

---

# Sending SIGKILL with `pkill`

```bash
pkill -9 firefox
```

or

```bash
pkill -SIGKILL firefox
```

---

# Matching Specific Users

Kill all Firefox processes owned by a user:

```bash
pkill -u hamad firefox
```

This avoids affecting other users.

---

# The `killall` Command

`killall` also terminates processes by name.

Example:

```bash
killall firefox
```

This sends `SIGTERM` to every Firefox process.

---

# Difference Between `pkill` and `killall`

| Feature | `pkill` | `killall` |
|---------|----------|-----------|
| Match by process name | Yes | Yes |
| Match using patterns | Yes | Limited |
| Match by user | Yes | Limited |
| Flexible filtering | Excellent | Basic |

In practice:

- Use `kill` when you know the PID.
- Use `pkill` for flexible filtering.
- Use `killall` when stopping all processes with the same name.

---

# Practical Scenario

A backup script has frozen.

First locate it:

```bash
ps aux | grep backup.sh
```

Example:

```text
hamad 5521 backup.sh
```

Attempt graceful termination:

```bash
kill 5521
```

If the process is still running:

```bash
kill -9 5521
```

Always try `SIGTERM` before `SIGKILL`.

---

# Hands-on Lab

## Step 1

Open Terminal A.

Run:

```bash
sleep 300
```

---

## Step 2

Open Terminal B.

Find the process:

```bash
ps aux | grep sleep
```

Record the PID.

---

## Step 3

Terminate gracefully:

```bash
kill PID
```

Verify:

```bash
ps -p PID
```

The process should no longer exist.

---

## Step 4

Run another process:

```bash
sleep 300
```

---

## Step 5

Pause it:

```bash
kill -STOP PID
```

Check:

```bash
ps -o pid,state,comm -p PID
```

Expected:

```text
T
```

---

## Step 6

Resume it:

```bash
kill -CONT PID
```

Verify that the state returns to sleeping (`S`).

---

## Step 7

Run several sleep processes:

```bash
sleep 300 &
sleep 300 &
sleep 300 &
```

View them:

```bash
ps aux | grep sleep
```

Terminate all of them:

```bash
pkill sleep
```

or

```bash
killall sleep
```

Verify:

```bash
ps aux | grep sleep
```

Only the `grep` command should remain.

---

# Common Mistakes

## Mistake 1

Using:

```bash
kill -9
```

for every process.

Always try `SIGTERM` first.

---

## Mistake 2

Killing the wrong PID.

Always verify the process before sending signals.

---

## Mistake 3

Confusing `Ctrl + C` with `kill -9`.

`Ctrl + C` sends `SIGINT`, allowing many applications to exit cleanly.

---

## Mistake 4

Stopping critical system services accidentally.

Always check:

```bash
ps -fp PID
```

before terminating a process.

---

# Best Practices

- Use `SIGTERM` whenever possible.
- Reserve `SIGKILL` for unresponsive processes.
- Verify the process before terminating it.
- Prefer `pkill` when matching processes by name and user.
- Be cautious when using `killall` on production systems.

---

# Interview Questions

1. What is a Linux signal?
2. What is the difference between `SIGTERM` and `SIGKILL`?
3. Which signal is sent by default with `kill`?
4. Why should `SIGKILL` be used as a last resort?
5. What is the difference between `kill`, `pkill`, and `killall`?
6. Which signal is generated by pressing `Ctrl + C`?
7. How can you pause and resume a process?
8. How do you list all available signals?

---

# Lesson Reflection

After completing this lesson, you should be able to answer:

- What are Linux signals?
- When should I use `SIGTERM` instead of `SIGKILL`?
- How do I terminate a process safely?
- When should I use `kill`, `pkill`, or `killall`?
- How can I pause and resume a running process?

---

# Summary

In this lesson, you learned how Linux uses signals to communicate with running processes. You explored common signals such as `SIGTERM`, `SIGKILL`, `SIGSTOP`, and `SIGCONT`, and learned how to send them using `kill`, `pkill`, and `killall`. You also practiced terminating processes gracefully, pausing and resuming execution, and stopping multiple processes safely. Mastering signals is a fundamental skill for Linux system administration and production troubleshooting.

---

# Cheat Sheet

```bash
# List all signals
kill -l

# Gracefully terminate a process
kill PID

# Forcefully terminate a process
kill -9 PID

# Send SIGTERM explicitly
kill -SIGTERM PID

# Pause a process
kill -STOP PID

# Resume a process
kill -CONT PID

# Kill by process name
pkill process_name

# Force kill by process name
pkill -9 process_name

# Kill processes by user
pkill -u username process_name

# Kill all processes with the same name
killall process_name

# Verify process
ps -fp PID
```

---

# Lab Assets

Create the following structure:

```text
assets/
└── chapter-04/
    └── lesson-05/
        ├── kill-list-signals.png
        ├── sleep-process.png
        ├── graceful-termination.png
        ├── sigstop-process.png
        ├── sigcont-process.png
        ├── pkill-example.png
        ├── killall-example.png
        └── notes.md
```

Capture:

- Output of `kill -l`.
- `sleep` process before termination.
- Graceful termination using `kill`.
- Process paused with `SIGSTOP`.
- Process resumed with `SIGCONT`.
- Using `pkill` by process name.
- Using `killall` to terminate multiple processes.
- Your observations in `notes.md`.

---

# Next Lesson

**Lesson 06 — Managing Services with `systemctl`**

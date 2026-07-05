# Chapter 04 — Process Management & System Monitoring

# Lesson 09 — Long-Running Commands with `nohup`, `screen`, and `tmux`

---

# Overview

In the previous lesson, you learned how to run commands in the background using `&`.

Although background jobs are useful, they have an important limitation:

**They belong to the current terminal session.**

If you close the terminal, log out, or lose your SSH connection, those background jobs are usually terminated.

For short tasks, this behavior is acceptable.

However, Linux administrators often execute commands that may run for hours or even days, such as:

- System backups
- Large file transfers
- Database migrations
- Software compilation
- Data processing
- Long-running scripts

These tasks must continue running even if the administrator disconnects.

Linux provides several solutions for this problem:

- `nohup`
- `screen`
- `tmux`

Understanding when to use each tool is an essential skill for Linux administrators and DevOps engineers.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain why ordinary background jobs terminate after logout.
- Understand the purpose of `nohup`.
- Run long-running commands using `nohup`.
- Explain what terminal multiplexers are.
- Create and manage sessions using `screen`.
- Create and manage sessions using `tmux`.
- Choose the appropriate tool for different situations.

---

# Prerequisites

Before continuing, you should understand:

- Linux processes
- Background jobs (`&`)
- Job control (`jobs`, `fg`, `bg`)

---

# Why Background Jobs Are Not Enough

Suppose you execute:

```bash
tar -czf backup.tar.gz project &
```

The backup starts successfully.

Everything looks fine.

Then your SSH connection drops.

Or you accidentally close the terminal.

The shell exits.

Since the background job belongs to that shell, Linux usually terminates it.

This means your backup never finishes.

For important administrative tasks, this is unacceptable.

---

# What is `nohup`?

`nohup` stands for:

```text
No Hang Up
```

Normally, when a terminal closes, it sends a **SIGHUP** (Hang Up) signal to its child processes.

Most programs terminate when they receive this signal.

`nohup` prevents the program from reacting to **SIGHUP**, allowing it to continue running after the terminal closes.

---

# Running Commands with `nohup`

Syntax:

```bash
nohup command &
```

Example:

```bash
nohup sleep 300 &
```

Output:

```text
nohup: ignoring input and appending output to 'nohup.out'
```

The process continues running even after logout.

---

# The `nohup.out` File

Unless redirected, program output is saved in:

```text
nohup.out
```

Example:

```bash
cat nohup.out
```

You can redirect output yourself:

```bash
nohup ./backup.sh > backup.log 2>&1 &
```

Explanation:

```text
> backup.log     Standard output
2>&1             Standard error
&                Background
```

---

# Advantages of `nohup`

- Very simple.
- Built into most Linux systems.
- Ideal for one-time long-running commands.
- Survives logout.

---

# Limitations of `nohup`

You cannot reconnect to the running program.

Once the terminal is closed, you can only monitor the process indirectly using tools like:

```bash
ps
top
htop
tail
```

---

# What is `screen`?

`screen` is a terminal multiplexer.

It allows you to create virtual terminal sessions that continue running independently of your SSH connection.

Even if you disconnect, the session remains active.

Later, you can reconnect exactly where you left off.

---

# Creating a Screen Session

Start a session:

```bash
screen
```

Or give it a name:

```bash
screen -S backup
```

---

# Detaching from Screen

Leave the session without stopping it:

```text
Ctrl + A
D
```

The session continues running.

---

# Listing Screen Sessions

```bash
screen -ls
```

Example:

```text
There is a screen on:
1234.backup
```

---

# Reconnecting to Screen

```bash
screen -r backup
```

or

```bash
screen -r 1234
```

Everything appears exactly as before.

---

# What is `tmux`?

`tmux` is another terminal multiplexer.

It provides similar functionality to `screen` but includes many modern features.

Many DevOps engineers prefer `tmux`.

---

# Creating a tmux Session

```bash
tmux
```

Named session:

```bash
tmux new -s backup
```

---

# Detaching from tmux

Press:

```text
Ctrl + B
D
```

The session continues running.

---

# Listing tmux Sessions

```bash
tmux ls
```

Example:

```text
backup: 1 windows
```

---

# Reconnecting

```bash
tmux attach -t backup
```

---

# `screen` vs `tmux`

| Feature | screen | tmux |
|----------|---------|------|
| Older | ✔ | |
| Modern | | ✔ |
| Session management | ✔ | ✔ |
| Multiple windows | ✔ | ✔ |
| Better customization | | ✔ |
| Popular in DevOps | | ✔ |

Both are excellent tools.

Learning either one is valuable.

---

# When Should You Use Each Tool?

| Situation | Recommended Tool |
|-----------|------------------|
| Quick command after logout | `nohup` |
| Interactive long-running work | `screen` |
| Professional terminal workflow | `tmux` |

---

# Practical Scenario

A database migration will require approximately three hours.

You are connected through SSH.

Using:

```bash
./migrate.sh &
```

would be risky.

If your internet connection fails, the migration stops.

Instead:

```bash
tmux new -s migration
```

Run:

```bash
./migrate.sh
```

Detach:

```text
Ctrl + B
D
```

Go home.

Reconnect later:

```bash
tmux attach -t migration
```

The migration is still running.

---

# Hands-on Lab

## Step 1

Run:

```bash
nohup sleep 300 &
```

Verify:

```bash
ps aux | grep sleep
```

---

## Step 2

Install tmux (if needed):

```bash
sudo apt install tmux
```

Create:

```bash
tmux new -s practice
```

Run:

```bash
sleep 300
```

Detach:

```text
Ctrl + B
D
```

Reconnect:

```bash
tmux attach -t practice
```

---

## Step 3

Repeat the same exercise using `screen`.

---

# Common Mistakes

## Mistake 1

Thinking `&` is the same as `nohup`.

It is not.

`&` only moves a process to the background.

`nohup` protects it from logout.

---

## Mistake 2

Closing the terminal after using `&`.

Unless another mechanism is used, the process usually terminates.

---

## Mistake 3

Forgetting to detach before closing SSH when using `screen` or `tmux`.

Detach first.

---

# Best Practices

- Use `nohup` for simple background commands.
- Use `tmux` for remote administration.
- Name sessions clearly.
- Close unused sessions after finishing work.
- Never run critical production jobs using only `&`.

---

# Interview Questions

1. Why do background jobs terminate after logout?
2. What does `nohup` do?
3. What is `nohup.out`?
4. What is a terminal multiplexer?
5. What is the difference between `screen` and `tmux`?
6. When should you choose `tmux` instead of `nohup`?
7. How do you detach from a `tmux` session?
8. How do you reconnect to a running `screen` session?

---

# Lesson Reflection

After completing this lesson, you should be able to answer:

- Why isn't `&` sufficient for long-running production tasks?
- When should I use `nohup`?
- Why do administrators use `tmux`?
- How can I safely continue work after disconnecting from SSH?

---

# Summary

In this lesson, you learned how to keep long-running commands alive after logging out using `nohup`, and how terminal multiplexers such as `screen` and `tmux` allow administrators to detach and later reconnect to interactive sessions. These tools are fundamental for reliable remote Linux administration and are widely used in production environments.

---

# Cheat Sheet

```bash
# Run after logout
nohup command &

# Named screen session
screen -S session

# List screen sessions
screen -ls

# Reattach screen
screen -r session

# New tmux session
tmux new -s session

# List tmux sessions
tmux ls

# Reattach tmux
tmux attach -t session

# Detach screen
Ctrl + A D

# Detach tmux
Ctrl + B D
```

---

# Lab Assets

Create the following structure:

```text
assets/
└── chapter-04/
    └── lesson-09/
        ├── nohup-example.png
        ├── nohup-output.png
        ├── screen-session.png
        ├── tmux-session.png
        ├── tmux-detach.png
        ├── tmux-attach.png
        └── notes.md
```

Capture:

- Running a command with `nohup`.
- Contents of `nohup.out`.
- Creating and reattaching a `screen` session.
- Creating and reattaching a `tmux` session.
- Notes comparing `nohup`, `screen`, and `tmux`.

---

# Next Lesson

**Chapter 04 Final Review, Practical Lab & Mini Production Scenario**

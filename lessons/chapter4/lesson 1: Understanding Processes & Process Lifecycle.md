# Chapter 04 — Process Management & Job Control

# Lesson 01 — Understanding Processes & Process Lifecycle

---

# Overview

Every program you run on Linux becomes a process.

When you open a terminal, launch a web browser, start a database server, or execute a Bash script, Linux creates one or more processes to perform the work.

Process management is one of the operating system's primary responsibilities.

Understanding how processes are created, managed, and terminated is a fundamental skill for Linux administrators and DevOps engineers.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what a process is.
- Distinguish between a program and a process.
- Understand the Linux process lifecycle.
- Identify common process states.
- Explain the concepts of Process ID (PID) and Parent Process ID (PPID).
- Understand why process management is important.

---

# Prerequisites

Before beginning this lesson, you should understand:

- Basic Linux commands
- Bash shell
- Files and directories

---

# What Is a Program?

A program is a file stored on disk.

Examples:

```text
/bin/ls
/usr/bin/python3
/usr/bin/vim
```

A program is inactive until it is executed.

Think of a program as a recipe in a cookbook.

The recipe exists, but nothing is being cooked.

---

# What Is a Process?

A process is a running instance of a program.

For example:

```bash
ls
```

Linux performs the following steps:

```text
Program on Disk

↓

Loaded into Memory

↓

Executed

↓

Running Process
```

Once the command finishes, the process exits.

---

# Program vs Process

| Program | Process |
|----------|----------|
| Stored on disk | Running in memory |
| Passive | Active |
| Static | Dynamic |
| Can exist without running | Exists only while running |

---

# Why Processes Matter

Without processes:

- No terminal
- No web browser
- No web server
- No SSH session
- No Docker containers
- No databases

Everything you do on Linux happens inside one or more processes.

---

# The Linux Process Lifecycle

Every process follows a lifecycle.

```text
Program

↓

Created

↓

Ready

↓

Running

↓

Waiting (optional)

↓

Running Again

↓

Finished
```

Eventually the process exits and Linux releases its resources.

---

# Process Creation

Suppose you run:

```bash
nano notes.txt
```

The shell asks the Linux kernel to create a new process.

```text
Bash

↓

Kernel

↓

nano Process
```

Linux assigns this process a unique identifier.

---

# Process ID (PID)

Every process has a unique number called a Process ID.

Example:

```text
PID: 4125
```

No two running processes share the same PID.

When the process exits, its PID may eventually be reused by another process.

---

# Parent Process (PPID)

Processes are usually started by another process.

Example:

```text
Bash

↓

Python

↓

Script
```

Here:

```text
Bash

Parent Process
```

Python becomes its child.

The Python process may then create additional child processes.

---

# Parent-Child Relationship

Example:

```text
systemd (PID 1)

│

├── sshd

│      └── bash

│             └── vim

│

└── nginx
```

Linux organizes processes into a hierarchy.

---

# Process States

Processes are not always actively using the CPU.

Common states include:

| State | Meaning |
|--------|---------|
| Running | Currently executing on the CPU |
| Ready | Waiting for CPU time |
| Sleeping | Waiting for an event (input, network, disk) |
| Stopped | Paused by a signal |
| Zombie | Finished execution but waiting for its parent to collect its exit status |

Most processes spend much of their time sleeping while waiting for work.

---

# Running vs Sleeping

Imagine a web server.

Most of the time it waits for users.

```text
Sleeping

↓

User Request

↓

Running

↓

Response Sent

↓

Sleeping
```

This behavior is completely normal.

---

# Process Resources

Every process uses system resources such as:

- CPU time
- Memory (RAM)
- Open files
- Network connections
- File descriptors

One reason administrators monitor processes is to ensure these resources are being used efficiently.

---

# Practical Scenario

You log in to a Linux server using SSH.

Immediately after logging in:

- The SSH daemon is running.
- A Bash shell starts.
- Every command you execute creates a new process.

For example:

```bash
cat README.md
```

creates a process that runs briefly and then exits.

---

# Hands-on Lab

## Step 1 — Open a Terminal

Run:

```bash
echo "Hello Linux"
```

Although the command finishes quickly, Linux created a process to execute it.

---

## Step 2 — Start a Long-Running Program

Run:

```bash
sleep 60
```

This process remains active for 60 seconds.

Do not close the terminal yet.

---

## Step 3 — Open a Second Terminal

Leave the first terminal running.

You will inspect the `sleep` process in the next lesson.

---

# Challenge

For each of the following, decide whether it is a **program** or a **process**:

```text
/bin/ls

Firefox currently running

/usr/bin/python3

A Bash script executing now

/usr/bin/vim

An active SSH session
```

Explain your reasoning.

---

# Common Mistakes

## Mistake 1

Thinking a program and a process are the same thing.

A program becomes a process only when it is executed.

---

## Mistake 2

Assuming only GUI applications create processes.

Every Linux command creates one or more processes.

---

## Mistake 3

Thinking sleeping processes are "broken."

Sleeping is often the normal state for servers waiting for work.

---

# Best Practices

- Learn to identify running processes.
- Monitor long-running applications.
- Understand parent-child relationships.
- Avoid killing processes without understanding their purpose.

---

# Interview Questions

1. What is the difference between a program and a process?

2. What is a PID?

3. What is a PPID?

4. Why does Linux assign each process a unique PID?

5. What is a zombie process?

6. Why are many server processes usually sleeping?

7. Which system resources does a process consume?

---

# Lesson Reflection

Before moving on, make sure you can answer:

- Can I explain the difference between a program and a process?
- Can I describe the process lifecycle?
- Can I explain what a PID is?
- Do I understand why parent-child relationships exist?

---

# Summary

In this lesson, you learned what a process is, how Linux creates processes, how processes move through their lifecycle, and why process management is essential.

This foundation prepares you for inspecting real processes using the `ps` command in the next lesson.

---

# Cheat Sheet

| Concept | Description |
|----------|-------------|
| Program | Executable file stored on disk |
| Process | Running instance of a program |
| PID | Process ID |
| PPID | Parent Process ID |
| Running | Currently using CPU |
| Sleeping | Waiting for an event |
| Zombie | Finished process awaiting parent cleanup |

---

# Lab Assets

Store your work in:

```text
assets/
└── chapter-04/
    └── lesson-01/
        ├── process-lifecycle-diagram.txt
        ├── program-vs-process.md
        ├── challenge-answers.md
        ├── screenshots/
        └── notes.md
```

Capture:

- Your explanation of program vs process
- ASCII lifecycle diagram
- Challenge answers
- Screenshot of the `sleep 60` command running
- Personal notes

---

# Next Lesson

**Lesson 02 — Viewing Processes with `ps`**

You'll learn how to inspect real Linux processes, understand process attributes, interpret process listings, and use the `ps` command like a system administrator.

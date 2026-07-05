# Chapter 04 — Process Management & System Monitoring

# Chapter 04 — Final Review, Practical Lab & Mini Production Scenario

---

# Chapter Overview

Congratulations!

You have completed Chapter 04, where you explored one of the most important areas of Linux system administration: **process and service management**.

Throughout this chapter, you learned how Linux creates, manages, monitors, and terminates processes. You also explored how modern Linux systems use `systemd` to manage services and how administrators investigate problems using system logs.

These are daily responsibilities for Linux System Administrators, Site Reliability Engineers (SREs), and DevOps Engineers.

---

# Learning Objectives Review

After completing this chapter, you should be able to:

- Explain how Linux manages processes.
- Monitor processes using `ps`, `top`, and `htop`.
- Understand process states.
- Send signals to processes.
- Safely terminate running programs.
- Adjust process priority.
- Manage Linux services using `systemctl`.
- Investigate service failures using `journalctl`.
- Manage shell jobs.
- Keep long-running commands alive using `nohup`, `screen`, and `tmux`.

---

# Chapter Commands Review

## Process Monitoring

```bash
ps
ps aux
ps -ef
top
htop
```

---

## Process Control

```bash
kill PID
kill -9 PID
pkill process
killall process
nice
renice
```

---

## Service Management

```bash
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl reload SERVICE
systemctl enable SERVICE
systemctl disable SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE
systemctl list-units --type=service
```

---

## System Logs

```bash
journalctl
journalctl -u SERVICE
journalctl -f
journalctl -fu SERVICE
journalctl -b
journalctl -n 20
journalctl --since today
```

---

## Job Control

```bash
command &
jobs
fg
bg
Ctrl + Z
```

---

## Long Running Sessions

```bash
nohup command &
screen
screen -S session
screen -ls
screen -r session
tmux new -s session
tmux ls
tmux attach -t session
```

---

# Practical Lab

Imagine you are responsible for a production Linux server.

During today's maintenance window, you must verify that critical services are healthy while simultaneously preparing a large backup.

Your tasks are:

### Task 1

List the running services.

---

### Task 2

Verify that the SSH service is active.

---

### Task 3

Check whether SSH starts automatically after reboot.

---

### Task 4

Display the latest 20 journal entries.

---

### Task 5

Display only SSH logs.

---

### Task 6

Start a long-running command in the background.

Example:

```bash
sleep 300 &
```

Verify it with:

```bash
jobs
```

---

### Task 7

Suspend a foreground process using:

```text
Ctrl + Z
```

Resume it in the background.

---

### Task 8

Run a command using:

```bash
nohup
```

Verify that it continues running.

---

### Task 9

Create a tmux session.

Run a command.

Detach.

Reconnect.

---

# Mini Production Scenario

## Incident

Monday morning.

Developers report that they cannot access the company's web application.

You receive the following information:

```text
"The website is down."
```

Without making assumptions, investigate the system.

Your investigation should follow this order:

1. Verify the web service status.
2. Read the service logs.
3. Determine whether the service failed.
4. Identify the root cause.
5. Restart or reload the service if appropriate.
6. Verify that users can access the application again.
7. Ensure the service is enabled for future boots.

This workflow reflects how experienced Linux administrators troubleshoot production systems.

---

# Common Mistakes

- Restarting services before checking logs.
- Using `kill -9` unnecessarily.
- Forgetting to enable important services after installation.
- Confusing **Active** with **Enabled**.
- Running critical production jobs with only `&`.
- Ignoring journal logs during troubleshooting.

---

# Best Practices

- Investigate before making changes.
- Read logs before restarting services.
- Prefer graceful termination (`SIGTERM`) over forceful termination (`SIGKILL`).
- Use `reload` instead of `restart` when supported.
- Use `tmux` for long administrative sessions.
- Keep important services enabled after installation.
- Verify every administrative action.

---

# Chapter Reflection

Ask yourself the following questions:

- Can I explain the lifecycle of a Linux process?
- Can I monitor resource usage efficiently?
- Do I know when to use `kill`, `pkill`, or `killall`?
- Can I troubleshoot a failed service using `systemctl` and `journalctl`?
- Do I understand the difference between a foreground job and a background job?
- Can I safely execute long-running commands over SSH?

If you can confidently answer these questions, you are ready to move to the next chapter.

---

# Interview Questions

1. What is the difference between a process and a service?
2. Why is `systemd` important?
3. Explain the difference between `restart` and `reload`.
4. What is the purpose of `journalctl`?
5. What is the difference between `SIGTERM` and `SIGKILL`?
6. Why is `tmux` commonly used by DevOps engineers?
7. Explain the difference between `Active` and `Enabled`.
8. Why shouldn't production jobs rely on `&` alone?
9. What would you check first if a service fails to start?
10. How do you investigate a production outage without making assumptions?

---

# Chapter Summary

Chapter 04 introduced the core concepts of Linux process and service management. You learned how to inspect and control running processes, adjust priorities, manage services using `systemctl`, investigate failures through `journalctl`, control jobs from the shell, and maintain long-running tasks with `nohup`, `screen`, and `tmux`.

These skills form the operational foundation required for system administration and DevOps. Nearly every production Linux environment relies on the concepts covered in this chapter.

---

# Chapter 04 Cheat Sheet

```bash
# Process Monitoring
ps
ps aux
top
htop

# Process Management
kill PID
kill -9 PID
pkill process
killall process
nice
renice

# Service Management
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl reload SERVICE
systemctl enable SERVICE
systemctl disable SERVICE
systemctl is-active SERVICE
systemctl is-enabled SERVICE

# Logs
journalctl
journalctl -u SERVICE
journalctl -fu SERVICE
journalctl -b

# Job Control
command &
jobs
fg
bg

# Long Running Sessions
nohup command &
screen
tmux
```

---

# Lab Assets

Create the following structure:

```text
assets/
└── chapter-04/
    ├── lesson-06/
    ├── lesson-07/
    ├── lesson-08/
    ├── lesson-09/
    └── final-lab/
        ├── process-monitoring.png
        ├── service-status.png
        ├── journalctl-logs.png
        ├── background-jobs.png
        ├── tmux-session.png
        ├── production-scenario-notes.md
        └── chapter-04-summary.md
```

Capture screenshots and notes for each major topic, along with your investigation steps for the production scenario.

---

# What's Next?

**Chapter 05 — User Management, Groups & Permissions**

In the next chapter, you'll learn how Linux manages users and groups, control file ownership and permissions in depth, understand authentication basics, and apply security best practices. These concepts are fundamental for administering multi-user Linux systems and are heavily used in real-world server environments.

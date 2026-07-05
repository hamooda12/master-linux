# Chapter 04 — Process Management & System Monitoring

# Lesson 06 — Managing Services with `systemctl`

---

# Overview

Not every program on a Linux system is started manually by a user.

Many programs run continuously in the background, providing essential functionality such as remote access, web hosting, databases, scheduled tasks, logging, and networking.

These long-running background programs are known as **services** (also called **daemons**).

Modern Linux distributions use **systemd** as the system and service manager. It is responsible for starting services during boot, stopping them during shutdown, monitoring their health, and allowing administrators to manage them efficiently.

As a Linux administrator, managing services is one of the most common daily tasks. Whether you are deploying a web server, troubleshooting Docker, restarting a database, or verifying that SSH is running, you will frequently interact with services using the `systemctl` command.

In this lesson, you will learn what Linux services are, how `systemd` manages them, and how to control them using `systemctl`.

---

# Learning Objectives

After completing this lesson, you will be able to:

- Explain what a Linux service is.
- Understand the role of `systemd`.
- Distinguish between a regular process and a service.
- View the status of system services.
- Start, stop, restart, and reload services.
- Enable and disable services at boot.
- Verify whether a service is active or enabled.
- List installed and running services.
- Apply best practices when managing production services.

---

# Prerequisites

Before continuing, you should understand:

- Linux processes
- Process monitoring with `ps`, `top`, and `htop`
- Linux signals (`kill`, `pkill`, `killall`)

---

# Why Services Exist

Imagine turning on a Linux server.

Even before a user logs in, many important programs are already running.

For example:

- The SSH server waits for remote administrators.
- The web server waits for incoming HTTP requests.
- Docker waits to manage containers.
- The scheduler waits to execute cron jobs.

These programs are not opened manually every time the system starts.

Instead, Linux starts them automatically as **services**.

Without services, a server would require an administrator to manually start every essential application after each reboot.

---

# What is a Service?

A **service** is a long-running background program that provides functionality to the operating system or other applications.

Unlike interactive applications, services usually run without direct user interaction.

Examples include:

- SSH server
- Docker Engine
- NGINX
- Apache
- MySQL
- Cron

Services often start automatically during system boot and continue running until they are stopped or the system shuts down.

---

# Process vs Service

Although every service is a process, not every process is a service.

| Process | Service |
|----------|----------|
| Started by a user or another process | Usually started by `systemd` |
| May be temporary | Usually long-running |
| Often interactive | Runs in the background |
| Ends when its task finishes | Continues waiting to provide a service |

Example:

```text
Firefox → Process

Docker → Service

SSH Server → Service

sleep 60 → Process
```

---

# What is `systemd`?

`systemd` is the system and service manager used by most modern Linux distributions.

It is the first userspace process started by the Linux kernel (PID 1).

Its responsibilities include:

- Starting services during boot.
- Stopping services during shutdown.
- Monitoring service health.
- Restarting failed services when configured.
- Managing dependencies between services.
- Recording service status and logs.

You can think of `systemd` as the manager responsible for coordinating all system services.

```text
Linux Kernel
      │
      ▼
 systemd (PID 1)
      │
 ├── ssh.service
 ├── docker.service
 ├── cron.service
 ├── nginx.service
 └── mysql.service
```

---
# The `systemctl` Command

`systemctl` is the primary command-line utility for managing services on Linux systems that use **systemd**.

Its name stands for **System Control**.

Syntax:

```bash
systemctl [command] service_name
```

Example:

```bash
systemctl status ssh
```

This command requests information about the SSH service from `systemd`.

---

# Viewing Service Status

The most frequently used command is:

```bash
systemctl status service_name
```

Example:

```bash
systemctl status ssh
```

Example output:

```text
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
     Active: active (running) since Mon 2026-07-06 08:15:42
       Docs: man:sshd(8)
   Main PID: 845
      Tasks: 1
     Memory: 6.2M
        CPU: 42ms
```

This output provides valuable information, including:

- Whether the service is running.
- Whether it starts automatically at boot.
- The main process ID (PID).
- Memory usage.
- CPU usage.
- The time the service started.
- Recent log messages.

---

# Understanding the Status Output

When reading `systemctl status`, pay attention to these fields:

### Loaded

```text
Loaded: loaded
```

Indicates whether the service definition (unit file) has been successfully loaded by `systemd`.

It also shows whether the service is enabled to start automatically during boot.

---

### Active

Example:

```text
Active: active (running)
```

Possible values include:

```text
active (running)
inactive (dead)
failed
activating
deactivating
```

This is usually the first line administrators check when troubleshooting.

---

### Main PID

```text
Main PID: 845
```

Every running service has one main process.

This PID can be inspected using other Linux tools such as:

```bash
ps -fp 845
```

---

# Starting a Service

To start a stopped service:

```bash
systemctl start service_name
```

Example:

```bash
sudo systemctl start nginx
```

If the service starts successfully, no output is normally displayed.

Always verify:

```bash
systemctl status nginx
```

---

# Stopping a Service

To stop a running service:

```bash
sudo systemctl stop nginx
```

After stopping it:

```bash
systemctl status nginx
```

Expected:

```text
Active: inactive (dead)
```

---

# Restarting a Service

Many configuration changes require restarting the service.

Syntax:

```bash
sudo systemctl restart service_name
```

Example:

```bash
sudo systemctl restart ssh
```

Restart performs two actions:

1. Stops the service.
2. Starts it again.

---

# Reloading a Service

Some services can reload their configuration without restarting.

```bash
sudo systemctl reload nginx
```

Reload tells the application:

> "Read your configuration files again."

The service continues running while applying the new configuration.

---

# Restart vs Reload

| Restart | Reload |
|----------|---------|
| Stops and starts the service | Keeps the service running |
| Existing connections may be interrupted | Existing connections usually remain active |
| Required after major changes | Used for configuration changes when supported |

Whenever possible, production systems prefer **reload** because it minimizes downtime.

---

# Enabling a Service

Starting a service does **not** mean it will automatically start after reboot.

To enable automatic startup:

```bash
sudo systemctl enable nginx
```

This creates the necessary symbolic links so that `systemd` starts the service during boot.

---

# Disabling a Service

To prevent a service from starting automatically:

```bash
sudo systemctl disable nginx
```

Disabling a service does **not** stop it immediately.

It only affects future boots.

---

# Active vs Enabled

These two concepts are commonly confused.

| Active | Enabled |
|----------|----------|
| Running right now | Starts automatically during boot |

Examples:

Service is running but not enabled:

```text
Active: Yes
Enabled: No
```

If the system reboots, the service will not start automatically.

Another example:

```text
Active: No
Enabled: Yes
```

The service is currently stopped, but it will start during the next system boot.

---

# Checking if a Service is Running

```bash
systemctl is-active nginx
```

Possible output:

```text
active
```

or

```text
inactive
```

This command is commonly used in automation scripts.

---

# Checking if a Service Starts at Boot

```bash
systemctl is-enabled nginx
```

Possible output:

```text
enabled
disabled
static
masked
```

---

# Listing Running Services

Display currently active services:

```bash
systemctl list-units --type=service
```

---

# Listing Installed Services

To display all installed service unit files:

```bash
systemctl list-unit-files --type=service
```

This includes services that are currently stopped.

---

# Practical Scenario

A user reports that the company website is unavailable.

As the Linux administrator, you suspect that the web server has stopped.

Investigation:

```bash
systemctl status nginx
```

If the output shows:

```text
Active: inactive (dead)
```

Start the service:

```bash
sudo systemctl start nginx
```

Verify:

```bash
systemctl status nginx
```

Finally, ensure it survives future reboots:

```bash
sudo systemctl enable nginx
```

---

# Hands-on Lab

## Step 1

List all running services:

```bash
systemctl list-units --type=service
```

---

## Step 2

Check the status of the SSH service:

```bash
systemctl status ssh
```

---

## Step 3

Verify whether SSH starts automatically:

```bash
systemctl is-enabled ssh
```

---

## Step 4

Check whether SSH is currently running:

```bash
systemctl is-active ssh
```

---

## Step 5

Choose a non-critical service on your system and:

- Stop it.
- Start it again.
- Restart it.
- If supported, reload it.

Always verify the status after each operation.

---

# Common Mistakes

## Mistake 1

Confusing **restart** with **reload**.

Restart stops the service completely.

Reload only refreshes its configuration.

---

## Mistake 2

Assuming that starting a service automatically enables it at boot.

Starting and enabling are separate actions.

---

## Mistake 3

Stopping critical production services without verifying dependencies.

Always understand the impact before stopping a service.

---

## Mistake 4

Ignoring the output of `systemctl status`.

The status output often contains the information needed to identify the root cause of a problem.

---

# Best Practices

- Always check the service status before restarting it.
- Prefer `reload` over `restart` when supported.
- Verify that important services are enabled after installation.
- Confirm changes using `systemctl status`.
- Avoid restarting critical production services during peak usage unless necessary.

---

# Interview Questions

1. What is a Linux service?
2. What is the role of `systemd`?
3. What is the purpose of `systemctl`?
4. What is the difference between `restart` and `reload`?
5. Explain the difference between an active service and an enabled service.
6. How do you check if a service is running?
7. How do you make a service start automatically after boot?
8. What information does `systemctl status` provide?

---

# Lesson Reflection

After completing this lesson, you should be able to answer:

- What is a service?
- Why does Linux use `systemd`?
- How do I manage services with `systemctl`?
- When should I restart a service?
- When is reloading a better choice?
- What is the difference between an active service and an enabled service?

---

# Summary

In this lesson, you learned how Linux manages long-running background services using `systemd` and the `systemctl` command. You explored how to inspect service status, start and stop services, restart or reload configurations, enable services at boot, and verify whether services are currently active or configured to start automatically. These are essential skills for Linux system administration and everyday DevOps operations.

---

# Cheat Sheet

```bash
# View service status
systemctl status SERVICE

# Start a service
sudo systemctl start SERVICE

# Stop a service
sudo systemctl stop SERVICE

# Restart a service
sudo systemctl restart SERVICE

# Reload configuration
sudo systemctl reload SERVICE

# Enable service at boot
sudo systemctl enable SERVICE

# Disable service at boot
sudo systemctl disable SERVICE

# Check if service is running
systemctl is-active SERVICE

# Check if service starts at boot
systemctl is-enabled SERVICE

# List running services
systemctl list-units --type=service

# List installed service files
systemctl list-unit-files --type=service
```

---

# Lab Assets

Create the following structure:

```text
assets/
└── chapter-04/
    └── lesson-06/
        ├── systemctl-status.png
        ├── systemctl-start.png
        ├── systemctl-stop.png
        ├── systemctl-restart.png
        ├── systemctl-reload.png
        ├── systemctl-enable.png
        ├── systemctl-disable.png
        ├── active-vs-enabled.md
        └── notes.md
```

Capture:

- Output of `systemctl status`.
- Starting and stopping a service.
- Restarting and reloading a service.
- Enabling and disabling a service.
- The difference between `is-active` and `is-enabled`.
- Your observations in `notes.md`.

---

# Next Lesson

**Lesson 07 — Viewing and Analyzing Service Logs with `journalctl`**



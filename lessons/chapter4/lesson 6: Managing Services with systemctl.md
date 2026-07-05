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

# What is `systemctl`?

`systemctl` is the command-line interface used to communicate with `systemd`.

Whenever you want to start, stop, restart, enable, disable, or inspect a service, you use `systemctl`.

Syntax:

```bash
systemctl [command] service_name
```

Example:

```bash
systemctl status ssh
```

This command asks `systemd` to display detailed information about the SSH service.

---

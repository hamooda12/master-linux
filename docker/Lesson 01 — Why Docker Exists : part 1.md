# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 01 — Why Docker Exists

## Part 1 — The Deployment Problem and “Works on My Machine”

---

## Overview

Docker was not created simply to give engineers another way to start applications. It was created because moving software between environments was unreliable, slow, and expensive.

An application could work perfectly on a developer's laptop and fail immediately on a test or production server. The source code might be identical, but the environment surrounding it was different.

This lesson begins before Docker existed. We will examine the deployment problems engineers faced, identify exactly what “works on my machine” means, and establish the requirements that eventually led to containers.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain why identical source code can behave differently across machines.
- Define the deployment environment of an application.
- Distinguish application code from application dependencies and configuration.
- Explain dependency conflicts and configuration drift.
- Describe the traditional application deployment process.
- Investigate a deployment failure using evidence instead of assumptions.
- Explain what engineers needed from a better deployment system.

---

## Prerequisites

You should already understand:

- Linux files and directories
- Processes and services
- Ports and sockets
- Environment variables
- Basic networking and DNS
- Package installation with a Linux package manager

You do not need previous Docker experience.

---

# 1. Why This Topic Exists

Imagine that a developer builds a Spring Boot application on a laptop.

The application works correctly:

```text
Developer laptop

Java 21
MySQL 8
Redis 7
APP_PORT=8080
Application starts successfully
```

The same application is copied to a production server:

```text
Production server

Java 17
MySQL 5.7
Redis is unavailable
APP_PORT is missing
Port 8080 is already occupied
Application fails
```

The source code did not necessarily change. The environment changed.

This produces the famous sentence:

> “It works on my machine.”

This sentence does not prove that the application is production-ready. It proves only that one combination of code, dependencies, configuration, operating-system behavior, and external services worked on one machine.

---

# 2. The Big Picture

An application is more than its source code.

```text
Application
│
├── Source code
├── Runtime
├── Libraries and packages
├── System tools
├── Configuration
├── Environment variables
├── Files and permissions
├── Network access
└── External services
```

A deployment succeeds only when the destination environment provides everything the application expects.

## Traditional delivery model

```text
Developer machine
      │
      │ source code or build artifact
      ▼
Deployment instructions
      │
      ▼
Test server
      │
      ▼
Production server
```

Only the code or compiled artifact was commonly transferred. Engineers then attempted to reproduce the required environment by installing and configuring software on each destination server.

## The consistency gap

```text
Code transferred successfully
            │
            ▼
Environment reproduced imperfectly
            │
            ▼
Different behavior in production
```

Docker's eventual goal was to reduce this gap by packaging the application together with much of the environment it needs.

---

# 3. What Is an Application Environment?

The environment is everything outside the application code that can affect how the application runs.

## 3.1 Runtime

The runtime executes the application.

Examples:

- Java Runtime Environment
- Python interpreter
- Node.js
- .NET runtime
- PHP runtime

An application written for Python 3.12 may fail on Python 3.8. A Java class compiled for a newer Java version may not run on an older runtime.

## 3.2 Application dependencies

Applications use libraries rather than implementing everything from zero.

Examples:

- Java JAR dependencies
- Python packages installed with `pip`
- Node.js packages installed with `npm`
- Native shared libraries such as OpenSSL

The correct package name is not always enough. The version can change behavior, available functions, security, or compatibility.

## 3.3 Operating-system dependencies

An application may require:

- A Linux distribution or particular system library
- Commands such as `curl`, `ffmpeg`, or `openssl`
- Certificate authorities
- A specific timezone database
- A system user or group
- Particular file permissions

## 3.4 Configuration

Configuration tells the same application how to behave in a particular environment.

Examples:

```text
SERVER_PORT=8080
DATABASE_HOST=mysql.internal
DATABASE_NAME=store
LOG_LEVEL=INFO
```

Configuration should normally change between development, testing, and production. The application artifact should not need to be rebuilt merely to change the database hostname.

## 3.5 External services

The application may depend on:

- Databases
- Caches
- Message brokers
- DNS
- Authentication services
- Object storage
- External APIs

The application can be correctly installed and still fail because one of these services is unreachable or incompatible.

---

# 4. Why “Works on My Machine” Happens

## 4.1 Version mismatch

```text
Developer:   Node.js 22
Production:  Node.js 18
```

The application uses a feature supported by Node.js 22 but unavailable in Node.js 18.

## 4.2 Missing dependency

The developer installed a required package several months ago and forgot that it was needed. The production server does not contain it.

## 4.3 Different configuration

```text
Developer:   DATABASE_HOST=localhost
Production:  DATABASE_HOST is missing
```

On the developer's laptop, a local database is running. On production, `localhost` refers to the production server itself, where no database is listening.

## 4.4 Different file permissions

The developer owns the application's directory and can write logs there. The production application runs as a restricted service account that cannot write to the same path.

## 4.5 Port conflict

The application expects port 8080, but another production process already owns it.

## 4.6 Configuration drift

Configuration drift occurs when machines that were supposed to be equivalent gradually become different.

```text
Server A                  Server B
────────                  ────────
Package updated           Old package
Config manually changed   Original config
New certificate           Expired certificate
Service restarted         Service still old
```

Manual changes, emergency fixes, incomplete automation, and different update histories all create drift.

---

# 5. Dependency Conflict

Suppose two applications share one server:

```text
Server
│
├── Payroll application → requires Python 3.10
└── Reporting service   → requires Python 3.12
```

Upgrading the system runtime for the reporting service might break payroll. Keeping the older runtime might prevent the reporting service from starting.

This is sometimes called dependency hell: multiple applications require incompatible versions of the same runtime or library.

Engineers attempted to solve this with:

- Separate physical servers
- Virtual machines
- Language-specific virtual environments
- Carefully managed packages
- Configuration-management tools

These approaches helped, but each had trade-offs. We will compare virtual machines and containers in Lesson 02.

---

# 6. Traditional Deployment Workflow

Before a traditional application could run, an operations engineer might need to:

1. Provision a server.
2. Install the correct operating-system packages.
3. Install the required runtime.
4. Create users and directories.
5. Copy the application artifact.
6. Install dependencies.
7. Create configuration files.
8. Configure permissions.
9. Configure networking and firewall rules.
10. Create a systemd service.
11. Start the application.
12. Investigate differences from development.

## Internal workflow

```text
Application artifact arrives
            │
            ▼
Engineer reads deployment document
            │
            ▼
Server is modified in place
            │
            ▼
Dependencies are installed
            │
            ▼
Configuration is created
            │
            ▼
Application is started
            │
       ┌────┴────┐
       │         │
    Success    Failure
                 │
                 ▼
       Compare environments manually
```

The problem was not that this workflow could never succeed. The problem was that it was difficult to reproduce reliably and repeatedly at scale.

---

# 7. Why Deployment Documentation Was Not Enough

A team might create instructions such as:

```text
1. Install Java.
2. Copy application.jar.
3. Set the database address.
4. Start the application.
```

Important questions remain unanswered:

- Which Java distribution and exact version?
- Which operating-system packages are required?
- Which user should run the process?
- Which directory should be the working directory?
- What permissions are required?
- Which environment variables are mandatory?
- What should happen when the process crashes?
- Which ports must be available?

Documentation can also become outdated. A human may skip a step, interpret it differently, or apply an emergency modification that is never added to the document.

The industry needed the environment to become executable and reproducible—not merely described in prose.

---

# 8. What Engineers Needed

A better deployment system needed to provide:

## Reproducibility

The same declared application environment should be created repeatedly.

## Isolation

Applications should not casually overwrite or conflict with one another's dependencies.

## Portability

The packaged application should run consistently across compatible developer laptops, test systems, and production hosts.

## Speed

Starting an isolated application environment should take seconds, not the time required to boot and prepare an entire server.

## Versioning

Teams should be able to identify exactly which application package was deployed and roll back to an earlier version.

## Automation

The package should fit naturally into CI/CD pipelines.

## Efficient resource usage

Multiple isolated applications should share infrastructure without each requiring a complete guest operating system.

These requirements led to modern container workflows. Docker made those workflows accessible and consistent for development and operations teams.

---

# 9. The Container Idea — First Look

The central idea is:

> Package the application and its required user-space environment into a standardized image, then run that image as an isolated process called a container.

```text
Traditional deployment

Application artifact + human instructions + preconfigured server


Containerized deployment

Application image + runtime configuration + container host
```

Docker does not package a complete hardware machine, and a Linux container does not normally include its own Linux kernel. Containers share the host kernel while isolating processes and resources. We will examine this deeply in Lesson 02.

## Important limitation

Containers improve consistency, but they do not magically solve every operational problem.

You must still manage:

- Runtime configuration
- Secrets
- Persistent data
- Networks
- Security
- Monitoring
- Resource limits
- Image vulnerabilities
- Compatibility with the host kernel and CPU architecture

Docker creates a strong deployment boundary; it does not remove the need for engineering.

---

# 10. Commands for Investigating Environment Differences

This lesson does not require Docker commands yet. First, we use Linux evidence to prove why a deployment differs.

## Check operating-system information

```bash
cat /etc/os-release
```

### What it does

Reads the operating system's identification data, including distribution name and version.

### When to use it

Use it when comparing development and production hosts or checking package compatibility.

---

## Check the kernel

```bash
uname -r
```

### What it does

Displays the running kernel release.

### Why it matters

Containers share the host kernel. Kernel capabilities therefore matter even when application user-space files come from an image.

---

## Check a runtime version

```bash
java -version
python3 --version
node --version
```

### What happens internally

The shell searches the directories listed in `PATH`, finds the executable, starts it, and asks it to print version information.

### Common mistake

Checking only the version number without checking which executable was selected. Several versions may be installed.

```bash
command -v python3
readlink -f "$(command -v python3)"
```

---

## Check environment variables

```bash
printenv
printenv DATABASE_HOST
```

### What it does

Displays environment variables inherited by the current process.

### Important limitation

Your interactive shell and a systemd service may receive different environments. Evidence from your shell does not automatically prove what the service received.

For a running process, inspect carefully with appropriate permissions:

```bash
tr '\0' '\n' < /proc/PROCESS_ID/environ
```

Do not expose secrets from a production process in tickets or public logs.

---

## Check listening ports

```bash
sudo ss -ltnp
```

### Option meaning

- `-l`: listening sockets
- `-t`: TCP sockets
- `-n`: numeric ports and addresses
- `-p`: owning processes

### When to use it

Use it to determine whether the expected port is already occupied or whether the application actually started listening.

---

## Check shared-library dependencies

```bash
ldd /path/to/binary
```

### What it does

Shows the shared libraries required by a dynamically linked executable and where they resolve.

### Warning

Do not run `ldd` casually against an untrusted executable. Safer static inspection tools may be appropriate during security analysis.

---

# 11. Hands-on Lab — Prove the Environment Matters

## Scenario

A script works when a developer starts it manually, but fails when another engineer runs it from a different shell.

## Step 1 — Create the application

Create `environment-demo.sh`:

```bash
#!/usr/bin/env bash

set -u

echo "Application starting"
echo "Runtime: $BASH_VERSION"
echo "Environment: $APP_ENV"
echo "Port: $APP_PORT"
echo "Application started successfully"
```

Make it executable:

```bash
chmod +x environment-demo.sh
```

## Step 2 — Run it without configuration

```bash
./environment-demo.sh
```

Expected result:

```text
Application starting
./environment-demo.sh: line ...: APP_ENV: unbound variable
```

The code exists and Bash exists, but the required environment is incomplete.

## Step 3 — Supply the required environment

```bash
APP_ENV=development APP_PORT=8080 ./environment-demo.sh
```

Expected result:

```text
Application starting
Runtime: ...
Environment: development
Port: 8080
Application started successfully
```

## Step 4 — Make only part of the configuration available

```bash
export APP_ENV=production
unset APP_PORT
./environment-demo.sh
```

The application still fails. One correct variable does not compensate for another missing requirement.

## Step 5 — Record evidence

```bash
printenv APP_ENV
printenv APP_PORT
command -v bash
bash --version
```

## Lab conclusion

The source file remained unchanged in every test. Behavior changed because the process environment changed.

Docker later allows us to package the script and its user-space dependencies into an image. Runtime-specific values can then be supplied deliberately when a container starts.

---

# 12. Production Scenario

## Incident

Developers report:

> “The payment API works locally, but it crashes immediately on the production server.”

The application log contains:

```text
UnsupportedClassVersionError
```

Development uses Java 21. Production uses Java 17.

## Investigation mindset

Do not begin by reinstalling random packages or blaming the network.

Gather evidence:

```bash
java -version
command -v java
readlink -f "$(command -v java)"
systemctl cat payment-api
systemctl status payment-api
journalctl -u payment-api -n 100 --no-pager
```

## Root cause

The application was compiled for a newer Java class-file version than the production runtime supports.

## Traditional correction

Install and select a compatible Java runtime, update the service configuration if necessary, restart the service, and verify it.

## Container-oriented direction

Build the application into an image whose runtime stage contains the required Java version. Test that exact image and deploy the same image to production.

This does not mean configuration disappears. Database addresses, credentials, and environment-specific policies should still be supplied securely at runtime.

---

# 13. Common Mistakes

## Mistake 1 — Treating source code as the complete application

The application also depends on its runtime, libraries, configuration, permissions, network, and external services.

## Mistake 2 — Installing the newest dependency without checking compatibility

“Newest” and “required” are not the same thing. Use tested, explicitly declared versions.

## Mistake 3 — Fixing production manually without recording the change

Undocumented changes increase configuration drift and make the next deployment less predictable.

## Mistake 4 — Assuming identical Linux distributions are identical environments

Package versions, configuration, users, permissions, kernel versions, and active services may still differ.

## Mistake 5 — Believing Docker packages an entire virtual machine

Containers generally share the host kernel. Images package application user-space files, not a complete independent hardware system.

## Mistake 6 — Believing Docker automatically makes an application secure

An insecure application inside a poorly configured container remains insecure.

## Mistake 7 — Putting environment-specific secrets into an image

Images are commonly copied to registries and cached on hosts. Secrets require a separate secure lifecycle.

---

# 14. Best Practices

- Declare dependencies and pin deliberate versions.
- Build once, then promote the same artifact or image through environments.
- Keep environment-specific configuration outside the application build.
- Automate environment creation instead of relying on undocumented manual steps.
- Treat deployment definitions as version-controlled code.
- Collect evidence before modifying a failed environment.
- Record image versions and avoid ambiguous production releases.
- Keep secrets out of source code and image layers.
- Verify deployment with health and functional checks, not only a running process.

## Important correction to a common Docker rule

You may hear:

> “Always use Alpine because it is smaller.”

Smaller is useful, but Alpine is not automatically the best base image. It uses `musl` instead of `glibc`, which can cause compatibility or debugging differences for some applications. Choose the smallest base image that is compatible, maintainable, secure, and operationally appropriate.

---

# 15. Interview Questions

## Junior

### 1. What does “works on my machine” mean?

It means the application succeeded in one particular environment, while another environment may differ in runtime, dependencies, configuration, permissions, networking, or services.

### 2. What is configuration drift?

It is the gradual development of differences between machines that were intended to have equivalent configurations.

### 3. Why is application code alone insufficient for deployment?

The code requires a compatible runtime, libraries, configuration, permissions, and supporting services.

## Intermediate

### 4. How do containers reduce environment inconsistency?

They run applications from versioned images that package a declared user-space filesystem and application dependencies, reducing dependence on packages installed directly on each host.

### 5. Should configuration be built into the image?

Stable defaults may be included, but environment-specific configuration and secrets should normally be supplied at runtime so the same image can be promoted across environments.

### 6. Why might an application still behave differently in two containers created from the same image?

They may receive different configuration, secrets, volumes, networks, resource limits, host kernels, CPU architectures, or external-service responses.

## Senior

### 7. Does containerization eliminate configuration drift?

It greatly reduces drift inside the packaged user-space environment, but drift can still exist in host kernels, Docker configuration, runtime options, secrets, volumes, networks, infrastructure, and external dependencies.

### 8. Why is “build once, promote the same image” important?

Rebuilding separately for each environment can produce different bytes because dependencies, timestamps, base images, or build systems may change. Promoting the same tested image preserves artifact identity.

### 9. What is the boundary of portability for a container image?

The image must be compatible with the runtime, host kernel capabilities, and CPU architecture. Portability is strong across compatible hosts, not unlimited across every platform.

---

# 16. Lesson Reflection

Before continuing, explain these ideas in your own words:

1. Why can identical source code behave differently on two servers?
2. What parts of an application environment exist outside the source code?
3. Why does manual server configuration create long-term risk?
4. What does containerization package, and what remains external?
5. Why does Docker reduce—but not completely eliminate—environment differences?

---

# 17. Summary

- Docker arose from real deployment inconsistency and dependency-conflict problems.
- An application is code plus runtime, dependencies, configuration, permissions, networking, and external services.
- “Works on my machine” means only that one specific environment satisfied the application's requirements.
- Traditional deployments modified servers in place and were vulnerable to manual error and configuration drift.
- Containers package an application's user-space environment into a versioned, repeatable image.
- Containers share the host kernel and still depend on runtime configuration and infrastructure.
- Docker improves reproducibility, isolation, portability, speed, and automation, but it does not replace careful operations and security.

---

# 18. Cheat Sheet

| Command | Purpose |
|---|---|
| `cat /etc/os-release` | Identify the Linux distribution and version |
| `uname -r` | Show the running kernel release |
| `java -version` | Check the active Java runtime |
| `python3 --version` | Check the active Python runtime |
| `node --version` | Check the active Node.js runtime |
| `command -v NAME` | Show which executable the shell selects |
| `readlink -f PATH` | Resolve the final path through symbolic links |
| `printenv` | Display environment variables |
| `printenv NAME` | Display one environment variable |
| `sudo ss -ltnp` | Show listening TCP ports and their processes |
| `ldd BINARY` | Inspect dynamically linked library dependencies |
| `systemctl cat SERVICE` | Display the effective systemd unit definition |
| `journalctl -u SERVICE` | Read logs for a systemd service |

---

# 19. Challenge — Diagnose Without Docker

An API works on a developer laptop but fails in production.

Known facts:

```text
Developer laptop:
- Python 3.12
- APP_PORT=5000
- Database runs on localhost:5432

Production:
- Python version unknown
- APP_PORT may be missing
- Database runs on db01.internal:5432
- Another application may already use port 5000
```

Without changing anything, design an evidence-first investigation.

Your answer must include:

1. The commands you would run.
2. What evidence each command would provide.
3. At least four possible root causes.
4. Why `localhost` is suspicious.
5. Which requirements a future container image should package.
6. Which configuration should remain external to the image.

Do not provide a fix until the evidence identifies the root cause.

---

## Lab Assets

Suggested repository locations:

```text
assets/chapter-07/lesson-01/
├── traditional-deployment-workflow.txt
├── application-environment-tree.txt
├── configuration-drift-comparison.txt
├── environment-demo-success.txt
└── environment-demo-failure.txt
```

---

## Next Part

**Lesson 01 — Part 2: From Physical Servers to Containers**

We will examine:

- Physical-server deployment
- Server underutilization
- Application isolation attempts
- Virtual machines as a major improvement
- The remaining operational cost of virtual machines
- Linux process isolation
- Why containers became practical
- Why Docker became widely adopted


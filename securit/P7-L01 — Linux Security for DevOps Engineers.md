# DevOps Task #13

## P7-L01 — Linux Security for DevOps Engineers

**Student:** Hamad Tarawa  
**Stage:** Part 7 — Linux Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering

---

## 1. Connection to the Previous Lesson

This is the first lesson in Part 7, so there is no previous Linux security
lesson.

This lesson introduces the main idea behind Linux security. The remaining
lessons will examine each protection layer separately.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain why Linux security matters in DevOps.
- Identify the main Linux protection layers.
- Distinguish a normal user from an administrator.
- Explain why applications should not normally run as `root`.
- Build a simple mental model of how Linux protects system resources.

---

## 3. Why This Matters for a DevOps Engineer

Many systems managed by DevOps engineers run on Linux:

- Spring Boot application servers
- Nginx reverse proxies
- PostgreSQL databases
- Docker containers
- Kubernetes worker nodes
- CI/CD runners
- Cloud virtual machines

A developer may mainly ask:

> Does the application work?

A DevOps engineer must also ask:

> Which Linux user runs the application?

> Which files can the application read or modify?

> Which network ports are publicly accessible?

> Who can change the server configuration?

DevOps engineers are not necessarily penetration testers. However, they are
responsible for creating and operating a secure environment for applications.

---

## 4. Simple Explanation

### What is Linux security?

Linux security is the collection of rules and controls that determine:

- Who can enter the system.
- What each user can do.
- Which files a user or application can access.
- Which programs are allowed to run.
- Which network services are exposed.
- Who can perform administrative operations.

Linux security is not one tool or one setting. It consists of several connected
protection layers.

### The central idea

Linux security answers four questions:

```text
1. Who is trying to perform an action?
2. Which resource are they trying to access?
3. Which action are they trying to perform?
4. Do the security rules allow that action?
```

For example, imagine that a Spring Boot application tries to read a
configuration file.

Linux checks:

1. Which Linux user owns the Spring Boot process?
2. Who owns the configuration file?
3. What permissions does the file have?
4. Does the application user have permission to read it?

Linux then allows or denies the operation.

---

## 5. The Main Linux Protection Layers

### Layer 1 — Users

A Linux user is an identity inside the operating system.

Examples:

```text
hamad
root
postgres
www-data
```

A user may represent:

- A real person.
- An administrator.
- An application.
- A system service.

Linux uses the user's identity when deciding whether an operation should be
allowed.

For example, PostgreSQL commonly runs under a service user named `postgres`.
The PostgreSQL process receives the permissions of that user instead of
receiving unlimited access to the entire server.

---

### Layer 2 — Groups

A Linux group is a collection of users.

Examples:

```text
developers
docker
sudo
```

Groups make shared access easier to manage.

Suppose three developers need permission to access deployment files. Instead
of configuring the same permission separately for each developer, an
administrator can:

1. Create a `developers` group.
2. Add the developers to that group.
3. Give the group access to the required files.

---

### Layer 3 — Files and Permissions

Linux servers contain important files:

- Application code
- Configuration files
- Environment files
- Deployment scripts
- SSH private keys
- Log files
- Database files

Linux permissions determine who can:

- Read a file: `r`
- Write or modify a file: `w`
- Execute a file: `x`

Different files need different permissions.

For example, a public HTML file may be readable by many users, but an SSH
private key must only be readable by its owner.

---

### Layer 4 — Processes

A process is a program that is currently running.

Examples:

- A running Spring Boot application
- An Nginx server
- A PostgreSQL server
- A Bash shell

Every process has an owner.

The process normally uses the permissions of its owner. Therefore, the Linux
user that runs an application affects what that application can do.

If Spring Boot runs under a limited user, it may only access the files and
resources needed by the application.

If it runs as `root`, it may have access to almost the entire server. This
increases the possible damage caused by a programming mistake or security
problem.

---

### Layer 5 — Network Ports

Applications use network ports to receive connections.

| Port | Common service | Typical exposure |
|---:|---|---|
| `22` | SSH | Restricted to trusted administrators |
| `80` | HTTP | Public when required |
| `443` | HTTPS | Public for secure web traffic |
| `8080` | Spring Boot | Usually internal behind a reverse proxy |
| `5432` | PostgreSQL | Usually private |

Opening a port creates a possible network entry point.

This does not mean that every listening port is automatically available from
the internet. Firewalls and network configuration also affect whether it is
reachable.

A secure DevOps design exposes only the required services.

For example:

- Users can access Nginx through port `443`.
- Nginx can communicate internally with Spring Boot on port `8080`.
- Spring Boot can communicate privately with PostgreSQL on port `5432`.
- Internet users cannot connect directly to PostgreSQL.

---

### Layer 6 — Administrative Privileges

Some operations can change the entire system:

- Installing software
- Creating users
- Modifying system files
- Managing services
- Changing firewall rules
- Shutting down the server

Linux reserves these powerful operations for administrators.

The main Linux administrator account is:

```text
root
```

The `root` user has almost complete control over the system.

Because this account is extremely powerful, it should not be used for normal
daily work or to run ordinary applications.

Approved users can execute a specific administrative command using `sudo`.

Example:

```bash
sudo apt update
```

This provides temporary administrative privilege for that command.

---

## 6. Important Terms

| Term | Simple meaning |
|---|---|
| Linux security | Controls that protect a Linux system and its resources |
| User | An identity for a person or service |
| Group | A collection of users that can share permissions |
| Permission | A rule that allows or denies an action |
| Process | A program that is currently running |
| Process owner | The Linux user whose permissions the process uses |
| Port | A numbered network entry point used by a service |
| `root` | The most powerful Linux administrator |
| `sudo` | A controlled way to run an administrative command |
| Service account | A user created to run an application or service |
| Least privilege | Giving only the permissions required to perform a task |

---

## 7. Simple Real-World Example

Imagine that you deploy this application:

```text
Internet User
      |
      | HTTPS :443
      v
Nginx Reverse Proxy
Runs as a limited Nginx user
      |
      | Internal connection :8080
      v
Spring Boot API
Runs as a dedicated application user
      |
      | Private connection :5432
      v
PostgreSQL
Runs as the postgres user
```

Linux security protects this system in several ways:

1. Nginx, Spring Boot, and PostgreSQL run under separate users.
2. Each service receives only the file access it needs.
3. Users reach the application through HTTPS on port `443`.
4. Spring Boot port `8080` can remain internal.
5. PostgreSQL port `5432` is not exposed to internet users.
6. Administrative changes require approved access.

If Spring Boot is compromised, the attacker does not automatically control the
whole server. The attacker's possible actions are limited by the permissions
of the user running Spring Boot.

This is called reducing the **blast radius**: one problem should not
automatically compromise every part of the system.

---

## 8. Commands and What They Mean

The following commands only inspect your system. They do not modify anything.

### Command 1 — Identify the current user

```bash
whoami
```

This prints the name of the user running the current shell.

Possible result:

```text
hamad
```

Security meaning:

> The result tells you which Linux identity your commands currently use.

---

### Command 2 — Inspect identity and groups

```bash
id
```

Possible result:

```text
uid=1000(hamad) gid=1000(hamad) groups=1000(hamad),27(sudo)
```

This output includes:

- `uid`: User ID
- `gid`: Primary Group ID
- `groups`: All groups containing the user

Belonging to the `sudo` group does not mean that every command automatically
runs as `root`. The user must still request administrative privilege using
`sudo`.

---

### Command 3 — Inspect processes and their owners

```bash
ps aux | head
```

`ps aux` lists running processes.

The first column shows the owner of each process.

The pipe and `head` show only the first few lines so that the output remains
easy to read.

Security meaning:

> Applications and services run as processes, and their owners help determine
> what those processes are allowed to access.

---

### Command 4 — Inspect listening ports

```bash
ss -tuln
```

Important options:

- `t`: Show TCP sockets.
- `u`: Show UDP sockets.
- `l`: Show listening sockets.
- `n`: Show numeric ports instead of converting them into service names.

Security meaning:

> The output identifies services currently waiting for network connections.

If no ports are shown inside WSL, that is acceptable. It may simply mean that
no network service is currently listening.

---

## 9. Safe Practical Lab

### Lab purpose

Inspect four Linux security elements:

1. Current user
2. Group membership
3. Process ownership
4. Listening ports

### Environment

Use your own WSL environment.

This lab is safe because all commands are read-only.

### Step 1 — Inspect your current identity

Run:

```bash
whoami
id
```

Record:

- Your username
- Your UID
- Your primary GID
- Your groups

### Step 2 — Inspect process ownership

Run:

```bash
ps aux | head
```

Look at the first column and identify the owner of at least two processes.

### Step 3 — Inspect listening ports

Run:

```bash
ss -tuln
```

If a port appears, record one port number.

If no listening ports appear, write:

> No listening ports were found in my WSL environment.

### Step 4 — Explain the security meaning

Answer:

> What do these outputs demonstrate about Linux security?

Model answer:

> The commands show that Linux assigns identities and groups to users, assigns
> owners to running processes, and identifies network ports that accept
> connections. These represent the identity, process, and network layers of
> Linux security.

---

## 10. Expected Result

Your result may look similar to:

```text
$ whoami
hamad

$ id
uid=1000(hamad) gid=1000(hamad) groups=1000(hamad),27(sudo)
```

This means:

- The current username is `hamad`.
- The user's numerical ID is `1000`.
- The primary group ID is `1000`.
- The user also belongs to the `sudo` group.

Your real output may be different, and that is normal.

---

## 11. Evidence to Save

Save the following evidence:

1. A screenshot showing:

   ```bash
   whoami
   id
   ```

2. A screenshot showing:

   ```bash
   ps aux | head
   ```

3. A screenshot showing:

   ```bash
   ss -tuln
   ```

4. A short explanation under the screenshots.

Suggested evidence caption:

> These commands identify my current Linux user and group membership, show
> that running processes have owners, and list the network ports accepting
> connections. They demonstrate the identity, process, and network protection
> layers used by Linux.

This is introductory evidence. The three main WSL security demonstrations
required by Part 7 will be completed in later lessons.

---

## 12. Common Mistakes

### Mistake 1 — Running every application as root

This gives the application much more power than it usually needs.

### Mistake 2 — Giving every user administrative access

Only approved users who actually need administrative tasks should receive that
ability.

### Mistake 3 — Giving everyone access to sensitive files

Configuration files, private keys, and secrets need restricted permissions.

### Mistake 4 — Opening every network port

Only ports required by the system architecture should be exposed.

### Mistake 5 — Assuming containers replace Linux security

Docker containers and Kubernetes workloads still depend on Linux users,
processes, permissions, and networking.

### Mistake 6 — Using `sudo` with every command

Use `sudo` only when an operation genuinely requires administrative privilege.

---

## 13. Task Report Notes

You may adapt the following text for your assignment:

> Linux security is important in DevOps because Linux commonly runs
> application servers, containers, CI/CD runners, databases, and cloud virtual
> machines. Linux protects these environments through several connected
> controls, including users, groups, file permissions, process ownership,
> network ports, and administrative privileges.

> Applications should normally run under dedicated and limited service
> accounts instead of the root user. This follows the Principle of Least
> Privilege and reduces the possible impact of an application compromise.

> A secure DevOps environment should also expose only the network services
> required by its architecture. For example, users may access an Nginx reverse
> proxy through HTTPS, while the Spring Boot API and PostgreSQL database remain
> internal.

---

## 14. Five Review Questions

1. Why does a DevOps engineer need to understand Linux security?
2. What is the difference between a normal user and the `root` user?
3. How does a process owner affect what the process can do?
4. Why should PostgreSQL port `5432` normally remain private?
5. What does the Principle of Least Privilege mean?

---

## 15. Lesson Summary

Linux security is not a single tool. It is a collection of connected
protection layers:

```text
Users and Groups
       |
       v
File Permissions
       |
       v
Process Ownership
       |
       v
Network Ports
       |
       v
Administrative Privileges
```

The most important idea is:

> Every person and application should receive only the access required to
> perform its job.

This reduces mistakes, limits the impact of security incidents, and makes
Linux systems safer to operate.

---

## 16. Progress

### Part 7 — Linux Security

- [x] P7-L01 — Linux Security for DevOps Engineers
- [ ] P7-L02 — Linux Users and Groups
- [ ] P7-L03 — Linux File Permissions
- [ ] P7-L04 — Changing Permissions with chmod
- [ ] P7-L05 — File Ownership with chown and chgrp
- [ ] P7-L06 — sudo and Least Privilege
- [ ] P7-L07 — SSH Security
- [ ] P7-L08 — Linux Firewall Basics
- [ ] P7-L09 — Linux Process Isolation
- [ ] P7-L10 — Part 7 WSL Lab and Report

---

## 17. Exact Next Lesson

**P7-L02 — Linux Users and Groups**

Do not continue until the student writes:

```text
next
```

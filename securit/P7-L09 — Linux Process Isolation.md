# DevOps Task #13

## P7-L09 — Linux Process Isolation

**Student:** Hamad Tarawa  
**Stage:** Part 7 — Linux Security  
**Estimated time:** 20–30 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Ubuntu

---

## 1. Connection to the Previous Lesson

In **P7-L08 — Linux Firewall Basics**, you learned that a firewall controls
which network traffic can reach a Linux system and its services.

However, reaching a service is only one part of security.

After traffic reaches an application, the application runs as a Linux
**process**. Linux must control what that process is allowed to access.

Example:

```text
User request
     |
     v
Firewall allows HTTPS
     |
     v
Nginx process
     |
     v
Spring Boot process
     |
     v
PostgreSQL process
```

Each running service is represented by one or more processes.

This lesson explains how process ownership and Linux permissions help prevent
one application from controlling every resource on the system.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Explain what a Linux process is.
- Explain what a Process ID (`PID`) is.
- Identify the owner of a process.
- Explain why processes run under specific users.
- Explain how user permissions limit process access.
- Understand basic process isolation.
- Explain the relationship between Linux processes and containers.
- Inspect processes using `ps`, `top`, and `pgrep`.
- Demonstrate process ownership safely in WSL.

---

## 3. Why This Matters for a DevOps Engineer

A DevOps engineer operates running applications, not only application files.

Examples include:

- Nginx
- Spring Boot
- PostgreSQL
- Docker
- Kubernetes components
- CI/CD runners
- Monitoring agents

Every running application consumes resources and performs actions through one
or more Linux processes.

When deploying a service, a DevOps engineer should ask:

- Which user owns this process?
- Does the process need root privileges?
- Which files can it access?
- Which other processes can it interact with?
- How much CPU and memory can it consume?
- What happens if the application is compromised?

Running services under separate, limited users helps reduce the impact of
mistakes and security incidents.

---

## 4. Simple Explanation

### 4.1 What is a process?

A process is a program that is currently running.

A file stored on disk is not automatically a process.

For example:

```text
Application file on disk
student-app.jar
        |
        | java -jar student-app.jar
        v
Running Java process
```

Other examples:

```text
nginx binary      -> Nginx process
postgres program  -> PostgreSQL process
bash program      -> Bash process
sleep command     -> sleep process
```

When a program starts, Linux creates a process and tracks information about
it.

---

### 4.2 What information does Linux track?

Linux tracks information such as:

- Process ID
- Parent process
- Process owner
- Current state
- CPU usage
- Memory usage
- Command being executed

This information helps administrators inspect, manage, and troubleshoot
running services.

---

### 4.3 What is a PID?

PID means:

```text
Process ID
```

A PID is a number that identifies a running process.

Example:

```text
PID 2451 -> java -jar student-app.jar
PID 2510 -> nginx
PID 2672 -> postgres
```

Linux uses the PID to distinguish one running process from another.

Administrators can use a PID to:

- Inspect a process.
- Monitor it.
- Send it a signal.
- Stop it when necessary.

A PID is temporary. If a process stops and starts again, it may receive a
different PID.

---

### 4.4 What is a process owner?

Every process runs under a Linux user.

That user is the process owner.

Example:

| Process | Possible owner |
|---|---|
| Bash shell | `hamad` |
| Nginx worker | `www-data` |
| PostgreSQL | `postgres` |
| Spring Boot | `appuser` |

The process normally receives the permissions of its owner.

If a process owned by `appuser` tries to read a file, Linux checks whether
`appuser` has permission to read that file.

```text
Spring Boot process
Owner: appuser
        |
        | tries to read /opt/app/config.yml
        v
Linux checks appuser's permissions
        |
    +---+---+
    |       |
  Allow    Deny
```

---

### 4.5 Why should applications use dedicated users?

Suppose Nginx, Spring Boot, and PostgreSQL all run as `root`.

If one service is compromised, it may inherit extremely powerful access to
the server.

A safer design uses separate limited users:

```text
Nginx       -> www-data
Spring Boot -> appuser
PostgreSQL  -> postgres
```

Then each service receives only the access required for its role.

For example:

- Nginx reads web files.
- Spring Boot reads its configuration and connects to the database.
- PostgreSQL accesses its database files.

Nginx should not need permission to modify PostgreSQL database files.
PostgreSQL should not need permission to modify the Spring Boot application
code.

This applies the Principle of Least Privilege to running processes.

---

## 5. Basic Process Isolation

Process isolation means limiting how running processes affect one another and
the rest of the system.

At a beginner level, Linux provides isolation through:

- Different process owners
- File permissions
- Separate process memory
- Process IDs
- Resource controls
- Additional container isolation

### Separate memory

Each process normally receives its own memory space.

One normal process cannot simply read or modify another process's memory as if
it were its own variables.

### Different users

Processes owned by different users receive different permissions.

Example:

```text
Process A
Owner: appuser
Can read: application configuration

Process B
Owner: postgres
Can read and write: database files
```

### Important limitation

Running applications under different users is helpful, but it is not a
complete security sandbox.

Processes may still interact through intentionally shared resources such as:

- Network connections
- Shared files
- Pipes
- Sockets

Security depends on correctly configuring those resources.

---

## 6. What Happens if an Application Is Compromised?

Imagine that a Spring Boot application contains a serious security
vulnerability.

If the application runs as `root`:

```text
Compromised Spring Boot
Owner: root
        |
        v
Potential access to many system resources
```

If it runs as a limited user:

```text
Compromised Spring Boot
Owner: appuser
        |
        v
Limited by appuser's permissions
```

The limited user does not make the application perfectly safe. However, it
reduces the possible **blast radius**.

The attacker may control the application process but should not automatically
receive permission to:

- Modify every system file.
- Read every user's private files.
- Change the SSH configuration.
- Access unrelated database files.
- Control every other service.

This is why process ownership matters to DevOps security.

---

## 7. Processes and Containers

A container is not a completely separate physical machine.

From the Linux host's perspective, containerized applications ultimately run
as Linux processes.

```text
Docker container
      |
      v
One or more Linux processes
      |
      v
Linux kernel
```

Containers add isolation features that limit what their processes can see and
use.

At a simple level, containers can provide:

- A separate view of processes.
- A separate filesystem view.
- A separate network environment.
- CPU and memory limits.

However:

- Containers still depend on the Linux kernel.
- A process inside a container should not automatically run with unnecessary
  root privilege.
- Container security does not remove the need for Linux security.

The important mental model is:

> A container runs isolated Linux processes; it is not magic and it is not a
> separate physical server.

---

## 8. Important Terms

| Term | Simple meaning |
|---|---|
| Process | A program that is currently running |
| PID | Numerical identifier assigned to a process |
| Process owner | Linux user whose permissions the process uses |
| Parent process | Process that started another process |
| Process isolation | Controls that limit how processes affect each other |
| Least privilege | Giving a process only the access it requires |
| Service account | Limited user created to run an application or service |
| Blast radius | The amount of the system affected by a failure or compromise |
| Signal | A message sent to a process, such as a request to stop |
| Container | An isolated environment containing one or more Linux processes |

---

## 9. Simple Real-World Example

Consider this web application:

```text
Internet user
      |
      | HTTPS :443
      v
Nginx process
Owner: www-data
      |
      | Internal request :8080
      v
Spring Boot process
Owner: appuser
      |
      | Database connection :5432
      v
PostgreSQL process
Owner: postgres
```

The process boundaries provide several benefits:

1. Nginx does not need access to PostgreSQL database files.
2. Spring Boot does not need permission to modify protected system files.
3. PostgreSQL receives access to its own data directory.
4. Each service can be monitored and restarted separately.
5. A problem in one service does not automatically grant the permissions of
   another service.

The firewall controls which traffic reaches these services.

Process ownership controls what each running service can do after it receives
traffic.

---

## 10. Commands and Configuration

### Command 1 — List processes with `ps aux`

```bash
ps aux
```

Purpose:

> Display running processes with their owners and resource usage.

Important columns:

| Column | Meaning |
|---|---|
| `USER` | Process owner |
| `PID` | Process ID |
| `%CPU` | Approximate CPU usage |
| `%MEM` | Approximate memory usage |
| `COMMAND` | Command that started the process |

The output may be long.

To show only the first few lines:

```bash
ps aux | head
```

---

### Command 2 — List processes with `ps -ef`

```bash
ps -ef
```

Purpose:

> Display processes using another common format.

Important columns:

| Column | Meaning |
|---|---|
| `UID` | Process owner |
| `PID` | Process ID |
| `PPID` | Parent Process ID |
| `CMD` | Command |

The `PPID` shows which process started another process.

To limit the output:

```bash
ps -ef | head
```

---

### Command 3 — Monitor processes with `top`

```bash
top
```

Purpose:

> Display a live, updating view of running processes and resource usage.

`top` commonly shows:

- CPU usage
- Memory usage
- Process owners
- PIDs
- Running commands

Press:

```text
q
```

to exit `top`.

`top` is interactive, so its display will continue to update until you quit.

---

### Command 4 — Find Bash processes

```bash
pgrep bash
```

Purpose:

> Display PIDs of processes named `bash`.

To display the PID and process command together:

```bash
pgrep -a bash
```

The `-a` option shows the full command line.

Depending on your shell and WSL session, you may see one or more Bash
processes.

---

### Command 5 — Inspect the current shell process

```bash
ps -o pid,ppid,user,group,comm -p $$
```

Explanation:

- `-o`: Selects the columns to display.
- `pid`: Process ID.
- `ppid`: Parent Process ID.
- `user`: Process owner.
- `group`: Process group owner.
- `comm`: Command name.
- `-p $$`: Selects the current shell process.

In Bash, `$$` contains the PID of the current shell.

This command helps connect your current Bash session with its PID and owner.

---

## 11. Safe Practical Lab

### Lab goal

Start a harmless background process, identify its PID and owner, and stop it
cleanly.

### Safety

The lab uses:

```bash
sleep 300
```

The `sleep` command simply waits. It does not modify files, networking, or
system configuration.

The process runs under your normal user and will be stopped before the lab
ends.

---

### Step 1 — Inspect your current Bash process

Run:

```bash
ps -o pid,ppid,user,group,comm -p $$
```

Record:

- Bash PID
- Parent PID
- User
- Group
- Command name

---

### Step 2 — Find existing Bash processes

Run:

```bash
pgrep -a bash
```

You should see at least the Bash process used by your current WSL terminal.

---

### Step 3 — Start a harmless background process

Run:

```bash
sleep 300 &
```

Explanation:

- `sleep 300`: Waits for 300 seconds.
- `&`: Runs the process in the background so the terminal remains available.

The shell may display something similar to:

```text
[1] 2451
```

The number `2451` is an example PID.

---

### Step 4 — Save the background PID

Immediately run:

```bash
LAB_PID=$!
```

In Bash, `$!` contains the PID of the most recently started background
process.

`LAB_PID` is only a shell variable used for this lab.

Display it:

```bash
echo "$LAB_PID"
```

---

### Step 5 — Inspect the process and its owner

Run:

```bash
ps -o pid,ppid,user,group,stat,comm -p "$LAB_PID"
```

You should see:

- The PID stored in `LAB_PID`.
- Your normal Linux username.
- Your group.
- The command `sleep`.

This proves that the process runs under your user identity.

---

### Step 6 — Find the process using `pgrep`

Run:

```bash
pgrep -a sleep
```

This may show your lab process and any other `sleep` processes running in the
environment.

Use the PID to identify your specific lab process.

---

### Step 7 — Stop only the lab process

Run:

```bash
kill "$LAB_PID"
```

Purpose:

> Send the normal termination signal to the specific process created by this
> lab.

Do not use a random PID. Use only the value saved in `LAB_PID`.

---

### Step 8 — Verify that it stopped

Run:

```bash
ps -p "$LAB_PID"
```

A possible result is:

```text
PID TTY          TIME CMD
```

with no process row underneath.

The shell may also display:

```text
Terminated
```

This confirms that the lab process ended.

---

### Optional Step — View live processes

Run:

```bash
top
```

Observe:

- PID
- User
- CPU usage
- Memory usage
- Command

Press `q` to exit.

---

## 12. Expected Result

Your result may look similar to:

```text
$ sleep 300 &
[1] 2451

$ LAB_PID=$!

$ echo "$LAB_PID"
2451

$ ps -o pid,ppid,user,group,stat,comm -p "$LAB_PID"
    PID    PPID USER     GROUP    STAT COMMAND
   2451    2100 hamad    hamad    S    sleep

$ kill "$LAB_PID"

$ ps -p "$LAB_PID"
    PID TTY          TIME CMD
```

Interpretation:

- `2451` is the example PID.
- `2100` is the example parent PID.
- `hamad` owns the process.
- The process command is `sleep`.
- After `kill`, the process no longer appears.

Your real PID and PPID will be different.

---

## 13. Evidence to Save

Save a screenshot showing:

```bash
sleep 300 &
LAB_PID=$!
echo "$LAB_PID"
ps -o pid,ppid,user,group,stat,comm -p "$LAB_PID"
kill "$LAB_PID"
ps -p "$LAB_PID"
```

The screenshot should demonstrate:

1. The process started.
2. Linux assigned it a PID.
3. The process ran under your normal user.
4. You stopped the specific process.
5. The final command verified that it ended.

### Suggested evidence title

```text
Linux Process PID, Ownership, and Lifecycle Demonstration in WSL
```

### Suggested evidence explanation

> I started a harmless `sleep` process in the background. Linux assigned it a
> PID, and the `ps` command showed that it was owned by my normal Linux user.
> This demonstrates that every running process has an identity and uses the
> permissions of its owner. I then sent a normal termination signal to the
> specific PID and verified that the process stopped.

### Part 7 demonstration status

This lab can be used as the required:

> Process ownership inspection using `ps aux`.

It also provides additional evidence using a process created safely by the
student.

---

## 14. Common Mistakes

### Mistake 1 — Running every service as root

This gives a compromised service much more access than it needs.

Use a limited service account when possible.

---

### Mistake 2 — Confusing a program file with a process

A program file is stored on disk.

A process is a running instance of that program.

---

### Mistake 3 — Assuming a PID is permanent

When a process restarts, Linux may assign it a different PID.

Automation should not assume that a manually observed PID remains valid
forever.

---

### Mistake 4 — Stopping the wrong process

Always confirm:

- PID
- Owner
- Command

before sending a signal.

In this lab, stop only the PID stored in `LAB_PID`.

---

### Mistake 5 — Immediately using `kill -9`

A normal `kill PID` requests graceful termination.

`kill -9 PID` forces immediate termination and does not give the process an
opportunity to clean up.

Use force only when necessary and after understanding the possible impact.

---

### Mistake 6 — Assuming different users provide perfect isolation

Different users and permissions provide an important protection layer, but
processes may still communicate through approved files, sockets, and networks.

Security requires multiple correctly configured controls.

---

### Mistake 7 — Assuming containers are virtual machines

Containers provide process isolation but still use the host's Linux kernel.

They do not remove the need for secure users, permissions, images, and runtime
configuration.

---

## 15. Task Report Notes

You may adapt the following text for your final assignment:

> A Linux process is a running instance of a program. Linux assigns each
> process a Process ID, owner, state, and resource information. The process
> normally uses the permissions of its owner, so running applications under
> separate and limited service accounts reduces unnecessary access.

> Process ownership supports the Principle of Least Privilege. For example,
> Nginx, Spring Boot, and PostgreSQL can run under different Linux users. Each
> service receives access only to the files and resources needed for its role.
> If one application is compromised, the attacker's possible actions remain
> limited by that process owner's permissions.

> Containers also run Linux processes. They add isolation for process,
> filesystem, network, and resource views, but they still depend on the Linux
> kernel. Containerized applications should therefore avoid unnecessary root
> privileges and continue to use secure Linux configurations.

### Practical demonstration note

> In WSL, I started a harmless background `sleep` process. I inspected its PID,
> parent PID, owner, group, state, and command. The process ran under my normal
> user account. I then stopped only that process and verified that it no longer
> appeared. This demonstrated process identity, ownership, and lifecycle
> management.

---

## 16. Five Review Questions

1. What is a Linux process?
2. What is a PID, and why can it change after a restart?
3. How does the owner of a process affect what the process can access?
4. Why should Nginx, Spring Boot, and PostgreSQL use separate limited users?
5. What is the relationship between a Linux process and a container?

---

## 17. Lesson Summary

A process is a running program.

Linux assigns every process:

- A PID
- An owner
- A parent
- A state
- Resource information

The owner is especially important for security because the process normally
uses that user's permissions.

```text
Application starts
       |
       v
Linux creates a process
       |
       v
Process receives PID and owner
       |
       v
Owner's permissions limit process access
```

Running services under separate limited accounts:

- Applies least privilege.
- Reduces unnecessary access.
- Limits the possible blast radius.
- Improves accountability.

Important commands:

```bash
ps aux
ps -ef
top
pgrep -a bash
ps -o pid,ppid,user,group,comm -p $$
```

Lab commands:

```bash
sleep 300 &
LAB_PID=$!
ps -o pid,ppid,user,group,stat,comm -p "$LAB_PID"
kill "$LAB_PID"
ps -p "$LAB_PID"
```

---

## 18. Progress

### Part 7 — Linux Security

- [x] P7-L01 — Linux Security for DevOps Engineers
- [x] P7-L02 — Linux Users and Groups
- [x] P7-L03 — Linux File Permissions
- [x] P7-L04 — Changing Permissions with chmod
- [x] P7-L05 — File Ownership with chown and chgrp
- [x] P7-L06 — sudo and Least Privilege
- [x] P7-L07 — SSH Security
- [x] P7-L08 — Linux Firewall Basics
- [x] P7-L09 — Linux Process Isolation
- [ ] P7-L10 — Part 7 WSL Lab and Report

---

## Exact Next Lesson

**P7-L10 — Part 7 WSL Lab and Report**

Do not continue until the student writes:

```text
next
```

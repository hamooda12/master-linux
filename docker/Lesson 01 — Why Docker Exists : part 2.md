# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 01 — Why Docker Exists

## Part 2 — From Physical Servers to Containers

---

## Overview

Containers did not suddenly appear because engineers wanted shorter commands. They are one stage in a long effort to run applications more efficiently, consistently, and safely.

This lesson follows that evolution:

```text
One application per physical server
                ↓
Several applications on one server
                ↓
Virtual machines
                ↓
Linux containers
                ↓
Docker's standardized workflow
```

We will distinguish the problems solved by virtual machines from those solved by containers. We will also establish an essential fact: Docker did not invent the fundamental Linux isolation mechanisms used by containers. Docker combined existing kernel features with images, tooling, registries, and a consistent developer experience.

---

## Learning Objectives

By the end of this part, you should be able to:

- Explain the limitations of one-application-per-server deployment.
- Describe why running unrelated applications directly on one host is risky.
- Explain how virtual machines improved isolation and resource utilization.
- Identify the operational costs of a guest operating system.
- Explain the difference between an operating-system process and a container.
- Describe the roles of namespaces and control groups at a high level.
- Explain why a container is not a lightweight virtual machine.
- Explain what Docker added to existing container technology.
- Choose whether a workload needs a physical server, VM, or container.

---

## Prerequisites

Before starting, you should understand:

- Processes and process IDs
- Linux users and permissions
- Filesystems
- Network interfaces and ports
- The deployment inconsistency problem from Lesson 01, Part 1

---

# 1. Why This Topic Exists

Suppose a company owns four applications:

- Payroll
- Inventory
- Customer portal
- Reporting

The simplest isolation strategy is to place every application on a separate physical server:

```text
Physical Server 1       Physical Server 2
Payroll                 Inventory

Physical Server 3       Physical Server 4
Customer portal         Reporting
```

This prevents many dependency and resource conflicts, but it creates new problems:

- Hardware is expensive.
- Procurement can take weeks.
- Servers are often underutilized.
- Failed hardware affects the application directly.
- Scaling requires more physical equipment.
- Every server requires power, cooling, networking, updates, and maintenance.

Engineers needed isolation without dedicating an entire physical machine to every application.

---

# 2. Stage One — Physical Servers

## 2.1 One application per server

Historically, organizations often isolated critical applications by dedicating a physical server to each one.

```text
Hardware
├── CPU
├── Memory
├── Disk
└── Network interface
        │
        ▼
Operating system
        │
        ▼
One application
```

## 2.2 Why it was attractive

- Strong hardware boundary
- Predictable resource ownership
- Fewer dependency conflicts
- Easier fault attribution
- Simple mental model

## 2.3 The utilization problem

Imagine a server with:

```text
16 CPU cores
64 GB RAM
2 TB disk
```

The application normally uses:

```text
1 CPU core
4 GB RAM
100 GB disk
```

Most capacity remains unused. The organization paid for the entire machine but uses only a small portion of it.

This is not merely a hardware-cost problem. Unused capacity still consumes rack space, power, cooling, monitoring, licensing, and administrative effort.

## 2.4 Slow provisioning

A request for a new environment might require:

1. Budget approval
2. Hardware purchase
3. Delivery
4. Rack installation
5. Network configuration
6. Operating-system installation
7. Security hardening
8. Application deployment

Application teams could wait days or weeks for a new environment.

---

# 3. Stage Two — Multiple Applications on One Server

To use hardware more efficiently, teams installed several applications directly on the same operating system.

```text
Physical Server
│
├── Linux kernel
├── Shared system libraries
├── Payroll application
├── Inventory application
├── Reporting application
└── Customer portal
```

This improved utilization, but weakened isolation.

## 3.1 Dependency conflicts

```text
Payroll requires library version 1
Reporting requires library version 2
Both use the host's shared library installation
```

An upgrade for one application can break another.

## 3.2 Port conflicts

Only one process can normally listen on the same combination of IP address, protocol, and port.

If two applications both expect `0.0.0.0:8080`, one will fail to bind.

## 3.3 Resource interference

One application may consume most of the host's:

- CPU
- Memory
- Disk space
- Disk I/O
- Network bandwidth
- Process identifiers

This is commonly called the **noisy-neighbor problem**.

## 3.4 Security blast radius

If an attacker compromises one application, weak isolation may make it easier to inspect or modify another application's files and processes.

Linux permissions improve separation, but all applications still share the same host environment and kernel.

## 3.5 Operational coupling

A host-level change affects every application:

```text
Kernel update
    ↓
Host restart
    ↓
All applications stop
```

The applications have become operationally coupled because they share one machine and operating-system instance.

---

# 4. Stage Three — Virtual Machines

Virtualization introduced a powerful abstraction: one physical server could behave like several independent computers.

## 4.1 The hypervisor

A hypervisor creates and manages virtual hardware for virtual machines.

```text
Physical Hardware
│
└── Hypervisor
    ├── VM 1
    │   ├── Guest OS
    │   └── Payroll
    ├── VM 2
    │   ├── Guest OS
    │   └── Inventory
    └── VM 3
        ├── Guest OS
        └── Reporting
```

Each VM receives virtual resources such as:

- Virtual CPUs
- Virtual memory
- Virtual disks
- Virtual network interfaces

Each VM boots its own guest operating system and kernel.

## 4.2 What virtual machines solved

### Better hardware utilization

Several virtual servers can share one physical server.

### Strong isolation

Each VM has its own kernel and operating-system environment.

### Different operating systems

A Linux VM and a Windows VM can run on the same compatible virtualization platform.

### Snapshots and templates

Administrators can create VM templates and snapshots to accelerate provisioning and recovery.

### Resource allocation

CPU and memory can be assigned to each VM, limiting some forms of noisy-neighbor interference.

## 4.3 Virtual machines changed infrastructure

Provisioning moved from purchasing new hardware toward creating a software-defined machine:

```text
Request new server
        ↓
Clone VM template
        ↓
Allocate CPU, memory, disk, network
        ↓
Boot guest operating system
        ↓
Configure and deploy application
```

This was a major improvement and remains fundamental in modern infrastructure and cloud computing.

---

# 5. What Remained Expensive with Virtual Machines?

Virtual machines are not obsolete or bad. They solve a different isolation problem. However, using one VM for every small application introduces overhead.

## 5.1 Every VM includes a guest operating system

```text
VM 1: Application + libraries + guest OS + guest kernel
VM 2: Application + libraries + guest OS + guest kernel
VM 3: Application + libraries + guest OS + guest kernel
```

Repeated guest operating systems consume:

- Memory
- Disk space
- CPU during background activity
- Boot time
- Update effort
- Security-maintenance effort

## 5.2 Slower startup

A VM starts by booting an operating system:

```text
Virtual hardware initialization
            ↓
Guest kernel boot
            ↓
System services start
            ↓
Application starts
```

A container normally starts an isolated application process without booting a separate guest kernel.

## 5.3 Lower workload density

A host may run tens of appropriately sized VMs but potentially far more lightweight containers, depending on resource requirements and safety constraints.

Do not memorize a fixed number. Density depends on workload size, kernel limits, storage, networking, security policies, and available resources.

## 5.4 Image size and transfer time

A VM image commonly includes an entire guest operating-system disk and may be several gigabytes. Container images often contain only the required user-space files and can be much smaller.

## 5.5 Duplicate administration

Every guest operating system needs:

- Package updates
- User management
- Security hardening
- Monitoring
- Logging
- Certificate management
- Vulnerability remediation

Automation reduces this work, but does not make it disappear.

---

# 6. Stage Four — Linux Containers

Linux containers take a different approach from virtual machines.

Instead of virtualizing an entire computer, containers isolate processes running on the same kernel.

```text
Physical or Virtual Host
│
├── Host Linux kernel
│
├── Container A
│   ├── Application process
│   └── User-space files
│
├── Container B
│   ├── Application process
│   └── User-space files
│
└── Container C
    ├── Application process
    └── User-space files
```

## Essential definition

> A container is an isolated process—or group of processes—with a controlled view of system resources and its own packaged user-space filesystem.

On Linux, a container process is still a process on the host. It is not a small computer hidden inside the server.

---

# 7. Is a Container Just a Normal Process?

The short answer is: it is a host process, but it runs with additional isolation and resource controls.

A normal process can usually observe shared host resources according to its permissions:

- Process IDs
- Network interfaces
- Mounted filesystems
- Hostname
- Users
- Inter-process communication

A containerized process receives a restricted or separate view of many of these resources.

```text
Host view
├── PID 1: systemd
├── PID 902: sshd
└── PID 2400: nginx container process

Container view
└── PID 1: nginx
```

The same nginx process can have PID `2400` from the host's perspective and PID `1` inside its container's PID namespace.

This is not two processes. It is one process observed through two namespace views.

---

# 8. The Linux Features Behind Containers

Docker relies on several Linux kernel features. Two foundational categories are namespaces and control groups.

## 8.1 Namespaces — “What can the process see?”

Namespaces isolate a process's view of system resources.

Important namespace types include:

| Namespace | Isolates |
|---|---|
| PID | Process ID numbering and process visibility |
| Network | Interfaces, routes, IP addresses, ports, firewall context |
| Mount | Filesystem mount points |
| UTS | Hostname and domain name |
| IPC | Inter-process communication resources |
| User | User and group ID mappings |
| Cgroup | Visibility of control-group hierarchy |
| Time | Selected system clocks |

### Network namespace example

Two containers can both listen on port 80 internally because each can have its own network namespace.

```text
Container A network namespace
nginx listens on 0.0.0.0:80

Container B network namespace
nginx listens on 0.0.0.0:80
```

The host must still map distinct host ports if both services need to be published through the same host address:

```text
Host port 8080 → Container A port 80
Host port 8081 → Container B port 80
```

## 8.2 Control groups — “How much can the process use?”

Control groups, usually called **cgroups**, organize processes and enforce or measure resource usage.

They can help control or account for resources such as:

- CPU
- Memory
- Process counts
- Block I/O

Conceptually:

```text
Container process
      │
      └── cgroup
          ├── Memory limit
          ├── CPU allocation
          └── Process limit
```

Namespaces and cgroups solve different problems:

```text
Namespaces → isolation and visibility
Cgroups    → resource control and accounting
```

## 8.3 Filesystem isolation

Container runtimes provide a filesystem assembled from an image plus a writable container layer.

The process may see:

```text
/
├── app
├── etc
├── usr
└── var
```

These paths do not necessarily represent the host's root filesystem. Mount namespaces and layered filesystems help create the container's view.

We will study image layers deeply in Lesson 05.

## 8.4 Security mechanisms

Production isolation may also use:

- Linux capabilities
- Seccomp syscall filtering
- AppArmor or SELinux
- Read-only filesystems
- User namespaces
- Non-root users

Namespaces alone are not a complete security boundary.

---

# 9. Containers vs Virtual Machines — First Comparison

| Area | Virtual Machine | Container |
|---|---|---|
| Virtualizes | Hardware/computer | Operating-system process environment |
| Kernel | Separate guest kernel | Shares host kernel |
| Startup | Boots guest OS | Starts isolated process |
| Typical size | Often gigabytes | Often megabytes to hundreds of megabytes |
| Isolation boundary | Usually stronger | Shared-kernel boundary |
| OS flexibility | Can run different guest kernels | Must be compatible with host kernel/runtime |
| Density | Lower | Often higher |
| Primary artifact | VM disk image/template | Container image |

This comparison is simplified. Security and performance depend on the hypervisor, container runtime, configuration, workload, kernel, and surrounding controls.

Lesson 02 will provide the full comparison.

---

# 10. Why Containers Start Quickly

When starting a VM:

```text
Create virtual machine
        ↓
Initialize virtual hardware
        ↓
Boot guest kernel
        ↓
Start operating-system services
        ↓
Start application
```

When starting a container:

```text
Prepare namespaces and cgroups
        ↓
Assemble container filesystem
        ↓
Configure networking
        ↓
Start application process
```

The container does not normally boot its own kernel. This is a major reason it can start quickly and consume fewer resources.

However, the application itself may still be slow. A container can start immediately while a large Java application takes additional time to initialize.

This distinction becomes important when designing health checks.

---

# 11. What Docker Added

Container-like isolation existed before Docker. Technologies and kernel mechanisms such as `chroot`, namespaces, cgroups, Linux Containers (LXC), and other systems contributed to the evolution.

Docker's importance was that it made containers easier to build, distribute, and operate through a consistent workflow.

## 11.1 A standard image format and workflow

Developers could describe an application environment with a Dockerfile and build a reusable image.

```text
Dockerfile
    ↓
docker build
    ↓
Image
```

## 11.2 Simple lifecycle commands

Docker offered a consistent interface for:

- Building images
- Pulling and pushing images
- Creating containers
- Starting and stopping containers
- Viewing logs
- Inspecting runtime configuration

## 11.3 Registries

Registries made container images distributable:

```text
Developer or CI
      │
      │ push image
      ▼
Container registry
      │
      │ pull image
      ▼
Test or production host
```

## 11.4 Layered images

Docker images reuse layers, improving storage efficiency and transfer speed when layers are already cached.

## 11.5 Developer experience

Docker turned several low-level Linux mechanisms into a workflow that application teams could use without manually constructing namespaces and cgroups.

## Precise conclusion

> Docker did not invent containers. Docker popularized a standardized, image-based method for building, distributing, and running them.

---

# 12. Internal Workflow — From Image to Running Process

At a high level, when a container is started, the runtime must:

```text
Receive container configuration
             ↓
Locate image layers
             ↓
Create writable container layer
             ↓
Create namespaces
             ↓
Assign cgroups and limits
             ↓
Configure mounts and networking
             ↓
Apply security settings
             ↓
Start the configured process
```

Later lessons will add the Docker daemon, containerd, and low-level runtime details to this workflow.

---

# 13. A Container Image Is Not a Running Container

This distinction prevents many beginner mistakes.

## Image

A versioned, read-only package containing application files, dependencies, filesystem layers, and default execution metadata.

## Container

A runtime instance created from an image, with runtime configuration and a writable layer.

```text
Image: nginx:1.28
      │
      ├── Container A
      ├── Container B
      └── Container C
```

One image can create many independent containers.

Analogy—with limits:

```text
Image     ≈ class or blueprint
Container ≈ running instance
```

Do not push the analogy too far. An image is a real filesystem and metadata artifact, not merely an abstract design.

---

# 14. Hands-on Lab — Observe Isolation Without Docker

This lab uses `unshare`, which may require installation and root privileges. Its purpose is to reveal the Linux concepts Docker automates.

## Safety

Run this only on your own lab machine or disposable VM. Do not experiment with namespaces on an unfamiliar production server.

## Step 1 — Observe the current hostname

```bash
hostname
```

Record the result.

## Step 2 — Start a shell in a new UTS namespace

```bash
sudo unshare --uts /bin/bash
```

You are still running a Bash process on the same kernel, but the shell has a separate UTS namespace.

## Step 3 — Change the hostname inside the namespace

```bash
hostname container-lab
hostname
```

The namespace reports:

```text
container-lab
```

## Step 4 — Exit the namespace

```bash
exit
hostname
```

The original host hostname remains unchanged.

## What happened internally?

```text
Original shell
    │
    └── unshare creates new UTS namespace
            │
            └── Child Bash sees separate hostname
```

You did not create a VM or boot another kernel. You created a process with a separate view of one kernel resource.

## Optional PID namespace observation

On a disposable lab system:

```bash
sudo unshare --fork --pid --mount-proc /bin/bash
ps -ef
```

Inside the new PID namespace, the Bash process appears as PID 1. From another host shell, it has a different host PID.

Exit when finished:

```bash
exit
```

## Lab conclusion

Namespaces are properties applied to processes. Docker combines several namespace types with cgroups, filesystems, networking, security controls, images, and lifecycle management.

---

# 15. Production Scenario — VM or Container?

## Request

A team wants to deploy 40 small stateless HTTP APIs. Each API:

- Uses the same Linux kernel family
- Requires 150 MB of memory
- Starts in under two seconds
- Stores no local business data
- Must be independently versioned and deployed

## Analysis

Creating 40 full VMs would provide strong isolation, but every VM would add a guest operating system and maintenance surface.

Containers are a strong candidate because:

- The workloads are stateless.
- They are compatible with a shared host kernel.
- They benefit from fast startup and higher density.
- Each application can have a separately versioned image.

## Important production correction

This does not necessarily mean running all 40 containers directly on one unmanaged Docker host.

You must also consider:

- Host failure and high availability
- Scheduling
- Resource limits
- Network policy
- Secrets
- Observability
- Rolling deployments
- Security boundaries

These needs lead naturally toward orchestration platforms such as Kubernetes.

## When a VM may be preferable

- A different guest kernel or operating system is required.
- The workload needs a stronger isolation boundary.
- Legacy software expects a complete operating system.
- Organizational or compliance rules demand VM separation.
- Kernel modules or low-level host control are required.

In practice, containers often run **inside virtual machines**, combining infrastructure isolation with container portability and density.

---

# 16. Common Mistakes

## Mistake 1 — Saying containers have no operating system

A Linux container image may contain an Ubuntu, Debian, or Alpine user-space filesystem. It does not normally contain or boot its own Linux kernel.

## Mistake 2 — Calling a container a lightweight VM

This can be a beginner analogy, but it hides the central difference: a VM virtualizes hardware and boots a guest kernel; a container isolates host processes sharing a kernel.

## Mistake 3 — Believing a container is invisible to the host

The host can observe and manage the container's processes because they are host processes placed in namespaces and cgroups.

## Mistake 4 — Assuming shared kernel means no isolation

Containers provide real isolation, but it is a different security boundary from a VM. Correct configuration matters.

## Mistake 5 — Assuming containers always outperform VMs

Performance depends on the workload, storage, networking, resource contention, virtualization platform, and configuration.

## Mistake 6 — Removing VMs from every architecture

Containers and VMs are complementary. Many managed container and Kubernetes platforms run worker nodes as VMs.

## Mistake 7 — Running without resource limits

Without suitable controls, one container can consume resources needed by others or by the host.

## Mistake 8 — Treating an image name as a guaranteed version

Mutable tags such as `latest` can point to different image contents over time. Production deployments need deliberate versioning and, when appropriate, immutable digests.

---

# 17. Best Practices

- Choose containers or VMs according to workload and isolation requirements.
- Remember that containers share the host kernel.
- Keep host kernels and container runtimes patched.
- Use resource requests and limits appropriate to the platform.
- Run one primary application responsibility per container.
- Keep containers replaceable rather than repairing them manually.
- Build versioned images through automation.
- Use minimal, compatible, maintained base images.
- Run application processes as non-root where possible.
- Combine containers with VMs when stronger infrastructure boundaries are useful.
- Test failure behavior, not only successful startup.

---

# 18. Interview Questions

## Junior

### 1. What is a physical server?

A physical computer with real CPU, memory, storage, and network hardware on which an operating system and applications run.

### 2. What is a hypervisor?

Software or firmware that creates and manages virtual machines and their virtual hardware.

### 3. Does a Linux container normally have its own kernel?

No. It normally shares the host Linux kernel while using an isolated user-space environment.

### 4. Is a container visible as a process on the host?

Yes. The container's application is a host process viewed through namespace isolation.

## Intermediate

### 5. What is the difference between namespaces and cgroups?

Namespaces isolate what processes can see; cgroups control and account for how many resources groups of processes can use.

### 6. Why do containers generally start faster than VMs?

They start isolated application processes instead of booting a separate guest kernel and full operating-system service stack.

### 7. Why can two containers both listen on port 80?

Each can have its own network namespace. If both are published on the same host address, they still require different host ports unless another routing layer is used.

### 8. What did Docker add if containers already existed?

Docker popularized a consistent image format and workflow for building, distributing, versioning, and running containers, along with registries and accessible tooling.

## Senior

### 9. Are containers a sufficient security boundary for every untrusted workload?

Not automatically. They share a kernel, so the threat model may require hardened runtimes, stronger sandboxing, dedicated nodes, VMs, or microVMs in addition to container controls.

### 10. Why are containers often run inside VMs?

VMs provide infrastructure and tenant isolation, while containers provide application packaging, rapid deployment, and workload density.

### 11. What happens when the main container process exits?

The container stops because its lifecycle is tied to its main process, commonly PID 1 inside the container namespace.

### 12. Does a container guarantee portability?

It improves portability across compatible environments, but CPU architecture, host-kernel capabilities, runtime configuration, mounted data, and external dependencies still matter.

---

# 19. Lesson Reflection

Explain these questions in your own words:

1. Why was one application per physical server inefficient?
2. What new risks appeared when several applications shared one host OS?
3. What did virtual machines solve?
4. Why does every VM have additional operational cost?
5. Why is a container a process rather than a miniature computer?
6. What do namespaces and cgroups contribute?
7. Why might an organization use both VMs and containers?

---

# 20. Summary

- Dedicated physical servers offered isolation but caused low utilization and slow provisioning.
- Installing many applications on one host improved utilization but introduced dependency, port, resource, and security conflicts.
- Virtual machines virtualize computers and boot separate guest kernels, providing strong isolation and operating-system flexibility.
- VMs also repeat guest operating systems, increasing memory, storage, startup, and maintenance costs.
- Linux containers isolate processes while sharing the host kernel.
- Namespaces control what container processes can see.
- Cgroups control and measure what resources processes can use.
- Docker did not invent the underlying idea of containers.
- Docker popularized containers with standardized images, registries, lifecycle tools, and a developer-friendly workflow.
- Containers and virtual machines are complementary technologies.

---

# 21. Cheat Sheet

| Command | Purpose |
|---|---|
| `hostname` | Display or, with privilege, set the hostname visible in the current UTS namespace |
| `ps -ef` | Display processes using full-format output |
| `sudo unshare --uts /bin/bash` | Start Bash in a new UTS namespace |
| `sudo unshare --fork --pid --mount-proc /bin/bash` | Start Bash with a new PID namespace and matching `/proc` view |
| `exit` | Leave the namespace shell |
| `lsns` | List Linux namespaces visible to the current user |
| `nsenter` | Enter selected namespaces of an existing process; use carefully |
| `systemd-cgls` | Display the systemd control-group hierarchy |
| `systemd-cgtop` | Show resource activity by control group |

These are Linux learning commands, not the normal manual method for creating production containers. Docker and lower-level runtimes automate this work.

---

# 22. Challenge — Choose the Correct Isolation Model

A company has three workloads:

## Workload A

- Twenty stateless Go APIs
- Same Linux architecture
- Frequent deployments
- Small memory footprint

## Workload B

- A legacy Windows application
- Requires Windows-specific system services
- Vendor supports only a full Windows Server environment

## Workload C

- Executes code submitted by unknown external users
- Strong hostile-tenant isolation is required
- Security is more important than startup speed

For each workload:

1. Choose physical server, VM, container, or a combination.
2. Explain the required isolation boundary.
3. Explain the resource and operational trade-offs.
4. Identify what must still be secured or managed.
5. Explain why the rejected options are less appropriate.

Do not answer using “containers are always better.” Make the decision from evidence and workload requirements.

---

## Lab Assets

Suggested repository locations:

```text
assets/chapter-07/lesson-01/
├── physical-server-model.txt
├── vm-architecture.txt
├── container-architecture.txt
├── namespace-pid-view.txt
├── vm-vs-container-table.md
└── unshare-lab-output.txt
```

---

## Next Lesson

**Lesson 02 — Containers vs Virtual Machines**

We will compare:

- Type 1 and Type 2 hypervisors
- Host operating systems and guest operating systems
- Kernel boundaries
- Boot and startup workflows
- CPU, memory, storage, and networking overhead
- Isolation and security trade-offs
- Real production architecture decisions

# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 02 — Containers vs Virtual Machines

## Part 1 — Architecture, Kernels, Isolation, and Resources

---

## Overview

Virtual machines and containers both allow several workloads to share physical infrastructure, but they create isolation at different layers.

A virtual machine behaves like a separate computer. It receives virtual hardware and boots its own guest operating system and kernel.

A Linux container isolates one or more processes. It receives a controlled view of the host, but normally uses the host's Linux kernel.

This distinction explains most differences in startup time, size, operating-system compatibility, isolation, maintenance, and workload density.

---

## Learning Objectives

By the end of this part, you should be able to:

- Explain hardware virtualization.
- Distinguish Type 1 and Type 2 hypervisors.
- Identify the host OS, guest OS, and guest kernel.
- Explain why a container does not boot a guest kernel.
- Compare VM and container startup workflows.
- Compare CPU, memory, storage, and networking overhead.
- Explain why containers are processes on the host.
- Understand the different isolation boundaries.
- Avoid misleading statements such as “containers have no OS.”

---

## Prerequisites

- Lesson 01, Parts 1 and 2
- Basic understanding of processes, kernels, filesystems, and networking
- Familiarity with CPU, memory, disk, and network interfaces

---

# 1. Why This Comparison Exists

Imagine that a company needs to run ten web applications on one physical server.

Two possible designs are:

```text
Option A: Ten virtual machines

Physical server
└── Hypervisor
    ├── VM 1 + guest OS + application
    ├── VM 2 + guest OS + application
    └── ...
```

```text
Option B: Ten containers

Physical or virtual server
└── Host Linux kernel
    ├── Container 1 process
    ├── Container 2 process
    └── ...
```

Both approaches isolate applications, but they do not provide identical isolation or compatibility.

The correct engineering question is not:

> Which technology is always better?

The correct question is:

> Which isolation boundary, operating-system flexibility, resource model, and operational workflow does this workload require?

---

# 2. Big Picture

## Virtual-machine boundary

```text
Application
Libraries
Guest user space
Guest kernel
Virtual hardware
Hypervisor
Physical hardware
```

## Container boundary

```text
Application
Packaged user space
Container isolation
Host kernel
Physical or virtual hardware
```

The guest kernel is the critical architectural difference.

---

# 3. What Is Virtualization?

Virtualization creates a software representation of computing resources.

A virtual machine may receive:

- Virtual CPUs
- Virtual RAM
- Virtual disks
- Virtual network cards
- Virtual firmware
- Virtual devices

To the guest operating system, these resources appear like hardware.

```text
Guest OS
   │
   │ sees virtual hardware
   ▼
vCPU ─ vRAM ─ vDisk ─ vNIC
   │
   ▼
Hypervisor maps work to physical resources
```

The hypervisor schedules VM CPU work on physical CPUs, maps guest memory, handles virtual devices, and enforces VM boundaries.

---

# 4. Hypervisors

## 4.1 Type 1 — Bare-metal hypervisor

A Type 1 hypervisor runs directly on the physical system, although real implementations may include management components that make the boundary more nuanced.

```text
VMs
│
Hypervisor
│
Physical hardware
```

Common examples include VMware ESXi, Microsoft Hyper-V, and Xen-based systems. Linux KVM turns the Linux kernel into a hypervisor and is widely used in cloud infrastructure.

### Typical use

- Data centers
- Private clouds
- Public-cloud infrastructure
- Production virtualization clusters

### Benefits

- Strong performance
- Centralized management
- High availability features
- Production-focused resource scheduling

## 4.2 Type 2 — Hosted hypervisor

A Type 2 hypervisor runs as software on a host operating system.

```text
Virtual machines
│
Hosted hypervisor application
│
Host operating system
│
Physical hardware
```

Examples include VirtualBox and VMware Workstation.

### Typical use

- Developer laptops
- Training labs
- Desktop testing
- Running another OS temporarily

### Trade-off

The host OS and its device drivers sit between the hypervisor application and hardware, adding another operational layer.

## Important warning

The Type 1 versus Type 2 classification is useful for learning but modern virtualization implementations can be architecturally complex. Do not use the label alone to predict security or performance.

---

# 5. Anatomy of a Virtual Machine

```text
VM
├── Virtual hardware
│   ├── vCPU
│   ├── vRAM
│   ├── vDisk
│   └── vNIC
├── Guest kernel
├── Guest operating-system user space
│   ├── init system
│   ├── system services
│   ├── package manager
│   └── system libraries
└── Application
```

## Host OS

The operating system associated with the physical machine or virtualization platform.

## Guest OS

The operating system installed inside a virtual machine.

## Guest kernel

The kernel booted by the guest OS. It manages the VM's virtual hardware from the guest's perspective.

## Consequence

Because a VM has its own kernel, a Linux host can potentially run a Windows guest VM when the hypervisor and hardware support it.

---

# 6. Anatomy of a Linux Container

```text
Container
├── Application process
├── Application dependencies
├── User-space system libraries
├── Configuration defaults
├── Namespace views
├── Cgroup membership
└── Security restrictions

Shared externally:
└── Host Linux kernel
```

The container may contain files associated with Ubuntu, Debian, Alpine, or another distribution. Those files form a **user space**. They do not include a running kernel for the container.

## Precise language

Avoid:

> “The container runs Ubuntu 24.04 with its own OS.”

Prefer:

> “The container uses an Ubuntu 24.04 user-space image while sharing the host Linux kernel.”

---

# 7. Shared Kernel Explained

The kernel provides system calls used by applications.

```text
Application
    │
    │ system call
    ▼
Linux kernel
    │
    ▼
CPU, memory, files, network
```

Containerized applications still make system calls to the host kernel.

## Example

When nginx inside a container accepts a TCP connection:

1. nginx calls networking-related system calls.
2. The shared host kernel processes them.
3. The container's network namespace affects what nginx can see.
4. Cgroups and security controls may restrict the process.
5. The physical or virtual network ultimately carries the packets.

## Compatibility consequence

A standard Linux container requires Linux-kernel semantics. A Windows container requires a compatible Windows kernel environment.

Docker Desktop can run Linux containers on Windows or macOS by using a Linux virtual machine behind the scenes.

```text
Windows or macOS
        │
Linux VM
        │
Linux kernel
        │
Linux containers
```

The graphical experience can hide this VM, but the kernel requirement remains.

---

# 8. VM Startup Workflow

Starting a powered-off VM commonly involves:

```text
Hypervisor allocates virtual resources
                ↓
Virtual firmware begins
                ↓
Bootloader loads guest kernel
                ↓
Guest kernel initializes devices
                ↓
Init system starts
                ↓
Guest system services start
                ↓
Application service starts
```

This creates a complete independently booted operating-system environment.

## Result

- Startup commonly takes seconds to minutes.
- Baseline memory is used before the application starts.
- Guest services run even if the main application does not need all of them.

Actual timing depends on the VM image, OS, hypervisor, storage, boot design, and application.

---

# 9. Container Startup Workflow

Starting a container commonly involves:

```text
Runtime reads container configuration
                ↓
Image filesystem is prepared
                ↓
Namespaces are created or joined
                ↓
Cgroups and limits are applied
                ↓
Networking and mounts are configured
                ↓
Security policy is applied
                ↓
Application process starts
```

There is normally no guest firmware, bootloader, or guest-kernel boot.

## Result

- Container creation can be very fast.
- Baseline overhead is relatively small.
- Application initialization may still take time.

## Critical distinction

```text
Container started ≠ Application ready
```

A Java container can be running while Spring Boot is still initializing. Production systems therefore use readiness or health checks rather than assuming that a running process is ready to serve traffic.

---

# 10. CPU Comparison

## Virtual machines

The hypervisor schedules virtual CPUs onto physical CPUs.

```text
Guest process
    ↓
Guest kernel scheduler
    ↓
vCPU
    ↓
Hypervisor / host scheduling
    ↓
Physical CPU
```

## Containers

Container processes are scheduled by the host kernel, with cgroups influencing allocation and limits.

```text
Container process
    ↓
Host kernel scheduler
    ↓
Physical or virtual CPU
```

Containers generally avoid a separate guest-kernel scheduling layer, but this does not mean unlimited or guaranteed CPU performance.

## CPU limit misconception

A CPU limit does not necessarily mean the container owns a dedicated CPU core. It may mean the process can consume only a defined amount of CPU time within a scheduling period.

For latency-sensitive workloads, CPU topology, contention, throttling, and platform scheduling policies matter.

---

# 11. Memory Comparison

## VM memory

A VM needs memory for:

- Guest kernel
- Guest services
- Filesystem cache
- Application
- Management overhead

Memory is assigned through the virtualization platform, sometimes with advanced techniques such as ballooning or overcommitment.

## Container memory

A container uses host memory for its processes and kernel-accounted resources. Cgroups can enforce a limit.

```text
Host memory
├── Kernel
├── Host services
├── Container A cgroup
├── Container B cgroup
└── Filesystem caches
```

## What happens at a memory limit?

If a container exceeds its configured memory constraints, allocation may fail or the kernel may terminate a process through an out-of-memory mechanism, depending on the configuration and circumstances.

From the application perspective, this can look like a sudden unexplained exit. Engineers must inspect container state, events, and host evidence.

---

# 12. Storage Comparison

## VM storage

A VM commonly uses one or more virtual disk images.

```text
Guest filesystem
      ↓
Virtual block device
      ↓
VM disk image or storage volume
      ↓
Physical storage
```

The disk contains the guest OS, system packages, logs, and application data.

## Container storage

A container commonly uses:

- Read-only image layers
- A thin writable container layer
- Volumes or bind mounts for persistent data

```text
Read-only image layers
          +
Writable container layer
          +
Persistent volume if needed
```

## Important consequence

The writable container layer is tied to the container lifecycle. Important data should normally live in explicitly managed persistent storage, not only in that layer.

---

# 13. Networking Comparison

## VM networking

A VM has a virtual network interface attached to a virtual switch or other virtual network.

```text
Application
    ↓
Guest network stack
    ↓
Virtual NIC
    ↓
Virtual switch
    ↓
Host physical network
```

The guest kernel owns its network stack.

## Container networking

A container can have its own network namespace with virtual interfaces, routes, ports, and DNS configuration.

```text
Container process
    ↓
Container network namespace
    ↓
Virtual Ethernet pair
    ↓
Docker bridge / host networking
    ↓
Host network stack
```

Docker may configure network-address translation and port publishing so host traffic reaches the container.

We will study this in depth in Lesson 08.

---

# 14. Isolation Boundary

## VM boundary

The application is separated from the host by a guest OS, guest kernel, virtual hardware, and hypervisor boundary.

An attacker escaping the application must generally cross multiple boundaries to reach the host.

## Container boundary

The application shares the host kernel with other containers and the host.

Isolation depends on:

- Namespaces
- Cgroups
- Linux permissions
- Capabilities
- Seccomp
- AppArmor or SELinux
- Runtime configuration
- Kernel correctness

## Security conclusion

VMs generally provide a stronger default isolation boundary because they do not share a guest kernel with one another. Containers provide meaningful isolation, but their shared-kernel threat model is different.

This does not mean containers are insecure. It means security decisions must match the workload.

---

# 15. Resource Comparison Table

| Characteristic | Virtual Machine | Container |
|---|---|---|
| Isolation unit | Virtual computer | Process or process group |
| Kernel | Separate guest kernel | Shared host kernel |
| Startup | Guest OS boot | Process start |
| Baseline memory | Higher | Usually lower |
| Artifact size | Often GBs | Often MBs to hundreds of MBs |
| Workload density | Usually lower | Usually higher |
| OS flexibility | Different guest OS/kernel possible | Must match kernel family expectations |
| Security boundary | Hypervisor and separate kernel | Shared kernel with isolation controls |
| Persistent storage | Virtual disks | Volumes/bind mounts/external storage |
| Normal lifecycle | Long-lived machine | Replaceable application instance |
| Patch responsibility | Host plus every guest OS | Host/kernel/runtime plus rebuilt images |

These are tendencies, not guarantees.

---

# 16. Internal Observation Commands

These commands help prove the architecture. Some require Docker, which will be installed in Lesson 04. You may keep this section for later practice.

## Identify host kernel

```bash
uname -r
```

Inside a Linux container:

```bash
uname -r
```

The release normally corresponds to the host kernel because the container does not boot its own kernel.

## Identify container user space

```bash
cat /etc/os-release
```

Inside an Ubuntu image, this may say Ubuntu even if the host uses another Linux distribution. This file describes user-space distribution files, not a separate running kernel.

## Observe process from host

```bash
docker inspect --format '{{.State.Pid}}' CONTAINER
ps -fp HOST_PID
```

The first command gets the container's main process ID as seen by the host. The second proves that it is a host-visible process.

## Inspect namespaces

```bash
sudo lsns -p HOST_PID
```

This shows namespaces associated with the process.

## Inspect cgroup membership

```bash
cat /proc/HOST_PID/cgroup
```

This displays the process's control-group membership. Exact output varies by cgroup version and runtime configuration.

---

# 17. Hands-on Lab — Prove the Shared Kernel

Complete this lab after Docker installation in Lesson 04.

## Step 1 — Record host information

```bash
uname -r
cat /etc/os-release
```

Record both outputs separately:

- `uname -r` identifies the running kernel.
- `/etc/os-release` identifies host user-space distribution information.

## Step 2 — Start an Ubuntu container

```bash
docker run --name kernel-lab ubuntu:24.04 sleep 600
```

## Step 3 — Compare container information

```bash
docker exec kernel-lab uname -r
docker exec kernel-lab cat /etc/os-release
```

Expected reasoning:

- The container identifies its user space as Ubuntu 24.04.
- The kernel release matches the host Linux kernel.

## Step 4 — Find the process on the host

```bash
docker inspect --format '{{.State.Pid}}' kernel-lab
```

Use the returned PID:

```bash
ps -fp HOST_PID
sudo lsns -p HOST_PID
cat /proc/HOST_PID/cgroup
```

## Step 5 — Clean up

```bash
docker rm -f kernel-lab
```

## Lab conclusion

The container had Ubuntu user-space files but used the host kernel. Its main command was visible as a process on the host and was associated with namespaces and cgroups.

---

# 18. Production Scenario — Startup and Density

## Scenario

A retail platform must scale its product API from 20 to 100 instances during a five-minute traffic spike.

Each instance:

- Uses Linux
- Is stateless
- Requires 300 MB of memory
- Starts the application in 8 seconds
- Uses the same tested image

## VM approach

If each instance requires a new VM, scaling includes guest operating-system boot and application startup. VM templates and warm pools can reduce the delay, but add infrastructure management.

## Container approach

If suitable capacity already exists on container hosts, the platform can start additional container processes without booting 80 guest kernels.

## Evidence required before choosing

- Available CPU and memory
- Application initialization time
- Image pull time
- Registry throughput
- Network and load-balancer update time
- Database connection capacity
- Security isolation requirements
- Failure-domain design

## Important lesson

Containers remove guest boot from the scaling path, but they do not remove application startup, image transfer, scheduling, networking, or dependency bottlenecks.

---

# 19. Common Mistakes

## Mistake 1 — “Containers do not have an OS”

They commonly contain operating-system user-space files. They normally do not boot a separate kernel.

## Mistake 2 — “Ubuntu container means Ubuntu kernel”

The image's `/etc/os-release` describes user space. `uname -r` reports the shared host kernel.

## Mistake 3 — “A container is always faster”

Startup and runtime performance depend on application behavior, storage, networking, contention, and configuration.

## Mistake 4 — “VMs are old technology”

VMs remain essential for cloud infrastructure, strong isolation, different kernels, and many legacy or regulated workloads.

## Mistake 5 — “Running means ready”

The process may exist while the application is initializing or unable to serve requests.

## Mistake 6 — “Container memory is free”

Container processes consume real host memory. Higher density makes limits and capacity planning more important, not less.

## Mistake 7 — “Deleting a container is like deleting a VM safely”

Data in the writable container layer can disappear with the container. Persistence must be designed explicitly.

---

# 20. Best Practices

- Use containers for portable application packaging, not as tiny manually maintained servers.
- Use VMs where separate kernels or stronger tenant isolation are required.
- Remember that containers commonly run inside VMs.
- Set and test resource controls.
- Keep important data outside the writable container layer.
- Build readiness and health checks around real application behavior.
- Patch the host kernel and container runtime.
- Rebuild images to update application dependencies rather than patching running containers manually.
- Compare workload requirements before selecting an isolation model.
- Measure startup, resource usage, and performance instead of relying on slogans.

---

# 21. Interview Questions

## Junior

### 1. What is the main architectural difference between a VM and a container?

A VM boots its own guest kernel on virtual hardware. A Linux container runs isolated processes that normally share the host kernel.

### 2. What is a guest OS?

The operating system installed and running inside a virtual machine.

### 3. Why do containers usually start faster?

They normally start application processes without booting virtual hardware, a guest kernel, and a complete guest service stack.

## Intermediate

### 4. Can an Ubuntu container run on a Fedora host?

Yes, if the application and image are compatible with the host Linux kernel and CPU architecture. The Ubuntu user space uses the Fedora host's kernel.

### 5. Why can Linux containers run on macOS Docker Desktop?

Docker Desktop uses a Linux VM that provides the Linux kernel required by those containers.

### 6. Why is VM isolation generally stronger?

VMs have separate guest kernels and a hypervisor boundary, while containers share the host kernel.

### 7. What is the difference between container start and application readiness?

Container start means its main process is running. Readiness means the application has completed initialization and can correctly serve traffic.

## Senior

### 8. Where can container density become dangerous?

High density increases shared failure impact, resource contention, kernel exposure, networking pressure, and the number of workloads affected by one host failure.

### 9. Why might a platform use VMs as Kubernetes nodes?

VMs provide infrastructure isolation and flexible node provisioning; containers provide application packaging and scheduling inside those nodes.

### 10. What factors affect whether a container memory limit causes termination?

Cgroup version and configuration, memory and swap controls, workload allocation behavior, kernel OOM decisions, and orchestrator policies all matter.

---

# 22. Lesson Reflection

Explain in your own words:

1. Which layer does a VM virtualize?
2. Which layer does a container isolate?
3. Why can a VM run a different kernel?
4. What does an Ubuntu image contain if it does not contain a running Ubuntu kernel?
5. Why is a container process visible on the host?
6. Why does container startup not guarantee application readiness?

---

# 23. Summary

- VMs virtualize computers and boot independent guest kernels.
- Containers isolate host processes and normally share the host kernel.
- Type 1 hypervisors are production-oriented virtualization layers; Type 2 hypervisors commonly run on desktop host operating systems.
- Containers generally start faster and use fewer baseline resources because they do not boot guest operating systems.
- VMs generally provide stronger default isolation and greater kernel flexibility.
- Container images package user-space files, not a separate running kernel.
- CPU, memory, storage, and networking still use real host resources.
- Running and ready are different states.
- VMs and containers are complementary and are frequently used together.

---

# 24. Cheat Sheet

| Command | Purpose |
|---|---|
| `uname -r` | Display the running kernel release |
| `cat /etc/os-release` | Identify user-space distribution information |
| `docker run --name NAME IMAGE COMMAND` | Create and start a container with a selected command |
| `docker exec NAME COMMAND` | Run an additional command inside a running container |
| `docker inspect --format '{{.State.Pid}}' NAME` | Display the main container process PID as seen by the host |
| `ps -fp PID` | Inspect a host process |
| `sudo lsns -p PID` | Display namespaces associated with a process |
| `cat /proc/PID/cgroup` | Inspect process cgroup membership |
| `docker rm -f NAME` | Force-remove a lab container |

Docker commands in this lesson are preview commands. Their syntax and internal behavior will be taught fully after Docker installation.

---

# 25. Challenge — Prove or Reject the Claims

Evaluate each claim. Mark it **correct**, **partly correct**, or **incorrect**, then explain why.

1. A container is a small virtual machine.
2. An Ubuntu container always uses an Ubuntu kernel.
3. Containers consume no memory when idle.
4. VMs always perform worse than containers.
5. Two containers can use port 80 internally.
6. If a container is running, its application is healthy.
7. Docker Desktop runs Linux containers directly on the macOS kernel.
8. VMs and containers should never be used together.
9. Deleting a container always preserves its application data.
10. Shared-kernel isolation and hypervisor isolation have identical threat models.

For each incorrect statement, rewrite it as an accurate engineering statement.

---

## Lab Assets

Suggested repository locations:

```text
assets/chapter-07/lesson-02/
├── type-1-hypervisor.txt
├── type-2-hypervisor.txt
├── vm-stack.txt
├── container-stack.txt
├── startup-workflows.txt
├── shared-kernel-lab-output.txt
└── resource-comparison.md
```

---

## Next Part

**Lesson 02 — Part 2: Security, Workload Selection, and Production Decisions**

We will cover:

- Isolation threat models
- Kernel attack surface
- Root inside a container
- Privileged containers
- Compliance and multi-tenancy
- Containers inside VMs
- Decision matrices
- Production architecture scenarios

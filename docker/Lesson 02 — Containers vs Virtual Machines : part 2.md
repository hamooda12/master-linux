# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 02 — Containers vs Virtual Machines

## Part 2 — Security, Workload Selection, and Production Decisions

---

## Overview

Containers and virtual machines are not competing answers to every problem. They provide different isolation boundaries and are frequently combined.

This part moves from architecture into engineering judgment. We will examine shared-kernel risk, root inside containers, privileged mode, hostile workloads, multi-tenancy, compliance, failure domains, and production selection criteria.

The goal is not to declare one technology the winner. The goal is to select the correct boundary for a workload and then operate it safely.

---

## Learning Objectives

By the end of this part, you should be able to:

- Explain the security boundary of a container.
- Explain why a VM generally provides stronger isolation.
- Distinguish root inside a container from root on the host.
- Explain the risks of privileged containers and host mounts.
- Understand container escape at a high level.
- Evaluate trusted and untrusted workloads differently.
- Explain failure domains and blast radius.
- Select physical servers, VMs, containers, or combinations.
- Defend a production architecture using evidence.

---

## Prerequisites

- Lesson 02, Part 1
- Linux users, permissions, processes, and capabilities at a basic level
- Networking and firewall fundamentals

---

# 1. Why This Topic Exists

Consider two workloads:

```text
Workload A
Internal company API written by a trusted team

Workload B
Service that executes code uploaded by unknown internet users
```

Both applications can technically run in containers, but they do not have the same threat model.

For Workload A, standard container isolation with strong runtime controls may be appropriate.

For Workload B, the application intentionally executes hostile or untrusted code. A stronger boundary—such as dedicated VMs, microVMs, sandboxed runtimes, or multiple combined controls—may be required.

The technology decision begins with risk, not convenience.

---

# 2. Big Picture — Layers of Isolation

Production systems often use several layers:

```text
Physical hardware
└── Hypervisor or cloud platform
    └── Virtual machine
        └── Host Linux kernel
            └── Container runtime
                └── Container
                    └── Application process
```

Each layer can reduce risk, but each also adds configuration, maintenance, and possible failure modes.

Security uses **defense in depth**:

```text
Application security
        +
Non-root process
        +
Minimal capabilities
        +
Seccomp and MAC policy
        +
Container isolation
        +
VM boundary
        +
Network controls
```

No single layer should be treated as magical protection.

---

# 3. Threat Models

A threat model identifies what you are protecting, who may attack it, how they may attack it, and what impact a compromise could have.

## Questions to ask

- Is the application code trusted?
- Is user-provided code executed?
- Is the container exposed directly to the internet?
- What secrets can the workload access?
- Does it process financial or personal data?
- Are workloads from different customers sharing a host?
- What happens if the host kernel is compromised?
- Which compliance rules apply?
- How quickly must a compromised workload be isolated?

## Trusted does not mean safe

Code written by your own team can still contain vulnerabilities. “Trusted” describes origin and intent; it does not prove security.

---

# 4. Container Security Boundary

A Linux container shares the host kernel. Its separation depends on correctly configured kernel mechanisms.

```text
Container A ─┐
Container B ─┼── Shared host kernel
Container C ─┘
```

The boundary may include:

- PID, mount, network, user, IPC, and UTS namespaces
- Cgroups
- Linux permissions
- Capabilities
- Seccomp filters
- AppArmor or SELinux policies
- Read-only mounts
- Runtime and daemon configuration

If these controls are weakened or the kernel/runtime contains an exploitable vulnerability, isolation may be breached.

## Container escape

A container escape is a situation where a process breaks out of its intended isolation and gains unauthorized access to the host or other workloads.

Possible contributing factors include:

- Kernel vulnerabilities
- Container-runtime vulnerabilities
- Excessive Linux capabilities
- Privileged mode
- Dangerous host mounts
- Exposed Docker socket
- Incorrect device access
- Misconfigured namespace sharing

Container escape is not the normal behavior of containers, but the shared-kernel design makes kernel and runtime security critical.

---

# 5. VM Security Boundary

VMs normally separate workloads using virtual hardware, guest kernels, and a hypervisor.

```text
VM A                    VM B
Application             Application
Guest kernel            Guest kernel
        └──── Hypervisor ────┘
                    │
              Physical host
```

An attacker compromising an application inside VM A reaches the guest environment first. Reaching VM B or the physical host generally requires crossing an additional virtualization boundary.

## Important correction

VMs are not invulnerable. Risks include:

- Hypervisor vulnerabilities
- Misconfigured virtual networks
- Exposed management interfaces
- Weak images or credentials
- Shared-storage mistakes
- Guest OS vulnerabilities

The claim is not “VMs are perfectly secure.” The claim is that separate guest kernels and a hypervisor usually provide a stronger default isolation boundary than ordinary shared-kernel containers.

---

# 6. Root Inside a Container

Many container images run their main process as UID 0 unless configured otherwise.

Inside the container, UID 0 is called root.

## Is container root automatically host root?

No. Namespaces, capabilities, mounts, security policies, and runtime configuration restrict what container root can do.

However, container root is still dangerous because:

- It has more power inside the container.
- A breakout vulnerability may have greater impact.
- Mounted files may be owned or modified as host UID 0.
- Misconfiguration can expose host devices or namespaces.

## Without user-namespace remapping

UID 0 inside a container often maps to UID 0 on the host, even though other isolation controls limit it.

## With user namespaces

Container UID 0 can map to an unprivileged host UID:

```text
Container UID 0
       ↓ maps to
Host UID 100000
```

This reduces risk, but compatibility and platform configuration must be considered.

## Best practice

Run the application as a dedicated non-root user whenever possible. This does not replace other controls, but it reduces available privilege.

---

# 7. Linux Capabilities

Traditional root possesses broad privileges. Linux capabilities divide many of those privileges into smaller units.

Examples include the ability to:

- Bind certain privileged ports
- Change network configuration
- Load kernel modules
- Change file ownership
- Send signals to other processes

Docker normally removes many capabilities by default but retains a limited set.

## Principle of least privilege

An application should receive only the capabilities it actually requires.

```bash
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE IMAGE
```

Conceptually:

- `--cap-drop ALL` removes all configurable capabilities.
- `--cap-add NET_BIND_SERVICE` adds back one specific capability if required.

Do not copy this pattern blindly. Test application behavior and understand the platform's defaults.

---

# 8. Privileged Containers

```bash
docker run --privileged IMAGE
```

Privileged mode grants the container extensive access to host devices and relaxes important security restrictions.

It can make the container far closer to a host-level process with broad power.

## Why engineers use it incorrectly

An application fails because of permissions. Instead of finding the exact missing permission, someone adds `--privileged`, and the application starts.

This hides the root cause by removing much of the security boundary.

## Evidence-first alternative

Investigate:

1. Which operation failed?
2. Which file, device, syscall, or capability was denied?
3. Does the application truly require it?
4. Can one narrow permission be granted?
5. Can the application design avoid the requirement?

## Production rule

Treat privileged containers as exceptional high-risk workloads requiring explicit review. Never use privileged mode as a generic permission fix.

---

# 9. Dangerous Host Mounts

Bind mounts expose host paths inside containers.

```bash
docker run -v /host/path:/container/path IMAGE
```

They can be useful, but the security boundary depends on what is mounted and whether it is writable.

## Mounting the host root

```bash
docker run -v /:/host IMAGE
```

A process with sufficient permissions may read or modify sensitive host files through `/host`.

## Mounting the Docker socket

```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock IMAGE
```

The Docker socket provides control over the Docker daemon. A process that can use it may create highly privileged containers, mount host paths, or control other containers.

Access to the Docker socket should generally be treated as host-equivalent administrative access.

## Read-only mounts

When an application only needs to read data:

```bash
docker run --mount type=bind,src=/host/config,dst=/app/config,readonly IMAGE
```

Read-only reduces modification risk, though sensitive data may still be disclosed.

---

# 10. Read-only Container Filesystems

A read-only root filesystem limits runtime modification of packaged application files.

```bash
docker run --read-only IMAGE
```

Applications may still need writable locations for temporary files or runtime state. These can be supplied deliberately:

```bash
docker run --read-only --tmpfs /tmp IMAGE
```

## Benefits

- Reduces accidental changes
- Makes persistence requirements explicit
- Restricts some attacker behavior
- Encourages immutable deployment practices

## Common problem

An application assumes it can write anywhere. The correct response is to identify its legitimate writable paths, not immediately disable the protection.

---

# 11. Seccomp, AppArmor, and SELinux

## Seccomp

Seccomp filters which system calls a process may make.

```text
Application syscall request
          ↓
Seccomp policy
      ┌───┴───┐
   Allowed   Blocked
```

Reducing available syscalls reduces kernel attack surface.

## AppArmor and SELinux

These Linux Security Modules provide mandatory access-control policies beyond standard user/group permissions.

They can restrict access to:

- Files
- Capabilities
- Network-related operations
- Other system resources

## Key idea

Container security is a stack of controls. Non-root execution, capabilities, seccomp, and mandatory access control solve different parts of the problem.

---

# 12. Secrets and Isolation

Isolation does not protect a secret that is carelessly provided.

Common risks include:

- Secrets embedded in images
- Secrets committed to Git
- Secrets passed visibly in commands
- Secrets printed in logs
- Overly broad secret access
- Host environment exposure

## Image risk

Deleting a secret in a later Dockerfile layer may not remove it from earlier image layers.

Secrets need:

- Secure storage
- Controlled injection
- Least-privilege access
- Rotation
- Auditability
- Protection from logs and image history

Docker secrets and orchestrator secret mechanisms will be covered later, but no mechanism makes careless application logging safe.

---

# 13. Trusted vs Untrusted Multi-tenancy

## Trusted internal workloads

Several applications owned by one organization may share container hosts when properly isolated and monitored.

## Untrusted tenants

When unrelated customers or unknown users execute code, the consequences of escape are greater.

Possible stronger designs include:

- Separate VMs per tenant or trust domain
- Dedicated nodes
- Sandboxed container runtimes
- MicroVMs
- Restricted language sandboxes
- Short-lived isolated execution environments

## Decision principle

Do not choose isolation only from workload size. Choose it from the **trust boundary** and impact of compromise.

---

# 14. Failure Domains and Blast Radius

A failure domain is a group of components that can fail together.

## One large host

```text
Host failure
    ↓
All containers on host stop
```

High container density improves utilization but may increase the number of workloads affected by one host failure.

## Several hosts

```text
Load-balanced service
├── Instance on Host A
├── Instance on Host B
└── Instance on Host C
```

Distributing replicas reduces dependence on one host.

## VM failure domain

A guest failure may affect one VM, but a physical host or hypervisor failure can affect every VM on that host.

## Engineering conclusion

Isolation boundaries do not automatically provide high availability. Workloads must be distributed across meaningful failure domains.

---

# 15. Operational Lifecycle Differences

## Traditional VM mindset

VMs are often treated as long-lived servers:

```text
Create → configure → patch repeatedly → repair → eventually replace
```

## Container mindset

Containers should generally be replaceable:

```text
Build image → deploy container → observe → replace with new image
```

## Anti-pattern

```bash
docker exec -it production-api bash
apt install package
edit configuration
```

This creates an undocumented difference between the running container and its image. If the container is replaced, the manual fix disappears.

Use interactive access for diagnosis when justified, but encode permanent corrections in the Dockerfile, image, configuration, or deployment definition.

---

# 16. Production Decision Matrix

| Requirement | Container | VM | Combination |
|---|---:|---:|---:|
| Fast application deployment | Strong | Moderate | Strong |
| High workload density | Strong | Moderate | Strong |
| Separate guest kernel | No | Yes | Yes at VM layer |
| Different guest OS family | Limited | Strong | Strong |
| Strong hostile-tenant boundary | Requires extra controls | Stronger default | Often strongest practical design |
| Legacy full-OS application | Sometimes awkward | Strong | Possible |
| Immutable app delivery | Strong | Possible | Strong |
| Hardware/device control | Risky/special | Often clearer | Workload-dependent |
| Small stateless services | Strong | More overhead | Common in cloud |
| Cloud-managed orchestration | Strong | Nodes commonly are VMs | Very common |

This matrix guides analysis; it does not replace workload testing or security review.

---

# 17. When to Prefer Containers

Containers are strong candidates when:

- Applications are stateless or externalize persistent state.
- Fast, repeatable deployment matters.
- Workloads use compatible kernel semantics.
- Teams want versioned application images.
- CI/CD frequently builds and promotes releases.
- Efficient resource utilization matters.
- Services need independent scaling.
- The platform supplies appropriate security and orchestration.

---

# 18. When to Prefer Virtual Machines

VMs are strong candidates when:

- A separate kernel is required.
- Different operating-system families are needed.
- Stronger tenant isolation is necessary.
- Legacy software expects a complete server environment.
- Compliance rules require VM boundaries.
- Kernel-level customization is needed.
- The workload cannot be redesigned safely for a container lifecycle.

---

# 19. When to Use Both

This is one of the most common production designs:

```text
Cloud physical infrastructure
└── Virtual machines
    └── Kubernetes or container runtime
        └── Application containers
```

## Why combine them?

VMs provide:

- Node-level infrastructure isolation
- Cloud provisioning and failure-domain controls
- Separate guest kernels between nodes or tenant groups

Containers provide:

- Application packaging
- Fast deployment
- Image versioning
- Workload density
- Consistent orchestration units

Using both is not duplication. Each layer solves a different problem.

---

# 20. Internal Workflow — Compromised Container

Suppose an attacker exploits a web application inside a container.

```text
Application vulnerability exploited
              ↓
Attacker controls application process
              ↓
Check process user and capabilities
              ↓
Attempt access to secrets and mounts
              ↓
Attempt kernel/runtime escape
              ↓
Security controls allow or block actions
```

Impact depends on:

- Whether the process runs as root
- Capabilities
- Seccomp and MAC policy
- Mounted paths
- Available secrets
- Network access
- Kernel/runtime vulnerabilities
- Host and tenant architecture

This is why “it runs inside Docker” is not a complete security assessment.

---

# 21. Investigation Commands

These commands are previews for use after Docker installation.

## Inspect configured user

```bash
docker inspect --format '{{.Config.User}}' CONTAINER
```

An empty value commonly means the image did not select a user and the default is root.

Verify from inside:

```bash
docker exec CONTAINER id
```

## Inspect privileged mode

```bash
docker inspect --format '{{.HostConfig.Privileged}}' CONTAINER
```

## Inspect capabilities

```bash
docker inspect --format '{{json .HostConfig.CapAdd}} {{json .HostConfig.CapDrop}}' CONTAINER
```

## Inspect mounts

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
```

Human-readable alternative:

```bash
docker inspect CONTAINER
```

## Inspect read-only root filesystem

```bash
docker inspect --format '{{.HostConfig.ReadonlyRootfs}}' CONTAINER
```

## Inspect security options

```bash
docker inspect --format '{{json .HostConfig.SecurityOpt}}' CONTAINER
```

## Important investigation rule

An empty inspect field does not always mean “no security.” Docker or the host may apply defaults that are not repeated in that field. Interpret output with runtime and platform documentation.

---

# 22. Hands-on Lab — Compare Root and Non-root

Complete this after installing Docker.

## Step 1 — Run the default image user

```bash
docker run --rm alpine:3.22 id
```

Typical output shows UID 0.

## Step 2 — Run as an explicit non-root UID

```bash
docker run --rm --user 1000:1000 alpine:3.22 id
```

The process now runs with UID and GID 1000 inside the container.

## Step 3 — Test a protected operation

```bash
docker run --rm --user 1000:1000 alpine:3.22 sh -c 'touch /root/test-file'
```

Expected result: permission denied.

## Step 4 — Test writable temporary storage

```bash
docker run --rm --user 1000:1000 alpine:3.22 sh -c 'touch /tmp/test-file && ls -l /tmp/test-file'
```

## Step 5 — Test a read-only root filesystem

```bash
docker run --rm --read-only alpine:3.22 sh -c 'touch /test-file'
```

Expected result: read-only filesystem error.

## Step 6 — Add controlled temporary storage

```bash
docker run --rm --read-only --tmpfs /tmp alpine:3.22 sh -c 'touch /tmp/test-file && ls -l /tmp/test-file'
```

## Lab conclusion

Non-root execution and read-only filesystems enforce different restrictions. Combining them creates defense in depth.

---

# 23. Production Scenario — CI Runner

## Request

A CI platform must execute build scripts from pull requests submitted by external contributors.

The initial proposal is:

```text
Run every build in a normal container
Mount /var/run/docker.sock
Run the container as privileged
```

## Risk analysis

The workload is untrusted, yet it receives:

- Privileged mode
- Docker daemon control
- A path to host-level container creation

The container boundary provides little meaningful protection under this configuration.

## Better architectural direction

Consider:

- Ephemeral isolated VMs or microVMs
- Dedicated worker pools for untrusted builds
- No shared Docker socket
- Short-lived credentials
- Restricted outbound network access
- Resource and time limits
- Workspace cleanup and destruction after each job
- Auditing and artifact scanning

## Evidence before approval

- Exact trust level of submitted code
- Secrets available during builds
- Network reachability
- Host and runtime controls
- Tenant separation
- Cleanup guarantees
- Consequence of a worker compromise

The final design must be reviewed against the organization's threat model.

---

# 24. Common Mistakes

## Mistake 1 — Using `--privileged` to solve permission errors

It removes important restrictions and hides the actual missing requirement.

## Mistake 2 — Assuming root inside a container is harmless

Isolation reduces its access, but root increases risk, especially with mounts, capabilities, or vulnerabilities.

## Mistake 3 — Mounting the Docker socket into ordinary applications

Docker daemon control can become host-equivalent control.

## Mistake 4 — Treating all workloads as equally trusted

An internal API and an untrusted code runner require different boundaries.

## Mistake 5 — Believing containers provide high availability

Containers on one host still fail together when that host fails.

## Mistake 6 — Manually repairing production containers

The repair disappears on replacement and creates configuration drift.

## Mistake 7 — Embedding secrets in images

Secrets can remain in layers, caches, registries, and image history.

## Mistake 8 — Selecting technology from performance alone

Security, compatibility, operations, compliance, and failure domains are equally important.

---

# 25. Best Practices

- Begin with a workload threat model.
- Run as a non-root user when possible.
- Drop unnecessary capabilities.
- Avoid privileged containers.
- Treat Docker socket access as highly privileged.
- Use read-only root filesystems when compatible.
- Make writable locations explicit.
- Keep secrets out of images and logs.
- Apply seccomp and AppArmor or SELinux policies.
- Patch host kernels and runtimes promptly.
- Separate workloads by trust level.
- Spread replicas across failure domains.
- Replace containers from versioned images instead of patching them manually.
- Use VMs and containers together when their boundaries complement each other.

---

# 26. Interview Questions

## Junior

### 1. Is root inside a container the same as unrestricted host root?

Not automatically. Namespace and runtime restrictions limit it, but it remains higher risk than a non-root process.

### 2. What does privileged mode do?

It grants extensive access to devices and relaxes important container security restrictions.

### 3. Why should application containers be replaceable?

Versioned images make deployments reproducible. Manual repairs create unrecorded state that disappears when the container is replaced.

## Intermediate

### 4. Why is mounting the Docker socket dangerous?

The process may control the daemon, create privileged containers, mount host filesystems, and effectively gain host-level control.

### 5. How do capabilities improve least privilege?

They divide broad root privilege into smaller units, allowing applications to receive only selected powers.

### 6. Why does high container density increase blast radius?

One host failure or compromise can affect more workloads when many containers share that host.

### 7. Why combine read-only root filesystems with tmpfs?

The application filesystem remains immutable while explicitly selected paths such as `/tmp` remain writable and temporary.

## Senior

### 8. How would you isolate untrusted CI jobs?

Use a boundary designed for hostile code, often ephemeral VMs, microVMs, or sandboxed runtimes; remove host-control paths; restrict secrets, network, time, and resources; and destroy the environment afterward.

### 9. When are normal containers insufficient as a tenant boundary?

When tenants are hostile or unrelated, consequences are severe, or policy requires a separate kernel or stronger isolation layer.

### 10. Why is “non-root” necessary but insufficient?

Non-root reduces privilege but does not fix application vulnerabilities, kernel flaws, dangerous mounts, exposed secrets, excessive network access, or weak runtime policy.

### 11. How do failure domains influence scheduling?

Replicas should be distributed across hosts, zones, or other independent domains so one failure does not remove the complete service.

---

# 27. Lesson Reflection

Explain in your own words:

1. Why does sharing a kernel change the threat model?
2. Why is root inside a container still dangerous?
3. Why is the Docker socket more than an ordinary file?
4. Why does privileged mode hide the real permission problem?
5. Why do containers not automatically provide high availability?
6. Why might Kubernetes nodes be virtual machines?
7. Which isolation would you choose for unknown user-submitted code, and why?

---

# 28. Summary

- Containers share the host kernel, while VMs normally use separate guest kernels.
- VM boundaries generally provide stronger default isolation for hostile workloads.
- Container root is restricted by isolation controls but remains higher risk than non-root execution.
- Privileged mode, dangerous mounts, and Docker socket access can severely weaken isolation.
- Capabilities, seccomp, AppArmor or SELinux, non-root users, and read-only filesystems provide defense in depth.
- Workload trust and impact should drive isolation decisions.
- High density can increase blast radius.
- Containers provide packaging and deployment benefits; VMs provide infrastructure and kernel boundaries.
- Many production platforms deliberately use both.

---

# 29. Cheat Sheet

| Command | Purpose |
|---|---|
| `docker exec CONTAINER id` | Show the user identity inside a running container |
| `docker inspect --format '{{.Config.User}}' CONTAINER` | Show the image/runtime user configuration |
| `docker inspect --format '{{.HostConfig.Privileged}}' CONTAINER` | Check privileged mode |
| `docker inspect --format '{{json .HostConfig.CapAdd}}' CONTAINER` | Show added capabilities |
| `docker inspect --format '{{json .HostConfig.CapDrop}}' CONTAINER` | Show dropped capabilities |
| `docker inspect --format '{{json .Mounts}}' CONTAINER` | Inspect mounts |
| `docker inspect --format '{{.HostConfig.ReadonlyRootfs}}' CONTAINER` | Check read-only root filesystem configuration |
| `docker run --user UID:GID IMAGE` | Start the main process with an explicit identity |
| `docker run --read-only IMAGE` | Use a read-only container root filesystem |
| `docker run --tmpfs /tmp IMAGE` | Provide an in-memory temporary filesystem |
| `docker run --cap-drop ALL IMAGE` | Remove configurable Linux capabilities |
| `docker run --privileged IMAGE` | Grant broad host-related privilege; avoid except for reviewed special cases |

---

# 30. Challenge — Architecture Review

You are reviewing a proposed multi-tenant learning platform.

Students can submit arbitrary Linux programs. The proposal runs all submissions as root inside ordinary containers on one Docker host. Each container receives:

```text
--privileged
-v /var/run/docker.sock:/var/run/docker.sock
-v /srv/student-files:/work
```

The host also runs the platform database and authentication service.

Produce a security review containing:

1. At least eight risks.
2. The trust boundary.
3. The likely blast radius of one compromised submission.
4. Why ordinary container isolation has been weakened.
5. A safer architecture.
6. Suitable VM, microVM, sandbox, or dedicated-node boundaries.
7. Network and secret restrictions.
8. Resource and time limits.
9. Cleanup and destruction strategy.
10. Evidence required before production approval.

Do not merely say “remove privileged mode.” Design a complete isolation strategy.

---

## Lab Assets

Suggested repository locations:

```text
assets/chapter-07/lesson-02/
├── defense-in-depth.txt
├── vm-security-boundary.txt
├── container-security-boundary.txt
├── uid-mapping.txt
├── failure-domains.txt
├── decision-matrix.md
└── root-vs-non-root-lab.txt
```

---

## Next Lesson

**Lesson 03 — Docker Architecture**

We will study:

- Docker Engine
- Docker CLI
- Docker daemon
- Docker API
- containerd
- OCI runtimes
- Registries and Docker Hub
- The complete internal workflow of `docker run`

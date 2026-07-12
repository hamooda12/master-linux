# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 01 — What Docker Is

> **Level:** Complete beginner  
> **Type:** Conceptual foundation  
> **Installation required:** No  
> **Estimated study time:** 60–90 minutes

---

## Overview

Before learning Docker commands, we must understand **why Docker exists**.

Docker helps engineers package an application together with what it needs and run that package consistently on different machines.

In this lesson, you will not study advanced internals such as namespaces, cgroups, `containerd`, `runc`, OCI, capabilities, or seccomp. Those topics matter later, but they are not required to understand the purpose of Docker.

This lesson is divided into three parts:

1. The problem Docker solves
2. The basic Docker objects and workflow
3. Containers compared with virtual machines

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain why an application may work on one machine but fail on another
- Explain Docker in simple words
- Describe what an application dependency is
- Distinguish a Docker image from a container
- Identify the Docker host and container registry
- Explain the **build, ship, run** workflow
- Explain why a container is not the same as a virtual machine
- Describe where Docker helps a DevOps engineer
- Read a simple Docker workflow diagram

---

## Prerequisites

You only need a basic understanding of:

- Applications and operating systems
- Files and directories
- Processes at a simple level
- Servers at a simple level

You do **not** need:

- Docker installed
- Programming experience
- Linux kernel knowledge
- Virtualization knowledge
- Kubernetes knowledge

---

# Part 1 — Why Docker Exists

## 1. The Real Problem

Imagine that a developer creates a web application on a laptop.

The application needs:

- The application source code
- Python 3.12
- Several Python libraries
- A configuration file
- A particular environment variable
- A system library

It works perfectly on the developer's laptop.

The developer sends the application code to the operations engineer. The engineer tries to run it on a server, but it fails.

Why?

The server may have:

- Python 3.10 instead of Python 3.12
- A missing library
- A different library version
- A missing configuration file
- A different filesystem layout
- A different environment variable

This creates the famous problem:

> “It works on my machine.”

The code is the same, but the **environment around the code** is different.

---

## 2. What Is an Application Dependency?

A dependency is something an application needs in order to work.

For example, a Python application may depend on:

```text
Application code
    ├── Python 3.12
    ├── Flask 3.x
    ├── PostgreSQL client library
    ├── Configuration
    └── Other system libraries
```

If one required dependency is missing or incompatible, the application may fail even when its own code is correct.

### Simple example

Suppose the application uses a feature available only in Python 3.12.

```text
Developer laptop: Python 3.12 → application works
Production server: Python 3.10 → application fails
```

The problem is not necessarily the application code. The runtime environment is different.

---

## 3. The Old Manual Approach

Without containers, an operations engineer might prepare the server manually:

1. Install the correct programming language
2. Install the required libraries
3. Copy the application files
4. Create configuration files
5. Set environment variables
6. Start the application

This can work, but problems appear as the number of applications and servers grows.

For example:

- Application A needs Python 3.10
- Application B needs Python 3.12
- Application C needs a conflicting library version
- The test server differs from the production server
- A manual step is forgotten during deployment

The process becomes difficult to repeat consistently.

---

## 4. Docker's Basic Solution

Docker lets the team create a package containing:

- The application
- Its runtime
- Its libraries
- Its required files
- Instructions describing how it should start

That package is called a **container image**, or simply an **image**.

Docker can use the image to start the application as a **container**.

```text
Application + dependencies + startup instructions
                        ↓
                 Container image
                        ↓
               Run it with Docker
                        ↓
                  Container
```

This gives the developer, test environment, and production server a much more consistent application package.

### Important accuracy note

Docker does not package an entire physical computer, and a normal Linux container does not contain its own independent Linux kernel.

You do not need the internal explanation yet. For now, remember:

> Docker packages the application environment, while the container uses the host system's kernel.

---

## 5. What Docker Is

Docker is a platform and set of tools used to:

- Build container images
- Download and upload images
- Create and run containers
- Stop and remove containers
- Connect containers through networks
- Attach persistent storage
- Inspect and troubleshoot containerized applications

### Beginner definition

> Docker is a tool that packages an application and its dependencies into an image, then uses that image to run the application as a container.

### What Docker does not automatically solve

Docker does not automatically:

- Fix broken application code
- Make an insecure application secure
- Store important data safely unless storage is designed correctly
- Replace monitoring, backups, or testing
- Manage a large production cluster by itself

Docker solves important packaging and execution problems, but good engineering is still required.

---

# Part 2 — The Basic Docker Objects

## 6. Beginner Mental Model

Start with this model:

```text
Image     = packaged application
Container = a running instance created from that image
```

This model is intentionally simple, but it is accurate enough for your first lessons.

---

## 7. Container Image

A container image is a packaged, reusable template for an application.

It normally includes:

- Application files
- Runtime files
- Required libraries
- Default configuration or metadata
- A default startup command

An image is not normally the running application. It is the package Docker uses to create a container.

### Analogy: class and object

If you know programming:

```text
Class  → template
Object → instance created from the template

Image     → packaged template
Container → instance created from the image
```

### Analogy: cake mold

If you do not know programming:

```text
Image     → reusable mold and recipe
Container → one result created from them
```

The analogy is not perfect. A container is a managed runtime environment, not a cake. Its purpose is only to help you remember that one image can create multiple containers.

```text
                    ┌── Container 1
Image ──────────────┼── Container 2
                    └── Container 3
```

Each container has its own identity and runtime state even though all three were created from the same image.

---

## 8. Container

A container is an instance created from an image.

When Docker starts a container, the application's main process runs inside that container's isolated environment.

For now, think of a container as:

> A managed environment in which the packaged application runs.

### Is a container always running?

No.

A container can exist in different states, such as:

- Created
- Running
- Stopped
- Removed

Therefore, this sentence is more accurate:

> A container is an instance of an image. When its main process is running, the container is running.

We will study container states and the main-process relationship in later lessons.

---

## 9. Image Versus Container

| Image | Container |
| --- | --- |
| Packaged template | Instance created from an image |
| Used to create containers | Used to run an application process |
| Reusable | Has its own identity and state |
| One image can create many containers | Each container is a separate instance |
| Comparable to an application package | Comparable to one execution of that package |

### Example

Suppose you have an Nginx image.

You can create three containers from it:

```text
nginx image
    ├── company-website container
    ├── test-website container
    └── documentation-site container
```

The image is not copied manually and installed three times in the traditional way. Docker uses the same image as the basis for three separate containers.

---

## 10. Docker Host

The Docker host is the machine on which Docker runs containers.

It may be:

- Your Linux laptop
- A virtual machine
- A cloud server
- A physical server in a data center

```text
Docker host
    ├── Container A
    ├── Container B
    └── Container C
```

The word **host** refers to the system providing the resources needed by the containers, such as CPU, memory, and the operating-system kernel.

### Common misunderstanding

The Docker host is not the same thing as a container.

```text
Host      = machine running Docker
Container = application instance managed by Docker on that machine
```

---

## 11. Container Registry

A container registry is a service that stores and distributes container images.

You can think of it as a remote image store.

Examples include:

- Docker Hub
- GitHub Container Registry
- Amazon Elastic Container Registry
- Google Artifact Registry
- Azure Container Registry
- A private company registry

### Basic flow

```text
Developer builds image
          ↓
Pushes image to registry
          ↓
Server pulls image from registry
          ↓
Docker starts a container
```

### Git repository versus container registry

| Git repository | Container registry |
| --- | --- |
| Primarily stores source code and its history | Stores container images and image versions |
| Developers clone or pull code | Docker pulls images |
| Example: a repository on GitHub | Example: Docker Hub or GitHub Container Registry |

A team commonly uses both: the Git repository stores the source code, and the registry stores the built image.

---

## 12. Build, Ship, Run

Docker is often explained through three actions.

### Build

Create an image containing the application and what it needs.

```text
Source code + dependencies + instructions → image
```

### Ship

Move or distribute the image, commonly through a registry.

```text
Developer/CI system → registry → server
```

### Run

Use the image to create and start a container.

```text
Image → container → application process
```

### Complete workflow

```text
Developer writes application
          ↓
Team builds an image
          ↓
Image is stored in a registry
          ↓
Docker host downloads the image
          ↓
Docker creates a container
          ↓
Application runs inside the container
```

---

## 13. The Objects Working Together

```text
┌────────────────────┐
│ Developer or CI/CD │
└─────────┬──────────┘
          │ builds and pushes
          ▼
┌────────────────────┐
│ Container registry │
│   stores images    │
└─────────┬──────────┘
          │ host pulls image
          ▼
┌────────────────────────────┐
│ Docker host                │
│                            │
│ Image ──creates──> Container│
│                  application│
└────────────────────────────┘
```

Read the diagram from top to bottom:

1. A developer or CI/CD system builds the image
2. The image is pushed to a registry
3. A Docker host pulls the image
4. Docker creates a container from the image
5. The application runs in the container

---

# Part 3 — Containers and Virtual Machines

## 14. Why This Comparison Matters

Beginners sometimes think:

> “A container is just a small virtual machine.”

This is not correct.

Both containers and virtual machines can provide separated environments, but they achieve this differently.

---

## 15. Virtual Machine Mental Model

A virtual machine behaves like a complete computer created in software.

A VM normally includes:

- A virtual CPU, memory, disk, and network interface
- A complete guest operating system
- Its own guest kernel
- System services
- The application and dependencies

```text
Physical or host machine
        ↓
Hypervisor
        ↓
Virtual machine
    ├── Guest operating system
    ├── Guest kernel
    ├── Libraries
    └── Application
```

The hypervisor is the virtualization layer that creates and manages virtual machines.

You do not need to study hypervisor types in this Docker lesson.

---

## 16. Container Mental Model

A normal Linux container does not boot a complete guest operating system with a separate guest kernel.

Instead, containers on the same Docker host share the host's Linux kernel while keeping application environments separated.

```text
Docker host
    ├── Host Linux kernel
    ├── Container A: application + required files
    ├── Container B: application + required files
    └── Container C: application + required files
```

How Linux creates that separation is an internal topic. We will study it only after you can use containers confidently.

---

## 17. Container Versus Virtual Machine

| Area | Container | Virtual machine |
| --- | --- | --- |
| What it packages | Application environment | Complete guest machine environment |
| Kernel | Shares the host kernel | Has its own guest kernel |
| Typical startup | Usually fast | Usually slower because an OS boots |
| Typical size | Often smaller | Often larger |
| Isolation boundary | Process/container isolation | Virtual hardware and guest OS isolation |
| Common use | Packaging and deploying applications | Running complete operating systems and stronger machine boundaries |

### Important: containers are not “better VMs”

Containers and VMs solve related but different problems.

Use containers when you want efficient, repeatable application packaging and execution.

Use VMs when you need a complete guest operating system, a different kernel, or a machine-level isolation boundary.

They are also commonly used together:

```text
Physical cloud server
        ↓
Virtual machine
        ↓
Docker
        ↓
Containers
```

For example, a cloud provider may give your company a Linux VM, and your team may run Docker containers inside that VM.

---

## 18. Can a Linux Container Run a Windows Kernel?

Not by itself.

A normal Linux container expects Linux kernel features because it shares a Linux kernel. A Windows container expects a compatible Windows kernel.

This is one reason a container is not a complete VM.

Docker Desktop on Windows can run Linux containers by using a Linux environment behind the scenes, commonly through virtualization. The Linux containers then share that Linux environment's kernel—not the Windows kernel directly.

You do not need to understand Docker Desktop architecture yet. The key idea is:

> The container's operating-system requirements must be compatible with the kernel that actually runs it.

---

## 19. Why DevOps Engineers Use Docker

Docker helps a DevOps team make deployments more repeatable.

### Development

Developers can work with defined application dependencies instead of manually reproducing every environment.

### Testing

CI/CD pipelines can test the image that the team plans to deploy.

### Deployment

Servers can pull a specific image version and create containers from it.

### Rollback

The team can keep older image versions and redeploy a known version when appropriate.

### Service separation

Different services can use different application dependencies without installing all of those dependencies directly into one shared host environment.

### Important limitation

Docker improves consistency, but it does not guarantee that every environment is identical. Differences can still exist in:

- CPU architecture
- Kernel capabilities
- Available memory and CPU
- Mounted data
- Secrets and configuration
- External databases and networks
- Security policies

A strong DevOps engineer uses Docker to reduce environment differences, then verifies the differences that remain.

---

## Big Picture

You now know the first Docker relationship:

```text
Application and dependencies
            ↓ packaged into
          Image
            ↓ used to create
        Container
            ↓ runs on
        Docker host
```

A registry gives the team a place to distribute images:

```text
Build image → push to registry → pull to host → run container
```

No advanced internal mechanism is needed to understand this workflow.

---

## Commands

There are no required Docker commands in this lesson.

This is intentional. Before typing commands, you should understand the objects the commands will control.

In later lessons, you will use commands to:

- Verify Docker installation
- Download images
- Run containers
- Inspect their state
- Stop and remove them

---

## Guided Hands-on Lab — Design a Container Workflow

This lab requires only paper, a notes application, or a text editor.

### Scenario

TechCorp has created a product API.

The API needs:

- Application code
- Python 3.12
- Python libraries
- A startup instruction

TechCorp wants to run it on a test server and later on a production server.

### Task 1 — Identify the package

What should contain the application, runtime, libraries, and startup instruction?

Write your answer before checking below.

<details>
<summary>Answer</summary>

A container image.

</details>

### Task 2 — Identify the running instance

What is created from the image when TechCorp wants to run the API?

<details>
<summary>Answer</summary>

A container.

</details>

### Task 3 — Identify the storage service

Where can the team store the image so test and production servers can download it?

<details>
<summary>Answer</summary>

A container registry.

</details>

### Task 4 — Identify the execution machine

What do we call the server running Docker and its containers?

<details>
<summary>Answer</summary>

The Docker host.

</details>

### Task 5 — Draw the workflow

Complete the missing terms:

```text
Application + dependencies
          ↓ build
       [ A ]
          ↓ push
       [ B ]
          ↓ pull to
       [ C ]
          ↓ create
       [ D ]
```

<details>
<summary>Answer</summary>

```text
A = image
B = container registry
C = Docker host
D = container
```

</details>

---

## Discovery Questions

Answer these in your own words before reading the explanations.

### Question 1

Why is copying only an application's source code sometimes insufficient?

**Explanation:** The destination system may not have the correct runtime, libraries, configuration, or other dependencies.

### Question 2

If three containers are created from one image, do we have one container or three?

**Explanation:** We have three separate containers. They share the same image as their starting package but have separate identities and runtime state.

### Question 3

If a container stops, does its image automatically disappear?

**Explanation:** No. The image and container are separate Docker objects. One image can remain available and create other containers.

### Question 4

Why is a normal Linux container not a complete virtual machine?

**Explanation:** It does not contain and boot its own independent guest kernel. It shares the host's Linux kernel.

---

## Production Scenario

TechCorp has ten application servers.

Without a consistent package, engineers manually install the runtime and dependencies on every server. Over time, the servers become different:

```text
Server 1 → Python 3.12, library 2.1
Server 2 → Python 3.11, library 2.0
Server 3 → Python 3.12, library missing
```

Deployments behave differently across servers.

With Docker, TechCorp can build a versioned application image and deploy that image to the Docker hosts.

```text
product-api:1.0 image
        ├── Server 1 → container
        ├── Server 2 → container
        └── Server 3 → container
```

This does not make the servers completely identical, but it makes the application package far more consistent and traceable.

### Evidence-first DevOps question

If one container still fails, do not immediately conclude that Docker is broken.

Check evidence about:

- Which image version was used
- Application configuration
- Environment variables
- Mounted data
- Network access
- Available CPU and memory
- Application logs

You will learn how to collect this evidence in later lessons.

---

## Common Mistakes

### Mistake 1 — Calling an image a container

An image is the package. A container is an instance created from it.

### Mistake 2 — Calling a container a small VM

A container normally shares the host kernel. A VM has its own guest kernel.

### Mistake 3 — Thinking Docker packages absolutely everything

Containers still depend on host resources, kernel compatibility, configuration, external services, and infrastructure.

### Mistake 4 — Thinking every container must run forever

A container runs while its main process runs. A short task may finish successfully and the container may then stop.

### Mistake 5 — Thinking a registry runs containers

A registry stores and distributes images. A Docker host runs containers.

### Mistake 6 — Believing Docker fixes application bugs

Docker can package broken code just as consistently as working code. Testing is still required.

### Mistake 7 — Studying internals before understanding the workflow

Namespaces and cgroups eventually explain how Linux implements isolation and resource control. They are not needed to understand image → container → application.

---

## Best Practices

- Learn Docker objects before memorizing commands
- Use the words **image** and **container** accurately
- Treat images as versioned application artifacts
- Keep application source code in version control
- Store distributable images in an appropriate registry
- Test the same image that you intend to deploy
- Remember that configuration and persistent data need separate design
- Verify application behavior after deployment
- Add internal knowledge only after understanding the visible behavior

---

## Interview Questions

### Junior Level

1. What problem does Docker solve?
2. What is a container image?
3. What is a container?
4. What is the difference between an image and a container?
5. Can one image create multiple containers?
6. What is a Docker host?
7. What is a container registry?
8. What do build, ship, and run mean?
9. Is a container the same as a virtual machine?
10. Does a normal Linux container have its own kernel?

### Suggested concise answers

1. Docker helps package and run applications with their dependencies consistently.
2. An image is a packaged, reusable application template.
3. A container is an instance created from an image.
4. The image is the package; the container is an instance with its own state.
5. Yes.
6. It is the machine on which Docker runs containers.
7. It is a service that stores and distributes container images.
8. Build creates an image, ship distributes it, and run creates a container from it.
9. No. They use different models and isolation boundaries.
10. No. A normal Linux container shares the host's Linux kernel.

### Intermediate Preview

These questions are previews. You are not expected to give deep answers yet.

1. What environment differences can remain even when the same image is deployed?
2. Why might a team run containers inside cloud virtual machines?
3. Why should teams version their images?

There are no senior-level questions yet because the lesson has not taught internal or production implementation depth.

---

## Lesson Reflection

Answer without copying the lesson:

1. In your own words, why was Docker created?
2. Give three examples of application dependencies.
3. Explain image versus container using your own analogy.
4. If an image is stored in a registry, is the application already running? Why?
5. What is the role of the Docker host?
6. Why is a Linux container not a complete virtual machine?
7. What parts of an environment can still differ after using Docker?
8. Describe build, ship, and run in one sentence each.

---

## Summary

- Applications can fail when their environments and dependencies differ
- Docker packages an application and its dependencies into an image
- An image is a reusable package or template
- A container is an instance created from an image
- One image can create multiple containers
- The Docker host is the machine that runs Docker and containers
- A registry stores and distributes images
- The basic workflow is build image → ship image → run container
- A Linux container normally shares the host's kernel
- A virtual machine has a complete guest operating system and guest kernel
- Docker improves consistency but does not replace testing, security, monitoring, or good infrastructure design

---

## Cheat Sheet

### Core vocabulary

| Term | Meaning |
| --- | --- |
| Dependency | Something an application needs to work |
| Docker | Tools and platform for building images and running containers |
| Image | Packaged, reusable application template |
| Container | Instance created from an image |
| Docker host | Machine on which Docker runs containers |
| Registry | Service that stores and distributes images |
| Build | Create an image |
| Ship | Distribute an image, commonly through a registry |
| Run | Create and start a container from an image |
| VM | Software-defined machine with a complete guest OS and kernel |

### Essential relationships

```text
Application + dependencies → image
Image → container
Registry → stores images
Docker host → runs containers
Linux container → shares host Linux kernel
VM → has its own guest kernel
```

### Commands learned

```text
None — this lesson establishes the mental model first.
```

---

## Challenge — Explain Docker to a New Employee

You are the junior DevOps engineer at TechCorp.

A new developer asks:

> “Why can't I just send you my application code? What are images, containers, hosts, and registries?”

Write a short answer that:

1. Explains the dependency problem
2. Defines an image
3. Defines a container
4. Explains the registry's role
5. Explains the Docker host's role
6. Corrects the statement “a container is just a small VM”

### Acceptance criteria

Your answer must correctly use these words:

```text
dependency
image
container
registry
Docker host
kernel
```

Do not use advanced terms such as namespace, cgroup, OCI, or `runc`.

---

## Lab Assets

Save the following items if you are maintaining the Linux Practice repository:

```text
assets/chapter-07/lesson-01/
├── docker-workflow-diagram.txt
├── image-vs-container-notes.md
├── container-vs-vm-comparison.md
├── techcorp-lab-answers.md
└── lesson-reflection.md
```

Suggested evidence:

- Your completed TechCorp workflow
- Your own image-versus-container analogy
- Your container-versus-VM explanation
- Your answers to the reflection questions
- Your completed challenge response

No terminal screenshot is required because Docker is not installed or executed in this lesson.

---

## Next Lesson

**Lesson 02 — Installing Docker on Ubuntu**

Now that you understand what Docker, images, containers, hosts, and registries are, the next lesson will install Docker safely and prove that its client and service work.

It will also directly address this common error:

```text
permission denied while trying to connect to the Docker API at unix:///var/run/docker.sock
```

You will learn why it happens and choose deliberately between:

- Running Docker commands with `sudo`
- Docker group membership, with its security implications
- Rootless Docker at an introductory level

Advanced container internals will still wait until the correct phase.

---

## Completion Check

Before moving to Lesson 02, confirm that you can explain this without reading:

```text
Why Docker exists
What an image is
What a container is
What a registry does
What a Docker host does
Why a container is not a VM
How build → ship → run works
```

If you can explain all seven points in your own words, you are ready for installation.

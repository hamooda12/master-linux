# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 04 — Container Lifecycle

> **Level:** Beginner  
> **Type:** Guided practical lesson  
> **Primary lab image:** `nginx:alpine`  
> **Estimated study time:** 120–150 minutes

---

## Overview

In Lesson 03, `hello-world` created a container, ran a short program, and stopped automatically when that program exited.

Now you will manage a long-running container deliberately.

You will move an Nginx container through these states:

```text
Created → Running → Paused → Running → Stopped → Running → Removed
```

The goal is not to memorize commands. The goal is to understand what changes—and what does not change—after every operation.

You will learn the critical differences between:

- `docker run` and `docker create` plus `docker start`
- `docker stop` and `docker kill`
- Stopping and removing
- `docker rm` and `docker rmi`
- Restarting an existing container and running a new container

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Describe the main container lifecycle states
- Create a container without starting it
- Start an existing created or stopped container
- Stop a container gracefully
- Explain the grace period used by `docker stop`
- Kill a container immediately when necessary
- Explain why `docker kill` is not the normal first choice
- Pause and unpause a container
- Restart an existing container
- Rename a container safely
- Remove stopped containers
- Explain forced removal and why it should be used carefully
- Prove whether a command reused an existing container or created a new one
- Distinguish container identity from image identity
- Use inspection evidence before and after every lifecycle change

---

## Prerequisites

You should understand:

- Image versus container
- `docker run`
- Local image pull behavior
- The container's main-process relationship
- `docker ps` versus `docker ps -a`
- Exit code zero
- Stopped versus removed at an introductory level

Your Docker access model must already work.

Examples use:

```bash
docker ...
```

If you deliberately selected explicit `sudo`, use:

```bash
sudo docker ...
```

---

## Why This Topic Exists

Real applications do not all behave like `hello-world`.

An Nginx web server normally remains running and waits for requests. During operations, engineers may need to:

- Create it now but start it later
- Stop it for maintenance
- Start the same container again
- Restart it after configuration changes
- Temporarily pause its processes
- Terminate it when graceful shutdown fails
- Remove it when it is no longer needed

Every action has a different meaning.

Using the wrong operation can:

- Interrupt requests
- Lose the chance for graceful cleanup
- Remove troubleshooting evidence
- Accidentally delete the wrong object
- Create duplicate containers

---

# Part 1 — The Lifecycle Mental Model

## 1. Container States

The states you will use first are:

| State | Beginner meaning |
| --- | --- |
| Created | Container object exists, but its main process has not started |
| Running | Main process is running |
| Paused | Container processes are temporarily suspended |
| Stopped/Exited | Main process is no longer running, but container still exists |
| Restarting | Docker is stopping/starting it or a restart policy is acting |
| Removed | Container object no longer exists |

### Important: removed is not a normal inspectable state

Once removed, the container no longer appears in `docker ps -a` and cannot be inspected by its old container ID through normal Docker commands.

So this is conceptually useful:

```text
Existing container states → created, running, paused, exited
No longer exists          → removed
```

## 2. State Diagram

```text
Image
  │ docker create
  ▼
Created
  │ docker start
  ▼
Running ──docker pause──> Paused
  ▲                         │
  │                         │ docker unpause
  └─────────────────────────┘
  │
  ├──docker stop──> Exited
  │                    │
  ├──docker kill──> Exited
  │                    │ docker start
  └<───────────────────┘

Exited ──docker rm──> Removed
```

## 3. Container Identity Across States

Stopping, starting, pausing, unpausing, restarting, and renaming do not create a new container.

The container keeps the same full ID.

```text
techcorp-web created  → ID abc123...
techcorp-web started  → ID abc123...
techcorp-web stopped  → ID abc123...
techcorp-web started  → ID abc123...
```

But:

```bash
docker run nginx:alpine
```

creates another new container with another ID.

## 4. Main Process Relationship

The container is running while its main process is running.

For the Nginx image, the main process runs in the foreground so Docker can track it.

```text
Nginx main process running → container running
Nginx main process exits   → container stopped
```

Running a service as a background daemon inside the container can cause the container to stop if the foreground main process exits. Container images are normally designed so the primary application stays in the foreground.

---

# Part 2 — Create Without Starting

## 5. Pull the Lab Image

```bash
docker pull nginx:alpine
```

This makes the image available before the lifecycle experiment.

Verify:

```bash
docker image ls nginx
```

## 6. Create a Named Container

```bash
docker create --name techcorp-web nginx:alpine
```

Expected output:

```text
<long-container-id>
```

### Syntax

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### Options and arguments

```text
create          → create a container but do not start it
--name          → assign a human-readable container name
techcorp-web    → chosen container name
nginx:alpine    → source image and tag
```

## 7. Verify Created State

```bash
docker ps
```

The container does not appear because it is not running.

```bash
docker ps -a --filter name=techcorp-web
```

You should see a status similar to:

```text
Created
```

## 8. What Exists in Created State?

Docker has created:

- Container identity
- Container name
- Configuration based on the image and options
- Container metadata
- A writable container layer

But the main Nginx process has not started.

## 9. Inspect the State Directly

```bash
docker inspect --format '{{.State.Status}}' techcorp-web
```

Expected:

```text
created
```

Record the full container ID:

```bash
docker inspect --format '{{.Id}}' techcorp-web
```

Save it. You will prove the identity remains the same.

---

# Part 3 — Start an Existing Container

## 10. Start It

```bash
docker start techcorp-web
```

Expected output:

```text
techcorp-web
```

`docker start` starts an existing created or stopped container.

It does not create a new container.

## 11. Verify Running State

```bash
docker ps --filter name=techcorp-web
docker inspect --format '{{.State.Status}}' techcorp-web
```

Expected status:

```text
running
```

## 12. Prove Identity Did Not Change

```bash
docker inspect --format '{{.Id}}' techcorp-web
```

Compare it with the ID recorded after `docker create`.

They should match.

## 13. `docker run` Versus `create` Plus `start`

These workflows reach a running container differently:

```text
docker run IMAGE
    └── create new container + start it

docker create IMAGE
    └── create new container only

docker start CONTAINER
    └── start an existing container
```

Conceptually:

```text
docker run ≈ docker create + docker start
```

The exact CLI behavior and attachment defaults can differ, but this mental model correctly explains the lifecycle relationship.

## 14. Why Separate Create and Start?

It is useful when you want to:

- Prepare container configuration before execution
- Create several containers, then start them later
- Inspect the created configuration
- Separate provisioning from activation in automation

Most everyday commands use `docker run`, but understanding the separate stages removes confusion.

---

# Part 4 — Pause and Unpause

## 15. Pause the Container

```bash
docker pause techcorp-web
```

Expected output:

```text
techcorp-web
```

## 16. Verify Paused State

```bash
docker ps --filter name=techcorp-web
docker inspect --format '{{.State.Status}}' techcorp-web
```

Typical `docker ps` status:

```text
Up ... (Paused)
```

Direct state:

```text
paused
```

## 17. What Pause Means

Pause temporarily suspends all processes in the container.

It does not:

- Gracefully stop the application
- Remove the container
- Create a new container
- Discard its memory state

Beginner mental model:

```text
Running processes → frozen in place → later continue
```

### Internal preview

On Linux, Docker implements pause using the freezer functionality of control groups. You are not expected to understand cgroups yet; they receive a full lesson later.

## 18. Operational Impact

A paused application does not process normal work while suspended.

Network clients may wait or time out. Therefore, pause is not equivalent to healthy maintenance mode.

Use it only when you understand the application and operational effect.

## 19. Unpause

```bash
docker unpause techcorp-web
```

Verify:

```bash
docker inspect --format '{{.State.Status}}' techcorp-web
```

Expected:

```text
running
```

The same processes continue rather than a new container being created.

---

# Part 5 — Graceful Stop

## 20. Stop the Container

```bash
docker stop techcorp-web
```

Expected output:

```text
techcorp-web
```

## 21. What `docker stop` Does

By default, Docker sends the container's configured stop signal to the main process. Commonly this is `SIGTERM`.

Docker then waits for a grace period.

If the process has not exited when the grace period ends, Docker sends `SIGKILL`.

```text
docker stop
     ↓
Send graceful termination signal
     ↓
Application gets time to clean up
     ↓
Did it exit?
  ├── Yes → stopped
  └── No  → SIGKILL after timeout
```

The image may define another stop signal, and the container may be configured with one. Do not assume every application receives the same first signal without checking its configuration.

## 22. Why Graceful Shutdown Matters

A well-designed application may need time to:

- Finish active requests
- Stop accepting new work
- Flush buffers
- Close files
- Close database connections
- Write final logs
- Release locks

Immediate termination can interrupt these actions.

## 23. Verify Stopped State

```bash
docker ps --filter name=techcorp-web
docker ps -a --filter name=techcorp-web
docker inspect --format '{{.State.Status}}' techcorp-web
docker inspect --format '{{.State.ExitCode}}' techcorp-web
```

The container disappears from running-only `docker ps`, remains in `docker ps -a`, and reports state `exited`.

## 24. Custom Stop Timeout

```bash
docker stop --timeout 20 techcorp-web
```

This allows up to 20 seconds before Docker forcefully terminates a process that has not exited.

Shortening the timeout can interrupt cleanup. Increasing it can delay deployments. Choose it based on application shutdown behavior and the surrounding platform's termination window.

## 25. Start the Same Container Again

```bash
docker start techcorp-web
```

Verify the full ID:

```bash
docker inspect --format '{{.Id}}' techcorp-web
```

It is the same container identity.

Starting again runs the configured main command again within that existing container object.

---

# Part 6 — Kill

## 26. Kill the Container

Ensure it is running:

```bash
docker start techcorp-web
```

Then:

```bash
docker kill techcorp-web
```

By default, `docker kill` sends `SIGKILL` to the container's main process.

## 27. What `SIGKILL` Means

`SIGKILL` immediately terminates the target process and cannot be caught or ignored by that process.

The application does not receive its normal graceful-shutdown opportunity.

```text
docker kill
     ↓
SIGKILL by default
     ↓
Immediate termination
```

## 28. Stop Versus Kill

| `docker stop` | `docker kill` default |
| --- | --- |
| Requests graceful termination first | Sends `SIGKILL` immediately |
| Waits for a grace period | Does not wait for graceful cleanup |
| Escalates if necessary | Forceful from the beginning |
| Normal operational choice | Emergency/tool-specific choice |

Use `docker stop` for normal shutdown.

Use `docker kill` when immediate termination is genuinely required or when deliberately testing signal behavior.

## 29. Custom Signal with `docker kill`

```bash
docker kill --signal=SIGHUP techcorp-web
```

Despite the command name, a custom signal is not necessarily fatal. The result depends on how the process handles that signal.

This is an important detail:

> `docker kill` defaults to a killing signal, but `--signal` can select another signal that may not stop the container.

Do not send application signals without knowing their documented meaning.

## 30. Inspect After Kill

```bash
docker ps -a --filter name=techcorp-web
docker inspect --format '{{.State.Status}} exit={{.State.ExitCode}}' techcorp-web
```

The non-zero exit code after a default kill reflects abnormal forced termination. Exact status details should be read from the actual output rather than guessed.

---

# Part 7 — Restart

## 31. Restart the Container

Start it if needed:

```bash
docker start techcorp-web
```

Then:

```bash
docker restart techcorp-web
```

`docker restart` stops and then starts the existing container.

It does not create a new container.

## 32. Verify Restart

```bash
docker inspect --format 'id={{.Id}} status={{.State.Status}} started={{.State.StartedAt}}' techcorp-web
```

The full container ID remains the same, while the start timestamp changes.

## 33. Restart Is Not Recreate

```text
Restart:
same container ID → main process starts again

Recreate:
old container removed → new container ID created from image
```

This matters when configuration changes.

Many container settings are fixed when the container is created. Restarting does not rebuild the container with new `docker run` options. To change creation-time configuration, you commonly replace/recreate the container.

## 34. Restart Timeout

```bash
docker restart --timeout 20 techcorp-web
```

The stop phase uses a termination timeout before the start phase.

Do not confuse manually running `docker restart` with a configured restart policy. Restart policies are taught later.

---

# Part 8 — Rename

## 35. Rename the Container

```bash
docker rename techcorp-web techcorp-nginx
```

Syntax:

```bash
docker rename OLD_NAME NEW_NAME
```

## 36. Verify Identity

```bash
docker ps -a --filter name=techcorp-nginx
docker inspect --format '{{.Id}}' techcorp-nginx
```

Renaming changes the human-readable name, not the container ID.

## 37. Why Names Matter

Names are easier for humans and scripts to use than long IDs.

Choose names that express purpose, such as:

```text
techcorp-nginx
product-api
billing-worker
```

Container names must be unique within the same Docker daemon.

Production orchestration platforms often manage workload names differently, so do not build an entire service-discovery design around ad hoc Docker names.

---

# Part 9 — Remove

## 38. Stop Before Normal Removal

```bash
docker stop techcorp-nginx
```

Then remove:

```bash
docker rm techcorp-nginx
```

Expected output:

```text
techcorp-nginx
```

## 39. Verify Removal

```bash
docker ps -a --filter name=techcorp-nginx
```

No matching container should appear.

Trying to inspect it should produce a not-found error:

```bash
docker inspect techcorp-nginx
```

## 40. The Image Still Exists

```bash
docker image ls nginx
```

The `nginx:alpine` image remains available.

```text
docker rm  CONTAINER → remove container
docker rmi IMAGE     → remove image
```

## 41. Why a Running Container Normally Cannot Be Removed

If you try:

```bash
docker rm RUNNING_CONTAINER
```

Docker normally refuses because the container is running.

This safety boundary makes you choose explicitly:

1. Stop it gracefully
2. Verify its stopped state
3. Remove it

## 42. Force Removal

```bash
docker rm --force CONTAINER
```

Short form:

```bash
docker rm -f CONTAINER
```

This forcefully removes a running container. It uses `SIGKILL` to terminate the main process before removal.

It is disruptive and removes evidence quickly.

Prefer:

```bash
docker stop CONTAINER
docker rm CONTAINER
```

unless immediate force removal is deliberately required.

## 43. Remove Several Stopped Containers

```bash
docker rm CONTAINER_1 CONTAINER_2
```

Docker also provides:

```bash
docker container prune
```

This removes **all stopped containers** after confirmation. It is too broad for casual cleanup on a shared or important host.

Inspect first:

```bash
docker ps -a --filter status=exited
```

Target individual containers when you are learning or when evidence may matter.

---

# Part 10 — `rm` Versus `rmi`

## 44. Container Object Versus Image Object

```text
nginx:alpine image
       │
       ├── container A
       ├── container B
       └── container C
```

Removing container A does not remove the source image or containers B and C.

## 45. Commands

```bash
docker rm CONTAINER
```

Removes a container object.

```bash
docker rmi IMAGE
```

Removes an image reference/content when Docker determines it can be removed.

Structured image form:

```bash
docker image rm IMAGE
```

## 46. Why Image Removal May Be Refused

Docker may refuse to remove an image that is still referenced by an existing container.

Even a stopped container can reference its source image.

Evidence:

```bash
docker ps -a --filter ancestor=nginx:alpine
```

Do not use forced image removal without understanding the references.

---

## Big Picture

```text
                    docker pause
                ┌────────────────> Paused
                │                    │
                │ docker unpause     │
                <────────────────────┘
                │
Image → Created → Running → Exited → Removed
        create     │  ▲       │        rm
                   │  │       │
                   │  └─start─┘
                   │
                   ├─stop: graceful first
                   └─kill: SIGKILL by default
```

Identity rule:

```text
start / stop / pause / unpause / restart / rename
                     ↓
             same container ID

docker run
     ↓
new container ID
```

---

## Commands — Detailed Reference

## 47. `docker create`

```bash
docker create --name NAME IMAGE
```

Creates a container without starting it.

## 48. `docker start`

```bash
docker start CONTAINER
```

Starts one or more existing stopped/created containers.

It does not create a new container.

## 49. `docker stop`

```bash
docker stop CONTAINER
docker stop --timeout 20 CONTAINER
docker stop --signal SIGTERM CONTAINER
```

Requests termination and waits before forceful termination.

## 50. `docker kill`

```bash
docker kill CONTAINER
docker kill --signal=SIGHUP CONTAINER
```

Sends `SIGKILL` by default or a selected signal.

## 51. `docker pause`

```bash
docker pause CONTAINER
```

Suspends all processes in the container.

## 52. `docker unpause`

```bash
docker unpause CONTAINER
```

Resumes the suspended processes.

## 53. `docker restart`

```bash
docker restart CONTAINER
docker restart --timeout 20 CONTAINER
```

Stops and starts the same container.

## 54. `docker rename`

```bash
docker rename OLD_NAME NEW_NAME
```

Changes the name without changing the ID.

## 55. `docker rm`

```bash
docker rm CONTAINER
docker rm CONTAINER_1 CONTAINER_2
docker rm --force CONTAINER
```

Removes containers. Normal removal expects them to be stopped.

## 56. `docker ps`

```bash
docker ps
docker ps -a
docker ps -a --filter name=NAME
docker ps -a --filter status=exited
```

Lists and filters containers.

## 57. State Inspection

```bash
docker inspect --format '{{.State.Status}}' CONTAINER
docker inspect --format '{{.State.ExitCode}}' CONTAINER
docker inspect --format '{{.Id}}' CONTAINER
```

Use inspection to prove state and identity rather than relying only on assumptions.

---

## Guided Hands-on Lab — Control One Container

### Scenario

TechCorp needs an Nginx container prepared, activated, paused during a controlled test, stopped for maintenance, restarted, renamed, and finally retired.

Your rule:

> After every state-changing command, verify state and identity.

### Phase A — Clean target name only

Check whether the target already exists:

```bash
docker ps -a --filter name=techcorp-web
```

If an unrelated or important container uses the name, choose another lab name. Never remove it blindly.

### Phase B — Create

```bash
docker pull nginx:alpine
docker create --name techcorp-web nginx:alpine
docker ps -a --filter name=techcorp-web
docker inspect --format 'id={{.Id}} status={{.State.Status}}' techcorp-web
```

Save the full ID.

### Phase C — Start

```bash
docker start techcorp-web
docker ps --filter name=techcorp-web
docker inspect --format 'id={{.Id}} status={{.State.Status}}' techcorp-web
```

Prove the ID did not change.

### Phase D — Pause

```bash
docker pause techcorp-web
docker ps --filter name=techcorp-web
docker inspect --format '{{.State.Status}}' techcorp-web
```

### Phase E — Unpause

```bash
docker unpause techcorp-web
docker inspect --format '{{.State.Status}}' techcorp-web
```

### Phase F — Graceful stop

```bash
docker stop techcorp-web
docker ps --filter name=techcorp-web
docker ps -a --filter name=techcorp-web
docker inspect --format 'status={{.State.Status}} exit={{.State.ExitCode}}' techcorp-web
```

### Phase G — Start existing container

```bash
docker start techcorp-web
docker inspect --format 'id={{.Id}} status={{.State.Status}}' techcorp-web
```

Prove it is the original ID.

### Phase H — Default kill

```bash
docker kill techcorp-web
docker inspect --format 'status={{.State.Status}} exit={{.State.ExitCode}}' techcorp-web
```

Compare this exit behavior with graceful stop.

### Phase I — Restart

```bash
docker start techcorp-web
docker restart techcorp-web
docker inspect --format 'id={{.Id}} status={{.State.Status}} started={{.State.StartedAt}}' techcorp-web
```

### Phase J — Rename

```bash
docker rename techcorp-web techcorp-nginx
docker ps -a --filter name=techcorp-nginx
docker inspect --format 'id={{.Id}} name={{.Name}}' techcorp-nginx
```

### Phase K — Remove safely

```bash
docker stop techcorp-nginx
docker rm techcorp-nginx
docker ps -a --filter name=techcorp-nginx
docker image ls nginx
```

### Acceptance Criteria

You must prove:

- Created state existed before starting
- Starting preserved the container ID
- Pausing changed state without creating a new container
- Unpausing resumed the same container
- Stopping left an exited container
- Starting again reused the existing container
- Default kill was forceful
- Restart preserved container ID
- Rename preserved container ID
- Normal removal happened only after stop
- Removal deleted the container but not the image

---

## Discovery Questions

### Question 1

Does `docker start techcorp-web` create another container?

<details>
<summary>Explanation</summary>

No. It starts the existing container and preserves its ID.

</details>

### Question 2

Why does `docker stop` wait before sending `SIGKILL`?

<details>
<summary>Explanation</summary>

The grace period gives the application an opportunity to handle its termination signal and clean up safely.

</details>

### Question 3

Is a paused container stopped?

<details>
<summary>Explanation</summary>

No. Its processes are suspended, not exited. Unpause resumes them.

</details>

### Question 4

Does `docker restart` read new `docker run` options?

<details>
<summary>Explanation</summary>

No. It restarts the existing container with its existing creation-time configuration. Configuration requiring different creation options normally requires recreation.

</details>

### Question 5

Why can an image still exist after its container is removed?

<details>
<summary>Explanation</summary>

Images and containers are separate Docker objects. Removing a container does not remove its source image.

</details>

---

## Production Scenario — Graceful Shutdown Failure

TechCorp deploys an API container. During release, the engineer uses:

```bash
docker kill product-api
```

Active requests fail and a buffered operation is lost.

The correct normal workflow would have been:

```bash
docker stop --timeout 30 product-api
```

provided the application:

- Handles the configured stop signal
- Stops accepting new work
- Completes or safely abandons in-flight work
- Exits within the allowed time

The incident investigation should ask:

1. Which signal was sent?
2. Did the application handle it?
3. What stop timeout was configured?
4. Did the process exit before escalation?
5. Were upstream requests drained first?
6. What did application logs show?

Docker can request graceful shutdown, but the application must implement graceful behavior.

---

## Troubleshooting Workflow

## 58. Name Already in Use

Symptom:

```text
Conflict. The container name ... is already in use
```

Investigate:

```bash
docker ps -a --filter name=techcorp-web
docker inspect techcorp-web
```

Choose whether to reuse, rename, or safely remove the existing container. Do not force-delete it merely to reuse a name.

## 59. Start Fails

Check:

```bash
docker ps -a --filter name=CONTAINER
docker inspect --format '{{json .State}}' CONTAINER
docker logs CONTAINER
```

Possible causes include command failure, port conflicts, mount problems, or invalid runtime configuration. These topics come later.

## 60. Stop Takes Too Long

Check:

- Application shutdown logs
- Configured stop signal
- Stop timeout
- Whether the main process handles the signal
- Whether it is blocked on external work

Do not immediately reduce the timeout or use kill without understanding the cleanup requirement.

## 61. Remove Fails

If Docker says the container is running:

```bash
docker inspect --format '{{.State.Status}}' CONTAINER
docker stop CONTAINER
docker rm CONTAINER
```

Force removal is a deliberate escalation.

## 62. Image Removal Fails

Find referencing containers:

```bash
docker ps -a --filter ancestor=IMAGE
```

Do not confuse an image reference conflict with a broken daemon.

---

## Common Mistakes

### 1. Using `docker run` to restart a stopped container

`docker run` creates another container. Use `docker start` for the existing one.

### 2. Treating pause as graceful maintenance mode

Paused processes cannot serve work; clients may time out.

### 3. Using kill for normal shutdown

Default kill prevents graceful application cleanup.

### 4. Assuming stop is always gentle forever

After its grace period, Docker escalates to `SIGKILL` if the process remains.

### 5. Assuming restart recreates configuration

Restart reuses the existing container configuration.

### 6. Removing before inspecting

Removal deletes container metadata needed for troubleshooting.

### 7. Using `docker rm -f` as routine cleanup

It forcefully terminates running processes and removes evidence.

### 8. Confusing `rm` and `rmi`

One removes containers; the other removes images.

### 9. Using `docker container prune` without reviewing targets

It removes all stopped containers, which may include valuable evidence.

### 10. Assuming a new name means a new container

Rename changes the name, not the container ID.

---

## Best Practices

- Give operationally useful names to manually managed containers
- Verify state after every lifecycle action
- Record container ID when proving identity across operations
- Use graceful stop for normal shutdown
- Match stop timeout to documented application behavior
- Reserve kill and force removal for justified situations
- Inspect logs and state before removal
- Avoid broad prune commands on important hosts without review
- Treat restart and recreation as different operations
- Design applications to handle termination signals correctly
- Let automation and orchestrators manage lifecycle consistently in production
- Document why a forceful action was required

---

## Interview Questions

### Junior

1. What is the difference between `docker create` and `docker run`?
2. What does `docker start` do?
3. Does `docker start` create a new container?
4. What are the created, running, paused, and exited states?
5. What does `docker stop` do?
6. What is the default behavior of `docker kill`?
7. What is the difference between stop and kill?
8. Does restart create a new container?
9. Does rename change the container ID?
10. What is the difference between `docker rm` and `docker rmi`?

### Intermediate

1. Why might graceful stop still end in `SIGKILL`?
2. How would you prove that restart preserved identity?
3. Why might pause cause client timeouts?
4. When must a container be recreated instead of restarted?
5. Why can a stopped container block image removal?
6. What evidence should you collect before force removal?
7. How should application shutdown time relate to deployment termination time?

### Senior Preview

Signal forwarding through shells, PID 1 responsibilities, restart policies, orchestrator termination flows, and zero-downtime draining are deferred until their supporting lessons.

---

## Lesson Reflection

Answer in your own words:

1. Draw the container lifecycle from image to removal.
2. Explain `run` versus `create` plus `start`.
3. Which commands preserve the container ID?
4. What happens during `docker stop`?
5. Why is default `docker kill` forceful?
6. Why is pause not the same as stop?
7. Explain restart versus recreate.
8. Why should you inspect before remove?
9. Explain `rm` versus `rmi`.
10. When would forced removal be justified?

---

## Summary

- `docker create` creates without starting
- `docker start` starts an existing container
- `docker run` creates a new container and starts it
- Running state depends on the main process
- Pause suspends container processes without stopping them
- Unpause resumes the same processes
- Stop requests graceful termination and later escalates if necessary
- Kill sends `SIGKILL` by default
- Restart stops and starts the same container
- Rename changes the name but preserves the ID
- Stopped containers still exist
- Normal removal expects a stopped container
- Force removal is disruptive
- `docker rm` removes containers
- `docker rmi` or `docker image rm` removes images
- Evidence should be collected before destructive cleanup

---

## Cheat Sheet

### Lifecycle

```text
docker create → Created
docker start  → Running
docker pause  → Paused
docker unpause→ Running
docker stop   → Exited (graceful first)
docker kill   → Exited (SIGKILL by default)
docker restart→ Same container stopped + started
docker rm     → Removed
```

### Commands

```bash
# Create but do not start
docker create --name techcorp-web nginx:alpine

# Start an existing container
docker start techcorp-web

# Stop gracefully
docker stop techcorp-web
docker stop --timeout 20 techcorp-web

# Force immediate termination by default
docker kill techcorp-web

# Suspend and resume processes
docker pause techcorp-web
docker unpause techcorp-web

# Stop and start same container
docker restart techcorp-web

# Rename without changing ID
docker rename techcorp-web techcorp-nginx

# Remove stopped container
docker rm techcorp-nginx

# Force-remove running container—use carefully
docker rm --force CONTAINER

# Inspect state and identity
docker inspect --format '{{.State.Status}}' CONTAINER
docker inspect --format '{{.State.ExitCode}}' CONTAINER
docker inspect --format '{{.Id}}' CONTAINER
```

### Critical differences

| Operation | Creates new container? | Graceful? | Removes object? |
| --- | --- | --- | --- |
| `run` | Yes | Not a shutdown action | No |
| `start` | No | Not a shutdown action | No |
| `stop` | No | Yes, then escalates | No |
| `kill` default | No | No | No |
| `restart` | No | Uses stop/start behavior | No |
| `rm` | No | Expects stopped | Yes |
| `rm -f` | No | Forceful | Yes |

---

## Challenge — Prove Identity Through the Lifecycle

### Scenario

Your senior engineer says:

> “I believe every start and restart creates a new container. Prove or disprove it using evidence.”

### Requirements

Use one `nginx:alpine` container named:

```text
lifecycle-challenge
```

Perform:

1. Create without starting
2. Record full ID and state
3. Start and record ID
4. Pause and record ID/state
5. Unpause and record ID/state
6. Stop and record ID/state/exit code
7. Start again and record ID
8. Restart and record ID/start time
9. Rename to `lifecycle-proven`
10. Record ID again
11. Stop and remove safely
12. Prove the image still exists

### Rules

- No `docker rm -f`
- No prune command
- Verify after every mutation
- Do not delete another user's container
- Explain stop versus kill even though the challenge uses safe stop

### Final conclusion template

```text
Original container ID:
ID after start:
ID after pause/unpause:
ID after stop/start:
ID after restart:
ID after rename:

Operations that preserved identity:

Operation that removed identity:

Why docker run would be different:

Why the image remained:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-04/
├── 01-created-state.txt
├── 02-original-id.txt
├── 03-running-state.txt
├── 04-paused-state.txt
├── 05-unpaused-state.txt
├── 06-graceful-stop.txt
├── 07-kill-result.txt
├── 08-restart-result.txt
├── 09-renamed-container.txt
├── 10-removal-proof.txt
├── 11-image-still-present.txt
└── lifecycle-analysis.md
```

Suggested capture:

```bash
docker inspect --format 'id={{.Id}} status={{.State.Status}}' techcorp-web \
  > 03-running-state.txt
```

Do not store secrets or unrelated system information.

---

## Official References

- [Docker container command reference](https://docs.docker.com/reference/cli/docker/container/)
- [Docker container stop](https://docs.docker.com/reference/cli/docker/container/stop/)
- [Docker container kill](https://docs.docker.com/reference/cli/docker/container/kill/)
- [Docker container pause](https://docs.docker.com/reference/cli/docker/container/pause/)
- [Docker container remove](https://docs.docker.com/reference/cli/docker/container/rm/)

---

## Next Lesson

**Lesson 05 — Foreground, Detached, and Interactive Containers**

So far, we have focused on container state. Next, we will focus on the relationship between your terminal and container processes.

You will learn:

```bash
docker run IMAGE
docker run -d IMAGE
docker run -it IMAGE sh
docker attach
docker exec
docker logs
```

And you will understand:

- Foreground mode
- Detached mode
- Standard input, output, and error
- Interactive input
- Pseudo-terminals
- Entering an existing running container
- Exiting a shell versus detaching from a container
- Why `docker exec` and `docker attach` are different

---

## Completion Check

Before moving forward, explain without reading:

```text
Created versus running
Run versus start
Pause versus stop
Stop versus kill
Restart versus recreate
Stopped versus removed
rm versus rmi
Why container ID proves identity
Why graceful shutdown depends on the application
Why force removal is an escalation
```

If you can demonstrate those differences safely with one Nginx container, you are ready to study foreground, detached, and interactive execution.

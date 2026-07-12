# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 03 — Your First Container

> **Level:** Beginner  
> **Type:** Guided practical lesson  
> **Estimated study time:** 90–120 minutes

---

## Overview

Docker is installed and the client can communicate with the daemon. Now you are ready to run and understand your first container.

You may already have used:

```bash
docker run hello-world
```

But using a command is not the same as understanding it.

In this lesson, we will slow the operation down and answer:

- What does `docker run` request?
- Why does Docker sometimes download an image?
- What is created from that image?
- What program runs inside the container?
- Why does `hello-world` stop almost immediately?
- Is a stopped container a failed container?
- Why is the second run different from the first?
- What remains after the program exits?

We will stay at the visible Docker-behavior level. Deep runtime internals still wait until later.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain the beginner workflow performed by `docker run`
- Distinguish an image from a container using real command output
- Explain Docker's local image check
- Explain why an image is pulled when it is missing
- Use `docker ps` and `docker ps -a`
- Use `docker image ls` and recognize important columns
- Explain why `hello-world` exits successfully
- Distinguish process exit, container stop, and container removal
- Read a container's status and exit code
- Compare the first and second run of the same image
- Explain the effect of `--rm`
- Diagnose basic first-container failures using evidence

---

## Prerequisites

You should have completed:

- Lesson 01 — What Docker Is
- Lesson 02 — Installing Docker Engine on Ubuntu

You should already know:

```text
Image     = packaged application template
Container = instance created from an image
Registry  = service that stores and distributes images
Host      = machine running Docker
```

Your selected Docker access model must work.

If you use Docker-group or rootless access:

```bash
docker version
```

If you deliberately use explicit rootful access:

```bash
sudo docker version
```

The output should contain both Client and Server sections.

### Command notation in this lesson

Examples use:

```bash
docker ...
```

If you selected explicit `sudo`, run the equivalent:

```bash
sudo docker ...
```

Do not solve permission errors by making `/var/run/docker.sock` world-writable.

---

## Why This Topic Exists

A beginner often sees this:

```text
Hello from Docker!
```

and thinks only one thing happened: Docker printed a message.

In reality, several Docker actions occurred:

```text
docker run hello-world
        ↓
Check whether image exists locally
        ↓
Pull image if it is missing
        ↓
Create a new container from the image
        ↓
Start the container's main process
        ↓
Process prints a message
        ↓
Process exits with a status code
        ↓
Container becomes stopped
```

Understanding this workflow prepares you for almost every later topic:

- Container lifecycle
- Logs
- Image management
- Command overrides
- Troubleshooting
- Networking
- Volumes
- Kubernetes Pods

---

# Part 1 — The Meaning of `docker run`

## 1. Beginner Mental Model

`docker run` asks Docker to create a new container from an image and start it.

```text
Image
  ↓ create
Container
  ↓ start
Main process runs
```

This is the most important statement in the lesson:

> Every normal `docker run` creates a new container.

It does not normally find an old stopped container and restart it.

## 2. General Syntax

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

For now, focus on:

```text
docker       → Docker CLI
run          → create and start a new container
hello-world  → image name
```

Our first command has no custom command and no additional arguments:

```bash
docker run hello-world
```

Docker therefore uses the default startup instruction stored in the image.

We will study command overriding, `CMD`, and `ENTRYPOINT` later.

## 3. `docker run` Is a Convenience Command

Conceptually, `docker run` combines two lifecycle actions:

```text
create container
       +
start container
       =
docker run
```

Later you will perform these separately using:

```bash
docker create
docker start
```

For this lesson, remember:

> `docker run` creates a new container and starts it immediately.

---

# Part 2 — Before the First Run

## 4. Inspect Running Containers

```bash
docker ps
```

Possible output:

```text
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```

Only the headings may appear. That means Docker currently has no running containers visible to this daemon/context.

It does not mean Docker is broken.

## 5. Inspect All Containers

```bash
docker ps -a
```

`-a` means `--all`.

Without `-a`, `docker ps` shows running containers. With `-a`, it also shows stopped containers.

```text
docker ps      → running only
docker ps -a   → running + stopped + other existing states
```

## 6. Inspect Local Images

```bash
docker image ls
```

The older equivalent form is:

```bash
docker images
```

This course prefers the structured form `docker image ls`, but you should recognize both.

Possible output before the first pull:

```text
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
```

If `hello-world` does not appear, the image is not currently available in this Docker daemon's local image store.

## 7. Record the Baseline

Run:

```bash
docker ps
docker ps -a
docker image ls
```

Record the output. You will compare it after the first run.

Evidence-first learning means observing before changing state.

---

# Part 3 — Run `hello-world`

## 8. The First Run

Run without `--rm` so the stopped container remains available for inspection:

```bash
docker run hello-world
```

A typical first run contains messages shaped like:

```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
```

Exact digest values, sizes, timestamps, and wording may change.

## 9. Do Not Read “Unable to Find” as Final Failure

This line:

```text
Unable to find image 'hello-world:latest' locally
```

does not mean the whole command failed.

It means:

```text
Local image check → image not found
```

Docker then continues to the next step:

```text
Pull image from a registry
```

Always interpret an output line in the context of what happens next.

## 10. The Implied Tag

You typed:

```bash
docker run hello-world
```

Docker interpreted it as:

```text
hello-world:latest
```

Here:

```text
hello-world → repository/image name
latest      → tag
```

If no tag is supplied, Docker normally uses the tag `latest`.

Important:

> `latest` is a tag name. It does not magically guarantee that an image is the newest or safest version.

Tags will be studied fully in the images lesson.

## 11. The Implied Registry Location

The short name `hello-world` normally resolves to Docker Hub's official/library namespace.

Beginner view:

```text
hello-world
     ↓ resolved to an image location
Docker Hub official image namespace
```

You do not need to memorize the fully qualified registry path yet.

## 12. Pulling the Image

When the image is absent locally, the daemon downloads the necessary image content and metadata.

This is called a **pull**.

```text
Registry
   │ image data
   ▼
Docker host local image store
```

The image now exists locally and can be reused for later containers.

## 13. Creating the Container

After the image is available, Docker creates a new container from it.

Docker assigns the container:

- A unique container ID
- A generated name, unless you provide one
- Container configuration
- Runtime state
- A writable container layer

You do not need to study image layers or the writable layer deeply yet.

## 14. Starting the Main Process

The `hello-world` image contains a small program and a default instruction to run it.

When the container starts:

```text
Container starts
       ↓
Main program starts
       ↓
Program prints explanatory text
```

The message comes from the program running in the container, not directly from the `docker` CLI.

## 15. Process Exit

After printing its message, the program has no more work.

It exits.

The container's lifetime is tied to its main process:

```text
Main process running → container running
Main process exits   → container stops
```

Therefore, `hello-world` stopping is expected and successful behavior.

---

# Part 4 — Inspect the Result

## 16. Check Running Containers

```bash
docker ps
```

You probably will not see the `hello-world` container because its process already finished.

This does not mean the container never existed.

## 17. Check All Containers

```bash
docker ps -a
```

Typical shape:

```text
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS   NAMES
abc123...      hello-world   "/hello"   20 seconds ago   Exited (0) 19 seconds ago           clever_name
```

## 18. Read the Columns

### `CONTAINER ID`

A shortened form of the unique container identifier.

You can commonly use the short ID in Docker commands as long as it is unambiguous.

### `IMAGE`

The image used to create the container.

Here:

```text
hello-world
```

### `COMMAND`

The main command configured for the container.

Here it may appear as:

```text
"/hello"
```

### `CREATED`

How long ago Docker created the container.

Created time is not the same as total running time.

### `STATUS`

Current state and useful status information.

```text
Exited (0) 19 seconds ago
```

This means:

```text
Exited       → main process finished; container is stopped
(0)          → process exit code was zero
19 sec ago   → time since it exited
```

### `PORTS`

Published or exposed port information. It is empty for this container.

Networking comes later.

### `NAMES`

Docker generated a readable name because you did not provide one.

## 19. Exit Code Zero

On Unix-like systems, exit code `0` conventionally means success.

```text
Exited (0) → stopped successfully
```

A non-zero exit code often indicates an error or special result, but its exact meaning depends on the program.

The state `Exited` alone does not tell you whether the task succeeded. Read the exit code and logs.

## 20. Inspect the Local Image

```bash
docker image ls hello-world
```

Typical shape:

```text
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    ...            ...            ...
```

### Important relationship

After the first run, two different objects normally remain:

```text
hello-world image
        ↓ created
stopped hello-world container
```

The container and image have different IDs and different purposes.

## 21. Inspect the Exact Exit Code

First copy the container name or ID from:

```bash
docker ps -a
```

Then run:

```bash
docker inspect CONTAINER_NAME_OR_ID
```

The full output is large JSON. To retrieve only the exit code:

```bash
docker inspect --format '{{.State.ExitCode}}' CONTAINER_NAME_OR_ID
```

Expected output:

```text
0
```

### What `--format` does

It selects and formats a specific field from Docker's inspection data.

You are not expected to memorize the template yet. The purpose is to prove the exit code directly.

---

# Part 5 — First Run Versus Second Run

## 22. Run It Again

```bash
docker run hello-world
```

Observe carefully.

The second run normally does not contain:

```text
Unable to find image ... locally
Pulling from ...
```

Why?

The image was downloaded during the first run and remains in the local image store.

## 23. The Second Run Still Creates a New Container

Even though Docker reuses the local image, it creates a new container:

```text
First docker run  → Container A
Second docker run → Container B
```

Confirm:

```bash
docker ps -a
```

You should normally see two `hello-world` containers with different IDs and names.

## 24. What Was Reused?

```text
Reused:     local image
Not reused: old stopped container
Created:    new container
```

This distinction is fundamental.

If you want to start an existing stopped container, you use a lifecycle command such as `docker start`, which comes in Lesson 04.

## 25. Compare the Two Runs

| Stage | First run | Second run |
| --- | --- | --- |
| Local image check | Image missing | Image found |
| Registry pull | Required | Normally not required |
| New container created | Yes | Yes |
| Main program runs | Yes | Yes |
| Container exits | Yes | Yes |
| New stopped container remains | Yes | Yes |

---

# Part 6 — Stopped Versus Removed

## 26. Stopping Is Not Removing

After the main process exits, the container becomes stopped.

Its Docker metadata and writable state still exist until the container is removed.

```text
Running container
       ↓ process exits
Stopped container
       ↓ docker rm
Removed container
```

## 27. Remove One Container

List containers:

```bash
docker ps -a
```

Remove one stopped container by ID or name:

```bash
docker rm CONTAINER_NAME_OR_ID
```

Verify:

```bash
docker ps -a
```

The removed container no longer appears.

## 28. Removing the Container Does Not Remove the Image

Verify:

```bash
docker image ls hello-world
```

The image still exists.

```text
docker rm  → removes container
docker rmi → removes image
```

Do not confuse them.

Image removal will be studied fully in Lesson 06.

## 29. Automatic Removal with `--rm`

Run:

```bash
docker run --rm hello-world
```

`--rm` tells Docker:

> Automatically remove this container after its process exits.

Compare the list before and after:

```bash
docker ps -a
```

The temporary container created by the `--rm` run should not remain in the list after it exits.

The image remains.

## 30. When Is `--rm` Useful?

It is useful for disposable tasks where you do not need the stopped container afterward.

Examples later may include:

- Test commands
- One-time transformations
- Short diagnostic tools
- Temporary interactive shells

Do not use automatic removal when you need the stopped container's state for investigation.

---

## Big Picture

```text
                            ┌──────────────────┐
                            │ Container registry│
                            └────────┬─────────┘
                                     │ pull if missing
                                     ▼
┌──────────────┐ request     ┌─────────────────┐
│ Docker CLI   │────────────>│ Docker daemon   │
└──────────────┘             └────────┬────────┘
                                     │ uses image
                                     ▼
                              ┌───────────────┐
                              │ New container │
                              └───────┬───────┘
                                      │ starts main process
                                      ▼
                              output → exit code
                                      │
                                      ▼
                              stopped container
```

The key sequence is:

```text
check image → pull if missing → create → start → process exits → stop
```

---

## Commands

## 31. `docker run`

Syntax:

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Examples:

```bash
docker run hello-world
docker run --rm hello-world
```

Purpose: create a new container from an image and start it.

Common mistake: believing it restarts a previous container.

## 32. `docker ps`

```bash
docker ps
```

Purpose: list running containers.

Use it when asking:

> What containers are running now?

## 33. `docker ps -a`

```bash
docker ps -a
```

Purpose: list all existing containers, including stopped containers.

Use it when a container seems to have “disappeared” from `docker ps`.

## 34. `docker image ls`

```bash
docker image ls
docker image ls hello-world
```

Purpose: list images stored locally.

Equivalent older command:

```bash
docker images
```

## 35. `docker inspect`

```bash
docker inspect CONTAINER
docker inspect --format '{{.State.ExitCode}}' CONTAINER
```

Purpose: view detailed Docker metadata or select a specific field.

## 36. `docker rm`

```bash
docker rm CONTAINER
```

Purpose: remove a stopped container.

It does not remove the image.

---

## Guided Hands-on Lab — Observe Two Runs

### Scenario

You are verifying a new Docker host for TechCorp. Your goal is not only to see “Hello from Docker,” but to prove every visible stage.

### Phase A — Verify access

```bash
docker version
```

Confirm Client and Server sections.

### Phase B — Capture baseline

```bash
docker ps
docker ps -a
docker image ls hello-world
```

If old `hello-world` objects exist, you may keep them and account for them, or remove only disposable lab containers you recognize.

### Phase C — First run

```bash
docker run hello-world
```

Record:

- Was the image found locally?
- Did Docker pull it?
- What application output appeared?
- Did the command return to the shell prompt?

### Phase D — Inspect

```bash
docker ps
docker ps -a
docker image ls hello-world
```

From `docker ps -a`, record:

- Container ID
- Image
- Command
- Status
- Exit code
- Generated name

Verify the exit code:

```bash
docker inspect --format '{{.State.ExitCode}}' CONTAINER
```

Replace `CONTAINER` with the name or ID.

### Phase E — Second run

```bash
docker run hello-world
```

Compare the output with the first run.

Then:

```bash
docker ps -a
```

Prove that a second container was created.

### Phase F — Automatic cleanup

Count or note current `hello-world` containers:

```bash
docker ps -a --filter ancestor=hello-world
```

Run:

```bash
docker run --rm hello-world
```

Check again:

```bash
docker ps -a --filter ancestor=hello-world
```

The `--rm` run should not leave another stopped container.

### Acceptance Criteria

You can prove:

- The image exists locally
- The first normal run left a stopped container
- The second normal run created a different container
- Both normal containers exited with code 0
- `docker ps` does not show them because they are stopped
- `docker ps -a` does show them
- The `--rm` run did not leave a stopped container
- Removing a container does not remove its image

---

## Discovery Questions

Answer before opening the explanation.

### Question 1

Why did the first run download data but the second normally did not?

<details>
<summary>Explanation</summary>

The image was absent during the first run, so Docker pulled it. It remained in the local image store and was reused during the second run.

</details>

### Question 2

Did the second run reuse the first container?

<details>
<summary>Explanation</summary>

No. Each `docker run` created a new container. Docker reused the image, not the old container.

</details>

### Question 3

Why does `docker ps` show nothing after `hello-world` prints its message?

<details>
<summary>Explanation</summary>

`docker ps` shows running containers by default. The `hello-world` main process finished, so the container stopped.

</details>

### Question 4

Does `Exited (0)` mean failure?

<details>
<summary>Explanation</summary>

No. Zero conventionally means the process completed successfully. The container stopped because its work was finished.

</details>

### Question 5

After `docker rm CONTAINER`, why can you still run `docker run hello-world` without downloading the image again?

<details>
<summary>Explanation</summary>

`docker rm` removes a container, not the image. The local image is still available to create a new container.

</details>

---

## Production Scenario

TechCorp deploys a database migration as a short containerized task.

The container:

1. Connects to the database
2. Applies the migration
3. Exits with code 0

A new engineer sees that the container is no longer running and opens an incident.

The correct investigation is:

```bash
docker ps -a
docker inspect --format '{{.State.ExitCode}}' migration-container
docker logs migration-container
```

If the intended job completed and exited with code 0, the stopped state is expected.

This teaches a production principle:

> Not every container is a permanent server. Some containers are finite jobs.

A web server normally needs a long-running main process. A migration, backup, test, or batch task may exit when its work is done.

---

## Basic Troubleshooting Workflow

## 37. Image Pull Fails

Possible symptoms include DNS failure, timeout, authentication failure, or registry rate limits.

Check:

```bash
docker pull hello-world
```

Then investigate the exact error rather than assuming the image name is wrong.

## 38. Permission Denied on Docker Socket

Check:

```bash
ls -l /var/run/docker.sock
id
docker version
```

Return to Lesson 02. This is an authorization problem before container creation.

## 39. Container Exits Non-zero

Check:

```bash
docker ps -a
docker logs CONTAINER
docker inspect --format '{{.State.ExitCode}}' CONTAINER
```

Collect state, logs, and exit code before removing the container.

## 40. Container Is Missing from `docker ps`

Use:

```bash
docker ps -a
```

It may simply be stopped.

If it was run with `--rm`, it may already have been removed.

---

## Common Mistakes

### Mistake 1 — Thinking `docker run` means only “start”

It creates a new container and starts it.

### Mistake 2 — Thinking every run downloads the image

Docker first checks the local image store.

### Mistake 3 — Thinking the second run reused the first container

It reused the image and created another container.

### Mistake 4 — Treating every stopped container as failed

Read its intended behavior, status, exit code, and logs.

### Mistake 5 — Looking only at `docker ps`

Use `docker ps -a` when investigating stopped containers.

### Mistake 6 — Confusing `latest` with guaranteed newest

`latest` is simply a tag.

### Mistake 7 — Confusing container removal with image removal

`docker rm` removes containers. `docker rmi` removes images.

### Mistake 8 — Using `--rm` during failure investigation

Automatic removal can eliminate the stopped container metadata you wanted to inspect.

### Mistake 9 — Assuming output came from the CLI

The `hello-world` message is printed by the program executed in the container.

### Mistake 10 — Removing evidence before diagnosing

Inspect status, exit code, and logs before cleanup.

---

## Best Practices

- Observe existing images and containers before running experiments
- Distinguish image reuse from container reuse
- Name containers when human-readable identity will help operations
- Use `--rm` for truly disposable successful tasks
- Avoid `--rm` when you may need post-exit inspection
- Check both state and exit code
- Read application logs before concluding why a container stopped
- Use explicit image tags in controlled deployments
- Treat mutable tags such as `latest` cautiously in production
- Keep evidence until the root cause is understood

---

## Interview Questions

### Junior

1. What does `docker run` do?
2. What happens if the requested image is not local?
3. Does every `docker run` create a new container?
4. What is the difference between `docker ps` and `docker ps -a`?
5. Why does `hello-world` stop?
6. What does `Exited (0)` mean?
7. Does a stopped container still exist?
8. What does `--rm` do?
9. Does `docker rm` remove the image?
10. Why is the second run normally faster or shorter in output?

### Intermediate

1. How would you prove whether a short-running container succeeded?
2. Why can a stopped container be healthy behavior for a job but unhealthy for a web server?
3. What evidence may be lost when using `--rm`?
4. How would you distinguish image reuse from container reuse?
5. Why should production deployments avoid relying blindly on the `latest` tag?

### Senior Preview

Image pull policies, content-addressable storage, runtime internals, signal handling, and production restart behavior are deferred until their prerequisites are taught.

---

## Lesson Reflection

Answer in your own words:

1. Describe every beginner-visible stage of `docker run hello-world`.
2. What exactly was downloaded during the first run?
3. What was reused during the second run?
4. What was newly created during the second run?
5. Why does the container stop after printing its message?
6. How can you prove it succeeded?
7. What remains after a normal run?
8. What remains after a run with `--rm`?
9. Explain stopped versus removed.
10. When would you avoid `--rm`?

---

## Summary

- `docker run` creates a new container from an image and starts it
- Docker checks for the image locally before pulling it
- A missing local image is normally pulled from a registry
- The image remains locally for reuse
- Each `docker run` normally creates a different container
- The container runs its configured main process
- When the main process exits, the container stops
- `hello-world` is a finite program, so stopping is expected
- Exit code 0 conventionally indicates success
- `docker ps` shows running containers by default
- `docker ps -a` includes stopped containers
- A stopped container still exists until removed
- `docker rm` removes a container, not its image
- `--rm` automatically removes the container after exit

---

## Cheat Sheet

### Workflow

```text
docker run IMAGE
      ↓
Check local image
      ↓
Pull if missing
      ↓
Create new container
      ↓
Start main process
      ↓
Process exits
      ↓
Container stops
```

### Essential commands

```bash
# Create and start a new container
docker run hello-world

# Run a disposable container
docker run --rm hello-world

# List running containers
docker ps

# List all existing containers
docker ps -a

# List local images
docker image ls
docker image ls hello-world

# Inspect a container
docker inspect CONTAINER

# Print only its exit code
docker inspect --format '{{.State.ExitCode}}' CONTAINER

# Remove a stopped container
docker rm CONTAINER
```

### Essential distinctions

```text
Image reused      ≠ container reused
Stopped           ≠ removed
Exited            ≠ automatically failed
docker ps         ≠ docker ps -a
docker rm         ≠ docker rmi
latest tag        ≠ guaranteed newest
```

---

## Challenge — Prove the Lifecycle

### Scenario

Your senior engineer says:

> “Run `hello-world` twice and prove exactly what Docker reused, what Docker created, and why nothing remains running.”

### Rules

- Do not delete objects before collecting evidence
- Do not call `Exited (0)` a failure
- Use command output to support every conclusion
- Use only commands taught in this lesson

### Required evidence

1. Local image list before the first run
2. First-run output
3. `docker ps` after the first run
4. `docker ps -a` after the first run
5. Exit-code inspection
6. Second-run output
7. Proof that a second container was created
8. Proof that the image was reused
9. A third run using `--rm`
10. Proof that the third container was automatically removed

### Final report template

```text
Image before first run:

First run pull behavior:

First container ID/name:

First container exit code:

Second run pull behavior:

Second container ID/name:

What was reused:

What was newly created:

Why no container is running:

Effect of --rm:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-03/
├── 01-baseline-running-containers.txt
├── 02-baseline-all-containers.txt
├── 03-baseline-images.txt
├── 04-first-run-output.txt
├── 05-after-first-run.txt
├── 06-first-exit-code.txt
├── 07-second-run-output.txt
├── 08-after-second-run.txt
├── 09-rm-run-output.txt
├── 10-final-container-list.txt
└── lifecycle-report.md
```

Useful capture pattern:

```bash
docker ps -a > 05-after-first-run.txt
docker image ls hello-world > 03-baseline-images.txt
```

To capture both standard output and error when needed:

```bash
docker run hello-world > 04-first-run-output.txt 2>&1
```

Do not store credentials, tokens, or unrelated sensitive environment information.

---

## Next Lesson

**Lesson 04 — Container Lifecycle**

You now understand automatic creation, start, process exit, and stopped state for a simple container.

Next, you will control lifecycle states deliberately:

```text
Created
   ↓ start
Running
   ↓ pause/unpause
Paused / Running
   ↓ stop or kill
Stopped
   ↓ start again or remove
```

You will learn:

```bash
docker create
docker start
docker stop
docker restart
docker kill
docker pause
docker unpause
docker rm
docker rename
```

Most importantly, you will compare:

- `docker run` versus `docker create` plus `docker start`
- `docker stop` versus `docker kill`
- Stopping versus removing
- `docker rm` versus `docker rmi`

---

## Completion Check

Before moving forward, explain without reading:

```text
What docker run does
Why the first run pulls an image
Why the second run normally does not
Why the second run still creates a new container
Why hello-world stops
What Exited (0) means
Why docker ps and docker ps -a differ
What --rm changes
Why removing a container does not remove its image
```

If you can prove those points using command output, you are ready for the complete container lifecycle.

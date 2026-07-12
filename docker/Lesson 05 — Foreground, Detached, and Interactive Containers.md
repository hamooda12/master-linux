# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 05 — Foreground, Detached, and Interactive Containers

> **Level:** Beginner  
> **Type:** Guided practical lesson  
> **Lab images:** `alpine:3.22` and `nginx:alpine`  
> **Estimated study time:** 120–150 minutes

---

## Overview

In the previous lesson, you controlled container states. Now you will study the connection between your terminal and processes inside a container.

You will answer:

- Why does one container occupy the terminal while another returns the prompt immediately?
- What does detached mode actually detach?
- Why are `-i` and `-t` separate options?
- What happens when you type `exit` in a container shell?
- What is the difference between exiting and detaching?
- Why is `docker attach` different from `docker exec`?
- Where do container logs come from?
- Why can a container be running even when no terminal is attached?

The main mental model is:

```text
Container lifecycle ≠ terminal attachment
```

A container can continue running after your terminal disconnects, or it can stop because the main process you were using exits.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain foreground and detached modes
- Explain standard input, standard output, and standard error
- Explain what `-i`, `-t`, `-d`, `--name`, and `--rm` do
- Start an interactive Alpine shell
- Explain why `exit` stops a shell-based container
- Detach from a suitable interactive container without stopping it
- Run Nginx in detached mode
- Read container output using `docker logs`
- Follow new log output
- Explain `docker attach` accurately
- Explain `docker exec` accurately
- Prove that `exec` creates an additional process, not another container
- Distinguish the main container process from an exec process
- Choose safely between logs, attach, and exec

---

## Prerequisites

You should understand:

- Image and container
- `docker run`
- Container main process
- Created, running, and exited states
- `docker ps` and `docker ps -a`
- `docker start`, `docker stop`, and `docker rm`
- Exit codes

Your Docker access model must work.

Examples use:

```bash
docker ...
```

If you selected explicit `sudo`, add `sudo` consistently.

---

## Why This Topic Exists

Consider two commands:

```bash
docker run nginx:alpine
```

```bash
docker run -d nginx:alpine
```

Both create and start an Nginx container. But the terminal experience is different.

The first keeps the Docker client attached to container output in the foreground. The second starts the container in detached mode and returns the shell prompt.

Now consider:

```bash
docker run -it alpine:3.22 sh
```

This provides an interactive shell.

These behaviors are not magic. They come from how Docker connects your terminal to the container's standard streams and whether it allocates a pseudo-terminal.

---

# Part 1 — Standard Streams

## 1. Every Process Has Standard Streams

Linux processes commonly use three standard streams:

| Stream | Number | Direction | Purpose |
| --- | ---: | --- | --- |
| Standard input (`STDIN`) | 0 | Into process | Input such as typed text or piped data |
| Standard output (`STDOUT`) | 1 | Out of process | Normal program output |
| Standard error (`STDERR`) | 2 | Out of process | Error and diagnostic output |

Beginner diagram:

```text
Your keyboard ──STDIN──> Process
Your screen  <─STDOUT── Process
Your screen  <─STDERR── Process
```

## 2. Docker and the Streams

Docker can connect the CLI to the main container process's streams:

```text
Local terminal
    │ STDIN
    ▼
Docker CLI ↔ Docker daemon ↔ container main process
    ▲                              │
    └──────── STDOUT/STDERR ───────┘
```

This connection is called **attachment**.

## 3. Attachment Does Not Create the Process

Attachment describes the I/O connection.

The container's main process can exist whether your current terminal is attached or not.

```text
Attached terminal closes/detaches
              ≠
Main process must always exit
```

The exact outcome depends on how you disconnect, which signals are sent, and how the process handles them.

---

# Part 2 — Foreground Mode

## 4. Default Foreground Behavior

Run a disposable Nginx container:

```bash
docker run --name foreground-web nginx:alpine
```

Your shell prompt does not immediately return because the Docker client remains attached to the container's output streams.

Nginx is still running. It may simply have no new output to show.

### Important

An apparently quiet terminal does not mean Docker is frozen.

The attached main process may be waiting for work.

## 5. Verify from a Second Terminal

Open another terminal and run:

```bash
docker ps --filter name=foreground-web
```

This proves the container is running while the first terminal remains attached.

## 6. What Foreground Means Here

Foreground mode means the Docker CLI remains connected to the container's output and, depending on options, input.

```text
Terminal occupied by docker client
       ↓
Client attached to container process streams
       ↓
Container runs while main process runs
```

It does not mean the container process is a normal host-shell foreground job in every implementation detail. It is a useful terminal relationship model.

## 7. `Ctrl+C` While Attached

When signal proxying is enabled—the default—`Ctrl+C` normally causes the Docker client to proxy `SIGINT` to the container's main process.

Whether the container stops depends on how that process handles `SIGINT` and on Linux's special treatment of PID 1.

Therefore, do not memorize:

```text
Ctrl+C always stops every container
```

Use evidence:

```bash
docker ps -a --filter name=foreground-web
```

If it remains running, stop it deliberately:

```bash
docker stop foreground-web
```

Then remove it:

```bash
docker rm foreground-web
```

---

# Part 3 — Detached Mode

## 8. Run Nginx Detached

```bash
docker run -d --name detached-web nginx:alpine
```

Expected output:

```text
<container-id>
```

Your shell prompt returns immediately.

## 9. Meaning of `-d`

`-d` is short for:

```text
--detach
```

Detached mode starts the container and leaves it running in the background from the CLI user's point of view.

```text
docker run -d
      ↓
Container starts
      ↓
CLI prints container ID
      ↓
Shell prompt returns
      ↓
Main process continues
```

## 10. Prove It Is Running

```bash
docker ps --filter name=detached-web
```

The terminal returned, but the container remains running because Nginx's main process remains running.

## 11. Detached Does Not Mean No Output Exists

The application can still write to `STDOUT` and `STDERR`.

Docker's logging configuration normally captures those streams, and you can retrieve them with:

```bash
docker logs detached-web
```

Detached mode changes your CLI attachment; it does not automatically silence or disable the application.

## 12. Detached Is Not a Container State

Docker state may be:

```text
running
```

Attachment mode may be:

```text
detached
```

These describe different dimensions:

| Question | Concept |
| --- | --- |
| Is the main process running? | Container state |
| Is this terminal connected to its streams? | Attachment mode |

---

# Part 4 — Interactive Input and TTY

## 13. Run an Alpine Shell

```bash
docker run --name alpine-shell -it alpine:3.22 sh
```

Typical prompt:

```text
/ #
```

You are now interacting with `sh` running as the container's main process.

## 14. Meaning of `-i`

`-i` is short for:

```text
--interactive
```

It keeps the container's standard input open even if the client is not currently attached.

Beginner meaning:

> Allow input to be sent to the container process.

## 15. Meaning of `-t`

`-t` is short for:

```text
--tty
```

It allocates a pseudo-terminal for the container process.

A pseudo-terminal provides terminal-like behavior expected by interactive shells and terminal applications, such as:

- A recognizable prompt
- Line editing
- Terminal control behavior
- Better formatting for interactive use

## 16. Why Use `-it` Together?

```text
-i → keep input open
-t → provide terminal-like device
```

Together:

```bash
-it
```

creates the normal interactive terminal experience.

They are separate features, even though they are commonly combined.

## 17. Prove You Are Inside the Container

At the Alpine prompt, run:

```sh
hostname
cat /etc/os-release
ps
pwd
```

Interpretation:

- `hostname` normally shows the container hostname, often based on its short ID
- `/etc/os-release` identifies the container image's userspace as Alpine
- `ps` shows processes visible inside the container
- `pwd` shows the shell's working directory inside the container

Do not conclude that this is a complete virtual machine. It is the container's process and filesystem environment.

## 18. Create a File

Inside the container:

```sh
echo "created in alpine-shell" > /tmp/lesson05.txt
cat /tmp/lesson05.txt
```

This file is in this container's writable layer. Persistence and data-loss behavior receive dedicated lessons later.

## 19. Type `exit`

```sh
exit
```

The shell was the main container process. When it exits:

```text
sh exits
   ↓
Main process no longer running
   ↓
Container stops
```

Verify:

```bash
docker ps -a --filter name=alpine-shell
```

## 20. Restart and Attach to Its Streams

Start it with attachment:

```bash
docker start -ai alpine-shell
```

Here:

```text
-a → attach to output/error
-i → attach standard input
```

The existing container starts its original configured `sh` command again.

Verify the file:

```sh
cat /tmp/lesson05.txt
```

It remains because this is the same container. Type `exit` again when finished.

---

# Part 5 — Exit Versus Detach

## 21. The Critical Difference

```text
exit
  → terminates the shell process
  → if shell is main process, container stops

detach
  → disconnects this Docker client
  → main process can continue running
```

## 22. Create a Suitable Detach Lab

Remove the stopped lab container if it exists:

```bash
docker rm alpine-shell
```

Create a new interactive container in detached mode:

```bash
docker run -dit --name detachable-shell alpine:3.22 sh
```

Options:

```text
-d → start detached
-i → keep standard input open
-t → allocate pseudo-TTY
```

## 23. Attach

```bash
docker attach detachable-shell
```

You are attached to the existing main `sh` process.

## 24. Detach Key Sequence

Press:

```text
Ctrl+P, then Ctrl+Q
```

These are sequential key chords:

1. Hold `Ctrl` and press `P`
2. Release
3. Hold `Ctrl` and press `Q`

Do not type the literal characters `Ctrl+P Ctrl+Q`.

For a container created with the appropriate interactive TTY configuration, Docker's default detach sequence disconnects the client without stopping the container.

Verify:

```bash
docker ps --filter name=detachable-shell
```

## 25. Why Detach May Not Work in Every Case

The default detach sequence is handled under appropriate attachment/TTY conditions. A container started without interactive TTY options may behave differently, and signals may be forwarded to its main process.

Therefore:

- Do not test detach keys blindly on an important container
- Know how the container was created
- Prefer `docker logs` or `docker exec` when they match your goal

## 26. Clean Up

```bash
docker stop detachable-shell
docker rm detachable-shell
```

---

# Part 6 — `docker attach`

## 27. What Attach Does

Syntax:

```bash
docker attach [OPTIONS] CONTAINER
```

It connects your terminal's streams to the container's existing main process.

```text
Before attach:
Container main process already running

After attach:
Same main process + your terminal connected to its streams
```

## 28. What Attach Does Not Do

`docker attach` does not:

- Start a new shell automatically
- Create a new container
- Run a new process
- Guarantee a command prompt

If the main process is Nginx, attaching connects you to Nginx's streams. It does not magically turn Nginx into a shell.

## 29. Why Attach Can Appear Frozen

The main process may simply not be producing output or reading interactive input.

```text
No new output ≠ attach failed
```

Check the process and state from another terminal.

## 30. Attach Risk

Input and signals may affect the main application process.

For administration and troubleshooting, attaching directly to the production main process can be risky. Prefer:

- `docker logs` to observe application output
- `docker exec` to run a separate diagnostic command

Use attach when you specifically need the existing main process's streams.

---

# Part 7 — `docker exec`

## 31. What Exec Does

Syntax:

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

`docker exec` starts an additional command inside an already-running container.

```text
Container
    ├── Main process: Nginx
    └── Exec process: sh
```

It does not create another container.

## 32. Start the Lab Web Container

If `detached-web` is not running, create or start it appropriately:

```bash
docker ps -a --filter name=detached-web
```

If it does not exist:

```bash
docker run -d --name detached-web nginx:alpine
```

If it exists but is stopped:

```bash
docker start detached-web
```

## 33. Execute a Non-interactive Command

```bash
docker exec detached-web hostname
```

The `hostname` command runs, prints output, and exits. Nginx continues running.

Verify:

```bash
docker ps --filter name=detached-web
```

## 34. Open an Interactive Shell

```bash
docker exec -it detached-web sh
```

This starts a new shell process inside the existing container.

The main process is still Nginx.

Inside:

```sh
ps
```

You should see Nginx-related processes and your new `sh`/`ps` processes.

## 35. Exit the Exec Shell

```sh
exit
```

Only the additional shell exits.

The main Nginx process remains, so the container remains running.

Verify:

```bash
docker ps --filter name=detached-web
```

Compare this with the earlier Alpine container where `sh` was the main process.

```text
Alpine run ... sh:
sh = main process → exit stops container

docker exec ... sh in Nginx:
sh = additional process → exit leaves Nginx/container running
```

## 36. Exec Requires a Running Container

If the main container process is stopped, an exec process cannot be started normally.

```bash
docker stop detached-web
docker exec detached-web hostname
```

Docker returns an error because the container is not running.

Start it again:

```bash
docker start detached-web
```

## 37. Correctly Execute Shell Expressions

This does not work as intended:

```bash
docker exec detached-web "echo one && echo two"
```

Docker tries to find an executable literally matching the quoted string.

Use a shell explicitly:

```bash
docker exec detached-web sh -c 'echo one && echo two'
```

Here:

```text
sh       → executable
-c       → ask shell to interpret following string
'...'    → shell command expression
```

## 38. Exec Options Preview

```bash
docker exec -it detached-web sh
docker exec -e LESSON=05 detached-web env
docker exec -w /tmp detached-web pwd
docker exec -u nginx detached-web id
```

| Option | Meaning |
| --- | --- |
| `-i` | Keep input open |
| `-t` | Allocate pseudo-TTY |
| `-e` | Set environment variable for exec process |
| `-w` | Choose working directory |
| `-u` | Select user or UID for exec process |

Do not use `--privileged` casually. It expands privileges and requires dedicated security understanding.

---

# Part 8 — Attach Versus Exec

## 39. Direct Comparison

| `docker attach` | `docker exec` |
| --- | --- |
| Connects to existing main process streams | Starts an additional process |
| Does not automatically start a shell | Can explicitly start `sh` or another command |
| Signals/input may affect main application | Exec shell is separate from main process |
| Exiting main shell can stop container | Exiting exec shell normally leaves main process running |
| Use when main process streams are the goal | Use for diagnostics and administration |

## 40. Which Should You Choose?

### Want application output?

Use:

```bash
docker logs CONTAINER
```

### Want a diagnostic command?

Use:

```bash
docker exec CONTAINER COMMAND
```

### Want an interactive diagnostic shell?

Use:

```bash
docker exec -it CONTAINER sh
```

### Want the existing main process's live streams?

Use:

```bash
docker attach CONTAINER
```

with a clear understanding of its input and signal behavior.

---

# Part 9 — Container Logs

## 41. Where Logs Come From

Containerized applications should normally write operational logs to `STDOUT` and `STDERR`.

Docker's configured logging driver handles those streams.

Beginner flow:

```text
Application
  ├── STDOUT ─┐
  └── STDERR ─┼→ Docker logging driver → docker logs
```

`docker logs` does not automatically read every log file anywhere inside the container.

If an application writes only to `/var/log/app.log`, that content may not appear in `docker logs` unless the image/application redirects or forwards it appropriately.

## 42. Read Logs

```bash
docker logs detached-web
```

This retrieves available logs for the container.

## 43. Show Recent Lines

```bash
docker logs --tail 20 detached-web
```

Shows the last 20 lines.

## 44. Follow Logs

```bash
docker logs --follow detached-web
```

Short form:

```bash
docker logs -f detached-web
```

It prints available output and waits for new log entries.

Use `Ctrl+C` to stop the local log-follow command. This exits the log client; it does not normally stop the container.

Verify:

```bash
docker ps --filter name=detached-web
```

## 45. Add Timestamps

```bash
docker logs --timestamps --tail 20 detached-web
```

Short form:

```bash
docker logs -t --tail 20 detached-web
```

Timestamps help correlate container output with incidents and other services.

## 46. Logs Versus Attach

```text
docker logs  → retrieve logs handled by logging driver
docker attach→ connect to live main-process streams
```

For routine observation, logs are usually the safer and more useful choice.

---

## Big Picture

```text
                         ┌── docker logs: read captured output
Container main process ──┼── docker attach: connect to its streams
                         └── container state follows this process

Running container
    ├── Main process
    ├── docker exec command A
    └── docker exec shell B
```

Terminal modes:

```text
Foreground → Docker client remains attached
Detached   → Docker client returns; container may continue
Interactive→ STDIN kept open
TTY        → terminal-like device allocated
```

---

## Commands — Detailed Reference

## 47. Foreground Run

```bash
docker run --name NAME IMAGE
```

Creates and starts a new container with the client attached to output by default.

## 48. Detached Run

```bash
docker run -d --name NAME IMAGE
```

Creates and starts the container, prints its ID, and returns the prompt.

## 49. Interactive TTY Run

```bash
docker run -it --name NAME IMAGE sh
```

Keeps input open and allocates a pseudo-terminal.

## 50. Disposable Interactive Container

```bash
docker run --rm -it alpine:3.22 sh
```

Automatically removes the container after the shell exits. Avoid `--rm` when you need post-exit inspection.

## 51. Attach

```bash
docker attach CONTAINER
```

Connects to the existing main process's standard streams.

Default detach sequence for an appropriate interactive TTY container:

```text
Ctrl+P, Ctrl+Q
```

## 52. Exec

```bash
docker exec CONTAINER COMMAND
docker exec -it CONTAINER sh
```

Starts an additional command inside a running container.

## 53. Logs

```bash
docker logs CONTAINER
docker logs --tail 20 CONTAINER
docker logs -f CONTAINER
docker logs -t --tail 20 CONTAINER
```

Retrieves output through the configured logging system.

## 54. Start with Attachment

```bash
docker start -ai CONTAINER
```

Starts an existing container and attaches output/input.

---

## Guided Hands-on Lab

### Scenario

TechCorp wants you to demonstrate four different terminal relationships:

1. Foreground Nginx
2. Detached Nginx
3. Interactive Alpine shell
4. An exec shell inside running Nginx

### Phase A — Foreground

Terminal 1:

```bash
docker run --name foreground-web nginx:alpine
```

Terminal 2:

```bash
docker ps --filter name=foreground-web
```

Return to Terminal 1 and use `Ctrl+C`. Then verify in Terminal 2:

```bash
docker ps -a --filter name=foreground-web
```

Record the actual result. Stop if still running, then remove safely.

### Phase B — Detached

```bash
docker run -d --name detached-web nginx:alpine
docker ps --filter name=detached-web
docker logs detached-web
```

Prove that the prompt returned while Nginx remained running.

### Phase C — Interactive Alpine

```bash
docker run --name alpine-shell -it alpine:3.22 sh
```

Inside:

```sh
hostname
cat /etc/os-release
ps
echo "lesson 05" > /tmp/proof.txt
cat /tmp/proof.txt
exit
```

Outside:

```bash
docker ps -a --filter name=alpine-shell
```

Explain why it stopped.

### Phase D — Start and attach

```bash
docker start -ai alpine-shell
```

Inside:

```sh
cat /tmp/proof.txt
exit
```

Explain why the file remained and why the container stopped again.

### Phase E — Detach instead of exit

```bash
docker rm alpine-shell
docker run -dit --name detachable-shell alpine:3.22 sh
docker attach detachable-shell
```

Use `Ctrl+P`, then `Ctrl+Q`.

Verify:

```bash
docker ps --filter name=detachable-shell
```

### Phase F — Exec into Nginx

```bash
docker exec detached-web hostname
docker exec -it detached-web sh
```

Inside:

```sh
ps
echo "exec shell" > /tmp/exec-proof.txt
cat /tmp/exec-proof.txt
exit
```

Outside:

```bash
docker ps --filter name=detached-web
docker exec detached-web cat /tmp/exec-proof.txt
```

Prove the exec shell exited but Nginx remained running.

### Phase G — Logs

```bash
docker logs --timestamps --tail 20 detached-web
docker logs -f detached-web
```

Press `Ctrl+C` to leave log follow.

Verify Nginx remains:

```bash
docker ps --filter name=detached-web
```

### Phase H — Cleanup

```bash
docker stop detached-web detachable-shell
docker rm detached-web detachable-shell alpine-shell 2>/dev/null
```

Before running cleanup, confirm those names belong to your lab containers.

### Acceptance Criteria

You can prove:

- Foreground attachment occupied the terminal
- Detached mode returned the prompt while the container ran
- `-i` and `-t` have different purposes
- Exiting the main Alpine shell stopped its container
- Detaching left a suitable interactive container running
- `attach` connected to an existing main process
- `exec` started an additional process
- Exiting the exec shell did not stop Nginx
- Leaving `docker logs -f` did not stop Nginx

---

## Discovery Questions

### Question 1

Why does `docker run -d nginx:alpine` return the prompt while Nginx keeps running?

<details>
<summary>Explanation</summary>

`-d` detaches the Docker client from the container's foreground streams. Nginx remains the running main process.

</details>

### Question 2

Why did `exit` stop the Alpine shell container?

<details>
<summary>Explanation</summary>

The shell was the container's main process. Exiting it ended the main process, so the container stopped.

</details>

### Question 3

Why did exiting `docker exec -it detached-web sh` not stop Nginx?

<details>
<summary>Explanation</summary>

The exec shell was an additional process. Nginx remained the main process, so the container remained running.

</details>

### Question 4

Does `docker attach detached-web` create a shell?

<details>
<summary>Explanation</summary>

No. It attaches to the streams of the existing main process, which is Nginx.

</details>

### Question 5

Why is `docker logs` normally safer than `docker attach` for observing production output?

<details>
<summary>Explanation</summary>

Logs retrieve captured output without connecting your terminal input and signals directly to the main application process.

</details>

---

## Production Scenario — Dangerous Attachment

An engineer wants to inspect a production API and runs:

```bash
docker attach product-api
```

They then press `Ctrl+C`, expecting only to leave the view. The signal is proxied to the container's main process, and the application stops because it handles `SIGINT` by exiting.

A safer observation command would have been:

```bash
docker logs --tail 100 -f product-api
```

For a diagnostic shell:

```bash
docker exec -it product-api sh
```

provided the image contains `sh`, access is authorized, and interactive production access follows policy.

The lesson:

> Choose the command that matches the goal: logs for output, exec for a new diagnostic process, attach only for the existing main process streams.

---

## Troubleshooting Workflow

## 55. Interactive Container Exits Immediately

Check:

```bash
docker ps -a
docker inspect --format '{{.Path}} {{json .Args}}' CONTAINER
docker logs CONTAINER
```

Possible cause: the main shell received no usable open input or exited because of how it was invoked.

Use `-i` when standard input must remain open and `-t` for terminal behavior.

## 56. `docker exec` Says Container Is Not Running

Check:

```bash
docker ps -a --filter name=CONTAINER
docker inspect --format '{{.State.Status}}' CONTAINER
```

Exec requires the primary container process to be running.

## 57. Shell Command Not Found

Not every image includes Bash.

Try only after checking image documentation or filesystem evidence:

```bash
docker exec -it CONTAINER sh
```

Minimal images may contain no interactive shell at all. That is not automatically a broken image.

## 58. Attach Shows Nothing

The main process may be quiet.

Check from a second terminal:

```bash
docker ps
docker logs --tail 20 CONTAINER
```

Do not type random input into an attached production process.

## 59. Logs Are Empty

Possible explanations:

- Application has produced no output
- Application writes to an internal file instead of standard streams
- Logging driver does not support local retrieval
- Wrong container was selected
- Logs were rotated or externalized

Inspect:

```bash
docker inspect --format '{{.HostConfig.LogConfig.Type}}' CONTAINER
docker logs CONTAINER
```

## 60. Detach Keys Do Not Work

Check how the container was created. The normal sequence is intended for an appropriately interactive TTY-enabled attachment.

Use another terminal to inspect and, if needed, stop the container deliberately.

---

## Common Mistakes

### 1. Thinking detached means stopped

Detached mode normally leaves the main process running.

### 2. Treating `-it` as one unexplained option

`-i` and `-t` perform different jobs.

### 3. Thinking attach opens a new shell

Attach connects to the existing main process streams.

### 4. Thinking exec creates a new container

Exec creates another process inside an existing running container.

### 5. Exiting a main shell and expecting the container to remain

If the shell is the main process, its exit stops the container.

### 6. Pressing `Ctrl+C` in attach without understanding signals

It may affect or terminate the main process.

### 7. Assuming every image has Bash

Minimal images often provide `sh` or no shell.

### 8. Using exec as a permanent configuration method

Changes made manually inside a container are not a reproducible deployment process and disappear when the container is replaced.

### 9. Expecting `docker logs` to read every internal file

It normally exposes output captured from configured container streams/logging drivers.

### 10. Using `--rm` when post-exit evidence is needed

Automatic removal eliminates the stopped container object.

---

## Best Practices

- Run server applications detached in ordinary manual workflows
- Write application logs to standard output and standard error
- Use `docker logs` for routine observation
- Use `docker exec` for bounded diagnostic commands
- Use `docker attach` only when the existing main process streams are the actual target
- Avoid changing production containers interactively as a configuration strategy
- Record diagnostic commands and findings
- Prefer non-interactive, reproducible automation
- Use the least privilege necessary for exec access
- Do not assume a shell exists in every image
- Verify container state after leaving an interactive operation
- Keep main applications in the foreground inside their containers

---

## Interview Questions

### Junior

1. What is foreground mode?
2. What does `-d` do?
3. What does `-i` do?
4. What does `-t` do?
5. Why are `-i` and `-t` often combined?
6. Why does exiting a main shell stop its container?
7. What is the difference between exit and detach?
8. What does `docker attach` do?
9. What does `docker exec` do?
10. What does `docker logs` show?

### Intermediate

1. Why can `Ctrl+C` during attach affect the application?
2. How would you prove an exec shell is not the main process?
3. Why may an image not contain Bash or any shell?
4. Why might `docker logs` be empty while an internal log file contains data?
5. When should you choose logs, attach, or exec?
6. Why should interactive container modification not replace image builds?
7. What happens to an exec process when the container's primary process stops?

### Senior Preview

PID 1 signal semantics, log-driver architecture, backpressure, centralized logging, immutable debugging, and Kubernetes exec/log behavior are deferred until later lessons.

---

## Lesson Reflection

Answer in your own words:

1. Explain the three standard streams.
2. Explain foreground versus detached mode.
3. Explain `-i` separately from `-t`.
4. Why did exiting Alpine stop one container?
5. Why did exiting an exec shell not stop Nginx?
6. Explain attach versus exec.
7. Explain attach versus logs.
8. What does the default detach sequence accomplish?
9. Why might `Ctrl+C` behave differently across main processes?
10. Which tool would you choose for production log observation and why?

---

## Summary

- Processes commonly use standard input, output, and error
- Foreground mode keeps the Docker client attached to container streams
- Detached mode returns the prompt while the main process can continue
- Detached is an attachment mode, not a container state
- `-i` keeps standard input open
- `-t` allocates a pseudo-terminal
- `-it` provides a normal interactive terminal experience
- Exiting the main shell stops a shell-based container
- Detaching disconnects the client while leaving the process running
- `docker attach` connects to the existing main process streams
- `docker exec` starts an additional command inside a running container
- Exiting an exec shell does not stop the container while its main process continues
- `docker logs` retrieves output through Docker's logging configuration
- Logs, attach, and exec solve different problems

---

## Cheat Sheet

### Options

```text
-d, --detach      Run detached from current client
-i, --interactive Keep STDIN open
-t, --tty         Allocate pseudo-TTY
--name NAME       Assign container name
--rm              Remove container automatically after exit
```

### Commands

```bash
# Foreground
docker run --name foreground-web nginx:alpine

# Detached
docker run -d --name detached-web nginx:alpine

# Interactive shell as main process
docker run -it --name alpine-shell alpine:3.22 sh

# Detached interactive TTY container
docker run -dit --name detachable-shell alpine:3.22 sh

# Attach to existing main process streams
docker attach detachable-shell

# Default detach sequence for suitable -it attachment
Ctrl+P, Ctrl+Q

# Start additional command in running container
docker exec detached-web hostname
docker exec -it detached-web sh

# Use shell for operators
docker exec detached-web sh -c 'echo one && echo two'

# Logs
docker logs detached-web
docker logs --tail 20 detached-web
docker logs -f detached-web
docker logs -t --tail 20 detached-web

# Start existing container with input/output attachment
docker start -ai alpine-shell
```

### Critical distinctions

```text
Foreground ≠ interactive
Detached   ≠ stopped
Exit       ≠ detach
Attach     ≠ exec
Exec process ≠ new container
Logs       ≠ every file inside container
```

---

## Challenge — Identify the Process You Are Controlling

### Scenario

Your senior engineer gives you an Nginx container and asks you to prove the difference between observing it, attaching to it, and executing a diagnostic shell.

### Required workflow

1. Start `nginx:alpine` detached as `interaction-challenge`
2. Prove that the prompt returned and container remained running
3. Read its last 20 log lines with timestamps
4. Open an exec shell
5. Use `ps` to identify Nginx and the shell
6. Exit the exec shell
7. Prove Nginx remains running
8. Explain what attach would connect to
9. Follow logs and leave with `Ctrl+C`
10. Prove leaving log follow did not stop the container
11. Stop and remove it gracefully

### Rules

- Do not modify the Docker socket
- Do not use forced removal
- Do not claim attach creates a shell
- Do not claim exec creates a container
- Verify state after every exit/disconnection

### Report template

```text
Container ID:
Main process:
Why -d returned the prompt:
Log command used:
Exec process used:
Why exec exit did not stop container:
What attach would connect to:
Why leaving log follow was safe:
Final cleanup evidence:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-05/
├── 01-foreground-state.txt
├── 02-detached-container.txt
├── 03-detached-logs.txt
├── 04-alpine-shell-processes.txt
├── 05-main-shell-exit.txt
├── 06-detach-proof.txt
├── 07-nginx-processes-with-exec.txt
├── 08-exec-exit-proof.txt
├── 09-follow-logs-proof.txt
└── interaction-analysis.md
```

Suggested safe captures:

```bash
docker ps --filter name=detached-web > 02-detached-container.txt
docker logs --timestamps --tail 20 detached-web > 03-detached-logs.txt 2>&1
docker exec detached-web ps > 07-nginx-processes-with-exec.txt
```

Do not save credentials, secrets, or unrelated environment information.

---

## Official References

- [Docker container run](https://docs.docker.com/reference/cli/docker/container/run/)
- [Docker container attach](https://docs.docker.com/reference/cli/docker/container/attach/)
- [Docker container exec](https://docs.docker.com/reference/cli/docker/container/exec/)
- [Docker container logs](https://docs.docker.com/reference/cli/docker/container/logs/)

---

## Next Lesson

**Lesson 06 — Docker Images**

You now know how containers run and how your terminal interacts with their processes. Next, you will study the reusable packages from which those containers are created.

You will learn:

```bash
docker pull
docker image ls
docker image inspect
docker image history
docker image rm
docker image prune
```

And understand:

- Repository and tag
- Image ID
- Digest
- Official images
- One image creating many containers
- Pulling and removing images safely
- Why an existing container may block image removal

---

## Completion Check

Before moving forward, explain without reading:

```text
STDIN, STDOUT, and STDERR
Foreground versus detached
-i versus -t
Exit versus detach
Attach versus exec
Main process versus exec process
Logs versus attach
Why exiting a main shell stops its container
Why exiting an exec shell usually does not
Why detached is not a container state
```

If you can prove each difference using container state and process output, you are ready for Docker images.

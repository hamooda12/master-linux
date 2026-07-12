# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 07 — Container Commands, `CMD`, and `ENTRYPOINT`

> **Level:** Beginner to early intermediate  
> **Type:** Concept plus guided practical lesson  
> **Lab images:** `alpine:3.22` and `nginx:alpine`  
> **Estimated study time:** 140–180 minutes

---

## Overview

You already know that a container runs while its main process runs.

Now we answer the next question:

> Which process does Docker start?

The answer comes from two places:

1. Defaults stored in the image
2. Overrides supplied when the container is created

Docker images can define:

- `CMD` — default command or default arguments
- `ENTRYPOINT` — the executable the container is designed to run

These instructions can appear in two forms:

- Exec form using a JSON array
- Shell form using a command string

In this lesson, you will inspect existing images, override their commands, compare short and long-running processes, and understand `CMD` and `ENTRYPOINT` before writing a Dockerfile yourself.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain what the container's main process is
- Explain why the container stops when that process exits
- Inspect an image's `Config.Cmd` and `Config.Entrypoint`
- Override an image's default command at runtime
- Distinguish a command from its arguments
- Explain the purpose of `CMD`
- Explain the purpose of `ENTRYPOINT`
- Predict how exec-form `ENTRYPOINT` and `CMD` combine
- Explain shell form versus exec form
- Explain why exec form does not automatically perform shell expansion
- Use `sh -c` deliberately when shell interpretation is required
- Use `docker top` to inspect container processes
- Distinguish the main process from child and exec processes
- Understand PID 1 at the correct introductory depth
- Diagnose containers that exit immediately because of command behavior

---

## Prerequisites

You should understand:

- Image and container
- Image configuration metadata
- `docker run`
- Foreground and detached mode
- Main process at a basic level
- `docker exec`
- Container exit codes
- `docker inspect`
- Basic Linux processes and signals

Examples use:

```bash
docker ...
```

Use your chosen Docker access model consistently.

---

## Why This Topic Exists

Consider these commands:

```bash
docker run --rm alpine:3.22
```

```bash
docker run --rm alpine:3.22 echo "hello"
```

```bash
docker run -d --name sleeper alpine:3.22 sleep 300
```

The same image behaves differently because different commands run.

```text
Alpine default shell without interactive input → may exit quickly
echo                                      → prints and exits
sleep 300                                 → remains running for 300 seconds
```

The container did not decide independently to stay alive. Its process behavior determined the lifecycle.

This matters in production because a container may exit immediately even though:

- Docker is healthy
- The image pulled successfully
- Networking is fine
- Permissions are correct

The actual application command may simply finish or fail.

---

# Part 1 — The Main Container Process

## 1. Main Process Mental Model

When Docker starts a container, it starts the configured main process.

Inside the container's process view, this process normally has process ID 1.

```text
Container starts
       ↓
Main process starts as PID 1
       ↓
Main process running → container running
       ↓
Main process exits → container stops
```

## 2. Container Is Not Kept Alive by Docker Magic

Docker does not need an empty container to “stay awake.”

The container represents a running process environment. If the main process has no more work and exits, the container stops normally.

```text
Short command → short container lifetime
Server process → potentially long container lifetime
```

## 3. Short Process Example

```bash
docker run --name short-command alpine:3.22 echo "task complete"
```

Expected output:

```text
task complete
```

Then:

```bash
docker ps -a --filter name=short-command
```

The container is stopped because `echo` finished.

This is successful behavior when the intended task was to print a message.

## 4. Long-running Process Example

```bash
docker run -d --name long-command alpine:3.22 sleep 300
```

Verify:

```bash
docker ps --filter name=long-command
```

The container remains running because `sleep` has not exited.

## 5. Inspect the Main Process

```bash
docker top long-command
```

Typical output includes a process command similar to:

```text
sleep 300
```

`docker top` asks Docker to display processes running in the container. Its output format and host-visible PID columns can vary by platform and options.

## 6. Inspect Configured Path and Arguments

```bash
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  long-command
```

Typical result:

```text
path=sleep args=["300"]
```

This separates:

```text
Executable/path → sleep
Argument         → 300
```

## 7. Command Versus Arguments

In:

```bash
sleep 300
```

```text
sleep → executable/command
300   → argument passed to sleep
```

In:

```bash
echo hello world
```

```text
echo        → command
hello world → arguments
```

The program decides what its arguments mean.

---

# Part 2 — Image Defaults

## 8. Where the Default Comes From

An image can store startup configuration in:

```text
Config.Entrypoint
Config.Cmd
```

Inspect Alpine:

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  alpine:3.22
```

Inspect Nginx:

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  nginx:alpine
```

Actual values can change between image versions. Read the output rather than memorizing a snapshot.

## 9. `null` Does Not Mean No Process Can Run

One field may be `null` while the other defines the executable or command.

The final startup command is formed from:

- Image entrypoint
- Image command/default arguments
- Runtime command or arguments
- Runtime entrypoint override if specified

Do not inspect only one field and conclude that the image cannot run.

## 10. Inspect Container's Final Configuration

After creating a container:

```bash
docker create --name command-inspect alpine:3.22 sleep 300
```

Inspect:

```bash
docker inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}} path={{.Path}} args={{json .Args}}' \
  command-inspect
```

The container configuration shows the result after runtime overrides were applied.

Remove:

```bash
docker rm command-inspect
```

---

# Part 3 — Runtime Command Override

## 11. `docker run` Syntax Revisited

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Text after the image reference is not another Docker option. It supplies a runtime command or arguments according to the image's entrypoint configuration.

## 12. Override Alpine's Default

```bash
docker run --rm alpine:3.22 echo "override worked"
```

The runtime command replaces Alpine's default `CMD`.

## 13. Run a Shell Expression Correctly

This uses a shell explicitly:

```bash
docker run --rm alpine:3.22 sh -c 'echo first && echo second'
```

Breakdown:

```text
sh            → executable
-c            → interpret the next argument as shell commands
echo ... &&... → command string interpreted by sh
```

## 14. Why Quotes Alone Are Not a Shell

This is incorrect:

```bash
docker run --rm alpine:3.22 "echo first && echo second"
```

Docker tries to execute a file whose name is the entire quoted string. It does not automatically add a shell merely because the argument contains spaces or `&&`.

Use:

```bash
docker run --rm alpine:3.22 sh -c 'echo first && echo second'
```

## 15. Run Nginx With an Override

Before doing this, inspect Nginx's entrypoint and command:

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  nginx:alpine
```

Now run:

```bash
docker run --rm nginx:alpine nginx -v
```

Depending on the image's entrypoint design, the runtime text after the image replaces its default `CMD` and is combined with the entrypoint.

The command prints version information and exits; therefore the container stops and is automatically removed.

## 16. Runtime Override Is Per Container

Running:

```bash
docker run --rm alpine:3.22 echo hello
```

does not modify the Alpine image.

The next container can use another command:

```bash
docker run --rm alpine:3.22 uname -a
```

Image configuration remains unchanged.

---

# Part 4 — `CMD`

## 17. Purpose of `CMD`

`CMD` provides defaults for an executing container.

It may provide:

1. The default executable plus arguments
2. Default arguments for an `ENTRYPOINT`

## 18. Exec-form `CMD`

Dockerfile example:

```dockerfile
CMD ["python", "app.py"]
```

This says:

```text
Default executable: python
Default argument:   app.py
```

No shell is automatically inserted.

## 19. Shell-form `CMD`

```dockerfile
CMD python app.py
```

Shell form runs through a command shell.

Conceptually on a typical Linux image:

```text
/bin/sh -c "python app.py"
```

The exact default shell can be affected by image platform and Dockerfile `SHELL` configuration.

## 20. `CMD` as Default Arguments

With an exec-form entrypoint:

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```

Default final command:

```text
python app.py --port 8080
```

Runtime override:

```bash
docker run my-api --port 9090
```

Final command:

```text
python app.py --port 9090
```

The entrypoint remains; the runtime arguments replace the `CMD` defaults.

## 21. Only the Last `CMD` Applies

If a Dockerfile contains multiple `CMD` instructions in the same build stage:

```dockerfile
CMD ["echo", "first"]
CMD ["echo", "second"]
```

Only the last `CMD` supplies the final default.

This does not run two commands.

---

# Part 5 — `ENTRYPOINT`

## 22. Purpose of `ENTRYPOINT`

`ENTRYPOINT` configures the container to behave like a particular executable.

Exec-form example:

```dockerfile
ENTRYPOINT ["ping"]
CMD ["-c", "4", "127.0.0.1"]
```

Default:

```text
ping -c 4 127.0.0.1
```

Runtime:

```bash
docker run my-ping -c 2 example.com
```

Final:

```text
ping -c 2 example.com
```

## 23. Exec-form `ENTRYPOINT`

```dockerfile
ENTRYPOINT ["/usr/local/bin/product-api"]
```

Advantages:

- Direct executable invocation
- Clear argument boundaries
- Better signal delivery to the intended application
- Runtime text after the image naturally becomes arguments

This is normally the preferred form for an entrypoint.

## 24. Shell-form `ENTRYPOINT`

```dockerfile
ENTRYPOINT /usr/local/bin/product-api
```

This invokes the command through a shell. The shell becomes involved in the process tree and signal handling.

It also prevents runtime `CMD` arguments from being appended in the useful way expected with exec-form entrypoint design.

Use shell form only when shell behavior is deliberately required and understood.

## 25. Only the Last `ENTRYPOINT` Applies

Like `CMD`, only the last `ENTRYPOINT` in a build stage supplies the final entrypoint.

## 26. Override Entrypoint at Runtime

Docker provides:

```bash
docker run --entrypoint EXECUTABLE IMAGE [ARG...]
```

Example:

```bash
docker run --rm --entrypoint sh nginx:alpine -c 'echo entrypoint-overridden'
```

This replaces the image's configured entrypoint for this new container.

It does not modify the image.

### Production caution

Overriding the entrypoint can bypass image initialization scripts, permission preparation, configuration rendering, or safety checks.

Use it for deliberate diagnostics, not as a blind workaround.

---

# Part 6 — How `CMD` and `ENTRYPOINT` Combine

## 27. Core Combination Table

| Image configuration | Runtime text after image | Result |
| --- | --- | --- |
| `CMD ["app", "--safe"]` only | none | `app --safe` |
| `CMD ["app", "--safe"]` only | `other --fast` | `other --fast` |
| `ENTRYPOINT ["app"]` + `CMD ["--safe"]` | none | `app --safe` |
| `ENTRYPOINT ["app"]` + `CMD ["--safe"]` | `--fast` | `app --fast` |
| `ENTRYPOINT ["app"]` | `--fast` | `app --fast` |

This table assumes exec-form designs.

## 28. The Best Beginner Rule

```text
CMD only:
runtime command replaces CMD

Exec ENTRYPOINT + CMD:
ENTRYPOINT stays
runtime arguments replace CMD defaults
```

## 29. Application-design Pattern

Use:

```dockerfile
ENTRYPOINT ["product-api"]
CMD ["--config", "/etc/product-api/config.yml"]
```

when the container should always behave like `product-api`, but users may replace its default arguments.

Use:

```dockerfile
CMD ["product-api", "--config", "/etc/product-api/config.yml"]
```

when users should easily replace the entire command.

Neither pattern is automatically correct for all images. Choose based on intended interface.

---

# Part 7 — Exec Form Versus Shell Form

## 30. Exec Form Syntax

```dockerfile
CMD ["executable", "arg1", "arg2"]
ENTRYPOINT ["executable", "arg1"]
```

It is JSON array syntax:

- Use double quotes
- Each argument is a separate element
- No shell is inserted automatically

## 31. Shell Form Syntax

```dockerfile
CMD executable arg1 arg2
ENTRYPOINT executable arg1
```

A command shell interprets the string.

## 32. Shell Features

These features require a shell interpreter:

- Variable expansion such as `$HOME`
- Globbing such as `*.log`
- Pipelines such as `cmd1 | cmd2`
- Redirection such as `> file`
- Operators such as `&&` and `||`

Exec form does not provide these automatically.

## 33. Explicit Shell With Exec Form

If shell behavior is required, invoke it explicitly:

```dockerfile
CMD ["sh", "-c", "echo $HOME && exec myapp"]
```

Now `sh` performs expansion and operators.

The final `exec myapp` replaces the shell with `myapp`, allowing the application to become the main process. Wrapper scripts and `exec` are explored more deeply in the image-building and signal lessons.

## 34. Variable Expansion Example

Runtime comparison:

```bash
docker run --rm -e NAME=Hamad alpine:3.22 echo '$NAME'
```

Because Docker directly invokes `echo`, no container shell expands `$NAME`; output is typically literal:

```text
$NAME
```

With a shell:

```bash
docker run --rm -e NAME=Hamad alpine:3.22 sh -c 'echo "$NAME"'
```

Output:

```text
Hamad
```

The host shell's single quotes prevent host-side expansion, and the container's `sh` expands the variable.

## 35. Quoting Has Multiple Layers

In a command like:

```bash
docker run --rm alpine:3.22 sh -c 'echo "$HOME"'
```

There can be:

1. Host-shell parsing
2. Docker argument transfer
3. Container-shell parsing

Quoting decides which layer interprets special characters.

Do not memorize random quote combinations. Ask:

> Which shell should expand this value?

---

# Part 8 — Main, Child, and Exec Processes

## 36. Main Process Can Create Children

A container can contain more than one process.

```text
PID 1: main application
├── child process A
├── child process B
└── worker process C
```

For example, Nginx normally has a master process and worker processes.

## 37. Inspect Nginx Processes

```bash
docker run -d --name process-nginx nginx:alpine
docker top process-nginx
```

You may see:

- Entrypoint/main Nginx process
- Worker processes

Exact user names, process arguments, and counts can vary.

## 38. Exec Adds Another Process

Open a second terminal and run:

```bash
docker exec -it process-nginx sh
```

From another terminal:

```bash
docker top process-nginx
```

Now the process list includes the additional shell.

Exit the exec shell:

```sh
exit
```

Run `docker top` again. The shell disappears, while Nginx remains.

## 39. Main Process Controls Container Lifetime

Child or exec processes can exit without stopping the container if the main process remains.

If the main process exits, Docker considers the container stopped; remaining processes do not make it a normal running container.

```text
Exec shell exits + main app runs → container runs
Main app exits                  → container stops
```

---

# Part 9 — PID 1 Introduction

## 40. What PID 1 Means Here

Inside a normal container process namespace, the main process appears as PID 1.

PID 1 has special Linux behavior and responsibilities, including:

- Signal-handling differences
- Reaping orphaned child processes

You do not need deep namespace or init-system knowledge yet.

The practical lesson is:

> The application or entrypoint used as PID 1 should handle signals and child processes correctly.

## 41. Why Shell Wrappers Matter

Poor pattern:

```text
PID 1: shell script
└── application child
```

If the wrapper does not forward signals or replace itself with the application, graceful shutdown may not reach the application as intended.

Better wrapper ending:

```sh
exec "$@"
```

`exec` replaces the wrapper process with the target command instead of keeping the application as its child.

Full wrapper-script safety comes later.

## 42. Docker `--init` Preview

Docker supports:

```bash
docker run --init IMAGE
```

This inserts a small init process as PID 1 to perform responsibilities such as reaping child processes and forwarding signals.

Do not add it blindly. First design the application process model correctly, then use an init when the workload benefits from it.

---

## Big Picture

```text
Image configuration
├── ENTRYPOINT: executable identity
└── CMD: default command or arguments
              │
              ▼
docker run IMAGE [runtime text]
              │
              ▼
Final executable + arguments
              │
              ▼
Main process (PID 1)
              │
              ├── child processes
              └── exec processes added later
              │
              ▼
Main process exits → container stops
```

---

## Commands — Detailed Reference

## 43. Override Default Command

```bash
docker run --rm alpine:3.22 echo hello
docker run --rm alpine:3.22 uname -a
docker run --rm alpine:3.22 sh -c 'echo one && echo two'
```

## 44. Override Entrypoint

```bash
docker run --rm --entrypoint sh nginx:alpine -c 'echo diagnostic'
```

Overrides the image entrypoint for this new container.

## 45. Inspect Image Command Metadata

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

## 46. Inspect Final Container Command

```bash
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  CONTAINER
```

## 47. Show Container Processes

```bash
docker top CONTAINER
```

## 48. Run Additional Process

```bash
docker exec CONTAINER COMMAND
docker exec -it CONTAINER sh
```

The exec command is not the main process and exists only while the container's primary process is running.

---

## Guided Hands-on Lab — Prove the Final Command

### Scenario

TechCorp received two images. Before deployment, you must prove their defaults, test overrides, and identify which process controls container lifetime.

### Phase A — Inspect Defaults

```bash
docker pull alpine:3.22
docker pull nginx:alpine

docker image inspect --format \
  'image=alpine entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  alpine:3.22

docker image inspect --format \
  'image=nginx entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  nginx:alpine
```

Record actual output.

### Phase B — Short Commands

```bash
docker run --name cmd-echo alpine:3.22 echo "complete"
docker ps -a --filter name=cmd-echo
docker inspect --format \
  'path={{.Path}} args={{json .Args}} exit={{.State.ExitCode}}' \
  cmd-echo
```

Explain why the container stopped successfully.

### Phase C — Long Command

```bash
docker run -d --name cmd-sleep alpine:3.22 sleep 300
docker ps --filter name=cmd-sleep
docker top cmd-sleep
docker inspect --format \
  'path={{.Path}} args={{json .Args}} status={{.State.Status}}' \
  cmd-sleep
```

### Phase D — Shell Interpretation

```bash
docker run --rm -e NAME=Hamad alpine:3.22 echo '$NAME'
docker run --rm -e NAME=Hamad alpine:3.22 sh -c 'echo "$NAME"'
```

Explain why output differs.

### Phase E — Nginx Default

```bash
docker run -d --name cmd-nginx nginx:alpine
docker top cmd-nginx
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  cmd-nginx
```

### Phase F — Additional Exec Process

Terminal 1:

```bash
docker exec -it cmd-nginx sh
```

Terminal 2:

```bash
docker top cmd-nginx
```

Exit Terminal 1's shell, then repeat `docker top`.

Prove the exec shell disappeared while Nginx remained.

### Phase G — Nginx Runtime Override

```bash
docker run --name nginx-version nginx:alpine nginx -v
docker ps -a --filter name=nginx-version
docker inspect --format \
  'path={{.Path}} args={{json .Args}} exit={{.State.ExitCode}}' \
  nginx-version
docker logs nginx-version
```

Explain how the image entrypoint and runtime command combined based on actual inspection.

### Phase H — Entrypoint Override

```bash
docker run --rm --entrypoint sh nginx:alpine \
  -c 'echo "custom entrypoint"; ps'
```

Explain what image startup behavior was bypassed.

### Phase I — Cleanup

```bash
docker stop cmd-sleep cmd-nginx
docker rm cmd-echo cmd-sleep cmd-nginx nginx-version
```

Verify names before removing them.

### Acceptance Criteria

You can prove:

- Alpine and Nginx image defaults
- Runtime command replacement for Alpine
- Short command exit versus long command running state
- Exact final `.Path` and `.Args`
- Why shell expansion required `sh -c`
- Nginx's main and worker processes
- Exec added a process without adding a container
- Exiting exec did not stop Nginx
- Entrypoint override changed startup behavior only for that container

---

## Discovery Questions

### Question 1

Why does `docker run --rm alpine echo hello` exit immediately?

<details>
<summary>Explanation</summary>

The runtime command `echo hello` replaces Alpine's default command. Echo prints and exits, so the main process ends and the container stops.

</details>

### Question 2

If an image has exec-form `ENTRYPOINT ["app"]` and `CMD ["--safe"]`, what does `docker run image --fast` execute?

<details>
<summary>Explanation</summary>

It executes `app --fast`. The runtime arguments replace the CMD default while the entrypoint remains.

</details>

### Question 3

Why does exec form not expand `$HOME` automatically?

<details>
<summary>Explanation</summary>

Exec form invokes the executable directly and does not automatically start a shell. A shell is normally responsible for variable expansion.

</details>

### Question 4

Why does exiting a `docker exec` shell not normally stop Nginx?

<details>
<summary>Explanation</summary>

The exec shell is an additional process. Nginx remains the container's main process.

</details>

### Question 5

Why might shell-form entrypoint complicate graceful shutdown?

<details>
<summary>Explanation</summary>

A shell may become PID 1 while the application is its child. Signal handling and forwarding then depend on the shell/wrapper design.

</details>

---

## Production Scenario — “Container Keeps Exiting”

TechCorp deploys an application with this image default:

```dockerfile
CMD ["python", "migrate.py"]
```

The migration completes successfully and exits with code 0. Operations expected a permanent API server and reports that Docker “cannot keep the container alive.”

The root cause is an incorrect startup command for the intended workload.

Evidence:

```bash
docker ps -a
docker logs CONTAINER
docker inspect --format \
  'path={{.Path}} args={{json .Args}} exit={{.State.ExitCode}}' \
  CONTAINER
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

The correction is not to add a fake infinite process merely to keep the container alive. The image or runtime configuration should launch the real API server as the main process.

```text
Correct goal:
Main process = actual workload

Wrong workaround:
Main process = meaningless sleep forever
application managed separately
```

---

## Troubleshooting Workflow

## 49. Container Exits Immediately

Collect:

```bash
docker ps -a
docker logs CONTAINER
docker inspect --format \
  'path={{.Path}} args={{json .Args}} status={{.State.Status}} exit={{.State.ExitCode}} error={{.State.Error}}' \
  CONTAINER
```

Ask:

- Was the command intended to be short?
- Did the executable exist?
- Were arguments valid?
- Did configuration or dependencies fail?
- Was a shell operator passed without a shell?

## 50. `executable file not found`

Possible causes:

- Wrong executable name
- Executable absent from image
- Incorrect `PATH`
- Entire shell expression passed as one executable name
- Script lacks correct interpreter or permissions

Inspect image contents only through a deliberate diagnostic method.

## 51. Variables Do Not Expand

Ask which process should perform expansion.

Direct exec:

```bash
docker run IMAGE echo '$VAR'
```

Shell interpretation:

```bash
docker run IMAGE sh -c 'echo "$VAR"'
```

Also verify the variable actually exists:

```bash
docker run --rm -e VAR=value IMAGE env
```

## 52. Runtime Arguments Seem Ignored

Inspect entrypoint form:

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

Shell-form `ENTRYPOINT` and wrapper scripts can change argument behavior.

## 53. Stop Signal Does Not Reach Application

Inspect process tree:

```bash
docker top CONTAINER
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  CONTAINER
```

Check whether a shell/wrapper is PID 1 and whether it uses `exec` or forwards signals.

---

## Common Mistakes

### 1. Thinking the container must run forever

Its correct lifetime depends on the workload.

### 2. Blaming Docker when a short command finishes

Inspect the intended command and exit code.

### 3. Confusing command with arguments

The executable and its argument list are separate.

### 4. Believing quotes automatically add a shell

Docker does not interpret `&&`, pipes, or variables without a shell.

### 5. Treating `CMD` as unchangeable

Runtime text after the image can override CMD.

### 6. Treating exec-form `ENTRYPOINT` arguments as a full replacement

Runtime arguments normally replace CMD defaults and are appended to the entrypoint.

### 7. Using shell-form entrypoint without understanding PID 1

It can complicate arguments and signals.

### 8. Using `sleep infinity` to hide a broken application command

Run the real workload as the main process.

### 9. Assuming exec creates another container

It adds a process to an existing running container.

### 10. Overriding an image entrypoint casually

You may bypass required initialization.

---

## Best Practices

- Run the real application as the container's main process
- Prefer exec-form `ENTRYPOINT` for executable-style images
- Use exec-form `CMD` for default arguments
- Use `CMD` alone when easy full-command replacement is desired
- Invoke `sh -c` explicitly only when shell behavior is needed
- End wrapper scripts with `exec "$@"` when appropriate
- Design PID 1 to handle termination signals
- Keep entrypoint scripts small, predictable, and tested
- Document supported runtime arguments
- Inspect final path and arguments during incidents
- Avoid overriding entrypoints in ordinary production deployment
- Do not keep broken containers alive with meaningless background loops

---

## Interview Questions

### Junior

1. What is the container's main process?
2. Why does a container stop when its main process exits?
3. What is the purpose of `CMD`?
4. What is the purpose of `ENTRYPOINT`?
5. How do you override an image's CMD at runtime?
6. What is the difference between a command and an argument?
7. What does `docker top` show?
8. Does `docker exec` create a new container?
9. What is PID 1 inside a container?
10. Why does `echo` create a short-lived container?

### Intermediate

1. How do exec-form `ENTRYPOINT` and `CMD` combine?
2. What happens to CMD when runtime arguments are supplied?
3. What is the difference between exec and shell forms?
4. Why does exec form not expand variables automatically?
5. Why can shell-form entrypoint interfere with signals?
6. What does `exec "$@"` do in a wrapper?
7. When should you override `--entrypoint`?
8. How would you diagnose an immediate exit?
9. Why is `sleep infinity` often a bad production workaround?

### Senior Preview

Signal dispositions for PID 1, zombie reaping, init shims, process supervision, orchestrator command/args mapping, and entrypoint supply-chain controls are deferred until advanced lessons.

---

## Lesson Reflection

Answer in your own words:

1. How does Docker decide which process to start?
2. Explain CMD-only override behavior.
3. Explain exec ENTRYPOINT plus CMD behavior.
4. Why do runtime arguments replace CMD but preserve exec entrypoint?
5. Explain shell form versus exec form.
6. Which process expands `$VAR` in a `sh -c` command?
7. Why can Nginx have several processes in one container?
8. Why can an exec shell exit without stopping Nginx?
9. What special concerns exist for PID 1?
10. What evidence proves a container's final executable and arguments?

---

## Summary

- Docker starts a main process for every running container
- That process normally appears as PID 1 inside the container
- The container stops when its main process exits
- Short commands create short-lived containers
- Image defaults come from `CMD` and `ENTRYPOINT`
- `CMD` provides a default command or arguments
- `ENTRYPOINT` defines the executable-style identity of a container
- Runtime text replaces CMD defaults
- With exec-form entrypoint, runtime text becomes entrypoint arguments
- Exec form invokes an executable directly without an automatic shell
- Shell form uses a command shell
- Shell expansion requires a shell
- A main process can create child processes
- `docker exec` adds another process to a running container
- PID 1 requires correct signal and child-process behavior

---

## Cheat Sheet

### Image inspection

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

### Container command inspection

```bash
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  CONTAINER
```

### Override command

```bash
docker run --rm alpine:3.22 echo hello
docker run --rm alpine:3.22 sleep 5
docker run --rm alpine:3.22 sh -c 'echo one && echo two'
```

### Override entrypoint

```bash
docker run --rm --entrypoint sh nginx:alpine -c 'echo diagnostic'
```

### Processes

```bash
docker top CONTAINER
docker exec -it CONTAINER sh
```

### Dockerfile forms

```dockerfile
# Exec form
CMD ["app", "--safe"]
ENTRYPOINT ["app"]

# Shell form
CMD app --safe
ENTRYPOINT app
```

### Combination rule

```text
CMD only + runtime command
→ runtime command replaces CMD

exec ENTRYPOINT + CMD + runtime args
→ ENTRYPOINT + runtime args
```

---

## Challenge — Predict, Run, Prove

### Scenario

Your senior gives you an unknown image and asks:

> “Do not guess why it exits. Prove its configured command, final command, process list, and exit result.”

Use `nginx:alpine` as the safe lab image.

### Required workflow

1. Inspect image entrypoint and CMD
2. Predict its default final command
3. Run it detached as `command-challenge`
4. Inspect `.Path` and `.Args`
5. Use `docker top`
6. Start an exec shell
7. Prove the shell is an additional process
8. Exit the shell and prove Nginx remains
9. Run another container with `nginx -v` as runtime text
10. Explain how entrypoint and runtime text combined
11. Run a container with `--entrypoint sh`
12. Explain what was bypassed
13. Clean up only your lab objects

### Rules

- No `sleep infinity` workaround
- No force removal
- No assumptions without inspection
- Do not edit the running container as a deployment fix
- State which process performs every variable expansion

### Report template

```text
Image ENTRYPOINT:
Image CMD:
Predicted default:
Actual container Path:
Actual container Args:
Main process:
Child processes:
Exec process:
Why exec exit did not stop container:
Runtime override result:
Entrypoint override result:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-07/
├── 01-alpine-command-metadata.txt
├── 02-nginx-command-metadata.txt
├── 03-short-command-result.txt
├── 04-long-command-processes.txt
├── 05-shell-expansion-comparison.txt
├── 06-nginx-main-processes.txt
├── 07-nginx-with-exec-process.txt
├── 08-runtime-override-result.txt
├── 09-entrypoint-override-result.txt
└── command-analysis.md
```

Suggested captures:

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  nginx:alpine > 02-nginx-command-metadata.txt

docker top cmd-nginx > 06-nginx-main-processes.txt
```

Do not store secrets or unrelated environment information.

---

## Official References

- [Dockerfile reference: shell and exec forms](https://docs.docker.com/reference/dockerfile/#shell-and-exec-form)
- [Dockerfile reference: CMD](https://docs.docker.com/reference/dockerfile/#cmd)
- [Dockerfile reference: ENTRYPOINT](https://docs.docker.com/reference/dockerfile/#entrypoint)
- [Docker container run](https://docs.docker.com/reference/cli/docker/container/run/)
- [Docker container top](https://docs.docker.com/reference/cli/docker/container/top/)

---

## Next Lesson

**Lesson 08 — Inspecting Containers and Reading Logs**

You have already used inspection and logs in small ways. Next, you will combine them into an evidence-first troubleshooting workflow.

You will study:

```bash
docker inspect
docker logs
docker logs -f
docker logs --tail
docker top
docker stats
docker events
docker diff
```

The lab will run a deliberately failing container, identify its final command and exit code, read its output, inspect state changes, and explain the root cause before any correction.

---

## Completion Check

Before moving forward, explain without reading:

```text
Main process and PID 1
Why process exit stops container
CMD purpose
ENTRYPOINT purpose
CMD-only runtime override
Exec ENTRYPOINT plus CMD combination
Exec form versus shell form
Why shell expansion requires a shell
Main versus child versus exec process
Why wrapper scripts often use exec
```

If you can predict and prove the final executable and arguments, you are ready for systematic container inspection and logging.

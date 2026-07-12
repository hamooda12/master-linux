# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 08 — Inspecting Containers and Reading Logs

> **Level:** Beginner to intermediate  
> **Type:** Evidence-first troubleshooting lesson  
> **Lab images:** `alpine:3.22` and `nginx:alpine`  
> **Estimated study time:** 150–190 minutes

---

## Overview

When a container fails, the first job is not to restart, rebuild, or delete it.

The first job is to answer:

```text
What happened?
```

Docker exposes different kinds of evidence:

| Evidence | Primary command |
| --- | --- |
| Container list and summarized state | `docker ps -a` |
| Detailed configuration and state | `docker inspect` |
| Application output | `docker logs` |
| Running processes | `docker top` |
| Live resource usage | `docker stats` |
| Docker daemon activity | `docker events` |
| Writable-filesystem changes | `docker diff` |

This lesson combines those tools into one professional workflow:

```text
Observe → collect evidence → form hypothesis → test → correct → verify
```

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Distinguish configuration from runtime state
- Read container status, exit code, error, and timestamps
- Inspect the final executable and arguments
- Use formatted inspection without memorizing the entire JSON document
- Explain where `docker logs` output comes from
- Filter logs by tail count and time
- Follow new logs safely
- Use `docker top` to inspect processes
- Interpret the main `docker stats` columns
- Capture a one-time statistics sample
- Use Docker events to build a lifecycle timeline
- Interpret `docker diff` symbols `A`, `C`, and `D`
- Diagnose a deliberately failing container without deleting evidence
- Explain the limitations of each diagnostic command
- Produce a concise root-cause report

---

## Prerequisites

You should understand:

- Container states and lifecycle
- Main process and PID 1
- Exit codes
- `CMD`, `ENTRYPOINT`, and runtime overrides
- `STDOUT` and `STDERR`
- `docker ps` and `docker ps -a`
- `docker exec`
- Image versus container writable layer

Use your established Docker access model consistently.

---

## Why This Topic Exists

Consider this report:

```text
The product-api container is not running.
```

That is a symptom, not a root cause.

Possible explanations include:

- The command completed successfully
- The executable does not exist
- An argument is invalid
- A configuration file is missing
- The application logged an error and exited
- The process was killed
- Docker could not start the process
- The container was manually stopped
- A resource limit was reached

Restarting before collecting evidence can:

- Change timestamps
- Add new log entries
- Hide the original exit behavior
- Trigger a loop
- Delay the real diagnosis

Removing the container can destroy even more evidence.

---

# Part 1 — Evidence Categories

## 1. Configuration

Configuration describes how the container was created:

- Image and image ID
- Entrypoint and command
- Environment variables
- Working directory
- User
- Mounts
- Networks
- Port bindings
- Resource limits
- Restart policy

Configuration answers:

> What was Docker asked to create?

## 2. Runtime State

State describes what happened during execution:

- Created, running, paused, restarting, or exited
- Start and finish timestamps
- Exit code
- Runtime error
- OOM-killed flag
- Process ID while running
- Health status when a healthcheck exists

State answers:

> What happened when Docker tried to run it?

## 3. Application Output

Logs normally expose output written through the container's standard output and standard error streams, as handled by the logging driver.

Logs answer:

> What did the application say?

## 4. Current Behavior

Processes and resource statistics answer:

> What is it doing now?

These tools apply mainly to running containers.

## 5. Change and Timeline Evidence

Events show Docker actions over time. Diff shows filesystem changes relative to the image.

They answer:

```text
Which Docker events occurred?
What changed inside the writable layer?
```

---

# Part 2 — Start With `docker ps -a`

## 6. First Command

```bash
docker ps -a
```

It quickly answers:

- Does the container exist?
- Is it running or exited?
- What image and command are summarized?
- When was it created?
- What exit code appears?
- What is its name?

## 7. Filter by Name

```bash
docker ps -a --filter name=product-api
```

The name filter can match partial names. Verify the exact selected container.

## 8. Filter by State

```bash
docker ps -a --filter status=exited
docker ps -a --filter status=running
docker ps -a --filter status=restarting
```

## 9. Filter by Exit Code

```bash
docker ps -a --filter exited=1
docker ps -a --filter exited=137
```

Exit-code filtering is useful for grouping symptoms, but a code alone does not prove root cause.

## 10. Custom Table

```bash
docker ps -a --format \
  'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Command}}'
```

Formatted output is useful for reports and narrow operational views.

Do not hide fields that may be important to the investigation.

---

# Part 3 — `docker inspect`

## 11. Full Inspection

```bash
docker inspect CONTAINER
```

The result is a JSON array containing detailed information.

Do not try to memorize the entire structure. Learn the questions and the relevant paths.

## 12. State Summary

```bash
docker inspect --format \
  'status={{.State.Status}} running={{.State.Running}} exit={{.State.ExitCode}} error={{json .State.Error}}' \
  CONTAINER
```

## 13. Important State Fields

| Field | Meaning |
| --- | --- |
| `.State.Status` | Current Docker state word |
| `.State.Running` | Whether container is currently running |
| `.State.Paused` | Whether processes are paused |
| `.State.Restarting` | Whether it is restarting |
| `.State.ExitCode` | Last main-process exit code |
| `.State.Error` | Runtime-level error recorded by Docker |
| `.State.OOMKilled` | Whether out-of-memory killing was recorded |
| `.State.StartedAt` | Last start timestamp |
| `.State.FinishedAt` | Last finish timestamp |
| `.State.Pid` | Host PID for main process while applicable |

## 14. Exit Code Versus State Error

```text
ExitCode
→ result returned when the process ran and exited

State.Error
→ Docker/runtime error associated with starting or managing it
```

An executable-not-found failure may appear differently from an application that starts, logs an error, and exits with code 1.

Inspect both.

## 15. OOM Evidence

```bash
docker inspect --format \
  'oom_killed={{.State.OOMKilled}} exit={{.State.ExitCode}}' \
  CONTAINER
```

`OOMKilled=true` is strong evidence of an out-of-memory kill recorded for the container.

`ExitCode=137` alone is not sufficient proof of OOM. It often represents termination by signal 9, which can have other causes.

## 16. Final Executable and Arguments

```bash
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  CONTAINER
```

This answers:

> What did Docker actually try to execute?

## 17. Image Identity

```bash
docker inspect --format \
  'configured_image={{.Config.Image}} image_id={{.Image}}' \
  CONTAINER
```

```text
.Config.Image → reference supplied during creation
.Image        → image configuration ID used
```

## 18. Environment Variables

```bash
docker inspect --format '{{json .Config.Env}}' CONTAINER
```

### Security warning

Environment variables may contain credentials or secrets. Do not paste full output into tickets, chat, screenshots, or public logs.

Prefer checking a specific non-sensitive configuration value or redact carefully.

## 19. Working Directory and User

```bash
docker inspect --format \
  'user={{json .Config.User}} workdir={{json .Config.WorkingDir}}' \
  CONTAINER
```

Empty `User` commonly means the image's default user rules apply, often root, but verify actual process identity rather than assuming.

## 20. Restart Policy

```bash
docker inspect --format \
  'restart={{.HostConfig.RestartPolicy.Name}} max={{.HostConfig.RestartPolicy.MaximumRetryCount}} count={{.RestartCount}}' \
  CONTAINER
```

This helps distinguish one exit from an automatic restart loop.

## 21. Mounts

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
```

Mount analysis becomes central in the volumes lesson.

## 22. Networks and IP Addresses

```bash
docker inspect --format '{{json .NetworkSettings.Networks}}' CONTAINER
```

Networking receives dedicated lessons. Avoid diagnosing an application-command failure as networking merely because network fields exist.

## 23. Container Size

```bash
docker inspect --size CONTAINER
```

This adds size fields for the container filesystem.

Also:

```bash
docker ps -a --size
```

Writable-layer size is not the same as volume size, log size, or exact total unique disk usage.

---

# Part 4 — `docker logs`

## 24. Read All Available Logs

```bash
docker logs CONTAINER
```

It retrieves log output available through the container's configured logging driver.

## 25. Log Source

```text
Application STDOUT ─┐
                    ├→ logging driver → docker logs
Application STDERR ─┘
```

It does not search every file in the container.

## 26. Last Lines

```bash
docker logs --tail 50 CONTAINER
```

Use a bounded tail first on noisy services.

## 27. Timestamps

```bash
docker logs --timestamps --tail 50 CONTAINER
```

Short option:

```bash
docker logs -t --tail 50 CONTAINER
```

Docker adds RFC3339-style timestamps to returned entries.

## 28. Logs Since a Time

Relative example:

```bash
docker logs --since 10m CONTAINER
```

Absolute example:

```bash
docker logs --since '2026-07-12T15:00:00Z' CONTAINER
```

Use explicit timezone information when correlating systems.

## 29. Logs Until a Time

```bash
docker logs \
  --since '2026-07-12T15:00:00Z' \
  --until '2026-07-12T15:10:00Z' \
  CONTAINER
```

This isolates an incident window.

## 30. Follow New Output

```bash
docker logs --follow CONTAINER
```

Short form:

```bash
docker logs -f CONTAINER
```

Use `Ctrl+C` to stop the local follow client. The container normally continues running.

## 31. Combine Tail and Follow

```bash
docker logs --tail 20 --follow CONTAINER
```

This shows recent context and then waits for new lines.

## 32. STDOUT and STDERR Are Combined in Display

`docker logs` retrieves both container standard output and standard error. Depending on TTY and logging details, the displayed stream may not preserve a visual distinction.

Applications should include severity and structured fields in their log lines when operational separation matters.

## 33. When Logs Are Empty

Possible reasons:

- Application printed nothing
- It writes only to internal files
- Logging driver does not support local reading
- Wrong container selected
- Output occurred before/after the chosen time range
- Log rotation or external logging changed availability

Inspect logging driver:

```bash
docker inspect --format '{{.HostConfig.LogConfig.Type}}' CONTAINER
```

---

# Part 5 — `docker top`

## 34. Show Processes

```bash
docker top CONTAINER
```

This is useful when the container is running.

It helps answer:

- Is the expected application process present?
- Are worker processes present?
- Did an exec shell add another process?
- Is an unexpected shell or process running?

## 35. Top Does Not Work as Historical Process Evidence

Once the container is stopped, its processes no longer run. `docker top` cannot reconstruct the complete historical process tree.

Use:

- Logs
- Inspection
- Application tracing/monitoring
- External observability

for historical evidence.

## 36. Host PID and Container PID

Output may show host-visible process identifiers, while inside the container the main process normally appears as PID 1.

This difference comes from process isolation, which is taught later.

For now:

> A process can have a host-visible PID and a different PID inside the container.

---

# Part 6 — `docker stats`

## 37. Live Statistics

```bash
docker stats
```

It streams resource usage for running containers.

Limit to one:

```bash
docker stats CONTAINER
```

## 38. Main Columns

| Column | Beginner meaning |
| --- | --- |
| `CPU %` | Recent CPU usage reported for container |
| `MEM USAGE / LIMIT` | Current memory usage and configured/available limit |
| `MEM %` | Memory usage relative to limit |
| `NET I/O` | Network bytes received/sent |
| `BLOCK I/O` | Block-device read/write activity |
| `PIDS` | Process and kernel-task/thread count |

## 39. Stop the Live Stream

Press:

```text
Ctrl+C
```

This stops the stats client, not the containers.

## 40. One-time Snapshot

```bash
docker stats --no-stream CONTAINER
```

This prints one sample and exits.

## 41. Formatted Snapshot

```bash
docker stats --no-stream --format \
  'table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.PIDs}}' \
  CONTAINER
```

## 42. Statistics Are Evidence, Not Complete Diagnosis

One CPU or memory sample does not prove a trend or root cause.

Production analysis normally needs:

- Time-series metrics
- Application latency/error metrics
- Host resource metrics
- Limit and throttling information
- Workload context

Use `docker stats` as immediate evidence, not as a full monitoring platform.

---

# Part 7 — `docker events`

## 43. What Events Shows

```bash
docker events
```

It streams events reported by the Docker daemon, such as container creation, start, stop, die, kill, destroy, image pull, and network actions.

## 44. Use a Second Terminal

Terminal 1:

```bash
docker events
```

Terminal 2:

```bash
docker run --name event-demo alpine:3.22 echo event-test
docker rm event-demo
```

Terminal 1 should show a sequence of relevant events.

Stop observation with `Ctrl+C`.

## 45. Filter by Container

```bash
docker events --filter container=CONTAINER
```

## 46. Filter by Event Type

```bash
docker events --filter type=container
docker events --filter event=die
```

Multiple filters narrow the result:

```bash
docker events \
  --filter type=container \
  --filter container=product-api
```

## 47. Time-bounded Historical Query

```bash
docker events --since 10m --until 0s
```

Event availability is daemon-scoped and not an unlimited durable audit history. For production, forward and retain events/metrics in an observability system.

## 48. Events Versus Logs

```text
docker events → what Docker did
docker logs   → what application wrote
```

Example:

```text
Event: container die
Log: database connection refused
```

Together they form a stronger timeline.

---

# Part 8 — `docker diff`

## 49. Inspect Writable-layer Changes

```bash
docker diff CONTAINER
```

It compares the container filesystem changes against its image base.

## 50. Symbols

| Symbol | Meaning |
| --- | --- |
| `A` | File or directory added |
| `C` | File or directory changed |
| `D` | File or directory deleted |

## 51. Create Controlled Changes

```bash
docker run -d --name diff-demo alpine:3.22 sleep 300
docker exec diff-demo sh -c \
  'echo added > /tmp/added.txt; echo changed >> /etc/hosts; rm /etc/motd 2>/dev/null || true'
docker diff diff-demo
```

Actual output includes application/runtime changes and may include more paths than your explicit commands.

## 52. What Diff Does Not Show

`docker diff` does not show:

- File contents
- Exact change timestamps
- Who changed the file
- Changes inside mounted volumes in the same layer-comparison sense
- Complete audit history

It is a clue, not a forensic audit system.

## 53. Operational Uses

- Detect unexpected manual modifications
- Identify runtime-generated files
- Understand writable-layer growth
- Compare immutable-image expectations with reality
- Locate clues after compromise or misconfiguration

Do not turn container mutations into a new image using `docker commit` as the normal deployment workflow. Reproduce intended changes in a Dockerfile and source control.

---

# Part 9 — Deliberately Failing Container

## 54. Create a Failure With Useful Logs

Run:

```bash
docker run --name failing-api alpine:3.22 sh -c \
  'echo "INFO starting product API"; echo "ERROR missing DB_HOST" >&2; exit 42'
```

The container:

1. Starts a shell
2. Writes an information line to standard output
3. Writes an error line to standard error
4. Exits with code 42

## 55. Observe Summary

```bash
docker ps -a --filter name=failing-api
```

Expected state shape:

```text
Exited (42)
```

## 56. Inspect State

```bash
docker inspect --format \
  'status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}} oom={{.State.OOMKilled}}' \
  failing-api
```

Expected interpretation:

```text
status=exited
exit=42
runtime error likely empty
oom=false
```

## 57. Inspect Final Command

```bash
docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  failing-api
```

This proves that `sh -c` executed the scripted behavior.

## 58. Read Logs

```bash
docker logs --timestamps failing-api
```

Both the info and error lines should be available.

## 59. Root Cause

The root cause in this controlled scenario is:

```text
The main shell command deliberately exited with status 42 after reporting a missing DB_HOST configuration.
```

Do not say:

```text
Docker crashed.
```

Docker successfully created and ran the container. The process exited as instructed.

## 60. Preserve Before Cleanup

Capture:

```bash
docker ps -a --filter name=failing-api
docker inspect failing-api
docker logs --timestamps failing-api
docker diff failing-api
```

Only after the lab report is complete:

```bash
docker rm failing-api
```

---

## Big Picture

```text
Symptom: container not running
            │
            ▼
docker ps -a ──> summarized state and exit code
            │
            ▼
docker inspect ─> exact configuration + runtime state
            │
            ├── docker logs  ─> application output
            ├── docker top   ─> current processes
            ├── docker stats ─> current resources
            ├── docker events─> Docker timeline
            └── docker diff  ─> filesystem changes
            │
            ▼
Evidence-supported root cause
            │
            ▼
Smallest correction → verification
```

---

## Commands — Detailed Reference

## 61. `docker inspect`

```bash
docker inspect CONTAINER
docker inspect --format 'TEMPLATE' CONTAINER
docker inspect --size CONTAINER
```

Displays detailed configuration and state.

## 62. `docker logs`

```bash
docker logs CONTAINER
docker logs --tail 50 CONTAINER
docker logs -t CONTAINER
docker logs --since 10m CONTAINER
docker logs --until 5m CONTAINER
docker logs --tail 20 -f CONTAINER
```

Retrieves logs supported by the configured logging driver.

## 63. `docker top`

```bash
docker top CONTAINER
```

Shows current container processes.

## 64. `docker stats`

```bash
docker stats
docker stats CONTAINER
docker stats --no-stream CONTAINER
```

Shows live or one-time resource statistics for running containers.

## 65. `docker events`

```bash
docker events
docker events --since 10m
docker events --filter type=container
docker events --filter container=CONTAINER
```

Shows Docker daemon events.

## 66. `docker diff`

```bash
docker diff CONTAINER
```

Lists added, changed, and deleted container-layer paths.

---

## Guided Hands-on Lab — Evidence Before Change

### Scenario

TechCorp reports:

```text
The product API container starts and immediately disappears from docker ps.
```

Your goal is to diagnose without restarting or deleting it first.

### Phase A — Create Incident

```bash
docker run --name failing-api alpine:3.22 sh -c \
  'echo "INFO starting product API"; echo "ERROR missing DB_HOST" >&2; exit 42'
```

### Phase B — Summary

```bash
docker ps
docker ps -a --filter name=failing-api
```

Explain why the two lists differ.

### Phase C — State

```bash
docker inspect --format \
  'status={{.State.Status}} running={{.State.Running}} exit={{.State.ExitCode}} error={{json .State.Error}} oom={{.State.OOMKilled}} started={{.State.StartedAt}} finished={{.State.FinishedAt}}' \
  failing-api
```

### Phase D — Configuration

```bash
docker inspect --format \
  'image={{.Config.Image}} image_id={{.Image}} path={{.Path}} args={{json .Args}}' \
  failing-api
```

### Phase E — Logs

```bash
docker logs failing-api
docker logs --timestamps --tail 20 failing-api
```

### Phase F — Filesystem Difference

```bash
docker diff failing-api
```

Do not assume every change is related to the failure.

### Phase G — Report

Write:

```text
Symptom:
State:
Exit code:
Runtime error:
OOM evidence:
Final executable:
Arguments:
Relevant logs:
Root cause:
Recommended correction:
Verification plan:
```

### Phase H — Running-container Evidence

```bash
docker run -d --name inspect-web nginx:alpine
docker top inspect-web
docker stats --no-stream inspect-web
docker diff inspect-web
```

Explain what each command adds.

### Phase I — Event Timeline

Terminal 1:

```bash
docker events --filter container=inspect-web
```

Terminal 2:

```bash
docker restart inspect-web
docker stop inspect-web
```

Stop event observation with `Ctrl+C` and describe the order.

### Phase J — Cleanup

After saving evidence:

```bash
docker rm failing-api
docker rm inspect-web
```

`inspect-web` must already be stopped.

### Acceptance Criteria

You can prove:

- Why the failing container was absent from `docker ps`
- Its exact exit code
- Its final executable and arguments
- Whether Docker recorded a runtime or OOM error
- The relevant application error line
- The difference between application logs and Docker events
- Current Nginx processes and one-time resource usage
- Writable-layer changes and their limitations
- A root cause based on evidence, not assumption

---

## Discovery Questions

### Question 1

If `.State.Error` is empty but exit code is 42, did Docker necessarily fail to start the process?

<details>
<summary>Explanation</summary>

No. The process ran and deliberately exited with 42. An empty runtime error supports that this was application/command behavior rather than a recorded Docker start error.

</details>

### Question 2

Does exit code 137 alone prove an OOM kill?

<details>
<summary>Explanation</summary>

No. It often indicates termination by signal 9. Check `.State.OOMKilled`, events, limits, host logs, and surrounding evidence.

</details>

### Question 3

Why can `docker logs` be empty even when an application has a file under `/var/log`?

<details>
<summary>Explanation</summary>

Docker logs normally retrieves standard-stream output through the logging driver; it does not automatically search internal log files.

</details>

### Question 4

What is the difference between `docker events` and application logs?

<details>
<summary>Explanation</summary>

Events report Docker daemon actions; logs report application standard-stream output.

</details>

### Question 5

Does `C /etc/config` in `docker diff` tell you who changed it or its new contents?

<details>
<summary>Explanation</summary>

No. It only indicates that the path differs from the image layer.

</details>

---

## Production Scenario — Restart Loop

TechCorp reports that `product-api` is “running,” but `docker ps` repeatedly shows:

```text
Restarting (...)
```

Evidence workflow:

```bash
docker ps -a --filter name=product-api
docker inspect --format \
  'status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}} oom={{.State.OOMKilled}} restarts={{.RestartCount}} policy={{.HostConfig.RestartPolicy.Name}}' \
  product-api
docker logs --timestamps --tail 100 product-api
docker events --since 10m --filter container=product-api
```

Possible evidence pattern:

```text
Application logs: missing configuration file
Exit code: 1
Restart policy: always
Restart count: increasing
Events: repeated start and die
```

Root cause:

```text
The application exits because configuration is missing; the restart policy repeatedly starts the unchanged failing configuration.
```

Restarting again is not a fix. Correct the configuration through the deployment source, recreate as required, and verify stable behavior.

---

## Troubleshooting Workflow

## 67. Container Missing

```bash
docker ps -a --filter name=NAME
```

If absent, verify exact name/context/daemon. Do not assume it was removed by Docker automatically unless `--rm` or external automation provides evidence.

## 68. Container Exited

```bash
docker inspect --format \
  'exit={{.State.ExitCode}} error={{json .State.Error}} oom={{.State.OOMKilled}}' \
  CONTAINER
docker logs --timestamps --tail 100 CONTAINER
```

## 69. Container Running but Unresponsive

```bash
docker top CONTAINER
docker stats --no-stream CONTAINER
docker logs --timestamps --tail 100 CONTAINER
docker inspect CONTAINER
```

Later networking lessons add socket, port, and request-path evidence.

## 70. Unexpected Files

```bash
docker diff CONTAINER
```

Then inspect specific non-sensitive paths using a controlled exec if the container is running. Preserve evidence before changing files.

## 71. High Resource Usage

```bash
docker stats CONTAINER
docker top CONTAINER
docker inspect --format \
  'memory={{.HostConfig.Memory}} cpu_quota={{.HostConfig.CpuQuota}} cpu_period={{.HostConfig.CpuPeriod}}' \
  CONTAINER
```

Correlate with time-series monitoring and workload demand.

---

## Common Mistakes

### 1. Restarting before inspecting

You change the evidence before understanding it.

### 2. Removing a failed container immediately

You lose metadata, diff information, and convenient log access.

### 3. Reading only the exit code

Combine state error, OOM flag, logs, command, and events.

### 4. Calling every non-zero exit a Docker failure

The application may have returned it deliberately.

### 5. Treating 137 as automatic proof of OOM

Check the OOM flag and wider evidence.

### 6. Expecting logs to read internal files

Logs normally reads captured standard streams.

### 7. Running unbounded log follow on a noisy service

Use time and tail filters.

### 8. Publishing inspect output containing secrets

Redact environment, labels, mounts, and credentials carefully.

### 9. Treating one stats sample as a trend

Use proper monitoring for time-series conclusions.

### 10. Treating diff as a full audit trail

It shows path change types, not who, when, why, or contents.

---

## Best Practices

- Capture evidence before restart or removal
- Begin with the least expensive summary command
- Correlate state, command, logs, events, and resources
- Add timestamps to incident log captures
- Use narrow time windows and tail counts
- Protect secrets in inspection output
- Preserve failed containers until useful evidence is collected
- Write applications to structured standard-stream logs
- Centralize logs, metrics, and events in production
- Use explicit container names and deployment labels
- Record image IDs/digests in incident reports
- State the root cause separately from the symptom
- Verify the correction with the same evidence path

---

## Interview Questions

### Junior

1. What does `docker inspect` show?
2. What does `docker logs` show?
3. What is the difference between configuration and state?
4. How do you find a container's exit code?
5. How do you show the last 50 log lines?
6. What does `docker top` show?
7. What does `docker stats` show?
8. What does `docker diff` show?
9. What do `A`, `C`, and `D` mean in diff output?
10. Why should you inspect before removing a failed container?

### Intermediate

1. What is the difference between `.State.Error` and `.State.ExitCode`?
2. Why does 137 not prove OOM by itself?
3. How do Docker events differ from logs?
4. Why might logs be empty?
5. How would you diagnose a restart loop?
6. What are the limitations of `docker stats`?
7. What evidence may inspect expose accidentally?
8. Why is diff not a forensic audit log?
9. How do you correlate logs from multiple time zones safely?

### Senior Preview

Logging-driver internals, cgroup metrics, event retention, OpenTelemetry correlation, daemon auditability, and live forensics are deferred until production observability and internals lessons.

---

## Lesson Reflection

Answer in your own words:

1. What question does each diagnostic command answer?
2. Explain configuration versus runtime state.
3. Why should exit code and state error be read together?
4. How do you prove or reject an OOM hypothesis?
5. Where do Docker logs normally come from?
6. Why are timestamps important?
7. Explain events versus logs.
8. Explain stats versus time-series monitoring.
9. Explain the limitations of diff.
10. Write the evidence order you will use in a future incident.

---

## Summary

- Troubleshooting begins with evidence, not correction
- `docker ps -a` provides the first state summary
- `docker inspect` exposes configuration and detailed runtime state
- Exit code, runtime error, and OOM flag answer different questions
- Final `.Path` and `.Args` prove the executed command
- `docker logs` retrieves captured standard-stream output
- Tail and time filters control log volume
- `docker top` shows current processes
- `docker stats` shows immediate resource usage
- `docker events` shows Docker daemon activity
- `docker diff` lists writable-layer path changes
- No single command proves every root cause
- Failed-container evidence should be preserved until analysis is complete

---

## Cheat Sheet

### First response

```bash
docker ps -a --filter name=CONTAINER

docker inspect --format \
  'status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}} oom={{.State.OOMKilled}}' \
  CONTAINER

docker inspect --format \
  'path={{.Path}} args={{json .Args}}' \
  CONTAINER

docker logs --timestamps --tail 100 CONTAINER
```

### Running behavior

```bash
docker top CONTAINER
docker stats --no-stream CONTAINER
```

### Timeline and changes

```bash
docker events --since 10m --filter container=CONTAINER
docker diff CONTAINER
```

### Log filters

```bash
docker logs --tail 50 CONTAINER
docker logs --since 10m CONTAINER
docker logs --since TIME --until TIME CONTAINER
docker logs --tail 20 -f CONTAINER
```

### Diff symbols

```text
A → added
C → changed
D → deleted
```

---

## Challenge — Diagnose Without Restarting

### Incident Setup

Create:

```bash
docker run --name challenge-api alpine:3.22 sh -c \
  'echo "INFO loading config"; touch /tmp/started; echo "FATAL cannot read /config/app.yml" >&2; exit 17'
```

### Your Mission

Without restarting, executing inside, or deleting the container:

1. Prove it exists
2. Record status and exit code
3. Check runtime error and OOM flag
4. Record start and finish timestamps
5. Prove final executable and arguments
6. Record image reference and image ID
7. Retrieve timestamped logs
8. Inspect filesystem differences
9. State the root cause
10. Propose the smallest correction
11. Define verification criteria
12. Save evidence, then remove only the challenge container

### Rules

- No restart before report
- No `docker exec`
- No force removal
- Do not expose unrelated environment variables
- Separate symptom, evidence, and root cause

### Report Template

```text
Symptom:
Container state:
Exit code:
Runtime error:
OOM killed:
Start/finish times:
Image reference:
Image ID:
Final executable:
Final arguments:
Relevant logs:
Filesystem changes:
Root cause:
Correction:
Verification:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-08/
├── 01-container-summary.txt
├── 02-state-inspection.txt
├── 03-command-inspection.txt
├── 04-timestamped-logs.txt
├── 05-container-diff.txt
├── 06-running-processes.txt
├── 07-resource-snapshot.txt
├── 08-event-timeline.txt
├── 09-full-inspect-redacted.json
└── incident-report.md
```

Suggested captures:

```bash
docker inspect --format \
  'status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}} oom={{.State.OOMKilled}}' \
  failing-api > 02-state-inspection.txt

docker logs --timestamps failing-api > 04-timestamped-logs.txt 2>&1
docker diff failing-api > 05-container-diff.txt
```

Review and redact full inspection output before sharing it.

---

## Official References

- [Docker container inspect](https://docs.docker.com/reference/cli/docker/container/inspect/)
- [Docker container logs](https://docs.docker.com/reference/cli/docker/container/logs/)
- [Docker container top](https://docs.docker.com/reference/cli/docker/container/top/)
- [Docker container stats](https://docs.docker.com/reference/cli/docker/container/stats/)
- [Docker system events](https://docs.docker.com/reference/cli/docker/system/events/)
- [Docker container diff](https://docs.docker.com/reference/cli/docker/container/diff/)

---

## Next Lesson

**Lesson 09 — Port Publishing**

You can now prove that an application process is running. Next, you will make a containerized web service reachable from the Docker host.

You will learn:

- Container port versus host port
- Listening addresses
- Port publishing
- `localhost` on the host versus inside the container
- `EXPOSE` versus published ports
- Host socket verification
- Wrong-port diagnosis

Commands will include:

```bash
docker run -p 8080:80 nginx
docker port
ss -ltnp
curl
```

---

## Completion Check

Before moving forward, explain without reading:

```text
Configuration versus state
Exit code versus runtime error
Why 137 does not prove OOM alone
Where docker logs comes from
Tail, follow, since, and until
Top versus stats
Events versus application logs
A, C, and D in docker diff
Why failed containers should not be deleted immediately
Your evidence-first troubleshooting order
```

If you can diagnose the failing lab container without restarting it, you are ready for port publishing.

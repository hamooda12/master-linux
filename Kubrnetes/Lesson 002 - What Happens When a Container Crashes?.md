# Lesson 002 - What Happens When a Container Crashes?

## Kubernetes Zero-to-Hero Course

**Stage:** 1 - Why Kubernetes Exists  
**Level:** Complete beginner  
**Estimated study time:** 90-120 minutes  
**Lab type:** Guided Docker failure and recovery lab  
**Kubernetes commands used:** None

---

## 1. Overview

In Lesson 001, you manually maintained three web replicas. When one stopped, nothing automatically returned the application to the required count. You had to notice the difference, start the container, and verify the repair.

Now we will look more closely at failure itself.

People often say:

> The container crashed.

That sentence is useful in normal conversation, but what usually happened technically?

Every container has a main process. When that main process exits, the container is no longer running. The process may:

- Finish successfully
- Exit because the application detected an error
- Receive a graceful termination signal
- Be killed forcefully
- Fail because configuration is missing
- Be terminated after using too much memory
- Remain running while the application is actually unhealthy

These situations are not identical. A good operator does not respond to all of them by blindly restarting.

In this lesson, you will create several safe failures, inspect the evidence, and compare three recovery approaches:

1. Restart the same container manually.
2. Let Docker apply a restart policy.
3. Remove the failed container and create a replacement.

Then you will discover why none of those actions alone is the complete solution for a distributed production application.

---

## 2. Learning Objectives

By the end of this lesson, you should be able to:

1. Explain why a container stops when its main process exits.
2. Distinguish successful completion from failure.
3. Inspect container status, exit code, error text, timestamps, and restart count.
4. Explain the difference between graceful termination and forced termination.
5. Restart the same container and prove that its container ID remains the same.
6. Replace a container and prove that its ID changes.
7. Explain Docker restart policies and their limits.
8. Explain why a running process does not always mean a healthy application.
9. Describe why production recovery requires more than automatic process restart.

---

## 3. Prerequisites

Before beginning, you should understand:

- Docker images and containers
- Foreground and detached containers
- A container's main process
- `docker run`, `docker ps`, `docker stop`, `docker start`, and `docker rm`
- The Lesson 001 ideas of desired state, actual state, replicas, and manual reconciliation

This lesson uses only Docker. Kubernetes commands begin later in the roadmap.

---

## 4. New Vocabulary

| Term | Easy meaning |
|---|---|
| Main process | The process whose lifetime normally controls the container's lifetime |
| Exit | A process finishes and returns control to the operating system |
| Exit code | A number reported when a process finishes |
| Termination | The end of a running process or container |
| Graceful stop | The process receives a request and has time to clean up before exiting |
| Forced kill | The process is ended immediately without normal cleanup |
| Restart | Start the same stopped container again |
| Replacement | Create a different container to provide the required function |
| Restart policy | A rule telling Docker when to start a stopped container again |
| Crash loop | A repeated cycle of start, failure, and restart |
| Health | Whether the application is actually working correctly |
| Recovery | Returning the service to an acceptable working state |

---

## 5. The Core Rule

A container is not a tiny virtual machine that must keep existing forever.

At runtime, the container has a main process.

```text
Container starts
     |
     v
Main process runs
     |
     v
Main process exits
     |
     v
Container becomes stopped/exited
```

The container may contain child processes, but the lifetime of the container is normally tied to its main process, commonly seen as PID 1 inside the container.

### Example 1 - A Short Task

```bash
docker run alpine:3.22 echo "Backup finished"
```

The `echo` process prints one line and exits successfully. The container then stops.

This is not a crash. The work finished.

### Example 2 - A Web Server

```bash
docker run nginx:alpine
```

Nginx keeps its main process running because it must continue serving requests. The container remains running while that process remains alive.

### Important Idea

```text
Stopped container does not always mean failure.
Running container does not always mean healthy application.
```

We need evidence to understand what happened.

---

## 6. Four Different Endings

### 6.1 Successful Completion

A one-time task finishes its intended work and exits with code 0.

Examples:

- A database migration completes
- A report is generated
- A backup finishes
- A test suite passes

Restarting it automatically may repeat work unnecessarily.

### 6.2 Application Failure

The application detects a problem and exits with a non-zero code.

Examples:

- Required configuration is missing
- A configuration file is invalid
- Startup validation fails
- The application encounters an unrecoverable error

### 6.3 Requested Graceful Stop

An operator or management system requests termination. The process receives a signal such as `SIGTERM` and may:

- Stop accepting new work
- Finish current requests
- Flush buffers
- Close files
- Close network connections
- Exit cleanly

### 6.4 Forced Termination

A process is killed immediately, commonly with `SIGKILL`.

The process cannot catch, ignore, or clean up after `SIGKILL`.

This can interrupt requests or leave incomplete external work. It should not be treated as equivalent to a graceful stop.

---

## 7. Exit Codes

When a process ends, it reports an exit code.

The basic convention is:

| Exit code | General meaning |
|---:|---|
| `0` | Successful completion |
| Non-zero | An error or special condition occurred |

Applications decide the precise meaning of many non-zero codes. Read application documentation and logs instead of guessing.

### Common Signal-Related Shape

Shells and container tools often represent signal termination using:

```text
128 + signal number
```

Examples commonly seen:

| Common exit code | Possible interpretation |
|---:|---|
| `137` | Process received signal 9, often `SIGKILL` |
| `143` | Process received signal 15, often `SIGTERM` |

These are clues, not complete diagnoses.

For example, exit code 137 may appear after an out-of-memory termination, but exit code 137 alone does **not** prove that memory was the cause. An operator could also have sent `SIGKILL`.

Use several pieces of evidence:

- Exit code
- Container state
- Error field
- Logs
- Memory or system evidence
- Operator actions
- Recent configuration changes

---

## 8. State Is Not the Same as Health

Suppose `docker ps` says:

```text
STATUS: Up 20 minutes
```

That proves that the container's main process is still running.

It does not prove that:

- The application accepts connections
- The correct port is listening
- Database access works
- Requests return correct responses
- The application is ready for traffic
- The application is not stuck

Consider:

```text
Container process: running
HTTP endpoint:      timeout
Application health: bad
```

Restart policies react to process exit. They do not automatically understand every application-level failure.

This is why an operator checks both runtime state and service behavior.

---

## 9. Restart Versus Replacement

These words describe different actions.

### 9.1 Restart the Same Container

```bash
docker start CONTAINER
```

Docker starts the existing container object again.

Normally retained:

- Same container ID
- Same name
- Same creation-time settings
- Same writable container layer
- Same configured port mappings
- Same configured environment values

The main process starts again inside that same container object.

### 9.2 Create a Replacement

```text
Remove old container -> create a new container from an image
```

The replacement has:

- A new container ID
- A new writable layer
- Configuration supplied during the new creation
- A new runtime lifetime

It may use the same image, name, ports, and external volumes, but it is still a different container object.

### Why the Difference Matters

Suppose application configuration must change.

Restarting the old container does not rebuild its creation-time configuration from a new `docker run` command. Replacing it allows the operator to create a new container with the required configuration.

In production platforms, failed disposable workload instances are often replaced rather than lovingly repaired forever.

---

## 10. Docker Restart Policies

Docker can be told when to restart an individual container.

The policy is chosen when the container is created:

```bash
docker run --restart=POLICY IMAGE
```

It can also be changed later with `docker update`.

### Policy Table

| Policy | Behavior |
|---|---|
| `no` | Do not restart automatically. This is the default. |
| `on-failure[:max-retries]` | Restart after a non-zero exit, optionally with a retry limit. |
| `always` | Restart after the container stops, subject to Docker's manual-stop behavior. |
| `unless-stopped` | Similar to `always`, but preserve an intentionally stopped state across daemon restarts. |

### Important Details

- A restart policy becomes active after the container has started successfully. Docker currently defines successful startup for this purpose as remaining up for at least 10 seconds.
- A manual stop suppresses the policy until the container is manually started again or until behavior defined for the relevant policy and daemon restart applies.
- Restart policies manage containers on that Docker host.
- A policy may create a crash loop if the real cause is never corrected.

### Why Restart Policies Are Useful

They can recover from a simple process failure without waiting for a person.

### Why They Are Not the Whole Kubernetes Story

A single-container restart policy does not by itself provide:

- A desired replica count across several servers
- Placement on a different healthy server
- A stable discovery address for replacement instances
- Traffic removal based on application readiness
- Coordinated multi-container updates
- Resource-aware scheduling
- Complete application-wide desired state

Restart policies are useful. The wider orchestration problem still exists.

---

## 11. Practical Scenario

You are the junior DevOps engineer at TechCorp.

The product API stops during a promotion. Your senior engineer does not want the answer:

> I restarted it and now it works.

The senior asks:

1. Was the process still running?
2. What exit code was recorded?
3. Was the stop graceful or forced?
4. Did you restart the same container or create a replacement?
5. Did the application actually respond after recovery?
6. What will happen if the same failure occurs again?

The lesson lab trains you to answer those questions with evidence.

---

## 12. Hands-on Lab - Observe Container Endings

### Lab Safety

Every container in this lab uses a clearly prefixed name. Do not remove unrelated containers.

### Step 1 - Verify Docker

```console
hamad@Hamooda:~$ docker version
```

```console
hamad@Hamooda:~$ docker ps
```

If Docker reports a daemon connection or socket permission problem, solve that separately before continuing.

### Step 2 - Check for Existing Lab Objects

```console
hamad@Hamooda:~$ docker ps -a --filter name=k8s-course-
```

Inspect anything returned before removing it.

---

## 13. Lab A - Successful Completion

### Predict

What should happen when the command prints one message and returns exit code 0?

Run it:

```console
hamad@Hamooda:~$ docker run \
  --name k8s-course-exit-zero \
  alpine:3.22 \
  sh -c 'echo "task completed"; exit 0'
```

Expected visible output:

```text
task completed
```

The terminal prompt returns because the process finished.

### Inspect All Containers

```console
hamad@Hamooda:~$ docker ps -a \
  --filter name=k8s-course-exit-zero
```

The container should be stopped with an exit code of 0.

### Inspect Exact State

```console
hamad@Hamooda:~$ docker inspect \
  --format 'status={{.State.Status}} exit={{.State.ExitCode}} started={{.State.StartedAt}} finished={{.State.FinishedAt}}' \
  k8s-course-exit-zero
```

Interpretation:

```text
status=exited
exit=0
```

The stopped state is expected. This container completed its job successfully.

### Key Question

Should a system automatically restart every container that exits?

No. Restarting a successful one-time task may repeat completed work.

---

## 14. Lab B - Application-Reported Failure

This container deliberately reports exit code 7.

```console
hamad@Hamooda:~$ docker run \
  --name k8s-course-exit-seven \
  alpine:3.22 \
  sh -c 'echo "required configuration is missing" >&2; exit 7'
```

Inspect it:

```console
hamad@Hamooda:~$ docker inspect \
  --format 'status={{.State.Status}} exit={{.State.ExitCode}} error={{json .State.Error}}' \
  k8s-course-exit-seven
```

Then read its logs:

```console
hamad@Hamooda:~$ docker logs k8s-course-exit-seven
```

You should combine two pieces of evidence:

```text
Exit code: non-zero
Application message: required configuration is missing
```

Blindly restarting the same configuration will probably produce the same failure.

### Lesson

Automation should restore transient failures, but it must also expose evidence when recovery repeatedly fails.

---

## 15. Lab C - Graceful Stop and Same-Container Restart

### Step 1 - Start a Long-Running Web Container

```console
hamad@Hamooda:~$ docker run -d \
  --name k8s-course-web \
  -p 8090:80 \
  nginx:alpine
```

### Step 2 - Verify Runtime and Application State

```console
hamad@Hamooda:~$ docker ps --filter name=k8s-course-web
```

```console
hamad@Hamooda:~$ curl -I http://127.0.0.1:8090
```

### Step 3 - Record the Container ID

```console
hamad@Hamooda:~$ docker inspect \
  --format '{{.Id}}' \
  k8s-course-web
```

Copy the value into your lab notes as `original ID`.

### Step 4 - Request a Graceful Stop

```console
hamad@Hamooda:~$ docker stop k8s-course-web
```

`docker stop` requests graceful termination first. If the process does not exit within the allowed timeout, Docker can force termination afterward.

### Step 5 - Inspect the Result

```console
hamad@Hamooda:~$ docker inspect \
  --format 'status={{.State.Status}} exit={{.State.ExitCode}} oom={{.State.OOMKilled}} error={{json .State.Error}}' \
  k8s-course-web
```

Read the actual output. Do not invent the exit code before observing it.

### Step 6 - Start the Same Container

```console
hamad@Hamooda:~$ docker start k8s-course-web
```

Verify the ID:

```console
hamad@Hamooda:~$ docker inspect \
  --format '{{.Id}}' \
  k8s-course-web
```

Compare it with `original ID`.

It should be the same because `docker start` restarted the existing container object.

### Step 7 - Verify the Application

```console
hamad@Hamooda:~$ curl -I http://127.0.0.1:8090
```

The recovery is proven only when the service responds.

---

## 16. Lab D - Forced Termination

Ensure the web container is running:

```console
hamad@Hamooda:~$ docker ps --filter name=k8s-course-web
```

Forcefully terminate it:

```console
hamad@Hamooda:~$ docker kill \
  --signal=SIGKILL \
  k8s-course-web
```

Inspect the evidence:

```console
hamad@Hamooda:~$ docker inspect \
  --format 'status={{.State.Status}} exit={{.State.ExitCode}} oom={{.State.OOMKilled}} finished={{.State.FinishedAt}}' \
  k8s-course-web
```

You will commonly observe exit code 137 after `SIGKILL`.

Because you know that you intentionally sent `SIGKILL`, you have strong evidence for the reason. If you saw code 137 during an unknown incident, you would still investigate memory and system events rather than assuming.

### Verify User Impact

```console
hamad@Hamooda:~$ curl -I --max-time 3 http://127.0.0.1:8090
```

The request should fail because the only web instance is stopped.

### Restore Service

```console
hamad@Hamooda:~$ docker start k8s-course-web
```

```console
hamad@Hamooda:~$ curl -I http://127.0.0.1:8090
```

---

## 17. Lab E - Replace the Container

You will now replace the web container instead of restarting it.

### Step 1 - Record the Old ID Again

```console
hamad@Hamooda:~$ docker inspect \
  --format '{{.Id}}' \
  k8s-course-web
```

Save it as `old ID`.

### Step 2 - Remove the Old Container

```console
hamad@Hamooda:~$ docker rm -f k8s-course-web
```

### Step 3 - Create the Replacement

```console
hamad@Hamooda:~$ docker run -d \
  --name k8s-course-web \
  -p 8090:80 \
  nginx:alpine
```

The name and port are the same, but the object is new.

### Step 4 - Prove the Identity Changed

```console
hamad@Hamooda:~$ docker inspect \
  --format '{{.Id}}' \
  k8s-course-web
```

Compare it with `old ID`.

```text
Same name: yes
Same image reference: yes
Same port mapping: yes
Same container ID: no
```

### Step 5 - Prove Service Recovery

```console
hamad@Hamooda:~$ curl -I http://127.0.0.1:8090
```

This demonstrates an important production idea:

> Service identity can remain meaningful even when an individual runtime instance is replaced.

We will build this idea slowly in later networking lessons.

---

## 18. Lab F - Observe a Restart Policy

This container deliberately fails after remaining alive long enough for Docker's restart policy to become active.

### Start the Container

```console
hamad@Hamooda:~$ docker run -d \
  --name k8s-course-retry \
  --restart=on-failure:2 \
  alpine:3.22 \
  sh -c 'echo "attempt starting"; date; sleep 11; echo "failing now"; exit 7'
```

The meaning of `on-failure:2` is:

- Restart after a non-zero exit.
- Allow at most two automatic retry attempts after the initial run.

### Watch the Logs

```console
hamad@Hamooda:~$ docker logs -f k8s-course-retry
```

You should see the start and failure messages more than once. Press `Ctrl+C` after enough time has passed for the retries to complete. `Ctrl+C` stops following the logs; it does not remove the container.

### Inspect Final Evidence

```console
hamad@Hamooda:~$ docker inspect \
  --format 'status={{.State.Status}} exit={{.State.ExitCode}} restarts={{.RestartCount}} policy={{.HostConfig.RestartPolicy.Name}} max={{.HostConfig.RestartPolicy.MaximumRetryCount}}' \
  k8s-course-retry
```

Expected shape after retries finish:

```text
status=exited exit=7 restarts=2 policy=on-failure max=2
```

Read your actual output because timing may show the container as running or restarting if you inspect before the final attempt ends.

### The Important Question

Did the restart policy repair the missing configuration or bad command?

No.

It repeated the same failing work.

Automatic restart improves recovery from some temporary failures, but repeated restart is not the same as fixing the root cause.

---

## 19. What Happens When the Whole Server Fails?

Docker restart policies are enforced by the Docker daemon on the host where the container exists.

Suppose:

```text
server-a
|-- Docker daemon
`-- product-api container with restart policy
```

If only the API process exits, the daemon may restart it according to policy.

But if `server-a` loses power:

```text
server-a unavailable
|-- Docker daemon unavailable
`-- product-api unavailable
```

The restart policy on that unavailable server cannot move the workload to `server-b`.

A wider management system needs a view of several servers and must decide:

- Which server is healthy?
- Where is spare capacity?
- How many replicas are required?
- Where should a replacement run?
- How will traffic find it?

This is one major step from container restart toward orchestration.

---

## 20. Why Blind Restarting Can Be Harmful

Restarting is an action, not a diagnosis.

### Case 1 - Missing Configuration

Every restart uses the same bad configuration and fails again.

### Case 2 - Corrupted or Incompatible Data

Repeated startup attempts may create additional risk depending on the application.

### Case 3 - External Dependency Failure

The database is unavailable. Hundreds of replicas restart simultaneously and increase pressure when the database begins recovering.

### Case 4 - Completed Batch Work

A successful job exits with code 0. An `always` policy repeats it even though it was meant to run once.

### Case 5 - Application Is Unhealthy but Process Lives

The restart policy sees no process exit and does nothing.

Good automation needs:

- Correct policy for the workload type
- Backoff between repeated attempts
- Health information
- Logs and events
- Alerting
- A limit or human escalation when automatic recovery fails

---

## 21. Human Recovery Loop

During this lesson, you performed the following loop:

```text
1. Detect a symptom
2. Inspect container state
3. Read exit and error evidence
4. Read application logs
5. Decide whether restart is appropriate
6. Restart or replace
7. Test the actual service
8. Watch for recurrence
```

This is better than:

```text
Problem -> restart -> hope
```

Kubernetes will later automate parts of recovery, but a Kubernetes engineer still needs this reasoning. Automation can restart a failed process. It cannot magically correct every application bug, wrong password, invalid configuration, or broken dependency.

---

## 22. Independent Challenge - TechCorp API Failure

### Scenario

The TechCorp API container should remain available on host port 8092.

### Part A - Create and Verify

Create a container with these requirements:

- Name: `k8s-course-api`
- Image: `nginx:alpine`
- Detached mode
- Host port: 8092
- Container port: 80
- No automatic restart policy

Prove both process state and HTTP response.

### Part B - Cause and Diagnose Failure

Forcefully terminate the container with `SIGKILL`.

Collect:

- Status
- Exit code
- OOMKilled value
- Finished timestamp
- Application connectivity result

Do not restore it before recording evidence.

### Part C - Restart the Same Container

Start it again and prove:

- The ID is unchanged
- The HTTP endpoint works

### Part D - Replace It

Remove it and create a replacement with the same name, image, and port mapping.

Prove:

- The ID changed
- The endpoint works again

### Part E - Explain

Answer aloud:

1. Why did the container stop?
2. What evidence shows forced termination?
3. What is the difference between restart and replacement?
4. Why would a restart policy not solve a complete server failure?
5. Why is HTTP verification required after the container starts?

### Acceptance Criteria

The challenge passes when all five answers are correct and every claim has command evidence.

---

## 23. Common Mistakes

### Mistake 1 - Calling Every Exit a Crash

Exit code 0 may represent successful completion.

### Mistake 2 - Treating a Non-Zero Code as a Full Diagnosis

It indicates failure or a special condition, but logs and application documentation explain the specific meaning.

### Mistake 3 - Assuming Exit Code 137 Always Means OOM

It commonly reflects `SIGKILL`. Memory termination is one possible cause, not the only cause.

### Mistake 4 - Assuming `docker start` Creates a New Container

It starts the existing container. Its ID stays the same.

### Mistake 5 - Assuming the Same Name Means the Same Container

A replacement may reuse a name but have a new ID.

### Mistake 6 - Thinking Restart Policy Fixes the Root Cause

It repeats the start attempt. A permanent configuration or code error may repeat forever or until the retry limit is reached.

### Mistake 7 - Checking Only `docker ps`

The process may be running while the application fails to serve requests.

### Mistake 8 - Force-Killing Before Trying Graceful Stop

Forced termination prevents normal cleanup. Use it only when necessary and understand the impact.

### Mistake 9 - Restarting Before Collecting Evidence

Immediate recovery may be necessary during a severe outage, but capture safe evidence first when possible.

---

## 24. Best Practices

1. Treat exit code as evidence, not the entire explanation.
2. Read previous or current logs before they are lost.
3. Prefer graceful termination when possible.
4. Match restart behavior to workload purpose.
5. Add retry limits or backoff for repeated failure where appropriate.
6. Verify application behavior after runtime recovery.
7. Keep important data outside disposable container writable layers.
8. Use replacement when creation-time configuration must change.
9. Design applications to tolerate unexpected restarts.
10. Document the desired replica count and recovery expectations.
11. Escalate repeated failure instead of hiding it with endless restarts.
12. Investigate the root cause after service restoration.

---

## 25. Interview Questions

### Question 1

**Why does a container stop when its main process exits?**

The container runtime treats the main process as the process controlling the container's running lifetime. When it terminates, the container enters a stopped or exited state.

### Question 2

**Does a stopped container always mean failure?**

No. A finite task may complete successfully and exit with code 0.

### Question 3

**What is the basic meaning of exit code 0 and non-zero?**

Zero generally means success. Non-zero generally signals an error or special condition whose exact meaning depends on the program.

### Question 4

**What is the difference between `docker start` and creating a replacement?**

`docker start` starts the same container object with the same ID. A replacement is a newly created object with a new ID and new writable layer.

### Question 5

**What does `on-failure` do?**

It asks Docker to restart a container after a non-zero exit, optionally up to a configured retry limit.

### Question 6

**Why is a restart policy not a complete high-availability solution?**

It does not by itself manage desired replicas and placement across multiple servers, stable discovery, traffic readiness, or coordinated application state.

### Question 7

**Can a container be running while the application is unhealthy?**

Yes. The main process may remain alive while requests time out, dependencies fail, or the application becomes stuck.

### Question 8

**Why should applications support graceful shutdown?**

It lets them stop accepting work, finish active requests, flush data, and close resources before exiting.

---

## 26. Lesson Reflection

Answer without looking back:

1. What exactly usually happens when someone says a container crashed?
2. Why is an exited container not automatically a failed container?
3. What does exit code 137 suggest, and why is it not a complete diagnosis?
4. What evidence would you collect before restarting a failed container?
5. What survives when you restart the same container?
6. What changes when you replace it?
7. Why may an automatic restart create a crash loop?
8. Why can a local Docker restart policy not recover a workload onto another server?
9. What is the difference between process state and application health?

### Mastery Statement

You are ready for Lesson 003 when you can explain this naturally:

> A container stops when its main process exits, but recovery depends on why it exited. Restarting the same container may restore a temporary failure, while replacement creates a new instance. Neither action alone provides complete application availability across multiple servers.

---

## 27. Summary

In this lesson, you learned:

- The lifetime of a running container is normally tied to its main process.
- A process can complete, fail, stop gracefully, or be killed forcefully.
- Exit code 0 usually means success; non-zero codes require investigation.
- Signal-related exit codes are clues, not complete diagnoses.
- Running state and application health are different.
- `docker start` starts the same container object.
- Recreating produces a replacement with a new identity.
- Docker restart policies can automate some local recovery.
- A retry loop does not repair a permanent root cause.
- Host failure requires a system that can place replacement work elsewhere.
- Recovery is not finished until the real application is verified.

The central idea is:

```text
Failure detected
      |
      v
Collect evidence
      |
      v
Understand the ending
      |
      v
Restart or replace
      |
      v
Verify service
      |
      v
Watch for recurrence and fix root cause
```

---

## 28. Cheat Sheet Update

Add this section to `CHEAT_SHEET.md`:

```markdown
## Container Failure and Recovery

- A container normally stops when its main process exits.
- Exit 0: successful completion by convention.
- Non-zero exit: failure or program-specific condition.
- 137 commonly indicates SIGKILL; it does not prove OOM by itself.
- Running process does not guarantee a healthy application.
- `docker start NAME`: restart the same container object and ID.
- Remove plus `docker run`: create a replacement with a new ID.
- Docker restart policies: no, on-failure, always, unless-stopped.
- Automatic restart can repeat a permanent failure.
- Verify recovery at both container and application levels.
```

Useful commands:

```bash
docker ps -a --filter name=NAME
docker logs NAME
docker inspect --format 'status={{.State.Status}} exit={{.State.ExitCode}}' NAME
docker stop NAME
docker kill --signal=SIGKILL NAME
docker start NAME
docker update --restart=unless-stopped NAME
```

---

## 29. Cleanup

Inspect the lab containers first:

```console
hamad@Hamooda:~$ docker ps -a --filter name=k8s-course-
```

Remove only the named lesson containers:

```console
hamad@Hamooda:~$ docker rm -f \
  k8s-course-exit-zero \
  k8s-course-exit-seven \
  k8s-course-web \
  k8s-course-retry \
  k8s-course-api
```

If you did not create the challenge container, you may omit `k8s-course-api`.

Verify cleanup:

```console
hamad@Hamooda:~$ docker ps -a --filter name=k8s-course-
```

---

## 30. Lab Assets

Suggested course repository paths:

```text
kubernetes-practice/
|-- lessons/
|   `-- lesson-002-what-happens-when-a-container-crashes.md
|-- assets/
|   `-- lesson-002/
|       |-- process-container-lifecycle.png
|       |-- successful-exit-evidence.png
|       |-- forced-exit-evidence.png
|       |-- restart-same-id.png
|       |-- replacement-new-id.png
|       `-- restart-policy-attempts.png
|-- labs/
|   `-- lesson-002/
|       |-- evidence.md
|       `-- challenge-notes.md
`-- CHEAT_SHEET.md
```

Capture these screenshots:

1. Exit code 0 after successful completion.
2. Exit code 7 and the application error log.
3. Forced-termination state evidence.
4. Same ID before and after `docker start`.
5. Different ID after replacement.
6. Restart count after the policy lab.

---

## 31. Official References

- [Docker - Start containers automatically](https://docs.docker.com/engine/containers/start-containers-automatically/)
- [Kubernetes - Pod lifecycle and container restarts](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

The Kubernetes reference shows where the course is going. Do not study its YAML or kubectl examples yet. Later lessons will introduce them in order.

---

## 32. Next Lesson

### Lesson 003 - What Happens When Traffic Grows?

In the next lesson, you will:

- Measure the difference between one instance and several instances
- Understand capacity and bottlenecks
- Scale a service manually
- Discover why replicas need traffic distribution
- See why adding containers does not guarantee linear performance
- Identify what an automated scaling system must observe and decide

We will continue learning the problem before introducing the Kubernetes solution.

# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 13 — tmpfs and Storage Decisions

> **Level:** Beginner to intermediate  
> **Type:** Storage decision lesson  
> **Lab image:** `alpine:3.22`  
> **Estimated study time:** 100–140 minutes

---

## Overview

You now know three storage behaviors:

```text
Container writable layer → survives restart, lost on removal
Named volume             → survives container replacement
Bind mount               → maps a chosen host path
```

There is another useful choice:

```text
tmpfs → temporary memory-backed storage
```

A tmpfs mount is designed for data that should not persist.

Examples:

- Temporary processing files
- Short-lived caches
- Runtime scratch space
- Sensitive temporary material that should not normally be written to persistent storage

In this lesson, you will write a file into tmpfs, stop and restart the container, and prove the file is gone.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain what tmpfs is at a beginner level
- Mount tmpfs into a container
- Prove tmpfs data disappears after stop/start
- Explain why tmpfs differs from the container writable layer
- Explain why tmpfs differs from a named volume
- Explain why tmpfs differs from a bind mount
- Inspect tmpfs mount configuration
- Create a read-only tmpfs mount
- Limit tmpfs size
- Set basic tmpfs permissions
- Explain the memory and swap security caveat
- Select storage based on persistence, ownership, sharing, security, and performance requirements
- Build a storage decision table for real applications

---

## Prerequisites

You should understand:

- Container writable layer
- Named and anonymous volumes
- Bind mounts
- Mount source and target
- Read-only mounts
- Container stop, restart, remove, and recreate
- Linux file permissions and UID/GID
- RAM, disk, and swap at a basic level

Use your established Docker access model.

---

## Why This Topic Exists

Not every file should survive.

Suppose an application processes uploaded documents:

```text
Receive document
      ↓
Create temporary conversion files
      ↓
Upload final result to permanent storage
      ↓
Delete temporary files
```

The conversion files are temporary. Writing them to persistent storage may:

- Leave unnecessary files behind
- Increase disk usage
- Require cleanup jobs
- Expose sensitive temporary content
- Add disk I/O

tmpfs provides a temporary filesystem backed by host memory.

---

# Part 1 — What tmpfs Is

## 1. Beginner Definition

> A tmpfs mount is temporary storage presented as a filesystem inside the container and backed primarily by the Docker host's memory.

```text
Container path /scratch
          │ tmpfs mount
          ▼
Host memory-backed temporary filesystem
```

## 2. tmpfs Is Outside the Writable Layer

When `/scratch` is tmpfs-mounted:

```text
/scratch/file.tmp → tmpfs
/tmp/other.txt    → container writable layer
```

The two paths have different lifecycles.

## 3. tmpfs Is Not Persistent

tmpfs data is lost when its mount is removed, including when the container stops or restarts.

```text
Container running → tmpfs exists and contains data
Container stops   → tmpfs mount removed; data lost
Container starts  → new empty tmpfs mount
```

## 4. Container Removal Is Not Required for Data Loss

This differs from the writable layer:

```text
Writable layer after stop/start → data remains
tmpfs after stop/start           → data is gone
```

## 5. tmpfs Does Not Create a Reusable Volume

You will not find tmpfs as a named item under:

```bash
docker volume ls
```

It belongs to the container's mount configuration and active runtime, not the named-volume list.

---

# Part 2 — Create a tmpfs Mount

## 6. Preferred `--mount` Syntax

```bash
docker run -d \
  --name tmpfs-lab \
  --mount type=tmpfs,target=/scratch \
  alpine:3.22 \
  sleep 3600
```

Breakdown:

| Field | Meaning |
| --- | --- |
| `type=tmpfs` | Create a tmpfs mount |
| `target=/scratch` | Path inside container |

There is no reusable source name because tmpfs is not a named persistent volume.

## 7. Verify the Mount

```bash
docker inspect --format '{{json .Mounts}}' tmpfs-lab
```

Typical information:

```text
Type: tmpfs
Source: empty
Destination: /scratch
RW: true
```

## 8. Verify Inside the Container

```bash
docker exec tmpfs-lab mount
```

Look for a tmpfs entry mounted at `/scratch`.

Minimal image output can vary; Docker inspection is the primary evidence.

## 9. Write a File

```bash
docker exec tmpfs-lab sh -c \
  'echo "temporary secret" > /scratch/temporary.txt'
```

## 10. Read It

```bash
docker exec tmpfs-lab cat /scratch/temporary.txt
```

Expected:

```text
temporary secret
```

---

# Part 3 — Compare tmpfs With Writable Layer

## 11. Create One File in Each Location

```bash
docker exec tmpfs-lab sh -c \
  'echo "memory data" > /scratch/memory.txt; echo "layer data" > /layer.txt'
```

Verify:

```bash
docker exec tmpfs-lab cat /scratch/memory.txt
docker exec tmpfs-lab cat /layer.txt
```

## 12. Check `docker diff`

```bash
docker diff tmpfs-lab
```

You should see `/layer.txt` as a writable-layer change.

The file under `/scratch` belongs to the tmpfs mount and is not an ordinary writable-layer change.

## 13. Stop the Container

```bash
docker stop tmpfs-lab
```

## 14. Start the Same Container

```bash
docker start tmpfs-lab
```

The container ID remains the same, but Docker creates a new empty tmpfs mount for this run.

## 15. Check the Writable-layer File

```bash
docker exec tmpfs-lab cat /layer.txt
```

Expected:

```text
layer data
```

## 16. Check the tmpfs File

```bash
docker exec tmpfs-lab sh -c \
  'test -f /scratch/memory.txt && cat /scratch/memory.txt || echo "memory.txt is gone"'
```

Expected:

```text
memory.txt is gone
```

## 17. The Exact Difference

```text
Same container after stop/start
├── writable layer remains → /layer.txt exists
└── tmpfs recreated empty  → /scratch/memory.txt gone
```

---

# Part 4 — `--tmpfs` Short Syntax

## 18. Alternative Syntax

Docker also supports:

```bash
docker run -d \
  --name tmpfs-short \
  --tmpfs /scratch \
  alpine:3.22 sleep 3600
```

## 19. Why Prefer `--mount`

```bash
--mount type=tmpfs,target=/scratch
```

is explicit and consistent with volume and bind-mount lessons.

`--tmpfs` is shorter and supports a colon-separated option string:

```bash
--tmpfs /scratch:rw,nosuid,nodev,size=64m
```

You should recognize both.

## 20. Swarm Note

The `--tmpfs` flag is not used for Swarm services; the explicit mount form is used there. Swarm is taught later.

---

# Part 5 — Limit tmpfs Size

## 21. Why Set a Limit?

tmpfs consumes host memory resources. An application that writes without control can create memory pressure.

Set a size limit based on workload requirements.

## 22. Size With `--mount`

```bash
docker run -d \
  --name tmpfs-limited \
  --mount type=tmpfs,target=/scratch,tmpfs-size=67108864 \
  alpine:3.22 sleep 3600
```

`67108864` bytes equals 64 MiB.

## 23. Size With `--tmpfs`

```bash
docker run -d \
  --name tmpfs-limited-short \
  --tmpfs /scratch:size=64m \
  alpine:3.22 sleep 3600
```

## 24. Verify Inside

```bash
docker exec tmpfs-limited df -h /scratch
```

The reported size should reflect the configured limit, subject to platform and formatting behavior.

## 25. Size Limit Is Not the Same as Container Memory Limit

```text
tmpfs-size → maximum size of this tmpfs mount
--memory   → broader container memory constraint
```

Resource-management details and cgroups come later.

---

# Part 6 — Permissions and Mode

## 26. Default Mode

tmpfs mounts commonly default to mode `1777`, similar to a shared temporary directory.

Verify actual result:

```bash
docker exec tmpfs-lab ls -ld /scratch
```

## 27. Set Mode

```bash
docker run -d \
  --name tmpfs-private \
  --mount type=tmpfs,target=/scratch,tmpfs-mode=0700 \
  alpine:3.22 sleep 3600
```

Verify:

```bash
docker exec tmpfs-private ls -ld /scratch
```

## 28. UID and GID Options

With `--tmpfs`, Linux mount options can include:

```bash
--tmpfs /scratch:uid=1000,gid=1000,mode=0700
```

Use the numeric identity expected by the application.

Do not choose ownership blindly. Inspect the container process user first.

## 29. Security Options

Useful options can include:

```text
nosuid → do not honor setuid/setgid execution behavior
nodev  → device files are not functional
noexec → do not execute binaries from the mount
```

Example:

```bash
--tmpfs /scratch:rw,nosuid,nodev,noexec,size=64m,mode=0700
```

Select options based on application needs. `noexec` may break software that legitimately executes files from the scratch directory.

---

# Part 7 — Read-only tmpfs

## 30. Create Read-only tmpfs

```bash
docker run -d \
  --name tmpfs-readonly \
  --tmpfs /scratch:ro \
  alpine:3.22 sleep 3600
```

## 31. Attempt Write

```bash
docker exec tmpfs-readonly sh -c \
  'echo blocked > /scratch/file.txt'
```

Expected: read-only filesystem error.

## 32. Why Would Empty Read-only tmpfs Be Useful?

It is not a common application-data pattern by itself. It can be used to cover a path with an empty read-only filesystem or support specialized hardening designs.

For ordinary static configuration, a read-only volume or bind mount is usually clearer.

---

# Part 8 — Memory and Swap Caveat

## 33. “Memory-backed” Needs Precision

tmpfs uses host virtual memory. Depending on host configuration, memory pages may be written to swap.

Therefore:

> tmpfs does not guarantee that bytes can never reach persistent storage.

## 34. Sensitive Data Guidance

tmpfs can reduce ordinary persistent-disk writes for temporary sensitive data, but security also requires:

- Host swap policy or encrypted swap
- Correct mount permissions
- Short data lifetime
- Application cleanup
- No logging of secrets
- No core dumps exposing data
- Host security
- Least-privileged container access

## 35. tmpfs Is Not a Secret Manager

It only provides temporary storage.

It does not provide:

- Secret creation
- Access policy distribution
- Rotation
- Audit logging
- Encryption-at-rest guarantees
- Central revocation

Secret fundamentals come in Lesson 15.

---

# Part 9 — Performance and Capacity

## 36. Why tmpfs Can Be Fast

tmpfs avoids ordinary persistent-disk I/O for its data and can be useful for fast temporary operations.

## 37. The Cost

The data consumes host memory resources and can contribute to memory pressure.

If many containers allocate large tmpfs mounts, the host may become unstable or begin swapping heavily.

## 38. A Size Limit Is Capacity, Not Reservation

Setting:

```text
tmpfs-size=64MiB
```

defines a maximum filesystem size. It does not necessarily reserve all 64 MiB immediately.

## 39. Monitor the Workload

Use:

```bash
docker stats CONTAINER
```

and host monitoring to understand real memory behavior.

One `docker stats` sample is not a complete capacity plan.

## 40. Do Not Use tmpfs for Large Persistent Datasets

tmpfs is wrong for:

- Database primary storage
- User uploads that must survive
- Backups
- Large artifacts needed after restart
- Data exceeding safe host memory capacity

---

# Part 10 — Storage Decision Framework

## 41. Ask About Persistence

```text
Must data survive container restart?
Must it survive container removal?
Must it survive host failure?
```

## 42. Ask About Ownership

```text
Should Docker manage the location?
Must host tools access the exact path?
Must several containers share it?
```

## 43. Ask About Security

```text
Does container need write access?
Is data sensitive?
Should it ever reach disk?
Which UID/GID requires access?
```

## 44. Ask About Performance

```text
Is workload write-heavy?
Is data temporary?
How large can it grow?
What happens under memory pressure?
```

## 45. Ask About Recovery

```text
Is it backed up?
Has restore been tested?
What are RPO and RTO?
Can data be regenerated?
```

---

# Part 11 — Comparison Table

## 46. Storage Types

| Requirement | Writable layer | Named volume | Bind mount | tmpfs |
| --- | --- | --- | --- | --- |
| Survives stop/start | Yes | Yes | Yes | No |
| Survives container removal | No | Yes | Yes | No |
| Docker manages location | Yes, internally | Yes | No | Runtime memory |
| Direct host-path access | No normal workflow | Not recommended directly | Yes | No |
| Good for persistent app data | No | Yes | Sometimes | No |
| Good for development source | No | Sometimes | Yes | No |
| Good for temporary scratch | Yes | Usually unnecessary | Sometimes | Yes |
| Avoid ordinary disk writes | No guarantee | No | No | Mostly, with swap caveat |
| Requires backup for important data | Not suitable | Yes | Yes | Not applicable to temporary data |

## 47. Quick Decision Rules

### Writable layer

Use for replaceable runtime files that may disappear with the container.

### Named volume

Use for persistent application data managed through Docker.

### Bind mount

Use when a specific host path must appear inside the container.

### tmpfs

Use for temporary, bounded, memory-backed files that should disappear when the container stops.

---

# Part 12 — Application Examples

## 48. Database

```text
Primary database files → named volume or managed storage
Temporary query files  → application-dependent; possibly tmpfs
Database backups       → separate backup storage
```

## 49. Web Application

```text
Application code → image
Uploaded files   → volume/object storage
Configuration    → read-only bind/config mechanism
Temporary cache  → writable layer or tmpfs depending requirements
```

## 50. CI Build Job

```text
Source checkout    → bind mount or workspace volume
Temporary compiler → writable layer/tmpfs
Final artifact     → persistent artifact store
Secrets            → secret mechanism, possibly exposed as temporary file
```

## 51. Nginx

```text
Default binary/config → image
Custom config         → read-only bind mount or custom image
TLS private key       → secret mechanism/read-only controlled mount
Cache                 → writable layer, volume, or tmpfs based on purpose
Access logs           → standard streams/central logging
```

---

## Big Picture

```text
Does data need to survive container removal?
        │
        ├── Yes ──> named volume or bind mount
        │              │
        │              ├── Docker-managed? → volume
        │              └── specific host path? → bind
        │
        └── No
             │
             ├── Should survive stop/start? → writable layer
             └── Should disappear on stop?  → tmpfs
```

Important data also needs a separate backup and restore plan.

---

## Commands — Detailed Reference

## 52. Create tmpfs With `--mount`

```bash
docker run --mount type=tmpfs,target=/scratch IMAGE
```

## 53. Set Size and Mode

```bash
docker run --mount \
  type=tmpfs,target=/scratch,tmpfs-size=67108864,tmpfs-mode=0700 \
  IMAGE
```

## 54. Use `--tmpfs`

```bash
docker run --tmpfs /scratch:size=64m,mode=0700 IMAGE
```

## 55. Inspect

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
docker exec CONTAINER mount
docker exec CONTAINER df -h /scratch
```

## 56. Compare Layer Changes

```bash
docker diff CONTAINER
```

Mounted tmpfs content does not appear as an ordinary writable-layer change.

---

## Guided Hands-on Lab — Prove tmpfs Is Temporary

### Scenario

TechCorp needs temporary document-processing files that disappear when a worker stops, while a small state marker should survive a restart of the same container.

### Phase A — Create Container

```bash
docker run -d \
  --name tmpfs-worker \
  --mount type=tmpfs,target=/scratch,tmpfs-size=67108864,tmpfs-mode=0700 \
  alpine:3.22 sleep 3600
```

### Phase B — Inspect Mount

```bash
docker inspect --format '{{json .Mounts}}' tmpfs-worker
docker exec tmpfs-worker df -h /scratch
docker exec tmpfs-worker ls -ld /scratch
```

### Phase C — Write Two Files

```bash
docker exec tmpfs-worker sh -c \
  'echo "temporary conversion" > /scratch/job.tmp; echo "layer marker" > /marker.txt'

docker exec tmpfs-worker cat /scratch/job.tmp
docker exec tmpfs-worker cat /marker.txt
```

### Phase D — Inspect Layer

```bash
docker diff tmpfs-worker
```

Explain why `/marker.txt` appears but `/scratch/job.tmp` is not an ordinary layer change.

### Phase E — Stop and Start

```bash
docker stop tmpfs-worker
docker start tmpfs-worker
```

### Phase F — Verify Different Lifecycles

```bash
docker exec tmpfs-worker cat /marker.txt

docker exec tmpfs-worker sh -c \
  'test -f /scratch/job.tmp && cat /scratch/job.tmp || echo "job.tmp is gone"'
```

### Phase G — Write Again and Restart

```bash
docker exec tmpfs-worker sh -c \
  'echo "second temporary job" > /scratch/job2.tmp'

docker restart tmpfs-worker

docker exec tmpfs-worker sh -c \
  'test -f /scratch/job2.tmp && cat /scratch/job2.tmp || echo "job2.tmp is gone"'

docker exec tmpfs-worker cat /marker.txt
```

### Phase H — Read-only Test

```bash
docker run -d \
  --name tmpfs-ro \
  --tmpfs /scratch:ro,size=16m \
  alpine:3.22 sleep 3600

docker exec tmpfs-ro sh -c 'echo blocked > /scratch/file'
```

### Phase I — Compare Persistent Volume

```bash
docker volume create tmpfs-comparison

docker run --rm \
  --mount type=volume,source=tmpfs-comparison,target=/data \
  alpine:3.22 sh -c 'echo persistent > /data/file.txt'

docker run --rm \
  --mount type=volume,source=tmpfs-comparison,target=/data \
  alpine:3.22 cat /data/file.txt
```

Explain why the volume file remains but tmpfs files do not.

### Phase J — Cleanup

```bash
docker stop tmpfs-worker tmpfs-ro
docker rm tmpfs-worker tmpfs-ro
docker volume rm tmpfs-comparison
```

### Acceptance Criteria

You can prove:

- `/scratch` is tmpfs
- Size and mode settings are visible
- File is readable while container runs
- Writable-layer marker appears in diff
- tmpfs file disappears after stop/start
- Writable-layer marker remains after stop/start
- tmpfs file disappears after restart
- Read-only tmpfs rejects writes
- Named volume survives separate container runs
- tmpfs does not appear in `docker volume ls`

---

## Discovery Questions

### Question 1

Why does `/marker.txt` survive while `/scratch/job.tmp` disappears?

<details>
<summary>Explanation</summary>

The marker belongs to the container writable layer, which remains while the container exists. `/scratch` is a tmpfs mount recreated empty after stop/start.

</details>

### Question 2

Must the container be removed for tmpfs data to disappear?

<details>
<summary>Explanation</summary>

No. Stopping the container removes the active tmpfs mount and its data. Starting creates a new empty tmpfs.

</details>

### Question 3

Why is tmpfs not shown by `docker volume ls`?

<details>
<summary>Explanation</summary>

It is a temporary mount configured for a container, not a persistent Docker volume object.

</details>

### Question 4

Does tmpfs guarantee secret bytes never reach disk?

<details>
<summary>Explanation</summary>

No. Host virtual-memory pages may be swapped to disk depending on system configuration. tmpfs is not a complete secret-management solution.

</details>

### Question 5

Why set a tmpfs size limit?

<details>
<summary>Explanation</summary>

It bounds how large the temporary filesystem can grow and reduces the risk of uncontrolled host memory consumption.

</details>

---

## Production Scenario — Unbounded Temporary Files

A document service mounts:

```bash
--mount type=tmpfs,target=/work
```

but sets no size limit and never deletes failed job files while running. Under load, many containers consume large amounts of memory and the host becomes unstable.

Evidence:

```bash
docker stats
docker inspect --format '{{json .Mounts}}' CONTAINER
docker exec CONTAINER df -h /work
docker logs --tail 100 CONTAINER
```

Root cause:

```text
Temporary-storage growth was not bounded or cleaned, causing memory pressure.
```

Correction:

- Set a justified tmpfs size
- Enforce application file-size and concurrency limits
- Clean completed/failed jobs
- Apply container memory limits
- Monitor memory and work-queue growth
- Reject work safely under pressure

tmpfs improves temporary-storage behavior but does not remove capacity planning.

---

## Troubleshooting Workflow

## 57. File Disappeared After Restart

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
```

If the target is tmpfs, disappearance is expected.

## 58. No Space Left on Device

```bash
docker exec CONTAINER df -h TMPFS_PATH
docker inspect --format '{{json .Mounts}}' CONTAINER
```

Check configured tmpfs size and application cleanup behavior. Do not increase size blindly.

## 59. Permission Denied

```bash
docker exec CONTAINER id
docker exec CONTAINER ls -ldn TMPFS_PATH
docker inspect --format '{{json .Mounts}}' CONTAINER
```

Check mode, UID, GID, read-only status, and security policies.

## 60. Host Memory Pressure

```bash
docker stats
free -h
docker exec CONTAINER df -h TMPFS_PATH
```

Correlate with host monitoring, workload demand, swap activity, and configured limits.

## 61. Data Needed After Stop

tmpfs is the wrong storage type. Use a named volume, bind mount, external object store, or another persistent service based on requirements.

---

## Common Mistakes

### 1. Thinking tmpfs survives restart

It is recreated empty.

### 2. Thinking removal is required for loss

Stopping is enough to lose tmpfs contents.

### 3. Using tmpfs for database primary data

It is intentionally non-persistent.

### 4. Leaving tmpfs unbounded

Large writes can contribute to memory pressure.

### 5. Thinking tmpfs is a named volume

It is not managed through `docker volume ls`.

### 6. Calling tmpfs a secret manager

It lacks distribution, policy, rotation, and audit features.

### 7. Ignoring swap

Memory pages may reach swap storage.

### 8. Setting permissions without checking application UID

The process may be unable to use the mount.

### 9. Assuming tmpfs is always faster enough to justify it

Measure the real workload and account for memory costs.

### 10. Confusing tmpfs loss with Docker corruption

Loss after stop is expected behavior.

---

## Best Practices

- Use tmpfs only for intentionally temporary data
- Set a justified size limit
- Use restrictive mode, UID, and GID settings
- Add `nosuid`, `nodev`, or `noexec` when compatible
- Monitor host and container memory
- Clean temporary files during long-running container sessions
- Keep final artifacts in persistent storage
- Use named volumes for durable application data
- Use bind mounts for deliberate host-path access
- Treat sensitive-data protection as a complete system design
- Document the expected lifecycle of every writable path
- Test behavior across stop, restart, removal, and host reboot where relevant

---

## Interview Questions

### Junior

1. What is tmpfs?
2. Where does tmpfs store data?
3. Does tmpfs survive container restart?
4. Does tmpfs appear in `docker volume ls`?
5. How do you create a tmpfs mount?
6. Why set a size limit?
7. What is the difference between tmpfs and a volume?
8. What is the difference between tmpfs and the writable layer?
9. When would you use tmpfs?
10. Can tmpfs replace a secret manager?

### Intermediate

1. Why can tmpfs data reach swap?
2. How do tmpfs mode, UID, and GID affect access?
3. What do `nosuid`, `nodev`, and `noexec` provide?
4. How would you investigate tmpfs memory pressure?
5. Why does a size limit not reserve all memory immediately?
6. How would you choose between writable layer, volume, bind mount, and tmpfs?
7. What acceptance tests prove a storage design?

### Senior Preview

Linux tmpfs internals, page accounting, cgroup interaction, swap encryption, memory pressure, Kubernetes `emptyDir`, and memory-backed ephemeral volumes are deferred.

---

## Lesson Reflection

Answer in your own words:

1. Draw tmpfs relative to the container writable layer.
2. Why does tmpfs disappear after stop?
3. Why does the writable-layer marker remain?
4. Explain tmpfs versus named volume.
5. Explain tmpfs versus bind mount.
6. What sensitive-data risk remains because of swap?
7. Why is a size limit required?
8. What permission options might you choose?
9. Select storage for a database, source code, and temporary conversion files.
10. Write the five questions you will ask before selecting storage.

---

## Summary

- tmpfs is temporary memory-backed filesystem storage
- It is mounted at a container path
- It exists outside the container writable layer
- tmpfs data disappears when the container stops or restarts
- The same container's writable layer survives stop/start
- tmpfs is not a Docker volume object
- `--mount type=tmpfs` is the explicit syntax
- `--tmpfs` is a shorter alternative
- Size and permission options should be configured deliberately
- tmpfs can reduce ordinary disk writes but may use swap
- tmpfs is not a secret manager
- Storage selection depends on persistence, ownership, security, capacity, and recovery requirements

---

## Cheat Sheet

### Basic tmpfs

```bash
docker run -d \
  --name tmpfs-lab \
  --mount type=tmpfs,target=/scratch \
  alpine:3.22 sleep 3600
```

### Size and mode

```bash
--mount type=tmpfs,target=/scratch,tmpfs-size=67108864,tmpfs-mode=0700
```

### Short syntax

```bash
--tmpfs /scratch:size=64m,mode=0700,nosuid,nodev,noexec
```

### Inspect

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
docker exec CONTAINER df -h /scratch
docker exec CONTAINER ls -ld /scratch
docker diff CONTAINER
```

### Lifecycle

```text
Writable layer → survives stop/start, lost on removal
Named volume   → survives replacement
Bind mount     → host path controls data
tmpfs          → lost on stop/restart
```

---

## Challenge — Choose the Correct Storage

### Scenario

TechCorp has four data paths:

```text
/app/code      → application code
/data          → customer database
/config        → host-managed configuration
/scratch       → temporary image-conversion files
```

### Mission

1. Choose the correct storage source for each path
2. Explain the required lifecycle
3. Create a named volume for `/data`
4. Create a read-only bind mount for `/config`
5. Create a 32 MiB private tmpfs for `/scratch`
6. Keep `/app/code` in the image
7. Write test data to all writable paths
8. Stop/start the container
9. Record which data remains
10. Remove and recreate the container with the same volume/mounts
11. Record which data remains
12. Explain backup requirements

### Rules

- No confidential data
- No `chmod 777`
- No unlimited tmpfs
- No volume prune
- Configuration bind mount must be read-only

### Expected Design

```text
/app/code → image
/data     → named volume
/config   → read-only bind mount
/scratch  → tmpfs
```

### Report Template

```text
Path:
Storage type:
Why:
Survives stop/start:
Survives removal/recreation:
Backup required:
Permission/security settings:
Verification result:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-13/
├── 01-tmpfs-inspect.txt
├── 02-tmpfs-size-mode.txt
├── 03-before-stop.txt
├── 04-after-stop-start.txt
├── 05-after-restart.txt
├── 06-diff-comparison.txt
├── 07-readonly-test.txt
├── 08-volume-comparison.txt
├── 09-storage-decision-table.md
└── tmpfs-incident-report.md
```

Do not store real secrets in tmpfs lab files.

---

## Official References

- [Docker tmpfs mounts](https://docs.docker.com/engine/storage/tmpfs/)
- [Docker storage overview](https://docs.docker.com/engine/storage/)
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)

---

## Next Lesson

**Lesson 14 — Environment Variables and Runtime Configuration**

You can now select storage based on data lifetime. Next, you will separate configuration from the image so the same image can run in development, testing, and production.

You will learn:

```bash
docker run -e
docker run --env-file
docker exec CONTAINER printenv
docker inspect
```

And understand:

- Configuration outside the image
- Image defaults and runtime overrides
- Environment files
- Configuration files through mounts
- Why a configuration change normally requires container recreation
- Why ordinary environment variables are not a complete secret solution

---

## Completion Check

Before moving forward, explain without reading:

```text
What tmpfs is
Why data disappears after stop
tmpfs versus writable layer
tmpfs versus volume
tmpfs versus bind mount
Size, mode, UID, and GID
Memory and swap caveat
Why tmpfs is not a secret manager
Storage decision questions
Storage choice for common application paths
```

If you can prove tmpfs loss after stop/start and explain why that behavior is useful, you are ready for runtime configuration.

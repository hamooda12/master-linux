# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 11 — Container Writable Layer and Data Loss

> **Level:** Beginner  
> **Type:** Discovery-based storage lesson  
> **Lab image:** `alpine:3.22`  
> **Estimated study time:** 100–140 minutes

---

## Overview

What happens when you create a file inside a container?

Three answers are commonly confused:

```text
Stop and start the same container → file remains
Restart the same container        → file remains
Remove it and create a new one    → file disappears
```

Why?

Every container receives its own writable layer above the read-only image layers.

```text
Container writable layer ← files changed by this container
────────────────────────
Read-only image layers    ← packaged image content
```

The writable layer belongs to that specific container. When the container is removed, its writable layer is removed with it.

This lesson lets you discover that behavior before learning volumes.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain read-only image layers at a beginner level
- Explain the container writable layer
- Explain why each container has different writable state
- Create and modify files inside a container
- Use `docker diff` to identify added, changed, and deleted paths
- Prove that data survives stop/start of the same container
- Prove that data survives restart of the same container
- Prove that data disappears after container removal and recreation
- Distinguish image data from container data
- Distinguish persistent requirements from disposable runtime state
- Explain why databases should not store important data only in the writable layer
- Explain the immutable-container mindset
- Identify when a volume or bind mount will be needed

---

## Prerequisites

You should understand:

- Image versus container
- Image layers at an introductory level
- Container ID and identity
- `docker run`, `start`, `stop`, `restart`, and `rm`
- `docker exec`
- `docker inspect`
- `docker diff`
- Stopped versus removed

Use your established Docker access model.

---

## Why This Topic Exists

Imagine running MySQL in a container.

MySQL writes company data inside the container. The container later fails, and an engineer removes it and creates a replacement.

The replacement starts from the image, but the old database files belonged to the removed container's writable layer.

```text
Old MySQL container removed
          ↓
Old writable layer removed
          ↓
New MySQL container created
          ↓
New empty writable layer
```

Without external persistent storage, the business data is lost.

That is why you must understand container storage before running databases or other stateful applications.

---

# Part 1 — Image Files Versus Container Changes

## 1. Image as the Starting Filesystem

An image packages a filesystem and configuration.

For Alpine, the image contains files such as:

```text
/bin/sh
/etc/os-release
/etc/passwd
```

When Docker creates a container, these image files become its starting filesystem view.

## 2. Image Layers Are Read-only

Docker does not normally rewrite the source image when a container changes a file.

```text
alpine:3.22 image
       ├── container A changes /tmp/a.txt
       └── container B changes /tmp/b.txt

Image remains unchanged
```

## 3. Container Writable Layer

Docker adds a thin writable layer for each new container.

```text
Container A writable layer
──────────────────────────
Alpine read-only image
```

All ordinary filesystem writes that are not sent to a mount are recorded in this container-specific layer.

Examples:

- Creating a file
- Editing a configuration file
- Deleting an image-provided file
- Application cache files
- Temporary files
- Package installation performed inside a running container

## 4. Every Container Gets Its Own Layer

```text
Container A writable layer
──────────────────────────
Shared Alpine image layers

Container B writable layer
──────────────────────────
Shared Alpine image layers
```

Container A's changes do not automatically appear in Container B.

## 5. Image Is Not Modified

If Container A creates:

```text
/tmp/secret-a.txt
```

then:

- The Alpine image does not gain that file
- A new container from Alpine does not inherit it
- Container B does not see it unless data is deliberately shared

---

# Part 2 — Read Behavior

## 6. Reading an Unchanged Image File

When a container reads:

```text
/etc/os-release
```

and it has not modified that path, Docker's storage system provides the image version.

Beginner model:

```text
Look in container changes
        ↓ not changed
Read from image layer
```

## 7. Reading a Newly Created File

If the container creates:

```text
/tmp/lesson11.txt
```

the file exists only in its writable layer.

```text
Look in container writable layer
        ↓ found
Read container's file
```

## 8. Modifying an Image File

Suppose the container modifies `/etc/motd`.

The image layer remains read-only. Docker's storage driver presents the container's changed version from its writable layer.

```text
Container version of /etc/motd ← visible
Image version of /etc/motd     ← still unchanged underneath
```

This is an introductory model. Storage-driver copy-on-write mechanics come later.

## 9. Deleting an Image File

The container can make an image-provided path appear deleted in its own view without changing the source image.

A new container still sees the original image file.

---

# Part 3 — First Experiment: Create Data

## 10. Start a Long-running Lab Container

```bash
docker run -d \
  --name data-lab \
  alpine:3.22 \
  sleep 3600
```

The `sleep` process keeps the container running for the lab.

## 11. Record Identity

```bash
docker inspect --format '{{.Id}}' data-lab
```

Save the full container ID.

## 12. Create a File

```bash
docker exec data-lab sh -c \
  'echo "important container data" > /data.txt'
```

## 13. Read the File

```bash
docker exec data-lab cat /data.txt
```

Expected:

```text
important container data
```

## 14. Change an Existing File

Use a safe lab path under `/tmp`:

```bash
docker exec data-lab sh -c \
  'mkdir -p /tmp/lesson11 && echo original > /tmp/lesson11/status.txt'

docker exec data-lab sh -c \
  'echo updated >> /tmp/lesson11/status.txt'
```

## 15. Delete a File

```bash
docker exec data-lab sh -c \
  'touch /tmp/delete-me && rm /tmp/delete-me'
```

A file created and deleted within the same writable layer may not leave a useful final `docker diff` entry because it no longer exists. To demonstrate `D`, remove an image-provided disposable file only after verifying it exists:

```bash
docker exec data-lab sh -c \
  'test -f /etc/motd && rm /etc/motd || true'
```

Image contents vary, so a `D /etc/motd` entry is not guaranteed.

---

# Part 4 — Inspect Changes

## 16. Use `docker diff`

```bash
docker diff data-lab
```

Symbols:

| Symbol | Meaning |
| --- | --- |
| `A` | Added path |
| `C` | Changed path |
| `D` | Deleted path |

You should find entries related to:

```text
/data.txt
/tmp/lesson11
```

Other runtime changes may also appear.

## 17. Diff Shows Paths, Not Contents

If output says:

```text
A /data.txt
```

it proves the path was added relative to the image.

It does not show:

- File contents
- Exact time of change
- Which process changed it
- Whether the data is important

## 18. Inspect Writable-layer Size

```bash
docker ps -a --size --filter name=data-lab
```

The `SIZE` column shows the container writable-layer size, plus a virtual-size view that includes image data.

The exact size may be larger than your visible text files because of filesystem metadata and other runtime changes.

## 19. Docker Disk Usage

```bash
docker system df -v
```

This provides a broader Docker disk-usage view.

Do not delete data based only on size. First identify ownership and persistence requirements.

---

# Part 5 — Stop and Start the Same Container

## 20. Stop

```bash
docker stop data-lab
```

The main process exits, so the container becomes stopped.

The container object still exists.

## 21. Verify It Still Exists

```bash
docker ps -a --filter name=data-lab
```

## 22. Start the Same Container

```bash
docker start data-lab
```

## 23. Verify the File

```bash
docker exec data-lab cat /data.txt
```

The file remains.

## 24. Prove Identity

```bash
docker inspect --format '{{.Id}}' data-lab
```

Compare with the original ID. It is the same container, so it has the same writable layer.

## 25. Core Rule

```text
stop → process stops
start → same container starts
writable layer remains
```

Stopping is not deletion.

---

# Part 6 — Restart the Same Container

## 26. Restart

```bash
docker restart data-lab
```

Restart stops and starts the same container.

## 27. Verify Again

```bash
docker exec data-lab cat /data.txt
docker inspect --format '{{.Id}}' data-lab
```

The file and ID remain.

## 28. Restart Is Not Recreation

```text
Restart
→ same container ID
→ same writable layer

Recreate
→ new container ID
→ new writable layer
```

This is why “restart Docker” and “replace the container” have different storage consequences.

---

# Part 7 — Remove the Container

## 29. Capture Evidence First

Before destructive removal:

```bash
docker inspect --format '{{.Id}}' data-lab
docker exec data-lab cat /data.txt
docker diff data-lab
docker ps -a --size --filter name=data-lab
```

## 30. Stop and Remove

```bash
docker stop data-lab
docker rm data-lab
```

## 31. Verify Removal

```bash
docker ps -a --filter name=data-lab
```

The old container no longer exists.

Its container-specific writable layer has been removed.

## 32. Image Still Exists

```bash
docker image ls alpine
```

Removing the container did not remove the Alpine image.

---

# Part 8 — Create a New Container From the Same Image

## 33. Recreate the Name

```bash
docker run -d \
  --name data-lab \
  alpine:3.22 \
  sleep 3600
```

The name is the same, but the container is new.

## 34. Compare ID

```bash
docker inspect --format '{{.Id}}' data-lab
```

It differs from the original container ID.

## 35. Look for the Old File

```bash
docker exec data-lab sh -c \
  'if [ -f /data.txt ]; then cat /data.txt; else echo "data.txt is missing"; fi'
```

Expected:

```text
data.txt is missing
```

## 36. Why the File Is Gone

```text
Original container
└── original writable layer contained /data.txt

docker rm
└── original writable layer removed

New container
└── new writable layer starts from Alpine image
```

The image never contained `/data.txt`, so the new container cannot inherit it.

## 37. Same Name Does Not Mean Same Container

Docker names are reusable after removal.

```text
Old data-lab name + ID A
removed
New data-lab name + ID B
```

Identity is proven by the container ID, not by the reused name alone.

---

# Part 9 — Independent Writable Layers

## 38. Create Two Containers

```bash
docker run -d --name data-a alpine:3.22 sleep 3600
docker run -d --name data-b alpine:3.22 sleep 3600
```

## 39. Write Only in A

```bash
docker exec data-a sh -c \
  'echo "belongs to A" > /owner.txt'
```

## 40. Read in A

```bash
docker exec data-a cat /owner.txt
```

Expected:

```text
belongs to A
```

## 41. Try in B

```bash
docker exec data-b sh -c \
  'test -f /owner.txt && cat /owner.txt || echo "owner.txt is missing in B"'
```

Expected:

```text
owner.txt is missing in B
```

## 42. Meaning

Both containers share read-only Alpine image layers, but each owns a different writable layer.

```text
data-a writable layer → /owner.txt
data-b writable layer → no /owner.txt
```

---

# Part 10 — What Belongs in the Writable Layer?

## 43. Appropriate Temporary Data

The writable layer is useful for disposable runtime state such as:

- Temporary files
- Replaceable caches
- Process IDs
- Generated files that do not need to survive replacement
- Short-lived job output that is exported elsewhere

## 44. Inappropriate Important Data

Do not rely on it alone for:

- Database files
- User uploads
- Business documents
- Persistent application configuration
- Certificates that must survive replacement
- Audit records
- Unique build artifacts
- Backups

## 45. Ask the Persistence Question

For every writable path, ask:

```text
If this container is removed right now,
is it acceptable for this data to disappear?
```

If the answer is no, design external persistent storage.

## 46. Logs Need a Strategy Too

`docker logs` can access container standard-stream output through a logging driver, but local log availability and lifecycle are not a complete retention strategy.

Production logs should normally be collected centrally and retained according to operational requirements.

---

# Part 11 — Immutable-container Mindset

## 47. Do Not Repair Containers Manually as the Deployment Model

Suppose an engineer runs:

```bash
docker exec -it product-api sh
```

and manually edits a configuration file or installs a package.

The current container may work, but the change exists only in its writable layer.

When the container is replaced, the change disappears.

## 48. Reproducible Correction

Intended application changes should be expressed in:

- Source code
- Dockerfile
- Build dependencies
- Versioned configuration
- Environment/configuration management
- Volumes or secrets when appropriate

Then:

```text
Build new image/configuration
        ↓
Test it
        ↓
Replace container
        ↓
Verify behavior
```

## 49. Immutable Does Not Mean Literally No Writes

Applications can still create temporary runtime data.

The mindset means:

> Do not depend on unique manual changes inside one container for your reproducible deployment.

## 50. Replace, Do Not Snowflake

A “snowflake container” is a manually changed instance that cannot be reproduced from the image and deployment definition.

Avoid:

```text
Only this container has the fix
```

Prefer:

```text
Every replacement created from the same versioned definition has the fix
```

---

# Part 12 — Mounts Preview

## 51. How Persistent Storage Solves the Problem

Docker can mount storage at a path inside the container.

```text
Container path /data
        │ mount
        ▼
Storage outside container writable layer
```

When the container is removed, the external storage can remain.

## 52. Main Storage Choices

| Type | Beginner purpose |
| --- | --- |
| Named volume | Docker-managed persistent data |
| Bind mount | Map a specific host path into container |
| tmpfs | Temporary memory-backed data; not persistent |

Lesson 12 teaches volumes and bind mounts fully.

## 53. Data Looks Like a Normal Path Inside

From the application's view, mounted data still appears at a directory or file path.

The key difference is where the data is stored and who controls its lifecycle.

---

## Big Picture

```text
Image: alpine:3.22
        │
        ├── create Container A
        │      └── Writable layer A → /data.txt
        │
        └── create Container B
               └── Writable layer B → no /data.txt

Stop/start A  → layer A remains
Restart A     → layer A remains
Remove A      → layer A removed
New A         → new empty writable layer
```

---

## Commands — Detailed Reference

## 54. Create Data

```bash
docker exec CONTAINER sh -c \
  'echo "data" > /data.txt'
```

## 55. Read Data

```bash
docker exec CONTAINER cat /data.txt
```

## 56. Inspect Differences

```bash
docker diff CONTAINER
```

## 57. Inspect Size

```bash
docker ps -a --size --filter name=CONTAINER
docker system df -v
```

## 58. Preserve Same Container

```bash
docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
```

These preserve its identity and writable layer.

## 59. Destroy Writable Layer

```bash
docker stop CONTAINER
docker rm CONTAINER
```

Removing the container removes its writable layer.

---

## Guided Hands-on Lab — Watch the Data Disappear

### Scenario

TechCorp stores a report inside a container. You must determine which lifecycle actions preserve it and which destroy it.

### Phase A — Create Container

```bash
docker run -d \
  --name persistence-test \
  alpine:3.22 \
  sleep 3600

docker inspect --format '{{.Id}}' persistence-test
```

Save the ID as `ORIGINAL_ID` in your notes—not necessarily as a shell variable.

### Phase B — Create Important File

```bash
docker exec persistence-test sh -c \
  'echo "TechCorp quarterly report" > /report.txt'

docker exec persistence-test cat /report.txt
docker diff persistence-test
```

### Phase C — Stop and Start

```bash
docker stop persistence-test
docker start persistence-test
docker inspect --format '{{.Id}}' persistence-test
docker exec persistence-test cat /report.txt
```

Prove the ID and file remain.

### Phase D — Restart

```bash
docker restart persistence-test
docker inspect --format '{{.Id}}' persistence-test
docker exec persistence-test cat /report.txt
```

### Phase E — Capture Before Destruction

```bash
docker inspect --format '{{.Id}}' persistence-test
docker exec persistence-test cat /report.txt
docker diff persistence-test
docker ps -a --size --filter name=persistence-test
```

### Phase F — Remove

```bash
docker stop persistence-test
docker rm persistence-test
```

### Phase G — Recreate Same Name

```bash
docker run -d \
  --name persistence-test \
  alpine:3.22 \
  sleep 3600

docker inspect --format '{{.Id}}' persistence-test
```

Compare the ID with the original.

### Phase H — Search for Report

```bash
docker exec persistence-test sh -c \
  'test -f /report.txt && cat /report.txt || echo "report.txt is gone"'
```

### Phase I — Independent Containers

```bash
docker run -d --name storage-a alpine:3.22 sleep 3600
docker run -d --name storage-b alpine:3.22 sleep 3600
docker exec storage-a sh -c 'echo A > /only-a.txt'
docker exec storage-a cat /only-a.txt
docker exec storage-b sh -c \
  'test -f /only-a.txt && cat /only-a.txt || echo "not in B"'
```

### Phase J — Cleanup

```bash
docker stop persistence-test storage-a storage-b
docker rm persistence-test storage-a storage-b
```

### Acceptance Criteria

You can prove:

- New file belonged to writable layer
- `docker diff` detected it
- Stop/start preserved file and container ID
- Restart preserved file and ID
- Remove destroyed the old container
- Reusing the name created a different ID
- New container did not contain old report
- Two containers from same image had independent writable data
- Alpine image remained unchanged

---

## Discovery Questions

### Question 1

Why does a file survive `docker stop`?

<details>
<summary>Explanation</summary>

Stopping ends the main process but does not remove the container object or its writable layer.

</details>

### Question 2

Why does the file disappear after `docker rm` and a new `docker run`?

<details>
<summary>Explanation</summary>

Removal deletes the old container's writable layer. The new container receives a fresh layer based on the unchanged image.

</details>

### Question 3

Does restarting a container rebuild it from the image?

<details>
<summary>Explanation</summary>

No. Restart stops and starts the same container with the same ID and writable layer.

</details>

### Question 4

Why can Container B not see a file created in Container A?

<details>
<summary>Explanation</summary>

They have separate writable layers. Sharing the image does not share container-specific writes.

</details>

### Question 5

If manual package installation fixes one running container, why is deployment still broken?

<details>
<summary>Explanation</summary>

The fix exists only in that container's writable layer and cannot be reproduced automatically after replacement.

</details>

---

## Production Scenario — Lost Database

TechCorp runs:

```bash
docker run -d --name orders-db mysql:TAG
```

No volume is mounted for MySQL's data directory. The database writes all records into the container writable layer.

During an incident, an engineer runs:

```bash
docker rm -f orders-db
docker run -d --name orders-db mysql:TAG
```

The replacement database starts empty.

Root cause:

```text
Important database data was stored only in the removed container's writable layer.
```

Prevention:

- Mount persistent storage at the database data directory
- Back it up
- Test restoration
- Document ownership and permissions
- Avoid destructive removal before confirming data design
- Use a managed database when appropriate

The image is not a backup of runtime database records.

---

## Troubleshooting Workflow

## 60. “My File Disappeared”

Check container identity:

```bash
docker ps -a --filter name=NAME
docker inspect --format '{{.Id}} created={{.Created}}' NAME
```

Ask:

- Was the original container removed?
- Was a new container created with the same name?
- Was the file in a mounted path?
- Was `--rm` used?
- Did an automated deployment replace the container?

## 61. “The File Survived Restart”

That is expected for the same container writable layer.

Do not use restart survival as proof of durable persistence.

Test the real requirement:

```text
Does data survive container replacement?
```

## 62. “Two Containers Show Different Files”

Inspect mounts and writable changes:

```bash
docker inspect --format '{{json .Mounts}}' A
docker inspect --format '{{json .Mounts}}' B
docker diff A
docker diff B
```

If no shared mount exists, independent writable layers are expected.

## 63. “Container Writable Layer Is Growing”

```bash
docker ps -a --size
docker system df -v
docker diff CONTAINER
```

Find which application paths generate data. Move persistent/write-heavy data to suitable storage and handle logs/caches intentionally.

---

## Common Mistakes

### 1. Thinking stop deletes data

Stop preserves the container and writable layer.

### 2. Thinking restart creates a clean container

Restart uses the same container.

### 3. Thinking same name means same container

Names can be reused; compare container IDs.

### 4. Thinking the image contains runtime writes

The source image remains unchanged.

### 5. Storing database data only inside the container

Removal deletes the writable layer.

### 6. Treating restart survival as persistence proof

Durability must be tested across replacement.

### 7. Manually repairing a container

The fix becomes a non-reproducible snowflake.

### 8. Using `docker commit` as normal deployment

Express changes in a Dockerfile and source control.

### 9. Removing before checking mounts

Understand which data is external and which belongs to the layer.

### 10. Assuming diff is a backup

It lists path changes, not recoverable content.

---

## Best Practices

- Assume containers are replaceable
- Keep application code and dependencies in the image
- Keep important runtime data outside the writable layer
- Use volumes for Docker-managed persistent data
- Use bind mounts when a deliberate host path must be shared
- Use tmpfs only for intentionally temporary data
- Back up and restore-test persistent storage
- Avoid manual production-container modifications
- Monitor writable-layer and log growth
- Verify persistence across container replacement
- Record container IDs during incidents
- Inspect storage design before destructive cleanup

---

## Interview Questions

### Junior

1. What is the container writable layer?
2. Are image layers writable?
3. Does a file survive `docker stop`?
4. Does it survive `docker restart`?
5. What happens after `docker rm`?
6. Do two containers share writable changes?
7. Does writing in a container modify the image?
8. What does `docker diff` show?
9. Why should database data use persistent storage?
10. What is an immutable-container mindset?

### Intermediate

1. Why does restart survival not prove durable persistence?
2. How do you prove a container was recreated?
3. What data belongs in the writable layer?
4. Why can write-heavy workloads perform better with volumes?
5. What are the limitations of `docker diff`?
6. Why are manual exec-based fixes unreliable?
7. How would you investigate writable-layer growth?
8. What should a persistence acceptance test include?

### Senior Preview

Copy-on-write, overlay filesystem lower/upper directories, whiteouts, storage drivers, snapshotters, inode behavior, and storage performance tuning are deferred until the internals phase.

---

## Lesson Reflection

Answer in your own words:

1. Draw image layers and a container writable layer.
2. Why do two containers see different runtime files?
3. Explain stop/start versus remove/recreate.
4. Why is container ID stronger identity evidence than name?
5. What happens to the image when a container changes a file?
6. What data is safe to keep only in the writable layer?
7. What data requires persistent storage?
8. Why is manual repair not reproducible?
9. What does “replaceable container” mean?
10. How will you test storage before running a database?

---

## Summary

- Image layers are read-only
- Every container receives its own writable layer
- Ordinary unmounted writes go to that layer
- Containers sharing an image do not share writable changes
- The source image remains unchanged
- Stop/start preserves the same container and layer
- Restart preserves the same container and layer
- Remove deletes the container writable layer
- A new container gets a fresh writable layer
- Reusing a name does not reuse identity or data
- Important data must live outside the container layer
- Manual changes inside one container are not reproducible deployment
- Volumes and bind mounts solve different external-storage needs

---

## Cheat Sheet

### Create and write

```bash
docker run -d --name data-lab alpine:3.22 sleep 3600
docker exec data-lab sh -c 'echo data > /data.txt'
docker exec data-lab cat /data.txt
```

### Inspect changes

```bash
docker diff data-lab
docker ps -a --size --filter name=data-lab
docker system df -v
```

### Same container: data remains

```bash
docker stop data-lab
docker start data-lab
docker restart data-lab
docker exec data-lab cat /data.txt
```

### Replacement: writable data disappears

```bash
docker stop data-lab
docker rm data-lab
docker run -d --name data-lab alpine:3.22 sleep 3600
```

### Critical rules

```text
Stop       → keep container layer
Start      → reuse container layer
Restart    → reuse container layer
Remove     → delete container layer
New run    → new container layer
Same image → shared read-only layers
Same image → not shared writable layer
```

---

## Challenge — Prove Persistence Correctly

### Scenario

A junior says:

> “The file survived a restart, so it is persistent and safe for production.”

Your task is to prove why that conclusion is incomplete.

### Required Workflow

1. Create `persistence-challenge` from Alpine
2. Record its full ID
3. Create `/important.txt`
4. Use diff to prove the addition
5. Stop/start and verify file/ID
6. Restart and verify file/ID
7. Capture evidence
8. Remove the container
9. Recreate the same name from the same image
10. Record the new ID
11. Prove `/important.txt` is absent
12. Explain why the image was unchanged
13. Clean up

### Rules

- No volume or bind mount yet
- No `docker commit`
- No force removal before evidence
- Do not call restart a recreation
- Distinguish name and ID

### Report Template

```text
Original ID:
File before stop:
File after start:
ID after start:
File after restart:
ID after restart:
New ID after recreation:
File after recreation:
What was deleted:
What remained:
Why restart was not a persistence test:
Required storage solution:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-11/
├── 01-original-container-id.txt
├── 02-created-file.txt
├── 03-container-diff.txt
├── 04-after-stop-start.txt
├── 05-after-restart.txt
├── 06-writable-layer-size.txt
├── 07-removal-proof.txt
├── 08-new-container-id.txt
├── 09-missing-file-proof.txt
├── 10-independent-layers.txt
└── data-loss-report.md
```

Suggested captures:

```bash
docker inspect --format '{{.Id}}' persistence-test \
  > 01-original-container-id.txt
docker diff persistence-test > 03-container-diff.txt
docker ps -a --size --filter name=persistence-test \
  > 06-writable-layer-size.txt
```

Do not use real confidential data in container storage experiments.

---

## Official References

- [Docker storage overview](https://docs.docker.com/engine/storage/)
- [Docker storage drivers and writable layers](https://docs.docker.com/engine/storage/drivers/)
- [Persisting container data](https://docs.docker.com/get-started/docker-concepts/running-containers/persisting-container-data/)
- [Docker container diff](https://docs.docker.com/reference/cli/docker/container/diff/)

---

## Next Lesson

**Lesson 12 — Docker Volumes and Bind Mounts**

You have now experienced data loss after container removal. Next, you will solve it.

You will learn:

- Named and anonymous volumes
- Bind mounts
- `--mount` and `-v`
- Read-only mounts
- Volume lifecycle
- Host paths
- Permissions and ownership
- Backup introduction

The main test will be:

```text
Create data → remove container → attach same storage to new container → data remains
```

---

## Completion Check

Before moving forward, explain without reading:

```text
Read-only image layer
Container writable layer
Why containers have independent changes
Stop/start versus remove/recreate
Why restart preserves data
Why removal loses data
Why same name does not mean same identity
Temporary versus important data
Immutable-container mindset
Why a database needs external storage
```

If you can make the report disappear deliberately and explain exactly why, you are ready to preserve data with mounts.

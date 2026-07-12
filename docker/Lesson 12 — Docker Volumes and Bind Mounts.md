# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 12 — Docker Volumes and Bind Mounts

> **Level:** Beginner to intermediate  
> **Type:** Persistent-storage lesson  
> **Lab image:** `alpine:3.22`  
> **Estimated study time:** 150–190 minutes

---

## Overview

In Lesson 11, you proved:

```text
Container removed → writable-layer data removed
```

Now you will store data outside that writable layer.

```text
Container path /data
        │ mount
        ▼
Persistent storage outside container lifecycle
```

You will learn two main storage types:

```text
Named volume → Docker manages the storage location
Bind mount   → You choose a host file or directory
```

The central experiment is:

```text
Create data
    ↓
Remove container
    ↓
Attach same storage to new container
    ↓
Data remains
```

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain why mounts exist
- Distinguish writable-layer data from mounted data
- Create, list, inspect, and remove named volumes
- Mount a named volume into a container
- Prove volume data survives container replacement
- Explain named versus anonymous volumes
- Explain volume lifecycle separately from container lifecycle
- Create a bind mount using an absolute host path
- Explain the difference between volume and bind-mount ownership
- Use `--mount` and recognize `-v`
- Create read-only mounts
- Diagnose permissions and ownership problems
- Explain why mounting over existing container files hides them
- Back up and restore a named volume at an introductory level
- Remove volumes safely without destructive pruning

---

## Prerequisites

You should understand:

- Image layers and container writable layers
- Stop/start versus remove/recreate
- Linux files, directories, permissions, ownership, UID, and GID
- Absolute host paths
- `tar` and basic backup verification
- `docker run`, `exec`, `inspect`, and `rm`
- `docker diff`

Use your established Docker access model consistently.

---

## Why This Topic Exists

Containers are designed to be replaceable.

But some data is not replaceable:

- Database records
- User uploads
- Generated reports
- Application state
- Certificates and keys
- Shared configuration

If this data exists only in a container writable layer, removing that container removes the data.

A mount separates the data lifecycle from the container lifecycle.

```text
Container A ─┐
             ├── persistent storage
Container B ─┘
```

Container A can be removed, and Container B can attach the same storage.

---

# Part 1 — The Mount Mental Model

## 1. What a Mount Does

A mount makes external storage appear at a path inside the container.

```text
Inside container:
/data/report.txt

Actual storage:
Docker volume or host path
```

The application reads and writes `/data` normally. Docker controls where that path's data is stored.

## 2. Mounted Path Versus Writable Layer

```text
Container filesystem
├── /tmp          → container writable layer
├── /etc          → image + container changes
└── /data         → mounted external storage
```

When the container is removed:

- `/tmp` changes in its writable layer disappear
- `/data` can remain because it lives in external storage

## 3. Main Mount Types

| Type | Storage location | Best beginner use |
| --- | --- | --- |
| Named volume | Managed by Docker | Persistent application data |
| Anonymous volume | Docker-managed with generated name | Temporary/implicit per-container storage; lifecycle can be confusing |
| Bind mount | Exact host path you choose | Source code, configuration, files shared with host |
| tmpfs | Host memory | Intentionally temporary data |

This lesson focuses on named volumes and bind mounts.

## 4. Mount Source and Target

```text
Source → where data comes from outside container
Target → path where it appears inside container
```

Example:

```text
Source: techcorp-data volume
Target: /data inside container
```

---

# Part 2 — Named Volumes

## 5. What a Named Volume Is

A named volume is persistent storage managed by Docker and identified by a human-readable name.

Example:

```text
techcorp-data
```

Docker manages its host storage location and lifecycle as a Docker object.

## 6. Create a Volume

```bash
docker volume create techcorp-data
```

Expected output:

```text
techcorp-data
```

## 7. List Volumes

```bash
docker volume ls
```

Filter:

```bash
docker volume ls --filter name=techcorp-data
```

Columns:

```text
DRIVER    VOLUME NAME
local     techcorp-data
```

## 8. Inspect the Volume

```bash
docker volume inspect techcorp-data
```

Important fields:

- Name
- Driver
- Mountpoint
- Created time
- Labels
- Scope
- Options

Formatted:

```bash
docker volume inspect --format \
  'name={{.Name}} driver={{.Driver}} scope={{.Scope}} mountpoint={{.Mountpoint}}' \
  techcorp-data
```

## 9. Do Not Manage Volume Files Through Docker's Internal Path

The mountpoint may look like:

```text
/var/lib/docker/volumes/techcorp-data/_data
```

Do not treat Docker's internal storage directories as your normal application interface.

Use containers and documented Docker operations to read, write, back up, and restore volume data. Direct modification can cause ownership, consistency, and portability problems.

---

# Part 3 — Mount a Named Volume

## 10. Preferred `--mount` Syntax

```bash
docker run -d \
  --name volume-writer \
  --mount type=volume,source=techcorp-data,target=/data \
  alpine:3.22 \
  sleep 3600
```

Breakdown:

| Field | Meaning |
| --- | --- |
| `type=volume` | Use a Docker volume |
| `source=techcorp-data` | Volume name |
| `target=/data` | Container path |

Synonyms recognized in mount syntax include `src` for source and `dst`/`destination` for target, but use one consistent style.

## 11. Verify Mount

```bash
docker inspect --format '{{json .Mounts}}' volume-writer
```

Typical information includes:

```text
Type: volume
Name: techcorp-data
Destination: /data
RW: true
```

## 12. Write Persistent Data

```bash
docker exec volume-writer sh -c \
  'echo "persistent TechCorp data" > /data/report.txt'
```

Verify:

```bash
docker exec volume-writer cat /data/report.txt
```

## 13. Create Disposable Data Too

```bash
docker exec volume-writer sh -c \
  'echo "temporary container data" > /temporary.txt'
```

Now the locations differ:

```text
/data/report.txt → volume
/temporary.txt   → container writable layer
```

## 14. Diff Behavior

```bash
docker diff volume-writer
```

You may see `/temporary.txt`, but mounted-volume contents are not ordinary writable-layer changes and are not reported in the same way by `docker diff`.

---

# Part 4 — Prove Persistence Across Replacement

## 15. Record Container ID

```bash
docker inspect --format '{{.Id}}' volume-writer
```

## 16. Remove the Container

```bash
docker stop volume-writer
docker rm volume-writer
```

## 17. Prove Volume Still Exists

```bash
docker volume ls --filter name=techcorp-data
```

## 18. Create a New Container

```bash
docker run -d \
  --name volume-reader \
  --mount type=volume,source=techcorp-data,target=/data \
  alpine:3.22 \
  sleep 3600
```

## 19. Read Persistent Data

```bash
docker exec volume-reader cat /data/report.txt
```

Expected:

```text
persistent TechCorp data
```

## 20. Check Disposable Data

```bash
docker exec volume-reader sh -c \
  'test -f /temporary.txt && cat /temporary.txt || echo "temporary.txt is gone"'
```

Expected:

```text
temporary.txt is gone
```

## 21. Core Result

```text
Old container removed
├── old writable layer removed
└── named volume remained

New container created
├── new writable layer
└── same named volume mounted
```

---

# Part 5 — `--mount` Versus `-v`

## 22. Long Explicit Syntax

```bash
--mount type=volume,source=techcorp-data,target=/data
```

## 23. Short Volume Syntax

```bash
-v techcorp-data:/data
```

Equivalent basic example:

```bash
docker run -d \
  --name short-volume \
  -v techcorp-data:/data \
  alpine:3.22 sleep 3600
```

## 24. Why This Course Prefers `--mount`

- Field names are explicit
- Volume and bind mounts are easier to distinguish
- Read-only and other options are clearer
- Mistakes are easier to review in scripts

You still need to recognize `-v` because it is common in documentation and existing automation.

## 25. Colon Order for `-v`

Named volume:

```text
-v VOLUME_NAME:CONTAINER_PATH[:OPTIONS]
```

Bind mount:

```text
-v HOST_PATH:CONTAINER_PATH[:OPTIONS]
```

The source meaning depends on whether the left side is a volume name or a filesystem path.

---

# Part 6 — Anonymous Volumes

## 26. What Makes It Anonymous?

If a volume target is requested without a human-selected source name, Docker creates a volume with a generated name.

Example:

```bash
docker run -d \
  --name anonymous-demo \
  --mount type=volume,target=/data \
  alpine:3.22 sleep 3600
```

## 27. Find the Generated Name

```bash
docker inspect --format '{{json .Mounts}}' anonymous-demo
```

The volume `Name` is a long generated identifier.

## 28. Anonymous Does Not Mean Non-persistent

An anonymous volume can remain after normal container removal.

The problem is manageability:

- Its name is not meaningful
- Reattaching it deliberately is harder
- Unused anonymous volumes can accumulate

## 29. `--rm` Special Behavior

For a container started with `--rm`, anonymous volumes associated with it are removed automatically when the container is removed. Named volumes are not automatically deleted by `--rm`.

## 30. Named Volume Recommendation

For important persistent data, use an intentional named volume:

```text
orders-db-data
product-uploads
grafana-storage
```

The name communicates ownership and purpose.

---

# Part 7 — Volume Removal

## 31. Remove a Volume

```bash
docker volume rm techcorp-data
```

Docker refuses if a container still references the volume, including a stopped container.

## 32. Find Containers Using a Volume

```bash
docker ps -a --filter volume=techcorp-data
```

## 33. Safe Removal Order

```text
Confirm backup/retention requirement
          ↓
Stop application consistently
          ↓
Remove or recreate referencing containers
          ↓
Verify volume is unused
          ↓
Remove volume deliberately
```

## 34. Volume Deletion Is Data Deletion

Unlike removing a replaceable container, removing a volume can delete the persistent business data you intended to preserve.

Never run volume cleanup merely because a volume is “unused” at this moment.

It may be:

- A stopped application's data
- A rollback volume
- A backup staging area
- Awaiting a replacement container

## 35. Prune Warning

```bash
docker volume prune
```

removes unused anonymous local volumes by default. With `-a`, it includes unused named volumes.

Avoid broad pruning on important hosts. Inspect and remove specific volumes.

---

# Part 8 — Bind Mounts

## 36. What a Bind Mount Is

A bind mount maps an existing host file or directory into a container.

```text
Host: /home/hamad/projects/site
             │ bind mount
             ▼
Container: /usr/share/nginx/html
```

Changes are visible on both sides, subject to permissions and caching/platform behavior.

## 37. When Bind Mounts Are Useful

- Development source code
- Configuration managed on the host
- Static files edited by host tools
- Local certificates or sockets when deliberately required
- Data that must be directly accessible to host processes

## 38. Create a Safe Lab Directory

On the host:

```bash
mkdir -p "$HOME/docker-bind-lab"
printf '%s\n' 'Hello from the host' > "$HOME/docker-bind-lab/index.txt"
```

These are host commands, not container commands.

## 39. Mount the Directory

```bash
docker run -d \
  --name bind-reader \
  --mount type=bind,source="$HOME/docker-bind-lab",target=/shared \
  alpine:3.22 sleep 3600
```

## 40. Read Host File Inside Container

```bash
docker exec bind-reader cat /shared/index.txt
```

Expected:

```text
Hello from the host
```

## 41. Write From Container

```bash
docker exec bind-reader sh -c \
  'echo "written by container" > /shared/container.txt'
```

On host:

```bash
cat "$HOME/docker-bind-lab/container.txt"
```

The file appears on the host because `/shared` directly maps the host directory.

## 42. Host Changes Appear in Container

Host:

```bash
printf '%s\n' 'updated on host' >> "$HOME/docker-bind-lab/index.txt"
```

Container:

```bash
docker exec bind-reader cat /shared/index.txt
```

---

# Part 9 — Bind Mount Path Rules

## 43. Source Belongs to Docker Daemon Host

The bind source path is resolved on the Docker daemon host.

For a local Docker Engine, that is your Ubuntu host.

For a remote daemon, a path on your client laptop is not automatically available on the remote Docker host.

## 44. Use Absolute Paths

Clear form:

```bash
--mount type=bind,source="$HOME/docker-bind-lab",target=/shared
```

Verify expansion if uncertain:

```bash
printf '%s\n' "$HOME/docker-bind-lab"
```

## 45. Missing Source Behavior

With bind `--mount`, Docker reports an error if the source path does not exist.

Create and verify it first:

```bash
mkdir -p "$HOME/docker-bind-lab"
ls -ld "$HOME/docker-bind-lab"
```

With `-v`, Docker may create a missing host path as a directory, which can hide path typos. This is another reason to prefer `--mount`.

## 46. Strong Host Coupling

A container definition using:

```text
/home/hamad/docker-bind-lab
```

depends on that path existing with suitable permissions on the target host.

Named volumes are usually more portable across Docker hosts at the definition level, though moving their data still requires a storage/backup plan.

---

# Part 10 — Read-only Mounts

## 47. Why Read-only?

If the container only needs to consume configuration or static content, it should not be able to modify it.

## 48. Read-only Bind Mount

```bash
docker run -d \
  --name bind-readonly \
  --mount type=bind,source="$HOME/docker-bind-lab",target=/shared,readonly \
  alpine:3.22 sleep 3600
```

Read:

```bash
docker exec bind-readonly cat /shared/index.txt
```

Attempt write:

```bash
docker exec bind-readonly sh -c \
  'echo blocked > /shared/blocked.txt'
```

Expected: read-only filesystem error.

## 49. Read-only Named Volume

```bash
docker run --rm \
  --mount type=volume,source=techcorp-data,target=/data,readonly \
  alpine:3.22 cat /data/report.txt
```

## 50. `-v` Read-only Syntax

```bash
-v "$HOME/docker-bind-lab":/shared:ro
```

Prefer explicit `readonly` with `--mount` in new scripts.

## 51. Read-only Is Path-specific

Making `/shared` read-only does not make the entire container filesystem read-only.

The container may still write elsewhere unless additional controls are used.

---

# Part 11 — Mounting Over Existing Data

## 52. The Obscuring Effect

If an image already contains files at `/app` and you mount another source at `/app`, the mounted content hides the image content at that path.

```text
Before mount:
/app → image files

After mount:
/app → mounted source files
Image files still exist underneath but are hidden
```

## 53. Familiar Linux Analogy

Mounting a USB drive at `/mnt` hides the directory's previous visible contents until unmounted.

Docker mounts behave similarly.

## 54. Common Nginx Mistake

Nginx image contains default web content at:

```text
/usr/share/nginx/html
```

If you bind an empty host directory there:

```bash
docker run -d \
  --mount type=bind,source=/empty/path,target=/usr/share/nginx/html \
  nginx:alpine
```

the default page becomes hidden by the empty mount.

The image is not broken. The mount changed what the path displays.

## 55. Recreate Without the Mount

You cannot normally detach a mount from a running standalone container and reveal the underlying files through a simple Docker command. Recreate the container without that mount.

---

# Part 12 — Permissions and Ownership

## 56. Container Processes Use UIDs and GIDs

Linux permissions operate on numeric user and group IDs.

A process inside a container may run as:

```text
UID 0
UID 1000
UID 101
another application-specific UID
```

Mounted files retain ownership/permission meaning from the underlying filesystem.

## 57. Inspect Container User

Image default:

```bash
docker image inspect --format '{{json .Config.User}}' IMAGE
```

Running process:

```bash
docker exec CONTAINER id
```

## 58. Inspect Host Directory

```bash
ls -ld "$HOME/docker-bind-lab"
ls -ln "$HOME/docker-bind-lab"
```

`-n` shows numeric UID/GID, which helps compare host and container identities.

## 59. Permission-denied Investigation

Collect:

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
docker exec CONTAINER id
ls -ldn HOST_PATH
namei -l HOST_PATH
```

Ask:

- Is mount read-only?
- Does source exist?
- Does process UID/GID have directory traversal permission?
- Does it have write permission on the target directory?
- Are SELinux/AppArmor policies involved?
- Is the filesystem itself read-only?

## 60. Avoid `chmod 777` as a Blind Fix

World-writable permissions can expose or corrupt host data.

Correct ownership and permissions for the specific application identity and access requirement.

## 61. Root in Container Is Security-sensitive

A root process with a writable bind mount can modify the mapped host files, subject to Docker security controls and user-namespace configuration.

Do not bind-mount sensitive host directories such as `/`, `/etc`, or the Docker socket into untrusted containers.

---

# Part 13 — Named Volume Versus Bind Mount

## 62. Comparison

| Area | Named volume | Bind mount |
| --- | --- | --- |
| Location | Managed by Docker | Chosen host path |
| Host coupling | Lower | Strong |
| Host access | Normally through containers/Docker workflow | Direct |
| Common use | Databases and persistent app data | Development code/config/files |
| Portability | More Docker-managed | Depends on path existing |
| Security risk | Controlled Docker storage | Can expose arbitrary host paths |
| Backup | Mount volume into backup tool/container | Use host backup tools carefully |

## 63. Decision Rule

Choose a named volume when:

- Application needs persistent data
- Docker should manage the location
- Host applications do not need direct path access

Choose a bind mount when:

- A specific host path is part of the design
- Host and container must both access the files
- Development source or managed configuration is shared

## 64. Neither Replaces Backups

Persistence means data survives container replacement.

Backup means you have a separate recoverable copy.

```text
Persistent volume ≠ backup
```

A volume can be corrupted, deleted, encrypted by malware, or lost with the host.

---

# Part 14 — Backup a Named Volume

## 65. Backup Goal

Create a tar archive of volume contents in a controlled host backup directory.

For application-consistent backups, the application may need to stop, flush, lock, or use its native backup tool. Copying live database files can produce an unusable backup.

## 66. Create Backup Directory

```bash
mkdir -p "$HOME/docker-volume-backups"
```

## 67. Stop Writer for Consistency

If `volume-reader` is only a lab container:

```bash
docker stop volume-reader
```

## 68. Create Archive

```bash
docker run --rm \
  --mount type=volume,source=techcorp-data,target=/data,readonly \
  --mount type=bind,source="$HOME/docker-volume-backups",target=/backup \
  alpine:3.22 \
  tar -czf /backup/techcorp-data.tar.gz -C /data .
```

Breakdown:

```text
Volume mounted read-only at /data
Host backup directory mounted at /backup
tar reads /data
Archive written to host backup directory
```

## 69. Verify Archive Exists

```bash
ls -lh "$HOME/docker-volume-backups/techcorp-data.tar.gz"
tar -tzf "$HOME/docker-volume-backups/techcorp-data.tar.gz"
```

Listing the archive is necessary but not sufficient. A complete backup process includes restore testing.

## 70. Checksum

```bash
sha256sum "$HOME/docker-volume-backups/techcorp-data.tar.gz" \
  > "$HOME/docker-volume-backups/techcorp-data.tar.gz.sha256"
```

Verify later:

```bash
cd "$HOME/docker-volume-backups"
sha256sum -c techcorp-data.tar.gz.sha256
```

The checksum detects file changes/corruption; it does not prove application consistency.

---

# Part 15 — Restore to a New Volume

## 71. Never Test Restore Over the Only Copy

Create a separate volume:

```bash
docker volume create techcorp-data-restore
```

## 72. Restore Archive

```bash
docker run --rm \
  --mount type=volume,source=techcorp-data-restore,target=/restore \
  --mount type=bind,source="$HOME/docker-volume-backups",target=/backup,readonly \
  alpine:3.22 \
  tar -xzf /backup/techcorp-data.tar.gz -C /restore
```

## 73. Verify Restored Data

```bash
docker run --rm \
  --mount type=volume,source=techcorp-data-restore,target=/data,readonly \
  alpine:3.22 \
  cat /data/report.txt
```

Expected:

```text
persistent TechCorp data
```

## 74. Real Database Restore

For MySQL, PostgreSQL, and other databases, use documented application-native backup and restore methods when appropriate. File-level tar examples teach volume movement, not guaranteed database consistency.

---

## Big Picture

```text
Container A writable layer → removed with A

Container A /data ─┐
                   ├── named volume → survives A
Container B /data ─┘

Container /shared ─── bind mount ─── host directory
```

Lifecycle:

```text
Container lifecycle: create → run → remove
Volume lifecycle:    create → mount → unmount → remain → remove deliberately
```

---

## Commands — Detailed Reference

## 75. Volume Management

```bash
docker volume create VOLUME
docker volume ls
docker volume inspect VOLUME
docker volume rm VOLUME
```

## 76. Named Volume Mount

```bash
docker run --mount \
  type=volume,source=VOLUME,target=/data \
  IMAGE
```

## 77. Bind Mount

```bash
docker run --mount \
  type=bind,source=/absolute/host/path,target=/container/path \
  IMAGE
```

## 78. Read-only

```bash
docker run --mount \
  type=volume,source=VOLUME,target=/data,readonly \
  IMAGE
```

## 79. Inspect Container Mounts

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
```

## 80. Find References

```bash
docker ps -a --filter volume=VOLUME
```

---

## Guided Hands-on Lab — Persist, Replace, Restore

### Scenario

TechCorp needs a report to survive container replacement and must also prove it can be restored from backup.

### Phase A — Create Volume

```bash
docker volume create techcorp-data
docker volume inspect techcorp-data
```

### Phase B — Write Data

```bash
docker run -d \
  --name volume-writer \
  --mount type=volume,source=techcorp-data,target=/data \
  alpine:3.22 sleep 3600

docker exec volume-writer sh -c \
  'echo "persistent TechCorp data" > /data/report.txt'

docker exec volume-writer sh -c \
  'echo "disposable" > /temporary.txt'
```

### Phase C — Inspect

```bash
docker inspect --format '{{json .Mounts}}' volume-writer
docker diff volume-writer
docker exec volume-writer cat /data/report.txt
```

### Phase D — Replace Container

```bash
docker stop volume-writer
docker rm volume-writer

docker run -d \
  --name volume-reader \
  --mount type=volume,source=techcorp-data,target=/data \
  alpine:3.22 sleep 3600
```

### Phase E — Verify Persistence Boundary

```bash
docker exec volume-reader cat /data/report.txt
docker exec volume-reader sh -c \
  'test -f /temporary.txt && cat /temporary.txt || echo "temporary.txt is gone"'
```

### Phase F — Read-only Consumer

```bash
docker run --rm \
  --mount type=volume,source=techcorp-data,target=/data,readonly \
  alpine:3.22 \
  cat /data/report.txt

docker run --rm \
  --mount type=volume,source=techcorp-data,target=/data,readonly \
  alpine:3.22 \
  sh -c 'echo blocked > /data/blocked.txt'
```

The second command should fail.

### Phase G — Bind Mount

```bash
mkdir -p "$HOME/docker-bind-lab"
printf '%s\n' 'host file' > "$HOME/docker-bind-lab/host.txt"

docker run -d \
  --name bind-lab \
  --mount type=bind,source="$HOME/docker-bind-lab",target=/shared \
  alpine:3.22 sleep 3600

docker exec bind-lab cat /shared/host.txt
docker exec bind-lab sh -c 'echo container > /shared/container.txt'
cat "$HOME/docker-bind-lab/container.txt"
```

### Phase H — Backup

```bash
mkdir -p "$HOME/docker-volume-backups"
docker stop volume-reader

docker run --rm \
  --mount type=volume,source=techcorp-data,target=/data,readonly \
  --mount type=bind,source="$HOME/docker-volume-backups",target=/backup \
  alpine:3.22 \
  tar -czf /backup/techcorp-data.tar.gz -C /data .

tar -tzf "$HOME/docker-volume-backups/techcorp-data.tar.gz"
```

### Phase I — Restore Test

```bash
docker volume create techcorp-data-restore

docker run --rm \
  --mount type=volume,source=techcorp-data-restore,target=/restore \
  --mount type=bind,source="$HOME/docker-volume-backups",target=/backup,readonly \
  alpine:3.22 \
  tar -xzf /backup/techcorp-data.tar.gz -C /restore

docker run --rm \
  --mount type=volume,source=techcorp-data-restore,target=/data,readonly \
  alpine:3.22 cat /data/report.txt
```

### Phase J — Cleanup Decision

Remove lab containers:

```bash
docker rm volume-reader
docker stop bind-lab
docker rm bind-lab
```

List volumes:

```bash
docker volume ls --filter name=techcorp-data
```

Remove volumes only after confirming the backup and restore evidence:

```bash
docker volume rm techcorp-data techcorp-data-restore
```

Host backup files and bind-lab directory remain until you remove them deliberately.

### Acceptance Criteria

You can prove:

- Named volume exists independently
- Mounted data survives container removal
- Writable-layer file does not survive replacement
- Read-only mount blocks writes
- Bind mount shows the same files on host and container
- Container UID/GID and host permissions affect access
- Archive contains the expected volume data
- Restored data works from a different volume
- Volume cleanup is separate from container cleanup

---

## Discovery Questions

### Question 1

Why does `/data/report.txt` survive while `/temporary.txt` disappears?

<details>
<summary>Explanation</summary>

`/data` is a mounted volume outside the container writable layer. `/temporary.txt` belonged to the removed writable layer.

</details>

### Question 2

Does removing a container automatically remove its named volume?

<details>
<summary>Explanation</summary>

No. Named volumes have an independent lifecycle and remain until explicitly removed.

</details>

### Question 3

Why can a bind-mounted container modify host files?

<details>
<summary>Explanation</summary>

The mapped path is the host directory itself. A writable mount lets container processes change it according to filesystem permissions and security controls.

</details>

### Question 4

Why might an empty bind mount make image files appear missing?

<details>
<summary>Explanation</summary>

The mount covers the container target path and obscures the image files underneath.

</details>

### Question 5

Why is a volume not a backup?

<details>
<summary>Explanation</summary>

It is the live persistent data. It can still be corrupted, deleted, or lost with the host. A backup is a separate recoverable copy tested through restoration.

</details>

---

## Production Scenario — Permission Denied After Migration

TechCorp moves an application to a new image whose process runs as UID `10001`. The named volume contains files owned by UID `999` and mode `0700`.

The container starts but logs:

```text
permission denied: /data/app.db
```

Evidence:

```bash
docker inspect --format '{{json .Mounts}}' app
docker inspect --format '{{json .Config.User}}' app
docker exec app id
docker logs --tail 100 app
```

Use a controlled helper container or documented application migration process to inspect numeric volume ownership. Do not blindly `chmod 777`.

Root cause:

```text
The new application UID/GID lacks permission for existing persistent files.
```

Correction must align ownership/permissions with the intended runtime identity, preserve least privilege, and include rollback and backup protection.

---

## Troubleshooting Workflow

## 81. Data Missing After Replacement

```bash
docker inspect --format '{{json .Mounts}}' OLD_OR_NEW_CONTAINER
docker volume ls
docker ps -a --filter volume=EXPECTED_VOLUME
```

Ask:

- Was the same volume name attached?
- Was the target path correct?
- Did the old data live in the writable layer instead?
- Did a mount obscure another path?
- Was the volume removed?

## 82. Permission Denied

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
docker exec CONTAINER id
docker logs --tail 100 CONTAINER
```

For bind mounts:

```bash
ls -ldn HOST_PATH
namei -l HOST_PATH
```

## 83. Volume Removal Fails

```bash
docker ps -a --filter volume=VOLUME
```

Stopped containers still count as references. Remove/recreate them only after confirming their purpose and data safety.

## 84. Bind Source Error

```bash
printf '%s\n' "$HOST_PATH"
ls -ld "$HOST_PATH"
```

Use an absolute existing path with `--mount`.

## 85. Image Files Disappeared

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
```

Check whether a mount covers their directory. Recreate without or with the correct populated mount.

---

## Common Mistakes

### 1. Thinking a named volume is deleted with the container

Its lifecycle is independent.

### 2. Thinking anonymous means temporary

Anonymous volumes can remain and accumulate.

### 3. Deleting an “unused” volume without checking ownership

Unused only means no current container reference.

### 4. Confusing volume name with host path

Named volumes are Docker-managed; bind mounts use explicit paths.

### 5. Using a relative or mistyped bind path

Prefer verified absolute paths and `--mount`.

### 6. Mounting an empty directory over application files

The mount hides the image content at that target.

### 7. Fixing permissions with `chmod 777`

Align UID/GID and minimal required access.

### 8. Giving untrusted containers sensitive writable bind mounts

They can modify the host paths.

### 9. Calling persistence a backup

Backups are separate and restore-tested.

### 10. Backing up a live database by copying files blindly

Use application-consistent database backup methods.

---

## Best Practices

- Use named volumes for persistent application data
- Use bind mounts only when host-path access is intentional
- Prefer `--mount` for readable configuration
- Use descriptive volume names and labels
- Mount read-only whenever write access is unnecessary
- Run containers as non-root where supported
- Align numeric UID/GID and permissions deliberately
- Back up volumes and test restoration
- Use database-native backup tools for databases
- Never expose sensitive host paths to untrusted images
- Review mounts before container removal or migration
- Avoid broad volume prune on important hosts
- Document ownership, backup, retention, and restore procedures

---

## Interview Questions

### Junior

1. Why do Docker volumes exist?
2. What is a named volume?
3. Does a named volume survive container removal?
4. What is a bind mount?
5. What is the difference between source and target?
6. What is the difference between `--mount` and `-v`?
7. What does read-only do?
8. What is an anonymous volume?
9. Why can mounting hide image files?
10. Is a volume a backup?

### Intermediate

1. When should you choose a volume versus a bind mount?
2. How do UID/GID values affect mounted files?
3. Why can a stopped container block volume removal?
4. How does `--rm` affect anonymous and named volumes?
5. Why is `--mount` safer for missing bind sources?
6. How would you back up and restore a named volume?
7. Why might a file-level live database backup be inconsistent?
8. How would you diagnose data hidden by a mount?
9. What security risks come from writable bind mounts?

### Senior Preview

Volume drivers, remote storage, mount propagation, SELinux relabeling, CSI, filesystem consistency, database snapshots, and Kubernetes PersistentVolumes are deferred until later phases.

---

## Lesson Reflection

Answer in your own words:

1. Draw the writable layer and mounted volume.
2. Explain volume lifecycle versus container lifecycle.
3. Why did report.txt survive replacement?
4. Explain named versus anonymous volume.
5. Explain volume versus bind mount.
6. Why is bind mount more coupled to the host?
7. Why can mounts hide image files?
8. How do UID and GID cause permission problems?
9. Why is read-only useful?
10. Why must backups be restore-tested?

---

## Summary

- Mounts store data outside the container writable layer
- Named volumes are managed by Docker
- Named volumes survive container removal
- New containers can attach the same volume
- Anonymous volumes have generated names and can persist unexpectedly
- Bind mounts map chosen host paths directly
- `--mount` is explicit and preferred in new examples
- `-v` is common and must be recognized
- Read-only mounts reduce modification risk
- Mounts can hide image files at the target path
- Numeric UID/GID and permissions control access
- Writable bind mounts can modify host data
- Volume removal is destructive data removal
- Persistent storage is not a backup
- Backup procedures must include restoration tests

---

## Cheat Sheet

### Volume lifecycle

```bash
docker volume create techcorp-data
docker volume ls
docker volume inspect techcorp-data
docker volume rm techcorp-data
```

### Named volume

```bash
docker run -d \
  --name app \
  --mount type=volume,source=techcorp-data,target=/data \
  alpine:3.22 sleep 3600
```

### Bind mount

```bash
docker run -d \
  --name app \
  --mount type=bind,source="$HOME/docker-bind-lab",target=/shared \
  alpine:3.22 sleep 3600
```

### Read-only

```bash
--mount type=volume,source=techcorp-data,target=/data,readonly
--mount type=bind,source="$HOME/config",target=/config,readonly
```

### Inspect and references

```bash
docker inspect --format '{{json .Mounts}}' CONTAINER
docker ps -a --filter volume=VOLUME
```

### Short syntax

```bash
-v techcorp-data:/data
-v "$HOME/docker-bind-lab":/shared
-v "$HOME/config":/config:ro
```

### Critical distinctions

```text
Writable layer ≠ volume
Volume name ≠ host path
Named ≠ anonymous
Persistent ≠ backed up
Read-only mount ≠ read-only whole container
Container root ≠ harmless access to host bind mounts
```

---

## Challenge — Replace Without Losing Data

### Mission

1. Create named volume `challenge-data`
2. Create `writer-v1` with volume mounted at `/data`
3. Write a versioned report
4. Write a temporary file outside the mount
5. Inspect mounts and diff
6. Remove `writer-v1`
7. Create `reader-v2` using the same volume
8. Prove report remains
9. Prove temporary file is gone
10. Start a read-only consumer and prove writing fails
11. Back up volume to a tar archive
12. Restore into `challenge-data-restore`
13. Verify restored report
14. Remove containers
15. Remove volumes only after verifying backup

### Rules

- Use `--mount`
- No `chmod 777`
- No volume prune
- No direct edits under `/var/lib/docker`
- Backup must be restore-tested

### Report Template

```text
Volume name:
Writer container ID:
Persistent file:
Temporary file:
Mount inspection:
Reader container ID:
Persistent result after replacement:
Temporary result after replacement:
Read-only test:
Backup archive:
Restore volume:
Restore verification:
Cleanup decision:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-12/
├── 01-volume-create.txt
├── 02-volume-inspect.txt
├── 03-writer-mounts.txt
├── 04-persistent-data.txt
├── 05-replacement-proof.txt
├── 06-readonly-test.txt
├── 07-bind-mount-proof.txt
├── 08-permission-evidence.txt
├── 09-backup-listing.txt
├── 10-restore-proof.txt
└── storage-report.md
```

Do not use confidential production data in labs. Redact host usernames and paths if sharing evidence publicly.

---

## Official References

- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Docker storage overview](https://docs.docker.com/engine/storage/)
- [Docker volume command](https://docs.docker.com/reference/cli/docker/volume/)
- [Docker volume remove](https://docs.docker.com/reference/cli/docker/volume/rm/)

---

## Next Lesson

**Lesson 13 — Environment Variables and Runtime Configuration**

You can now persist data independently from containers. Next, you will separate runtime configuration from image content.

You will learn:

- Image defaults versus runtime configuration
- `ENV` and `docker run -e`
- Environment files
- Inspecting configuration safely
- Why passwords should not be treated as ordinary environment values
- Configuration changes and container recreation
- Dev, test, and production configuration patterns

---

## Completion Check

Before moving forward, explain without reading:

```text
Mount source and target
Named and anonymous volumes
Volume and container lifecycle
Volume versus bind mount
--mount versus -v
Read-write versus read-only
Why mounts hide existing files
UID/GID permission behavior
Why volume removal is destructive
Persistence versus backup
Backup and restore-test workflow
```

If you can remove the writer, attach the same volume to a reader, and restore the data into a new volume, you understand Docker persistence fundamentals.

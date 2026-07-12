# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 06 — Docker Images

> **Level:** Beginner to early intermediate  
> **Type:** Guided practical lesson  
> **Primary lab images:** `nginx:alpine` and `nginx:stable-alpine`  
> **Estimated study time:** 120–160 minutes

---

## Overview

Every container you have run was created from an image.

Until now, “image” meant a packaged application template. In this lesson, you will make that model precise enough for real DevOps work.

You will learn:

- How Docker names an image
- What repository, namespace, registry, and tag mean
- Why `latest` is not a promise
- What an image ID identifies
- What a registry digest identifies
- Why tags can move but digests are content-based
- How images contain read-only layers and metadata
- How one image creates many containers
- How to inspect and compare images
- How to remove images safely
- Why removal may only untag an image
- Why an existing container can block image deletion

This is still not the image-building lesson. You will inspect existing images first; later you will build your own with Dockerfiles.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Parse a complete Docker image reference
- Explain registry, namespace, repository, and tag
- Explain the default values in the short name `nginx`
- List local images and interpret their columns
- Pull a specific image tag
- Explain image pull output at a beginner-friendly layer level
- Distinguish tag, image ID, and digest
- Explain why different tags can share one image ID
- Explain why tags should not be treated as immutable
- Inspect image configuration and platform information
- Read image history without assuming it reconstructs the Dockerfile perfectly
- Create several containers from one image
- Find containers that reference an image
- Remove an image reference safely
- Explain dangling images and use prune cautiously

---

## Prerequisites

You should understand:

- Image versus container
- Container registry
- `docker run`
- `docker ps -a`
- `docker rm`
- Container lifecycle
- Main process
- Basic foreground and detached execution

Examples use:

```bash
docker ...
```

Use your chosen access model consistently.

---

## Why This Topic Exists

Imagine that production uses:

```text
company-api:latest
```

Yesterday, `latest` pointed to one image. Today, a CI/CD pipeline pushes a new image under the same tag.

The name looks unchanged, but the referenced content may now be different.

If an incident occurs, the team must answer:

- Which exact image content was deployed?
- Which tag was used?
- Did the tag move?
- Which architecture was selected?
- Which containers were created from the image?
- Can the exact artifact be reproduced or rolled back?

Understanding image identity is essential for reliable deployments.

---

# Part 1 — Image Mental Model

## 1. What an Image Contains

A Docker image contains:

- A packaged filesystem
- Application files and dependencies
- Image configuration metadata
- A default command or entrypoint when defined
- Environment defaults when defined
- Working-directory, user, port, and other metadata when defined

Beginner model:

```text
Image
├── Application files
├── Runtime and libraries
├── Filesystem content
└── Startup/configuration metadata
```

## 2. Image Is Read-only

Image layers are treated as read-only templates.

When Docker creates a container, it adds a container-specific writable layer above the image.

```text
Container writable layer   ← changes made by this container
────────────────────────
Image layer                ← read-only
Image layer                ← read-only
Base image layer           ← read-only
```

This explains two important facts:

1. Multiple containers can share the same underlying image content
2. Changes in one container do not rewrite the image

The container writable layer and data-loss behavior receive a dedicated lesson later.

## 3. One Image, Many Containers

```text
nginx:alpine image
    ├── frontend-a container
    ├── frontend-b container
    └── frontend-c container
```

Each container has:

- Its own container ID
- Its own name
- Its own lifecycle state
- Its own writable layer

They can share the image's read-only layers.

## 4. Image Is Not a Running Process

```text
Image     → stored package and metadata
Container → instance created from image
Process   → runs when container starts
```

An image can exist locally without any container created from it.

---

# Part 2 — Image Names

## 5. Full Image Reference Format

```text
[REGISTRY_HOST[:PORT]/]NAMESPACE/REPOSITORY[:TAG]
```

Example:

```text
registry.example.com:5000/techcorp/product-api:2.4.1
```

Breakdown:

| Part | Value | Meaning |
| --- | --- | --- |
| Registry host | `registry.example.com` | Service storing/distributing image |
| Port | `5000` | Optional registry network port |
| Namespace | `techcorp` | User, team, or organization scope |
| Repository | `product-api` | Image repository name |
| Tag | `2.4.1` | Human-readable version/variant reference |

## 6. What `nginx` Expands To

When you write:

```text
nginx
```

Docker applies defaults similar to:

```text
docker.io/library/nginx:latest
```

```text
docker.io → default Docker Hub registry
library   → official-image namespace
nginx     → repository
latest    → default tag
```

You do not need to type the full form for normal official-image usage, but you should understand what was implied.

## 7. Registry Versus Repository

```text
Registry
└── Namespace
    └── Repository
        ├── tag: latest
        ├── tag: stable-alpine
        └── tag: alpine
```

The registry is the service. A repository groups related image versions or variants.

## 8. Official Images

Docker Official Images are curated image repositories with documented maintenance practices and clear Docker Hub presentation.

Examples include:

```text
nginx
alpine
ubuntu
postgres
redis
```

“Official” does not mean:

- Automatically secure for every configuration
- Free from all vulnerabilities
- Suitable without version selection or testing
- Maintained by Docker alone in every sense

Always read the image documentation, supported tags, configuration, and security guidance.

---

# Part 3 — Tags

## 9. What a Tag Is

A tag is a human-readable reference within a repository.

Examples:

```text
nginx:latest
nginx:alpine
nginx:stable-alpine
ubuntu:24.04
company-api:2.4.1
```

Tags often communicate:

- Application version
- Operating-system base variant
- Release channel
- Build or environment convention

## 10. Tags Can Move

A registry publisher can push newer image content using the same tag.

```text
Monday:
nginx:alpine → content A

Later:
nginx:alpine → content B
```

The string stayed the same while the referenced content changed.

Therefore:

> A tag is a convenient reference, not an immutable identity guarantee.

## 11. `latest` Is Just a Tag

If you omit a tag, Docker normally uses `latest`.

```bash
docker pull nginx
```

means:

```bash
docker pull nginx:latest
```

But `latest` does not automatically mean:

- Highest semantic version
- Most recently built image
- Most secure image
- Recommended production version

It means the publisher assigned content to the tag named `latest`.

## 12. Multiple Tags Can Reference the Same Image

```text
repository:tag-a ─┐
                  ├──> same local image ID
repository:tag-b ─┘
```

If two tags resolve to identical image configuration and layers for your platform, Docker may show the same image ID for both.

This does not duplicate all underlying content on disk.

---

# Part 4 — Pull Images

## 13. Pull a Specific Tag

```bash
docker pull nginx:alpine
```

General syntax:

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

## 14. Pull Output

Typical shape:

```text
alpine: Pulling from library/nginx
<layer-id>: Pull complete
<layer-id>: Pull complete
...
Digest: sha256:...
Status: Downloaded newer image for nginx:alpine
```

Exact layer IDs, digest, size, and wording will change over time.

## 15. Layer Lines

Each layer represents a set of filesystem changes or related image content.

Docker downloads layers that are not already available locally.

If another image uses an identical layer, Docker can reuse that content.

```text
Image A ─┐
         ├── shared layer stored once
Image B ─┘
```

Layer internals and storage drivers are taught later. For now:

> Images are composed of reusable read-only layers.

## 16. Pull Again

```bash
docker pull nginx:alpine
```

Docker checks the registry reference again and reports whether the local image is current or newer content was downloaded.

This differs from merely running with the default missing-only pull behavior.

## 17. Pull Policy with Run

Docker supports:

```bash
docker run --pull=missing IMAGE
docker run --pull=always IMAGE
docker run --pull=never IMAGE
```

| Policy | Meaning |
| --- | --- |
| `missing` | Default: pull only when image is absent locally |
| `always` | Attempt a pull before creating the container |
| `never` | Use only local image; fail if absent |

This explains why re-running a mutable tag without pulling may continue to use local content.

## 18. Pull Two Nginx Tags

```bash
docker pull nginx:alpine
docker pull nginx:stable-alpine
```

Do not assume they will have different image IDs. The publisher's current tag mapping determines the result.

The lab will observe rather than predict.

---

# Part 5 — List Images

## 19. List Local Images

```bash
docker image ls
```

Alias:

```bash
docker images
```

Structured form is preferred in this course:

```bash
docker image ls
```

## 20. Filter by Repository

```bash
docker image ls nginx
```

Typical table:

```text
REPOSITORY   TAG             IMAGE ID       CREATED        SIZE
nginx        alpine          abc123...      ...            ...
nginx        stable-alpine   def456...      ...            ...
```

## 21. Read the Columns

### Repository

Repository part of the local image reference.

### Tag

Human-readable tag attached to the image reference.

### Image ID

Short form of the content-addressable local image configuration identifier.

### Created

Image creation time recorded in its metadata, not the time you downloaded it.

### Size

Reported image size reflects unpacked/content size semantics, not necessarily the network download size or additional disk space consumed when layers are shared.

Do not add the displayed sizes and assume that exact amount is uniquely consumed on disk.

## 22. Show Digests

```bash
docker image ls --digests nginx
```

This adds a `DIGEST` column when repository digest information is available.

## 23. Quiet Output

```bash
docker image ls -q
```

`-q` or `--quiet` prints image IDs only.

Useful in scripts, but dangerous when blindly passed to a removal command. Inspect what a generated list contains before performing bulk changes.

## 24. Filter References

```bash
docker image ls --filter reference='nginx:*'
```

This filters image references matching the pattern.

---

# Part 6 — Image ID Versus Digest

## 25. Image ID

The image ID is a SHA-256-based content-addressable identifier for the image's configuration, which references its layers.

Docker normally displays a shortened ID:

```text
abc123def456
```

Full form can be inspected:

```bash
docker image inspect --format '{{.Id}}' nginx:alpine
```

Typical shape:

```text
sha256:<long-value>
```

## 26. Registry Digest

A registry digest is a content-based identifier for the pulled registry manifest or image index reference.

Typical reference:

```text
nginx@sha256:<long-digest>
```

A digest lets you request specific published content rather than following a movable tag.

## 27. Tag Versus Digest

```text
Tag:
nginx:alpine
human-friendly, publisher can move it

Digest:
nginx@sha256:...
content-based, pins the referenced registry content
```

## 28. Why Image ID and Digest May Differ

They may identify different objects in the image-distribution model:

- Image ID identifies local image configuration content
- Registry digest may identify a manifest or multi-platform image index

Do not expect the full image ID string and repository digest string to always be identical.

## 29. Multi-platform Images

One tag can refer to a multi-platform image index containing variants for architectures such as:

```text
linux/amd64
linux/arm64
linux/arm/v7
```

Docker selects a compatible variant for the host unless a platform is explicitly requested.

```text
nginx:alpine tag
       ↓ image index
       ├── linux/amd64 manifest
       ├── linux/arm64 manifest
       └── other platforms
```

This is why deployment teams record both image identity and platform.

## 30. Pull by Digest

After obtaining a real current digest from pull output or trusted registry metadata:

```bash
docker pull nginx@sha256:ACTUAL_DIGEST
```

Never copy a placeholder literally.

Advantages:

- Reproducible content selection
- Protection from tag movement
- Stronger deployment traceability

Trade-off:

- Less human-readable
- Updates require intentionally changing the digest

A useful production form can record both:

```text
nginx:alpine@sha256:ACTUAL_DIGEST
```

The tag communicates intent; the digest pins content.

---

# Part 7 — Inspect Images

## 31. Full Inspection

```bash
docker image inspect nginx:alpine
```

This returns detailed JSON metadata.

Possible areas include:

- ID
- Repository tags
- Repository digests
- Creation timestamp
- Architecture and operating system
- Configuration
- Layer information
- Size

## 32. Select Useful Fields

```bash
docker image inspect --format '{{.Id}}' nginx:alpine
```

```bash
docker image inspect --format '{{.Os}}/{{.Architecture}}' nginx:alpine
```

```bash
docker image inspect --format '{{json .RepoTags}}' nginx:alpine
```

```bash
docker image inspect --format '{{json .RepoDigests}}' nginx:alpine
```

```bash
docker image inspect --format '{{json .Config.Cmd}}' nginx:alpine
```

```bash
docker image inspect --format '{{json .Config.Entrypoint}}' nginx:alpine
```

## 33. Interpret Missing Values

An entrypoint or command may appear as `null` because the image defines behavior differently or the other field supplies it.

Do not interpret one field in isolation. The final container command comes from the combined image configuration and any runtime overrides.

That full topic comes in Lesson 07.

## 34. Compare Two Images

```bash
docker image inspect --format \
  'ref=nginx:alpine id={{.Id}} platform={{.Os}}/{{.Architecture}} size={{.Size}}' \
  nginx:alpine
```

```bash
docker image inspect --format \
  'ref=nginx:stable-alpine id={{.Id}} platform={{.Os}}/{{.Architecture}} size={{.Size}}' \
  nginx:stable-alpine
```

Observe:

- Same or different ID
- Same or different size
- Same platform
- Different repository digests

Do not assume tag names guarantee the result.

---

# Part 8 — Image History

## 35. Show History

```bash
docker image history nginx:alpine
```

Columns typically include:

```text
IMAGE
CREATED
CREATED BY
SIZE
COMMENT
```

## 36. What History Helps With

It can help you understand:

- Approximate layer-producing steps
- Which steps added significant size
- Metadata instructions that created no filesystem size
- General image construction pattern

## 37. `<missing>` Is Not Automatically Corruption

History may show:

```text
<missing>
```

for some intermediate layer IDs. Modern image formats and build processes do not require each historical entry to have a locally addressable old image ID.

Do not conclude the image is broken.

## 38. History Is Not a Perfect Dockerfile Recovery Tool

History output can be truncated and transformed. Build secrets or full original source instructions should not be expected to be recoverable safely from it.

Use:

```bash
docker image history --no-trunc nginx:alpine
```

for more detail, but keep the original Dockerfile and build context in version control.

## 39. Security Warning

Never place secrets in Dockerfile instructions or build arguments assuming they will disappear harmlessly. Image history, layers, build caches, or metadata may expose them.

Secret-safe builds are taught later.

---

# Part 9 — Create Containers From Images

## 40. Run Two Variants

```bash
docker run -d --name nginx-alpine-a nginx:alpine
docker run -d --name nginx-stable-a nginx:stable-alpine
```

## 41. Inspect Their Image References

```bash
docker inspect --format \
  'container={{.Name}} image-config-id={{.Image}}' \
  nginx-alpine-a nginx-stable-a
```

The `.Image` value records the image configuration ID used by the container.

## 42. Create More Than One From the Same Image

```bash
docker run -d --name nginx-alpine-b nginx:alpine
```

Now:

```text
nginx:alpine image
    ├── nginx-alpine-a
    └── nginx-alpine-b
```

Verify:

```bash
docker ps --filter ancestor=nginx:alpine
```

Each container has a different container ID but can show the same image ID.

## 43. Tag Movement Does Not Rewrite Existing Containers

When a tag later points to different content, existing containers are not automatically recreated from the new image.

```text
Existing container → retains reference to image content used at creation
Updated tag        → affects future pull/run selection
```

Updating an image tag locally does not patch a running container. Deployment requires intentional replacement/recreation.

---

# Part 10 — Remove Images Safely

## 44. Image Removal Command

```bash
docker image rm IMAGE
```

Aliases:

```bash
docker rmi IMAGE
docker image remove IMAGE
```

Structured form is preferred:

```bash
docker image rm nginx:alpine
```

## 45. Removal Versus Untagging

If multiple tags point to the same image ID, removing one tag may only remove that reference.

```text
Before:
tag-a ─┐
       ├── image ID abc
tag-b ─┘

Remove tag-a:
tag-b ───> image ID abc
```

Docker may report:

```text
Untagged: repository:tag
```

The underlying image content remains because another reference still uses it.

## 46. Existing Container Conflict

Docker may refuse image removal while an existing container references it—even if that container is stopped.

Find references:

```bash
docker ps -a --filter ancestor=nginx:alpine
```

Then decide deliberately:

1. Is the container needed?
2. Does it contain troubleshooting evidence?
3. Can it be stopped safely?
4. Can it be removed?
5. Is image removal actually necessary?

## 47. Safe Lab Cleanup Order

```bash
docker stop nginx-alpine-a nginx-alpine-b nginx-stable-a
docker rm nginx-alpine-a nginx-alpine-b nginx-stable-a
docker image rm nginx:alpine nginx:stable-alpine
```

This removes dependent lab containers first, then image references.

Do not run image removal if other projects use those local images and cleanup is not required.

## 48. Force Removal

```bash
docker image rm --force IMAGE
```

Force can remove references and bypass some conflict protections. It does not make a careless cleanup safe.

Avoid force until you understand:

- All tags pointing to the ID
- All containers referencing it
- Whether content is shared
- Whether the operation affects running work or evidence

## 49. Registry Is Unchanged

Removing a local image does not delete it from Docker Hub or another registry.

```text
docker image rm → local Docker host
registry image  → remains remote
```

Deleting remote registry artifacts requires registry-specific permissions and operations.

---

# Part 11 — Dangling and Unused Images

## 50. Dangling Image

A dangling image is generally an untagged image not referenced by a tag and often produced when a tag moves to a newer build.

It may appear as:

```text
REPOSITORY   TAG      IMAGE ID
<none>       <none>   abc123...
```

## 51. List Dangling Images

```bash
docker image ls --filter dangling=true
```

## 52. Prune Dangling Images

```bash
docker image prune
```

Docker asks for confirmation and removes eligible dangling images.

## 53. Broader Prune

```bash
docker image prune -a
```

`-a` removes all images not used by any existing container, not only dangling images.

This can cause:

- Large future re-downloads
- Loss of useful rollback artifacts
- Slower incident recovery
- Removal of locally built images not pushed elsewhere

Inspect before pruning:

```bash
docker image ls
docker ps -a
```

Do not treat prune as a routine magic disk-cleaning command.

---

## Big Picture

```text
Registry
  └── namespace/repository
       ├── tag: alpine ─────┐
       └── tag: stable ──┐  │
                         ▼  ▼
                    manifest/index + digest
                              │ pull platform variant
                              ▼
Docker host local image store
                              │
                              ├── Container A
                              ├── Container B
                              └── Container C
```

Identity hierarchy:

```text
Tag      → convenient movable reference
Digest   → content-based registry reference
Image ID → local image configuration identity
Container ID → instance identity
```

---

## Commands — Detailed Reference

## 54. `docker pull`

```bash
docker pull NAME:TAG
docker pull NAME@sha256:DIGEST
```

Downloads or updates an image reference from a registry.

## 55. `docker image ls`

```bash
docker image ls
docker image ls nginx
docker image ls --digests nginx
docker image ls --filter reference='nginx:*'
docker image ls --filter dangling=true
```

Lists and filters locally available images.

## 56. `docker image inspect`

```bash
docker image inspect IMAGE
docker image inspect --format '{{.Id}}' IMAGE
docker image inspect --format '{{.Os}}/{{.Architecture}}' IMAGE
```

Displays detailed image metadata.

## 57. `docker image history`

```bash
docker image history IMAGE
docker image history --no-trunc IMAGE
```

Shows image history entries and sizes.

## 58. `docker image rm`

```bash
docker image rm IMAGE
docker image rm IMAGE_1 IMAGE_2
```

Removes image references/content from the local host when allowed.

## 59. `docker image prune`

```bash
docker image prune
docker image prune -a
```

Removes dangling images or, with `-a`, images unused by any container. Review targets and consequences first.

---

## Guided Hands-on Lab — Compare Two Nginx Tags

### Scenario

TechCorp is evaluating two Nginx Alpine tags. Your task is to record their current identities, inspect configuration, create containers, and clean up without confusing containers and images.

### Phase A — Baseline

```bash
docker image ls --digests nginx
docker ps -a --filter ancestor=nginx:alpine
docker ps -a --filter ancestor=nginx:stable-alpine
```

Record existing objects. Do not delete unrelated resources.

### Phase B — Pull

```bash
docker pull nginx:alpine
docker pull nginx:stable-alpine
```

Record each reported digest.

### Phase C — List and Compare

```bash
docker image ls --digests nginx
```

Answer:

- Are image IDs the same or different?
- Are digests the same or different?
- Are sizes the same or different?
- What does the current evidence show?

Do not infer future tag behavior from today's result.

### Phase D — Inspect

```bash
docker image inspect --format \
  'id={{.Id}} platform={{.Os}}/{{.Architecture}} size={{.Size}}' \
  nginx:alpine

docker image inspect --format \
  'id={{.Id}} platform={{.Os}}/{{.Architecture}} size={{.Size}}' \
  nginx:stable-alpine
```

Inspect startup metadata:

```bash
docker image inspect --format \
  'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  nginx:alpine
```

### Phase E — History

```bash
docker image history nginx:alpine
docker image history nginx:stable-alpine
```

Identify the largest visible history entry in each.

### Phase F — Create Containers

```bash
docker run -d --name nginx-alpine-a nginx:alpine
docker run -d --name nginx-alpine-b nginx:alpine
docker run -d --name nginx-stable-a nginx:stable-alpine
```

Verify:

```bash
docker ps --filter ancestor=nginx:alpine
docker ps --filter ancestor=nginx:stable-alpine
```

Record container and image IDs:

```bash
docker inspect --format \
  'container={{.Name}} container-id={{.Id}} image-id={{.Image}}' \
  nginx-alpine-a nginx-alpine-b nginx-stable-a
```

### Phase G — Test Removal Conflict

Attempt only if these are disposable lab resources:

```bash
docker image rm nginx:alpine
```

Read the error or untag result carefully. Do not add `--force`.

Investigate:

```bash
docker ps -a --filter ancestor=nginx:alpine
```

### Phase H — Safe Cleanup

```bash
docker stop nginx-alpine-a nginx-alpine-b nginx-stable-a
docker rm nginx-alpine-a nginx-alpine-b nginx-stable-a
```

If you want to retain images for later lessons, stop here.

If deliberate local image cleanup is part of the lab:

```bash
docker image rm nginx:alpine nginx:stable-alpine
```

### Acceptance Criteria

You can prove:

- Each tag's current repository digest
- Each tag's local image ID
- The selected OS/architecture
- Whether the tags currently point to identical or different content
- Two containers from one image have different container IDs
- Those containers can share the same image ID
- An image reference is different from a container object
- Existing containers affect safe image removal
- Local removal does not delete registry content

---

## Discovery Questions

### Question 1

If two tags show the same image ID, are there necessarily two full copies of the image data?

<details>
<summary>Explanation</summary>

No. Tags can reference the same content, and Docker's content-addressable store reuses identical layers.

</details>

### Question 2

If you remove one of two tags pointing to an image ID, must the underlying image data disappear?

<details>
<summary>Explanation</summary>

No. Docker may only remove that tag reference while other references keep the image content reachable.

</details>

### Question 3

Why is `company-api:2.4.1` easier to read but less immutable than a digest reference?

<details>
<summary>Explanation</summary>

The tag is human-friendly but can be reassigned by a publisher. A digest selects content by a cryptographic content identifier.

</details>

### Question 4

Why can the displayed image creation time be older than your pull time?

<details>
<summary>Explanation</summary>

The column describes when the image content was created, not when your host downloaded it.

</details>

### Question 5

Does pulling a newer image under the same tag automatically replace existing running containers?

<details>
<summary>Explanation</summary>

No. Existing containers keep using the image content from which they were created. Deployment requires intentional container replacement.

</details>

---

## Production Scenario — The Moving Tag

TechCorp deploys:

```text
product-api:production
```

The tag is overwritten during every successful build. Two servers pull at different times:

```text
Server A pulls Monday    → digest X
Server B pulls Wednesday → digest Y
```

Both deployment records say `product-api:production`, but the actual content differs.

Evidence to collect:

```bash
docker image ls --digests
docker image inspect IMAGE
docker inspect CONTAINER
```

Production improvement:

- Generate an immutable version tag per build
- Record the registry digest
- Deploy a tested digest or immutable release reference
- Record target platform
- Promote artifacts rather than rebuilding differently for each environment

The tag can communicate release meaning; the digest provides exact content identity.

---

## Troubleshooting Workflow

## 60. Pull Reports `manifest unknown`

Possible causes:

- Tag does not exist
- Repository name is wrong
- Registry/namespace is wrong
- Authentication hides private content

Verify the full reference and trusted registry documentation.

## 61. Pull Reports Platform Mismatch

Check host and image platform:

```bash
docker info --format '{{.OSType}}/{{.Architecture}}'
docker image inspect --format '{{.Os}}/{{.Architecture}}' IMAGE
```

Multi-platform behavior and emulation will be taught later.

## 62. Image Removal Conflict

Find containers:

```bash
docker ps -a --filter ancestor=IMAGE
```

Find multiple references:

```bash
docker image ls --no-trunc
docker image inspect IMAGE
```

Do not use force until you understand the conflict.

## 63. Disk Usage Seems Larger Than Expected

Use Docker's view:

```bash
docker system df
docker system df -v
```

Displayed image sizes, shared layers, build cache, writable container layers, volumes, and logs all affect disk usage differently.

Do not prune blindly.

## 64. Tag Pull Does Not Change Running App

Expected behavior: pulling changes local image availability/reference, not an existing container.

Verify container image ID:

```bash
docker inspect --format '{{.Image}}' CONTAINER
docker image inspect --format '{{.Id}}' REPOSITORY:TAG
```

If IDs differ, the container must be replaced through the deployment process to use the new image.

---

## Common Mistakes

### 1. Treating `latest` as guaranteed newest

It is an ordinary tag controlled by the publisher.

### 2. Treating a tag as immutable

Tags can be moved to different content.

### 3. Confusing registry, repository, and image

The registry is the service; the repository groups related image references.

### 4. Confusing image ID and container ID

The image identifies the template; the container identifies an instance.

### 5. Expecting digest and image ID strings always to match

They may identify different content objects in a multi-platform distribution model.

### 6. Adding displayed image sizes to calculate exact disk use

Shared layers make simple addition inaccurate.

### 7. Thinking pulling updates running containers

Existing containers are not automatically recreated.

### 8. Using force removal before inspecting references

This hides the dependency relationship instead of understanding it.

### 9. Running prune casually

Unused local images may still be important rollback or build artifacts.

### 10. Assuming image history safely hides secrets

Never place secrets into image build instructions or layers.

---

## Best Practices

- Use explicit version/variant tags instead of relying blindly on `latest`
- Record repository digest for production deployments
- Pin digests where exact reproducibility is required
- Record target platform for multi-platform images
- Use trusted and documented base images
- Inspect image configuration before deployment
- Scan images and maintain an update process
- Keep Dockerfiles and build definitions in version control
- Never bake secrets into image layers
- Remove dependent containers before image cleanup
- Review prune targets and rollback requirements
- Promote the same tested artifact across environments
- Recreate containers intentionally when deploying a new image

---

## Interview Questions

### Junior

1. What is a Docker image?
2. What is an image layer?
3. What is an image repository?
4. What is a tag?
5. What does Docker assume when no tag is supplied?
6. Does `latest` guarantee the newest image?
7. What is an image ID?
8. Can one image create many containers?
9. What does `docker pull` do?
10. What is the difference between `docker rm` and `docker image rm`?

### Intermediate

1. What is the difference between a tag and a digest?
2. Why can multiple tags show the same image ID?
3. Why may image ID differ from repository digest?
4. How do multi-platform images affect digest interpretation?
5. Why does pulling a moved tag not update running containers?
6. What does “Untagged” mean during image removal?
7. Why can a stopped container block image removal?
8. Why is displayed image size not equal to unique disk consumption?
9. What evidence would you record for a reproducible deployment?

### Senior Preview

Manifest indexes, layer diff IDs, chain IDs, content stores, snapshotters, signing, attestations, SBOMs, and supply-chain policy are deferred until advanced image and security lessons.

---

## Lesson Reflection

Answer in your own words:

1. Parse `registry.example.com:5000/techcorp/api:2.4.1`.
2. What defaults are applied to `nginx`?
3. Explain tag versus digest.
4. Explain image ID versus container ID.
5. Why can several tags share one image ID?
6. Why are image layers read-only?
7. What happens when a container changes a file?
8. Why does pulling not update an existing container?
9. What can image history tell you, and what can it not guarantee?
10. Why is prune a production decision rather than harmless housekeeping?

---

## Summary

- An image is a read-only packaged filesystem plus configuration metadata
- Images are built from reusable layers
- Containers add their own writable layer
- One image can create many containers
- A full reference can include registry, port, namespace, repository, and tag
- Docker Hub, `library`, and `latest` are common defaults for short official names
- A tag is human-readable and movable
- `latest` is only a tag
- An image ID identifies local image configuration content
- A registry digest provides content-based published identity
- Multi-platform indexes can contain several platform variants
- `docker pull` downloads missing content and updates references
- Inspection exposes image configuration and platform metadata
- History shows build-layer history but is not perfect Dockerfile recovery
- Removing a tag may only untag the image
- Existing containers can block safe image removal
- Prune operations require review

---

## Cheat Sheet

### Reference format

```text
[REGISTRY[:PORT]/]NAMESPACE/REPOSITORY[:TAG]

nginx
= docker.io/library/nginx:latest
```

### Identity

```text
Tag          → human-friendly, movable
Digest       → content-based registry reference
Image ID     → local image configuration identity
Container ID → created instance identity
```

### Commands

```bash
# Pull
docker pull nginx:alpine
docker pull nginx@sha256:ACTUAL_DIGEST

# List
docker image ls
docker image ls nginx
docker image ls --digests nginx
docker image ls --filter reference='nginx:*'
docker image ls --filter dangling=true

# Inspect
docker image inspect nginx:alpine
docker image inspect --format '{{.Id}}' nginx:alpine
docker image inspect --format '{{.Os}}/{{.Architecture}}' nginx:alpine
docker image inspect --format '{{json .RepoDigests}}' nginx:alpine

# History
docker image history nginx:alpine
docker image history --no-trunc nginx:alpine

# Find containers referencing image
docker ps -a --filter ancestor=nginx:alpine

# Remove local image reference
docker image rm nginx:alpine

# Review dangling cleanup
docker image prune

# Broader unused-image cleanup—use carefully
docker image prune -a

# Disk-usage evidence
docker system df
docker system df -v
```

---

## Challenge — Prove Image Identity

### Scenario

Your senior engineer claims:

> “If two tags have different names, Docker must store two completely separate images, and pulling a new tag automatically updates every running container.”

Use evidence to evaluate both claims.

### Required workflow

1. Pull `nginx:alpine`
2. Pull `nginx:stable-alpine`
3. Record tags, image IDs, digests, platforms, and sizes
4. State whether the IDs currently match
5. Create two containers from `nginx:alpine`
6. Prove the container IDs differ
7. Prove the image IDs used by those containers match
8. Pull `nginx:alpine` again
9. Prove whether existing container image IDs changed
10. Attempt safe image removal and interpret the result
11. Remove only your lab containers
12. Decide whether to keep or remove the images

### Rules

- Do not predict tag mapping; record current evidence
- Do not use force removal
- Do not prune the whole host
- Do not remove unrelated containers
- Do not claim the registry was modified by local removal

### Report template

```text
Tag A:
Image ID A:
Digest A:
Platform A:

Tag B:
Image ID B:
Digest B:
Platform B:

Do tags currently share content?
Evidence:

Container IDs:
Image IDs used:

Did pull update existing containers?
Evidence:

Image-removal result:
Explanation:
```

---

## Lab Assets

```text
assets/chapter-07/lesson-06/
├── 01-image-baseline.txt
├── 02-nginx-alpine-pull.txt
├── 03-nginx-stable-alpine-pull.txt
├── 04-images-with-digests.txt
├── 05-image-platform-comparison.txt
├── 06-image-configuration.txt
├── 07-image-history-alpine.txt
├── 08-image-history-stable.txt
├── 09-container-image-ids.txt
├── 10-removal-conflict.txt
└── image-identity-report.md
```

Suggested captures:

```bash
docker image ls --digests nginx > 04-images-with-digests.txt
docker image history nginx:alpine > 07-image-history-alpine.txt
docker inspect --format \
  'container={{.Name}} container-id={{.Id}} image-id={{.Image}}' \
  nginx-alpine-a nginx-alpine-b > 09-container-image-ids.txt
```

Do not save registry passwords, authentication tokens, or unrelated sensitive metadata.

---

## Official References

- [What is an image?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)
- [Docker image pull](https://docs.docker.com/reference/cli/docker/image/pull/)
- [Docker image list](https://docs.docker.com/reference/cli/docker/image/ls/)
- [Docker image inspect](https://docs.docker.com/reference/cli/docker/image/inspect/)
- [Docker image history](https://docs.docker.com/reference/cli/docker/image/history/)
- [Docker image remove](https://docs.docker.com/reference/cli/docker/image/rm/)

---

## Next Lesson

**Lesson 07 — Container Commands, `CMD`, and `ENTRYPOINT`**

You now understand the image as filesystem content plus metadata. Next, you will focus on the metadata that decides which process starts.

You will learn:

- Main container process
- Default image command
- Runtime command override
- Arguments
- `CMD`
- `ENTRYPOINT`
- Why a container exits when its main process exits
- Child processes at a beginner-friendly level
- An introduction to PID 1 behavior

Commands will include:

```bash
docker run IMAGE COMMAND
docker image inspect IMAGE
docker top CONTAINER
docker exec CONTAINER COMMAND
```

---

## Completion Check

Before moving forward, explain without reading:

```text
Registry, namespace, repository, and tag
What nginx expands to
Why latest is not a guarantee
Tag versus digest
Image ID versus digest
Image ID versus container ID
Read-only image layers
Why tags can share image content
Why pulling does not update running containers
Why image removal may only untag
Why a container can block image removal
```

If you can prove these using pull, list, inspect, history, and container evidence, you are ready to study container commands.

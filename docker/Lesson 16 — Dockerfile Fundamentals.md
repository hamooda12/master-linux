# Chapter 07 — Docker for DevOps Engineers

# Lesson 16 — Dockerfile Fundamentals

> **Lesson goal:** Read and write a Dockerfile confidently, understand what each core instruction changes, and build a small non-root web image from start to finish.

---

## Overview

Until now, you have mostly used images created by other people:

```bash
docker run alpine:3.22
docker run nginx:alpine
```

Real DevOps work also requires building images for your own applications.

A **Dockerfile** is a text file containing ordered instructions that tell Docker how to construct an image.

Example:

```dockerfile
FROM alpine:3.22
WORKDIR /app
COPY . .
RUN chmod +x start.sh
ENV APP_ENV=production
EXPOSE 8080
USER 10001:10001
ENTRYPOINT ["./start.sh"]
CMD ["--port", "8080"]
```

At a beginner level, think of the process like this:

```text
Dockerfile + application files
              |
              v
         docker build
              |
              v
            image
              |
              v
          docker run
              |
              v
           container
```

This lesson explains:

- The purpose and syntax of a Dockerfile
- `FROM`
- `WORKDIR`
- `COPY`
- `RUN`
- `ENV`
- `EXPOSE`
- `USER`
- `CMD`
- `ENTRYPOINT`
- `.dockerignore`
- Build-time versus runtime behavior
- Introductory layer effects
- A complete static-site image lab

Build context, cache keys, multi-stage builds, optimization, and advanced image security have dedicated later lessons. Here, the goal is to establish the correct foundation.

---

## Learning Objectives

By the end of this lesson, you should be able to:

1. Explain what a Dockerfile is and what it produces.
2. Distinguish build time from container runtime.
3. Explain each core Dockerfile instruction in plain language.
4. Predict whether an instruction changes filesystem content or image configuration.
5. Choose safe base-image references.
6. Set a predictable working directory.
7. Copy only required application files.
8. Use `RUN` for build-time preparation.
9. Define non-sensitive runtime defaults with `ENV`.
10. Explain why `EXPOSE` does not publish a host port.
11. Configure a non-root runtime identity with `USER`.
12. Explain and combine `ENTRYPOINT` and `CMD`.
13. Use `.dockerignore` to exclude irrelevant or sensitive files.
14. Build, run, test, inspect, and clean up a custom image.

---

## Prerequisites

You should already understand:

- Images versus containers
- Image tags and digests
- Container writable layers
- Port publishing with `-p`
- Environment variables
- Secrets fundamentals
- Foreground main processes
- `docker run`, `docker ps`, `docker logs`, and `docker inspect`

Verify Docker:

```bash
docker version
```

Verify the Alpine image can be pulled or is already present:

```bash
docker image ls alpine:3.22
```

The lab uses only fake, non-sensitive content.

---

## Part 1 — What a Dockerfile Is

### 1. Definition

A Dockerfile is a plain-text recipe for building an image.

The conventional filename is:

```text
Dockerfile
```

with a capital `D` and no file extension.

### 2. Instructions Run in Order

Docker reads instructions from top to bottom:

```dockerfile
FROM alpine:3.22
WORKDIR /app
COPY . .
RUN chmod +x start.sh
CMD ["./start.sh"]
```

Each step receives the result of the earlier steps.

For example, `RUN chmod +x start.sh` works only after `start.sh` has been copied to the location where the command expects it.

### 3. Declarative Recipe, Not a Running Container Script

The Dockerfile describes an image build. It is not re-executed every time a container starts.

```text
Dockerfile instructions
      run during image build
                |
                v
              image
                |
                v
CMD/ENTRYPOINT become runtime defaults
```

Some instructions perform build work. Others store image configuration that affects future containers.

### 4. Comments

A comment begins with `#`:

```dockerfile
# Use the approved Alpine release
FROM alpine:3.22
```

Use comments to explain important decisions, not to repeat obvious syntax.

Useful:

```dockerfile
# Port 8080 allows the service to run without root privileges
EXPOSE 8080
```

Low-value:

```dockerfile
# Expose port 8080
EXPOSE 8080
```

### 5. Line Continuation

A backslash can continue one instruction across lines:

```dockerfile
RUN command-one \
    && command-two \
    && command-three
```

This remains one `RUN` instruction.

Do not confuse Dockerfile continuation with several separate Dockerfile instructions.

---

## Part 2 — Build Time Versus Runtime

### 6. Build Time

Build time is when Docker turns the Dockerfile and build files into an image.

Command:

```bash
docker build -t techcorp-site:1.0 .
```

Typical build-time activities:

- Selecting a base image
- Creating directories
- Copying source files
- Installing packages
- Compiling code
- Setting image metadata and defaults

### 7. Runtime

Runtime begins when Docker creates and starts a container from the image.

Command:

```bash
docker run techcorp-site:1.0
```

Typical runtime activities:

- Starting the main application process
- Reading runtime environment configuration
- Listening on a port
- Reading mounted files and volumes
- Handling requests
- Writing temporary or persistent data

### 8. The Key Separation

```text
Build time
  create reusable application artifact

Runtime
  run that artifact with environment-specific configuration
```

Do not start the long-running production service in a `RUN` instruction. `RUN` operates during image construction. `CMD` and `ENTRYPOINT` define what future containers start.

### 9. Instruction Classification

| Instruction | Primary build-time action | Main image result | Direct runtime role |
|---|---|---|---|
| `FROM` | Select base | Starts image stage from base content/config | Base userspace and defaults are inherited |
| `WORKDIR` | Set following working location | Stores working-directory config | Container starts commands relative to it |
| `COPY` | Add files | Files become image content | Files are available to containers |
| `RUN` | Execute build command | Resulting filesystem changes become image content | Command itself does not rerun automatically |
| `ENV` | Set value for later build steps | Stores environment default | Container inherits it unless overridden |
| `EXPOSE` | Record intended port | Stores port metadata | Does not publish a host port |
| `USER` | Select user for following steps | Stores default runtime identity | Main process runs as that user |
| `CMD` | Store default command/arguments | One effective default | Used when container starts unless overridden |
| `ENTRYPOINT` | Store main executable | One effective entrypoint | Runs as container's main executable |

---

## Part 3 — Introductory Image-Layer Model

### 10. What Is a Layer Here?

An image is built from immutable content layers plus image configuration.

Introductory view:

```text
Image configuration
  working directory, environment, user, command, entrypoint, exposed ports

Filesystem layers
  base files, copied files, installed packages, generated files
```

### 11. Filesystem-Changing Instructions

`COPY` and `RUN` commonly create filesystem changes that become content layers:

```dockerfile
COPY site/ /site/
RUN chmod -R a+rX /site
```

### 12. Metadata-Oriented Instructions

Instructions such as these mainly update image configuration:

```dockerfile
ENV APP_ENV=production
EXPOSE 8080
USER 10001:10001
CMD ["app"]
```

Build output or image history may show them as build steps, but they do not necessarily add a normal filesystem-content layer.

### 13. Do Not Use “One Instruction Equals One File Layer” as an Absolute Rule

Modern builders and image formats distinguish filesystem changes from configuration/history records. For this lesson, use this practical rule:

```text
COPY/RUN -> usually create filesystem content changes
ENV/EXPOSE/USER/CMD/ENTRYPOINT -> mainly image configuration
```

Layer internals and cache behavior are taught in Lessons 17 and 18.

---

## Part 4 — `FROM`

### 14. Purpose

`FROM` selects the starting image for a build stage.

```dockerfile
FROM alpine:3.22
```

The new image starts with Alpine's filesystem and image configuration.

### 15. Why a Base Image Is Useful

Without a base, you would need to provide every required userspace file yourself.

An Alpine base provides items such as:

```text
/bin/sh
/bin/busybox
/etc
/lib
package-management metadata and tools
```

It does **not** provide a separate kernel. Containers use the host kernel.

### 16. Reference Format

Common form:

```dockerfile
FROM repository:tag
```

Example:

```dockerfile
FROM alpine:3.22
```

More reproducible publication pinning may use a digest:

```dockerfile
FROM alpine:3.22@sha256:<approved-digest>
```

The digest must be real and appropriate for the intended image/manifest. Do not type the placeholder literally.

### 17. Build-Time Behavior

Docker resolves the base reference and makes its content/configuration the starting point for the stage. It may pull missing content from a registry.

### 18. Resulting Image Effect

The final image inherits the base image's:

- Filesystem content
- Default environment values
- Working directory if defined
- User if defined
- Command and entrypoint unless later replaced
- Other image metadata

Later Dockerfile instructions can add to or override parts of that state.

### 19. Introductory Layer Effect

`FROM` starts the stage using the base image's existing layers. It does not normally copy all base files into one brand-new duplicate layer.

```text
base layers
    |
    v
your new image layers
```

### 20. Common Mistakes

- Using `latest` and assuming it never changes
- Selecting an untrusted base
- Choosing a large general-purpose base without need
- Assuming a small image is automatically secure
- Forgetting that the base may define `USER`, `WORKDIR`, `ENTRYPOINT`, or `CMD`
- Using an unsupported or unpatched version

### 21. Production Recommendation

- Use trusted, maintained bases.
- Choose an explicit version tag according to team policy.
- Consider digest pinning where strict reproducibility is required.
- Regularly rebuild to receive approved security updates.
- Scan the final image, not only your application files.
- Understand the base defaults before overriding them.

### 22. Minimal Example

```dockerfile
FROM alpine:3.22
CMD ["cat", "/etc/alpine-release"]
```

The `cat` program and `/etc/alpine-release` file come from the Alpine base.

---

## Part 5 — `WORKDIR`

### 23. Purpose

`WORKDIR` sets the working directory for instructions that follow and for the default runtime process.

```dockerfile
WORKDIR /app
```

### 24. Think of It as a Persistent `cd`

In a normal shell:

```bash
cd /app
```

changes the current shell location.

In a Dockerfile:

```dockerfile
WORKDIR /app
```

affects later instructions such as:

- `RUN`
- `COPY` and `ADD` destinations when relative
- `CMD`
- `ENTRYPOINT`

It also becomes the configured working directory of containers unless overridden.

### 25. Directory Creation

If the directory does not exist, the builder creates it:

```dockerfile
FROM alpine:3.22
WORKDIR /opt/techcorp/app
```

You do not need a separate `RUN mkdir -p /opt/techcorp/app` merely to establish the working directory.

### 26. Relative Paths After `WORKDIR`

```dockerfile
WORKDIR /app
COPY start.sh .
RUN chmod +x start.sh
CMD ["./start.sh"]
```

Interpretation:

```text
COPY destination .  -> /app
RUN working path     -> /app
CMD relative path    -> /app/start.sh at runtime
```

### 27. Multiple `WORKDIR` Instructions

Relative values build on the current working directory:

```dockerfile
WORKDIR /opt
WORKDIR techcorp
WORKDIR app
```

Final path:

```text
/opt/techcorp/app
```

Absolute paths are clearer:

```dockerfile
WORKDIR /opt/techcorp/app
```

### 28. Build-Time Behavior

The builder sets the working directory for later instructions and ensures the path exists.

### 29. Resulting Image Effect

The image records the configured working directory. Future containers use it unless another runtime option overrides it.

Inspect it:

```bash
docker image inspect \
  --format '{{.Config.WorkingDir}}' \
  IMAGE
```

### 30. Introductory Layer Effect

`WORKDIR` is primarily a configuration instruction, although the path is made available as a directory. Do not use it to reason about application data size; copied and generated content usually dominates filesystem layers.

### 31. Common Mistakes

- Using `RUN cd /app` and expecting the next `RUN` to remain there
- Using ambiguous relative paths
- Forgetting inherited `WORKDIR` from a base image
- Copying to `.` without knowing the current `WORKDIR`
- Using a working directory that the runtime user cannot access

This does **not** persist across instructions:

```dockerfile
RUN cd /app
RUN pwd
```

Each `RUN` uses a new temporary build container based on the previous result. Shell process state such as `cd` does not carry forward.

### 32. Production Recommendation

- Use one clear absolute application path.
- Set it explicitly even if the base currently has a suitable default.
- Ensure the runtime user can read/execute required content and write only where needed.
- Prefer `WORKDIR` over repeated `cd` commands.

---

## Part 6 — `COPY`

### 33. Purpose

`COPY` copies files or directories from the build context into the image.

```dockerfile
COPY index.html /site/index.html
```

### 34. Source and Destination

```dockerfile
COPY SOURCE DESTINATION
```

Example:

```dockerfile
WORKDIR /app
COPY start.sh .
```

Source:

```text
start.sh in the build context
```

Destination:

```text
/app/start.sh in the image
```

### 35. Copy a Directory's Contents

```dockerfile
COPY site/ /site/
```

This copies the contents selected from the context path `site/` to `/site/` in the image.

Exact source semantics, context boundaries, wildcards, and advanced flags are covered with build context in Lesson 17.

### 36. `COPY . .`

Common form:

```dockerfile
WORKDIR /app
COPY . .
```

It means:

```text
copy selected content from current build context
into current image working directory
```

This can be convenient, but it can accidentally include:

- Git metadata
- Local logs
- Test artifacts
- Editor files
- Build outputs
- `.env` files
- Private keys
- Large dependency folders

Use a narrow copy and `.dockerignore`.

### 37. Build-Time Behavior

The builder reads eligible source files from the build context and adds them at the destination in the image filesystem.

`COPY` cannot normally read an arbitrary path outside the build context simply because you write `../`.

### 38. Resulting Image Effect

Copied files become immutable image content available to every future container.

At runtime, a bind mount or volume over the same destination can hide the image-provided content while the mount exists.

### 39. Introductory Layer Effect

`COPY` creates a filesystem change. The copied bytes contribute to image content and can affect cache reuse.

```text
earlier image state
       + copied files
       = new filesystem layer/result
```

Deleting copied content in a later instruction may not remove it from earlier layer data.

### 40. Ownership

Files normally arrive with build-defined ownership, commonly root ownership unless changed or copied with an ownership option.

Example:

```dockerfile
COPY --chown=10001:10001 app/ /app/
```

This sets ownership during the copy.

Numeric IDs avoid depending on name resolution during the copy, but production images should still have a deliberate identity design.

### 41. Common Mistakes

- Copying the whole project unnecessarily
- Copying secrets
- Copying local dependency or cache directories
- Assuming source paths are relative to `WORKDIR` rather than the build context
- Assuming destination paths are always host paths
- Forgetting runtime-user permissions
- Copying files before a directory is arranged as intended

### 42. Production Recommendation

- Copy only required files.
- Keep secrets outside the build context.
- Maintain a reviewed `.dockerignore`.
- Set appropriate ownership during copy where practical.
- Separate dependency metadata from rapidly changing source when later optimizing cache.
- Inspect the final image content.

---

## Part 7 — `RUN`

### 43. Purpose

`RUN` executes a command while the image is being built.

```dockerfile
RUN apk add --no-cache curl
```

The command runs in a temporary build container. Its resulting filesystem changes are saved into the image build result.

### 44. What `RUN` Is Used For

Common uses:

- Installing packages
- Compiling applications
- Creating directories and files
- Adjusting permissions
- Removing build leftovers in the same build step
- Running build-time validation

### 45. `RUN` Does Not Define the Container's Main Process

Incorrect mental model:

```dockerfile
RUN httpd -f -p 8080
```

This attempts to start the service during the build. A long-running foreground service can make the build hang, and even if it exits, it will not automatically become the runtime process.

Correct separation:

```dockerfile
RUN command-that-prepares-files
CMD ["httpd", "-f", "-p", "8080"]
```

### 46. Shell Form

```dockerfile
RUN echo "hello" > /app/message.txt
```

Docker uses a command shell for the instruction. Shell features such as redirection and `&&` work.

### 47. Exec Form

```dockerfile
RUN ["executable", "argument-one", "argument-two"]
```

Exec form does not automatically provide normal shell expansion, redirection, or pipelines.

Most package-install examples use shell form because they need `&&`, variables, or continuation.

### 48. Combine Related Package Operations

Alpine example:

```dockerfile
RUN apk add --no-cache curl ca-certificates
```

Debian/Ubuntu-style example:

```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*
```

Do not blindly use one distribution's package manager in another base image.

### 49. Why Cleanup Belongs in the Same `RUN`

Unsafe size pattern:

```dockerfile
RUN create-large-temporary-file
RUN use-large-temporary-file
RUN rm large-temporary-file
```

The file may remain in an earlier layer.

Better pattern:

```dockerfile
RUN create-large-temporary-file \
    && use-large-temporary-file \
    && rm large-temporary-file
```

The temporary file does not need to be committed into a completed intermediate filesystem layer between separate `RUN` instructions.

### 50. Build-Time Behavior

Docker creates a temporary container-like build environment from the current image state, runs the command, and captures resulting filesystem changes when successful.

### 51. Resulting Image Effect

Installed packages, generated files, deletions, and permission changes become part of the image result. The build process itself is not left running.

### 52. Introductory Layer Effect

`RUN` normally produces a filesystem difference layer when it changes files.

Its command also appears in image build history. Never place a secret directly in the instruction.

### 53. Common Mistakes

- Starting the application in `RUN`
- Using the wrong package manager
- Separating package index update from installation carelessly
- Leaving caches and temporary files
- Using `RUN cd` to affect later instructions
- Downloading unverified artifacts
- Running unpinned remote scripts blindly
- Supplying secrets through the command text
- Creating many unnecessary build steps without understanding the result

### 54. Production Recommendation

- Make build commands deterministic and non-interactive.
- Verify downloaded artifacts.
- Install only runtime requirements.
- Clean temporary build data in the same `RUN` where appropriate.
- Never embed credentials in command text.
- Use build secret mounts for temporary build authentication.
- Test that build commands fail clearly when preparation fails.

---

## Part 8 — `ENV`

### 55. Purpose

`ENV` defines an environment value in the image configuration.

```dockerfile
ENV APP_ENV=production
```

### 56. Build and Runtime Availability

The value is available to later build instructions:

```dockerfile
ENV APP_HOME=/opt/techcorp
RUN mkdir -p "$APP_HOME"
```

It is also inherited by containers:

```bash
docker run --rm IMAGE printenv APP_HOME
```

### 57. Runtime Override

Image default:

```dockerfile
ENV LOG_LEVEL=info
```

Runtime override:

```bash
docker run -e LOG_LEVEL=debug IMAGE
```

The container receives `debug` for that variable.

### 58. Build-Time Behavior

The builder stores the variable and makes it available to later Dockerfile instructions where expansion is supported.

### 59. Resulting Image Effect

The value persists in image configuration and becomes a default for future containers.

Inspect it:

```bash
docker image inspect \
  --format '{{json .Config.Env}}' \
  IMAGE
```

### 60. Introductory Layer Effect

`ENV` mainly changes image configuration rather than adding application file content. The value is still inspectable metadata and must not contain a secret.

### 61. Common Mistakes

- Putting passwords or tokens in `ENV`
- Assuming the value is encrypted
- Hard-coding environment-specific endpoints
- Using unclear names
- Expecting Docker to make the application use the variable automatically
- Forgetting runtime overrides

### 62. Production Recommendation

- Use `ENV` for safe, portable defaults.
- Supply environment-specific values at deployment time.
- Document supported variables.
- Validate required values in the application.
- Keep all secrets out of `ENV`.

---

## Part 9 — `EXPOSE`

### 63. Purpose

`EXPOSE` documents the network port the image's application intends to listen on.

```dockerfile
EXPOSE 8080
```

### 64. What It Does Not Do

`EXPOSE 8080` does **not**:

- Start the application
- Make the application listen on 8080
- Open the host firewall
- Publish host port 8080
- Create a `-p` mapping

The application must listen, and the operator must publish the port when host access is needed.

### 65. Runtime Port Publishing

Dockerfile:

```dockerfile
EXPOSE 8080
```

Runtime:

```bash
docker run -p 8080:8080 IMAGE
```

Meaning:

```text
host port 8080 -> container port 8080
```

### 66. Protocol

TCP is assumed when no protocol is specified:

```dockerfile
EXPOSE 8080
```

Explicit examples:

```dockerfile
EXPOSE 8080/tcp
EXPOSE 5514/udp
```

This remains image metadata, not a firewall rule.

### 67. Build-Time Behavior

The builder records the declared port/protocol in image configuration.

### 68. Resulting Image Effect

Tools and users can inspect the intended port. `docker run -P` can use exposed-port metadata to publish to automatically selected host ports.

Inspect:

```bash
docker image inspect \
  --format '{{json .Config.ExposedPorts}}' \
  IMAGE
```

### 69. Introductory Layer Effect

`EXPOSE` is image metadata. It does not add a network service or filesystem content layer.

### 70. Common Mistakes

- Believing `EXPOSE` publishes the port
- Exposing a port where no process listens
- Publishing the wrong container port
- Binding an application only to `127.0.0.1` inside the container when external container-network access is expected
- Exposing every possible port without purpose

### 71. Production Recommendation

- Document only the ports the image is designed to serve.
- Prefer a non-privileged application port such as 8080 for non-root processes.
- Publish only required ports at deployment time.
- Verify the process listens on the expected container address and port.

---

## Part 10 — `USER`

### 72. Purpose

`USER` defines the user and optional group for subsequent instructions and the default runtime process.

```dockerfile
USER 10001:10001
```

### 73. Why Non-Root Matters

Many base images default to root. If the application does not need root, run it as a dedicated unprivileged identity.

```text
root process in container
  has more container privileges

non-root process
  has fewer permissions by default
```

This is one security layer, not complete isolation.

### 74. Create a Named User

Alpine example:

```dockerfile
RUN addgroup -S appgroup \
    && adduser -S -D -H -u 10001 -G appgroup appuser

USER appuser
```

Or use a numeric identity:

```dockerfile
USER 10001:10001
```

A numeric identity can run even without a matching `/etc/passwd` name, but some applications expect a home directory or user record. Choose deliberately.

### 75. Order Matters

Package installation commo

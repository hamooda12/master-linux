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

Package installation commonly needs root:

```dockerfile
FROM alpine:3.22
RUN apk add --no-cache curl
USER 10001:10001
```

This fails if reversed and the unprivileged user cannot use the package manager:

```dockerfile
USER 10001:10001
RUN apk add --no-cache curl
```

Do privileged build preparation first, set ownership, and switch to the runtime user near the end.

### 76. Runtime Filesystem Permissions

The runtime user must be able to:

- Read application files
- Execute required programs
- Access mounted configuration and secret files
- Write only to directories that require writes

Example:

```dockerfile
RUN mkdir -p /var/lib/myapp \
    && chown 10001:10001 /var/lib/myapp
USER 10001:10001
```

Do not make the entire filesystem world-writable to avoid permission design.

### 77. Build-Time Behavior

After `USER`, later `RUN` instructions execute as that identity unless another `USER` changes it. The builder records the final user setting.

### 78. Resulting Image Effect

Containers start their main process as the configured user unless a runtime option overrides it.

Inspect:

```bash
docker image inspect \
  --format '{{.Config.User}}' \
  IMAGE
```

Verify at runtime:

```bash
docker run --rm IMAGE id
```

This overrides the image's normal `CMD` with `id` only when the image does not have an entrypoint that changes argument behavior. With an entrypoint, use an appropriate test or `--entrypoint`.

### 79. Introductory Layer Effect

`USER` mainly updates image configuration. Creating the user and changing ownership with `RUN` creates filesystem changes.

### 80. Common Mistakes

- Leaving the application as root without need
- Switching to non-root before privileged build steps are complete
- Forgetting ownership of copied files or writable directories
- Using `chmod 777` as a universal fix
- Assuming a username exists in every base image
- Using a privileged port below 1024 without a deliberate design

### 81. Production Recommendation

- Run as a dedicated non-root identity.
- Use a stable UID/GID strategy compatible with mounted storage.
- Grant writes only to required paths.
- Set ownership deliberately.
- Test the image under its final runtime user.
- Avoid runtime user override back to root unless there is an audited reason.

---

## Part 11 — `CMD`

### 82. Purpose

`CMD` supplies the default command or default arguments for containers created from the image.

```dockerfile
CMD ["httpd", "-f", "-p", "8080"]
```

### 83. Exec Form

Preferred for direct process execution:

```dockerfile
CMD ["executable", "argument-one", "argument-two"]
```

Example:

```dockerfile
CMD ["sleep", "3600"]
```

This is a JSON array. Each element must use double quotes.

Incorrect JSON:

```dockerfile
CMD ['sleep', '3600']
```

### 84. Shell Form

```dockerfile
CMD sleep 3600
```

This uses a command shell. Shell form may be useful when deliberate shell expansion is required, but it introduces an extra shell process and can affect signal handling.

Prefer exec form for the main process unless you intentionally need a shell.

### 85. Runtime Override

Dockerfile:

```dockerfile
CMD ["sleep", "3600"]
```

Default:

```bash
docker run IMAGE
```

Override:

```bash
docker run IMAGE sleep 10
```

The arguments after the image replace `CMD` when there is no fixed entrypoint.

### 86. Only One Effective `CMD`

```dockerfile
CMD ["echo", "first"]
CMD ["echo", "second"]
```

Only the last `CMD` is effective.

Dockerfile instructions are not a sequence of runtime commands. `CMD` stores one default.

### 87. Build-Time Behavior

The command does not run during the build. The builder stores it in image configuration.

### 88. Resulting Image Effect

Future containers use it as their default command or as default arguments for an `ENTRYPOINT`.

Inspect:

```bash
docker image inspect \
  --format '{{json .Config.Cmd}}' \
  IMAGE
```

### 89. Introductory Layer Effect

`CMD` changes image configuration. It does not install the executable or create its files. The referenced program must already exist in the image.

### 90. Common Mistakes

- Using several `CMD` instructions expecting all to run
- Using single quotes in JSON exec form
- Referencing a missing executable
- Starting a daemon that immediately forks to the background
- Using shell form without understanding signals
- Expecting `CMD` to run during build
- Confusing runtime arguments with Docker options placed before the image name

### 91. Production Recommendation

- Use one foreground main process.
- Prefer exec form.
- Make the default useful for normal operation.
- Allow deliberate runtime overrides where appropriate.
- Ensure the process handles termination signals correctly.

---

## Part 12 — `ENTRYPOINT`

### 92. Purpose

`ENTRYPOINT` defines the image's main executable.

```dockerfile
ENTRYPOINT ["httpd"]
```

With `CMD`, it can separate a stable executable from default arguments:

```dockerfile
ENTRYPOINT ["httpd"]
CMD ["-f", "-p", "8080", "-h", "/site"]
```

Default runtime command becomes:

```text
httpd -f -p 8080 -h /site
```

### 93. Runtime Arguments Append to Exec-Form `ENTRYPOINT`

Dockerfile:

```dockerfile
ENTRYPOINT ["httpd"]
CMD ["-f", "-p", "8080", "-h", "/site"]
```

Default:

```bash
docker run IMAGE
```

Effective process:

```text
httpd -f -p 8080 -h /site
```

Custom arguments:

```bash
docker run IMAGE -f -p 9090 -h /site
```

Effective process:

```text
httpd -f -p 9090 -h /site
```

The supplied arguments replace `CMD` and append to the entrypoint.

### 94. Override the Entrypoint

For debugging:

```bash
docker run --rm \
  --entrypoint sh \
  IMAGE
```

This replaces the image entrypoint for that container.

Use runtime overrides carefully. A successful debug shell does not prove the normal entrypoint works.

### 95. Exec Form Versus Shell Form

Preferred:

```dockerfile
ENTRYPOINT ["/app/start"]
```

Shell form:

```dockerfile
ENTRYPOINT /app/start
```

Shell form runs through a shell and can make runtime arguments and signal delivery behave unexpectedly. Prefer exec form.

### 96. Entrypoint Scripts

An entrypoint script can perform small startup tasks:

```sh
#!/bin/sh
set -eu

test -n "${APP_ENV:-}"
exec /app/server "$@"
```

The final `exec` replaces the shell with the application so the application becomes the main process and receives signals directly.

Avoid turning the script into a complex configuration-management system.

### 97. Build-Time Behavior

The entrypoint does not run during the build. Docker stores it in image configuration.

### 98. Resulting Image Effect

Containers use it as their main executable unless overridden with `--entrypoint`.

Inspect:

```bash
docker image inspect \
  --format '{{json .Config.Entrypoint}}' \
  IMAGE
```

### 99. Introductory Layer Effect

`ENTRYPOINT` changes image configuration. If it refers to a script, that script must be added separately with `COPY` and have suitable permissions.

### 100. Common Mistakes

- Using shell form accidentally
- Forgetting `exec` in a wrapper script
- Writing a large fragile startup script
- Expecting arguments after the image to replace the entrypoint
- Referencing a missing or non-executable script
- Using Windows-style line endings in a Linux shell script
- Defining multiple entrypoints and expecting all to apply

### 101. Production Recommendation

- Use exec form.
- Keep the entrypoint stable and focused.
- Put overridable default arguments in `CMD`.
- End wrapper scripts with `exec` for the main process.
- Test stop behavior and signals.
- Document supported runtime arguments.

---

## Part 13 — `CMD` and `ENTRYPOINT` Together

### 102. Four Common Models

| Dockerfile configuration | Arguments after image | Effective behavior |
|---|---|---|
| `CMD ["app", "--mode", "normal"]` | None | Runs default command |
| `CMD ["app", "--mode", "normal"]` | `other-command` | Replaces `CMD` |
| `ENTRYPOINT ["app"]` | `--mode debug` | Appends arguments to entrypoint |
| `ENTRYPOINT ["app"]` + `CMD ["--mode", "normal"]` | None | Entrypoint plus default arguments |
| `ENTRYPOINT ["app"]` + `CMD ["--mode", "normal"]` | `--mode debug` | Entrypoint plus replacement arguments |

### 103. Decision Rule

Use `CMD` alone when the whole command should be easy to replace:

```dockerfile
CMD ["python", "app.py"]
```

Use `ENTRYPOINT` plus `CMD` when the image behaves like a dedicated executable:

```dockerfile
ENTRYPOINT ["myapp"]
CMD ["serve", "--port", "8080"]
```

### 104. Visual Model

```text
ENTRYPOINT = stable executable
CMD        = default arguments

docker run IMAGE custom arguments
                       |
                       v
ENTRYPOINT + custom arguments
```

If no entrypoint is configured:

```text
docker run IMAGE custom command
                       |
                       v
custom command replaces CMD
```

### 105. Inspect Both

```bash
docker image inspect \
  --format 'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

Do not guess how a base image starts. Inspect it.

---

## Part 14 — `.dockerignore`

### 106. What It Is

`.dockerignore` is not a Dockerfile instruction. It is a separate file that tells the builder which paths should be excluded from the build context.

Project:

```text
my-project/
├── Dockerfile
├── .dockerignore
├── site/
│   └── index.html
├── notes/
├── debug.log
└── production.env
```

Example `.dockerignore`:

```dockerignore
.git
*.log
notes/
*.env
secrets/
```

### 107. Why It Matters

It can:

- Reduce build-context size
- Avoid sending irrelevant files to the builder
- Prevent accidental `COPY . .` inclusion
- Reduce unnecessary cache invalidation
- Add a safety barrier against secret files

### 108. Security Boundary

`.dockerignore` helps prevent accidents, but it is not a secret manager.

Best design:

```text
real secret is outside project/build context
             +
.dockerignore blocks common secret patterns
             +
image scan verifies result
```

### 109. Build-Time Behavior

The builder evaluates ignore patterns while preparing/reading the build context. Excluded paths are unavailable to normal `COPY` instructions from that context.

### 110. Resulting Image Effect

Ignored files do not enter the image through ordinary context copying. They also do not contribute their bytes to the transmitted/processed context.

### 111. Introductory Layer Effect

`.dockerignore` does not create an image layer. It controls which source paths are available before `COPY` creates layers.

### 112. Common Mistakes

- Confusing `.dockerignore` with `.gitignore`
- Assuming Git-ignored files are automatically Docker-ignored
- Ignoring a file the build actually needs
- Using broad patterns without testing
- Keeping real secrets in the context and relying only on ignore rules
- Forgetting nested dependency or output directories

### 113. Production Recommendation

- Commit and review `.dockerignore` with the Dockerfile.
- Exclude VCS metadata, logs, local dependencies, build outputs, editor files, tests when unnecessary, and secret patterns.
- Keep real credentials outside the context entirely.
- Verify the final image contains only intended files.
- Revisit patterns as the project changes.

Build-context mechanics and pattern behavior are explored in Lesson 17.

---

## Part 15 — Instruction Order

### 114. Correctness Comes First

Instructions must appear after their prerequisites.

Incorrect:

```dockerfile
RUN chmod +x /app/start.sh
COPY start.sh /app/start.sh
```

Correct:

```dockerfile
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh
```

### 115. Privilege Order

Prepare files as root, then switch:

```dockerfile
RUN mkdir -p /app/data \
    && chown 10001:10001 /app/data
USER 10001:10001
```

### 116. Stable Default Order

A simple application Dockerfile often has this shape:

```dockerfile
FROM trusted-base:version

WORKDIR /app

COPY required-files ./
RUN build-or-permission-step

ENV SAFE_DEFAULT=value
EXPOSE 8080

USER nonroot

ENTRYPOINT ["application"]
CMD ["default", "arguments"]
```

This is a learning template, not a universal order. Dependency caching, multi-stage builds, and language-specific patterns may rearrange instructions.

---

## Part 16 — Full Dockerfile Reading Exercise

### 117. Example

```dockerfile
FROM alpine:3.22

WORKDIR /site

COPY --chown=10001:10001 site/ ./

RUN test -s index.html

ENV SITE_ENV=production

EXPOSE 8080

USER 10001:10001

ENTRYPOINT ["httpd"]
CMD ["-f", "-p", "8080", "-h", "/site"]
```

### 118. Read It in Plain English

1. Start with Alpine 3.22.
2. Use `/site` as the build and runtime working directory.
3. Copy the local `site/` content into `/site` and assign numeric ownership.
4. Fail the build if `index.html` is missing or empty.
5. Define `SITE_ENV=production` as a safe default.
6. Document that the service uses TCP port 8080.
7. Run later instructions and containers as UID/GID 10001.
8. Use BusyBox `httpd` as the main executable.
9. Supply foreground, port, and document-root arguments by default.

### 119. Build-Time Timeline

```text
resolve Alpine base
        |
set /site working directory
        |
copy site files
        |
validate index.html
        |
store environment, port, and user metadata
        |
store entrypoint and command metadata
        |
produce image
```

### 120. Runtime Timeline

```text
docker run IMAGE
        |
container uses /site
        |
process runs as 10001:10001
        |
effective command:
httpd -f -p 8080 -h /site
        |
server remains in foreground as main process
```

---

## Commands — Detailed Reference

### Build with a Name and Tag

```bash
docker build -t NAME:TAG .
```

The final `.` selects the current directory as the build context. Context details come next lesson.

### List the Built Image

```bash
docker image ls NAME:TAG
```

### Inspect Core Image Configuration

```bash
docker image inspect \
  --format 'workdir={{.Config.WorkingDir}} user={{.Config.User}} ports={{json .Config.ExposedPorts}} entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  NAME:TAG
```

### Run and Publish a Port

```bash
docker run -d \
  --name CONTAINER \
  -p HOST_PORT:CONTAINER_PORT \
  NAME:TAG
```

### Test HTTP

```bash
curl http://127.0.0.1:HOST_PORT/
```

### Inspect the Runtime User with an Entrypoint Override

```bash
docker run --rm \
  --entrypoint id \
  NAME:TAG
```

### Inspect Effective Container Configuration

```bash
docker inspect \
  --format 'user={{.Config.User}} entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  CONTAINER
```

### Remove Lab Container and Image

```bash
docker rm -f CONTAINER
docker image rm NAME:TAG
```

---

## Guided Hands-on Lab — Build a Non-Root Static Website

### Lab Goal

Build `techcorp-site:1.0`, run it as numeric user `10001:10001`, publish its port, verify its content, and inspect every important image default.

The site uses BusyBox `httpd`, already included in Alpine through BusyBox. No package installation is required.

### Step 1 — Create the Project

Host prompt:

```text
hamad@Hamooda:~$
```

Run:

```bash
mkdir -p "$HOME/dockerfile-fundamentals-lab/site"
cd "$HOME/dockerfile-fundamentals-lab"
```

Your location should now be:

```text
hamad@Hamooda:~/dockerfile-fundamentals-lab$
```

### Step 2 — Create the Website

Create `site/index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>TechCorp Docker Lab</title>
  </head>
  <body>
    <h1>TechCorp Site</h1>
    <p>This page came from a custom Docker image.</p>
  </body>
</html>
```

Create `site/health.txt`:

```text
healthy
```

### Step 3 — Create `.dockerignore`

Create `.dockerignore`:

```dockerignore
.git
*.log
*.env
secrets/
notes/
```

This lab needs only `Dockerfile`, `.dockerignore`, and `site/`.

### Step 4 — Create the Dockerfile

Create `Dockerfile`:

```dockerfile
FROM alpine:3.22

WORKDIR /site

COPY --chown=10001:10001 site/ ./

RUN test -s index.html \
    && test -s health.txt

ENV SITE_ENV=production

EXPOSE 8080

USER 10001:10001

ENTRYPOINT ["httpd"]
CMD ["-f", "-p", "8080", "-h", "/site"]
```

### Step 5 — Predict Before Building

Answer:

1. Which files will be added to the image?
2. Which command runs during the build?
3. Which command runs when a container starts?
4. Which user runs the server?
5. Does `EXPOSE 8080` publish host port 8080?

Expected reasoning:

```text
COPY adds index.html and health.txt.
RUN executes the two test commands during build.
ENTRYPOINT + CMD starts httpd at runtime.
UID/GID 10001 runs the server.
EXPOSE documents container port 8080; -p publishes it.
```

### Step 6 — Build the Image

From:

```text
hamad@Hamooda:~/dockerfile-fundamentals-lab$
```

Run:

```bash
docker build -t techcorp-site:1.0 .
```

Read the build output. You should see steps corresponding to the Dockerfile. Modern BuildKit output may group, cache, or display steps differently from older examples.

### Step 7 — Verify the Image Exists

```bash
docker image ls techcorp-site:1.0
```

Do not rely only on the successful build message. Verify the local image reference.

### Step 8 — Inspect Image Defaults

```bash
docker image inspect \
  --format 'workdir={{.Config.WorkingDir}} user={{.Config.User}} env={{json .Config.Env}} ports={{json .Config.ExposedPorts}} entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  techcorp-site:1.0
```

Confirm:

```text
workdir=/site
user=10001:10001
SITE_ENV=production appears in env
8080/tcp appears in exposed ports
entrypoint=["httpd"]
cmd=["-f","-p","8080","-h","/site"]
```

Other environment values inherited from Alpine may also appear.

### Step 9 — Verify Runtime Identity

Because the image has an entrypoint, override it for this diagnostic:

```bash
docker run --rm \
  --entrypoint id \
  techcorp-site:1.0
```

Expected shape:

```text
uid=10001 gid=10001
```

The numeric names may be displayed without friendly usernames because the lab did not create `/etc/passwd` entries for them.

### Step 10 — Run the Website

```bash
docker run -d \
  --name techcorp-site \
  -p 8080:8080 \
  techcorp-site:1.0
```

### Step 11 — Verify Container State

```bash
docker ps --filter name=techcorp-site
```

If it is absent, inspect stopped containers and logs:

```bash
docker ps -a --filter name=techcorp-site
docker logs techcorp-site
```

### Step 12 — Test the Website

```bash
curl http://127.0.0.1:8080/
```

Expected HTML includes:

```html
<h1>TechCorp Site</h1>
```

### Step 13 — Test the Health File

```bash
curl http://127.0.0.1:8080/health.txt
```

Expected:

```text
healthy
```

This is a static health resource for the lesson. Docker `HEALTHCHECK` is a separate feature taught later.

### Step 14 — Verify the Environment Default

```bash
docker exec techcorp-site printenv SITE_ENV
```

Expected:

```text
production
```

### Step 15 — Verify Working Directory

```bash
docker exec techcorp-site pwd
```

Expected:

```text
/site
```

### Step 16 — Verify the Running User

```bash
docker exec techcorp-site id
```

Expected shape:

```text
uid=10001 gid=10001
```

### Step 17 — Verify Port Metadata Versus Publishing

Inspect published ports:

```bash
docker port techcorp-site
```

Expected shape:

```text
8080/tcp -> 0.0.0.0:8080
```

IPv6 output may also appear depending on Docker host configuration.

The mapping exists because of:

```bash
-p 8080:8080
```

not merely because the Dockerfile contains:

```dockerfile
EXPOSE 8080
```

### Step 18 — Override `CMD` Arguments

Stop and remove the current container:

```bash
docker rm -f techcorp-site
```

Start the same entrypoint with replacement arguments on container port 9090:

```bash
docker run -d \
  --name techcorp-site \
  -p 9090:9090 \
  techcorp-site:1.0 \
  -f -p 9090 -h /site
```

Effective command:

```text
httpd -f -p 9090 -h /site
```

Test:

```bash
curl http://127.0.0.1:9090/health.txt
```

Expected:

```text
healthy
```

This proves runtime arguments replaced `CMD` but kept the exec-form `ENTRYPOINT`.

### Step 19 — Override the Image Environment Default

Recreate with a runtime value:

```bash
docker rm -f techcorp-site
```

```bash
docker run -d \
  --name techcorp-site \
  -p 8080:8080 \
  -e SITE_ENV=testing \
  techcorp-site:1.0
```

Verify:

```bash
docker exec techcorp-site printenv SITE_ENV
```

Expected:

```text
testing
```

The image remains unchanged. Only this container received the override.

### Step 20 — Inspect Files in the Image

```bash
docker exec techcorp-site \
  ls -ln /site
```

Confirm that the content exists and is associated with UID/GID 10001 as requested by `COPY --chown`.

### Step 21 — Cleanup

```bash
docker rm -f techcorp-site
```

Optional image removal:

```bash
docker image rm techcorp-site:1.0
```

Keep the project directory for Lesson 17, where you will modify and rebuild it while studying build context and cache output.

---

## Discovery Questions

### Question 1

Does Docker run every Dockerfile instruction whenever a container starts?

<details>
<summary>Answer</summary>

No. Dockerfile instructions are processed while building the image. At runtime, the stored image configuration and content are used; `CMD` and `ENTRYPOINT` provide startup defaults.

</details>

### Question 2

Why does `RUN cd /app` not set the working directory for the next `RUN`?

<details>
<summary>Answer</summary>

The shell created for one `RUN` exits when that instruction ends. Use `WORKDIR /app` to persist the configured working directory across later instructions.

</details>

### Question 3

If `COPY . .` appears after `WORKDIR /app`, where do the files go?

<details>
<summary>Answer</summary>

The source `.` refers to selected content from the build context. The relative destination `.` refers to the current image working directory, `/app`.

</details>

### Question 4

Does `RUN apk add nginx` install Nginx every time a container starts?

<details>
<summary>Answer</summary>

No. It installs Nginx during the image build. The resulting files become image content used by containers.

</details>

### Question 5

Does `EXPOSE 8080` make `http://localhost:8080` reachable?

<details>
<summary>Answer</summary>

No. The application must listen on the container port, and the operator must publish it, for example with `-p 8080:8080`.

</details>

### Question 6

Why is `USER 10001:10001` placed after privileged build preparation?

<details>
<summary>Answer</summary>

Package installation and ownership setup often require root during the build. After preparation, `USER` sets the safer non-root identity for remaining steps and runtime.

</details>

### Question 7

What happens when a Dockerfile contains two `CMD` instructions?

<details>
<summary>Answer</summary>

Only the last `CMD` is effective in the resulting stage.

</details>

### Question 8

With `ENTRYPOINT ["myapp"]` and `CMD ["serve"]`, what does `docker run IMAGE debug` execute?

<details>
<summary>Answer</summary>

The runtime argument replaces `CMD` and appends to the entrypoint, producing `myapp debug`.

</details>

### Question 9

Is `.dockerignore` the same file as `.gitignore`?

<details>
<summary>Answer</summary>

No. They are separate files used by different tools. A Git-ignored file is not automatically Docker-ignored.

</details>

### Question 10

Can `ENV API_TOKEN=...` safely hide a token in an image?

<details>
<summary>Answer</summary>

No. `ENV` persists in inspectable image configuration and must not contain secrets.

</details>

---

## Practical Scenario — Container Exits After a Successful Build

### Incident

TechCorp built `reports-api:1.0` successfully, but its container exits immediately.

Dockerfile excerpt:

```dockerfile
FROM alpine:3.22
WORKDIR /app
COPY reports-api /app/reports-api
RUN /app/reports-api --serve
```

### Junior Assumption

```text
The service started during RUN, so it should still be running in the image.
```

### Investigation

Inspect container state:

```bash
docker ps -a --filter ancestor=reports-api:1.0
```

Inspect image startup defaults:

```bash
docker image inspect \
  --format 'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  reports-api:1.0
```

Suppose both are empty or inherited inappropriately.

### Root Cause

`RUN` belongs to build time. It does not define a persistent runtime process. The image has no correct startup command for the application.

### Correct Design

```dockerfile
FROM alpine:3.22
WORKDIR /app
COPY --chown=10001:10001 reports-api /app/reports-api
RUN chmod 0555 /app/reports-api
USER 10001:10001
EXPOSE 8080
ENTRYPOINT ["/app/reports-api"]
CMD ["--serve", "--port", "8080"]
```

The exact binary, port, arguments, libraries, and user requirements must be verified for the real application.

### Verification

After rebuilding:

```bash
docker run -d \
  --name reports-api \
  -p 8080:8080 \
  reports-api:1.1
```

Then check:

```bash
docker ps --filter name=reports-api
docker logs reports-api
curl http://127.0.0.1:8080/health
```

### Lesson

```text
RUN prepares the image.
ENTRYPOINT/CMD start the container process.
```

---

## Troubleshooting Workflow

### Problem 1 — `COPY` Reports File Not Found

Check:

- Current host directory
- Build-context path at the end of `docker build`
- Source path relative to the context
- `.dockerignore` patterns
- Filename capitalization
- Whether the file exists before the build

Evidence:

```bash
pwd
find . -maxdepth 3 -type f -print
```

Do not solve the issue by using an unnecessarily huge parent directory as context.

### Problem 2 — `RUN` Command Not Found

Check:

- Does the base image contain the executable?
- Was it copied or installed before this step?
- Is the package manager correct for the distribution?
- Is the path correct?
- Is the script executable?
- Does it have Linux-compatible line endings and a valid shebang?

### Problem 3 — Container Exits Immediately

Check:

```bash
docker ps -a --filter name=CONTAINER
docker logs CONTAINER
docker inspect \
  --format 'exit={{.State.ExitCode}} error={{json .State.Error}} cmd={{json .Config.Cmd}} entrypoint={{json .Config.Entrypoint}}' \
  CONTAINER
```

Ask:

- Is the executable present?
- Does the main process stay in the foreground?
- Are arguments valid?
- Does the runtime user have permission?
- Is required configuration present?

### Problem 4 — Permission Denied at Runtime

Inspect identity and files:

```bash
docker run --rm --entrypoint id IMAGE
docker run --rm --entrypoint ls IMAGE -ln /app
```

Fix ownership and modes deliberately in the Dockerfile. Avoid `chmod -R 777`.

### Problem 5 — Published Port Does Not Respond

Check:

```bash
docker ps --filter name=CONTAINER
docker port CONTAINER
docker logs CONTAINER
```

Then verify:

- The application is running.
- It listens on the expected container port.
- The container port in `-p` is correct.
- It listens on an appropriate container address, not only loopback.
- Host firewall and routing are appropriate.

Remember: `EXPOSE` alone does not publish.

### Problem 6 — Runtime Arguments Behave Unexpectedly

Inspect:

```bash
docker image inspect \
  --format 'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

Determine whether the image uses:

- `CMD` only
- `ENTRYPOINT` only
- Both together
- Shell form
- Exec form

### Problem 7 — Secret Appears in the Image

1. Rotate or revoke it immediately.
2. Stop distributing the affected image.
3. Remove it from source and build context.
4. Correct `.dockerignore` and the Dockerfile.
5. Build a clean image.
6. Scan and verify.
7. Replace affected deployments.

Deleting it in a later layer is not enough.

---

## Common Mistakes

### Mistake 1 — Using `RUN` to Start the Runtime Service

`RUN` is build time. Use `CMD` or `ENTRYPOINT` for container startup.

### Mistake 2 — Assuming `EXPOSE` Publishes a Port

Use runtime `-p` or another deployment network mechanism.

### Mistake 3 — Copying the Entire Laptop Directory

Choose a focused build context and reviewed `.dockerignore`.

### Mistake 4 — Baking Secrets into the Image

Never use `COPY`, `ENV`, or command text to embed credentials.

### Mistake 5 — Leaving the Application as Root

Use a dedicated non-root identity unless the application has a reviewed requirement.

### Mistake 6 — Using `chmod 777`

Fix ownership and minimum permissions instead of granting everyone full access.

### Mistake 7 — Using Several `CMD` Instructions

Only the last one is effective.

### Mistake 8 — Using Single Quotes in Exec-Form JSON

JSON arrays require double quotes.

### Mistake 9 — Forgetting the Base Image's Defaults

Inspect inherited working directory, user, environment, command, and entrypoint.

### Mistake 10 — Expecting Shell Expansion in Exec Form

Exec form passes literal arguments. Use a deliberate shell only when required.

### Mistake 11 — Using a Backgrounding Application Mode

The main process should remain in the foreground so Docker can track its lifecycle.

### Mistake 12 — Assuming a Successful Build Means a Working Image

Run and test the image under realistic runtime settings.

---

## Best Practices

1. Use a trusted, maintained, explicitly versioned base image.
2. Set an explicit absolute `WORKDIR`.
3. Copy only required application content.
4. Keep secrets outside the build context and image.
5. Maintain a reviewed `.dockerignore`.
6. Use `RUN` only for build-time preparation.
7. Make build commands deterministic, non-interactive, and verifiable.
8. Clean temporary files in the same build step where practical.
9. Use `ENV` only for safe defaults.
10. Use `EXPOSE` to document intended container ports, not as a publishing mechanism.
11. Run the application as a dedicated non-root user.
12. Set ownership and permissions deliberately.
13. Prefer exec-form `ENTRYPOINT` and `CMD`.
14. Keep the main process in the foreground.
15. End wrapper scripts with `exec` when launching the main application.
16. Inspect image configuration after building.
17. Test startup, shutdown, ports, permissions, and configuration overrides.
18. Tag images deliberately and avoid relying blindly on mutable `latest`.
19. Scan the final image and rebuild regularly for base updates.

---

## Interview Questions

### 1. What is a Dockerfile?

A text recipe containing ordered instructions used to build a Docker image.

### 2. What is the difference between `RUN` and `CMD`?

`RUN` executes during image build and saves filesystem results. `CMD` stores a default used when a container starts.

### 3. What does `FROM` do?

It starts a build stage from a base image and its existing filesystem layers and configuration.

### 4. Why use `WORKDIR` instead of `RUN cd`?

`WORKDIR` persists as build/image configuration for later instructions and runtime. A shell's `cd` ends with that individual `RUN` process.

### 5. What does `COPY` do?

It copies eligible files from the build context into the image filesystem.

### 6. Does `EXPOSE` publish a host port?

No. It records intended container-port metadata. Runtime publishing uses `-p`, `-P`, or an orchestration mechanism.

### 7. Why use `USER`?

To run later build steps and the default container process as a deliberately selected, usually non-root identity.

### 8. What is the difference between `CMD` and `ENTRYPOINT`?

`ENTRYPOINT` defines the stable executable. `CMD` defines a replaceable default command or default arguments.

### 9. What happens to arguments after the image when exec-form `ENTRYPOINT` exists?

They replace `CMD` and are appended to the entrypoint.

### 10. Why prefer exec form for the main process?

It avoids an unnecessary shell and generally gives clearer argument and signal behavior.

### 11. What does `.dockerignore` do?

It excludes matching paths from the build context before normal `COPY` operations can use them.

### 12. Is `.dockerignore` a secret manager?

No. It is a context filter and safety barrier. Real secrets should remain outside the context and use appropriate secret mechanisms.

### 13. Why can deleting a large file in a later `RUN` fail to reduce image size?

The bytes may remain in an earlier immutable layer where the file was added.

### 14. How do you inspect an image's configured startup command?

```bash
docker image inspect \
  --format 'entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  IMAGE
```

### 15. Why build as root but run as non-root?

Some preparation needs elevated build permissions, while the runtime application usually needs fewer privileges. Switch after preparation and permission setup.

---

## Lesson Reflection

You should now be able to read this:

```dockerfile
FROM alpine:3.22
WORKDIR /site
COPY --chown=10001:10001 site/ ./
RUN test -s index.html
ENV SITE_ENV=production
EXPOSE 8080
USER 10001:10001
ENTRYPOINT ["httpd"]
CMD ["-f", "-p", "8080", "-h", "/site"]
```

as a complete story:

```text
Use Alpine
-> work in /site
-> copy and validate website content
-> provide a safe environment default
-> document port 8080
-> drop to a non-root identity
-> start httpd in the foreground with default arguments
```

The most important separation is:

```text
FROM, COPY, RUN, WORKDIR preparation -> build the image
ENTRYPOINT and CMD                    -> start future containers
ENV, EXPOSE, USER, WORKDIR            -> store runtime-relevant defaults
```

---

## Summary

- A Dockerfile is an ordered text recipe for building an image.
- Docker processes the Dockerfile at build time, not every time a container starts.
- `FROM` chooses the base image.
- `WORKDIR` sets a persistent build and runtime working directory.
- `COPY` adds selected build-context files to the image.
- `RUN` executes build-time preparation and saves filesystem results.
- `ENV` stores non-sensitive environment defaults.
- `EXPOSE` records intended container-port metadata but does not publish a host port.
- `USER` selects the identity for later build steps and runtime.
- `CMD` provides a replaceable default command or arguments.
- `ENTRYPOINT` provides a stable main executable.
- Exec form is generally preferred for runtime process and signal behavior.
- `.dockerignore` excludes unnecessary paths from the build context.
- Filesystem-changing steps and image-configuration steps have different effects.
- Secrets must never be embedded through `COPY`, `RUN`, `ARG`, or `ENV`.
- A successful build must be followed by runtime verification.

---

## Cheat Sheet

### Basic Dockerfile

```dockerfile
FROM alpine:3.22
WORKDIR /app
COPY --chown=10001:10001 app/ ./
RUN test -x ./server
ENV APP_ENV=production
EXPOSE 8080
USER 10001:10001
ENTRYPOINT ["./server"]
CMD ["--port", "8080"]
```

### `FROM`

```dockerfile
FROM repository:tag
```

Select trusted, maintained, explicitly versioned bases.

### `WORKDIR`

```dockerfile
WORKDIR /app
```

Sets later build and runtime working directory.

### `COPY`

```dockerfile
COPY source destination
COPY --chown=10001:10001 app/ /app/
```

Copies context files into image content.

### `RUN`

```dockerfile
RUN command-one \
    && command-two
```

Executes at build time; results become image content.

### `ENV`

```dockerfile
ENV APP_ENV=production
```

Stores an inspectable runtime default. Never put secrets here.

### `EXPOSE`

```dockerfile
EXPOSE 8080/tcp
```

Documents a container port. It does not publish it.

### `USER`

```dockerfile
USER 10001:10001
```

Sets later build and default runtime identity.

### `CMD`

```dockerfile
CMD ["application", "--port", "8080"]
```

One effective, replaceable default.

### `ENTRYPOINT` Plus `CMD`

```dockerfile
ENTRYPOINT ["application"]
CMD ["--port", "8080"]
```

Effective default:

```text
application --port 8080
```

### `.dockerignore`

```dockerignore
.git
*.log
*.env
secrets/
```

### Build

```bash
docker build -t NAME:TAG .
```

### Run

```bash
docker run -d \
  --name CONTAINER \
  -p 8080:8080 \
  NAME:TAG
```

### Inspect Image Configuration

```bash
docker image inspect \
  --format 'workdir={{.Config.WorkingDir}} user={{.Config.User}} entrypoint={{json .Config.Entrypoint}} cmd={{json .Config.Cmd}}' \
  NAME:TAG
```

### Build-Time Versus Runtime

```text
RUN        -> build-time command
CMD        -> runtime default
ENTRYPOINT -> runtime executable
```

---

## Challenge — Fix the Broken Dockerfile

### Scenario

A junior engineer submits:

```dockerfile
FROM alpine:latest

ENV API_TOKEN=real-production-token

USER 10001:10001

RUN apk add nginx

RUN cd /website
COPY . .

RUN nginx

EXPOSE 80

CMD ["nginx"]
CMD ["sleep", "infinity"]
```

The value shown is a fictional placeholder. In a real incident, do not repeat or preserve the exposed token.

### Your Tasks

1. Identify at least ten problems or uncertainties.
2. Replace the mutable base reference with an approved explicit version.
3. Remove the secret from the Dockerfile and design runtime secret delivery.
4. Put privileged build preparation before `USER`.
5. Replace `RUN cd` with `WORKDIR`.
6. Copy only the website content.
7. Add a suitable `.dockerignore`.
8. Configure an unprivileged listening port.
9. Keep the server in the foreground.
10. Use one deliberate `CMD`/`ENTRYPOINT` design.
11. Build the corrected image.
12. Run it as non-root.
13. Publish and test its port.
14. Inspect the final image configuration.

### Acceptance Criteria

- No secret exists in image content or metadata.
- The base version is explicit.
- Package installation occurs with sufficient build permissions.
- Runtime uses a dedicated non-root identity.
- Only required files enter the image.
- The main process remains in the foreground.
- Exactly one effective `CMD` and one deliberate `ENTRYPOINT` design are present.
- `EXPOSE` matches the application container port.
- Host publishing is performed at runtime.
- Build and runtime verification evidence is recorded.

### Senior Review Questions

1. Why was every instruction placed in that order?
2. Which instructions changed filesystem content?
3. Which instructions changed image configuration?
4. How would you rotate the API token without rebuilding the image?
5. What would happen if the runtime user could not read the website files?
6. How do you prove the application is not running as root?

---

## Lab Assets

Suggested paths:

```text
assets/chapter-07/lesson-16/
├── 01-project-tree.png
├── 02-dockerfile.png
├── 03-build-output.png
├── 04-image-inspect.png
├── 05-runtime-user.png
├── 06-http-response.png
├── 07-cmd-override.png
├── 08-environment-override.png
├── instruction-analysis.md
├── challenge-review.md
└── lab-results.txt
```

Do not capture real environment secrets, registry credentials, private repository URLs with tokens, or sensitive host paths in screenshots.

---

## Official References

- [Dockerfile overview](https://docs.docker.com/build/concepts/dockerfile/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker build overview](https://docs.docker.com/build/concepts/overview/)
- [Docker build best practices](https://docs.docker.com/build/building/best-practices/)
- [Build context](https://docs.docker.com/build/concepts/context/)
- [Dockerfile `USER`](https://docs.docker.com/reference/dockerfile/#user)
- [Dockerfile `ENTRYPOINT`](https://docs.docker.com/reference/dockerfile/#entrypoint)
- [Dockerfile `CMD`](https://docs.docker.com/reference/dockerfile/#cmd)

---

## Next Lesson

**Lesson 17 — Build Context and `docker build`**

You can now read and write the Dockerfile itself. Next, you will examine exactly which files Docker can see during a build and how the `docker build` command controls the result.

You will learn:

```bash
docker build .
docker build -t NAME:TAG .
docker build -f FILE .
docker image history
```

And understand:

- What the build context is
- Why the final command argument matters
- Dockerfile selection with `-f`
- Image names and tags during builds
- Build output
- Cache introduction
- `.dockerignore` pattern behavior
- Accidental secret inclusion

---

## Completion Check

Before continuing, explain without reading:

```text
What a Dockerfile produces
Build time versus runtime
FROM
WORKDIR
COPY
RUN
ENV
EXPOSE
USER
CMD
ENTRYPOINT
.dockerignore
Filesystem content versus image configuration
Why RUN does not start the future container
Why EXPOSE does not publish a host port
How ENTRYPOINT and CMD combine
Why secrets must remain outside the image
```

If you can explain these and complete the static-site lab, you are ready for Lesson 17.

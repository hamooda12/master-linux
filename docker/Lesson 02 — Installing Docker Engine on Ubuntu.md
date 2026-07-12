# 🐳 Chapter 07 — Docker for DevOps Engineers

# Lesson 02 — Installing Docker Engine on Ubuntu

> **Level:** Beginner  
> **Type:** Guided practical lesson  
> **Platform:** Supported 64-bit Ubuntu release  
> **Estimated study time:** 90–120 minutes  
> **Last installation-method verification:** July 12, 2026

---

## Overview

In Lesson 01, you learned the Docker mental model:

```text
Application + dependencies → image → container
Registry → stores images
Docker host → runs containers
```

Now you will turn your Ubuntu machine into a Docker host.

You will:

1. Inspect the machine before changing it
2. Remove only conflicting packages if they exist
3. Configure Docker's official APT repository
4. Install Docker Engine and its related packages
5. Verify the Docker CLI and service separately
6. Run the `hello-world` verification container
7. Diagnose Docker socket permission errors
8. Choose an access model deliberately

This lesson introduces only the architecture required to understand installation and permissions. Deep runtime internals still wait until later.

---

## Learning Objectives

By the end of this lesson, you should be able to:

- Explain the difference between Docker Engine, the Docker daemon, and the Docker CLI
- Install Docker Engine from Docker's official Ubuntu repository
- Explain why repository signing keys are used
- Check whether the Docker service is active and enabled
- Distinguish “command not found” from “cannot reach daemon” and “permission denied”
- Explain the role of `/var/run/docker.sock`
- Diagnose your exact Docker socket permission error
- Explain why membership in the `docker` group is security-sensitive
- Compare `sudo`, `docker` group membership, and rootless Docker
- Prove that both the Docker client and daemon work

---

## Prerequisites

You should understand:

- Image versus container
- Docker host
- Container registry
- Basic Linux commands
- Basic users, groups, permissions, APT, and `systemctl`

You need:

- A supported 64-bit Ubuntu installation
- A user allowed to run `sudo`
- Internet access to Docker's repository and registry
- A terminal

### Back up important Docker data first

If this machine already contains Docker images, containers, volumes, or configuration that matter, stop before removing or replacing packages. Package removal and data deletion are different operations, but installation changes should never be made without understanding the existing environment.

This lesson does **not** instruct you to delete `/var/lib/docker` or `/var/lib/containerd`.

---

## Why This Topic Exists

Typing `docker` involves more than one component.

```text
You type a command
       ↓
Docker CLI sends a request
       ↓
Docker daemon receives it
       ↓
Daemon manages images and containers
```

Several different failures are possible:

```text
docker: command not found
```

The CLI is not installed or is not in your command search path.

```text
Cannot connect to the Docker daemon
```

The CLI exists, but it cannot successfully contact the daemon.

```text
permission denied while trying to connect to the Docker API at unix:///var/run/docker.sock
```

The CLI exists and knows the daemon's socket address, but your user lacks permission to access that socket.

These errors are different. A professional diagnoses the layer that failed before changing anything.

---

# Part 1 — The Minimum Architecture

## 1. Docker Engine

Docker Engine is the core container platform installed on the Linux host.

For this lesson, it is enough to know that the installation provides a client and a service that work together.

## 2. Docker CLI

The Docker command-line interface is the `docker` program you run in your terminal.

Example:

```bash
docker version
```

The CLI accepts your command and sends a request to the Docker daemon.

The CLI does not normally create the container by itself.

## 3. Docker Daemon

The Docker daemon is the long-running background service, normally named `dockerd`.

It manages Docker objects such as:

- Images
- Containers
- Networks
- Volumes

On a standard rootful installation, systemd manages the Docker service.

## 4. Docker Socket

On a standard local Linux installation, the CLI commonly contacts the daemon through a Unix socket:

```text
/var/run/docker.sock
```

A Unix socket is a local communication endpoint. It lets processes on the same machine exchange requests and responses.

Beginner mental model:

```text
Docker CLI
    │ request
    ▼
/var/run/docker.sock
    │
    ▼
Docker daemon
```

The socket is not the daemon itself. It is the local doorway through which authorized clients communicate with the daemon.

## 5. Client and Server

Docker uses a client-server design:

| Component | Role |
| --- | --- |
| Docker CLI/client | Accepts your command and sends an API request |
| Docker daemon/server | Performs Docker operations |
| Unix socket | Local communication endpoint between them |

This explains why `docker --version` can succeed while `docker run` fails. The first command can report the installed CLI version without proving that the CLI can access a working daemon.

---

# Part 2 — Pre-installation Investigation

## 6. Confirm the Operating System

Run:

```bash
cat /etc/os-release
```

Important fields include:

```text
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION_CODENAME=noble
```

Your exact version may differ.

### Why check?

The repository configuration uses your Ubuntu codename to select compatible packages.

## 7. Check CPU Architecture

```bash
dpkg --print-architecture
```

Common output on an Intel or AMD 64-bit laptop:

```text
amd64
```

This does not mean your CPU must be made by AMD. Ubuntu uses `amd64` as the package architecture name for the common x86-64 architecture used by both AMD and Intel processors.

## 8. Check Whether Docker Already Exists

```bash
command -v docker
docker --version
```

Possible result:

```text
/usr/bin/docker
Docker version 29.x.x, build ...
```

Or:

```text
docker: command not found
```

### What `command -v` proves

It shows whether your shell can locate the `docker` executable.

It does not prove that:

- The Docker daemon is running
- Your user can access the daemon
- Containers can run

## 9. Check Existing Packages

```bash
dpkg -l | grep -E 'docker|containerd|runc'
```

This is evidence collection. Do not remove everything merely because its name appears.

You are looking for conflicting distribution packages such as:

```text
docker.io
docker-compose
docker-compose-v2
docker-doc
podman-docker
containerd
runc
```

Docker's official Engine packages use `containerd.io`, which bundles compatible container runtime dependencies.

## 10. Check Existing Service State

```bash
systemctl status docker --no-pager
```

Possible outcomes:

- `active (running)` — a Docker service is running
- `inactive` or `failed` — the unit exists but is not working normally
- `Unit docker.service could not be found` — the service unit is not installed

If Docker already works and contains important workloads, do not blindly reinstall it. Identify how it was installed and why a change is needed.

---

# Part 3 — Install from Docker's Official APT Repository

## 11. Why Use the Official Repository?

Ubuntu's repositories may offer a package named `docker.io`. Docker also provides its own repository with packages such as `docker-ce`.

This course uses Docker's official APT repository because it provides Docker's current maintained Engine packages and a clear upgrade path through APT.

Do not confuse:

```text
docker.io package  ≠  Docker Hub website
```

`docker.io` is an Ubuntu/Debian package name. Docker Hub is a container registry.

## 12. Remove Conflicting Packages

First inspect the packages as shown earlier. If you intend to use Docker's official packages, use Docker's current conflict-removal command:

```bash
sudo apt remove $(dpkg --get-selections \
  docker.io docker-compose docker-compose-v2 docker-doc \
  podman-docker containerd runc | cut -f1)
```

APT may report that none of these packages are installed. That is normal.

### Command explanation

```text
dpkg --get-selections ...  → lists matching installed package selections
cut -f1                    → keeps the package-name field
$(...)                     → supplies those names to apt remove
sudo apt remove            → removes the selected conflicting packages
```

### Safety note

This step can affect an existing container environment. Inspect first. On a new learning machine with no important workloads, the risk is much smaller.

Removing packages does not automatically remove data stored under `/var/lib/docker`, but that is not permission to ignore backups.

## 13. Refresh APT Metadata

```bash
sudo apt update
```

This downloads current package metadata from repositories already configured on the system.

## 14. Install Repository Prerequisites

```bash
sudo apt install ca-certificates curl
```

### Packages

| Package | Purpose here |
| --- | --- |
| `ca-certificates` | Helps the system verify HTTPS certificates |
| `curl` | Downloads Docker's signing key over HTTPS |

## 15. Create the APT Keyring Directory

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Breakdown:

| Part | Meaning |
| --- | --- |
| `install` | Can create directories/files with chosen ownership and permissions |
| `-d` | Create a directory |
| `-m 0755` | Set directory mode to `rwxr-xr-x` |
| `/etc/apt/keyrings` | Directory used for repository signing keys |

The command is safe if the directory already exists.

## 16. Download Docker's Official Signing Key

```bash
sudo curl -fsSL \
  https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
```

### `curl` options

| Option | Meaning |
| --- | --- |
| `-f` | Fail on HTTP error responses |
| `-s` | Silent mode |
| `-S` | Still show an error when silent mode is used |
| `-L` | Follow redirects |
| `-o FILE` | Write output to the named file |

## 17. Make the Key Readable by APT

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

`a+r` means add read permission for all user classes. APT processes need to read the public signing key.

### What the key does

The repository's public signing key lets APT verify that repository metadata was signed by the expected key and was not silently modified.

It does not encrypt the installed Docker packages.

## 18. Add the Docker Repository

Run this block exactly:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### What is happening?

The shell sends the lines between `<<EOF` and `EOF` to `sudo tee`.

`tee` writes them to:

```text
/etc/apt/sources.list.d/docker.sources
```

### Why not use normal `>` redirection with sudo?

This commonly fails:

```bash
sudo echo "text" > /etc/protected-file
```

`sudo` applies to `echo`, but your current non-root shell performs the `>` redirection. `sudo tee` performs the protected write itself.

### Dynamic values

```bash
$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
```

This reads the Ubuntu release information and outputs the appropriate codename.

```bash
$(dpkg --print-architecture)
```

This outputs the system's package architecture, such as `amd64`.

### Verify the created file

```bash
cat /etc/apt/sources.list.d/docker.sources
```

For a typical Ubuntu 24.04 amd64 host, you might see:

```text
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: noble
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
```

## 19. Refresh APT Again

```bash
sudo apt update
```

Why twice?

- The first update refreshed repositories that already existed
- You then added Docker's repository
- The second update downloads metadata from the newly added repository

Look for a Docker repository line and check that APT reports no signature error.

## 20. Confirm the Package Source

```bash
apt-cache policy docker-ce
```

The output should show a candidate version from:

```text
https://download.docker.com/linux/ubuntu
```

This is evidence that APT can see Docker's official repository.

## 21. Install Docker Packages

```bash
sudo apt install \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

### What each package provides

| Package | Beginner purpose |
| --- | --- |
| `docker-ce` | Docker Engine and daemon |
| `docker-ce-cli` | The `docker` command-line client |
| `containerd.io` | Bundled runtime dependencies used by Docker Engine |
| `docker-buildx-plugin` | Modern extended image-building command support |
| `docker-compose-plugin` | The `docker compose` command for multi-container applications |

You do not need to study containerd, Buildx, or Compose deeply now. They will receive their own lessons when you have the required foundation.

---

# Part 4 — Verify Every Layer

## 22. Verify the CLI Exists

```bash
docker --version
```

Example shape:

```text
Docker version 29.x.x, build ...
```

### What this proves

- The shell can find the Docker CLI
- The CLI executable can run

### What it does not prove

- The daemon is active
- Your user can access the socket
- A container can run

## 23. Check Docker Service Status

```bash
sudo systemctl status docker --no-pager
```

Look for:

```text
Loaded: loaded (...; enabled; ...)
Active: active (running)
```

### `loaded`

The systemd unit file was found and loaded.

### `enabled`

The service is configured to start automatically during boot.

### `active (running)`

The service is running now.

These are separate facts:

```text
enabled → start on future boots
active  → running right now
```

## 24. Start and Enable When Necessary

Start now:

```bash
sudo systemctl start docker
```

Enable at boot:

```bash
sudo systemctl enable docker
```

Perform both actions in one command:

```bash
sudo systemctl enable --now docker
```

| Option | Meaning |
| --- | --- |
| `enable` | Configure automatic start at boot |
| `--now` | Also start the service immediately |

On Ubuntu, official package installation normally starts Docker automatically, but verification is still required.

## 25. Inspect Logs if the Service Failed

```bash
sudo journalctl -u docker -n 50 --no-pager
```

| Part | Meaning |
| --- | --- |
| `-u docker` | Show messages for the Docker service |
| `-n 50` | Show the latest 50 lines |
| `--no-pager` | Print directly instead of opening a pager |

Do not restart repeatedly without reading the failure evidence.

## 26. Verify Client and Server Together

Initially use `sudo` with a standard rootful installation:

```bash
sudo docker version
```

This normally displays two major sections:

```text
Client:
 Version: ...

Server:
 Engine:
  Version: ...
```

### Interpretation

- Client section only: the CLI runs, but it did not receive daemon information
- Client and Server sections: the CLI successfully communicated with the daemon

## 27. Run the Verification Container

```bash
sudo docker run --rm hello-world
```

### What happens at a beginner level

```text
CLI contacts daemon
        ↓
Daemon looks for hello-world image locally
        ↓
If missing, daemon downloads it from a registry
        ↓
Daemon creates and starts a container
        ↓
Program prints a confirmation message
        ↓
Program exits
        ↓
--rm removes the stopped test container
```

### Why `--rm`?

It tells Docker to remove the container automatically after its process exits. The downloaded image remains available.

We will study `docker run` fully in Lesson 03. Here it is used only as an end-to-end verification.

---

# Part 5 — Understand Your Permission Error

## 28. The Exact Symptom

You previously ran:

```bash
docker run --rm alpine:3.22 id
```

and received:

```text
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

This tells us several things:

1. The `docker` CLI was found and executed
2. The CLI attempted to use `/var/run/docker.sock`
3. The operating system rejected access to that socket
4. The Alpine container did not start

This is not evidence that the Alpine image is broken.

The request was blocked before Docker could perform the container operation.

## 29. Inspect the Socket

```bash
ls -l /var/run/docker.sock
```

Typical output:

```text
srw-rw---- 1 root docker 0 Jul 12 10:00 /var/run/docker.sock
```

Breakdown:

```text
s          → socket file
rw-        → owner root can read/write
rw-        → group docker can read/write
---        → other users have no permission
root       → owner
docker     → group
```

If your user is not root and is not a member of the socket's group, access is denied.

## 30. Inspect Your Current Groups

```bash
id
groups
```

Example without Docker access:

```text
hamad sudo users
```

Example with Docker group membership:

```text
hamad sudo users docker
```

### Important session detail

`sudo usermod -aG docker "$USER"` changes account configuration. Your already-running login session may still have the old group list.

That is why logging out and back in is normally required.

---

# Part 6 — Choose an Access Model

## 31. Option A — Use `sudo`

Example:

```bash
sudo docker run --rm hello-world
```

### Benefits

- Simple and explicit
- No need to place your normal account in the `docker` group
- Good beginner default while learning permissions

### Trade-offs

- You must use `sudo` for daemon operations
- Docker configuration files created under different privilege contexts may later cause ownership confusion

For your current machine, using `sudo` is a valid immediate solution if you do not yet want to choose another access model.

## 32. Option B — Join the `docker` Group

First check whether the group exists:

```bash
getent group docker
```

The official packages normally create it. If it does not exist:

```bash
sudo groupadd docker
```

Add your current user:

```bash
sudo usermod -aG docker "$USER"
```

Breakdown:

| Part | Meaning |
| --- | --- |
| `usermod` | Modify a user account |
| `-G docker` | Set a supplementary group in this operation |
| `-a` | Append instead of replacing existing supplementary groups |
| `$USER` | Current login username |

### Why `-aG`, not only `-G`?

Without `-a`, `usermod -G docker "$USER"` may replace the user's existing supplementary group list. That can remove important memberships such as administrative groups.

### Activate the membership

Preferred approach:

1. Log out completely
2. Log back in
3. Open a new terminal

For a temporary subshell, you can use:

```bash
newgrp docker
```

But understanding the normal login-session refresh is better than relying on `newgrp` without knowing what it does.

### Verify

```bash
id
docker version
docker run --rm hello-world
```

### Critical security warning

Membership in the `docker` group grants root-level power in a standard rootful Docker installation.

Why? An authorized Docker user can ask the root-running daemon to perform highly privileged operations, including mounting host paths into containers. Therefore:

> Treat membership in the `docker` group like privileged administrative access, not like harmless application access.

Never add an untrusted user or service account merely to make an error disappear.

## 33. Fix `~/.docker` Ownership if Needed

If you previously used Docker with `sudo`, you might later see:

```text
WARNING: Error loading config file: /home/user/.docker/config.json:
permission denied
```

First inspect:

```bash
ls -ld "$HOME/.docker"
ls -l "$HOME/.docker" 2>/dev/null
```

If the directory exists and is incorrectly owned by root, Docker's official correction is:

```bash
sudo chown "$USER":"$USER" "$HOME/.docker" -R
sudo chmod g+rwx "$HOME/.docker" -R
```

Do not run ownership commands blindly. Confirm that `$HOME` and the target directory are correct first.

## 34. Option C — Rootless Docker

Rootless mode runs both the Docker daemon and containers without root privileges for that user.

Beginner comparison:

```text
Rootful + docker group:
user → system socket → root-running daemon

Rootless:
user → user socket → user-running daemon
```

### Benefits

- Reduces risks associated with a root-running daemon
- Avoids granting the user root-level access through the system Docker socket

### Trade-offs

- Has prerequisites and operational differences
- Some networking, resource-management, or system integration behavior differs
- Ubuntu 24.04 and later may require appropriate AppArmor configuration for unprivileged user namespaces

Rootless installation is not performed in this beginner lab because it is a separate deployment decision, not merely a quick permission fix.

## 35. Access Model Comparison

| Model | Daemon identity | Command style | Security meaning | Beginner guidance |
| --- | --- | --- | --- | --- |
| `sudo docker ...` | Root | Explicit `sudo` | Privilege requested per command | Safe starting choice |
| `docker` group | Root | No `sudo` | Group member has root-level Docker power | Convenient, but deliberate |
| Rootless Docker | Normal user | No `sudo` | Daemon and containers lack host root | Strong option with extra setup |

There is no universal choice for every environment. For a personal learning laptop, either explicit `sudo` or deliberate Docker-group membership is common. Production access must follow the organization's security model.

---

# Part 7 — Evidence-first Troubleshooting

## 36. Troubleshooting Decision Tree

### Symptom A — `docker: command not found`

Check:

```bash
command -v docker
dpkg -l docker-ce-cli
```

Likely layer: CLI installation or shell path.

### Symptom B — `Cannot connect to the Docker daemon`

Check:

```bash
sudo systemctl status docker --no-pager
sudo journalctl -u docker -n 50 --no-pager
```

Likely layers: daemon state, socket, Docker context, or configuration.

### Symptom C — Socket `permission denied`

Check:

```bash
ls -l /var/run/docker.sock
id
groups
sudo docker version
```

Interpretation:

- If `sudo docker version` works but `docker version` fails with permission denied, the daemon is probably reachable and the difference is your user's authorization
- Compare socket owner/group/mode with your current group memberships

### Symptom D — Service is `failed`

Check evidence:

```bash
sudo systemctl status docker --no-pager -l
sudo journalctl -u docker -n 100 --no-pager
```

Do not change configuration until the error message gives you a testable hypothesis.

### Symptom E — Image download fails

Check:

```bash
curl -I https://registry-1.docker.io/v2/
getent hosts registry-1.docker.io
```

An HTTP `401 Unauthorized` from the registry endpoint can still prove DNS, TCP, and TLS connectivity; the endpoint requires registry authentication behavior. A timeout or DNS failure points to a different layer.

## 37. Do Not “Fix” the Socket with `chmod 666`

Avoid:

```bash
sudo chmod 666 /var/run/docker.sock
```

That grants every local user read/write access to a root-controlled Docker daemon, effectively giving every local user root-level power through Docker.

It may also be temporary because the socket can be recreated with managed permissions.

Correct the access model, not merely the visible mode bits.

## 38. Do Not Run Random Internet Scripts Blindly

Docker provides a convenience installation script, but official documentation recommends it primarily for testing and development, not as the preferred production installation approach.

Why this course uses the repository steps:

- You see which repository is added
- You see which key is trusted
- You see which packages are installed
- Future updates work through APT
- The procedure is easier to audit

Never use this pattern without reviewing the source:

```bash
curl URL | sudo sh
```

It downloads code and immediately executes it with administrative privileges.

---

## Commands — Detailed Reference

### `docker --version`

```bash
docker --version
```

Purpose: show the CLI's Docker version string.

Use it to prove that the CLI executable is installed. Do not use it alone to prove daemon connectivity.

### `docker version`

```bash
sudo docker version
```

Purpose: show detailed client and server version information.

Use the presence of both Client and Server sections as connectivity evidence.

### `systemctl status`

```bash
sudo systemctl status docker --no-pager
```

Purpose: inspect the service's current state and recent messages.

### `systemctl enable --now`

```bash
sudo systemctl enable --now docker
```

Purpose: enable the service for boot and start it immediately.

### `ls -l` on the socket

```bash
ls -l /var/run/docker.sock
```

Purpose: inspect the socket type, owner, group, and permission bits.

### `groups` and `id`

```bash
groups
id
```

Purpose: inspect the current session's identity and group memberships.

### `docker run --rm hello-world`

```bash
sudo docker run --rm hello-world
```

Purpose in this lesson: verify the complete path from CLI to daemon, registry/image availability, container creation, execution, and exit.

Full `docker run` behavior comes in Lesson 03.

---

## Guided Hands-on Lab

### Scenario

Your Ubuntu laptop must become a safe Docker learning host. You previously received a permission error when contacting `/var/run/docker.sock`.

Follow one step at a time. Read the output before continuing.

### Phase A — Observe

```bash
cat /etc/os-release
dpkg --print-architecture
command -v docker
docker --version
dpkg -l | grep -E 'docker|containerd|runc'
systemctl status docker --no-pager
```

Record:

- Ubuntu version and codename
- Architecture
- Whether the CLI exists
- Existing package names
- Service state

### Phase B — Prepare Official Repository

Only proceed with installation/replacement after interpreting Phase A.

```bash
sudo apt remove $(dpkg --get-selections \
  docker.io docker-compose docker-compose-v2 docker-doc \
  podman-docker containerd runc | cut -f1)

sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the source:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Verify:

```bash
cat /etc/apt/sources.list.d/docker.sources
sudo apt update
apt-cache policy docker-ce
```

### Phase C — Install

```bash
sudo apt install \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### Phase D — Verify Rootful Docker

```bash
docker --version
sudo systemctl status docker --no-pager
sudo docker version
sudo docker run --rm hello-world
```

Do not move on until all four checks are understood.

### Phase E — Investigate Non-sudo Access

```bash
ls -l /var/run/docker.sock
id
groups
docker version
```

If the last command fails, explain the mismatch between socket permissions and your groups.

### Phase F — Choose

Choose one:

#### Choice 1: Continue with explicit sudo

```bash
sudo docker run --rm hello-world
```

#### Choice 2: Grant your account Docker-group access

```bash
sudo usermod -aG docker "$USER"
```

Then log out completely and log back in.

Verify:

```bash
id
docker version
docker run --rm hello-world
```

Do not choose group membership unless you accept that it grants root-level Docker power.

### Acceptance Criteria

The selected access model must make both commands succeed:

```bash
docker version
docker run --rm hello-world
```

If you selected explicit `sudo`, use:

```bash
sudo docker version
sudo docker run --rm hello-world
```

You must also be able to explain:

- Why the other form fails or succeeds
- Who owns the socket
- Which group can write to it
- Whether your current session belongs to that group

---

## Discovery Questions

### Question 1

Why can `docker --version` work while `docker run hello-world` fails?

<details>
<summary>Explanation</summary>

`docker --version` can read the installed CLI version locally. Running a container requires successful communication with the Docker daemon.

</details>

### Question 2

If `sudo docker version` works but `docker version` returns socket permission denied, is the daemon probably stopped?

<details>
<summary>Explanation</summary>

No. The successful sudo command is evidence that the daemon is reachable. The difference points to the normal user's socket authorization.

</details>

### Question 3

Why must you log out and back in after `usermod -aG docker`?

<details>
<summary>Explanation</summary>

The existing login session normally keeps the group list established when the session began. A new login re-evaluates membership.

</details>

### Question 4

Why is `chmod 666 /var/run/docker.sock` dangerous?

<details>
<summary>Explanation</summary>

It lets every local user control the root-running daemon, which can effectively provide root-level power.

</details>

---

## Production Scenario

A junior engineer reports:

```text
docker: permission denied while trying to connect to /var/run/docker.sock
```

Another engineer suggests:

```bash
sudo chmod 666 /var/run/docker.sock
```

Your evidence-first response should be:

1. Confirm the exact command and error
2. Inspect `systemctl status docker`
3. Inspect `ls -l /var/run/docker.sock`
4. Inspect the user's identity with `id`
5. Test with authorized escalation if permitted
6. Choose the organization's approved access model
7. Verify client and server communication
8. Document the security impact of any group membership

The smallest correct fix may be adding an approved administrator to the Docker group, using `sudo`, or deploying rootless Docker. World-writable socket permissions are not a responsible solution.

---

## Common Mistakes

### 1. Installing multiple conflicting Docker packages

Do not mix `docker.io`, official `docker-ce`, and unrelated runtime packages without understanding package ownership.

### 2. Assuming CLI installation proves daemon health

Use `docker version` and service inspection, not only `docker --version`.

### 3. Adding every user to the Docker group

The group grants root-level power in a standard installation.

### 4. Forgetting to refresh the login session

Account configuration may be correct while the current shell still has the old groups.

### 5. Using `usermod -G` without `-a`

This can remove other supplementary group memberships.

### 6. Making the Docker socket world-writable

This creates a serious local privilege boundary failure.

### 7. Repeatedly restarting a failed service

Read `systemctl status` and `journalctl` evidence first.

### 8. Deleting `/var/lib/docker` to fix installation

That path may contain images, containers, and volumes. Deletion is destructive and is not a normal troubleshooting step.

### 9. Running downloaded scripts without inspection

Especially avoid piping remote content directly into a root shell.

### 10. Confusing Docker Engine with Docker Desktop

Docker Engine is the native server component used on Linux hosts. Docker Desktop is a separate desktop product with additional tooling and architecture.

---

## Best Practices

- Inspect the existing system before installing or replacing packages
- Use supported Ubuntu releases and Docker's documented repository
- Verify repository signatures and APT errors
- Verify the CLI, service, socket, and end-to-end container execution separately
- Treat Docker daemon access as privileged access
- Prefer explicit, auditable access decisions
- Use rootless mode when its security and operational model fits
- Never make the Docker socket world-writable
- Read service logs before modifying configuration
- Back up important Docker data before package or host changes
- Test Docker upgrades outside production first
- Keep installation automation version-aware and auditable

---

## Interview Questions

### Junior

1. What is the Docker CLI?
2. What is the Docker daemon?
3. What is `/var/run/docker.sock`?
4. What is the difference between `docker --version` and `docker version`?
5. What does `systemctl enable --now docker` do?
6. Why might Docker require `sudo`?
7. Why must you log in again after joining the Docker group?
8. Why is `usermod -aG` safer than `usermod -G` for adding a group?
9. What does `hello-world` verify?
10. Why should you not use `chmod 666` on the Docker socket?

### Intermediate

1. Why does Docker separate the CLI from the daemon?
2. How would you distinguish CLI, daemon, and authorization failures?
3. Why does the Docker group effectively grant root-level privileges?
4. What is the security difference between rootless Docker and the Docker group?
5. Why is an official APT repository preferable to a blind convenience script in an auditable production process?
6. What evidence would you collect before replacing an existing Docker installation?

### Senior Preview

Senior-level daemon hardening, remote socket protection, firewall integration, and runtime security are deliberately deferred until the security and internals phases.

---

## Lesson Reflection

Answer in your own words:

1. Explain the path from a typed Docker command to the daemon.
2. Why did your original Alpine command fail before the container started?
3. How do socket owner, group, and mode control daemon access?
4. What does `sudo docker version` prove that `docker --version` does not?
5. What is the difference between a service being active and enabled?
6. Why is the Docker group not an ordinary convenience group?
7. Which access model did you choose, and why?
8. What would you check if Docker's service entered the failed state?

---

## Summary

- Docker uses a client-server architecture
- The CLI sends requests to the Docker daemon
- A standard local rootful installation commonly uses `/var/run/docker.sock`
- The CLI and daemon must be verified separately
- Docker's official APT repository provides maintained Engine packages
- `docker --version` proves only that the CLI can run
- `docker version` can prove client-server communication
- `systemctl status docker` shows current service state
- `enable` and `active` are different concepts
- Socket permission denied means the OS rejected access to the communication endpoint
- `sudo`, Docker-group membership, and rootless mode are different access models
- The Docker group grants root-level power on standard rootful Docker
- The socket must never be made world-writable as a quick fix

---

## Cheat Sheet

### Inspect before installation

```bash
cat /etc/os-release
dpkg --print-architecture
command -v docker
docker --version
dpkg -l | grep -E 'docker|containerd|runc'
systemctl status docker --no-pager
```

### Configure Docker's repository

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### Install

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### Verify

```bash
docker --version
sudo systemctl status docker --no-pager
sudo docker version
sudo docker run --rm hello-world
```

### Diagnose permissions

```bash
ls -l /var/run/docker.sock
id
groups
sudo docker version
```

### Docker group option

```bash
sudo usermod -aG docker "$USER"
# Log out completely and log back in.
id
docker version
docker run --rm hello-world
```

### Service troubleshooting

```bash
sudo systemctl status docker --no-pager -l
sudo journalctl -u docker -n 100 --no-pager
sudo systemctl enable --now docker
```

---

## Challenge — Diagnose Before Fixing

### Incident

TechCorp gives you this report:

```text
$ docker --version
Docker version 29.x.x, build ...

$ docker run --rm hello-world
permission denied while trying to connect to the Docker API at unix:///var/run/docker.sock

$ ls -l /var/run/docker.sock
srw-rw---- 1 root docker 0 Jul 12 10:00 /var/run/docker.sock

$ groups
junior sudo developers
```

### Your tasks

1. State what `docker --version` proves
2. State why `docker run` failed
3. Name two legitimate access choices
4. Explain the security cost of the Docker-group choice
5. Write the commands you would use to verify your hypothesis
6. Explain why `chmod 666` is rejected
7. Define the acceptance criteria after correction

### Rules

- Evidence before change
- No world-writable socket
- Do not reinstall Docker without evidence of an installation problem
- Do not assume the daemon is stopped

---

## Lab Assets

```text
assets/chapter-07/lesson-02/
├── os-release.txt
├── architecture.txt
├── existing-packages.txt
├── docker-source.txt
├── docker-version.txt
├── docker-service-status.txt
├── docker-socket-permissions.txt
├── user-groups-before.txt
├── user-groups-after.txt
├── hello-world-output.txt
└── permission-incident-notes.md
```

Suggested capture commands:

```bash
cat /etc/os-release > os-release.txt
dpkg --print-architecture > architecture.txt
docker --version > docker-version.txt
ls -l /var/run/docker.sock > docker-socket-permissions.txt
id > user-groups-after.txt
```

Do not store passwords, tokens, private registry credentials, or unrelated sensitive environment information in lab assets.

---

## Official References

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/)
- [Docker rootless mode](https://docs.docker.com/engine/security/rootless/)

Installation procedures change over time. Recheck the official Ubuntu installation page before installing Docker on an important or production system.

---

## Next Lesson

**Lesson 03 — Your First Container**

Now that the client and daemon work, you are ready to study `docker run` properly.

You will run `hello-world` twice and observe:

```text
Check for image locally
        ↓
Pull image if missing
        ↓
Create container
        ↓
Start its main process
        ↓
Print output
        ↓
Process exits
        ↓
Container becomes stopped
```

You will also learn the first essential inspection commands:

```bash
docker run
docker ps
docker ps -a
docker images
```

The goal will be to understand exactly why `hello-world` exits and why that behavior is successful—not broken.

---

## Completion Check

Do not move to Lesson 03 until you can prove the selected access model works:

```bash
docker version
docker run --rm hello-world
```

Or, if you deliberately selected explicit sudo:

```bash
sudo docker version
sudo docker run --rm hello-world
```

You should also be able to explain, without reading:

```text
CLI versus daemon
Socket purpose
Active versus enabled
Why permission denied occurred
Why docker-group access is powerful
Why chmod 666 is not a fix
```

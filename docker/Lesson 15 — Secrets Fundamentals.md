# Chapter 07 — Docker for DevOps Engineers

# Lesson 15 — Secrets Fundamentals

> **Lesson goal:** Recognize sensitive data, keep it out of images and ordinary configuration, deliver it only to authorized workloads, and plan safe rotation.

---

## Overview

Lesson 14 separated ordinary runtime configuration from the image:

```text
APP_ENV=production
LOG_LEVEL=info
API_PORT=8080
```

Those values usually describe how an application should behave. They are not normally confidential.

Now consider:

```text
DATABASE_PASSWORD=...
API_TOKEN=...
PRIVATE_KEY=...
```

These values grant access. If they are exposed, another person or process may be able to authenticate, decrypt data, sign requests, or impersonate a service.

That makes them **secrets**.

This lesson builds a practical model:

```text
Secret lifecycle

create -> store -> authorize -> deliver -> use -> rotate -> revoke
```

Security is explained at an operational level. Advanced kernel isolation, Linux capabilities, mandatory access control, and container-runtime internals remain in later security lessons.

---

## Learning Objectives

By the end of this lesson, you should be able to:

1. Distinguish ordinary configuration from a secret.
2. Identify common secret types.
3. Explain why a secret must not be baked into an image.
4. Explain the risks of command-line arguments and ordinary environment variables.
5. Deliver a lab secret through a read-only file.
6. Apply least privilege to secret access.
7. Describe a basic secret-rotation workflow.
8. Understand the purpose of Docker Compose secrets.
9. Understand where Docker Swarm secrets and production secret managers fit.
10. Respond correctly when a secret is exposed.

---

## Prerequisites

Before beginning, you should understand:

- Images and containers
- Image layers at an introductory level
- `docker run`
- `docker exec`
- `docker inspect`
- Environment variables and `--env-file`
- Bind mounts and read-only mounts
- Basic Linux file permissions

Verify Docker:

```bash
docker version
```

All credentials used in this lesson are fake lab values. Never replace them with real credentials while experimenting.

---

## Part 1 — Configuration Versus Secret

### 1. What Is Configuration?

Configuration changes application behavior.

Examples:

```text
APP_ENV=production
LOG_LEVEL=warning
API_PORT=8080
REGION=eu-central
FEATURE_REPORTS=true
```

You may not want every configuration value published, but exposure of `LOG_LEVEL=info` does not normally grant access to another system.

### 2. What Is a Secret?

A secret is sensitive data that proves identity, grants access, decrypts information, or performs another privileged operation.

Examples:

- Passwords
- API tokens
- OAuth client secrets
- Database credentials
- Private cryptographic keys
- TLS private keys
- SSH private keys
- Signing keys
- Encryption keys
- Cloud access credentials
- Session-signing secrets

The exact value should be known only by authorized people and workloads.

### 3. Public Data Can Be Related to a Secret

Not every security-related file is secret.

| Item | Normally secret? | Reason |
|---|---:|---|
| TLS certificate | No | It contains public identity information and a public key |
| TLS private key | Yes | It proves control of the identity |
| SSH public key | No | It is deliberately shared |
| SSH private key | Yes | It authenticates its holder |
| Database hostname | Usually no | It locates the service but does not authenticate |
| Database password | Yes | It grants authenticated access |
| API token identifier | Depends | It may be metadata rather than the credential |
| API token value | Yes | It authorizes API calls |

Classification depends on what the value enables, not only its filename.

### 4. A Useful Test

Ask:

```text
If an unauthorized person gets this value, what can they do?
```

If they can log in, call an API, decrypt data, sign requests, or impersonate a service, treat it as a secret.

### 5. Secret Versus Sensitive Information

Some information is sensitive even when it is not an authentication secret:

- Customer personal data
- Internal architecture details
- Incident reports
- Private business information

This lesson focuses on credentials and keys used by containerized applications. Sensitive business data also requires protection, but it has a broader data-governance lifecycle.

---

## Part 2 — The Secret Lifecycle

### 6. Secrets Are More Than Values

A secret-management design must answer:

```text
Who creates it?
Where is it stored?
Which workload may receive it?
How is it delivered?
How long is it valid?
How is access audited?
How is it rotated?
How is it revoked?
```

Protecting only one stage is not enough.

### 7. Create

Secrets should be generated using an appropriate secure process. Human-chosen passwords are often weaker and harder to rotate safely than generated credentials.

This lesson does not teach cryptographic generation commands because the correct method depends on the credential type and consuming system.

### 8. Store

A production secret should live in an approved system designed to control access, rather than in:

- Application source code
- A Dockerfile
- A public or private Git repository
- A shared chat message
- A ticket comment
- An image layer
- A developer's shell history
- An unprotected `.env` file

### 9. Authorize

Only workloads that need a secret should receive it.

```text
database password
   |
   +-- product-api: allowed
   +-- frontend: not allowed
   +-- log collector: not allowed
```

### 10. Deliver

The delivery channel should minimize exposure. A common container pattern is to make the secret available as a file at runtime:

```text
/run/secrets/database_password
```

The application reads the file when needed. The value is not part of the image.

### 11. Use

The application should:

- Read only the required secret
- Avoid printing it
- Avoid placing it in error messages
- Avoid copying it to a less protected location
- Keep it in memory only as long as necessary when practical
- Handle missing or unreadable secret files clearly

### 12. Rotate and Revoke

Secrets should not remain valid forever.

Rotation replaces an old secret with a new one. Revocation makes an old or exposed secret unusable.

```text
old secret active
       |
create new version
       |
update authorized consumers
       |
verify new version works
       |
revoke old version
```

---

## Part 3 — Why Secrets Must Not Be Baked into Images

### 13. What “Baked into the Image” Means

A value is baked into an image when it becomes part of image content or metadata during the build.

Dangerous examples:

```dockerfile
ENV DATABASE_PASSWORD=real-password
```

```dockerfile
COPY production.env /app/production.env
```

```dockerfile
COPY id_rsa /root/.ssh/id_rsa
```

```dockerfile
RUN curl -H "Authorization: Bearer real-token" https://private.example/file
```

Do not use real values in these patterns.

### 14. Images Are Distributed Artifacts

An image may be:

- Pushed to a registry
- Pulled to many hosts
- Cached by build systems
- Shared with another team
- Exported to a file
- Retained in backups
- Inspected long after the original deployment

If a secret is inside the image, every copy becomes a potential exposure point.

### 15. Deleting Later May Not Remove Earlier Evidence

This Dockerfile is unsafe:

```dockerfile
COPY secret.txt /tmp/secret.txt
RUN use-secret /tmp/secret.txt
RUN rm /tmp/secret.txt
```

The introductory layer model is:

```text
Layer 1: secret.txt added
Layer 2: secret used
Layer 3: secret appears deleted in final view
```

The later deletion changes the final filesystem view, but the content may still exist in an earlier image layer.

### 16. Image Metadata Can Also Expose Values

Secrets can appear in:

- `ENV` configuration
- Build commands or history
- Labels
- Copied files
- Generated artifacts
- Package-manager configuration

Inspecting only the final running filesystem is not enough to prove that an image never contained a secret.

### 17. Safe Build-Time Secret Direction

Sometimes a build needs temporary authentication to download a private dependency. Modern Docker builds support build secret mounts for this purpose.

Conceptual command:

```bash
docker build --secret id=repository_token,src=token.txt .
```

Conceptual Dockerfile use:

```dockerfile
RUN --mount=type=secret,id=repository_token \
    command-that-reads /run/secrets/repository_token
```

This is only an introduction. BuildKit secret mounts, cache behavior, multi-stage builds, and secure dependency downloads will be taught after Dockerfile fundamentals.

Key rule for now:

```text
Do not use Dockerfile ARG or ENV to pass build secrets.
```

---

## Part 4 — Risks of Command-Line Arguments

### 18. Visible Commands

This is unsafe with a real value:

```bash
docker run myapp:1.0 --api-token real-token
```

The token may appear in:

- Shell history
- Process listings
- Command auditing
- CI/CD logs
- Deployment definitions
- Docker container configuration

### 19. Docker Inspection

Container command arguments are part of container configuration.

Inspect them:

```bash
docker inspect --format '{{json .Config.Cmd}}' CONTAINER
```

If a secret was provided as an argument, a sufficiently privileged user may retrieve it.

### 20. Hiding Output Is Not the Same as Protecting the Secret

Disabling terminal echo or masking a CI log can reduce accidental display, but it does not automatically remove the value from:

- Container metadata
- Process arguments
- Audit systems
- Deployment state

Evaluate the entire path the secret travels.

---

## Part 5 — Risks of Ordinary Environment Variables

### 21. Environment Variables Are Convenient

Many applications support:

```text
DATABASE_PASSWORD=...
API_TOKEN=...
```

This can be operationally convenient, and some platforms inject secrets through environment variables. However, an ordinary environment variable is not automatically secret merely because its name contains `PASSWORD`.

### 22. Demonstrate Inspection with a Fake Token

Use a deliberately fake lab value:

```bash
docker run -d \
  --name insecure-env-demo \
  -e API_TOKEN=FAKE-LAB-TOKEN-NOT-VALID \
  alpine:3.22 sleep 3600
```

Inspect it:

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  insecure-env-demo
```

You can see:

```text
API_TOKEN=FAKE-LAB-TOKEN-NOT-VALID
```

Remove the lab container:

```bash
docker rm -f insecure-env-demo
```

### 23. Other Exposure Paths

Environment values may also reach:

- Application debug pages
- Crash reports
- Diagnostic bundles
- Logs that print the environment
- Child processes
- Monitoring agents
- Deployment platform metadata

The exact risk depends on the application, OS, Docker access, and deployment platform.

### 24. Do Not Dump the Environment Blindly

This is useful in a controlled lab:

```bash
printenv
```

In production, printing the complete environment can disclose unrelated credentials.

Prefer a targeted check:

```bash
printenv APP_ENV
```

For a suspected secret, confirm presence without printing its value when possible. For example, an application can log:

```text
DATABASE_PASSWORD is configured
```

not:

```text
DATABASE_PASSWORD=actual-value
```

### 25. Practical Conclusion

Environment-variable delivery is used in real systems, but it carries visibility and lifecycle tradeoffs. Use the method approved by your platform and threat model.

For this course, the safer introductory pattern is:

```text
ordinary configuration -> environment variable
secret                 -> controlled runtime secret file
```

Later platforms may provide other controlled delivery methods.

---

## Part 6 — Secret Files

### 26. The File-Based Pattern

Instead of passing the value directly as an argument or ordinary environment value, a platform can expose it as a runtime file:

```text
/run/secrets/database_password
```

The application reads the file content:

```sh
database_password="$(cat /run/secrets/database_password)"
```

Do not print the variable afterward.

### 27. Why `/run/secrets`?

`/run/secrets` is a common convention used by Docker secret mechanisms. The directory name itself provides no security. Protection comes from:

- How the secret is stored
- How it is delivered
- Who can access the workload
- File access controls
- Runtime isolation
- Rotation and revocation

Creating an ordinary world-readable file under `/run/secrets` would still be insecure.

### 28. `_FILE` Convention

Some container images support variables ending in `_FILE`:

```text
DATABASE_PASSWORD_FILE=/run/secrets/database_password
```

The application or entrypoint reads the secret from that path.

Important:

```text
Docker does not make every application understand _FILE automatically.
```

The image or application must explicitly support the convention.

### 29. Read-Only Delivery

A secret should normally be read-only to the consuming application:

```text
secret source -> read-only file -> authorized process
```

Read-only delivery prevents the application from accidentally changing the source through that mount. It does not prevent an authorized process from reading and leaking the value, so application behavior and access control still matter.

### 30. File Permission Is One Layer

On a normal host file:

```bash
chmod 600 secret-file
```

This permits access for the owner and blocks normal group/other access.

But permissions alone do not provide:

- Centralized rotation
- Audit history
- Automatic expiration
- Encryption at rest
- Identity-based application authorization
- Protection from a privileged host administrator

Permissions are necessary in many cases, but they are not a complete secret-management system.

---

## Part 7 — Guided Secret-File Lab

### 31. Lab Goal

Create a fake secret on the host, mount only that file into a container as read-only, read it without printing it, and compare the result with an environment-variable secret.

This lab demonstrates the **delivery pattern**, not a production secret manager.

### 32. Create a Protected Lab Directory

Host prompt:

```text
hamad@Hamooda:~$
```

Run:

```bash
mkdir -p "$HOME/docker-secret-lab"
chmod 700 "$HOME/docker-secret-lab"
```

### 33. Create a Fake Secret Safely

Set a restrictive creation mask in the current shell:

```bash
umask 077
```

Create a fake value:

```bash
printf '%s' 'FAKE-DB-PASSWORD-FOR-LAB-ONLY' \
  > "$HOME/docker-secret-lab/database_password"
```

Verify permissions without printing content:

```bash
ls -l "$HOME/docker-secret-lab/database_password"
```

Expected permission shape:

```text
-rw-------
```

The exact owner, timestamp, and path will differ.

### 34. Start a Container with a Read-Only File Mount

```bash
docker run -d \
  --name secret-file-lab \
  --mount type=bind,source="$HOME/docker-secret-lab/database_password",target=/run/secrets/database_password,readonly \
  alpine:3.22 sleep 3600
```

### 35. Verify the File Exists

```bash
docker exec secret-file-lab \
  ls -l /run/secrets/database_password
```

The file is available at the intended path.

### 36. Verify Without Revealing the Value

Check that the file is non-empty:

```bash
docker exec secret-file-lab sh -c \
  'test -s /run/secrets/database_password && echo "secret file is present and non-empty"'
```

Expected:

```text
secret file is present and non-empty
```

Calculate a checksum for lab comparison without printing the value:

```bash
sha256sum "$HOME/docker-secret-lab/database_password"
```

```bash
docker exec secret-file-lab \
  sha256sum /run/secrets/database_password
```

The hashes should match. A checksum is not a safe way to publish or verify low-entropy real passwords because hashes may assist guessing attacks. Here it is used only with a fake lab value.

### 37. Prove the Mount Is Read-Only

Attempt to replace the content:

```bash
docker exec secret-file-lab sh -c \
  'printf changed > /run/secrets/database_password'
```

The write should fail with a read-only filesystem or permission-related error.

Verify the host file still exists:

```bash
ls -l "$HOME/docker-secret-lab/database_password"
```

Do not print a real secret to verify it.

### 38. Inspect Container Environment

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  secret-file-lab
```

The fake database password is not stored in `.Config.Env` because you did not use `-e`.

### 39. Inspect the Mount

```bash
docker inspect \
  --format '{{json .Mounts}}' \
  secret-file-lab
```

Docker can reveal mount metadata such as the host source path and container destination. A bind mount hides the value from `.Config.Env`, but it does not hide the existence or location of the source file from privileged Docker users.

### 40. Read the File in an Application-Style Command

```bash
docker exec secret-file-lab sh -c '
  secret_path=/run/secrets/database_password
  if [ ! -s "$secret_path" ]; then
    echo "required database password is missing" >&2
    exit 1
  fi
  echo "database password loaded successfully"
'
```

Expected:

```text
database password loaded successfully
```

Notice that the command verifies and uses the path without printing its content.

### 41. Cleanup

Remove the container:

```bash
docker rm -f secret-file-lab
```

Remove the fake lab file and directory:

```bash
rm "$HOME/docker-secret-lab/database_password"
rmdir "$HOME/docker-secret-lab"
```

Do not assume `rm` provides forensic secure erasure on every filesystem or SSD. In this lab the value was deliberately fake.

### 42. What This Lab Did and Did Not Prove

It proved:

- A secret can be supplied at runtime instead of copied into the image.
- The application can read a file without printing it.
- The file can be mounted read-only.
- The value does not appear in the container's environment array.

It did **not** prove:

- Centralized encrypted storage
- Automatic rotation
- Audit logging
- Protection from a privileged host user
- Dynamic short-lived credentials
- Production-grade secret management

The source was still a plain host file. That is why this is a delivery-pattern lab, not a complete production solution.

---

## Part 8 — Least Privilege

### 43. Meaning of Least Privilege

Least privilege means giving only the access required to complete the intended task.

For secrets, ask:

```text
Which workload needs it?
Which exact secret does it need?
Does it need read or write access?
How long should access last?
Which identity is using it?
```

### 44. Service-Level Isolation

Suppose TechCorp has:

```text
frontend
product-api
billing-worker
log-collector
```

Secret grants should look like:

| Secret | Frontend | Product API | Billing Worker | Log Collector |
|---|---:|---:|---:|---:|
| Product database password | No | Yes | No | No |
| Billing API token | No | No | Yes | No |
| TLS private key | Maybe through proxy design | No | No | No |

Do not attach every secret to every container for convenience.

### 45. Why “It Is on the Same Host” Is Not Authorization

Multiple containers may share one Docker host, but that does not mean they should share credentials.

```text
same host != same trust requirement
```

Each service should receive only its required secret.

### 46. Read-Only Access

Applications typically need to read credentials, not modify them. Deliver secret files read-only unless there is a specific, reviewed reason for write access.

### 47. Run as a Non-Root User

A later Dockerfile lesson will teach `USER`. Running the application as a dedicated non-root user can reduce the effect of an application compromise.

The secret file must still be readable by the intended application identity and unreadable by unnecessary identities. File ownership and mode must match the runtime user model.

### 48. Limit Human Access Too

Least privilege applies to:

- Developers
- Operators
- CI/CD systems
- Support staff
- Backup systems
- Monitoring tools
- The application itself

Human convenience is not a reason to give broad permanent access to production credentials.

---

## Part 9 — Secret Rotation

### 49. Why Rotate?

Rotate a secret when:

- Its regular lifetime ends
- A person or workload no longer needs access
- Exposure is suspected
- A deployment boundary changes
- Policy requires it
- A cryptographic key reaches its planned replacement date

### 50. Rotation Is Not Just Editing a File

A safe rotation coordinates the credential issuer and every authorized consumer.

Example:

```text
1. Create database password version 2
2. Configure database to accept version 2
3. Deliver version 2 to product-api
4. Restart/redeploy or reload consumer safely
5. Verify application health and database access
6. Revoke password version 1
7. Verify version 1 no longer works
8. Record the rotation result
```

The exact order depends on whether the backend can accept old and new credentials simultaneously.

### 51. Avoid Downtime

If the provider supports overlapping credentials:

```text
old valid + new valid
        |
move consumers to new
        |
verify
        |
revoke old
```

If it supports only one value, rotation may require a coordinated maintenance or reload strategy.

### 52. Immutable Version Names

A common pattern uses versioned names:

```text
database_password_v1
database_password_v2
```

Consumers move to the new version, then the old version is revoked and removed. This makes the transition explicit and supports rollback while both versions are temporarily valid.

### 53. Rotation Requires Application Behavior

An application may read a secret:

- Once at startup
- On every request
- Periodically
- When it receives a reload signal
- Through a client library that refreshes credentials

Changing a secret file does not guarantee the application rereads it. Know the consumer's behavior and test the full rotation path.

### 54. Short-Lived Credentials

Production secret managers may issue credentials that expire automatically. Short-lived credentials reduce the useful lifetime of a stolen value, although they require reliable renewal and failure handling.

This is an architectural direction, not a requirement for the beginner lab.

---

## Part 10 — Docker Compose Secrets Introduction

### 55. Why Introduce Compose Secrets Here?

A multi-container application may have several services and several secrets. Docker Compose lets the project declare secrets centrally and grant them only to selected services.

This section introduces the model. Full Docker Compose syntax and operations come later in the course.

### 56. Conceptual Compose File

Project structure:

```text
compose-secret-demo/
├── compose.yaml
└── secrets/
    └── db_password.txt
```

Example `compose.yaml`:

```yaml
services:
  app:
    image: alpine:3.22
    command:
      - sh
      - -c
      - |
        test -s /run/secrets/db_password
        echo "secret file is available"
        sleep 3600
    secrets:
      - db_password

  public-worker:
    image: alpine:3.22
    command: ["sleep", "3600"]

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 57. The Grant Matters

The top-level section defines the secret source:

```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

The service-level section grants access:

```yaml
services:
  app:
    secrets:
      - db_password
```

`public-worker` does not declare the secret, so it should not receive it.

### 58. Default Container Path

The consuming service normally receives the secret as:

```text
/run/secrets/db_password
```

The application reads the file instead of expecting the secret value in an ordinary environment variable.

### 59. Compose Secret Source Versus Production Secret Manager

In the simple local example, Compose reads a host file. That improves explicit service-level delivery, but the source file must still be protected.

```text
Compose secret declaration != automatically centralized enterprise vault
```

Production environments may integrate deployment systems with dedicated secret stores.

### 60. Optional Local Demonstration

If Docker Compose is available, verify it:

```bash
docker compose version
```

Use only a fake lab value. From the project directory:

```bash
docker compose up -d
```

Verify the authorized service:

```bash
docker compose exec app \
  test -s /run/secrets/db_password
```

Verify the unauthorized service does not have it:

```bash
docker compose exec public-worker \
  test ! -e /run/secrets/db_password
```

Cleanup:

```bash
docker compose down
```

The full Compose workflow will be taught later. Do not worry if you have not yet practiced project names, service networks, or Compose lifecycle commands.

---

## Part 11 — Docker Swarm Secrets Introduction

### 61. What Docker Swarm Adds

Docker Swarm has a secret-management mechanism for Swarm services. At a high level:

- A secret is stored in the Swarm manager's encrypted Raft log.
- Secrets are encrypted in transit.
- Access is granted to specific services.
- The secret is made available only to tasks for authorized services.
- Linux service containers normally receive it under `/run/secrets`.

### 62. Important Scope Boundary

Swarm secrets are for Swarm services, not ordinary standalone containers created only with `docker run`.

```text
docker run container      -> no Swarm service-secret grant
docker service workload   -> can receive an authorized Swarm secret
```

Do not initialize Swarm merely to complete this beginner lesson. A later orchestration lesson can teach service creation and operational consequences properly.

### 63. Why the Distinction Matters

The word “Docker secret” may refer to different contexts:

| Context | Purpose |
|---|---|
| BuildKit build secret | Temporary secret needed during image build |
| Docker Compose secret | Declare and grant a runtime secret to Compose services |
| Docker Swarm secret | Managed secret delivered to authorized Swarm service tasks |
| Bind-mounted file | Manual runtime file delivery; not itself a secret manager |

Always ask which mechanism and lifecycle are being discussed.

---

## Part 12 — Production Secret Managers

### 64. Why Teams Use Dedicated Systems

As environments grow, plain host files become difficult to manage safely. Production secret managers can provide capabilities such as:

- Centralized storage
- Encryption at rest and in transit
- Identity-based access policies
- Audit logs
- Versioning
- Rotation workflows
- Short-lived credentials
- Revocation
- Integration with cloud and orchestration platforms

### 65. Common Categories

Examples include:

- Cloud-provider secret services
- Dedicated vault systems
- Kubernetes-integrated secret delivery systems
- Hardware-backed key-management systems
- CI/CD protected secret stores

This lesson intentionally does not select a vendor. The correct choice depends on infrastructure, compliance requirements, authentication model, availability needs, and operational maturity.

### 66. A Secret Manager Does Not Fix Unsafe Application Behavior

Even a strong secret manager cannot help if an application immediately prints the retrieved value into logs.

End-to-end safety requires:

```text
secure storage
    + correct authorization
    + controlled delivery
    + safe application use
    + rotation and revocation
```

### 67. Avoid Permanent Master Credentials

Prefer workload identity and short-lived credentials when the platform supports them. This reduces the need to distribute long-lived static secrets.

However, identity systems also require careful design. “Passwordless” does not mean “security-free”; it changes how trust is established.

---

## Part 13 — Exposure and Incident Response

### 68. Assume an Exposed Secret Is Compromised

If a real credential appears in:

- Git history
- A public registry image
- CI logs
- Chat
- A ticket
- Shell history on a shared system
- Monitoring output

do not simply delete the visible copy and continue using the credential.

### 69. Immediate Response

A general response is:

```text
1. Stop further exposure
2. Revoke or rotate the credential
3. Deploy the replacement safely
4. Verify consumers work
5. Review access and audit evidence
6. Remove unsafe copies where possible
7. Identify and correct the root cause
```

Revocation or rotation is the critical step because copies may already exist elsewhere.

### 70. Git Exposure

Deleting a secret from the newest commit does not necessarily remove it from repository history, forks, clones, caches, or CI logs.

First rotate the credential. Then follow the repository host's approved history-cleaning and incident-response process.

### 71. Image Exposure

If an image contains a secret:

1. Revoke or rotate the secret.
2. Stop distributing the affected image.
3. Build a clean image using a safe method.
4. Replace affected deployments.
5. Review registry, cache, and host copies.
6. Scan related image versions.
7. Document the root cause.

Deleting one tag does not prove every image copy or layer is gone.

### 72. Never Share the Secret During Investigation

Record safe metadata:

```text
secret name: product-api/database
version: 7
exposure time: 2026-07-13T08:00Z
status: revoked
```

Do not paste the secret value into the incident report.

---

## Commands — Detailed Reference

### Create a Protected Lab Directory

```bash
mkdir -p "$HOME/docker-secret-lab"
chmod 700 "$HOME/docker-secret-lab"
```

### Restrict Newly Created Files in the Current Shell

```bash
umask 077
```

### Restrict an Existing File

```bash
chmod 600 secret-file
```

### Mount One File Read-Only

```bash
docker run \
  --mount type=bind,source=/host/secret,target=/run/secrets/secret,readonly \
  IMAGE
```

This is manual file delivery, not a production secret manager.

### Verify a Secret File Exists Without Printing It

```bash
docker exec CONTAINER \
  test -s /run/secrets/secret
```

### Inspect Environment Configuration

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  CONTAINER
```

Use care: this may display sensitive environment values.

### Inspect Container Command Arguments

```bash
docker inspect \
  --format '{{json .Config.Cmd}}' \
  CONTAINER
```

### Inspect Mount Metadata

```bash
docker inspect \
  --format '{{json .Mounts}}' \
  CONTAINER
```

### Docker Compose Secret Grant

```yaml
services:
  app:
    secrets:
      - api_token

secrets:
  api_token:
    file: ./secrets/api_token.txt
```

### Build Secret Introduction

```bash
docker build --secret id=mysecret,src=secret.txt .
```

Requires matching BuildKit secret-mount use in the Dockerfile. Do not replace this with `ARG` or `ENV` for a real build secret.

---

## Discovery Questions

### Question 1

Is a database hostname a secret?

<details>
<summary>Answer</summary>

Usually it is ordinary configuration because knowing the location does not itself authenticate a user. It may still be internal or sensitive information depending on the environment.

</details>

### Question 2

Is a TLS certificate a secret?

<details>
<summary>Answer</summary>

Normally the certificate is public. Its corresponding private key is secret.

</details>

### Question 3

Why is `COPY secret.txt /app/secret.txt` unsafe even if a later layer deletes it?

<details>
<summary>Answer</summary>

The added content may remain available in an earlier immutable image layer even though it disappears from the final filesystem view.

</details>

### Question 4

Does moving a token from `docker run -e` into a plain `.env` file make it secure?

<details>
<summary>Answer</summary>

No. It can reduce command-line exposure, but the source is still plain text and the resulting container environment may remain inspectable.

</details>

### Question 5

Does a read-only secret file stop an authorized process from reading and leaking it?

<details>
<summary>Answer</summary>

No. Read-only prevents modification through the mount. The authorized process can still read the value, so application security and least privilege remain essential.

</details>

### Question 6

Will every application automatically read `PASSWORD_FILE`?

<details>
<summary>Answer</summary>

No. The application or its entrypoint must explicitly support that variable or file path.

</details>

### Question 7

Why should an exposed credential be rotated instead of merely deleted from a log?

<details>
<summary>Answer</summary>

Because unauthorized copies may already exist. Rotation or revocation makes the exposed value unusable.

</details>

### Question 8

Should every container on one host receive the same secrets?

<details>
<summary>Answer</summary>

No. Each workload should receive only the exact secrets required for its function.

</details>

---

## Practical Scenario — A Token Found in an Image

### Incident

A security scanner reports that `techcorp/report-api:4.2.0` contains a cloud API token in:

```text
/app/config/production.env
```

The Dockerfile used:

```dockerfile
COPY . /app
```

The developer's build directory included `production.env`.

### Incorrect Response

```text
Delete production.env in a new Dockerfile layer and keep using the same token.
```

This fails because:

- The secret may remain in an earlier image layer.
- The image may already exist on registries and hosts.
- The credential may already have been copied.
- The token remains valid.

### Correct Response

#### 1. Treat the token as exposed

Do not print or copy it into the incident ticket.

#### 2. Revoke or rotate it

Coordinate with the credential provider immediately.

#### 3. Stop deploying the affected image

Identify containers and environments using it.

#### 4. Correct the build context

Add an appropriate `.dockerignore` rule and remove secrets from the source tree used for image builds.

Example:

```dockerignore
*.env
secrets/
*.pem
```

Rules must match the project's real file layout. Do not ignore required public files accidentally.

#### 5. Rebuild from a clean source

Use runtime secret delivery or a secure build secret mount depending on why the credential is needed.

#### 6. Scan and verify

Check the new image and deployment without assuming the fix worked.

#### 7. Replace affected containers

Deploy the clean image and new credential through the approved mechanism.

#### 8. Investigate scope

Review registry access, build logs, CI artifacts, caches, and audit evidence.

### Root Cause

A broad `COPY . /app` instruction included a secret-bearing file from the build context.

### Prevention

- Keep secrets outside the build context.
- Use `.dockerignore` as an additional barrier.
- Scan source and images for secrets.
- Use approved runtime and build-time secret mechanisms.
- Review Dockerfiles and CI pipelines.
- Rotate credentials regularly.

`.dockerignore` is important, but it is not permission to keep real secrets casually mixed with application source.

---

## Troubleshooting Workflow

### Problem 1 — Application Reports Secret File Missing

Check the expected path without printing content:

```bash
docker exec CONTAINER \
  ls -l /run/secrets
```

```bash
docker exec CONTAINER \
  test -s /run/secrets/EXPECTED_NAME
```

Then verify mount or Compose configuration:

```bash
docker inspect \
  --format '{{json .Mounts}}' \
  CONTAINER
```

Check:

- Secret name
- Target filename
- Service-level grant
- Host source path
- Container user permissions
- Whether the container was recreated

### Problem 2 — Permission Denied

Identify the application user:

```bash
docker exec CONTAINER id
```

Inspect file metadata:

```bash
docker exec CONTAINER \
  ls -ln /run/secrets/SECRET_NAME
```

Do not solve the problem by making the secret world-readable without understanding the user, group, mount, and platform model.

### Problem 3 — Application Still Uses Old Credential

Check whether the application:

- Reads only at startup
- Supports live reload
- Caches connections
- Needs container recreation
- Received the new secret version

Verify the new credential through application health and backend authentication, not by logging its value.

### Problem 4 — Secret Is Visible in `docker inspect`

Determine whether it was supplied as:

- An environment variable
- A command argument
- Image `ENV`

Treat any real exposed value according to incident policy. Rotate it, then move delivery to the approved mechanism.

### Problem 5 — Secret Was Committed to Git

1. Rotate or revoke it first.
2. Stop further distribution.
3. Review repository and CI access.
4. Clean history using the repository host's approved procedure.
5. Add prevention controls.

Do not delay rotation while attempting perfect history cleanup.

### Problem 6 — Container Has More Secrets Than Expected

Review service grants and mounts. Remove unnecessary secrets and recreate or redeploy the workload.

This is a least-privilege failure even if no exposure has yet been detected.

---

## Common Mistakes

### Mistake 1 — Treating Every Setting as a Secret

Over-classification makes systems difficult to operate. Classify values based on what unauthorized access enables.

### Mistake 2 — Treating Every `.env` File as Secure

The filename does not provide encryption, authorization, audit, or rotation.

### Mistake 3 — Baking Secrets into Dockerfile `ENV`

Image metadata can expose the value and every image copy inherits the risk.

### Mistake 4 — Copying a Secret and Deleting It Later

Earlier image layers may retain the content.

### Mistake 5 — Passing Real Secrets as CLI Arguments

They may appear in history, logs, process arguments, or Docker inspection.

### Mistake 6 — Printing `printenv` in Production

The output may contain credentials unrelated to the current investigation.

### Mistake 7 — Giving Every Service Every Secret

One compromised service could expose unrelated systems.

### Mistake 8 — Assuming Read-Only Means Unreadable

Read-only means the consumer cannot modify the mounted content. It must still be able to read it.

### Mistake 9 — Updating the Secret Store but Not the Consumer

The application may continue using a cached or startup-loaded value.

### Mistake 10 — Deleting Instead of Rotating

Removing visible evidence does not invalidate copies already obtained.

### Mistake 11 — Logging Secret Checks with the Value

Log that a required secret was loaded, not the secret itself.

### Mistake 12 — Calling a Bind Mount a Secret Manager

A read-only bind mount demonstrates file delivery but lacks many production lifecycle controls.

---

## Best Practices

1. Classify values before choosing how to store and deliver them.
2. Keep secrets out of source code, Dockerfiles, build contexts, image layers, and image metadata.
3. Never use Dockerfile `ARG` or `ENV` for real build secrets.
4. Use secure build secret mounts when a build genuinely needs temporary credentials.
5. Prefer controlled runtime secret files over ordinary environment variables when supported.
6. Grant each service only the secrets it needs.
7. Deliver secret files read-only unless write access is explicitly required.
8. Run applications as an appropriate non-root user.
9. Do not print secrets in logs, errors, health checks, or debugging output.
10. Protect local lab/source files with restrictive permissions.
11. Use an approved production secret manager for centralized storage and lifecycle controls.
12. Plan and test rotation before an emergency.
13. Revoke exposed credentials immediately.
14. Prefer versioned or short-lived credentials when supported.
15. Audit secret access and service grants.
16. Keep real secrets out of Git; commit only safe templates.
17. Scan source, build contexts, images, and CI logs for accidental exposure.
18. Verify the consuming application can reload or receive new secret versions safely.

---

## Interview Questions

### 1. What is the difference between configuration and a secret?

Configuration controls application behavior. A secret is sensitive data that grants access, proves identity, signs, or decrypts.

### 2. Why should secrets not be stored in Docker images?

Images are distributed, cached, inspectable artifacts. A secret can remain in layers or metadata and spread to every image copy.

### 3. Why is deleting a secret in a later image layer insufficient?

The content may remain in the earlier layer where it was originally added.

### 4. Why can command-line secret arguments be dangerous?

They may be visible through shell history, process listings, auditing, logs, and container inspection.

### 5. Why are ordinary environment variables not automatically secure?

They may be visible through Docker inspection, application diagnostics, crash reports, child processes, or deployment metadata.

### 6. What is a secret-file pattern?

The secret is delivered at runtime as a controlled file, commonly under `/run/secrets`, and the application reads it without embedding it in the image.

### 7. Does Docker automatically support variables ending in `_FILE`?

No. The image or application must implement that convention.

### 8. What does least privilege mean for secrets?

Only the required workload identity receives the exact secret and access needed for the shortest necessary time.

### 9. What is secret rotation?

It is the controlled process of creating a replacement credential, updating and verifying consumers, and revoking the old credential.

### 10. What should you do if a secret is committed to Git?

Revoke or rotate it immediately, then contain distribution, investigate access, clean history appropriately, and correct the process that caused the exposure.

### 11. How do Compose secrets apply least privilege?

A secret is declared at the project level but must be explicitly granted to each service that needs it.

### 12. Can a standalone `docker run` container consume a Swarm secret?

Not through the Swarm service-secret mechanism. Swarm secrets are delivered to authorized Swarm service tasks.

### 13. What is the difference between a bind-mounted secret file and a secret manager?

A bind mount delivers a host file. A secret manager normally adds controlled storage, identity-based access, audit, versioning, rotation, and other lifecycle capabilities.

### 14. Why might short-lived credentials be safer?

An exposed value automatically loses usefulness after a limited time, reducing the exposure window.

---

## Lesson Reflection

You should now be able to explain:

```text
Ordinary configuration
  tells the application how to behave

Secret
  grants access or proves identity
```

And this delivery model:

```text
secret manager or protected source
              |
              v
authorized workload only
              |
              v
runtime secret file
              |
              v
application reads without logging
```

The most important rule is:

```text
If a secret is exposed, remove its power through rotation or revocation.
Deleting the visible copy is not enough.
```

---

## Summary

- Configuration changes behavior; a secret grants privileged access or proves identity.
- Passwords, tokens, private keys, and encryption keys are secrets.
- Public certificates and public keys are normally not secret; their private keys are.
- Images are distributed artifacts and must not contain secrets.
- Deleting a copied secret in a later layer may leave it in an earlier layer.
- Dockerfile `ARG` and `ENV` are not safe build-secret mechanisms.
- CLI arguments and ordinary environment variables can expose sensitive values.
- Runtime secret files reduce some exposure paths and keep values out of images.
- A read-only bind mount is a useful lab pattern but not a complete secret manager.
- Least privilege means granting each service only the secrets it requires.
- Rotation creates and deploys a replacement before the old value is revoked.
- Compose secrets declare sources and grant access to selected services.
- Swarm secrets are designed for Swarm services, not ordinary standalone containers.
- Production secret managers add controlled storage, identity, auditing, versioning, and rotation.
- An exposed credential must be rotated or revoked.

---

## Cheat Sheet

### Classification

```text
APP_ENV=production       -> ordinary configuration
LOG_LEVEL=info           -> ordinary configuration
DATABASE_HOST=db01       -> usually configuration
DATABASE_PASSWORD=...    -> secret
API_TOKEN=...            -> secret
TLS certificate          -> normally public
TLS private key          -> secret
```

### Unsafe Image Patterns

```dockerfile
ENV API_TOKEN=real-token
COPY production.env /app/
COPY private-key.pem /app/
```

### Unsafe Runtime Patterns

```bash
docker run IMAGE --api-token real-token
docker run -e API_TOKEN=real-token IMAGE
```

These forms can expose real values through arguments, history, logs, environment metadata, or inspection.

### Protected Lab File

```bash
mkdir -p "$HOME/docker-secret-lab"
chmod 700 "$HOME/docker-secret-lab"
umask 077
printf '%s' 'FAKE-LAB-VALUE' > "$HOME/docker-secret-lab/secret"
```

### Read-Only File Delivery

```bash
docker run -d \
  --name secret-demo \
  --mount type=bind,source="$HOME/docker-secret-lab/secret",target=/run/secrets/secret,readonly \
  alpine:3.22 sleep 3600
```

### Verify Without Printing

```bash
docker exec secret-demo \
  test -s /run/secrets/secret
```

### Check File Metadata

```bash
docker exec secret-demo \
  ls -ln /run/secrets/secret
```

### Inspect Possible Environment Exposure

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  CONTAINER
```

### Inspect Possible Argument Exposure

```bash
docker inspect \
  --format '{{json .Config.Cmd}}' \
  CONTAINER
```

### Compose Secret Pattern

```yaml
services:
  app:
    secrets:
      - database_password

secrets:
  database_password:
    file: ./secrets/database_password.txt
```

Default path in the authorized service:

```text
/run/secrets/database_password
```

### Rotation Pattern

```text
create new -> authorize/deliver -> update consumer
-> verify -> revoke old -> audit
```

### Exposure Response

```text
contain -> rotate/revoke -> deploy replacement
-> verify -> investigate -> prevent recurrence
```

---

## Challenge — Design Secret Access for TechCorp

### Scenario

TechCorp runs four containers:

```text
public-web
product-api
billing-worker
log-collector
```

Available values:

```text
APP_ENV=production
LOG_LEVEL=info
PRODUCT_DATABASE_PASSWORD
BILLING_API_TOKEN
TLS_CERTIFICATE
TLS_PRIVATE_KEY
```

### Your Tasks

1. Classify every value as ordinary configuration, public security material, or secret.
2. Decide which service receives each value.
3. Explain why unrelated services must not receive it.
4. Design a file path for every runtime secret.
5. Mark each secret mount read-only.
6. Decide which non-secret values can use environment variables.
7. Create an access matrix.
8. Design a rotation sequence for `BILLING_API_TOKEN`.
9. Write an exposure response if the TLS private key appears in CI logs.
10. Explain what a production secret manager adds beyond a bind-mounted file.

### Acceptance Criteria

- No secret is baked into an image.
- No real secret value appears in the answer.
- Every secret has only the required service grants.
- Public certificate and private key are classified differently.
- Rotation includes creation, consumer update, verification, and old-value revocation.
- Exposure response begins with containment and prompt rotation/revocation.
- The design distinguishes file delivery from secret management.

### Example Access-Matrix Shape

Complete the values yourself:

| Value | Classification | Public Web | Product API | Billing Worker | Log Collector | Delivery |
|---|---|---:|---:|---:|---:|---|
| `APP_ENV` | ? | ? | ? | ? | ? | ? |
| `PRODUCT_DATABASE_PASSWORD` | ? | ? | ? | ? | ? | ? |
| `TLS_PRIVATE_KEY` | ? | ? | ? | ? | ? | ? |

---

## Lab Assets

Suggested paths:

```text
assets/chapter-07/lesson-15/
├── 01-protected-host-file.png
├── 02-secret-file-mounted.png
├── 03-read-only-write-failure.png
├── 04-env-inspection-comparison.png
├── 05-compose-service-grants.png
├── secret-access-matrix.md
├── rotation-plan.md
└── incident-response-notes.md
```

Capture only safe evidence. Never place a real secret value in a screenshot, filename, terminal recording, lab note, or repository.

---

## Official References

- [Docker Compose: Use secrets](https://docs.docker.com/compose/how-tos/use-secrets/)
- [Docker Compose file reference: Secrets](https://docs.docker.com/reference/compose-file/secrets/)
- [Docker Swarm secrets](https://docs.docker.com/engine/swarm/secrets/)
- [Docker build secrets](https://docs.docker.com/build/building/secrets/)
- [Docker build variables](https://docs.docker.com/build/building/variables/)

---

## Next Lesson

**Lesson 16 — Dockerfile Fundamentals**

You now know which information belongs outside an image. Next, you will learn how to define the image itself with a Dockerfile.

You will learn:

```dockerfile
FROM
WORKDIR
COPY
RUN
ENV
EXPOSE
USER
CMD
ENTRYPOINT
```

You will also learn:

- What each instruction changes
- Whether it acts at build time or runtime
- How it affects image layers or image metadata
- Common mistakes
- How `.dockerignore` protects and reduces the build context
- How to build and test a small application image

---

## Completion Check

Before moving forward, explain without reading:

```text
The difference between configuration and a secret
Why secrets must not be baked into images
Why deleting a secret in a later layer may be insufficient
The risks of CLI arguments and ordinary environment variables
What a runtime secret file is
Why a read-only bind mount is not a complete secret manager
What least privilege means for service secrets
The safe order of credential rotation
What Compose secrets introduce
What to do immediately after secret exposure
```

If you can explain these clearly and complete the lab, you are ready to build images safely in Lesson 16.

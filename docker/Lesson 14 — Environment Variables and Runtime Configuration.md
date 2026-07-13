# Chapter 07 — Docker for DevOps Engineers

# Lesson 14 — Environment Variables and Runtime Configuration

> **Lesson goal:** Keep configuration outside the container image so one tested image can run with different settings in development, testing, and production.

---

## Overview

An image should normally contain the application and the files it needs to run. It should not need a separate rebuild merely because a server address, log level, application mode, or feature setting changed.

Docker lets you pass environment variables when a container is created:

```text
Same image
   |
   +-- development container: APP_ENV=development, LOG_LEVEL=debug
   +-- testing container:     APP_ENV=testing,     LOG_LEVEL=info
   +-- production container:  APP_ENV=production,  LOG_LEVEL=warning
```

The application reads those values at runtime and changes its behavior without changing the image.

In this lesson, you will learn:

- What configuration means
- What an environment variable is
- The difference between an image default and a runtime override
- How to pass variables with `-e` and `--env`
- How to load multiple variables with `--env-file`
- How to inspect the environment of a running container
- Why editing an environment file does not update an existing container
- When to use an environment variable and when to mount a configuration file
- Why ordinary environment variables should not be treated as a complete secret-management system

---

## Learning Objectives

By the end of this lesson, you should be able to:

1. Explain the difference between application code and application configuration.
2. Pass one or more environment variables to a container.
3. read variables from inside a container using `printenv`.
4. Define safe defaults in an image with Dockerfile `ENV`.
5. Override an image default when creating a container.
6. Create and use a Docker environment file.
7. Prove that configuration is attached to a container at creation time.
8. Recreate a container when its runtime configuration changes.
9. Choose between environment variables and mounted configuration files.
10. Avoid exposing sensitive values through commands, inspection output, shell history, and source control.

---

## Prerequisites

Before beginning, you should understand:

- The difference between an image and a container
- `docker run`
- `docker exec`
- `docker inspect`
- Container names
- The difference between stopping and recreating a container
- Basic bind mounts from Lesson 12

Verify Docker:

```bash
docker version
```

If your user requires `sudo` for Docker commands, add `sudo` consistently in your own terminal.

---

## Part 1 — What Is Runtime Configuration?

### 1. Application Code Versus Configuration

Application code defines what the application can do.

Configuration tells that application how it should behave in a particular environment.

Examples of configuration include:

- Application mode: `development`, `testing`, or `production`
- Log level: `debug`, `info`, `warning`, or `error`
- Backend hostname
- Database hostname and port
- Feature switches
- Worker count
- Public URL
- Region name

Example:

```text
Application code: connect to a database
Configuration:    database host is db.internal and port is 5432
```

The code remains the same while the values may differ between environments.

### 2. The Problem with Baking Configuration into an Image

Suppose you build three different images:

```text
product-api:development
product-api:testing
product-api:production
```

If the only difference is a hostname or log level, you now have multiple image builds to test, scan, store, and track.

A cleaner model is:

```text
product-api:2.4.1
        |
        +-- container configured for development
        +-- container configured for testing
        +-- container configured for production
```

This gives you one image artifact and several runtime configurations.

### 3. Why This Matters in DevOps

A common delivery path is:

```text
Build once -> test image -> deploy same image -> supply environment configuration
```

If production receives exactly the image that passed testing, there is less uncertainty than rebuilding the application separately for every environment.

Configuration can still be incorrect, so it must also be reviewed and tested. The important point is that changing a setting does not require changing the application artifact.

---

## Part 2 — Environment Variable Fundamentals

### 4. What Is an Environment Variable?

An environment variable is a named value available to a process.

It usually has this form:

```text
NAME=value
```

Examples:

```text
APP_ENV=production
LOG_LEVEL=info
API_PORT=8080
FEATURE_REPORTS=true
```

The name identifies the setting. The value contains the setting supplied to the process.

### 5. Environment Variables Belong to Processes

An environment variable is not a magical global Docker database.

Docker prepares an environment for the container's main process. Child processes normally inherit environment values from their parent process unless something changes them.

Introductory model:

```text
docker run -e APP_ENV=testing
              |
              v
container main process
APP_ENV=testing
              |
              v
child process usually inherits APP_ENV=testing
```

The application must be written or configured to read the variable. Passing `LOG_LEVEL=debug` does nothing if the application never reads `LOG_LEVEL`.

### 6. Variable Names and Values

Use clear, consistent names. Uppercase with underscores is common:

```text
APP_ENV
LOG_LEVEL
DATABASE_HOST
DATABASE_PORT
```

Environment variable values are delivered as text. The application decides how to interpret them.

For example:

```text
WORKERS=4
FEATURE_SEARCH=false
```

The application may convert `4` to a number and `false` to a Boolean value. Docker itself does not understand the application's meaning.

### 7. Variables Are Not Files

An environment variable is a named value:

```text
LOG_LEVEL=debug
```

A configuration file is a file with structured content:

```ini
[logging]
level=debug
format=json
```

Both can configure an application, but they are supplied and consumed differently.

---

## Part 3 — Pass One Variable with `-e`

### 8. Start a Simple Container

Run Alpine and pass `APP_ENV`:

```bash
docker run --rm \
  --name env-once \
  -e APP_ENV=development \
  alpine:3.22 printenv APP_ENV
```

Expected output:

```text
development
```

Command breakdown:

```text
docker run                  create and start a container
--rm                        remove it after the command exits
--name env-once             assign a readable name
-e APP_ENV=development      set one environment variable
alpine:3.22                 use this image
printenv APP_ENV            print the value inside the container
```

### 9. Long Option Form

`--env` is the long form of `-e`:

```bash
docker run --rm \
  --env APP_ENV=testing \
  alpine:3.22 printenv APP_ENV
```

Both commands express the same idea:

```text
-e NAME=value
--env NAME=value
```

### 10. Pass Multiple Variables

Use one option for each variable:

```bash
docker run --rm \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  -e API_PORT=8080 \
  alpine:3.22 printenv
```

The full output includes variables supplied by you and other environment values already provided by the image or container runtime.

Filter only the variables for this lab:

```bash
docker run --rm \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  -e API_PORT=8080 \
  alpine:3.22 sh -c 'printenv | sort | grep -E "^(APP_ENV|LOG_LEVEL|API_PORT)="'
```

Expected:

```text
API_PORT=8080
APP_ENV=production
LOG_LEVEL=info
```

The single quotes are important here: they cause the command to be evaluated by the shell inside the container rather than expanding container variable references in your host shell.

### 11. Spaces in Values

Quote a value containing spaces so your host shell treats it as one argument:

```bash
docker run --rm \
  -e 'WELCOME_MESSAGE=Hello from the testing environment' \
  alpine:3.22 printenv WELCOME_MESSAGE
```

Expected:

```text
Hello from the testing environment
```

### 12. Empty Versus Missing

An empty variable exists but contains no characters:

```bash
docker run --rm \
  -e EMPTY_VALUE= \
  alpine:3.22 sh -c 'printenv EMPTY_VALUE; printenv EMPTY_VALUE >/dev/null; echo "status=$?"'
```

A missing variable is not present:

```bash
docker run --rm \
  alpine:3.22 sh -c 'printenv NOT_DEFINED; echo "status=$?"'
```

Applications may treat an empty value and a missing value differently.

---

## Part 4 — Read Variables Inside a Running Container

### 13. Create a Long-Running Container

```bash
docker run -d \
  --name env-lab \
  -e APP_ENV=testing \
  -e LOG_LEVEL=debug \
  alpine:3.22 sleep 3600
```

### 14. Read One Variable

```bash
docker exec env-lab printenv APP_ENV
```

Expected:

```text
testing
```

Read another:

```bash
docker exec env-lab printenv LOG_LEVEL
```

Expected:

```text
debug
```

### 15. Read the Whole Environment

```bash
docker exec env-lab printenv
```

For sorted output:

```bash
docker exec env-lab sh -c 'printenv | sort'
```

Do not assume every displayed variable was manually supplied with `-e`. Some values may come from the base image or Docker's container setup.

### 16. Shell Expansion Trap

This command uses double quotes:

```bash
docker exec env-lab sh -c "echo $APP_ENV"
```

Your **host shell** tries to expand `$APP_ENV` before Docker sends the command into the container. If the host variable is absent, the container may receive only `echo`.

Use single quotes when you want the container shell to expand the variable:

```bash
docker exec env-lab sh -c 'echo "$APP_ENV"'
```

Expected:

```text
testing
```

Remember:

```text
"$APP_ENV" in the host command -> host may expand it
'$APP_ENV' in the host command -> text reaches container shell for expansion
```

The exact quoting needs depend on which shell should interpret the value.

---

## Part 5 — Inspect Container Configuration

### 17. Inspect the Environment Array

Docker stores the environment configured for the container.

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  env-lab
```

You should find:

```text
APP_ENV=testing
LOG_LEVEL=debug
```

You will also see other values inherited from the image.

### 18. Inspect as JSON

```bash
docker inspect \
  --format '{{json .Config.Env}}' \
  env-lab
```

Typical shape:

```json
["APP_ENV=testing","LOG_LEVEL=debug","PATH=..."]
```

### 19. Why Inspection Matters

Inspection helps answer:

- Was the variable attached to this container?
- Is the name spelled correctly?
- Did the container receive the intended value?
- Did an image default remain active?
- Is the problem in Docker configuration or application behavior?

Evidence-first troubleshooting:

```text
Expected variable
      |
      v
Check docker inspect
      |
      +-- absent/wrong -> container creation configuration problem
      |
      +-- correct -> check process environment and application behavior
```

### 20. Security Consequence

Environment values in container configuration can be visible through inspection to users or systems with sufficient Docker access.

Therefore, do not think:

```text
It is hidden because it is inside an environment variable.
```

Environment variables are convenient configuration channels, not automatic encryption or complete secret protection.

---

## Part 6 — Image Defaults with Dockerfile `ENV`

### 21. Why an Image May Provide Defaults

An image author can define a reasonable default:

```dockerfile
ENV APP_ENV=production
ENV LOG_LEVEL=info
```

Containers created from the image receive those values unless a later configuration source overrides them.

Good defaults make an image easy to start while still allowing operators to change environment-specific behavior.

### 22. Create the Lab Dockerfile

Create a directory:

```bash
mkdir -p "$HOME/docker-env-image-lab"
cd "$HOME/docker-env-image-lab"
```

Create `Dockerfile`:

```dockerfile
FROM alpine:3.22

ENV APP_ENV=production
ENV LOG_LEVEL=info

CMD ["sh", "-c", "printf 'APP_ENV=%s\nLOG_LEVEL=%s\n' \"$APP_ENV\" \"$LOG_LEVEL\""]
```

### 23. Build the Image

```bash
docker build -t env-demo:1.0 .
```

### 24. Run with Image Defaults

```bash
docker run --rm env-demo:1.0
```

Expected:

```text
APP_ENV=production
LOG_LEVEL=info
```

The values came from Dockerfile `ENV` instructions stored in the image configuration.

### 25. Override One Default

```bash
docker run --rm \
  -e LOG_LEVEL=debug \
  env-demo:1.0
```

Expected:

```text
APP_ENV=production
LOG_LEVEL=debug
```

Only `LOG_LEVEL` changed. `APP_ENV` retained its image default.

### 26. Override Both Defaults

```bash
docker run --rm \
  -e APP_ENV=testing \
  -e LOG_LEVEL=warning \
  env-demo:1.0
```

Expected:

```text
APP_ENV=testing
LOG_LEVEL=warning
```

Introductory precedence model for this lesson:

```text
Dockerfile ENV default
         |
         v
docker run -e runtime value wins for that container
```

### 27. `ENV` Versus `ARG`

Do not confuse Dockerfile `ENV` with Dockerfile `ARG`:

| Instruction | Main purpose | Available to the running container by default? |
|---|---|---:|
| `ARG` | Supply a build-time value | No |
| `ENV` | Set an image/container environment value | Yes |

Build arguments will be covered properly in the image-building section. For now, remember:

```text
ARG -> build-time input
ENV -> image default available at runtime
```

Neither should be assumed to be a safe way to embed secrets into an image.

---

## Part 7 — Use an Environment File

### 28. Why Use `--env-file`?

Passing many `-e` options becomes repetitive:

```bash
docker run \
  -e APP_ENV=testing \
  -e LOG_LEVEL=debug \
  -e API_HOST=api.test.internal \
  -e API_PORT=8080 \
  -e FEATURE_REPORTS=true \
  IMAGE
```

An environment file keeps the values together:

```text
APP_ENV=testing
LOG_LEVEL=debug
API_HOST=api.test.internal
API_PORT=8080
FEATURE_REPORTS=true
```

### 29. Create a Lab Environment File

Create a directory:

```bash
mkdir -p "$HOME/docker-env-lab"
cd "$HOME/docker-env-lab"
```

Create `testing.env`:

```dotenv
# Runtime configuration for the testing environment
APP_ENV=testing
LOG_LEVEL=debug
API_HOST=api.test.internal
API_PORT=8080
FEATURE_REPORTS=true
```

Basic rules used in this lab:

- Put one `NAME=value` entry on each line.
- Lines beginning with `#` can document the file.
- Keep names consistent with what the application expects.
- Do not write `export NAME=value` in this Docker env file.
- Avoid unnecessary shell syntax; an env file is not a general shell script.

### 30. Run with `--env-file`

```bash
docker run --rm \
  --env-file "$HOME/docker-env-lab/testing.env" \
  alpine:3.22 sh -c \
  'printf "environment=%s\nlog=%s\napi=%s:%s\nreports=%s\n" \
    "$APP_ENV" "$LOG_LEVEL" "$API_HOST" "$API_PORT" "$FEATURE_REPORTS"'
```

Expected:

```text
environment=testing
log=debug
api=api.test.internal:8080
reports=true
```

### 31. Create a Persistent Lab Container

```bash
docker run -d \
  --name configured-app \
  --env-file "$HOME/docker-env-lab/testing.env" \
  alpine:3.22 sleep 3600
```

Verify:

```bash
docker exec configured-app sh -c \
  'printf "%s | %s | %s:%s\n" "$APP_ENV" "$LOG_LEVEL" "$API_HOST" "$API_PORT"'
```

Expected:

```text
testing | debug | api.test.internal:8080
```

### 32. Environment-Specific Files

A team might maintain non-sensitive configuration such as:

```text
development.env
testing.env
production.env
```

Example:

```dotenv
# development.env
APP_ENV=development
LOG_LEVEL=debug
API_HOST=api.dev.internal
```

```dotenv
# production.env
APP_ENV=production
LOG_LEVEL=warning
API_HOST=api.prod.internal
```

The image can remain the same:

```bash
docker run --env-file development.env product-api:2.4.1
docker run --env-file production.env  product-api:2.4.1
```

These commands must use different names, ports, or separate hosts if both containers run simultaneously. They illustrate the configuration model, not a complete deployment design.

---

## Part 8 — Configuration Is Fixed When the Container Is Created

### 33. Edit the File on the Host

Change this line in `testing.env`:

```text
LOG_LEVEL=debug
```

to:

```text
LOG_LEVEL=warning
```

### 34. Check the Existing Container

```bash
docker exec configured-app printenv LOG_LEVEL
```

It still reports:

```text
debug
```

Why?

`--env-file` was read when Docker **created** the container. Docker copied the resulting environment values into the container configuration. It did not create a live connection between the host file and the running container.

```text
testing.env at creation time
          |
          v
container configuration: LOG_LEVEL=debug

later edit testing.env to warning
          |
          X  no automatic update
          |
existing container still has debug
```

### 35. Restart Is Not Recreate

Restart the container:

```bash
docker restart configured-app
```

Check again:

```bash
docker exec configured-app printenv LOG_LEVEL
```

It still reports:

```text
debug
```

A restart starts the same container with the same creation-time configuration.

### 36. Recreate with the New Configuration

Remove and create the container again:

```bash
docker rm -f configured-app
```

```bash
docker run -d \
  --name configured-app \
  --env-file "$HOME/docker-env-lab/testing.env" \
  alpine:3.22 sleep 3600
```

Verify:

```bash
docker exec configured-app printenv LOG_LEVEL
```

Expected:

```text
warning
```

Key rule:

```text
Changed creation-time environment configuration -> recreate container
```

---

## Part 9 — Host Variable Pass-Through

### 37. Pass a Host Variable by Name

Docker also accepts a variable name without writing its value in the command:

```bash
export RELEASE_CHANNEL=canary
```

```bash
docker run --rm \
  -e RELEASE_CHANNEL \
  alpine:3.22 printenv RELEASE_CHANNEL
```

Expected:

```text
canary
```

The host shell has `RELEASE_CHANNEL=canary`, and `-e RELEASE_CHANNEL` asks Docker to use that host value.

### 38. Be Careful with Hidden Dependencies

This is concise:

```bash
docker run -e RELEASE_CHANNEL IMAGE
```

But a reader cannot see the value in the command. Automated deployment systems should make the source of configuration clear and validate required values.

A useful check before running:

```bash
printenv RELEASE_CHANNEL
```

Do not dump the entire host environment into logs because it may contain sensitive data.

### 39. Unset the Host Lab Variable

```bash
unset RELEASE_CHANNEL
```

This removes it from your current host shell session. It does not change a container that was already created.

---

## Part 10 — Environment Variables Versus Mounted Configuration Files

### 40. When Environment Variables Fit Well

Environment variables work well for small, independent settings:

```text
APP_ENV=production
LOG_LEVEL=info
PORT=8080
WORKER_COUNT=4
FEATURE_REPORTS=true
```

They are easy to inject through Docker and common deployment platforms.

### 41. When a Configuration File Fits Better

A file may be better when configuration is:

- Large
- Hierarchical or deeply nested
- Already required in a particular format
- Managed by another host tool
- Difficult to express as separate variables
- Expected to include many related rules

Example Nginx-style structure:

```nginx
server {
    listen 8080;

    location /health {
        return 200 "healthy\n";
    }
}
```

That structure is more naturally represented as a file than as dozens of environment variables.

### 42. Mount a Configuration File

Suppose the host contains:

```text
$HOME/app-config/application.conf
```

Mount it read-only:

```bash
docker run -d \
  --name configured-service \
  --mount type=bind,source="$HOME/app-config/application.conf",target=/etc/myapp/application.conf,readonly \
  myapp:1.0
```

The exact target path and file format depend on the application.

### 43. Important Difference from `--env-file`

These two options sound similar but do different things:

| Mechanism | What Docker does |
|---|---|
| `--env-file settings.env` | Reads entries and adds them to the container environment at creation time |
| `--mount type=bind,...` | Makes a host file or directory visible at a container path |

An env file is not automatically mounted inside the container.

A bind-mounted file remains a mounted file. Host changes may become visible inside the container, but whether the application notices or reloads them depends on the application. Some programs require a reload or restart.

### 44. Decision Table

| Requirement | Typical choice | Reason |
|---|---|---|
| A few simple runtime values | Environment variables | Direct and portable |
| Many environment values | `--env-file` | Keeps invocation readable |
| Complex structured configuration | Read-only bind mount | Preserves file format |
| Persistent application data | Named volume | Data lifecycle is separate from configuration |
| Temporary in-memory data | `tmpfs` | Non-persistent temporary storage |
| Sensitive credential | Dedicated secret mechanism | Better controls than ordinary environment variables |

Do not use a volume merely because something is “outside the image.” Choose storage and configuration mechanisms based on the data's role and lifecycle.

---

## Part 11 — Sensitive Values and Secrets

### 45. Why Passwords Need Special Care

This command is technically possible:

```bash
docker run -e DB_PASSWORD=example-password myapp:1.0
```

But the value may be exposed through places such as:

- Shell history
- Process or command auditing
- Deployment logs
- `docker inspect` output
- CI/CD job output
- Shared environment files
- Accidental source-control commits

Do not use the example as a production recommendation.

### 46. An Environment File Is Not Automatically Secure

Moving a password from the command line into `production.env` improves readability but does not automatically make it secure.

The file is still plain text unless stronger controls protect it.

At minimum:

- Do not commit real secret files to Git.
- Restrict file permissions.
- Avoid printing sensitive values.
- Limit access to Docker and the deployment host.
- Use the secret-management mechanism provided by the deployment platform.
- Rotate exposed credentials.

Example restrictive permission:

```bash
chmod 600 production.env
```

This limits normal filesystem access but does not solve every secret-management problem.

### 47. `.gitignore` Is a Safety Net, Not a Vault

A project may include:

```gitignore
*.env
!.env.example
```

Then commit only a template:

```dotenv
# .env.example
APP_ENV=production
LOG_LEVEL=info
DATABASE_HOST=
DATABASE_PORT=5432
```

The template documents required names without containing real credentials.

Still verify staged files before committing:

```bash
git status
git diff --staged
```

### 48. Do Not Bake Secrets into Images

Avoid placing credentials in:

- Dockerfile `ENV`
- Dockerfile layers through `COPY`
- Application source files
- Image labels
- Public image metadata

Removing a secret in a later Dockerfile instruction does not necessarily remove it from earlier image layers or build history.

Secret management will be covered more deeply in the security and orchestration lessons.

---

## Part 12 — Application Design for Runtime Configuration

### 49. Use Clear Defaults

A well-designed application may use a safe default for a non-sensitive option:

```text
LOG_LEVEL defaults to info
```

The application can then allow a runtime override:

```text
LOG_LEVEL=debug
```

Avoid dangerous defaults that silently connect to the wrong service or disable security.

### 50. Validate Required Variables

If a value is essential, the application should fail clearly when it is missing.

Example shell check:

```sh
: "${API_HOST:?API_HOST is required}"
```

If `API_HOST` is missing or empty, the shell exits with an explanatory error.

This is better than starting an application that fails later with an unclear connection error.

### 51. Log Names, Not Sensitive Values

Useful startup log:

```text
Configuration loaded: APP_ENV, API_HOST, API_PORT
```

Risky startup log:

```text
Configuration loaded: DB_PASSWORD=actual-secret
```

Logs should help diagnose configuration without exposing credentials.

### 52. Document the Configuration Contract

For every supported variable, document:

| Field | Meaning |
|---|---|
| Name | Exact environment variable name |
| Required? | Whether startup can proceed without it |
| Default | Behavior when absent |
| Example | Safe example value |
| Sensitive? | Whether it must be protected |
| Validation | Allowed values or format |

Example:

| Variable | Required? | Default | Example | Sensitive? |
|---|---:|---|---|---:|
| `APP_ENV` | No | `production` | `testing` | No |
| `LOG_LEVEL` | No | `info` | `debug` | No |
| `API_HOST` | Yes | None | `api.internal` | No |
| `API_TOKEN` | Yes | None | Not documented | Yes |

---

## Part 13 — Commands: Detailed Reference

### `docker run -e`

Pass one runtime environment value:

```bash
docker run -e NAME=value IMAGE
```

Pass several values:

```bash
docker run \
  -e NAME_ONE=value-one \
  -e NAME_TWO=value-two \
  IMAGE
```

Pass the value of a variable from the host environment:

```bash
docker run -e NAME IMAGE
```

### `docker run --env-file`

Read environment entries from a host file when creating the container:

```bash
docker run --env-file PATH IMAGE
```

Example:

```bash
docker run --env-file ./testing.env myapp:1.0
```

### `docker exec ... printenv`

Read one value inside a running container:

```bash
docker exec CONTAINER printenv NAME
```

Read all visible environment values for the executed process:

```bash
docker exec CONTAINER printenv
```

### `docker inspect`

Inspect the environment stored in container configuration:

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  CONTAINER
```

### Dockerfile `ENV`

Set an image default:

```dockerfile
ENV NAME=value
```

Runtime `-e NAME=another-value` can override it for a newly created container.

---

## Guided Hands-on Lab — One Image, Two Environments

### Lab Goal

Build one small image and run it twice with different runtime configuration. Then change a configuration file and prove that the existing container must be recreated.

### Step 1 — Prepare the Directory

Host prompt:

```text
hamad@Hamooda:~$
```

Run:

```bash
mkdir -p "$HOME/docker-runtime-config-lab"
cd "$HOME/docker-runtime-config-lab"
```

### Step 2 — Create the Dockerfile

Create `Dockerfile`:

```dockerfile
FROM alpine:3.22

ENV APP_ENV=production
ENV LOG_LEVEL=info

CMD ["sh", "-c", "printf 'Starting app: env=%s log=%s region=%s\n' \"$APP_ENV\" \"$LOG_LEVEL\" \"${REGION:-not-set}\"; sleep 3600"]
```

The image contains generic defaults. It does not contain environment-specific hostnames or credentials.

### Step 3 — Build Once

```bash
docker build -t runtime-config-demo:1.0 .
```

Verify:

```bash
docker image ls runtime-config-demo:1.0
```

### Step 4 — Create `development.env`

```dotenv
APP_ENV=development
LOG_LEVEL=debug
REGION=local
```

### Step 5 — Create `production.env`

```dotenv
APP_ENV=production
LOG_LEVEL=warning
REGION=eu-central
```

### Step 6 — Start Development Container

```bash
docker run -d \
  --name demo-development \
  --env-file development.env \
  runtime-config-demo:1.0
```

Check its startup log:

```bash
docker logs demo-development
```

Expected:

```text
Starting app: env=development log=debug region=local
```

### Step 7 — Start Production Container

```bash
docker run -d \
  --name demo-production \
  --env-file production.env \
  runtime-config-demo:1.0
```

Check its startup log:

```bash
docker logs demo-production
```

Expected:

```text
Starting app: env=production log=warning region=eu-central
```

### Step 8 — Prove Both Use the Same Image

```bash
docker inspect \
  --format '{{.Name}} image={{.Config.Image}} image-id={{.Image}}' \
  demo-development demo-production
```

Both container objects are different, but both reference `runtime-config-demo:1.0` and should show the same image ID.

### Step 9 — Compare Their Runtime Values

```bash
docker exec demo-development sh -c \
  'printf "%s %s %s\n" "$APP_ENV" "$LOG_LEVEL" "$REGION"'
```

```bash
docker exec demo-production sh -c \
  'printf "%s %s %s\n" "$APP_ENV" "$LOG_LEVEL" "$REGION"'
```

Expected:

```text
development debug local
```

and:

```text
production warning eu-central
```

### Step 10 — Change Development Configuration

Edit `development.env`:

```dotenv
APP_ENV=development
LOG_LEVEL=info
REGION=local
```

You changed `LOG_LEVEL` from `debug` to `info`.

### Step 11 — Prove the Existing Container Did Not Change

```bash
docker exec demo-development printenv LOG_LEVEL
```

Expected:

```text
debug
```

Restart it:

```bash
docker restart demo-development
```

Check again:

```bash
docker exec demo-development printenv LOG_LEVEL
```

It is still:

```text
debug
```

### Step 12 — Recreate Development Container

```bash
docker rm -f demo-development
```

```bash
docker run -d \
  --name demo-development \
  --env-file development.env \
  runtime-config-demo:1.0
```

Verify:

```bash
docker exec demo-development printenv LOG_LEVEL
```

Expected:

```text
info
```

### Step 13 — Prove the Production Container Was Unaffected

```bash
docker exec demo-production printenv LOG_LEVEL
```

Expected:

```text
warning
```

### Step 14 — Cleanup

```bash
docker rm -f demo-development demo-production
```

Optional image removal:

```bash
docker image rm runtime-config-demo:1.0
```

Keep the lab files if you want to repeat the exercise.

---

## Discovery Questions

Answer these before reading the explanations.

### Question 1

Why can two containers created from the same image have different `LOG_LEVEL` values?

<details>
<summary>Answer</summary>

Because the image can provide a default while each container receives its own runtime configuration when it is created.

</details>

### Question 2

Does `--env-file testing.env` mount `testing.env` inside the container?

<details>
<summary>Answer</summary>

No. Docker reads the entries during container creation and places the resulting values in the container environment.

</details>

### Question 3

Why does editing `testing.env` not update an existing container?

<details>
<summary>Answer</summary>

The file was read at creation time. The existing container already has its own stored configuration and no live connection to that file.

</details>

### Question 4

Will `docker restart` read the env file again?

<details>
<summary>Answer</summary>

No. Restart starts the same container with its existing creation-time environment. Recreate the container to apply new values.

</details>

### Question 5

Why might `docker exec env-lab sh -c "echo $APP_ENV"` print the wrong value or a blank line?

<details>
<summary>Answer</summary>

The host shell may expand `$APP_ENV` before Docker sends the command to the container. Single quotes allow the container shell to perform the expansion.

</details>

### Question 6

Does passing `FEATURE_REPORTS=true` guarantee the feature is enabled?

<details>
<summary>Answer</summary>

No. The application must read that exact variable and interpret the text `true` as intended.

</details>

### Question 7

Is a plain `.env` file a secret manager?

<details>
<summary>Answer</summary>

No. It is normally a plain-text configuration file. Sensitive values require controlled access and an appropriate secret-management solution.

</details>

---

## Production Scenario — Correct Image, Wrong Backend

### Incident

`product-api:2.4.1` passed testing. After deployment, the production container tries to contact:

```text
api.test.internal:8080
```

The expected production endpoint is:

```text
api.prod.internal:8080
```

### Evidence-First Investigation

#### 1. Identify the container

```bash
docker ps --filter name=product-api
```

#### 2. Inspect its stored environment

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  product-api
```

Relevant evidence:

```text
API_HOST=api.test.internal
API_PORT=8080
```

#### 3. Confirm from inside

```bash
docker exec product-api sh -c \
  'printf "%s:%s\n" "$API_HOST" "$API_PORT"'
```

#### 4. Check the intended deployment source

Review the production environment file or deployment configuration without printing unrelated secrets.

#### 5. Correct and recreate

After correcting the source configuration, recreate the container with the approved image and correct runtime values.

#### 6. Verify

Check:

- Container environment
- Application logs
- Health endpoint
- Backend connectivity
- User-facing behavior

### Root Cause

The image was correct. The container was created with testing configuration.

### Prevention

- Validate required values before deployment.
- Keep environment configuration clearly separated.
- Review deployment differences.
- Use health checks and post-deployment verification.
- Avoid ambiguous file names such as `current.env` when explicit names are safer.

---

## Troubleshooting Workflow

### Problem 1 — Variable Is Missing

Check container configuration:

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  CONTAINER
```

Check inside:

```bash
docker exec CONTAINER printenv VARIABLE_NAME
```

Then verify:

- Exact spelling and capitalization
- Correct `-e` syntax
- Correct env-file path
- Correct `NAME=value` format
- Whether the container was recreated after a change

### Problem 2 — Variable Is Present but Application Ignores It

Check:

- Does the application support that variable?
- Is the expected name exact?
- Does it expect a file or command-line option instead?
- Is the value valid?
- Does the application read configuration only at startup?
- Is another application-level source overriding it?

### Problem 3 — Host Value Appears Instead of Container Value

Suspect host-shell expansion.

Risky:

```bash
docker exec CONTAINER sh -c "echo $NAME"
```

Safer for container-side expansion:

```bash
docker exec CONTAINER sh -c 'echo "$NAME"'
```

### Problem 4 — Env File Changed but Container Did Not

Expected behavior. Recreate the container.

```text
edit env file -> remove/replace old container -> create new container -> verify
```

### Problem 5 — Unexpected Image Default

Inspect the image:

```bash
docker image inspect \
  --format '{{json .Config.Env}}' \
  IMAGE
```

Then inspect the container to compare image defaults with runtime configuration.

### Problem 6 — Secret Appeared in Logs or History

Treat it as exposed:

1. Stop further disclosure.
2. Rotate or revoke the credential.
3. Remove it from unsafe configuration and history where appropriate.
4. Review logs and source-control history.
5. Move to an approved secret-management mechanism.

Deleting one visible copy does not make an exposed credential trustworthy again.

---

## Common Mistakes

### Mistake 1 — Rebuilding for Every Simple Setting

Changing a log level or service hostname normally should not require rebuilding application code into a new image.

### Mistake 2 — Expecting Docker to Understand the Variable

Docker transports the value. The application must use it.

### Mistake 3 — Using the Wrong Variable Name

Environment names are case-sensitive in Linux environments:

```text
LOG_LEVEL
log_level
```

These are different names.

### Mistake 4 — Expecting Restart to Load a Modified Env File

Restart preserves container configuration. Recreate to apply changes.

### Mistake 5 — Confusing `--env-file` with a Bind Mount

`--env-file` loads values. It does not mount the source file.

### Mistake 6 — Committing Real Secrets

Do not commit production passwords, keys, or tokens in `.env` files.

### Mistake 7 — Printing the Whole Environment During Debugging

The environment may contain sensitive values. Query only what you need and protect logs.

### Mistake 8 — Baking Environment-Specific Values into the Image

This reduces image portability and may require unnecessary rebuilds.

### Mistake 9 — Assuming Empty and Missing Are Identical

Applications may distinguish them. Validate both cases.

### Mistake 10 — Forgetting Quotes Around Values with Spaces

The host shell may split the value into several arguments.

---

## Best Practices

1. Build one immutable image and supply environment-specific configuration at runtime.
2. Use clear, uppercase variable names with underscores.
3. Define safe defaults only for non-sensitive settings.
4. Validate required configuration during application startup.
5. Fail with a clear message when essential configuration is missing or invalid.
6. Document every supported variable and its expected format.
7. Use `--env-file` to organize multiple non-sensitive runtime values.
8. Recreate containers after changing creation-time environment values.
9. Use read-only mounted files for complex file-based configuration.
10. Keep real secrets out of images, repositories, shell history, and ordinary logs.
11. Use a platform-appropriate secret-management system for credentials.
12. Inspect and verify the effective configuration after deployment.
13. Do not log complete environments in production.
14. Keep `.env.example` templates free of real sensitive values.

---

## Interview Questions

### 1. Why should configuration be separated from a Docker image?

So the same tested image can run in different environments without rebuilding it for every hostname, mode, or log level.

### 2. What does `docker run -e NAME=value IMAGE` do?

It creates the container with `NAME=value` in its configured process environment.

### 3. What is Dockerfile `ENV` used for?

It defines an environment value in the image configuration that containers inherit by default.

### 4. Can `docker run -e` override Dockerfile `ENV`?

Yes. A runtime value for the same name overrides the image default for that container.

### 5. What does `--env-file` do?

It reads environment entries from a host file when Docker creates the container.

### 6. Does Docker mount the file used by `--env-file`?

No. It reads the values; the file is not automatically mounted.

### 7. Why does changing an env file not update a running container?

The file was read at container creation time. The container stores its resulting environment configuration independently.

### 8. Will restarting the container apply changed environment values?

No. The container normally must be recreated with the new configuration.

### 9. How can you inspect a container's configured environment?

Use `docker inspect` and examine `.Config.Env`. Use care because output may contain sensitive values.

### 10. Why are environment variables not a complete secret solution?

Values may be visible through inspection, logs, shell history, deployment systems, or other privileged access paths.

### 11. When might a mounted configuration file be better?

When configuration is large, structured, file-oriented, or already managed in a required format.

### 12. What is the difference between Dockerfile `ARG` and `ENV`?

`ARG` supplies build-time input. `ENV` defines a value that persists in image configuration and is available to containers by default.

---

## Lesson Reflection

You should now be able to explain this model:

```text
Image
  application + runtime dependencies + safe defaults
                          |
                          v
Container creation
  runtime environment values override defaults
                          |
                          v
Application process
  reads and validates effective configuration
```

The key operational insight is:

```text
An image is reusable.
A container receives environment-specific configuration when it is created.
Changing that creation-time configuration normally means recreating the container.
```

---

## Summary

- Configuration controls how an application behaves in a particular environment.
- Environment variables are named text values available to processes.
- Use `docker run -e NAME=value` for individual runtime values.
- Use `docker run --env-file FILE` for multiple values.
- Use `docker exec CONTAINER printenv` to check values from inside.
- Use `docker inspect` to inspect the container's stored environment configuration.
- Dockerfile `ENV` supplies an image default.
- Runtime `-e` can override an image default for a new container.
- Editing an env file does not modify an existing container.
- Restarting is not recreating.
- Simple settings fit environment variables; complex structured configuration may fit a mounted file.
- Plain environment variables and env files are not complete secret-management solutions.
- One tested image can serve several environments through separate runtime configuration.

---

## Cheat Sheet

### Pass One Variable

```bash
docker run -e APP_ENV=testing IMAGE
```

### Pass Several Variables

```bash
docker run \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  IMAGE
```

### Pass a Host Variable

```bash
export APP_ENV=testing
docker run -e APP_ENV IMAGE
```

### Load an Environment File

```bash
docker run --env-file ./testing.env IMAGE
```

### Basic Env-File Format

```dotenv
APP_ENV=testing
LOG_LEVEL=debug
API_PORT=8080
```

### Read One Variable Inside a Container

```bash
docker exec CONTAINER printenv APP_ENV
```

### Read and Sort the Environment

```bash
docker exec CONTAINER sh -c 'printenv | sort'
```

### Expand a Variable Inside the Container

```bash
docker exec CONTAINER sh -c 'echo "$APP_ENV"'
```

### Inspect Stored Container Environment

```bash
docker inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  CONTAINER
```

### Inspect Image Defaults

```bash
docker image inspect \
  --format '{{json .Config.Env}}' \
  IMAGE
```

### Define an Image Default

```dockerfile
ENV APP_ENV=production
```

### Apply Changed Environment Configuration

```text
change source configuration
        -> remove/replace old container
        -> create new container
        -> verify effective configuration
```

### Protect a Local Sensitive File

```bash
chmod 600 production.env
```

This is a basic filesystem control, not a complete secret-management solution.

---

## Challenge — Deploy One Image Three Ways

### Scenario

TechCorp has one approved image:

```text
report-api:3.1
```

It must run with these non-sensitive settings:

| Environment | `APP_ENV` | `LOG_LEVEL` | `REPORT_FORMAT` |
|---|---|---|---|
| Development | `development` | `debug` | `text` |
| Testing | `testing` | `info` | `json` |
| Production | `production` | `warning` | `json` |

### Your Tasks

1. Create one env file for each environment.
2. Start three containers from the same image.
3. Give each container a unique name.
4. If the application publishes a port, give each container a unique host port.
5. Verify the three variables inside every container.
6. Prove all containers reference the same image ID.
7. Change testing `LOG_LEVEL` to `debug`.
8. Prove restart does not apply the changed file.
9. Recreate only the testing container.
10. Verify development and production remained unchanged.
11. Remove all challenge containers.

### Acceptance Criteria

- No image rebuild is used to change the environment.
- Each container has the correct configuration.
- The same image ID is used by all three containers.
- The testing change appears only after recreation.
- No real password, token, or private key is placed in the files.
- Verification commands and observed output are recorded.

### Optional Design Question

The application now needs a 70-line structured routing configuration. Decide whether to create 70 environment variables or mount a read-only configuration file. Explain your decision.

---

## Lab Assets

Suggested paths for screenshots and notes:

```text
assets/chapter-07/lesson-14/
├── 01-image-build.png
├── 02-development-environment.png
├── 03-production-environment.png
├── 04-same-image-id.png
├── 05-restart-old-value.png
├── 06-recreate-new-value.png
├── environment-decision-notes.md
└── lab-results.txt
```

Recommended evidence to capture:

- The image build result
- Development and production logs
- `docker inspect` output showing the same image ID
- Old value after restart
- New value after recreation
- Your completed configuration decision table

---

## Official References

- [Docker run: environment variables](https://docs.docker.com/engine/containers/run/#environment-variables)
- [Dockerfile `ENV` reference](https://docs.docker.com/reference/dockerfile/#env)
- [Docker CLI: `docker container run`](https://docs.docker.com/reference/cli/docker/container/run/)
- [Docker build variables](https://docs.docker.com/build/building/variables/)
- [Docker secrets](https://docs.docker.com/engine/swarm/secrets/)

---

## Next Lesson

**Lesson 15 — Secrets Fundamentals**

You can now separate ordinary runtime configuration from the image. Next, you will learn why passwords, tokens, and private keys need stronger handling than normal configuration values.

You will understand:

- Configuration versus secrets
- Why secrets must not be baked into images
- Risks of command-line arguments and ordinary environment variables
- Secret files
- Least-privilege access
- Credential rotation
- Docker Compose secrets introduction
- Production secret-manager introduction

---

## Completion Check

Before moving forward, explain without reading:

```text
What an environment variable is
Why configuration should stay outside the image
How -e differs from --env-file
How Dockerfile ENV differs from a runtime override
Why editing an env file does not update an existing container
Why restart does not apply new creation-time configuration
When to mount a configuration file
Why ordinary environment variables are not a secret manager
```

If you can explain these clearly and complete the lab, you are ready for Lesson 15.

# 🐳 Docker Complete Cheatsheet

> **Purpose:** Quick-reference for all Docker CLI commands, flags, and `grep` combos — built for interview prep and daily use.

---

## Table of Contents

1. [Global Flags (apply to any `docker` command)](#global-flags)
2. [Container Lifecycle](#container-lifecycle)
3. [Container Inspection & Monitoring](#container-inspection-monitoring)
4. [Container Interaction](#container-interaction)
5. [Image Commands](#image-commands)
6. [Image Build & Dockerfile](#image-build-dockerfile)
7. [Network Commands](#network-commands)
8. [Volume Commands](#volume-commands)
9. [Docker Compose](#docker-compose)
10. [Docker System & Cleanup](#docker-system-cleanup)
11. [Docker Registry & Login](#docker-registry-login)
12. [Docker Context & Config](#docker-context-config)
13. [Docker Swarm (Orchestration)](#docker-swarm-orchestration)
14. [Docker Service (Swarm Services)](#docker-service-swarm-services)
15. [Docker Stack (Swarm Stacks)](#docker-stack-swarm-stacks)
16. [Docker Secret & Config (Swarm)](#docker-secret-config-swarm)
17. [Docker Plugin](#docker-plugin)
18. [Docker Checkpoint (Experimental)](#docker-checkpoint-experimental)
19. [Docker Trust & Content Trust](#docker-trust-content-trust)
20. [Docker Manifest (Multi-arch)](#docker-manifest-multi-arch)
21. [Docker Buildx (Advanced Builds)](#docker-buildx-advanced-builds)
22. [Docker Scout (Security)](#docker-scout-security)
23. [Grep Combos with Docker](#grep-combos-with-docker)
24. [One-Liners & Power Tricks](#one-liners-power-tricks)
25. [Interview Quick-Fire Q&A](#interview-quick-fire-qa)

---

## Global Flags

These flags can be placed **before any subcommand** (e.g., `docker --debug ps`).

| Flag | Short | Description |
|------|-------|-------------|
| `--config` | | Location of client config files (default `~/.docker`) |
| `--context` | `-c` | Name of the context to use for connecting to the daemon |
| `--debug` | `-D` | Enable debug mode |
| `--host` | `-H` | Daemon socket to connect to (e.g., `tcp://0.0.0.0:2375`) |
| `--log-level` | `-l` | Set logging level: `debug`, `info`, `warn`, `error`, `fatal` |
| `--tls` | | Use TLS; implied by `--tlsverify` |
| `--tlscacert` | | Trust certs signed only by this CA (default `~/.docker/ca.pem`) |
| `--tlscert` | | Path to TLS certificate file |
| `--tlskey` | | Path to TLS key file |
| `--tlsverify` | | Use TLS and verify the remote |
| `--version` | `-v` | Print version information and quit |

```bash
# Examples
docker --debug ps                          # Run ps with debug logging
docker -H tcp://remote:2375 info           # Talk to a remote daemon
docker --context prod ps                   # Use a named context
```

---

## Container Lifecycle

### `docker run` — Create and start a container

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

| Flag | Short | Description |
|------|-------|-------------|
| `--name` | | Assign a name to the container |
| `--detach` | `-d` | Run in background and print container ID |
| `--interactive` | `-i` | Keep STDIN open |
| `--tty` | `-t` | Allocate a pseudo-TTY |
| `--rm` | | Automatically remove container on exit |
| `--publish` | `-p` | Publish port(s) `host:container` |
| `--publish-all` | `-P` | Publish all exposed ports to random host ports |
| `--env` | `-e` | Set environment variable `KEY=VALUE` |
| `--env-file` | | Read env vars from a file |
| `--volume` | `-v` | Bind mount a volume `host:container[:opts]` |
| `--mount` | | Attach a filesystem mount (more explicit than `-v`) |
| `--network` | | Connect to a network |
| `--hostname` | `-h` | Container hostname |
| `--workdir` | `-w` | Working directory inside the container |
| `--user` | `-u` | Username or UID (format: `name\|uid[:group\|gid]`) |
| `--restart` | | Restart policy: `no`, `always`, `unless-stopped`, `on-failure[:max]` |
| `--memory` | `-m` | Memory limit (e.g., `512m`, `1g`) |
| `--cpus` | | Number of CPUs (e.g., `1.5`) |
| `--cpu-shares` | `-c` | CPU shares (relative weight) |
| `--gpus` | | GPU devices to add (`all` or device IDs) |
| `--entrypoint` | | Override the default ENTRYPOINT |
| `--link` | | Add link to another container (**legacy**, use networks) |
| `--add-host` | | Add custom host-to-IP mapping (`host:ip`) |
| `--dns` | | Set custom DNS servers |
| `--log-driver` | | Logging driver (e.g., `json-file`, `syslog`, `none`) |
| `--log-opt` | | Log driver options |
| `--privileged` | | Give extended privileges to the container |
| `--cap-add` | | Add Linux capabilities |
| `--cap-drop` | | Drop Linux capabilities |
| `--security-opt` | | Security options (e.g., `no-new-privileges`) |
| `--read-only` | | Mount the root filesystem as read-only |
| `--tmpfs` | | Mount a tmpfs directory |
| `--pid` | | PID namespace to use |
| `--ipc` | | IPC namespace to use |
| `--init` | | Run an init inside the container (tini) |
| `--label` | `-l` | Set metadata label on the container |
| `--platform` | | Set platform (`linux/amd64`, `linux/arm64`) |
| `--pull` | | Pull image before running: `always`, `missing`, `never` |
| `--sig-proxy` | | Proxy received signals to the process (default `true`) |
| `--stop-signal` | | Signal to stop a container (default `SIGTERM`) |
| `--stop-timeout` | | Timeout (seconds) to stop a container |
| `--storage-opt` | | Storage driver options |
| `--ulimit` | | Ulimit options (e.g., `nofile=1024:2048`) |
| `--device` | | Add a host device to the container |
| `--cgroup-parent` | | Parent cgroup for the container |
| `--cidfile` | | Write the container ID to a file |
| `--health-cmd` | | Command to run for health checks |
| `--health-interval` | | Time between health checks (default `30s`) |
| `--health-retries` | | Consecutive failures needed to report unhealthy |
| `--health-timeout` | | Maximum time for a health check |
| `--no-healthcheck` | | Disable any health check |
| `--shm-size` | | Size of `/dev/shm` (default `64m`) |
| `--isolation` | | Container isolation technology |

```bash
# Common patterns
docker run -d --name web -p 8080:80 nginx                     # Detached with port
docker run -it --rm ubuntu bash                                # Interactive, auto-remove
docker run -d -e DB_HOST=db --network mynet app:latest         # With env & network
docker run -d -v /data:/app/data --restart unless-stopped app  # Volume + restart
docker run -d --memory 512m --cpus 1.5 --name limited app     # Resource limits
docker run -d --gpus all tensorflow/tensorflow:latest-gpu      # GPU access
docker run --read-only --tmpfs /tmp --cap-drop ALL app         # Hardened container
```

### `docker create` — Create a container without starting it

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
# Same flags as `docker run`
```

### `docker start` / `stop` / `restart` / `kill`

| Command | Key Flags | Description |
|---------|-----------|-------------|
| `docker start` | `-i` (interactive), `-a` (attach) | Start stopped container(s) |
| `docker stop` | `-t` (timeout, default 10s) | Graceful stop (SIGTERM → SIGKILL) |
| `docker restart` | `-t` (timeout) | Stop then start |
| `docker kill` | `-s` (signal, default SIGKILL) | Send signal to running container |

```bash
docker stop -t 30 mycontainer     # 30s grace period
docker kill -s SIGUSR1 mycontainer
docker restart mycontainer
```

### `docker pause` / `unpause`

```bash
docker pause CONTAINER      # Freeze all processes (SIGSTOP via cgroups)
docker unpause CONTAINER    # Resume
```

### `docker rm` — Remove containers

| Flag | Description |
|------|-------------|
| `-f`, `--force` | Force remove a running container (uses SIGKILL) |
| `-v`, `--volumes` | Remove anonymous volumes associated with container |
| `-l`, `--link` | Remove the specified link |

```bash
docker rm mycontainer
docker rm -f $(docker ps -aq)      # Force remove ALL containers
docker rm -v mycontainer           # Also clean up anonymous volumes
```

### `docker rename`

```bash
docker rename OLD_NAME NEW_NAME
```

### `docker update` — Update container configuration

| Flag | Description |
|------|-------------|
| `--cpus` | Number of CPUs |
| `--memory` / `-m` | Memory limit |
| `--memory-swap` | Swap limit |
| `--restart` | Restart policy |
| `--cpu-shares` | CPU shares |
| `--blkio-weight` | Block IO weight |
| `--pids-limit` | Limit number of PIDs |

```bash
docker update --memory 1g --restart always mycontainer
```

### `docker wait`

```bash
docker wait CONTAINER     # Block until container stops, then print exit code
```

---

## Container Inspection & Monitoring

### `docker ps` — List containers

| Flag | Short | Description |
|------|-------|-------------|
| `--all` | `-a` | Show all containers (default shows only running) |
| `--filter` | `-f` | Filter output (e.g., `status=running`, `name=web`, `label=env=prod`) |
| `--format` | | Pretty-print using Go template |
| `--last` | `-n` | Show N last created containers |
| `--latest` | `-l` | Show the latest created container |
| `--no-trunc` | | Don't truncate output |
| `--quiet` | `-q` | Only display container IDs |
| `--size` | `-s` | Display total file sizes |

```bash
docker ps                                      # Running containers
docker ps -a                                   # All containers
docker ps -aq                                  # All container IDs only
docker ps --filter "status=exited"             # Exited containers
docker ps --filter "name=web"                  # Filter by name
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
docker ps -s                                   # Show disk usage
```

> [!TIP]
> **Common filters for `docker ps -f`:**
> `status` (created, restarting, running, paused, exited, dead), `name`, `id`, `label`, `ancestor` (image), `network`, `publish`/`expose` (port), `health`, `volume`, `before`/`since` (container).

### `docker inspect` — Low-level info (JSON)

| Flag | Description |
|------|-------------|
| `--format` / `-f` | Format output using Go template |
| `--type` | Return JSON for `container`, `image`, `network`, `volume`, etc. |
| `--size` / `-s` | Display total file sizes (containers) |

```bash
docker inspect mycontainer
docker inspect -f '{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect -f '{{.State.Status}}' mycontainer
docker inspect -f '{{json .Config.Env}}' mycontainer | jq
docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' mycontainer
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mycontainer
```

### `docker logs` — Fetch container logs

| Flag | Short | Description |
|------|-------|-------------|
| `--follow` | `-f` | Follow log output (like `tail -f`) |
| `--tail` | `-n` | Number of lines from the end |
| `--since` | | Show logs since timestamp (`2024-01-01T00:00:00`) or relative (`10m`) |
| `--until` | | Show logs before timestamp |
| `--timestamps` | `-t` | Show timestamps |
| `--details` | | Show extra details from log driver |

```bash
docker logs mycontainer
docker logs -f mycontainer                 # Stream logs
docker logs --tail 100 mycontainer         # Last 100 lines
docker logs --since 30m mycontainer        # Last 30 minutes
docker logs -t mycontainer                 # With timestamps
docker logs --since 2024-01-01 --until 2024-01-02 mycontainer
```

### `docker top` — Running processes

```bash
docker top mycontainer
docker top mycontainer -aux       # Pass ps flags
```

### `docker stats` — Live resource usage

| Flag | Description |
|------|-------------|
| `--all` / `-a` | Show all containers (not just running) |
| `--no-stream` | Disable streaming (one snapshot) |
| `--no-trunc` | Don't truncate output |
| `--format` | Go template format |

```bash
docker stats                           # All running containers (live)
docker stats --no-stream               # Single snapshot
docker stats mycontainer               # Specific container
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### `docker port`

```bash
docker port mycontainer           # Show all port mappings
docker port mycontainer 80/tcp    # Specific port
```

### `docker diff`

```bash
docker diff mycontainer    # Show filesystem changes (A=added, C=changed, D=deleted)
```

### `docker events` — Real-time events from the daemon

| Flag | Description |
|------|-------------|
| `--filter` / `-f` | Filter by event type, container, image, etc. |
| `--since` | Show events since timestamp |
| `--until` | Show events until timestamp |
| `--format` | Format output |

```bash
docker events
docker events --filter 'type=container'
docker events --filter 'event=stop'
docker events --since '2024-01-01'
```

---

## Container Interaction

### `docker exec` — Run command in running container

| Flag | Short | Description |
|------|-------|-------------|
| `--interactive` | `-i` | Keep STDIN open |
| `--tty` | `-t` | Allocate a pseudo-TTY |
| `--detach` | `-d` | Run in background |
| `--env` | `-e` | Set environment variable |
| `--env-file` | | Read env vars from file |
| `--workdir` | `-w` | Working directory inside the container |
| `--user` | `-u` | Username or UID |
| `--privileged` | | Give extended privileges |

```bash
docker exec -it mycontainer bash                # Interactive shell
docker exec -it mycontainer sh                  # For Alpine/minimal images
docker exec mycontainer cat /etc/os-release     # Run single command
docker exec -u root mycontainer apt update      # Run as root
docker exec -e MY_VAR=hello mycontainer env     # With env var
docker exec -w /app mycontainer ls              # Set working dir
```

### `docker attach` — Attach to running container's STDIO

| Flag | Description |
|------|-------------|
| `--detach-keys` | Override detach key sequence (default `ctrl-p,ctrl-q`) |
| `--no-stdin` | Do not attach STDIN |
| `--sig-proxy` | Proxy all received signals (default `true`) |

```bash
docker attach mycontainer
# Detach without stopping: Ctrl+P, Ctrl+Q
```

> [!WARNING]
> `docker attach` connects to PID 1. If you type `exit`, the container **stops**. Use `exec` for a separate shell.

### `docker cp` — Copy files between container and host

```bash
docker cp mycontainer:/app/log.txt ./log.txt      # Container → Host
docker cp ./config.yml mycontainer:/app/config.yml # Host → Container
docker cp mycontainer:/app/data/ ./backup/         # Copy directory
```

| Flag | Description |
|------|-------------|
| `-a`, `--archive` | Archive mode (copy all uid/gid information) |
| `-L`, `--follow-link` | Follow symlinks in source |

### `docker export` / `docker import`

```bash
docker export mycontainer > container.tar       # Export filesystem as tar
docker export -o container.tar mycontainer      # Same, with -o flag

cat container.tar | docker import - myimage:v1  # Import as image
docker import container.tar myimage:v1          # Same
docker import --change "ENV DEBUG=true" container.tar myimage:v1
```

> [!NOTE]
> `export`/`import` works on **containers** (flattened filesystem). For **images** with layers, use `save`/`load`.

---

## Image Commands

### `docker images` / `docker image ls`

| Flag | Short | Description |
|------|-------|-------------|
| `--all` | `-a` | Show all images (including intermediates) |
| `--digests` | | Show digests |
| `--filter` | `-f` | Filter (`dangling=true`, `reference=nginx`, `before=image`, `since=image`, `label=key=val`) |
| `--format` | | Go template |
| `--no-trunc` | | Don't truncate output |
| `--quiet` | `-q` | Only show image IDs |

```bash
docker images                              # List images
docker images -a                           # Include intermediate layers
docker images -q                           # IDs only
docker images --filter "dangling=true"     # Untagged images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
docker images nginx                        # Filter by repository name
```

### `docker pull`

| Flag | Description |
|------|-------------|
| `-a`, `--all-tags` | Pull all tagged images |
| `--platform` | Set platform (e.g., `linux/arm64`) |
| `-q`, `--quiet` | Suppress verbose output |
| `--disable-content-trust` | Skip image verification |

```bash
docker pull nginx
docker pull nginx:1.25-alpine
docker pull --platform linux/arm64 nginx
docker pull myregistry.io/app:v2.0
```

### `docker push`

| Flag | Description |
|------|-------------|
| `-a`, `--all-tags` | Push all tags of an image |
| `-q`, `--quiet` | Suppress verbose output |
| `--disable-content-trust` | Skip image signing |

```bash
docker push myrepo/myimage:latest
docker push --all-tags myrepo/myimage
```

### `docker tag`

```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
docker tag myapp:latest myregistry.io/myapp:v1.0
docker tag abc123 myapp:production        # Tag by image ID
```

### `docker rmi` — Remove images

| Flag | Description |
|------|-------------|
| `-f`, `--force` | Force removal |
| `--no-prune` | Do not delete untagged parents |

```bash
docker rmi nginx:latest
docker rmi -f $(docker images -q)          # Remove ALL images
docker rmi $(docker images -f "dangling=true" -q)   # Remove dangling
```

### `docker history` — Image layer history

| Flag | Description |
|------|-------------|
| `--no-trunc` | Don't truncate output |
| `--format` | Go template |
| `-q`, `--quiet` | Only show image IDs |
| `-H`, `--human` | Human-readable sizes (default `true`) |

```bash
docker history nginx
docker history --no-trunc nginx
docker history --format "table {{.CreatedBy}}\t{{.Size}}" nginx
```

### `docker save` / `docker load`

```bash
docker save -o images.tar nginx:latest redis:latest   # Save image(s) to tar
docker save nginx:latest | gzip > nginx.tar.gz        # Compressed

docker load -i images.tar                              # Load from tar
docker load < nginx.tar.gz                             # From stdin
```

> [!IMPORTANT]
> `save`/`load` preserves **layers and metadata**. Use for offline image transfer.  
> `export`/`import` flattens to a single layer. Use for container filesystem snapshots.

---

## Image Build & Dockerfile

### `docker build`

| Flag | Short | Description |
|------|-------|-------------|
| `--tag` | `-t` | Name and optionally tag (`name:tag`) |
| `--file` | `-f` | Path to Dockerfile (default `./Dockerfile`) |
| `--build-arg` | | Set build-time variable |
| `--no-cache` | | Do not use cache when building |
| `--pull` | | Always pull a newer version of the base image |
| `--target` | | Set the target build stage (multi-stage) |
| `--platform` | | Set platform (`linux/amd64,linux/arm64`) |
| `--progress` | | Set type of progress output (`auto`, `plain`, `tty`) |
| `--secret` | | Expose a secret to the build |
| `--ssh` | | SSH agent socket or keys |
| `--cache-from` | | External cache source |
| `--cache-to` | | Export build cache |
| `--output` / `-o` | | Output destination (`type=local,dest=./out`) |
| `--network` | | Network mode for RUN instructions |
| `--label` | | Set metadata label |
| `--squash` | | Squash newly built layers into a single layer (**experimental**) |
| `--compress` | | Compress build context using gzip |
| `--rm` | | Remove intermediate containers (default `true`) |
| `--force-rm` | | Always remove intermediate containers |
| `--memory` / `-m` | | Memory limit for build |
| `--shm-size` | | Size of `/dev/shm` |
| `--ulimit` | | Ulimit options |
| `--add-host` | | Add custom host-to-IP mapping |
| `--iidfile` | | Write image ID to a file |

```bash
docker build -t myapp:v1 .
docker build -t myapp:v1 -f Dockerfile.prod .
docker build --build-arg VERSION=1.2 -t myapp .
docker build --no-cache -t myapp .
docker build --target builder -t myapp:builder .           # Multi-stage
docker build --platform linux/amd64,linux/arm64 -t myapp .
docker build --secret id=mysecret,src=secret.txt -t myapp .
```

### Dockerfile Instructions Reference

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:20-alpine AS builder` |
| `RUN` | Execute command during build | `RUN apt-get update && apt-get install -y curl` |
| `CMD` | Default command (overridable) | `CMD ["node", "server.js"]` |
| `ENTRYPOINT` | Main executable (not easily overridden) | `ENTRYPOINT ["python", "app.py"]` |
| `COPY` | Copy files from build context | `COPY --chown=node:node . /app` |
| `ADD` | Copy + auto-extract tar + URL support | `ADD app.tar.gz /app` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `ENV` | Set environment variable | `ENV NODE_ENV=production` |
| `ARG` | Build-time variable | `ARG VERSION=latest` |
| `EXPOSE` | Document listening port | `EXPOSE 3000` |
| `VOLUME` | Create mount point | `VOLUME ["/data"]` |
| `USER` | Set user for subsequent instructions | `USER node` |
| `LABEL` | Add metadata | `LABEL version="1.0" maintainer="me"` |
| `HEALTHCHECK` | Container health check | `HEALTHCHECK CMD curl -f http://localhost/ \|\| exit 1` |
| `SHELL` | Override default shell | `SHELL ["/bin/bash", "-c"]` |
| `STOPSIGNAL` | Signal to stop the container | `STOPSIGNAL SIGQUIT` |
| `ONBUILD` | Trigger on downstream build | `ONBUILD COPY . /app` |

> [!TIP]
> **Multi-stage build pattern:**
> ```dockerfile
> FROM node:20 AS builder
> WORKDIR /app
> COPY package*.json ./
> RUN npm ci
> COPY . .
> RUN npm run build
>
> FROM node:20-alpine
> COPY --from=builder /app/dist ./dist
> CMD ["node", "dist/server.js"]
> ```

---

## Network Commands

### `docker network` subcommands

| Command | Description |
|---------|-------------|
| `docker network create` | Create a network |
| `docker network ls` | List networks |
| `docker network inspect` | Detailed network info |
| `docker network connect` | Connect a container to a network |
| `docker network disconnect` | Disconnect a container |
| `docker network rm` | Remove a network |
| `docker network prune` | Remove all unused networks |

### `docker network create` flags

| Flag | Short | Description |
|------|-------|-------------|
| `--driver` | `-d` | Driver: `bridge` (default), `overlay`, `host`, `none`, `macvlan` |
| `--subnet` | | Subnet in CIDR format (e.g., `172.20.0.0/16`) |
| `--gateway` | | Gateway for the subnet |
| `--ip-range` | | Allocate container IP from a sub-range |
| `--internal` | | Restrict external access |
| `--attachable` | | Allow manual container attachment (overlay) |
| `--label` | | Set metadata |
| `--ipv6` | | Enable IPv6 |
| `--opt` / `-o` | | Driver-specific options |
| `--scope` | | Network scope |

```bash
docker network create mynet
docker network create --driver overlay --attachable myoverlay
docker network create --subnet 10.5.0.0/16 --gateway 10.5.0.1 custom_net
docker network ls
docker network ls --filter driver=bridge
docker network inspect mynet
docker network connect mynet mycontainer
docker network disconnect mynet mycontainer
docker network rm mynet
docker network prune                     # Remove ALL unused networks
docker network prune --filter "until=24h"
```

### Network Drivers Summary

| Driver | Scope | Use Case |
|--------|-------|----------|
| `bridge` | Single host | Default; isolated container-to-container communication |
| `host` | Single host | Container shares host's network stack (no isolation) |
| `none` | Single host | No networking |
| `overlay` | Multi-host (Swarm) | Cross-host communication in Swarm |
| `macvlan` | Single host | Assign MAC address; appear as physical device on network |
| `ipvlan` | Single host | Similar to macvlan without unique MAC |

---

## Volume Commands

### `docker volume` subcommands

| Command | Description |
|---------|-------------|
| `docker volume create` | Create a volume |
| `docker volume ls` | List volumes |
| `docker volume inspect` | Detailed volume info |
| `docker volume rm` | Remove volume(s) |
| `docker volume prune` | Remove all unused volumes |

### `docker volume create` flags

| Flag | Description |
|------|-------------|
| `--driver` / `-d` | Volume driver (default `local`) |
| `--label` | Set metadata |
| `--opt` / `-o` | Driver-specific options |
| `--name` | Volume name |

```bash
docker volume create mydata
docker volume create --driver local --opt type=nfs --opt o=addr=10.0.0.1,rw --opt device=:/export/data nfs_vol
docker volume ls
docker volume ls --filter "dangling=true"
docker volume inspect mydata
docker volume rm mydata
docker volume prune                       # Remove ALL unused volumes
docker volume prune --filter "label!=keep"
```

### Mount Types Comparison

| Type | Syntax | Use Case |
|------|--------|----------|
| **Volume** | `-v mydata:/app/data` or `--mount type=volume,src=mydata,dst=/app/data` | Persistent data managed by Docker |
| **Bind** | `-v /host/path:/container/path` or `--mount type=bind,src=/host/path,dst=/container/path` | Share host files with container |
| **tmpfs** | `--tmpfs /app/tmp` or `--mount type=tmpfs,dst=/app/tmp` | In-memory temporary storage |

---

## Docker Compose

### Core Commands

| Command | Description |
|---------|-------------|
| `docker compose up` | Create and start containers |
| `docker compose down` | Stop and remove containers, networks |
| `docker compose build` | Build or rebuild services |
| `docker compose start` | Start existing containers |
| `docker compose stop` | Stop running containers |
| `docker compose restart` | Restart containers |
| `docker compose ps` | List containers |
| `docker compose logs` | View output from containers |
| `docker compose exec` | Execute a command in a running container |
| `docker compose run` | Run a one-off command |
| `docker compose pull` | Pull service images |
| `docker compose push` | Push service images |
| `docker compose config` | Validate and view the Compose file |
| `docker compose create` | Create containers without starting |
| `docker compose kill` | Force stop containers |
| `docker compose rm` | Remove stopped containers |
| `docker compose top` | Display running processes |
| `docker compose images` | List images used by services |
| `docker compose port` | Print public port for a port binding |
| `docker compose pause` / `unpause` | Pause/Unpause services |
| `docker compose events` | Receive real-time events |
| `docker compose cp` | Copy files between service containers and host |
| `docker compose ls` | List running compose projects |
| `docker compose watch` | Watch build context and rebuild/sync on changes |
| `docker compose alpha` | Experimental commands |

### `docker compose up` flags

| Flag | Short | Description |
|------|-------|-------------|
| `--detach` | `-d` | Run in background |
| `--build` | | Build images before starting |
| `--force-recreate` | | Recreate containers even if unchanged |
| `--no-recreate` | | Don't recreate if already exists |
| `--no-build` | | Don't build images |
| `--no-start` | | Create containers but don't start |
| `--remove-orphans` | | Remove containers for undefined services |
| `--scale` | | Scale SERVICE to NUM instances |
| `--timeout` | `-t` | Shutdown timeout |
| `--wait` | | Wait for services to be healthy |
| `--pull` | | Pull image before running: `always`, `missing`, `never` |
| `--abort-on-container-exit` | | Stop all containers if any container stops |
| `--no-deps` | | Don't start linked services |
| `--no-log-prefix` | | Don't prefix log output with service name |
| `--quiet-pull` | | Pull without printing progress |
| `--timestamps` | | Show timestamps |

### `docker compose down` flags

| Flag | Description |
|------|-------------|
| `-v`, `--volumes` | Remove named volumes declared in `volumes` section |
| `--rmi` | Remove images: `all` or `local` |
| `--remove-orphans` | Remove orphaned containers |
| `-t`, `--timeout` | Shutdown timeout |

### Compose Global Flags (before subcommand)

| Flag | Short | Description |
|------|-------|-------------|
| `--file` | `-f` | Compose file path (repeatable) |
| `--project-name` | `-p` | Project name |
| `--project-directory` | | Alternate working directory |
| `--profile` | | Activate a profile |
| `--env-file` | | Env file path |
| `--progress` | | Progress output type |
| `--ansi` | | Control ANSI output |
| `--no-ansi` | | Do not print ANSI control characters |
| `--verbose` | | Show more output |
| `--parallel` | | Max parallelism for engine calls |

```bash
docker compose up -d                                  # Start in background
docker compose up -d --build                          # Rebuild & start
docker compose up -d --scale web=3                    # Scale web to 3 replicas
docker compose down -v                                # Stop + remove volumes
docker compose down --rmi all                         # Stop + remove images
docker compose logs -f web                            # Follow web service logs
docker compose exec web bash                          # Shell into web service
docker compose run --rm web npm test                  # One-off command
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d  # Override files
docker compose config                                 # Validate compose file
docker compose ps --format json                       # JSON output
docker compose watch                                  # Dev mode with hot-reload
```

---

## Docker System & Cleanup

### `docker system` subcommands

| Command | Description |
|---------|-------------|
| `docker system df` | Show Docker disk usage |
| `docker system prune` | Remove unused data |
| `docker system info` | Display system-wide information |
| `docker system events` | Real-time events (alias for `docker events`) |

### `docker system prune` flags

| Flag | Description |
|------|-------------|
| `-a`, `--all` | Remove all unused images, not just dangling |
| `-f`, `--force` | Do not prompt for confirmation |
| `--volumes` | Also prune volumes |
| `--filter` | Filter (e.g., `until=24h`, `label=temp`) |

```bash
docker system df                         # Disk usage summary
docker system df -v                      # Verbose (per-image/container/volume)
docker system prune                      # Remove dangling images, stopped containers, unused networks
docker system prune -a --volumes -f      # Nuclear clean (everything unused, no prompt)
docker system prune --filter "until=168h"  # Older than 1 week
docker system info                       # Full daemon info
```

### Individual Prune Commands

```bash
docker container prune [-f] [--filter]   # Remove stopped containers
docker image prune [-a] [-f] [--filter]  # Remove unused images
docker network prune [-f] [--filter]     # Remove unused networks
docker volume prune [-f] [--filter]      # Remove unused volumes
docker builder prune [-a] [-f]           # Remove build cache
```

---

## Docker Registry & Login

### `docker login` / `docker logout`

| Flag | Description |
|------|-------------|
| `-u`, `--username` | Username |
| `-p`, `--password` | Password (insecure, prefer `--password-stdin`) |
| `--password-stdin` | Read password from stdin |

```bash
docker login                                              # Docker Hub (interactive)
docker login -u myuser --password-stdin < password.txt    # Secure login
docker login myregistry.io                                # Custom registry
docker logout
docker logout myregistry.io
```

### `docker search` — Search Docker Hub

| Flag | Description |
|------|-------------|
| `--filter` / `-f` | Filter (`stars=50`, `is-official=true`, `is-automated=true`) |
| `--format` | Go template |
| `--limit` | Max number of results (default 25) |
| `--no-trunc` | Don't truncate output |

```bash
docker search nginx
docker search --filter "is-official=true" --filter "stars=100" nginx
docker search --limit 5 --format "table {{.Name}}\t{{.StarCount}}\t{{.IsOfficial}}" redis
```

---

## Docker Context & Config

### `docker context` subcommands

| Command | Description |
|---------|-------------|
| `docker context create` | Create a new context |
| `docker context ls` | List contexts |
| `docker context inspect` | Inspect a context |
| `docker context use` | Switch to a context |
| `docker context rm` | Remove a context |
| `docker context update` | Update a context |
| `docker context export` / `import` | Export/import contexts |

```bash
docker context create remote --docker "host=ssh://user@remote-host"
docker context create ecs myecs --from-env          # AWS ECS context
docker context ls
docker context use remote
docker context inspect remote
docker context rm remote
```

---

## Docker Swarm (Orchestration)

### `docker swarm` subcommands

| Command | Key Flags | Description |
|---------|-----------|-------------|
| `docker swarm init` | `--advertise-addr`, `--listen-addr`, `--force-new-cluster`, `--availability` | Initialize a Swarm |
| `docker swarm join` | `--token` | Join as worker or manager |
| `docker swarm leave` | `-f` (force) | Leave the Swarm |
| `docker swarm update` | `--cert-expiry`, `--task-history-limit`, `--autolock` | Update Swarm settings |
| `docker swarm join-token` | `-q` (quiet), `--rotate` | Manage join tokens |
| `docker swarm ca` | `--rotate`, `--cert-expiry` | Manage root CA |
| `docker swarm unlock` | | Unlock a locked manager |
| `docker swarm unlock-key` | `-q`, `--rotate` | Manage unlock key |

```bash
docker swarm init --advertise-addr 192.168.1.10
docker swarm join --token SWMTKN-xxx 192.168.1.10:2377
docker swarm join-token worker            # Print worker join command
docker swarm join-token manager           # Print manager join command
docker swarm join-token --rotate worker   # Rotate token
docker swarm leave -f                     # Force leave (for managers)
```

### `docker node` subcommands

| Command | Description |
|---------|-------------|
| `docker node ls` | List nodes in the Swarm |
| `docker node inspect` | Inspect a node |
| `docker node update` | Update node (role, availability, labels) |
| `docker node promote` / `demote` | Change role |
| `docker node rm` | Remove node from Swarm |
| `docker node ps` | List tasks running on a node |

```bash
docker node ls
docker node inspect self --pretty
docker node update --availability drain worker1   # Drain node for maintenance
docker node update --label-add zone=us-east worker1
docker node promote worker1                       # Promote to manager
docker node demote manager2                       # Demote to worker
docker node rm worker1                            # Remove node
docker node ps                                    # Tasks on current node
```

---

## Docker Service (Swarm Services)

### `docker service` subcommands

| Command | Description |
|---------|-------------|
| `docker service create` | Create a new service |
| `docker service ls` | List services |
| `docker service inspect` | Inspect a service |
| `docker service ps` | List tasks of a service |
| `docker service logs` | Fetch service logs |
| `docker service update` | Update a service |
| `docker service scale` | Scale one or more services |
| `docker service rm` | Remove service(s) |
| `docker service rollback` | Rollback to previous spec |

### `docker service create` key flags

| Flag | Description |
|------|-------------|
| `--name` | Service name |
| `--replicas` | Number of tasks/replicas |
| `--mode` | `replicated` (default) or `global` |
| `--publish` / `-p` | Publish port |
| `--network` | Attach to network |
| `--env` / `-e` | Set environment variable |
| `--mount` | Attach mount |
| `--secret` | Attach a secret |
| `--config` | Attach a config |
| `--constraint` | Placement constraint |
| `--placement-pref` | Placement preference |
| `--limit-cpu` / `--limit-memory` | Resource limits |
| `--reserve-cpu` / `--reserve-memory` | Resource reservations |
| `--update-parallelism` | Max simultaneous updates |
| `--update-delay` | Delay between updates |
| `--update-failure-action` | Action on update failure: `pause`, `continue`, `rollback` |
| `--rollback-parallelism` | Max simultaneous rollbacks |
| `--restart-condition` | `none`, `on-failure`, `any` |
| `--restart-max-attempts` | Max restart attempts |
| `--health-cmd` | Health check command |
| `--with-registry-auth` | Send registry auth to Swarm agents |
| `--endpoint-mode` | `vip` (default) or `dnsrr` |

```bash
docker service create --name web --replicas 3 -p 80:80 nginx
docker service ls
docker service ps web
docker service logs -f web
docker service scale web=5
docker service update --image nginx:1.25 web
docker service update --env-add NEW_VAR=val --env-rm OLD_VAR web
docker service rollback web
docker service rm web
```

---

## Docker Stack (Swarm Stacks)

| Command | Description |
|---------|-------------|
| `docker stack deploy` | Deploy or update a stack |
| `docker stack ls` | List stacks |
| `docker stack ps` | List tasks in a stack |
| `docker stack services` | List services in a stack |
| `docker stack rm` | Remove a stack |

```bash
docker stack deploy -c docker-compose.yml mystack
docker stack ls
docker stack services mystack
docker stack ps mystack
docker stack rm mystack
```

---

## Docker Secret & Config (Swarm)

### Secrets

| Command | Description |
|---------|-------------|
| `docker secret create` | Create from file or STDIN |
| `docker secret ls` | List secrets |
| `docker secret inspect` | Inspect (metadata only, not the value) |
| `docker secret rm` | Remove a secret |

```bash
echo "s3cr3t" | docker secret create db_password -
docker secret create tls_cert ./cert.pem
docker secret ls
docker secret inspect db_password
docker secret rm db_password

# Use in service:
docker service create --name web --secret db_password nginx
# Secret available at /run/secrets/db_password inside the container
```

### Configs

| Command | Description |
|---------|-------------|
| `docker config create` | Create a config |
| `docker config ls` | List configs |
| `docker config inspect` | Inspect (shows content!) |
| `docker config rm` | Remove a config |

```bash
docker config create nginx_conf ./nginx.conf
docker config ls
docker config inspect nginx_conf
docker service create --name web --config source=nginx_conf,target=/etc/nginx/nginx.conf nginx
```

---

## Docker Plugin

| Command | Description |
|---------|-------------|
| `docker plugin install` | Install a plugin |
| `docker plugin ls` | List plugins |
| `docker plugin inspect` | Inspect a plugin |
| `docker plugin enable` / `disable` | Enable/disable a plugin |
| `docker plugin rm` | Remove a plugin |
| `docker plugin create` | Create a plugin from a rootfs |
| `docker plugin push` | Push plugin to registry |
| `docker plugin set` | Change plugin settings |
| `docker plugin upgrade` | Upgrade a plugin |

```bash
docker plugin install vieux/sshfs
docker plugin ls
docker plugin disable vieux/sshfs
docker plugin rm vieux/sshfs
```

---

## Docker Checkpoint (Experimental)

| Command | Description |
|---------|-------------|
| `docker checkpoint create` | Create a checkpoint |
| `docker checkpoint ls` | List checkpoints |
| `docker checkpoint rm` | Delete a checkpoint |

```bash
docker checkpoint create mycontainer mycp1
docker start --checkpoint mycp1 mycontainer   # Restore from checkpoint
```

> [!NOTE]
> Requires `--experimental` daemon flag and CRIU installed.

---

## Docker Trust & Content Trust

### `docker trust` subcommands

| Command | Description |
|---------|-------------|
| `docker trust inspect` | Inspect signature data |
| `docker trust sign` | Sign an image |
| `docker trust revoke` | Revoke trust for an image |
| `docker trust key generate` | Generate a signing key |
| `docker trust key load` | Load a private key |
| `docker trust signer add` | Add a signer |
| `docker trust signer remove` | Remove a signer |

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

docker trust inspect --pretty nginx:latest
docker trust sign myrepo/myimage:v1
docker trust revoke myrepo/myimage:v1
docker trust key generate mykey
```

---

## Docker Manifest (Multi-arch)

| Command | Description |
|---------|-------------|
| `docker manifest create` | Create a manifest list |
| `docker manifest inspect` | Inspect a manifest |
| `docker manifest push` | Push a manifest list |
| `docker manifest annotate` | Add platform info to a manifest entry |
| `docker manifest rm` | Remove a local manifest list |

```bash
docker manifest create myrepo/app:latest \
  myrepo/app:amd64 \
  myrepo/app:arm64

docker manifest annotate myrepo/app:latest myrepo/app:arm64 \
  --os linux --arch arm64

docker manifest push myrepo/app:latest
docker manifest inspect myrepo/app:latest
```

---

## Docker Buildx (Advanced Builds)

| Command | Description |
|---------|-------------|
| `docker buildx create` | Create a new builder instance |
| `docker buildx use` | Switch the current builder |
| `docker buildx ls` | List builder instances |
| `docker buildx inspect` | Inspect a builder |
| `docker buildx build` | Build with BuildKit |
| `docker buildx bake` | Build from HCL/JSON/Compose file |
| `docker buildx rm` | Remove a builder |
| `docker buildx stop` | Stop a builder |
| `docker buildx prune` | Remove build cache |
| `docker buildx imagetools` | Inspect/create images in registry |
| `docker buildx du` | Disk usage |

### `docker buildx build` additional flags

| Flag | Description |
|------|-------------|
| `--builder` | Override current builder |
| `--load` | Shorthand for `--output=type=docker` (load into local daemon) |
| `--push` | Shorthand for `--output=type=registry` |
| `--platform` | Target platform(s) for multi-arch |
| `--provenance` | Attach provenance attestation |
| `--sbom` | Attach SBOM attestation |
| `--attest` | Attestation parameters |

```bash
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap

# Multi-platform build & push
docker buildx build --platform linux/amd64,linux/arm64 -t myrepo/app:v1 --push .

# Build and load into local daemon
docker buildx build --load -t myapp:test .

# Build with provenance
docker buildx build --provenance=true --sbom=true -t myapp:v1 --push .

docker buildx ls
docker buildx rm mybuilder
docker buildx prune -a -f
```

---

## Docker Scout (Security)

| Command | Description |
|---------|-------------|
| `docker scout quickview` | Quick overview of image vulnerabilities |
| `docker scout cves` | Display CVEs identified in an image |
| `docker scout recommendations` | Display recommendations |
| `docker scout compare` | Compare two images |
| `docker scout sbom` | Generate SBOM for an image |
| `docker scout attestation` | Manage attestations |
| `docker scout enroll` | Enroll an organization |
| `docker scout repo` | Manage repos |
| `docker scout environment` | Manage environments |
| `docker scout watch` | Watch repos for changes |
| `docker scout cache` | Manage local cache |

```bash
docker scout quickview nginx:latest
docker scout cves nginx:latest
docker scout cves --only-severity critical,high nginx:latest
docker scout recommendations nginx:latest
docker scout compare --to nginx:1.24 nginx:1.25
docker scout sbom nginx:latest
```

---

## Grep Combos with Docker

### Container Grep Patterns

```bash
# Filter running containers by name pattern
docker ps | grep "web"

# Find containers using a specific image
docker ps -a | grep "nginx"

# Find containers with a specific status
docker ps -a | grep "Exited"
docker ps -a | grep "Up"

# Get only container IDs matching a pattern
docker ps -a | grep "web" | awk '{print $1}'

# Find containers by port
docker ps | grep "8080"
docker ps | grep "0.0.0.0:80"

# Count running containers matching pattern
docker ps | grep -c "web"

# Find containers NOT matching pattern
docker ps | grep -v "pause"

# Case-insensitive search
docker ps -a | grep -i "NGINX"
```

### Image Grep Patterns

```bash
# Find images by name pattern
docker images | grep "node"

# Find images with specific tag
docker images | grep "alpine"

# Find large images (look for GB)
docker images | grep "GB"

# Find images without a tag (<none>)
docker images | grep "<none>"

# List image IDs matching pattern
docker images | grep "myapp" | awk '{print $3}'

# Find images created recently
docker images | grep "minutes ago"
docker images | grep "hours ago"

# Count images matching pattern
docker images | grep -c "python"
```

### Log Grep Patterns

```bash
# Search for errors in container logs
docker logs mycontainer 2>&1 | grep -i "error"

# Search for specific HTTP status codes
docker logs mycontainer 2>&1 | grep "HTTP/1.1\" 500"
docker logs mycontainer 2>&1 | grep -E "HTTP/1\.[01]\" [45][0-9]{2}"

# Count occurrences of a pattern
docker logs mycontainer 2>&1 | grep -c "ERROR"

# Show context around matches
docker logs mycontainer 2>&1 | grep -A 3 -B 1 "Exception"

# Grep with timestamps
docker logs -t mycontainer 2>&1 | grep "error"

# Follow logs with grep (real-time filtering)
docker logs -f mycontainer 2>&1 | grep --line-buffered "error"

# Search across a time range
docker logs --since 1h mycontainer 2>&1 | grep "timeout"

# Multiple patterns (OR)
docker logs mycontainer 2>&1 | grep -E "error|warning|fatal"

# Exclude patterns
docker logs mycontainer 2>&1 | grep -v "healthcheck"

# Extract IPs from logs
docker logs mycontainer 2>&1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'
```

### Inspect Grep Patterns

```bash
# Search environment variables
docker inspect mycontainer | grep -A 1 "Env"

# Find IP address
docker inspect mycontainer | grep "IPAddress"

# Find mounted volumes
docker inspect mycontainer | grep -A 5 "Mounts"

# Find port bindings
docker inspect mycontainer | grep -A 5 "PortBindings"

# Check restart policy
docker inspect mycontainer | grep -A 3 "RestartPolicy"

# Find image hash
docker inspect mycontainer | grep "Image"

# Check health status
docker inspect mycontainer | grep -i "health"

# Find network name
docker inspect mycontainer | grep "NetworkMode"
```

### Network & Volume Grep Patterns

```bash
# Find specific network
docker network ls | grep "bridge"

# Find unused volumes
docker volume ls | grep -v "VOLUME"

# Find network by driver
docker network ls | grep "overlay"

# Check which containers are on a network
docker network inspect mynet | grep "Name"
```

### Docker Compose + Grep

```bash
# Find services in a specific state
docker compose ps | grep "running"
docker compose ps | grep "exited"

# Search compose logs for errors
docker compose logs 2>&1 | grep -i "error"

# Filter logs for specific service
docker compose logs web 2>&1 | grep "404"

# Follow all service logs filtered
docker compose logs -f 2>&1 | grep --line-buffered "ERROR"
```

### Advanced Grep Patterns

```bash
# Regex: Find containers with port mappings
docker ps | grep -E "0\.0\.0\.0:[0-9]+"

# Perl-compatible regex (PCRE): Extract container names
docker ps --format '{{.Names}}' | grep -P "web-\d+"

# Invert match + count
docker ps -a | grep -vc "Up"          # Count stopped containers

# Recursive grep in container filesystem
docker exec mycontainer grep -r "password" /app/config/

# Grep docker-compose.yml for service dependencies
grep -A 5 "depends_on" docker-compose.yml

# Find Dockerfiles with specific base images
grep -r "FROM.*python" . --include="Dockerfile*"

# Find exposed ports in Dockerfiles
grep -r "EXPOSE" . --include="Dockerfile*"

# Check which containers are using a specific volume
docker ps -a --filter volume=mydata --format '{{.Names}}'
```

### Grep Flag Quick Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Case-insensitive | `grep -i "error"` |
| `-v` | Invert match (exclude) | `grep -v "healthcheck"` |
| `-c` | Count matches | `grep -c "ERROR"` |
| `-n` | Show line numbers | `grep -n "error" file` |
| `-l` | Show only filenames | `grep -rl "FROM" .` |
| `-r` | Recursive search | `grep -r "pattern" /dir` |
| `-E` | Extended regex (ERE) | `grep -E "error\|warn"` |
| `-P` | Perl-compatible regex | `grep -P "\d{3}"` |
| `-o` | Show only matched part | `grep -oE '[0-9.]+'` |
| `-A N` | N lines after match | `grep -A 3 "error"` |
| `-B N` | N lines before match | `grep -B 2 "error"` |
| `-C N` | N lines around match | `grep -C 5 "error"` |
| `-w` | Match whole word | `grep -w "error"` |
| `-x` | Match whole line | `grep -x "exact line"` |
| `--color` | Highlight matches | `grep --color "pattern"` |
| `--line-buffered` | Flush output per line (for pipes) | `docker logs -f c1 \| grep --line-buffered "err"` |

---

## One-Liners & Power Tricks

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker rm $(docker ps -aq -f status=exited)

# Remove all images
docker rmi $(docker images -q)

# Remove dangling images
docker rmi $(docker images -f "dangling=true" -q)

# Remove all unused (dangling + unreferenced) images
docker image prune -a -f

# Kill all running containers
docker kill $(docker ps -q)

# Get IP of a running container
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' CONTAINER

# Get container ID by name
docker inspect -f '{{.Id}}' CONTAINER_NAME

# Run and remove after exit
docker run --rm -it alpine sh

# Copy file from stopped container
docker cp $(docker create --name tmp IMAGE):/path/to/file ./file && docker rm tmp

# Inspect all env vars
docker exec CONTAINER env

# Watch resource usage
watch docker stats --no-stream

# Check image layers & size
docker history --no-trunc IMAGE

# Export all images to tar files
docker images --format '{{.Repository}}:{{.Tag}}' | xargs -I {} sh -c 'docker save {} > $(echo {} | tr "/" "_" | tr ":" "_").tar'

# Test a Dockerfile without building
docker build --check .

# Run a container with host networking
docker run --rm --network host curlimages/curl http://localhost:8080

# Attach to all logs at once (compose)
docker compose logs -f --tail=0

# Find which process in container is using most memory
docker exec CONTAINER ps aux --sort=-%mem | head

# Quick web server from current directory
docker run --rm -v $(pwd):/usr/share/nginx/html:ro -p 8080:80 nginx

# Run a database for quick testing
docker run --rm -d -p 5432:5432 -e POSTGRES_PASSWORD=pass postgres:16-alpine
docker run --rm -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pass mysql:8
docker run --rm -d -p 6379:6379 redis:alpine
docker run --rm -d -p 27017:27017 mongo:7
```

---

## Interview Quick-Fire Q&A

| # | Question | Answer |
|---|----------|--------|
| 1 | **`CMD` vs `ENTRYPOINT`?** | `CMD` = default args (easily overridden). `ENTRYPOINT` = main executable (use `--entrypoint` to override). Best practice: use together — `ENTRYPOINT ["python"]` + `CMD ["app.py"]` |
| 2 | **`COPY` vs `ADD`?** | `COPY` = simple copy. `ADD` = copy + auto-extract tar + fetch URLs. **Prefer `COPY`** for clarity. |
| 3 | **`docker stop` vs `docker kill`?** | `stop` sends SIGTERM (graceful, 10s default), then SIGKILL. `kill` sends SIGKILL immediately. |
| 4 | **`docker run` vs `docker exec`?** | `run` = create **new** container. `exec` = run command in **existing** running container. |
| 5 | **Dangling image?** | Image with no tag (`<none>:<none>`). Caused by rebuilds. Clean with `docker image prune`. |
| 6 | **Named volume vs bind mount?** | Named volume = Docker-managed, portable, in `/var/lib/docker/volumes`. Bind mount = host path directly mounted. |
| 7 | **`bridge` vs `host` network?** | `bridge` = isolated network with port mapping. `host` = shares host's network (no isolation, no port mapping needed). |
| 8 | **Multi-stage build?** | Multiple `FROM` statements. Build in one stage, copy artifacts to a minimal final stage. Reduces image size. |
| 9 | **`docker save` vs `docker export`?** | `save` = image with layers → `load`. `export` = container filesystem (flat) → `import`. |
| 10 | **Restart policies?** | `no` (default), `always`, `unless-stopped` (not after manual stop), `on-failure[:max-retries]`. |
| 11 | **Docker layer caching?** | Each Dockerfile instruction creates a layer. Unchanged layers are cached. **Order matters**: put rarely-changing instructions first. |
| 12 | **`.dockerignore`?** | Like `.gitignore` for build context. Excludes files from `COPY`/`ADD`. Reduces build time and image size. |
| 13 | **What is BuildKit?** | Next-gen build engine (default since Docker 23.0). Parallel builds, better caching, secret mounts, SSH forwarding. |
| 14 | **Docker Compose vs Swarm vs K8s?** | Compose = single-host multi-container. Swarm = built-in Docker orchestration. K8s = industry-standard orchestration (more complex, more features). |
| 15 | **Container vs VM?** | Container = shares host kernel, lightweight, fast. VM = full OS + hypervisor, heavier, stronger isolation. |
| 16 | **Docker image layers?** | Read-only layers stacked. Each instruction = layer. Container adds a thin R/W layer on top (union filesystem). |
| 17 | **Health checks?** | `HEALTHCHECK CMD curl -f http://localhost/ \|\| exit 1`. States: `starting`, `healthy`, `unhealthy`. Restart policies can act on health. |
| 18 | **What's `--init`?** | Adds `tini` as PID 1. Properly reaps zombie processes and forwards signals. Use for apps that don't handle signals. |
| 19 | **Overlay network?** | Multi-host networking in Swarm. Uses VXLAN. Containers on different hosts can communicate as if on same LAN. |
| 20 | **Docker Content Trust?** | `DOCKER_CONTENT_TRUST=1`. Ensures images are signed and verified. Uses Notary for signing. |

---

> [!TIP]
> **Pro tip for interviews:** When discussing Docker, mention security best practices:
> - Run as non-root user (`USER`)
> - Use minimal base images (`alpine`, `distroless`)
> - Scan images for vulnerabilities (`docker scout`)
> - Don't store secrets in images (use `--secret` at build, secrets/configs at runtime)
> - Use `.dockerignore` to exclude sensitive files
> - Set `--read-only` and drop capabilities (`--cap-drop ALL`)
> - Pin image versions (avoid `latest` in production)

---

*Last updated: August 2026 | Docker CLI v27+*

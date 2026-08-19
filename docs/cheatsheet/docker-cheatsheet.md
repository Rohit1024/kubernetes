# Docker complete cheatsheet

> **Purpose:** Reference guide for Docker CLI commands, flags, and grep patterns for operational troubleshooting and interviews.

---

## Table of contents

1. [Global flags (apply to any `docker` command)](#global-flags)
2. [Container lifecycle](#container-lifecycle)
3. [Container inspection and monitoring](#container-inspection-and-monitoring)
4. [Container interaction](#container-interaction)
5. [Image commands](#image-commands)
6. [Image build and Dockerfile](#image-build-and-dockerfile)
7. [Network commands](#network-commands)
8. [Volume commands](#volume-commands)
9. [Docker Compose](#docker-compose)
10. [Docker system and cleanup](#docker-system-and-cleanup)
11. [Docker registry and login](#docker-registry-and-login)
12. [Docker context and config](#docker-context-and-config)
13. [Docker Swarm orchestration](#docker-swarm-orchestration)
14. [Docker service commands](#docker-service-commands)
15. [Docker stack commands](#docker-stack-commands)
16. [Docker secrets and configs](#docker-secrets-and-configs)
17. [Docker plugin](#docker-plugin)
18. [Docker checkpoint](#docker-checkpoint)
19. [Docker trust and content trust](#docker-trust-and-content-trust)
20. [Docker manifest](#docker-manifest)
21. [Docker Buildx](#docker-buildx)
22. [Docker Scout security](#docker-scout-security)
23. [Grep combos with Docker](#grep-combos-with-docker)
24. [One-liners and power tricks](#one-liners-and-power-tricks)
25. [Interview quick-fire questions](#interview-quick-fire-questions)

---

## Global flags

These flags can be placed before any subcommand (for example, `docker --debug ps`).

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

## Container lifecycle

### `docker run`: Create and start a container

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
| `--link` | | Add link to another container (legacy, use networks) |
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
| `--init` | | Run an init process inside the container (tini) |
| `--label` | `-l` | Set metadata label on the container |
| `--platform` | | Set platform (`linux/amd64`, `linux/arm64`) |
| `--pull` | | Pull image before running: `always`, `missing`, `never` |
| `--sig-proxy` | | Proxy received signals to the process (default `true`) |
| `--stop-signal` | | Signal to stop a container (default `SIGTERM`) |
| `--stop-timeout` | | Timeout in seconds to stop a container |
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

### `docker create`: Create a container without starting it

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### `docker start`, `stop`, `restart`, `kill`

| Command | Key flags | Description |
|---------|-----------|-------------|
| `docker start` | `-i` (interactive), `-a` (attach) | Start stopped container(s) |
| `docker stop` | `-t` (timeout, default 10s) | Graceful stop (SIGTERM then SIGKILL) |
| `docker restart` | `-t` (timeout) | Stop then start |
| `docker kill` | `-s` (signal, default SIGKILL) | Send signal to running container |

```bash
docker stop -t 30 mycontainer     # 30s grace period
docker kill -s SIGUSR1 mycontainer
docker restart mycontainer
```

### `docker pause` and `unpause`

```bash
docker pause CONTAINER      # Freeze all processes (SIGSTOP via cgroups)
docker unpause CONTAINER    # Resume
```

### `docker rm`: Remove containers

| Flag | Description |
|------|-------------|
| `-f`, `--force` | Force remove a running container (uses SIGKILL) |
| `-v`, `--volumes` | Remove anonymous volumes associated with container |
| `-l`, `--link` | Remove the specified link |

```bash
docker rm mycontainer
docker rm -f $(docker ps -aq)      # Force remove all containers
docker rm -v mycontainer           # Clean up anonymous volumes
```

### `docker rename`

```bash
docker rename OLD_NAME NEW_NAME
```

### `docker update`: Update container configuration

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

## Container inspection and monitoring

### `docker ps`: List containers

| Flag | Short | Description |
|------|-------|-------------|
| `--all` | `-a` | Show all containers (default shows only running) |
| `--filter` | `-f` | Filter output (e.g., `status=running`, `name=web`, `label=env=prod`) |
| `--format` | | Pretty-print using Go template |
| `--last` | `-n` | Show N last created containers |
| `--latest` | `-l` | Show the latest created container |
| `--no-trunc` | | Do not truncate output |
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

### `docker inspect`: Low-level JSON inspection

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

### `docker logs`: Fetch container logs

| Flag | Short | Description |
|------|-------|-------------|
| `--follow` | `-f` | Follow log output |
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

### `docker top`: Running processes

```bash
docker top mycontainer
docker top mycontainer -aux       # Pass ps flags
```

### `docker stats`: Live resource usage

| Flag | Description |
|------|-------------|
| `--all` / `-a` | Show all containers (not just running) |
| `--no-stream` | Disable streaming (single snapshot) |
| `--no-trunc` | Do not truncate output |
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

### `docker events`: Real-time daemon events

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

## Container interaction

### `docker exec`: Run command in running container

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

### `docker attach`: Attach to container STDIO

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
> `docker attach` connects to PID 1. Typing `exit` stops the container. Use `exec` for a separate debugging shell.

### `docker cp`: Copy files between container and host

```bash
docker cp mycontainer:/app/log.txt ./log.txt      # Container to Host
docker cp ./config.yml mycontainer:/app/config.yml # Host to Container
docker cp mycontainer:/app/data/ ./backup/         # Copy directory
```

| Flag | Description |
|------|-------------|
| `-a`, `--archive` | Archive mode (preserve uid/gid metadata) |
| `-L`, `--follow-link` | Follow symlinks in source |

### `docker export` and `docker import`

```bash
docker export mycontainer > container.tar       # Export filesystem as tar
docker export -o container.tar mycontainer      # Same, with -o flag

cat container.tar | docker import - myimage:v1  # Import as image
docker import container.tar myimage:v1          # Same
docker import --change "ENV DEBUG=true" container.tar myimage:v1
```

---

## Image commands

### `docker images` and `docker image ls`

| Flag | Short | Description |
|------|-------|-------------|
| `--all` | `-a` | Show all images (including intermediate layers) |
| `--digests` | | Show digests |
| `--filter` | `-f` | Filter (`dangling=true`, `reference=nginx`, `label=key=val`) |
| `--format` | | Go template |
| `--no-trunc` | | Do not truncate output |
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

### `docker rmi`: Remove images

| Flag | Description |
|------|-------------|
| `-f`, `--force` | Force removal |
| `--no-prune` | Do not delete untagged parents |

```bash
docker rmi nginx:latest
docker rmi -f $(docker images -q)          # Remove all images
docker rmi $(docker images -f "dangling=true" -q)   # Remove dangling
```

### `docker history`: Image layer inspection

| Flag | Description |
|------|-------------|
| `--no-trunc` | Do not truncate output |
| `--format` | Go template |
| `-q`, `--quiet` | Only show image IDs |
| `-H`, `--human` | Human-readable sizes (default `true`) |

```bash
docker history nginx
docker history --no-trunc nginx
docker history --format "table {{.CreatedBy}}\t{{.Size}}" nginx
```

### `docker save` and `docker load`

```bash
docker save -o images.tar nginx:latest redis:latest   # Save images to tar
docker save nginx:latest | gzip > nginx.tar.gz        # Compressed archive

docker load -i images.tar                              # Load from tar
docker load < nginx.tar.gz                             # From stdin
```

---

## Image build and Dockerfile

### `docker build`

| Flag | Short | Description |
|------|-------|-------------|
| `--tag` | `-t` | Name and tag (`name:tag`) |
| `--file` | `-f` | Path to Dockerfile (default `./Dockerfile`) |
| `--build-arg` | | Set build-time variable |
| `--no-cache` | | Do not use cache during build |
| `--pull` | | Always pull a newer version of the base image |
| `--target` | | Set the target build stage (multi-stage) |
| `--platform` | | Set platform (`linux/amd64,linux/arm64`) |
| `--progress` | | Progress output type (`auto`, `plain`, `tty`) |
| `--secret` | | Expose a secret to the build |
| `--ssh` | | SSH agent socket or keys |
| `--cache-from` | | External cache source |
| `--cache-to` | | Export build cache |
| `--output` / `-o` | | Output destination (`type=local,dest=./out`) |
| `--network` | | Network mode for RUN instructions |
| `--label` | | Set metadata label |
| `--compress` | | Compress build context with gzip |
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

### Dockerfile instructions reference

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:20-alpine AS builder` |
| `RUN` | Execute command during build | `RUN apt-get update && apt-get install -y curl` |
| `CMD` | Default command (overridable) | `CMD ["node", "server.js"]` |
| `ENTRYPOINT` | Main executable process | `ENTRYPOINT ["python", "app.py"]` |
| `COPY` | Copy files from build context | `COPY --chown=node:node . /app` |
| `ADD` | Copy, auto-extract tar, and fetch URLs | `ADD app.tar.gz /app` |
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

---

## Network commands

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
docker network prune
```

---

## Volume commands

### `docker volume` subcommands

| Command | Description |
|---------|-------------|
| `docker volume create` | Create a volume |
| `docker volume ls` | List volumes |
| `docker volume inspect` | Detailed volume info |
| `docker volume rm` | Remove volume(s) |
| `docker volume prune` | Remove all unused volumes |

### Mount types comparison

| Type | Syntax | Use case |
|------|--------|----------|
| **Volume** | `-v mydata:/app/data` or `--mount type=volume,src=mydata,dst=/app/data` | Persistent data managed by Docker |
| **Bind** | `-v /host/path:/container/path` or `--mount type=bind,src=/host/path,dst=/container/path` | Direct host filesystem access |
| **tmpfs** | `--tmpfs /app/tmp` or `--mount type=tmpfs,dst=/app/tmp` | In-memory temporary storage |

---

## Docker Compose

### Core commands

| Command | Description |
|---------|-------------|
| `docker compose up` | Create and start containers |
| `docker compose down` | Stop and remove containers and networks |
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
| `docker compose pause` / `unpause` | Pause or unpause services |
| `docker compose events` | Receive real-time events |
| `docker compose cp` | Copy files between service containers and host |
| `docker compose ls` | List running compose projects |
| `docker compose watch` | Watch build context and rebuild or sync on changes |

```bash
docker compose up -d                                  # Start in background
docker compose up -d --build                          # Rebuild & start
docker compose up -d --scale web=3                    # Scale web to 3 replicas
docker compose down -v                                # Stop and remove volumes
docker compose down --rmi all                         # Stop and remove images
docker compose logs -f web                            # Follow web service logs
docker compose exec web bash                          # Shell into web service
docker compose run --rm web npm test                  # One-off command
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
docker compose config                                 # Validate compose file
```

---

## Docker system and cleanup

### `docker system` subcommands

| Command | Description |
|---------|-------------|
| `docker system df` | Show Docker disk usage |
| `docker system prune` | Remove unused data |
| `docker system info` | Display system-wide information |
| `docker system events` | Real-time events |

```bash
docker system df                         # Disk usage summary
docker system df -v                      # Verbose breakdown
docker system prune                      # Remove dangling images, stopped containers, unused networks
docker system prune -a --volumes -f      # Remove all unused containers, images, and volumes
```

---

## Docker registry and login

```bash
docker login                                              # Interactive login to Docker Hub
docker login -u myuser --password-stdin < password.txt    # Stdin login
docker login myregistry.io                                # Custom registry
docker logout myregistry.io
```

---

## Docker context and config

```bash
docker context create remote --docker "host=ssh://user@remote-host"
docker context ls
docker context use remote
docker context inspect remote
docker context rm remote
```

---

## Docker Swarm orchestration

```bash
docker swarm init --advertise-addr 192.168.1.10
docker swarm join --token SWMTKN-xxx 192.168.1.10:2377
docker swarm join-token worker            # Print worker join command
docker swarm join-token manager           # Print manager join command
docker swarm leave -f                     # Force leave
```

---

## Docker service commands

```bash
docker service create --name web --replicas 3 -p 80:80 nginx
docker service ls
docker service ps web
docker service logs -f web
docker service scale web=5
docker service update --image nginx:1.25 web
docker service rollback web
docker service rm web
```

---

## Docker stack commands

```bash
docker stack deploy -c docker-compose.yml mystack
docker stack ls
docker stack services mystack
docker stack ps mystack
docker stack rm mystack
```

---

## Docker secrets and configs

```bash
echo "db-secret-value" | docker secret create db_password -
docker secret ls
docker secret rm db_password

docker config create nginx_conf ./nginx.conf
docker config ls
docker config rm nginx_conf
```

---

## Docker plugin

```bash
docker plugin install vieux/sshfs
docker plugin ls
docker plugin disable vieux/sshfs
docker plugin rm vieux/sshfs
```

---

## Docker checkpoint

```bash
docker checkpoint create mycontainer mycp1
docker start --checkpoint mycp1 mycontainer
```

---

## Docker trust and content trust

```bash
export DOCKER_CONTENT_TRUST=1
docker trust inspect --pretty nginx:latest
docker trust sign myrepo/myimage:v1
docker trust revoke myrepo/myimage:v1
```

---

## Docker manifest

```bash
docker manifest create myrepo/app:latest \
  myrepo/app:amd64 \
  myrepo/app:arm64

docker manifest annotate myrepo/app:latest myrepo/app:arm64 \
  --os linux --arch arm64

docker manifest push myrepo/app:latest
```

---

## Docker Buildx

```bash
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap

# Multi-platform build and push
docker buildx build --platform linux/amd64,linux/arm64 -t myrepo/app:v1 --push .

# Build and load locally
docker buildx build --load -t myapp:test .
```

---

## Docker Scout security

```bash
docker scout quickview nginx:latest
docker scout cves nginx:latest
docker scout cves --only-severity critical,high nginx:latest
docker scout recommendations nginx:latest
docker scout compare --to nginx:1.24 nginx:1.25
```

---

## Grep combos with Docker

### Container grep patterns

```bash
# Filter running containers by name
docker ps | grep "web"

# Find containers using a specific image
docker ps -a | grep "nginx"

# Find exited containers
docker ps -a | grep "Exited"

# Extract container IDs matching a pattern
docker ps -a | grep "web" | awk '{print $1}'

# Find containers listening on a specific port
docker ps | grep "8080"
```

### Log grep patterns

```bash
# Search for errors in container logs
docker logs mycontainer 2>&1 | grep -i "error"

# Search for specific HTTP status codes
docker logs mycontainer 2>&1 | grep -E "HTTP/1\.[01]\" [45][0-9]{2}"

# Follow logs with unbuffered line filtering
docker logs -f mycontainer 2>&1 | grep --line-buffered "error"
```

---

## One-liners and power tricks

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker rm $(docker ps -aq -f status=exited)

# Remove dangling images
docker rmi $(docker images -f "dangling=true" -q)

# Extract IP address of a running container
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' CONTAINER

# Test a Dockerfile syntax without building
docker build --check .

# Quick ephemeral test database
docker run --rm -d -p 5432:5432 -e POSTGRES_PASSWORD=pass postgres:16-alpine
```

---

## Interview quick-fire questions

| # | Question | Answer |
|---|----------|--------|
| 1 | **`CMD` vs `ENTRYPOINT`?** | `CMD` sets default parameters that are easily overridden. `ENTRYPOINT` defines the executable process. Standard pattern: `ENTRYPOINT ["python"]` + `CMD ["app.py"]`. |
| 2 | **`COPY` vs `ADD`?** | `COPY` copies files from the build context. `ADD` also auto-extracts local tar files and fetches remote URLs. Use `COPY` by default. |
| 3 | **`docker stop` vs `docker kill`?** | `stop` sends SIGTERM with a graceful timeout (default 10s), followed by SIGKILL. `kill` sends SIGKILL immediately. |
| 4 | **`docker run` vs `docker exec`?** | `run` creates and starts a new container. `exec` executes a process inside an existing running container. |
| 5 | **Dangling image?** | An unreferenced image layer with no repository or tag name (`<none>:<none>`). Clean with `docker image prune`. |
| 6 | **Named volume vs bind mount?** | Named volumes are managed by Docker in `/var/lib/docker/volumes`. Bind mounts mount an arbitrary directory from the host filesystem. |
| 7 | **`bridge` vs `host` network?** | `bridge` provides private networking with NAT port forwarding. `host` shares the host network namespace directly without isolation. |
| 8 | **Multi-stage builds?** | Use multiple `FROM` stages to compile artifacts in a builder stage and copy only binaries to a lean runtime image. |
| 9 | **`docker save` vs `docker export`?** | `save` writes image layers and metadata for `docker load`. `export` flattens a container filesystem for `docker import`. |
| 10 | **Restart policies?** | `no` (default), `always`, `unless-stopped`, `on-failure[:max-retries]`. |

---

[← Cheatsheets overview](./index.md) | [Kubernetes cheatsheet →](./kubernetes-cheatsheet.md)

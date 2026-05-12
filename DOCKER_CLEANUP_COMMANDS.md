---

# Docker Cleanup & Pruning Commands

During local POC and infrastructure experimentation, Docker resources such as containers, images, volumes, networks, and build cache can accumulate and consume significant disk space.

This section provides commonly used cleanup and pruning commands for maintaining a healthy local Docker environment.

---

# Docker Resource Types

| Resource Type | Purpose | Common Issue |
| ------------- | ------- | ------------ |
| Containers | Running/stopped applications | Old stopped containers |
| Images | Docker templates | Large unused images |
| Volumes | Persistent data storage | Old database data |
| Networks | Container communication | Orphan networks |
| Build Cache | Image build layers | Disk consumption |
| System Data | Combined Docker storage | Overall Docker size growth |

---

# Container Commands

## View Running Containers

```bash
docker ps
```

---

## View All Containers

```bash
docker ps -a
```

---

## Remove Stopped Containers

```bash
docker container prune
```

---

## Remove Specific Container

```bash
docker rm -f <container-name>
```

Example:

```bash
docker rm -f mongodb-atlas-local
```

---

# Docker Image Commands

## View Docker Images

```bash
docker images
```

---

## Remove Unused Images

```bash
docker image prune
```

---

## Remove All Unused Images

```bash
docker image prune -a
```

IMPORTANT:

This removes all unused Docker images not currently attached to running containers.

---

## Remove Specific Image

```bash
docker rmi <image-id>
```

Example:

```bash
docker rmi mongodb/mongodb-atlas-local:latest
```

---

# Docker Volume Commands

Volumes are especially important for databases such as:

- MongoDB
- Kafka
- PostgreSQL
- Redis

Volumes preserve persistent data.

---

## View Volumes

```bash
docker volume ls
```

---

## Inspect Volume

```bash
docker volume inspect <volume-name>
```

---

## Remove Unused Volumes

```bash
docker volume prune
```

---

## Remove Specific Volume

```bash
docker volume rm <volume-name>
```

---

# Docker Network Commands

Docker Compose automatically creates Docker networks.

Unused networks may accumulate over time.

---

## View Networks

```bash
docker network ls
```

---

## Inspect Network

```bash
docker network inspect <network-name>
```

---

## Remove Unused Networks

```bash
docker network prune
```

---

# Docker Build Cache Commands

Docker build cache can consume significant disk space during repeated builds.

Especially useful for:

- Spring Boot image builds
- Dockerfile experiments
- Multi-stage builds

---

## View Build Cache

```bash
docker builder prune --dry-run
```

---

## Remove Build Cache

```bash
docker builder prune
```

---

## Remove All Build Cache

```bash
docker builder prune -a
```

---

# Docker System Disk Usage

## View Overall Docker Disk Usage

```bash
docker system df
```

Example:

```text
TYPE            TOTAL     ACTIVE    SIZE
Images          10        4         8GB
Containers      5         2         500MB
Volumes         8         3         12GB
Build Cache     20                  5GB
```

---

## Detailed Docker Disk Usage

```bash
docker system df -v
```

---

# Docker System Cleanup Commands

## Safe Cleanup

```bash
docker system prune
```

Removes:

- stopped containers
- unused networks
- dangling images
- build cache

---

## Aggressive Cleanup

```bash
docker system prune -a --volumes
```

Removes:

- stopped containers
- unused images
- unused volumes
- unused networks
- build cache

WARNING:

This can remove important local data.

Use carefully.

---

# Recommended Cleanup Strategy

## Weekly Safe Cleanup

```bash
docker container prune
docker image prune
docker volume prune
docker network prune
```

---

## Monthly Deep Cleanup

```bash
docker system prune -a
docker builder prune -a
```

---

## Full Local Environment Reset

```bash
docker compose down -v
docker system prune -a --volumes
```

Useful when:

- MongoDB replica-set issues occur
- Kafka metadata becomes corrupted
- old Docker networks cause problems
- stale volumes create startup failures

---

# Common Docker Issues Resolved by Cleanup

| Problem | Possible Cleanup Solution |
| -------- | ------------------------- |
| Port conflicts | Remove old containers |
| ReplicaSetNoPrimary | Remove old MongoDB volumes |
| Kafka startup failures | Remove old Kafka volumes |
| Docker disk full | System prune |
| Build failures | Builder prune |
| Network conflicts | Network prune |
| Corrupted metadata | Volume cleanup |

---

# Recommended Local Enterprise Architecture

```text
Spring Boot
     ↓
Kafka
     ↓
Consumer
     ↓
MongoDB Atlas Local
```

Over time, repeated local experimentation can create:

- stale volumes
- orphan networks
- old images
- unused containers
- corrupted persisted metadata

Periodic cleanup helps maintain stable local development environments.

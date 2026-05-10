# README_MAC_DOCKER_KAFKA.md

# Mac M1 Docker Kafka Setup Guide

Complete local Apache Kafka setup using Docker on Apple Silicon MacBook Air M1.

---

# System Configuration & Compatibility Note

## Compatibility Note

This setup guide is created using Apple Silicon MacBook Air M1 with macOS Sonoma, but the same steps can also be used for other Mac systems.

Supported environments include:

- Apple Silicon Macs (M1 / M2 / M3)
- Intel-based Macs

For Intel-based Macs, Docker images may automatically use:

```text
linux/amd64
```

while Apple Silicon Macs use:

```text
linux/arm64
```

Docker Desktop typically handles the architecture compatibility automatically.

This repository is intended to provide a reusable local enterprise-like Kafka and Docker setup for Mac-based development and proof-of-concept experimentation.

## Machine Details

| Component    | Value               |
| ------------ | ------------------- |
| Device       | MacBook Air M1 2020 |
| Chip         | Apple M1            |
| OS           | macOS Sonoma 14.6.1 |
| Memory       | 8 GB                |
| Architecture | ARM64               |

---

# Objective

This guide helps to set up:

- Docker Desktop
- Apache Kafka
- Zookeeper
- Local Kafka topics
- Spring Boot Kafka development environment

on Apple Silicon (M1/M2) Macs.

---

# Why ARM64 Configuration Matters

Apple M1 uses ARM64 architecture.

Some older Kafka Docker images support only:

```text
x86_64
```

This setup uses ARM64-compatible images to avoid:

- container crashes
- Rosetta emulation issues
- Kafka startup failures
- performance problems

---

# Prerequisites

Install the following:

- Docker Desktop
- Homebrew
- Java 17
- Maven
- VSCode

---

# STEP 1 — Verify Docker Installation

Check Docker version:

```bash
docker --version
```

Verify Docker daemon:

```bash
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```

If you see:

```text
Cannot connect to the Docker daemon
```

Start:

- Docker Desktop

---

# STEP 2 — Verify ARM64 Architecture

Run:

```bash
uname -m
```

Expected:

```text
arm64
```

---

# STEP 3 — Install Homebrew

Verify Homebrew:

```bash
brew --version
```

Install Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

# STEP 4 — Install Java 17

Install OpenJDK 17:

```bash
brew install openjdk@17
```

Configure Java:

```bash
sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
```

Add Java path:

```bash
echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Verify Java:

```bash
java -version
```

Expected:

```text
openjdk version "17"
```

---

# STEP 5 — Install Maven

Install Maven:

```bash
brew install maven
```

Verify Maven:

```bash
mvn -version
```

---

# STEP 6 — Create Kafka Working Directory

Create local project:

```bash
mkdir kafka-local
cd kafka-local
```

Create Docker KAFKA Compose file:

```bash
touch docker-kafka-compose.yml
```

---

# STEP 7 — Add Kafka Docker KAFKA Compose Configuration

Open:

```text
docker-kafka-compose.yml
```

Add:

```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    platform: linux/arm64
    container_name: zookeeper

    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

    ports:
      - '2181:2181'

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    platform: linux/arm64
    container_name: kafka

    depends_on:
      - zookeeper

    ports:
      - '9092:9092'

    environment:
      KAFKA_BROKER_ID: 1

      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181

      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

---

# STEP 8 — Start Kafka Containers

Run:

```bash
docker compose -f docker-kafka-compose.yml up -d
```

This will:

- pull Kafka image
- pull Zookeeper image
- create containers
- start services

---

# STEP 9 — Verify Running Containers

Check containers:

```bash
docker ps
```

Expected:

| Container | Status |
| --------- | ------ |
| kafka     | Up     |
| zookeeper | Up     |

---

# STEP 10 — Verify Kafka Logs

Check Kafka logs:

```bash
docker logs <kafka-container-name:output of (docker ps)>
```

Expected:

```text
Kafka Server started
```

---

# STEP 11 — Enter Kafka Container

Open Kafka container shell:

```bash
docker exec -it <kafka-container-name:output of (docker ps)> bash
```

You are now inside the Kafka Linux container.

---

# STEP 12 — Create Kafka Topic

Create topic:

```bash
kafka-topics --create \
--topic demo-topic \
--bootstrap-server localhost:9092 \
--partitions 1 \
--replication-factor 1
```

---

# STEP 13 — Verify Kafka Topic

List topics:

```bash
kafka-topics --list --bootstrap-server localhost:9092
```

Expected:

```text
demo-topic
```

---

# Kafka Architecture Flow

```text
Docker Desktop
        ↓
Zookeeper Container
        ↓
Kafka Broker Container
        ↓
Kafka Topic
        ↓
Producer / Consumer
```

---

# Useful Docker Commands

## View Running Containers

```bash
docker ps
```

## Stop Containers

```bash
docker compose -f docker-kafka-compose.yml down
```

## Restart Containers

```bash
docker compose -f docker-kafka-compose.yml restart
```

## View Logs

```bash
docker logs kafka
```

## Follow Live Logs

```bash
docker logs -f kafka
```

---

# Recommended Docker Resource Settings

Since the machine has:

```text
8 GB RAM
```

Recommended Docker Desktop configuration:

| Resource | Recommended |
| -------- | ----------- |
| Memory   | 4 GB        |
| CPU      | 4           |

Docker Desktop:

```text
Settings → Resources
```

---

# Common Issues

## Docker Daemon Not Running

Error:

```text
Cannot connect to the Docker daemon
```

Solution:

- Start Docker Desktop

---

## Kafka Container Restarting

Possible reasons:

- insufficient memory
- port conflict
- corrupted container

Check logs:

```bash
docker logs kafka
```

---

## Port Already In Use

Error:

```text
port is already allocated
```

Check:

```bash
lsof -i :9092
```

# Notes

This setup is optimized for:

- Apple Silicon M1/M2
- ARM64 Docker Images
- Local Kafka poc
- Event-driven architecture practice

---

---

# Important Note About Custom Docker Compose File Names

By default, Docker automatically detects:

```text
docker-compose.yml
```

If the compose file is renamed to a custom name such as:

```text
docker-kafka-compose.yml
```

Docker commands must explicitly specify the compose file using:

```bash
-f <filename>
```

Example:

## Start Containers

```bash
docker compose -f docker-kafka-compose.yml up -d
```

## Stop Containers

```bash
docker compose -f docker-kafka-compose.yml down
```

## Restart Containers

```bash
docker compose -f docker-kafka-compose.yml restart
```

This approach is useful for modular infrastructure setups where separate compose files are maintained for different technologies.

Example:

```text
docker-kafka-compose.yml
docker-mongodb-compose.yml
docker-redis-compose.yml
docker-monitoring-compose.yml
```

This structure follows enterprise-style infrastructure organization and helps isolate environments and services independently.

---

---

# Important Note About Docker Containers and Images

Docker containers and Docker images are different components.

| Component | Meaning                             |
| --------- | ----------------------------------- |
| Image     | Blueprint or template               |
| Container | Running instance created from image |

Example:

```text
Kafka Image
    ↓
Kafka Container
```

---

# Docker Lifecycle Commands

## STEP 1 — Verify Running Containers

```bash
docker ps
```

Displays all currently running containers.

---

## STEP 2 — Exit Kafka Container Shell

If currently inside Kafka container:

```bash
exit
```

or press:

```text
CTRL + D
```

This returns back to the macOS terminal.

---

## STEP 3 — Stop and Remove Containers

```bash
docker compose -f docker-kafka-compose.yml down
```

This command:

- stops Kafka container
- stops Zookeeper container
- removes containers
- preserves Docker images

---

## STEP 4 — Verify Containers Stopped

```bash
docker ps
```

Expected:

```text
No running containers
```

---

## STEP 5 — Verify Docker Images Still Exist

```bash
docker images
```

This displays locally available Docker images.

Kafka and Zookeeper images will still exist locally.

---

## STEP 6 — Restart Containers Using Existing Images

```bash
docker compose -f docker-kafka-compose.yml up -d
```

Containers will be recreated using already downloaded Docker images.

---

## STEP 7 — Remove Containers and Images

```bash
docker compose -f docker-kafka-compose.yml down --rmi all
```

This removes:

- containers
- Docker images

---

## STEP 8 — Remove Unused Containers

```bash
docker container prune
```

---

## STEP 9 — Remove Unused Images

```bash
docker image prune
```

---

# Repository Purpose

Hands-on Kafka and Docker POC repository focused on setting up enterprise-like local environments and performing proof-of-concept implementations on top of them by simulating real-time enterprise scenarios, understanding event-driven microservice flows, analyzing integration issues, troubleshooting failures, and researching practical solution approaches across distributed systems.

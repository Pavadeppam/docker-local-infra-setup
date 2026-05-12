# README_MAC_DOCKER_ATLAS_MONGODB.md

# Mac (M1/M2/M3) Docker MongoDB Atlas Local Setup Guide

Complete local MongoDB Atlas Local setup using Docker on Apple Silicon Mac.

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

This repository is intended to provide a reusable local enterprise-like MongoDB Atlas environment for Mac-based development and proof-of-concept experimentation.

---

# Tested Machine Details

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
- MongoDB Atlas Local
- Replica-set-enabled MongoDB
- MongoDB Compass
- Spring Boot MongoDB local development environment
- Local enterprise-like MongoDB environment

on Apple Silicon (M1/M2/M3) Macs.

---

# What is MongoDB Atlas Local

MongoDB Atlas Local is a Docker-based local MongoDB environment provided by MongoDB.

It provides:

- Replica set support
- Transactions
- Change streams
- Atlas-like local development environment
- Vector-search-ready environment
- Enterprise-like MongoDB behavior

Useful for:

- Spring Boot microservices
- Kafka consumers
- Event-driven architecture
- AI / RAG learning
- Vector database experimentation
- Local MongoDB research

---

# Why ARM64 Configuration Matters

Apple M1 uses ARM64 architecture.

Some older MongoDB Docker images support only:

```text
x86_64
```

This setup uses ARM64-compatible Docker images to avoid:

- container crashes
- Rosetta emulation issues
- MongoDB startup failures
- performance problems

---

# Prerequisites

Install the following:

- Docker Desktop
- MongoDB Compass (optional)
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

# STEP 3 — Create MongoDB Working Directory

Create local project:

```bash
mkdir mongodb-atlas-local
cd mongodb-atlas-local
```

Create Docker compose file:

```bash
touch docker-atlas-mongodb-compose.yml
```

---

# STEP 4 — Add MongoDB Atlas Local Docker Compose Configuration

Open:

```text
docker-atlas-mongodb-compose.yml
```

Add:

```yaml
version: '3.9'

services:
  mongodb-atlas-local:
    image: mongodb/mongodb-atlas-local:latest

    platform: linux/arm64

    container_name: mongodb-atlas-local

    hostname: mongodb-atlas-local

    ports:
      - '27017:27017'

    environment:
      MONGODB_INITDB_ROOT_USERNAME: admin
      MONGODB_INITDB_ROOT_PASSWORD: password123

    restart: unless-stopped
```

IMPORTANT:

Initially avoid persistent Docker volume mapping until successful first startup verification is completed.

---

# STEP 5 — Start MongoDB Atlas Local Container

Run:

```bash
docker compose -f docker-atlas-mongodb-compose.yml up -d
```

This will:

- pull MongoDB Atlas Local image
- create MongoDB container
- initialize replica set
- start MongoDB service

---

# STEP 6 — Verify Running Containers

Check containers:

```bash
docker ps
```

Expected:

| Container           | Status |
| ------------------- | ------ |
| mongodb-atlas-local | Up     |

Expected PORTS column:

```text
0.0.0.0:27017->27017/tcp
```

---

# STEP 7 — Verify MongoDB Logs

Check MongoDB logs:

```bash
docker logs mongodb-atlas-local
```

Live logs:

```bash
docker logs -f mongodb-atlas-local
```

Expected:

```text
Waiting for connections
```

---

# STEP 8 — Verify MongoDB Shell Connectivity

Open MongoDB shell:

```bash
docker exec -it mongodb-atlas-local mongosh -u admin -p password123
```

Expected:

```text
test>
```

---

# STEP 9 — Verify Replica Set Status

Inside mongosh:

```javascript
rs.status();
```

Expected important output:

```text
stateStr: 'PRIMARY'
```

This confirms:

- replica set initialized
- transactions supported
- change streams supported
- Atlas Local working correctly

---

# STEP 10 — Create Database

```javascript
use retaildb
```

---

# STEP 11 — Insert Sample Document

```javascript
db.products.insertOne({
  sku: 'SKU1001',
  brand: 'Nike',
  price: 5000,
  inventory: 100,
});
```

---

# STEP 12 — Verify Data

```javascript
db.products.find().pretty();
```

Expected sample output:

```json
{
  "_id": ObjectId(...),
  "sku": "SKU1001",
  "brand": "Nike",
  "price": 5000,
  "inventory": 100
}
```

---

# MongoDB Compass Setup

Install MongoDB Compass:

https://www.mongodb.com/products/tools/compass

---

# MongoDB Compass Connection String

Use EXACTLY:

```text
mongodb://admin:password123@127.0.0.1:27017/?authSource=admin&directConnection=true
```

IMPORTANT:

Use:

```text
127.0.0.1
```

NOT:

```text
localhost
```

Reason:

macOS Docker networking and MongoDB replica-set hostname resolution sometimes causes monitor connection failures when using localhost.

---

# Spring Boot MongoDB Connection URI

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://admin:password123@127.0.0.1:27017/retaildb?authSource=admin&directConnection=true
```

---

# MongoDB Atlas Local Architecture Flow

```text
Docker Desktop
        ↓
MongoDB Atlas Local Container
        ↓
Replica Set Initialization
        ↓
MongoDB Database
        ↓
Spring Boot / Kafka Consumer / MongoDB Compass
```

---

# Useful Docker Commands

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

## View Logs

```bash
docker logs mongodb-atlas-local
```

---

## Follow Live Logs

```bash
docker logs -f mongodb-atlas-local
```

---

## Search Logs (Case Insensitive)

```bash
docker logs mongodb-atlas-local 2>&1 | grep -i "waiting for connections"
```

---

## Enter Container Shell

```bash
docker exec -it mongodb-atlas-local bash
```

---

## Open MongoDB Shell

```bash
docker exec -it mongodb-atlas-local mongosh -u admin -p password123
```

---

## Stop Container

```bash
docker stop mongodb-atlas-local
```

---

## Start Existing Container

```bash
docker start mongodb-atlas-local
```

---

## Shutdown Using Compose

```bash
docker compose -f docker-atlas-mongodb-compose.yml down
```

---

# Docker Lifecycle Commands

## Remove Containers

```bash
docker rm -f mongodb-atlas-local
```

---

## Remove Images

```bash
docker rmi mongodb/mongodb-atlas-local:latest
```

---

## Remove Containers and Images Together

```bash
docker compose -f docker-atlas-mongodb-compose.yml down --rmi all
```

---

## Remove Unused Containers

```bash
docker container prune
```

---

## Remove Unused Images

```bash
docker image prune
```

---

# Docker Volume Commands

## View Volumes

```bash
docker volume ls
```

---

## Remove Specific Volume

```bash
docker volume rm <volume-name>
```

---

## Remove Unused Volumes

```bash
docker volume prune
```

---

## Remove Everything Including Volumes

```bash
docker compose -f docker-atlas-mongodb-compose.yml down -v
```

This removes:

- containers
- networks
- volumes

---

# Common Issues

## Port Already In Use

Error:

```text
bind: address already in use
```

Check:

```bash
lsof -i :27017
```

OR:

```bash
docker ps
```

Solution:

- stop existing MongoDB
- OR use another port

Example:

```yaml
ports:
  - '27018:27017'
```

---

## MongoDB Compass Connection Failure

Error:

```text
MongoServerSelectionError
connection <monitor> closed
```

Solution:

Use:

```text
mongodb://admin:password123@127.0.0.1:27017/?authSource=admin&directConnection=true
```

---

## ReplicaSetNoPrimary Error

Error:

```text
ReplicaSetNoPrimary
```

Solution:

```bash
docker compose -f docker-atlas-mongodb-compose.yml down -v
docker rm -f mongodb-atlas-local
docker volume prune
docker compose -f docker-atlas-mongodb-compose.yml up -d
```

---

## keyfile Missing Error

Error:

```text
Unable to acquire security key[s]
```

OR:

```text
Error reading file /data/configdb/keyfile
```

Solution:

```bash
docker compose -f docker-atlas-mongodb-compose.yml down -v
docker volume prune
docker compose -f docker-atlas-mongodb-compose.yml up -d
```

---

## ECONNREFUSED Error

Error:

```text
connect ECONNREFUSED 127.0.0.1:27017
```

Cause:

- MongoDB startup failure
- replica-set initialization incomplete

Check logs:

```bash
docker logs mongodb-atlas-local
```

Wait until:

```text
Waiting for connections
```

appears.

---

## Container Crash Loop

Check logs:

```bash
docker logs -f mongodb-atlas-local
```

Possible causes:

- corrupted volume
- failed replica-set initialization
- incompatible old persisted metadata

Solution:

- remove volumes
- recreate container cleanly

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

Useful for:

- microservices
- retail event processing
- AI ingestion pipelines
- RAG architectures
- CDC/event sourcing

---

# Recommended Docker Desktop Resource Settings

Since machine has:

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

# Repository Purpose

Hands-on MongoDB Atlas Local and Docker POC repository focused on setting up enterprise-like local environments.

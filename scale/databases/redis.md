---
title: Scaling with Redis
description: Configure Ant Media Server to use Redis as the cluster database, leveraging its in-memory speed, caching, and pub/sub capabilities for live streaming.
keywords: [Redis, Ant Media Server Redis, cluster database, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 1
---

# Scaling with Redis Database

Ant Media Server supports Redis as a cluster database, in addition to MongoDB and MapDB. Redis offers distinct advantages in certain use cases for live streaming infrastructure.

## Why Use Redis?

- **Speed and Low Latency**: In-memory data storage enables lightning-fast performance and minimal latency, ideal for real-time streaming.
- **Advanced Caching**: Reduces load on primary data sources and improves read operation speed.
- **Pub/Sub Messaging**: Enables real-time communication and event-driven architectures — useful for live chat, analytics, and signaling.
- **Scalability and High Availability**: Supports standalone and clustered deployments with fault tolerance.

## How to Deploy Redis

### 1. Self-Managed Deployment

Install and configure Redis manually on a server. See the [Redis documentation](https://redis.io/docs/getting-started/) for guidance.

### 2. Cloud-Based Managed Services

| Cloud | Service |
|---|---|
| AWS | MemoryDB for Redis / ElastiCache for Redis |
| Azure | Azure Cache for Redis |
| GCP | Google Cloud Memorystore |

### 3. Docker Deployment

Pull and run the [Redis Docker image](https://redis.io/download/#redis-downloads).

## Configure AMS to Use Redis

### Using `change_server_mode.sh` (for AMS running as a service)

```bash
# Standalone mode
sudo ./change_server_mode.sh standalone redis://[username:password@]host:port

# Cluster mode
sudo ./change_server_mode.sh cluster redis://[username:password@]host:port
```

### Using `start.sh` (for manual start / Kubernetes / Docker)

```bash
# Standalone mode
sudo ./start.sh -m standalone -h redis://[username:password@]host:port

# Cluster mode
sudo ./start.sh -m cluster -h redis://[username:password@]host:port
```

### TLS/SSL Connection

If your Redis server uses TLS, use the `rediss://` scheme (double `s`):

```bash
sudo ./change_server_mode.sh standalone rediss://[username:password@]host:port
```

### Kubernetes Deployment

In Kubernetes deployment YAML files, pass the Redis host as the `-h` argument:

```yaml
args: ["-g", "true", "-s", "true", "-r", "true", "-m", "cluster",
       "-h", "redis://username:password@host:port"]
```

---
title: MongoDB Atlas
description: Use MongoDB Atlas as the cluster database for Ant Media Server with the mongodb+srv connection string for multi-cloud resilient deployments.
keywords: [MongoDB Atlas, AMS cluster database, mongodb+srv, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 2
---

# Using MongoDB Atlas with AMS

MongoDB Atlas is a multi-cloud database service that simplifies deploying and managing databases while offering resilient and performant global applications across AWS, Azure, and GCP.

## Create a MongoDB Atlas Database

1. Navigate to the **Database** section of your Atlas account and click **Create a database**.
2. Choose the cluster type:
   - **Serverless**: Pay-per-request, ideal for variable workloads
   - **Dedicated**: Fixed capacity, best for consistent production loads
   - **Shared**: Free tier for development/testing
3. Select your preferred cloud provider and region.
4. Click **Create cluster**. The database will be ready in a few minutes.

## Configure Network Access

Navigate to **Network Access → Add IP Address** and add the IP addresses of your Ant Media Server instances (or `0.0.0.0/0` for all IPs — not recommended for production).

## Create Database Users

Navigate to **Database Access** and create a database user with the appropriate read/write permissions.

## Connect AMS to MongoDB Atlas

After creating the cluster, MongoDB Atlas provides a connection string in the format:

```
mongodb+srv://<username>:<password>@<url>
```

### Using `change_server_mode.sh` (for AMS running as a service)

```bash
cd /usr/local/antmedia
sudo ./change_server_mode.sh cluster mongodb+srv://<username>:<password>@<url>
```

### Using `start.sh` (for Kubernetes / Docker)

```bash
sudo ./start.sh -m cluster -h mongodb+srv://username:password@url
```

:::info
When specifying a `mongodb://` or `mongodb+srv://` URL, it is mandatory to provide the username and password in the connection string.
:::

## Verify the Connection

After switching to cluster mode, restart Ant Media Server and verify nodes are registered in the cluster:

```bash
sudo service antmedia restart
```

Check the **Cluster** section in the AMS dashboard to confirm nodes are connected.

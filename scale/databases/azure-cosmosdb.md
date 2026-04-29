---
title: Azure CosmosDB
description: Use Azure Cosmos DB for MongoDB as the cluster database for Ant Media Server with the mongodb+srv connection string.
keywords: [Azure CosmosDB, AMS cluster database, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 4
---

# Scaling with Azure CosmosDB

Azure Cosmos DB is a fully managed, globally distributed database service designed for scalability and high availability with MongoDB workloads. It provides automatic scaling, low latency, and multi-region availability.

## Prerequisites

- Your AMS server must be able to connect to Cosmos DB (check firewall and VNet settings).
- An Azure account with an active subscription.
- Select **MongoDB vCore API** when creating the Cosmos DB instance.

## Step 1: Create a CosmosDB Cluster

1. In the Azure Portal, click **Create a resource**.
2. Search for **Azure Cosmos DB** and select it.
3. On the **Which API best suits your workload?** page, choose **Azure Cosmos DB for MongoDB**, then click **Create**.
4. On the **Which type of resource?** page, select **Create** under the **vCore cluster** section.
5. Configure the **Cluster tier** as needed.
6. Enable **High availability** for production workloads.
7. Fill in the cluster details (name, username, password, region).
8. Under **Networking**, add firewall rules to allow your AMS server's IP to access the cluster.
9. Click **Review + create** and then **Create**.

## Step 2: Get the Connection String

After the cluster is created, go to the cluster page and copy the `mongodb+srv://` connection string from the **Connection strings** section.

## Step 3: Connect AMS to CosmosDB

In the `/usr/local/antmedia` directory, run:

```bash
sudo ./change_server_mode.sh cluster mongodb+srv://username:password@url
```

Example:

```bash
sudo ./change_server_mode.sh cluster "mongodb://username:password@cosmosdb-account-name.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&retryWrites=false"
```

Restart Ant Media Server after switching modes:

```bash
sudo service antmedia restart
```

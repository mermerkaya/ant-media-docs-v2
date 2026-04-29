---
title: AWS DocumentDB
description: Use AWS DocumentDB as the cluster database for Ant Media Server with a MongoDB-compatible connection string on the same VPC.
keywords: [AWS DocumentDB, AMS cluster database, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 3
---

# Scaling with AWS DocumentDB

AWS DocumentDB is a managed database service designed for scalability, high availability, and compatibility with MongoDB workloads. It simplifies the deployment and management of databases while providing flexibility for global applications on AWS.

## Prerequisites

- Your AMS server (standalone or cluster) must operate in the **same VPC** as your DocumentDB cluster.
- TLS must be **disabled** in DocumentDB (AMS does not use TLS for MongoDB connections by default).

## Step 1: Create a Parameter Group to Disable TLS

1. Open the Amazon DocumentDB service and go to **Parameter groups**.
2. Create a new Cluster Parameter Group.
3. In the parameters, select `tls`, click **Edit**, and set it to **disabled**.
4. Save the parameter group.

## Step 2: Create the DocumentDB Cluster

1. Go to **Clusters** and click **Create**.
2. Select instance class and number of instances.
3. Define your **username** and **password** for authentication.
4. Under **Advanced settings**, select the **Cluster Parameter Group** you created in Step 1.
5. Click **Create** to create the cluster.

## Step 3: Get the Connection String

1. Select your cluster and go to the **Connectivity & Security** tab.
2. Copy the connection string (endpoint URL).

## Step 4: Connect AMS to DocumentDB

In the `/usr/local/antmedia` directory, run:

```bash
sudo ./change_server_mode.sh cluster mongodb+srv://username:password@url
```

Example:

```bash
sudo ./change_server_mode.sh cluster "mongodb://testadmin:password@docdb-2024-08-25-19-28-55.cluster-crg1b1lxnbdb.ap-south-1.docdb.amazonaws.com:27017/?replicaSet=rs0&readPreference=secondaryPreferred&retryWrites=false"
```

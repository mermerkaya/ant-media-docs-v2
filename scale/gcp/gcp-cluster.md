---
title: GCP Cluster Deployment
description: Deploy Ant Media Server in a scalable cluster on Google Cloud Platform using Instance Templates, Managed Instance Groups, and Application Load Balancers.
keywords: [GCP cluster, Google Cloud Platform, Ant Media Server GCP, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 1
---

# Google Cloud Platform (GCP) Cluster Deployment

This guide walks through expanding your Ant Media Server infrastructure on GCP for optimal performance and scalability.

## Step 1: Install MongoDB Server

Create a VM instance in GCP:

1. Go to **VM Instances → CREATE INSTANCE** and select Ubuntu 22.04 as the boot disk.
2. Under **Advanced Options → Management → Automation**, add the following startup script:

```bash
#!/bin/bash
wget https://raw.githubusercontent.com/ant-media/Scripts/master/install_mongodb.sh \
  && chmod +x install_mongodb.sh
./install_mongodb.sh
```

3. Click **Create**. Note the instance's **private IP address**.

## Step 2: Install Ant Media Server Enterprise Edition

1. Launch **Ant Media Server Enterprise Edition** through the [GCP Marketplace](https://console.cloud.google.com/marketplace).
2. After deployment, **stop** the instance.
3. Go to **Images → CREATE IMAGE** and select the stopped instance as the **Source disk**.

This creates a custom machine image with AMS pre-installed.

## Step 3: Create an Instance Template

1. Navigate to **Instance Templates → CREATE INSTANCE TEMPLATE**.
2. Select the instance type (e.g., c2-standard-4).
3. Under **Boot Disk → Change → Custom Images**, select the image created in Step 2.
4. Under **Advanced Options → Management → Automation**, add the startup script:

```bash
#!/bin/bash
rm -rf /usr/local/antmedia/conf/instanceId
rm -rf /usr/local/antmedia/*.db.*
rm -rf /usr/local/antmedia/*.db
cd /usr/local/antmedia
./change_server_mode.sh cluster {MONGODB_PRIVATE_IP}
```

Replace `{MONGODB_PRIVATE_IP}` with your MongoDB instance's private IP.

5. Click **Create**.

## Step 4: Create Instance Groups (Origin and Edge)

1. Go to **Compute Engine → Instance Groups → CREATE INSTANCE GROUP**.
2. Select the instance template created above.
3. Choose **Multiple Zones** for location.
4. Configure autoscaling: minimum/maximum instance counts, CPU Utilization at 60%.
5. Click **Create**.

Repeat the same process for the **Edge group**.

## Step 5: Create an Application Load Balancer

1. Search for **Load Balancing** and click **CREATE LOAD BALANCER → Application Load Balancer**.
2. Add a new **SSL certificate** (or upload your own).
3. Under **Backend configuration**, create backend pools for both **Origin** and **Edge** instance groups.
4. Create a **Health Check** for the backends.
5. Configure routing:
   - Port **443** → Origin backend pool
   - Port **5443** → Edge backend pool
6. Click **Create** to finalize the Load Balancer.

## Step 6: Configure Firewall Rules

### Load Balancer Health Checks

Go to **VPC Network → Firewall → Create firewall rule** and allow health check traffic from Google's IP ranges to the AMS instances.

### Cluster Internal and WebRTC Traffic

Create a firewall rule allowing:
- **TCP port 5000** — internal cluster communication
- **UDP ports 50000-60000** — WebRTC media traffic

## Step 7: Configure DNS

In your DNS management, add records pointing to the Load Balancer IP addresses for both Origin and Edge.

## Testing

After all components are deployed:

- **Publish**: `https://origin-lb-domain/live/`
- **Play**: `https://edge-lb-domain:5443/live/player.html`

---
title: GCP Jinja Template
description: Deploy an Ant Media Server cluster on Google Cloud Platform using Jinja Templates and Google Deployment Manager for automated infrastructure provisioning.
keywords: [GCP Jinja template, Google Deployment Manager, AMS cluster, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 2
---

# AMS Cluster with GCP Jinja Template

Google Deployment Manager combined with Jinja templates lets you deploy a full AMS cluster — Origin, Edge, MongoDB, and Load Balancers — with a single CLI command.

```mermaid
flowchart TD
    YAML["antmedia.yaml\nZone, Region, Machine Types"]
    JINJA["antmedia-loadbalancer-template.jinja\nSSL Certificate"]
    DM["gcloud deployment-manager\ncreate ams-cluster"]
    OLB["ams-origin\nLoad Balancer"]
    ELB["ams-edge\nLoad Balancer"]
    OG["Origin Instance Group"]
    EG["Edge Instance Group"]
    MDB["MongoDB VM"]

    YAML --> DM
    JINJA --> DM
    DM --> OLB & ELB & OG & EG & MDB
    OLB --> OG
    ELB --> EG
```

## Prerequisites

- Google Cloud Platform account with billing enabled
- [Google Cloud SDK (gcloud)](https://cloud.google.com/sdk/docs/install) installed
- Compute Engine API enabled
- Infrastructure Manager API enabled
- An AMS Marketplace image created in your GCP project (note the image name, e.g. `ams-latest`)
- [AMS Cluster Jinja Template](https://github.com/ant-media/Scripts) downloaded from GitHub

## Step 1: Authenticate with GCP

```bash
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
```

## Step 2: Edit antmedia.yaml

Open `antmedia.yaml` from the downloaded template files and adjust region, zone, and instance types:

```yaml
properties:
  default_zone: us-central1-a
  default_region: us-central1
  mongodb_machine_type: e2-standard-2
  origin_machine_type: c2d-standard-4
  edge_machine_type: c2d-standard-4
```

If your AMS image is named something other than `ams-latest`:

```yaml
  image_id: "your-image-name"
```

## Step 3: Add SSL Certificate to the Jinja Template

Edit `antmedia-loadbalancer-template.jinja` and add your certificate content:

```yaml
- name: ams-ssl-cert-{{ scenario }}
  type: compute.v1.sslCertificate
  properties:
    certificate: |
      -----BEGIN CERTIFICATE-----
      <your certificate content>
      -----END CERTIFICATE-----
    privateKey: |
      -----BEGIN PRIVATE KEY-----
      <your private key content>
      -----END PRIVATE KEY-----
```

Ensure the certificate and private key content are correctly indented.

## Step 4: Deploy the Cluster

```bash
gcloud deployment-manager deployments create ams-cluster --config antmedia.yaml
```

Check deployment status:

```bash
gcloud deployment-manager deployments describe ams-cluster
```

If the deployment fails and needs to be restarted:

```bash
gcloud deployment-manager deployments delete ams-cluster
gcloud deployment-manager deployments create ams-cluster --config antmedia.yaml
```

## Step 5: Access Your Cluster

1. In your GCP Console, go to **Load Balancing**.
2. Two load balancers will be present: `ams-origin` and `ams-edge`.
3. Access the AMS dashboard via the **Origin Load Balancer's Public IP**.
4. Create your admin credentials on first login.

## What Was Deployed

| Resource | Description |
|---|---|
| Origin Instance Group | AMS nodes for ingest and transcoding |
| Edge Instance Group | AMS nodes for playback |
| MongoDB VM | Cluster metadata database |
| ams-origin Load Balancer | Routes traffic to Origin group |
| ams-edge Load Balancer | Routes traffic to Edge group |
| SSL Certificate | Attached to both load balancers |

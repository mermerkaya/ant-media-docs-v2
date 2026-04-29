---
title: Deploy AMS on Kubernetes
description: Manually deploy Ant Media Server on Kubernetes with Horizontal Pod Autoscaling, Nginx Ingress, MongoDB, and Let's Encrypt SSL certificates.
keywords: [Kubernetes, Deploy Ant Media Server on Kubernetes, HPA, Nginx Ingress, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 1
---

# Deploy Ant Media Server on Kubernetes

This guide explains how to manually deploy an auto-scaling Kubernetes environment for Ant Media Server.

:::info
You need the [Kubernetes CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/) and [Helm](https://helm.sh/docs/intro/install/) installed on your machine.
:::

## Architecture

Separate Origin and Edge deployments are strongly recommended:

- **Publish URL**: `ORIGIN_LOAD_BALANCER_URL/WebRTCAppEE`
- **Play URL**: `EDGE_LOAD_BALANCER_URL/WebRTCAppEE/player.html`

## Step 1: Install Metrics Server (for HPA)

Check if the Metrics Server is already installed:

```bash
kubectl get pods --all-namespaces | grep -i "metric"
```

If not installed, deploy it manually:

```bash
# Download components.yaml
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Edit components.yaml — add --kubelet-insecure-tls to args at line ~132
# spec.containers[0].args:
#   - --kubelet-insecure-tls

# Deploy
kubectl apply -f components.yaml

# Verify
kubectl get apiservices | grep "v1beta1.metrics.k8s.io"
```

## Step 2: Install Nginx Ingress Controller

```bash
# Via Helm
wget -qO- https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz | tar zxvf -
cd linux-amd64/
./helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
./helm repo update
./helm install ingress-nginx ingress-nginx/ingress-nginx

# Verify
kubectl get pods -n default | grep "ingress"
```

## Step 3: Deploy AMS with HostNetwork

```bash
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-mongodb.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-deployment-edge.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-deployment-origin.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-hpa-origin.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-hpa-edge.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-ingress-origin.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-ingress-edge.yaml
kubectl create -f https://raw.githubusercontent.com/ant-media/Scripts/master/kubernetes/ams-k8s-rtmp.yaml
```

## Step 4: Configure Horizontal Pod Autoscaling

Edit the CPU resource request in the deployment:

```bash
kubectl edit deployment ant-media-server-origin
kubectl edit deployment ant-media-server-edge
```

Set the CPU request (in millicores, 1000m = 1 core):

```yaml
resources:
  requests:
    cpu: 4000m
```

Create the HPA objects:

```bash
kubectl autoscale deployment ant-media-server-origin --cpu-percent=60 --min=1 --max=10
kubectl autoscale deployment ant-media-server-edge --cpu-percent=60 --min=1 --max=10
```

Monitor HPA status:

```bash
kubectl get hpa
```

## Step 5: Install SSL Certificate

### Option A: Custom Certificate

```bash
kubectl create secret tls antmedia-cert --key="ams.key" --cert="ams.crt"
```

### Option B: Let's Encrypt with cert-manager

```bash
# Install cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.9.1 \
  --set installCRDs=true

kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.9.1/cert-manager.crds.yaml
```

Create `ams-k8s-issuer-production.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-production
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-production
    solvers:
      - http01:
          ingress:
            class: nginx
```

```bash
kubectl create -f ams-k8s-issuer-production.yaml

# Delete self-signed certs
kubectl delete -n antmedia secret antmedia-cert-edge
kubectl delete -n antmedia secret antmedia-cert-origin

# Annotate ingress to use cert-manager
kubectl annotate ingress cert-manager.io/cluster-issuer=letsencrypt-production --all

# Verify certificates
kubectl get -n antmedia certificate
```

## Useful Commands

```bash
# Check HPA status
kubectl get hpa

# Check node resource usage
kubectl top nodes

# Check running pods
kubectl get pods

# Describe a deployment
kubectl describe deployment/ant-media-server-origin
```

## Deployment YAML Parameters

| Parameter | Description |
|---|---|
| `imagePullPolicy: IfNotPresent` | Use local image if available |
| `image:` | Your AMS Docker image name |
| `-g true` | Use public IP for internal cluster communication |
| `-s true` | Use public IP as server name |
| `-r true` | Replace local IP in ICE candidates with server name |
| `-m cluster` | Run in cluster mode |
| `-h <mongo-host>` | MongoDB host address |
| `-u <username>` | MongoDB username (if auth enabled) |
| `-p <password>` | MongoDB password (if auth enabled) |
| `-l <license>` | Ant Media Server license key |
| `hostNetwork: true` | Required for WebRTC UDP port range access |

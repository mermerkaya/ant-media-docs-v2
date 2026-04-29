---
title: NVIDIA GPU on Docker
description: Use NVIDIA hardware-based GPU encoding (NVENC) with Ant Media Server running in a Docker container via nvidia-container-toolkit.
keywords: [NVIDIA Docker, GPU encoding, NVENC, AMS Docker, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 2
---

# Using NVIDIA Hardware Encoder on Docker

Ant Media Server automatically uses NVIDIA GPU encoding (NVENC) when running in a Docker container with the NVIDIA Container Toolkit.

## Requirements

- Ubuntu 20.04 or 22.04
- NVIDIA GPU with NVENC support (check [Video Encode/Decode Support Matrix](https://developer.nvidia.com/video-encode-decode-gpu-support-matrix))
- CUDA Drivers installed (see [Using NVIDIA GPUs](../../advanced/using-nvidia-gpu.md))

## Step 1: Install Docker CE

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

## Step 2: Install NVIDIA Container Toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## Step 3: Run AMS with GPU Encoding

### Option A: Use the NVIDIA CUDA Base Image

Start a container with GPU access and install AMS Enterprise Edition inside:

```bash
docker run -d \
  --name nvidia \
  --runtime=nvidia \
  --privileged \
  --network host \
  -e NVIDIA_VISIBLE_DEVICES=all \
  -e NVIDIA_DRIVER_CAPABILITIES=compute,utility,video \
  -it nvidia/cuda:11.8.0-runtime-ubuntu22.04
```

Then install AMS Enterprise Edition inside the running container.

### Option B: Build a Custom GPU-Enabled AMS Image

Download the [AMS Dockerfile](https://github.com/ant-media/Scripts/blob/master/docker/Dockerfile_Process) and change the base image line from:

```dockerfile
FROM ubuntu:22.04
```

to:

```dockerfile
FROM nvidia/cuda:12.6.0-runtime-ubuntu22.04
```

Build the image (AMS Enterprise Edition ZIP must be in the same directory):

```bash
docker build --network=host \
  -t antmediaserver \
  --build-arg AntMediaServer=ant-media-server-enterprise.zip .
```

Run the container:

```bash
docker run -d \
  --name nvidia \
  --runtime=nvidia \
  --privileged \
  --network host \
  -e NVIDIA_VISIBLE_DEVICES=all \
  -e NVIDIA_DRIVER_CAPABILITIES=compute,utility,video \
  -it antmediaserver
```

## Verify GPU Usage

Inside the container or on the host:

```bash
nvidia-smi
```

When AMS is actively encoding, you will see GPU utilization in the `nvidia-smi` output. AMS automatically detects and uses the NVENC encoder when CUDA is available.

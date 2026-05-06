---
title: Create Android Project
sidebar_position: 1
description: Create an Android App Project in Android Studio for Ant Media Server WebRTC SDK
keywords: [Android SDK User Guide, Android Project, Android Studio, Ant Media Server Documentation, Ant Media Server Tutorials]
---

# Create Android Project

This guide walks you through creating a new Android app project in Android Studio for integrating Ant Media Server's WebRTC SDK.

```mermaid
flowchart LR
    AS[Android Studio] -->|New Project| Template[Select Template]
    Template -->|No Activity| Naming[Name Application]
    Naming -->|Finish| Project[MyWebRTCStreamingApp]
    Project --> AddSDK[Add SDK Dependency]
    AddSDK --> Develop[Develop Streaming App]
```

## Step 1: Open Android Studio

1. Launch Android Studio.

2. Click "Create New Project".

## Step 2: Select Project Template

1. In the project template selection, choose "No Activity".

2. Click Next.

## Step 3: Name Your Application

1. Enter a name for your application. For this guide, we will use:

   `MyWebRTCStreamingApp`.

2. Click Finish to create the project.

## Congratulations!

You have successfully created your WebRTC Android SDK application project!

Next, you can integrate the Ant Media Server SDK, configure your app for streaming, and start building your live streaming Android application.

Your project is now ready for coding, adding dependencies, and implementing WebRTC features.

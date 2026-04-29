---
title: Webinar Installation
description: Install the Circle Webinar application on Ant Media Server by uploading the WAR file through the Management Panel.
keywords: [Webinar installation, Circle Webinar install, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 2
---

# Installation of Circle Webinar on AMS

Circle Webinar is a web application that runs on Ant Media Server. To install the webinar tool, you need a WAR file of the webinar application. Download the latest WAR file from the [Ant Media releases page](https://github.com/ant-media/conference-call-application/releases).

## Application Installation

1. Log in to the Ant Media Server Management Panel.
2. On the Dashboard page, click the **New Application** button.
3. Click **Choose File** and browse to the downloaded WAR file.
4. Give the application a name (e.g., `webinar`).
5. Click the **Create** button.

After installation, visit:

```
https://domain:5443/webinar
```

Verify that the webinar room page loads successfully.

## Usage Summary

Once installed, use the following URL format to join a webinar room in each role:

| Role | URL Format |
|---|---|
| Host | `https://domain:5443/webinar/room1?role=host&streamName=host&skipSpeedTest=true` |
| Speaker | `https://domain:5443/webinar/room1?role=speaker&streamName=speaker&skipSpeedTest=true` |
| Listener | `https://domain:5443/webinar/room1?role=listener&streamName=listener&skipSpeedTest=true&playOnly=true` |

See the [Webinar Usage](./webinar-usage) documentation for detailed role behavior and host controls.

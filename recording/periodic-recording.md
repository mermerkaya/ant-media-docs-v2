---
title: Periodic Stream Recording
description: Capture periodic MP4 clips from live streams using the Clip Creator Plugin in Ant Media Server.
keywords: [Periodic Stream Recording, MP4 clips, Clip Creator Plugin, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 4
---

# Periodic Stream Recording

Ant Media Server supports **Periodic Stream Recording**, a powerful feature that lets you **capture short MP4 clips** from your live streams without interrupting the broadcast. It's perfect for saving highlights, generating snippets, or sharing quick moments with your audience.

This feature is powered by the [**Clip Creator Plugin**](https://github.com/ant-media/Plugins/tree/master/ClipCreatorPlugin) in the Enterprise Edition.

## What You Can Do

- Automatically record and save short MP4 clips from live streams.
- Define how often clips are saved (e.g., every 10 minutes).
- Generate MP4 clips instantly on demand via a REST API.
- Avoid manual HLS segment merging and configurations.

## Installation

### 1. Install FFmpeg

```bash
sudo apt install ffmpeg
```

### 2. Download the Plugin JAR File

Download the latest `clip-creator.jar` file from [Sonatype](https://oss.sonatype.org/#nexus-search;gav~io.antmedia.plugin~clip-creator~~~).

### 3. Place the Plugin in the Plugins Directory

```bash
sudo cp clip-creator.jar /usr/local/antmedia/plugins/
```

### 4. Restart Ant Media Server

```bash
sudo service antmedia restart
```

## Configuration

### 1. Enable HLS Streaming

Ensure [HLS is enabled](https://antmedia.io/docs/guides/playing-live-stream/hls-playing/#enable-hls) in your application settings.

### 2. Set Playlist Type to "event"

In Advanced App Settings:

```json
"hlsPlayListType": "event"
```

### 3. Set the Clip Recording Interval

Under Advanced App Settings, add the following in `customSettings`:

```json
"customSettings": {
  "plugin.clip-creator": {
    "mp4CreationIntervalSeconds": 600
  }
}
```

Replace `600` with your desired interval in seconds (e.g., `1800` = 30 minutes).

## Clip Storage Location

Generated MP4 clips are saved under:

```
/usr/local/antmedia/webapps/{AppName}/streams
```

Each clip is also registered as a **VoD entry** in the database.

## How Periodic Recording Works

At each configured interval, Ant Media Server:

1. Locates the most recent segment in the `.m3u8` playlist.
2. Goes back by the specified number of seconds.
3. Captures that range of segments.
4. Converts them into a single MP4 file.

Clip duration = `mp4CreationIntervalSeconds`

## REST API Endpoints

### Start Periodic Clip Creation

```bash
curl -X POST \
  "https://{YOUR_SERVER}:{5443}/{APP}/rest/clip-creator/periodic-recording/{periodSeconds}" \
  -H "Content-Type: application/json"
```

### Create MP4 Clip On-Demand

Trigger an MP4 clip generation instantly for a specific stream:

```bash
curl -X POST \
  "https://{YOUR_SERVER}:{5443}/{APP}/rest/clip-creator/mp4/{STREAM_ID}?returnFile=true" \
  -H "Content-Type: application/json"
```

- `returnFile=true`: The server creates the MP4 immediately and returns the file content as a response.
- `returnFile=false` (default): Returns a JSON response indicating whether the VoD creation was successful, including the `vodId` in the `dataId` field.

If a periodic MP4 has already been created, this captures the clip from the last MP4 creation time to now. If there is no MP4 created so far, the maximum duration will be around `mp4CreationIntervalSeconds`.

### Stop Periodic Clip Creation

```bash
curl -X DELETE \
  "https://{YOUR_SERVER}:{5443}/{APP}/rest/clip-creator/periodic-recording" \
  -H "Content-Type: application/json"
```

## Webhook Notification

Each time a new MP4 clip is created, a `vodReady` webhook is triggered. Check out the [Webhooks documentation](https://antmedia.io/docs/guides/advanced-usage/webhooks/) for more details.

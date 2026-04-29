---
title: Play Recorded Files
description: Play recorded MP4, WebM, and VoD files from Ant Media Server using embedded player or direct URLs.
keywords: [Play Recorded Files, VoD playback, live stream recording, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 3
---

# Play Recorded Files

This section covers how to play recorded files and use VoD-related APIs.

## Play VOD Files with MP4

First, make sure that MP4 recording is enabled in your application settings on the Web panel.

For example, if a live stream with streamId `stream1` is published to the `live` application on Ant Media Server, the MP4 file will be generated automatically once the stream finishes publishing.

**Community and Enterprise Editions — default MP4 URL:**
```
https://domain-or-IP:5443/LiveApp/streams/Stream_Id.mp4
```

**Enterprise Edition with Adaptive Bitrate (ABR) enabled:**
```
https://domain-or-IP:5443/LiveApp/streams/stream1_240p500kbps.mp4
https://domain-or-IP:5443/LiveApp/streams/stream1_480p1000kbps.mp4
```

## Play VOD Files with WebM

First, confirm that your application has WebM recording enabled. WebM can be recorded if the VP8 codec is enabled in the application's settings.

:::info
In the Community Edition, the VP8 codec is not available so WebM cannot be recorded.
:::

**Enterprise Edition with Adaptive Bitrate enabled:**
```
https://domain-or-IP:5443/LiveApp/streams/stream1_240p500kbps.webm
https://domain-or-IP:5443/LiveApp/streams/stream1_480p1000kbps.webm
```

## Play VoD Streams with Embedded Player

The embedded player (`play.html`) is available in both the Community and Enterprise Editions of Ant Media Server. Both live and VoD (recorded or uploaded) streams can be played by this player.

**When the live stream is over (recorded stream):**
```
https://domain-or-IP:5443/LiveApp/play.html?name=streamId&playOrder=vod
```

**For uploaded VOD:**
```
https://domain-or-IP:5443/LiveApp/play.html?name=vod-Id&playOrder=vod
```

Check out the [embedded player documentation](https://antmedia.io/docs/guides/playing-live-stream/embedded-web-player/) for more options.

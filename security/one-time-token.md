---
title: One-Time Token Control
description: Secure stream publishing and playback with one-time tokens in Ant Media Server.
keywords: [One Time Token Control, Stream Security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 2
---

# One-Time Token Control

You can enable One Time Token for publishing and playing from the application's settings. You have the option to use both the publish and playback tokens simultaneously or just one at a time.

![onetime-token](https://github.com/ant-media/ant-media-documentation/assets/86982446/2f118822-f997-4326-a5cc-f367e548bcd8)

Sending a token parameter with every publish request and play request is required if one-time token control is enabled. There will be an unauthorized access error if there is no token.

## Generate One-Time Token

The token can be generated with the [one-time token REST API](https://antmedia.io/rest/#/default/getTokenV2), which takes `streamId`, `expireDate`, and `type` as query parameters.

The `streamId` and `type` parameters must be properly defined because `tokenId` needs to match both `streamId` and `type`.

**Publish token:**
```bash
curl -X 'GET' 'https://IP-address-or-domain:5443/live/rest/v2/broadcasts/streamId/token?expireDate=Expire_Date&type=publish' -H 'accept: application/json'
```

**Play token:**
```bash
curl -X 'GET' 'https://IP-address-or-domain:5443/live/rest/v2/broadcasts/streamId/token?expireDate=Expire_Date&type=play' -H 'accept: application/json'
```

The expiration date should be provided as a Unix timestamp in seconds. You can convert dates to Unix timestamps using [epochconverter.com](https://www.epochconverter.com/).

## Token Usage by Protocol

### Publish URLs

| Protocol | URL Format |
|----------|-----------|
| RTMP | `rtmp://IP-address-or-domain/live/StreamId?token=tokenId` |
| SRT | `srt://IP-address-or-domain:4200?streamid=live/your-streamId,token=tokenId` |
| WebRTC | `https://domain:5443/live?id=streamId&token=tokenId` |

For WebRTC via WebSocket, insert the token in the WebSocket message:

```json
{
  "command": "publish",
  "streamId": "stream1",
  "token": "token"
}
```

### Playback URLs

**VOD:**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=streams/stream_id.mp4&playOrder=vod&token=tokenId
https://IP-address-or-domain:5443/Application_Name/streams/stream_id.mp4?token=tokenId
```

**HLS:**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=stream_id&playOrder=hls&token=tokenId
https://IP-address-or-domain:5443/Application_Name/streams/stream_id.m3u8?token=tokenId
```

:::info
If **Adaptive Bitrate (ABR)** is enabled and **WebRTC** stream is published, the original `.m3u8` file will **not be generated**. Use the adaptive or resolution-specific playlists instead:

```
https://<server>:5443/live/streams/<streamId>_adaptive.m3u8?token=<token>
```
:::

**CMAF (DASH):**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=stream_id&playOrder=dash&token=tokenId
https://IP-address-or-domain:5443/Application_Name/streams/streamId/streamId.mpd?token=tokenId
```

**WebRTC:**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=streamId&token=tokenId
```

For WebRTC via WebSocket:

```json
{
  "command": "play",
  "streamId": "stream1",
  "token": "token"
}
```

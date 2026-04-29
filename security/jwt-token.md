---
title: JWT Stream Security Filter
description: Secure stream publishing and playback with JWT (JSON Web Token) authentication in Ant Media Server.
keywords: [JWT, JWT Stream Security, Stream Security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 3
---

# JWT Stream Security Filter

You can enable JWT Stream Security Filter for publishing and playing from the application's settings via the AMS web panel. You have the option to use both the publish and playback tokens simultaneously or just one at a time.

![](@site/static/img/ant-media-server-jwt-stream-security-filter-dashboard.png)

Sending a token parameter with every publish request and play request is required if the JWT token is enabled. There will be an unauthorized access error if there is no token.

## Generate JWT Token

JWT Token can be generated in two ways:
1. Using the [JWT debugger](https://jwt.io/#debugger-io)
2. Using the [JWT token REST API](https://antmedia.io/rest/#/default/getJwtTokenV2)

To generate the token in both ways, `streamId`, `expireDate`, and `type` parameters are required. It is important that `streamId` and `type` are properly defined because `tokenId` needs to match both.

A **secret key** is also necessary. Once you enable the JWT token for publish or play in application settings, you need to generate the secret key as shown in the above screenshot.

### Generate JWT Token with Debugger

Using [JWT debugger at jwt.io](https://jwt.io/#debugger-io):

- Algorithm: HS256
- Secret key: your generated secret (e.g., `zautXStXM9iW3aD3FuyPH0TdK4GHPmHq`)
- Payload must include `streamId`, `expireDate`, and `type` parameters

![](@site/static/img/generate-jwt-stream-token-with-expiration.png)

The expiration time unit is Unix timestamp. When it expires, the JWT token becomes invalid.

### Generate JWT Token with REST API

**Publish token:**
```bash
curl -X 'GET' 'https://IP-address-or-domain:5443/live/rest/v2/broadcasts/streamId/jwt-token?expireDate=Expire_Date&type=publish' -H 'accept: application/json'
```

**Play token:**
```bash
curl -X 'GET' 'https://IP-address-or-domain:5443/live/rest/v2/broadcasts/streamId/jwt-token?expireDate=Expire_Date&type=play' -H 'accept: application/json'
```

Expire Date format is Unix Timestamp. You can get the timestamp [here](https://www.epochconverter.com/).

## JWT Token Usage by Protocol

### Publish URLs

| Protocol | URL Format |
|----------|-----------|
| RTMP | `rtmp://IP-address-or-domain/live/StreamId?token=tokenId` |
| SRT | `srt://IP-address-or-domain:4200?streamid=live/your-streamId,token=tokenId` |
| WebRTC | `https://domain:5443/live?id=streamId&token=tokenId` |

WebSocket WebRTC publish:
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

**CMAF (DASH):**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=stream_id&playOrder=dash&token=tokenId
https://IP-address-or-domain:5443/Application_Name/streams/streamId/streamId.mpd?token=tokenId
```

**WebRTC:**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=streamId&token=tokenId
```

WebSocket WebRTC play:
```json
{
  "command": "play",
  "streamId": "stream1",
  "token": "token"
}
```

## JWT vs One-Time Token

| Feature | One-Time Token | JWT Token |
|---------|----------------|-----------|
| Reusability | Single use | Configurable expiration |
| Flexibility | Per-session | Supports multiple scenarios |
| Use case | Temporary access | Persistent authenticated access |

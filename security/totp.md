---
title: Time-Based One-Time Password (TOTP)
description: Secure streams with TOTP (Time-Based One-Time Password) subscriber authentication in Ant Media Server.
keywords: [TOTP, Time Based One Time Password, Stream Security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 4
---

# Time-Based One-Time Password (TOTP)

The Time-based One-time Password algorithm (TOTP) is an extension of the HMAC-based One-time Password algorithm (HOTP) that generates a one-time password (OTP) by taking uniqueness from the current time.

A publisher or player is defined as a **subscriber**. If TOTP token is enabled, a subscriber must be created for the stream to be able to publish or play. Each subscriber has a `SubscriberId` and a `SubscriberCode`. When a subscriber requests to publish or play a stream, they must provide their `SubscriberId` and `SubscriberCode`. Otherwise, the server won't accept the request.

## Enable TOTP

You can enable TOTP for publishing and playing from the application's settings via the AMS web panel:

![](@site/static/img/stream-security/totp-enable.png)

To create a token, a secret key is required. Generate it by clicking the `Generate` option in the dashboard.

:::info
By default, the secret key is 6 bytes long when you click Generate. But in order to pre-register the subscriber, the secret key should be 8 bytes long.
:::

## Subscriber Operations

After enabling TOTP on the server, perform the following operations to register a subscriber.

You can generate the TOTP token without first registering the subscriber, but if `Accept Undefined Streams` option is not allowed, only pre-registered subscribers with pre-registered streamId can publish and play streams.

**Register a publisher subscriber:**
```bash
curl -X POST -H "Accept: Application/json" -H "Content-Type: application/json" \
  'http://Ip-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers' \
  -d '{"subscriberId":"publisherA","b32Secret":"SecretKey","type":"publish"}'
```

**Register a player subscriber:**
```bash
curl -X POST -H "Accept: Application/json" -H "Content-Type: application/json" \
  'http://Ip-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers' \
  -d '{"subscriberId":"playerA","b32Secret":"SecretKey","type":"play"}'
```

**Get subscriber list:**
```bash
curl -X 'GET' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscriber-stats/list/0/10' -H 'accept: application/json'
```

**Delete subscribers:**
```bash
curl -X 'DELETE' 'https://IP-address-or-domain:5443/Application_Name/rest/v2/broadcasts/streamId/subscribers' -H 'accept: application/json'
```

## TOTP Token Creation

TOTP token can be created using the [TOTP REST API](https://antmedia.io/rest/#/default/getTOTP).

By default, the TOTP generated for playback remains valid for 60 seconds. You can change the default TOTP time by changing:

```json
"timeTokenPeriod": 60
```

**Create TOTP for publish:**
```bash
curl -X 'GET' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers/SubscriberId/totp?type=publish' -H 'accept: application/json'
```

**Create TOTP for play:**
```bash
curl -X 'GET' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers/SubscriberId/totp?type=play' -H 'accept: application/json'
```

## Subscriber Block

The subscriber block feature allows blocking a specific user from publishing, playback, or both. The subscriber block feature is available in version 2.7.0 and later.

Enable the following property in the application advanced settings:

```json
"timeTokenSubscriberOnly": true
```

### Block Publish

After obtaining the TOTP token (6-byte `subscriberCode`), publish using the JavaScript SDK:

```js
webRTCAdaptor.publish(streamId, tokenId, subscriberId, subscriberCode);
// Example:
webRTCAdaptor.publish("teststream", null, "lastpeony", "451222");
```

Block publisher for 120 seconds:
```bash
curl -X 'PUT' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers/subscriberId/block/120/publish' -H 'accept: application/json'
```

Remove the block (set duration to 0):
```bash
curl -X 'PUT' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers/subscriberId/block/0/publish' -H 'accept: application/json'
```

### Block Play

Play using the JavaScript SDK with TOTP:
```js
webRTCAdaptor.play(streamId, tokenId, subscriberId, subscriberCode);
// Example:
webRTCAdaptor.play("teststream", null, "lastpeony", "451222");
```

Block player for 120 seconds:
```bash
curl -X 'PUT' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers/subscriberId/block/120/play' -H 'accept: application/json'
```

### Block Both Publish and Play

```bash
curl -X 'PUT' 'http://IP-address-or-domain:5080/live/rest/v2/broadcasts/streamId/subscribers/subscriberId/block/120/publish_play' -H 'accept: application/json'
```

## TOTP Usage by Protocol

### Publish URLs

**RTMP:**
```
rtmp://IP-address-or-domain/live/StreamId?subscriberId=your-subscriber&subscriberCode=totp-token
```

**SRT:**
```
srt://IP-address-or-domain:4200?streamid=live/your-streamId,subscriberId=your-subscriber,subscriberCode=totp-token
```

**WebRTC:**
```
https://domain:5443/live?id=streamId&subscriberId=your-subscriber&subscriberCode=totp-token
```

WebSocket WebRTC publish:
```json
{
  "command": "publish",
  "streamId": "stream1",
  "token": "token",
  "subscriberCode": "subscriberCode",
  "subscriberId": "subscriberId"
}
```

### Playback URLs

**HLS:**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=stream_Id&playOrder=hls&subscriberId=your-subscriber&subscriberCode=totp-token
https://IP-address-or-domain:5443/Application_Name/streams/stream_Id.m3u8?subscriberId=your-subscriber&subscriberCode=totp-token
```

**WebRTC:**
```
https://IP-address-or-domain:5443/Application_Name/play.html?id=streamId&subscriberId=your-subscriber&subscriberCode=totp-token
```

WebSocket WebRTC play:
```json
{
  "command": "play",
  "streamId": "stream1",
  "token": "token",
  "subscriberCode": "subscriberCode",
  "subscriberId": "subscriberId"
}
```

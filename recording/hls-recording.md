---
title: HLS Recording
description: Record live streams in HLS format and push HLS files to HTTP endpoints or S3 cloud storage with Ant Media Server.
keywords: [HLS Recording, HLS streaming, live stream recording, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 2
---

# HLS Recording

HLS streaming is a more cost-effective and secure method of streaming than video-on-demand (VOD). Furthermore, you can also record your live streams with HLS.

## Enable HLS Recording

To enable HLS recording for your live streams and store all the HLS `m3u8 and ts` files:

Navigate to `Applications → live → Settings → Advanced`, and configure the following settings:

![](@site/static/img/live-setting.png)

By default, only a certain number of TS files corresponding to segments are retained in the streams directory at any given time. By configuring the HLS playlist type to `event`, the server continuously generates TS files, allowing for permanent storage.

```json
"hlsPlayListType": "event"
```

To store HLS files permanently after the stream ends:

```json
"deleteHLSFilesOnEnded": false
```

To avoid overwriting old HLS files when reusing the same streamId, set the `append_list` attribute in the `hlsflags` property:

```json
"hlsflags": "+append_list"
```

Without `append_list`, restarting the same streamId would reset segment numbering from 0 and overwrite existing `.ts` files. With `append_list`, the numbering continues from where it left off.

To add date and timestamp to HLS files (useful when using the same stream ID again):

```json
"addDateTimeToHlsFileName": true
```

![](@site/static/img/hls_datetime.png)

After making the changes, scroll down and save the settings. Your streams will now be recorded as HLS.

## Record HLS Files to Cloud Storage

### HLS HTTP Endpoint

HLS HTTP Endpoint allows you to push HLS `m3u8 and ts` files to any HTTP endpoint, such as CDN, S3 bucket, or your own HTTP endpoint.

1. Open the management panel of your AMS. Go to Application settings → Advanced settings.

2. Find and edit the following property:

   ```json
   "hlsHttpEndpoint": "https://example.com/hls-stream/"
   ```

3. Save to apply the settings.

After that, when you push a stream with streamId `stream123`, AMS will push the files to the following endpoints with the PUT method:

```
https://example.com/hls-stream/stream123.m3u8
https://example.com/hls-stream/stream123_360p800kbps0001.ts
https://example.com/hls-stream/stream123_360p800kbps0002.ts
https://example.com/hls-stream/stream123_360p800kbps0003.ts
...
```

### Real-Time Upload to S3 Bucket

When you use standard S3 integration, your record is uploaded after the livestream finishes. If you want to upload HLS files **in real-time** to S3-compatible systems (AWS, OVH, Digital Ocean, etc.), use the `HLS Upload` servlet.

1. First, enter S3 credentials into the management console as defined in the S3 recording category.

2. Locate the setting `hlsHttpEndpoint` and set it to:

   ```json
   "hlsHttpEndpoint": "http://Domain-or-IP:5080/live/hls-upload"
   ```

   or:

   ```json
   "hlsHttpEndpoint": "http://127.0.0.1:5080/live/hls-upload"
   ```

   Replace `live` with your preferred application name.

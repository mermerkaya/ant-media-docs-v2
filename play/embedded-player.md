---
title: Embedded Web Player
description: Embed AMS Web Player to your webpage using iframe or the open-source Web Player React component.
keywords: [Embedded Web Player, Embedded Player using iFrame, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 5
---

# Embedded Web Player

There are several methods available for embedding an Ant Media Server custom video player onto your website to watch streams hosted on the Ant Media Server.

## play.html URL Parameters

The URL parameters listed below are accepted by the `play.html` page:

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `id` or `name` | Yes | — | The streamId to play |
| `token` | If token security is enabled | — | Token to access the stream |
| `autoplay` | No | true | Begin playback immediately if stream is accessible |
| `mute` | No | true | Begin playing with mute |
| `playOrder` | No | `webrtc,hls` | Playback protocol order. Values: `webrtc`, `hls`, `dash`, `vod` |
| `playType` | No | `mp4` | Required for recording playback. Values: `webm`, `mp4` |
| `targetLatency` | No | `3` | DASH player target latency |
| `is360` | No | false | Play 360-degree input stream |
| `backupStreamId` | No | — | Backup stream ID for failover playback |

Default WebRTC URL (no token):
```
https://AMS-domain-name:5443/live/play.html?name=streamId
```

With token:
```
https://AMS-domain-name:5443/live/play.html?name=streamId&token=generated-token
```

For HLS, Dash, or VOD playback:
```
https://AMS-domain-name:5443/live/play.html?name=streamId&playOrder=hls
```

## Method 1: iFrame Embedding

Using an `iframe` is a quick, but least customizable, way to integrate Ant Media Server streams into your website. The `play.html` page is in the application folder on Ant Media Server:

```
/usr/local/antmedia/webapps/live/play.html
```

You can copy the Embed code for a specific stream directly from the AMS dashboard.

Sample iFrame embed code:

```html
<iframe 
  width="560" 
  height="315" 
  src="https://AMS-domain-name:5443/live/play.html?name=stream1" 
  frameborder="0" 
  allowfullscreen>
</iframe>
```

To specify WebRTC playback only:
```html
<iframe 
  src="https://AMS-domain-name:5443/live/play.html?name=stream1&playOrder=webrtc"
  ...>
</iframe>
```

:::info
With `playOrder=webrtc`, the player will attempt WebRTC only. If a WebRTC stream isn't available, it will not play the stream at all. Without specifying `playOrder`, the player will fall back to HLS if WebRTC is unavailable.

Some secured websites do not accept embedded code with an HTTP URL — ensure SSL is configured on your Ant Media Server. See the [SSL section](https://antmedia.io/docs/guides/installing-on-linux/setting-up-ssl/).
:::

## Method 2: Ant Media Server Web Player (React)

The [Web Player](https://github.com/ant-media/Web-Player) is a custom video player developed by Ant Media, designed to facilitate playback of live streams. It accommodates all playback protocols (WebRTC, HLS, or CMAF Dash) and offers extensive customization options. It is fully open-source.

The Web Player utilizes the Ant Media Server Javascript SDK for WebRTC playback functionality.

### Step 1: Installation

```shell
npm i @antmedia/web_player
```

### Step 2: Import Web Player

```js
import { WebPlayer } from "@antmedia/web_player";
```

### Step 3: Add Video Player Container and Placeholder

Include a video player container and a placeholder in your component's render function:

```html
<div>
  <div style={{ display: 'flex', alignItems: 'center', flexDirection: 'column' }}>
    <span>Ant Media Embedded Player</span>
    <div style={{ display: 'flex', height: '360px', width: "640px" }} id="videoContainer" ref={videoRef}></div>
    <div
      id="placeHolder"
      ref={placeHolderRef}
      className="placeholder"
      style={{ height: '360px', overflow: 'hidden', display: 'flex', alignItems: 'center', justifyContent: 'center' }}
    >
      The streaming will begin shortly...
    </div>
  </div>
</div>
```

### Step 4: Initialize Web Player inside useEffect

```js
useEffect(() => {
  embeddedPlayerRef.current = new WebPlayer({
    streamId: "teststream",
    httpBaseURL: "http://localhost:5080/live/",
    videoHTMLContent: '<video id="video-player" class="video-js vjs-default-skin vjs-big-play-centered" controls playsinline style="width:100%;height:100%"></video>',
    playOrder: playOrderLocal
  }, videoRef.current, placeHolderRef.current);

  embeddedPlayerRef.current.initialize().then(() => {
    embeddedPlayerRef.current.play();
  }).catch((error) => {
    console.error("Error while initializing embedded player: " + error);
  });
}, []);
```

**Parameters:**
- `streamId`: The streamId for the stream your player will display.
- `httpBaseURL`: The URL to your AMS server and application. In production: `https://your_ams_url:5443/AppName/`
- `videoHTMLContent`: Content that the web player will inject into the videoContainer.
- `playOrder`: Array specifying the order of playback protocols.
- `videoRef.current`: Reference to the videoContainer element.
- `placeHolderRef.current`: Reference to the placeholder element.

### Full React Component Example

```jsx
import { useEffect, useRef } from 'react';
import { WebPlayer } from "@antmedia/web_player";

function App() {
  const videoRef = useRef(null);
  const placeHolderRef = useRef(null);
  const embeddedPlayerRef = useRef(null);
  const playOrderLocal = ["webrtc", "hls", "dash"];

  useEffect(() => {
    embeddedPlayerRef.current = new WebPlayer({
      streamId: "teststream",
      httpBaseURL: "http://localhost:5080/live/",
      videoHTMLContent: '<video id="video-player" class="video-js vjs-default-skin vjs-big-play-centered" controls playsinline style="width:100%;height:100%"></video>',
      playOrder: playOrderLocal
    }, videoRef.current, placeHolderRef.current);

    embeddedPlayerRef.current.initialize().then(() => {
      embeddedPlayerRef.current.play();
    }).catch((error) => {
      console.error("Error while initializing embedded player: " + error);
    });
  }, []);

  return (
    <div>
      <div style={{ display: 'flex', alignItems: 'center', flexDirection: 'column' }}>
        <span>Ant Media Embedded Player</span>
        <div style={{ display: 'flex', height: '360px', width: "640px" }} ref={videoRef} id="video_container"></div>
        <div
          ref={placeHolderRef}
          className="placeholder"
          style={{ height: '360px', overflow: 'hidden', display: 'flex', alignItems: 'center', justifyContent: 'center' }}
        >
          The streaming will begin shortly...
        </div>
      </div>
    </div>
  );
}

export default App;
```

### Using with Token Authentication

```js
const Token = "your-generated-token-value";

useEffect(() => {
  embeddedPlayerRef.current = new WebPlayer({
    streamId: "test",
    httpBaseURL: "https://test.antmedia.io:5443/live/",
    videoHTMLContent: '...',
    playOrder: playOrderLocal,
    token: Token
  }, videoRef.current, placeHolderRef.current);
```

## FAQs Related to Embedded Player

- Embedded player occasionally displaying a network warning: see [this discussion](https://github.com/orgs/ant-media/discussions/4923).
- Changing the language on the player while the stream is not active: see [this](https://github.com/orgs/ant-media/discussions/4880).
- Displaying poster image instead of text when stream is not active: see [this](https://github.com/orgs/ant-media/discussions/4877).

---
title: Accept Undefined Streams
description: Control whether Ant Media Server accepts streams with unregistered stream IDs.
keywords: [Accept Undefined Streams, Stream security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 1
---

# Accept Undefined Streams

This application setting controls whether the live stream is registered on Ant Media Server before it is allowed to publish.

```mermaid
graph TD
    Publisher["Publisher<br/>(any streamId)"] --> AMS["Ant Media Server"]
    AMS --> Check{"Accept Undefined<br/>Streams enabled?"}
    Check -->|Yes| Accept["Accept any streamId<br/>(allow all publishers)"]
    Check -->|No| Lookup{"streamId in<br/>database?"}
    Lookup -->|Yes| Accept2["Accept stream<br/>(registered publisher)"]
    Lookup -->|No| Reject["Reject stream<br/>(not_allowed_unregistered_streams error)"]
```

## Configure Undefined Stream Acceptance

![undefined-streams](https://github.com/ant-media/ant-media-documentation/assets/86982446/f456c3e9-dbae-42af-8a6f-34ee0aa177e8)

- If Ant Media Server **accepts undefined streams**, it will accept any incoming streams regardless of whether they are registered.
- If the `Accept Undefined Streams` option is **disabled** in application settings, then only streams with their streamId in the database are being accepted by Ant Media Server.

You can find more details about AMS application properties [here](https://antmedia.io/docs/guides/configuration-and-testing/ams-application-configuration/).

## Register a Stream

If this setting is disabled, first register the stream on the server by creating a live stream in the application with stream ID and stream name.

![](@site/static/img/stream-security/create-broadcast.png)

Now only the live stream with the created streamId will be published on the server and all other unregistered streams will be rejected.

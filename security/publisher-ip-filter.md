---
title: Publisher IP Filter
description: Restrict RTMP publishing to specific IP addresses or CIDR ranges with Ant Media Server.
keywords: [Publisher IP Filter, RTMP security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 6
---

# Publisher IP Filter

:::info
Publisher IP Filter feature currently works only for RTMP publishing.
:::

Publisher IP filter feature allows you to specify the IP addresses allowed for publishing. You can define multiple allowed IPs in CIDR format as comma (`,`) separated.

## Configuration

You can make changes in Publisher IP Filter from the application's Advanced settings via the AMS web panel.

```json
"allowedPublisherCIDR": "10.20.30.40/24,127.0.0.1"
```

This example allows IPs `10.20.30.[0-255]` and `127.0.0.1`. You can define a range or allow a single IP as well.

You can read more about CIDR notation [here](https://whatismyipaddress.com/cidr/).

## CIDR Format Examples

| CIDR | Allowed IPs |
|------|-------------|
| `192.168.1.0/24` | 192.168.1.0 – 192.168.1.255 |
| `10.0.0.0/8` | 10.0.0.0 – 10.255.255.255 |
| `127.0.0.1` | Only localhost |
| `0.0.0.0/0` | All IPs (no restriction) |

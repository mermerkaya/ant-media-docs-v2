---
title: License in Restricted Regions
description: Activate your Ant Media Server license from restricted geographic locations like China or Hong Kong using a free proxy service or a self-hosted Squid proxy.
keywords: [restricted geo license, China proxy, Squid proxy, license activation, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 3
---

# Activate AMS License in Restricted Regions

Ant Media Server uses Google services to verify license keys, which are blocked in China, Hong Kong, and some other regions. Two options are available to work around this restriction.

```mermaid
flowchart LR
    AMS["Ant Media Server\n(Restricted Region)"]
    PROXY["Proxy Server\n(Unrestricted Region)"]
    GOOG["Google License\nVerification Endpoint"]

    AMS -->|"proxy.address config"| PROXY
    PROXY --> GOOG
    GOOG -->|"valid / invalid"| PROXY
    PROXY -->|"result"| AMS
```

## Option 1: Free Proxy for Enterprise Users

Ant Media provides a free proxy service for Enterprise license holders. Email **support@antmedia.io** to receive your proxy credentials (username and password).

Once you have them:

```bash
echo "proxy.address=username:password@license-verification.antmedia.io:80" >> /usr/local/antmedia/conf/red5.properties
sudo systemctl restart antmedia
```

## Option 2: Self-Hosted Squid Proxy

Set up a Squid proxy on an Ubuntu server **in an unrestricted region**.

### 1. Install Squid

```bash
apt update
apt install squid apache2-utils -y
```

### 2. Backup Existing Configuration

```bash
mv /etc/squid/squid.conf{,_bck}
```

### 3. Configure Squid

Create `/etc/squid/squid.conf`:

```
acl whitelist dstdomain us-central1-ant-media-server-license.cloudfunctions.net
acl SSL_ports port 443
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
include /etc/squid/conf.d/*.conf
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED
http_access allow localhost
http_access allow authenticated whitelist
http_access deny all
http_port 3199
coredump_dir /var/spool/squid
```

### 4. Create Proxy Credentials

```bash
htpasswd -c /etc/squid/passwords username
```

Restart Squid:

```bash
systemctl restart squid
```

### 5. Test the Proxy

```bash
curl -x "http://username:password@your_proxy_server:3199" \
  -X POST \
  -H "Content-Type:application/json" \
  https://us-central1-ant-media-server-license.cloudfunctions.net/license_valid \
  -d '{"key":"your_license_key"}' \
  -w "\n"
```

A `"valid"` response confirms the proxy is working.

### 6. Configure AMS to Use Your Proxy

```bash
echo "proxy.address=username:password@your_proxy_server:3199" >> /usr/local/antmedia/conf/red5.properties
sudo systemctl restart antmedia
```

Open the AMS Dashboard and enter your license key in **Settings**. The verification request will be routed through your proxy.

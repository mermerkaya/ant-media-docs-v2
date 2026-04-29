---
title: HAProxy Load Balancer
description: Configure HAProxy as a load balancer for Ant Media Server with SSL termination, routing publish and play traffic to Origin and Edge nodes.
keywords: [HAProxy Load Balancer, SSL termination, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 4
---

# HAProxy Load Balancer with SSL Termination

HAProxy is a high-performance load balancer that can be used as the entry point for Ant Media Server clusters with SSL termination.

## Step 1: Install HAProxy

```bash
sudo apt-get update
sudo apt-get install haproxy
```

## Step 2: Install SSL Certificate

```bash
# Install Certbot
sudo apt-get install software-properties-common certbot

# Get the certificate
sudo certbot certonly --standalone -d example.com -d www.example.com

# Combine PEM files for HAProxy
sudo mkdir -p /etc/haproxy/certs
DOMAIN='example.com'
sudo -E bash -c "cat /etc/letsencrypt/live/$DOMAIN/fullchain.pem \
  /etc/letsencrypt/live/$DOMAIN/privkey.pem > /etc/haproxy/certs/$DOMAIN.pem"
sudo chmod -R go-rwx /etc/haproxy/certs
```

## Step 3: Configure HAProxy

Backup and create the configuration:

```bash
sudo mv /etc/haproxy/haproxy.cfg{,_backup}
sudo nano /etc/haproxy/haproxy.cfg
```

Paste the following configuration. Replace `{AMS_ORIGIN1_IP}`, `{AMS_ORIGIN2_IP}`, `{AMS_EDGE1_IP}`, and `$DOMAIN` with your actual values:

```haproxy
global
    log 127.0.0.1 local0 notice
    maxconn 2000
    user haproxy
    group haproxy

defaults
    log global
    mode http
    option forwardfor
    option http-server-close
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client  5000
    timeout server  5000
    timeout tunnel  2h    # Important for WebSocket connections
    timeout client-fin 5000

# HAProxy statistics panel
listen stats
    bind :6080
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /haproxy_stats
    stats auth username:password

# RTMP (TCP mode)
frontend rtmp_lb
    bind *:1935
    mode tcp
    default_backend backend_rtmp

backend backend_rtmp
    mode tcp
    server ams1 {AMS_ORIGIN1_IP}:1935 check
    server ams2 {AMS_ORIGIN2_IP}:1935 check

# HTTP Origin (port 80)
frontend http_lb_origin
    bind *:80
    mode http
    http-request add-header X-Forwarded-Proto http
    default_backend origin_backend_http

# HTTP Edge (port 5080)
frontend http_lb_edge
    bind *:5080
    mode http
    http-request add-header X-Forwarded-Proto http
    default_backend edge_backend_http

# HTTPS Origin (port 443) — Publish
frontend frontend_origin_https
    bind *:443 ssl crt /etc/haproxy/certs/$DOMAIN.pem
    http-request add-header X-Forwarded-Proto https
    default_backend origin_backend_http

# HTTPS Edge (port 5443) — Play
frontend frontend_edge_https
    bind *:5443 ssl crt /etc/haproxy/certs/$DOMAIN.pem
    http-request add-header X-Forwarded-Proto https
    default_backend edge_backend_http

backend origin_backend_http
    balance leastconn
    redirect scheme https if !{ ssl_fc }
    cookie JSESSIONID prefix nocache
    server origin1 {AMS_ORIGIN1_IP}:5080 check cookie origin1

backend edge_backend_http
    balance leastconn
    redirect scheme https if !{ ssl_fc }
    cookie JSESSIONID prefix nocache
    server edge1 {AMS_EDGE1_IP}:5080 check cookie edge1

# Dashboard (port 4444)
frontend frontend_dashboard
    bind *:4444 ssl crt /etc/haproxy/certs/$DOMAIN.pem
    http-request add-header X-Forwarded-Proto https
    default_backend dashboard_backend_http

backend dashboard_backend_http
    balance leastconn
    redirect scheme https if !{ ssl_fc }
    cookie JSESSIONID prefix nocache
    server dashboard1 {AMS_ORIGIN1_IP}:5080 check cookie dashboard1
    server dashboard2 {AMS_EDGE1_IP}:5080 check cookie dashboard2
```

## Step 4: Start HAProxy

```bash
sudo systemctl restart haproxy
```

## Access Ant Media Server

- **Dashboard**: `https://haproxy-domain:4444`
- **Publish**: Connect to port `443`
- **Play**: Connect to port `5443`

## Access HAProxy Statistics

View backend status at:

```
http://haproxy-domain:6080/haproxy_stats
```

Use the username and password defined in the configuration above.

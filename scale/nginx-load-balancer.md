---
title: Nginx Load Balancer
description: Set up Nginx as a load balancer for Ant Media Server with SSL termination, routing publish requests to Origin and play requests to Edge nodes.
keywords: [Nginx Load Balancer, Ant Media Server load balancer, SSL termination, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 3
---

# Nginx Load Balancer

Nginx is an open-source web server that also serves as a reverse proxy, HTTP load balancer, and email proxy. This guide explains how to set up Nginx as a load balancer for Ant Media Server.

## Port Routing Convention

| Port | Direction | Target |
|---|---|---|
| 443 (HTTPS) | Publish requests | Origin Group |
| 5443 (HTTPS) | Play requests | Edge Group |
| 4444 (HTTPS) | Dashboard access | Origin/Edge (ip_hash) |
| 1935 (TCP) | RTMP ingest | Origin Group |

## Option 1: Automated Script Installation

Download and run the automated Nginx configuration script:

```bash
wget https://raw.githubusercontent.com/ant-media/Scripts/master/nginx/install_and_configure_nginx.sh \
  && chmod +x install_and_configure_nginx.sh
```

View usage options:

```bash
./install_and_configure_nginx.sh
```

Example — install Nginx, configure Origin/Edge, and set up SSL with Let's Encrypt:

```bash
./install_and_configure_nginx.sh \
  -o "192.168.1.201" \
  -e "192.168.1.202,192.168.1.203" \
  -d example.com \
  -m user@example.com
```

Script options:

| Option | Description |
|---|---|
| `-o "IP1,IP2"` | Origin server IPs |
| `-e "IP1,IP2"` | Edge server IPs |
| `-d domain.com` | Your domain name |
| `-m email@example.com` | Email for Let's Encrypt |
| `-s` | Enable SSL in config |
| `-c` | Create Nginx config only (no install) |

## Option 2: Manual Step-by-Step Installation

### Step 1: Install Nginx

```bash
sudo apt install curl ca-certificates lsb-release -y

echo "deb http://nginx.org/packages/`lsb_release -d | awk '{print $2}' | tr '[:upper:]' '[:lower:]'` `lsb_release -cs` nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list

curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -

apt update && apt install nginx -y
```

### Step 2: Install Let's Encrypt SSL

```bash
sudo apt install certbot python3-certbot-nginx -y
certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Add certificate auto-renewal to crontab:

```
0 0 */80 * * root certbot -q renew --nginx
```

### Step 3: Configure Nginx

Backup the default configuration:

```bash
mv /etc/nginx/nginx.conf{,_bck}
```

Create a new `/etc/nginx/nginx.conf` with the following content. Replace `{AMS_ORIGIN1_IP}`, `{AMS_ORIGIN2_IP}`, `{AMS_EDGE1_IP}`, `{AMS_EDGE2_IP}`, and `{YOUR_DOMAIN}` with your actual values:

```nginx
# RTMP stream configuration
stream {
    upstream stream_backend {
        server {AMS_ORIGIN1_IP}:1935;
        server {AMS_ORIGIN2_IP}:1935;
    }
    
    server {
        listen 1935;
        proxy_pass stream_backend;
        proxy_timeout 3s;
        proxy_connect_timeout 1s;
    }
}

user nginx;
worker_processes auto;
pid /var/run/nginx.pid;
worker_rlimit_nofile 1048576;

events {
    worker_connections 1048576;
    multi_accept on;
    use epoll;
}

http {
    # Ant Media Origin
    upstream antmedia_origin {
        least_conn;
        server {AMS_ORIGIN1_IP}:5080;
        server {AMS_ORIGIN2_IP}:5080;
    }

    # Ant Media Edge
    upstream antmedia_edge {
        least_conn;
        server {AMS_EDGE1_IP}:5080;
        server {AMS_EDGE2_IP}:5080;
    }

    # Dashboard (ip_hash for session stickiness)
    upstream antmedia_dashboard {
        ip_hash;
        server {AMS_EDGE1_IP}:5080;
        server {AMS_ORIGIN1_IP}:5080;
    }

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    server_tokens off;
    keepalive_timeout 300s;

    ssl_protocols TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Redirect HTTP to HTTPS
    server {
        listen 80 default_server;
        server_name _;
        return 301 https://$host$request_uri;
    }

    # Origin (Publish) on port 443
    server {
        listen 443 ssl;
        ssl_certificate /etc/letsencrypt/live/{YOUR_DOMAIN}/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/{YOUR_DOMAIN}/privkey.pem;
        server_name yourdomain.com;

        location / {
            proxy_pass http://antmedia_origin;
            proxy_http_version 1.1;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header X-Forwarded-Proto https;
        }
    }

    # Edge (Play) on port 5443
    server {
        listen 5443 ssl;
        ssl_certificate /etc/letsencrypt/live/{YOUR_DOMAIN}/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/{YOUR_DOMAIN}/privkey.pem;
        server_name yourdomain.com;

        location / {
            proxy_pass http://antmedia_edge;
            proxy_http_version 1.1;
            proxy_connect_timeout 7d;
            proxy_send_timeout 7d;
            proxy_read_timeout 7d;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header X-Forwarded-Proto https;
        }
    }

    # Dashboard on port 4444
    server {
        listen 4444 ssl;
        ssl_certificate /etc/letsencrypt/live/{YOUR_DOMAIN}/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/{YOUR_DOMAIN}/privkey.pem;
        server_name yourdomain.com;

        location / {
            proxy_pass http://antmedia_dashboard;
            proxy_http_version 1.1;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
        }
    }
}
```

### Step 4: Apply Configuration

```bash
sudo rm -f /etc/nginx/sites-enabled/default
nginx -t
systemctl enable nginx
systemctl restart nginx
```

:::info
When using Nginx as a Load Balancer, use **port 4444** to access the AMS Dashboard.
:::

# N8N Home Install

## Setup Environemtn Variables

Create .env file, e.g.

```bash
DOMAIN_NAME=lizhao.net

SUBDOMAIN=n8n

GENERIC_TIMEZONE=Asia/Singapore

SSL_EMAIL=dev@lizhao.net
```

## Start n8n

```bash
docker compose up -d
```

## DNS in container

At beginning my container failed to connect to Internet service due to DNS errors. Not sure whether it's relevant or not, to update `/etc/docker/daemon.json` to remove `"bridge": false`.
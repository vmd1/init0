---
date: '2025-10-05T17:09:13Z'
draft: false
title: 'Notifications, Notifications… and More Notifications'
tags:
- Lab
- Guide
- Services
---
Staying updated when it comes to any part of your life is always crucial, but even more so when it comes to your homelab. 

> Enter [ntfy.sh](http://ntfy.sh) - a simple notification system powered by HTTP Requests and topics 

## Hosting

Ntfy has a cloud-hosted option, hosted at [ntfy.sh](http://ntfy.sh), however there’s no fun in that, so let’s get into how to self-host it, using Docker.

```yaml
services:
  ntfy:
    image: binwiederhier/ntfy
    container_name: ntfy
    command:
      - serve
    environment:
      - TZ=UTC    # optional: set desired timezone
    volumes:
      - /srv/ntfy/cache:/var/cache/ntfy
      - /srv/ntfy/lib:/var/lib/ntfy
      - /srv/ntfy:/etc/ntfy
    networks:
	    - proxy
    restart: unless-stopped
    
networks:
	proxy:
		external: true
```

Here’s an example config file at `/srv/ntfy/server.yml`  - see the docs [here](https://docs.ntfy.sh/config/):

```yaml
base-url: "https://ntfy.example.com"
listen-http: "127.0.0.1:2586"
cache-file: "/var/cache/ntfy/cache.db"
behind-proxy: true
attachment-cache-dir: "/var/cache/ntfy/attachments"
smtp-sender-addr: "######"
smtp-sender-user: "#########"
smtp-sender-pass: "################"
smtp-sender-from: "#########@example.com"
keepalive-interval: "45s"
auth-file: "/var/lib/ntfy/user.db"
auth-default-access: "deny-all"
auth-users:
  - "USERNAME:PASSWORD_HASH:GROUP"
auth-access:
  - "USERNAME:TOPIC:PERM"
```

You can create a password hash [here.](https://bcrypt-generator.com/)

```bash
docker compose up -d
```

And, you’re up and running.

## Integrations

### SSH Login

When a login occurs to your server over SSH, a script can be used to send a notification to ntfy, allowing you to know that this happened. This is relatively simple - all you need to do is add a few lines to your PAM config, and create a script.

Here’s an example script (which we’ll assume is at `/scripts/ntfy-ssh.sh`):

```bash
#!/bin/bash
machine_name=$(uname -n)

if [ "${PAM_TYPE}" = "open_session" ]; then
  curl \
    -H "X-Tags: warning,ssh" \
    -d "[${machine_name}] SSH login: ${PAM_USER} from ${PAM_RHOST}" \
    https://ntfy.example.com/ssh-logins
fi
```

And then, edit and append the following to your PAM SSH config:

```bash
sudo nano /etc/pam.d/sshd
```

```bash
session optional pam_exec.so /scripts/ntfy-ssh.sh
```

### Home Assistant

Connecting ntfy to HA is pretty easy as well, all you need to do is add the integration. Once you do so, you can call the service `notify.notify` with ntfy notification entity as the `entity_id` , as well as any title/message.

Ntfy really is quite a nifty little tool, helping keep you updated about things going on, yet remaining pretty lightweight.
---
title: "Basic UFW Firewall Setup"
draft: false
date: 2025-05-19
tags:
  - linux
  - security
  - sysadmin
---

UFW (Uncomplicated Firewall) provides a user-friendly interface for managing iptables on Linux systems. It is highly recommended to configure it on any internet-facing server.

Install UFW if it is not already present:

```bash
sudo apt update
sudo apt install ufw
```

Set the default policies to deny incoming connections and allow outgoing connections:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH connections to prevent locking yourself out of the server. If you use a custom SSH port (e.g., 2222), specify the port instead:

```bash
sudo ufw allow ssh
# OR for custom port
sudo ufw allow 2222/tcp
```

Allow standard web traffic (HTTP/HTTPS):

```bash
sudo ufw allow http
sudo ufw allow https
```

Enable the firewall and check its status:

```bash
sudo ufw enable
sudo ufw status verbose
```

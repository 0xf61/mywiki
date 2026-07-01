---
title: "Basic UFW Firewall Configuration"
draft: false
date: 2026-05-22
tags:
  - security
  - system-administration
---

## 1. Check UFW Status

Ensure UFW is installed and check its current status.

```bash
sudo ufw status verbose
```

## 2. Set Default Policies

Deny all incoming traffic and allow all outgoing traffic by default.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## 3. Allow SSH Connections

Always allow SSH connections before enabling the firewall to prevent locking yourself out.

```bash
sudo ufw allow ssh
```

## 4. Allow Specific Services

Allow HTTP and HTTPS traffic for web servers.

```bash
sudo ufw allow http
sudo ufw allow https
```

To allow a specific IP address to access a specific port:

```bash
sudo ufw allow from 192.168.1.100 to any port 3306 proto tcp
```

## 5. Enable the Firewall

Apply the rules and enable UFW to start on boot.

```bash
sudo ufw enable
```

## 6. Delete a Rule

To remove an existing rule, view the numbered rules:

```bash
sudo ufw status numbered
```

Then delete the rule by its number (e.g., rule 3):

```bash
sudo ufw delete 3
```

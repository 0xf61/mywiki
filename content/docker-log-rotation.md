---
title: "Configure Docker Log Rotation"
draft: false
date: 2025-05-16
tags:
  - docker
  - devops
  - sysadmin
---

Docker containers can generate large log files that consume significant disk space. Configure log rotation in the Docker daemon to limit log size.

Create or edit the Docker daemon configuration file `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

This configuration limits each container's log file to 10 megabytes and keeps a maximum of 3 log files per container.

Restart the Docker service to apply the changes:

```bash
sudo systemctl restart docker
```

These settings apply to new containers. Recreate existing containers for the new log rotation policies to take effect.

---
title: "Setting up Docker Log Rotation"
draft: false
date: 2025-05-15
tags:
  - docker
  - devops
  - sysadmin
---

Docker containers can generate large amounts of log data, potentially consuming all available disk space if not managed. This issue can be prevented by configuring log rotation.

To configure log rotation globally for all new Docker containers, create or modify the `/etc/docker/daemon.json` file.

Add the following configuration:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

This configuration limits each container's log file to a maximum of 10 megabytes and keeps up to 3 log files per container.

Apply the changes by restarting the Docker service:

```bash
sudo systemctl restart docker
```

Existing containers will not automatically use these new settings. You must recreate them for the new logging limits to take effect.

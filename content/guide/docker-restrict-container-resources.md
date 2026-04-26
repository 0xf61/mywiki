---
title: "Restrict Docker Container Resources"
draft: false
date: 2026-04-26
tags:
  - docker
  - devops
---

You can limit the memory and CPU a Docker container uses to prevent it from consuming all host resources.

### Restrict Memory

To limit the maximum memory a container can use, use the `-m` or `--memory` flag.

```bash
docker run -d --name my_app -m 512m my_image
```

This limits the container to 512 megabytes of RAM.

### Restrict CPU

To limit the CPU usage, use the `--cpus` flag.

```bash
docker run -d --name my_app --cpus="1.5" my_image
```

This restricts the container to use at most 1.5 CPUs.

### Apply Both Limits

You can combine these flags to restrict both memory and CPU simultaneously.

```bash
docker run -d --name my_app -m 512m --cpus="1.5" my_image
```

For docker-compose, use the `deploy.resources` section:

```yaml
services:
  my_app:
    image: my_image
    deploy:
      resources:
        limits:
          cpus: "1.5"
          memory: 512M
```

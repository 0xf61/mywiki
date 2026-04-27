---
title: "How to Clean Up Unused Docker Resources"
draft: false
date: 2026-04-27
tags:
  - docker
  - devops
  - administration
---

To free up disk space by removing unused Docker resources, you can use the `docker system prune` command.

Remove all stopped containers, unused networks, dangling images, and build cache:

```bash
docker system prune
```

To also remove all unused images (not just dangling ones) and unused volumes, add the `--all` and `--volumes` flags:

```bash
docker system prune --all --volumes
```

If you want to skip the confirmation prompt, append the `-f` or `--force` flag:

```bash
docker system prune -a --volumes -f
```

You can also target specific resources instead of the entire system:

Remove stopped containers:

```bash
docker container prune
```

Remove unused images:

```bash
docker image prune -a
```

Remove unused volumes:

```bash
docker volume prune
```

Remove unused networks:

```bash
docker network prune
```

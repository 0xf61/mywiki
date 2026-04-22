---
title: "Docker Healthcheck Configuration"
draft: false
date: 2026-04-22
tags:
  - docker
  - devops
  - reliability
---

Docker Healthchecks allow you to verify if an application running inside a container is truly ready and healthy, rather than just checking if the process is running. This is crucial for load balancing and depending services.

Add a `HEALTHCHECK` instruction to your Dockerfile:

```dockerfile
FROM nginx:alpine

# Check every 30s, timeout after 3s, allow 3 retries
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

Alternatively, configure the healthcheck in `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
```

The `start_period` gives the application time to initialize before healthchecks start failing. Dependent services can use `depends_on` with `condition: service_healthy` to wait for the application to be ready.

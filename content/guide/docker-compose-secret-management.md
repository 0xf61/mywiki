---
title: "Secure Secret Management in Docker Compose"
draft: false
date: 2026-04-25
tags:
  - docker
  - security
  - devops
---

Using environment variables for sensitive data in `docker-compose.yml` can expose credentials in system logs and process lists. Utilize Docker Compose's built-in `secrets` feature to mount sensitive data as temporary files instead.

1. **Create secret files:** Store credentials in plain text files. Ensure these files are excluded from version control (e.g., add to `.gitignore`).

```bash
echo "super_secret_db_password" > db_password.txt
echo "my_api_key_12345" > api_key.txt
```

2. **Define secrets in `docker-compose.yml`:** Reference the files in the top-level `secrets` block and attach them to services.

```yaml
version: "3.8"

services:
  app:
    image: my-app:latest
    secrets:
      - api_key
    environment:
      # Application should read from the mounted file path instead of env var
      - API_KEY_FILE=/run/secrets/api_key

  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      # Official database images often support _FILE environment variables
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    file: ./db_password.txt
  api_key:
    file: ./api_key.txt
```

Docker mounts these secrets in `/run/secrets/` inside the container as read-only tmpfs filesystems. They are never written to the container's disk and are removed when the container stops.

---
title: "Securely Managing Secrets in Docker Compose"
draft: false
date: 2026-05-04
tags:
  - docker
  - devops
  - security
---

Instead of hardcoding sensitive data or relying entirely on `.env` files that might be accidentally committed, use Docker Compose's built-in `secrets` feature to mount sensitive values directly into the container's memory via `tmpfs`.

### 1. Create Secret Files

Store your sensitive data in text files. Add these files to your `.gitignore` to prevent them from being committed to source control.

```bash
echo "super_secret_password" > db_password.txt
```

### 2. Define Secrets in `docker-compose.yml`

Define the secrets at the top level of your `docker-compose.yml` and point them to the corresponding files.

```yaml
version: "3.8"

secrets:
  db_password:
    file: ./db_password.txt
```

### 3. Grant Services Access to Secrets

In the service definition, specify which secrets the service should have access to.

```yaml
services:
  database:
    image: postgres:15
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
```

Docker mounts the secret file into the container at `/run/secrets/<secret_name>`. The containerized application can then read the secret from this path. Many official Docker images support reading environment variables directly from a file path using a `_FILE` suffix.

---
title: "Managing Secrets with Docker Compose"
draft: false
date: 2025-05-14
tags:
  - docker
  - devops
  - security
---

Docker Compose provides a built-in mechanism for securely managing sensitive information like passwords, API keys, and certificates using Docker Secrets. This avoids exposing secrets in source code or environment variables.

To use secrets in Docker Compose, define them in the `secrets` section of your `docker-compose.yml` file and reference them in the services that need access.

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: "admin"
      POSTGRES_PASSWORD_FILE: "/run/secrets/db_password"
    secrets:
      - db_password

  web:
    image: my-web-app:latest
    secrets:
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt
```

In this example:

1. Create a directory named `secrets` alongside your `docker-compose.yml` file.
2. Inside `secrets`, create two text files: `db_password.txt` and `api_key.txt`. Place the corresponding secret values in these files.
3. The `db` service reads the database password from the file mounted at `/run/secrets/db_password`.
4. The `web` service can access its API key by reading the file at `/run/secrets/api_key`.

Docker handles mounting these files securely as temporary, in-memory filesystems inside the containers. Ensure the `secrets` directory and its contents are excluded from version control (e.g., add `secrets/` to your `.gitignore` file).

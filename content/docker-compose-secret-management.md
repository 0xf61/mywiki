---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-05
tags:
  - docker
  - security
  - devops
---

Using environment variables in `docker-compose.yml` can expose sensitive data in your repository. Docker Compose secrets provide a secure way to pass sensitive information to your containers.

### 1. Create a Secret File

Create a file on your host machine to store the secret. Do not commit this file to your repository.

```bash
echo "my_super_secret_database_password" > db_password.txt
```

### 2. Define the Secret in Docker Compose

In your `docker-compose.yml`, define the secret at the top level and reference it in your service.

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      # Some images support appending _FILE to read from a secret file
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

### 3. Read the Secret inside the Container

When the container starts, the secret is mounted as an in-memory file at `/run/secrets/<secret_name>`.
Applications can read this file to obtain the secret value. If your application or image does not natively support `_FILE` environment variables, you might need to adjust its entrypoint script to read from `/run/secrets/db_password`.

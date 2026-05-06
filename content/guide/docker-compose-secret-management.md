---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-06
tags:
  - docker
  - devops
  - secrets
---

Docker Compose provides a secure way to manage sensitive data like passwords, API keys, and database credentials using the `secrets` attribute. This approach prevents hardcoding sensitive information in your `docker-compose.yml` file.

### 1. Create a Secret File

Create a file on your host machine to store the secret. For example, create a file named `db_password.txt`:

```bash
echo "my_super_secret_password" > db_password.txt
```

### 2. Define Secrets in `docker-compose.yml`

In your `docker-compose.yml`, define the secrets at the top level and reference them in the service configuration.

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

### 3. How It Works

- The `secrets` block at the bottom maps the secret name (`db_password`) to the local file (`./db_password.txt`).
- The `secrets` section inside the `db` service grants the container access to this secret.
- Docker mounts the secret inside the container at `/run/secrets/<secret_name>` (e.g., `/run/secrets/db_password`).
- The `POSTGRES_PASSWORD_FILE` environment variable tells the PostgreSQL image to read the password from that file path.

This ensures that your secrets are never exposed in environment variables or configuration files committed to version control.

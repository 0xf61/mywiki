---
title: "Secure Secret Management in Docker Compose"
draft: false
date: 2025-05-11
tags:
  - docker
  - devops
  - security
---

Managing sensitive information like database passwords, API keys, and TLS certificates directly in environment variables or code can expose them to unauthorized access. Docker Compose supports managing secrets securely, keeping them out of your `docker-compose.yml` configuration and image layers.

### 1. Create a Secret File

Create a file on your host machine to store the secret. Ensure it has strict permissions so only authorized users can read it.

```bash
echo "my_super_secret_db_password_123!" > db_password.txt
chmod 600 db_password.txt
```

### 2. Define the Secret in Docker Compose

In your `docker-compose.yml`, define the secret at the top level and reference it in the services that need access to it.

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
    volumes:
      - db_data:/var/lib/postgresql/data

secrets:
  db_password:
    file: ./db_password.txt

volumes:
  db_data:
```

### 3. How It Works

- The `secrets` section at the bottom maps the secret name `db_password` to the local file `./db_password.txt`.
- The `secrets` definition under the `db` service grants that specific container access to the secret.
- Docker injects the secret into the container at the default path `/run/secrets/db_password`. It is mounted as an in-memory `tmpfs` file, meaning it is never written to disk within the container.
- We configure the `postgres` image to read the password from this file using the `POSTGRES_PASSWORD_FILE` environment variable, a standard pattern for images supporting Docker secrets.

### 4. Running the Stack

Deploy your application as usual:

```bash
docker-compose up -d
```

Using this approach keeps your secrets secure, avoiding accidental exposure in version control or container inspection.

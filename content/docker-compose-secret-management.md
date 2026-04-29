---
title: "Docker Compose Secret Management"
draft: false
date: 2025-04-29
tags:
  - docker
  - devops
  - security
---

Managing secrets securely in Docker Compose prevents exposing sensitive data like database passwords or API keys in your `docker-compose.yml` file. Use Docker secrets to inject these values into your containers securely.

First, create a text file containing your secret value. For example, `db_password.txt`:

```bash
echo "my_super_secret_password" > db_password.txt
```

Next, define the secret in your `docker-compose.yml` file and grant the service access to it.

```yaml
version: "3.8"

services:
  database:
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

When the container runs, Docker mounts the secret file at `/run/secrets/db_password`. The application inside the container can read the secret from this path.

To start the services, run:

```bash
docker compose up -d
```

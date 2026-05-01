---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-01
tags:
  - docker
  - devops
  - self-hosting
---

Manage secrets in Docker Compose using the `secrets` attribute. This avoids hardcoding passwords or tokens in your `docker-compose.yml`.

Create a file containing your secret data. For example, create `db_password.txt` with the database password.

```bash
echo "my_secure_password" > db_password.txt
```

Reference this file in your `docker-compose.yml`.

```yaml
version: "3.8"

services:
  database:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

Docker mounts the secret file inside the container at `/run/secrets/db_password`. The service reads the password directly from this path.

---
title: "Docker Compose Secret Management"
draft: false
date: 2024-05-12
tags:
  - docker
  - devops
  - self-hosting
---

Managing secrets like database passwords or API keys in `docker-compose.yaml` using environment variables can be insecure. A better approach is to use Docker Compose's built-in secrets feature.

First, create a file containing your secret. For example, `db_password.txt`:

```bash
echo "my_super_secret_password" > db_password.txt
```

Next, reference this secret in your `docker-compose.yaml`:

```yaml
version: "3.8"

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

In this setup, the secret is mounted as a file inside the container at `/run/secrets/db_password`. The application reads the password from this file rather than an environment variable, improving security by not exposing the secret in the container's environment space.

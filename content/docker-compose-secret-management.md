---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-09
tags:
  - docker
  - devops
  - security
---

Docker Compose provides a secure way to manage sensitive data like passwords and API keys using the `secrets` attribute. This prevents hardcoding credentials in your `docker-compose.yml` file.

First, create a text file containing your secret, for example, `db_password.txt`:

```bash
echo "my_secure_password" > db_password.txt
```

Next, define the secret in your `docker-compose.yml` file and grant the service access to it:

```yaml
version: "3.8"

services:
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

When the container starts, Docker mounts the secret file into the container at `/run/secrets/db_password`. The application can then read the secret directly from this file.

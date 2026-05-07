---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-07
tags:
  - docker
  - devops
  - security
---

To use secrets in Docker Compose without exposing them in environment variables, define them in the `secrets` section of your `docker-compose.yml` file.

1. Create a file containing your secret, for example, `db_password.txt`.
2. Define the secret in `docker-compose.yml`:

```yaml
version: "3.8"
services:
  db:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

3. The secret will be mounted as a file inside the container at `/run/secrets/db_password`. Ensure your application reads the password from this file.

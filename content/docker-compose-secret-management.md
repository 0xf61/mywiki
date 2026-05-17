---
title: "Managing Secrets with Docker Compose"
draft: false
date: 2024-05-20
tags:
  - docker
  - docker-compose
  - security
---

When running services with Docker Compose, it is important to securely manage sensitive data such as database passwords, API keys, and certificates. Avoid hardcoding these secrets directly in your `docker-compose.yml` file. Instead, use Docker's built-in secrets management or environment variables.

### 1. Using Environment Files

The simplest approach is to use an `.env` file to pass secrets into your containers. Docker Compose automatically looks for an `.env` file in the same directory.

Create an `.env` file (ensure this file is added to `.gitignore` so it's not committed to your repository):

```ini
DB_PASSWORD=supersecretpassword
API_KEY=myapikey123
```

Reference these variables in your `docker-compose.yml`:

```yaml
version: "3.8"

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  app:
    image: myapp:latest
    environment:
      API_KEY: ${API_KEY}
```

### 2. Using Docker Secrets

Docker Compose also supports Docker Secrets, which provides a more robust way to handle sensitive data by passing them as files rather than environment variables.

Create a file containing your secret, for example, `db_password.txt`:

```text
supersecretpassword
```

Configure your `docker-compose.yml` to use the secret:

```yaml
version: "3.8"

services:
  db:
    image: postgres:14
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./db_password.txt
```

In this setup, the content of `db_password.txt` is mounted inside the container at `/run/secrets/db_password`. Many official Docker images, like Postgres or MySQL, support ending environment variable names with `_FILE` to automatically read the secret from the file.

---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-03
tags:
  - docker
  - devops
  - security
---

Using environment variables directly in your `docker-compose.yml` can expose sensitive data like passwords or API keys. Docker Compose provides a more secure `secrets` feature to inject sensitive information into containers at runtime.

First, create a text file containing the secret. For example, create a file named `db_password.txt` and save the password inside it:

```bash
echo "my_super_secret_password" > db_password.txt
```

Next, define the secret in your `docker-compose.yml` file under the top-level `secrets` block, and grant access to specific services using the `secrets` list.

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

In the configuration above, Docker reads the `db_password.txt` file on the host machine and mounts it inside the `database` container at `/run/secrets/db_password`. The application, configured via the `POSTGRES_PASSWORD_FILE` variable, can then securely read the secret from that file without exposing it in the environment configuration.

To run the containers, simply execute:

```bash
docker-compose up -d
```

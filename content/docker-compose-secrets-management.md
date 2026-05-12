---
title: "Docker Compose Secret Management"
draft: false
date: 2025-05-12
tags:
  - docker
  - devops
  - security
---

Storing sensitive information like database passwords or API keys directly in `docker-compose.yml` as environment variables is a security risk. Docker Compose supports managing secrets securely by injecting them into containers at runtime via files.

First, create a text file containing the secret, for example, `db_password.txt`:

```bash
echo "SuperSecretPassword123" > db_password.txt
```

Next, define the secret in your `docker-compose.yml` and grant services access to it:

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./db_password.txt
```

When the container starts, Docker automatically mounts the secret file to `/run/secrets/db_password` inside the container.

Many official images (like PostgreSQL, MySQL, and WordPress) natively support reading secrets from files by appending `_FILE` to the environment variable name. If your application does not natively support this, you must read the file directly within your code:

```python
# Example in Python
with open('/run/secrets/db_password', 'r') as file:
    db_password = file.read().strip()
```

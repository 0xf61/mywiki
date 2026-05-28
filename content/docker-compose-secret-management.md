---
title: "Docker Compose Secret Management"
draft: false
date: 2025-06-05
tags:
  - docker
  - devops
  - security
---

Manage secrets in Docker Compose using the native Docker Secrets feature to avoid exposing sensitive data in environment variables.

1. Create files for your secrets.

```bash
echo "super_secret_password" > db_password.txt
```

2. Define and attach secrets in your `docker-compose.yml` file.

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./db_password.txt
```

3. Read the secrets inside your application. Docker mounts secrets as read-only files at `/run/secrets/<secret_name>`.

```python
with open("/run/secrets/db_password", "r") as file:
    db_password = file.read().strip()
```

4. Exclude secret files from version control by adding them to `.gitignore`.

```bash
echo "db_password.txt" >> .gitignore
```

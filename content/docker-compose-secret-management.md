---
title: "Docker Compose Secret Management"
draft: false
date: 2025-05-26
tags:
  - docker
  - devops
  - security
---

To securely pass secrets to Docker containers without exposing them in environment variables or hardcoding them in `docker-compose.yml`, use Docker secrets.

**1. Create a secret file**
Create a text file containing your secret data (e.g., `db_password.txt`):

```bash
echo "SuperSecretPassword123!" > db_password.txt
```

**2. Define secrets in docker-compose.yml**
Reference the file in the top-level `secrets` block and grant access to the specific service:

```yaml
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

**3. Read the secret in your application**
Docker mounts the secret file at `/run/secrets/<secret_name>` inside the container. Modify your application or entrypoint scripts to read the sensitive value directly from this file path instead of an environment variable.

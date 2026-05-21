---
title: "Docker Compose Secret Management"
draft: false
date: 2025-06-05
tags:
  - docker
  - devops
  - security
---

Storing sensitive data directly in `docker-compose.yml` or environment files exposes your application to security risks. Instead, use Docker Compose secrets to securely provide credentials like API keys or database passwords to containers.

### 1. Create Secret Files

Create plain text files containing only your sensitive data. Keep these files out of version control by adding them to `.gitignore`.

```bash
echo "super_secret_password" > db_password.txt
echo "sk_test_123456789" > api_key.txt
```

### 2. Define Secrets in `docker-compose.yml`

Reference the secret files in a top-level `secrets` block, and map them to the corresponding services.

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

  app:
    image: myapp:latest
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./db_password.txt
  api_key:
    file: ./api_key.txt
```

### 3. Read Secrets in Your Application

Docker mounts the secrets to the `/run/secrets/` directory inside the container. Modify your application logic to read from these files instead of environment variables.

**Example (Python/Flask):**

```python
import os

def get_secret(secret_name):
    try:
        with open(f"/run/secrets/{secret_name}", "r") as f:
            return f.read().strip()
    except IOError:
        return None

db_password = get_secret("db_password")
api_key = get_secret("api_key")

if db_password:
    print("Database password loaded successfully.")
```

### Advantages

- Avoids exposing secrets in `.env` or `docker-compose.yml`.
- Works natively with Docker Swarm for production deployments.
- Prevents secrets from appearing in `docker inspect` environment outputs.

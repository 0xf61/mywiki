---
title: "Managing Secrets Securely with Docker Compose"
draft: false
date: 2025-01-01
tags:
  - docker
  - devops
  - security
---

Using Docker secrets prevents exposing sensitive data like passwords, API keys, or certificates in environment variables or configuration files.

1. Create secret files on the host machine. These files should contain only the secret value.

```bash
echo "my-super-secret-db-password" > db_password.txt
echo "my-secret-api-key" > api_key.txt
```

2. Define the secrets in your `docker-compose.yml` file.

```yaml
version: "3.8"

services:
  myapp:
    image: myapp:latest
    secrets:
      - db_password
      - api_key
    environment:
      # Pass the path to the secret file, NOT the secret value itself
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - API_KEY_FILE=/run/secrets/api_key

secrets:
  db_password:
    file: ./db_password.txt
  api_key:
    file: ./api_key.txt
```

3. Modify your application code to read the secrets from the files located at `/run/secrets/`.

```python
# Example in Python
import os

def get_secret(secret_name):
    try:
        with open(f'/run/secrets/{secret_name}', 'r') as f:
            return f.read().strip()
    except IOError:
        return None

db_password = get_secret('db_password')
api_key = get_secret('api_key')
```

4. Deploy the stack.

```bash
docker compose up -d
```

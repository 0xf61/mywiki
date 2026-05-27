---
title: "Managing Secrets in Docker Compose"
draft: false
date: 2025-05-11
tags:
  - docker
  - security
  - devops
---

Storing sensitive information like database passwords or API keys directly in `docker-compose.yml` or environment variables can expose them in version control or system logs. Docker Compose provides a dedicated `secrets` mechanism to handle this securely.

### 1. Create Secret Files

Create plain text files containing your secrets. Ensure these files are excluded from version control (e.g., via `.gitignore`).

```bash
echo "supersecretdbpassword" > db_password.txt
echo "mysecretapikey" > api_key.txt
```

### 2. Define Secrets in `docker-compose.yml`

Reference the secret files at the top level of your `docker-compose.yml` and grant services access to them.

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
    image: my-app:latest
    secrets:
      - api_key

secrets:
  db_password:
    file: ./db_password.txt
  api_key:
    file: ./api_key.txt
```

### 3. Access Secrets Inside the Container

When a service is granted access to a secret, Docker mounts the secret file into the container at `/run/secrets/<secret_name>`.

For many official images (like PostgreSQL, MySQL), you can pass the path to the secret file using environment variables ending in `_FILE`.

For custom applications, read the contents of the file at `/run/secrets/api_key` directly in your code.

**Python Example:**

```python
with open("/run/secrets/api_key", "r") as f:
    api_key = f.read().strip()
```

**Bash Example:**

```bash
API_KEY=$(cat /run/secrets/api_key)
```

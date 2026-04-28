---
title: "Docker Compose Secret Management"
draft: false
date: 2026-04-28
tags:
  - docker
  - security
  - devops
---

Use Docker secrets instead of environment variables to safely pass sensitive data (like passwords or API keys) to containers.

**1. Create secret files on the host**

Store your sensitive data in plain text files. Do not commit these files to version control.

```bash
echo "my_super_secret_db_password" > db_password.txt
echo "my_api_key_12345" > api_key.txt
```

**2. Configure `docker-compose.yml`**

Define the secrets at the top level and assign them to services.

```yaml
version: "3.9"

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
      - api_key

secrets:
  db_password:
    file: ./db_password.txt
  api_key:
    file: ./api_key.txt
```

**3. Access secrets inside the container**

Docker mounts secrets in the `/run/secrets/` directory. Read the file contents from your application code.

**Python Example:**

```python
def get_secret(secret_name):
    try:
        with open(f"/run/secrets/{secret_name}", "r") as f:
            return f.read().strip()
    except IOError:
        return None

db_password = get_secret("db_password")
api_key = get_secret("api_key")
```

**Bash Example:**

```bash
DB_PASSWORD=$(cat /run/secrets/db_password)
```

**4. Start the stack**

Run `docker-compose up -d`. The secrets will be securely mounted in memory.

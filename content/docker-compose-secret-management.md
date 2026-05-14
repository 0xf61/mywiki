---
title: "Docker Compose Secret Management"
draft: false
date: 2025-05-14
tags:
  - docker
  - security
  - devops
---

Managing secrets securely in Docker Compose helps prevent sensitive data like passwords and API keys from being exposed in plain text in your `docker-compose.yml` files or version control systems.

Docker Compose provides a built-in `secrets` feature.

**1. Create the Secret File**

Create a plain text file containing your secret data. For example, a database password:

```bash
echo "my_super_secret_password_123!" > db_password.txt
```

_Note: Ensure this file is added to your `.gitignore` to prevent committing it to your repository._

**2. Define Secrets in `docker-compose.yml`**

In your `docker-compose.yml`, define the secret at the top level and reference it within the service that needs it.

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

secrets:
  db_password:
    file: ./db_password.txt
```

**How it works:**

- The top-level `secrets` block defines `db_password` and points it to the local file `./db_password.txt`.
- The `db` service explicitly requests access to the `db_password` secret under its own `secrets` block.
- Docker mounts the secret file inside the container at `/run/secrets/db_password`.
- Many official images (like `postgres`, `mysql`, `wordpress`) support environment variables ending in `_FILE` to read the secret directly from the mounted file.

**3. Using Secrets in Custom Images**

If your application doesn't support `_FILE` environment variables automatically, you can read the file directly in your application code or entrypoint script.

Example reading the secret in a Python script:

```python
import os

def get_secret(secret_name):
    try:
        with open(f"/run/secrets/{secret_name}", "r") as f:
            return f.read().strip()
    except IOError:
        return None

db_password = get_secret('db_password')
```

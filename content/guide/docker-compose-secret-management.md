---
title: "Securely Managing Secrets in Docker Compose"
draft: false
date: 2026-05-02
tags:
  - docker
  - security
  - devops
---

Using environment variables directly in `docker-compose.yml` can expose sensitive data. Docker provides a built-in `secrets` management system that mounts secrets into containers as read-only files in memory (`/run/secrets/`).

### 1. Create Secret Files

Create a directory to store your secrets (ensure this directory is added to `.gitignore`):

```bash
mkdir .secrets
echo "super_secret_db_password" > .secrets/db_password.txt
echo "my_api_key_12345" > .secrets/api_key.txt
```

### 2. Configure docker-compose.yml

Define the secrets at the top level and reference them within your services.

```yaml
version: "3.8"

services:
  app:
    image: my-app:latest
    secrets:
      - api_key
    environment:
      # Read the secret file path in your application code
      - API_KEY_FILE=/run/secrets/api_key

  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      # Many official images support the _FILE suffix automatically
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    file: ./.secrets/db_password.txt
  api_key:
    file: ./.secrets/api_key.txt
```

### 3. Access Secrets in Application Code

Instead of reading environment variables directly, your application should read the contents of the mounted file.

Example in Python:

```python
import os

def get_secret(secret_name):
    try:
        with open(f'/run/secrets/{secret_name}', 'r') as file:
            return file.read().strip()
    except IOError:
        return None

# Usage
api_key = get_secret('api_key')
```

Example in Node.js:

```javascript
const fs = require("fs")

function getSecret(secretName) {
  try {
    return fs.readFileSync(`/run/secrets/${secretName}`, "utf8").trim()
  } catch (err) {
    return null
  }
}

// Usage
const apiKey = getSecret("api_key")
```

This approach ensures secrets are never written to disk within the container and are not exposed in container metadata or environment variable inspections.

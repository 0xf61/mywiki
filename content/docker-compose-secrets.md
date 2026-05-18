---
title: "Managing Secrets in Docker Compose"
draft: false
date: 2025-05-18
tags:
  - docker
  - devops
  - security
---

Docker secrets allow you to safely manage sensitive data like passwords and API keys without hardcoding them in your configuration files.

Here is how to use secrets in Docker Compose.

1. Create a file to store your secret data.

```bash
echo "my_super_secret_password" > db_password.txt
```

2. Update your `docker-compose.yml` to define the secret and pass it to a service.

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

3. Start your services.

```bash
docker-compose up -d
```

The service will read the password securely from the `/run/secrets/db_password` file inside the container.

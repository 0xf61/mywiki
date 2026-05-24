---
title: "Secure Secret Management in Docker Compose"
draft: false
date: 2024-07-25
tags:
  - docker
  - devops
  - security
---

Instead of hardcoding passwords or API keys in your `docker-compose.yml` file via the `environment` section, use Docker secrets or external environment variable files to prevent exposing sensitive data in version control.

Here is an example using the `secrets` attribute.

1. Create a plain text file containing your secret, such as `db_password.txt`.

```bash
echo "my_super_secret_password" > db_password.txt
```

2. Reference this file in your `docker-compose.yml`.

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./db_password.txt
```

This ensures the secret is mounted at `/run/secrets/db_password` securely within the container without being directly defined in the compose file. Ensure `db_password.txt` is added to your `.gitignore`.

---
title: "Docker Compose Secret Management"
draft: false
date: 2026-05-08
tags:
  - docker
  - devops
  - security
---

To securely manage sensitive data like passwords or API keys in Docker Compose, use the `secrets` configuration instead of environment variables.

1. Create a file containing your secret data, for example, `db_password.txt`.

   ```bash
   echo "my_super_secret_password" > db_password.txt
   ```

2. Define the secret and grant access to the service in your `docker-compose.yml`.

   ```yaml
   services:
     database:
       image: postgres:14
       environment:
         POSTGRES_PASSWORD_FILE: /run/secrets/db_password
       secrets:
         - db_password

   secrets:
     db_password:
       file: ./db_password.txt
   ```

3. Docker mounts the secret file at `/run/secrets/db_password` inside the container. The service will read the secret directly from this location.

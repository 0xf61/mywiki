---
title: "Docker Compose Secret Management"
draft: false
date: 2026-04-23
tags:
  - docker
  - security
  - devops
---

Using environment variables in `docker-compose.yml` can expose sensitive data in logs or container metadata. Docker Compose provides a built-in `secrets` mechanism to securely pass information.

1. **Create the secret files on the host system**

   Create plain text files containing the secrets. Do not commit these files to version control.

   ```bash
   echo "super_secret_password" > db_password.txt
   echo "secret_api_token" > api_token.txt
   ```

2. **Define secrets in `docker-compose.yml`**

   Add a top-level `secrets` block and map the secrets to your services.

   ```yaml
   version: "3.8"

   services:
     web:
       image: my-web-app:latest
       secrets:
         - api_token
       environment:
         - API_TOKEN_FILE=/run/secrets/api_token

     db:
       image: postgres:15
       secrets:
         - db_password
       environment:
         - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

   secrets:
     db_password:
       file: ./db_password.txt
     api_token:
       file: ./api_token.txt
   ```

3. **Access secrets inside the container**

   Secrets are mounted as read-only files at `/run/secrets/<secret_name>` by default. Applications or scripts must be configured to read the value from these files rather than reading environment variables directly. In the example above, `POSTGRES_PASSWORD_FILE` tells PostgreSQL to read the password from `/run/secrets/db_password`.

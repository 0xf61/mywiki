---
title: "Docker Compose ile Güvenli Secret Yönetimi"
draft: false
date: 2026-04-20
tags:
  - docker
  - devops
  - security
---

Çevresel değişkenler (environment variables) veya `.env` dosyaları üzerinden şifre ve API anahtarı gibi hassas verileri Docker container'larına aktarmak güvenlik riskleri barındırır. Bunun yerine Docker'ın yerleşik `secrets` özelliği kullanılmalıdır.

### 1. Secret Dosyasını Oluşturun

Hassas veriyi içeren bir dosya oluşturun:

```bash
echo "cok-gizli-veritabani-sifresi" > db_password.txt
```

### 2. docker-compose.yml Yapılandırması

Servise ve global seviyeye secret tanımını ekleyin:

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

### 3. Container İçerisinden Erişin

Secret'lar container içerisine `/run/secrets/` dizininde dosya olarak mount edilir. Uygulamanızın veya veritabanınızın (örn: Postgres için `POSTGRES_PASSWORD_FILE`) değerleri bu dosyalardan okumasını sağlayın.

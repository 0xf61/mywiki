---
title: "Docker Compose Secret Yönetimi"
draft: false
date: 2026-04-17
tags:
  - docker
  - devops
  - security
---

Çevresel değişkenler (environment variables) yerine Docker Compose sırlarını (secrets) kullanmak, hassas verileri güvende tutmanın standart bir yoludur. Parolaları ve API anahtarlarını düz metin dosyalarda tutmak yerine, dışarıdan veya dosya sistemi üzerinden Docker Compose'a güvenli bir şekilde verebilirsiniz.

Aşağıda `.env` ve düz metin şifreler kullanmak yerine Docker sırları kullanmanın basit bir örneği verilmiştir.

**1. Sır dosyalarını oluşturun:**

```bash
echo "benimgizlisifrem" > db_password.txt
echo "birkullanici" > db_user.txt
```

**2. docker-compose.yml dosyanızı güncelleyin:**

```yaml
version: "3.8"

services:
  db:
    image: postgres:14
    secrets:
      - db_password
      - db_user
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_USER_FILE: /run/secrets/db_user

secrets:
  db_password:
    file: ./db_password.txt
  db_user:
    file: ./db_user.txt
```

Konteyner ayağa kalktığında, `secrets` bloğuyla belirtilen dosyalar konteyner içindeki `/run/secrets/` dizinine salt-okunur (read-only) olarak eklenir. İlgili servis veya uygulama, şifre ve kullanıcı adını ortam değişkeni yerine bu dosyalardan okur.

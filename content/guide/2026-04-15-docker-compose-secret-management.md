---
title: "Docker Compose ile Güvenli Secret Yönetimi"
draft: false
date: 2026-04-15
tags:
  - docker
  - devops
  - security
---

Docker Compose projelerinde veritabanı şifreleri, API anahtarları gibi hassas bilgileri `.env` veya `docker-compose.yml` içinde açık metin olarak saklamak güvenlik riskidir. Docker'ın yerleşik secret yönetimini kullanarak bu bilgileri güvenli bir şekilde konteynerlere aktarabilirsiniz.

**Adım 1: Secret dosyalarını oluşturun**

Hassas bilgileri içeren metin dosyaları oluşturun. Bu dosyaları versiyon kontrol sistemine (git) eklemeyin (`.gitignore`'a eklediğinizden emin olun).

```bash
echo "c0mpl3x_p4ssw0rd!" > db_password.txt
echo "sk_test_1234567890abcdef" > api_key.txt
```

**Adım 2: docker-compose.yml dosyasını yapılandırın**

Dosyanın altına `secrets` bloğunu ekleyin ve servislerin içinde `secrets` dizisini kullanarak konteynerlere bağlayın.

```yaml
version: "3.8"

services:
  app:
    image: my-app:latest
    secrets:
      - api_key
    environment:
      - API_KEY_FILE=/run/secrets/api_key

  db:
    image: postgres:15-alpine
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

secrets:
  api_key:
    file: ./api_key.txt
  db_password:
    file: ./db_password.txt
```

**Adım 3: Uygulama içinden secret okuma**

Docker, belirtilen secret'ları konteyner içinde `/run/secrets/` dizinine birer dosya olarak bağlar. Uygulamanız bu dosyaları okuyarak değerleri almalıdır. `POSTGRES_PASSWORD_FILE` gibi ortam değişkenleri resmi imajlar tarafından genellikle desteklenir.

Kendi uygulamanızda secret okumak için (Örnek Python):

```python
import os

def get_secret(secret_name):
    try:
        with open(f"/run/secrets/{secret_name}", "r") as f:
            return f.read().strip()
    except IOError:
        return None

api_key = get_secret('api_key')
```

Bu yöntemle şifreler ortam değişkenlerinde görünmez (`docker inspect` komutunda listelenmez) ve sadece yetkili servisler dosya sisteminden secret değerlerine erişebilir.

---
title: "Docker Compose'da Güvenli Secret Yönetimi"
draft: false
date: 2026-04-16
tags:
  - docker
  - devops
  - security
---

Docker Compose dosyalarında (`docker-compose.yml`) şifre, API anahtarı veya veritabanı kimlik bilgileri gibi hassas verileri doğrudan düz metin olarak (`environment` altında) tanımlamak güvenlik riski oluşturur. Bu bilgileri ortam değişkenlerinde (environment variables) veya dış dosyalarda tutmak daha güvenlidir.

Docker Compose'un yerleşik `secrets` özelliği ile hassas verilerinizi güvenli bir şekilde yönetebilirsiniz.

### Adım 1: Secret Dosyalarını Oluşturun

Öncelikle şifrelerinizi içerecek olan düz metin dosyalarını proje dizininde oluşturun. (Bu dosyaları git'e dahil etmemek için `.gitignore` dosyasına eklemeyi unutmayın!)

```bash
echo "cokgizlisifre123" > db_password.txt
echo "benim-api-anahtarim" > api_key.txt
```

### Adım 2: docker-compose.yml Yapılandırması

Servislerinizi tanımlarken `secrets` bloğunu kullanın ve ortam değişkenleri yerine uygulamanızın bu secret'ları okuması gereken dosyaları işaret edin.

```yaml
version: "3.8"

services:
  app:
    image: my-app:latest
    secrets:
      - api_key
    environment:
      # Uygulamanız bu dosyadan API anahtarını okumalıdır.
      - API_KEY_FILE=/run/secrets/api_key

  db:
    image: postgres:15
    secrets:
      - db_password
    environment:
      # Postgres imajı otomatik olarak "_FILE" takısını destekler.
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

secrets:
  api_key:
    file: ./api_key.txt
  db_password:
    file: ./db_password.txt
```

### Çalışma Mantığı

Bu yöntemle:

1. Şifreler ve anahtarlar `docker-compose.yml` dosyasında sızdırılmaz.
2. Container başlatıldığında, `secrets` özelliği sayesinde ilgili dosyalar geçici bellek üzerinden container içerisinde `/run/secrets/` dizini altına salt okunur (read-only) olarak bağlanır.
3. Disk üzerine yazılmadığı için kalıcı bir iz bırakmaz.

Uygulamalarınızın bu yöntemi desteklemesi için şifre veya anahtarları ortam değişkenlerinden okumak yerine belirtilen dosya yolundan okuyacak şekilde yapılandırılmış (veya `_FILE` desteğine sahip) olması gerekir.

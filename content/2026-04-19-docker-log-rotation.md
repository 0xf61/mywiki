---
title: "Docker Log Rotation Yapılandırması"
draft: false
date: 2026-04-19
tags:
  - docker
  - devops
  - sysadmin
---

Docker konteynerleri varsayılan olarak logları sınırsız büyütür ve bu durum zamanla diskin dolmasına neden olur. Bunu engellemek için log rotation (log döndürme) yapılandırması gereklidir.

### 1. Global Yapılandırma (Tüm Yeni Konteynerler)

`/etc/docker/daemon.json` dosyasını düzenleyerek (yoksa oluşturarak) varsayılan log limitlerini ayarlayın:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "20m",
    "max-file": "3"
  }
}
```

Değişiklikleri uygulamak için Docker servisini yeniden başlatın:

```bash
sudo systemctl restart docker
```

_Not: Bu ayar sadece yeni oluşturulacak konteynerleri etkiler. Mevcut konteynerleri yeniden oluşturmanız gerekir._

### 2. Docker Compose Yapılandırması (Servis Bazlı)

Sadece belirli bir projede veya serviste log limitini belirlemek için `docker-compose.yml` içerisindeki `logging` parametresini kullanın:

```yaml
services:
  web:
    image: nginx:alpine
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "3"
```

Değişiklikleri uygulamak için servisi yeniden başlatın:

```bash
docker compose up -d --force-recreate
```

### 3. Diskteki Mevcut Logları Hızlıca Temizleme

Diskiniz dolduysa ve logları hemen silmek istiyorsanız aşağıdaki komut ile tüm mevcut Docker loglarını sıfırlayabilirsiniz:

```bash
sudo sh -c 'truncate -s 0 /var/lib/docker/containers/*/*-json.log'
```

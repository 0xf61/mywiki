---
title: "Docker Loglarını Otomatik Döndürme (Log Rotation) Yapılandırması"
draft: false
date: 2026-04-21
tags:
  - docker
  - devops
  - sysadmin
---

Varsayılan olarak Docker, container loglarını sınır olmadan biriktirir ve bu durum disk alanını tüketebilir. Bunu önlemek için `daemon.json` dosyası üzerinden global log rotation yapılandırması yapılmalıdır.

1. `/etc/docker/daemon.json` dosyasını oluşturun veya düzenleyin:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```

2. Docker servisini yeniden başlatarak değişiklikleri uygulayın:

```bash
sudo systemctl restart docker
```

Bu yapılandırma, her bir container için maksimum 50 MB boyutunda en fazla 3 log dosyası tutulmasını sağlar.

Mevcut çalışan container'lar global ayardan etkilenmez. Sadece yeni oluşturulanlar bu kuralları takip eder. İstenirse sadece belirli bir uygulama için `docker-compose.yml` içinde de tanımlama yapılabilir:

```yaml
services:
  app:
    image: my-app:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
```

---
title: "Docker Compose ile Linux Yeteneklerini Sınırlandırma (cap_drop)"
draft: false
date: 2026-04-18
tags:
  - docker
  - devops
  - security
---

Kapsayıcılar (containers) varsayılan olarak root kullanıcısı ile çalışsa da, tam bir sanal makine gibi davranmadıkları için Linux Kernel yeteneklerinin (capabilities) yalnızca bir alt kümesini kullanırlar. Güvenlik sıkılaştırması (hardening) yaparken `cap_drop` kullanarak kapsayıcıların sahip olduğu bu varsayılan yetenekleri asgari düzeye indirmek en iyi uygulamadır.

İşte Docker Compose ile en az ayrıcalık (least privilege) prensibini uygulamak için örnek bir kullanım:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    # 1. Adım: Tüm yetenekleri kaldır
    cap_drop:
      - ALL
    # 2. Adım: Sadece uygulamanın çalışması için gerekli olanları geri ekle
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - NET_BIND_SERVICE
    # Ekstra Güvenlik: Salt okunur dosya sistemi
    read_only: true
    # Konteynerın kendi kendini durdurmasını önlemek için tmpfs noktaları (nginx özelinde)
    tmpfs:
      - /var/cache/nginx
      - /var/run
```

**Nasıl Test Edilir?**

Kapsayıcı içindeki yetenekleri doğrulamak için `capsh` komutunu kullanabilirsiniz (içeride yüklü olması şartıyla):

```bash
docker exec -it <container_id> capsh --print
```

**Not:** Hangi yeteneklere (capabilities) ihtiyaç duyduğunuzu bulmak genellikle deneme yanılma gerektirir. `cap_drop: - ALL` uyguladıktan sonra logları takip edip uygulamanın hata verdiği yeteneği `cap_add` altına ekleyerek güvenliği en üst düzeye çıkarabilirsiniz.

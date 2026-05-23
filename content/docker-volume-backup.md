---
title: "Docker Volume Backup and Restore"
draft: false
date: 2025-05-24
tags:
  - docker
  - devops
  - administration
---

To back up a Docker volume, run a temporary container that mounts the target volume and a local backup directory. The container creates a compressed tarball of the volume's data.

### Backup

Run the following command to back up a volume named `my_volume` to the current directory:

```bash
docker run --rm \
  -v my_volume:/volume \
  -v $(pwd):/backup \
  alpine \
  tar -czvf /backup/my_volume_backup.tar.gz -C /volume .
```

### Restore

To restore the backup into a new or existing volume, run a temporary container that extracts the tarball into the volume.

1. Create a new volume (if needed):

```bash
docker volume create my_new_volume
```

2. Restore the data:

```bash
docker run --rm \
  -v my_new_volume:/volume \
  -v $(pwd):/backup \
  alpine \
  sh -c "cd /volume && tar -xzvf /backup/my_volume_backup.tar.gz"
```

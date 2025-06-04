---
title: "Migrate from Docker Desktop to Podman on Darwin"
date: 2025-06-04
draft: true
tags:
  - darwin
  - docker
---

Lately, Docker Desktop on Mac has been behaving strangely, especially with its inconsistent memory use.
Because I don't want to constantly monitor the memory usage, I looked for other options and found a few: Colima, Podman, and Orbstack.

Since I like open-source software, I naturally preferred Colima and Podman. I felt that Podman was more refined. I tried it, and it worked well. Here's how I set it up to be compatible with tools that use the docker command and socket.

## Installing Podman

```bash
brew install --cask podman-desktop
brew install podman
brew install podman-compose
```

## Shell Config

I use Fish, but this can be used with other shells.

```bash
switch (uname)
case Darwin
# backward compatibility with docker command
function docker
podman $argv
end

# make it work with tools relying on docker socket
function lazydocker
DOCKER_HOST="unix://$(podman machine inspect --format '{{.ConnectionInfo.PodmanSocket.Path}}')" "$HOME/.nix-profile/bin/lazydocker"
end
end
```

[Reference](https://karnwong.me/posts/2025/03/migrate-from-docker-desktop-to-podman-on-darwin/)

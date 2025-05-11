---
title: Systemctl in Docker
draft: false
tags:
  - docker
---

If the `--privileged` argument is not preferred, the required mounts are as follows.

The necessary mounts for running Docker with Systemd can be configured as shown below. Additionally, by setting the DISPLAY environment variable at the mount points before the image, windows in the GUI run within the container can be viewed. At the same time, the Share directory in the home directory can be accessed from inside the container.

To only display GUI applications, the container can be run as follows:

```bash
docker run -ti -v /tmp/.X11-unix:/tmp/.X11-unix -v $XAUTHORITY:/root/.Xauthority -e DISPLAY archlinux:latest bash
```

Systemd:

```bash
docker run \
      --entrypoint=/usr/lib/systemd/systemd \
      --env container=docker \
      --mount type=bind,source=/sys/fs/cgroup,target=/sys/fs/cgroup \
      --mount type=bind,source=/sys/fs/fuse,target=/sys/fs/fuse \
      --mount type=tmpfs,destination=/tmp \
      --mount type=tmpfs,destination=/run \
      --mount type=tmpfs,destination=/run/lock \
      --net=host --name=Arch --workdir /root -ti \
      --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" -v="$HOME/Share:/root/Share" \
      archlinux:latest --log-level=info --unit=sysinit.target
```

After that, close the terminal and get an interactive shell with the desired command `docker exec -ti Arch bash`. Afterwards, it will be seen that systemd interactions can be entered with `systemctl status`.

To close the container, instead of doing `docker stop Arch`, don't forget to do `shutdown now` inside the container :)

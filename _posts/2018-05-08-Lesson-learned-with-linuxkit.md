---
layout: post
title: Lesson learned using linuxkit
---

Linuxkit is a docker maintained project for building secure, lean and portable linux based subsystems for running containers. If you've ever wondered how the docker daemon can run on a none linux platform this is the project for you. Within its github organisation repositorys like [lcow](#lcow-project) and [docker-for-mac](#docker-for-mac-readme) unearth this hidden processes. However, it's main selling point is instead how seamlessly it integrates with container ecosystem originally spawned by docker.

Here's a example file which can be found at the [top layer of the linuxkit repo](#example-linuxkit-yaml). We'll go though it's components below:

```
kernel:
  image: linuxkit/kernel:4.14.22
  cmdline: "console=tty0 console=ttyS0 console=ttyAMA0"
init:
  - linuxkit/init:d899eee3560a40aa3b4bdd67b3bb82703714b2b9
  - linuxkit/runc:7c39a68490a12cde830e1922f171c451fb08e731
  - linuxkit/containerd:37e397ebfc6bd5d8e18695b121166ffd0cbfd9f0
  - linuxkit/ca-certificates:v0.2
onboot:
  - name: sysctl
    image: linuxkit/sysctl:v0.2
  - name: dhcpcd
    image: linuxkit/dhcpcd:v0.2
    command: ["/sbin/dhcpcd", "--nobackground", "-f", "/dhcpcd.conf", "-1"]
onshutdown:
  - name: shutdown
    image: busybox:latest
    command: ["/bin/echo", "so long and thanks for all the fish"]
services:
  - name: getty
    image: linuxkit/getty:v0.2
    env:
     - INSECURE=true
  - name: rngd
    image: linuxkit/rngd:v0.2
  - name: nginx
    image: nginx:1.13.8-alpine
    capabilities:
     - CAP_NET_BIND_SERVICE
     - CAP_CHOWN
     - CAP_SETUID
     - CAP_SETGID
     - CAP_DAC_OVERRIDE
    binds:
     - /etc/resolv.conf:/etc/resolv.conf
files:
  - path: etc/containerd/config.toml
    contents: |
      state = "/run/containerd"
      root = "/var/lib/containerd"
      snapshotter = "io.containerd.snapshotter.v1.overlayfs"
      differ = "io.containerd.differ.v1.base-diff"
      subreaper = false
      [grpc]
      address = "/run/containerd/containerd.sock"
      uid = 0
      gid = 0
      [debug]
      address = "/run/containerd/debug.sock"
      level = "info"
      [metrics]
      address = ":13337"
  - path: etc/linuxkit-config
    metadata: yaml
trust:
  org:
    - linuxkit
    - library
```

[lcow-project]:https://github.com/linuxkit/lcow
[docker-for-mac-readme]:https://github.com/linuxkit/linuxkit/blob/8999d8aadaaf6d59ee91f72ae58f9c2e11c31cc8/examples/docker-for-mac.md
[example-linuxkit-yaml]:https://github.com/linuxkit/linuxkit/blob/8999d8aadaaf6d59ee91f72ae58f9c2e11c31cc8/linuxkit.yml

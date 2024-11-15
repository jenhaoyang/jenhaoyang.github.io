---
layout: post
title: 讓docker pull回復上次失敗的下載
date: 2024-02-19 17:31 +0800
---

# 解決方法
https://github.com/docker/for-linux/issues/1187#issuecomment-1570501567

```bash
sudo mkdir /etc/docker
sudo gedit /etc/docker/daemon.json
```
加入這個屬性
```json
{
    "features": {"containerd-snapshotter": true}
}
```

```bash
sudo systemctl restart docker
```

# 自動重新下載docker image
```
while ! docker pull nvcr.io/nvidia/deepstream:6.4-gc-triton-devel; do sleep .1; done && echo OK
```
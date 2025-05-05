---
layout: post
title: Linux下的Docker更改image儲存位置
date: 2023-06-14 10:54 +0800
---
# 官方方法
修改/etc/docker/daemon.json加入下面設定
```json
{
  "data-root": "/home/docker-data"
}
```

重新啟動docker service 
```bash
sudo systemctl stop docker.service
sudo systemctl start docker.service
```

確認路徑已經修改
```bash
docker info | grep "Docker Root Dir"
```

https://docs.docker.com/config/daemon/#daemon-data-directory

# 步驟參考(Docker更新後原本設定的位置會被改回去，不推薦)
https://linuxconfig.org/how-to-move-docker-s-default-var-lib-docker-to-another-directory-on-ubuntu-debian-linux
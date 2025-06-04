---
title: fiftyone讀取dataset錯誤
date: 2025-06-04 16:14:40
categories:
tags:
---
* 錯誤訊息
```
{"t":{"$date":"2025-06-04T08:11:32.830Z"},"s":"I",  "c":"CONTROL",  "id":20697,   "ctx":"-","msg":"Renamed existing log file","attr":{"oldLogPath":"/home/iisi/.fiftyone/var/lib/mongo/log/mongo.log","newLogPath":"/home/iisi/.fiftyone/var/lib/mongo/log/mongo.log.2025-06-04T08-11-32"}}
Subprocess ['/home/iisi/2005013/model_training/YOLO/venv/lib/python3.10/site-packages/fiftyone/db/bin/mongod', '--dbpath', '/home/iisi/.fiftyone/var/lib/mongo', '--logpath', '/home/iisi/.fiftyone/var/lib/mongo/log/mongo.log', '--port', '0', '--nounixsocket'] exited with error 100:
```

解法

https://docs.voxel51.com/user_guide/config.html#configuring-a-mongodb-connection


用自建的mongo db

* 安裝MongoDB
https://www.mongodb.com/zh-cn/docs/manual/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition

* 啟動MongoDB deamon
```bash
sudo systemctl start mongod
```

You can achieve this by adding the following entry to your ~/.fiftyone/config.json file:
environment variable
```json
export FIFTYONE_DATABASE_URI=mongodb://[username:password@]host[:port]
```


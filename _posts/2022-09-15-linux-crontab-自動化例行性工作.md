---
layout: post
title: Linux crontab 自動化例行性工作
date: 2022-09-15 21:20 +0800
categories: [環境設定與部屬]
tags: [Linux]
---

# 查看例行性工作
```
# 查看自己的 crontab
crontab -l
```

# 建立例行性工作
```
# 編輯 crontab 內容
crontab -e
```
如果要用高權限執行例行性工作可以加上sudo
```
sudo crontab -e
```

# 工作設定的格式
每個例行工作會有時間搭配要下達的指令，`*`代表所對應的時間只要一改變就執行，例如下面範例每分鐘都會執行一次`python3 test.py`指令(因為全部是`*`號)
```
*        *           *        *        *            python3 test.py
MIN(分鐘) HOUR（小時） DOM（日） MON（月） DOW（星期幾）  指令(CMD)
```

# 指令
```
sudo crontab -e
01 14 * * * /home/joe/myscript >> /home/log/myscript.log 2>&1
```

參考:  
https://ithelp.ithome.com.tw/articles/10293218  
https://blog.gtwang.org/linux/linux-crontab-cron-job-tutorial-and-examples/  
https://askubuntu.com/a/121560  

---
layout: post
title: Linux使用SOCKS5-proxy上網
date: 2023-06-09 10:54 +0800
pin: true
---

# 簡介
這裡將介紹一個只需要使用SSH連線到一台可以連到對網網路的主機，就可以讓防火牆內的主機上網的方法
[圖片說明](https://ma.ttias.be/socks-proxy-linux-ssh-bypass-content-filters/)

# 利用ssh連接到可以上網的機器
```shell
ssh -D 4444 -q -C -N user@ma.ttias.be
```
# 檢查SOCKS是否開通了
https://superuser.com/questions/303251/how-to-check-if-a-socks5-proxy-works

以下指令可以檢查開通的port
```shell
netstat -tlnp
```
假設我們在4444port開SOCKS proxy，如果有成功開啟SOCKS proxy，應該會看到下面這行
```shell
tcp        0      0 127.0.0.1:4444          0.0.0.0:*               LISTEN  
```

# 安裝Proxychains4
https://blog.csdn.net/leishupei/article/details/120736869
sudo apt-get install proxychains4

# 設定Proxychains4
https://feifei.tw/proxychains4/  

`sudo nano /usr/local/etc/proxychains.conf`

設定檔位置


https://askubuntu.com/a/1477936

proxy_dns 的功能不要關掉

# 手動設定DNS server
export DNS_SERVER=8.8.8.8

https://github.com/rofl0r/proxychains-ng/issues/178#issuecomment-347439800

# 測試apt指令
proxychains4 sudo apt install zip

# docker pull 無法使用proxychains4的解法
`sudo nano /etc/systemd/system/docker.service.d/proxy.conf`

```
[Service]
Environment="HTTP_PROXY=socks5://127.0.0.1:9050"
Environment="HTTPS_PROXY=socks5://127.0.0.1:9050"
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

https://markvanlent.dev/2022/05/10/pulling-docker-images-via-a-socks5-proxy/



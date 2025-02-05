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
首先本機電腦和遠端電腦都必須有ssh server
首先在本機電腦ssh登入到遠端沒有外網的遠端電腦，然後在`遠端電腦`使用以下指令連回去`本機電腦`建立一條SOCKS5 proxy的通道。
```shell
ssh -D 4444 -q -C -N local_username@192.168.50.1
```
# 檢查SOCKS是否開通了
https://superuser.com/questions/303251/how-to-check-if-a-socks5-proxy-works

在`遠端電腦`用以下指令可以檢查開通的port
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

# 在還沒安裝Proxychains4機器上用SOCKS5安裝Proxychains4
```
 apt -o Acquire::http::Proxy="socks5h://127.0.0.1:4444" -o Acquire::https::Proxy="socks5h://127.0.0.1:4444" update

apt -o Acquire::http::Proxy="socks5h://127.0.0.1:4444" -o Acquire::https::Proxy="socks5h://127.0.0.1:4444" install proxychains4 
```

# 設定Proxychains4
https://feifei.tw/proxychains4/  

`sudo nano /usr/local/etc/proxychains.conf`
proxy_dns 的功能不要關掉

設定檔位置
https://askubuntu.com/a/1477936

# 手動設定DNS server
export DNS_SERVER=8.8.8.8

https://github.com/rofl0r/proxychains-ng/issues/178#issuecomment-347439800

# 測試apt指令
proxychains4 sudo apt install zip

# docker pull 無法使用proxychains4的解法
`sudo nano /etc/systemd/system/docker.service.d/proxy.conf`

```
[Service]
Environment="HTTP_PROXY=socks5://127.0.0.1:4444"
Environment="HTTPS_PROXY=socks5://127.0.0.1:4444"
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```
https://docs.docker.com/engine/daemon/proxy/  

https://markvanlent.dev/2022/05/10/pulling-docker-images-via-a-socks5-proxy/



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
sudo nano /usr/local/etc/proxychains.conf

proxy_dns 的功能不要關掉

# 手動設定DNS server
export DNS_SERVER=8.8.8.8

https://github.com/rofl0r/proxychains-ng/issues/178#issuecomment-347439800

# 測試apt指令
proxychains4 sudo apt install zip

<!-- # 用apt update測試
用下面指令測試看看是否可以更新套件
```shell
sudo apt -o Acquire::http::proxy="socks5h://127.0.0.1:4444,/" update
```

# git ssh通過SOCKS5 prosy
https://stackoverflow.com/a/67513102

設定`~/.ssh/config`並加入以下內容，下面以github為例，注意要將port改成剛剛開的proxy port。
```shell
Host github.com
HostName github.com
User git
ProxyCommand nc -v -x 127.0.0.1:4444 %h %p
```

其他設定參考:
```shell
# Method 1. git http + proxy http
git config --global http.proxy "http://127.0.0.1:1080"
git config --global https.proxy "http://127.0.0.1:1080"

# Method 2. git http + proxy shocks
git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"

# to unset
git config --global --unset http.proxy
git config --global --unset https.proxy

# Method 3. git ssh + proxy http
vim ~/.ssh/config
Host github.com
HostName github.com
User git
ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=1087

# Method 4. git ssh + proxy socks
vim ~/.ssh/config
Host github.com
HostName github.com
User git
ProxyCommand nc -v -x 127.0.0.1:1080 %h %p
```
# 圖解參考
https://erev0s.com/blog/ssh-local-remote-and-dynamic-port-forwarding-explain-it-i-am-five/ -->
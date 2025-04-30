---
title: Ubuntu加入WindowsAD
date: 2025-04-30 18:31:36
categories:
tags:
---



查詢AD server
文件SSSD
https://documentation.ubuntu.com/server/how-to/sssd/with-active-directory/#sssd-with-active-directory


echo %logonserver%
nslookup

範例:
Microsoft Windows [Version 10.0.19045.5011]
(c) Microsoft Corporation. All rights reserved.

C:\Users\2005013>echo %logonserver%
\\IEDC022

C:\Users\2005013>nslookup \\IEDC022
Server:  DNS221.iead.local
Address:  192.168.16.44

Name:    IEDC022.iead.local
Address:  192.168.16.32


C:\Users\2005013>




# 確保 DNS 配置正確
將 AD 的 DNS 伺服器地址添加到 nameserver，例如：
```
nameserver <AD_DNS_SERVER_IP>
search <AD_DOMAIN>
```

範例
nameserver 192.168.11.32
search iiiid.local




# GPT

GitHub Copilot: 在你的情境下，**使用 SSSD 整合 Ubuntu Server 與 Windows Active Directory (AD)** 是最適合的選擇。SSSD 提供了與 AD 的良好整合，並且支援 Linux 系統的身份驗證和授權，讓擁有 AD 帳號的用戶可以登入 Ubuntu Server。

---

### **為什麼選擇 SSSD？**
1. **跨平台支援**：SSSD 是 Linux 上的身份驗證解決方案，專為與 AD 整合設計。
2. **簡化管理**：SSSD 支援 Kerberos 和 LDAP，能直接與 AD 通信，並且可以緩存身份驗證數據，減少對 AD 的依賴。
3. **靈活性**：除了身份驗證，SSSD 還支援基於 AD 群組的授權控制。
4. **穩定性**：SSSD 是 Ubuntu 官方推薦的解決方案，並且有良好的社群支持。

---

### **實現步驟**
以下是將 Ubuntu Server 加入 Windows AD 並允許 AD 用戶登入的基本步驟：

#### **1. 安裝必要的套件**
在 Ubuntu Server 上安裝 SSSD 和相關工具：
```bash
sudo apt update
sudo apt install sssd-ad sssd-tools realmd adcli krb5-user packagekit
```

#### **2. 確保 DNS 配置正確**
AD 的身份驗證依賴正確的 DNS 設定，請確保 Ubuntu Server 能解析 AD 網域名稱：
```bash
sudo nano /etc/resolv.conf
```
將 AD 的 DNS 伺服器地址添加到 `nameserver`，例如：
```
nameserver <AD_DNS_SERVER_IP>
search <AD_DOMAIN>
```

#### **3. 加入 AD 網域**
使用 `realm` 工具將 Ubuntu Server 加入 AD 網域：
```bash
sudo realm join --user=<AD_ADMIN_USERNAME> <AD_DOMAIN>
```
系統會提示輸入 AD 管理員的密碼，成功後會自動配置 SSSD。

#### **4. 驗證網域加入狀態**
確認 Ubuntu Server 已成功加入 AD：
```bash
realm list
```
你應該能看到 AD 網域的相關資訊。

#### **5. 配置 SSSD**
檢查或修改 SSSD 配置檔案 `/etc/sssd/sssd.conf`，確保內容類似以下：
```ini
[sssd]
services = nss, pam
domains = <AD_DOMAIN>

[domain/<AD_DOMAIN>]
id_provider = ad
access_provider = ad
override_homedir = /home/%d/%u
default_shell = /bin/bash
```
保存後，重啟 SSSD：
```bash
sudo systemctl restart sssd
```

#### **6. 測試 AD 用戶登入**
嘗試使用 AD 用戶登入 Ubuntu Server：
```bash
ssh <AD_USERNAME>@<UBUNTU_SERVER_IP>
```
如果配置正確，AD 用戶應該能成功登入。

#### **7. 可選：限制登入權限**
如果你希望只允許特定 AD 群組的用戶登入，可以在 `/etc/sssd/sssd.conf` 中添加：
```ini
access_provider = ad
ad_access_filter = (memberOf=CN=<GROUP_NAME>,OU=<OU_NAME>,DC=<DOMAIN>,DC=<COM>)
```
重啟 SSSD 後，只有屬於該群組的用戶才能登入。

---

### **結論**
使用 SSSD 是將 Ubuntu Server 加入 Windows AD 的最佳方式，因為它提供了穩定、高效且靈活的身份驗證解決方案。完成上述配置後，AD 用戶即可使用其帳號登入 Ubuntu Server，並且可以根據需求進一步限制登入權限。

# 參考:

https://poweradm.com/join-ubuntu-debian-active-directory-domain/
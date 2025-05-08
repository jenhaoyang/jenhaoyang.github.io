---
title: Nvidia Driver安裝
date: 2025-01-15 14:27:49
categories:
tags:
---

# 使用Ubuntu driver管理器安裝(推薦)
https://ubuntu.com/server/docs/nvidia-drivers-installation

```bash
sudo apt install linux-headers-$(uname -r)
sudo ubuntu-drivers list --gpgpu
sudo ubuntu-drivers install --gpgpu nvidia:570-server
```



# 移除舊驅動
https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/#removing-nvidia-driver

```
sudo apt-get remove --purge -V "nvidia-driver*" "libxnvctrl*"
sudo apt-get autoremove --purge -V
```


手動停用nouveau driver
檢查nouveau有沒被載入
`lsmod | grep nouveau`

停用nouveau
```
cat <<EOF | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF

sudo update-initramfs -u

sudo reboot
```

https://docs.nvidia.com/ai-enterprise/deployment/vmware/latest/nouveau.html#disable-nouveau

nouveau driver會自動被安裝程式停用，校啟用可以依照以下指示重新啟用。(可能會失敗)
```
One or more modprobe configuration files to disable Nouveau have been written.  You will need to reboot your system and possibly rebuild the initramfs before these changes can take effect.  Note if you later wish to reenable Nouveau, you will    
  need to delete these files: /usr/lib/modprobe.d/nvidia-installer-disable-nouveau.conf, /etc/modprobe.d/nvidia-installer-disable-nouveau.conf
```


# 安裝驅動
https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/#frequently-asked-questions

https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Installation.html#install-nvidia-driver-535-183-06-for-data-center-gpus-and-560-35-03-for-rtx-gpus
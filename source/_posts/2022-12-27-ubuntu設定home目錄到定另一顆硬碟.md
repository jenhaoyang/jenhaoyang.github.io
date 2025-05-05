---
layout: post
title: Ubuntu設定home目錄到定另一顆硬碟
date: 2022-12-27 17:39 +0800
tags: [伺服器管理, Ubuntu]
pin: true
---


在現在常見的SSD作業系統碟加上HDD資料碟的配置，下面接介紹如何手動將/home移動到HDD資料碟

# 格式化硬碟(完全新的硬碟才需要)，這裡假設整顆硬碟不再分割磁碟
https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux  

```bash
lsblk #找出硬碟的名稱
sudo parted -a opt  /dev/sda  mkpart primary ext4 0%  100% #分割磁碟
sudo mkfs.ext4 -L datapartition /dev/sda1 #格式化磁碟並加加上label datapartition 
#You can add a partition label with the -L flag. Select a name that will help you identify this particular drive
```

# 將資料碟mount在一個暫時的資料夾下面
```
sudo mkdir /mnt/tmp
sudo mount /dev/sdb1 /mnt/tmp
```

# 複製原本/home裡面的資料
```
sudo rsync -avx /home/ /mnt/tmp
```

# 建立/home的永久mount點
* 先用以下指令查詢資料碟的UUID
```
sudo blkid
```
* 用`sudo nano /etc/fstab   # or any other editor`將下面一行寫入`/etc/fstab`文件最後面來設定mount點

```
UUID=<noted number from above>    /home    ext4    defaults   0  2
```
# 重開機檢查是否生效


# (危險區)刪除舊的/home
以下指令會刪掉舊的/home。務必先unmount新的home以免刪錯
```
sudo umount /home    # unmount the new home first!
sudo rm -rf /home/*  # deletes the old home
```

# 掛載另一顆硬碟

# 將/home掛載到另一顆硬碟

參考: 
https://askubuntu.com/a/50539  

https://www.tecmint.com/convert-home-directory-partition-linux/  

https://help.ubuntu.com/community/DiskSpace  
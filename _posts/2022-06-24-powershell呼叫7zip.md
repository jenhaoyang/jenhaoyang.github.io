---
layout: post
title: Powershell呼叫7zip
date: 2022-06-24 12:18 +0800
categories: [環境設定與部屬]
tags: [7zip]
---
如果在powershell 發現找不到7z 指令，可以用下面的script替代

```shell
$7zipPath = "$env:ProgramFiles\7-Zip\7z.exe"

if (-not (Test-Path -Path $7zipPath -PathType Leaf)) {
    throw "7 zip file '$7zipPath' not found"
}

Set-Alias 7zip $7zipPath

$Source = "c:\BackupFrom\backMeUp.txt"
$Target = "c:\BackupFolder\backup.zip"

7zip a -v50m -mx=9 $Target $Source #-v50m 代表將檔案分割成每50MB一包
```

[7-zip 指令參考](https://info.nrao.edu/computing/guide/file-access-and-archiving/7zip/7z-7za-command-line-guide)


---
參考:
https://stackoverflow.com/a/25288780
---
title: win11-local-account
date: 2026-01-08 13:40:29
categories:
tags:
---


🛡️ 方案一：最新、最快的替代指令 (推薦)
如果舊的 bypassnro 無效，請直接使用這個新的內部指令！簡單兩步完成逃脫。
 * 時機： 在 OOBE 過程的「連線到網路」畫面。
 * 指令操作： 按下 Shift + F10 叫出 CMD 視窗，輸入並按下 Enter：
   start ms-cxh:localonly
> ✨ 結果： OOBE 介面會自動重新整理，直接跳轉到本機帳號建立頁面。
> 
⛏️ 方案二：萬用「登錄檔」繞過法 (終極保險)
這是指令被移除時的「硬核」解決方案，幾乎適用於所有 Windows 11 版本。
 * 開啟登錄檔： 在「連線到網路」畫面，按下 Shift + F10 叫出 CMD 視窗，輸入 regedit 開啟。
 * 定位路徑： 導航到以下位置：
   HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\OOBE
 * 新增數值： 在右側新增一個 DWORD (32-位元) 值，命名為：BypassNRO
 * 設定數值： 將 BypassNRO 的數值資料設定為 1。
 * 重啟： 關閉 Regedit，在 CMD 輸入 shutdown /r /t 0 重新啟動電腦。
> ✨ 結果： 重啟後，你就能看到「我沒有網際網路」選項了！
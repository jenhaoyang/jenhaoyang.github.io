---
layout: post
title: 好用的git log alias
date: 2022-10-07 16:10 +0800
categories: [環境設定與部屬]
tags: [git]
---
```
git config --global alias.slog "log --graph --all --topo-order --pretty='format:%h %ai %s%d (%an)'"
```
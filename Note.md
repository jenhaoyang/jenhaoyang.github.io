# 安裝jekyll
https://jekyllrb.com/docs/installation/ubuntu/

# 安裝套件
bundle

# run server
bundle exec jekyll s --port 4001

建立新文章(已設定為使用jekyll-compose)
bundle exec jekyll help 查看有那些文章類型可以建立

建立新post
bundle exec jekyll compose "文章名稱" --collection "posts"

啟動本地端server預覽
bundle exec jekyll s

文章設定，layout已經預設為post，不需要再設定  
為了讓文章時間準確，每篇文章最好也設定timezone
例如+/-TTTT 改為  +0800
tag永遠都要用小寫

開啟Latex
https://github.com/cotes2020/jekyll-theme-chirpy/issues/55


```
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT  
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```


目前類別:
環境設定與部屬
IDE
程式撰寫
機器學習
網路運作原理
深度學習工具
Web API開發
MLOps
模型部屬
深度學習
開發工具
伺服器管理
網路除錯
攝影機設定

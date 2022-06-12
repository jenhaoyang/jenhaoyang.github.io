
run server
bundle exec jekyll s

建立新文章(已設定為使用jekyll-compose)
bundle exec jekyll help 查看有那些文章類型可以建立

建立新post
bundle exec jekyll compose "My New Post" --collection "posts"

文章設定，layout已經預設為post，不需要再設定  
為了讓文章時間準確，每篇文章最好也設定timezone
例如+/-TTTT 改為  +0800

```
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT  
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```
---
layout: post
title: fastapi使用自己的openapi檔
date: 2022-08-14 17:46 +0800
categories: [WebAPI開發]
tags: [fastapi]
---
# 步驟
* 參考官方離線版文件的[說明](https://fastapi.tiangolo.com/advanced/extending-openapi/#self-hosting-javascript-and-css-for-docs)
* 寫一個route可以回傳openapi的json
* 把openapi_url=app.openapi_url改型成 
```
openapi_url=<回傳openapi json 的route>
```
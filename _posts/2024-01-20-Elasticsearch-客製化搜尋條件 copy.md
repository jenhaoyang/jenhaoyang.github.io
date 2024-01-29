---
layout: post
title: 2024-01-20-Elasticsearch-客製化搜尋條件
date: 2024-01-20 17:31 +0800
---

# Painless Script
https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-walkthrough.html

# 只選擇夜間的資料
https://stackoverflow.com/a/35596907
```
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "OccurrenceTime": {
              "gte": "2023-12-01",
              "lt": "2024-01-20"
            }
          }
        },
        {
          "script": {
            "script": "doc.OccurrenceTime.value.hour >= 10 && doc.OccurrenceTime.value.hour <= 21"
          }
        }
      ]
    }
  }
}
```
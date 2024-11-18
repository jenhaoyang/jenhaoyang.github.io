---
layout: post
title: elasticsearch API
date: 2023-05-24 17:28 +0800
categories:
  - ELK
  - elasticsearch API
tags: [elk]     # TAG names should always be lowercase
---

# search
使用keyword使得搜尋條件必須完全符合
```
GET people-2023.04/_search
{ "query":
  { 
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gt" : "2023-04-08T10:22:28Z",
              "lt" : "2023-04-08T10:22:44Z"
            }
          }
        },
        {
          "match": {
            "StreamID.keyword": "eb2baea2-d52c-434c-af44-256c0301f4df"
          }
        }
      ]
    }
  } 
}
```

# delete
```
POST poc/_delete_by_query
{
    "query":{
        "range":{
            "@timestamp":{  
              "gt" : "2018-05-04T23:04:00Z"
              }
        }
    }
}
```
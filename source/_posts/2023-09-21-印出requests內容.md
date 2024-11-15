---
layout: post
title: 印出requests內容
date: 2023-09-21 17:31 +0800
---

```python
import http
import requests
def patch_send():
    old_send = http.client.HTTPConnection.send
    def new_send(self, data):
        print(f'{"-"*9} BEGIN REQUEST {"-"*9}')
        print(data.decode('utf-8').strip())
        print(f'{"-"*10} END REQUEST {"-"*10}')
        return old_send(self, data)
    http.client.HTTPConnection.send = new_send
patch_send()
requests.get("http://secariolabs.com")
```

https://secariolabs.com/logging-raw-http-requests-in-python/
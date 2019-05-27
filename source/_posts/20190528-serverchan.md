---
title: 从服务器推报警和日志到手机的工具
date: 2019-05-28 01:09:19
tags: [python]
categories:
top_img:
---

## 0x00

玩一下 server 酱

<!--more-->

## 0x01

```python
# coding=utf-8

import requests

import os
from dotenv import load_dotenv
import logging
load_dotenv()

sckey = os.getenv('SCKEY', None)

if not sckey:
    raise ValueError()

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)
handler = logging.StreamHandler()
handler.setLevel(logging.DEBUG)
logger.addHandler(handler)


def main(text, desp):
    """
    text：消息标题，最长为256，必填
    desp：消息内容，最长64Kb，可空，支持MarkDown
    """
    url = F'https://sc.ftqq.com/{sckey}.send'

    data = {
        'text': text,
        'desp': desp,
    }
    res = requests.post(url, data=data)
    logger.info(res.status_code)


if __name__ == "__main__":
    text = 'server chan 测试'
    desp = '今天是个好日子'
    main(text, desp)
```

- 结果

```
$ python serverchan.py
Starting new HTTPS connection (1): sc.ftqq.com:443
https://sc.ftqq.com:443 "POST /SCU15260Tdeb31f47a7c4564a1cdb288e4ebe1d9559fdf3ea065f8.send HTTP/1.1" 200 None
200
```

通过接入 [server 酱](http://sc.ftqq.com/3.version)，程序报错的时候，给微信发送错误信息，以供排查

还有其他应用场景靠自己想像
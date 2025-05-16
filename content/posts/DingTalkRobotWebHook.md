---
date: '2025-05-17T01:02:28+08:00'
draft: false
title: '钉钉机器人WebHook调用脚本'
tags: ["Robot", "DingTalk", "WebHook", "Alert"]
---

**分享一个钉钉机器人WEBHOOK的通知机器人脚本，常用于业务告警等场景**

一共需要需要传4个参数
1. 消息主题
2. 消息内容
3. 机器人`token`
4. 加签

---

- [钉钉开放平台机器人手册](https://open.dingtalk.com/document/orgapp/enterprise-internal-robots-send-markdown-messages)

---

具体内容如下：

```python
#!/usr/bin/env python3
# _*_coding:utf-8 _*_

import sys, requests, json, time, hmac, hashlib, base64, urllib.parse

PROXY_CONFIG = None
"""
PROXY_CONFIG = {
    'http': 'http://127.0.0.1:1234', 
    'https': 'http://127.0.0.1:1234' 
}
"""

subject = str(sys.argv[1])
message = str(sys.argv[2])
robot_token = str(sys.argv[3])
secret = str(sys.argv[4])

timestamp = str(round(time.time() * 1000))
secret_enc = secret.encode('utf-8')
string_to_sign = '{}\n{}'.format(timestamp, secret)
string_to_sign_enc = string_to_sign.encode('utf-8')
hmac_code = hmac.new(secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
sign = urllib.parse.quote_plus(base64.b64encode(hmac_code))

robot = 'https://oapi.dingtalk.com/robot/send?access_token=' + robot_token + '&timestamp=' + timestamp + '&sign=' + sign

data = {
    "msgtype": "markdown",
    'at': {
        'isAtAll': True
    },
    'markdown': {
        'title': 'Title',
        'text': subject + '\n' + message
    }
}
headers = {
    'Content-Type': 'application/json'
}

response = requests.post(url=robot, data=json.dumps(data), headers=headers, proxies=PROXY_CONFIG)

print(response.json())
```
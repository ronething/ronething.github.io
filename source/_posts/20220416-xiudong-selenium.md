---
title: 某动自动抢票 selenium 版本发布
date: 2022-04-16 23:27:44
tags: [python, selenium]
categories:
top_img:
---

## 0x00

时间过的真快，想不到我又回来了。

<!--more-->

## 0x01

去年有一段时间比较喜欢看演出，但是却抢不到票 让我很痛苦 机缘巧合之下，喜欢的乐队加场了，刚好又是周末，所以我用 selenium 简单实现了一版，后面幸运的成功抢到票

现项目代码做简单开源。

项目地址: https://github.com/ronething/xiudong-selenium

### xiudong-selenium

> 此版本为 selenium 模拟浏览器操作 目前暂时不考虑继续维护。

试试能不能抢到椅子乐团演出票

#### usage

chromedriver 程序需要下载一下

https://chromedriver.chromium.org/downloads

```
git clone https://github.com/ronething/xiudong-selenium.git
pip3 install -r req.txt
python3 main.py
```

提供了两个 api

- 一个是跳转到登录页面，请自行登录 `/login`
- 一个是通用购买演出票 api，请自行传入对应参数, 支持定时功能 `/buy?event=xxx&ticketId=xxx`

- 如果多次刷新 login 且登录页面没有进行登录，可能会存在线程阻塞问题，因为 max_workers 设置了 10 个, 暂时可以通过关闭窗口解决
- 如果确认订单页面显示已售罄，需要不断刷新直到出现立即支付，这一点在捡漏的时候很有用
- 只是给大家提供点思路，其他请自行阅读代码，祝大家好运

### 成功截图

昨晚抢到了。不错。

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202204162335934.png)

### acknowledgement

- https://selenium-python.readthedocs.io/getting-started.html -> 模拟浏览器操作
- https://flask.palletsprojects.com/en/2.0.x -> web 框架
- https://github.com/yangn0/Xiudong-Script -> 提供了思路

## 0x02

不知下次博客更新是几时。
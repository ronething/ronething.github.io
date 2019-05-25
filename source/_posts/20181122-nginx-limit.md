---
title: nginx 出现上传文件限制
date: 2018-11-22 01:17:13
tags: [linux,nginx]
categories:
top_img:
---

## 0x00

nginx 默认上传文件限制是 1M，如何解决？

<!--more-->

## 0x01

在 `nginx.conf` 的 `http` 块里面加上

```
client_max_body_size 8M;
client_body_buffer_size 128k;
```
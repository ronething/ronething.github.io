---
title: vscode remote ssh 终端中文显示问题
date: 2019-08-19 00:09:32
tags: [vscode,linux]
categories:
top_img:
---

## 0x00

使用了 vscode remote ssh 插件连接远程主机出现 git log 显示不了中文问题。

<!--more-->

## 0x01

经排查发现其实是安装了 vscode 中文语言插件问题，卸载之后就没有出现上述问题了。

![卸载前](https://blog-1253523830.cosgz.myqcloud.com/assets/img/62094672-7e08b980-b2b0-11e9-9950-984c4d519fae.png)

![卸载后](https://blog-1253523830.cosgz.myqcloud.com/assets/img/62094850-1d2db100-b2b1-11e9-8c03-45f16cac6db8.png)



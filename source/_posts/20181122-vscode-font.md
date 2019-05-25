---
title: mac vscode 修改字体
date: 2018-11-22 01:06:59
tags: [mac,vscode]
categories:
top_img:
---

## 0x00

因本地安装有 zsh，但是打开 Visual Studio Code 的时候发现主题的特效变成框框，原因是 Visual Studio Code 的终端字体并不是 powerline 的，怎么办？

<!--more-->

## 0x01

在 Visual Studio Code 中，点击 Code -> 首选项 ->设置，在用户设置中添加如下配置：

```json
"terminal.integrated.fontFamily": "Source Code Pro for Powerline",
```

保存配置发现 Visual Studio Code 的终端中已经出现了原来的效果。如果没有 powerline 的字体，请使用如下方法安装：

```sh
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts
./install.sh
cd ..
```

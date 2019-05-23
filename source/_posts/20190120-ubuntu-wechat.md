---
title: ubuntu 18.04 安装 deepin-wechat
date: 2019-01-20 00:23:16
tags: [wechat,linux]
categories:
top_img: https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190524002356.png
---

## 0x00

ubuntu 如何更好地使用 wechat？安装 deepin-wechat 吧

<!--more-->

## 0x01

~~[deepin-wechat](https://aur.archlinux.org/packages/deepin-wechat/)~~

~~这个`deepin.com.wechat_2.6.2.31deepin0_i386.deb`包下载下来放在另一个目录 暂且不会用到~~

直接下载我分享的文件 [`deepin-wine.tar.gz`](https://drive.google.com/file/d/1C9mbDWWkwRohWykDB8hFW3lybFejkss8/view?usp=sharing) 也行。里面包含必须的依赖 deb 包 和 上面的 wechat deb 包

### 安装 deepin-wine

- 安装部分依赖

```
sudo apt install libasound2:i386
sudo apt install liblcms2-2:i386 libldap-2.4-2:i386 libmpg123-0:i386 libopenal1:i386 
sudo apt install libstdc++6:i386 libudev1:i386 libusb-1.0-0:i386 libxext6:i386 libxml2:i386 libxcursor1:i386 libxi6:i386 libxxf86vm1:i386 libxrender1:i386 libxrandr2:i386 libxfixes3:i386
sudo apt install libxinerama1:i386 libxcomposite1:i386 libgl1-mesa-glx:i386 libglu1-mesa:i386 libosmesa6:i386 libxslt1.1:i386 libncurses5:i386 libv4l-0:i386
sudo apt install libfreetype6:i386 libcups2:i386 libfontconfig1:i386 libgsm1:i386 libtiff5:i386
sudo apt install libgcrypt11-dev:i386
```

- 安装 gcc:i386

```
sudo apt install cpp:i386
sudo apt install binutils:i386
sudo apt install gcc-7:i386
sudo apt install gcc:i386
```

- 安装依赖包

```sh
# 注意这里 deepin-wine.tar.gz 是开头提到的下载的文件
tar -zxvf deepin-wine.tar.gz
cd deepin-wine
sudo dpkg -i *.deb
```

- 📝如果出现依赖错误：

```
sudo apt install -f
```

### 安装 deepin-wechat

```sh
# deepin-wine 里面有一个 wechat 目录
cd wechat
sudo dpkg -i deepin.com.wechat_2.6.2.31deepin0_i386.deb
```

### 最终效果

![deepin-wechat](https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190524003120.png)

## 0x02

封面出处：https://www.pixiv.net/member_illust.php?mode=medium&illust_id=69973784
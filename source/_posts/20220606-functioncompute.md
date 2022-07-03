---
title: function compute
date: 2022-06-06 09:59:23
tags: [golang, function compute]
categories:
top_img:


---

## 0x00

最近在玩函数计算

<!--more-->

- 什么是函数计算

全称：Function Compute

提到函数计算一般会提到 Serverless，可以理解为 Serverless 架构下，函数计算是其中一部分。

用户只需要关注自己的业务代码怎么写即可，极大降低了运维成本 例如弹性扩容，监控等。

具体的，可以参考下云商的一些文档说明。

- 应用场景

我为什么需要用到函数计算。

说到底其实是为了解决一些潜在的问题。是的 我说的是之前的运营平台。打算升级一下 2.0

服务端工作节点支持函数计算。

还有就是 可以顺便学习函数计算，多年以前看过 不过一直没有玩过 比较懒。

端午假期第一天改代码已经写的差不多了。接下来做一些兼容性的逻辑。

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202207031838428.png)

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202207031838990.png)

替换完之后，会实际运行测试一段时间，看看效果如何。

- 样例开源仓库

目前开源了一个 [fc 样例仓库](https://github.com/cloud-org/fc-in-action-opensource)，因为目前使用的云商是 aliyun，所以主要还是跟 aliyun 相关的样例。

主要包括：

1、sdk client 请求带 HTTP 触发器的云函数

2、sdk client invokeFunction 调用函数，StopStatefulAsyncInvocationWithOptions 终止异步任务

3、sdk client 封装 body 传数据到云函数

4、使用 serverless-devs 工具进行函数管理构建部署调试

等等。

- git 仓库脱敏

因为要进行仓库开源，原先代码中写死了一些密钥信息，单纯的修改代码无法解决密钥泄漏的问题。

于是找到了 [bfg](https://github.com/rtyley/bfg-repo-cleaner) 工具进行 git 仓库脱敏。本地环境需要有 java jdk。

下载好 jar 包之后，主要的操作是创建一个文件记录上需要脱敏的条目

> replcae.txt

```
password1
key1
secret1
```

> 脱敏操作

```sh
git clone --mirror https://github.com/cloud-org/fc-in-action.git # --mirror 克隆裸仓库
java -jar bfg-1.14.0.jar --replace-text replace.txt fc-in-action.git
cd fc-in-action.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive # git gc
git push --mirror https://github.com/cloud-org/fc-in-action-opensource.git # push 到对应要开源的仓库地址即可
```

## 0x01

那么加油。
---
title: zhengzaitv-go 发布
date: 2022-06-29 09:59:23
tags: [golang]
categories:
top_img:
---

## 0x00

基于 golang 实现的正在现场请求样例

<!--more-->

## 0x01

github repo: https://github.com/ronething/zhengzaitv-go.git 防止部分爬虫乱转载

### usage

- 下载对应 [releases](https://github.com/ronething/zhengzaitv-go/releases) 即可

- 如果你需要手动编译 可以考虑执行以下命令

```shell
git clone https://github.com/ronething/zhengzaitv-go.git && cd zhengzaitv-go
go mod tidy && go build
```

![cli](https://github.com/ronething/zhengzaitv-go/raw/main/img/cli.png)

命令行工具提供几个示例

- 查询观演人列表
- 列出指定场次票种列表
- 获取地址列表

场次 id 通常可以在 url 中看到 `/ticket/detail?id=11525xxxxx2714`

修改 `cli.yaml` 中的 cookies 为对应的 cookies 值

```shell
./zhengzaitv-go tickets -a 11525xxxxx2714 --config cli.yaml
./zhengzaitv-go people --config cli.yaml
./zhengzaitv-go address --config cli.yaml
```

PS: 目前不考虑提供 `order` 命令，只是提供一点思路，仅供研究学习，祝大家好运~


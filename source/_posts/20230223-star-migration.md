---
title: GitHub Star Migration
date: 2023-02-23 20:29:22
tags: [GitHub]
categories:
top_img:
---

### 0x00

本文记录下使用 ChatGPT 完成 Github Star 迁移任务。

<!--more-->

### 0x01

笔者有两个 GitHub 账号，先前主账号 star 了很多奇奇怪怪的仓库，总数达到近 2000，所以在几个月前产生迁移主账号 star 仓库列表到另一个 GitHub 账号的想法，虽然理论上是一个不难的小工具，不过由于懒所以一直没搞，最近刚好在玩 ChatGPT，想着也许可以用 ChatGPT 来写写这个小工具。（实际时间还是花了 2 小时左右）

#### ChatGPT 提问过程

![简单阐述需求](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232031038.png)

![细节说明](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232031323.png)

![进一步说明](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232032989.png)

后续都是进行代码细节的提示说明，篇幅问题这里不再说明。

#### 迁移过程

首先需要给两个 GitHub 账号生成对应的 personal access token，权限选择 repo 或者可以进行读写 star 仓库。

![personal access token](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232034014.png)

0、导出环境变量

程序运行过程中会获取这个环境变量

```
$ export GITHUB_TOKEN=${your github token}
```

1、主账号 star 仓库列表导出列表

```
$ ./star-migration export -f ronething_stars.txt
```

```
$ wc -l ronething_stars.txt 
    1960 ronething_stars.txt
```

可以看到主账号总共 star 了 1960 个仓库

**注意运行此命令需要使用主账号生成的 GitHub Token**

2、副账号导入主账号 star 仓库列表

```
./star-migration import -f ronething_stars.txt
```

**注意运行此命令需要使用副账号生成的 GitHub Token**

3、主账号删除 star 仓库列表

```
./star-migration erase -f ronething_stars.txt
```

**注意运行此命令需要使用主账号生成的 GitHub Token**

- 效果

![貌似最多只会展示最近 star 的 300 个仓库](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232033755.jpeg)

![主账号 star 数量已经降低](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232033683.png)

#### 程序 star-migration

- 程序的完整命令如下

```bash
$ ./star-migration -h                                                                            
Usage:
  star-migration [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  erase       unstar repos from file
  export      export star repos to file
  help        Help about any command
  import      add star repos from file
  list        list current user star repos
  star        Star a GitHub repository
  unstar      UnStar a GitHub repository

Flags:
  -h, --help   help for star-migration

Use "star-migration [command] --help" for more information about a command.
```

- 最后还让 ChatGPT 帮忙写了一个程序介绍

![intro](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202302232034502.png)

PS：代码已开源 [star-migration](https://github.com/ronething/star-migration)

### 0x02

通过此案例我发现可以让 ChatGPT 写那些它能写但是你不太想写或者写起来觉得比较麻烦的内容，希望继续发掘其更有趣的功能，辅助工作辅助生活。


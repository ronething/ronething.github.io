---
title: git commit modify author name and email
date: 2019-08-31 14:15:21
tags: [git]
categories:
top_img: https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190831143010.png
---

## 0x00

要修改 `git commit` 信息，怎么办？

<!--more-->

## 0x01

由于一些原因，想要批量修改 `git commit` 的用户信息，

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/image_2019-08-31_11-37-58.png)

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/image_2019-08-31_11-38-33.png)

```sh
git clone [repo] && cd [repo]
vim commit.sh
```

复制以下内容并编辑相应信息

`OLD_EMAIL`、`CORRECT_NAME`、`CORRECT_EMAIL`

```sh
#!/bin/sh

git filter-branch --env-filter '
OLD_EMAIL="xxx"
CORRECT_NAME="xxx"
CORRECT_EMAIL="xxx"
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

```sh
chmod +x commit.sh
./commit.sh
```

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190831142442.png)

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190831142528.png)

## 0x02

封面出处：https://www.pixiv.net/member_illust.php?mode=medium&illust_id=60748017

参考资料：

- https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E5%86%99%E5%8E%86%E5%8F%B2
- https://stackoverflow.com/questions/750172/how-to-change-the-author-and-committer-name-and-e-mail-of-multiple-commits-in-git
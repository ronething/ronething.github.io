---
title: python 格式化输出
date: 2019-05-25 00:34:29
tags: [python]
categories:
top_img: https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190525010314.png
---

## 0x00

你喜欢哪一种方式进行格式化输出？

<!--more-->

## 0x01

- `%s` 

```sh
In [1]: "hello %s" % "world"
Out[1]: 'hello world'
```

参数化(不知道这样叫合不合适)

```sh
In [2]: vari = dict(name="ronething",title="python format")
In [3]: "for %(name)s %(title)s" % vari                                                                                         
Out[3]: 'for ronething python format'
```

- `format()`

```sh
In [7]: 'hello {}'.format('world')
Out[7]: 'hello world'
```

```sh
In [10]: "for {name} {title}".format(**vari)
Out[10]: 'for ronething python format'
```

索引

```sh
In [11]: "{0} {2} {1}".format("python","java","golang")
Out[11]: 'python golang java'
```

- `F''` or `f''`

```sh
In [13]: welcome = "hello world"                  
In [14]: F"for {welcome}"   
Out[14]: 'for hello world'
```

## 0x02

封面出处：https://www.pixiv.net/member_illust.php?mode=medium&illust_id=72798502
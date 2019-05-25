---
title: python 元组的使用
date: 2019-05-25 12:56:49
tags: [python]
categories:
top_img: https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190525145740.png
---

## 0x00

简单📝一下。

<!--more-->

## 0x01

- 元组的元素不可改变，列表可以。

```sh
In [1]: a = [1,2,3]                                                             

In [2]: a[0]=4                                                                  

In [3]: print(a)                                                                
[4, 2, 3]

In [8]: b = (1,2,3)                                                             

In [9]: b[0]=4                                                                  
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-9-07d5598ad652> in <module>
----> 1 b[0]=4

TypeError: 'tuple' object does not support item assignment
```

- 当想创建只有一个元素的元组时，需要在元素后加上`,`

```sh
In [11]: (1)                                                                    
Out[11]: 1 # 这里是一个 int 类型

In [12]: (1,)                                                                   
Out[12]: (1,) # 这里是一个 tuple 类型
```

- 还有一种创建元组的方式 使用 `tuple([iterable])` 传入一个迭代器 如果不传则返回一个空的元组

```sh
In [27]: tuple()                                                                
Out[27]: ()

In [28]: tuple('123')                                                           
Out[28]: ('1', '2', '3')

In [35]: tuple(['123','456','789'])                                             
Out[35]: ('123', '456', '789')
```

## 0x02

封面出处：http://simpledesktops.com/browse/desktops/2015/sep/25/siri/
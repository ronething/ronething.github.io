---
title: MySQL5.7 日期字段默认值 '0000-00-00 00:00:00' 报错
date: 2019-12-21 16:07:49
tags:
categories:
top_img:
---

## 0x00

最近遇到一个问题，MySQL5.7 日期字段默认值 '0000-00-00 00:00:00' 报错

<!--more-->

## 0x01

```sql
# 进入 mysql cli
mysql> SHOW VARIABLES LIKE 'sql_mode%'\G
*************************** 1. row ***************************
Variable_name: sql_mode
        Value: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
1 row in set (0.00 sec)
```

原因为 `sql_mode` 中含有 `NO_ZERO_IN_DATE,NO_ZERO_DATE`

解决方案：

```sh
# sudo vim /etc/my.cnf
[mysqld] # 在 [mysqld] 下面添加
# modify date 2019/12/21 13:19，将 sql mode 进行修改
# before: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
sql-mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

重启 MySQL 服务

```sql
mysql> SHOW VARIABLES LIKE 'sql_mode%'\G
*************************** 1. row ***************************
Variable_name: sql_mode
        Value: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
1 row in set (0.00 sec)
```

配置生效重新进行操作

## 0x02

参考资料：

- https://www.cnblogs.com/wang666/p/9186559.html
---
title: Dockerfile
date: 2019-06-15 00:59:31
tags: [linux,docker]
categories:
top_img: https://blog-1253523830.cosgz.myqcloud.com/assets/img/20190615011058.png
---

## 0x00

由于某些原因，需要将开发的 flask 项目容器化部署，只好手动制作镜像。

<!--more-->

## 0x01

- 开发使用 pipenv 进行 python 环境管理 需要安装的包如下

```sh
[packages]
blinker= "==1.4"
Faker= "==1.0.5"
filetype= "==1.0.5"
Flask-Cors= "==3.0.7"
Flask-Migrate= "==2.4.0"
Flask-Script= "==2.0.6"
jwcrypto= "==0.6.0"
PyJWT= "==1.7.1"
PyMySQL= "==0.9.3"
python-ldap= "==3.2.0"
request= "==2019.4.13"
shellescape= "==3.4.1"
tzlocal= "==1.5.1"
alembic = "==1.0.8"
certifi = "==2019.3.9"
chardet = "==3.0.4"
idna = "==2.8"
requests = "==2.21.0"
shortuuid = "==0.5.0"
naked = "==0.1.31"
python-dotenv = "==0.10.3"
uwsgi = "==2.0.18"
supervisor = "==4.0.2"
redis-py-cluster = "==1.3.6"
psutil = "==5.6.3"
```

- 项目还依赖于 crontab 所以容器中需要跑着 crond 服务
- 项目需要访问的数据库需要配置 dns 服务器

## 0x02

docker 安装配置略过

- 登陆内部 registry

`docker login xxx` 输入用户名还有密码按回车

- 设置宿主机 dns `/etc/resolv.conf`

```sh
nameserver 172.18.xx.xx
```

设置 dns 之后创建的容器会自动继承宿主机的 dns 否则容器的 dns 会采用 8.8.8.8 and 8.8.4.4

- Dockerfile 编写

```dockerfile
FROM python:3.7.3-alpine3.8

# Application
RUN mkdir /app
WORKDIR /app
COPY Pipfile Pipfile
COPY Pipfile.lock Pipfile.lock
COPY pip.conf /etc/pip.conf

RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.8/main/" > /etc/apk/repositories \
	&& apk add --update --upgrade \
	&& apk add tzdata libldap gcc libc-dev openldap-dev libffi-dev libuuid pcre mailcap linux-headers pcre-dev nginx\
	&& python3 -m pip install --no-cache-dir pipenv \
	&& pipenv install --system \
	&& cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	&& apk del gcc libc-dev linux-headers libc-dev openldap-dev tzdata \
	&& rm -rf /tmp/* \
	&& rm -rf /var/cache/apk/*

CMD ["/bin/sh"]
```

其中 tzdata 是为了解决 timezone 时区问题

libldap、openldap-dev 等是为了解决 pipfile 中 python-ldap 模块安装问题

libffi-dev 是为了解决 cffi 模块安装问题

nginx 不用说

其他的则是为了解决 uwsgi 安装问题

虽然看着简单，但是这些资料其实并不好找，不是为了说明什么，只希望以后可以少做一些重复无意义的事情吧。

为了使打包的镜像大小尽量少，通过 apk del 还有 rm -rf /tmp/* 等删除一些缓存，不过现在 pipenv 还有 pip 安装的软件源的缓存还没有删除，后续应该进行删除。

- build && push

```sh
docker build -t ronething:v8 .
docker tag ronething:v8 xxxx:v0.0.3
docker push xxx:v0.0.3
```

- 服务器部署容器服务

```sh
docker pull xxx:v0.0.3
```

先拉取镜像 然后 run 此处需要结合 flask repo gitlab 仓库

```sh
#!/bin/bash
docker stop xxx
docker rm -f xxx
docker run -it -p 80:80 -p 8080:8080 -v `pwd`:/app -v `pwd`/conf.d:/etc/nginx/conf.d -v /usr/share/nginx/html/xxx:/usr/share/nginx/html/xxx -v /var/log/nginx:/var/log/nginx -d --name xxx xxx:v0.0.3 /bin/sh start_server.sh
```

其中 start_server.sh 作为启动脚本

```sh
supervisord -c supervisord.conf
# mkdir /run/nginx 是因爲如果不創建 nginx 目錄會報錯導致 nginx 啓動不了
mkdir /run/nginx
nginx -g "daemon on;"
# 開啓 crond 定時任務服務
# download alarm email
rm -rf /tmp/cron.`whoami`
echo "* * * * * cd /app/app/tasks && python donwload_mail.py prod" >> /tmp/cron.`whoami`

# Sender alarm
echo "* * * * * cd /app/app/tasks && python sender_alarms.py" >> /tmp/cron.`whoami`

crontab -u `whoami` /tmp/cron.`whoami`
crond

# sh 放在最後 阻止進程掛掉
/bin/sh
```

此处使用了 crontab 定时任务，有两种方案，一种在宿主机中写入定时任务 另一种直接在容器中启用 crond 服务并写入定时任务，选用了第二种，因为考虑到以后要用 flask restful api 操作 crontab

## 0x03

封面出处：https://www.pixiv.net/member_illust.php?mode=medium&illust_id=64517908

参考资料：

- https://github.com/blockstack/blockstack.js/issues/95
- https://github.com/TimeBye/oneindex/blob/master/docker-entrypoint.sh
- https://note.qidong.name/2017/06/28/python-ldap-in-docker/
- https://note.qidong.name/2017/06/28/uwsgi-in-docker/
- https://www.jianshu.com/p/cd1636c94f9f 
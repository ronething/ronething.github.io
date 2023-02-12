---
title: Build Apache APISIX From Source On M2 Pro
date: 2023-02-12 15:50:19
tags: [Apache APISIX, M2 Pro]
categories: 
top_img:
---

## 0x00

介绍下如何在 Apple M2 Pro 上从零开始源码构建 Apache APISIX

<!--more-->

### 0x01 介绍

[Apache APISIX](https://github.com/apache/apisix) 是一个动态、实时、高性能的开源 API 网关，内置各种插件，提供负载均衡、动态上游、灰度发布、服务熔断、身份认证、可观测性等丰富的流量管理功能。

### 0x02 源码构建

以下操作基于 APISIX master branch 进行操作

```shell
# 克隆代码仓库
git clone https://github.com/apache/apisix.git && cd apisix/
```

```shell
# 安装系统相应依赖
$ ./utils/install-dependencies.sh
```

```shell
# 安装 lua 第三方模块
$ make deps
```

```shell
# 安装并启动 etcd
wget https://github.com/etcd-io/etcd/releases/download/v3.5.7/etcd-v3.5.7-darwin-arm64.zip
unzip etcd-v3.5.7-darwin-arm64.zip && cd etcd-v3.5.7-darwin-arm64
# 启动 etcd
./etcd
```

通常走到这一步，Apache APISIX 就可以进行启动

```shell
$ ./bin/apisix start 
/opt/homebrew/Cellar/openresty/1.21.4.2_1//luajit/bin/luajit ./apisix/cli/apisix.lua start
```

然而实际过程中，可能会遇到各种各样的问题，我会将我遇到的问题以及解决方案放到下面说明

1、`make deps` 可能会遇到 prce 相关的问题，详见 issue#4945: [request help: How to install Apache APISIX 2.9 on Mac M1](https://github.com/apache/apisix/issues/4945) ，解决方案对应 [comment-909850338](https://github.com/apache/apisix/issues/4945#issuecomment-909850338)

```shell
# install pcre
brew install pcre
```

```shell
# vim ~/.luarocks/config-5.1.lua
lua_interpreter = "luajit"
variables = {
   OPENSSL_INCDIR = "/opt/homebrew/opt/openresty-openssl111/include",
   OPENSSL_LIBDIR = "/opt/homebrew/opt/openresty-openssl111/lib",
   PCRE_DIR = "/opt/homebrew/Cellar/pcre/8.45",
   PCRE_INCDIR = "/opt/homebrew/Cellar/pcre/8.45/include"
}
```

2、启动报错问题 NYI: cannot call this C function (yet)

错误日志类似如下：

```
...Cellar/openresty/1.21.4.1_1/lualib/resty/core/shdict.lua:320: NYI: cannot call this C function (yet)
stack traceback:
[C]: in function 'ngx_lua_ffi_shdict_get'
...Cellar/openresty/1.21.4.1_1/lualib/resty/core/shdict.lua:320: in function 'get'
...space/apisix//deps/share/lua/5.1/resty/worker/events.lua:119: in function 'get_event_id'
...space/apisix//deps/share/lua/5.1/resty/worker/events.lua:610: in function 'configure'
...rs/louis/Documents/idea_workspace/apisix/apisix/init.lua:112: in function 'http_init_worker'
init_worker_by_lua:2: in main chunk
```

详见 issue#7313: [help request: Mac M1 make run error](https://github.com/apache/apisix/issues/7313)

解决方案对应 [comment-1327511752](https://github.com/apache/apisix/issues/7313#issuecomment-1327511752)

不过此评论说的比较简洁，具体的意思就是需要 git apply 两个 patch，然后通过 brew 重新进行 openresty 的安装

```shell
# 1、下载 openresty 对应版本包
wget https://openresty.org/download/openresty-1.21.4.1.tar.gz
tar -zxvf openresty-1.21.4.1.tar.gz && cd openresty-1.21.4.1/
# 2.1、git patch 第一个
wget https://github.com/openresty/lua-resty-core/commit/da705fcaaf85f351fa87480816b4f41a65fc4bd5.patch
cd bundle/lua-resty-core-0.1.23 # 对应 patch 目录
git apply da705fcaaf85f351fa87480816b4f41a65fc4bd5.patch
# 2.2、git patch 第二个
wget https://github.com/openresty/lua-nginx-module/commit/653d6a36f46b077cb902d7ba40824c299cf9bbf4.patch
cd bundle/ngx_lua-0.10.21 # 对应 patch 目录
git apply 653d6a36f46b077cb902d7ba40824c299cf9bbf4.patch
# 3.1、重新打包 回到上级目录
tar -zcvf openresty-1.21.4.2.tar.gz openresty-1.21.4.1
# 3.2、下载对应 homebrew openresty.rb 文件
wget https://raw.githubusercontent.com/openresty/homebrew-brew/99ef8e51aab7a5f9095f28086a70f258575e7e1e/Formula/openresty.rb
# vim openresty.rb 修改对应如下
     6	  VERSION = "1.21.4.2".freeze
     7	  revision 1
     8	  url "http://127.0.0.1:8081/openresty-#{VERSION}.tar.gz"
     9	  sha256 "1078923e80f2da30d1e9c3921222505cba37614e97894052e7b98562d82c6919"
# 可以通过 shasum -a 256 openresty-1.21.4.2.tar.gz 计算对应 sha 值
# 3.3、启动文件服务器提供 openresty 代码包 其实就是对应上面的 127.0.0.1:8081
python3 -m http.server 8081
# 3.4、homebrew 本地重新安装 openresty
brew install ./openresty.rb
```

参考文档：[Building APISIX from source](https://apisix.apache.org/docs/apisix/next/building-apisix/)

注：如果你想直接部署 Apache APISIX，可以考虑使用 [apisix-docker](https://github.com/apache/apisix-docker) 或者 [apisix-helm-chart](https://github.com/apache/apisix-helm-chart) 进行部署测试

### 0x03 测试

配置一个 Apache APISIX 中限速的示例

```shell
# 使用 httpbin 作为上游服务
docker run -d -p 8080:80 --rm kennethreitz/httpbin
```

```shell
curl -i http://127.0.0.1:9180/apisix/admin/routes/1 \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/get",
    "plugins": {
        "limit-count": {
            "count": 2,
            "time_window": 60,
            "rejected_code": 503,
            "key_type": "var",
            "key": "remote_addr"
        }
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:8080": 1
        }
    }
}'

HTTP/1.1 200 OK
Date: Sun, 12 Feb 2023 05:01:35 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Server: APISIX/3.1.0
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: *
Access-Control-Max-Age: 3600
X-API-VERSION: v3

{"key":"/apisix/routes/1","value":{"uri":"/get","priority":0,"upstream":{"type":"roundrobin","pass_host":"pass","hash_on":"vars","scheme":"http","nodes":{"127.0.0.1:8080":1}},"id":"1","create_time":1676177913,"status":1,"plugins":{"limit-count":{"key_type":"var","allow_degradation":false,"key":"remote_addr","time_window":60,"show_limit_quota_header":true,"policy":"local","count":2,"rejected_code":503}},"update_time":1676178095}}
```

```
# [13:02:06] 
$ curl -i http://127.0.0.1:9080/get                    
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 221
Connection: keep-alive
X-RateLimit-Limit: 2
X-RateLimit-Remaining: 1
X-RateLimit-Reset: 60
Date: Sun, 12 Feb 2023 05:02:07 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Server: APISIX/3.1.0

{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "127.0.0.1:9080", 
    "User-Agent": "curl/7.86.0", 
    "X-Forwarded-Host": "127.0.0.1"
  }, 
  "origin": "127.0.0.1", 
  "url": "http://127.0.0.1/get"
}

# [13:02:07] 
$ curl -i http://127.0.0.1:9080/get
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 221
Connection: keep-alive
X-RateLimit-Limit: 2
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 59
Date: Sun, 12 Feb 2023 05:02:08 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Server: APISIX/3.1.0

{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "127.0.0.1:9080", 
    "User-Agent": "curl/7.86.0", 
    "X-Forwarded-Host": "127.0.0.1"
  }, 
  "origin": "127.0.0.1", 
  "url": "http://127.0.0.1/get"
}

# [13:02:08] 
$ curl -i http://127.0.0.1:9080/get
HTTP/1.1 503 Service Temporarily Unavailable
Date: Sun, 12 Feb 2023 05:02:08 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 194
Connection: keep-alive
X-RateLimit-Limit: 2
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 0
Server: APISIX/3.1.0

<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>openresty</center>
</body>
</html>
```

该路由配置表示，在固定时间窗口，time_window 为 60s 的情况下，如果同一个 remote_addr 访问 /get 超过 2 次则会返回 503 HTTP 错误码

### 0x0F 总结

本文主要介绍了如何在 Apple M2 Pro 上进行  Apache APISIX 的源码构建，当然这同样适用于 M1 芯片，并通过一个例子展示如何在 Apache APISIX 中做限速操作，可以检验我们的源码构建是否成功，同时后续如果给 Apache APISIX 做 PR 贡献，也可以很方便的基于此进行修改。

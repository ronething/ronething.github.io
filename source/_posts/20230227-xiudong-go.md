---
title: Xiudong-Go Release
date: 2023-02-27 18:22:20
tags: [golang]
categories:
top_img:
---

### 0x00

Web 版秀动请求样例学习

<!--more-->

### 0x01 

简单开源下，仅供学习交流

github repo: https://github.com/ronething/xiudong-go

#### interface 

```go
// API showstart server api
type API interface {
	GetAddressList(pageNo int64) ([]Address, error)
	GetAddress() (*Address, error)
	GetCpList(pageNo int64) ([]CpItem, error)
	GetTicketList(activityId string) ([]TicketListResult, error)
}
```

#### usage

- 下载对应 [releases](https://github.com/ronething/xiudong-go/releases) 即可

- 如果你需要手动编译 可以考虑执行以下命令

```shell
git clone https://github.com/ronething/xiudong-go.git && cd xiudong-go/cli
make build
```

具体使用可参考 [./cli](https://github.com/ronething/xiudong-go/tree/main/cli) 和 [./docs](https://github.com/ronething/xiudong-go/tree/main/docs)

notice: 此程序更多是学习用途，可以简单分析下自己写 order 命令，祝大家好运~

```
$ ./showstart         
showstart cli sample
Usage:
  showstart [command]
Available Commands:
  address     查询个人地址
  help        Help about any command
  idCard      查询已绑定观演人 id
  tickets     列出指定场次 ticketId 列表
  version     check version
Flags:
      --config string   config file (default is $HOME/.showstart.yaml)
  -h, --help            help for showstart
Use "showstart [command] --help" for more information about a command.
```

#### 免责声明

- 见 [Disclaimer](https://github.com/ronething/xiudong-go/blob/main/Disclaimer.md)

#### acknowledgement

- wap.showstart.com

### 0x02

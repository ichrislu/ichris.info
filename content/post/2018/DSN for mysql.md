---
title: "关于mysql驱动的DSN字符问题"
date: "2018-12-01"
lastMod: "2018-12-01"
categories: ["it"]
tags: ["golang", "mysql", "DSN"]
---

此次重构发现之前的loc参数一直为空，不太记得是当时刻意为之，还是当时发生了错误未解决
所以在生产上就加了配置asia/shanghai，报错了：invalid DSN: did you forget to escape a param value?
这个好办，转换一下就行了，改成：asia%2Fshanghai

启动继续测试，又报错了：open C:\Go/lib/time/zoneinfo.zip: no such file or directory
吓我一跳，不是交叉编译吗？？！！服务器上是Linux环境，怎么还出了个C:\Go？怎么跟开发环境一样呢？执行go env，确定编译环境没有问题：
```
go env
set GOARCH=amd64
set GOBIN=
set GOCACHE=C:\Users\chris\AppData\Local\go-build
set GOEXE=
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOOS=linux
set GOPATH=C:\Users\chris\go
set GOPROXY=
set GORACE=
set GOROOT=C:\Go
set GOTMPDIR=
set GOTOOLDIR=C:\Go\pkg\tool\windows_amd64
set GCCGO=gccgo
set CC=gcc
set CXX=g++
set CGO_ENABLED=0
set GOMOD=
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-fPIC -m64 -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\chris\AppData\Local\Temp\go-build799983026=/tmp/go-build -gno-record-gcc-switches
```

后面发现其实是大小写问题，因为按提示，打开对应路径，Asia和Shanghai首字母都是大写。
修改后ok！

问题虽然临时解决，但仍然没有找到原因。

官方文档：
- <https://github.com/go-sql-driver/mysql>
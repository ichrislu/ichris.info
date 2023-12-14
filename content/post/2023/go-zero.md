---
title: "go-zero"
date: "2023-12-05"
lastMod: "2023-12-05"
categories: ["it"]
tags: ["golang", "go-zero"]
---

## 安装
参考官方文档1

### 安装golang
*略*

### 配置GO111MODULE和GOPROXY
```bash
# 设置
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct

# 验证
go env GO111MODULE
go env GOPROXY
```

### 安装go-ctl
goctl是go-zero的内置脚手架，提升开发效率的利器，可以一键生成代码、文档、部署k8s yaml、dockerfile等。
```bash
# 安装
go install github.com/zeromicro/go-zero/tools/goctl@latest

# 验证
goctl -v
```

goctl报错：command not found: goctl，则需要将$GOPATH加到$PATH中，以zsh为例
```bash
vim ~/.zshrc

# 添加如下，重启终端/iTerm2生效
export PATH=$HOME/go/bin:$PATH
```

### 安装protoc
protoc是一个用于生成代码的工具，可以根据proto文件生成C++、Java、Python、Go、PHP等多种语言代码。  
gRPC的代码生成还依赖`protoc-gen-go`、`protoc-gen-go-grpc`插件配合生成Go语言的gRPC代码。

#### 一键安装
通过goctl可以一键安装`protoc`、`protoc-gen-go`、`protoc-gen-go-grpc`相关组件。

```bash
goctl env check --install --verbose --force
```

#### 手动安装
1. 在 [Github](https://github.com/protocolbuffers/protobuf/releases) 下载
2. 解压缩下载的文件，并移到`$GOPATH/bin`目录

查看`$GOPATH`目录：`go env GOPATH`

### 验证
`goctl env check --verbose`
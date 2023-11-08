---
title: "golang环境配置"
date: "2018-12-30"
lastMod: "2019-07-09"
categories: ["it"]
tags: ["golang"]
---

环境配置

- GOROOT：go 的安装目录，设置这个环境变量自定义 go 路径
- GOPATH：go 的工作目录（项目目录），编译或运行时从这个环境变量中去查找包、依赖

自定义GOPATH，以zsh为例

```shell
# vim ~/.zshrc，添加下行
export GOPATH=/Users/chris/Documents/go

# 立即生效
source ~/.zshrc
```

允许多个目录，当有多个目录时，请注意分隔符，Linux冒号，Windows分号; 当有多个`GOPATH时`默认将`go get`获取的包存放在第一个目录下 

在GOPATH目录，会有3个文件夹：

- bin：编译后生成的可执行文件
- pkg：编译时生成的中间文件
- src：存放源码，按照golang默认约定，go run，go install等命令的当前工作路径

---
title: "go 1.13开发环境搭建"
date: "2019-11-27"
lastMod: "2020-04-15"
categories: ["it"]
tags: ["golang", "GoLang", "GOPROXY", "nexus", "GoLand"]
---


进入项目目录：go mod init <project name>

设置环境变量：GOPROXY="https://goproxy.io,direct"

运行go get的时候，发现报410 gone的错误，目前网上查到的资料都是扯淡，查官方文档，在1.13版本加了版本验证，官方说如果因为防火墙或代理问题验证不了，可以添加环境变量：GOSUMDB=off关闭验证。记得：命令行窗口需要退出重新打开才能生效！！！

GoLand常用设置

- Go -> GoROOT：选择Go版本
- Go -> GoPATH：添加Global GoPATH，一般在～/go，里面包含bin、pkg、src
- Go -> Go Modules(vgo)：enable，并设置Proxy
- Tool -> File Watchers：添加go fmt，可以设置为Global
- Editor -> Inspections：找到Proofreading，取消Typo的勾选
- Keymap，Main Menu -> Code -> Completion：Basic(Option + /)，Smart Type(Option + Shift + /)，取消Cyclic Expand Word和Cyclic Expand Word (Backward)
- MacOS启动调试时报错：go build -i results in "/usr/local/go/pkg/darwin_amd64/runtime/cgo.a: permission denied"，配置Run/Debug Configurations，删除Go tool arguments的-i参数，参考：https://github.com/golang/go/issues/37962



参考：
- <https://goproxy.io/>
- <https://tip.golang.org/doc/go1.13#version-validation>
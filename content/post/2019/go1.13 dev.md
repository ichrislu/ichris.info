---
title: "go 1.13开发环境搭建"
date: "2019-11-27 10:50:03"
lastMod: "2019-11-27 10:50:03"
tags: ["go", "GoLang", "GOPROXY", "nexus", "Athens"]
---


进入项目目录：go mod init <project name>

设置环境变量：GOPROXY="https://goproxy.io,direct"

运行go get的时候，发现报410 gone的错误，目前网上查到的资料都是扯淡，查官方文档，在1.13版本加了版本验证，官方说如果因为防火墙或代理问题验证不了，可以添加环境变量：GOSUMDB=off关闭验证。记得：命令行窗口需要退出重新打开才能生效！！！



参考：

 https://goproxy.io/ 

 https://tip.golang.org/doc/go1.13#version-validation 
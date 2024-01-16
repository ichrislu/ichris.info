---
title: "Golang私有仓库依赖解决方案"
date: "2024-01-12"
lastMod: "2024-01-12"
categories: ["it"]
tags: ["golang", "go mod", "git"]
---

# 缘由：
以往Java日常开发中，模块依赖问题有成熟解决方案：
1. 开发阶段，被依赖模块做为单独项目开发完成，导出jar包就可以被其他项目依赖；或者上传到Sonatype Nexus Repository共享，其他人/项目通过Maven解决依赖问题。
2. 运行阶段，因为Java编译成中间代码，被依赖的包放在classpath中在需要时被加载运行。

与Golang开发的主要区别是Golang直接编译成二进制文件，而且是一个独立完整的可运行文件，其中包含了完整的依赖。
在开发阶段，Golang的依赖通常都是通过类似github之类仓库解决，这种方式有一个问题：企业开发过程中，可能会有一些被依赖的模块并不一定希望仓库是公开的，更适合的做法是保存在企业内部的Git服务器上；但是在开发阶段的依赖关系，在`import`和`go mod`中写企业内部仓库地址显然行不通。

当然也有对应解决方案：
1. 在`go mod`文件中用`replace`，但感觉不太优雅，没有实际尝试；
2. 了解过`go work`，这个方案更适用于个人本地开发，服务器上CICD不能与开发环境做到统一；
3. 网上也有很多示例，都是基于大仓模式，企业开发不适用。

# 原理：
利用`go get`技术原理，“欺骗”go获取正确的私有Git仓库上的依赖源码和认证问题。

2021年的时候，找了一个偏门的解决方案，当时只是非常简单的记录了下来；这次又遇到同样问题，找了一些资料，尝试了一些方法，优化如下：
1. 之前用固定的文件给`go get`响应消息，这次因为采用了BFF的微服务架构，可能会有更多的模块需要被依赖，所以参考了又拍云的方案，专门写了一个服务，不再为每个被依赖模块添加一个`go get`响应文件了
2. 响应消息中`go-import`消息由https改为ssh，这个改变的起原是因为做`go get`测试时，始终报错：
	```bash
	go: module git.<domain>.com/<module>: git ls-remote -q origin in /Users/<user>/go/pkg/mod/cache/vcs/a5b2*********************9375: exit status 128:
		致命错误：could not read Username for 'https://192.168.100.8': terminal prompts disabled
	```
	改为ssh后报错：
	```bash
	go: downloading git.<domain>.com/<module> v0.0.0-20240112055938-c06dc8ee94d9
	go: git.<domain>.com/<module>@upgrade (v0.0.0-20240112055938-c06dc8ee94d9) requires git.<domain>.com/<module>@v0.0.0-20240112055938-c06dc8ee94d9: parsing go.mod:
		module declares its path as: <module>
				but was required as: git.<domain>.com/<module>
	```
	这个错误原因很简单，最开始创建依赖模块的时候，模块名没有写完整路径，改一下就行了
3. 使用单独的域名，独立于企业官网，互不干扰

# 实践：
## 运行构建容器
```dockerfile
FROM bitnami/golang:1.21.6
ENV LANG zh_CN.UTF-8
ENV LC_ALL C.UTF-8
COPY localtime /etc/localtime
RUN rm -fr /go/*

# 配置环境
RUN go env -w GOPROXY="https://goproxy.cn,direct" GONOPROXY="git.<domain>.com/*" GONOSUMDB="git.<domain>.com/*" GOPRIVATE="git.<domain>.com/*"
RUN git config --global url."git@192.168.100.8:".insteadOf "https://192.168.100.8/"

# 安装openssh-client
RUN apt-get update && apt-get install -y openssh-client

# ssh验证依赖
COPY id_rsa /root/.ssh/id_rsa
RUN chmod 600 /root/.ssh/id_rsa

# 因为是非交互方式，需要禁用主机公钥检查
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config
```

## 开发环境
go env -w
```bash
GONOPROXY='git.<domain>.com/*'
GONOSUMDB='git.<domain>.com/*'
GOPRIVATE='git.<domain>.com/*'
GOPROXY='https://goproxy.cn,direct'
```

## 源码
go get代理服务源码：
逻辑简单，一个源码文件就行了：
```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
	"net/http"
	"os"
)

var temptUrl, gitBaseUrl, project string

func init() {
	var ok bool

	gitBaseUrl, ok = os.LookupEnv("GIT_BASE_URL")
	if !ok {
		log.Fatalln("environment variable \"GIT_BASE_URL\" is not set")
	}
}

func main() {
	r := gin.Default()
	r.LoadHTMLFiles("template.tmpl")

	r.GET("/:project", proxyHandler)

	_ = r.Run(":8080")
}

func proxyHandler(ctx *gin.Context) {
	project = ctx.Param("project")
	temptUrl = ctx.Request.Host

	ctx.HTML(http.StatusOK, "template.tmpl", gin.H{
		"tempt_url":    temptUrl,
		"git_base_url": gitBaseUrl,
		"project":      project,
	})
}
```

模板：
```html
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <meta name="go-import" content="{{ .tempt_url }}/{{ .project }} git {{ .git_base_url }}/{{ .project }}">
    <meta name="go-source" content="{{ .tempt_url }}/{{ .project }} {{ .git_base_url }}/{{ .project }} {{ .tempt_url }}/{{ .project }}/-/tree/master{/dir} {{ .tempt_url }}/{{ .project }}/-/blob/master{/dir}/{file}#L{line}">
    <title>Go Repo Proxy</title>
  </head>
  <body>
    <pre>
     ***************************************************************
     *                          _ooOoo_                            *
     *                         o8888888o                           *
     *                         88" . "88                           *
     *                         (| ^_^ |)                           *
     *                         O\  =  /O                           *
     *                      ____/`---'\____                        *
     *                    .'  \\|     |//  `.                      *
     *                   /  \\|||  :  |||//  \                     *
     *                  /  _||||| -:- |||||-  \                    *
     *                  |   | \\\  -  /// |   |                    *
     *                  | \_|  ''\---/''  |   |                    *
     *                  \  .-\__  `-`  ___/-. /                    *
     *                ___`. .'  /--.--\  `. . ___                  *
     *              ."" '<  `.___\_<|>_/___.'  >'"".               *
     *            | | :  `- \`.;`\ _ /`;.`/ - ` : | |              *
     *            \  \ `-.   \_ __\ /__ _/   .-` /  /              *
     *      ========`-.____`-.___\_____/___.-`____.-'========      *
     *                           `=---='                           *
     *^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^*
     *          佛祖保佑          永无BUG          永不修改           *
     ***************************************************************
    </pre>
  </body>
</html>
```

## 运行
1. 新建域名解析
2. k8s创建deployment/pod，容器环境变量配置：
	```env
	# https方式为：https://192.168.100.8/<group>
	GIT_BASE_URL=ssh://git@192.168.100.8/<group>
	GIN_MODE=release
	```
3. k8s创建服务，暴露集群IP和映射端口
3. k8s创建ingress

# 参考：
- <https://zhuanlan.zhihu.com/p/384621827>
- <https://stackoverflow.com/questions/59797272/receiving-fatal-could-not-read-username-for-https-github-com-terminal-prom>
- <https://go.dev/ref/mod#serving-from-proxy>
- <https://ichris.info/post/2021/golang/私有仓库/>
- <https://go.dev/ref/mod#private-modules>

# 未测试方案：
https://goproxy.cn/#self-hosted-go-module-proxy
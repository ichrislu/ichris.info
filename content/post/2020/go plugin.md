---
title: "go plugin从入门到放弃"
date: "2020-06-27 10:50:03"
lastMod: "2020-06-29 15:53:03"
tags: ["go", "plugin", "插件"]
---

最开始看到go plugin很兴奋，太实用了。正好手上项目也能用得上，所以花了几天心思去研究一下，所以就有了这篇文章的心德：go plugin从入门到放弃

优势当然是明显的，主程序写好框架，需要适配不同环境的插件则动态编写/替换就好了

但现实很骨感，至少当前版本(1.14.2)的go，plugin远不够在生产环境或大规模应用，原因如下：

插件编译成so没问题，但是主程序加载so的时候报错：plugin was built with a different version of package golang.org/x/sys/unix。

找到的资料说是插件编译环境与主程序的编译环境不一致，或者依赖不一致。但其实我逐一对比了所有go.mod文件的直接和间接依赖，但凡相同的依赖版本是一致的，至于提示中golang.org/x/sys/unix这个包应该是其他第三方依赖包的间接依赖，在Goland中确实可以看到插件和主程序中External Libraries部分这个包的版本并不一致，而且还有其他几个包也不一致，通过执行go get -u无效，这个问题看上去，基本是很难解了，参考的文章也有提到这个问题，几乎所有人都说现阶段的go plugin暂时不能用，此其一。

其二，因为要做成plugin方式，把原来的程序拆开了，于是出现有依赖问题，比如把model拆开，供队列的生产者和消费者使用，go并没有类似于java的公共包打成jar的形式分发，再加入go mod，进而问题变得复杂起来。大致是这样：将model单独成一个项目，由生产者和消费者使用，网上都是说的将代码上传至github.com，但并不适于于企业开发这类并不开源的项目，上传到内网的Gitlab，但是require和import也并不支持内网ip（只支持域名），做成本地的项目依赖也不行，因为我已经把公司项目都做成gitlab-ci来构建了，这个方案直接否决了可移植性。最终决定使用git submodules + go mod和replace，代码如下：

1. 导入

```go
import "collector-driver-north-aliiot/model"
```

2. go.mod

```
replace collector-driver-north-aliiot/model => ./model
```

3. 引用项目的根目录执行：

```shell
git submodule add git@192.168.1.10:iot/collector-model.git model
```

原理很简单，引用的项目根目录以git submodule的形式将依赖项目导入（git的特性，与上一级目录的版本控制分开的，独立的），类似于将本地依赖项目当成本项目的一部分。虽然不完美，也能勉强接受。

编译依赖的项目和引用的项目都很顺利：

```shell
go build -buildmode=plugin -gcflags=-trimpath=$GOPATH -asmflags=-trimpath=$GOPATH -ldflags "-w -s"
```

将so文件复制到主程序，启动时却遇到了问题上面描述的问题。

个人的感觉这东西还不成熟，还不能大量用于生产环境，先暂停研究，转向程序整体自动下载更新替换，并重启的方案。

参考文章：
https://studygolang.com/articles/17365
https://www.jianshu.com/p/b51e955eb1a7
https://mojotv.cn/go/golang-plugin-tutorial
https://studygolang.com/articles/13179

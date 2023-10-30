---
title: "关于golang的workspaces的理解"
date: "2023-10-27 17:01:51"
lastMod: "2023-10-27 17:01:51"
categories: ["it"]
tags: ["golang", "workspaces"]
---

最近重新捡起架构和开发工作，了解了一下golang的“新”特性：workspaces。其实也不算新，只是在上一次做架构和技术选型的时候，golang的最新版本还是1.16.*，而workspaces是Go1.18发布的。

看了官方的特性描述，并没有尝试过。更感觉是module的补充，不必像go module需要submit后其他人才能pull并使用到最新版本。workspaces则更像是一个建立在各个相关的module之上的逻辑概念，在workspaces下面的所有配置了引用的模块，就可以像在一个Project一样相互调用。

更适用于类似开发公司公共包、工具包的开发人员，在其他模块上直接引用做测试（但golang本来就可以在项目内写测试代码呀！）；多人开发的时候，还是需要提交代码了其他人才能使用新特性，这跟Module有什么分别？而且在做了DevOps的团队，自动化构建可能还会存在依赖切换问题。

思来想去，workspaces可能只适合那种一个人，或者少数人开发All in one的项目，当前微服务模式下似乎没有用武之地。
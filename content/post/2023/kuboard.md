---
title: "kuboard"
date: "2023-11-16"
lastMod: "2023-11-16"
categories: ["it"]
tags: ["k8s", "kuboard", "yaml"]
---

每天都有小坑踩！

k8s装好了，想把kuboard也装上去。
看官方文档挺简单，官方推荐docker run方式运行，因为主机上还有其他2个服务也要运行，我当然是选择用compose，既然docker run能运行，compose没有理由不可以呀！
PS：2年没怎么关注docker，发现又有大变化，compose升到V2了，不再是以前那个python项目，改用go开发，做为docker的一个插件，在官方安装文档中推荐安装方式就已经包含了compose插件，不用再次安装了。

照着kuboard官方文档的`docker run`改成了`commpose.yaml`格式，启动却报错了
```
kuboard  | [LOG] 2023/11/16 - 16:59:32.793   | /login.AddLoginRoutes                                         28 | error | 认证模块初始化失败：Get "http://127.0.0.1:5556/sso/.well-known/openid-configuration": dial tcp 127.0.0.1:5556: connect: connection refused
kuboard  | panic: runtime error: invalid memory address or nil pointer dereference
kuboard  | [signal SIGSEGV: segmentation violation code=0x1 addr=0x48 pc=0xd74bd7]
kuboard  |
kuboard  | goroutine 1 [running]:
kuboard  | github.com/coreos/go-oidc.(*Provider).Verifier(...)
kuboard  | 	/usr/src/kuboard/third-party/go-oidc/verify.go:111
kuboard  | github.com/shaohq/kuboard/server/login.AddLoginRoutes(0xc000503d40)
kuboard  | 	/usr/src/kuboard/server/login/login.go:30 +0xf7
kuboard  | main.getRoutes()
kuboard  | 	/usr/src/kuboard/server/kuboard-server.go:193 +0x345
kuboard  | main.main()
kuboard  | 	/usr/src/kuboard/server/kuboard-server.go:65 +0x185
kuboard  |
kuboard  | 启动 kuboard-server 失败，此问题通常是因为 Etcd 未能及时启动或者连接不上，系统将在 15 秒后重新尝试：
kuboard  |   1. 如果您使用 docker run 的方式运行 Kuboard，请耐心等候一会儿或者执行 docker restart kuboard；
kuboard  |   2. 如果您将 Kuboard 安装在 Kubernetes 中，请检查 kuboard/kuboard-etcd 是否正常启动。
```

查了好久找不到原因，尝试用官方的`docker run`运行正常！
排查过网络、卷权限、容器镜像版本等等问题，甚至用过朋友给的yaml（在他那边正常），能正常运行。

大量肉眼对比无果后，最终，终于一次次尝试中找到了原因：`environment`节点下有2种写法，分别是Map和Array，Map可以用引号，Array不能用引号。
我写的是Array语法，用了引号，语法不正确。朋友的是Map语法，虽然有引号，但语法上是正确的。

最终版本：
```yaml
services:
  kuboard:
    image: eipwork/kuboard:v3
    container_name: kuboard
    ports:
      - "80:80/tcp"
      - "10081:10081/tcp"
    volumes:
      - /data/kuboard:/data:rw
    environment:
      KUBOARD_ENDPOINT: "http://192.168.100.8"
      KUBOARD_AGENT_SERVER_TCP_PORT: "10081"
```

官方文档：
https://docs.docker.com/compose/compose-file/05-services/#environment
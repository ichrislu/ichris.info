---
title: "docker保存和加载镜像"
date: "2018-01-25 11:45:00"
lastMod: "2019-07-22 15:06:00"
tags: ["docker", "镜像保存", "镜像加载"]
---

### 背景：
有一些自定义的镜像，不想通过dockerfile创建，没有仓库或者不想上传到仓库中，可以使用保存和加载命令实现环境迁移

### 保存
```shell
docker save -o xxx.tar repo:tag # 注：不能写IMAGE ID
docker image save helloworld > helloworld.tar # 官方写法
```

### 加载：
```shell
docker load < xxx.tar # 或 docker load -i xxx.tar
docker image load -i helloworld.tar # 官方写法
```


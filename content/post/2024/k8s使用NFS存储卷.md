---
title: "k8s使用NFS存储卷"
date: "2024-03-14"
lastMod: "2024-03-15"
categories: ["it"]
tags: ["golang", "go-zero", "gateway", "k8s", "nfs", "pv", "pvc"]
---

# 缘由：
go-zero gateway主要内容开发接近完成了，有一个问题一直没有解决，马上要部署了，是时候解决了。

因为主要是RESTful <--> gRPC，go-zero通过解析pb文件访问后端gRPC服务，pb文件则是通过proto文件生成，proto文件是定义接口的，接口变化会导致pb文件的变化和更新，但是通常gateway模块的功能开发相对稳定和单一，完成后很少会有修改。不希望因为pb文件的变化而导致gateway重新构建、版本发布，pb文件则是更像配置文件存在。

但是k8s中配置文件只能是纯文本，对于pb这类二进制文件只能通过其他方式解决，于是存储卷便成了首选。从2018年玩k8s以来，一直都没有接触过存储卷。

底层的共享实现方式，NFS是最简单直接的。

# 原理：
## nfs

网络文件系统，这个不陌生。

## pv

持久化卷，对底层的共享存储的抽象，由管理员创建和配置，与具体底层共享存储技术的实现方式有关，如NFS，通过插件与共享存储对接。

## pvc

持久卷声明，是用户存储的声明。

pvc与pod类似。pod消耗节点，pvc消耗pv；pod请求CPU/内存，pvc请求存储空间/访问模式

## storage class

更像是动态pv，感觉用不上，没有做过多了解

# 实践：
## 安装NFS服务
```bash
# 安装nfs，rpcbind待依赖会自动安装
sudo yum install -y nfs-utils

# 禁止firewalld，这是最简单的办法！！！开发环境简单操作，不值得研究开放端口。否则会报错：mount.nfs: No route to host
sudo systemctl disable firewalld

# 编辑nfs配置文件
sudo vim /etc/exports

# 如添加/opt共享给所有来源，只读&同步
cat /etc/exports
/opt        *(ro,sync)

# 修改文件后，执行以下指令生效
exportfs -rv

# 启用nfs服务，rpcbind会自动启用
sudo systemctl enable nfs

# 启动nfs服务，rpcbind会自动启动
sudo systemctl start nfs
```

## k8s配置
pv-nfs.yaml
```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  accessModes:
    - ReadOnlyMany
  capacity:
    storage: 1G
  nfs:
    path: <path>
    server: <ip>
  persistentVolumeReclaimPolicy: Retain

```

pvc-nfs.yaml

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
  namespace: dev
spec:
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 1G

```

在deployment中添加存储卷配置：配置存储卷声明中填/选择：pvc-nfs；配置挂载到容器中路径。

## 更简单办法

以kuboard为例，在配置deployment的存储卷中，选择NFS，直接配置NFS Serve和NFS Path，可省去创建pv和pvc。开发环境，简单处理就好！

## 最后一步

本以为到这里就可以了，启动deployment报错：

```bash
文件系统类型错误、选项错误 上有坏超级块、        缺少代码页或助手程序，或其他错误
(对某些文件系统(如 nfs、cifs) 您可能需要一款 /sbin/mount.<类型> 助手程序)
    有些情况下在 syslog 中可以找到一些有用信息- 请尝试
    dmesg | tail  这样的命令看看。
```

经查，忘了在k8s工作节点上安装nfs-utils，默认系统是没有nfs客户端工具的

```bash
sudo yum install -y nfs-utils
sudo systemctl enable nfs
sudo systemctl start nfs
```

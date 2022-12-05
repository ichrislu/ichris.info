---
title: "emqx集群搭建"
date: "2019-12-04 11:17:03"
lastMod: "2019-12-04 11:17:03"
categories: ["it"]
tags: ["emqx", "mqtt", "docker"]
---

最近使用emqx替换原来mosca，公司要求搭建集群模式，本文记录简要过程。

EMQ X 版本支持多种策略的节点自动发现与集群:

| 策略   | 说明                    |
| :----- | :---------------------- |
| manual | 手工命令创建集群        |
| static | 静态节点列表自动集群    |
| mcast  | UDP 组播方式自动集群    |
| dns    | DNS A 记录自动集群      |
| etcd   | 通过 etcd 自动集群      |
| k8s    | Kubernetes 服务自动集群 |

想尝试manual，终觉太麻烦，直接放弃；试过mcast，在同一个compose内还可以，跨机则不行，估计是因是docker网桥在与跨机器的组播是有问题的，没有深究，但还是很看好这种方式的，以后如果能做基于k8s的集群可以继续研究一下；nds/etcd/k8s因为目前没有涉及，只放放弃；那么就剩下static，规模不是太大的情况下，也不是太麻烦，简单记录一下：

先放docker-compose.yml

```yaml
version: '3'
services:
  emqx:
    image: emqx/emqx:v3.2.2
    ports:
    - 1883:1883
    - 8083:8083
    - 4369:4369
    - 7369-7379:7369-7379
    - 18083:18083
    hostname: s1.emqx.io
    environment:
    - EMQX_NAME=mq17
    - EMQX_HOST=192.168.200.17
    - EMQX_NODE__NAME=mq17@192.168.200.17
    - EMQX_CLUSTER__NAME=emqxcl
    - EMQX_CLUSTER__DISCOVERY=static
    - EMQX_CLUSTER__STATIC__SEEDS=mq53@192.168.200.53,mq17@192.168.200.17
    - EMQX_LOADED_PLUGINS="emqx_recon,emqx_retainer,emqx_management,emqx_dashboard,emqx_auth_username"
    - EMQX_NODE__DIST_LISTEN_MIN=7369
    - EMQX_NODE__DIST_LISTEN_MAX=7379
    restart: unless-stopped
    networks:
    - emq

networks:
  emq:
    driver: bridge
```

官方文档跳坑说明：

- 防火墙开放4369端口和一个TCP端口段，同理需要映射到宿主机，以及通过环境变量方式添加到配置
- 官方说开放6369-7369，整整1001个端口，start/stop容器花了较长时间，以至于超时；由于端口范围太广泛，容易出现端口被占用，我改成7369-7379也能正常运行
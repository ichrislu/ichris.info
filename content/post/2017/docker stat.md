---
title: "自定义资源监控"
date: "2017-11-22"
lastMod: "2017-11-22"
categories: ["it"]
tags: ["docker", "stat"]
---

查看docker资源占用
```shell
docker stats --format='table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}'
```
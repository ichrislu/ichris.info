---
title: "自定义资源监控"
date: "2017-11-22 11:51:00"
lastMod: "2017-11-22 11:51:00"
tags: ["docker", "stat"]
---

查看docker资源占用
```shell
docker stats --format='table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}'
```
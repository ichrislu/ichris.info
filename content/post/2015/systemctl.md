---
title: "systemctl"
date: "2015-10-09"
lastMod: "2015-10-09"
categories: ["it"]
tags: ["linux", "CentOS", "CentOS 7", "systemctl"]
---

systemctl是CentOS 7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体

```bash
# 启动一个服务
systemctl start firewalld.service

# 关闭一个服务
systemctl stop firewalld.service

# 重启一个服务
systemctl restart firewalld.service

# 显示一个服务的状态
systemctl status firewalld.service

# 在开机时启用一个服务
systemctl enable firewalld.service

# 在开机时禁用一个服务
systemctl disable firewalld.service

# 查看服务是否开机启动
systemctl is-enabled firewalld.service

# 查看已启动的服务列表
systemctl list-unit-files|grep enabled

# 查看启动失败的服务列表
systemctl --failed
```
---
title: "firewalld常用操作"
date: "2015-10-09"
lastMod: "2018-03-09"
categories: ["it"]
tags: ["linux", "CentOS", "CentOS 7", "firewalld"]
---

CentOS 7之后默认使用firewall做防火墙，配置文件位置：
`/etc/firewalld/zones/public.xml`

firewalld的基本使用
```sh
# 启动
systemctl start firewalld

# 查看状态
systemctl status firewalld 

# 停止
systemctl disable firewalld

# 禁用
systemctl stop firewalld

# 添加端口
# --permanent永久生效，没有此参数重启后失效
firewall-cmd --zone=public --add-port=22/tcp --permanent

# 查询已开放的端口
firewall-cmd --zone=public --list-ports

# 查看
firewall-cmd --zone=public --query-port=80/tcp

# 删除
firewall-cmd --zone=public --remove-port=80/tcp --permanent

# 显示状态
firewall-cmd --state

# 查看区域信息
firewall-cmd --get-active-zones

# 查看指定接口所属区域
firewall-cmd --get-zone-of-interface=eth0

# 拒绝所有包
firewall-cmd --panic-on

# 取消拒绝状态
firewall-cmd --panic-off

# 查看是否拒绝
firewall-cmd --query-panic

# 修改了规则，必须要执行reload才能生效
firewall-cmd --reload
```
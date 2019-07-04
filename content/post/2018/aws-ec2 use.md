---
title: "AWS-EC2使用手记"
date: "2018-12-02 00:56:46"
lastMod: "2018-12-02 00:56:46"
tags: ["AWS", "EC2"]
---

### 关于购买预留实例

老外们的思维方式不同，在中国其实就是购买实例的时候，选择费用方式为包月包年
预留实例在左边菜单的入口处购买

### EC2修改时区
#### 列出时区
```shell
timedatectl list-timezones 
```

#### 修改时区
```shell
timedatectl set-timezone Asia/Shanghai
```

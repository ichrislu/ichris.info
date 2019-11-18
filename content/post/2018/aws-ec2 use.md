---
title: "AWS-EC2使用手记"
date: "2018-12-02 00:56:46"
lastMod: "2019-11-18 18:56:46"
tags: ["AWS", "EC2"]
---

### 关于购买预留实例

老外们的思维方式不同，在中国其实就是购买实例的时候，选择费用方式为包月包年
预留实例在左边菜单的入口处购买

### 初始化服务器
原来的服务器到期了，去年mysql+go+vue架构换成了github+caddy+hugo，配置要求不那么高，所以换了个便宜点新实例，初始化过程如下：
- 设置时区
```shell
timedatectl set-timezone Asia/Shanghai
```

- 安装必需的工具
```shell
sudo yum install -y vim telnet wget unzip epel-release git
```

- 安装hugo
```shell
wget -c https://github.com/gohugoio/hugo/releases/download/v0.59.1/hugo_0.59.1_Linux-64bit.tar.gz
tar zxf hugo_0.59.1_Linux-64bit.tar.gz
sudo mv hugo /usr/local/bin
```

- 安装caddy
```shell
todo...
```

- 安装docker、docker-compose
```shell
todo...
```
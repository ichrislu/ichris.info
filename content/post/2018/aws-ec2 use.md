---
title: "AWS-EC2搭建hugo+caddy，安装shadowsocks手记"
date: "2018-12-02 00:56:46"
lastMod: "2019-11-18 18:56:46"
categories: ["it"]
tags: ["AWS", "EC2", "shadowsocks", "hugo", "caddy"]
---

### 关于购买预留实例

老外们的思维方式不同，在中国其实就是购买实例的时候，选择费用方式为包月包年
预留实例在左边菜单的入口处购买

### 初始化服务器
原来的服务器到期了，去年mysql+go+vue架构换成了github+caddy+hugo，配置要求不那么高，所以换了个便宜点新实例（2vcpu+2G+8G，这么低的配置都感觉有些浪费了），初始化过程如下：
#### 设置时区
```shell
timedatectl set-timezone Asia/Shanghai
```

#### 安装必需的工具
```shell
sudo yum install -y vim telnet wget unzip epel-release git
```

#### 安装hugo
```shell
wget -c https://github.com/gohugoio/hugo/releases/download/v0.59.1/hugo_0.59.1_Linux-64bit.tar.gz
tar zxf hugo_0.59.1_Linux-64bit.tar.gz
sudo mv hugo /usr/local/bin
```

#### 安装caddy 1.0.4
1. 下载caddy，插件勾选http.git
2. 添加系统服务，配置权限。官方文档参考：https://github.com/caddyserver/caddy/tree/v1.0.4/dist/init/linux-systemd

注意：
1. GID33在CentOS中已经存在，要改一下
2. 文档中/var/www/example.com权限为555，我这边需要从github拉md文件给hugo动态生成网站文件，所以需要权限为755
3. 如果在Caddyfile中重写了日志或错误的重定向到文件，则对应的文件和逐级目录需要给启动caddy的账号权限，官方默认账号为www-data（这个问题找了很久，因为没有权限写日志，所以启动失败，stdout又查不到原因。其实最终我还是删除了错误日志的重定向写文件，因为静态web基本不会有运行时错误，基本上错误都在启动时出现并解决了）
4. 官方的caddy.service有问题，需要删除

#### 安装docker、docker-compose
[安装官方文档] (https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository)

#### 安装ss-server
```yaml
version: '3'
services:
  s3:
    image: mritd/shadowsocks:3.3.3
    ports:
    - ****:****
    - ****:****/udp
    container_name: s3
    restart: always
    command: -m "ss-server" -s "-s 0.0.0.0 -p **** -m aes-256-cfb -k ************************* --fast-open" -x -e "kcpserver" -k "-t 127.0.0.1:**** -l :**** -mode fast3"
```
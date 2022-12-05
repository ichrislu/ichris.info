---
title: "Prometheus探针node_exporter和mysqld_exporter服务化安装"
date: "2019-07-24 19:11:03"
lastMod: "2019-07-25 10:15:03"
categories: ["it"]
tags: ["Prometheus", "node_exporter", "mysqld_exporter"]
---

Prometheus + Grafana已经搭建完成，之前嘱咐下面的人把探针装上，昨天做压力测试，发现数据经常中断。一查才知道，使用的是比较老的版本，而且各服务器版本还不一致，有些与当前监控模板并不一定兼容；一部分竟然是前台启动，ssh断了程序就停了……还有一部分是nohup xx &启动的，那么每次开机都要手动运行？很无语，什么事都要自己亲自来才放心

### node_exporter：

- 上官网下载了最新版程序，解压后在目录中看到一个二进制文件和两个说明文件，很明显的go风格……

- 将二进制文件mv到/usr/local/bin，因为解压后已有执行权限，不用再chmod +x
- 创建服务：

```bash
sudo vim /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
After=syslog.target
After=network.target

[Service]
Type=simple
User=nobody
Group=nobody
Restart=always
SuccessExitStatus=143
ExecStart=/usr/local/bin/node_exporter
ExecStop=/usr/bin/kill -15 $MAINPID

[Install]
WantedBy=default.target
```

- 应用服务：

```bash
sudo systemctl daemon-reload
systemctl start node_exporter.service
systemctl enable node_exporter.service
```

### mysqld_exporter：

与node_exporter类似，这里只描述差异

mysqld_exporter启动要求有配置，官方文档描述配置有两种方式：

1. 配置文件

2. 环境变量

配置文件方式可能不太适合，因为mysql主从有多台，每台都要再写一个配置文件，多一份维护负担，这里选择环境变量，而且直接将环境变量写到service配置文件中，如下：

```bash
sudo vim /etc/systemd/system/mysqld_exporter.service

[Unit]
Description=Node Exporter
After=syslog.target
After=network.target

[Service]
Type=simple
User=nobody
Group=nobody
Restart=always
SuccessExitStatus=143
Environment=DATA_SOURCE_NAME=<user>:<password>@(localhost:3306)/
ExecStart=/usr/local/bin/mysqld_exporter
ExecStop=/usr/bin/kill -15 $MAINPID

[Install]
WantedBy=default.target
```

注：试过使用有库所有权限的子账号，启动报错，说是缺少权限（错误信息没有保存下来）应该是取mysql性能数据还需要读mysql系统库吧，子账号当然是没有权限的，我就简单的直接使用了root账号，只是localhost访问，而且也只是取性能数据而已。


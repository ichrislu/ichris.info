---
title: "docker18.09安装"
date: "2022-06-29"
lastMod: "2022-06-29"
categories: ["it"]
tags: ["docker", "engine", "cli", "containerd"]
---

Docker 18.09开始，拆分为"engine", "cli", and "containerd"，需要分别单独下载和安装。
如：containerd.io-1.2.0-3.el7.x86_64.rpm、docker-ce-18.09.0-3.el7.x86_64.rpm、docker-ce-cli-18.09.0-3.el7.x86_64.rpm

且安装顺序为containerd、cli、engine

```bash
sudo yum install -y containerd.io-1.2.0-3.el7.x86_64.rpm
sudo yum install -y docker-ce-cli-18.09.0-3.el7.x86_64.rpm
sudo yum install -y docker-ce-18.09.0-3.el7.x86_64.rpm
```

也可以：yum install -y containerd.io-1.2.0-3.el7.x86_64.rpm docker-ce-cli-18.09.0-3.el7.x86_64.rpm docker-ce-18.09.0-3.el7.x86_64.rpm

containerd单独做为systemd服务。
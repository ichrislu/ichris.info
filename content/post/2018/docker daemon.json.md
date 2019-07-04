---
title: "Docker配置文件"
date: "2018-02-01 15:12:00"
lastMod: "2018-02-01 15:12:00"
tags: ["docker"]
---

默认dockerd全局配置文件在/etc/docker/daemon.json，该文件默认不存在，创建就好了，常用内容：

```json
{
"data-root": "/data/docker",
"log-driver":"json-file",
	"log-opts":{
		"max-size": "10m",
		"max-file": "3"
	}
}
```

设置docker主目录，docker运行产生的数据都保存在这里，如镜像、容器、日志、卷等，会占用较大空间，默认在/var/docker下，**务必修改！！！**
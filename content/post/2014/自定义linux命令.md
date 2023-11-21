---
title: "自定义linux命令"
date: "2014-11-13"
lastMod: "2014-11-13"
categories: ["it"]
tags: ["linux"]
---

### 创建自定义命令myjps

1. 用root账户进入/bin目录
2. 建立一个名为myjps的文件，内容：
	```bash
	pwdx `jps -l|grep -v jps|awk '{print $1}'`
	```
3. 赋予权限：chmod 755 myjps

其他账户也能使用myjps命令了
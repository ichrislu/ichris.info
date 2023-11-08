---
title: "解决MySQL报too many open files的问题"
date: "2019-04-04"
lastMod: "2019-04-04"
categories: ["it"]
tags: ["ulimit", "mysql"]
---

uat环境一直有偶尔报too many open files，改过ulimit，已经从默认值改到了102400，仍然没有解决
后来网上查，发现其实在MySQL内部也有这个限制，官方文档描述：https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_open_files_limit

在想像中，编辑my.conf，在[mysqld]中添加open_files_limit = 102400，重启应该就行了
但其实至少在5.7.21版本里是行不通的，需要修改service配置文件，以CentOS7.*为例：
```bash
# 编辑service文件
vim /usr/lib/systemd/system/mysqld.service

# Sets open_files_limit
LimitNOFILE = 102400

systemctl daemon-reload
systemctl restart mysqld
```

检查该配置是否生效
```sql
SHOW GLOBAL VARIABLES LIKE 'open%'
```


见官方文档
https://dev.mysql.com/doc/refman/5.7/en/using-systemd.html
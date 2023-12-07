---
title: "MySQL8配置"
date: "2023-12-07"
lastMod: "2023-12-07"
categories: ["it"]
tags: ["mysql", "mysql8", "my.ini", "mysql.conf"]
---

2017年有篇文章写过关于MySQL中文排序的问题，最早这个问题在2007年MySQL 5.5的时候就有发现，今天研究了一下MySQL 8，找到一个熟悉的字眼`collation`、`utf8mb4_zh_*`，翻了一下官方文档，终于这个问题得以完美解决了。修改配置文件，重启MySQL后，数据库、表默认字符集和排序都正常。整理配置如下：
```ini
# 因为本次数据库安装在Windows上，所以为Windows格式配置文件my.ini
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
default-time_zone='+8:00'

character-set-server=utf8mb4
collation-server=utf8mb4_zh_0900_as_cs

skip-name-resolve
explicit_defaults_for_timestamp=true

group_concat_max_len=102400

[mysqldump]
quick
```
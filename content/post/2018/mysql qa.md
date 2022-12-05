---
title: "mysql常见问题"
date: "2018-03-31 12:09:00"
lastMod: "2018-08-28 19:22:00"
categories: ["it"]
tags: ["mysql"]
---

#### 问题
create table: Specified key was too long; max key length is 767 bytes

#### 原因
数据库表采用utf8编码，其中varchar(255)的column进行了唯一键索引
而mysql默认情况下单个列的索引不能超过767位(不同版本可能存在差异)

于是utf8字符编码下，255*3 byte 超过限制

#### 解决
1  使用innodb引擎；
2  启用innodb_large_prefix选项，将约束项扩展至3072byte；
3  重新创建数据库；

my.cnf配置：
default-storage-engine=INNODB
innodb_large_prefix=on

一般情况下不建议使用这么长的索引，对性能有一定影响；

> 参考文档：
> https://dev.mysql.com/doc/refman/5.5/en/innodb-restrictions.html

---

#### 问题
默认的错误消息找不到，导致SQL出错时，只有错误编号，没有错误信息，启动时报警告：[ERROR] Can't find error-message file '/usr/local/mysql/share/mysql/errmsg.sys'. Check error-message file location and 'lc-messages-dir' configuration directive。

#### 解决：
配置文件加上：
lc-messages-dir=/usr/share/mysql
lc-messages=en_US
在MySQL 5.7中废弃了language配置的老式指定方法，改用上面两个配置结合
官方：
https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_lc-messages-dir
https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_lc_messages

---

#### 问题
启动报警告：[Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).

#### 解决：
https://blog.csdn.net/shaochenshuo/article/details/50577868，未验证！！！
官方：
https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_explicit_defaults_for_timestamp

---

#### 问题
时区不对

#### 解决
编辑my.cnf
[mysqld]内添加
default-time_zone='+8:00'
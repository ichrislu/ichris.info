---
title: "CentOS7安装MySQL8的大坑"
date: "2024-03-22"
lastMod: "2024-03-22"
categories: ["it"]
tags: ["CentOS", "MySQL"]
---

按官方文档，安装好MySQL后，配置好/etc/my.conf文件，但是启动失败，查日志发现报错：

```bash
/usr/sbin/restorecon:  lstat(/data/mysql) failed:  No such file or directory
```

是因为datadir文件夹没有写权限

```bash
sudo chown -R mysql:mysql /data/mysql/

# 这步操作是否必要，没有验证过，理论上是不需要的
chmod -R 755 /data/mysql/
```

然后仍然无法启动，于是关闭selinux，重启，日志报错：
```bash
2024-03-22T08:37:07.660421Z 0 [Warning] [MY-010091] [Server] Can't create test file /data/mysql/mysqld_tmp_file_case_insensitive_test.lower-test
2024-03-22T08:37:07.660426Z 0 [Warning] [MY-010159] [Server] Setting lower_case_table_names=2 because file system for /data/mysql/ is case insensitive
mysqld: File './binlog.index' not found (OS errno 13 - Permission denied)
2024-03-22T08:37:07.660794Z 0 [ERROR] [MY-010119] [Server] Aborting
```

继续不行，直到发现有人说需要删除配置文件my.conf中的空行才行，实在没办法了，尝试了一下，果然。。。太坑了！

==============

重启成功后，在默认日志文件：/var/log/mysqld.log中找到临时密码登录，并修改密码：
```bash
mysql -uroot -p
mysql> use mysql;
mysql> set password for root@localhost='***********';
```

报错：

```bash
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

密码策略太严格，于是修改密码策略：
```bash
set global validate_password.policy=0;
```

注意这里的全局变量名，跟网上有些文章提到的不一样！！！可能是版本不同吧。
可以在这里找到正确的变量名：

```bash
mysql> SHOW VARIABLES LIKE 'validate_password%';
+-------------------------------------------------+--------+
| Variable_name                                   | Value  |
+-------------------------------------------------+--------+
| validate_password.changed_characters_percentage | 0      |
| validate_password.check_user_name               | ON     |
| validate_password.dictionary_file               |        |
| validate_password.length                        | 8      |
| validate_password.mixed_case_count              | 1      |
| validate_password.number_count                  | 1      |
| validate_password.policy                        | MEDIUM |
| validate_password.special_char_count            | 1      |
+-------------------------------------------------+--------+
```

然后再次修改密码，刷新权限即可：
```bash
flush privileges;
```

参考：
<https://blog.csdn.net/qq_36408717/article/details/126705287>
<https://blog.csdn.net/ayychiguoguo/article/details/120370686>
<https://blog.csdn.net/wltsysterm/article/details/79649484>
<https://blog.csdn.net/qq_27884227/article/details/132596525>
<https://www.cnblogs.com/soft-test/p/15659006.html>
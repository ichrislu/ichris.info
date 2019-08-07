---
title: "MySQL中间件选择"
date: "2018-11-05 14:52:00"
lastMod: "2019-08-06 18:22:00"
tags: ["mysql", "中间件", "读写分离"]
---

### MyCAT(1.6.5)
先天不足，发现部分应用有兼容性问题
1. XXL-JOB官方文档有说明：如果mysql做主从,调度中心集群节点务必强制走主库
2. flyway也不支持！

### TiDB(2.0版)
对运行环境硬件配置要求高得变态，某公司的服务器上了高性能SSD仍然不能满足IOPS要求（<40000）

### Sharding-JDBC 3.0.0（Proxy也应该是这样，但未测试）
SQL限制很多，不支持冗余括号、CASE WHEN、DISTINCT、HAVING、UNION (ALL)，有限支持子查询。http://shardingsphere.io/document/current/cn/features/sharding/usage-standard/sql/

### Sharding-Proxy 3.1.0/4.0.0-RC1（2019-08-06测试）

SQL限制未详细研究，但是发现致命问题，数据表中有is_active/is_start这样的字段，其值为0或1，很明显这是保存的状态，语义类型为布尔类型，但是查出来结果却让人惊讶。

环境说明：

- 151/152分别为start.sh和docker run运行的Sharding-Proxy 3.1.0/4.0.0-RC1
- 153/154/155为MySQL主从，未发现有延迟
- 153 -> master
- 154/155 -> slave

查询语句

```sql
SELECT is_active,is_start FROM <table> WHERE id_ = '<id>';
```

**数据库值（153、154、155）**
mysql客户端和navcat等客户端都返回
1、0

**sharding-proxy（151、152）**
mysql客户端返回
true、false

navcate返回
1、1

抛开不同工具的差异，这确实给程序造成了麻烦，通过压力测试反馈出来了……

网上找了一圈，没人反馈这个问题，github上issue也没人提，难道大家用这个就没发现这个问题吗？时间有限，现在我没时间去查源码，只能先换用其他中间件。

### ProxySQL

官方说比MaxScale好

经测试结果如下：

**功能**

经测试sharding-proxy的问题不存在

**性能**

测试中。。。

**问题**

服务器1，安装很顺利

服务器2，安装后一直报错：

```bash
[root@copl-srv013-152 ~]# systemctl status proxysql
● proxysql.service - LSB: High Performance Advanced Proxy for MySQL
   Loaded: loaded (/etc/rc.d/init.d/proxysql; bad; vendor preset: disabled)
   Active: inactive (dead) since Wed 2019-08-07 16:26:45 CST; 16s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 6022 ExecStop=/etc/rc.d/init.d/proxysql stop (code=exited, status=0/SUCCESS)
  Process: 2744 ExecStart=/etc/rc.d/init.d/proxysql start (code=exited, status=0/SUCCESS)

Aug 07 16:09:37 copl-srv013-152 systemd[1]: Starting LSB: High Performance Advanced Proxy for MySQL...
Aug 07 16:09:37 copl-srv013-152 su[2747]: (to proxysql) root on none
Aug 07 16:09:37 copl-srv013-152 proxysql[2744]: Starting ProxySQL: 2019-08-07 16:09:37 main.cpp:720:ProxySQL_Main_process_global_variables(): [WARNING] Unable to open config file /etc/proxysql.cnf
Aug 07 16:09:37 copl-srv013-152 proxysql[2744]: 2019-08-07 16:09:37 main.cpp:722:ProxySQL_Main_process_global_variables(): [ERROR] Unable to open config file /etc/proxysql.cnf specified in the command line. Aborting!
Aug 07 16:09:37 copl-srv013-152 proxysql[2744]: DONE!
Aug 07 16:09:37 copl-srv013-152 systemd[1]: Started LSB: High Performance Advanced Proxy for MySQL.
```

因为经过了服务器1的安装，所以服务器2安装过程较快，直接复制了服务器1的配置文件/etc/proxysql.cnf过来，启动，则报上面的错误，文件内容是没问题的。服务器1与此不同的是之前不是通过/etc/proxysql.cnf配置加载的，是通用方式，启动后，生成了proxysql.db文件，通过sql语句添加的配置，后来发现配置文件更具有可移植性，所以才换成配置文件方式，非常顺利！

最后解决方案很奇葩！！！通过以上日志可看出，proxysql还是使用centos7之前的service的管理方式，只是针对centos7的systemctl做了兼容处理，于是打开/etc/rc.d/init.d/proxysql文件，看一看内容，找到了这些内容：

```bash
Usage: ProxySQL {start|stop|status|reload|restart|initial}
```

于是尝试一下

```base
[root@copl-srv013-152 ~]# /etc/rc.d/init.d/proxysql initial
Starting ProxySQL: 2019-08-07 17:12:28 [INFO] Using config file /etc/proxysql.cnf
Renaming database file /var/lib/proxysql/proxysql.db
2019-08-07 17:12:28 [INFO] No SSL keys/certificates found in datadir (/var/lib/proxysql). Generating new keys/certificates.
DONE!
[root@copl-srv013-152 ~]# /etc/rc.d/init.d/proxysql restart
Shutting down ProxySQL: DONE!
Starting ProxySQL: 2019-08-07 17:12:39 [INFO] Using config file /etc/proxysql.cnf
2019-08-07 17:12:39 [INFO] SSL keys/certificates found in datadir (/var/lib/proxysql): loading them.
DONE!
[root@copl-srv013-152 ~]# systemctl status proxysql
● proxysql.service - LSB: High Performance Advanced Proxy for MySQL
   Loaded: loaded (/etc/rc.d/init.d/proxysql; bad; vendor preset: disabled)
   Active: active (exited) since Wed 2019-08-07 16:27:05 CST; 45min ago
     Docs: man:systemd-sysv-generator(8)

Aug 07 16:27:04 copl-srv013-152 systemd[1]: Starting LSB: High Performance Advanced Proxy for MySQL...
Aug 07 16:27:04 copl-srv013-152 su[6065]: (to proxysql) root on none
Aug 07 16:27:05 copl-srv013-152 proxysql[6062]: Starting ProxySQL: 2019-08-07 16:27:05 main.cpp:720:ProxySQL_Main_process_global_variables(): [WARNING] Unable to open config file /etc/proxysql.cnf
Aug 07 16:27:05 copl-srv013-152 proxysql[6062]: 2019-08-07 16:27:05 main.cpp:722:ProxySQL_Main_process_global_variables(): [ERROR] Unable to open config file /etc/proxysql.cnf specified in the command line. Aborting!
Aug 07 16:27:05 copl-srv013-152 proxysql[6062]: DONE!
Aug 07 16:27:05 copl-srv013-152 systemd[1]: Started LSB: High Performance Advanced Proxy for MySQL.
[root@copl-srv013-152 ~]# systemctl restart proxysql
[root@copl-srv013-152 ~]# systemctl status proxysql
● proxysql.service - LSB: High Performance Advanced Proxy for MySQL
   Loaded: loaded (/etc/rc.d/init.d/proxysql; bad; vendor preset: disabled)
   Active: active (exited) since Wed 2019-08-07 17:13:00 CST; 1s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 18958 ExecStop=/etc/rc.d/init.d/proxysql stop (code=exited, status=0/SUCCESS)
  Process: 19151 ExecStart=/etc/rc.d/init.d/proxysql start (code=exited, status=0/SUCCESS)

Aug 07 17:12:59 copl-srv013-152 systemd[1]: Starting LSB: High Performance Advanced Proxy for MySQL...
Aug 07 17:12:59 copl-srv013-152 su[19153]: (to proxysql) root on none
Aug 07 17:12:59 copl-srv013-152 proxysql[19151]: Starting ProxySQL: 2019-08-07 17:12:59 [INFO] Using config file /etc/proxysql.cnf
Aug 07 17:12:59 copl-srv013-152 proxysql[19151]: 2019-08-07 17:12:59 [INFO] SSL keys/certificates found in datadir (/var/lib/proxysql): loading them.
Aug 07 17:12:59 copl-srv013-152 proxysql[19151]: DONE!
Aug 07 17:13:00 copl-srv013-152 systemd[1]: Started LSB: High Performance Advanced Proxy for MySQL.
```

搞定！这是一个很奇葩的bug！！！

### MaxScale
Maridb是MySQL的开源分支，也是MySQL作者弄的，特殊的官方吧
待测试评估的

### Vitess
Google开发的
待测试评估的

### OneProxy
商业软件
待测试评估的

### *Amoeba*
*好像不维护了*

### *DRDS*
*阿里云产品，局限性显而易见*

---
以下两个是官方的，对比说明：https://blog.csdn.net/henu_xiaohei/article/details/83145330
### MySQL-Proxy

### MySQL-Router
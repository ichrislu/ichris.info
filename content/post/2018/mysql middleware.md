---
title: "MySQL中间件选择"
date: "2018-11-05 14:52:00"
lastMod: "2019-08-09 18:22:00"
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

事后在另外一篇文章中找到相同的解决办法：https://blog.51cto.com/bigboss/2103290

**原理：**

仅在第一次(/var/lib/proxysql/proxysql.db文件不存在)启动时有效

启动后可以在proxysql管理端中通过修改数据库的方式修改配置并生效(官方推荐方式)

如果想重新初始化，删除proxysql.db，改好proxysql.cnf，再启动程序也可以

**/etc/proxysql.cnf配置文件示例：**

```cnf
#file proxysql.cfg

########################################################################################
# This config file is parsed using libconfig , and its grammar is described in:
# http://www.hyperrealm.com/libconfig/libconfig_manual.html#Configuration-File-Grammar
# Grammar is also copied at the end of this file
########################################################################################

########################################################################################
# IMPORTANT INFORMATION REGARDING THIS CONFIGURATION FILE:
########################################################################################
# On startup, ProxySQL reads its config file (if present) to determine its datadir.
# What happens next depends on if the database file (disk) is present in the defined
# datadir (i.e. "/var/lib/proxysql/proxysql.db").
#
# If the database file is found, ProxySQL initializes its in-memory configuration from
# the persisted on-disk database. So, disk configuration gets loaded into memory and
# then propagated towards the runtime configuration.
#
# If the database file is not found and a config file exists, the config file is parsed
# and its content is loaded into the in-memory database, to then be both saved on-disk
# database and loaded at runtime.
#
# IMPORTANT: If a database file is found, the config file is NOT parsed. In this case
#            ProxySQL initializes its in-memory configuration from the persisted on-disk
#            database ONLY. In other words, the configuration found in the proxysql.cnf
#            file is only used to initial the on-disk database read on the first startup.
#
# In order to FORCE a re-initialise of the on-disk database from the configuration file
# the ProxySQL service should be started with "service proxysql initial".
#
########################################################################################

datadir="/var/lib/proxysql"
errorlog="/var/lib/proxysql/proxysql.log"

admin_variables=
{
        admin_credentials="admin:admin"
#       mysql_ifaces="127.0.0.1:6032;/tmp/proxysql_admin.sock"
        mysql_ifaces="0.0.0.0:6032"
#       refresh_interval=2000
#       debug=true
}

mysql_variables=
{
        threads=32
        max_connections=2048
        default_query_delay=0
        default_query_timeout=36000000
        have_compress=true
        poll_timeout=2000
#       interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
        interfaces="0.0.0.0:6033"
        default_schema="information_schema"
        stacksize=1048576
        server_version="5.7.23"
        connect_timeout_server=3000
# make sure to configure monitor username and password
# https://github.com/sysown/proxysql/wiki/Global-variables#mysql-monitor_username-mysql-monitor_password
        monitor_username="monitor"
        monitor_password="monitor"
        monitor_history=600000
        monitor_connect_interval=60000
        monitor_ping_interval=10000
        monitor_read_only_interval=1500
        monitor_read_only_timeout=500
        ping_interval_server_msec=120000
        ping_timeout_server=500
        commands_stats=true
        sessions_sort=true
        connect_retries_on_failure=10
}


# defines all the MySQL servers
mysql_servers =
(
        {
                address = "server1.ip",
                port = 3306,
                hostgroup = 0,
                max_connections = 2000
        },
        {
                address = "server2.ip",
                port = 3306,
                hostgroup = 1,
                max_connections = 2000
        },
        {
                address = "server3.ip",
                port = 3306,
                hostgroup = 1,
                max_connections = 2000
        }
)


# defines all the MySQL users
mysql_users:
(
        {
                username = "<username>",
                password = "<password>",
                default_hostgroup = 0,
                max_connections = 2000,
                default_schema = "<database>",
                active = 1
        },
        {
                username = "<username>",
                password = "<password>",
                default_hostgroup = 1,
                max_connections = 2000,
                default_schema = "<database>",
                active = 1
        },
)



#defines MySQL Query Rules
mysql_query_rules:
(
        {
                rule_id = 1,
                active = 1,
                match_pattern = "^SELECT .* FOR UPDATE$",
                destination_hostgroup = 0,
                apply=1
        },
        {
                rule_id = 2,
                active = 1,
                match_pattern = "^SELECT",
                destination_hostgroup = 1,
                apply = 1
        }
)

scheduler=
(
#  {
#    id=1
#    active=0
#    interval_ms=10000
#    filename="/var/lib/proxysql/proxysql_galera_checker.sh"
#    arg1="0"
#    arg2="0"
#    arg3="0"
#    arg4="1"
#    arg5="/var/lib/proxysql/proxysql_galera_checker.log"
#  }
)


mysql_replication_hostgroups=
(
        {
                writer_hostgroup = 10,
                reader_hostgroup = 20,
                comment = "proxy"
        }
)




# http://www.hyperrealm.com/libconfig/libconfig_manual.html#Configuration-File-Grammar
#
# Below is the BNF grammar for configuration files. Comments and include directives are not part of the grammar, so they are not included here.
#
# configuration = setting-list | empty
#
# setting-list = setting | setting-list setting
#
# setting = name (":" | "=") value (";" | "," | empty)
#
# value = scalar-value | array | list | group
#
# value-list = value | value-list "," value
#
# scalar-value = boolean | integer | integer64 | hex | hex64 | float
#                | string
#
# scalar-value-list = scalar-value | scalar-value-list "," scalar-value
#
# array = "[" (scalar-value-list | empty) "]"
#
# list = "(" (value-list | empty) ")"
#
# group = "{" (setting-list | empty) "}"
#
# empty =
```



**差异**

当前了解到ProxySQL和MyCat最大差异在于不需要在代理层添加访问子账号，而是直接使用主从物理数据库的子账号，所以应该是在物理数据库中创建好相同的账号（密码也应该相同），程序配置ProxySQL的端口和地址，使用物理数据库中创建的账号

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


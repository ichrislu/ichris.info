---
title: "MySQL主从延迟解决方案"
date: "2018-12-11"
lastMod: "2018-12-11"
categories: ["it"]
tags: ["mysql", "主从同步", "主从延迟"]
---

### 版本：5.7.23

### 现象
过一段时间会发现从库追不上主库，而且差距越来越大

此时通过在主从库上分别查询：select * from information_schema.processlist where info is not null order by host
得知：
1. 主库一直有不断的增删改操作
2. 从库一直只有查询操作

查从机：show slave status，得知：

```
Slave_IO_Running:Yes
Slave_SQL_Running:Yes
Slave_IO_State:Waiting for master to send event
Read_Master_Log_Pos:一直有变化
Relay_Log_Pos:一直有变化
Last_Error:0
Skip_Counter:0
Exec_Master_Log_Pos:一直有变化
Relay_Log_Space:1327259515，一直在增长
Seconds_Behind_Master:3512，一直在增长
Last_IO_Error:0
Last_SQL_Error:0
SQL_Delay:0
SQL_Remaining_Delay:null
Slave_SQL_Running_State:Reading event from the relay log 或 System lock
Last_IO_Error_Timestamp:
Last_SQL_Error_Timestamp:
Retrieved_Gtid_Set:ccd0b8db-b518-11e8-bb2a-0050569aa2a7:10995977-59328840，一直有变化
Executed_Gtid_Set:ccd0b8db-b518-11e8-bb2a-0050569aa2a7:1-56751544，一直有变化

```

### 原因：
主库写入速度太快，从库跟不上
怀疑还可能有其他原因，因为这个这个原因无法解释从库为什么一直只有查询操作

### 解决方案：
开启并行复制

### 实施：
编辑MySQL配置文件my.cnf，找到[mysqld]配置块，添加如下：
```properties
slave_parallel_workers=16
slave_parallel_type=LOGICAL_CLOCK
```

> 参考
> 
> - 原因：https://www.cnblogs.com/cnmenglang/p/6393769.html
> - 解决：http://blog.51cto.com/sumongodb/1958723
> - 更多：https://blog.csdn.net/jh993627471/article/details/79009313
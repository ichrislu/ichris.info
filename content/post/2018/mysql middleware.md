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

正在测试中……

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
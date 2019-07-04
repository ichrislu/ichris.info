---
title: "MySQL中间件选择"
date: "2018-11-05 14:52:00"
lastMod: "2018-11-05 14:52:00"
tags: ["mysql", "中间件", "读写分离"]
---

### MyCAT(1.6.5)
先天不足，发现部分应用有兼容性问题
1. XXL-JOB官方文档有说明：如果mysql做主从,调度中心集群节点务必强制走主库
2. flyway也不支持！

### TiDB(2.0版)
对运行环境要求高得变态，某公司的服务器上了高性能SSD仍然不能满足IOPS要求（<40000）

### Sharding-Share 3.0.0（本次使用的是JDBC；Proxy也应该是这样，但未测试）
SQL限制很多，不支持冗余括号、CASE WHEN、DISTINCT、HAVING、UNION (ALL)，有限支持子查询。http://shardingsphere.io/document/current/cn/features/sharding/usage-standard/sql/

### ProxySQL
待测试评估的

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
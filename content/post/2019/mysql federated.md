---
title: "mysql federated引擎使用"
date: "2019-12-31"
lastMod: "2019-12-31"
categories: ["it"]
tags: ["mysql", "federated", "跨服务操作"]
---

跨服务器操作mysql数据库的需求，mysql提供了federated引擎，进行表映射，然后进行操作

步骤：

1. 确认federated引擎是否启用，执行：show engines查看引擎状态，找到federated的Support，默认是关闭的，即NO

2. 启用federated引擎，修改mysql配置文件，在[mysqld]下添加federated

3. 重启mysql

4. 再次查看引擎状态

5. 以上，在需要做表映射的mysql服务上开户federated引擎支持

6. 然后创建一张与目标表的表结构一样的数据表，唯一区别的是表描述信息添加：ENGINE=FEDERATED CONNECTION='mysql://<user>:<password>@<hostname>:<port>/<database>/<table>'
   示例：

   ```sql
   CREATE TABLE `test` (
   	`id_` INT NOT NULL AUTO_INCREMENT,
   	`ts` TIMESTAMP NOT NULL DEFAULT NOW(),
   	PRIMARY KEY (`id_`)
   ) ENGINE=FEDERATED CONNECTION='mysql://<user>:<password>@<hostname>:<port>/<database>/<table>'
   COMMENT='for test'
   COLLATE='utf8mb4_unicode_ci'
   ;
   ```

至此，跨服务器表映射完成！

经测试，在映射表的服务器与被映射表的服务器，都可以进行查询、添加等操作


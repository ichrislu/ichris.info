---
title: "MySQL5.7修改root密码"
date: "2017-07-10"
lastMod: "2017-07-10"
categories: ["it"]
tags: ["mysql"]
---

当前以CentOS 7，MySQL5.7.18为例，小版本有差异的！！！

编辑/etc/my.cnf
在[mysqld]下面添加
```properties
skip-grant-tables=1
```
重启mysql
```bash
systemctl restart mysqld
```

修改root密码，字段并非某些文章里说的password字段（5.7）版改为authentication_string字段，语句
```sql
update user set  authentication_string=password('root') where user='root';
```

退出mysql，删除skip-grant-tables=1配置
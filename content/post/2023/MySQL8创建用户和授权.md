---
title: "MySQL8创建用户和授权"
date: "2023-12-26"
lastMod: "2023-12-26"
categories: ["it"]
tags: ["mysql", "mysql8"]
---

按之前版本MySQL创建用户和授权

```sql
# 创建用户
CREATE USER '<user>'@'%' IDENTIFIED BY '<password>';

# 授权
GRANT ALL PRIVILEGES ON <database>.* TO '<user>'@'%';

# 刷新权限
FLUSH PRIVILEGES;
```

在正常登录后报`1142`错误，经查解决方案如下：
```sql
# 替换成相应用户
SELECT * FROM `user` WHERE user='<user>';
```

在显示的查询结果中，将以下7个字段值改为Y：
`Select_priv, Insert_priv, Update_priv, Delete_priv, Create_priv, Create_priv, Drop_priv, Reload_priv`

最重要的，**重启**MySQL服务生效！
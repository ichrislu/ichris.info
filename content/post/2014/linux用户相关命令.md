---
title: "用户相关命令"
date: "2014-04-26"
lastMod: "2014-04-26"
categories: ["it"]
tags: ["linux"]
---

```bash
# 创建用户
useradd <user> -d <dir>

# 删除用户
userdel <user>

# 增加用户组
groupadd <group>

# 删除用户组
groupdel <group>

# 暂时终止用户
passwd -l <user>

# 恢复被终止用户
passwd -u <user>

# 修改用户密码
passwd <user>

# 清除用户
userdel -r

# 删除已命名用户的密码(只有根用户才能进行此操作)
sudo passwd -d <user>
```


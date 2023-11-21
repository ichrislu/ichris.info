---
title: "sudo"
date: "2018-12-22"
lastMod: "2018-12-22"
categories: ["it"]
tags: ["linux", "sudo"]
---

```bash
vim /etc/sudoers

# %开头为组，若当前用户为管理员组，只需要将%admin该行添加NOPASSWD: ALL即可
# 像这样
# 原来：%admin          ALL = (ALL) ALL
# 改后：%admin          ALL = (ALL) NOPASSWD:ALL

# 如果不是管理员组，则按格式新建一行（未验证）
```
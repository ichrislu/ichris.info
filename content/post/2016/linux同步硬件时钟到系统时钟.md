---
title: "linux同步硬件时钟到系统时钟"
date: "2016-10-10"
lastMod: "2016-10-10"
categories: ["it"]
tags: ["linux", "硬件时钟", "系统时钟"]
---

### linux时钟设置
- `hwclock -s, --hctosys` 从硬件时钟设置系统时间
- `hwclock -w, --systohc` 从当前系统时间设置硬件时钟

### linux同步硬件时钟到系统时钟
编辑：`/etc/crontab`，尾部添加：
```bash
*/10 * * * * root hwclock --hctosys
# 表示每10分钟同步一次（将硬件时钟同步至系统时钟）

0 3 * * * root hwclock --hctosys
# 表示每天3点同步一次（将硬件时钟同步至系统时钟）
```
**注意文件尾部需要换行符**

查看同步日志：`less /var/log/cron`

修改`/etc/crontab`这种方法只有root用户能用，这种方法更加方便与直接直接给其他用户设置计划任务，而且还可以指定执行shell等等，推荐这种方法。  
用`crontab -e`这种所有用户都可以使用，普通用户也只能为自己设置计划任务。
---
title: "linux环境变量设置区别"
date: "2016-06-16"
lastMod: "2016-06-16"
categories: ["it"]
tags: ["linux", "profile", "profile.d", "环境变量"]
---

### linux环境变量设置区别
/etc/profile 和 /etc/profile.d/
1. 两个文件都是设置环境变量文件的，/etc/profile是永久性的环境变量,是全局变量，/etc/profile.d/设置所有用户生效
2. /etc/profile.d/比/etc/profile好维护，不想要什么变量直接删除/etc/profile.d/下对应的shell脚本即可，不用像/etc/profile需要改动此文件
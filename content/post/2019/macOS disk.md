---
title: "mac分区说明"
date: "2019-03-30 23:59:38"
lastMod: "2019-03-30 23:59:38"
categories: ["it"]
tags: ["macOS", "guid分区", "apple分区"]
---

此次重装Mac mini，格式化磁盘时，第一次遇到mac上不同的分区方案，记录下来：

- guid分区和apple分区是启动OS X用的
- guid是启动intel芯片的
- apple是启动power pc芯片的
- 主引导区是启动windows的分区

现在Mac都是是Intel芯片，所以需要选择guid分区
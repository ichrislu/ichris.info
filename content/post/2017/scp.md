---
title: "使用scp传输文件和文件夹"
date: "2017-12-23"
lastMod: "2017-12-23"
categories: ["it"]
tags: ["linux", "scp"]
---

服务器间传输文档，不用再传到Windows跳板机或SSH客户端机器，而是使用scp命令直接在服务器上操作。
比如要把A服务器的/home/root/xxx.zip传输到B服务器的/home/root/目录，
进入A服务器执行：`scp /home/root/xxx.zip root@192.168.200.19:/home/root`  
添加-r参数可以传输文档夹：`/home/root/eyas/ root@192.168.200.19:/home/root`
---
title: "关于Dockerfile"
date: "2018-04-20 12:16:00"
lastMod: "2018-04-20 12:16:00"
categories: ["it"]
tags: ["docker"]
---

1. ARG和ENV
ARG指令定义了用户可以在编译时或者运行时传递的变量，如使用如下命令：--build-arg <varname>=<value>
ENV指令是在dockerfile里面设置环境变量，不能在编译时或运行时传递。
ARG和ENV的有效结合：ARG var ENV var=${var}

2. ENTRYPOINT和CMD的shell form和exec form两种形式
exec form并不会调用shell，也就是说不会执行变量替换，如果需要进行变量替换，可以使用shell form或直接执行shell

如：



- ENTRYPOINT [ "echo", "$HOME" ] 不会执行变量替换
- ENTRYPOINT [ "sh", "-c", "echo $HOME" ]则可以


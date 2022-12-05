---
title: "Docker环境变量"
date: "2018-04-27 10:52:00"
lastMod: "2018-04-27 10:52:00"
categories: ["it"]
tags: ["docker", "环境变量"]
---

1. env_file，environment中定义的环境变量是传给container用的，不是在docker-compose.yml中的环境变量用的
2. docker-compose.yml中的环境变量${VARIABLE:-default}引用的是在.env中定义的或者同个shell export出来的
3. docker-compose中替换.env文件变量的研究，.env文件的环境变量：
   1. 只能用大写
   2. 不能出现"."号
   3. 可以用"_"
   4. 只能用在up -d

后记：不用太过研究，限制多，且应用得少

> 参考
>
> <https://docs.docker.com/compose/environment-variables/>
> <https://docs.docker.com/compose/environment-variables/>
> <https://docs.docker.com/compose/environment-variables/#the-env-file>
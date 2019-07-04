---
title: "修改tomcat容器时区"
date: "2018-05-04 14:53:00"
lastMod: "2018-05-04 14:53:00"
tags: ["docker", "tomcat", "时区"]
---

默认tomcat容器时区为UTC0，与中国东8区相差8小时

会导致某些依赖时间的应用获取时间不正确，如：xxl-job-admin，设置的cron中指定了小时级以上则会不正常执行（小时级以下不涉及具体时区）

解决办法：一般来说宿主机的时区设置都是对的，所以将宿主机时区配置映射至容器，如：

```yml
volumes:
  - /etc/localtime:/etc/localtime:ro
```

此法目前最好，因为不需要重做一次镜像，不需要再修改容器时区，只要保障宿主机时区是正确的即可。

后来发现，部分java程序，如xxl-job-admin，还需要设置JVM时区，否则仍然不正确，如下：

```yml
environment:
  JAVA_OPTS: -Duser.timezone=GMT+08
```

> 参考
> 
> <https://blog.csdn.net/yin138/article/details/52765089>
> <https://my.oschina.net/huawu/blog/4646>
---
title: "Docker常见问题"
date: "2017-07-22 00:57:00"
lastMod: "2017-07-22 00:57:00"
categories: ["it"]
tags: ["docker", "常见问题"]
---

### 以下是实践过程中遇到的一些问题，记下来以防忘记
> - **现象：**Repository ***************** already being pulled by another client. Waiting.
> - **原因：**暂时不清楚
> - **解决办法：**service docker restart或systemctl restart docker

---

> - **现象：**Cannot connect to the Docker daemon. Is 'docker -d' running on this host? 
> - **原因：**经查docker进程不在了，但确定是以服务方式启动的（service docker start或systemctl start docker），有可能是传说中的bug导致Docker退出了吧
> - **解决办法：**启动docker就好了，在var目录未找到相关日志

---

> - **现象：**docker运行容器后agetty进程cpu占用率100%
> - **原因：**据传是因为使用"docker run"运行容器时使用了 "/sbin/init"和"--privileged"参数。
>   使用--privileged的原因是需要启用systemctl，用户systemctl最开始的想法是希望系统启动的时候（CentOS 7），能自动启动服务，但其实有更好办法，不必要这样。
>   URL：<http://welcomeweb.blog.51cto.com/10487763/1735251>
> - **解决办法：**<http://blog.chinaunix.net/uid-26212859-id-5759744.html>

---

> - **现象：**\# docker-compose rm -f inspservice
>   Going to remove std40dev_inspservice_1
>   Removing std40dev_inspservice_1 ... error
>   ERROR: for std40dev_inspservice_1  driver "overlay" failed to remove root filesystem for 32f8c314b798791e30e5fe9ddad26e61a222b06af64d9e4cde0077abb2d08702: remove /eyas/docker/sys/overlay/103a719b500d2f68845551653117260be206d3b07f54101d38d68d484b093995/merged: device or resource busy
> - **原因：**可能是有其他进程占用，最大可能是容器停止的时候并未释放对该容器数据文件的操作句柄
> - **解决办法：**unmount不行，可能是版本不对，找不到对应的mount记录
>   重启docker也不行，重启操作系统解决

---

> - **现象：**当启动的服务过多，或机器负载过重，可能会出现以下错误：
>   ERROR: for std40dev_sysservice_1  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)
>   导致查看日志的shell报错：
>   \# docker logs --tail=100 -f std40dev_inspservice_1
>   error from daemon in stream: Error grabbing logs: EOF
> - **原因：**不明
> - **解决办法：**注销，重新登录就好了，也有可能过一会儿重试就好了
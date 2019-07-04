---
title: "Let's Encrypt - Free SSL/TLS Certificates"
date: "2018-12-15 21:50:00"
lastMod: "2018-12-15 21:50:00"
tags: ["Let's Encrypt", "SSL", "TLS"]
---

### 1. 获取证书：

#### 先停Nginx
```shell
docker stop nginx
```

#### 生成证书
```shell
./certbot-auto certonly --standalone --email chrislu.name@gmail.com -d ichris.info -d www.ichris.info -d api.ichris.info -d console.ichris.info
```

#### 创建符号链接，可选操作，建议做。因为生成证书后，复制证书麻烦，而且每次更新后都需要复制一次；配置指向/etc目录可能会没有权限，符号链接是最简单高效的办法。
```shell
ln -s /etc/letsencrypt/live/[path]/fullchain.pem /[path]/[to]/[web]/[certs]/fullchain.pem
ln -s /etc/letsencrypt/live/[path]/privkey.pem /[path]/[to]/[web]/[certs]/privkey.pem
```

#### 启动Nginx
```shell
docker start nginx
```

### 2. 更新证书：
```shell
certbot-auto renew --pre-hook "docker stop nginx" --post-hook "docker start nginx"
```

### 3. 添加到调度中，每周1凌晨3点检测一次
```shell
0 3 * * 1 /opt/service/certbot-0.28.0/certbot-auto renew --pre-hook "docker stop nginx" --post-hook "docker start nginx"
```

> 参考
> 
> - 基础入门：https://blog.csdn.net/kikajack/article/details/79122701
> - 更新与生成一样会启动一个临时web服务，所以需要加钩子，自动停止nginx释放端口，更新完成后自动启动nginx：https://certbot.eff.org/docs/using.html#renewing-certificates
> - 调度说明：https://www.cnblogs.com/xd502djj/p/4292781.html
> - certbot-auto是letsencrypt-auto的替代品
> https://community.letsencrypt.org/t/upgrading-from-letsencrypt-auto-to-certbot-auto/16807
> https://community.letsencrypt.org/t/should-we-use-certbot-auto-or-letsencrypt-auto/28290
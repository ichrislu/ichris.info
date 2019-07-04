---
title: "docker registry"
date: "2018-07-03 16:59:00"
lastMod: "2018-08-07 16:59:00"
tags: ["docker", "registry", "token认证", "oss"]
---

#### registry和registry-web配合使用Token认证的问题

之前使用的是htpasswd方式，但registry-web需要使用token方式。

其原理很重要，需要搞明白：

* 先生成key和密钥，分别部署于registry和registry-web
* 在registry中配置token认证方式realm地址为registry-web的地址
* 在registry-web中配置的url为registry的api地址


当前测试可用的配置：

##### 生成认证文件：

```shell
openssl req -new -newkey rsa:4096 -days 365 -subj "/CN=localhost" -nodes -x509 -keyout auth.key -out auth.cert
```

##### docker-compose片断：

```yaml
registry:
  image: registry:2.6.2
  volumes:
    \- /data/ops/registry/:/etc/docker/registry/
    \- /data/ops/registry/auth.cert:/etc/docker/registry/auth.cert:ro
    \- /data/ops/registry/config.yml:/etc/docker/registry/config.yml:ro
  restart: unless-stopped
  networks:
    \- ops
registry-web:
  image: hyper/docker-registry-web:v0.1.2
  volumes:
    \- /data/ops/registry-web/db:/data
    \- /data/ops/registry-web/auth.key:/conf/auth.key:ro
    \- /data/ops/registry-web/config.yml:/conf/config.yml:ro
  restart: unless-stopped
  ports:
    \- 8089:8080
  networks:
    \- ops
```

##### registry/config.yml

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  oss:
    accesskeyid: <access-key-id>
    accesskeysecret: <access-key-secret>
    region: <region>
    bucket: <bucket>
    rootdirectory: /
  delete:
    enabled: true
http:
  addr: :8080
  headers:
    X-Content-Type-Options: [nosniff]
    Access-Control-Allow-Headers: ['*']
    Access-Control-Allow-Origin: ['*']
    Access-Control-Allow-Methods: ['GET,POST,PUT,DELETE']
auth:
  token:
    realm: https://registry-web.xxx.com/api/auth
    service: registry:8080
    issuer: 'Chris'
    rootcertbundle: /etc/docker/registry/auth.cert
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

##### registry-web/config.yml

```yaml
registry:
  url: http://registry:8080/v2
  name: registry:8080
  readonly: false
  auth:
    enabled: true
    issuer: 'Chris'
    key: /conf/auth.key
```

> 参考

> 详见官方文档说明：<https://github.com/mkuchin/docker-registry-web>
> 官方示例：<https://github.com/mkuchin/docker-registry-web/tree/master/examples>

**最后，并不推荐再使用registry-web这个项目，已经超过2年未更新，有可能停止维护了！**

---

#### Docker Regitstry With Aliyun OSS

命令：

```shell
docker run -d -p 5000:5000 -p 443:443 --name registry --restart always -v /data/registry/:/etc/docker/registry/ registry:2.6.2
```

完整配置文件：

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  oss:
    accesskeyid: <access-key-id>
    accesskeysecret: <access-key-secret>
    region: <oss-cn-shenzhen>
    bucket: <bucket>
    rootdirectory: /
http:
  addr: :443
  tls:
    certificate: /etc/docker/registry/tls/xxx.pem
    key: /etc/docker/registry/tls/xxx.key
  headers:
    X-Content-Type-Options: [nosniff]
    Access-Control-Allow-Headers: ['*']
    Access-Control-Allow-Origin: ['*']
    Access-Control-Allow-Methods: ['GET,POST,PUT,DELETE']
auth:
  htpasswd:
    realm: Registry Realm
    path: /etc/docker/registry/auth/htpasswd
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

**注：**仅存储用OSS，流量还是走ECS

**坑：**

1. 官方配置说rootdirectory是可选配置，但其实是必须要有这项配置，内容可以为官方说的默认值：/
2. push失败，是因为开启了proxy功能，关闭就好了，具体原因没时间了解：<https://blog.csdn.net/qq_22543991/article/details/78590615>
3. htpasswd实现登录校验的时候，CentOS默认未安装，可以手动安装yum install httpd-tools（内含ab等工具噢！），但不太建议，因为registry镜像中其实已经包含了，可以通过如下完成：docker run --entrypoint htpasswd registry:2 -Bbn admin 123456 > /usr/local/auth/passwd。需要注意：虽然是同样的命令，但在不同的主机，生成的结果并不相同。因此，在主机A上生成的密码文件不能用作主机B上进行认证。不过，当前已确认是在宿主机上生成的密码，可以在容器里生效。<https://segmentfault.com/a/1190000015108428>
4. htpasswd时，不能用官方配置的Realm：basic-realm，而要用：Registry Realm